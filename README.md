# What Makes a Bengaluru Restaurant Highly Rated and Popular?

*Business Analytics (23CSE452) — Individual Case Study*
*Durgam Poojitha | CB.SC.U4CSE23320 | CSE-D*
*Amrita School of Computing, Coimbatore*

## Problem Statement

Restaurant owners and delivery platforms typically rely on a single overall rating that
blends two very different things: the dine-in experience (ambience, seating, table
reservations) and the delivery/takeout experience (packaging, online ordering). Two
restaurants can end up with the same rating for entirely different underlying reasons,
which means the rating alone doesn't tell an owner what to fix. It also says nothing
about **popularity** — how many people are actually reviewing the restaurant. Without
separating what drives rating from what drives popularity, an owner can end up investing
in the wrong thing, such as renovating the dining area when the real problem is delivery
service. This is a common issue for smaller restaurant businesses that can't afford
formal market research.

## Objectives

- Determine which restaurant features – price, review quantity, cuisine, location and
  All the features of a service (parking, reservations, etc.) — are statistically
  The higher the customer rating, the more associated they are with this.
- Identify if the same features that contribute to a rating contribute to popularity
  Whether it is volume of review (as measured by the number of times the book is read) or something else.
- Construct and use classification models to predict if a restaurant will be successful or not.
  They should be highly rated, and rank them to see which restaurant features are most important.

## Dataset and Data Collection

- **Domain:** Restaurant and food-service industry
- **Location:** Bengaluru, Karnataka
- **Source:** Google Maps restaurant listings
- **Collection method:** Web scraping (Apify Google Places Scraper), output as JSON
  and converted to CSV, since a direct CSV export dropped columns and records
- **Raw dataset:** 10,164 restaurant records, 24 attributes
- **Cleaned dataset:** 6,335 restaurants, 35 columns

This is an original dataset that was collected specifically for this case study directly from
Google Maps, not a pre-existing dataset from a repository such as Kaggle or UCI.

Restaurants with no or no reviews were eliminated, as they do not offer any
rating signal and only add noise — this took the dataset from 10,164 to 6,335 rows.
**The report has values taken approximately**

**Key variables:**

| Variable | Description |
|---|---|
| `totalScore` | Average customer rating out of 5 ("Rating") |
| `reviewsCount` | Number of customer reviews (proxy for popularity) |
| `price` | Listed price range, later cleaned into `price_numeric` |
| `imagesCount` | Number of customer-uploaded photos |
| `categoryName` | Cuisine / category |
| `neighborhood` | Location |
| `Features` | Free-text amenities, later split into 9 binary service columns |

## Data Preprocessing and Feature Engineering

- Filtered out restaurants that have no or missing reviews (10,164 → 6,335 rows, ~38%)
  No exact duplicates found (removed).Used the convert_price_range function to convert price ranges (e.g. ₹200-400) into a numeric `price_numeric`: midpoint
  The distribution is skewed and the median (₹300) was used to fill in the missing prices: - Filled 1,622 missing prices with the median (₧300), since the distribution is skewed
  right-skewed
- Made a new variable, called High Rated, which takes the value of 1 if Rating ≥ 4.0 or 0 if Rating < 4.0
- Converted the free-text column called Features into nine binary service columns:
  Reservations, Parking, Delivery, Outdoor Seating, Takeout, Vegetarian Options,
  Wheelchair Accessible, Family Friendly, Pet Friendly
It is found that the distribution of classes is as follows: 71.3% are High Rated and 28.7% are not High Rated.


## Analytics Methods

**Exploratory Data Analysis**
- Rating distribution
- Price distribution
- Review count distribution
- Correlation heatmap (core numeric variables only, since Pearson correlation requires
  numeric inputs)

**Statistical Analysis**
- Chi-Square tests — cuisine and neighbourhood vs. High Rated
- Independent t-tests — each service feature vs. rating
- ANOVA — popularity, price, and rating comparisons across groups

**Machine Learning**
Three classification models were trained and compared:
- Logistic Regression
- Decision Tree
- Random Forest

Implementation details:
- 75/25 stratified train-test split (4,751 train / 1,584 test)
- Feature standardisation for Logistic Regression
- GridSearchCV with 5-fold cross-validation for hyperparameter tuning
- Class-weight balancing and threshold tuning to address class imbalance
- ROC-AUC used as the primary comparison metric

Feature importance was examined using four independent methods: Logistic Regression
coefficients, Decision Tree importance, Random Forest importance, and Permutation
Importance.

## Model Results

| Model | Accuracy | Precision | Recall | F1 Score | AUC |
|---|---|---|---|---|---|
| Logistic Regression | 0.713 | 0.713 | 1.000 | 0.832 | 0.551 |
| Decision Tree | 0.698 | 0.710 | 0.972 | 0.821 | 0.570 |
| Random Forest | 0.711 | 0.713 | 0.996 | 0.831 | 0.593 |

Accuracy alone is not a reliable metric here because of the 71.3% class imbalance — a
model that always predicts "High Rated" would already score close to 71% without
learning anything. AUC was therefore used as the main comparison metric. **Random
Forest achieved the highest AUC (0.593)** among the three models. Hyperparameter
tuning (GridSearchCV, 5-fold CV) raised the cross-validated AUC to 0.641, but test-set
AUC improved only slightly, from 0.593 to approximately 0.596. This indicates that the
available structured listing features have limited predictive power for identifying
highly rated restaurants.


## Key Findings / Business Insights

- Rating has only weak linear relationships with price, review count, and image count.
- Parking showed the widest rating gap among the tested service features.
- Reservations was the most consistently important service feature across all four
  feature-importance methods.
- Review count ranked highly in feature-importance analysis despite a weak direct
  correlation with rating.
- Delivery is associated much more strongly with popularity/review volume than with
  rating.
- Cuisine and neighbourhood did not show statistically significant relationships with
  the High Rated outcome.
- High popularity was associated more strongly with price and service availability
  than with rating.
- The classification models improved only modestly over the majority-class baseline.

## Comparison with Existing Studies

**1. Choudhary et al. (2021)** — Zomato listings in Bengaluru, using Linear Regression,
Decision Tree, and Random Forest (reported accuracy: Random Forest 87%, Decision Tree
85%, Linear Regression 24%). These accuracy figures are not directly comparable to this
study's results, since this case study emphasizes AUC specifically to account for class
imbalance, which raw accuracy does not.

**2. Li, Yu, Li & Gao (2023)** — Aspect-Based Sentiment Analysis combined with a machine
learning conditional survival forest, applied to online restaurant reviews. This study
uses richer, review-text-derived information, compared with the structured listing
metadata (price, service flags, review counts) used in this case study.

**3. Li & Hecht (2021)** — A cross-platform comparison of Google Maps and Yelp
restaurant ratings. Relevant for understanding platform-level differences in observed
ratings, since this case study's data comes from Google Maps alone.

## Conclusion

A total of 6,335 restaurants in Bengaluru were analysed in this case study obtained from Google Maps.
Structured listing variables and service features give some helpful hints; but they have
Limited ability to predict highly rated restaurants. Random Forest was
the best-performing model was the one with an AUC of 0.593 (around 0.596 after
hyperparameter tuning). Examine volume and chosen service characteristics — including
The most important and consistent signals across were Parking and Reservations.
multiple feature-importance methods. Future work might include review of the findings.
Use sentiment/text analysis to monitor sentiment over time, and differentiate dine-in and
delivery ratings

## References

[1] Choudhary, N., Panwar, V., Mittal, S., & Sahu, G. (2021). Zomato Restaurants Data
Analysis Using Machine Learning Algorithms. *JETIR*, 8(2).

[2] Li, H., Yu, B. X. B., Li, G., & Gao, H. (2023). Restaurant survival prediction
using customer-generated content: An aspect-based sentiment analysis of online
reviews. *Tourism Management*, 96, 104707.

[3] Li, H., & Hecht, B. (2021). 3 Stars on Yelp, 4 Stars on Google Maps: A
Cross-Platform Examination of Restaurant Ratings. *Proceedings of the ACM on
Human-Computer Interaction*, 4(CSCW3), Article 254, 1-25.

[4] Apify. Google Maps Scraper (Google Places Crawler). Apify Store.

[5] Google Maps restaurant listings, Bengaluru, Karnataka, India — primary data
source for this case study.

## Repository Contents

- `README.md` — this file
- `restaurant_preprocessing.ipynb` — data cleaning and feature engineering notebook
- `restaurant_eda_analysis.ipynb` — exploratory analysis, statistical tests, and
  machine learning notebook
- `restaurants_10000.csv` — raw dataset before preprocessing
- `restaurant_cleaned.csv` — cleaned dataset used for analysis and modelling
- `Business_Analytics_Case_Study_Report.pdf` — full case study report
