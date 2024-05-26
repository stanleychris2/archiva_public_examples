# Calculate SDTM --DY variable from --DTC dates

## Steps:
1. Define macro variables and flags

```sas
%macro xcdtcdy(dtcdate /*Required. Name of character --DTC variable to calculate --DY for.*/,
               refdate /*Required. Reference character date.*/);
    %local end_xcdtcdy params_to_check i param_name param_value;

    %*Flag for premature macro termination;
    %let end_xcdtcdy = N;

    %*Define required parameter names that should be checked whether they are empty;
    %let params_to_check = dtcdate refdate;

    %*Macro parameter checks;
    %*Iterate through parameters;
    %do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
        %*Sub-select macro name;
        %let param_name = %scan(&params_to_check., &i., %str( ));

        %*Sub-select macro value;
        %let param_value = &%scan(&params_to_check., &i., %str( ));

        %*Check whether required parameters are empty;
        %if %length(&param_value.) = 0 %then %do;
            %put %str(ERR)%str(OR: xcdtcdy.sas - &param_name. is required parameter and should not be NULL);
            %let end_xcdtcdy = Y;
        %end;
    %end;

    %*Stop macro if one of the parameters broke the requirements;
    %if &end_xcdtcdy. = Y %then %goto endmac;
    
%mend xcdtcdy;
```

2. Check for required parameters

```sas
%*Define required parameter names that should be checked whether they are empty;
%let params_to_check = dtcdate refdate;

%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %*Sub-select macro name;
    %let param_name = %scan(&params_to_check., &i., %str( ));

    %*Sub-select macro value;
    %let param_value = &%scan(&params_to_check., &i., %str( ));

    %*Check whether required parameters are empty;
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xcdtcdy.sas - &param_name. is required parameter and should not be NULL);
        %let end_xcdtcdy = Y;
    %end;
%end;
```

3. Check for invalid characters in dates

```sas
%*Check if dates have invalid characters;
if prxmatch("/[a-zA-SU-Z]/", &dtcdate.) or prxmatch("/[a-zA-SU-Z]/", &refdate.) then put "ERR" "OR: xcdtcdy.sas - invalid argument value: dtcdate=" &dtcdate. "refdate=" &refdate.;
else if length(&dtcdate.) >= 10 and length(&refdate.) >= 10 then do;
    %*Calculate  --DY variables;
    %upcase(%substr(%cmpres(&dtcdate.), 1, %length(%cmpres(&dtcdate.)) - 3))DY = input(substr(&dtcdate., 1 , 10), e8601da.) - input(substr(&refdate., 1, 10), e8601da.) + (input(substr(&dtcdate., 1, 10), e8601da.) >= input(substr(&refdate., 1, 10), e8601da.));
end;
```

4. Calculate --DY variable

```sas
%*Calculate  --DY variables;
%upcase(%substr(%cmpres(&dtcdate.), 1, %length(%cmpres(&dtcdate.)) - 3))DY = input(substr(&dtcdate., 1 , 10), e8601da.) - input(substr(&refdate., 1, 10), e8601da.) + (input(substr(&dtcdate., 1, 10), e8601da.) >= input(substr(&refdate., 1, 10), e8601da.));
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xcdtcdy.sas)