  

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

Looking at the relationship between average rating and price, bar and density countour plots are shown below. 

<iframe src="assets/ratingCompetitorsBar.html" width="600" height="450" frameborder="0"></iframe>

The bar plot of number of competitors by rating shows that for most of the ratings distribution number of competitors does not differ too much by median. In the top few bars of the ratings distribution there is a difference and reduction in median number of competitors. The may be seen as a counter intuitive result, since you would expect the quality to increase along with number of competitors. This may reflect that the result from the previous plots, since higher priced restaurants have fewer competitors and higher ratings. The bottom of the distribution also as few competitors which is an expected result. 

Further visualizing this relationship, a scatter plot of ratings by number of competitors within the same price range shows a slightly positive trend, although very it is very small. This may indicate there is a relatively small relationship between competition and quality in this dataset. 


<iframe src="assets/priceRangeCompetitorsRating.html" width="600" height="450" frameborder="0"></iframe>


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

## Differences in Number of Competitors and Average Rating by Price Category

  

I test whether different price categories have different distributions of competitors and average ratings and generally find that these different price ranges are different. I run sets of two sample Kolmogorov-Smirnov tests for each set of price categories to compare whether their distributions are the same. I find that the distributions of average rating differ across price categories, particularly very cheap locations have are different from any other category. For example, when compared with the most expensive price category, the cumulative distribution function for average ratings of the least expensive category has a difference of 32 percentage points.

  

<iframe src="assets/avgRatingsPriceCDF.html" width="600" height="450" frameborder="0"></iframe>


  

****

****

  

I also find deviation in between the distributions of the number of competitors. Again using two sample Kolmogorov-Smirnov tests I find that the distributions differ amongst price categories, particularly, the $ and \$$ distributions and the $ and \$\$$ differ significantly at the one percent significance level. Since the ECDF for $ locations is lower than the others, this would indicate that more of the $ EPDF's mass is concentrated amongst locations with many competitors.

It is important to note that this does not represent causality, given that price is correlated with omitted variables such as location in a city, since cities have more competition due to close proximity, this biases any result. 






  
  
<iframe src="assets/numCompetitorsCDF.html" width="600" height="450" frameborder="0"></iframe>

  

****

  
Further refining competition to close similar businesses within the same price range. This is important because for many businesses that are similar such as restaurants, based on their price they may not actually be direct competitors. For example, and expensive sit down restaurant should not be considered a direct competitor to a fast food restaurant. Therefore, I examine differences in distributions of number of competitors within the given price range. 

<iframe src="assets/numCompetitorsPriceRangeCDF.html" width="600" height="450" frameborder="0"></iframe>


The statistics from the Kolmogorov-Smirnov tests are summarized in the table below, only differences which are significant at the 1% level are reported:

| Average Rating | Number of Competitors | Number of Competitors within Price Range |
| -------------- | --------------------- | ---------------------------------------- |
| \$ vs \$\$ (0.131)    | \$ vs \$\$ (0.082)  | \$ vs \$\$ (0.057)   |
| \$ vs \$\$\$ (0.177)   | \$ vs \$\$\$ (0.147) | \$ vs \$\$\$ (0.466)  |
| \$ vs \$\$\$\$ (0.320)  |                      | \$ vs \$\$\$\$ (0.512) |
| \$\$ vs \$\$\$\$ (0.261) |                      | \$\$ vs \$\$\$ (0.479) |
| \$\$\$ vs \$\$\$\$ (0.201)|                      | \$\$ vs \$\$\$\$ (0.526)|


## Ratings Regression Models

  
  

## Price Classification Models
