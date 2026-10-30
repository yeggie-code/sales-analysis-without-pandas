# Electronics Sales Analysis

This project analyzes a small electronics sales CSV using Python's standard
library. It cleans messy quantity and price values, calculates transaction
revenue, summarizes revenue by category, ranks products, and computes basic
statistics. A pandas-based function independently checks the headline results.

## Project files

- `analysis.py` contains the executable analysis and reusable functions.
- `data/electronics_sales.csv` is the input dataset.
- `starter (1).py` is a separate exercise scaffold. Its analysis functions
  still contain `NotImplementedError`, so run `analysis.py` for the working
  analysis.

## Input data

The CSV has 50 sales records and these columns:

| Column | Meaning |
| --- | --- |
| `sale_id` | Unique sale identifier |
| `sale_date` | Date of sale |
| `store` | Store where the sale occurred |
| `product` | Product sold |
| `quantity` | Units sold |
| `unit_price_kes` | Price per unit in Kenyan shillings |
| `payment_method` | Payment method used |

Some numeric values are deliberately inconsistent: quantities may be written
as words, prices may include `KES` or comma separators, and some numeric fields
are blank or marked `N/A`.

## Run the analysis

From the repository root, run:

```powershell
python analysis.py
```

The input path in the script is `data/electronics_sales.csv`, relative to the
current working directory. Run the command from the repository root so that
the CSV can be found.

The standard-library analysis does not need pandas. The final verification
step does; install pandas in the Python environment used to run the script if
it is not already available:

```powershell
python -m pip install pandas
```

## Data cleaning and revenue

### Reading the CSV

`read_sales(path)` uses `csv.DictReader` and returns a list of dictionaries.
Each initial field value is text. It also prints the column names and the
number of rows it read.

### Parsing numbers

`parse_number(value)` normalizes a numeric field and returns a float, or
`None` if the value cannot be converted:

1. Leading and trailing whitespace is removed and the text is lowercased.
2. The words `one`, `two`, `three`, `four`, and `five` are converted to their
   numeric values.
3. The text `kes`, commas, and `n/a` are removed.
4. The remaining text is converted to a float. Blank or invalid text returns
   `None`.

Examples:

| Input | Result |
| --- | ---: |
| `two` | `2.0` |
| `KES 18,999` | `18999.0` |
| `N/A` | `None` |
| an empty field | `None` |

### Keeping valid rows

`convert_rows(rows)` parses `quantity` and `unit_price_kes` for every sale. A
row is included in the clean list only if both values are valid. The returned
clean row is a copy of the input row with those two fields changed to numeric
values. Invalid rows go into a separate skipped list with their sale ID and a
reason; the original row list is not modified.

For the supplied CSV, 47 records are clean and three are skipped:

- `S-3030`: invalid or missing unit price (`N/A`)
- `S-3035`: invalid or missing quantity (blank)
- `S-3039`: invalid or missing unit price (blank)

For each clean transaction, revenue is calculated as:

```text
quantity * unit_price_kes
```

Revenue is calculated when needed rather than stored as a new field in the
clean row.

## Analysis functions

### Overall revenue

`total_revenue(rows)` sums the revenue of every row in the supplied clean
rows.

### Revenue grouped by a field

`revenue_by(rows, field)` returns a plain dictionary mapping each distinct
field value to the sum of revenue for rows with that value. It is used for
`product`, `store`, and `payment_method`.

### Product ranking

`top_n(totals, n)` accepts the dictionary returned by `revenue_by` and returns
up to `n` `(name, revenue)` tuples. Revenue is sorted highest first. When
revenues tie, names are ordered alphabetically.

### Transaction statistics

The script builds one revenue value per clean transaction and calculates:

- `mean(values)`: sum of the values divided by the number of values.
- `median(values)`: middle value after sorting, or the average of the two
  middle values for an even number of items.
- `minimum_maximum(values)`: the lowest and highest values, found in one
  traversal without calling the built-in `min()` or `max()`.

All three functions raise `ValueError` with a clear message if given an empty
list.

### High-value transactions

`high_value(rows, threshold)` is intended to select transactions whose
revenue is greater than the threshold. The main script uses a threshold of
KES 50,000, prints qualifying sales, and builds a dictionary with the count
for each store.

**Current implementation limitation:** the function in `analysis.py` currently
returns from inside its loop, so it returns only the first qualifying sale
(or `None` if no sale qualifies). It does not currently return all qualifying
sales, and the per-store counts printed by the script therefore do not
represent all qualifying transactions. The intended implementation is a list
comprehension over all rows.

## Pandas verification

`verify_with_pandas(path, my_results)` independently reads the source CSV
with pandas, applies the same text normalization and word-number conversions
to the quantity and unit-price columns, and treats invalid numeric values as
missing. It includes a row only when both numeric values are valid, computes
the transaction revenue, and recalculates:

- `total_revenue`
- `mean_transaction_revenue`
- `median_transaction_revenue`
- `minimum_transaction_revenue`
- `maximum_transaction_revenue`
- `top_products` (the top three, with the same alphabetical tie-break)

Pass these results in a dictionary. The script constructs all six keys. The
verifier requires `total_revenue` and `top_products`; for other metrics, it
prints a comparison when that key is present. Each comparison prints
`MATCH` or `MISMATCH`, along with the standard-library and pandas values.
Floating-point values are compared with a small tolerance to avoid harmless
representation differences.

For the supplied data, the standard-library and pandas results match:

| Metric | Result |
| --- | ---: |
| Clean transactions | 47 |
| Skipped transactions | 3 |
| Total revenue | KES 1,317,210 |
| Mean transaction revenue | KES 28,025.744680851065 |
| Median transaction revenue | KES 10,998 |
| Minimum transaction revenue | KES 1,499 |
| Maximum transaction revenue | KES 259,996 |

The top three products by revenue are:

| Rank | Product | Revenue |
| ---: | --- | ---: |
| 1 | HP 15 Laptop | KES 389,994 |
| 2 | Hisense 43in Smart TV | KES 272,993 |
| 3 | Lenovo Tab M10 | KES 199,992 |

There are no pandas mismatches for these headline results, so no discrepancy
explanation is needed. The high-value limitation above is separate from the
pandas comparison: the verifier does not currently compare high-value sales
or store counts.
