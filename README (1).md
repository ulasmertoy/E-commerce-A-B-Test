# A/B Testing: Landing Page Conversion Analysis

An end-to-end A/B test analysis evaluating whether a redesigned landing page improves conversion rates for an e-commerce site. The project combines frequentist hypothesis testing with a Bayesian approach to deliver a business-ready recommendation.

## Project Overview

An e-commerce company designed a new landing page and ran a randomized experiment with ~294K users split into two groups:

- **Control group** — shown the old page
- **Treatment group** — shown the new page

**Research question:** Does the new landing page change the conversion rate?

- **H₀ (Null):** p₁ = p₂ — no difference in conversion rate between the two pages
- **H₁ (Alternative):** p₁ ≠ p₂ — a significant difference exists (two-tailed)
- **Significance level (α):** 0.05
- **Target power (1−β):** 0.80

## Tech Stack

- **Python 3** — core language
- **pandas** — data cleaning and manipulation
- **NumPy** — numerical computation
- **SciPy** — statistical tests (`norm`, `chi2_contingency`, `beta`)
- **Matplotlib** — visualization
- **Jupyter Notebook** — analysis environment

## Dataset

- **File:** `ab_data.csv`
- **Size:** ~294,000 rows before cleaning, 290,585 after
- **Columns:** `user_id`, `timestamp`, `group` (control/treatment), `landing_page` (old_page/new_page), `converted` (0/1)

## Methodology

The analysis follows a six-step structured pipeline:

### Step 1 — Data Loading
Load the raw dataset and perform an initial inspection.

### Step 2 — Data Cleaning
Two data-quality issues were identified and addressed:

1. **Mismatch records** (3,893 rows) — users in the control group who saw the new page, and vice versa. These violate the experimental design and are dropped rather than reassigned, because a contaminated user does not cleanly represent either group.
2. **Duplicate user IDs** — users who appear more than once. A/B tests require independent observations, so only the first record per user is kept.

Post-cleaning validation confirms zero mismatches, zero duplicates, and near-perfectly balanced group sizes (ratio 0.9997).

### Step 3 — Exploratory Data Analysis (EDA)
- Control conversion rate: **12.04%**
- Treatment conversion rate: **11.88%**
- Absolute difference: **−0.16 percentage points**
- Relative difference: **−1.31%**

Visualizations include a bar chart of conversion rates and a stacked bar chart of raw counts per group.

### Step 4 — Sample Size & Power Analysis
Before drawing conclusions, the test's ability to detect an effect is verified:

- A custom `calculate_sample_size()` function implements the two-proportion z-test sample-size formula.
- A `calculate_power()` function computes the actual statistical power given the observed sample.
- **Result:** The test has only **25.8% power** to detect the observed 0.16pp difference, but can reliably detect any effect ≥ 0.5pp. This framing is critical — a "not significant" result for such a small effect still provides meaningful business signal.

### Step 5 — Frequentist Hypothesis Testing
Two complementary tests are run:

| Test | Statistic | P-value | Result |
|------|-----------|---------|--------|
| Two-proportion Z-test | z = −1.31 | 0.1897 | Fail to reject H₀ |
| Chi-square test | χ² = 1.71 | 0.1916 | Fail to reject H₀ |

Supporting metrics:

- **95% Confidence Interval** for the difference: [−0.39pp, +0.08pp] — contains zero
- **Cohen's h:** −0.005 — negligible effect size

Both tests agree: no statistically significant difference between the old and new page.

### Step 6 — Bayesian A/B Test
The frequentist tests answer "reject or not." The Bayesian analysis answers a more actionable question: *"What is the probability that the old page is actually better?"*

- **Prior:** Beta(1, 1) — non-informative (uniform)
- **Posterior:** Beta(1 + conversions, 1 + non-conversions)
- **Simulation:** 100,000 Monte Carlo samples drawn from each posterior

| Metric | Value |
|--------|-------|
| P(old page is better) | **90.6%** |
| P(new page is better) | 9.4% |
| Expected loss — keeping old page | 0.00005 |
| Expected loss — switching to new page | 0.00164 |
| Mean relative lift | −1.31% |
| Lift 95% CI | [−3.25%, +0.66%] |

## Key Findings

1. The new landing page shows a small **negative** effect (−0.16pp) vs. the old page.
2. Frequentist tests **fail to reject** the null hypothesis — the difference is not statistically significant at α = 0.05.
3. Bayesian analysis estimates a **90.6% probability** that the old page outperforms the new one.
4. The **expected loss** of switching to the new page is roughly **30x higher** than staying with the old page.
5. Effect size (Cohen's h = −0.005) is negligible even if real.

## Business Recommendation

**Keep the old landing page.** The new design shows no measurable improvement and likely underperforms. Given the 90.6% posterior probability that the old page is better and the asymmetric expected-loss profile, rolling out the new page is not justified on the current evidence.

If product teams want to revisit this, the next iteration should either (a) target a larger, more meaningful design change capable of producing an effect ≥ 0.5pp, or (b) extend the experiment to gain power for smaller effects.

## Why This Project Matters

This notebook demonstrates several concepts that matter for real-world product analytics:

- **Data quality before inference** — mismatches and duplicates are handled explicitly rather than swept into the analysis.
- **Power analysis as context, not ceremony** — the report makes clear what the test *can* and *cannot* detect, avoiding the common mistake of treating a non-significant p-value as proof of no effect.
- **Frequentist + Bayesian side by side** — the Bayesian layer converts statistical output into a direct probability stakeholders can act on, plus an expected-loss calculation that frames the decision in business terms.
- **Reproducible simulation** — the Monte Carlo step uses a fixed random seed (`np.random.seed(42)`).

## How to Run

```bash
# Clone the repo
git clone <your-repo-url>
cd <repo-folder>

# Install dependencies
pip install pandas numpy scipy matplotlib jupyter

# Launch the notebook
jupyter notebook abtest.ipynb
```

Make sure `ab_data.csv` is in the same directory as the notebook.

## File Structure

```
.
├── abtest.ipynb     # Main analysis notebook
├── ab_data.csv      # Raw experiment data
└── README.md        # This file
```

