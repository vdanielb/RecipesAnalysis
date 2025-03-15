# Is American Food Unhealthy?

**An investigation by**
- [Daniel Budidharma](https://vdanielb.github.io)
- [Tristan Leo](https://www.linkedin.com/in/tristan-leo-0b12a9340/)

# Table of Contents
- [Is American Food Unhealthy?](#is-american-food-unhealthy)
- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Data Cleaning and Exploratory Data Analysis](#data-cleaning-and-exploratory-data-analysis)
  - [Data Cleaning Steps](#data-cleaning-steps)
  - [Exploratory Data Analysis](#exploratory-data-analysis)
    - [Univariate Analysis](#univariate-analysis)
    - [Bivariate Analysis](#bivariate-analysis)
    - [Interesting Aggregates](#interesting-aggregates)
- [Assessment of Missingness](#assessment-of-missingness)
- [Hypothesis Test: Are American recipes more unhealthy?](#hypothesis-test-are-american-recipes-more-unhealthy)
- [Framing a Prediction Problem](#framing-a-prediction-problem)
- [Baseline Model](#baseline-model)
- [Final Model](#final-model)
  - [Good Recipes vs All Recipes](#good-recipes-vs-all-recipes)
  - [Optimizing GridSearchCV on RMSE vs R^2](#optimizing-gridsearchcv-on-rmse-vs-r2)
  - [Hyperparameter Tuning](#hyperparameter-tuning)
  - [Comparison of Models](#comparison-of-models)
  - [Our Final Model](#our-final-model)
- [Fairness Analysis](#fairness-analysis)

# Introduction

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

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning Steps
1. To begin our data cleaning process, we first removed the `Unnamed: 0` column from our `recipes` dataframe, which represents the index number on the original dataset before we took a subset of it.
1. We replaced ratings of 0 with NaN because the lowest possible rating is 1. So, a rating of 0 means that the reviewer did not leave a rating.
1. We also noticed that the values in the `tags`, `nutrition`, `steps`, and `ingredients`columns are represented as strings, so we had to convert them into lists in order to perform list operations.
1. We did a left merge on the `recipes` and `ratings` dataframes to obtain a new dataframe with each row corresponding to a review for a particular recipe.

**Note:** Most of our analyses won't involve the merged dataframe. The purpose of the merged dataframe is just to compute the average rating and number of reviews of each recipe.

1. We computed the average rating per recipe.
1. We added a new column `is_american`, which has a value of 1 if the recipe has an "american" tag or 0 otherwise.
1. We also added a new column `saturated_fat`, which has the saturated fat content of each recipe in percentage of daily value.

| name                                 |     id |   minutes |   contributor_id | submitted   | tags                                                                                                                                                                                                                                                                                               | nutrition                                                   |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | description                                                                                                                                                                                                                                                                                                                                                                       | ingredients                                                                                                                                                                                                                             |   n_ingredients |   average_rating |   is_american |   saturated_fat |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------|----------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------------:|--------------:|----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', ...]                                                                        | ['138.4', '10.0', '50.0', '3.0', '3.0', '19.0', '6.0']      |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass...']                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour']                                                          |               9 |                4 |             0 |              19 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                                                                                                      | ['595.1', '46.0', '211.0', '22.0', '13.0', '51.0', '26.0']  |        12 | ['pre-heat oven the 350 degrees f', 'in a mixing bowl ', 'sift together the flours and baking powder', ...']                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            | ['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips']                                                                             |              11 |                5 |             0 |              51 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                                                                                               | ['194.8', '20.0', '6.0', '32.0', '22.0', '36.0', '3.0']     |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray ', 'set aside', ...']                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']                                                                   |               9 |                5 |             0 |              36 |
| millionaire pound cake               | 286009 |       120 |           461724 | 2008-02-12  | ['time-to-make', 'course', 'cuisine', 'preparation', 'occasion', 'north-american', 'desserts', 'american', 'southern-united-states', 'dinner-party', ...] | ['878.3', '63.0', '326.0', '13.0', '20.0', '123.0', '39.0'] |         7 | ['freheat the oven to 300 degrees', 'grease a 10-inch tube pan with butter ', 'dust the bottom and sides with flour ', ...']                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                | ['butter', 'sugar', 'eggs', 'all-purpose flour', 'whole milk', 'pure vanilla extract', 'almond extract']                                                                                                                                |               7 |                5 |             1 |             123 |
| 2000 meatloaf                        | 475785 |        90 |          2202916 | 2012-03-06  | ['time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'potatoes', 'vegetables', '4-hours-or-less', 'meatloaf', 'simply-potatoes2']                                                                                                                                             | ['267.0', '30.0', '12.0', '12.0', '29.0', '48.0', '2.0']    |        17 | ['pan fry bacon ', 'and set aside on a paper towel to absorb excess grease', 'mince yellow onion ', 'red bell pepper ', 'and add to your mixing bowl', ...'] | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         | ['meatloaf mixture', 'unsmoked bacon', 'goat cheese', 'unsalted butter', 'eggs', 'baby spinach', 'yellow onion', 'red bell pepper', 'simply potatoes shredded hash browns', 'fresh garlic', 'kosher salt', 'white pepper', 'olive oil'] |              13 |                5 |             0 |              48 |

## Exploratory Data Analysis
To understand more about our data and explore possible features for our model, we will perform EDA. 

### Univariate Analysis
Firstly, we made a plot that shows the distribution of average ratings.

<iframe src="assets/average_rating_histogram.html" width=800 height=600 frameBorder=0></iframe>

The plot shows a large number of 5-star ratings by a huge margin. This could mean that people are either generous with their ratings or the fact that higher reviews would lead to more views, resulting in even more reviews.

However, this isn't ideal because the average rating doesn't say much about the actual quality of the recipe. Therefore, we chose not to make further analyses involving the average ratings.

Secondly, we were interested in the number of reviews of each recipe.

<iframe src="assets/num_reviews_histogram.html" width="800" height="600"></iframe>


The plot shows that the majority of recipes only have 1 review, so further analyses involving this would also lead to weak conclusions.

### Bivariate Analysis
We'll conduct a bivariate analysis next. We are interested in the saturated fat content in American vs Non-American recipes. We will plot a percentage histogram to compare the distributions.

**Note:** We zoomed in on the plot to see the distribution better without outliers. 

<iframe src="assets/american_vs_nonamerican.html" width="1000" height="600"></iframe>

While it might seem that their distributions are the same, notice that non-American recipes have a higher percentage of low saturated fat food. 

We'll do box plot next to examine this difference further.

**Note:** We zoomed in on the plot to see the distribution better without outliers. 

<iframe src="assets/american_vs_nonamerican_boxplot.html" width="800" height="600"></iframe>

Below is the plot made into a dataframe, if you want to see the numerical values directly:

|   is_american |   count |    mean |     std |   min |   25% |   50% |   75% |   max |
|--------------:|--------:|--------:|--------:|------:|------:|------:|------:|------:|
|             0 |   74525 | 39.6932 | 80.0602 |     0 |     6 |    20 |    48 |  6875 |
|             1 |    9257 | 44.6811 | 87.3526 |     0 |     8 |    25 |    55 |  4610 |

It seems from these plots that American recipes typically have higher saturated fat content. We will use a hypothesis test later to test if this is statistically significant.

### Interesting Aggregates
We categorized the saturated fat content of each recipe into several bins, grouped by those bins, and calculated the average rating across each bin. We do this to see if higher saturated fat recipes have higher ratings or not.

**Note:** We didn't include recipes with a saturated fat content above 200% of daily value so you can view the plot better.

<iframe src="assets/rating_vs_saturatedfat_plot.html" width="800" height="600"></iframe>

The plot suggests that the average rating for each bin is roughly equal. This makes sense given that a vast majority of the data has a high rating as we saw in the **Univariate Analysis** earlier. This further enforces the idea that the rating column isn't a good measure of the quality of the recipe.

# Assessment of Missingness
Moving on, we wanted to assess the missingness in our data.

Here is a breakdown of the missing data in `recipes`:

total missing values:  2680

|column                |    no. missing |
|:---------------|-----:|
| id             |    0 |
| name           |    1 |
| minutes        |    0 |
| contributor_id |    0 |
| submitted      |    0 |
| tags           |    0 |
| nutrition      |    0 |
| n_steps        |    0 |
| steps          |    0 |
| description    |   70 |
| ingredients    |    0 |
| n_ingredients  |    0 |
| average_rating | 2609 |
| num_reviews    |    0 |
| is_american    |    0 |
| saturated_fat  |    0 |

Here is a breakdown of the missing data in `interactions`:

total missing values:  52001

|column           |     no. missing |
|:----------|------:|
| user_id   |     0 |
| recipe_id |     0 |
| date      |     0 |
| rating    | 51832 |
| review    |   169 |

1. `name`:
    - We notice that there is only 1 missing value across tens of thousands of rows. Performing any missingness analyses on this column would not be insightful, but if we were to assess its missingness, it would most likely be MCAR because any permutation tests to verify this would consist of 1 row compared to tens of thousands of rows.

2. `review`:
    - We believe that the missingness of `review` is NMAR, because a review is usually only written if a user has strong feelings towards a recipe. So we suspect reviews that convey an impression of "average" are more likely to be missing.
  
3. `description`:
    - We believe that the missing values here are NMAR because some users may assume their recipe is already common knowledge or it’s self-explanatory. So we suspect that descriptions that would have been too short or convey nothing new would be missing.

4. `average_rating`:
    - The missingness of this column is trivial, since we made this column ourselves in data cleaning and wrangling. If all the `ratings` for a recipe is missing, then `average_rating` would be missing, meaning it's MAR depending on the `ratings`.

5. `ratings`:
    - We believe that missing values are MCAR, but there is a possibility that it's MAR because users might be more likely to finish and therefore leave ratings for recipes with fewer steps or shorter cooking times. To be sure, we perform a permutation test with the following hypotheses:

- **Null Hypothesis:** The missingness of `ratings` is not dependent on `n_steps`.
- **Alternative Hypothesis:** The missingness of `ratings` is dependent on `n_steps`.
- **Test statistic:** Absolute difference betweem mean n_steps of recipes with missing ratings and mean n_steps of recipes with non-missing ratings. Although we will also use ks2samp to verify our results.
- **Significance level:** 0.05

**Note:** We are performing the permutation test on the merged dataframe.

This is the result of the permutation test using the absolute difference of means:
<iframe src="assets/permtest.html" width="800" height="600"></iframe>

The observed difference is far from the results from our simulations under the null, as a result our **p-value is 0**. Using the Kolmogorov-Smirnov test statistic also yields a p-value of 0. Therefore, we **reject the null hypothesis**, which suggests that the missingness of `ratings` is dependent on `n_steps`. 

Let's look at the distribution of `n_steps` based on if `ratings` has a missing value or not.

<iframe src="assets/n_steps_rating_missing.html" width="800" height="600"></iframe>

The plot shows that at around n=7 steps, there are more non-missing than missing ratings. It might be because at around n=7 steps, people become more faithful to following the recipe step by step. Below 7, the dish is simple enough to infer the steps yourself. Higher than 7, the dish is too complex that people get lazy to follow all the steps. When you don't faithfully follow the recipe, you are less likely to leave a rating because the end product would be more your responsibility than the recipe's. But around n=7 seem to be the sweet spot where people would faithfully follow the recipe and therefore give a rating. The distribution plot seems consistent with the results of our permutation test: The missingness is MAR depending on n_steps.

Now, we want to find another column where the missingness of ratings doesn't depend on the column. We performed the same permutation test with the remaining numerical columns: `minutes` and `n_ingredients`.

We discovered that `minutes` is the only option where we fail to reject using the absolute difference of means.

- **Null Hypothesis:** The missingness of `ratings` is not dependent on `minutes`.
- **Alternative Hypothesis:** The missingness of `ratings` is dependent on `minutes`.
- **Test statistic:** The absolute difference between mean minutes of recipes with missing ratings and mean minutes of recipes with non-missing ratings.
- **Significance level:** 0.05

<iframe src="assets/permtest2.html" width="800" height="600"></iframe>

Our **p-value for this test was 0.129**, which is not statistically significant, meaning we **fail to reject the null**. This suggests that the missingness of `ratings` is not dependent on `minutes`. **However, when we verified using ks2_samp, the p-value was 0**. Let's look at the distribution of `minutes` based on if `ratings` has a missing value or not.

**Note:** We zoomed in on the plot to see the distribution better without outliers. 

<iframe src="assets/minutes_missingness.html" width="800" height="600"></iframe>

It makes sense from this that the ks_2samp would tell us to reject the null. It seems that higher cooking times have a higher proportion of non-missing values. This makes sense because if you're willing to commit that much time to cook a recipe, we would imagine you'd have stronger feelings about the outcome. For example, if I spent 3 hours cooking something following a recipe online and it turned out bad, I would be disappointed enough to leave a bad rating instead of just commenting without rating. 

This is also true for recipes past 2000 minutes because there are only few recipes with a cooking time higher than that. We tried the ks_2samp on every other possible numerical column, including all the nutritional values, but none of them failed to reject the null, even after taking into account outliers. This is completely unexpected, but possible. **But at the very least the hypothesis test using absolute difference of means suggests that the missingness of `ratings` is not dependent on `minutes`.**

# Hypothesis Test: Are American recipes more unhealthy?
Now to test our question: Are American foods significantly more unhealthy than non-American foods? 

Now, a healthy diet is usually a balanced diet, so we can't conclude one nutrient is objectively better to always have more of. But we can at the very least say saturated fat is objectively **bad** for you. Many national and international health organizations, such as [The American Heart Association](https://www.heart.org/en/healthy-living/healthy-eating/eat-smart/fats/saturated-fats) and [World Health Organization](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates) recommend either limiting or replacing saturated fat intake.

We then want to perform a hypothesis test to determine if American recipes on food.com have more saturated fat than non-American foods, and if this difference is statistically significant.

We observed that American recipes have higher median saturated fat than non-American recipes on the EDA. So, that will be our alternative hypothesis. Our hypotheses are:
- **Null Hypothesis:** American and Non-American recipes on food.com have the same amount of saturated fat.
- **Alternative Hypothesis:** American recipes have more saturated fat than Non-American recipes.
- **Test statistic:** `Median saturated fat in American recipes` - `Median saturated fat in Non-American recipes`
- Significance level: 0.05

**Note that we chose to use the median as our test statistic**. This is because our EDA revealed many outliers and also revealed that the saturated fat distribution is skewed. In cases like these, the median performs better.



Our **p-value here is 0**. This means we can confidently **reject the null hypothesis**. This suggests that American recipes have more saturated fat than non-American recipes, and therefore more unhealthy.

Out of curiosity, we also did the same permutation test but with difference of means. Our result is the same. The p-value is 0 and we reject the null.

<iframe src="assets/observed_vs_distofstats_diffofmeans.html" width="1000" height="600"></iframe>

**So are American recipes unhealthy?** All our results, from exploratory data analysis to this hypothesis test suggest **yes**. Of course, this might not be the only reason why America has such a high obesity rate, but it is a factor. Other factors might include car dependency, access to food, health awareness, lifestyle, and many more.

# Framing a Prediction Problem
One challenge we face as college students is trying to manage time. So we decided to build a model that could predict the total cooking time (in minutes) of whatever one might want to cook. This will be a regression problem. 

We will prioritize RMSE as our performance metric. We feel this is better than R^2 for this problem because when I want an estimate of how long a recipe will take to make, I'd be more worried about how "off" that estimate might be compared to how "good" the fit of my model is. RMSE is also more interpretable: if my RMSE is 10 minutes, then that means my estimate will probably be off by 10 minutes on average. So we will prioritize RMSE, but we will also still keep track of R^2 to see the fit of our model.

One easy way to build a really accurate model for this is to look at the tags with that say '60-minutes-or-less' or '30-minutes-or-less'. However, this would be uninteresting and also kind of defeat the purpose. In the "real world", when you're trying to cook a new recipe, you won't know those tags. So we'll ignore that. We will also ignore nutrition values other than calories. There's no way we could know exactly how much carbs or protein our recipe will have, but people are generally more familiar with estimating calories, so we'll use that.

Our data also had a lot of outliers. We decided to filter out rows where `minutes` exceeded 3 hours, and rows where `calories` exceeded 2000, since 2000 calories is the recommended daily calorie intake of an adult male. Another reason is that we find it unrealistic that a recipe would have 2000 calories **per serving**. These are probably recipes uploaded in bad faith or unseriously by users.

# Baseline Model
For our baseline model, these will be our features along with their types:
1.  number of ingredients (discrete) 
2.  number of steps (discrete)
3.  calories per serving (continuous)

`n_ingredients` and `n_steps` were already present in our dataframe. For calories per serving, we had to use ColumnTransformer to extract the value from `nutrition`. We put all the feature engineering and model training into one sklearn pipeline.

We tried out 2 baseline models, a linear regression and a decision tree regressor. We suspect a linear regression model won't work well. In our notebook, we made some scatterplots to explain why this is the case. The data points we plotted showed no patterns that could be modeled by a line. However, we'll try both models anyway. We do a train test split to avoid overfitting. All the metrics that are reported here are calculated using the test data.

1. **Linear Regression**
    - Test R^2:  0.21
    - Test RMSE:  27.37

As you can see it performs pretty badly. So, we choose to use a decision tree instead. We will set max_depth = 10 to avoid overfitting and set random_state=12 for reproducability. 

2. **Decison Tree Regressor**
    Using K-Fold Cross Validation:
   - Mean RMSE: 28.32
   - Mean R2 Score: 0.17

On average, our current models are predicting around 27 minutes off the true value, which is quite inaccurate. For instance, if our model predicted that a recipe would take 30 minutes to make, but actually takes an hour, I would feel cheated.

We suspect Random Forest Regressor will get better after we tune its hyperparameters. To improve our final model, we'll use a random forest to avoid overfitting and GridSearchCV to tune our hyperparameters. We'll also include more features and see if LinearRegression or RandomForestRegressor is better.

# Final Model
Our final model uses the following features:
1.  number of ingredients (discrete) 
2.  number of steps (discrete)
3.  calories per serving (continuous): Extracted from `nutrition`
4.  ingredients (nominal):  There are too many unique ingredients in this whole dataset to feasibly one hot encode, so we'll focus on a few common ingredients. That is, we're going to feature engineer if a recipe contains these ingredients: `['beef', 'pork', 'chicken', 'corn', 'potatoes', 'rice', 'bread', 'pasta', 'milk', 'cheese', 'butter', 'sugar', 'flour', 'tomatoes', 'squash']`. Since there could be many types of the same ingredient (e.g. sweet corn vs normal corn, unsalted butter vs salted butter), we will make it so that any instance of that word appearing in the ingredients column means the ingredient is present. For example, if a recipe has 'sweet corn' as an ingredient, we consider that as containing corn. From this one-hot-encoding, we will have multiple new columns (e.g. `has_beef` is 1 if the recipe contains beef or 0 otherwise, `has_pork` is 1 if the recipe contains pork or 0 otherwise, etc.)
5. recipe type (nominal): We one hot encode if a recipe is of category 'breakfast', 'lunch', 'dinner-party', 'desserts', or 'snacks'.

We end up with a total of 23 features.

## Good Recipes vs All Recipes
We suspected that the fit of our model is bad because of bad data quality. Some people on food.com could just upload random recipes, with random n_steps, random minutes, random ingredients, etc. without proper correctness checks. So we'll train some models on only "good quality" data points and see the results for that as well. We decide a data point is of "good quality" if it has an average rating >=4. From the EDA, we saw there are a lot of recipes with an average rating above 4, so this wouldn't hurt our sample size too much. The pipeline will still be the same.

## Optimizing GridSearchCV on RMSE vs R^2
We will also try out GridSearchCV optimized on minimizing RMSE vs maximizing R^2 to see if there are any differences.

## Hyperparameter Tuning
When using GridSearchCV for our Random Forest model, we will tune:
  - **the number of trees in our forest**. This is to find a sweet spot between bias and variance, and ultimately avoid underfitting or overfitting. How we get the optimal number of tress is by trial and error.
  - **the max depth of each tree**. Same reason as above. How we get the optimal max depth is by trial and error.
  - **criterions**. This parameter tells RandomForestRegressor how it decides the best split. The reason we don't go straight for mean squared error as the criterion is that a DecisionTreeRegressor only uses a criterion to optimize for the best local split, not necessarily minimize RMSE for the whole model. So it's possible that the poisson criterion, for example, minimizes overall RMSE. It is for that reason we try out every possible criterion in the sklearn DecisionTreeRegressor object except for MAE. This is because MAE is very slow and makes GridSearchCV run for hours. Sadly, that is a limitation.
  - **max_features**. This decides how many features each tree in the forest should be trained on. We try out None(all features), sqrt, and log2. This is basically trying out every possible option allowed by RandomForestRegressor, other than setting our own specific number.


## Comparison of Models
We end up with **7 models** for us to compare the performances of and choose the best one. The table below shows each model along with their RMSE and R^2 test scores, sorted by smallest RMSE first. If RMSEs are tied, the higher R^2 is shown first.

|                                                                 |   RMSE |     R^2 |
|:----------------------------------------------------------------|-------:|--------:|
| Random Forest Regressor on full recipes, optimized on R^2       |  25.94 |    0.29 |
| Random Forest Regressor on full recipes, optimized on RMSE      |  25.94 | -672.79 |
| Random Forest Regressor on only good recipes, optimized on R^2  |  25.97 |    0.29 |
| Random Forest Regressor on only good recipes, optimized on RMSE |  25.97 | -674.24 |
| Linear Regression on full recipes                               |  26.62 |    0.25 |
| Baseline Linear Regression                                      |  27.37 |    0.21 |
| Baseline Decision Tree Regressor                                |  28.32 |    0.17 |

Curiously, RMSE is the same regardless of if we optimize GridSearchCV for R^2 or RMSE. R^2 however becomes negative when optimizing on RMSE. There are a few possible explanations for why this is. The main idea is that when we optimize for RMSE (minimizing it), the model is focusing solely on reducing the absolute prediction error. However, this doesn't guarantee that the model captures any underlying patterns in our data, which is what R^2 measures. So it is quite possible that for the same RMSE, we get different R^2.

## Our Final Model
Our final model will be a **Random Forest Regressor fit and tested on full recipes, tuned with GridSearchCV to maximize R^2 and therefore minimize RMSE**. 

Compared to the baseline:

    - R^2 improved from 0.17 to 0.29
    - RMSE improved from 28.32 minutes to 25.94 minutes

The best hyperparameters ended up being:
- 'regressor__criterion': 'squared_error'
- 'regressor__max_depth': np.int64(16)
- 'regressor__max_features': 'sqrt'
- 'regressor__n_estimators': np.int64(120)

This RandomForestRegressor performs better than our baseline due to a few key differences:
- More features
  - This model is able to make predictions based on what ingredients are being used. We would expect this to have some correlation with cooking time. For example, a recipe with no beef or pork, but with butter, sugar, and flour would suggest that it is a baking recipe, which typically takes longer to make. On the flip side, if a recipe only contains beef, it probably means you're just cooking the beef on a frying pan, and it would probably be faster. 
  - It is also able to make predictions based on the food type. For example, we would expect breakfast recipes to be faster to make because people don't generally eat heavy and complex meals for breakfast.
  - More features also explains why our linear regression model performed better compared to the baseline.
- Random Forest vs Decision Tree
  - Our final model uses a RandomForestRegressor instead of a DecisionTreeRegressor. It only makes sense that a more complex model built from multiple trees would be able to make better predictions than one decision tree.
- Fine tuning with GridSearchCV
  - Our final model has also been tuned with GridSearchCV to find the best hyperparameters


Looking at feature importances, it seems the top 3 most important features for our final model are n_steps, n_ingredients, and calories. This is to be expected.

Still, we believe our final model isn't very accurate. It's still off by 26 minutes on average, after all. We believe the model is still inaccurate because the published recipes on food.com don't have proper checks or consistent guidelines, which makes it possible for recipes to contain inaccurate information. However, we've included every possible feature that is both in our dataset, and a person could "know" before starting to cook. Any more than this wouldn't be feasible with our limited compute and dataset. 

# Fairness Analysis
Since food.com is an American website, we decide to see if our model works better on American recipes vs non-American recipes.

We observe that non-American RMSE is higher than American RMSE, so our hypothesis test will be:
- **Null hypothesis:** Our model's RMSE is the same for American recipes and non-American recipes
- **Alternative hypothesis:** Our model's RMSE is higher for non-American recipes than for American recipes
- **Test statistic:** RMSE in non-American recipes - RMSE in American recipes
- **Significance level:** 0.05

<iframe src="assets/fairness_permtest.html" width="800" height="600"></iframe>

**Our P-value is 0.355, which is higher than 0.05, so we fail to reject the null hypothesis**. This suggests that our model's RMSE is the same for American recipes and non-American recipes, and is therefore fair between American recipes and non-American recipes.
