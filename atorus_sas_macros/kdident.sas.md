# Character Variable Splitting Macro

## Step 1: Check if required parameters are empty
```sas
%*Define required parameter names that should be checked whether they are empty;
%let params_to_check = invar maxln outvar;

%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %*Sub-select macro name;
    %let param_name = %scan(&params_to_check., &i., %str( ));

    %*Sub-select macro value;
    %let param_value = &%scan(&params_to_check., &i., %str( )).;

    %*Check whether required parameters are empty;
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: kdident.sas - &param_name. is required parameter and should not be NULL);
        %let end_kdident = Y;
    %end;
%end;
```

## Step 2: Check input variable length
```sas
if length(&invar.) >= 10000 then put "WAR" "NING: kdident.sas - input variable length is more significant than 10000. Need to update macro conditions";
```

## Step 3: Split input variable into output variable based on max line length
```sas
length &outvar. $10000 __tmp_&invar.l1-__tmp_&invar.l1000 8;
%*Array of debug variables, will store position of the new line (should be right after newline character and tabs);
array __tmp_&invar.ls {*} __tmp_&invar.l1-__tmp_&invar.l1000;
&outvar. = strip(scan(&invar., 1, " "));
```

## Step 4: Handle debug mode to delete temporary variables
```sas
%*Delete/keep temporary variables for debug purposes;
%if &debug. = N %then %do;
    drop __tmp_:;
%end;
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kdident.sas)