# Bite-Sized Insights: Exploring What Makes Recipes Shine

## Introduction
This project investigates recipe ratings using a dataset from Food.com that includes over 200,000 recipes and user-submitted reviews since 2008. Each recipe includes metadata such as preparation time, number of steps, and detailed nutritional information. The central question I explore is: 
* **What features of a recipe are associated with receiving a higher average rating?** 

Understanding which types of recipes perform well could be useful for food bloggers, home chefs, or platforms recommending recipes. The dataset used has approximately X rows and includes relevant columns such as:
* **minutes**: the recipe's preparation time (in minutes)
* **n_steps**: the number of steps in a recipe
* **calories**: the number of calories in a recipe
* parsed nutritional values like **calories** (kcal), **protein**, and **sugar** (in PDV - Percent Daily Value)

## Data Cleaning
To prepare the recipes dataset for analysis, I first loaded two CSV files: one containing detailed recipe information and another containing user-submitted ratings and reviews. I merged these two datasets using a left join on the recipe ID, which ensured that all recipes were retained in the merged dataset, even those that did not receive any ratings. This allowed for a more complete view of the recipe collection.

Next, I addressed issues in the ratings data. Since ratings on Food.com are expected to fall between 1 and 5, I treated any rating of 0 as invalid or missing. These were likely placeholder values, so I replaced them with NaN to avoid skewing average rating calculations or future modeling tasks.

To prepare for any time-based analysis, I also converted the submitted and date columns to datetime objects. This step enabled more efficient filtering, time comparisons, and grouping by year or other time periods, which could be useful in understanding recipe trends over time.

I then calculated the average rating for each recipe by grouping the data by recipe ID and computing the mean of all non-missing ratings. This average rating was added back into the dataset as a new column. Having a single summary value for each recipe's rating allows for easier analysis and visualization, especially when examining recipe-level trends.

Finally, I cleaned the nutrition column, which originally stored its values as a string-formatted list. To make the nutrition data usable, I parsed the string to extract individual numeric values, assigning each one to its corresponding nutrient: **calories, total fat, sugar, sodium, protein, saturated fat**, and **carbohydrates**. These new columns made it possible to analyze nutritional components separately and incorporate them into visualizations or predictive models.

Together, these data cleaning steps ensured that the dataset was well-structured and ready for meaningful analysis.


## Univariate Analysis
The histogram below shows the distribution of average recipe ratings. The distribution is right-skewed, with most recipes receiving ratings of 4.0 or higher, suggesting that users generally rate recipes favorably. This is important because it means there's limited variation in ratings, so small differences in recipe features may be associated with subtle but meaningful changes in rating, which is an important consideration when identifying what makes a recipe highly rated.
<iframe src="assets/avg_rating_dist.html" width="800" height="600" frameborder="0"></iframe>

## Bivariate Analysis
The scatter plot below shows no strong association between preparation time and average rating. However, a large proportion of recipes that take 60 minutes or less tend to have average ratings between 4 and 5, suggesting that quicker recipes are both popular and well-received.
* The visualization also only includes recipes with preparation times less than 500 minutes for ease of reading.
<iframe src="assets/prep_time_vs_mins.html" width="800" height="600" frameborder="0"></iframe>

## Interesting Aggregates
### Prep Time vs. Average Rating
The table below shows the average rating of recipes grouped by preparation time. Recipes that take 60 minutes or less tend to receive the highest average ratings, with a slight drop observed for longer cooking times (particularly those between 4 and 8 hours), suggesting that users generally favor quicker, more convenient recipes.

| time_bin    |   avg_rating |
|:------------|-------------:|
| ≤30 min     |      4.69494 |
| 31–60 min   |      4.66619 |
| 61–120 min  |      4.67641 |
| 121–240 min |      4.66316 |
| 241-480 min |      4.56732 |

### Protein (PDV) vs. Average Rating
The table below shows the average recipe rating grouped by protein content, measured as percent daily value (PDV). Recipes with lower to moderate protein levels (up to 40% PDV) tend to receive slightly higher ratings on average, while recipes with very high protein content see a gradual decline in average rating. This may suggest that users prefer recipes with balanced or moderate nutritional profiles over extremely protein-dense options.

| protein_bin   |   avg_rating |
|:--------------|-------------:|
| 0–20 %DV      |      4.68035 |
| 21–40 %DV     |      4.67754 |
| 41–60 %DV     |      4.6644  |
| 61-80 %DV     |      4.66423 |
| 81+ %DV       |      4.65417 |

## Prediction Problem
For my prediction problem, I aim to predict the average rating a recipe will receive based on features available at the time it is submitted. This is a regression problem, since the target variable (avg_rating) is continuous. I chose this target because it reflects user satisfaction and is central to understanding what makes a recipe successful. I will evaluate model performance using Root Mean Squared Error (RMSE), which penalizes larger errors more heavily and is standard for regression problems. I avoid using accuracy or classification metrics, since this is not a classification task.

## Baseline Model
For my baseline model, I used a linear regression model to predict a recipe’s average rating (avg_rating) based on three features: **minutes, n_steps**, and **calories**. All of these features are quantitative and represent meaningful aspects of a recipe—how long it takes to prepare, how complex it is, and how nutritionally dense it is. This baseline model did not include any ordinal or nominal features, so no encoding was necessary.

To prepare the data, I removed rows with missing values in either the input features or the target variable. I then applied a StandardScaler to standardize the numerical features, ensuring they were on the same scale for training. Both the preprocessing and the model itself were implemented in a single scikit-learn pipeline.

I evaluated the model using **Root Mean Squared Error (RMSE)**, which measures the average difference between the predicted and actual average ratings and is expressed in the same units as the target variable. The model achieved an RMSE of **0.4922** on the test set. Given that recipe ratings tend to fall between 4 and 5, this level of error indicates that the model struggles to make accurate predictions using only the selected features. While the performance is limited, this is expected for a simple baseline model, and it sets the stage for improvement in the next step through feature engineering and more advanced modeling.


## Final Model
For my final model, I aimed to improve upon the baseline by engineering new features, adding a categorical variable, and using a more flexible regression algorithm. In addition to the original quantitative features (minutes, n_steps, and calories), I added the following engineered features:

* **is_quick**: a binary variable indicating whether a recipe takes under 30 minutes to prepare, reflecting user preference for quicker meals.

* **calories per step**: a measure of how calorically dense a recipe is relative to its complexity, potentially indicating whether a recipe is "worth the effort" for the calories it delivers.

* **protein-to-carbohydrate ratio**: to reflect the nutritional balance of a recipe, which may impact user satisfaction.

I also created a categorical feature by binning preparation time (minutes) into five time ranges (≤30, 31–60, 61–120, 121–240, and 240+), and encoded it using a OneHotEncoder.

For modeling, I used a Random Forest Regressor, which is capable of capturing nonlinear relationships and interactions between features. I performed a grid search over key hyperparameters (n_estimators (number of trees) and max_depth (maximum tree depth)) using 3-fold cross-validation to tune the model. Importantly, I evaluated model performance on the same test set used in my baseline model to ensure a fair comparison.

The final model achieved an RMSE of **0.358**, improving substantially over the baseline model’s RMSE of 0.4922. While it’s true that adding more features can sometimes lead to apparent improvements simply by increasing model complexity, this result reflects genuine generalization improvement, not just overfitting, because the test set was held out during training, and the engineered features were chosen based on their relevance to the data-generating process. The results suggest that these additional inputs helped the model more accurately capture the factors that influence recipe ratings.

