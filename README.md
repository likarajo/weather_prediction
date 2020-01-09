# Weather Prediction

Predicting temperature

## Data

API [Powered by Dark Sky](https://darksky.net/poweredby/)

The **Forecast Request** returns the current weather conditions, a minute-by-minute forecast for the next hour (where available), an hour-by-hour forecast for the next 48 hours, and a day-by-day forecast for the next week.

Documentation: https://darksky.net/dev/docs#forecast-request

Forecast Request: `https://api.darksky.net/forecast/[key]/[latitude],[longitude]?<parameters>`

The API is used to obtain the current weather conditions along with the day-by-day forecast for the next week.

Sample Request:
```
https://api.darksky.net/forecast/[key]/22.5726,88.3639?exclude=[currently,minutely,hourly,alerts,flags]&units=auto
```

Sample Response:
```
{ 
    "latitude":22.5726,
    "longitude":88.3639,
    "timezone":"Asia/Kolkata",
    "daily":{ 
        "summary":"No precipitation throughout the week.",
        "icon":"clear-day",
        "data":[
            { 
                "time":1579113000,
                "summary":"Clear throughout the day.",
                "icon":"clear-day",
                "sunriseTime":1579135800,
                "sunsetTime":1579175040,
                "moonPhase":0.72,
                "precipIntensity":0.0031,
                "precipIntensityMax":0.0074,
                "precipIntensityMaxTime":1579174800,
                "precipProbability":0.04,
                "precipType":"rain",
                "temperatureHigh":27.59,
                "temperatureHighTime":1579163700,
                "temperatureLow":15.31,
                "temperatureLowTime":1579218000,
                "apparentTemperatureHigh":27.31,
                "apparentTemperatureHighTime":1579163700,
                "apparentTemperatureLow":15.58,
                "apparentTemperatureLowTime":1579218000,
                "dewPoint":11.8,
                "humidity":0.58,
                "pressure":1014.8,
                "windSpeed":2.21,
                "windGust":5.92,
                "windGustTime":1579142700,
                "windBearing":6,
                "cloudCover":0,
                "uvIndex":7,
                "uvIndexTime":1579155300,
                "visibility":16.093,
                "ozone":246.8,
                "temperatureMin":15.27,
                "temperatureMinTime":1579131780,
                "temperatureMax":27.59,
                "temperatureMaxTime":1579163700,
                "apparentTemperatureMin":15.54,
                "apparentTemperatureMinTime":1579131780,
                "apparentTemperatureMax":27.31,
                "apparentTemperatureMaxTime":1579163700
            }
        ]
    },
    "offset":5.5
}
```

---

## Dependencies

* Datetime
* Requests
* Pandas

`pip install -r requirements.txt`

---

## Data Collection

* Specify *API Key*, *Latitude*, *Longitude*, and *parameters* in the **Forecast** API.
* Specify the features that are needed to be parsed from the responses returned from the API. 
  * The features are the keys present in the ***daily -> data*** portion of the JSON response.
* Use the features to define a *namedtuple* that will be used to organize the data.

Data is collected and saved in ***data/year-mon-date_hour_min.csv*** using the script in [weather_data_collection.ipynb](weather_data_collection.ipynb)

---

## Intuition

Machine Learning experiments often have a few characteristics that are *oxymoronic* or self-contradictory, as well as highly influential self-explanatory variables and patterns that arise out of having almost a naive or at least very open and minimal presuppositions about the data.

Therefore, it is required to have the knowledge-based intuition to look for potentially useful features and patterns as well as unforeseen idiosyncrasies in an unbiased manner for a successful analytics project.

In this regards, we have selected a number of features of the weather data, but many of these might be uninformative in predicting weather depending on the model being used. So we need to rigorously investigate the data

---

## Feature Engineering

* Develop an **algorithm to find out the N previous days measurement of a feature**  
  * For each day (row) and for a given feature (column), find the value for that feature N days prior.  
  * For each value of N, make a new column for that feature representing the Nth prior day's measurement.
* Use the algorithm to find out the **3 previous days measurement** of each feature.

```
Features obtained:

date
summary
cloudCover
temperatureHigh
temperatureLow
humidity
precipProbability
precipType
dewPoint
pressure
windSpeed
windGust
visibility
uvIndex
ozone
cloudCover_-1
cloudCover_-2
cloudCover_-3
temperatureHigh_-1
temperatureHigh_-2
temperatureHigh_-3
temperatureLow_-1
temperatureLow_-2
temperatureLow_-3
humidity_-1
humidity_-2
humidity_-3
precipProbability_-1
precipProbability_-2
precipProbability_-3
precipType_-1
precipType_-2
precipType_-3
dewPoint_-1
dewPoint_-2
dewPoint_-3
pressure_-1
pressure_-2
pressure_-3
windSpeed_-1
windSpeed_-2
windSpeed_-3
windGust_-1
windGust_-2
windGust_-3
visibility_-1
visibility_-2
visibility_-3
uvIndex_-1
uvIndex_-2
uvIndex_-3
ozone_-1
ozone_-2
ozone_-3
```

---

## Data Preprocessing

Identify unnecessary data, missing values, consistency of data types, outliers and handle them.

* Drop unnecessary columns.

* Convert **categorical columns to numeric**, if any.

* Check features' statistics to **find outliers**.
  * Find the inter quartile range (IQR)
    * Difference between third and first quartiles
  * Check for the outliers
    * Those which are 3 IQRs below first quartile
    * Those which are 3 IQRs above third quartile
  * Get the features which contain outliers.

```
10 features have outliers

cloudCover
cloudCover_-1
cloudCover_-2
cloudCover_-3
precipProbability
precipProbability_-1
precipProbability_-2
precipProbability_-3
uvIndex
uvIndex_-1
```

* Analyzing and handling the outliers
  * Distribution of cloudCover: It si natural to have clear skies on some days.
  * Distribution of precipProbability: Since the dry days (ie, no precipitation) are much more frequent, it is sensible to see outliers here.
  * Similar is the situation for uvIndex where some days have higher measurement and others have lower.

* Check for **missing data**
  * The features contain relatively few missing (null / NaN) values, only the ones introduced due to the N prior days values.

* Handle missing data
  * Fill missing values with the mean column values

---

## Data Analysis

* Find the relationship between the dependent variable and each independent variable.
  * [Pearson Correlation Coefficient](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient): A measurement of the amount of linear correlation between equal length arrays which outputs a value ranging -1 to 1.  
    * 0 to 1 represent increasingly positive correlation, i.e. strong linear relationship.
    * -1 to 0 represent inverse, or negative, correlation, i.e. weak linear relationship.
```
0.8 - 1.0	Very Strong
0.6 - 0.8	Strong
0.4 - 0.6	Moderate
0.2 - 0.4	Weak
0.0 - 0.2	Very Weak
```

## Keep relevant features

* The features that have correlation values less than the absolute value of 0.6 are not to be used as predictors.

* The columns with the min and max temperature of the day are not useful for the model as the target 'meanTemp' is simply the mean value of these twofeatures.
  * 'temperatureLow' and 'temperatureLow are removed.

## Split data into Training and Test sets

* Test size = 20%

## Create and Train Model

* Linear Regression

```
LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None, normalize=False)
```

## Evaluate the Model

* Explained Variance: 95.63 %
  * The model is able to explain 95.63% of the variance observed in the outcome variable.
* Mean Absolute Error: 0.47 units
  * On average the predicted value is about 0.47 units off.
* Median Absolute Error: 0.47 units
  * Half of the time the predicted value is off by about 0.47 units.
