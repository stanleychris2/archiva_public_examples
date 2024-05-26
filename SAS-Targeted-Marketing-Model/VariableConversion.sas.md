# Data Import, Exploration, Transformation, and Modeling for Regression Analysis

## Steps:
1. Import data from csv file
2. Check for missing observations
3. Explore data using proc contents and proc print
4. Transform variables for regression analysis
5. Create new variables based on existing ones
6. Convert categorical variables to binary
7. Generate graphs for predictors by response
8. Test linear vs quadratic models for selected variables
9. Create final dataset for modeling
10. Check dataset contents and print

## Relevant Code Snippets:

### Step 1: Import data from csv file
```sas
libname storage '/folders/myfolders/';

proc import datafile='/folders/myfolders/Practise/cd.csv' out=resp_data dbms=csv replace;
run;

%include '/folders/myfolders/Practise/data_prep_macros.sas';
```

### Step 2: Check for missing observations
```sas
proc means data=resp_data nmiss;
run;
```

### Step 3: Explore data using proc contents and proc print
```sas
proc contents data=resp_data;
run;

proc print data=resp_data (obs=10);
run;
```

### Step 4: Transform variables for regression analysis
```sas
/* Variable transformation code snippet */
```

### Step 5: Create new variables based on existing ones
```sas
/* New variable creation code snippet */
```

### Step 6: Convert categorical variables to binary
```sas
/* Categorical to binary conversion code snippet */
```

### Step 7: Generate graphs for predictors by response
```sas
/* Graph generation code snippet */
```

### Step 8: Test linear vs quadratic models for selected variables
```sas
/* Linear vs Quadratic model testing code snippet */
```

### Step 9: Create final dataset for modeling
```sas
/* Final dataset creation code snippet */
```

### Step 10: Check dataset contents and print
```sas
/* Dataset contents checking and printing code snippet */
```

For detailed code implementation, refer to the [code file](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase3/Variable%20Conversion.sas) - Variable Conversion.sas