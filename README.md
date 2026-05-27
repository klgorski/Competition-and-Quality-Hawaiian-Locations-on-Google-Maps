# Competition-and-Quality-Hawaiian-Locations-on-Google-Maps
I use google maps data to examine the effects of competition as measured by proximity to other similar businesses on price and average rating. This is a final project for Data Science 80 at businesses UC San Diego


In this project I look at how competition and proximity to similar locations correlates with variables such as ratings and price. 

## Data Cleaning and Feature Engineering 

The primary features which I create are, the close locations, similar locations, and similar and close locations which I classify as a location's "competitors". Since not all locations are businesses, they may not be competitors in the traditional economic sense, however similar attractions such as two waterfalls may still be seen as competing with each other for visitors. First, I measure the distance between locations using haversine distance which calculates the distance between two locations on a sphere using their latitude and longitude, which are the location variables given in the dataset. I then measure "similarity" in locations using the cosine similarity between each location's name, description, and some elements of the miscellaneous information. Particularly I exclude miscellaneous information on accessibility and health and safety, since that information is shared across many unrelated information. Combining this information I create as set of locations that are both similar and close for each location. 

## Exploratory Data Analysis 

## Differences in Number of Competitors and Average Rating by Price Category 


## Ratings Regression Models


## Price Classification Models