+++
date = '2025-07-30T20:47:41+02:00'
draft = false
title = 'Performing PCA on Genetic Data Using Smartpca'
tags = ["PLINK", "PCA", "smartpca", "convertf", "Data Conversion"]
+++

This post is a continuation of the previous one, where I demonstrated how to perform PCA with PLINK. While PLINK’s PCA is great for quick, exploratory analysis, **smartpca** (part of the EIGENSOFT toolset) is more commonly used in published genetic studies.

Smartpca needs to be compiled on Linux or macOS. I covered how to install and prepare the toolchain on Linux in this earlier post: [From EIGENSTRAT to PACKEDPED]({{< relref "posts/2-from-eigenstrat-to-bed.md" >}}).

As before, I’ll use a small subset. The focus here is on the technical process, not on interpreting the results. One key difference in this post is that I’ll perform **Linkage Disequilibrium (LD) pruning**, which helps reduce SNP redundancy and improves the detection of population structure in PCA.

---

### LD Pruning

``` bash
# Window size: 50 SNPs
# Step size: 5 SNPs
# LD threshold: r² > 0.2 will be pruned
plink --bfile input --indep-pairwise 50 5 0.2

# Extract the pruned SNPs into a new dataset
plink --bfile input --extract plink.prune.in --make-bed --out pruned_data
```

**Note:** If you’re working with low-coverage ancient samples, applying LD pruning directly to a mixed dataset (modern + ancient) may introduce bias, since SNP retention will favor samples with higher coverage. A better approach:
1. Prune only the modern dataset.
2. Use the same pruned SNP list (`plink.prune.in`) to extract SNPs from the ancient dataset.
3. Merge both datasets.

Here’s how you can do that:

``` bash
# Step 1: Prune modern samples
plink --bfile modern --indep-pairwise 50 5 0.2 

# Step 2: Create pruned datasets for both groups
plink --bfile modern --extract plink.prune.in --make-bed --out pruned_modern
plink --bfile ancient --extract plink.prune.in --make-bed --out ancient_extracted

# Step 3: Merge datasets
plink --bfile pruned_modern --bmerge ancient_extracted --make-bed --out final
```

---

### Converting the Dataset to EIGENSTRAT Format
Smartpca requires data in EIGENSTRAT format. Assuming your merged/pruned data is in PACKEDPED (`.bed/.bim/.fam`) format, you can use convertf (part of EIGENSOFT) to convert it.

Create a parameter file named `convert` (or a name of your choice, adjust the parameters accordingly) with the following content:
```text
genotypename:    final.bed 
snpname:         final.bim 
indivname:       final.fam 
outputformat:    EIGENSTRAT 
genotypeoutname: pca.geno 
snpoutname:      pca.snp 
indivoutname:    pca.ind
````

Now run

```bash
convertf -p convert 
```

This will generate the three required input files for smartpca: `pca.geno`, `pca.snp`, and `pca.ind`.

---

### Creating the Smartpca Parameter File
Like most Reich Lab tools, smartpca uses a parameter file to specify input and output options. Here's a minimal example:
``` Text
genotypename:     pca.geno
snpname:          pca.snp
indivname:        pca.ind
evecoutname:      pca.evec
evaloutname:      pca.eval

altnormstyle:     NO 
numoutevec:       10
numoutliter:      0
lsqproject:       YES
numthreads:       5
```
`lsqproject: YES` enables projection mode, meaning projected samples don’t influence the PCA axes. In this example, all samples are high-coverage modern individuals, so projection isn’t technically needed—but I demonstrate it here for completeness. Save the parameter file with a name like `pca_param` (no extension).

To project samples, you’ll need to modify the `.ind` file of your EIGENSTRAT dataset. Change the third column from `Case` to `P`:
``` Text
      ...
      2032:TLA018.HO M       Case
      2033:TLA019.HO M       P
      2034:TLA020.HO M       Case
      ...
```

--- 

### Running Smartpca
Now we run smartpca with:
``` bash
smartpca -p pca_param
```

This generates two files: `pca.eigenval` (lists how much variance each eigenvector explains) and `pca.evec` (contains sample IDs and principal component values).

---

### Preparing for Plotting
To make the output compatible with the plotting script from the previous post, convert the `.evec` file to CSV:
``` bash
awk '{ $NF=""; sub(/[ \t]+$/, ""); gsub(/[ \t]+/, ","); print }' pca.evec > smartpca.csv
```
Then, open the resulting file and manually delete the first line (which begins with `#eigvals`).

---

### Final Plot
After loading the cleaned CSV into the plotting script, this is the resulting plot for my subset:
![PCA Plot](/popgen-blog/images/Figure_2.png)
