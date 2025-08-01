+++
date = '2025-07-29T15:00:00'
draft = false
title = 'Getting Started with Population Genetic Analysis'
tags = ["Data Preparation", "EIGENSTRAT"]
+++

A Linux environment is unavoidable when it comes to bioinformatical data processing and preparation. You can use your favorite distribution.  

For Windows users, the **Windows Subsystem for Linux (WSL)** provides a good alternative to dual booting or setting up a full virtual machine.

### Installing WSL with Debian

Open **PowerShell as Administrator** and run:

```bash
wsl --install -d Debian
```

Once installed, update the system:
``` bash
sudo apt update && sudo apt upgrade -y
```

### Downloading A Genetic Dataset
Next, you need to obtain a genetic dataset. A good and comprehensive resource is the [Allen Ancient DNA Resource (AADR)](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/FFIDCW). 

If you're only interested in ancient samples, download the following three files from the AADR:  
`v62.0_1240k_public.geno`, `.snp`, and `.ind`.  
(If you want information on sample origins, you should also download the corresponding `.anno` file.)

If you'd like to include modern samples as well, which can be useful for personal genetic comparisons, download the same file types, but with the prefix `v62.0_HO_public`.

Since these files are large, it's best to download them using `wget` or `curl` (both over Linux) from the direct download links. This avoids browser interruptions.

#### Example using `wget`:

```bash
# Install wget
sudo apt install wget

# Example for the .geno
wget -O v62.0_HO_public.geno "https://dataverse.harvard.edu/api/access/datafile/10537419"
