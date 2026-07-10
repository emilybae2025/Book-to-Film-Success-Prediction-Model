# Book-to-Movie Adaptation Success Predictor

Can you predict whether a book-to-movie adaptation will be a hit or a flop — before filming even begins? We built a machine learning classifier to find out.

---

## The Problem

Books like *The Lord of the Rings* and *Harry Potter* became massive box office hits. Others like *Eragon* and *The Golden Compass*, despite huge fanbases, flopped. We wanted to know: **what actually drives adaptation success?**

---

## What We Built

An XGBoost classifier that predicts whether a book-to-movie adaptation will be **profitable (hit)** or **unprofitable (flop)** using only pre-release information — no post-release data, no cheating.

**Example prediction:**
> A studio plans to adapt a book with a 4.2 Goodreads rating, 350 pages, and a $5M budget.
> Model output: **FLOP** (79% confidence)

---

## Dataset

No ready-made dataset existed, so we built one by merging:
- **CMU Movie Dataset** — IMDb ratings, budget, revenue
- **Goodreads** (scraped via Wikipedia IDs) — book ratings, page count, vote counts

After cleaning (filtering zero values, dropping identifiers, standardizing ratings to a 10-point scale), we had a comprehensive dataset linking books to their film adaptations.

---

## Key Findings

- **Book ratings predict film quality** — highly rated books tend to produce higher-rated films
- **Book ratings do NOT predict box office success** — critical acclaim ≠ commercial success
- **Two factors dominate profitability prediction:**
  - Production budget (~60% feature importance)
  - Book fanbase size / total votes (~30% feature importance)

---

## Models

| Model | Accuracy | Notes |
|---|---|---|
| XGBoost Classifier | 93% | 95% hit detection, 92.4% flop detection |
| Logistic Regression | 71% (AUC 76.3%) | Baseline comparison |

**Challenges solved:**
- Built custom dataset from two sources with no existing merge
- Removed post-release features to prevent data leakage
- Handled severe class imbalance (85% flops / 15% hits) using `scale_pos_weight`

---

## Tech Stack

- Python (XGBoost, scikit-learn, pandas)
- Goodreads web scraping
- CMU Movie Dataset

---

## Future Work

- Add features: genre, director reputation, marketing budget, release timing
- Validate on 2024–2025 adaptations
- Build an interactive web app for studios to input book details and get instant predictions
