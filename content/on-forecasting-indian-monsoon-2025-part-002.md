+++
title = "On Forecasting AISMR 2025 - Part 2 / N"
subtitle = "On identifying clusters of regions having homogenous rainfall patterns"
date = 2025-04-09T23:02:42+05:30
draft = false
katex = true
+++

# Introduction

<!-- explain to the readers what is done in this blog post -->
<!-- & what will the readers learn from this blog post. -->

In this post, we will analyse how to identify different clusters in India that have similar rainfall patterns. This analysis is the second step (i.e. 2 / N) to accurately and timely forecast All India Summer Monsoon Rainfall (AISMR) 2025. 


# On identifying clusters of regions having homogenous rainfall patterns

## Methodolgy

We go through a simplified example. Assume we have 9 regions - \\( R_1 ... R_9 \\). The goal is to identify 3 clusters - \\( C_1 ... C_3 \\) - such that each cluster contains regions having similar rainfall patterns. We calculate \\( \rho (R_i, R_j) \\) between all pairwise regions \\( R_i\\) and \\( R_j\\). 

> \\( \rho \\) is defined as [the spearman correlation](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html)


The pairwise correlation scores can be represented as a matrix \\( M \\) where -

\\( M_{ij} = \rho (R_i, R_j) \\). 

We then derive a distance matrix \\( D \\) from correlation matrix \\( M \\) as -

\\( D_{ij} = 1 -  |M_{ij}|\\)

The distance matrix is fed into the [k-medoids clustering algorithm](https://scikit-learn-extra.readthedocs.io/en/stable/generated/sklearn_extra.cluster.KMedoids.html) to obtain the clusters.


{{< 
    figure
    src="/images/20250412-clustering-example.svg"
    alt="The idea behind clustering"
>}}





## Analysis

We apply domain expertise to identify the correct number of clusters.

### NUM_CLUSTERS = 4


{{< 
    figure
    src="/images/4-india-image.png"
>}}

|Cluster|Remarks|
|-|-|
|\\( C_1 \\)|Peninsular Region|
|\\( C_2 \\)|Gangetic Plains|
|\\( C_3 \\)|Northwest India|
|\\( C_4 \\)|Northeast India|

IMD also classifies India into 4 clusters (TODO add reference.) - and we notice their clustering and our clustering is remarkably similar.

> Domain Expertise Remarks: The Western Ghats area receives more rainfall than the Saurashtra region. Similarly the Eastern Ghats and Central India region receive different amounts of rainfall. Hence, we need more granular clusters.

### NUM_CLUSTERS = 5

{{< 
    figure
    src="/images/5-india-image.png"
>}}

|Cluster|Remarks|
|-|-|
|\\( C_1 \\)|Jammu & Kashmir region|
|\\( C_2 \\)|Gangetic Plains & North East India|
|\\( C_3 \\)|Gujarat and Rajasthan (the Saurashtra region)|
|\\( C_4 \\)|Bengal and Orissa region|
|\\( C_5 \\)|Western Ghats and Central Peninsular region|

> Domain Expertise Remarks: The Western Ghats area receives more rainfall than the central peninsular region. Similarly the North East region receives more rainfall than the Gangetic plains. Hence, we need more granular clusters.


### NUM_CLUSTERS = 8


{{< 
    figure
    src="/images/8-india-image.png"
>}}

|Cluster|Remarks|
|-|-|
|\\( C_1 \\)|Jammu & Kashmir & Ladakh region|
|\\( C_2 \\)|Himachal area|
|\\( C_3 \\)|Gangetic Plain|
|\\( C_4 \\)|Northeast India|
|\\( C_5 \\)|Gujarat and Rajasthan|
|\\( C_6 \\)|Central India|
|\\( C_7 \\)|Eastern Ghats and Central Peninsular region|
|\\( C_8 \\)|Western Ghats and Southern Peninsular region|

> Domain Expertise Remarks: This clustering has captured the key trends.


## Results

We proceed with `NUM_CLUSTERS=8` proceeding forward.

For each of the clusters \\( C_1 ... C_8 \\), we present key details.

|Cluster|Geographical Area|LTA (in mm)|Area wise %age|%age rainfall received by India|
|-|-|-|-|-|
|\\( C_1 \\)|Jammu & Kashmir & Ladakh region||||
|\\( C_2 \\)|Himachal area||||
|\\( C_3 \\)|Gangetic Plain||||
|\\( C_4 \\)|Northeast India||||
|\\( C_5 \\)|Gujarat and Rajasthan||||
|\\( C_6 \\)|Central India||||
|\\( C_7 \\)|Eastern Ghats and Central Peninsular region||||
|\\( C_8 \\)|Western Ghats and Southern Peninsular region||||

# Next Steps

The end goal of this analysis is to forecast AISMR for 2025 and compare the same with IMD's bulletin of Long Range Monsoon Forecast generally released during the end of April. Next, we will look into drivers of Indian Monsoon - the ENSO and IOD phenomena.


