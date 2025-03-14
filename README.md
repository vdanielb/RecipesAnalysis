# Is American food just unhealthy?

**An investigation by**
- [Daniel Budidharma](https://vdanielb.github.io)
- [Tristan Leo](https://www.linkedin.com/in/tristan-leo-0b12a9340/)

## Table of Contents
- [Is American food just unhealthy?](#is-american-food-just-unhealthy)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Data Cleaning and Exploratory Data Analysis](#data-cleaning-and-exploratory-data-analysis)

## Introduction

The United States has one of the highest obesity rates in the world. According to the CDC, [more than 2 in 5 U.S. adults are considered obese](https://www.cdc.gov/obesity/adult-obesity-facts/index.html). Why is this? Is it just the food that Americans eat? 

Today, we will explore data on nearly 100,000 recipes scraped from [food.com](https://www.food.com) to explore this question. The dataset we will use was scraped by the authors of [this](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) paper, and is linked [here](https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions). For practical reasons, we will only be using a subset of this data. In addition, we also have data on over 70,000 reviews and ratings on food.com.  

Our first dataset `recipes` contains 83,782 rows, each corresponding to a unique recipe uploaded by a user. It has 12 columns:

| Column name | Description                                                                                          |
| :------ :---------------------------------------------------------------------------------------------------- |
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

The second dataset is `interactions`. It contains 731,927 rows, each row corresponding to a single review on food.com towards a recipe. It has 5 columns:

| Column name   | Description         |
| :------------ | :------------------ |
| `'user_id'`   | id of user who uploaded review             |
| `'recipe_id'` | id of recipe being reviewed          |
| `'date'`      | Date review got posted |
| `'rating'`    | Rating given        |
| `'review'`    | The review         |

## Data Cleaning and Exploratory Data Analysis
