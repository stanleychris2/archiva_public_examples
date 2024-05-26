# SAS Dataset Upload and Processing Macro

### Step 1: Check required parameters are not empty
```sas
%let params_to_check = inds sourcelib mode;
```

### Step 2: Check mode parameter values
```sas
%if %lowcase(&param_name.) = mode %then %do;
```

### Step 3: Process based on mode (smart, keep, remove)
```sas
%if %lowcase(&mode.) = smart %then %do;
```

### Step 4: Process each dataset
```sas
%do i = 1 %to &nwrd.;
```

### Step 5: Remove unexpected formats/informats
```sas
%if %lowcase(&mode.) = smart %then %do;
```

### Step 6: Upload dataset from library
```sas
data &wrd.;
```

### Step 7: Sort dataset if specified
```sas
%if &sortvars. ^= %then %do;
```

### Step 8: Delete temporary datasets for debug
```sas
%if &debug. = N %then %do;
```

For more details, you can refer to the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuload.sas).