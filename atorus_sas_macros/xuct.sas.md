# CDISC SDTM Controlled Terminology and Codelists Validation

## Steps:
1. Check if required parameters are not empty
```sas
%macro xuct(inds /*Required. Input dataset name.*/,
            invar /*Required. Variable name that will be checked for CT.*/,
            codelist /*Required. Name of the codelist to be used.*/,
            subset=%str(1=1) /*Default: all observations. To be used when variable may have multiple codelists*/,
            filename=SDTM_spec_Codelists.csv /*Default: SDTM_spec_Codelists.csv. Defines source file to use.*/,
            filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.specs /*Default: final/specs. Indicates path to source file.*/,
            ctfile=SDTM Terminology.csv /*Default: SDTM Terminology.csv. Defines source CT file to be converted to misc._ct.sas7bdat, if _ct.sas7bdat not created yet.*/,
            ctpath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.final&II.raw&II.dict/*Default: final/raw/dict. Indicates path to CT csv file.*/,
            checkct=Y /*Default: Y. Defines whether variable values should checked for compliance with CDISC CT. If N, then values checked only for compliance with spec Codelist tab.*/, 
            debug=N /*Default: N. If Y then deletes temporary datasets.*/);
```

2. Check if codelist metadata file is present
```sas
%if %sysfunc(fileexist(&filepath.&II.&filename.)) = 0 %then %do;
    %put %str(ERR)%str(OR: xuct.sas - input &filename. file was not found in &filepath.);
    %let end_xuct = Y;
%end;
```

3. Check if _ct.sas7bdat file is present
```sas
%if %sysfunc(exist(misc._ct)) = 0 %then %do;
    %put %str(NO)%str(TE: xuct.sas - misc._ct data set does not exist. Trying to create it.);
```

4. Import spec codelist metafile
```sas
filename codelist "&filepath.&II.&filename." termstr = CRLF;

proc import out=_&sysmacroname._cl_all datafile = codelist dbms = csv replace;
    delimiter = "$";
    datarow = 2;
    getnames = yes;
    guessingrows = max;
run;
```

5. Subset spec codelists for target variable
```sas
data _&sysmacroname._cl_&cl.(keep = ID Name "NCI Codelist Code"n "Data Type"n Term "NCI Term Code"n  "Decoded Value"n
            rename = (ID = cl_id Name = cl_name "NCI Codelist Code"n = cl_nci_codelist_code "Data Type"n = cl_data_type Term = cl_term "NCI Term Code"n = cl_nci_term_code "Decoded Value"n = cl_decoded_value));
    set _&sysmacroname._cl_all;
    if lowcase(ID) = lowcase("&codelist.");
```

6. Check for non-printable characters in spec codelists
```sas
data _null_;
    set _&sysmacroname._cl_&cl.;
    array charvars _character_;
    length _tmp_npchars $161;
    retain _tmp_npchars;
    if _N_ = 1 then do; 
        do _tmp_npchar_i = 0 to 31, 127 to 255;
            if _tmp_npchar_i = 0 then _tmp_npchars = byte(_tmp_npchar_i);
            else _tmp_npchars = trim(_tmp_npchars) || byte(_tmp_npchar_i);
        end;
    end;
    do over charvars;
        if indexc(charvars, _tmp_npchars) then put "WAR" "NING: xuct.sas - Non printable/special character found in %upcase(&codelist.) codelist: " charvars=;
    end;
run;
```

7. Merge spec CT to dataset
```sas
proc sql noprint undo_policy=none;
    create table _&sysmacroname._&cl. as 
        select * from &inds. left join _&sysmacroname._cl_&cl. 
        on %if &data_type = numeric %then %do; &inds..&invar. = input(_&sysmacroname._cl_&cl..cl_term, best.); %end;
                %else %do; &inds..&invar. = _&sysmacroname._cl_&cl..cl_term; %end;
quit;
```

8. Subset CDISC CT for target variable
```sas
data _&sysmacroname._ct_&cl. (keep = var4);
    set misc._ct;
    if lowcase(var5) = lowcase("&ct_cl.");
run;
```

9. Merge CDISC CT list to dataset
```sas
data _&sysmacroname._ct_&cl.(keep = var1 var2 var4 var5 var6 ct_nci_preferred_term
                    rename = (var1 = ct_code var2 = ct_codelist_code var4 = ct_codelist_name var5 = ct_cdisc_submission_value var6 = ct_cdisc_synonym));
    merge misc._ct _&sysmacroname._ct_&cl.(in = b);
    by var4;
    if b;
    if not missing(var2);
    length ct_nci_preferred_term $256;
    ct_nci_preferred_term = strip(var8);
run;
```

10. Find synonyms from CT CDISC Synonym column
```sas
data _&sysmacroname._&ct_cl._syn;
    set _&sysmacroname._ct_&cl.;
    length ct_cdisc_synonym_term $200;
    if not missing(ct_cdisc_synonym) and index(ct_cdisc_synonym, ";") ^= 0 then do;
        do _tmp_i = 1 to count(ct_cdisc_synonym, ";") + 1;
            ct_cdisc_synonym_term = strip(scan(ct_cdisc_synonym, _tmp_i, ";"));
            output;
        end;
    end;
    else if not missing(ct_cdisc_synonym) then do;
        ct_cdisc_synonym_term = strip(ct_cdisc_synonym);
        output;
    end;
    else if not missing(ct_cdisc_submission_value) then do;
        ct_cdisc_synonym_term = strip(ct_cdisc_submission_value);
        output;
    end;
run;
```

11. Prepare report and put messages to log file
```sas
data _&sysmacroname._&cl._report;
    set _&sysmacroname._&cl.;
    length ctmissfl clmissfl $1;
    if missing(ctmissfl) then call missing(ctmissfl);
    if missing(clmissfl) then call missing(clmissfl);
    if missing(ct_cdisc_submission_value) then call missing(ct_cdisc_submission_value);
    if not missing(&invar.) then do;
        if &checkct. = Y then do;
            if missing(ct_cdisc_submission_value) then do;
                if not missing(cl_term) then do;
                    put "NO" "TE: xuct.sas - %upcase(&inds..&invar)=" &invar. "not found in CDISC CT, but in %upcase(&codelist.) codelist.";
                    put "NO" "TE: xuct.sas - Please check whether %upcase(&codelist.) codelist is extensible and there is no suitable CDISC submission value.";
                    call missing(ctmissfl);
                end;
                else do;
                    if not missing(ct_suggest) then do;
                        put "WARN" "ING: xuct.sas - %upcase(&inds..&invar)=" &invar. "not found in CDISC CT. Check whether " ct_suggest "value should be used for remap";
                    end;
                    else do;
                        put "WARN" "ING: xuct.sas - %upcase(&inds..&invar)=" &invar. "not found in CDISC CT. Check CDISC CT for possible values or add " &invar. "to %upcase(&codelist.) codelist in spec";
                    end;
                    ctmissfl = "Y";
                end;
            end;
            if missing(cl_term) then do;
                if not missing(ct_cdisc_submission_value) then put "WARN" "ING: xuct.sas - %upcase(&inds..&invar)=" &invar. "not found in %upcase(&codelist.) codelist, but found in CDISC CT. Add " &invar. "value to %upcase(&codelist.) codelist in spec.";
                else put "WARN" "ING: xuct.sas - %upcase(&inds..&invar)=" &invar. "not found in %upcase(&codelist.) codelist in spec";
                clmissfl = "Y";
            end;
        end;
    end;
run;
```

12. Delete/keep temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
    proc datasets nolist nodetails lib = work;
        delete _&sysmacroname.:;
    run;
%end;
``` 

[Link to full code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuct.sas)