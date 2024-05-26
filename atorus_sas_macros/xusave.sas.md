# Dataset Saving Macro with Sorting and Labeling

### Step 1: Define macro xusave with input parameters
```sas
%macro xusave(inds /*Required. Name of the input dataset.*/,
              outlib /*Required. Name of the output library.*/,
              outds=&inds. /*Default: &inds. Name of the output dataset.*/,
              keepvars= /*Optional. List of variables to keep.*/,
              sortvars= /*Optional. Sort order for the dataset.*/,
              dslbl= /*Optional. Label for the dataset.*/,
              xpt=Y /*Default: Y. If Y then saves .xpt file.*/,
              debug=N /*Default: N. If N then delete temporary datasets.*/);
```

### Step 2: Check if required parameters are not empty
```sas
%let params_to_check = inds outlib outds;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( ));

    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xusave.sas - &param_name. is required parameter and should not be NULL);
        %let end_xusave = Y;
    %end;
%end;
```

### Step 3: Create intermediate dataset
```sas
data __tmp_&sysmacroname._&inds.;
    set &inds.;
run;
```

### Step 4: Sort the dataset if sort order is specified
```sas
%if &sortvars. ^= %then %do;
    proc sort data = __tmp_&sysmacroname._&inds.;
        by &sortvars.;
    run;
%end;
```

### Step 5: Save .sas7bdat file with needed variables and dataset label
```sas
data &outlib..&outds. (%if &dslbl. ^= %then %do; label = "&dslbl." %end;
                       %if &keepvars. ^= %then %do; keep = &keepvars. %end;); 
    set __tmp_&sysmacroname._&inds.;
    %if &sortvars. ^= %then %do; by &sortvars.; %end;
run;
```

### Step 6: Save .xpt file if xpt flag is set to Y
```sas
%if &xpt. = Y %then %do;
    %let xportlib = %sysfunc(pathname(&outlib.));
    libname xptfile xport "&xportlib.&II.%sysfunc(lowcase(&outds.)).xpt";

    data xptfile.&outds. (%if &dslbl. ^= %then %do; label = "&dslbl." %end;
                         %if &keepvars. ^= %then %do; keep = &keepvars. %end;);
        set __tmp_&sysmacroname._&inds.;
        %if &sortvars. ^= %then %do; by &sortvars.; %end;
    run;
%end;
```

### Step 7: Delete or keep temporary datasets based on debug flag
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
```

**Code Link:** [_link](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xusave.sas)