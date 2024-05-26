# Macro for Deriving EPOCH Variable

### Step 1: Define macro parameters and local variables
```sas
%macro xuepoch(inds /*Required. Name of the input dataset.*/, 
               dtcdate /*Required. Name of the date variable.*/,
               outds=&inds. /*Default: &inds. Name of the output dataset.*/,
               debug=N /*Default: N. If N then deletes temporary variables.*/);
```

### Step 2: Check if required parameters are not empty
```sas
%*Check whether required parameters are empty;
%if %length(&param_value.) = 0 %then %do;
    %put %str(ERR)%str(OR: xuepoch.sas - &param_name. is required parameter and should not be NULL);
    %let end_xuepoch = Y;
%end;
```

### Step 3: Upload SE domain and sort data
```sas
%xuload(se, sdtm);

proc sort data = se out = __tmp_&sysmacroname._1;
    by usubjid sestdtc;
run;
```

### Step 4: Transpose SE domain to get one record per subject dataset
```sas
proc transpose data = __tmp_&sysmacroname._1 out = __tmp_&sysmacroname._2 prefix = tm_st_;
    by usubjid;
    var sestdtc;
    id epoch;
run;
```

### Step 5: Merge input dataset and transposed SE domain
```sas
data &outds. %if &debug. = N %then %do; (drop = tm_st_: __tmp_: _name_ _label_) %end;;
    merge __tmp_&sysmacroname._3(in = a) __tmp_&sysmacroname._2;
    by usubjid;
    if a;

    array __tmp_st [*] $ tm_st_:;
```

### Step 6: Derive EPOCH for each record
```sas
do __tmp_i = 1 to dim(__tmp_st);
    %*Get the minimum length among two dates;
    __tmp_length = min(length(__tmp_st[__tmp_i]), length(&dtcdate.));

    %*If both dates have time then compare them without truncation;
    if __tmp_length = 16 then do;
        if __tmp_st[__tmp_i] <= &dtcdate. then epoch = substr(vname(__tmp_st[__tmp_i]), 7);
    end;
    %*If at least one date has no time, then compare only date parts;
    else if __tmp_length = 10 then do;
        if substr(__tmp_st[__tmp_i], 1, __tmp_length) <= substr(&dtcdate., 1, __tmp_length) then epoch = substr(vname(__tmp_st[__tmp_i]), 7);
    end;
    %*If at least one date is partial, then compare by the length of that date;
    else if cmiss(__tmp_st[__tmp_i], &dtcdate.) = 0 then do;
        if substr(__tmp_st[__tmp_i], 1, __tmp_length) <= substr(&dtcdate., 1, __tmp_length) then epoch = substr(vname(__tmp_st[__tmp_i]), 7);
    end;
end;
```

### Step 7: Delete temporary datasets if debug flag is set to N
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

Code Link: [xuepoch.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuepoch.sas)