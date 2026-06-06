## Introduction


I investigate how competition, location, and other features correlate with and predict whether a business closes. This is an important question for business owners; if you can predict whether a business will fail based on a set of features it can inform the business owner's optimal decisions. The setting for the data is Hawaiian Google maps data, which describes the metadata and information on a subset of reviews for locations. This is a relevant setting because much of the publicly available information on locations, such as their ratings, location, and number of competitors can be key predictors for the business's success. The dataset on location metadata has 21507 locations; relevant columns include the address, text information including name, description and miscellaneous information, latitude and longitude, average ratings and the number of reviews, the hours of the location, and the business's state. Address, latitude, longitude, name, and description are self explanatory. Miscellaneous information contains other information on the location such as if it does takeout, if it is wheelchair accessible, and more. Average rating is the average user reported rating on Google maps and the number of reviews is the total number of reviews for that location. The hours of the location contains information on which days of the week the location is open, and which hours it is open. The business's state contains information on whether the location is currently open, closed, or permanently closed. This provides the response I wish to predict in this project.




## Data Cleaning and Exploratory Data Analysis


**Data Cleaning**


Important features which I create are the close locations, similar locations, and similar and close locations which I classify as a location's "competitors". Since not all locations are businesses, they may not be competitors in the traditional economic sense, however similar attractions such as two waterfalls may still be seen as competing with each other for visitors. First, I measure the distance between locations using haversine distance which calculates the distance between two locations on a sphere using their latitude and longitude, which are the location variables given in the dataset. I then measure "similarity" in locations using the cosine similarity between each location's name, description, and some elements of the miscellaneous information. Particularly I exclude miscellaneous information on accessibility and health and safety, since that information is shared across many unrelated locations. Combining this information I create a set of locations that are both similar and close for each location. Finally, I filter for locations that are similar and within the same price category, although since many price values are missing this will not be the primary unit for analysis.


I also clean information like price and zipcode by removing values which should not be in the dataset such as ₩, non-English zip codes, and non-Hawaiian zipcode. I extracted zipcodes from address information using regular expressions. I then create a feature which characterizes whether a zip code is rural, semi-urban, or urban using the RUCA codes developed by the USDA. I characterize urban as 1-3, semi-urban as 4-7, and rural as 8-10.


Other potentially useful features about individual reviews are aggregated then merged into the location level dataset. I create features for review count, minimum rating, median rating, and the average rating amongst repeat reviewers who have more than five reviews. I construct a variable for the average response time of the business owner by taking the difference between the time the review was posted and the response time, then aggregating these differences. However, most of the analysis does not use these variables, since many locations in the metadata dataset do not have a match in the reviews dataset.


I create variables for the number of days the business is open and the average number of hours the business is open. I construct this information from the hours variable in the original metadata dataset. Finally, I construct an indicator for whether a business is closed, this will serve as the dependent variable in the analysis. The head of the dataset is shown below.


| gmap_id                               | name                        | address                                            |   description |   latitude |   longitude |
|:--------------------------------------|:----------------------------|:---------------------------------------------------|--------------:|-----------:|------------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  | Hale Pops                   | Hale Pops, 55-370 Kamehameha Hwy, Laie, HI 96762   |           nan |    21.6378 |    -157.921 |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | SMP - Single Marine Program | SMP - Single Marine Program, G St, Kailua, HI 9... |           nan |    21.4403 |    -157.754 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | 2 Cheesy Guys               | 2 Cheesy Guys, 1486 HI-30, Wailuku, HI 96793       |           nan |    20.853  |    -156.504 |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | Kraken Coffee Kahului       | Kraken Coffee Kahului, 520 Keolani Pl, Kahului,... |           nan |    20.8882 |    -156.451 |
| 0x7c006df045b01715:0xe945c308688e1a46 | Akasatana Ramen Kyoto       | Akasatana Ramen Kyoto, 1450 Ala Moana Blvd, Hon... |           nan |    21.2905 |    -157.844 |


| gmap_id                               | category              |   avg_rating |   num_of_reviews | price   | hours                                              |
|:--------------------------------------|:----------------------|-------------:|-----------------:|:--------|:---------------------------------------------------|
| 0x7c00456eecad3111:0x8217f9600c51f33  | ['Restaurant']        |          4.4 |               18 | nan     | [['Thursday', '11AM–8PM'], ['Friday', '11AM–8PM... |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | ['Recreation center'] |          4.1 |               18 | nan     | [['Thursday', '8AM–9PM'], ['Friday', '8AM–9PM']... |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | ['Food court']        |          5   |                6 | nan     | [['Thursday', 'Closed'], ['Friday', '11AM–6PM']... |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | ['Coffee shop']       |          4.8 |                8 | $       | [['Thursday', '6:30AM–7PM'], ['Friday', '6:30AM... |
| 0x7c006df045b01715:0xe945c308688e1a46 | ['Ramen restaurant']  |          5   |                1 | nan     | [['Thursday', '11AM–8:30PM'], ['Friday', '11AM–... |




| gmap_id                               | url                                                |   zipcode |
|:--------------------------------------|:---------------------------------------------------|----------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  | https://www.Google.com/maps/place//data=!4m2!3m... |     96762 |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | https://www.Google.com/maps/place//data=!4m2!3m... |     96734 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | https://www.Google.com/maps/place//data=!4m2!3m... |     96793 |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | https://www.Google.com/maps/place//data=!4m2!3m... |     96732 |
| 0x7c006df045b01715:0xe945c308688e1a46 | https://www.Google.com/maps/place//data=!4m2!3m... |     96814 |




| gmap_id                               | MISC                                               | state                   | relative_results                                   |
|:--------------------------------------|:---------------------------------------------------|:------------------------|:---------------------------------------------------|
| 0x7c00456eecad3111:0x8217f9600c51f33  | {'Service options': ['Outdoor seating', 'Takeou... | Closed ⋅ Opens 11AM     | ['0x7c00451360f80cf1:0x930291a38bab3132', '0x7c... |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | {'Accessibility': ['Wheelchair accessible entra... | Opens soon ⋅ 8AM        | ['0x7c00155df23846af:0xfbe051d208292028', '0x7c... |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | {'Service options': ['Curbside pickup', 'Takeou... | Closed ⋅ Opens 11AM Fri | nan                                                |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | {'Service options': ['Curbside pickup', 'No-con... | Open ⋅ Closes 7PM       | nan                                                |
| 0x7c006df045b01715:0xe945c308688e1a46 | {'Service options': ['Takeout', 'Dine-in', 'Del... | Closed ⋅ Opens 11AM     | ['0x7c006df018f6177d:0x9beb6db40fadcb2', '0x7c0... |


| gmap_id                               | closeLocations                                     |   numCloseLocations | textInfoCombined                                   |
|:--------------------------------------|:---------------------------------------------------|--------------------:|:---------------------------------------------------|
| 0x7c00456eecad3111:0x8217f9600c51f33  | [['0x7c0045691b9c528d:0x3e845bb85fb2e7d7' 0.456... |                  74 | Hale Pops  Outdoor seating Takeout Delivery Lun... |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | [['0x7c001539ceab03cb:0x5675966bbb2d9542' 1.168... |                  83 | SMP - Single Marine Program                        |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | [['0x7954d30aed2259cf:0x3032e1e2429eaa85' 1.763... |                  32 | 2 Cheesy Guys  Curbside pickup Takeout             |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | [['0x7954d2debd4d5355:0x9c17afdd76f81793' 1.971... |                 473 | Kraken Coffee Kahului  Curbside pickup No-conta... |
| 0x7c006df045b01715:0xe945c308688e1a46 | [['0x7c006de89f2d86e1:0x23d998532e9317a6' 1.120... |                2550 | Akasatana Ramen Kyoto  Takeout Dine-in Delivery... |


| gmap_id                               | similarLocations                                   | competitors                                        |   numCompetitors |
|:--------------------------------------|:---------------------------------------------------|:---------------------------------------------------|-----------------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  | ['0x7954d370921ff6bd:0x3193ba783e26d032', '0x7c... | ['0x7c00456924310067:0x38a12cfc44bec058', '0x7c... |               14 |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | ['0x7c07a977e68506dd:0xe786233fe66435a5', '0x7c... | []                                                 |                0 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | ['0x7954d370921ff6bd:0x3193ba783e26d032', '0x7c... | []                                                 |                0 |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | ['0x7c00456eecad3111:0x8217f9600c51f33', '0x795... | ['0x7954d2c01aed2bcf:0x7c49262729c9c421', '0x79... |               93 |
| 0x7c006df045b01715:0xe945c308688e1a46 | ['0x7c00456eecad3111:0x8217f9600c51f33', '0x795... | ['0x7c006dee35049c21:0xfc2e9fbd1dbed7e8', '0x7c... |              646 |


| gmap_id                               | CompetitorsPriceRange                              |   numCompetitorsPriceRange |   reviewCount |   minRating |
|:--------------------------------------|:---------------------------------------------------|---------------------------:|--------------:|------------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  | []                                                 |                          0 |           nan |         nan |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e | []                                                 |                          0 |            20 |           1 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 | []                                                 |                          0 |           nan |         nan |
| 0x7954d370921ff6bd:0x3193ba783e26d032 | ['0x7954d2c0e04c1e7f:0x41f080467ecda138', '0x79... |                         16 |           nan |         nan |
| 0x7c006df045b01715:0xe945c308688e1a46 | []                                                 |                          0 |           nan |         nan |


| gmap_id                               |   medianRating |   avgResponseTime |   avgRatingRepeatReviewer | ruralUrbanClass   |   daysOpen |
|:--------------------------------------|---------------:|------------------:|--------------------------:|:------------------|-----------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  |            nan |               nan |                       nan | Semi-Urban        |          6 |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e |              5 |               nan |                         4 | Urban             |          7 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 |            nan |               nan |                       nan | Urban             |          4 |
| 0x7954d370921ff6bd:0x3193ba783e26d032 |            nan |               nan |                       nan | Urban             |          7 |
| 0x7c006df045b01715:0xe945c308688e1a46 |            nan |               nan |                       nan | Urban             |          7 |


| gmap_id                               |   avgHoursOpen |   isClosed |
|:--------------------------------------|---------------:|-----------:|
| 0x7c00456eecad3111:0x8217f9600c51f33  |         9      |          0 |
| 0x7c00159b5b1b1d25:0x8d2d85d4a758290e |        11.2857 |          0 |
| 0x7954d376a8b12db3:0xa51dd57e1cc14ca9 |         6.25   |          0 |
| 0x7954d370921ff6bd:0x3193ba783e26d032 |        12.5    |          0 |
| 0x7c006df045b01715:0xe945c308688e1a46 |         9.5    |          0 |


**Univariate Analysis**


I plot the histograms for five variables of interest below.


<iframe src="assets/isClosed.html" width="600" height="450" frameborder="0"></iframe>


As would be expected by far the majority of the locations in the dataset are classified as operating (0), while only a few are closed (1). This is likely due to Google removing businesses which are closed for too long. An important consideration is that this is a very unbalanced distribution, which will make prediction difficult.
 
<iframe src="assets/avgRating.html" width="600" height="450" frameborder="0"></iframe>


The histogram of average ratings shows that on average ratings are relatively high, primarily centered around 4.5 stars with a spike at 5 stars. This could potentially be because many firms leave fake reviews or because people genuinely do rate locations highly. There is likely some form of survivorship bias at play; locations which do not perform well are not going to survive and then may not appear in the dataset.




<iframe src="assets/price.html" width="600" height="450" frameborder="0"></iframe>


The most common non-missing price category in the dataset is $$ followed by $. More expensive options are much less common. However, most values in the dataset are missing, which could potentially be problematic when analyzing interactions with price.


<iframe src="assets/numCloseLocations.html" width="600" height="450" frameborder="0"></iframe>


There are a significant number of locations with few close locations, however there are also many with a great deal of close locations. This is the expected result in a location like Hawaii, where there are many rural areas with few locations, but also urban areas such as Honolulu where there will be many locations in close proximity.


<iframe src="assets/numCompetitors.html" width="600" height="450" frameborder="0"></iframe>


Most firms have very few competitors or no competitors which is expected since in most cases there will be only a few similar businesses within a small radius. Notably, there are also a few locations with many competitors; assuming that this location doesn't actually have over a thousand competitors, this reflects the limitations of the cosine similarity methodology in correctly classifying which locations are actually similar to each other.


**Bivariate Analysis**


I begin by analyzing connections between whether a location is closed and characteristics such as price and average rating. Looking at the relationship between average rating and whether a business is closed or not, there does seem to be a relationship where as rating increases, the proportion of that rating that is closed decreases. For very small ratings, the proportion is very high, although this could be a result of the low number of observations at this small rating.


<iframe src="assets/isCloseAvgRating.html" width="600" height="450" frameborder="0"></iframe>


Looking at the relationship between price and whether a location is closed, the relationship does appear to be less significant. I plot the relationship below. However despite this there does seem to be more closed locations for \$\$\$ priced locations than the other categories. There may be a niche where a location is expensive, but not the very best so it closes.


<iframe src="assets/isClosePriceRange.html" width="600" height="450" frameborder="0"></iframe>


Now I consider secondary relationships, such as the price range and the number of competitors in the dataset. A bar plot showing the number of competitors by price range is shown below.
<iframe src="assets/competitorsbyPriceRange.html" width="600" height="450" frameborder="0"></iframe>


Across price categories, the number of competitors is somewhat similar, with a decreasing median and maximum number of competitors as price increases. The main shift occurs going from \$\$ to \$\$\$, with the two lower price categories and the two higher price categories generally being similar. This plot provides some evidence to support that lower number of competitors is correlated with a higher price.

<iframe src="assets/ratingCompetitorsBar.html" width="600" height="450" frameborder="0"></iframe>

The bar plot of number of competitors by rating shows that for most of the ratings distribution the number of competitors does not differ too much by median. In the top few bars of the ratings distribution there is a difference and reduction in median number of competitors. This may be seen as a counter intuitive result, since you would expect the quality to increase along with the number of competitors. This may reflect that higher priced restaurants have fewer competitors and higher ratings. The bottom of the distribution also has few competitors which is an expected result.


Further visualizing this relationship, a scatter plot of ratings by number of competitors within the same price range shows a slightly negative trend, although it is small. This may indicate there is a negative relationship between competition and quality in this dataset.


<iframe src="assets/priceRangeCompetitorsRating.html" width="600" height="450" frameborder="0"></iframe>


Looking at the relationship between average rating and price, the empirical cumulative distribution function is shown below. It shows that lower prices have more cumulative density at lower ratings, which is evidence that cheaper locations are worse rated.

<iframe src="assets/avgRatingsPriceCDF.html" width="600" height="450" frameborder="0"></iframe>




**Aggregates**




I also look at aggregates of various numerical variables by price categories. Below is an aggregate table of the averages for key variables, grouped by closure status.




|   isClosed |   avg_rating |   num_of_reviews |   numCloseLocations |   numCompetitors |   numCompetitorsPriceRange |   avgRatingRepeatReviewer |
|-----------:|-------------:|-----------------:|--------------------:|-----------------:|---------------------------:|--------------------------:|
|          0 |      4.35497 |         152.754  |             620.559 |          73.3215 |                    3.0586  |                   4.38421 |
|          1 |      4.19948 |          58.7924 |             925.763 |          64.7316 |                    2.81145 |                   4.24305 |


Looking at this table, it is apparent that the average rating and the number of reviews is lower for closed businesses. Closed businesses also have many more close locations, but fewer competitors. This may contradict the hypothesis that competition predicts closure, or it could be the result of an omitted variable.  




Looking at aggregates by price category also yields interesting results, below is an aggregate table of the averages of various numerical variables, grouped by price category.


| price   |   avg_rating |   num_of_reviews |   numCloseLocations |   numCompetitors |   numCompetitorsPriceRange |   avgRatingRepeatReviewer |
|:--------|-------------:|-----------------:|--------------------:|-----------------:|---------------------------:|--------------------------:|
| $       |      4.14963 |          253.112 |             709.056 |         150.522  |                   15.9227  |                   4.22675 |
| $$      |      4.22167 |          373.789 |             796.37  |         133.881  |                   18.8965  |                   4.29488 |
| $$$     |      4.28697 |          333.218 |             987.5   |         108.416  |                    1.92437 |                   4.3412  |
| \$\$\$\$    |      4.42479 |          410.299 |            1357.21  |         114.675  |                    1.2906  |                   4.50137 |
| missing |      4.37477 |          103.853 |             611.981 |          57.3987 |                    0       |                   4.41803 |






As expected, average rating increases on average as price increases. The number of reviews also generally increases, which could reflect that people are enthusiastic to write a review if they have spent lots of money, for example an upscale restaurant. The number of close locations also increases as price increases. This is likely because more expensive locations are going to be in cities, where there are many close locations. On the other hand, the number of competitors is generally decreasing as price increases. This aligns with the hypothesis that competition is negatively correlated with price.




Finally, I look at the correlation matrix of all numeric variables in the dataset, the table is shown below


|                          |    latitude |   longitude |   avg_rating |   num_of_reviews |   numCloseLocations |   numCompetitors |
|:-------------------------|------------:|------------:|-------------:|-----------------:|--------------------:|-----------------:|
| latitude                 |  1          | -0.799411   |  -0.0505116  |       0.0109798  |           0.162997  |       0.0105284  |
| longitude                | -0.799411   |  1          |   0.0478632  |      -0.00831672 |          -0.152572  |      -0.00496762 |
| avg_rating               | -0.0505116  |  0.0478632  |   1          |       0.022655   |          -0.102559  |      -0.00574929 |
| num_of_reviews           |  0.0109798  | -0.00831672 |   0.022655   |       1          |           0.0177304 |       0.0564894  |
| numCloseLocations        |  0.162997   | -0.152572   |  -0.102559   |       0.0177304  |           1         |       0.0417118  |
| numCompetitors           |  0.0105284  | -0.00496762 |  -0.00574929 |       0.0564894  |           0.0417118 |       1          |
| numCompetitorsPriceRange |  0.00988639 | -0.00761741 |  -0.0501929  |       0.121231   |           0.0280777 |       0.585198   |
| reviewCount              |  0.0091602  | -0.0099879  |   0.0162932  |       0.938328   |           0.0184497 |       0.0373647  |
| minRating                | -0.0309636  |  0.0354586  |   0.414606   |      -0.215394   |          -0.0749525 |      -0.0614253  |
| medianRating             | -0.085764   |  0.0973149  |   0.677001   |       0.00639691 |          -0.113109  |      -0.0176854  |
| avgResponseTime          |  0.0191386  | -0.0235953  |  -0.046816   |       0.101079   |           0.0182418 |       0.0409205  |
| avgRatingRepeatReviewer  | -0.0790339  |  0.0886897  |   0.869965   |       0.0263627  |          -0.139208  |      -0.0320886  |
| daysOpen                 |  0.0398374  | -0.0287897  |  -0.0256825  |       0.0836683  |           0.117039  |       0.186942   |
| avgHoursOpen             |  0.0292882  | -0.0343694  |  -0.135978   |       0.072827   |          -0.0495687 |      -0.016761   |
| isClosed                 |  0.0237229  | -0.0220442  |  -0.0736397  |      -0.060047   |           0.101524  |      -0.0160096  |


|                          |   numCompetitorsPriceRange |   reviewCount |   minRating |   medianRating |   avgResponseTime |   avgRatingRepeatReviewer |
|:-------------------------|---------------------------:|--------------:|------------:|---------------:|------------------:|--------------------------:|
| latitude                 |                 0.00988639 |    0.0091602  |  -0.0309636 |    -0.085764   |         0.0191386 |               -0.0790339  |
| longitude                |                -0.00761741 |   -0.0099879  |   0.0354586 |     0.0973149  |        -0.0235953 |                0.0886897  |
| avg_rating               |                -0.0501929  |    0.0162932  |   0.414606  |     0.677001   |        -0.046816  |                0.869965   |
| num_of_reviews           |                 0.121231   |    0.938328   |  -0.215394  |     0.00639691 |         0.101079  |                0.0263627  |
| numCloseLocations        |                 0.0280777  |    0.0184497  |  -0.0749525 |    -0.113109   |         0.0182418 |               -0.139208   |
| numCompetitors           |                 0.585198   |    0.0373647  |  -0.0614253 |    -0.0176854  |         0.0409205 |               -0.0320886  |
| numCompetitorsPriceRange |                 1          |    0.0958757  |  -0.138255  |    -0.057429   |         0.0546866 |               -0.0864127  |
| reviewCount              |                 0.0958757  |    1          |  -0.23389   |    -0.0303645  |         0.0972593 |               -0.00494624 |
| minRating                |                -0.138255   |   -0.23389    |   1         |     0.232642   |        -0.0927678 |                0.53435    |
| medianRating             |                -0.057429   |   -0.0303645  |   0.232642  |     1          |        -0.0359438 |                0.756847   |
| avgResponseTime          |                 0.0546866  |    0.0972593  |  -0.0927678 |    -0.0359438  |         1         |               -0.0476742  |
| avgRatingRepeatReviewer  |                -0.0864127  |   -0.00494624 |   0.53435   |     0.756847   |        -0.0476742 |                1          |
| daysOpen                 |                 0.142133   |    0.0551216  |  -0.13685   |    -0.0746035  |         0.0245861 |               -0.116869   |
| avgHoursOpen             |                 0.0258801  |    0.121998   |  -0.106077  |    -0.197849   |         0.0286633 |               -0.19272    |
| isClosed                 |                -0.00533659 |   -0.0767103  |   0.0109391 |    -0.099847   |        -0.0100969 |               -0.105669   |


|                          |   daysOpen |   avgHoursOpen |    isClosed |
|:-------------------------|-----------:|---------------:|------------:|
| latitude                 |  0.0398374 |      0.0292882 |  0.0237229  |
| longitude                | -0.0287897 |     -0.0343694 | -0.0220442  |
| avg_rating               | -0.0256825 |     -0.135978  | -0.0736397  |
| num_of_reviews           |  0.0836683 |      0.072827  | -0.060047   |
| numCloseLocations        |  0.117039  |     -0.0495687 |  0.101524   |
| numCompetitors           |  0.186942  |     -0.016761  | -0.0160096  |
| numCompetitorsPriceRange |  0.142133  |      0.0258801 | -0.00533659 |
| reviewCount              |  0.0551216 |      0.121998  | -0.0767103  |
| minRating                | -0.13685   |     -0.106077  |  0.0109391  |
| medianRating             | -0.0746035 |     -0.197849  | -0.099847   |
| avgResponseTime          |  0.0245861 |      0.0286633 | -0.0100969  |
| avgRatingRepeatReviewer  | -0.116869  |     -0.19272   | -0.105669   |
| daysOpen                 |  1         |      0.392848  |  0.152135   |
| avgHoursOpen             |  0.392848  |      1         | -0.0591715  |
| isClosed                 |  0.152135  |     -0.0591715 |  1          |




Some of the significant correlations from this matrix are closure status and the average rating by repeat reviewers, which are negatively correlated. The number of reviews is also positively correlated with the average response time. Another notable result is that ratings are  negatively correlated with the number of close locations, competitors, and competitors within the same price range.



## Assessment of Missingness


 


One important consideration for this dataset is the missingness of various data. Particularly, important variables like price and description are mostly missing. For price, it is likely that it is not missing at random (NMAR). Locations which have no cost or a very low cost are more likely to be missing than highly expensive restaurants. Particularly, reviewers from low cost locations are less likely to note the price in their review or post information such as the menu which Google uses to categorize the price. Therefore it is reasonable to say that price's missingness depends on the price itself.




An example of data which is likely to be MAR is the text of individual user reviews. Intuitively, more extreme ratings are more likely to have text associated with them, either positive or negative; text is missing depending on the user's rating. To test this, I use permutation testing with two sample Kolmogorov-Smirnov tests comparing the sample for which text is missing and the sample for which text is present. The result of this permutation test rejects the null that missingness does not depend on rating, which would indicate text missingness is missing at random dependent on at least rating. I plot the results of the permutation test below.


<iframe src="assets/textNAFull.html" width="600" height="450" frameborder="0"></iframe>


 One consideration is that this test may have too much power and lead to detecting trivially small differences in the distributions. To address this potential issue, I take a completely random subsample of the dataset and compute the same permutation test on it; the result still rejects the null. I plot the results from this test below.


 <iframe src="assets/textNASubset.html" width="600" height="450" frameborder="0"></iframe>




I also find that missingness does not depend on the time the review was sent. This is expected because time should not have a significant effect on whether a person writes text in the review. Again, I perform the permutation testing procedure on the reduced sample, since when performing it on the full sample the test may have too much power. The results are plotted below.


<iframe src="assets/textNATime.html" width="600" height="450" frameborder="0"></iframe>


One caveat is that reducing the sample in such a way is highly susceptible to which rows get picked. Although this is necessary to reduce the power when running this permutation test, these results should be interpreted with caution.






## Hypothesis Testing






**Differences in Rate of Closure by Price Category and Average Rating**




I conduct hypothesis testing to examine whether rate of closure differs by price category and whether the average rating is the same across closed locations and open locations.


First, I test whether the difference in the rate of closure is the same across price categories. To do this, I compare each pair of price categories with a difference in means to test with the null hypothesis that the difference between the means is zero and the alternative hypothesis that the difference is not equal to zero. 


The results align with what was seen in the exploratory data analysis, particularly that the difference is not too large except for the difference between one $ and three \$\$\$ which is significant at the 5% level with a p-value of 0.04.


I also test whether the mean average rating of closed business is the same as operating business. To do this, again I use a difference in means t test with the null that the difference in means is zero versus the alternative hypothesis that the difference is not equal to zero. 


This test strongly rejects the null hypothesis at the 1% level with a p-value of 3.76e-27. This is evidence that the average rating of operating locations is higher than the average rating of closed locations; the observed difference is 0.155. This aligns with what would be expected, closed locations having worse ratings on average than operating locations which indicates that quality may have an impact on location closure. As further evidence of this difference, I plot the empirical probability density function for each category below; closed locations' density is located towards lower ratings than operating locations' density.


<iframe src="assets/avgRatingsClosurePDF.html" width="600" height="450" frameborder="0"></iframe>

Difference in means is the correct test statistic for this, because we do not need to detect differences in what the distributions look like, just the differences in the means between different groups. A non-parametric test like Kolmogorov-Smirnov may also have less power for this application since it is non-parametric and requires fewer assumptions. 


## Framing a Prediction Problem


**Closure Classification**


An important consideration for business owners is the risk of closure for their business. Using an information set to predict whether a location will close may be useful for a current or prospective business owner when making decisions. Therefore, I build a binary classification model which predicts whether a location is operating versus permanently or temporarily closed. As an evaluation metric, I will primarily use an F-1 score. This is the best metric because of how it balances between precision and recall. Both are important for this problem because you wish to both avoid a closure while also avoiding being overly cautious and missing opportunities. I also choose it over accuracy because the dataset is heavily unbalanced; since so many more businesses are operating than closed, a model could have good accuracy by only predicting that the location is open. I calculate F1-score performance on a test set of the data.




## Baseline Model


As a baseline model for prediction I use logistic regression with price and number of competitors. I chose to not include variables such as the number of days the location is open and its average hours, since naturally closed businesses will have no days open. The information that the location has no days open would not be available at time of prediction, so I do not include it. I choose to one hot encode price category even though it is ordinal because much of the price information is missing; an ordinal encoding would not work since 'missing' does not fit well into the ordinal scheme. I also simply standardize the number of competitors which is a numerical variable. There is one nominal variable (price) and one discrete quantitative variable (number of competitors). As stated before, I am encoding price as nominal in this model. I choose logistic regression as a baseline because it is a simple and interpretable model which does not require hyperparameter tuning. The result of this model is a macro f1 score of 0.539, notably dragged down by the f1 score of 0.213 when the model predicts a location is closed. The confusion matrix is shown below.


|                |   Predict Operating |   Predict Closed |
|:---------------|--------------------:|-----------------:|
| True Operating |                2096 |              392 |
| True Closed    |                 258 |               88 |


This model performs decently, but is by no means good. About $\frac15$ predictions of closed are correct, meaning it is very bad at characterizing when a location is actually closed. To improve predictions, in the next section I consider adding in more features and different models which may improve predictions, particularly the f1 score for closed.


## Final model


When testing models, I add a variety of features from the dataset into these models. I use one hot encoding of categorical variables such as zip code and rural urban status. These one hot encodings seek to capture variation by location and how urban a zipcode is. For numerical variables, I transform them into degree two polynomial features, which are useful for capturing any nonlinear relationships. I add numerical variables such as the number of reviews, average rating, number of close locations, and number of competitors within price range. I also add quantile features for average rating and number of competitors. This quantile features seeks to identify variation in these variables by quantile since there may be variation for very small for very small quantiles and very large quantiles of this data. For example, many business have no competitors, so it makes sense to group this together in a quantile and use this effect. I add these features because they should intuitively be factors which affect a business closing. It would be expected that there will be a higher rate of closure for locations in a city or a very low average rating. Adding these features improves the model because they capture variation which was not captured simply by price and the number of competitors such as geographic variation and the consumer's feedback. Some of these variables suffer from being near collinear, for example the number of competitors and the number of competitors within price range. Depending on which class of model is chosen, the number of variables may be reduced to account for these multicollinearity issues. For models such as K neighbors classification, multicollinearity is a much larger issue than for models such as a random forest.


I now consider several classification models, such as logistic regression, K neighbors classification, decision trees, and random forest classification on this larger set of features. These models vary significantly in their complexity, interpretability, and performance. Comparing the models, KNN classification performs very poorly, having an F-1 score of 0.071 when predicting closure, mainly due to a very low recall. This means the model heavily underpredicts the business closing, which is particularly bad for this application because it will lead to overconfidence about whether or not the location will succeed.


Logistic regression and decision tree classification perform similarly to each other; although logistic regression seems to do slightly better. One notable aspect of their performance is that their precision is low while the recall is higher for predictions of closed. This would indicate that these models are predicting the business is closed often, but many of these predictions are incorrect. Since I am also interested in reducing the rate of false negatives, these models are not optimal for the task.


Finally, the random forest model is the best performing on macro average F1 score across all the models and has the best f1 score when predicting closed specifically. It is also the most balanced model with relatively consistent performance on both precision and recall. Therefore, I proceed using this class of models.


Random forests have various hyperparameters; tuning them is necessary to gain an optimal model. To find a good set of hyperparameters, I use grid search with five fold cross validation. For the number of estimators, I select from amongst 100,200,300, and 400. For maximum depth, I select from amongst none, 10, and 20. For minimum samples per leaf, I select from 1, 3, 8, 12, 15, 20, and 25. In this case grid search looks at each combination of hyperparameters and chooses the model with the best F1 score averaged across five fold cross validation. This methodology avoids choosing a model which overfits the data with unrealistic hyperparameters. I end up with a max depth of 20, a minimum number of samples per leaf of 8, and a number of estimators of 300. The hyperparameter selection ends up trading some precision for an increase in recall. This provides an F1 score on the closed class of 0.303, which is a modestly good prediction given the heavily biased sample. I provide the confusion matrix of this model below. Notably, it has more errors where it predicts a business will be closed but it isn't. This is acceptable in this case, since it catches more actually closed businesses while still maintaining a reasonable balance. This is an improvement over the baseline model, which had very poor precision and recall when predicting closure. It also maintains a very similar f1 score when predicting a business is operating.  




|                |   Predict Operating |   Predict Closed |
|:---------------|--------------------:|-----------------:|
| True Operating |                2129 |              359 |
| True Closed    |                 220 |              126 |




Although this model does a modestly good job at predicting whether a business is closed, it is still far from perfect and isn't able to identify closed businesses the majority of the time. This is not primarily due to the model type or hyperparameter selection, but the relatively weak features which do not have an ideal amount of correlation with the response variable. This is a significant limitation of the model which can only be accounted for by collecting new data or doing more feature engineering. Additionally, the dataset is very unbalanced with a relatively low percentage of locations in the dataset being closed. This makes the binary task difficult, since a business being operational is far more common. These results are also limited to businesses which have significant information; this model may not generalize to locations which do not have information on their state.


## Fairness Analysis


Another important consideration for this model is whether its predictions perform fairly across different groups. For example, if the model performs worse for rural locations than urban ones, this represents an unfairly designed model. To examine model fairness I look at accuracy parity comparing between the combinations of Rural, urban, and semi-urban. To test the significance, I use permutation testing under the null that the absolute difference in prediction accuracy between groups is zero and the alternative hypothesis that the absolute difference is greater than zero.


I find that the difference between Urban and Rural accuracy is significant at the 1% level with an observed p-value of 0.004. This suggests that the model does not have the same prediction accuracy for urban and rural locations, meaning the model is not fair in this way. When examining the difference between rural and semi-urban locations, I fail to reject the null hypothesis at the 1% level with an observed p-value of 0.043. Therefore it seems that urban locations have significantly worse prediction accuracy compared to rural locations, while semi-urban locations do not. The results from the permutation test between rural and urban is shown below, notably the result is very significant.


<iframe src="assets/fairnessRuralUrban.html" width="600" height="450" frameborder="0"></iframe>


I also test whether there is a significant difference in prediction accuracy between semi-Urban and Urban. I fail to reject the null hypothesis at the 1% level with a p-value of 0.047. This suggests that there is not a significant difference in prediction accuracy between urban and semi-urban locations and that the model is fair between these two categories. The results of the permutation test are shown below.


<iframe src="assets/fairnessUrbanSemi.html" width="600" height="450" frameborder="0"></iframe>


Furthermore, there is also a visible difference in the false negative rate, rural and semiurban observations have much higher false negative rates as shown in the figure below. 


<iframe src="assets/fairnessFalseNegative.html" width="600" height="450" frameborder="0"></iframe>


There seems to be significant evidence that in some ways the model isn't completely fair in its predictions between urban and rural locations. 


## Conclusions

In this project I have examined location closure in Hawaii using a variety of statistical analyses. I have confirmed many hypothesized results, such as closed locations having a lower average rating than open businesses. Finally, I built a prediction model which is moderately good at predicting closure, but is somewhat limited because of the unbalanced sample and a lack of strong features. 
