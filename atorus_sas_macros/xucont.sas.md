# SAS Macro for Executing Contents Procedure

## Step 1: Define macro parameters and local variables
```sas
%macro xucont(inds /*Required. Name(s) of the input dataset(s).*/,
              sourcelib /*Required. Name of the input library.*/,
              contopts=varnum /*Default: varnum. Options for the contents procedure.*/);
```

## Step 2: Check if required parameters are empty
```sas
%let params_to_check = inds sourcelib;
```

## Step 3: Count number of datasets specified in INDS parameter
```sas
%let nwrd = %sysfunc(countw(&inds.));
```

## Step 4: Process each dataset name
```sas
%do i = 1 %to &nwrd.;
```

## Step 5: Check if dataset is present in the library
```sas
%if %sysfunc(exist(&sourcelib..&wrd.)) = 0 %then %do;
```

## Step 6: Execute contents procedure
```sas
proc contents data=&sourcelib..&wrd. &contopts.;
run;
``` 

_code_link: [https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xucont.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xucont.sas)_