# NYC Airbnb 2019 – Data Quality Assessment & Exploratory Data Analysis

A Python project that audits and explores the **New York City Airbnb Open Data (2019)** dataset in two stages:

| Stage | Goal | Output |
|-------|------|--------|
| **Task 1 – Data Quality Assessment** | Find missing values, duplicates, incorrect data types and inconsistent values, then clean the data | `data_quality_check.py`, `AB_NYC_2019_cleaned.csv` |
| **Task 2 – Exploratory Data Analysis (EDA)** | Calculate statistics, find trends, patterns and anomalies, and explain the insights with charts | `NYC_Airbnb_EDA.ipynb`, `eda_analysis.py`, `eda_charts/`, `eda_tables/` |

---

## Table of Contents

1. [Dataset](#dataset)
2. [Project Structure](#project-structure)
3. [Requirements](#requirements)
4. [Task 1 – Data Quality Assessment](#task-1--data-quality-assessment)
5. [Task 2 – Exploratory Data Analysis](#task-2--exploratory-data-analysis)
6. [Charts](#charts)
7. [Key Insights](#key-insights)
8. [How to Run](#how-to-run)
9. [Limitations](#limitations)
10. [Next Steps](#next-steps)
11. [Data Source](#data-source)
12. [License](#license)

---

## Dataset

| Property | Value |
|----------|-------|
| Raw file | `AB_NYC_2019.csv` |
| Rows (raw) | 48,895 |
| Rows (cleaned) | 48,884 (11 listings with `price = 0` removed) |
| Columns | 16 |
| Scope | Airbnb listings across the five NYC boroughs (2019) |

### Column Dictionary

| Column | Description | Expected Type |
|--------|-------------|---------------|
| `id` | Unique listing ID | int |
| `name` | Listing title | string |
| `host_id` | Unique host ID | int |
| `host_name` | Host's first name | string |
| `neighbourhood_group` | Borough (Manhattan, Brooklyn, Queens, Bronx, Staten Island) | category |
| `neighbourhood` | Neighbourhood within the borough | category |
| `latitude` / `longitude` | Listing coordinates | float |
| `room_type` | Entire home/apt, Private room, Shared room | category |
| `price` | Nightly price (USD) | int |
| `minimum_nights` | Minimum nights per booking | int |
| `number_of_reviews` | Total reviews received | int |
| `last_review` | Date of most recent review | datetime |
| `reviews_per_month` | Average reviews per month | float |
| `calculated_host_listings_count` | Number of listings held by the host | int |
| `availability_365` | Days available in the next 365 days | int |

---

## Project Structure

```
.
├── AB_NYC_2019.csv                # Raw dataset
├── AB_NYC_2019_cleaned.csv        # Cleaned dataset (output of Task 1)
├── data_quality_check.py          # Task 1: data quality audit script
├── NYC_Airbnb_EDA.ipynb           # Task 2: EDA notebook (cell-by-cell, charts inline)
├── eda_analysis.py                # Task 2: EDA as a single script
├── eda_charts/                    # 12 charts produced by the EDA
│   ├── 01_price_distribution.png
│   ├── 02_listings_by_borough_roomtype.png
│   ├── 03_median_price_borough_roomtype.png
│   ├── 04_price_boxplot_borough.png
│   ├── 05_top_neighbourhoods.png
│   ├── 06_correlation_heatmap.png
│   ├── 07_availability.png
│   ├── 08_price_map.png
│   ├── 09_review_timeline.png
│   ├── 10_minimum_nights.png
│   ├── 11_top_hosts.png
│   └── 12_outlier_boxplots.png
├── eda_tables/                    # Summary tables and written insights
│   ├── descriptive_statistics.csv
│   ├── borough_stats.csv
│   ├── room_type_stats.csv
│   ├── median_price_borough_roomtype.csv
│   ├── neighbourhood_stats.csv
│   ├── minimum_stay_stats.csv
│   └── insights.txt
└── README.md
```

---

## Requirements

- Python 3.8+
- pandas
- numpy
- matplotlib
- seaborn
- jupyter (only for the notebook)
- xlrd (only if your cleaned file is a genuine Excel 97-2003 `.xls`)

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

---

## Task 1 – Data Quality Assessment

`data_quality_check.py` prints a report covering four checks, then applies the cleaning steps and saves `AB_NYC_2019_cleaned.csv`.

### 1. Missing Values
- Null count and percentage per column
- Hidden missing values (blank strings or placeholders such as `N/A`, `unknown`)
- Whether missing `reviews_per_month` is linked to `number_of_reviews == 0`

### 2. Duplicate Records
- Fully duplicated rows
- Duplicate listing `id`s
- Duplicates ignoring `id`
- Duplicates on `host_id` + `name` + `latitude` + `longitude`

### 3. Incorrect Data Types
- Compares actual vs. expected dtypes for every column
- Tests whether `last_review` parses as a valid date
- Converts `last_review` to datetime and categorical columns to `category`

### 4. Inconsistent Values
- Categorical label consistency (`neighbourhood_group`, `room_type`)
- Leading/trailing spaces, double spaces, and case variants in text columns
- Neighbourhoods mapped to more than one borough
- Impossible or suspicious numeric values (zero prices, `minimum_nights` > 365, availability outside 0–365, coordinates outside NYC)
- Cross-column logic (e.g., reviews = 0 but a `last_review` date exists)
- Non-standard characters in names
- IQR-based outlier counts

### Data Quality Findings

| Category | Finding |
|----------|---------|
| **Missing values** | `name`: 16 · `host_name`: 21 · `last_review`: 10,052 · `reviews_per_month`: 10,052 |
| **Missing value cause** | All 10,052 missing review fields belong to listings with 0 reviews (structural, not errors) |
| **Duplicates** | No fully duplicated rows and no repeated `id` values |
| **Data types** | `last_review` is stored as text and should be `datetime`; categorical columns can be `category` |
| **Inconsistent values** | 238 listing names with leading/trailing whitespace · 11 listings with `price = 0` · 14 listings with `minimum_nights` > 365 (max 1,250) · prices up to $10,000 |
| **Borough / room type labels** | Clean (5 boroughs, 3 room types) |
| **Worth reviewing** | 17,533 listings have `availability_365 = 0` (plausible if the host blocked the calendar) |

### Cleaning Steps Applied

| Issue | Action |
|-------|--------|
| Missing `name` / `host_name` | Filled with `"Unknown"` |
| Missing `reviews_per_month` | Filled with `0` |
| Missing `last_review` | Kept as `NaT` (no reviews exist) |
| `last_review` as string | Converted to datetime |
| Whitespace in text | Removed with `str.strip()` |
| `price = 0` | Rows dropped (11 rows) |
| Extreme `minimum_nights` / `price` | Kept, but flagged as outliers in the EDA |

---

## Task 2 – Exploratory Data Analysis

The EDA runs on the cleaned dataset. It repeats the key cleaning steps first (safe to re-run), so it also works if a file still contains the zero-price rows.

### What the EDA Covers

| Area | Details |
|------|---------|
| **Descriptive statistics** | Mean, median, percentiles, skew for all numeric columns |
| **Group comparisons** | Borough, room type, neighbourhood, host type, minimum-stay band |
| **Charts** | 12 charts covering distributions, comparisons, correlations, geography and time |
| **Anomalies** | IQR outliers, extreme prices and stays, unusual review rates, very large hosts |
| **Insights** | 8 data-driven insights, with numbers calculated directly from the data |

### Summary Statistics

| Metric | Value |
|--------|-------|
| Listings analysed | 48,884 |
| Median price | $106 |
| Mean price | $153 |
| 95th percentile price | $355 |
| 99th percentile price | $799 |
| Maximum price | $10,000 |
| Price skewness | 19.1 |
| Median minimum nights | 3 |

| Borough | Listings | Median price |
|---------|---------:|-------------:|
| Manhattan | 21,660 | $150 |
| Brooklyn | 20,095 | $90 |
| Queens | 5,666 | $75 |
| Bronx | 1,090 | $65 |
| Staten Island | 373 | $75 |

| Room type | Listings | Median price |
|-----------|---------:|-------------:|
| Entire home/apt | 25,407 | $160 |
| Private room | 22,319 | $70 |
| Shared room | 1,158 | $45 |

---

## Charts

### Price distribution
Prices are heavily right-skewed: most listings are under $200, with a long tail up to $10,000.

![Price distribution](eda_charts/01_price_distribution.png)

### Listings by borough and room type
![Listings by borough and room type](eda_charts/02_listings_by_borough_roomtype.png)

### Median price by borough and room type
![Median price by borough and room type](eda_charts/03_median_price_borough_roomtype.png)

### Price spread by borough
![Price spread by borough](eda_charts/04_price_boxplot_borough.png)

### Top neighbourhoods
![Top neighbourhoods](eda_charts/05_top_neighbourhoods.png)

### Correlation heatmap
![Correlation heatmap](eda_charts/06_correlation_heatmap.png)

### Availability
![Availability](eda_charts/07_availability.png)

### Geographic price map
High prices cluster in lower Manhattan and Midtown.

![Geographic price map](eda_charts/08_price_map.png)

### Review activity over time
![Review activity over time](eda_charts/09_review_timeline.png)

### Minimum nights
![Minimum nights](eda_charts/10_minimum_nights.png)

### Top hosts
![Top hosts](eda_charts/11_top_hosts.png)

### Outlier boxplots
![Outlier boxplots](eda_charts/12_outlier_boxplots.png)

---

## Key Insights

1. **Two boroughs dominate supply, and Manhattan is the most expensive.** Manhattan and Brooklyn hold 85% of all listings. Manhattan's median price is $150, against $90 in Brooklyn and $65 in the Bronx (about 2.3x the Bronx).

2. **Room type is the biggest price driver.** Entire homes have a median of $160, private rooms $70 and shared rooms $45. An entire home costs about 2.3x a private room, and entire homes cost more than private rooms in every borough.

3. **Prices are heavily right-skewed, so the median is more reliable than the mean.** The mean ($153) is well above the median ($106), with a skew of 19.1. 95% of listings cost under $355, but the 99th percentile is $799 and the maximum is $10,000.

4. **Location within a borough matters as much as the borough.** Tribeca has the highest median price ($295) among neighbourhoods with 30+ listings, while Corona has the lowest ($40). The price map shows high prices clustered in lower Manhattan and Midtown.

5. **Multi-listing (professional) hosts keep listings available far more often.** They average 179 available days a year against 79 for single-listing hosts. 86% of hosts own only one listing, and the top 10 hosts control 2.6% of all listings, so supply is spread widely with a small professional tail (12 hosts own 50+ listings each).

6. **Long minimum stays form a distinct, less-reviewed segment.** 9.2% of listings require 30+ nights. They average 8.3 reviews against 28.6 for listings with a 1–3 night minimum. This suggests that short-stay listings turn over and collect reviews faster, although the data shows the link, not the cause.

7. **Review activity is concentrated in 2019, and price barely correlates with other numeric fields.** 65% of listings that have a review were last reviewed in 2019, and 21% have never been reviewed. The strongest Spearman correlation with price is with `calculated_host_listings_count` (ρ = −0.11), so price is explained mainly by location and room type rather than by review counts.

8. **Anomalies need handling before modelling.** 239 listings cost more than $1,000 a night (26 cost $5,000 or more), 14 require stays longer than a year, and 36% have zero availability, which likely means inactive or blocked calendars.

The full written insights are also saved in `eda_tables/insights.txt`.

---

## How to Run

### Option A – Jupyter Notebook (recommended)

1. Put `AB_NYC_2019_cleaned.xls` (or `.csv`) in your working folder.
2. In the second code cell of `NYC_Airbnb_EDA.ipynb`, set `PATH` to your file location.
3. Run **Kernel → Restart & Run All**. Charts appear inline.

### Option B – Python scripts

```bash
# Task 1: data quality audit and cleaning
python data_quality_check.py AB_NYC_2019.csv

# Task 2: exploratory data analysis (charts and tables are saved to folders)
python eda_analysis.py AB_NYC_2019_cleaned.xls
```

> **Note:** a file ending in `.xls` can actually be a CSV. The EDA code tries `read_excel` first and falls back to `read_csv`, so both cases work.

---

## Limitations

- **Snapshot data.** The dataset covers a single point in 2019, so it shows no price trends over time.
- **Listed prices, not booked prices.** Prices are what hosts ask, not what guests paid.
- **Associations, not causes.** Findings such as the link between minimum stay and review count describe patterns only.
- **Outliers were kept.** Extreme prices and minimum stays remain in the cleaned data. Medians and log scales are used to limit their effect.
- **Zero availability is ambiguous.** It may mean an inactive listing or a fully booked one.

---

## Next Steps

- Build a predictive model for nightly price (location, room type, minimum nights, availability)
- Cluster neighbourhoods by price and activity level
- Compare outlier-trimmed and full-data results
- Build an interactive dashboard (e.g., Streamlit or Power BI)
- Add review text or seasonality data if it becomes available

---

## Data Source

The dataset is the publicly available *New York City Airbnb Open Data* (2019), originally derived from Inside Airbnb.

## License

Specify your preferred license here (e.g., MIT).
