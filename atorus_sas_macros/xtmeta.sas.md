# Zero Record Dataset Creation from Metadata

## Steps:
1. Check required parameters for empty values.
```sas
%let params_to_check = dsname filepath;
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
    ...
%end;
%else %do;
    %put %str(ERR)%str(OR: xtmeta.sas - &dsname. does not exist in &infile.. KEEPSTRING and EMPTY_&dsname. are not created);
%end;
```

6. Delete temporary datasets if debug flag is N.
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

**Code Link:** [_xtmeta.sas_](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xtmeta.sas)