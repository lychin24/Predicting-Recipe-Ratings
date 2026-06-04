# Investigating what types of recipes tend to have higher average ratings

Author: Lydia Chin

## Overview

## Introduction

For this project, I am analyzing two datasets consisting of recipes and ratings posted since 2008 on [food.com](https://www.food.com/). This dataset was originally used for the recommender system research paper, [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder et al.

The first dataset, `RAW_recipes`, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

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
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                         |

The second dataset, `interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

## Exploratory Data Analysis

In order to perform accurate analyses on these datasets, I cleaned them in the following manner: 

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This allows us to use both recipe data and their review for analysis


1. Fill all ratings of 0 with np.nan.

   - Rating is generally on a scale from 1 to 5, 1 meaning the lowest rating while 5 means the highest rating. With that being said, a rating of 0 indicates missing values in rating. Thus, to avoid bias in the ratings, I filled the value 0 with np.nan.

1. Add column `'avg_rating'` containing average rating per recipe.

   - Since a recipe can have numerous ratings from different users, I take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe.

1. Rename columns to better represent their values

1. Changed datatypes of selected columns

1. Added `'recipe_age'` column containing days since recipe submission


1. Split values in the nutrition column to individual columns of floats.

   - Even though the values in the nutrition column look like a list, they are actually objects, which act like strings. Given by the description of the columns of the recipe dataset, I know what each individual values inside the brackets mean. To split up the values, I applied a lambda function then converted the columns to floats. It will allow us to conduct numerical calculations on the columns.

1. Removed rows with a recpie preparation time longer than 1 week

#### Result
The resulting dataframe contains 234208 rows and 19 columns. Here are all of its features:
| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'recipe_id'`           | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'recipe_age'`          | int64          |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'calories'`            | float64        |
| `'total fat'`           | float64        |
|  `sugar'`               | float64        |
| `'sodium'`              | float64        |
| `'protein'`             | float64        |
| `'saturated fat'`       | float64        |
| `'carbohydrates'`       | float64        |
| `'review'`              | object         |
| `'avg_rating'`          | float64        |
