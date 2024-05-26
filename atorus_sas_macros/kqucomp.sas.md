# Dataset Comparison Macro

### Step 1: Define macro parameters with default values.
```sas
%macro kqucomp(base=&domain. /*Default: &domain. Name of the dataset which is used as base in compare procedure.*/,
               qc=qc /*Default: qc. Prefix of the qc dataset name.*/,
               id=&&&domain.sortstring /*Default: &&&domain.sortstring. List of id variables.*/,
               prod_lib=sdtm /*Default: sdtm. Name of the library from which base dataset is uploaded.*/,
               qc_lib=work /*Default: work. Name of the library from which qc dataset is uploaded.*/,
               crit= /*Optional. Specify the criterion for the compare procedure.*/,
               supp=N /*Default: N. Compare the SUPP-- dataset, or not.*/,
               com=N /*Default: N. Compare the comments dataset, or not.*/);
```

### Step 2: Check for required parameters that should not be empty.
```sas
%let params_to_check = base prod_lib qc_lib;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( ));

    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: kqucomp.sas - &param_name. is required parameter and should not be NULL);
        %let end_kqucomp = Y;
    %end;
%end;
```

### Step 3: Perform proc contents on base dataset.
```sas
proc contents data = &prod_lib..&base.;
run;
```

### Step 4: Perform compare procedure on base and qc datasets.
```sas
proc compare data = &prod_lib..&base. comp = &qc_lib..&qc.&domain. listall %if &crit ^=  %then %do; criterion = &crit. %end;;
    %if &id. ^= %then %do; id &id.; %end;
run;
```

### Step 5: Compare SUPP-- dataset if needed.
```sas
%if %lowcase(&supp.) = y %then %do;
    %local nobs1_supp nobs2_supp;

    proc sql noprint;
        select count(*) into:nobs1_supp
            from &prod_lib..supp&base.;

        select count(*) into:nobs2_supp
            from &qc_lib..&qc.supp&domain.;
    quit;

    %if &nobs1_supp. ne 0 or &nobs2_supp. ne 0 %then %do;
        proc contents data = &prod_lib..supp&base.;
        run;

        proc compare data = &prod_lib..supp&base. comp = &qc_lib..&qc.supp&domain. listall; 
            id studyid usubjid idvar idvarval qnam;
        run;
    %end;
%end;
```

### Step 6: Compare comments dataset if needed.
```sas
%if %lowcase(&com.) = y %then %do;
    %local nobs1_comm nobs2_comm;

    proc sql noprint;
        select count(*) into:nobs1_comm
            from &prod_lib..&base._comm;

        select count(*) into:nobs2_comm
            from &qc_lib..&qc.&base._comm;
    quit;

    %if &nobs1_comm. ne 0 or &nobs2_comm. ne 0 %then %do;
        proc contents data = &prod_lib..&base._comm;
        run;

        proc compare data = &prod_lib..&base._comm comp = &qc_lib..&qc.&base._comm listall; 
            id studyid usubjid idvar idvarval ;
        run;
    %end;
%end;
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kqucomp.sas)