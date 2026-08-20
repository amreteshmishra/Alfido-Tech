# Alfido-Tech

# Zomato Bangalore — Restaurant & Ratings Analysis

**Data cleaning, exploratory insights, and platform recommendations**

*Prepared for an Alfido Tech–style analytics platform*
*Source: zomato.csv (Bangalore restaurant listings)*

---

## Executive Summary

This report analyses restaurant listings scraped from Zomato for Bangalore, covering ratings, cuisines, locations, pricing, and service features (online ordering, table booking). The raw file contained a significant data-quality problem — roughly half of all rows had review text bleeding into structured columns — which was detected and cleaned before analysis. The resulting dataset of just over 10,000 unique restaurants shows clear patterns: certain premium central neighbourhoods and mid-to-niche cuisines (Continental, Cafe, Desserts, Asian) consistently out-rate the high-volume categories (North Indian, Fast Food, Biryani); price and table-booking availability are the strongest correlates of a higher rating; and restaurant supply is heavily concentrated in a handful of outer-Bangalore micro-markets that don't necessarily rate the best. Five recommendations for a food-discovery / analytics platform are provided at the end of this report.

---

## 1. Data Cleaning

### 1.1 The corruption problem

The source CSV had 13 well-formed columns per row structurally (every row parsed to exactly 13 fields), but for a large share of records the actual *values* were shifted — fields such as `rate`, `location`, or `listed_in(type)` contained fragments of restaurant review text rather than the values those columns should hold. This is an upstream data-generation issue, not a parsing artifact.

Each structural column was validated against its expected pattern (e.g. `rate` should match `4.1/5`, `NEW`, `-`, or be blank; `online_order`/`book_table` should be `Yes`/`No`; `listed_in(type)` should be one of seven known categories). Rows failing any check were treated as unrecoverable and dropped rather than guessed at.

| Step | Rows |
|---|---|
| Raw rows | 56,252 |
| Structurally valid (kept) | 29,549 (52.5%) |
| After removing exact duplicate listings | 10,296 |

### 1.2 Field-level cleaning

- **Rating:** extracted numeric value from `X.X/5` strings; `NEW` and `-` treated as missing.
- **Votes:** converted to integer.
- **Cost for two:** stripped thousands-separator commas and converted to numeric Rs. value (already in Indian Rupees — no currency conversion required).
- **Text fields** (name, location, rest_type, cuisines, dish_liked): whitespace-trimmed; empty/`nan` strings standardised to missing.
- **Duplicates:** exact duplicate restaurant listings (same name + address, arising because a restaurant can appear once per service category) were dropped, leaving one row per unique restaurant.

### 1.3 Feature engineering

- **primary_cuisine** / **primary_rest_type** — first cuisine / restaurant-type tag from each comma-separated list.
- **n_cuisines** — count of cuisines offered.
- **cost_bucket** — price bracket (less than Rs.300, Rs.300–600, Rs.600–1000, Rs.1000–1500, Rs.1500–2500, Rs.2500+).

---

## 2. Exploratory Analysis

### 2.1 Rating distribution

Across 10,296 cleaned restaurant listings, the average rating is **3.62 / 5**. Ratings cluster tightly between 3.0 and 4.3 — very few restaurants score below 2.5 or above 4.5, suggesting rating inflation/compression typical of platform review systems.

![Distribution of Restaurant Ratings](images/01_rating_distribution.png)

### 2.2 Location hotspots

Restaurant supply is heavily concentrated in a handful of outer-Bangalore neighbourhoods — Electronic City, BTM, HSR, Whitefield and JP Nagar together account for a large share of all listings.

![Top 15 Locations by Number of Restaurants](images/02_top_locations_count.png)

However, the highest *average* ratings are found in more central, premium areas rather than the highest-supply ones: Lavelle Road (4.04), St. Marks Road (3.93), Church Street (3.93), and Koramangala 5th Block (3.88) lead among locations with at least 30 restaurants — while high-volume Electronic City and Marathahalli average closer to 3.4–3.5 in cuisine-specific cuts (see heatmap, section 2.5).

### 2.3 Cuisine popularity vs. quality

North Indian, South Indian, Biryani, Fast Food and Chinese dominate by sheer restaurant count.

![Top 15 Primary Cuisines by Number of Restaurants](images/03_top_cuisines_count.png)

But popularity and rating quality diverge: niche/premium cuisines rate noticeably higher on average than the high-volume staples.

![Average Rating by Cuisine](images/04_avg_rating_by_cuisine.png)

*Asian (3.98), American (3.97) and Continental (3.82) lead; Biryani (3.49), Fast Food (3.48) and Rolls (3.45) trail, despite far higher restaurant counts.*

### 2.4 Price vs. rating

Cost for two and rating are positively correlated (Pearson r ≈ 0.30) — pricier restaurants tend to be rated higher, likely reflecting better service, ambience, and food consistency at higher price points.

![Cost for Two vs Rating](images/05_cost_vs_rating.png)

![Average Rating by Price Bracket](images/06_rating_by_price_bracket.png)

*Average rating rises steadily with price bracket, from 3.55 (under Rs.300) to 4.19 (Rs.2500+ for two).*

### 2.5 Location × cuisine hotspot map

Cross-referencing the 12 highest-supply locations against the 8 most common cuisines reveals where specific cuisine types under- or out-perform locally — useful for both restaurant partnerships and consumer guidance.

![Avg Rating Heatmap: Top Locations x Top Cuisines](images/08_location_cuisine_heatmap.png)

*Indiranagar and Jayanagar rate strongly across most cuisines; Fast Food underperforms almost everywhere, most sharply in Sarjapur Road and Bannerghatta Road.*

### 2.6 Service features: online ordering & table booking

Restaurants offering table booking average **4.07** vs. **3.57** for those that don't — the single largest gap found in this analysis. Online order availability shows a smaller, still-positive gap (3.64 vs. 3.58).

![Rating by Online Ordering and Table Booking](images/07_service_features_rating.png)

### 2.7 Restaurant format

Format is a stronger differentiator than cuisine alone: Fine Dining (4.14), Pubs (3.98) and Lounges (3.84) rate best; Food Courts (3.43), Mess (3.46) and Takeaway (3.49) rate lowest.

![Average Rating by Restaurant Type](images/10_rating_by_rest_type.png)

### 2.8 Correlation overview

![Correlation Matrix](images/09_correlation_heatmap.png)

*Votes, cost, and rating are all mildly-to-moderately positively correlated; number of cuisines offered has little relationship with rating.*

### 2.9 What are people ordering? (Word clouds)

![Most Popular Dishes](images/11_wordcloud_dishes.png)

*Most frequently liked dishes across all restaurants.*

![Most Common Cuisines](images/12_wordcloud_cuisines.png)

*Most common cuisine tags across all restaurants.*

---

## 3. Key Findings

- **Data quality:** ~53% of raw rows were structurally corrupted (fields shifted by embedded review text) and had to be dropped; the cleaned dataset covers 10,296 unique restaurants.
- **Ratings skew positive but rarely excellent:** average 3.62/5, tightly clustered between 3.0–4.3.
- **Supply ≠ quality:** the highest-volume locations (Electronic City, Marathahalli) are not the highest-rated; premium central areas (Lavelle Road, Church Street, Koramangala) rate best.
- **Niche cuisines outperform staples:** Continental, Cafe, Desserts, Italian and Asian rate higher on average than North Indian, Fast Food and Biryani, despite far smaller restaurant counts.
- **Price correlates with rating** (r ≈ 0.30) — restaurants above Rs.1,000 for two average well over 4.0.
- **Table booking is the strongest service-feature signal** — a +0.50 average rating gap versus restaurants without it.
- **Format matters more than cuisine** — Fine Dining, Pubs and Lounges rate highest; Quick Bites, Takeaway and Food Courts rate lowest.

---

## 4. Recommendations for an Alfido-Tech-style Platform

### 1. Target partnerships in high-quality, mid-supply micro-markets
Prioritise onboarding drives in areas like Koramangala (4th/5th/6th Block), Indiranagar and Jayanagar, which combine solid restaurant density with above-average ratings — a more efficient partnership ROI than oversaturated, mixed-quality zones like Electronic City.

### 2. Build a "Hidden High-Raters" discovery feature
Surface Continental, Asian, Italian, Cafe and Dessert concepts — cuisines that consistently out-rate the market despite much lower restaurant counts — giving users better choices and giving smaller/niche restaurants visibility disproportionate to their size.

### 3. Launch price-tiered content and curated collections
Since rating rises steadily with price bracket, replace generic "best restaurants" lists with budget-aware content ("Best under Rs.500", "Worth the splurge: Rs.1500+") so users across spending levels get genuinely well-matched, well-rated recommendations.

### 4. Incentivise table-booking and online-order adoption among partners
Table booking shows the single largest rating association in this dataset (+0.50 points on average). An onboarding programme that helps restaurants enable booking (and, to a lesser extent, online ordering) could lift marketplace-wide quality and user trust.

### 5. Operationalise the location × cuisine heatmap
Use heatmaps like the one in Section 2.5 to guide where to court new restaurant partners (e.g. better Continental/Cafe options in under-served but strong locations) and where to flag weak spots to users (e.g. Fast Food in Sarjapur Road and Electronic City rates poorly and may need quality-improvement outreach).

---

*End of report — see accompanying Jupyter notebook (`Zomato_Analysis.ipynb`) for the full, reproducible code and cleaned dataset.*


# Loan Approval Model — Summary Report

**Dataset:** 614 historical loan applications, 12 borrower/application features, target = `Loan_Status` (approved/denied). Base rate: 69% approved, 31% denied.

## What we built

A preprocessing pipeline (median/mode imputation, one-hot encoding, standard scaling) feeding six model configurations — Logistic Regression, Decision Tree, Random Forest, and XGBoost — each combined with either **class-weighting** or **SMOTE** to correct for the moderate class imbalance. Models were compared on a held-out 20% test set and validated with 5-fold cross-validation.

## Results

| Model | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|
| **Logistic Regression (SMOTE)** | 0.862 | 0.882 | 0.872 | **0.875** |
| Logistic Regression (class-weighted) | 0.872 | 0.882 | 0.877 | 0.856 |
| Random Forest (SMOTE) | 0.837 | 0.906 | 0.870 | 0.811 |
| Random Forest (class-weighted) | 0.839 | 0.918 | 0.876 | 0.809 |
| XGBoost (scale_pos_weight) | 0.843 | 0.824 | 0.833 | 0.780 |
| Decision Tree (class-weighted) | 0.822 | 0.871 | 0.846 | 0.735 |

*(Precision/Recall/F1 reported for the "approved" class at the default 0.5 threshold.)*

**Logistic Regression came out on top**, and by a meaningful margin on ROC-AUC. This is a signal that the relationship between the features — especially credit history — and the approval outcome is close to linear, so the added flexibility of tree ensembles doesn't pay off on a dataset this small (491 training rows). 5-fold cross-validation confirmed the result wasn't a fluke of one particular train/test split.

## What actually drives approvals

`Credit_History` is by far the strongest predictor in every model tested — applicants with a documented good credit history are approved at a much higher rate, independent of income or requested loan amount. `Property_Area` (semi-urban applicants tend to do somewhat better than rural ones) and marital status have smaller but consistent effects. Income and loan amount carry surprisingly little independent signal on their own — likely because affordability screening already happened before this data was recorded, filtering out applications where the loan amount didn't fit the applicant's income.

## Trade-offs and threshold recommendation

Model precision and recall trade off directly against the decision threshold used to convert predicted probability into an approve/deny call:

- **Raising the threshold** (e.g., to ~0.6–0.65) increases precision — fewer risky loans get funded — at the cost of recall: more legitimately good applicants get declined or pushed to manual review.
- **Lowering the threshold** (e.g., to ~0.4–0.45) does the opposite: more good applicants get through, but more bad loans slip in.

Which direction to move depends on which mistake costs the business more. For most lenders, a bad loan (false approval) is more expensive than a missed good customer (false denial), so we recommend:

- **Auto-approve above ≈ 0.6** predicted probability.
- **Auto-deny below ≈ 0.35.**
- **Route the ≈ 0.35–0.6 band to manual underwriting** — this is where the model is least confident and most error-prone, and where human judgment adds the most value per case reviewed.

This keeps the model's job to what it's actually good at (confidently sorting the clear cases) rather than forcing a single binary cutoff onto every application.

## Caveats before production use

- **Small sample size.** 614 rows total means test-set metrics carry real sampling noise; the 5-fold CV results should be treated as the more reliable estimate, and both should be revisited as more data accumulates.
- **Fairness review needed.** Since the model leans heavily on credit history, and credit history itself can encode historical lending bias, we'd recommend an explicit fairness audit (approval/error rates broken out by gender, marital status, and property area) before this model influences real decisions.
- **The model imitates past decisions, not necessarily optimal ones.** Training labels come from the lender's own historical approvals, so the model will reproduce whatever patterns — good or bad — were already present in that process.

# Alfido Tech — Instagram Content & Engagement Strategy

*One-page recommended posting plan, based on internal engagement analysis + 2026 industry posting-time benchmarks*

## What our data shows

- Photo and carousel posts out-perform standalone video on engagement rate; lean on carousels for tutorials/tips and photos for product/lifestyle shots.
- Filtered posts modestly out-perform unfiltered — keep a light, consistent visual treatment/filter across posts.
- Hashtag volume barely moves engagement — topic relevance and subject matter drive it far more than tag count. Use 3–5 targeted tags, not 10+.
- Account growth has been steady and roughly linear — there's no seasonal spike to chase; growth compounds from consistency, not one viral moment.
- Verified / high-activity accounts post substantially more often than average — posting frequency itself is a growth signal, not just content quality.

## Recommended weekly content calendar

| Day | Time window* | Format to prioritize | Why |
|---|---|---|---|
| Mon | Light day | Behind-the-scenes Story | Warm the audience back up after the weekend |
| Tue | 12–2 PM & 6–8 PM | Carousel (tips/tutorial) | Midweek lunch + evening peaks; carousels reward the longer dwell time |
| Wed | 12 PM & 6 PM | Reel (product/demo) | Consistently the single strongest day across 2026 industry studies |
| Thu | 9 AM & 4–5 PM | Reel or feed photo (case study) | Second-strongest day; morning + late-afternoon double peak |
| Fri | 10–11 AM | Community post / UGC repost | Engagement tapers by afternoon — post early, keep it light |
| Sat | 11 AM | Light/skip | Lowest-engagement day across all studies — low effort only |
| Sun | 12–3 PM | Weekly recap carousel | Planning-ahead mindset; good for save-worthy recap content |

*\*Time windows are 2026 cross-industry Instagram benchmarks (Buffer, Sprout Social, Hootsuite, Later — 19M+ posts analyzed), since Alfido Tech's own historical post-time data isn't yet available for a native analysis. Validate against Alfido Tech's own Insights once 4–6 weeks of data accumulate, and shift the schedule to match.*

## 5 strategies to increase engagement

1. **Shift format mix toward carousels & photos.** Cut standalone video's share of the calendar and reallocate 1–2 slots/week to carousels — our data's clearest content-type lever.
2. **Standardize a visual filter/preset.** A consistent, on-brand filter tracked with higher engagement here and builds recognizable feed identity — pick one look and use it across 80%+ of posts.
3. **Trade hashtag quantity for hashtag precision.** Replace generic high-volume tags with 3–5 tags tightly matched to post topic/location — density didn't correlate with engagement, relevance did.
4. **Post on a fixed midweek-heavy cadence (Tue/Wed/Thu anchors).** Concentrate the best content in the Tue–Thu window identified above, and use Mon/Fri/weekend slots for lighter, lower-effort formats.
5. **Track engagement rate (not raw likes) per post, segmented by format and day, monthly.** Once Alfido Tech has its own 4–6 weeks of Insights data, replace the benchmark time windows above with an internal analysis for a fully data-driven schedule.




*Full code, plots, and threshold sweep are in the accompanying notebook (`loan_approval_modeling.ipynb`).*
