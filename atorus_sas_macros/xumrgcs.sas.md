# Macro for Merging SUPP-- and CO Datasets

## Steps:
1. Define macro variables and flags
2. Check for required parameters and values
3. Process each input dataset
4. Merge SUPP-- dataset if specified
5. Merge CO dataset if specified
6. Delete temporary datasets if debug flag is set

### Relevant Code Snippets:
```sas
%macro xumrgcs(inds /*Required. Name(s) of the input dataset(s).*/,
               sourcelib /*Required. Name of the input library.*/,
               supp=Y /*Default: Y. If Y then merge SUPP-- dataset.*/,
               co=Y /*Default: Y. If Y then merge CO dataset.*/,
               debug=N /*Default: N. If N then delete temporary datasets.*/);
```

```sas
%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
```

```sas
%if %lowcase(&supp.) = n and %lowcase(&co.) = n %then %do;
```

```sas
%do i = 1 %to %sysfunc(countw(&inds.));
```

```sas
%if %sysfunc(exist(&sourcelib..supp&wrd.)) %then %do;
```

```sas
%if %lowcase(&co.) = y %then %do;
```

```sas
%if %sysfunc(exist(&sourcelib..co)) %then %do;
```

```sas
%if %sysfunc(exist(work.__tmp_&sysmacroname._&wrd.)) %then %do;
```

```sas
%if &debug. = N %then %do;
``` 

For the full code, please refer to the [link](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xumrgcs.sas).