# Macro for Creating CO Domain Records

## Steps:
1. Check for required parameters and their values
```sas
%macro xucomm(inds /*Required. Name of the input dataset to get comments from.*/,
              invar /*Required. Name of the input variable to get comment from.*/,
              idvar= /*Optional. Name of the id variable.*/,
              qc=N /*Default: N. Specify Y if macro is used on QC side.*/,
              debug=N /*Default: N. If N then delete temporary datasets.*/);
```

2. Resolve domain variable
```sas
%if %symexist(domain) = 0 %then %do;
    %put %str(ERR)%str(OR: xucomm.sas - please resolve variable %nrstr(&domain) with domain name (example: DM) before macro call);
    %let end_xucomm = Y;
%end;
```

3. Handle idvar parameter based on domain value
```sas
%if &idvar. ^= %then %do;
    %*If &domain. = dm and idvar is specified then put log message;
    %if %lowcase(&domain.) = dm %then %do;
        %put %str(WAR)%str(NING: xucomm.sas - idvar parameter is ignored if %nrstr(&domain) = DM. Please leave idvar empty to suppress that message);
        %let idv = ;
    %end;
```

4. Get metadata for CO
```sas
%xtmeta(CO, qc=&qc., debug=&debug.);
```

5. Order CO metadata
```sas
%xtorder(CO, debug=&debug.);
```

6. Split input dataset based on input variable
```sas
%xusplit(&inds., &invar., outds=__tmp_xucomm_0, prefix=__comm, debug=&debug.)
```

7. Create temporary dataset with necessary variables
```sas
data __tmp_&sysmacroname._1;
    set EMPTY_CO
        __tmp_&sysmacroname._0(rename = (domain = __tmp_domain &invar. = __comm) where = (not missing(__comm)));
run;
```

8. Determine number of COVALx variables to create
```sas
proc sql noprint;
    select strip(put(count(*), best.)) into: nvars from sashelp.vcolumn
    where lowcase(libname) = "work" and lowcase(memname) = lowcase("__tmp_&sysmacroname._1") and index(lowcase(name), "__comm");
quit;
```

9. Determine type of input variable
```sas
%if &idv. ^= %then %do;
    data _null_;
        set __tmp_&sysmacroname._1;

        call symputx("vtype", vtype(&idv.));
    run;
%end;
```

10. Create final dataset with necessary variables
```sas
data __tmp_&sysmacroname._2;
    set __tmp_&sysmacroname._1;

    domain = "CO";
    if not missing(__tmp_domain) then rdomain = strip(__tmp_domain);

    %*If &idv. is not empty then create IDVAR/IDVARVAL based on the type of &idv.;
    %if &idv. ^= %then %do;
        idvar = "%upcase(&idv.)";
        %if &vtype. = N %then %do;
            idvarval = strip(put(&idv., best.));
        %end;
        %if &vtype. = C %then %do;
            idvarval = strip(&idv.);
        %end;
    %end;

    array __tmp_cols [*] $ __comm %if &nvars. > 1 %then %do; __comm1-__comm%eval(&nvars. - 1) %end;;
    array __tmp_coval [*] $ coval %if &nvars. > 1 %then %do; coval1-coval%eval(&nvars. - 1) %end;;

    do i = 1 to &nvars.;
        __tmp_coval[i] = strip(__tmp_cols[i]);
    end;
run;
```

11. Check for missing COVALx variables in spec
```sas
%*Check to determine if any of COVALx is in dataset but not in spec;
proc sql noprint;
    select upcase(name) into: dscheck separated by " " from sashelp.vcolumn
    where lowcase(libname) = "work" and lowcase(memname) = lowcase("__tmp_&sysmacroname._2") and index(lowcase(name), "coval") order by name;

    select upcase(name) into: speccheck separated by " " from sashelp.vcolumn
    where lowcase(libname) = "work" and lowcase(memname) = lowcase("EMPTY_CO") and index(lowcase(name), "coval") order by name;
quit;
```

12. Compute COSEQ variable
```sas
data __tmp_&sysmacroname._3;
    set __tmp_&sysmacroname._2;
    by studyid rdomain usubjid idvar idvarval;

    retain __tmp_seq 0;

    if first.usubjid then __tmp_seq = 1;
    else __tmp_seq + 1;

    coseq = __tmp_seq;
run;
```

13. Create intermediate XX_comm dataset for QC purposes
```sas
proc sort data = __tmp_&sysmacroname._3(keep = &cokeepstring.) %if &qc. = N %then %do; out = sdtm.&domain._comm; %end;
                                                %else %do; out = qc&domain._comm; %end;
    by &cosortstring.;
run;
```

14. Delete/keep temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname.: EMPTY_CO;
    run;
%end;

%endmac:

%mend xucomm;
```

Code Link: [xucomm.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xucomm.sas)