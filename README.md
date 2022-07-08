# :star2: Data Mining :star2: #
## :ambulance: Mining the Diabetes Mellitus Database :ambulance: ##

### :one: Summary of features ###

First, I load the dataset into JupyterNotebook environment and examine its first 5
rows:

![image](https://user-images.githubusercontent.com/100490285/178081917-871d623f-787e-4c66-a17b-6a289ade6c3b.png)

There are a total of 88 features (including the target variable) in this dataset, and some of them have missing values. Due to the difference in statistcal value, numerical and categorical columns are separated based on their data types. After that, their corresponding data dictionaries are generated to give an overall view on the dataset. I also decided to discard the variable encounter_id as it is the unique ID of each patient, and readmission_status as it has only 1 value: 0.

### Categorical data ###

The summary statistics of categorical columns are as below:

![image](https://user-images.githubusercontent.com/100490285/178081964-2f1c9fec-36f1-467b-ae18-62626280a5a1.png)

In terms of gender, male patients are dominant with almost 43 thousand observations. The ethnicity field has around 1000 missing values, with Caucasian being the most common class. As for icu_type, Med-Surg ICU is the most frequently occuring value.

### Numerical data ###

There are 2 data types associated with numerical data: "float64" and "int64". Hence, all columns in those 2 types will be listed in numerical data. As there are 84 numerical variables in the dataset, the output is not presented here to save space. It is clear that several numerical columns like d1_albumin_max, d1_bilirubin_max have large amount of missing values (43-46 thousands).

### :two: Data pre-processing ###

### Missing values ###

First, the total missing values of the dataset are calculated:

Missing values: 2828626 (40.61%)

It is stated in Sharafoddini et al. (2019) that missing values from patient profiles in hospitals (especially intensive care units, ICUs) are inevitable and not at random (MNAR). For example, because of the essence of ICU, doctors will only carry out specific tests for each patient based on their condition. Therefore, the missing rate of 40% in this dataset is understandable. However, if missing data are not carefully handled, they will exert profound impacts on the data mining process.

Columns that have more than 60% missing values will be dropped, as they are unlikely to contain any useful information. Below are those columns, along with their corresponding missing rate:

![image](https://user-images.githubusercontent.com/100490285/178082159-2a3da1c6-71d0-45e5-ac46-5770584c5a8c.png)

![image](https://user-images.githubusercontent.com/100490285/178082168-61090f23-f663-4e88-8f32-4ce57d113232.png)

The remaining columns are inspected again for their missing value rate:

![image](https://user-images.githubusercontent.com/100490285/178082230-cf5ae489-6604-4daa-926c-fa0e53632281.png)

![image](https://user-images.githubusercontent.com/100490285/178082239-8910b917-d072-4693-8ee6-af9869658353.png)

There are still some columns with high missing rate (d1_albumin_max and min, d1_bilirubin_max and min) with almost 60%. However, as mentioned above, those data are MNARs, therefore I will deal with them later.

### Find dulplicates and highly correlated information ###

No duplicated columns were found, and highly correlated information was detected using a correlation matrix to find columns that are more than 70% correlated. The results are illustrated below:

![image](https://user-images.githubusercontent.com/100490285/178082286-79a051b3-2131-48f1-afbd-246fa8febb36.png)

![image](https://user-images.githubusercontent.com/100490285/178082300-4cf4b1a9-6032-4ede-ac4a-d8c1150afd03.png)

1. Glasgow Coma Scale (GCS)

According to [Teasdale et al. (1979)](https://pubmed.ncbi.nlm.nih.gov/290137/ "Teasdale, G., Murray, G., Parker, L. & Jennett, B. (1979), ‘Adding up the glasgow coma score’, Acta Neurochirurgica Supplement 28(1), 13–6."), the GCS score is the combination of 3 tests to assess a person based on their ability to perform eye movements, speak, and move their body. The tests are: E for eye, V for verbal, and M for motor. In the dataset, there are 2 highly correlated variables namely gcs_motor_apache and gcs_eyes_apache, which are 2 components of the GCS score. Therefore, I will take the sum of those 2 variables to form gcs_score and drop the original columns.

2. Weight and BMI

The Body Mass Index (BMI) could be calculated using height and weight. BMI is more representative in terms of obesity as it measures the relativity between height and weight. Thus, it is reasonable to remove weight and keep BMI.

3. Lab based variables

* heart_rate_apache and d1_heartrate_max / wbc_apache and d1_wbc_max

Based on the data dictionary provided, _apache variables and _max variables are basically the same, as both measure the maximum value of a metric during the first 24 hours. _max variables will be useful when accompanied with _min variables, so I will drop the _apache columns.

*  _min and _max columns

Some of the _min columns contain values that are higher than their corresponding max columns. Therefore, I will exchange the values between any cells that have min > max.

![image](https://user-images.githubusercontent.com/100490285/178082471-6f6b55fe-b050-47fd-8bee-a50c0db1c0ed.png)

* _mean columns

As the min/max columns are also highly correlated, I will combine them into a _mean variable that captures the characteristics of both min and max columns.

### Modelling missingness of lab-based data ###

A heatmap of missing data is produced, and it indicates that some columns like d1_albumin_mean and d1_bilirubin_mean are missing together.

![image](https://user-images.githubusercontent.com/100490285/178082560-142fd6ee-319b-4e8f-81f2-eca59d71206c.png)

According to [Sharafoddini et al. (2019)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6329436/ "Sharafoddini, A., Dubin, J., Maslove, D. & Lee, J. (2019), ‘A new insight into missing data in intensive care unit patient profiles: Observational study’, JMIR Medical Informatics 7, e11605."), the missingness of medical data are even more informative than the measured data itself. First, missing data might reflect a variety of opinions among doctors. Second, patients with special conditions that deter them from going through some tests also result in missing data. They also proposed that creating dummy variables for each missing column, and at the same time impute missing data could significantly enhance the predictive power. Therefore, I will create dummy variables for each missing columns, with 1 being missing and 0 otherwise.

New dummy variables for the missingness of d1_albumin_mean and d1_bilirubin_mean are found to be highly correlated so I dropped one of them.

![image](https://user-images.githubusercontent.com/100490285/178082877-ccc040cb-8577-467a-94b8-e165c421e00d.png)

### Encoding categorical variables ###

Before imputing the dataset, the aforementioned categorical columns have to be encoded. For the gender attribute, I used label encoder because they only have 2 unique values: F and M. OneHotEncoder is then used for ethnicity and icu_type to produce dummy variables for each unique value.

### Multiple Imputation (MI) ###

Several studies stated that Multiple Imputation (MI) methods are particularly effective for clinical data ([Sterne et al. 2009](https://pubmed.ncbi.nlm.nih.gov/19564179/ "Sterne, J. A. C., White, I. R., Carlin, J. B., Spratt, M., Royston, P., Kenward, M. G., Wood, A. M. & Carpenter, J. R. (2009), ‘Multiple imputation for missing data in epidemiological and clinical research: potential and pitfalls’, The BMJ 338.")), ([Austin et al. 2021](https://pubmed.ncbi.nlm.nih.gov/33276049/ "Austin, P. C., White, I. R., Lee, D. S. & van Buuren, S. (2021), ‘Missing data in clinical research: A tutorial on multiple imputation’, Canadian Journal of Cardiology 37(9), 1322– 1331. URL: https://www.sciencedirect.com/science/article/pii/S0828282X20311119.")). It uses regression on all columns of the dataset to predict missing values. The process is repeated multiple times and missing data will be imputed using an average value.

After imputing, kernel density estimate (KDE) plots are used to compare the distribution of original data and imputed data.

### References ###

Austin, P. C., White, I. R., Lee, D. S. & van Buuren, S. (2021), ‘Missing data in clinical research: A tutorial on multiple imputation’, Canadian Journal of Cardiology 37(9), 1322– 1331. URL: https://www.sciencedirect.com/science/article/pii/S0828282X20311119.

Sharafoddini, A., Dubin, J., Maslove, D. & Lee, J. (2019), ‘A new insight into missing data in intensive care unit patient profiles: Observational study’, JMIR Medical Informatics 7, e11605.

Sterne, J. A. C., White, I. R., Carlin, J. B., Spratt, M., Royston, P., Kenward, M. G., Wood, A. M. & Carpenter, J. R. (2009), ‘Multiple imputation for missing data in epidemiological and clinical research: potential and pitfalls’, The BMJ 338.

Teasdale, G., Murray, G., Parker, L. & Jennett, B. (1979), ‘Adding up the glasgow coma score’, Acta Neurochirurgica Supplement 28(1), 13–6.
