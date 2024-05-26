# Dataset Creation and Metadata Handling

## Steps:
1. Check required parameters for empty values.
```sas
%let params_to_check = dsname filepath;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( ));
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xtmeta.sas - &param_name. is required parameter and should not be NULL);
        %let end_xtmeta = Y;
    %end;
%end;
```

2. Determine metadata file based on dataset name.
```sas
%if &filename. = %then %do;
    %if %length(%cmpres(&dsname.)) = 2 or %index(%lowcase(&dsname.), supp) > 0 or %lowcase(%cmpres(&dsname.)) = relrec or (%length(%cmpres(&dsname.)) = 4 and %substr(%lowcase(%cmpres(&dsname.)), 1, 2) = ap) %then %do;
        %let infile = &filepath.&II.SDTM_spec_Variables.csv;
    %end;
    %else %do;
        %let infile = &filepath.&II.ADAM_spec_Variables.csv;
    %end;
%end;
%else %do;
    %let infile = &filepath.&II.&filename.;
%end;
```

3. Import metadata file.
```sas
filename metadata "&infile." termstr = CRLF;

proc import out = __tmp_&sysmacroname._1 datafile = metadata dbms = csv replace;
    guessingrows = max;
    delimiter = "$";
run;
```

4. Check if dataset exists in metadata file.
```sas
proc sql noprint;
    select strip(put(count(*), best.)) into: dscheck from __tmp_&sysmacroname._1
        where strip(lowcase(dataset)) = strip(lowcase("&dsname."));
quit;
```

5. Create macro variables and dataset if dataset exists.
```sas
%if &dscheck. > 0 %then %do;
    %*Sort the dataset by expected specified variable order;
    proc sort data = __tmp_&sysmacroname._1 sortseq = linguistic(numeric_collation = on);
        where (strip(lowcase(dataset)) = strip(lowcase("&dsname.")) and lowcase("Use (y)"n) = "y");
        by order;
    run;

    %local miss_len len_type;
    proc sql noprint;
        select distinct variable into :miss_len separated by ", " 
        from __tmp_&sysmacroname._1
        where missing(length);

        select lowcase(type) into: len_type
        from sashelp.vcolumn
        where lowcase(libname) = "work" and lowcase(memname) = lowcase("__tmp_&sysmacroname._1") and lowcase(name) = "length";
    quit;

    ...
%end;
```

6. Delete temporary datasets if debug flag is set to N.
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xtmeta.sas)