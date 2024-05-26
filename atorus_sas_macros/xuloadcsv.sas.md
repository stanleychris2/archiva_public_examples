# Combine Multiple Raw .csv Datasets into One Dataset

### Steps:
1. Check required parameters for empty values
2. Create dataset with list of files in a folder filtered by regex
3. Count number of files
4. Iterate over files and import data
5. Choose max length for each variable
6. Set all raw data together and assign max length for each variable
7. Delete temporary datasets if debug is set to N

### Relevant Code Snippets:
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

```sas
%*Choose max length for each variable;
data __tmp_&sysmacroname._names;
    set __tmp_&sysmacroname._names2_:;
run;

proc sql noprint;
    create table __tmp_&sysmacroname._names2 as 
    select name, type, max(length) as max_len 
    from __tmp_&sysmacroname._names 
    group by name, type 
    order by varnum;
    
    select cat(%if %lowcase(&compress_names.) = y %then %do; compress(name, "_", "kda") %end; 
                %else %do; catt("\'", name, "\'n") %end;, 
                ifc(type = 1, " ", " $"), ifn(type = 1, max_len, max_len + 10)) 
                    into: _var_length_statement separated by " " from __tmp_&sysmacroname._names2;
quit;
```

For the full code, please refer to the [link](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuloadcsv.sas).