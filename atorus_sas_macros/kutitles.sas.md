# Macro for Assigning Titles from TFL_titles.csv

## Steps:
1. Check required parameters for macro
2. Check if TFL_titles.csv exists in the directory
3. Remove existing titles
4. Import titles file
5. Check for multiple records for unique program name and sequence
6. Derive sponsor specific titles
7. Create macro variables for titles assignment
8. Assign titles
9. Delete temporary datasets for debug purposes

## Relevant Code Snippets:

### Step 1: Check required parameters for macro
```sas
%macro kutitles(progname, filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.specs, seq=1, justify=c, escapechar=$, debug=N);
```

### Step 2: Check if TFL_titles.csv exists in the directory
```sas
%if %sysfunc(fileexist("&filepath.&II.TFL_titles.csv")) = 0 %then %do;
```

### Step 3: Remove existing titles
```sas
title;
%do i = 1 %to 5;
    %if %symexist(title&i.) %then %do;
        %symdel title&i.;
    %end;
%end;
```

### Step 4: Import titles file
```sas
proc import file="&filepath.&II.TFL_titles.csv" dbms = csv replace out = __tmp_&sysmacroname._1;
```

### Step 5: Check for multiple records for unique program name and sequence
```sas
proc sql noprint;
    select count(*) into: nrows from __tmp_&sysmacroname._1 where progname = "&progname." and seq = &seq.;
quit;
```

### Step 6: Derive sponsor specific titles
```sas
data __tmp_&sysmacroname._2(drop = global:);
```

### Step 7: Create macro variables for titles assignment
```sas
data __tmp_&sysmacroname._&progname._&seq.;
```

### Step 8: Assign titles
```sas
%do i = 1 %to 5;
    %if %sysfunc(symexist(title&i.)) %then %do;
        title&i %unquote(&&title&i.);
    %end;
%end;
```

### Step 9: Delete temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to Code File](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kutitles.sas)