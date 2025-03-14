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

<iframe
  src="util/Protein Proportion.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

For this analysis, we examined the distribution of the rating of the recipe conditioned between the high protein recipes and low protein recipes. The graph below shows that recipes are more or less evenly rated regardless of the protein content.

<iframe
  src="util/Num recipe for rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

