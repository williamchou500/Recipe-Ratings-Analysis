# Analysis of the Relationship Between Nutritional Value and Recipe Ratings

Author: William Chou

## Introduction

I have always understand that eating healthy diet is an essential part of leading a healthy life but unhealthy foods that high in carbohydrates, sodium, fats, etc. tend to be more delicious and bring a lot of immediate happiness in the short term. This led me to question how this dilemma would be reflected in food reviews for recipes online. Would the health benefits of a food boost thereception of a recipe or is taste and satisfaction more important to reviewers?

The question I am interested in answering is: **'What is the relationship between the nutritional value of a recipe and its average rating?'**.

This question is most significant for creators who are publishing their recipes on food.com. Knowing the answer to the above question may help them understand what qualities they should attempt to include in their recipes in order for their submissions to be more well received, show up more frequently in search results, and improve their popularity and online presence. For people just looking for recipes online, this answer to this question can still be beneficial. The information can help a consumer be more conscious of what may or may not skew the rating of a dish that is not simply the quality of the recipe. 

Important to answering this question is data about recipes, their nutritional values, and consumer reviews. Two datasets have been compiled containing these data and more to answer the question.


The first dataset that will be used for this analysis is the *recipes* dataset. This dataset contains 83782 rows and 12 columns. The visual below includes the columns found in the dataset, what each column contains, and the data types each column holds.

| Column         | Description                                                                                                                                                                                      | Data Types   |
|:---------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------|
| name           | Recipe name                                                                                                                                                                                      | object       |
| id             | Recipe ID                                                                                                                                                                                        | int64        |
| minutes        | Minutes to prepare recipe                                                                                                                                                                        | int64        |
| contributor_id | User ID who submitted this recipe                                                                                                                                                                | int64        |
| submitted      | Date recipe was submitted                                                                                                                                                                        | object       |
| tags           | food.com tags for recipe                                                                                                                                                                         | object       |
| nutrition      | Nutrition information in the form [calories(#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" | object       |
| n_steps        | Number of steps in recipe                                                                                                                                                                        | int64        |
| steps          | Text for recipe steps, in order                                                                                                                                                                  | object       |
| description    | User-provided description                                                                                                                                                                        | object       |
| ingredients    | Ingredients used in recipe                                                                                                                                                                       | object       |
| n_ingredients  | Number of ingredients in recipe                                                                                                                                                                  | int64        |

The second dataset that will be used for this analysis is the *interactions* dataset. This dataset contains 731927 rows and 5 columns. The visual below includes the columns found in the dataset, what each column contains, and the data types each column holds.


| Column    | Description         | Data Types   |
|:----------|:--------------------|:-------------|
| user_id   | User ID             | int64        |
| recipe_id | Recipe ID           | int64        |
| date      | Date of interaction | object       |
| rating    | Rating given        | int64        |
| review    | Review text         | object       |

The data types match what is expected based on their description with the exception of the 'nutrition' column, which is the string representation of a list, and the 'submitted' column, which is a string instead of a datetime object. Though the majority of the data is in the right format, these datasets in their current form are still insufficient for the purpose of this analysis and will need to be merged, cleaned, and modified greatly.

Since the goal of the analysis is to check for a relationship between nutritional value and average ratings, the most important columns for this investigation are the nutritional values, each split into their own columns and the average rating of each recipe. There should also be a column for identifying each recipe as 'healthy' or 'unhealthy' to group the data upon.

## Data Cleaning and Exploratory Data Analysis

#### Step 1: Merging our datasets

So that data across both datasets can be aggregated and grouped upon, the datasets need to be merged together. Both datasets share a column representing the recipe id, so those columns will used to merge our datasets together. They will be merged so that all the recipes in the *recipes* dataset are kept.

#### Step 2: Cleaning the 'rating' column

The 'rating' column from the *interactions* dataset contains the rating 0 as one of its values. This represents people who left a review but not a rating. Leaving these values as 0 will skew the average rating down and a rating of 0 will not always properly represent the views of the reviewer. To account for this, we replace every instance of 0 with np.nan. Now, the 'rating' column will only have values from 1-5.

#### Step 3: Converting 'submitted' column into datetime objects

As strings, the dates are not ordinal data that can be compared. In order to allow for comparisons, the strings will be converted into datetime objects. While this analysis will not track trends over time, cleaning the data in this way allows for the dataset to be used for such analyses.

#### Step 4: Adding a column for average rating

The average rating for each recipe was obtained by grouping on the recipe id and aggregating using the mean function. These average values were appended to the dataset of recipes and reviews with another merge. This column is added to the dataset as 'avg_rating'

#### Step 5: Extracting nutritional values

The column 'nutrition' contains string representations of lists which contain information about calorie counts and PDV values. Each element of these lists were extracted and separated to be values in their own columns. The dataset now includes the columns 'calories', 'total fat', 'sugar', 'sodium', 'protein', 'saturated fat', 'carbohydrates'.

#### Step 6: Converting from PDV to actual quantity

PDV values for different nutrients are different from one another. A PDV of 50% for sodium is different from a PDV of 50% for carbohydrates, for example. For every column of PDV data in the dataset, there will be a new column containing the actual quantity of that particular nutrient gained by multiplying the PDV by the recommended daily total. The columns will ba named with the format '{name of the nutrient}\_{g or mg}'.

#### Step 7: Adding a column for whether a recipe is healthy or not

For the purpose of grouping and for the base model, a column of bools labeling a recipe as healthy or unhealthy will be added. For the sake of this analysis, a recipe is labeled as healthy if its ratio of carbohydrates to protein is less than 2.

#### Step 8: Removing recipes with less than 15 reviews

Since average rating is what we are training our model to find and what the analysis hopes to draw insights on, the average rating should be a good representation of the recipe. A recipe with only 1 or 2 reviews, for example, may be made to seem a lot better or worse than it actually is. For my analysis, I chose to include only recipes with more than 15 reviews and removed all duplicate recipes. A separate variable was assigned for this restricted dataset.

### Cleaned Dataset

The dataset contains 234429 rows and 33 columns without being filtered to only recipes with 15 or more reviews and 1379 rows and 33 columns when filtered. For this analysis, the filtered dataset will be always be used unless mentioned otherwise.

The columns and their data types for the new data set are shown below.

| Column             | Data Type   |
|:-------------------|:------------|
| name               | object      |
| id                 | int64       |
| minutes            | int64       |
| contributor_id     | int64       |
| submitted          | object      |
| tags               | object      |
| nutrition          | object      |
| n_steps            | int64       |
| steps              | object      |
| description        | object      |
| ingredients        | object      |
| n_ingredients      | int64       |
| user_id            | float64     |
| recipe_id          | float64     |
| date               | object      |
| rating             | float64     |
| review             | object      |
| avg_rating         | float64     |
| calories           | float64     |
| total fat          | float64     |
| sugar              | float64     |
| sodium             | float64     |
| protein            | float64     |
| saturated fat      | float64     |
| carbohydrates      | float64     |
| total fat_g        | float64     |
| sugar_g            | float64     |
| sodium_mg          | float64     |
| protein_g          | float64     |
| saturated fat_g    | float64     |
| carbohydrates_g    | float64     |
| carb_protein_ratio | float64     |
| healthy            | bool        |

<iframe
	src="assets/fig_ratios_20.html"
	width="800"
	height="600"
	frameborder="0"
></iframe>

| ratings_bracket   |   n_ingredients |   n_steps |
|:------------------|----------------:|----------:|
| 2-3               |         8.33333 |  12.1667  |
| 3-4               |         9       |   9.74359 |
| 4-5               |         8.93478 |  10.1582  |