# Is American Food Unhealthy?

**An investigation by**
- [Daniel Budidharma](https://vdanielb.github.io)
- [Tristan Leo](https://www.linkedin.com/in/tristan-leo-0b12a9340/)

## Table of Contents
- [Is American Food Unhealthy?](#is-american-food-unhealthy)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Data Cleaning and Exploratory Data Analysis](#data-cleaning-and-exploratory-data-analysis)

## Introduction
Provide an introduction to your dataset, and clearly state the one question your project is centered around. Why should readers of your website care about the dataset and your question specifically? Report the number of rows in the dataset, the names of the columns that are relevant to your question, and descriptions of those relevant columns.

**The United States has one of the highest obesity rates in the world**. According to the CDC, [more than 2 in 5 U.S. adults are considered obese](https://www.cdc.gov/obesity/adult-obesity-facts/index.html). Why is this? Is it just the food that Americans eat? 

Today, we will explore data on nearly 100,000 recipes scraped from [food.com](https://www.food.com) to explore this question. The dataset we will use was scraped by the authors of [this](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) paper, and is linked [here](https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions). For practical reasons, we will only be using a subset of this data. In addition, we also have data on over 70,000 reviews and ratings on food.com.  

Our first dataset `recipes` contains 83,782 rows, each corresponding to a unique recipe uploaded by a user. It has 12 columns:

| Column name | Description                                                                                          |
| :---------- | :---------------------------------------------------------------------------------------------------- |
| `'name'`           | Name of the recipe                                                             |
| `'id'`             | id of the recipe                                                 |
| `'minutes'`        | How long it takes to make the recipe in minutes |  
| `'contributor_id'` | User ID belonging to the uploader of this recipe              |
| `'submitted'`      | Date this recipe was uploaded                  |
| `'tags'`           | Tags associated with recipe (e.g. American, 30-minutes-or-less, vegan, etc.)                |
| `'nutrition'`      | List of nutritional values in the order of calories, total fat, sugar, sodium, protein, saturated fat,carbohydrates. Note: other than calories, nutrition is measured as a percentage of daily value |
| `'n_steps'`        | Number of steps to make the recipe                |
| `'steps'`          | Steps to make the recipes                       |
| `'description'`    | Description of the recipe                             |
| `'ingredients'`    | List of ingredients for the recipe                              |
| `'n_ingredients'`  | Number of ingredients in the recipe                              |

The second dataset is `interactions`. It contains 731,927 rows, each corresponding to a single review on food.com towards a recipe. It has 5 columns:

| Column name   | Description         |
| :------------ | :------------------ |
| `'user_id'`   | id of user who uploaded review             |
| `'recipe_id'` | id of recipe being reviewed          |
| `'date'`      | Date review got posted |
| `'rating'`    | Rating given        |
| `'review'`    | The review         |

## Data Cleaning and Exploratory Data Analysis
1. To begin our data cleaning process, we first removed the `Unnamed: 0` column from our `recipes` dataframe, which represents the index number on the original dataset before we took a subset of it.
1. We replaced ratings of 0 with NaN because the lowest possible rating is 1. So, a rating of 0 means that the reviewer did not leave a rating.
1. We also noticed that the values in the `tags`, `nutrition`, `steps`, and `ingredients`columns are represented as strings, so we had to convert them into lists in order to perform list operations.
1. We did a left merge on the `recipes` and `ratings` dataframes to obtain a new dataframe with each row corresponding to a review for a particular recipe.

**Note:** Most of our analyses won't involve the merged dataframe. The purpose of the merged dataframe is just to compute the average rating and number of reviews of each recipe.

1. We computed the average rating per recipe.
1. We added a new column `is_american`, which has a value of 1 if the recipe has an "american" tag or 0 otherwise. <TODO>

To understand more about our data and explore possible features for our model, we will perform EDA. 

Firstly, we made a plot that shows the distribution of average ratings.

<iframe src="assets/average_rating_histogram.html" width=800 height=600 frameBorder=0></iframe>

The plot shows a arge number of 5-star ratings by a huge margin. This could mean that people are either generous with their ratings or the fact that higher reviews would lead to more views, resulting in even more reviews.

However, this isn't ideal because the average rating doesn't say much about the actual quality of the recipe. Therefore, we chose not to make further analyses involving the average ratings.

Secondly, we were interested in the number of reviews of each recipe.

<PLOT>

Now the plot shows that the majority of recipes only have 1 review, so further analyses involving this would also lead to weak conclusions.

<BIVARIATE_ANALYSIS>
<INTERESTING_AGGREGATES>

Moving on, we wanted to assess the missingness in our data.

<RECIPES_MISSING_VALUES>
1. `name` column:
    - We notice that there is only 1 missing value across tens of thousands of rows. Performing any missingness analyses on this column would not be insightful, but if we were to assess its missingness, it would most likely be MCAR because any permutation tests to verify this would consist of 1 row compared to tens of thousands of rows.

2. `description` column:
    - We believe that the missing values are NMAR because some users may assume their recipe is already common knowledge—perhaps it’s a well-known dish that everyone recognizes. Because of this assumption, they might not see the value in providing a detailed explanation, concluding it’s self-explanatory. Consequently, they skip writing a description entirely. In other words, if a dish’s name alone conveys enough context, users might feel no motivation to elaborate, viewing it as an unnecessary effort. This mindset could lead to laziness when it comes to generating a detailed write-up, resulting in missing descriptions.

<INTERACTIONS_MISSING_VALUES>

Looking at the `interactions` dataframe, we notice over 51,000 missing values in the `ratings` column. We think that it could be MCAR because some users may interact with a recipe by leaving review but not providing a rating simply because they forgot or chose not to rate, but we can verify this by performing a permutation test.

<PERMUTATION_TEST>

As mentioned in the introduction, we're curious to see if American food is unhealthy. To make room for comparison, we'll focus on the saturated fat content in American and Asian recipes, since we can at least say that saturated fat is objectively bad for your health. 



To investigate further, we performed some data wrangling on the `recipes` dataframe to prepare varaiables that will be used for our **hypothesis test**:
    1. Extract the saturated fat content from the `nutrition` column and store it in a new column called `saturated_fat`.

    1. Label each recipe as American or Asian based on whether the recipe contains either tag in the `tags` column.
    >>>

ran a **hypothesis test** with the following hypotheses, test statistic, and significance level.

We observe that American recipes have higher saturated fat on average. So that will be our alternative hypothesis. Our hypotheses are:
- **Null Hypothesis:** American and Asian recipes on food.com have the same amount of saturated fat.
- **Alternative Hypothesis:** American recipes have more saturated fat than Asian recipes.
- **Test statistic:** `Mean saturated fat in American recipes` - `Mean saturated fat in Asian recipes`


