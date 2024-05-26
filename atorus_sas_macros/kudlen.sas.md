# Variable Length Adjustment Macro

## Step 1: Check required parameters for empty values
```sas
%*Define required parameter names that should be checked whether they are empty;
%let params_to_check = inds outds;

%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %*Sub-select macro name;
    %let param_name = %scan(&params_to_check., &i., %str( ));

    %*Sub-select macro value;
    %let param_value = &%scan(&params_to_check., &i., %str( ));

    %*Check whether required parameters are empty;
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: kudlen.sas - &param_name. is required parameter and should not be NULL);
        %let end_kudlen = Y;
    %end;
%end;
```

## Step 2: Create temporary datasets
```sas
data __tmp_&sysmacroname._1;
    set &inds.;
run;
```

## Step 3: Calculate maximum length of variables
```sas
data __tmp_&sysmacroname._4 (keep = len_:);
    set __tmp_&sysmacroname._1 end = last;

    %*Count length of each variable;
    %do i = 1 %to &nwrd.;
        %let wrd = %scan(&charvars., &i.);
        len_&wrd. = length(&wrd.);
    %end;
    
run;
```

## Step 4: Apply variables length, label, format, and rename
```sas
%*Apply variables length, label, format and rename them.;
data &outds. %if &debug. = N %then (drop = __tmp_kudlen:);;
    length &lenvar.;
    set __tmp_&sysmacroname._1;
    &reset.;
    label &lblset.;
    format &fmt.;;
run;
```

## Step 5: Delete temporary datasets if debug flag is set to N
```sas
%*Delete/keep temporary datasets for debug purposes;
%if &debug. = N %then %do;
    proc datasets library = work nolist;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

**Code Link:** [_https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kudlen.sas_](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kudlen.sas)