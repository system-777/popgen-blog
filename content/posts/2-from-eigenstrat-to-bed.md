+++
date = '2025-07-29T15:30:00'
draft = false
title = 'From EIGENSTRAT to PACKEDPED'
tags = ["EIGENSTRAT", "PACKEDPED", "PLINK", "convertf", "Data Processing"]
+++

The files downloaded in the previous blog post are in **EIGENSTRAT** format. In this post, we’ll look at how to convert them to **PACKEDPED** format.

PACKEDPED format allows for easier downstream processing using the **PLINK** toolset. With PLINK, it becomes straightforward to extract sample subsets, filter SNPs, and perform a wide range of analyses.

---

### Downloading PLINK

I use [PLINK 1.9](https://www.cog-genomics.org/plink/). While there is a newer version (2.0), I prefer 1.9 because it includes several features that were deprecated or removed in the newer release.

Choose and download the binary suitable for your operating system.  
If you're on Linux or WSL, you can make the binary globally accessible like this:

```bash
# Make PLINK globally accessible
# Run this from the directory where you downloaded the binary
sudo cp plink /usr/local/bin/

# Test if PLINK works
plink
```

---

### Compiling EIGENSOFT
Before we can use PLINK on the downloaded dataset, we have to convert it to PACKEDPED format. We will use the convertf tool from the EIGENSOFT toolset. 

#### Install Dependencies
First we install the required dependencies:

``` bash
sudo apt update
sudo apt install -y build-essential gfortran liblapack-dev liblapacke-dev libgsl-dev
sudo apt install -y libopenblas-dev
```
#### Download and Compile EIGENSOFT
Then clone the GitHub repository, enter the `src` directory, and compile:
``` bash
# Install git first with: sudo apt install git

git clone https://github.com/DReichLab/EIG
cd EIG/src
LDLIBS="-llapacke" make
```
This will generate several necessary binaries. The most relevant ones are: `convertf` and `mergeit` in the `src` folder, and `smartpca` in the `eigensrc` subdirectory. You can confirm the files were compiled with:

``` bash
ls
```
##### Make tools globally accessible
To use `convertf` and `mergeit` from anywhere, move them to a global location like /usr/local/bin/:

``` bash
sudo cp convertf mergeit /usr/local/bin/
```
#### Accessing  smartpca
If the build was successful, you’ll find `smartpca` in the eigensrc directory:

``` bash
# Navigate into the eigensrc folder
cd ~/EIG/src/eigensrc

# Check files in folder
ls
```
Make it globally accessible:

``` bash
sudo cp smartpca /usr/local/bin/
```
I will provide examples on using smartpca for principal component analysis (PCA) in another post. Since it was compiled alongside the other EIGENSOFT tools, I’m just mentioning it here for completeness.

---

### Converting from EIGENSTRAT to PACKEDPED
Now we can convert the EIGENSTRAT dataset to PACKEDPED. Navigate into the folder or directory you store the dataset. Then with a text editor of your choice generate a file with the following content (adjust the input prefixes to those of your dataset):
``` text
genotypename:    v62.0_HO_public.geno 
snpname:         v62.0_HO_public.snp 
indivname:       v62.0_HO_public.ind 
outputformat:    PACKEDPED 
genotypeoutname: data.bed 
snpoutname:      data.bim 
indivoutname:    data.fam
```

Save the file with any name you like, I named mine simply `parameter` (no file extension). This file tells convertf what to do. Start the conversion with:
``` bash
convertf -p parameter
```
The process may take a while, as the dataset is large.

Once complete, you’ll have a set of PLINK-compatible binary files: `data.bed`, `data.bim`, and `data.fam`, ready for downstream processing.