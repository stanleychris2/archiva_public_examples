# Text Variable Splitter Macro

## Steps:
1. Define macro parameters and required checks
2. Create temporary order variable
3. Split text into sub-variables based on length
4. Transpose data to create sub-variables
5. Determine number of sub-variables to create
6. Set length of input variable
7. Merge and create output dataset
8. Delete temporary datasets if debug flag is set to N

### Relevant Code Snippets:
```sas
%macro xusplit(inds /*Required. Name of the input dataset.*/,
               invar /*Required. Name of the input variable to split.*/,
               outds=&inds. /*Default: &outds. Name of the output dataset.*/,
               prefix=&invar. /*Default: &invar. Prefix for the output variables names.*/,
               len=200 /*Default: 200. Length of the output variables.*/,
               debug=N /*Default: N. If N then delete temporary datasets.*/);
```

```sas
%*Create temporary order variable;
data __tmp_&sysmacroname._1;
    set &inds.;

    __tmp_ord = _n_;
run;
```

```sas
%*If text length is more than &len. then split it to sub-variables with meaningful text less than &len.;
data __tmp_&sysmacroname._2;
    set __tmp_&sysmacroname._1(where = (not missing(&invar.)));
    length __tmp_word __tmp_nystr $1000;

    &invar. = compbl(strip(&invar.));

    do i = 1 to countw(&invar., " ");
        __tmp_word = scan(&invar., i, " ");
        if length(__tmp_nystr) + length(__tmp_word) + 1 > &len. then do;
            output;
            __tmp_nystr = __tmp_word;
        end;
        else __tmp_nystr = catx(" ", __tmp_nystr, scan(&invar., i, " "));
    end;
    if __tmp_nystr ^= "" then output;
run;
```

```sas
%*Determine how many sub-variables should be created;
proc sql noprint;    
    select strip(put(count(*), best.)) into: nvars from sashelp.vcolumn
    where lowcase(libname) = "work" and lowcase(memname) = lowcase("__tmp_&sysmacroname._3") and index(lowcase(name), "__tmp_c");
quit;
```

```sas
%*Set the length of the input variable same as length defined in &len.;
proc sql;
    alter table __tmp_&sysmacroname._1 modify &invar. character(&len.);
quit;
```

For more details, refer to the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xusplit.sas).