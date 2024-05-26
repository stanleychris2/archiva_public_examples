# SAS Macro for Executing Contents Procedure

## Step 1: Define macro xucont with input parameters inds, sourcelib, and contopts.
```sas
%macro xucont(inds /*Required. Name(s) of the input dataset(s).*/,
              sourcelib /*Required. Name of the input library.*/,
              contopts=varnum /*Default: varnum. Options for the contents procedure.*/);
```

## Step 2: Check if required parameters are not empty.
```sas
%*Check whether required parameters are empty;
%if %length(&param_value.) = 0 %then %do;
    %put %str(ERR)%str(OR: xucont.sas - &param_name. is required parameter and should not be NULL);
    %let end_xucont = Y;
%end;
```

## Step 3: Count number of datasets specified in inds parameter.
```sas
%let nwrd = %sysfunc(countw(&inds.));
```

## Step 4: Loop through each dataset name.
```sas
%do i = 1 %to &nwrd.;
```

## Step 5: Check if dataset is present in the library.
```sas
%if %sysfunc(exist(&sourcelib..&wrd.)) = 0 %then %do;
    %put %str(ERR)%str(OR: xucont.sas - dataset %upcase(&wrd.) was not found in %upcase(&sourcelib.) library);
%end;
```

## Step 6: Execute contents procedure for each dataset.
```sas
%*Execute contents procedure;
proc contents data=&sourcelib..&wrd. &contopts.;
run;
``` 

For more details, you can access the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xucont.sas).