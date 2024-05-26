# Macro for Adding ADSL Core Variables to Analysis Dataset

## Step 1: Check and validate input parameters
```sas
%macro xtcore(inds /*Required. Name of the parent dataset for adding ADSL core variables.*/,
              outds=&inds. /*Default: &inds. Name of the output dataset.*/,
              filename=ADAM_spec_Variables.csv /*Default: ADaM_spec_Codelist.csv. Indicates source file name.*/,
              filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.specs /*Default: final/specs. Indicates path to source file.*/,
              debug=N /*Default: N. If Y then deletes temporary datasets.*/);
```

## Step 2: Load metadata information into macro variable
```sas
filename metadata "&filepath.&II.&filename." termstr = CRLF;

proc import out = __tmp_&sysmacroname._var_0 datafile = metadata dbms = csv replace;
    guessingrows = max;
    delimiter = "$";
run;
```

## Step 3: Sort the dataset by specified variable order
```sas
proc sort data = __tmp_&sysmacroname._var_0 (where = (strip(lowcase(strip(dataset))) = "adsl"));
    by order;
run;
```

## Step 4: Define core variables and input dataset variable names
```sas
select variable into :core_vars separated by " " from __tmp_&sysmacroname._var_0 where lowcase(strip(core)) = "y";
```

## Step 5: Find intersection between core variable names and input dataset variable names
```sas
create table __tmp_&sysmacroname._intersect_vars as
        select a.name 
        from __tmp_&sysmacroname._inds_vars as a left join __tmp_&sysmacroname._var_0 as b
        on lowcase(a.name) = lowcase(b.variable)
        where lowcase(strip(b.core)) = "y";
```

## Step 6: Merge ADSL to parent dataset
```sas
proc sort data = adam.adsl out = adsl;
    by usubjid;
run;

proc sort data = &inds.;
    by usubjid;
run;

data &outds.;
    merge &inds.(in = ina %if %symexist(drop_vars) %then drop = &drop_vars.;)
          adsl(in = inb keep = usubjid &core_vars.);
    by usubjid;
    if ina and inb;
run;
```

## Step 7: Delete/keep temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
    proc datasets lib = work nolist;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to the code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xtcore.sas)