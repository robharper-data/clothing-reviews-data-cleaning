# Women's clothing reviews: parsing, cleaning and integrating data

A data-wrangling pipeline in Python: parse nested XML, audit and clean every column, log each problem found and how it was handled, then integrate the result with a second dataset that uses a different schema.

Completed in 2024 during my Master of Data Science at RMIT University.

## Data

Both datasets are versions of the [Women's E-Commerce Clothing Reviews](https://www.kaggle.com/datasets/nicapotato/womens-ecommerce-clothing-reviews) dataset from Kaggle: one review per record, with the item reviewed, the reviewer's age, title, rating, product category, price and whether they would recommend it.

The versions I worked with were modified with deliberate errors for a data-cleaning exercise and are not mine to redistribute, so no data is included. The notebooks keep their original outputs, so the work can be followed without it.

## Part 1: parsing and cleaning

`01_parse_and_clean.ipynb` parses 6,406 review records from XML, where two fields (section and department) were nested, into a pandas DataFrame. It then audits each column with summary statistics and boxplots.

Every problem found is recorded in `error_log.csv` with the row affected, the original and new values, the type of error and the reasoning, including problems deliberately left unchanged. The main findings:

| Problem | Records | Action |
|---|---|---|
| All 11 columns parsed as text | All | Converted to integer, float or boolean as appropriate |
| Reviewer ages of 2 to 4 years | 5 | Impossible values. Replaced with the median of valid ages (41) |
| Reviewer ages of 80 and over | 33 | Flagged as outliers but kept: plausible, and removing them would bias the data against older reviewers |
| Ratings of 5.25 on a 1 to 5 scale | 2 | Corrected to 5 |
| Misspelt categories ("Jacktes", "Kints", "kSirts") | 3 | Corrected |
| Review writing times over 650 minutes | 14 | Implausible extremes. Replaced with the median |
| Prices of -1 | 8 | Regression imputation was tried and rejected (R-squared 0.08). Replaced with the mean price of the same item |
| Large positive-feedback counts | Several | Kept: likely genuine and informative |

## Part 2: integration

`02_integrate.ipynb` merges the clean data with a second dataset of 7,500 reviews that uses different column names and types.

- **Schema conflicts:** nine columns renamed to the second dataset's schema, the recommendation flag converted back from boolean to integer, columns reordered, and time and price rounded consistently so that matching records compare as equal.
- **Duplicates:** the merged table had 13,906 records. 171 records appeared in both datasets and were removed, leaving 13,735.
- **Unique key:** there is no review ID, so combinations of columns were tested for uniqueness. Clothing ID, price and title together was the smallest combination with no duplicates.

## Limitations

- **Exact duplicates only.** The duplicate check matches records that are identical in every column. The same review with a small difference, such as a rounding discrepancy, would not be caught.
- **The proposed key is fragile.** 1,178 reviews in the second dataset have no title, so for those the key rests on clothing ID and price alone, and two reviews could share all three by coincidence. The robust fix, as the notebook notes, is a review ID in the source system.
- **Median and mean imputation** keeps every record but understates the true spread of the imputed columns.
- Six records in the second dataset have no division, department or class. They were left as supplied.

## Files

- `01_parse_and_clean.ipynb`: parsing and cleaning
- `02_integrate.ipynb`: integration, duplicate removal and key selection
- `error_log.csv`: every problem found in part 1 and how it was handled
