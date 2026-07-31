---
layout: post
title: "Day 8 — From Raw Data to Paper Figures: A Complete Microbiome Stats Workflow"
date: 2026-03-11
description: "Every method from Days 1–7, assembled into the actual order you run them, with the biological question that justifies each step — and the checklist you copy into every new project."
comments: true
giscus_comments: true
featured: true
permalink: /blog/day8-complete-microbiome-stats-workflow/
tags: [workflow, PERMANOVA, RDA, regression, WGCNA, Mantel, functional-redundancy, metagenomics, R]
series: applied-stats
series_title: "Applied Statistics for Microbiome Data"
order: 8
---

#### Day 7 — From Raw Data to Paper Figures: A Complete Microbiome Stats Workflow

\*Day 7 of 7 in the series **Applied Statistics for Microbiome & Genomics Data.\*** & 🧬 Day 57 of Daily Bioinformatics from Jojy's Desk
_Dataset: 61 MAGs × 27 metagenomes, Chesapeake Bay + Delaware Bay._
_Paper: John et al. 2026, ISME Communications — Functional redundancy and metabolic flexibility of microbial communities in two Mid-Atlantic bays._
_All R code: [github.com/jojyjohn28/microbiome_stats](https://github.com/jojyjohn28/microbiome_stats)_

---

Days 1–6 introduced six methods. Each was presented as its own lesson. But in a real analysis — one that ends in a published paper — they don't run in isolation. They run in sequence, each answer feeding the next question.

This is the post I wish I had before starting. Not "here is how PERMANOVA works," but: **here is the exact order you run everything, the biological question that justifies each step, and what you do when each test tells you something unexpected.**

The workflow below is the one I used for our recent ISME Communications paper. I'll show you how each method connects, where the Mantel test fits as a bridge between community structure and environment, how I structure the stats section of a paper, and end with a reusable checklist for every future project.

---

#### The Core Principle: Let the Biology Drive the Statistics

The single most common mistake in microbiome data analysis is running every available test and reporting the ones that are significant. This produces p-value mining, not science.

Every step in this workflow is anchored to a biological question. If you cannot write the question in plain English before running the test, do not run it.

The questions I started with, directly from our paper's hypotheses:

1. Does community composition differ across environmental conditions (season, salinity, bay)?
2. Which environmental gradients structure the community — and how much variance do they explain?
3. Is community dissimilarity correlated with environmental dissimilarity?
4. Which individual MAGs are enriched under which conditions, and how large are those effects?
5. How does MAG abundance change continuously along salinity and temperature gradients?
6. Do MAGs form coherent ecological guilds, and which environmental driver controls each guild?
7. What environmental variables predict functional redundancy, and does the answer differ between potential and expressed FRed?

One question, one method. Seven questions, seven methods.

---

#### The Full Workflow

##### Stage 1 — Data preparation (before any statistics)

This stage has no p-values. It is entirely about understanding your data before you model it. Skip it and every downstream result is suspect.

```r
library(vegan)
library(ggplot2)
library(patchwork)

## ── Load ──────────────────────────────────────────────────────
metadata <- read.csv("../day1/data/metadata.csv",
                     header=TRUE, row.names=1)
mag_raw  <- read.csv("../day1/data/mag_abundance.csv",
                     header=TRUE, row.names=1)
mag_t    <- t(mag_raw)               # samples × MAGs

## ── Always align rows before anything else ────────────────────
metadata <- metadata[match(rownames(mag_t), rownames(metadata)), ]
stopifnot(all(rownames(mag_t) == rownames(metadata)))
cat("n =", nrow(mag_t), "| MAGs =", ncol(mag_t), "\n")

## ── Assess sparsity ───────────────────────────────────────────
zero_frac <- sum(mag_t == 0) / length(mag_t)
cat("Zero fraction:", round(zero_frac * 100, 1), "%\n")

## ── Distribution check ────────────────────────────────────────
# Visualise raw abundance distribution — expect right-skewed
hist(log1p(as.vector(mag_t)), breaks=50,
     xlab="log(abundance+1)", main="MAG abundance distribution")
```

What you are checking:

▶ Does the row alignment hold? If metadata and abundance rows are out of order, every downstream result will be silently wrong.

▶ How sparse is the matrix? Our MAG data had ~68% zeros — common for genome-resolved metagenomics. This rules out parametric tests on raw counts.

▶ What does the abundance distribution look like? Right-skewed with zero inflation confirms you need Hellinger or log transformation before any Euclidean-distance-based method.

---

##### Stage 2 — Community composition: ordination (RDA/dbRDA)

**Biological question:** Which environmental variables explain the most variation in overall community structure, and in what direction do they pull the community in multivariate space?

```r
## ── Hellinger transformation ──────────────────────────────────
hell <- decostand(mag_t, method = "hellinger")

## ── RDA: unconstrained first to see variance structure ────────
pca <- rda(hell)
screeplot(pca, bstick=TRUE)          # how many axes worth keeping?

## ── dbRDA: constrain on environmental variables ───────────────
env_vars <- metadata[, c("Salinity", "Temperature", "ChlA",
                          "Bacterial_Production")]
env_sc   <- scale(env_vars)

dbrda <- dbrda(hell ~ ., data=as.data.frame(env_sc),
                dist="bray")
anova(dbrda, by="margin", permutations=999)
RsquareAdj(dbrda)
```

In our paper, the dbRDA confirmed that season (operating through its correlated gradients — temperature and salinity) structured the community along the first constrained axis, with bay identity adding a secondary axis. This plot became Figure 1B and defined the framing for all downstream tests: **the dominant gradient is seasonal/salinity, not spatial.**

Once you have the ordination, the next question becomes: is that visual separation statistically supported?

---

##### Stage 3 — PERMANOVA: is the separation real?

**Biological question:** Do communities differ significantly between predefined groups, and how much of total variance is attributable to each factor?

```r
## ── Bray-Curtis dissimilarity ─────────────────────────────────
bc <- vegdist(hell, method="bray")

## ── Always check dispersion first ────────────────────────────
# If groups differ in spread, PERMANOVA p-values can be misleading
disp <- betadisper(bc, metadata$Season)
anova(disp)          # want p > 0.05

## ── PERMANOVA ─────────────────────────────────────────────────
set.seed(42)
perm <- adonis2(bc ~ Season + Bay + Size_Fraction,
                 data = metadata,
                 permutations = 999,
                 by = "margin")
print(perm)
# Season: R² = 0.325, p = 0.001 ***
# Bay:    R² = 0.049, p = 0.121 ns
# SF:     R² = 0.019, p = 0.550 ns
```

The PERMANOVA result (Season R² = 0.325, p = 0.001) does two things: it validates the ordination visually, and it anchors the variance explained number in the paper. We can now say — with a defensible test — that **season explains 32.5% of variation in MAG community composition**, more than bay identity and size fraction combined.

But PERMANOVA treats season as a discrete label. It cannot tell you _which_ environmental gradient along the seasonal axis drives the pattern. That requires the Mantel test.

---

##### Stage 4 — Mantel test: the bridge between community and environment

**Biological question:** Is the dissimilarity between communities correlated with the dissimilarity between their environmental contexts? If so, which environmental distance matrix best predicts community distance?

This is the step most microbiome workflows skip. It matters because PERMANOVA tests group membership; the Mantel test tests gradients. The two can give different answers, and both are informative.

```r
library(vegan)

## ── Community distance ────────────────────────────────────────
bc <- vegdist(hell, method = "bray")

## ── Environmental distance matrices ──────────────────────────
# Single-variable: does salinity distance predict community distance?
sal_dist  <- dist(scale(metadata$Salinity))
temp_dist <- dist(scale(metadata$Temperature))

# Multivariate: full environmental dissimilarity
env_full  <- scale(metadata[, c("Salinity", "Temperature",
                                  "ChlA", "Bacterial_Production",
                                  "Nitrate", "Phosphate")])
env_dist  <- dist(env_full)

## ── Mantel tests ──────────────────────────────────────────────
set.seed(42)
mantel(bc, sal_dist,  method="pearson", permutations=9999)
# r = 0.61  p = 0.0001
mantel(bc, temp_dist, method="pearson", permutations=9999)
# r = 0.58  p = 0.0001
mantel(bc, env_dist,  method="pearson", permutations=9999)
# r = 0.70  p = 0.0001

## ── Partial Mantel: salinity after controlling for temperature ─
mantel.partial(bc, sal_dist, temp_dist,
               method="pearson", permutations=9999)
# r = 0.31  p = 0.008
# Salinity still explains unique variance in community structure
# even after removing the shared variance with temperature
```

The Mantel test confirmed that community distance is strongly correlated with environmental distance (r = 0.70, p < 0.0001). The partial Mantel test showed that salinity explains unique variance in community structure (r = 0.31, p = 0.008) beyond its correlation with temperature — directly supporting the regression results from Day 5.

**How to report a Mantel test in a paper:**

> "Bray-Curtis community dissimilarity was significantly correlated with multivariate environmental distance (Mantel r = 0.70, p < 0.001, 9999 permutations). Partial Mantel analysis showed that salinity retained a significant partial correlation with community dissimilarity after controlling for temperature (r = 0.31, p = 0.008), consistent with salinity as an independent driver of community composition."

---

##### Stage 5 — Differential abundance: which MAGs, how big

**Biological question:** Which individual MAGs are enriched in one condition versus another, and what is the magnitude of that enrichment?

```r
library(coin)    # wilcox_test() with exact p-values

mag_log  <- log1p(mag_t)
seasons  <- factor(metadata$Season)
mag_names <- colnames(mag_log)

results_wilcox <- do.call(rbind, lapply(mag_names, function(mag) {
  x_spring <- mag_log[seasons == "Spring", mag]
  x_summer <- mag_log[seasons == "Summer", mag]

  wt    <- wilcox.test(x_summer, x_spring, exact=FALSE)
  n1    <- length(x_summer); n2 <- length(x_spring)
  U     <- wt$statistic
  delta <- (U / (n1 * n2)) * 2 - 1   # Cliff's delta

  data.frame(MAG       = mag,
             p_value   = wt$p.value,
             cliffs_d  = delta,
             direction = ifelse(delta > 0, "Summer", "Spring"))
}))

results_wilcox$q_value <- p.adjust(results_wilcox$p_value,
                                    method = "BH")
n_sig <- sum(results_wilcox$q_value < 0.05)
cat("Season-significant MAGs (q<0.05):", n_sig, "of",
    length(mag_names), "\n")
# 40 of 61

# Top hits — all Summer-enriched, Cliff's δ = +1.00 (perfect separation)
head(results_wilcox[order(results_wilcox$q_value), ], 10)
```

The Wilcoxon + Cliff's delta combination answers a question PERMANOVA cannot: not just "do seasons differ" but "which MAGs are responsible and how completely are they partitioned?" All top 40 season-significant MAGs were Summer-enriched. Cliff's δ = +1.00 for the top hits means complete rank separation — not a single Spring sample exceeded a Summer sample in abundance for those MAGs.

In the paper, this result connected directly to the FRed analysis: MAGs enriched in summer were part of the high-FRed community driving the seasonal FRed increase.

---

##### Stage 6 — Regression: how much does abundance change per unit environment

**Biological question:** How does MAG abundance change quantitatively as a continuous environmental variable changes, and does that relationship hold after controlling for correlated predictors?

```r
## ── Fit additive models for all 61 MAGs ──────────────────────
results_reg <- do.call(rbind, lapply(mag_names, function(mag) {
  df_m <- cbind(metadata, abundance = mag_log[, mag])
  m    <- lm(abundance ~ Salinity + Temperature + Season_num,
              data = df_m)
  sm   <- summary(m)
  cf   <- coef(sm)
  data.frame(
    MAG       = mag,
    R2        = sm$r.squared,
    adj_R2    = sm$adj.r.squared,
    beta_sal  = cf["Salinity", "Estimate"],
    p_sal     = cf["Salinity", "Pr(>|t|)"],
    beta_temp = cf["Temperature", "Estimate"],
    p_temp    = cf["Temperature", "Pr(>|t|)"]
  )
}))

results_reg$q_sal  <- p.adjust(results_reg$p_sal,  "BH")
results_reg$q_temp <- p.adjust(results_reg$p_temp, "BH")

cat("Salinity-significant MAGs (q<0.05):",
    sum(results_reg$q_sal < 0.05), "\n")

## ── For our paper: regression of FRed against environment ─────
# (from John et al. 2026, Table 1 and Table S5)
# Potential FRed ~ temperature + season + salinity + size_fraction
m_fred <- lm(Potential_FRed ~ Temperature + Season + Salinity +
               Size_Fraction, data = metadata)
summary(m_fred)
# Adj R² = 0.76, F = 8.52, p < 0.0001
# Temperature: p < 0.007 ***
# Season:      p < 0.001 ***
# Salinity:    p = 0.037 *
# Size Frac:   p < 0.001 ***

# Expressed FRed ~ salinity + ammonium
m_exp <- lm(Expressed_FRed ~ Salinity + Ammonium, data = metadata)
summary(m_exp)
# Adj R² = 0.39, F = 5.01, p < 0.012
```

This regression framework — the same one from Day 5, applied here to functional redundancy rather than just individual MAG abundance — is one of the core results in the paper. Potential FRed was explained by a model that accounted for 76% of variance, with temperature, season, salinity, and size fraction all as significant predictors. Expressed FRed was harder to predict (38% variance explained), with salinity and ammonium as the only significant drivers — a biologically interpretable result that points to nutrient limitation suppressing non-essential metabolic expression.

---

##### Stage 7 — WGCNA: from individual MAGs to ecological guilds

**Biological question:** Do the MAGs significant in previous steps form coherent co-abundance modules, and which single environmental driver controls each module as a unit?

```r
library(WGCNA)
options(stringsAsFactors=FALSE)
enableWGCNAThreads()

hell <- decostand(mag_t, method = "hellinger")

# Soft threshold selection → β = 6
adjacency <- adjacency(hell, power = 6, type = "signed")
TOM       <- TOMsimilarity(adjacency, TOMType = "signed")
dissTOM   <- 1 - TOM

TaxaTree     <- hclust(as.dist(dissTOM), method = "average")
dynamicMods  <- cutreeDynamic(TaxaTree, distM = dissTOM,
                               deepSplit = 2, minClusterSize = 5)
moduleColors <- labels2colors(dynamicMods)
merge        <- mergeCloseModules(hell, moduleColors,
                                   cutHeight = 0.25)
moduleColors <- merge$colors

# Eigengene–trait correlation
MEs         <- orderMEs(moduleEigengenes(hell, moduleColors)$eigengenes)
modTraitCor <- cor(MEs, datTraits, use = "p")
# Blue module (41 MAGs): r = -0.97 with Temperature
# Brown module (4 MAGs):  r = -0.89 with Salinity
```

WGCNA revealed that 41 of the 40 Wilcoxon-significant MAGs belong to a single module — the blue module — with a module eigengene that correlates at r = −0.97 with temperature. What appeared as 40 independent seasonal signals in Stage 5 is, in network terms, one coordinated guild responding to one driver. This is the kind of biological synthesis that only becomes visible when you move from individual tests to the full workflow.

---

##### Stage 8 — Correlation analysis of functional redundancy

**Biological question:** Is functional redundancy correlated with species richness and functional diversity? Do potential and expressed FRed respond differently to environmental variables?

This stage is specific to our paper but illustrates a general principle: after the community-level and taxon-level analyses are complete, you sometimes need a dedicated analysis for derived ecological metrics.

```r
## ── FRed was computed using the Adiv package ─────────────────
# FRed = uniqueness() from Adiv, based on Euclidean distance
# between MAG trait profiles (187 metabolic genes)

library(adiv)

# Trait table: MAGs × metabolic marker genes (METABOLIC V4.0 HMM hits)
# One row per MAG, one column per gene (binary or count)
trait_table <- read.csv("results/metabolic_trait_table.csv",
                         row.names = 1)
trait_dist  <- dist(trait_table, method = "euclidean")

# FRed per sample: weighted by MAG abundance
# uniqueness() = 1 - average uniqueness → ranges 0 (all distinct)
#                                                 to 1 (all identical)
# fred_results: samples × FRed values

## ── FRed correlations ─────────────────────────────────────────
# Pearson r: FRed vs species richness
cor.test(fred_df$Potential_FRed, fred_df$Species_Richness,
         method = "pearson")
# r = 0.47, R² = 0.22, p = 0.037

# Pearson r: FRed vs functional diversity (Q)
cor.test(fred_df$Potential_FRed, fred_df$Q,
         method = "pearson")
# r = 0.13, R² = 0.02, p = 0.58   (non-significant)

## ── Key environmental correlations ───────────────────────────
# Table 1 from paper:
# Potential FRed ~ Temperature: PA: R²=0.651, p=.041
#                               FL: R²=0.912, p=.0001
# Expressed FRed ~ Salinity:    PA: R²=0.654, p=.04
# Expressed FRed ~ Nitrate:     FL: R²=−0.949, p=.005

## ── Wilcoxon tests: season and size fraction effects ──────────
wilcox.test(Potential_FRed ~ Season, data=fred_df)
# p < 0.002: FRed higher in Summer than Spring in both bays

wilcox.test(Potential_FRed ~ Size_Fraction, data=fred_df)
# p < 0.04: PA community higher FRed than FL community
```

The FRed correlation analysis produced a biologically nuanced result: FRed is moderately correlated with species richness (r = 0.47) but not with functional diversity (r = 0.13). This means communities with more species tend toward more redundancy, but communities with higher functional diversity are not necessarily more redundant — a distinction with direct implications for ecosystem stability under perturbation.

---

## How I Structure the Statistics Section of a Paper

After running the above workflow, the challenge becomes translating it into a methods section that is reproducible and a results section that tells a coherent story. Here is the exact structure I used for John et al. 2026.

**Methods — order of presentation:**

1. Data preprocessing and transformation (Hellinger, log1p — justify each)
2. Ordination (RDA/dbRDA — state the dissimilarity metric and transformation)
3. Community-level tests (betadisper + PERMANOVA — always report both)
4. Mantel tests (state number of permutations, correction method)
5. Differential abundance (Wilcoxon + Cliff's delta + BH correction — report n significant / n tested)
6. Regression (model formula, diagnostics run, whether VIF was checked)
7. Network analysis (WGCNA — β value, cutHeight, minClusterSize — all three)
8. FRed quantification (Adiv package, trait table construction, normalization)
9. FRed statistics (linear regression, Wilcoxon, correlation — state which variables were removed for collinearity)

**Results — order of presentation:**

Lead with the community-level result (PERMANOVA: Season R² = 0.325). This orients the reader — the community differs by season, and season explains more variance than any other factor. Then zoom in: Mantel confirms it tracks the environmental gradient. Then zoom in further: Wilcoxon identifies the individual drivers. Then zoom out conceptually: WGCNA shows those drivers are one coherent guild. Then shift to the functional layer: regression on FRed shows how the community-level pattern translates to ecosystem process stability.

The flow is: **community → environment bridge → taxa → guilds → function.**

---

## The Reusable Analysis Checklist

Copy this into every new project. Check each box before moving to the next step.

```
PRE-ANALYSIS
□ Row alignment: metadata and abundance rows match (stopifnot)
□ Zero fraction reported: know your sparsity before choosing tests
□ Abundance distribution visualised: skewness, outliers
□ Transformation chosen with justification (not just "we used Hellinger")
□ Biological questions written out in plain English — one per test

COMMUNITY STRUCTURE
□ PCoA or RDA plotted — visual inspection before any tests
□ betadisper checked before PERMANOVA — dispersion not significantly different
□ PERMANOVA run with marginal SS (by="margin") — not sequential
□ R² and p reported for every factor, not just significant ones
□ Pairwise PERMANOVA only run if overall test is significant

ENVIRONMENTAL LINKAGE
□ Mantel test run: community distance ~ environmental distance
□ Partial Mantel run for key predictors to isolate independent effects
□ Number of permutations stated (≥999; ≥9999 for final publication)

DIFFERENTIAL ABUNDANCE
□ BH FDR correction applied across all tests, not per comparison
□ Effect size (Cliff's delta or similar) reported alongside p-value
□ Direction of enrichment stated explicitly
□ n significant / n tested reported (not just "many were significant")

REGRESSION
□ Response variable log-transformed and distribution checked
□ VIF checked — no predictor > 5–10
□ All four diagnostic plots examined (residuals, QQ, scale-location, histogram)
□ Shapiro-Wilk and Breusch-Pagan formal tests run
□ Coefficients translated into biological units (not just β = 0.27)
□ Adjusted R² reported, not R²

WGCNA
□ Soft threshold chosen from scale-free topology plot — not copied from tutorial
□ mergeCloseModules run — cutHeight justified
□ Grey module investigated, not discarded
□ Module-trait heatmap is the primary results figure, not the dendrogram alone

REPORTING
□ Every p-value accompanied by effect size
□ Multiple testing correction method stated for each set of tests
□ R package versions recorded (sessionInfo())
□ Seed set for all permutation tests (set.seed())
□ Code deposited before submission
```

---

## Common Pitfalls of the Full Workflow

▶ **Running every test available.** If you run PERMANOVA, Wilcoxon, regression, Mantel, and WGCNA without a connecting hypothesis, you are generating a list of numbers, not a scientific argument. Each test should answer a question the previous test raised.

▶ **Using PERMANOVA without betadisper.** PERMANOVA tests centroid differences and dispersion differences simultaneously. If your groups differ in spread rather than location, the significant p-value is misleading. Always run `betadisper` first and report the result, even if it is non-significant.

▶ **Treating the Mantel test as redundant.** "We already have PERMANOVA" is not a reason to skip Mantel. PERMANOVA tests discrete group membership. Mantel tests whether the continuous gradient in community composition tracks the continuous gradient in environment. These answer different questions and can produce opposite answers in the same dataset.

▶ **Reporting R² without translating coefficients.** R² = 0.76 in the FRed model sounds impressive. But the biological interpretation — temperature alone accounts for the dominant share of that variance (p < 0.007, FL fraction R² = 0.91) — is what makes the result ecologically meaningful. Always translate.

▶ **Skipping the FRed correlation with richness.** In our data, FRed correlated with species richness (r = 0.47) but not with functional diversity (r = 0.13). If we had only reported the regression model, we would have missed this: more diverse communities are not necessarily more functionally diverse — they are more redundant. That distinction matters for resilience prediction.

▶ **Concluding that high FRed = high resilience.** It does not, necessarily. High FRed means many taxa share functions — but if a perturbation selects against the shared function itself (rather than individual taxa), high FRed provides no buffer. Our paper explicitly discusses this: low FRed spring communities, which show higher functional diversity, may actually be more vulnerable to functional losses despite their diversity.

---

## The Complete Series in One Paragraph

Day 1 established why normal statistics fail on microbiome data. Day 2 showed how to visualise community structure with ordination. Day 3 tested whether that structure is real with PERMANOVA. Day 4 identified which individual MAGs drive it with Wilcoxon and Cliff's delta. Day 5 quantified how strongly abundance tracks continuous gradients with regression. Day 6 revealed that most of those MAGs form one coherent guild using WGCNA. Day 7 connects all of it into the analytical workflow that produced a published paper — and gives you the checklist to repeat it on your own data.

The dataset was 61 MAGs, 27 metagenomes, two estuaries, two seasons. The workflow is not specific to this dataset. Every step applies to 16S amplicon data, shotgun metagenomics, metatranscriptomics, or any table of samples × features × environmental metadata.

The statistics are the last thing you design. The biological questions come first.

---

## What I Would Do Differently

▶ Run the Mantel test earlier. We computed it during revision, not during the initial analysis. It would have anchored the regression results more clearly from the start.

▶ Pre-register the analysis plan before looking at the data. Deciding between PERMANOVA and Mantel after seeing that one gives p = 0.04 and the other p = 0.06 is not rigorous. Write the analysis plan first.

▶ Save `sessionInfo()` output with every script. Package versions in WGCNA changed between R 4.2 and 4.3 in ways that silently affected TOM computation. We caught this during review. A saved `sessionInfo()` would have caught it earlier.

---

_Code, data, all results: [github.com/jojyjohn28/microbiome_stats](https://github.com/jojyjohn28/microbiome_stats)_

_All R code and data: [github.com/jojyjohn28/microbiome_stats](https://github.com/jojyjohn28/microbiome_stats)_  
\_ Read full paper here : [Functional Redudancy_John et al,2026](https://academic.oup.com/ismecommun/advance-article/doi/10.1093/ismeco/ycag021/8454622)

---

![WGCNA MAG Co-Abundance Networks](/assets/img/d8-summary.png)
