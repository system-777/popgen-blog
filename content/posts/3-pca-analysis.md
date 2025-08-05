+++
date = '2025-07-29T16:00:00'
draft = false
title = 'Performing PCA on Genetic Data Using PLINK'
tags = ["PLINK", "PCA", "Data Preparation"]
+++

In this post, I’ll demonstrate how to perform a PCA on a PLINK dataset.
Before we begin, we need to prepare a subset of samples we're interested in analyzing.

To do this, we’ll extract sample information from the `.fam` file.
But first, we need to identify the samples of interest. For example, those from a specific population such as Sardinians.

The easiest way is to open the corresponding `.ind` file and look at the population column, which is the third column in each row. Open the file in a text editor like Notepad++, and search for the population name, in this case, Sardinian.

You should find entries like this:
``` Text
        ...
        HGDP01075.HO M Sardinian.HO
        HGDP01076.HO M Sardinian.HO
        HGDP01077.HO M Sardinian.HO
        ...
```
These entries tell us which **IIDs (individual IDs)** belong to Sardinian samples.
Now take note of these IIDs, for example: HGDP01075.HO.

Next, open your `.fam` file (from the previously created PACKEDPED dataset). You’ll see something like:
``` Text
        ...
  1664   HGDP01075.HO 0 0 1 2
  1665   HGDP01076.HO 0 0 1 2
  1666   HGDP01077.HO 0 0 1 2
        ...
```
Now match the IIDs from the `.ind` file to those in the `.fam` file. For each matching line in the `.fam`, copy the **entire line** into a new file. This new file will be used with PLINK's `--keep` flag to retain only the samples of interest.  

**Note:** Since the `.ind` and `.fam` files are row-aligned, if you identify a block of consecutive samples in the `.ind` file belonging to the same population (e.g., 10 Sardinians in a row), you can safely copy the corresponding 10 lines from the `.fam` file without having to check each ID manually.

After selecting a subset of samples of interest — for example, several European populations — and saving them to a file (in this case named `samples`, without a file extension), we can use PLINK to generate a new dataset:
``` bash
plink --bfile data --keep samples --make-bed --out subset
```
This will create a new binary PLINK dataset (`subset.bed`, `.bim`, `.fam`) containing only the selected individuals.

---

### Running PCA
Now we can run the PCA on the subset:
```bash
# --out specifies the output file prefix
# The default number of principal components is 10,
# so "10" could be omitted unless you want to change the number
plink --bfile subset --pca 10 --out pca
```
This will output two files: `pca.eigenval` and `pca.eigenvec`. The `pca.eigenvec` is what you’ll use for plotting or downstream analysis.

To keep only the IIDs and the principal components (and discard FIDs), you can use the following awk one-liner on Linux:
``` bash
awk '{$1=""; sub(/^ /, ""); gsub(/ /, ","); print}' pca.eigenvec > output.csv
```
This will generate a CSV-output file where each line contains a sample ID followed by its principal component values.

The resulting file can be loaded into Python or R for downstream analysis. It’s also compatible with hobbyist tools like Vahaduo — though I personally don’t endorse such platforms, nor do I consider their models accurate or reliable.

In fact, you may observe with your own dataset that distances can vary significantly depending on how many and which samples are included. For example, if your PCA is based only on a regional subset, much more variance will remain **within** that region. As a result, samples may appear more distant from each other compared to what you’d see in global reference PCA datasets like G25. 

---

### Plotting the PCA
The previously created CSV-file can be used to make a PCA-plot. However, since the file contains only sample IDs and principal component values — without population labels — we’ll first generate a separate label file to make the plot easier to interpret. To do this, we’ll match each sample in the `.fam` file to its corresponding population label from the original `.ind` file. The `.ind` file contains the population in the third column. We’ll extract that and output it in the same order as the .fam file, so the labels line up with the PCA values.

Here’s a one-liner using `awk`:
``` bash
awk 'NR==FNR {map[$1]=$3; next} {print map[$2]}' v62.0_HO_public.ind data.fam > labels
```
This will create a file called `labels`, with one population label per line, matching the sample order in your PCA output.

Now you can use a scripting language like **Python** (or alternatively **R**) to load the PCA output and plot it.

Below is an example Python script that reads the eigenvector file (`output.csv`) and the corresponding label file (`labels`), and generates a PCA plot using **seaborn** and **matplotlib**:
``` Python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Load eigenvectors
df = pd.read_csv("output.csv", header=None)
df.rename(columns={0: "ID"}, inplace=True)

# Load labels
with open("labels") as f:
    labels = [line.strip() for line in f]

if len(labels) != df.shape[0]:
    raise ValueError("Mismatch between number of labels and samples")

df["Label"] = labels

# Rename eigenvector columns
for i in range(1, df.shape[1] - 1):
    df.rename(columns={i: f"PC{i}"}, inplace=True)

# Seaborn aesthetics
# Alternatively remove this line for no grid
sns.set(style="whitegrid", context="talk")
fig, ax = plt.subplots(figsize=(12, 7))

# Plot
sns.scatterplot(
    data=df,
    x="PC1",
    y="PC2",
    hue="Label",
    palette="deep",
    s=50,
    edgecolor="black",
    ax=ax
)

# Title and labels
ax.set_title("PCA", fontsize=18)
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")

# Legend outside the plot to the right
ax.legend(
    title="Population",
    bbox_to_anchor=(1.02, 1),
    loc="upper left",
    borderaxespad=0,
    frameon=True
)

# Adjust layout to make room for legend
plt.tight_layout(rect=[0, 0, 0.85, 1])
plt.show()
```

The PCA plot below shows the structure observed in my test subset:
![PCA Plot](/popgen-blog/images/Figure_1.png)
