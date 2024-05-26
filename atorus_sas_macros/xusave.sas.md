# Dataset Saving Macro with Sorting and Labeling

### Step 1: Check required parameters for empty values
```sas
%let params_to_check = inds outlib outds;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( )).;

    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xusave.sas - &param_name. is required parameter and should not be NULL);
        %let end_xusave = Y;
    %end;
%end;
```

### Step 2: Create intermediate dataset
```sas
data __tmp_&sysmacroname._&inds.;
    set &inds.;
run;
```

### Step 3: Sort the dataset if sort order is specified
```sas
%if &sortvars. ^= %then %do;
    proc sort data = __tmp_&sysmacroname._&inds.;
        by &sortvars.;
    run;
%end;
```

### Step 4: Save .sas7bdat file with needed variables and dataset label
```sas
data &outlib..&outds. (%if &dslbl. ^= %then %do; label = "&dslbl." %end;
                       %if &keepvars. ^= %then %do; keep = &keepvars. %end;); 
    set __tmp_&sysmacroname._&inds.;
    %if &sortvars. ^= %then %do; by &sortvars.; %end;
run;
```

### Step 5: Save .xpt file if specified
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

### Step 6: Delete temporary datasets if debug flag is set to N
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xusave.sas)