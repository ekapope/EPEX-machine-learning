# Machine Learning Approaches for Electricity Markets Trading (EPEX spot)
Define a strategy to trade between day ahead and intraday electricity markets with the help of machine learning models. The project was initiated in collaboration with Delaware Consulting and IÉSEG School of Management. The final business presentation can be found [here](https://github.com/ekapope/EPEX-machine-learning/blob/master/EPEX_Hackathon_Business_Presentation.pdf).

## Project Workflow Summary
- Business understanding
    - Address business problem(s)
    - Potential trading strategy
- Data gathering and preparation
    - EPEX spot market data for France and Germany: R - web scraping [emarketcrawlR package](https://github.com/wagnertimo/emarketcrawlR)
    - Historical weather forecast: Python, API wrapper [wwo-hist](https://github.com/ekapope/WorldWeatherOnline)
    - French public holidays
    - Feature extraction and feature engineering
- Methodology
    - Experimental setup
    - Target variable definition and machine learning models
    - Insights and business application

---------------

## Project Background and Market Structure

Electricity price forecasting is a branch of energy forecasting which focuses on predicting the spot and forward prices in wholesale electricity markets. Over the years electricity price forecasts have become a fundamental input to energy companies' decision-making mechanisms at the corporate level.

In this project, we aim to forecast the prices of only the day ahead and the intraday market of France electricity market. The aim would also be to compare the prices on same days and hours in the intraday and day ahead market.

#### Day-Ahead Market

The price building mechanism for hourly products at the day-ahead market is conducted through an auction. Participating agents submit supply and demand bids containing information about quantity, price and delivery period on the following day to EPEX Spot. Bids can be submitted to EPEX Spot until the auction takes place at 12:00 a.m. every day, including weekend and public holidays. 

#### Intraday Market

The intraday market for hourly contracts at EPEX Spot is organized as a continuous trading market. Each contract can be bought and sold throughout until 30 minutes before delivery. Hence, traders are more flexible regarding trading time compared to the day-ahead auction. Like the bidding mechanism at the day-ahead market, players submit buy or sell orders for a certain contract with information about volume and price to EPEX Spot. 
 
## Data Selection and Motivation

Although, this project is mainly focused in France, we only choose data that is freely available so that the proposed models can be applied to other EU markets. In particular, we select the period from 01/01/2014 to 31/12/2018 as the time range of study, and we consider the following data:

1.	Day-ahead and intraday prices from the EPEX-France and Germany power exchanges.  

2.	Day-ahead forecasts of the grid load and generation capacity in France. Like in other European markets, these forecasts are available before the bid deadline on the website of the transmission system operators (TSOs): ENTSO-E data for France.  However, this was later excluded for this project due to data quality issue.

3.	Weather data (WorldWeatherOnline data)
Historical weather data could also be an important factor for the forecasting, for our research, we have decided to include them for wind and solar farm regions since 18% of overall production are from renewable energy.

4.	Calendar of public holidays in France in the defined time range. Public holidays influence in a significant way the economic and domestic activity and consequently the consumption of electricity. Therefore, we take them into account for the forecast.


## Experimental Setup
### 1.	Data partitioning (Training / Validation / Test)
To avoid data leakage, instead of random split, we must split data up and respect the temporal order in which values were observed. Therefore, we select arbitrary split points in the ordered list of observations and creating three new datasets. The first three years of data is now allocated for training set, one year for validation, and the last year for testing set. 
2014 – 2016 data was taken to be the train data set. 

#### Training set (60% of the original data set)
The training set is used to fit the models. In this phase, we created multiple algorithms to compare their performances during the Validation Phase. 

#### Validation set (20% of the original data set)
The validation set is used to estimate prediction error for model selection. This data set is used to compare the performances of the prediction algorithms that were created based on the training set. We choose the algorithm that has the best performance.
2017 was used to be the validation set

#### Test set (20% of the original data set)
We apply our chosen prediction algorithm on the test set in order to see how it is going to perform so we can have an idea about algorithm’s performance on unseen data.
              Year 2018 was held out to be the Test set. 
 
### 2.	Resampling methods and Cross Validation.
#### Random Oversampling
They can be used to alter the class distribution of the training data and both methods have been used to deal with class imbalance. The reason that altering the class distribution of the training data aids learning with highly-skewed data sets is that it effectively imposes non-uniform misclassification costs. 

The main disadvantage with oversampling is that by making exact copies of existing examples, it makes overfitting likely. 
#### SMOTE: Synthetic Minority Over-sampling Technique
The minority class is over-sampled by taking each minority class sample and introducing synthetic examples along the line segments joining any/any/all the k minority class nearest neighbors. Depending upon the amount of over-sampling required, neighbors from the k nearest neighbors are randomly chosen. 

#### Cross Validation Procedure
In this project, we will use walk forward validation method. 
This means for each validation step starting from January 2017, the data until the immediate previous months was used to train the model (January 2014 till end of December 2016). 

Then, data from January 2014 till end of January 2017 was used to train the model. And validation performance will be measured using February 2017 validation set.

To validate March 2017, the data from January 2014 till end of February 2017 was used. This process continues until December 2017. In the end, all 12 months, validation results are aggregated to calculate final performance of the algorithms. 


### 3.	Evaluation Metrics
Below metrics are used to evaluate the models.
#### AUC: Area Under the ROC Curve
AUC provides an aggregate measure of performance across all possible classification thresholds. One way of interpreting AUC is as the probability that the model ranks a random positive example more highly than a random negative example.
Sensitivity, Specificity
Sensitivity measures the proportion of actual positives that are correctly identified.
Specificity measures the proportion of actual negatives that are correctly identified.
#### Confusion matrix
True positives, true negatives, false positives, false negatives, and also true positive/false positive ratio.
#### F1 score
The F1 score is the harmonic mean of precision and sensitivity. It can be interpreted as a weighted average of the precision and recall, where an F1 score reaches its best value at 1 and worst score at 0. The relative contribution of precision and recall to the F1 score are equal.  

### Target variable definition and machine learning models
#### 1.	Target variable definition
This project is a binary classification problem. The target is defined as 1 when the price difference between intraday and day ahead is above or equal certain pre-defined threshold, and 0 otherwise. The interested thresholds are varied between 0 to 3 standard deviation of the average price differences, with 0.5 standard deviation incremental step as per below table.  

#### 2.	Variable Selection
Additionally, multiple algorithms like Forward Selection, Backward Selection, Hybrid Selection were tried out and finally Fisher score was selected to be the final one.

Fisher Score Technique was used to select the most importance predictors. For each training step, after calculated the fisher score, top 20 predictors are selected and fed into the model. This fisher score variable selection were applied to only logistic regression and lightGBM models due to time limitation of this project.

### ML Models
Below models were executed as part of this project.
-	Logistic Regression
-	Random Forest
-	XGBoost
-	LightGBM 

All 4 algorithms were trained on 12 walk-forward validation steps, with 7 different target thresholds and 2 random oversampling techniques. In the end, we will have 4 x 12 x 7 x 2 = 672 training iterations. The best model in term of annual financial gain will be selected and applied to test data set in year 2018.

### Profit and loss calculation
The result for each model (with each target threshold) will be compared with the baseline. The baseline scenario is defined as; for all time blocks in the validation/testing sets, we will buy at day ahead price and sell them at intraday price on the next day. With this setting, we can obtain the profit/loss metrics suppose that we do not use any prediction models.

### Financial metrics
Since the profit/loss for each predicted class is not equal, the false negative prediction is an opportunity loss, but the false positive likely leads to ‘actual loss’ since we take the wrong action. Therefore, we cannot only select our models based on the statistical measures of the performance. Several additional financial metrics are taken into consideration in order to justify the models. These additional metrics are defined as following. 

|Actual|Predicted|Type|Action|
| ------------- | ------------- | ------------- | ------------- |
|1	|0	|False negative|No action (opportunity loss) |
|1	|1	|True positive|Take action |
|0	|1	|False positive|Take action |
|0	|0	|True negative|	No action |

### References

Stoiber, J. (2017). The behavior of electricity prices at the German intraday market: A probabilistic functional data approach. 

Lago, J., De Ridder, F., Vrancx, P. and De Schutter, B. (2018). Forecasting day-ahead electricity prices in Europe: The importance of considering market integration.

Mccarthy, K., Zabar, B., & Weiss, G. (2005). Cost-Sensitive Learning vs. Sampling: Which is Best for
Handling Unbalanced Classes with Unequal Error Costs? Department of Computer and Information Science, Fordham University. Bronx, NY, USA.

Chawla, N., Bowyer, K., Hall, L. and Kegelmeyer, W. (2002). SMOTE: Synthetic Minority Over-sampling Technique. Journal of Artificial Intelligence Research, 16, pp.321-357
