+++
date = '2025-08-05T13:21:16+02:00'
draft = false
title = 'Running ADMIXTURE in Supervised Mode'
+++
This post is a short follow-up to the previous one on [Estimating Ancestry Components Using ADMIXTURE]({{< relref "posts/5-unsupervised-admixture.md" >}}). Here, we’ll explore supervised ADMIXTURE, a mode that allows you to explicitly define ancestral populations and infer the ancestry proportions of unassigned individuals based on those references.

---

### What Is a Supervised Run?
In supervised mode, ADMIXTURE skips the component discovery step and instead uses **user-defined groupings** to represent ancestral components. The benefit: if you already have solid candidates for reference populations, you can use them to quickly infer ancestry proportions for target or admixed individuals.

Supervised runs are **much faster**, especially with large datasets, since ADMIXTURE no longer has to determine the internal structure of each cluster.

---

### Creating a `.pop` File
Supervised ADMIXTURE requires an additional file with the extension `.pop`. This file must:
* Have the exact same prefix as your .bed input file.
* Contain the same number of rows as your .fam file (one row per sample).
* Use labels (strings or integers) to assign each individual to a known ancestral population.
* Use `-` (or leave the line empty) for individuals whose ancestry you want ADMIXTURE to infer.

For example, given a dataset `admix.final.bed`, create a file named `admix.final.pop`:

``` text
Northern West Asia
Northern West Asia
Northern West Asia
-
Northern West Asia
```

In this example, the first three and the fifth samples are assigned to a component labeled Northern_West_Asia. The fourth sample (`-`) is unassigned and will be inferred based on the others.
You can define as many distinct labels as components you want ADMIXTURE to model (Ks). The number of unique labels in the `.pop` file determines K, the number of ancestral populations you’ll specify in the run.

---

### Running ADMIXTURE in Supervised Mode
Once your `.pop` file is ready, you can start the supervised run with:
``` bash
# Assuming you used six distinct populations

./admixture32 --supervised admix.final.bed 6
```
ADMIXTURE will then use the population labels from your `.pop` file to build the ancestral components and infer proportions for unassigned individuals.

---

### When to Use Supervised Mode
When to Use Supervised Mode

Supervised ADMIXTURE is especially useful when:
* You have high-confidence reference populations (ancient samples or unadmixed modern groups).
* You want to test how certain samples relate to predefined clusters.
* You want faster runs compared to iterative unsupervised testing.

It’s important to note that component quality is entirely in your hands here. ADMIXTURE won’t "fix" poor references. Ideally, your reference populations should be homogeneous, representative, and contain many individuals.

---

### Visualizing Results
The output files are the same as in unsupervised runs:
* `.Q`: Ancestry proportions per individual.
* `.P`: Allele frequencies per component.

You can reuse the same plotting script from the previous post. Since your `.pop` labels now directly correspond to components, interpreting the bar plot becomes more straightforward.