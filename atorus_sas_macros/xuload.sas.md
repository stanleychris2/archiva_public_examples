# Dataset Upload and Processing Macro

### Step 1: Check required parameters for emptiness
```sas
%let params_to_check = inds sourcelib mode;
```

### Step 2: Validate mode parameter values
```sas
%if %lowcase(&param_name.) = mode %then %do;
```

### Step 3: Process datasets based on mode
```sas
%if %lowcase(&mode.) = smart %then %do;
```

### Step 4: Remove unexpected formats/informats
```sas
%if %lowcase(&mode.) = remove %then %do;
```

### Step 5: Upload datasets from library
```sas
data &wrd.;
```

### Step 6: Sort datasets if specified
```sas
%if &sortvars. ^= %then %do;
```

### Step 7: Delete temporary datasets for debug purposes
```sas
%if &debug. = N %then %do;
```

For the full code, you can check the [code snippet](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuload.sas).