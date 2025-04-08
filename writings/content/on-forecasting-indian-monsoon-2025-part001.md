+++
title = "On Forecasting AISMR 2025 - Part 1 / N"
date = 2025-04-03T22:21:30+05:30
draft = false
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

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}
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

{{< /details >}}

Once the NetCDF files have been downloaded for each year from 1901 to 2024. We process the NetCDF files to obtain data in the following tabular format and load the same in a local SQLite table for future use.


| year | month | day | lat   | lon   | rainfall (in mm) |
| ---- | ----- | --- | ----- | ----- | ---------------- |
| ...  | ...   | ... | ...   | ...   | ...              |
| 2024 | 06    | 01  | 27.25 | 92.25 | 2.183660         |
| ...  | ...   | ... | ...   | ...   | ...              |


# Exploratory Data Analysis

## How many distinct \\(0.25\degree * 0.25\degree\\) grid points cover the Indian landmass ?

There are \\(4964\\) grid points - each \\(0.25\degree * 0.25\degree\\) in size covering the Indian landmass.

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}
import sqlite3
import pandas as pd
import numpy as np

conn = sqlite3.connect('rainfall.db')
cursor = conn.cursor()

query = "SELECT distinct lat, lon from rainfall"
cursor.execute(query)

rows = cursor.fetchall()
df = pd.DataFrame(rows, columns=np.array(cursor.description)[:, 0])
print(df.shape)
# (4964, 2)
conn.close()
```

{{< /details >}}


## How much rainfall did India receive over the course of 2024 ?

The entire landmass of India recived a total of \\(6074999.43\ mm\\) of rainfall during 2024. On dividing this by \\(4964\\) - the number of grid points covering the Indian landmass - each grid point received roughly \\(1223.81\ mm\\) of rainfall.

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}
import sqlite3
import pandas as pd
import numpy as np
import duckdb

conn = sqlite3.connect('rainfall.db')
cursor = conn.cursor()
query = "SELECT * from rainfall where substr(date, 1, 4) = '2024'"
cursor.execute(query)
df = pd.DataFrame(rows, columns=np.array(cursor.description)[:, 0])
conn.close()

agg_df = duckdb.query("select date, sum(rf::DOUBLE) as sum_rf from df group by date order by date").df()

# rainfall over India 2024
print(agg_df['sum_rf'].sum())
# 6074999.43
print(agg_df['sum_rf'].sum() / 4964)
# 1223.81
```
{{< /details >}}


## How much rainfall did India receive over the course of Summer Monsoon 2024 ?

The Summer Monsoon 2024 spans over 4 months:
- June
- July
- August
- September

Over the course of these 4 months - India received \\(4701855.98\ mm\\) of rainfall. The area averaged rainfall is \\(947.19\ mm\\). [IMD's official press release](https://www.imdpune.gov.in/latestnews/features_10_2024.pdf) states the area averaged rainfall is \\(934.8\ mm\\). Thus our calculations are close enough.

> The area averaged rainfall received during June, July, August and September months is defined as All India Summer Monsoon Rainfall (AISMR). Thus, the AISMR 2024 is \\(947.19\ mm\\).

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}
import sqlite3
import pandas as pd
import numpy as np
import duckdb

conn = sqlite3.connect('rainfall.db')
cursor = conn.cursor()
query = "SELECT * from rainfall where substr(date, 1, 4) = '2024'"
cursor.execute(query)
df = pd.DataFrame(rows, columns=np.array(cursor.description)[:, 0])
conn.close()

agg_df = duckdb.query("select date, sum(rf::DOUBLE) as sum_rf from df group by date order by date").df()

# rainfall over India JJAS 2024
jjas_agg_df = duckdb.query("select date, sum_rf from agg_df where substr(date, 6, 2) in ('06', '07', '08', '09')").df()

print(jjas_agg_df['sum_rf'].sum())
# 4701855.98
print(jjas_agg_df['sum_rf'].sum() / 4964)
# 947.19
```
{{< /details >}}


> India received 77.4 % of annual rainfall volume during the Summer Monsoon period. This highlights the importance of Indian Monsoon to India's agricultural, industrial and economic activity.

## What is the Long Period Average (LPA) of AISMR ?

[IMD Monsoon FAQs](https://mausam.imd.gov.in/imd_latest/monsoonfaq.pdf) defines LPA as the 50 year time-average of AISMR from 1961 to 2010. IMD's current LPA definition is :

$$
LPA := 880.6\ mm
$$

We calculate the LPA as:

$$
LPA = \frac {\sum_{t=1961}^{2010}{AISMR_{t}}} {50} = 850.37\ mm 
$$

Our calculations are close enough.

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}
import sqlite3
import pandas as pd
import numpy as np
import duckdb

conn = sqlite3.connect('rainfall.db')
cursor = conn.cursor()

query = """
SELECT CAST(SUBSTR(date, 6, 2) AS INT) as month, CAST(SUBSTR(date, 1, 4) AS INT) as year, CAST(rf as DOUBLE) as rf 
from rainfall
where month in (6, 7, 8, 9)
and year >= 1961 
and year <= 2010
"""

cursor.execute(query)

rows = cursor.fetchall()
df = pd.DataFrame(rows, columns=np.array(cursor.description)[:, 0])
conn.close()

jjas_agg_df = duckdb.query("select month, year, sum(rf) / 4964 as rf from df group by month, year order by year, month").df()

agg_df = duckdb.query("select year, sum(rf) as rf from jjas_agg_df group by year order by year").df()

print(agg_df['rf'].mean())
# 850.37
```
{{< /details >}}


> Remarks: The LPA definition of IMD is inconsistent. The [IMD Monsoon FAQs](https://mausam.imd.gov.in/imd_latest/monsoonfaq.pdf) has specified it as 880.6 mm while the  [IMD 2024 monsoon report](https://www.imdpune.gov.in/latestnews/features_10_2024.pdf) has calculated it as 868.6 mm. To stay away from such inconsistencies, we define the Long Term Average (LTA) of AISMR and use the same going forward in this analysis.


### What is Long Term Average (LTA) of AISMR ? 

The LTA calculated during year T is defined as the time-averaged AISMR over a 30 years timeframe.

$$
LTA_T = \frac {\sum_{t=T-30}^{T-1}{AISMR_{t}}} {30} 
$$

Thus, 

$$
LTA_{2025} = \frac {\sum_{t=1995}^{2024}{AISMR_{t}}} {30} 
$$

and 

$$
LTA_{2024} = \frac {\sum_{t=1994}^{2023}{AISMR_{t}}} {30} = 
$$

{{< details summary="See the details" >}}

```python {linenos=inline hl_lines=[] style=emacs}

```
{{< /details >}}

> The AISMR for 2024 is 947.19 mm while the LTA calculated in 2024 is 853.16. Thus, 2024 received 11 % higher rainfall than LTA.

### How has AISMR and LTA changed over the decades ?

> IMD's categories the monsoon each year as Normal, Above Normal or Below Normal. However, similar to LPA, their classification is inconsistent. To stay away from such inconsistencies, we define:
> 
> **Normal Rainfall** \\( \coloneqq  AISMR_T \in (95, 105 ) \\) % of \\( LTA_T \\)
>
> **Above Normal Rainfall** \\( \coloneqq \\) \\( AISMR_T \\) \\(\\ge \\) 105 %  \\( LTA_T \\)
>
> **Below Normal Rainfall** \\( \coloneqq \\) \\( AISMR_T \\)  \\(\leq \\) 95 %  \\( LTA_T \\)

| year | AISMR (in mm) | LTA (in mm) | status     |
| ---: | ------------: | ----------: | ---------: |
| 1951 |       709.901 |      883.83 | below ❌   |
| 1952 |       799.627 |     877.977 | below ❌   |
| 1953 |       913.848 |      875.13 | normal 🟡 |
| 1954 |       887.135 |     877.951 | normal 🟡 |
| 1955 |       913.362 |     877.493 | normal 🟡 |
| 1956 |       934.827 |     880.618 | above ✅   |
| 1957 |       814.743 |     881.807 | below ❌   |
| 1958 |       928.126 |     879.984 | above ✅   |
| 1959 |        957.38 |     884.802 | above ✅   |
| 1960 |       852.253 |     889.407 | normal 🟡 |
| 1961 |       989.018 |     890.265 | above ✅   |
| 1962 |       854.456 |     892.713 | normal 🟡 |
| 1963 |       834.031 |     892.904 | below ❌   |
| 1964 |       935.972 |     888.047 | above ✅   |
| 1965 |       711.559 |      888.38 | below ❌   |
| 1966 |       758.374 |     882.896 | below ❌   |
| 1967 |       854.244 |     877.015 | normal 🟡 |
| 1968 |       754.356 |     876.872 | below ❌   |
| 1969 |       823.993 |     870.622 | below ❌   |
| 1970 |       949.253 |     871.168 | above ✅   |
| 1971 |       886.678 |     874.514 | normal 🟡 |
| 1972 |       661.302 |     878.578 | below ❌   |
| 1973 |       904.052 |     868.536 | normal 🟡 |
| 1974 |        804.69 |     868.866 | below ❌   |
| 1975 |       994.829 |     864.758 | above ✅   |
| 1976 |       886.949 |     867.307 | normal 🟡 |
| 1977 |       885.147 |     866.098 | normal 🟡 |
| 1978 |       922.309 |     864.344 | above ✅   |
| 1979 |       709.393 |     864.793 | below ❌   |
| 1980 |        920.51 |     857.751 | above ✅   |
| 1981 |       863.571 |     858.411 | normal 🟡 |
| 1982 |       760.289 |     863.533 | below ❌   |
| 1983 |       963.374 |     862.222 | above ✅   |
| 1984 |       863.582 |     863.873 | normal 🟡 |
| 1985 |       806.406 |     863.087 | below ❌   |
| 1986 |       769.907 |     859.522 | below ❌   |
| 1987 |       743.828 |     854.025 | below ❌   |
| 1988 |       1008.34 |     851.661 | above ✅   |
| 1989 |       864.364 |     854.335 | normal 🟡 |
| 1990 |       931.193 |     851.234 | above ✅   |
| 1991 |       834.961 |     853.866 | normal 🟡 |
| 1992 |       825.259 |     848.731 | normal 🟡 |
| 1993 |       872.294 |     847.757 | normal 🟡 |
| 1994 |       945.225 |     849.033 | above ✅   |
| 1995 |       888.767 |     849.341 | normal 🟡 |
| 1996 |        852.59 |     855.248 | normal 🟡 |
| 1997 |       848.355 |     858.389 | normal 🟡 |
| 1998 |       878.207 |     858.192 | normal 🟡 |
| 1999 |       803.414 |     862.321 | below ❌   |
| 2000 |       783.602 |     861.635 | below ❌   |
| 2001 |       799.312 |     856.113 | below ❌   |
| 2002 |       693.952 |     853.201 | below ❌   |
| 2003 |       894.261 |     854.289 | normal 🟡 |
| 2004 |       762.101 |     853.963 | below ❌   |
| 2005 |        866.43 |     852.543 | normal 🟡 |
| 2006 |       882.929 |     848.263 | normal 🟡 |
| 2007 |       933.815 |     848.129 | above ✅   |
| 2008 |       890.284 |     849.751 | normal 🟡 |
| 2009 |       725.103 |     848.684 | below ❌   |
| 2010 |       921.684 |     849.208 | above ✅   |
| 2011 |         920.5 |     849.247 | above ✅   |
| 2012 |        831.08 |     851.144 | normal 🟡 |
| 2013 |       926.049 |     853.504 | above ✅   |
| 2014 |       766.994 |      852.26 | below ❌   |
| 2015 |       763.798 |      849.04 | below ❌   |
| 2016 |       859.043 |      847.62 | normal 🟡 |
| 2017 |       846.442 |     850.591 | normal 🟡 |
| 2018 |       796.853 |     854.012 | below ❌   |
| 2019 |       960.585 |     846.962 | above ✅   |
| 2020 |       956.956 |     850.169 | above ✅   |
| 2021 |       865.392 |     851.028 | normal 🟡 |
| 2022 |       919.938 |     852.042 | above ✅   |
| 2023 |       811.267 |     855.198 | below ❌   |
| 2024 |       947.191 |     853.164 | above ✅   |

> Interesting: the LTA has been decreasing over the decades. From ~883 mm in 1951 to ~853 mm in 2024.

> The LTA for 2025 is 853.24 mm


# Next Steps

The end goal of this analysis is to forecast AISMR for 2025 and compare the same with IMD's bulletin of Long Range Monsoon Forecast generally released during the end of April.


