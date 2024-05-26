# Zipcode Demographic File Analysis and Factor Analysis

## Steps:
1. Check the data using proc contents and proc print

```sas
/*library statement*/
*	Note: adjust this as necessary;
libname storage '/folders/myfolders/';

/*Check the data*/
proc contents data=storage.dmefzip position;
title "Contents of Zipcode Demographic File";
run;
proc print data=storage.dmefzip (obs=10);
run;
title;
```

2. Calculate summary statistics using proc means

```sas
proc means data=storage.dmefzip;
title "Summary Statistics for Zipcode File";
run;
title;
```

3. Count missing values and isolate records with missing values

```sas
proc means data=storage.dmefzip n nmiss maxdec=0;
title "Missing Values Report for Zipcode File";
run;
title;
```

4. Change variables using proc standard

```sas
proc standard data=storage.dmefzip replace out=prefactor;
run;
```

5. Check if factor analysis is appropriate using proc corr

```sas
proc corr data=prefactor rank;
run;
```

6. Standardize data using proc standard

```sas
proc standard data=prefactor out=storage.standard mean=0 std=1;
var _numeric_;
run;
```

7. Perform factor analysis using proc factor

```sas
proc factor data=storage.standard scree mineigen=0 ;
run;
```

8. Create scores using proc factor

```sas
proc factor data=storage.standard rotate=varimax scree nfactors=5 fuzz=.4 out=storage.census_factors;
run;
proc print data=storage.census_factors (obs=10);
run;
```

For the full SAS code, click [here](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase2/Factor%20Analysis.sas)