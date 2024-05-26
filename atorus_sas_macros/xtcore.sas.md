# Macro for Adding ADSL Core Variables

## Steps:
1. Define macro variables and parameters
2. Check for required parameters
3. Check for metadata file existence
4. Load metadata information
5. Sort dataset by specified variable order
6. Define core variables and input dataset variable names
7. Find intersection between core variables and input dataset variables
8. Merge ADSL to parent dataset
9. Delete temporary datasets for debug purposes

## Relevant Code Snippets:
```sas
%macro xtcore(inds /*Required. Name of the parent dataset for adding ADSL core variables.*/,
              outds=&inds. /*Default: &inds. Name of the output dataset.*/,
              filename=ADAM_spec_Variables.csv /*Default: ADaM_spec_Codelist.csv. Indicates source file name.*/,
              filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.specs /*Default: final/specs. Indicates path to source file.*/,
              debug=N /*Default: N. If Y then deletes temporary datasets.*/);
```

```sas
%let params_to_check = inds outds filename filepath;
```

```sas
%if %length(&param_value.) = 0 %then %do;
    %put %str(ERR)%str(OR: xtcore.sas - &param_name. is required parameter and should not be NULL);
    %let end_xtcore = Y;
%end;
```

```sas
filename metadata "&filepath.&II.&filename." termstr = CRLF;
```

```sas
proc import out = __tmp_&sysmacroname._var_0 datafile = metadata dbms = csv replace;
    guessingrows = max;
    delimiter = "$";
run;
```

```sas
proc sort data = __tmp_&sysmacroname._var_0 (where = (strip(lowcase(strip(dataset))) = "adsl"));
    by order;
run;
```

```sas
select variable into :core_vars separated by " " from __tmp_&sysmacroname._var_0 where lowcase(strip(core)) = "y";
```

```sas
create table __tmp_&sysmacroname._inds_vars as 
    select name 
    from dictionary.columns
    where lowcase(memname) = lowcase("&inds.") and lowcase(libname) = "work" and lowcase(name) not in ("studyid", "usubjid");
```

```sas
create table __tmp_&sysmacroname._intersect_vars as
        select a.name 
        from __tmp_&sysmacroname._inds_vars as a left join __tmp_&sysmacroname._var_0 as b
        on lowcase(a.name) = lowcase(b.variable)
        where lowcase(strip(b.core)) = "y";
```

```sas
select name into :drop_vars separated by " " from __tmp_&sysmacroname._intersect_vars;
```

```sas
proc sort data = adam.adsl out = adsl;
    by usubjid;
run;
```

```sas
data &outds.;
    merge &inds.(in = ina %if %symexist(drop_vars) %then drop = &drop_vars.;)
          adsl(in = inb keep = usubjid &core_vars.);
    by usubjid;
    if ina and inb;
run;
```

```sas
%if &debug. = N %then %do;
    proc datasets lib = work nolist;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

For the full code, please refer to the [xtcore.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xtcore.sas) file.