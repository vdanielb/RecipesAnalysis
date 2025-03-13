# Recipes Analysis
**Authors**
- [Daniel Budidharma](https://vdanielb.github.io)
- [Tristan Leo](https://www.linkedin.com/in/tristan-leo-0b12a9340/)

## Introduction
Provide an introduction to your dataset, and clearly state the one question your project is centered around. Why should readers of your website care about the dataset and your question specifically? Report the number of rows in the dataset, the names of the columns that are relevant to your question, and descriptions of those relevant columns.

**Is American food just unhealthy?** The United States has one of the highest obesity rates in the world. According to the CDC, [more than 2 in 5 U.S. adults are considered obese](https://www.cdc.gov/obesity/adult-obesity-facts/index.html). Why is this? Is it just the food that Americans eat? 

Today, we will explore data on nearly 100,000 recipes scraped from [food.com](https://www.food.com) to explore this question. The dataset we will use was scraped by the authors of [this](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) paper, and is linked [here](https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions). For practical reasons, we will only be using a subset of this data. In addition, we also have data on over 70,000 reviews and ratings on food.com.  

Our first dataset `recipes` contains 83,782 rows, each corresponding to a unique recipe uploaded by a user. It has 12 columns:

| Column name | Description                                                                                          |
| :------ :---------------------------------------------------------------------------------------------------- |
| `'name'`           | Name of the recipe                                                             |
| `'id'`             | id of the recipe                                                 |
| `'minutes'`        | How long it takes to make the recipe in minutes |  
| `'contributor_id'` | The user ID belonging to the uploader of this recipe              |
| `'submitted'`      | Date this recipe was uploaded                  |
| `'tags'`           | Tags associated with recipe (e.g. American, 30-minutes-or-less, vegan, etc.)                |
| `'nutrition'`      | List of nutritional values in the order of calories, total fat, sugar, sodium, protein, saturated fat,carbohydrates. Other than calories, nutrition is measured as a percentage of daily value |
| `'n_steps'`        | Number of steps to make the recipe                |
| `'steps'`          | Steps to make the recipes                       |
| `'description'`    | Description of the recipe                             |
| `'ingredients'`    | List of ingredients for the recipe                              |
| `'n_ingredients'`  | Number of ingredients in the recipe                              |

The second dataset is `interactions`. It contains 73,1927, each row corresponding to a single review on food.com towards a recipe. It has 5 columns:

| Column name       | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

## Data Cleaning and Exploratory Data Analysis
