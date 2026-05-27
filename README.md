
In this project I look at how competition and proximity to similar locations correlates with variables such as ratings and price. 

## Data Cleaning and Feature Engineering 



The primary features which I create are, the close locations, similar locations, and similar and close locations which I classify as a location's "competitors". Since not all locations are businesses, they may not be competitors in the traditional economic sense, however similar attractions such as two waterfalls may still be seen as competing with each other for visitors. First, I measure the distance between locations using haversine distance which calculates the distance between two locations on a sphere using their latitude and longitude, which are the location variables given in the dataset. I then measure "similarity" in locations using the cosine similarity between each location's name, description, and some elements of the miscellaneous information. Particularly I exclude miscellaneous information on accessibility and health and safety, since that information is shared across many unrelated information. Combining this information I create as set of locations that are both similar and close for each location. 

Other useful features about individual reviews is aggregated then merged into the location level dataset. I create features for review count, minimum rating, median rating, and the average rating amongst repeat reviewers who have more than five reviews. 



## Exploratory Data Analysis 



## Assessments of Missingness 

One important consideration for this dataset is the missingness of various data. Particularly, important variables like price and description are mostly missing. However 

## Differences in Number of Competitors and Average Rating by Price Category 

I test whether different price categories have different distributions of competitors and average ratings and generally find that these different price ranges are different. I run sets of two sample Kolmogorov-Smirnov tests for each set of price categories to compare whether their distributions are the same. I find that the distributions of average rating differ across price categories, particularly very cheap locations have are different from any other category. For example, when compared with the most expensive price category, the cumulative distribution function for average ratings of the least expensive category has a difference of 32 percentage points. 

<iframe
  src="assets/numCompetitorsCDF.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

****

<iframe
  src="assets/avgRatingsPriceCDF.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Ratings Regression Models


## Price Classification Models