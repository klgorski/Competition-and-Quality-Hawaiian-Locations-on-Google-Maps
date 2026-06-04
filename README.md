  

## Data Cleaning and Feature Engineering

  
  
  

The primary features which I create are, the close locations, similar locations, and similar and close locations which I classify as a location's "competitors". Since not all locations are businesses, they may not be competitors in the traditional economic sense, however similar attractions such as two waterfalls may still be seen as competing with each other for visitors. First, I measure the distance between locations using haversine distance which calculates the distance between two locations on a sphere using their latitude and longitude, which are the location variables given in the dataset. I then measure "similarity" in locations using the cosine similarity between each location's name, description, and some elements of the miscellaneous information. Particularly I exclude miscellaneous information on accessibility and health and safety, since that information is shared across many unrelated information. Combining this information I create as set of locations that are both similar and close for each location. Finally, I filter for locations that are similar and within the same price category, although since many price values are missing this will not be the primary unit for analysis. 

I also clean information like price and zipcode by removing values which should not be in the dataset such as ₩, non-english zip codes, and non-Hawian zipcode. I extracted zipcodes from address information using regular expressions. 

Other useful features about individual reviews is aggregated then merged into the location level dataset. I create features for review count, minimum rating, median rating, and the average rating amongst repeat reviewers who have more than five reviews. I construct a variable for the average response time of the business owner by taking the difference between the time the review was posted and the response time, then aggregating these differences. 



  
  
  

## Exploratory Data Analysis

**Univariate Analysis**

I plot the histograms of the four main variables of interest below. 
  
<iframe src="assets/avgRating.html" width="600" height="450" frameborder="0"></iframe>

The histogram of average ratings shows that on average ratings are relatively high, primarily centered around 4.5 stars with a spike at 5 stars. This could potentially be because many firms leave fake reviews or because people genuinely do rate locations highly. There is likely some form of adverse selection at play; locations which do not perform well are not going to survive and then may not appear in the dataset. 


<iframe src="assets/price.html" width="600" height="450" frameborder="0"></iframe>

The most common non-missing price category in the dataset is $$ followed by $. More expensive options are much less common. However, most values in the dataset are missing, which could potentially be problematic when analying interactions with price. 

<iframe src="assets/numCloseLocations.html" width="600" height="450" frameborder="0"></iframe>

There are a significant number of locations with few close locations, however there are also many with a great deal of close locations. This is the expected result in a location like Hawaii, where there are many rural areas with few locations, but also urban areas such as Honolulu where there will be many locations in close proximity. 

<iframe src="assets/numCompetitors.html" width="600" height="450" frameborder="0"></iframe>

Most firms have very few competitors or no competitors which is expected since in most cases there will be only a few similar businesses within a small radius. Notably, there are also a few locations with many competitors; assuming that this location doesn't actually have over a thousand competitors, this reflects the limitations of the cosine similarity methodology in correctly classifying which locations are actually similar to each other. 

**Bivariate Analysis**

I begin by analyzing connections between price range and the number of competitors in the dataset. A bar plot showing the number of competitors by price range is shown below. 
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

  

One important consideration for this dataset is the missingness of various data. Particularly, important variables like price and description are mostly missing. Notably, for price it is likely that it is not missing at random, locations which have no cost are more likely to be missing than highly expensive restaurants. Even presuming this is not the case, price is missing at random dependent on average rating, with locations around the average rating on average being less missing. This can be seen in the plot below 
<iframe src="assets/priceNAavgRating.html" width="600" height="450" frameborder="0"></iframe>

Furthermore, a permutation test using Kolmogorov-Smirnov tests provides strong evidence that the data would be MAR. 

An example of data which is likely to be MAR is the text of individual user reviews. This is dependent on the user's rating. I use a similar testing procudure to show that this data is missing at random dependent on the user's rating. Looking at the plot of missing values based on rating it is apparant that reviews with a rating of one have a significantly lower number of missing values for the text of the review. Intuitively, this is because customers with a strong negative opinion are more likely write in their review.

<iframe src="assets/permTestTextRating.html" width="600" height="450" frameborder="0"></iframe>

The distribution of test statistics for the permutation test of missingness by rating is shown below. The plot shows that the simulated test statistics are all very far from the observed test statistic; there clearly is evidence missingness is dependent on rating. 

<iframe src="assets/textNAPermTest.html" width="600" height="450" frameborder="0"></iframe>

I also test whether variables such as the location's hours and miscelaneous information are missing at random or missing completely at random. These variables seem unlikely to be not missing at random since the actual values for this information would not influence the probability of missingness. I test whether these variables are missing dependent on average rating. The results from the permutation tests are shown below 

<iframe src="assets/hoursNAPermTest.html" width="600" height="450" frameborder="0"></iframe>
<iframe src="assets/miscNAPermTest.html" width="600" height="450" frameborder="0"></iframe>

Average rating seems to be a main determinant of whether many variables are missing. This is intuitively reasonable because locations with higher rating may be more likely to update their information or have customers who update it for them. Other variables such as owners response time to reviews are likely missing dependent on the average rating as well, given that owners with poor ratings are also less likely to engage with reviewers. 



## Hypothesis Testing

**Differences in Number of Competitors and Average Rating by Price Category**

I test whether different price categories have different distributions of competitors and average ratings and generally find that these different price ranges are different. I run sets of two sample Kolmogorov-Smirnov tests for each set of price categories to compare whether their distributions are the same. I find that the distributions of average rating differ across price categories, particularly very cheap locations have are different from any other category. For example, when compared with the most expensive price category, the cumulative distribution function for average ratings of the least expensive category has a difference of 32 percentage points.

  

<iframe src="assets/avgRatingsPriceCDF.html" width="600" height="450" frameborder="0"></iframe>



I also find deviation in between the distributions of the number of competitors. Again using two sample Kolmogorov-Smirnov tests I find that the distributions differ amongst price categories, particularly, the $ and \$$ distributions and the $ and \$\$$ differ significantly at the one percent significance level. Since the ECDF for $ locations is lower than the others, this would indicate that more of the $ EPDF's mass is concentrated amongst locations with many competitors.

It is important to note that this does not represent causality, given that price is correlated with omitted variables such as location in a city, since cities have more competition due to close proximity, this biases any result. 


  
<iframe src="assets/numCompetitorsCDF.html" width="600" height="450" frameborder="0"></iframe>



  
Further refining competition to close similar businesses within the same price range. This is important because for many businesses that are similar such as restaurants, based on their price they may not actually be direct competitors. For example, and expensive sit down restaurant should not be considered a direct competitor to a fast food restaurant. Therefore, I examine differences in distributions of number of competitors within the given price range. These results are limited, since as stated before price is potentially not missing at random. 

<iframe src="assets/numCompetitorsPriceRangeCDF.html" width="600" height="450" frameborder="0"></iframe>


The statistics from the Kolmogorov-Smirnov tests are summarized in the table below, only differences which are significant at the 1% level are reported:

| Average Rating | Number of Competitors | Number of Competitors within Price Range |
| -------------- | --------------------- | ---------------------------------------- |
| \$ vs \$\$ (0.131)    | \$ vs \$\$ (0.082)  | \$ vs \$\$ (0.057)   |
| \$ vs \$\$\$ (0.177)   | \$ vs \$\$\$ (0.147) | \$ vs \$\$\$ (0.466)  |
| \$ vs \$\$\$\$ (0.320)  |                      | \$ vs \$\$\$\$ (0.512) |
| \$\$ vs \$\$\$\$ (0.261) |                      | \$\$ vs \$\$\$ (0.479) |
| \$\$\$ vs \$\$\$\$ (0.201)|                      | \$\$ vs \$\$\$\$ (0.526)|


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

Random forests have various hyperparameters; tuning them is necessary to gain an optimal model. To find a good set of hyperparameters, I use grid search with five fold cross validation. For number of estimaters, I select from amongst 100,200,300, and 400. For maximum depth, I select from amongst none, 10, and 20. For minimum samples per leaf, I select from 1,3,8,12,15. In this case grid seach looks at each combination of hyperparameters and chooses the model with the best average F1 score under five fold cross validation. This methodology avoids chosing a model which overfits the data with unrealistic hyperparameters. I end up with a max depth of 0, a minimum number of samples per leaf of 8, and a number of estimators of 400. This provides an F1 score on the closed class of 0.342


Although this model does a modestly good job at predicting whether a business is closed, it is still far from perfect and isn't able to identify closed businesses the majority of the time. This is not primarily due to the model type or hyperparameter selection, but the relatively weak features which do not have an ideal amount of correlation with the response variable. This is a significant limitation of the model which can only be accounted for by collecting new data or doing more feature engineering. Additionally, the dataset is very unbalanced with a relatively low percentage of locations in the dataset being closed. This makes the binary task difficult, since a business being operational is far more common. These results are also limited to business which have significant information; this model may not generalize to locations which do not have information on their state.

## Fairness Analysis 

