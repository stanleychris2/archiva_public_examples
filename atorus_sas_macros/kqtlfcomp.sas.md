# TLF Dataset Comparison Macro

## Steps
1. Define macro parameters and required parameter names to check if they are empty.
```sas
%macro kqtlfcomp(output= /*Output name. The macro will look for that dataset name.*/,
                 prod_drop= /*Optional. Specifies which variables have to be dropped from the pruduction dataset.*/,
                 dropchecked=N /*Default: N. Specifies whether qcer dropped and checked any variables from production dataset.*/,
                 prod_lib=tfl /*Default: tfl. Library of the production output dataset.*/,
                 qc_lib=vtfl /*Default: vtfl. Library of the qc output dataset.*/,
                 qc_output= /*Optional. pecify when you want to manually select your dataset.*/,
                 where=1 /*Default: 1. Specify a where statement to facilitate investitations.*/,
                 ignspl=Y /*Default: Y. If Y then the comparison changes every split character to a space.*/,
                 chrspl=| /*Default: |. Split character.*/,
                 ignln=Y /*Default: Y. If Y then ignores variable lengths while comparing.*/,
                 ignspc=N /*Default: N. If Y then the comparison strips and compbl the results before comparing.*/,
                 ignlbl=Y /*Default: Y. If Y then the comparison removes the variable labels before comparing.*/,
                 ignfor=Y /*Default: Y. If Y then the comparison removes the variable formats before comparing.*/,
                 ignifor=Y /*Default: Y. If Y then the comparison removes the variable informats before comparing.*/,
                 compress=N /*Default: N. If Y then the comparison removes all spaces before comparing.*/,
                 crit= /*Optional. Specifies the criterion for the compare procedure.*/,
                 debug=N /*Default: N. If Y then keep temporary datasets to allow debugging.*/);
```

2. Check if required parameters are empty.
```sas
%let params_to_check = output prod_lib qc_lib qc_output;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( ));

    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: kqtlfcomp.sas - &param_name. is required parameter and should not be NULL);
        %let end_kqtlfcomp = Y;
    %end;
%end;
```

3. Check and validate Y/N parameters.
```sas
%let yncheck = dropchecked ignspl ignspc ignlbl ignfor ignifor compress debug ignln;

%do j = 1 %to %sysfunc(countc(&yncheck., %str( ))) + 1;
    %let yncheck1 = %scan(&yncheck., &j., %str( ));
    %let yncheck2 = &%scan(&yncheck., &j., %str( ));

    %if %upcase(&yncheck2.) ^= Y and %upcase(&yncheck2.) ^= N %then %do;
        %put %str(ERR)%str(OR: kqtlfcomp.sas - please use either Y or N for parameter &yncheck1. = &yncheck2.);
        %let end_kqtlfcomp = Y;
    %end;

    %if &end_kqtlfcomp = Y %then %goto endmac;
%end;
```

4. Retrieve datasets and perform comparisons.
```sas
%do a = 1 %to 2;
    %if &a. = 1 %then %let lib = &prod_lib.;
    %if &a. = 2 %then %let lib = &qc_lib.;

    %let nmemname = 0;

    %if &a. = 2  %then %do;
        %if "&qc_output." = "" %then %do;
            proc sql noprint;
                select count(distinct memname), memname, lowcase(memname), memlabel into: nmemname, : memname, : qc_output, : memlabel&a. from dictionary.tables
                    where libname = strip(upcase("&lib.")) and index(strip(upcase(reverse(memname))), strip(upcase(reverse("&output.")))) = 1;
            quit;
        %end;
        %else %do;
            %let nmemname = 1;
            %let memname = &qc_output.;
            %let qc_output = %sysfunc(lowcase(&qc_output.));
        %end;
    %end;

    %if &a. = 2 and &nmemname. > 1 %then %do;
        %put %sysfunc(compress(ERR OR:)) kqtlfcomp.sas - qc library &lib. did not contain a clear &output. dataset to take along;
        %goto endmac;
    %end;

    %if &a. = 1 %then %do;
        %if %sysfunc(exist(&lib..&output.)) = 0 %then %do;
            %put %sysfunc(compress(ERR OR:)) kqtlfcomp.sas - production file &lib..&output. did not exist;
            %goto endmac;
        %end;
    %end;
    %else %do;
        %if %sysfunc(exist(&lib..&memname.)) = 0 %then %do;
            %put %sysfunc(compress(ERR OR:)) kqtlfcomp.sas - qc file &lib..&output. did not exist;
            %goto endmac;
        %end;
    %end;
%end;
```

5. Compare dataset metadata and variable values.
```sas
%put %str(INF)%str(O: kqtlfcomp: start comparing &output. VS qc &qc_output.);

%*Check if there is something to compare;
%if &records1. = 0 or &records2. = 0 %then %do;
    %if &records1. = 0 and &records2. = 0 %then %put %str(IN)%str(FO: No records found for output &output. in both production and qc. Please check.);
    %else %do;
        %if &records1. = 0 %then %put %str(ERR)%str(OR: kqtlfcomp.sas - no filled dataset found in library &prod_lib. for domain &output., or selection resulted in no records);
        %if &records2. = 0 %then %put %str(ERR)%str(OR: kqtlfcomp.sas - no filled dataset found in library &qc_lib for domain &output., or selection resulted in no records);
    %end;
    %goto compile;
%end;

title "Production &output. VS qc &qc_output.";
proc compare data = prod_&output2. compare = qc_&output2.  out = _comp1 outdif outbase outcomp outnoeq %if &crit ^=  %then %do; criterion=&crit. %end;;
run;
title;
``` 

6. Check for differences and create output.
```sas
%global check_compare_rc;
%let check_compare_rc = &sysinfo.;

data _null_;
    if substr(reverse(put(&check_compare_rc., binary16.)), 1, 1) = '1' then put "WARNIN" "G: kqtlfcomp.sas - &output. " "Proc Compare: Data set labels differ.";
    if substr(reverse(put(&check_compare_rc., binary16.)), 2, 1) = '1' then put "WARNIN" "G: kqtlfcomp.sas - &output. " "Proc Compare: Data set types differ.";
    ...
run;

%local nobs;

data _null_;
    if 0 then set _comp1 nobs = comp_tlf_obs;
    call symput('nobs', compress(put(comp_tlf_obs, best.)));
    stop;
run;

data _comp2 (drop = difr baser compr);
    %if &nobs. = 0 %then %do;
        length _type_ $8;
        _type_ = "BASE";
        _obs_ = 1;
        output;
        length _type_ $8;
        _type_ = "COMPARE";
        _obs_ = 1;
        output;
        length _type_ $8;
        _type_ = "DIF";
        _obs_ = 1;
        output;
    %end;
    set _comp1 end = end;
    %if &nobs. = 0 %then %do;
        stop;
    %end;
    retain difr compr baser 0;
    ...
run;
```

7. Clean up temporary datasets if debug mode is off.
```sas
%if &debug. = N %then %do;
    proc datasets lib = work nolist nodetails;
        delete dif: _comp: _finding1: _finding2: _meta:;
    run;
%end;
``` 

For more details, you can check the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/kqtlfcomp.sas).