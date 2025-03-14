# Analysis of the Relationship Between Nutritional Value and Recipe Ratings

Author: William Chou

## Introduction

I have always understood that eating healthy diet is an essential part of leading a healthy life, but unhealthy foods that are high in carbohydrates, sodium, fats, etc. tend to be more delicious and bring a lot of immediate happiness in the short term. This led me to question how this dilemma would be reflected in food reviews for recipes online. Would the health benefits of a food boost the reception of a recipe or is taste and satisfaction more important to reviewers?

The question I am interested in answering is: **'What is the relationship between the nutritional value of a recipe and its average rating?'**.

This question can be significant for creators who are publishing their recipes on food.com. Knowing the answer to the above question may help them understand what qualities they should attempt to include in their recipes in order for their submissions to be more well received, show up more frequently in search results, and improve their popularity and online presence. For people just looking for recipes online, this answer to this question can still be beneficial. The information can help a consumer be more conscious of what may or may not skew the rating of a dish that is not simply the quality of the recipe. 

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

For the purpose of grouping and for the base model, a column of booleans labeling a recipe as healthy or unhealthy will be added. For the sake of this analysis, a recipe is labeled as healthy if its ratio of carbohydrates to protein is less than 2. The carbohydrate-to-protein ratio will be saved as a column named 'carb_protein_ratio' and the column of booleans will be labled as 'healthy'.

#### Step 8: Removing recipes with less than 10 reviews

Since average rating is what we are training our model to find and what the analysis hopes to draw insights on, the average rating should be a good representation of the recipe. A recipe with only 1 or 2 reviews, for example, may be made to seem a lot better or worse than it actually is. For my analysis, I chose to include only recipes with 10 or more reviews and removed all duplicate recipes. A separate variable was assigned for this restricted dataset.

### Cleaned Dataset

The dataset contains 234429 rows and 33 columns without being filtered to only recipes with 15 or more reviews and 3369 rows and 33 columns when filtered. For this analysis, the filtered dataset will be always be used unless mentioned otherwise.

The columns and their data types for the new data set are shown below.

| Column             | Data Type      |
|:-------------------|:---------------|
| name               | object         |
| id                 | int64          |
| minutes            | int64          |
| contributor_id     | int64          |
| submitted          | datetime64[ns] |
| tags               | object         |
| nutrition          | object         |
| n_steps            | int64          |
| steps              | object         |
| description        | object         |
| ingredients        | object         |
| n_ingredients      | int64          |
| user_id            | float64        |
| recipe_id          | float64        |
| date               | object         |
| rating             | float64        |
| review             | object         |
| avg_rating         | float64        |
| calories           | float64        |
| total fat          | float64        |
| sugar              | float64        |
| sodium             | float64        |
| protein            | float64        |
| saturated fat      | float64        |
| carbohydrates      | float64        |
| total fat_g        | float64        |
| sugar_g            | float64        |
| sodium_mg          | float64        |
| protein_g          | float64        |
| saturated fat_g    | float64        |
| carbohydrates_g    | float64        |
| carb_protein_ratio | float64        |
| healthy            | bool           |

### Univariate Analysis

For my Univariate Analysis, I will look at the distribution of average rating as well as the the distribution of carbohydrate-to-protein ratios as both columns are essential for the permutation testing I will be performing as well as the for the models for my prediction problem.

#### avg_rating 

<iframe
	src="assets/fig_ratings.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

Looking at the distribution of 'avg_rating', we can see that the dataset is saturated with very high average ratings for recipes. We can also see that none of the recipes have an average rating of below a 2. These facts mean that our model for our prediction problem, which seeks to predict the average rating of a recipe, will be more likely to overrate recipes rather than underrate.

#### carb_protein_ratio

<iframe
	src="assets/fig_ratios.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

The vertical red line is placed at where the carbohydrate-to-protein ratio is equal 2, the value used to separate healthy and unhealthy recipes. The data looks like it contains some outliers. The largest value in the column is 208, but the majority of the data seems to lie under a ratio of 20. The next graph includes only rows with a ratio under 20 for a better view of the data.

<iframe
	src="assets/fig_ratios_20.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

Most of the distribution lies at the lower end of possible values. The split between healthy and unhealthy looks to be evenly distributed so grouping on the 'healthy' column should be quite effective.

### Bivariate Analysis

<iframe
	src="assets/healthy_ratings.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

It seems that the distribution of ratings for healthy and unhealthy foods is very similar. However, a permutation test to see if the two distributions come from different populations would still be worthwhile. Like the distribution of ratings without grouping on the 'healthy' column, there is still a heavy concentration of high ratings.

### Interesting Aggregates

The following pivot table was constructed by first placing all the recipes into brackets based on their average rating. The brackets go from non-inclusive to inclusive. 2-3 represents average ratings within (2, 3], 3-4 represents ratings within (3, 4], etc. It looks like recipes with a lower average rating have more steps.

| ratings_bracket   |   n_ingredients |   n_steps |
|:------------------|----------------:|----------:|
| 2-3               |         8.90909 |  12.1818  |
| 3-4               |         9.14    |  11.41    |
| 4-5               |         8.79282 |   9.70227 |

## Assessment of Missingness

Looking at the dataset that was not filtered to include only recipes with 15 or more recipes, there are three columns that stand out for having a large quantity of missing values. These columns are 'description', 'rating', and 'review'.

### NMAR Analysis

The columns 'description' and 'review' are likely NMAR. 

The publisher of a recipe might not put a description for a recipe if the dish is self-explanatory and commonly known. The publisher may also just be apathetic about the recipe they are sharing and did not feel like including a description.

A reviewer of a recipe might not include review text if they did not feel particularly strong about a recipe. Whether the recipe was good or not, if they do not feel strongly about the recipe, they may just leave their default rating, different from person to person, and not leave any message behind it.

### Missingness Dependency

The column 'rating' is likely MAR. To check this, I will run a permutation test with 1000 iterations and a significance value of 0.05 to see if the missingness of 'rating' depends on 'saturated fat' or 'minutes'. 

A new column 'missing_rated' was added to a copy of the dataset not filtered to have only those with 15 or more reviews. That dataset was used because the filtered dataset also removes duplicates and this gets rid of a lot of individual ratings. This column was shuffled and used to compare differences in group means for 'saturated fat' and 'minutes'.

### Permutation Test on 'saturated fat'

**Null Hypothesis:** The missingness of 'rating' does not depend on the PDV of saturated fat contained in the recipe

**Alternative Hypothesis:** The missingness of 'rating' does depend on the PDV of saturated fat contained in the recipe

<iframe
	src="assets/missingness_sat_fat.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

The permutation test returned a p-value of **0**. It is essentially impossible to get the observed difference in group means by chance alone. Thus, we reject the null hypothesis and can conclude that the missingness of rating does depend on the amount of saturated fat is in the recipe.

### Permutation Test on 'minutes'

**Null Hypothesis:** The missingness of 'rating' does not depend on the minutes required to complete the recipe

**Alternative Hypothesis:** The missingness of 'rating' does depend on the minutes required to complete the recipe

<iframe
	src="assets/missingness_minutes.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

The permutation test returned a p-value of **0.126**. Since this is greater than our significance value 0.05, we fail to reject the null and cannot conclude that the missingness of 'rating' depends on the minutes required to complete the recipe.

## Hypothesis Testing

<iframe
	src="assets/fig_bivariate.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

As found in the bivariate analysis, the distribution of average rating for both healthy and unhealthy looks very similar, suggesting that the healthiness of a food does not influence average rating much.

Comparing the group means, healthy foods have an average rating of 4.72 and unhealthy foods have an average rating of 4.70, leaving a difference in group means of 0.02. A permutation test using difference in group means as the test statistic will be run to check if this difference is significant at the 0.05 significance level.

**Null Hypothesis:** Whether a recipe is healthy or not has no relation with its average rating and observed differences in the sample are due to random chance

**Alternative Hypothesis:** Healthy recipes have higher average ratings than unhealthy ones on average and the observed differences in the sample are not due to random chance

After 1,000 iterations, a graph of the empirical distribution of group mean differences can be made. The red line represents the observed difference found in our data.

<iframe
	src="assets/fig_mean_diffs.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

We found a p-value of **0.009** from our permutation test, so we reject the null at the 0.05 significance level. We can conclude that healthy recipes tend to have higher average ratings than healthy ones.

## Framing a Prediction Problem

In order to predict the average rating of recipes, I will use a regression model with the nutritional values of each recipe as the primary features.

I am choosing to predict average rating because the average rating of a recipe is the best measure of how positively a recipe is received by the community. This analysis was intended to see how the nutritional value of a dish affected the reception of a recipe, so average rating is the most logical response variable.

The root mean square error will be used to evaluate the model. RMSE is an effective metric to use for regression models as it measures how far off the predicted value is from the actual value. I am choosing to use this metric over the R-squared value because it is easier to understand and interpret.

At the time of prediction, all the nutritional values will be available and only rows gathered to be training data will be used to fit the model. The nutritional values are included by default in every recipe posted on food.com, even before any reviews are made.

## Baseline Model

For the baseline model, two columns from the dataset will be used to predict average rating: 'saturated fat' and 'healthy'

The 'saturated fat' column is already in numeric form and will be left as is.

The 'healthy' column is a categorical column with two possibilities, True or False. One Hot Encoding with the first column being dropped was used to convert the boolean column into a column of 1s and 0s.

The dataset was split into training and test sets with the test set being 25% of the original data.

The model had an RMSE of **0.2837608674635276** on the test set.

The plot below shows the predicted ratings and how far off they were from the actual rating.

<iframe
	src="assets/base_resids_plot.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

The model never predicted anything below a 4 and there is a strange hole in the range about 4.72 to 4.75 that the model would never predict. The model also tends to overrate recipes, likely due to the sheer quantity of highly rated recipes featured in the dataset.

## Final Model

The final model incorporates more nutritional values as features, using 'calories', 'total fat', 'sugar', 'sodium', 'protein', 'saturated fat', and 'carbohydrates' to predict average rating. The column 'healthy' that was used in the base model is not used here because it was attained by transforming 'protein' and 'carbohydrates'. Including it would be redundant.

The same training and test data that was used in the base model was also used for this model.

All the features aside from calories are PDV values whereas 'calories' simply states the number of calories the recipe contains. It also has the greatest variance and standard deviation of all the feataures. For these reasons, the 'calories' column was not left as is and it was transformed with Binarizer.

The threshold at the Binarizer was set to was a hyperparameter for this model. Different percentiles as well as the mean of the 'calories' column were tested to see which made for the most optimal model. The percentiles used were the 10th to the 90th in increments of 10.

The model had an RMSE of **0.28357740547392907** on the test set. This is a very small improvement from the base model. With such a small improvement from the base model despite including more features, it suggests that the features added do not have much weight when it comes to how well rated a recipe is.

The plot below shows the predicted ratings and how far off they were from the actual rating.

<iframe
	src="assets/final_resids_plot.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

This model does not contain the hole in the predicted ratings from 4.72 to 4.75 and predicted a larger variety of ratings. Interestingly, it also rated one recipe to have an average rating of over 5. Just like the base model, it overrates recipes more than it underrates them. The range of the residuals is about the same as the base model.

## Fairness Analysis

For my fairness analysis, I decided to group my dataset based on the number of steps required to make the dish. To the dataset containing the predicted values and residuals of my final model, I added a boolean column 'under_10_steps' that was True if the recipe required less than 10 steps.

Grouping by this categorical column and then finding the RMSE values of both groups, I found that recipes with over 10 steps had an RMSE of about 0.33 and recipes with under 10 steps had an RMSE of about 0.29. Though the values seem similar, a permutation test with a test statistic of difference in RMSE was conducted to see if the difference of 0.04 was statistically significant at the 0.05 significance level.

**Null Hypothesis:** The model is fair. Its precision for recipes with over and under 10 steps are roughly the same and differences are due to random chance.

**Alternative Hypothesis:** The model is unfair. The precision for recipes with under 10 steps is better than its precision for recipes with over 10 steps.

1,000 iterations were run by grouping on shuffled permutations of the 'under_10_steps' column and collecting the differences in RMSE.

The graph below shows the empirical distribution of differences in RMSE for those iterations and the red line shows the observed difference.

<iframe
	src="assets/fairness_rmse.html"
	width="660"
	height="440"
	frameborder="0"
></iframe>

The test returned a p-value of **0.03**, meaning that the difference is significant at a 0.05 significance level and the precision for recipes with under 10 steps is not the same as the precision for recipes with over 10 steps.