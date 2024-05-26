# Convert Character DTC Variable into Analysis Date Variables

## Steps:
1. Define macro parameters and local variables
2. Check if required parameters are not empty
3. Handle different date formats
4. Calculate relative analysis day if reference date is provided

### Step 1: Define macro parameters and local variables
```sas
%macro xcdtc2dt(dtcdate /*Required. Name of --DTC variable.*/,
                prefix=a /*Default: a. Prefix for numeric -DT -TM -DTM -DY analysis variables.*/,
                refdate= /*Optional. Reference numeric date.*/);
```

### Step 2: Check if required parameters are not empty
```sas
%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( ))); 
```

### Step 3: Handle different date formats
```sas
%*Handle date without time or date with partial time.;
if 10 <= length(strip(&dtcdate.)) < 16 then %upcase(&prefix.dt) = input(substr(strip(&dtcdate.), 1, 10), e8601da.);
%*Handle date with time.;
else if length(strip(&dtcdate.)) = 16 or length(strip(&dtcdate.)) = 19 then do;
    %upcase(&prefix.dtm) = input(strip(&dtcdate.), e8601dt.);
    %upcase(&prefix.dt) = datepart(&prefix.dtm);
    %upcase(&prefix.tm) = timepart(&prefix.dtm);
end;
```

### Step 4: Calculate relative analysis day if reference date is provided
```sas
%*Calculate relative analysis day only if both analysis dates are non-missing.;
%if &refdate. ^= %then %do;
    if nmiss(&prefix.dt, &refdate.) = 0 then %upcase(&prefix.dy) = &prefix.dt - &refdate. + (&prefix.dt >= &refdate.);
%end;
``` 

Code Link: [xcdtc2dt.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xcdtc2dt.sas)