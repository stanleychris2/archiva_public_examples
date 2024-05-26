# Number Alignment Macro for PDF and RTF

## Step 1: Define local macro variables
```sas
%macro xdalign(invar /*Required. Input variable name.*/,
               type=RTF, /*Default: RTF. Type of the output, extension.*/
               escapechar=$ /*Default: $. Escape character.*/,
               len=6 /*Default: 6. Length of the whole variable in the output, including indentation.*/);
```

## Step 2: Check required parameters
```sas
%let params_to_check = invar type len;
```

## Step 3: Check if parameters are empty
```sas
%if %length(&param_value.) = 0 %then %do;
    %put %str(ERR)%str(OR: xdalign.sas - &param_name. is required parameter and should not be NULL);
    %let end_xdalign = Y;
%end;
```

## Step 4: Check if output type is PDF or RTF
```sas
%if %lowcase(&type.) = pdf %then %do;
    if length(scan(&invar., 1, ".")) = 1 then &invar. = "&escapechar.{nbspace %eval(&len. - 1)}" || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 2 then &invar. = "&escapechar.{nbspace %eval(&len. - 2)}" || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 3 then &invar. = "&escapechar.{nbspace %eval(&len. - 3)}" || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 4 then &invar. = "&escapechar.{nbspace %eval(&len. - 4)}" || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 5 then &invar. = "&escapechar.{nbspace %eval(&len. - 5)}" || strip(&invar.);
    else if length(scan(&invar., 1, ".")) > 5 then put "WARN" "ING: xdalign.sas - interger part is too long. Need to update macro conditions";
%end;
```

## Step 5: Add spaces based on output type and length
```sas
%else %if %lowcase(&type.) = rtf %then %do;
    if length(scan(&invar., 1, ".")) = 1 then &invar. = repeat("&escapechar. ", %eval(&len. - 2)) || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 2 then &invar. = repeat("&escapechar. ", %eval(&len. - 3)) || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 3 then &invar. = repeat("&escapechar. ", %eval(&len. - 4)) || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 4 then &invar. = repeat("&escapechar. ", %eval(&len. - 5)) || strip(&invar.);
    else if length(scan(&invar., 1, ".")) = 5 then &invar. = repeat("&escapechar. ", %eval(&len. - 6)) || strip(&invar.);
    else if length(scan(&invar., 1, ".")) > 5 then put "WARN" "ING: xdalign.sas - interger part is too long. Need to update macro conditions";
%end;
``` 

[Link to the code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xdalign.sas)