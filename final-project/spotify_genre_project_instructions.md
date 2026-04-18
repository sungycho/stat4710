# Spotify Genre Coherence Project — Claude Code Instructions

## Purpose of this file
This is a complete instruction set for Claude Code to generate a draft `.Rmd` file for a STAT 4710/5710 Modern Data Mining final project. Read every section before writing any code. The `.Rmd` should be a fully runnable draft with all analysis sections, narrative prose placeholders, and code chunks in the correct order.

---

## Project identity

- **Course:** STAT 4710/5710 — Modern Data Mining, Spring 2026
- **Deliverable:** Final written report as a compiled `.Rmd` → PDF or HTML
- **Group size:** 3 people
- **File naming convention:** `LastNames_GenreCoherence.Rmd` (leave last names as placeholder)

---

## Abstract (use this verbatim in the executive summary section)

> Music streaming platforms assign genre labels to tracks as a primary means of organization and discovery, yet it remains unclear whether these labels reflect genuine musical coherence or function primarily as marketing categories. This project investigates the relationship between Spotify's genre taxonomy and the underlying audio feature space using a dataset of 114,000 tracks spanning 125 genres. We apply principal component analysis to reduce the high-dimensional audio feature space and use k-means and hierarchical clustering to assess whether unsupervised groupings recover genre labels, quantified by adjusted Rand index and silhouette score. We further compute within-genre variance across all 125 genres to produce a genre coherence ranking, train a multi-class classifier to identify boundary songs that are musically proximate to genres other than their assigned label, and fit lasso regression models — both globally and stratified by macro-genre — to assess whether genre label predicts track popularity beyond what audio features alone explain. Together, these analyses ask whether genre is a meaningful descriptor of musical identity or an artifact of industry convention, with implications for how streaming platforms structure discovery and recommendation.

---

## Dataset

### Source
- **Name:** Spotify Tracks Dataset
- **Author:** maharshipandya on Kaggle
- **URL:** `https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset`
- **Format:** Single CSV file (`dataset.csv`)
- **Size:** 114,000 rows × 20 columns

### Column reference (exact names from dataset)

| Column | Type | Description |
|--------|------|-------------|
| `track_id` | character | Spotify track ID |
| `artists` | character | Artist name(s), semicolon-separated if multiple |
| `album_name` | character | Album name |
| `track_name` | character | Track name |
| `popularity` | numeric (0–100) | Popularity score; based on recent play counts |
| `duration_ms` | numeric | Track length in milliseconds |
| `explicit` | logical | Whether track has explicit lyrics |
| `danceability` | numeric (0–1) | Suitability for dancing |
| `energy` | numeric (0–1) | Perceptual intensity and activity |
| `key` | integer (−1 to 11) | Pitch class; −1 = no key detected |
| `loudness` | numeric (dB) | Overall loudness |
| `mode` | binary (0/1) | 0 = minor, 1 = major |
| `speechiness` | numeric (0–1) | Presence of spoken words |
| `acousticness` | numeric (0–1) | Confidence of being acoustic |
| `instrumentalness` | numeric (0–1) | Likelihood of no vocals |
| `liveness` | numeric (0–1) | Presence of live audience |
| `valence` | numeric (0–1) | Musical positiveness |
| `tempo` | numeric (BPM) | Estimated tempo |
| `time_signature` | integer (3–7) | Beats per bar |
| `track_genre` | character | Genre label (125 unique values) |

### Audio features to use as the primary feature matrix
The following 10 continuous features form the core audio feature matrix for all modeling:

```
danceability, energy, loudness, speechiness, acousticness,
instrumentalness, liveness, valence, tempo, duration_ms
```

Exclude `key`, `mode`, `time_signature` from the primary feature matrix (they are discrete/categorical and behave differently in distance-based methods). They can be included as supplementary variables in EDA.

### Preprocessing steps to implement
1. Remove duplicate `track_id` entries — keep first occurrence
2. Drop rows with any `NA` in the 10 audio features
3. Standardize (z-score) all 10 audio features before PCA and clustering
4. Create a `macro_genre` column by mapping the 125 fine-grained genres into ~10 macro-genre families. Use the following mapping as a starting point (Claude Code should implement this as a named vector or lookup table):

```
Electronic / Dance: edm, electro, electronic, house, techno, trance, dubstep, drum-and-bass, garage, chicago-house, deep-house, detroit-techno, minimal-techno, progressive-house, psych-rock → actually rock
Hip-Hop / R&B: hip-hop, r-n-b, rap, trap
Pop: pop, dance, power-pop, synth-pop, k-pop, cantopop, mandopop
Rock / Metal: rock, alt-rock, alternative, grunge, hard-rock, heavy-metal, metal, death-metal, black-metal, metalcore, punk, punk-rock, emo, goth
Classical / Jazz: classical, piano, opera, jazz
Folk / Country / Acoustic: folk, country, bluegrass, acoustic, singer-songwriter, americana
Latin: latin, salsa, samba, bossanova, sertanejo, pagode, mpb, reggaeton, tango
World / Other: afrobeat, anime, brazil, british, club, forro, french, german, indian, iranian, malay, new-age, romance, show-tunes, ska, sleep, songwriter, spanish, swedish, turkish, world-music
Soul / Funk / Blues: soul, funk, blues, groove, disco
Reggae / Caribbean: reggae, dancehall
```

Note: some genres may not appear in the dataset or may be spelled differently. Handle mismatches gracefully with a fallback `"Other"` category.

---

## Report structure

The `.Rmd` must contain the following sections in this exact order, as level-2 (`##`) headers:

1. Executive Summary
2. Introduction and Motivation
3. Data Description
4. Exploratory Data Analysis
5. Methods
6. Results
7. Discussion and Conclusion
8. Statement on AI Use
9. Appendix

---

## Analysis plan — implement each as a named code chunk

### Section 4: Exploratory Data Analysis

**Chunk: `eda-distributions`**
- Histogram of `popularity` (note its distribution — likely right-skewed with a spike at 0)
- Histograms or density plots of each of the 10 audio features
- Flag any features with extreme bimodality (instrumentalness is known to be highly bimodal)

**Chunk: `eda-correlation`**
- Correlation matrix of the 10 audio features
- Visualize as a heatmap using `corrplot` or `ggplot2` + `reshape2`
- Note the known strong correlations: energy ↔ loudness (~0.76), valence ↔ danceability (~0.46), energy ↔ acousticness (negative ~−0.71)

**Chunk: `eda-genre-counts`**
- Bar chart of track counts per `macro_genre`
- Flag any severely imbalanced macro-genres

**Chunk: `eda-radar`**
- For each `macro_genre`, compute the mean of each standardized audio feature
- Visualize as a radar/spider chart using `fmsb` package
- This is the "audio fingerprint" of each genre family — keep it visually clean, one chart per macro-genre or a faceted comparison

**Chunk: `eda-popularity-by-genre`**
- Boxplot of `popularity` by `macro_genre`
- Note which genres have higher median popularity vs. wider variance

---

### Section 5 + 6: Methods and Results — five angles

#### Angle 1: PCA + Clustering

**Chunk: `pca-fit`**
- Standardize the 10-feature matrix
- Fit PCA using `prcomp(scale. = TRUE)`
- Scree plot: variance explained per component
- Biplot of PC1 vs PC2, colored by `macro_genre`
- Report cumulative variance explained by first 2, 3, 5 components

**Chunk: `clustering-kmeans`**
- Run k-means for k = 5, 10, 15, 20, 30 on the standardized feature matrix
- For each k, compute within-cluster sum of squares (elbow plot) and silhouette score
- Select best k based on elbow + silhouette
- At best k, compute adjusted Rand index (ARI) between cluster assignments and `macro_genre` labels using `mclust::adjustedRandIndex()`
- Visualize cluster assignments on PC1-PC2 plot

**Chunk: `clustering-hierarchical`**
- Fit hierarchical clustering using Ward's method (`hclust(dist(...), method = "ward.D2")`)
- Cut dendrogram at the same k as selected above
- Compute ARI against `macro_genre`
- Compare ARI between k-means and hierarchical — report which recovers genre structure better

**Prose to write:** Interpret the ARI values. An ARI near 0 = random; near 1 = perfect recovery. If ARI is low (likely), that is the central finding — genre labels do not map cleanly to audio feature clusters.

---

#### Angle 2: Within-Genre Variance (Genre Coherence Ranking)

**Chunk: `genre-coherence`**
- For each of the 125 fine-grained genres, compute the mean within-genre variance across all 10 standardized audio features
- Define coherence score as the inverse of mean within-genre variance (higher = more coherent)
- Produce a ranked table: top 15 most coherent genres and bottom 15 least coherent genres
- Visualize as a horizontal bar chart, sorted by coherence score
- Separately: compute coherence at the `macro_genre` level and compare

**Prose to write:** Discuss which genres are tight vs. diffuse and whether this reflects cultural/historical factors. Classical and death metal are expected to be tight; pop and indie are expected to be diffuse. Report whether the data confirms this.

---

#### Angle 3: Popularity Regression

**Chunk: `regression-global`**
- Fit OLS: `popularity ~ danceability + energy + loudness + speechiness + acousticness + instrumentalness + liveness + valence + tempo + duration_ms + macro_genre`
- Report adjusted R², coefficients, and significance
- Note multicollinearity issues (energy/loudness) — consider VIF

**Chunk: `regression-lasso`**
- Fit lasso using `glmnet` with `cv.glmnet` for lambda selection (use `alpha = 1`)
- Use the same feature set as OLS plus `macro_genre` dummy variables
- Report which features survive at `lambda.min` and `lambda.1se`
- Plot coefficient path

**Chunk: `regression-stratified`**
- Fit separate OLS models for each `macro_genre` (at least the 5 largest)
- Compare coefficient signs and magnitudes across genres
- Key question: does the effect of `valence` or `energy` on popularity differ by macro-genre?
- Visualize as a coefficient comparison plot (faceted or dot plot with CIs)

**Prose to write:** Interpret whether `macro_genre` dummies are significant after controlling for audio features. If yes: genre label carries information beyond what the audio features encode, suggesting labels have cultural/market value. If no: audio features fully explain popularity, suggesting labels are redundant.

---

#### Angle 4: Boundary Song Detection (Classifier + Misclassification Analysis)

**Chunk: `classifier-fit`**
- Train a multi-class classifier to predict `macro_genre` from the 10 audio features
- Use random forest (`randomForest` package) — appropriate for multi-class, handles nonlinearity
- Split 80/20 train/test, set seed for reproducibility (`set.seed(42)`)
- Report overall accuracy and confusion matrix on test set
- Report macro-averaged precision, recall, F1 using `caret` or manual computation

**Chunk: `boundary-songs`**
- From the test set, extract predicted class probabilities for each track
- Define "boundary songs" as tracks where: (a) the true genre label is predicted with low confidence (e.g., max probability < 0.4), AND (b) a different macro-genre receives the highest probability
- List the top 20 most interesting boundary songs with: `track_name`, `artists`, `true macro_genre`, `predicted macro_genre`, `max probability`
- These are the songs the model thinks belong to a different genre than their label

**Prose to write:** Surface 3–5 specific well-known examples from the boundary song list and interpret them musically. This is the most humanly engaging part of the paper — make it concrete.

---

#### Angle 5: Feature Importance Summary

**Chunk: `feature-importance`**
- Extract variable importance from the random forest (`importance()` function)
- Plot as a ranked bar chart (Mean Decrease in Gini or Mean Decrease Accuracy)
- Cross-reference with lasso coefficients from Angle 3
- Ask: which features matter most for distinguishing genres (classifier) vs. predicting popularity (lasso)? Are they the same features?

**Prose to write:** This synthesizes Angles 3 and 4 — connect what drives genre separation to what drives popularity.

---

## R packages to load in setup chunk

```r
library(tidyverse)      # data wrangling and ggplot2
library(corrplot)       # correlation heatmap
library(fmsb)           # radar charts
library(cluster)        # silhouette scores
library(mclust)         # adjustedRandIndex
library(glmnet)         # lasso regression
library(randomForest)   # random forest classifier
library(caret)          # confusion matrix and metrics
library(factoextra)     # PCA visualization (fviz_pca_biplot, fviz_scree)
library(ggrepel)        # label repelling in ggplot2
library(knitr)          # kable for tables
library(kableExtra)     # table formatting
```

Add `install.packages()` calls in a commented block at the top for any packages not in base R.

---

## Rmd formatting requirements

- **Output:** `html_document` with `toc: true`, `toc_float: true`, `theme: flatly`
- **Code chunk options:** Set globally with `knitr::opts_chunk$set(echo = TRUE, warning = FALSE, message = FALSE, fig.width = 8, fig.height = 5)`
- **Main body:** Max 15 pages when compiled to PDF — keep prose tight
- **Figures:** Every figure must have a caption set via `fig.cap` in the chunk options
- **Tables:** Use `kable()` + `kableExtra` for all tables, not raw `print()`
- **Section flow:** Each section should have 1–3 sentences of prose before the first code chunk, explaining what the section does and why
- **Placeholder prose:** Where narrative interpretation is needed (Discussion, Conclusion), write `[PLACEHOLDER: interpret results here]` so the group knows where to fill in after running the analysis
- **Seed:** Set `set.seed(42)` in the setup chunk and again before any stochastic operation (k-means, random forest)

---

## Key analytical decisions to hard-code (do not leave as parameters)

- Number of PCA components to retain: choose based on scree plot, but default to showing first 2 in biplots
- k-means k: run elbow from 5 to 30, select programmatically using the "elbow" heuristic or silhouette peak
- Train/test split: 80/20, stratified by `macro_genre` using `caret::createDataPartition()`
- Lasso lambda: use `lambda.1se` for the final model (more regularized, more interpretable)
- Boundary song threshold: max predicted probability < 0.4 — adjust if too few or too many results

---

## Things to avoid

- Do not paste raw `summary()` output into the report — extract specific values and present in prose or table
- Do not use base R `plot()` for final figures — use `ggplot2` throughout for visual consistency
- Do not include all 125 genres in any single visualization — always aggregate to `macro_genre` level for plots, use fine-grained level only for the coherence ranking table
- Do not run clustering on un-standardized features — always standardize first
- Do not interpret ARI or silhouette scores without stating their range and what values mean

---

## Output file

Save the generated `.Rmd` as `LastNames_GenreCoherence.Rmd`. It should be fully runnable from top to bottom assuming the dataset CSV is in the working directory named `dataset.csv`. Include a comment at the top of the setup chunk:

```r
# Set working directory to the folder containing dataset.csv before knitting
# dataset source: https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset
```
