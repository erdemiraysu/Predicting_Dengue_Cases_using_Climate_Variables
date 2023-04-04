# Predict_Dengue_Outbreaks
![image](https://user-images.githubusercontent.com/61121277/229658964-31698d73-ddd9-4cf3-bf54-c82fa941c301.png)

## Overview:
***

**Dengue fever** is a mosquito-borne disease that occurs in tropical and sub-tropical parts of the world. In mild cases, symptoms are similar to the flu: fever, rash, and muscle and joint pain. In severe cases, dengue fever can cause severe bleeding, low blood pressure, and even death.

Because it is carried by mosquitoes, the transmission dynamics of dengue are related to climate variables such as temperature and precipitation; however the relationship to climate is known to be complex.  The way the disease spreads and causes endemics has significant public health implications worldwide.

* CDC is interested in predicting local epidemics of dengue fever so that they can take necessary precautions and efforts before the next spike. They want to know if we can predict the number of dengue fever cases reported each week in San Juan, Puerto Rico.

* My goal is to build several machine learning models to forecast the upcoming weekly dengue cases as accurately as possible.  

## Business and Data Understanding
***

The data was obtained from [DrivenData](https://www.drivendata.org/competitions/44/dengai-predicting-disease-spread/). The data set included weekly dengue case counts along with environmental data collected by various U.S. Federal Government agencies—from the Centers for Disease Control and Prevention to the National Oceanic and Atmospheric Administration in the U.S. Department of Commerce. 

* The full dataset included cases from year 1990 to 2008. The data from 2008-2013 included only features without case counts. 
* 
* In this project I will be focusing on data on `Puerto Rico` only. The relevant variables/features included in the dataset are:

**Target Feature**: 
* `total_cases` - Weekly total dengue cases.

**Predictive Features**:

***Date Indicators***:
* `week_start_date` - Date given in yyyy-mm-dd format.

***NOAA's GHCN daily climate data weather station measurements***:
* `station_max_temp_c` - Maximum temperature
* `station_min_temp_c` - Minimum temperature
* `station_avg_temp_c` - Average temperature
* `station_precip_mm` - Total precipitation
* `station_diur_temp_rng_c` - Diurnal temperature range

***PERSIANN satellite precipitation measurements (0.25x0.25 degree scale)***:
* `precipitation_amt_mm` - Total precipitation

***NOAA's NCEP Climate Forecast System Reanalysis measurements (0.5x0.5 degree scale)***:
* `reanalysis_sat_precip_amt_mm` - Total precipitation
* `reanalysis_dew_point_temp_k` - Mean dew point temperature
* `reanalysis_air_temp_k` - Mean air temperature
* `reanalysis_relative_humidity_percent` - Mean relative humidity
* `reanalysis_specific_humidity_g_per_kg` - Mean specific humidity
* `reanalysis_precip_amt_kg_per_m2` - Total precipitation
* `reanalysis_max_air_temp_k` - Maximum air temperature
* `reanalysis_min_air_temp_k` - Minimum air temperature
* `reanalysis_avg_temp_k` - Average air temperature
* `reanalysis_tdtr_k` - Diurnal temperature range

***Satellite vegetation -greenness - Normalized difference vegetation index (NDVI) - NOAA's CDR Normalized Difference Vegetation Index (0.5x0.5 degree scale) measurements***: 
* `ndvi_se` - Pixel southeast of city centroid
* `ndvi_sw` - Pixel southeast of city centroid
* `ndvi_ne` - Pixel southeast of city centroid
* `ndvi_nw` - Pixel southeast of city centroid



## Preprocessing: 
***

### Null Replacement:

* Null values for the climate features - except the four ndvi fatures - were imputed with **interpolation** since the missing data points are scarse.
* Null values for the four ndvi fatures were imputed using **k-Nearest Neighbors - KNN** since there were bigger chunks of missing values.

Below graph shows the data matrix with null values before null replacement, and the missing ndvi index values before and after applying KNN:

![missingno_original](https://user-images.githubusercontent.com/61121277/229639553-b237297d-c7f2-469e-84b6-6da2f58f4748.png)

![KNN_ndvi](https://user-images.githubusercontent.com/61121277/229639430-fd8b933f-9dd3-4d12-abfe-448ba78d10c5.png)



### Feature Engineering:

* Create `month` and `seasons`: Created new variables representing the month and seasons. 
* Create `average_ndvi` and its **categorical** version: Created a new feature representing the average NDVI values using the four different locations. Then created a categorical version of average_ndvi to represent watery, soily, sparce_grassy areas.
* Create **shifts** and **rolled averages** for the main climate variables:
Research seems to indicate that past sustained heat, precipitation or humidity impacts dengue cases more profoundly than the climate situation right at the time of cases. 
  - Shifted the variables by 2 weeks to account for the mosquito to reach adulthood and the incubation period of the virus until someone tests positive.
  - Create rolled means with a range of lags to see the variable with the highest correlation. The lag with the highest corralation was kept in the final dataset.
  
![Screen Shot 2023-04-03 at 7 57 50 PM](https://user-images.githubusercontent.com/61121277/229659091-27e4ea6b-577a-4aa2-9ad0-3a5ad1aebe6a.png)


## Modeling
***

The dengue data with labels (1990-2008) was split into training and test sets using the first 80% of the data as train, and the final 20% for test. Additional dataset aside with climate features only (without the knowledge of true case counts)(2008-2013) was used to forecast upcoming case counts  for the best performing models.

1. The data was split into training and test sets.
2. The data was pre-processed. 
3. Several versions of machine learning models were built, tuned and validated to be able to forecast the time series data:

  - **Negative Binomial Regression** (multiple regression used for count data following the negative binomial). This method was chosen specifically because total_cases could be described by a negative binomial distribution with a population variance that is much larger than the population mean. 

  - **Sarimax** (Seasonal Autoregressive Integrated Moving Average Exogenous model)- a generalization of an autoregressive moving average (ARMA) model which supports time series data with a seasonal component.

  - **XGBoost (Extreme Gradient Boosting) Regression** Gradient-boosted decision tree algorithm used for regression predictive modeling.

  - **LSTM (long short-term memory network)** A variety of recurrent neural networks (RNNs) that are capable of learning long-term dependencies, especially in sequence prediction problems.

## Evaluation
***

4. Scoring Metric: **Mean Absolute Error** was used afor tuning hyperparameters and evaluating model performance. MAE is a popular metric to use as the error value is easily interpreted. This is because the value is on the same scale as the target you are predicting for.

**XGBoost** give the best performance on both train (tells if model is confident in it’s learning) and test datasets (tells if the results are negeralizable to an unknown dataset). 

Results from the best fitting model are:
    - **Mean Absolute Error score of 14.6** for train, 
    - **Mean Absolute Error score of 17.7** for test.
 
Below graph shows the model fitted on train and forecasted on test in relation to actual observed values: 
![XGB_Predict](https://user-images.githubusercontent.com/61121277/229638111-9e64b1a8-85c5-4559-8037-39e43be4111f.png)

Below graph shows the model re-fitted on the whole dataset and forecasted into the future in relation to actual observed values from the whole dataset: 
![XGB_Forecast](https://user-images.githubusercontent.com/61121277/229638217-027f53b9-3d90-404c-bee1-d414eb2d6bad.png)

Below graph shows feature importance from the model fitted on the whole dataset:
![XGB_FeatureImportance](https://user-images.githubusercontent.com/61121277/229638499-49fdca54-b306-495e-8f8a-b48d215f56f9.png)

* The most important features in predicting whether a person would get the seasonal vacccine are:
-  Cumulative humidity
-  Cumulative Maximum temperature
-  Cumulative Minimum temperature
-  Cumulative Vegetation Index representing soil


## Recommendations:
***
* 
* 



## Limitations, Improvements, Next Steps
***
* 
* 
* More recent data needs to be collected after the Covid-19 pandemic since the pandemic might have altered people’s attitude towards flu vaccine as well

## Reproducibility:

## Repository Structure
    .
    ├── images 
    ├── data 
    ├── notebook_preprocessing.ipynb  
    ├── notebook_modeling.ipynb  
    ├── notebook_preprocessing.pdf 
    ├── notebook_modeling.pdf 
    ├── presentation.pdf 
    ├── env.txt
    └── README.md   

## Contact Info:
* Email: erdemiraysu@gmail.com
* GitHub: [@erdemiraysu](https://github.com/erdemiraysu/)
* [LinkedIn](https://www.linkedin.com/in/aysuerdemir)
