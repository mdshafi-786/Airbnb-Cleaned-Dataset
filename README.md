# NYC Airbnb 2019 – Data Quality Assessment

A Python-based data quality audit of the **New York City Airbnb Open Data (2019)** dataset. The project identifies four classes of data problems:

1. Missing values
2. Duplicate records
3. Incorrect data types
4. Inconsistent values

---

## Dataset

| Property | Value |
|----------|-------|
| File | `AB_NYC_2019.csv` |
| Rows | 48,895 |
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
├── AB_NYC_2019.csv          # Raw dataset
├── data_quality_check.py    # Data quality audit script
└── README.md
```

---

## Requirements

- Python 3.8+
- pandas
- numpy

```bash
pip install pandas numpy
```

---

## Usage

1. Place `AB_NYC_2019.csv` in the same folder as the script.
2. Run:

```bash
python data_quality_check.py
```

The script prints a report to the console covering each check below.

---

## What the Script Checks

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

---

## Key Findings

| Category | Finding |
|----------|---------|
| **Missing values** | `name`: 16 · `host_name`: 21 · `last_review`: 10,052 · `reviews_per_month`: 10,052 |
| **Missing value cause** | All 10,052 missing review fields belong to listings with 0 reviews (structural, not errors) |
| **Duplicates** | No fully duplicated rows and no repeated `id` values |
| **Data types** | `last_review` is stored as text and should be `datetime`; categorical columns can be `category` |
| **Inconsistent values** | 238 listing names with leading/trailing whitespace · 11 listings with `price = 0` · 14 listings with `minimum_nights` > 365 (max 1,250) · prices up to $10,000 |
| **Borough / room type labels** | Clean (5 boroughs, 3 room types) |
| **Worth reviewing** | 17,533 listings have `availability_365 = 0` (plausible if the host blocked the calendar) |

---

## Suggested Cleaning Steps

```python
df_clean["name"] = df_clean["name"].str.strip().fillna("Unknown")
df_clean["host_name"] = df_clean["host_name"].str.strip().fillna("Unknown")
df_clean["reviews_per_month"] = df_clean["reviews_per_month"].fillna(0)
df_clean["last_review"] = pd.to_datetime(df_clean["last_review"], errors="coerce")
df_clean = df_clean[df_clean["price"] > 0]
```

| Issue | Recommended action |
|-------|--------------------|
| Missing `name` / `host_name` | Fill with `"Unknown"` |
| Missing `reviews_per_month` | Fill with `0` |
| Missing `last_review` | Keep as `NaT` (no reviews exist) |
| `last_review` as string | Convert to datetime |
| Whitespace in text | Apply `str.strip()` |
| `price = 0` | Drop or investigate |
| Extreme `minimum_nights` / `price` | Flag as outliers; cap or exclude depending on the analysis |

---

## Next Steps

- Apply the cleaning steps and export a cleaned CSV
- Run exploratory data analysis on the cleaned data (price by borough, room-type distribution, etc.)
- Build visualizations and a geographic map of listings

---

## Data Source

The dataset is the publicly available *New York City Airbnb Open Data* (2019), originally derived from Inside Airbnb.

## License

Specify your preferred license here (e.g., MIT).
