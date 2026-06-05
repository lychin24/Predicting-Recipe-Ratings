# Investigating what types of recipes tend to have higher average ratings

Author: Lydia Chin

## Overview

## Introduction
Recipes vary widely in their preparation time, nutritional content, and overall popularity. Understanding the factors that contribute to a highly rated recipe can provide valuable insights for home cooks, recipe developers, and food websites. For this project, I am analyzing two datasets consisting of recipes and ratings posted since 2008 on [food.com](https://www.food.com/). This dataset was originally used for the recommender system research paper, [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder et al.

Given this data, an overarching question I hope to answer is "What is the relationship between the cooking time and average rating of recipes?"

This question is interesting because cooking time often influences how appealing a recipe is to users. While some people may prefer quick and convenient meals, others may associate longer preparation times with higher-quality dishes. By examining the relationship between cooking time and average recipe ratings, we can better understand whether the amount of time required to prepare a recipe affects how users perceive and rate it.

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


To answer the given question, the most relevant columns are `'minutes'` from `RAW_recipes` and `'rating'` from `interactons` since they directly address the research quesiton. In addition to these two, `'n_steps'` and `'n_ingredients '` from `minutes` could potentially provide context about the recipe and it's complexity which could then be used to explore possible reasons for any observed relationships between cooking time and rating. 



## Exploratory Data Analysis

In order to perform accurate analyses on these datasets, I cleaned them in the following manner: 

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This allows us to use both recipe data and their review for analysis

1. Fill all ratings of 0 with np.nan.

    - Rating is generally on a scale from 1 to 5, 1 meaning the lowest rating while 5 means the highest rating. With that being said, a rating of 0 indicates missing values in rating. Thus, to avoid bias in the ratings, I filled the value 0 with np.nan.

1. Add column `'avg_rating'` containing average rating per recipe.

    - Since a recipe can have numerous ratings from different users, I take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe. After performing this action, I then dropped the original `'rating'` column as its data was better represented in the `'avg_rating'` column


1. Checked the datatypes of all columns and changed them for selected columns
    | Column             | Description |
    | :----------------- | :---------- |
    | `'name'`           | object      |
    | `'recipe_id'`      | int64       |
    | `'minutes'`        | int64       |
    | `'contributor_id'` | int64       |
    | `'submitted_date'` | object      |
    | `'tags'`           | object      |
    | `'nutrition'`      | object      |
    | `'n_steps'`        | int64       |
    | `'steps'`          | object      |
    | `'description'`    | object      |
    | `'ingredients'`    | object      |
    | `'n_ingredients'`  | int64       |
    | `'user_id'`        | float64     |
    | `'date'`           | object      |
    | `'review'`         | float64     |
    | `'avg_rating'`     | object      |

    - I changed `'recipe_id'`, `'contributor_id'`, and `'user_id'` to be strings as they are categorical variables despite their numerical appearance. I also changed 

1. Split values in the nutrition column to individual columns of floats.

    - Although the values in the nutrition column appear to be stored as lists, they are actually stored as objects (strings). Using the dataset documentation, I identified the meaning of each value within the brackets. I then applied a lambda function to extract each nutritional component into its own column and converted the resulting values to floats. This transformation allowed the nutritional information to be used in numerical calculations and data analysis.
    
1. Removed rows with a recipe preparation time longer than 1 week

    - As recipe preparation time is my main focus in this analysis, I do not want a small number of extreme values to disproportionately skew the results. Recipes with preparation times longer than one week are highly unusual and are unlikely to accurately represent typical cooking behavior. By limiting the dataset to recipes that take less than one week, I can reduce the influence of outliers and obtain a clearer picture of the relationship between cooking time and recipe ratings.


#### Result
- The resulting dataframe contains 234208 rows and 18 columns. Here are all of its features:
    | Column             | Description |
    | :----------------- | :---------- |
    | `'name'`           | object      |
    | `'recipe_id'`      | object      |
    | `'minutes'`        | int64       |
    | `'contributor_id'` | object      |
    | `'submitted_date'` | object      |
    | `'n_steps'`        | int64       |
    | `'steps'`          | object      |
    | `'description'`    | object      |
    | `'ingredients'`    | object      |
    | `'n_ingredients'`  | int64       |
    | `'calories'`       | float64     |
    | `'total fat'`      | float64     |
    | `'sugar'`          | float64     |
    | `'sodium'`.        | float64     |
    | `'protein'`        | float64     |
    | `'saturated fat'`  | float64     |
    | `'carbohydrates'`  | float64     |
    | `'review'`         | float64     |
    | `'avg_rating'`     | object      |


And here are the first five unique rows of it, including the most relevant columns as there are too many to all display.

|   recipe_id |   minutes |   n_steps |   n_ingredients |   calories |   avg_rating |
|------------:|----------:|----------:|----------------:|-----------:|-------------:|
|      333281 |        40 |        10 |               9 |      138.4 |      4       |
|      453467 |        45 |        12 |              11 |      595.1 |      5       |
|      306168 |        40 |         6 |               9 |      194.8 |      5       |
|      286009 |       120 |         7 |               7 |      878.3 |      5       |
|      475785 |        90 |        17 |              13 |      267   |      5       |

### Univariate Analysis

For further analysis of the data, I examined the distribution of the minutes column

<iframe
  src="images/minutes_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As you can see, because of the outliers, the distribution is difficult to see. To help, I also plotted the distribution scaled up.

<iframe
  src="images/minutes_dist_scaled.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

From this plot, I concluded that the distribution is skewed to the right meaning most recipes have a shorter cooking time. The density of the data also decreases as the minutes increases, indicating that as cooking time of a recipe gets higher, the amount of those recipes decreases. 

### Bivariate Analysis

I created a scatterplot to visualize the relationship between a recipe's average rating and the cooking time.

<iframe
  src="images/minutes_rating_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

However, this plot wasn't very helpful as the datapoints are clustered so tightly. Given this, I decided to also create a binned scatter plot to get a general idea of the rating for each recipe with a similar cooking time

<iframe
  src="images/mins_rating_bins.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot suggests that as the cooking time increases for recipes, the average rating decreases, especially for recipes that take 4-8 hours to cook. This could be explained by the fact that people might not want to spend long amounts of time preparing their meals, so a recipe that is fast to finish would be recieved and rated better than a recipe that takes longer. 

### Aggregate Analysis

 For further exploration of the relationship between cooking time and other features of the data, I grouped the recipes by cooking time and calculated the average rating, number of steps, and number of ingredients within each group. The resulting table is below:

| time_bin   |   avg_rating |   avg_steps |   avg_ingredients |
|:-----------|-------------:|------------:|------------------:|
| 0-15       |      4.71486 |     5.52554 |           6.47991 |
| 15-30      |      4.67837 |     9.24833 |           8.75278 |
| 30-60      |      4.66619 |    11.5552  |          10.003   |
| 1-2 hr     |      4.67641 |    13.0207  |          10.8929  |
| 2-4 hr     |      4.66316 |    13.6719  |          10.2993  |
| 4-8 hr     |      4.56732 |    11.2141  |           9.61101 |
| 8+ hr      |      4.62108 |    11.7323  |           9.81778 |

The results show that recipes requiring longer cooking times generally involve more steps and ingredients, indicating that they are more complex. However, average ratings remain relatively consistent across most groups, ranging from approximately 4.56 to 4.71. While there is a slight decline in average rating among some of the longest-duration recipes, the differences are small compared to the overall rating scale. These findings suggest that although longer recipes tend to be more complex, increased preparation time does not necessarily translate to substantially higher user ratings.

## Assessment of Missingness

### NMAR Analysis

I believe that the data in the `'review'` coulumn is likely to be Not Missing at Random (NMAR). If a person doesn't have any strong feelings for a recipe, they are less inclined to leave a review or rating. It is only when they have thoughts about the recipe that they believe others should hear, either negative or positive, that they will take the time out of their day to write and post a review. 

### Missingness Dependency

To further analyze the missingness of the `'avg_rating'`, I explored its dependency. To do so, I investigated whether the missingness of `'rating'` depends on how long a recipe takes to make or how many ingredients it uses. This mean performing a permutation test to ask whether the samples grouped by missingness came from the same underlying distribution

 > Ratings Missingness vs. Minutes

**Null Hypothesis:** The missingness of ratings does not depend on the time it takes to make the recipe in minutes

**Alternate Hypothesis:** The missingness of ratings does depend on the time it takes to make the recipe in minutes

**Significance Level**: 0.05

To first get an idea of the distributions of recipe preperation time grouped by whether the recipe's rating is missing or not, I plotted the distributions in a box plot, one fully and another just looking at where the center is. 

<iframe
  src="images/mins_missing_dist_small.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="images/mins_missing_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As shown in the plot, recipes with a missing data seem to have a similar center and quartiles, however the data without missing values seem to have a larger maximum and range of cooking times

Due to the skew of the data, I decided to use the test statistic Absolute Difference in Medians for the permutation test. As the median is not as sensitive to outliers as the mean is, using the median will allow me to get a more accurate result from the test. 

Using this test statistic, I ran a permutation test shuffling the missingness of the `'avg_rating'` column 5000 times to collect 5000 simulated differences in medians between the two distributions. 

<iframe
  src="images/mins_perm_med.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This test had an observed statistic of ___ as shown with the dashed line on the graph. The resulting p-value was 0.0 which is below the significance level. However, the resulting statistics seemed to almost all be zero. 

To make sure outliers weren't affecting the outcome, I also performed a permutation test with log minutes to hopefully reduce the power of the extreme values and the test statistic Absolute Difference in Means.

<iframe
  src="images/mins_perm_log.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This test had an observed statistic of ___ plotted on the graph and a p-value of ___. 
Since both tests returned a p-value below the significance level, I reject the null hypothesis and conclude that there is enough evidence that missingness in `'avg_rating'` is associated with recipe preparation time.
In other words, there is evidence that recipes with missing ratings tend to have different preparation times than recipes with observed ratings.

Therefore, the missingness mechanism for `'avg_rating'` is likely Missing at Random (MAR) dependent on `'minutes'`

> Ratings Missingness vs. N_Ingredients

**Null Hypothesis:** The missingness of ratings does not depend on the number of ingredients it uses

**Alternate Hypothesis:** The missingness of ratings does depend on the number of ingredients it uses

**Significance Level**: 0.05

Again, before performing any tests, I plotted the distributions in a box plot to get a sense of possible differences.

<iframe
  src="images/n_ing_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
Looking at this plot, the distributions for ingredient amount grouped by rating missingness look very similar.

Moving forward with the test, I used the test stastic Absolute Difference in Means as this column didn't have any extreme datapoints to be wary of

<iframe
  src="images/n_ing_perm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic of ___ as shown with the dashed line. The resulting p-value was ___ which is greater than our significance level of 0.05. This means I fail to reject the null hypothesis and conclude that the missingness of a recipe's rating doesn't depend on the number of ingredients it uses; or there isn't enough evidence to state otherwise.


## Hypothesis Testing
As stated previously, the question I'm exploring is does the average rating of a recipe depend on how long it takes to make? To answer this question, I performed a permutation test comparing the absolute difference in medians of average rating between recipes that take shorter than 35 minutes to make and recipes that took longer than 35 minutes. I chose to do a permutation test to test whether the two distribution seemingly came from the same population. To perform the test, I binarized the minutes column into 0 for less than 35 minutse and 1 for greater and shufffled them  then calculated the difference in medians between the two groups for each permutation, generating a null distribution of the test statistic. I chose to do difference in medians rather than means due to the skew of the data and picked a cutoff of 35 minutes because it was the median of the 'minutes' column. 

**Null Hypothesis:** Recipes requiring 35 minutes or less and recipes requiring more than 35 minutes come from the same rating distribution.

**Alternative Hypothesis:** The rating distributions differ.

**Test Statistic:** Absolute difference in median average rating.

**Significance Level:** 0.05



<iframe
  src="images/hyp_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic was 0.035. After shuffling the labels 5000 times and collecting 5000 simulated median distances. The resulting p-value was >0.01. This value is less than the significance level therefore I reject the null hypothesis. There is evidence that the length of time a recipe takes to make has an effect on its average rating. 




## Framing a Prediction Problem
For the prediction task, I chose to predict whether a recipe will receive a rating above 4.5 stars. This is a binary classification problem, where recipes with ratings greater than 4.5 are classified as "highly rated" and all others are classified as "not highly rated."

I selected 4.5 as the threshold because it represents the upper tier of the 1–5 rating scale. Additionally, recipe ratings in this dataset are heavily concentrated near the high end of the scale, making the distinction between highly rated recipes and all other recipes more meaningful than predicting exact rating values.

At the time of prediction, only information available when a recipe is posted will be used as predictors. Examples include preparation time, nutritional information, number of ingredients, and number of steps. User ratings and other information that would only become available after publication are excluded to avoid data leakage.

The primary evaluation metric is the F1-score. Since the classes are imbalanced and most recipes receive relatively high ratings, accuracy alone would be misleading. For example, a model that predicts every recipe as highly rated could achieve high accuracy while failing to identify meaningful differences between recipes. The F1-score balances precision and recall, making it a more appropriate metric for evaluating performance on this classification task.

## Baseline Model
The baseline model I chose is a Random Forest classifier that predicts whether a recipe will receive a rating above 4.5 stars. The model uses three quantitative features available at the time a recipe is posted: preparation time (`minutes`), number of ingredients (`n_ingredients`), and number of recipe steps (`n_steps`). 

To do so, I binarized 'avg_ratings' to create the column 'high rating' that contains 0 if the average rating of the recipe was below 4.5 and 1 if above.

The resulting f1 and classification scores were 0.85 and 0.74 respectively. Additionally, the classification report returned the following:

| Class | Precision | Recall | F1-Score | Support |
|---------|---------:|---------:|---------:|---------:|
| 0 | 0.97 | 0.75 | 0.85 | 12035 |
| 1 | 0.92 | 0.99 | 0.95 | 34254 |
| Accuracy |  |  | 0.93 | 46289 |
| Macro Avg | 0.95 | 0.87 | 0.90 | 46289 |
| Weighted Avg | 0.93 | 0.93 | 0.93 | 46289 |

This baseline model performed relatively well, achieving an accuracy of 93% and a weighted f1-score of 0.93. This means the model was able to accurately identify both highly rated and lower-reated recipes, suggesting that the initial features already contain substantial predictive information about recipe ratings.

Despite this strong performance, there is still room for improvement. To improve the model, I plan to incorporate additional features like nutritional information. I will also tune the model's hyperparameters to reduce overfitting and improve its ability to generalize to unseen data. These changes will hopefully help the model further improve its performance.


## Final Model

To improve the model's metrics, I used GridSearchCV to finetune the hyperparameters of the model. This output the parameters {'criterion': 'entropy', 'max_depth': None, 'min_samples_split': 2}

Using these, the model returned an f1 score of 0.94 and a classification score of 0.999. Additionally, 

Additionally, I thought I would explore how a random forest model
