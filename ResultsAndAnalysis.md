## Libraries Used
- pandas
- scikit-learn
- numpy
- RuleFit

## Data Cleaning
I checked to ensure that the data was complete, the unique values in the categorical variable columns made sense, and that extreme values which did not make sense were removed. I was able to validate the data by plotting each variable (including salary) to remove any rows that did not make sense. I also made sure to check all the counts for categorical summed to 1 million. The response variable 'salary' had a few zero values. Those rows were removed to avoid outliers. The count plots showed uniformly distributed categories for each categorical feature except 'major' in which half the jobs correspond to ‘NONE’ (no major) and the other half are somewhat uniformly distributed across majors.

## Algorithms Used
I chose Gradient Boosting Regressor as my final algorithm since it is known for its robustness and results. It uses a bunch of weak learners to improve on its prediction power iteratively. It is good for modeling non-linearity and provides opportunities for parameter tuning to avoid overfitting. Moreover, it does not lose interpretability either since it ranks features based on how much of an improvement in prediction they resulted in. I considered using Linear Regression, Lasso, Random Forests and RuleFit. The last of these methods, RuleFit is a powerful method which combines the interpretability of Linear Regression and the robustness of ensemble techniques e.g. random forests or gradient boosting to produce highly interpretable and accurate results after parameter tuning. However, the current libraries for RuleFit are not optimized to work well with large datasets and large number of estimators. To show how it works, I have presented an unoptimized version of rulefit in my notebook file.

## Gradient Boosting
Gradient Boosting involves three elements: a loss function to be minimized - in our case it is squared error loss, weak learners which are only slightly better than randomly predicting, and an additive model to which weak learners are added iteratively. In GB, trees are used as weak learners. These trees can be modified based on parameters such as tree depth, splits, nodes, etc. In my model I used grid search to optimize the depth parameter and settled on 7 levels. Splits are made using the greedy approach i.e. at every level the split is chosen based on how much it minimizes loss. Weak learners are constrained using parameters mentioned above so that they continue to be weak. Trees are added to the model using gradient descent. They are parametrized so that they follow the gradient and minimize loss. Eventually the results of all trees are combined in proportion to the learning rate. In my model the optimal learning rate was 0.2. Learning rate is generally used to slow down learning by the algorithm and therefore prevent overfitting. A greater number of trees is required with a smaller learning rate.

## Features Used
Broadly speaking there were two types of features: categorical and continuous. Within categorical features, I identified two types: ordinal categorical features and nominal categorical features. Degree and job type features could be ordered as follows:

1. Janitor, Junior, Senior, Manager, Vice President, CTO, CFO, CEO
2. None, High School, Bachelors, Masters, Doctoral

These ordinal features were therefore coded as ordered ints ranging from 1 to number of categories where 1 represents the lowest level.

The other categorical features - major and industry - became one-hot-encoded dummy variables where each category with the feature became represented as a variable with 1 representing the feature being present in the row, 0 otherwise. One hot encoding is a fairly common approach to dealing with categorical variables.

I tested algorithms using ordinal variables as mentioned above as well as using one hot encoded versions of job type and degree. Ordinal encoding beat one hot encoding for these two features.

Another interesting thing to note was that the variable ‘years of experience’ had 25 different categories (or years) which could be binned into fewer categories and used as an ordinally encoded variable. However, since I do not possess complete knowledge of how years of experience impact salary, and therefore how to bin them e.g. bin size, etc., I decided to leave it as is. The variable miles from metropolis was also used as is.

I left out company ID as a feature since it did not represent a lot information currently. That feature could perhaps be used in a variable called counts where the number of times each id appears is counted. However, that would not be a very good indicator of the size of the company or salary. Perhaps with data regarding company size etc, company ID would become useful.

## Model Training and Issues Encountered
The first stage for picking the right model was model selection. For this stage, I split the data into train and validation sets. For each algorithm, the model was trained on 66% of the training set and predictions were made for the remaining 34%. Each algorithm was evaluated based on mean squared error, a common performance metric for regression.

Eventually, gradient boosting proved to be the best algorithm for this model. This method allows room for a lot of parameter tuning and once those parameters are fine tuned, it results in close predictions. My parameters of choice for fine tuning were learning rate, depth and n_estimators. I held number of trees (n_estimators) fixed at 50 and used grid search to cross validate over the following parameter space:

- Tree depth: 7,8,9
- Learning Rate: 0.2, 0.1, 0.05

According to the cross validated grid search results, the optimal parameter combination was tree depth of 7 and learning rate of 0.2. Other than these i tried increasing learning rate but the improvement was minimal. I also experimented with various numbers of estimators and settled on 40 because above 40, the improvements in MSE were marginal. The biggest issue that concerned me during training was overfitting. I wanted an optimized algorithm but not over optimized. Overfitting occurs when the model fits the training set too well and the test set not so much. In practice, when test error exceeds training error, the model is said to be overfitting. Therefore, I tried to keep the depth, the number of trees as well as the learning rate small to prevent overfitting.

##Assessing Prediction Accuracy
I used Mean Squared Error (MSE) as my metric of choice to measure performance of each model. Generally in regression the options are MSE, MAE (Mean Absolute Error and Median Absolute Error), and R_squared. MSE is the most common approach to
measuring performance and it is especially useful since it penalizes large errors heavily. MAE is another good approach since it is more interpretable than MSE - it literally tells you how off you are on average in your predictions. RMSE (root mean squared error) is not the same as MAE but is a good proxy for Mean Absolute Error.

Lastly, R squared which represents the degree of variability in the response that is a direct result of predictor variables is another commonly used metric. In this case however, feature importance represents how much variability each feature results in. Using R squared therefore, was somewhat redundant here.

##Feature Importance
One of the best qualities of gradient boosting is how it identifies feature importance. Feature importance is defined by the extent to which predictions were improved as a result of the introduction of a variable. The greater the improvement in prediction, the higher importance is attached to that feature. The following graph summarizes the feature importances relative to the most important feature in the prediction of salary.

|<img src="https://github.com/zdanish1/SalaryPrediction/blob/master/feature_importance.png" alt="feature importance" width="380" height="300">|
| :--: |
|Fig 1|

As the graph above shows, job type i.e. the position (CEO vs Janitor), years of experience and miles from metropolis were the most important features in predicting salary. Degree (PhD versus High School) also plays an important role in salary prediction. Amongst the industries, some industries were more important than others e.g. web and health. Lastly, the major of studies was the least important predictor for salary.
