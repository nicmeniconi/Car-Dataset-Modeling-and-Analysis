### Car Price Prediction Using Linear Models

In this project, I developed a linear regression model to predict car prices using a dataset that included various features such as car specifications and geographical information. The aim was to explore how different factors contribute to a car’s price, identify the most important features, and suggest future improvements to the --

### Car Price Prediction Using Linear Models

In this project, I developed a linear regression model to predict car prices using a dataset that included various features such as car specifications and geographical information. The aim was to explore how different factors contribute to a car’s price, identify the most important features, and suggest future improvements to the model.

#### Data Overview and Preprocessing

The dataset consisted of 308,225 entries and 19 columns, including categorical features such as 'manufacturer,' 'transmission,' and 'fuel type,' and numerical features such as 'year,' 'odometer,' and latitude/longitude coordinates. After exploring the data, several preprocessing steps were applied:

1. **Handling Raw Data:** Initially, the dataset contained irrelevant columns such as 'id' and 'VIN,' which were removed since they wouldn't contribute to modeling. After identifying the categorical and numerical columns, string values in categorical columns were standardized by converting them to lowercase and stripping extra spaces.

2. **Handling Missing Values:**  
   A detailed inspection of missing data was performed to understand the extent of missing values in different columns. For example, 'size' had more than 71% missing values, so it was dropped. Columns like 'cylinders' (41.62% missing) and 'condition' (40.78% missing) were imputed using various strategies.

   - **Cylinders Imputation:** Missing values in the 'cylinders' column were imputed by first identifying the most common cylinder configuration for each 'model' and 'manufacturer.' If the model had a frequent cylinder value, it was used for imputation; otherwise, the most common value for the manufacturer was used. This approach reduced the missing values in 'cylinders' from 41.62% to 0.83%.
   - **Condition Imputation:** The 'condition' column was ordinally ranked, and a logistic regression model was trained on 'odometer' and 'year' to predict the missing 'condition' values. This reduced missing values in 'condition' from 40.78% to 0.85%.
   - **Drive Imputation:** Missing values in 'drive' were first imputed based on the most common value within the same 'model.' For entries still missing 'drive' after this, the imputation was performed based on the most common value for the 'type' of vehicle. This reduced missing 'drive' values from 30.58% to 2.03%.
   - **Type Imputation:** Similar to 'drive,' the 'type' of the vehicle was imputed based on the most common value for the 'model' first, and then using combinations of 'drive,' 'fuel,' and 'cylinders.' This method brought down missing values in 'type' from 21.75% to 2.21%.

3. **Feature Engineering:**  
   Latitude and longitude coordinates were added for each 'state' and 'region' using external JSON files that mapped geographical coordinates. This enhanced the dataset with location-specific features that could capture regional price variations. The missing values in categorical columns such as 'paint_color' were handled by dropping the column entirely, as imputing this feature was not feasible.

4. **Log Transformation and Scaling:**  
   The target variable, 'price,' was log-transformed to reduce skewness and stabilize variance. Numerical features were standardized using `StandardScaler` to ensure that all variables were on the same scale, which is important for models like linear regression.

5. **Pipeline Construction:**  
   A custom pipeline was built to apply all these preprocessing steps, including cleaning categorical data, imputing missing values, and mapping coordinates. This pipeline was applied to both the training and testing datasets, ensuring consistent preprocessing across the data splits.

After these preprocessing steps, the dataset was split into training and testing sets, with 90.13% of the original dataset retained after imputation and elimination of NaNs. This process allowed the model to learn from a cleastrategies, and raw data cleaning. cle strategies, and raw data cleaning. Selection (SFS).

#### Model Performance

The baseline Mean Squared Error (MSE) was calculated as 7.41 for the training set and 7.32 for the test set, representing the error when predicting the average price for every car.

1. **Simple Linear Regression:** Using features with the highest correlation to 'price' (such as 'condition' and 'cylinders'), the model achieved a slight reduction in MSE (Train: 7.35, Test: 7.27). The improvement over the baseline was minimal.
2. **Polynomial Regression:** Adding polynomial features reduced the MSE further to 6.86 for training and 6.77 for testing. This shows that non-linear relationships were present in the data, and capturing them improved predictive performance.
3. **Lasso vs Ridge vs GridSearchCV:**  
   Lasso and Ridge regression were applied to improve the model through regularization, reducing the likelihood of overfitting and managing multicollinearity among features.

   - **Lasso Regression:** This method adds an L1 regularization penalty, shrinking some coefficients to zero, which can lead to feature selection. In the model, Lasso achieved a test MSE of 7.03, a significant improvement over the baseline. The sparsity induced by Lasso allowed it to focus on the most relevant features, eliminating noise from less important ones.

   - **Ridge Regression:** Ridge uses an L2 penalty, shrinking coefficients but keeping all features in the model. It performed similarly to Lasso with a test MSE of 6.97. Ridge tends to perform better when all features are potentially relevant because it doesn't entirely eliminate features like Lasso does.

   - **GridSearchCV on Lasso and Ridge:** To optimize the regularization strength (`alpha`), I applied GridSearchCV to both models. The best hyperparameters were identified as alpha = 0.1 for Lasso and Ridge. Post-tuning, both models performed similarly (Lasso Test MSE: 6.97, Ridge Test MSE: 6.97), but the regularized models outperformed the baseline and untuned versions. The tuning process indicated that moderate regularization worked best for this dataset, suggesting that while overfitting was not a major issue, slight regularization improved the generalization of the models.

4. **Sequential Feature Selection (SFS):**  
   Sequential Feature Selection (SFS) was employed to select the most relevant features for the linear regression model by sequentially adding features that contributed most to reducing the error. SFS selected five features: 'transmission,' 'cylinders,' 'condition,' 'region_lat,' and 'odometer.' 

   After applying SFS, the model's MSE decreased to 7.07 for training and 6.98 for testing, showing a modest improvement compared to the baseline and untuned models. The SFS process demonstrated that even a simple subset of features could yield reasonably accurate predictions without including all available data, further reducing the risk of overfitting.

#### Feature Importance

One of the key goals of this analysis was to identify which features were most important in predicting car prices. The importance of each feature was determined by examining the coefficients assigned to them by different models, with larger absolute values indicating higher importance.

Across models, the most important features consistently included:
- **Transmission:** This feature had a positive impact across all models, with automatic transmissions leading to higher car prices. In both Lasso and Ridge models, transmission had the largest positive coefficient.
- **Cylinders:** More cylinders were associated with higher prices, likely due to the association with performance and larger, more powerful vehicles.
- **Condition:** Better condition ('like new' or 'excellent') was consistently linked to higher prices, while poor condition had a negative impact.
- **Odometer:** As expected, higher mileage was associated with lower car prices, as cars with more miles typically require more maintenance and have a reduced lifespan.

However, there were some differences in the most important features across models:
- **Lasso:** Due to Lasso's ability to shrink irrelevant feature coefficients to zero, some features were dropped, particularly those with little explanatory power. This resulted in a more concise model that focused on a subset of the most relevant predictors. For instance, Lasso dropped some geographic coordinates, which might suggest that they were less critical in determining price variations once other factors like transmission and condition were included.
- **Ridge:** Ridge retained all features, distributing the impact more evenly across variables. As a result, features like 'state_lon' and 'region_lat,' which represent geographic information, were still included in the model, albeit with smaller contributions compared to transmission and cylinders.
- **Sequential Feature Selection (SFS):** The selected features ('transmission,' 'cylinders,' 'condition,' 'region_lat,' and 'odometer') were also identified as key features by other models, validating their importance. The SFS method proved that even with fewer variables, the model could maintain strong predictive performance.

#### Should the Model's Predictions be Considered?

The performance of the models suggests that the predictions are reasonable but should be interpreted with caution. While the models reduced error significantly compared to the baseline, certain limitations should be considered:
- **Simplicity of Linear Models:** Linear regression assumes a linear relationship between features and price. Although polynomial features help capture some non-linearity, more complex models like Gradient Boosting or Random Forests might better capture intricate patterns in the data.
- **Geographical Factors:** While geographical coordinates were included in the models, they may not fully capture regional variations in car prices. Introducing more sophisticated geographic markers (e.g., local demand or competition) could improve the model’s accuracy.
- **Omitted Features:** Certain factors, such as the color of the car, interior features, or brand reputation, were not included in the dataset but may have a significant impact on car prices.

#### Suggestions for Future Work

1. **Handling Interaction Terms:** While polynomial features were effective, exploring interactions between categorical and numerical variables (e.g., 'manufacturer' and 'year') might yield better performance.
2. **Improving Feature Engineering:** Introducing new features, such as average regional car prices or additional geographical markers, could improve the model’s accuracy by capturing reginear Models:** While linear models provided a solid baseline, more complex models like Gradient Boosting Machines (GBMs) or Random Forests might capture additional patterns in the data.
4. **Further Regularization Tuning:** Hyperparameter tuning of Lasso and Ridge regularization strength could yield a model that better balances bias and variance, potentially improving generalization performance.
