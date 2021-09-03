# Predicting Eviction Rates  

**Author:** Jack Mannix

## Overview 

To address the skyrocketing rate of evictions in the United States, the focus of this project is to predict the eviction rate of a  census tract based upon its socioeconomic demographics and history of evictions.


## Background 

<p>A person is evicted when they are expelled from a property by the property’s landlord. Landlords can evict tenants for a variety of reasons, including unpaid rent, or damaged properties; however, landlords can perform ‘no-fault evictions,’ when a tenant has not violated their leasing contract. Evictions should not only be observed as an issue of housing, but as an issue of public health as well. A family that experiences an eviction is much more likely to experience a host of mental health issues, job loss, and homelessness. As the COVID-19 pandemic persists and the national eviction moratorium has ended, many Americans are finding that they are at a higher risk of eviction as contracting COVID-19 as a direct result (Eviction Lab). </p> 

<p>The ability to foresee a census tract’s eviction rate is crucial for the allocation of appropriate resources. 90% of tenants facing evictions do not receive legal representation for their legal case, and are much more likely to be evicted than the 10% with representation. Knowing this, the ability to foresee a census tract’s eviction rate is crucial for allocating free legal resources in areas that face the greatest risks of eviction (source).</p>


## Methodology 
### Data Aquisition
 <p>The data used for this project was obtained from the Eviction Lab and the American Community Survey (ACS) from census.gov. The Eviction Lab is a team of researchers based at Princeton University  who have created a national database from over 10,000,000 eviction records from 2000 to 2016. </p> 
 
### Data Preparation
<p>The Eviction Lab provides their data at various levels of granularity (state, county, census tract, and block group). This project utilizes the data at the census tract level as it provides a good balance of granularity, unlike the state and county levels, without having large amounts of missing data, like the block group level.</p>

<p>While the data has been collected by census tract from 2000 - 2016, the majority of census tracts are missing records from different years, making time series analysis difficult. To avoid subsetting a very small dataset with consistency in its recorded years, the data is aggregated by census tract, so that each continuous feature value is the mean of all available annual records from a particular census tract. Mean was selected as the method of aggregation over median in order to produce a model that can make sufficient predictions despite the existence of outliers.</p>

<p>Typically, a model built to predict the eviction rate of a census tract for the year 2022 would not be able to utilize the total number of evictions or eviction filings as features as these records as these numbers would not be available until the end of the year. However, by taking the mean of every available year from a particular census tract, these features can be used to make predictions for upcoming years. This aggregation method proves to be very advantageous, as these features are identified as highly important throughout the iterations of modeling.</p>

### Data Cleaning
Dropped Data
1. Census tracts with null eviction rates after aggregation 
2. Census tracts with a population of 0
3. Census tracts from states with counts < 400
    - NM, HI, IN, CA, RI, DE, ME, KS, WY, OK, LA, VT, MO, TN
4. Census tracts with eviction rates > 90%
    - Eviction Rate =  # Evictions / # renter occupied properties
    - Based upon the formulate for eviction rate it is possible for census tracts to have extremely high eviction rates if there is a single eviction and a low number of renter occupied properties
    
## Data Exploration
### Eviction Rates Amongst Ethnicity Majorities

<p>Knowing the housing market is one that is built and run upon racial lines (study), it is important to analyze how race might impact eviction rates in this dataset.</p>

Key Findings
1. Groups with higher eviction rates when possessing a majority 
    - African American 
    - Hispanic
2. Groups with higher eviction rates when possessing the minority
    - White
    - Asian
    - Native American
3. All t-tests conducted on the eviction rates of a particular groups majority and minority census tracts were significant (<0.05)
4. There are no census tracts in which Native Hawaiin / Pacific Islander, Other, or Multiple have a majority 

### Eviction Rates Amongst Gender Majorities
<p>The same analysis above is also conducted for gender.</p> 

Key Findings
1. Census tracts with women holding a majority have a median eviction rate that is 0.21% higher than the median eviction rate of census tracts with men holding a majority 
3. All t-test results are significant (<0.05)

## Modeling
### Models Used
1. Linear Regression
2. RandomForest
3. XGBoost
4. Neural Networks (MLN)

### Feature Engineering
<p>The curse of dimensionality is a limiting factor for trying to engineer geographical features. Treating each census tract or their respective counties as individual geographic features would result in thousands of features. To create features utilizing geographical groupings, the nine US Census Regions (map) are used to group census tracts to their respective regions.</p>

### Feature Selection
- **(Target) Eviction Rate** = #Evictions/#Renter Occupied Households
- **Eviction Filing Rate** = #Filed Evictions /#Renter Occupied Households
- **Renter Occupied Households** = # Renter Occupied Households
- **Pct Renter Occupied Households** = # Renter Occupied Households / Population

<p>Based upon their formulas, the inclusion of all of the features above would result in the model using these numbers to perfectly calculate the target variable (eviction rate). Because of this, different feature combinations were used during separate iterations. Significant improvement in R-Squared and RMSE scores resulted from using Eviction Filing Rate and  (#)Evictions.</p>

### Outlier Removal
<p>Initial exploration reveals many outliers throughout the dataset, so IsolationForest is implemented with an iterative approach to determine how many and which outliers are removed. After several iterations of IsolationForest with varying contamination parameters, a contamination of 0.05%, removing 18 outliers from the training dataset, yields an improvement in both R-Squared and RMSE values</p>.

## Final Model
<p>XGBoost (trained on IsolationForest data)</p>
- R-Squared
    - Train = 0.9467
    - Test = 0.9435
- RMSE 
    - 0.8258
    - 0.8191


<p>The final model is able to account for 94% of the variance in observed census tract eviction rates based upon the available features, and is able to predict an eviction rate within 0.81 percentage points.</p>















 
 


    
    
## Data and Methods 

All models were trained and tested on data collected from 47,304 wells   
- Data provided by [TAARIFA](https://taarifa.org/) and [DrivenData](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/page/23/)
- Data collected 2011 - 2013
- Models used
    1. Logistic Regression
    2. DecisionTree
    3. RandomForest
    5. XGBoost
    
    
### Assumptions
The original data provided three target variables
1. Functional
2. Non-functional
3. Funcional-Needs Repair


Due to a major class imbalance against 'Functional-Needs Repair', preliminary models produced severly low recall for this variable

![img](./images/class_imbalance.png)


To address this imbalance, 'Non-functional' and 'Functional-Needs Repair' were combined due to the following considerations:
    1. A well that is 'Functional-Needs Repair' is not fully functional, and will decline to 'Nonfunctional' if not addressed
    2. Additional issues of misclassification arise:
        - Wells that are 'Non-functinonal' that are classified as 'Functional-Needs Repair' would be less of a priority to fix 
        - Wells that are 'Functional-Needs Repair' that are classified as 'Functional' miss the chance for being repaired before becoming 'Non-functional' 
    3. Additional misclassifications cannot be risked when working with human rights such as access to clean water 
    

---------------------------------------------------------------

# Process 

## Data Cleaning 

This dataset was mainly comprised of 30 object features, and only 9 features with continuous values 
- Object features had many unique values, causing extreme multidimensionality (>1,000 features) after one-hot-encoding 
- For object features not dropped, unique values with counts <1,000 were aggregated into an 'Other' value 

**Columns Dropped (17):**
- id
- scheme_name
- subvillage
- waterpoint_type_group
- quantity_group 
- water_quality 
- payment 
- funder
- installer
- date_recorded
- year_recorded
- source_type
- extraction_type
- recorded_by
- ward
- lga
- management


**Feature Engineering:**
- month_recorded
- well_age 

**Null Values:**
- 17,542 null values dropped
- 48,630 entries remain 

**Filler Data:**
- # Nonsensical 'longitude' - dropped
- '0' in object features combined into 'Other' 

**Outliers:**
- >1000 entries collected in 2002 and 2004 - dropped 



## EDA


![img](./images/tanz.png)

No glaring concentrations of a particular well classification to discern from geographical data

- Regions with greater concentrations of "Non-functioning" wells than "Functioning"
    - Mara, Mtwara, Tabora, Rukwa, Mwanza, Kigoma, Lindi, Dodoma, Mbeya, Singida, Dar es Salaam 

<p float="left">
  <img src="./images/region.png" width="350" />
  <img src="./images/region_map.png" width="350" /> 
</p>  

- Wells over the age of 24 tended to have more "Non-functioning" wells than "Functioning"

![img](./images/age.png) 


- Nearly all wells with quantity = 'dry' are "Non-functioning' 
- Wells with quantity = 'insufficient' have high "Non-functioning' counts (while still lower than 'Functioning')

![img](./images/quantity.png)

- January, April, and July experienced higher reports of "Non-functioning" than "Functioning" 

![img](./images/month_rain.png)

- Wells with quality_group = 'salty' have a higher 'Non-functioning' count than 'Functioning' 

![img](./images/quality.png) 



## Modeling 

**Models Used (In Order):**
1. Logistic Regression 
2. DecisionTree
3. RandomForest
5. XGBoost (FINAL) 

### Final Model 
The final XGBoost model was achieved after three iterations of optimization via GridSearch

**Baseline:**

<p float="left">
  <img src="./images/base_cr.png" width="350" />
  <img src="./images/base_cm.png" width="350" /> 
</p> 

- This baseline was the first baseline to not completely overfit the training data

**Final Model:**

<p float="left">
  <img src="./images/final_cr.png" width="350" />
  <img src="./images/final_cm.png" width="350" /> 
</p> 

- Considering that False Negatives are much more harmful than False Positives, the difference between the two is acceptable
    - This model produced the lowest amount of False Negatives (859)
- Compared to the previous best model (RandomForest), recall and f1 scores have improved by 1%
    - the improvement of f1 scores along with recall indicates that the model is not completely sacrificing precision to achieve higher recall 

**Hyperparameters:**
1. sub_sample = 0.5
2. scale_pos_weight = 2
3. learning_rate = 0.2
4. max_depth = 8


---------------------------------------------------------------

# Going Forward

## Reccomendations 
With the predictive capabilities the model and the insight of the initial EDA, NONPROFIT should consider the following while implementing the use of the model: 

**Heightened Monitoring**
- Regions in which Non-functional wells outnumber functional should be monitored with heightened attention:
    - Mara
    - Mtwara
    - Tabora
    - Rukwa
    - Kigoma
    - Lindi
    - Dodoma
    - Mbeya
    - Dar es Salaam
- Monitoring efforts should increase during months in which Non-functional wells outnumber functional  
    - January, April, and July
    
**Alerts**
- Send alerts encouraging more frequent checks on wells age 25 and up 
- Send alerts when a well's water level (quantity) is reported as 'insufficient' or 'dry' 
- Send alerts when a well's water quality is reported as 'salty' 

## Further Investigation 

- Conduct PCA during initial data cleaning in order to reduce dimensionality in a more calculated manner
- Advocate for additional  metrics to be used to describe “Water Quality”
- Advocate for higher recording rates during dry season


---------------------------------------------------------------

# Further Information

For further information regarding the analysis, please view the [Jupyter Notebook](final_notebook.ipynb) or review the findings presentation [HERE](phase_3_presentation.pdf) 









