+++
title = "On Forecasting AIMSR 2025 - Part 1 / N"
date = 2025-04-03T22:21:30+05:30
draft = true
tags = ["experiments"]
categories = ["experiments"]
katex = true
+++

# Data collection and EDA | What does India's rainfall patterns tells us ?

## Data collection



Collect the daily rainfall for each \\(0.25\degree * 0.25\degree\\) grid over India for the time range \\(1901 - 2024\\). The data is obtained from [IMD Pune website](https://www.imdpune.gov.in/cmpg/Griddata/Rainfall_25_NetCDF.html). NetCDF files have been downloaded for each year for each year from 1901 to 2024. We process the NetCDF files to obtain data in the following format.

| year | month | day | lat   | lon   | rainfall (in mm) |
| ---- | ----- | --- | ----- | ----- | ---------------- |
| ...  | ...   | ... | ...   | ...   | ...              |
| 2024 | 06    | 01  | 27.25 | 92.25 | 2.183660         |
| ...  | ...   | ... | ...   | ...   | ...              |



<!-- 

This is a cool way to comment content in markdown

inline latex
\\(e = mc^2\\)

new line latex - approach 01
\\[\int_a^b f(x)\\]

new line latex - approach 02
$$\sum_{x=1}^5 y^z$$ -->


## EDA

### What is the current LTA of AISMR ?

LTA = 30 years average of area-averaged-rainfall received during JJAS months over entire India.

### How has the LTA changed over the years ?

### How is a normal, below normal and above normal monsoon defined ? What are the historical trends on this ?

### What are some homogenous monsoon rainfall clusters in India ?

How does this compare to the 5 clusters defined by IMD ?

What are these cluster regions in India ? 

> How does this relate to the industry, agriculture in these areas ? 

### How does the LTA and rainfall trends looks for each homogenous cluster ?


## Next Steps

The end goal is to forecast the monsoon rainfall for Summer Monsoon Rainfall 2025 (JJAS) for each of these homogenous clusters. This will also give us an estimate of the AISMR 2025 that we can compare and contrast with IMD's bulletin of Long Range Monsoon Forecast generally released during the end of April.


