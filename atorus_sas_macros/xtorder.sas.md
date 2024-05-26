# Macro for Creating Global Macro Variable from Dataset Metadata

## Steps:
1. Define macro parameters and global/local variables.
```sas
%macro xtorder(dsname /*Required. Dataset to look for in the metadata file.*/,
               filename= /*Optional. Name of the CSV file with metadata. Determined automatically if that parameter is empty.*/,
               filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.specs /*Default: final/specs. Full pass to CSV file with metadata.*/,
               debug=N /*Default: N. If N then delete temporary datasets.*/);
```

2. Check if required parameters are empty.
```sas
%let params_to_check = dsname filepath;
```

3. Determine metadata file based on input dataset name.
```sas
%if &filename. = %then %do;
    %if %length(%cmpres(&dsname.)) = 2 or %index(%lowcase(&dsname.),supp) > 0 or %lowcase(%cmpres(&dsname.)) = relrec or (%length(%cmpres(&dsname.)) = 4 and %substr(%lowcase(%cmpres(&dsname.)), 1, 2) = ap) %then %do;
        %let infile = &filepath.&II.SDTM_spec_Datasets.csv;
    %end;
    %else %do;
        %let infile = &filepath.&II.ADAM_spec_Datasets.csv;
    %end;
%end;
%else %do;
    %let infile = &filepath.&II.&filename.;
%end;
```

4. Check if metadata file exists.
```sas
%if %sysfunc(fileexist(&infile.)) = 0 %then %do;
    %put %str(ERR)%str(OR: xtorder.sas - input &filename. file was not found in &filepath.);
    %let end_xtorder = Y;
%end;
```

5. Import metadata file.
```sas
filename metadata "&infile." termstr = CRLF;

proc import out = __tmp_&sysmacroname._1 datafile = metadata dbms = csv replace;
    guessingrows = max;
    delimiter = "$";
run;
```

6. Check if dataset exists in metadata.
```sas
proc sql noprint;
    select strip(put(count(*), best.)) into: dscheck from __tmp_&sysmacroname._1
    where strip(lowcase(dataset)) = strip(lowcase("&dsname."));
quit;
```

7. Create SORTSTRING macro variable if dataset exists.
```sas
%if &dscheck. > 0 %then %do;
    data _null_;
        set __tmp_&sysmacroname._1(where = (strip(lowcase(dataset)) = strip(lowcase("&dsname.")));
        
        call symputx(compress("&dsname." || "SORTSTRING"), translate(compress("Key Variables"n), " ", ","));
    run;
%end;
%else %do;
    %put %str(ERR)%str(OR: xtorder.sas - &dsname. does not exist in &infile.. SORTSTRING variable is not created);
%end;
```

8. Delete temporary datasets based on debug flag.
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xtorder.sas)