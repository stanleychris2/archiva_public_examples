# Variable Renaming Macro

## Step 1: Define macro parameters and local variables

```sas
%macro xurnm(inds /*Required. Name of the input dataset.*/,
            mode=add /*Default: add. Working mode (add/remove).*/,
            prefix=_ /*Default: _. Characters to add before the variable name.*/,
            rchar=1 /*Default: 1. Number of characters to remove from the beginning of the variable name.*/,
            outds=&inds. /*Default: input dataset. Name of output dataset.*/,
            debug=N /*Default: N. If N then deletes temporary datasets.*/);
```

## Step 2: Check for required parameters and their values

```sas
%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
```

## Step 3: Create dataset metadata and sort the dataset

```sas
proc contents data = &inds. out = __tmp_&inds._1 noprint varnum;
run;

proc sort data = __tmp_&inds._1;
    by memname varnum;
run;
```

## Step 4: Add prefix or remove characters from variable names

```sas
%*Add prefix to each variable in the dataset.;
%if %lowcase(&mode.) = add %then %do;
```

## Step 5: Delete temporary datasets if debug flag is set to N

```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails library = work;
        delete __tmp_&inds.:;
    run;
%end;
```

## Step 6: Rename variables in the output dataset

```sas
data &outds.;
    set &inds.(rename = (&rename_list.));
run;
```

_Code Link: [xurnm.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xurnm.sas)_