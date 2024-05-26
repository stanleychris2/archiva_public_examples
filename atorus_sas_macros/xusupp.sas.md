# Macro for Creating and Adding Variables to SUPP-- Domain

## Steps:
1. Check required parameters for emptiness
2. Resolve domain variable
3. Handle idvar parameter based on domain
4. Sort input dataset
5. Transpose dataset
6. Determine variable type
7. Check for _label_ existence
8. Create SUPP-- domain
9. Append records to SUPP--
10. Sort SUPP--
11. Delete temporary datasets if debug=N

## Relevant Code Snippets:

### Check required parameters for emptiness
```sas
%let params_to_check = inds invar;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( )).;

    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xusupp.sas - &param_name. is required parameter and should not be NULL);
        %let end_xusupp = Y;
    %end;
%end;
```

### Resolve domain variable
```sas
%if %symexist(domain) = 0 %then %do;
    %put %str(ERR)%str(OR: xusupp.sas - please resolve variable %nrstr(&domain) with domain name (example: DM) before macro call);
    %let end_xusupp = Y;
%end;
```

### Handle idvar parameter based on domain
```sas
%if &idvar. ^= %then %do;
    %*If &domain. = dm and idvar is specified then put log message;
    %if %lowcase(&domain.) = dm %then %do;
        %put %str(WAR)%str(NING: xusupp.sas - idvar parameter is ignored if %nrstr(&domain) = DM. Please leave idvar empty to suppress that message);
        %let idv = ;
    %end;
    %*Else if &domain. ^= dm and idvar is specified then use idvar;
    %else %do;
        %let idv = &idvar.;
    %end;
%end;
```

### Sort input dataset
```sas
proc sort data = &inds. out = __tmp_&sysmacroname._1;
    by studyid domain usubjid &idv.;
run;
```

### Transpose dataset
```sas
proc transpose data = __tmp_&sysmacroname._1(keep = studyid domain usubjid &idv. &invar.) out = __tmp_&sysmacroname._2;
    by studyid domain usubjid &idv.;
    var &invar.;
run;
```

### Determine variable type
```sas
%if &idv. ^= %then %do;
    data _null_;
        set __tmp_&sysmacroname._2;

        call symputx("vtype", vtype(&idv.));
    run;
%end;
```

### Check for _label_ existence
```sas
data _null_;
    dset = open("__tmp_&sysmacroname._2");
    call symputx("labchk", varnum(dset, "_LABEL_"));
run;
```

### Create SUPP-- domain
```sas
data __tmp_&sysmacroname._3(drop = domain _: col1 &idv.);
    set EMPTY_SUPP&domain.
        __tmp_&sysmacroname._2(where = (not missing(col1)));

        if not missing(domain) then rdomain = strip(domain);

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

        if not missing(_name_) then qnam = upcase(strip(_name_));

        %*If qlabel parameter specified, then use that value for QLABEL variable;
        %if &qlabel. ^= %then %do;
            qlabel = "&qlabel.";
        %end;
        %*Else if variable _label_ exists, then use it;
        %else %if &labchk. > 0 %then %do;
            if not missing(_label_) then qlabel = strip(_label_);
        %end;
        %*Else create a log message;
        %else %do;
            %put %str(WAR)%str(NING: xusupp.sas - QLABEL is set to NULL. Either assign a label for %upcase(&invar.) or specify label in qlabel parameter);
        %end;

        if not missing(col1) then qval = strip(col1);
        qorig = "&qorig.";
        qeval = "&qeval.";
run;
```

### Append records to SUPP--
```sas
proc append base = supp&domain. data = __tmp_&sysmacroname._3 force;
run;
```

### Sort SUPP--
```sas
proc sort data = supp&domain. nodupkey;
    by _all_;
run;
```

### Delete temporary datasets if debug=N
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails library = work;
        delete __tmp_&sysmacroname.:;
    run;
%end;
``` 

[Link to the full code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xusupp.sas)