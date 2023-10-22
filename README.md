# Final-Project-Statistical-Modelling-with-Python

## Project/Goals
To shape bike rental station data from Vancouver and surrounding points of interest.  Build a dataset and statistical model (linear regression using the statsmodel module in Python) to reveal the relationship between number of bikes for rental in each bike rental station, and the characteristics of the points of interest nearby.

Subgoals:
- Use Python to query 3 different APIs and parse JSON data from their payloads into Pandas dataframes.
- Use Pandas to manipulate, shape, clean and pivot dataframes for analysis.
- Use matplotlib and seaborn modules to render some visualizations to help with EDA and ideas for the linear regression/statistical model building.
- Use sqlite3 module to create local file DB and store dataframe data into the DB.
- Use statsmodel module to create multivariate linear regression models, using model evaluation methods to determine which features to include, and evaluate/interpret the results.

## Process
### Preparation: Analysis/Examination of APIs
1. Consider several factors which inform the design of the dictionary structure holding the JSON payload and structure of receiving Pandas dataframes:
- Number of API calls available in "free tier" of the 3 APIs
- Authorization types (Yelp had a Bearer Token, Foursquare had just an API key)
- If API call limit reset daily or monthly.  When in the day does the daily limit reset?
    - Foursquare:  40,000 free calls.  Resets monthly at 1st of each month.
    - Yelp:  500 free calls daily.  Resets @ Midnight UTC timezone.
    - Citybikes: Free API, no call limits.  Number of free bikes changes as bikes are rented, so these values are highly subject to time of day/season when API call is made.  Decided not to use that, but use "slots" instead.
2. Examine Yelp and Foursquare documentation to decide upon which endpoints are most useful.
3. Examine Yelp and Foursquare documentation to understand query parameters, and determine where this parameters will come from.
4. Examine Yelp and Foursquare payload to understand the type of information returned, and the format of same.
### API Calls
1. Call citybikes API (2 endpoints) to get information about bike stations in Vancouver.
2. Call Yelp API (Businesses Search endpoint) to get information about points of interest within 1 km of each bike station in Vancouver.
- 3 API calls per bike station (3 different groupings of categories).
3. Call Foursquare API (Place Search endpoint) to get information about points of interest within 1 km of each bike station in Vancouver.
- 6 API calls per bike station (6 different groupings of categories).
4. Parse each set of data into separate dataframes (3 in total).
### Initial EDA for Data Cleaning
1. Examine returned data and execute data cleaning and validation on returned data on all 3 dataframes.
### Data Transformation in Preparation for Model Building
1. Join various dataframes to obtain dataframes for further investigation, graphing, and ultimately model building.
2. Pivot and "collapse" dataframes to obtain aggregated data that is in a format that lends itself to model building/linear regression.
### Write dataframe to SQLite Database
1. Create a local SQLite DB and table to hold dataframe information.
2. Insert rows from the dataframe into the SQLite table.
3. Confirm the inserts were complete.
### Build Statistical Model and Interpret Results
1. Use model evaluation methods to determine which features to include in the linear regression.
2. Create linear regression model with appropriate features.
3. Evaluate Adjusted R-squared, p-values of the model summary and interpret results.

## Results
1. The number of certain types of points of interest (from Foursquare), nor the minimum distance to the nearest point of interest, do not have a strong predictive impact on the number of rental slots in a given bike station.
- The two features that have an impact are:
    - Number of POIs with "outdoor" characteristics (Beach, Protected & Historic Site, Scenic Lookout), with coefficient of 0.6735 (p-value=9.162179e-07) within 1 km radius from bike station
    - Number of POIs with "snack" characteristics (Cafe, Coffee and Tea House), with coefficient of -0.0496 (p-value=0.015)

## Challenges 
The largest challenges were Time and API call limits.
1. Time:  Given more than 60 hours (3 days over a weekend) to complete this project, there were more investigations that would have been interesting (see "Future Goals" below).
2. API call limits
- API payload item count limits and no pagination ability from Yelp or Foursquare, not enough familiarity with GIS/GPS methods to calculate how to break 1 km radius circle into smaller quadrants to make successive API calls to get the complete dataset and no time to investigate whether Yelp or Foursquare had this feature already built in (Foursquare has query parameters denoting rectangular search boxes denoted by lat/long coordinates, which may have been leveragable with more time to investigate).
- Call limits forced a need to combine categories for the Yelp dataset, causing a number of these queries to max out at 50 returned results.
- Call limits on Yelp forced me to drop some categories altogether (example: "coffee" and "juicebars" categories needed to be dropped because for some sample bike stations, these were both returning at the max item limit of 50, which could have skewed the linear regression I was planning)
- Call limits were not as much of an issue with the Foursquare API (40,000 calls per month), but time constraints and presumed difficulty aggregating the resulting Pandas dataframe forced me to make a choice to combine certain categories together.

## Future Goals
Questions to ponder with more time:
1. Does the model and correlation change if citybikes were to provide the ability to track the company operating the bike station?  From their networks API, I know these five companies operate in Vancouver [Vancouver Bike Share Inc., CycleHop LLC, City of Vancouver, Shaw Communications Inc., Fifteen], but not which stations belong to which company.  It is possible, even likely, that the way these companies plan their bike stations and the number of slots available at each station, vary.  Yet, the dataset I was using presumed a uniform approach operated by one company.
2. If I run the linear regression with the Yelp dataset, which includes number of ratings and average rating, is there a stronger model and correlation/predictive capability resulting from the features in that dataset?
3. Given that the "number of slots" (number of bikes available for rent) is at heart an optimization problem, a better dataset to use to build a predictive model or to look at correlations, would be to collect the following information:
- utilization of bikes (e.g. number of bikes on rent / number of bikes available totally)
    - sampled regularly throughout the day
    - sampled regularly throughout the year - to take into account seasonal changes
- correlated with a more rich set of points of interest
    - split in more granular categories when API call limits are not a limitation
    - API calls adjusted so we have an accurate measure of number of places of certain characteristics (ie. calls adjusted so as not to 'max out' the payload return)
