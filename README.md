# Predict_Dengue_Cases

## Overview
***
- **CDC wants to understand the leading factors in determining whether a person would take the sesoanal flu vaccine so that they could focus on the right strategies for their public efforts and vaccination campaigns to educate the public, raise awareness and maximize vaccine intake.**

- They also want to know the likelihood to receive the seasonal flu vaccine for specific demographic groups and have feedback about whether their efforts are successfull. 

- **GOAL**: is build a classifier to predict seasonal flu vaccination status using information they shared about their backgrounds, opinions, and health behaviors. My main purpose was to make predictions as accurately as possible while balancing between sensitivity and specificity.

## Business and Data Understanding
***
* The data was obtained from the **National 2009 H1N1 Flu Survey** provided at [DrivenData](https://www.drivendata.org/competitions/66/flu-shot-learning/). 

* This phone survey contained dfata from 26707 particiants, and asked people whether they had received H1N1 and seasonal flu vaccines, in conjunction with information they shared about their lives, opinions, and behaviors. 

* In this project I will be focusing on `seasonal flu` only and information regarding individuals' opinions about the H1N1 vaccine were excluded from the analyses. The relevant variables/features included in the dataset are:

**Target Feature**: 
* `seasonal_vaccine` - Whether respondent received seasonal flu vaccine or not.

**Predictive Features**:

* `behavioral_antiviral_meds` - Has taken antiviral medications. (binary)
* `behavioral_avoidance` - Has avoided close contact with others with flu-like symptoms. (binary)
* `behavioral_face_mask` - Has bought a face mask. (binary)
* `behavioral_wash_hands` - Has frequently washed hands or used hand sanitizer. (binary)
* `behavioral_large_gatherings` - Has reduced time at large gatherings. (binary)
* `behavioral_outside_home` - Has reduced contact with people outside of own household. (binary)
* `behavioral_touch_face` - Has avoided touching eyes, nose, or mouth. (binary)
* `doctor_recc_seasonal` - Seasonal flu vaccine was recommended by doctor. (binary)
* `chronic_med_condition` - Has any of the following chronic medical conditions: asthma or an other lung condition, diabetes, a heart condition, a kidney condition, sickle cell anemia or other anemia, a neurological or neuromuscular condition, a liver condition, or a weakened immune system caused by a chronic illness or by medicines taken for a chronic illness. (binary)
* `child_under_6_months` - Has regular close contact with a child under the age of six months. (binary)
* `health_worker` - Is a healthcare worker. (binary)
* `health_insurance` - Has health insurance. (binary)
* `opinion_seas_vacc_effective` - Respondent's opinion about seasonal flu vaccine effectiveness. 1 = Not at all effective; 2 = Not very effective; 3 = Don't know; 4 = Somewhat effective; 5 = Very effective.
* `opinion_seas_risk` - Respondent's opinion about risk of getting sick with seasonal flu without vaccine. 1 = Very Low; 2 = Somewhat low; 3 = Don't know; 4 = Somewhat high; 5 = Very high.
* `opinion_seas_sick_from_vacc` - Respondent's worry of getting sick from taking seasonal flu vaccine. 1 = Not at all worried; 2 = Not very worried; 3 = Don't know; 4 = Somewhat worried; 5 = Very worried. 
* `age_group` - Age group of respondent.
* `education` - Self-reported education level.
* `race` - Race of respondent.
* `sex` - Sex of respondent.
* `income_poverty` - Household annual income of respondent with respect to 2008 Census poverty thresholds.
* `marital_status` - Marital status of respondent.
* `rent_or_own` - Housing situation of respondent.
* `employment_status` - Employment status of respondent.
* `hhs_geo_region` - Respondent's residence using a 10-region geographic classification defined by the U.S. Dept. of Health and Human Services. Values are represented as short random character strings.
* `census_msa` - Respondent's residence within metropolitan statistical areas (MSA) as defined by the U.S. Census.
* `household_adults` - Number of other adults in household, top-coded to 3.
* `household_children` - Number of children in household, top-coded to 3.
* `employment_industry` - Type of industry respondent is employed in. Values are represented as short random character strings.
* `employment_occupation` - Type of occupation of respondent. Values are represented as short random character strings.


## Data Cleaning:
***

Null replacement using interpolation and predictive modeling:
* Null values of features except the four ndvi fatures were imputed with **interpolation** since the missing data points are scarse.
* Null values for the four ndvi fatures were imputed using **k-Nearest Neighbors - KNN** since there were bigger chunks of missing values.

Interpolation:

* Below graph shows the data matrix with null values before null replacement, and the missing ndvi index values before and after applying KNN:

![DataMatrix_BeforeAfterCleaning](https://user-images.githubusercontent.com/61121277/199284666-c8f26292-406a-43c3-a6dd-36a780364683.png)

## Preprocessing
***
#### Binary (yes/no) Columns:
* Many of the variables in float type are actually binary (yes/no).
* Given that the proportion of null values are not too high for these variables, the null values will be replaced with the **most frequent**. 

    ##### `health_insurance`:
    * A **predictive model** will be used to impute the missing values and then these values will be merged into the dataset. 

#### Numerical Columns:
* Some of variables in float type are **ordinal** (some sense of ordering to its categories), so they will be treated as **numerical**. 
* The null values will be replaced with the **Median**. 
* Standard Scaling for the numerical variables only (since binary and categorical variables are already encoded as 0 and 1)

#### Categorical Columns:
* The variables in object type are **nominal** (no intrinsic ordering to its categories), so they will be treated as **categorical**. 
* The null values will be replaced with a **contant('missing')** creating its own level before one-hot encoding these variables. 

    ##### `income_poverty`:
    * A **predictive model** will be used to impute the missing values and then these values will be merged into the dataset. 

## Modeling
***
1. The data was split into training and test sets.
2. The data was pre-processed. 
3. Several types of classifiers were built, tuned (using GridSearchCV to test combinations of hyperparameters) and validated:

    - Logistic Regression
    - Decision Tree
    - Random Forest
    - XGradient Boosted
    - Stacking Classifier (using above models)

4. Scoring Metric: Roc_Auc was used afor tuning hyperparameters and evaluating model performance, because:
 
* We care equally about true positives and true negatives, and the roc curve gives a desirable balance between **sensitivity/recall (maximizing True positive Rate)** and and **1 - specificity  (minimizing False Positive Rate - Probability that a true negative will test positive)**

* We want to utilizes **"probabilities"** of class prediction. Based on this, we’re able to more precisely evaluate and compare the models. 

## Evaluation
***

![Compare_RocCurve_Models](https://user-images.githubusercontent.com/61121277/199509396-629dd8df-bbb7-4715-85b8-e7626bf4c289.png)

* **XGBoost** gives the best performance on both train (tells if model is confident in it’s learning) and test datasets (tells if the results are negeralizable to an unknown dataset). It gives Roc_Auc values of 88% (on Train) and 87% (on test), which is considered **GOOD**.

![XGB_Results1](https://user-images.githubusercontent.com/61121277/199540455-5f1c29eb-cead-421f-bae3-2dad7efe6304.png)
![XGB_Results2](https://user-images.githubusercontent.com/61121277/199540456-eebd5573-1875-47c2-8501-edcb9ef0633b.png)

* Results from the best fitting model on the test set are:
    - **accuracy score of 79%**, 
    - **sensitivity/recall score of 79%** 
    - **specificity score of 82%** 

![XGBoost_FeatureImportance](https://user-images.githubusercontent.com/61121277/199292805-566c278c-e1e8-4a4a-b643-044569ce0812.png)

* The most important 6 features in predicting whether a person would get the seasonal vacccine are:

- `doctor_recc_seasonal`
- `healht_insurance`
- `opinion_seas_vacc_effective`  
- `opinion_seas_risk`
- `age_group_65+ Years`
- `health_worker` 

## Conclusion
***

![MostImportantFeatures_Probability_BarPlot](https://user-images.githubusercontent.com/61121277/199609041-e03dd4f4-2340-4512-a684-608f90204cc6.png)

You are more likely to get the vaccine if you:

- have a doctor recommending the vaccine
- have health insurance
- think the vaccine is effective
- think you can get sick from flu
- are older, especially +65
- are a health worker
   
## Recommendations
***
* Educate the physicians on the importance of vaccination. Make sure they recommend it to their patients.
* Consider offering universal health coverage. Inform the public that flu vaccine is covered by insurance. 
* Inform people about the effectiveness and safety of the vaccine, and their risk of falling ill and developing complications if not vaccinated. 
* Keep focusing on older age groups, because they are at more risk of developing flu-related complications. Also target younger people since their vaccination rates are much lower.

## Next Steps
***
* Encrypted employment industry, employment occupation, and geographical region info, hard to make any specific suggestions based on these features. 
* Results on health insurance are not very reliable due to 40% of the data being null and being encoded using predictive modeling. More emphasis needs to be given to this variable next time the survey is conducted even it is a significant feature in predicting vaccine outcome. 
* More recent data needs to be collected after the Covid-19 pandemic since the pandemic might have altered people’s attitude towards flu vaccine as well. 

## Repository Structure
    .
    ├── images 
    ├── data 
    ├── Notebook.ipynb     
    ├── Notebook.pdf 
    ├── Presentation.pdf                                             
    └── README.md   

## Contact Info:
* Email: erdemiraysu@gmail.com
* GitHub: [@erdemiraysu](https://github.com/erdemiraysu/)
* [LinkedIn](https://www.linkedin.com/in/aysuerdemir)
