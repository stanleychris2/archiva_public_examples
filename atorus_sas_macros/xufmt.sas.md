# SAS Format Creation from Codelist Metadata

## Steps:
1. Check required parameters for empty values
```sas
%let params_to_check = fmt filename filepath;
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
    %let param_name = %scan(&params_to_check., &i., %str( ));
    %let param_value = &%scan(&params_to_check., &i., %str( ));
    %if %length(&param_value.) = 0 %then %do;
        %put %str(ERR)%str(OR: xufmt.sas - &param_name. is required parameter and should not be NULL);
        %let end_xufmt = Y;
    %end;
%end;
```

2. Check if codelist metadata file is present
```sas
%if %sysfunc(fileexist(&filepath.&II.&filename.)) = 0 %then %do;
    %put %str(ERR)%str(OR: xufmt.sas - input &filename. file was not found in &filepath.);
    %let end_xufmt = Y;
%end;
```

3. Import codelist metafile
```sas
filename codelist "&filepath.&II.&filename." termstr = CRLF;

proc import out = __tmp_&sysmacroname._cl_0 datafile = codelist dbms = csv replace;
    delimiter = "$";
    getnames = yes;
    guessingrows = max;
run;
```

4. Create formats from codelists
```sas
data __tmp_&sysmacroname._cl_2;
    set __tmp_&sysmacroname._cl_1;
    
    length fmtname $31 type $20 start label $200.;

    %*Set hlo to avoid unwanted range interpretations by proc format;
    hlo = "";
    if not missing(order) then do;
        if strip(term) ^= "" then do;
            %*One format for order to term;
            fmtname = strip(id)||"OT";
            start = strip(put(order, ?? best.));
            label = strip(term);
            type = "n";
            output;
            %*One format for term to order;
            fmtname = strip(id)||"TO";
            type = "i";
            start = strip(term);
            label = strip(put(order, ?? best.));
            output;
        end;
        if strip(decoded_value) ^= "" then do;
            %*One format for order to decod;
            fmtname = strip(id)||"OD";
            start = strip(put(order, ?? best.));
            label = strip(decoded_value);
            type = "n";
            output;
            %*One format for decod to order;
            fmtname = strip(id)||"DO";
            type = "i";
            start = strip(decoded_value);
            label = strip(put(order, ?? best.));
            output;
        end;
    end;

    if strip(term) ^= "" and strip(decoded_value) ^= "" then do;
        %*One format for decoded_value to term;
        fmtname = strip(id)||"DT";
        type = "c";
        start = strip(decoded_value);
        label = strip(term);
        output;
        %*One format for term to decoded_value;
        fmtname = strip(id)||"TD";
        type = "c";
        start = strip(term);
        label = strip(decoded_value);
        output;
    end;
    
run;
```

5. Sort and remove non-unique formats
```sas
proc sort data = __tmp_&sysmacroname._cl_2 out = __tmp_&sysmacroname._cl_3 (keep = fmtname start type label hlo) nodupkey;
    by fmtname start type label;
run;

proc sql noprint;
    create table __tmp_&sysmacroname._cl_4 as 
        select distinct fmtname, type, start, label, hlo, count(start) as n 
        from __tmp_&sysmacroname._cl_3 
        group by fmtname, start;

    create table __tmp_&sysmacroname._cl as 
        select distinct fmtname, type, start, label, hlo, max(n) as unique from __tmp_&sysmacroname._cl_4 
        group by fmtname 
        order by fmtname, type, start, label;
quit;

proc format cntlin = __tmp_&sysmacroname._cl (where = (unique = 1)); 
quit;
```

6. Delete temporary datasets if debug flag is set to N
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete __tmp_&sysmacroname._:;
    run;
%end;
``` 

Code Link: [xufmt.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xufmt.sas)