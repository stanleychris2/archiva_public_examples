# CDISC SDTM Controlled Terminology Compliance Checker

## Steps:
1. Check if required parameters are not empty
```sas
%let params_to_check = inds invar codelist subset filename filepath;
```

2. Check if codelist metadata file is present
```sas
%if %sysfunc(fileexist(&filepath.&II.&filename.)) = 0 %then %do;
```

3. Check if _ct.sas7bdat is present
```sas
%if %sysfunc(exist(misc._ct)) = 0 %then %do;
```

4. Import spec codelist metafile
```sas
filename codelist "&filepath.&II.&filename." termstr = CRLF;
```

5. Subset spec codelists for target variable
```sas
data _&sysmacroname._cl_&cl.(keep = ID Name "NCI Codelist Code"n "Data Type"n Term "NCI Term Code"n  "Decoded Value"n\n\t\t\t\t\t\t\t\t rename = (ID = cl_id Name = cl_name "NCI Codelist Code"n = cl_nci_codelist_code "Data Type"n = cl_data_type Term = cl_term "NCI Term Code"n = cl_nci_term_code "Decoded Value"n = cl_decoded_value));
```

6. Merge spec CT to dataset
```sas
proc sql noprint undo_policy=none;
```

7. Check CDISC CT for target variable
```sas
data _&sysmacroname._ct_&cl. (keep = var4);
```

8. Merge CDISC CT list to dataset
```sas
proc sql noprint undo_policy = none;
```

9. Find synonyms from CT CDISC Synonym column
```sas
data _&sysmacroname._&ct_cl._syn;
```

10. Prepare report and put messages to log file
```sas
data _&sysmacroname._&cl._report;
```

11. Delete/keep temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
``` 

[Link to code](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuct.sas)