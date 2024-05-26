# Targeted Cross-Sell Marketing Campaign Analysis

## Steps:
1. Merge US Census Zipcode data for Texas with FDIC data for branch-level details of Wells Fargo
```python
# Merge datasets
merged_data = pd.merge(census_data, fdic_data, on='zipcode', how='inner')
```

2. Clean the merged dataset by analysing and deleting missing records
```python
# Check for missing values
missing_values = merged_data.isnull().sum()
# Drop rows with missing values
cleaned_data = merged_data.dropna()
```

3. Segregate highly correlated attributes and eliminate one with less discrimination in samples
```python
# Calculate correlation matrix
correlation_matrix = cleaned_data.corr()
# Identify highly correlated attributes
highly_correlated = correlation_matrix[correlation_matrix > 0.8]
# Eliminate one of the highly correlated attributes
cleaned_data = cleaned_data.drop(columns=['attribute_to_eliminate'])
```

4. Detect themes in the dataset for the remaining attributes
```python
# Perform thematic analysis
themes = detect_themes(cleaned_data)
```

5. Generate Clusters to segment Customers as per Socio-Economic conditions
```python
# Perform clustering analysis
clusters = generate_clusters(cleaned_data, n_clusters=5)
```

6. Convert Nominal variables to Binary and set Ordinal variables to middle of the range values
```python
# Convert Nominal variables to Binary
cleaned_data['nominal_variable'] = pd.get_dummies(cleaned_data['nominal_variable'])
# Set Ordinal variables to middle values
cleaned_data['ordinal_variable'] = cleaned_data['ordinal_variable'].apply(lambda x: x if x != 'missing' else cleaned_data['ordinal_variable'].median())
```

7. Check Ordinal variables for Linearity
```python
# Check for linearity in Ordinal variables
linearity_check(cleaned_data['ordinal_variable'])
```

8. Segregate dataset into Training and Testing samples
```python
# Split dataset into Training and Testing samples
X_train, X_test, y_train, y_test = train_test_split(cleaned_data.drop('target_variable', axis=1), cleaned_data['target_variable'], test_size=0.2, random_state=42)
```

9. Run Logistic Regression algorithm on Training sample
```python
# Fit Logistic Regression model
logreg = LogisticRegression()
logreg.fit(X_train, y_train)
```

10. Score the model for Testing sample
```python
# Score the Logistic Regression model
accuracy = logreg.score(X_test, y_test)
```

11. Run Linear Regression algorithm with important Predictors
```python
# Fit Linear Regression model
linreg = LinearRegression()
linreg.fit(X_train[['important_predictor1', 'important_predictor2']], y_train)
```

For more details, refer to the full code [here](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/README.md)