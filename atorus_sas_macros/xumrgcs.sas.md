# SDTM Dataset Merge Macro

## Step 1: Define macro variables and flags
```sas
%macro xumrgcs(inds /*Required. Name(s) of the input dataset(s).*/,
               sourcelib /*Required. Name of the input library.*/,
               supp=Y /*Default: Y. If Y then merge SUPP-- dataset.*/,
               co=Y /*Default: Y. If Y then merge CO dataset.*/,
               debug=N /*Default: N. If N then delete temporary datasets.*/);
```

## Step 2: Check required parameters for emptiness
```sas
%let params_to_check = inds sourcelib supp co;
```

## Step 3: Check validity of supp and co parameters
```sas
%if %lowcase(&supp.) = n and %lowcase(&co.) = n %then %do;
    %put %str(ERR)%str(OR: xumrgcs.sas - both parameters supp and co cannot be specified as N);
    %let end_xumrgcs = Y;
%end;
```

## Step 4: Process each dataset name
```sas
%do i = 1 %to %sysfunc(countw(&inds.));
```

## Step 5: Merge SUPP-- and CO datasets if specified
```sas
%if %lowcase(&supp.) = y %then %do;
```

## Step 6: Delete temporary datasets if debug flag is N
```sas
%if &debug. = N %then %do;
``` 

For more details, you can access the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xumrgcs.sas).