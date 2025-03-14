# Investigation on the Relationship Between Amount of Protein and Rating of Recipes

Authors: Vishal Rana & Akshay Uppal

## Overview

This data science project, conducted at UCSD, focus on exploring the relationship between the rating of a recipe and the proportion of sugar contributing to the total calories of the given recipe.

## Introduction

Food is a fundamental part of our daily lives, and for many, cooking is both a passion and a source of enjoyment. While some individuals actively seek high-protein meals for their health benefits, others may not prioritize protein content when choosing recipes. Since protein is essential for muscle growth, satiety, and overall well-being, it is worth exploring how it influences people's food preferences.

Our goal is to examine whether the protein content in a recipe affects its rating. Do people tend to rate high-protein recipes more favorably due to their nutritional value, or do other factors, such as taste and texture, have a greater impact? To answer this, we analyze two datasets containing recipes and user ratings from Food.com dating back to 2008.

The first dataset, `recipe`, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `RAW_interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Given the datasets, we are investigating whether people rate recipes with higher protein count and the recipes with lower protein count on the same scale.** We are investigating whether people rate high-protein and low-protein recipes differently. To support our analysis, we separated the values in the 'nutrition' column into distinct attributes, including 'calories (#)', 'total fat (PDV)', 'protein (PDV)', etc. PDV, or percent daily value, indicates how much a nutrient in a serving contributes to a total daily diet. Additionally, we calculated the proportion of protein-derived calories relative to the total calories in a recipe and stored this information in a new column, 'prop_protein'. Here, we define high-protein recipes as those with a 'prop_protein' value higher than the average 'prop_protein' across all recipes.

The most relevant columns for our investigation include 'calories (#)', 'protein (PDV)', 'prop_protein' (as defined above), 'rating' (the rating given by users for each recipe), and 'average rating' (the average of all ratings for each unique recipe).

By addressing this question, we aim to gain insights into user preferences for high-protein recipes, which could help contributors on Food.com refine and optimize their recipes to better align with public interest. Furthermore, our findings could pave the way for future research on how aware people are of the health benefits associated with higher protein consumption.

## Data Cleaning and Exploratory Data Analysis

To make our analysis of the dataset more efficient and convenient, we conducted the following data cleaning steps.

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This step helps match the unique recipes with their rating and review.


1. We checked the data types of all the columns.

   - This step helps us evaluate what columns require data type conversion
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |

1. Fill all ratings of 0 with NaN.

   - Rating is on a scale from 1 to 5. Hence, a rating of 0 indicates missing values, so we filled the value 0 with np.nan.

1. Split values in the nutrition column to individual columns of floats.

   - Even though the values in the nutrition column look like a list, they are actually objects, which act like strings. Given by the description of the columns of the recipe dataset, we know what each individual values inside the brackets mean. To split up the values, we applied a lambda function then converted the columns to floats. It will allow us to conduct numerical calculations on the columns.

1. Add `'is_dessert'` to the dataframe

   - `'is_dessert'` is a boolean column checking if the tags of recipes contain 'dessert' since desserts typically have a higher amount of sugar. This step separates the recipes into two groups, ones that are dessert and ones that are not. This provides us another way to compare ratings of recipes with more sugar and ones with less sugar.

1. Add `'protein_proportion'` to the dataframe
   - protein_proportion is the proportion of protein_pdv of the total pdv in a recipe. To calculate this, we use the values in the protein_pdv column to divide by sum of all other pdv columns. The experimentation allows us to understand the nutrition formula used on the website for recipes.
  
#### Result
Here are all the columns of the cleaned df.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'submitted'`           | datetime64[ns] |
| `'tags'`                | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'user_id'`             | float64        |
| `'recipe_id'`           | float64        |
| `'date'`                | datetime64[ns] |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'total_pdv'`           | float64        |
| `'protein_proportion'`     | float64        |


Our cleaned dataframe ended up with 234428 rows and 27 columns. Here are the first 5 rows of ~unique recipes of our cleaned dataframe for illustration. Since there is a lot of columns for the merged dataframe, we selected the columns that are most relevant to our questions for display. Scroll right to view more columns.

| name                                 |     id |   minutes | submitted           |   rating |   average rating |   calories (#) | protein (PDV) | High Protein | prop_sugar |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | True         |    0.361272  |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |    0.354562  |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |    0.0308008 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 00:00:00 |        5 |                5 |          878.3 |           326 | True         |    0.371172  |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 00:00:00 |        5 |                5 |          267   |            12 | False        |    0.0449438 |


### Univariate Analysis

For this analysis, we examined the distribution of the proportion of protein in a recipe. The first plot shows the ratings, displaying the number of ratings for each rating. As the second plot below shows, most of the recipes on food.com have a low proportion of protein. There is also a decreasing trend, indicating that as the proportion of protein in recipes gets higher, there are less of those recipes on food.com.

<iframe
  src="util/Ratings.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis

For this analysis, we examined the distribution of the rating of the recipe conditioned between the protein proportion and number of recipes. The graph below shows that recipes are more likely to have less protein content.

<iframe
  src="util/Protein Proportion.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

For this section, we investigated the relationship between the Protein content and rating of the recipes. First, we created a small pivot table, to store the ratings for high and low protein recipes. Later we visualized the table.

| High Protein | 1.0  | 2.0  | 3.0  | 4.0   | 5.0   |
|-------------|------|------|------|-------|-------|
| **False**   | 1764 | 1327 | 3789 | 19222 | 96638 |
| **True**    | 1106 | 1041 | 3383 | 18085 | 73038 |

 
According to the plot, recipes have a high rating could be either more protein containing.

<iframe
  src="util/Num recipe for rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Interestingly, the graph shows that as protein content increases, there is no significant change in the distribution of ratings. 

## Assessment of Missingness

Three columns, `'date'`, `'rating'`, and `'review'`, in the merged dataset have a significant amount of missing values, so we decided to assess the missingness on the dataframe.

### NMAR Analysis

We think the missing values in the 'review' column are Not Missing At Random (NMAR) because the likelihood of leaving a review depends on how strongly someone feels about a recipe. If a person feels neutral or indifferent about a dish, they may not see a reason to leave a review since they don’t have much to say. On the other hand, people who have strong opinions—whether positive or negative—are more motivated to go through the effort of visiting the page, clicking multiple buttons, and taking the time to write a review. For instance, someone who really enjoyed a recipe would likely be willing to go through this process to share their positive experience.

### Missingness Dependency

We moved on to examine the missingness of `'rating'` in the merged DataFrame by testing the dependency of its missingness. We are investigating whether the missiness in the `'rating'` column depends on the column `'protein_propotion'`, which is the proportion of protein out of the total pdv, or the column `'minutes'`, which is the number of minutes to make the recipe.

> Proportion of Sugar and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the proportion of protein in the recipe.

**Alternate Hypothesis:** The missingness of ratings does depend on the proportion of protein in the recipe.

**Test Statistic:** The absolute difference of mean in the proportion of protein of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe
  src="util/protein_proportion_comparison.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran a permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="util/protein_proportion_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **0.0052** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is < 0.05 which is the significance level that we set, we **reject the null hypothesis**. The missingness of `'rating'` does depend on the `'protein_proportion'`, which is proportion of protein in the recipe.

> Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the cooking time of the recipe in minutes.

**Alternate Hypothesis:** The missingness of ratings does depend on the cooking time of the recipe in minutes.

**Test Statistic:** The absolute difference of mean in cooking time of the recipe in minutes of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

Due to the outliers in cooking time, it is difficult to identify the shapes of the two distributions, so we update the scale to take a closer look.

<iframe
  src="util/minutes_comparison.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran another permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="util/minutes_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.497** is indicated by the red vertical line on the graph. Since the **p-value** that we found **(0.113)** is > 0.05 which is the significance level that we set, we **fail to reject the null hypothesis**. The missingness of rating does not depend on the cooking time in minutes of the recipe.

## Hypothesis Testing

As mentioned in the introduction, we are curious about whether people rate high-protein recipes and low-protein recipes on the same scale. Proportion of protein is referring to the values in `'protein_proportion'`.

To investigate the question, we ran a **permutation test** with the following hypotheses, test statistic, and significance level.

**Null Hypothesis:** People rate all the recipes on the same scale.

**Alternative Hypothesis:** People rate high-protein recipes higher than low-protein recipes.

**Test Statistic:** The difference in mean between rating of high-protein recipes and low-protein recipes.

**Significance Level:** 0.05

The reason we chose to run a permutation test is because we do not have any information of any population, and we want to check if the two distributions look like they come from the same population. We proposed that **people rate the high-protein recipes higher** because people might be concerned with the health benefits relating to the recipe, and we would like to know all the opinions from the users, so we used rating instead of average rating of the recipes. For the test statistic, we chose the difference in mean of the ratings of two groups of recipes instead of absolute difference in mean.

To run the test, we first split the data points into two groups, high-protein, which are recipes with high proportion of protein, and the rest of the data points are in the low-protein group. The **observed statistic** is **-0.027**.

Then we shuffled the ratings for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic. We got a **p-value** of **1.0**.

<iframe
  src="util/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Conclusion of Permutation Test

Since the **p-value** that we found **(1.0)** is not less than the significance level of 0.05, we **do not reject the null hypothesis**. People appear to rate all the recipes on the same scale. One plausible explanation for this founding could be that people are not very concerned with protein content when opposed to taste.

## Framing a Prediction Problem

We plan to **predict rating of a recipe** which would be a **classification problem** since rating is a categorical variable. To address our prediction problem, we will build a multi-class classifier since our ratings have 5 possible values that the model will predict from.

We chose to predict rating because it is a good representation of the overall recipe, and is responsive to features while only having 5 possible values.

To evaluate our model, we will use the RMSE and R^2, because the distribution for the ratings are heavily skewed left with most ratings concentrated in the higher ratings (4-5).

Before making our prediction, we rely on all the columns in the rating dataset, as listed in the introduction. These columns contain features specific to the recipes themselves, meaning we can access this information even if no one has rated or reviewed them yet.

## Baseline Model

For our baseline model, we are utilizing a Random Forest Regressor to predict recipe ratings. The dataset is split into training (80%) and test (20%) sets to ensure proper model evaluation.
The features used in this model are 'minutes' and 'n_steps', both of which are numerical variables representing the preparation time and the number of steps required for a recipe. These features are standardized using a StandardScaler, and we also apply PolynomialFeatures to capture potential non-linear relationships in the data.
To build the model, we define a Scikit-Learn pipeline, which includes:
1. Standard Scaling - Normalizing numerical features.
2. Polynomial Features - Expanding feature interactions.
3. Random Forest Regressor - A robust ensemble model for predictions.

After fitting the model, we evaluate its performance using Root Mean Squared Error (RMSE) and R² score. The baseline model achieved the following results:
- Baseline Model RMSE: 0.7192
- R² Score: -0.0077

The negative R² score suggests that the model performs worse than predicting the mean rating for all recipes. This indicates that our current feature set may not be strongly predictive of recipe ratings. Further feature engineering and hyperparameter tuning will be necessary to improve the model's performance.

## Final Model

For our final model, we aimed to improve upon the baseline model by refining our feature selection, transformation techniques, and hyperparameter tuning.
Baseline Model Recap
In our baseline model, we used minutes and n_steps as the features to predict the average rating of a recipe. We utilized a RandomForestRegressor as the model and conducted minimal feature engineering. The performance metrics for the baseline model were:
- Baseline RMSE: 0.7192
- Baseline R² Score: -0.0077

The negative R² value indicated that our baseline model performed worse than simply predicting the mean of the dataset. This suggested that our feature set and transformations were insufficient to capture the underlying patterns in the data.

Feature Selection & Transformations
To improve our model, we introduced additional engineered features and preprocessing transformations:
1. Steps per Minute (steps_per_minute):
   - This feature was computed as n_steps / minutes, capturing the density of steps in a given recipe.
   - We handled infinities and missing values by replacing them with 0.
2. Preprocessing Pipeline:
   - StandardScaler: Standardized steps_per_minute to normalize its distribution.
   - QuantileTransformer: Applied to n_steps and minutes to achieve a normal-like distribution.
   - PolynomialFeatures: Introduced non-linearity by adding polynomial interactions, with the optimal degree determined through GridSearchCV.

Hyperparameter Tuning
To optimize our RandomForestRegressor, we conducted GridSearchCV with cross-validation, testing:
- Polynomial degrees: 1, 2, 3
- Number of trees (n_estimators): 50, 100
- Maximum tree depth (max_depth): 5, 10, None
- Minimum samples required to split a node (min_samples_split): 2, 5

The best hyperparameters found were:
- ('model_max_depth': 10, 'model_min_samples_split': 2, 'model_n_estimators': 100, 'poly_degree': 2)

Using these optimized values, we trained our final model.

Final Model Performance
After applying these improvements, the final model achieved:
- Final RMSE: 0.7143
- Final R² Score: 0.0062

Compared to the baseline model, this represents:
- A reduction in RMSE (from 0.7192 to 0.7143), indicating an improvement in prediction accuracy.
- A shift from negative to slightly positive R² (from -0.0077 to 0.0062), meaning the model now performs marginally better than simply predicting the mean.

Final Thoughts
- While the improvement is modest, the refined feature engineering and hyperparameter tuning slightly enhanced prediction accuracy.
- The continued low R² suggests that recipe rating predictions are highly complex, potentially requiring additional features (such as ingredient compositions, user reviews, or textual recipe descriptions).
- Future work could explore more advanced models like Gradient Boosting (XGBoost, LightGBM) or Neural Networks to capture deeper patterns in the data.


## Fairness Analysis

For our fairness analysis, we split the recipes into two groups based on their preparation duration: short recipes and long recipes. We designated short recipes as those with a preparation time of ≤ mean duration and long recipes as those with a preparation time of > mean duration. The mean duration of all recipes in the test set was used as the threshold for this split.
We chose to analyze fairness by evaluating the Root Mean Squared Error (RMSE) for each group. RMSE provides insight into how well the model predicts ratings for short and long recipes. If our model exhibits fairness, the RMSE should be approximately equal across both groups.
To test this, we conducted a permutation test, where we randomly shuffled the duration labels and recalculated the RMSE difference multiple times to generate a null distribution. This allows us to determine whether the observed RMSE difference is statistically significant.

**Null Hypothesis**: Our model is fair. The RMSE for short and long recipes is approximately equal, and any observed difference is due to random chance.

**Alternative Hypothesis**: Our model is unfair. The RMSE for short recipes is significantly different from the RMSE for long recipes.

**Test Statistic**: Difference in RMSE between short and long recipes (RMSE_short- RMSE_long)

**Significance Level**: 0.05

After running the analysis, we obtained the following RMSE values:
- RMSE for short recipes (≤ mean duration): 0.7021
- RMSE for long recipes (> mean duration): 0.792
- Observed RMSE Difference: 0.7143
- Permutation Test p-value: 1.00

<iframe
  src="util/Fairness analysis1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To visualize the fairness analysis, we plotted a histogram of the permutation differences in RMSE. The observed difference is marked on the plot to compare it against the null distribution.
If the p-value is less than 0.05, we reject the null hypothesis and conclude that the model exhibits unfairness in predicting ratings based on recipe duration. Otherwise, we fail to reject the null hypothesis, meaning any differences in RMSE could be due to random variation.
