# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data science project analyzing the relationship between book adaptations and their corresponding movies. The project combines data from multiple sources (CMU Movie Summary Corpus, Goodreads, IMDb, TMDB, Wikidata) to analyze factors like ratings, revenue, and genres.

## Data Pipeline Architecture

The project follows a multi-stage pipeline:

1. **Data Collection** ([CreatingDataset.ipynb](CreatingDataset.ipynb))
   - Downloads CMU Movie Summary Corpus
   - Queries Wikidata SPARQL endpoint for book-movie relationships
   - Downloads Goodreads book datasets from Kaggle
   - Downloads IMDb ratings data
   - Downloads TMDB movie metadata
   - Fetches CPI data from FRED for inflation adjustment

2. **Data Cleaning** ([DataCleaning.ipynb](DataCleaning.ipynb), [clean.ipynb](clean.ipynb))
   - Merges multiple data sources on join keys (`join_title`, `join_author`)
   - Adjusts movie budgets and revenues for inflation using CPI data
   - Handles missing values and duplicates
   - Creates normalized comparison columns (Book Rating and Movie Rating as percentages)

3. **Exploratory Data Analysis** ([EDA.ipynb](EDA.ipynb))
   - Visualizes distributions and relationships
   - Creates scatter plots with regression lines for book vs movie ratings
   - Analyzes movie budget vs revenue correlations
   - Computes correlation coefficients (r values)

4. **Modeling** ([modeling.ipynb](modeling.ipynb))
   - Contains predictive modeling work

## Key Data Files

- `book1-100k.csv` - Goodreads book data (100k books)
- `book_adaptation_df_clean.csv` / `book_adaptation_df_clean.xlsx` - Initial merged dataset with all movies
- `cleaned_df.csv` - Filtered dataset containing only movies with matching book data (3,265 rows)
- `final_data.csv` - Final processed dataset with computed features (book_profit, normalized ratings)

## Dataset Schema

The final dataset includes:
- **Movie fields**: `movie_title`, `imdb_id`, `movie_budget`, `movie_revenue`, `movie_release`, `imdb_rating`, `imdb_total_votes`, `movie_profit`
- **Book fields**: `Publisher`, `Pages`, `Rating`, `RatingDistTotal`, `Authors`
- **Normalized fields**: `Book Rating` (0-1 scale), `Movie Rating` (0-1 scale)
- **Join keys**: `join_title`, `join_author` (cleaned, lowercase versions for merging)

## Common Data Processing Patterns

### Title and Author Normalization

The project uses consistent cleaning functions for matching across datasets:

```python
def clean_title(title_series: pd.Series) -> pd.Series:
    return (title_series
            .str.split('(').str[0]
            .str.split(':').str[0]
            .str.lower()
            .str.replace('and', '&')
            .str.replace('.', '')
            .str.replace("'", '')
            .str.replace('-', ' ')
            .str.replace(r'\s+', ' ', regex=True)
            .str.strip()
    )

def clean_author(author_series: pd.Series) -> pd.Series:
    initial_letter = (author_series
                      .str.strip()
                      .str[0]
                      .str.lower())
    last_name = (author_series
                 .str.split(r"(\s|-|')", regex=True)
                 .str[-1]
                 .str.replace('.', '')
                 .str.replace("'", '')
                 .str.replace(r'\s+', ' ', regex=True)
                 .str.strip()
                 .str.lower()
                 )
    return initial_letter + " " + last_name
```

### Handling Missing Values

- Replace 0 values with `np.nan` or `pd.NA` for numeric columns before analysis
- Use `.fillna()` to combine multiple data sources (e.g., CMU revenue vs TMDB revenue)
- Use `errors='coerce'` when converting strings to numeric/datetime

## Environment Setup

Install dependencies:
```bash
pip install -r requirements.txt
```

### Kaggle Setup (for data downloads)

Some notebooks download datasets from Kaggle. To use the Kaggle API:
1. Create a Kaggle account and generate API credentials
2. Place `kaggle.json` in `~/.kaggle/`
3. Set permissions: `chmod 600 ~/.kaggle/kaggle.json`

## Running Notebooks

The notebooks should be run in this order for the full pipeline:
1. [CreatingDataset.ipynb](CreatingDataset.ipynb) - Data collection
2. [clean.ipynb](clean.ipynb) - Data merging and cleaning
3. [EDA.ipynb](EDA.ipynb) - Exploratory analysis and visualization
4. [modeling.ipynb](modeling.ipynb) - Predictive modeling

Alternatively, work directly with the pre-processed CSV files (`cleaned_df.csv`, `final_data.csv`).

## External APIs and Data Sources

- **Wikidata SPARQL**: `https://query.wikidata.org/sparql` - Requires well-formed SPARQL queries, includes retry logic for rate limits
- **WikiMapper**: Maps Wikipedia IDs to Wikidata IDs (requires downloading and indexing Wikipedia dumps)
- **IMDb datasets**: `https://datasets.imdbws.com/` - Public TSV files, compressed with gzip
- **FRED (Federal Reserve)**: `https://fred.stlouisfed.org/graph/fredgraph.csv?id=CPIAUCSL` - CPI data for inflation adjustment
- **Kaggle**: Requires API authentication for dataset downloads

## Important Notes

- Budget and revenue values are inflation-adjusted to current dollars using CPI data
- The dataset has significant missing values for `movie_budget` (~70% missing)
- Book ratings are on a 5-point scale, movie ratings on a 10-point scale (normalized to 0-1 for comparison)
- Multiple books can adapt to the same movie, creating duplicate movie entries
- The project focuses on English-language books and films
