# Team Guide — Genre Coherence Project

**STAT 4710/5710 · Spring 2026 · Sung Cho, Etienne Lee, Leo Li**

---

## 1. Getting Started

### Download the Dataset
1. Go to: https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset
2. Download `dataset.csv` (you need a free Kaggle account)
3. Place it in the same folder as `LastNames_GenreCoherence.Rmd` — the file must be named exactly `dataset.csv`

### Install R Packages
Run this once in R before knitting:

```r
install.packages(c(
  "tidyverse", "corrplot", "fmsb", "cluster", "mclust",
  "glmnet", "randomForest", "caret", "factoextra",
  "ggrepel", "knitr", "kableExtra", "gridExtra",
  "RColorBrewer", "car"
))
```

> **Note:** `car` is needed for the VIF check in the OLS section. `gridExtra` and `RColorBrewer` are used in several plots but not listed in the original setup chunk — install them too.

### Knit the Document
- Open `LastNames_GenreCoherence.Rmd` in RStudio
- Make sure the working directory is set to the folder containing `dataset.csv`
  - In RStudio: Session → Set Working Directory → To Source File Location
- Click **Knit** (or `Ctrl/Cmd + Shift + K`)
- Output: `LastNames_GenreCoherence.html`

> **Runtime warning:** The random forest (`ntree = 300`) and k-means loop will take several minutes on a laptop. First knit may take 10–20 minutes total.

---

## 2. Placeholders to Fill In

All placeholders are marked `[PLACEHOLDER: ...]` in the `.Rmd`. **Knit the document first**, then open the HTML output and use those numbers to write the prose. Replace each placeholder in the `.Rmd` and re-knit for the final version.

| Line | Section | What to write |
|------|---------|---------------|
| 285 | EDA — Popularity by Genre | Which macro-genres have higher median popularity / wider spread (read off the boxplot) |
| 355 | PCA | % variance explained by first 2–5 PCs (from the table printed below the scree plot); whether genres visually separate in the biplot |
| 419 | K-Means ARI | Your printed ARI value; note that ARI near 0 = random alignment, near 1 = perfect recovery |
| 446 | Hierarchical vs. K-Means ARI | Compare the two ARI values from the printed table; conclude whether genre ↔ cluster alignment is low |
| 505 | Genre Coherence Ranking | Which specific genres are tightest / most diffuse (read top/bottom 15 bar chart); compare macro vs. fine-grained coherence scores |
| 552 | OLS Global | Adjusted R² (printed in console output); which audio features and macro-genre dummies are significant; note multicollinearity between energy and loudness |
| 585 | Lasso | How many features survive at λ.1se vs. λ.min; whether macro_genre dummies are retained; direction of key surviving coefficients |
| 620 | Stratified OLS | Whether valence/energy effects differ across macro-genres; note any sign reversals in the coefficient plot |
| 669 | RF Classifier | Overall accuracy and macro F1 (from printed output); which macro-genres have lowest F1 |
| 697 | Boundary Songs | Pick 3–5 real songs from the printed table and interpret them musically — this is the most engaging part of the paper |
| 745 | Feature Importance | Whether the top RF features (genre classification) and top lasso features (popularity) overlap or diverge |
| 751 | Discussion & Conclusion | Full synthesis tying all 5 angles together — see the detailed prompt already in the placeholder |
| 769 | Statement on AI Use | Describe which AI tools were used and how (Claude Code for scaffolding, etc.) — per course policy |

---

## 3. File Rename

Before final submission, rename the file:

```
LastNames_GenreCoherence.Rmd  →  Cho_Lee_Li_GenreCoherence.Rmd
```

Also update the `author:` field in the YAML header if not already done.

---

## 4. Division of Writing (suggested)

| Placeholder | Suggested owner |
|-------------|-----------------|
| EDA prose (line 285) | Anyone — straightforward read-off |
| PCA + Clustering (lines 355, 419, 446) | Whoever understands unsupervised methods best |
| Genre Coherence (line 505) | Anyone — read the table and interpret |
| Regression (lines 552, 585, 620) | Whoever ran HW3 lasso section |
| Boundary Songs (line 697) | Best prose writer — most humanly engaging |
| Feature Importance (line 745) | Same person doing regression |
| Discussion & Conclusion (line 751) | Whole group — synthesizes everything |
| AI Use Statement (line 769) | Whole group — agree on what to disclose |

---

## 5. Quick Troubleshooting

| Problem | Fix |
|---------|-----|
| `dataset.csv` not found | Set working directory to the `.Rmd` folder before knitting |
| `fmsb` radar chart errors | Make sure `RColorBrewer` is installed |
| `car::vif()` not found | Run `install.packages("car")` |
| `gridExtra` not found | Run `install.packages("gridExtra")` |
| Random forest takes too long | Reduce `ntree = 300` to `ntree = 100` for a test run |
| K-means subsample is slow | Reduce `min(10000, ...)` to `min(3000, ...)` for a test run |
| Boundary song table is empty | Lower threshold from `0.4` to `0.5` in the `boundary-songs` chunk |
