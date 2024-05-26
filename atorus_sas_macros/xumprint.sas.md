# SAS Macro for Log and Lst File Management

## Steps:
1. Define local variables
2. Check required parameters
3. Delete macros from SAS cache if route=YES
4. Call xuprogpath macro
5. Save log and lst files if route=YES
6. Display log and lst files if route=NO
7. Convert notes of interest into warnings

### Relevant Code Snippets:
1. **Define local variables**
```sas
%macro xumprint(route=NO /*Default: NO. If NO then uploads log file back to SAS EG. Specify YES to save output lst and log files.*/);
    %local end_xumprint params_to_check i param_name param_value;
    %*Flag for premature macro termination;
    %let end_xumprint = N;
    %*Define required parameter names that should be checked whether they are empty;
    %let params_to_check = route;
```

2. **Check required parameters**
```sas
%*Macro parameter checks;
%*Iterate through parameters;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %*Sub-select macro name;
    %let param_name = %scan(&params_to_check., &i., %str( ));
    
    %*Sub-select macro value;
    %let param_value = &%scan(&params_to_check., &i., %str( )).;
    
    %*Check whether required parameters are empty;
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xumprint.sas - &param_name. is required parameter and should not be NULL);
        %let end_xumprint = Y;
    %end;
    %else %if %lowcase(&param_value.) ^= yes and %lowcase(&param_value.) ^= no %then %do;
        %put %str(ERR)%str(OR: xumprint.sas - parameter &param_name. should be YES or NO);
        %let end_xumprint = Y;
    %end;
%end;
```

3. **Delete macros from SAS cache if route=YES**
```sas
%if %lowcase(&route.) = yes %then %do;
    %local macrodel;
    
    %*Delete macro which were already resolved in SAS session;
    %*Macro name prefixes are standardized according to BM-00-W-03 WI;
    proc sql noprint;
        select strip(objname) into: macrodel separated by " " from sashelp.vcatalg
        where upcase(libname) = "WORK" and index(upcase(memname), "SASMAC") and upcase(objname) ^= "XUMPRINT" and substr(upcase(objname), 1, 2) in ("XC", "XD", "XG", "XM", "XQ", "XS", "XT", "XU",
                            "KC", "KD", "KG", "KM", "KQ", "KS", "KT", "KU",
                            "JC", "JD", "JG", "JM", "JQ", "JS", "JT", "JU");
    quit;
    
    %if &macrodel. ^= %then %do;
        %put %str(NO)%str(TE: xumprint.sas - &macrodel. macros were deleted from SAS cache before program execution);
        %do i = 1 %to %sysfunc(countw(&macrodel., %str( )));
            %sysmacdelete %scan(&macrodel., &i., %str( ));
        %end;
    %end;
``` 

4. **Call xuprogpath macro**
```sas
%*Call xuprogpath to create task and program related globals.;
%xuprogpath;
```

5. **Save log and lst files if route=YES**
```sas
%*If route=YES then .log and .lst files will be saved to task-specific directories.;
%if %symexist(__p_name) %then %do;
    %if %lowcase(&route.) = yes %then %do;
        %if %length(&__p_name.) ^= 0 and %length(&__log_path.) ^= 0 and %length(&__lst_path.) ^= 0 %then %do;
            proc printto print = "&__lst_path.&II.&__p_name..lst" log = "&__log_path.&II.&__p_name..log" new;
            run;
``` 

6. **Display log and lst files if route=NO**
```sas
%*If route=NO then .log and .lst files will be displayed in SAS EG.;
%else %do;
    %local __origopts;
    
    %*Turning off macro options;
    %let __origopts = %sysfunc(getoption(mprint)) %sysfunc(getoption(symbolgen)) %sysfunc(getoption(mlogic));
    options nomprint nosymbolgen nomlogic;
    
    proc printto log = log print=print;
    run;
    
    %if %sysfunc(fileexist("&__log_path.&II.&__p_name..log")) %then %do;
        data _null_;
            length line $32767;
            
            infile "&__log_path.&II.&__p_name..log" length = __length;
            input @1 line $varying32767. __length;
            
            %*Converting NOTE xxx-xxx to NOTE:;
            if kscan(line, 1, " ") in ("ERROR", "WARNING", "NOTE") then do;
                length code $10;
                code = kscan(line, 2, " ");
                if kindex(code, ":") = klength(code) and not kverify(kstrip(code), "1234567890-:") then line = kscan(line, 1, " ") || ":" || ksubstr(line, kindex(line, ":") + 1);
            end;
            
            %*Converting NOTE of interest into warnings;
            if ksubstr(line, 1, 6) = "NOTE: " then do;
                if kindex(klowcase(line), "uninitialized") or
                   kindex(klowcase(line), "invalid") or
                   kindex(klowcase(line), "division by zero") or
                   kindex(klowcase(line), "some graph legends have been dropped due to size constraints") or
                       kindex(klowcase(line), "at least one w.d format was too small") or
                       kindex(klowcase(line), "merge statement has more than one data set with repeats of by values") or
                       kindex(klowcase(line), "missing values were generated as a result of performing an operation on missing values") or
                       kindex(klowcase(line), "was not found or could not be loaded") or
                       kindex(klowcase(line), "have been converted to") then line = "W" || "ARNING: " || ksubstr(line, 7);
            end;
            
            %*Print modified log;
            put line;
        run;
    %end;
    
    %*Restore original options;
    options &__origopts.;
%end;
``` 

7. **Convert notes of interest into warnings**
```sas
%*Converting NOTE of interest into warnings;
if ksubstr(line, 1, 6) = "NOTE: " then do;
    if kindex(klowcase(line), "uninitialized") or
       kindex(klowcase(line), "invalid") or
       kindex(klowcase(line), "division by zero") or
       kindex(klowcase(line), "some graph legends have been dropped due to size constraints") or
       kindex(klowcase(line), "at least one w.d format was too small") or
       kindex(klowcase(line), "merge statement has more than one data set with repeats of by values") or
       kindex(klowcase(line), "missing values were generated as a result of performing an operation on missing values") or
       kindex(klowcase(line), "was not found or could not be loaded") or
       kindex(klowcase(line), "have been converted to") then line = "W" || "ARNING: " || ksubstr(line, 7);
end;
``` 

For more details, you can refer to the [SAS Macro Code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xumprint.sas).