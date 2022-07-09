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

![image](https://user-images.githubusercontent.com/100490285/178083514-8ed61766-edf9-4d1a-bb61-7190ed689491.png)

The remaining columns are inspected again for their missing value rate:

![image](https://user-images.githubusercontent.com/100490285/178083524-6de23859-25c3-41ca-b68b-efe7438b3181.png)

There are still some columns with high missing rate (d1_albumin_max and min, d1_bilirubin_max and min) with almost 60%. However, as mentioned above, those data are MNARs, therefore I will deal with them later.

### Find dulplicates and highly correlated information ###

No duplicated columns were found, and highly correlated information was detected using a correlation matrix to find columns that are more than 70% correlated. The results are illustrated below:

![image](https://user-images.githubusercontent.com/100490285/178083550-84edf03b-82ad-4901-af21-bfe7255c1394.png)

![image](https://user-images.githubusercontent.com/100490285/178083567-2582affa-782f-4e4f-8124-8bbbbb56de72.png)

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

![image](https://user-images.githubusercontent.com/100490285/178083221-4f6b5f29-3098-4ae0-ad1b-42e4ba7235d5.png)

By looking at the plots, we could see that the distribution of variables does not changed much after imputing, except for d1_albumin_mean due to its high portion of missing values. However, its plot does not seem to be skewed to the sides, therefore, the imputed data could be used for further analysis.

### Outliers detection ###

Boxplots’ quantiles are used to calculate the percentages of outliers in the dataset. However, the result produced by such quantiles are not reliable because there are normal people who naturally have high indicators so they should not be considered as outliers. The summary below shows the number of outliers calculated by using boxplots:

![image](https://user-images.githubusercontent.com/100490285/178083637-4fbd1cc6-60b8-4a5e-aa30-d58edcecec3e.png)

![image](https://user-images.githubusercontent.com/100490285/178083258-e40ea634-03a6-4874-aec3-ed02286aa765.png)

After approximating the percentages of outliers in the dataset, I used both Isolation Forest and Local Outlier Factor (LOF) algorithms to determine the true value of outliers. They produce the same result because I tuned the contamination parameter to be exactly 4.93%, according to the boxplots.

![image](https://user-images.githubusercontent.com/100490285/178083714-a8091798-3fdb-47f9-a849-29fa8121f0d3.png)

Based on the result, I discarded all outliers to produce the final dataset, with 72869 rows and 50 features.

### :three: Supervised model training and evaluation ###

### Train-test split and re-sampling ###

Before doing any sampling or feature selection, the dataset should be split into train and test set. The plot below illustrates that the dataset is imbalance, because people with diabetes mellitus are minority class. Therefore, splitting with stratification is more useful in this case.

![image](https://user-images.githubusercontent.com/100490285/178083323-ca2763e2-92c9-4e29-bf33-80f539bbcf67.png)

30% of the dataset will be used for testing, and the remaining are for training. As the dataset is large, the training set will be further split into a validation set of 30%. For the training set, I will use downsampling method to deal with class imbalance.

First, the dataset is large enough to be downsampled without losing any useful information. Second, upsampling techniques usually involve generating artificial data points so it is not reliable. Last but not least, it reduces the size of the dataset, making it less heavy for computational purposes. After downsampling, the training set is reduced to about 24 thousand observations.

### Feature Selection ###

The SelectKBest algorithm with two score functions f_classif and mutual_info_classif are used to rank features based on their relevance. The best 10 features selected by 2 score functions are almost identical.

![image](https://user-images.githubusercontent.com/100490285/178083374-51d9720f-8792-40ab-926b-c362da2e3195.png)

There are one dummy variable for modelling missingness in the list: i_glucose, and several more in top 15, 20. However, I will only use this list as a reference for my classification pipeline, which will be discussed below.

### Classification ###

Six classification algorithms with default settings were used for benchmarking purposes: KNN, SVM, Decision Tree, Logistic Regression, Multi Layer Perceptron (MLP), and Random Forest. MLP model achieves the highest accuracy score of 82%, whereas a single decision tree performs worst with only 74%.

![image](https://user-images.githubusercontent.com/100490285/178083424-37bc0a99-09d1-4e16-a424-43878a5b7f6f.png)

After that, the validation dataset is used for hyper-parameter tuning. I used a classification pipeline that have a transformer, a feature selection algorithm, a classifier and grid search to find the best parameters for each algorithm. All of which are then perform on the test set for final evaluation. The result shows that the pipeline of SVC outperformed its counterparts in all 4 evaluation metrics with overall 84% accuracy, 88% recall, 73% precision, and 80% F1-score on the minority class.

![image](https://user-images.githubusercontent.com/100490285/178083743-f8906f03-91be-40bd-995d-f95ce5881c9f.png)

The confusion matrix and ROC curve of SVC classifier is illustrated below:

![image](https://user-images.githubusercontent.com/100490285/178083760-2aad8ae1-a514-477e-9b4a-37a7de836d9c.png)

According to the metrics and visualizations, my model performs quite effectively with high recall score for the minority class i.e. people who have diabetes mellitus. However, due to the low precision score, the model misclassifies normal people to have diabetes mellitus by more than 17%. Nevertheless, this could still be considered to be a good model with a balance in the F1-score of 80%. As the cost of misclassifying normal people is lower than misclassifying sick ones, models that prioritize on predicting minority class is generally better.

### :four: Unsupervised Clustering (Additional task) ###

The validation dataset is used for clustering algorithms because it has not been downsampled. First, the target variable will be omitted before any clustering. Second, Principle Component Analysis (PCA) was performed to reduce data dimensions. By plotting the explained variance of each component, the optimal number of features (8) was chosen.

![image](https://user-images.githubusercontent.com/100490285/178083854-2dc045fb-4967-4e5d-8e74-c43601444f35.png)

After that, the Elbow method and Silhouette score are used cho choose the optimal number of clusters, with the objective of minimizing the within cluster sum of square (WCSS) and maximizing the between clusters sum of squares (BCSS).

![image](https://user-images.githubusercontent.com/100490285/178083872-c0f12b2b-22a3-4131-b395-1daf3c226b4e.png)

It is clear from the plots that the optimal number of clusters for this dataset should be 2. The Silhoutte score is calculated to see how well the clusters formed.

![image](https://user-images.githubusercontent.com/100490285/178083923-9129bc3f-7a2e-40c3-a92d-4d66023b7611.png)

The relatively low Silhoutte score implies that the clusters are not entirely separated. Next, K-means algorithm is fitted to the PCA dataset and its labels will be compared with real data labels.

![image](https://user-images.githubusercontent.com/100490285/178083934-b43f64b2-44a9-40b4-84df-e2128416af9d.png)

By looking at the plot, we can see that this algorithm did not perform quite well with the dataset. In addition, the Adjusted Mutual Information (AMI) score and the Adjusted Rand Index score (ARI) will be used to evaluate the quality of this algorithm.

![image](https://user-images.githubusercontent.com/100490285/178083952-60f15b2b-007c-4e84-b7eb-20c735337099.png)

Both of the metrics are really low, it means that this algorithm is randomly labelling data points, independent of the number of clusters (k). A confusion matrix and classification report between supervised and cluster labels is used for comparison.

![image](https://user-images.githubusercontent.com/100490285/178083981-ff7b65d0-1968-483e-a2de-e3c9f96aab5c.png)

The K-means algorithm has relatively low recall score for class 0. It implies that there are normal people with naturally high indicators that match with sick people to form the cluster.

### References ###

Austin, P. C., White, I. R., Lee, D. S. & van Buuren, S. (2021), ‘Missing data in clinical research: A tutorial on multiple imputation’, Canadian Journal of Cardiology 37(9), 1322– 1331. URL: https://www.sciencedirect.com/science/article/pii/S0828282X20311119.

Sharafoddini, A., Dubin, J., Maslove, D. & Lee, J. (2019), ‘A new insight into missing data in intensive care unit patient profiles: Observational study’, JMIR Medical Informatics 7, e11605.

Sterne, J. A. C., White, I. R., Carlin, J. B., Spratt, M., Royston, P., Kenward, M. G., Wood, A. M. & Carpenter, J. R. (2009), ‘Multiple imputation for missing data in epidemiological and clinical research: potential and pitfalls’, The BMJ 338.

Teasdale, G., Murray, G., Parker, L. & Jennett, B. (1979), ‘Adding up the glasgow coma score’, Acta Neurochirurgica Supplement 28(1), 13–6.
