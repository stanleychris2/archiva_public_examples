# Combine Multiple Raw .csv Datasets into One Dataset

### Steps:
1. Check required parameters for empty values.
```sas
%macro xuloadcsv(filename /*Required. Regular expression to filter the files in source directory.*/, 
                filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.raw&II.external /*Default: final/raw/external. Path to raw .csv datasets.*/, 
                splitchar=$ /*Default: $. Delimiter used in .csv files.*/,
                datarow=2 /*Default: 2. Datarow option in import procedure.*/,
                subjid= /*Optional. Regular expression to parse filename and get subject id.*/,
                visit= /*Optional. Regular expression to parse filename and get visit.*/,
                outds=raw_all /*Default: raw_all. Name of the output dataset.*/,
                getnames=yes /*Default: yes. Getnames option in import procedure.*/,
                compress_names=Y /*Default: Y. Option to remove spaces from variable names.*/,
                debug=N /*Default: N. Option to clean up intermediate datasets after macro execution.*/);
```

2. Create a dataset with a list of files in a folder filtered by regex.
```sas
data __tmp_&sysmacroname._filenames;
    length fref $8 fname subjid_derived visit_derived $200;
    
    did = filename(fref, "&filepath.");
    did = dopen(fref);
    
    do i = 1 to dnum(did);
        fname = dread(did, i);
        if prxmatch("/&filename./", strip(fname)) and index(fname, "xlsx") = 0 and index(fname, "xlt") = 0 then do;
            subjid_derived = "_";
            ...
        end;
    end;
    
    did = dclose(did);
    did = filename(fref);
run;
```

3. Count the number of files.
```sas
data _null_;
    if 0 then set __tmp_&sysmacroname._filenames nobs = n;
    call symputx("__filenames_nobs", n);
run;

proc sql noprint;
    select catx("#", fname, subjid_derived, visit_derived) into: fname_list separated by "," from __tmp_&sysmacroname._filenames;
quit;
```

4. Iterate over files, import data, and handle variable names.
```sas
%do i=1 %to %sysfunc(countw(%bquote(&fname_list.), %str(,)));

    %let curr_fname = %scan(%bquote(&fname_list.), &i., %str(,));
    %let __fname = %scan(%bquote(&curr_fname.), 1, %str(#));
    %let __subjid = %scan(%bquote(&curr_fname.), 2, %str(#));
    %let __visit = %scan(%bquote(&curr_fname.), 3, %str(#));
    
    ...
    
%end;
```

5. Choose max length for each variable.
```sas
data __tmp_&sysmacroname._names;
    set __tmp_&sysmacroname._names2_:;
run;

proc sql noprint;
    create table __tmp_&sysmacroname._names2 as 
    select name, type, max(length) as max_len 
    from __tmp_&sysmacroname._names 
    group by name, type 
    order by varnum;
    
    ...
quit;
```

6. Set all raw data together and assign max length for each variable.
```sas
data &outds.;
    length &_var_length_statement.;
    set __tmp_&sysmacroname._raw_data_:;
    
    ...
run;
```

7. Delete/keep temporary datasets for debug purposes.
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname._:;
    run;
%end;
``` 

Code Link: [xuloadcsv.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuloadcsv.sas)