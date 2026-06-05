## Introduction 

I investigate how competition, location, and other features correlate with and predict whether a business permenantly closes. This is an important question for business owners; if you can predict whether a business will fail based on a set of features it can inform the business owner's optimal decisions. The setting for the data is Hawaiian google maps data, which describes the metadata and information on a subset of reviews for locations. This is a relevant setting because much of the publicly available information on locations, such as their ratings, location, and number of competitors can be key predictors for the business's success. The dataset on location metadata has 21507 locations; relevant columns include the address, text information including name, description and miscelaneous information, latitude and longitude, average ratings and the number of reviews, the hours of the location, and the business's state. Address, latitude, longitude, name, and description are self explanitory. Miscelaneous information contains other information on the location such as if it does takeout, if it is wheelchair accessable, and more. Average rating is the average user reported rating on google maps and the number of reviews is the total number of reviews for that location. The hours of the location contains information on which days of the week the location is open, and which hours it is open. The business's state contains information on whether the location is currently open, close, or permenantly closed. This provides the response I wish to predict in this project. 


## Data Cleaning and Feature Engineering


Important features which I create are, the close locations, similar locations, and similar and close locations which I classify as a location's "competitors". Since not all locations are businesses, they may not be competitors in the traditional economic sense, however similar attractions such as two waterfalls may still be seen as competing with each other for visitors. First, I measure the distance between locations using haversine distance which calculates the distance between two locations on a sphere using their latitude and longitude, which are the location variables given in the dataset. I then measure "similarity" in locations using the cosine similarity between each location's name, description, and some elements of the miscellaneous information. Particularly I exclude miscellaneous information on accessibility and health and safety, since that information is shared across many unrelated information. Combining this information I create as set of locations that are both similar and close for each location. Finally, I filter for locations that are similar and within the same price category, although since many price values are missing this will not be the primary unit for analysis. 

I also clean information like price and zipcode by removing values which should not be in the dataset such as ₩, non-english zip codes, and non-Hawian zipcode. I extracted zipcodes from address information using regular expressions. I then create a feature which characterizes whether a location is rural, semi-urban, or urban using the RUCA codes developed by the USDA. I characterize urban as 1-3, semi-urban as 4-7, and rural as 8-10. 

Other potentially useful features about individual reviews are aggregated then merged into the location level dataset. I create features for review count, minimum rating, median rating, and the average rating amongst repeat reviewers who have more than five reviews. I construct a variable for the average response time of the business owner by taking the difference between the time the review was posted and the response time, then aggregating these differences. However, most of the analysis does not use these variables, since many locations in the metadata dataset do not have a match in the reviews dataset.

I create variables for the number of days the business is open and the average number of hours the business is open. I construct this information from the hours variable in the orignial metadata dataset. Finally, I construct an indicator for whether a business is closed, this will serve as the dependent variable in the analysis.The head of the dataset is shown below. 

<iframe src="assets/mainDF.html" width="600" height="450" frameborder="0"></iframe>

  

## Exploratory Data Analysis

**Univariate Analysis**

I plot the histograms for five variables of interest below. 

<iframe src="assets/isClosed.html" width="600" height="450" frameborder="0"></iframe>

As would be expected by far the majority of the locations in the dataset are classified as operating (0), while only a few are closed (1). This is likely due to google removing businesses which are closed for too long. An important consideration is that this is a very unbalanced distribution, which will make prediction difficult.
  
<iframe src="assets/avgRating.html" width="600" height="450" frameborder="0"></iframe>

The histogram of average ratings shows that on average ratings are relatively high, primarily centered around 4.5 stars with a spike at 5 stars. This could potentially be because many firms leave fake reviews or because people genuinely do rate locations highly. There is likely some form of adverse selection at play; locations which do not perform well are not going to survive and then may not appear in the dataset. 


<iframe src="assets/price.html" width="600" height="450" frameborder="0"></iframe>

The most common non-missing price category in the dataset is $$ followed by $. More expensive options are much less common. However, most values in the dataset are missing, which could potentially be problematic when analying interactions with price. 

<iframe src="assets/numCloseLocations.html" width="600" height="450" frameborder="0"></iframe>

There are a significant number of locations with few close locations, however there are also many with a great deal of close locations. This is the expected result in a location like Hawaii, where there are many rural areas with few locations, but also urban areas such as Honolulu where there will be many locations in close proximity. 

<iframe src="assets/numCompetitors.html" width="600" height="450" frameborder="0"></iframe>

Most firms have very few competitors or no competitors which is expected since in most cases there will be only a few similar businesses within a small radius. Notably, there are also a few locations with many competitors; assuming that this location doesn't actually have over a thousand competitors, this reflects the limitations of the cosine similarity methodology in correctly classifying which locations are actually similar to each other. 

**Bivariate Analysis**

I begin by analyzing connections between whether a location is closed and characteristics such as price and average rating. Looking at the relationship between average rating and whether a business is closed or not, there does seem to be a relationship where as rating increases, the proportion of that rating that a closed decreases. For very small ratings, the proportion is very high, although this could be a result of the low amount of observations at this small rating. 

<iframe src="assets/isCloseAvgRating.html" width="600" height="450" frameborder="0"></iframe>

Looking at the relationship between price and whether a location is closed, the relationship does appear to be less significant. I plot the relationship below. However despite this there does seem to be more closed locations for \$\$\$ priced locations than the other categories. There may be a nieche where a location is expensive, but not the very best so it closes. 

<iframe src="assets/isClosePriceRange.html" width="600" height="450" frameborder="0"></iframe>

Now I consider secondary relationships, such as the price range and the number of competitors in the dataset. A bar plot showing the number of competitors by price range is shown below. 
<iframe src="assets/competitorsbyPriceRange.html" width="600" height="450" frameborder="0"></iframe>

Across price categories, the number of competitors are somewhat similar, with a decreasing median and maximum number of competitors as price increase. The main shift occurs going from $$ to $$$, with the two lower price categories and the two higher price categories generally being similar. This plot provides some evidence to support a lower number of competitors is correlatated with a higher price. 

Looking at the relationship between average rating and price, bar plot is shown below. 

<iframe src="assets/ratingCompetitorsBar.html" width="600" height="450" frameborder="0"></iframe>

The bar plot of number of competitors by rating shows that for most of the ratings distribution number of competitors does not differ too much by median. In the top few bars of the ratings distribution there is a difference and reduction in median number of competitors. The may be seen as a counter intuitive result, since you would expect the quality to increase along with number of competitors. This may reflect that the result from the previous plots, since higher priced restaurants have fewer competitors and higher ratings. The bottom of the distribution also as few competitors which is an expected result. 

Further visualizing this relationship, a scatter plot of ratings by number of competitors within the same price range shows a slightly positive trend, although very it is very small. This may indicate there is a relatively small relationship between competition and quality in this dataset. 


<iframe src="assets/priceRangeCompetitorsRating.html" width="600" height="450" frameborder="0"></iframe>


**Aggregates**

Looking at aggregates by price category also yields interesting results, below is an aggregate table of the averages of various numerical variables, aggregated by price category. 

<iframe src="assets/groupbyPrice.html" width="600" height="450" frameborder="0"></iframe>

As expected, average rating increases on average as price increases. Number of reviews also increases, which could reflect that people are enthusiastic to write a review if they have spent lots of money, for example an upscale restaurant. The number of close locations also increases as price increases. This is likely because more expensive locations are going to be in cities, where there are many close locations. On the other hand, number of competitors is generally decreasing as price increases. This aligns with the hypothesis that competition is negatively correlated with price. 


Finally, I look at the correlation matrix of all numeric variables in the dataset, the table is shown below

<iframe src="assets/correlationMatrix.html" width="600" height="450" frameborder="0"></iframe>

Some of the significant correlations from this matrix are number of reviews and average rating, which are positively correlated. Number of reviews is also positively correlated with the average response time. Another notable result is that ratings are  negatively correlated with the number of close locations and competitors, but slightly positively correlated with competitors within the same price range. 


## Assessments of Missingness

  

One important consideration for this dataset is the missingness of various data. Particularly, important variables like price and description are mostly missing. Fpr price, it is likely that it is not missing at random. Locations which have no cost or a very low cost are more likely to be missing than highly expensive restaurants. Particually, reviewers from low cost locations are less likely to note the price in their review or post information such as the menu which google uses to categorize the price. Therefore it is reasonable to say that price's missingness depends on the price itself. 


An example of data which is likely to be MAR is the text of individual user reviews. Intuitively, more extreme ratings are more likely to have text associated with them, either positive or negative; text is missing dependent on the user's rating. To test this, I use permutation testing with two sample Kolmogorov-Smirnov tests comparing the sample for which text is missing and the sample for which text is present. The result of this permutation test rejects the null that missingess does not depend on rating, which would indicate text missingness is missing at random dependent on at least rating. I plot the results of the permutation test below. 

<iframe src="assets/textNAFull.html" width="600" height="450" frameborder="0"></iframe>

 One consideration is that this test may have too much power and lead to false positives. To address this potential issue, I take a completely random subsample of the dataset and compute the same permutation test on it; the result still rejects the null. I plot the results from this test below. 

 <iframe src="assets/textNASubset.html" width="600" height="450" frameborder="0"></iframe>


I also find that missing does not depend on the time the review was sent. This is expected because time should not have a significant effect on whether a person writes text in the review. Again, I perform the permutation testing procdure on the reduced sample, since when performing it on the full sample the test has too much power. The results are plotted below. 


<iframe src="assets/textNATime.html" width="600" height="450" frameborder="0"></iframe>

One caviate is that reducing the sample in such a way is highly suceptible to which rows get picked. Although this is necessary to reduce the rates of false positives when running this permuation test, these results should be interpreted with caution. 



## Hypothesis Testing



**Differences is Rate of Closure by Price Category and Average Rating**


I conduct hypothesis testing to examine whether rate of closure differs by price category and whether the average rating is the same across closed locations and open locations.

First, I test whether the difference in the rate of closure is the same across price categories. To do this, I compare each pair of price categories with a difference in means t test with the null hypothesis that the difference between the means is zero and the alternative hypothesis that the difference is not equal to zero. $$H_0: \bar x - \bar y = 0, \ \ H_a: \bar x - \bar y \ne 0$$ 

The results align with what was seen in the exploritory data analyis, particularly that the difference is not too large except for the difference between one $ and three \$\$\$ which is significant at the 5% level. 

I also test whether the mean average rating of closed business is the same as operating business. To do this, again I use a difference in means t test with the null that $$H_0: \bar x - \bar y = 0, \ \ H_a: \bar x - \bar y \ne 0$$

This test strongly rejects the null hypothesis at the 1% level. This is evidence that the average rating of operating locations is higher than the average rating of closed locations; the oberserved difference is 0.155. This aligns with what would be expected, closed locations having worse ratings on average than operating locations which indicates that quality may have an impact on location closure. As further evidence of this differnce, I plot the empirical probability density function for each category below; closed locations' density is located towards lower ratings than operating locations' density. 

<iframe src="assets/avgRatingsClosurePDF.html" width="600" height="450" frameborder="0"></iframe>



## Closure Classification 

An important consideration for business owners is the risk of closure for their business. Using an information set to predict whether a location will close may be useful for a current or prospective business owner when making decisions. Therefore, I build a binary classification model which predicts whether a location is operating versus permenantly or temporarily closed. As an evaluation metric, I will primarily use F-1 score. This is the best metric because of how it balances precision and recall. Both are important for this problem because you wish to both avoid a closure while also avoiding being overly cautious. I calculate this metric using perfomance on a test set of the data. 


**Baseline Model**

As a baseline model for prediction I use logistic regression with price and number of competitors. I chose to one hot encode price category even though it is ordinal because much of the price information is missing; an ordinal encoding would not work since 'missing' does not fit well into the ordinal scheme. I also simply standardize number of competitors. I choose logistic regression as a baseline becausae it is a simple and interpretable model which does not require hyperparameter tuning. The result of this model is a macro f1 score of 0.539, notably dragged down by the f1 score of 0.213 when the model predicts a location is closed. The confusion matrix is shown below.

<iframe src="assets/baselineConfusionMatrix.html" width="600" height="450" frameborder="0"></iframe>

This model performs decently, but is by no means good. About $\frac15$ predictions of closed are correct, meaning it is very bad at characterizing when a location is actually closed. To improve predictions, in the next section I consider adding in more features and different models which may improve predictions. 




## Final model

When testing models, I add a variety of features from the dataset into these models. I use one hot encodings of categorical variables such as zipcode and rural urban status. These one hot encodings seek to capture variation by location and how urban a zipcode is. For numerical variables, I standardize them before using them for prediction, which is useful for evaluating feature importance. I add numerical variables such as the number of reviews, average rating, number of days open, average house open, number of close locations, and number of competitors within price range. Some of these variables suffer from being near colinear, for example the number of competitors and the number of competitors within price range. In a final model specification, the number of variables will be reduced to account for these multicollinearity issues.  

I now consider several classification models, such as logistic regression, K neighbors classification, decision trees, and random forrest classification on this larger set of features. These models vary significantly in their complexity, interpretability, and performance. Comparing the models, KNN classification performs very poorly, having an F-1 score of 0.144 when predicting closure, mainly due to a very low recall. This means the model heavily underpredicts the business closing, which is particularly bad for this application because it will lead to overconfidence about whether or not the location will succeed.

Logistic regression and decision tree classification perform similarly to each other; both have a macro average arround .5 and similar accuracy and recall. One notable aspect of their performance is that their accuracy is low while the recall is higher for predictions of closed. This would indicate that these models are predicting the business is closed often, but many of these predictions are incorrect. Since I am not only interested in reducing the rate of false negatives, these models are not optimal for the task. 

Finally, the random forest model is the best performing on macro average F1 score across all the model and has the best f1 score when predicting closed specifically. It is also the most balanced model with consistent performance on both precision and recall. Therefore, I proceed using this class of model.

Random forests have various hyperparameters; tuning them is necessary to gain an optimal model. To find a good set of hyperparameters, I use grid search with five fold cross validation. For number of estimaters, I select from amongst 100,200,300, and 400. For maximum depth, I select from amongst none, 10, and 20. For minimum samples per leaf, I select from 1,3,8,12,15. In this case grid seach looks at each combination of hyperparameters and chooses the model with the best average F1 score under five fold cross validation. This methodology avoids chosing a model which overfits the data with unrealistic hyperparameters. I end up with a max depth of 0, a minimum number of samples per leaf of 8, and a number of estimators of 400. The hyperparameter selection ends up trading some precision for a significant increase recall. This provides an F1 score on the closed class of 0.342, which is a modestly good prediction given the heavily biased sample. I provide the confusion matrix of this model below. Notably, it has more errors where it predicts a business will be closed but it isn't. This is acceptable in this case, since it catches more actually closed businesses while still maintaining a reasonable balance. 


<iframe src="assets/fairnessFalseNegative.html" width="600" height="450" frameborder="0"></iframe>


Although this model does a modestly good job at predicting whether a business is closed, it is still far from perfect and isn't able to identify closed businesses the majority of the time. This is not primarily due to the model type or hyperparameter selection, but the relatively weak features which do not have an ideal amount of correlation with the response variable. This is a significant limitation of the model which can only be accounted for by collecting new data or doing more feature engineering. Additionally, the dataset is very unbalanced with a relatively low percentage of locations in the dataset being closed. This makes the binary task difficult, since a business being operational is far more common. These results are also limited to business which have significant information; this model may not generalize to locations which do not have information on their state.

## Fairness Analysis 

Another imporatant consideration for this model is whether it's predictions perform fairly across different groups. For example, if the model performs worse for rural locations than urban ones, this represents an unfairly designed model. To examine model fairness I look at accuracy parity comparing between the combinations of Rural, urban, and semi-urban. To test the significance, I use permutation testing under the null that the difference . 

I find that the difference between Urban and Rural accuracy is not significant at the 5% level. Futhermore the difference between rural and semi-urban is also not significant. This suggests when comparing those two groups, model accuracy is fair. However when compairing semi-urban to urban, there is a significant difference which suggests that the model is unfair and has a lower accuracy for Urban than semi-urban. 

Notably, the observed difference in accuracies between Rural and Urban is bigger than the difference between semi-urban and urban. However, because there are so few rural observations, the permutation test is underpowered. 

Furthermore, there is also a clear difference in the false negative rate, rural and semiurban observations have much higher false negative rates as shown in the figure below. 

<iframe src="assets/fairnessFalseNegative.html" width="600" height="450" frameborder="0"></iframe>

There seems to be significant evidence that in some ways the model isn't completely fair in it's predictions. 

