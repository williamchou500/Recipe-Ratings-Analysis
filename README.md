# Analysis of the Relationship Between Nutritional Value and Recipe Ratings

Author: William Chou

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

| Column    | Description         | Data Types   |
|:----------|:--------------------|:-------------|
| user_id   | User ID             | int64        |
| recipe_id | Recipe ID           | int64        |
| date      | Date of interaction | object       |
| rating    | Rating given        | int64        |
| review    | Review text         | object       |

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