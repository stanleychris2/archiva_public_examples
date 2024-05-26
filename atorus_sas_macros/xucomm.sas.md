# CO Domain Record Creation Macro

## Steps:
1. Check required parameters for emptiness
```sas
%let params_to_check = inds invar;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( )).;
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xucomm.sas - &param_name. is required parameter and should not be NULL);
        %let end_xucomm = Y;
    %end;
%end;
```

2. Resolve domain variable
```sas
%if %symexist(domain) = 0 %then %do;
    %put %str(ERR)%str(OR: xucomm.sas - please resolve variable %nrstr(&domain) with domain name (example: DM) before macro call);
    %let end_xucomm = Y;
%end;
```

3. Handle idvar parameter based on domain
```sas
%else %if &idvar. ^= %then %do;
    %if %lowcase(&domain.) = dm %then %do;
        %put %str(WAR)%str(NING: xucomm.sas - idvar parameter is ignored if %nrstr(&domain) = DM. Please leave idvar empty to suppress that message);
        %let idv = ;
    %end;
    %else %do;
        %let idv = &idvar.;
    %end;
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

6. Split input dataset
```sas
%xusplit(&inds., &invar., outds=__tmp_xucomm_0, prefix=__comm, debug=&debug.)
```

7. Create temporary dataset with domain records
```sas
data __tmp_&sysmacroname._1;
    set EMPTY_CO
        __tmp_&sysmacroname._0(rename = (domain = __tmp_domain &invar. = __comm) where = (not missing(__comm)));
run;
```

8. Determine number of COVALx variables
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

10. Create IDVAR/IDVARVAL based on type
```sas
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
```  

For more details, please refer to the [code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xucomm.sas) in the repository.