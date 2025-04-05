+++
title = "On Forecasting AIMSR 2025 - Part 1 / N"
date = 2025-04-03T22:21:30+05:30
draft = true
tags = ["experiments"]
categories = ["experiments"]
katex = true
+++

# Introduction

<!-- explain to the readers what is done in this blog post -->
<!-- & what will the readers learn from this blog post. -->
In this post, we will analyse India's historic monsoon rainfall patterns. This analysis is the first step (i.e. 1 / N) to accurately and timely forecast All India Summer Monsoon Rainfall (AISMR) 2025 - and compare it with IMD's official forecast released in May 2025.


# Data Collection

<!-- what data -->
We require India's historic rainfall data. [IMD Pune](https://www.imdpune.gov.in/cmpg/Griddata/Rainfall_25_NetCDF.html) provides us with daily rainfall data over each \\(0.25\degree * 0.25\degree\\) grid over India for the time range \\(1901 - 2024\\).

<!-- getting a sense of 0.25 X 0.25 -->
How big is a \\(0.25\degree * 0.25\degree\\) grid ? Let's get a sense by overlaying the map of Delhi and the map of Bangalore on such a grid.

{{< 
    figure
    src="/images/20250405-delhi-0.25-0.25-grid-ss.png"
    caption="A snapshot of Delhi's map on top 0f \\(0.25\degree * 0.25\degree\\) grid"
    alt="A snapshot of Delhi's map"
>}}

{{< 
    figure
    src="/images/20250405-bangalore-0.25-0.25-grid-ss.png"
    caption="A snapshot of Bangalore's map on top 0f \\(0.25\degree * 0.25\degree\\) grid"
    alt="A snapshot of Bangalore's map on top 0f \\(0.25\degree * 0.25\degree\\) grid"
>}}

For each year from 1901 to 2024, IMD has provided the daily rainfall data over each Indian grid point as a NetCDF file. On their website we can downloaded the NetCDF file yearwise. 

{{< 
    figure
    src="/images/image.png"
    alt="A snapshot of IMD Pune data portal"
>}}

How to download the 124 NetCDF data from [IMD Pune's webpage](https://www.imdpune.gov.in/cmpg/Griddata/Rainfall_25_NetCDF.html) without manually clicking download button 124 times ? The following python script can be used to automate the download process.


```python
import requests
from multiprocessing import Pool, cpu_count

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Accept-Language': 'en-GB,en-US;q=0.9,en;q=0.8,hi;q=0.7',
    'Cache-Control': 'max-age=0',
    'Connection': 'keep-alive',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'https://www.imdpune.gov.in',
    'Referer': 'https://www.imdpune.gov.in/',
    'Sec-Fetch-Dest': 'document',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-Site': 'same-origin',
    'Sec-Fetch-User': '?1',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Mobile Safari/537.36',
    'sec-ch-ua': '"Chromium";v="134", "Not:A-Brand";v="24", "Google Chrome";v="134"',
    'sec-ch-ua-mobile': '?1',
    'sec-ch-ua-platform': '"Android"',
}

def download_file(year):
    data = {
        'RF25': str(year),
    }

    response = requests.post('https://www.imdpune.gov.in/cmpg/Griddata/RF25.php', headers=headers, data=data, stream=True)
    filename = f"data/{str(year)}.nc"
    print(filename)
    if response.status_code == 200:
        with open(filename, "wb") as file:  # Open file in binary write mode
            for chunk in response.iter_content(chunk_size=8192):  # Read in chunks
                file.write(chunk)
        print("File downloaded successfully")
    else:
        print(f"Failed to download file: {response.status_code}")


def main():
    years = list(range(1901, 2025))  # List of years
    
    # Use a pool of worker processes to download files in parallel
    num_workers = min(cpu_count(), len(years))  # Use available CPU cores
    with Pool(processes=num_workers) as pool:
        pool.map(download_file, years)  # Distribute work across processes

if __name__ == '__main__':
    main()
```



Once the NetCDF files have been downloaded for each year from 1901 to 2024. We process the NetCDF files to obtain data in the following tabular format and load the same in a local SQLite table for future use.


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


# Exploratory Data Analysis

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


