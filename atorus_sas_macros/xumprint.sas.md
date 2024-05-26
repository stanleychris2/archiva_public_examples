# Log and Lst File Management Macro

### Step 1: Define local variables
```sas
%local end_xumprint params_to_check i param_name param_value;
```

### Step 2: Check and validate parameters
```sas
%let params_to_check = route;
```

### Step 3: Delete existing macros if route=YES
```sas
%*Delete macro which were already resolved in SAS session;
```

### Step 4: Call xuprogpath macro
```sas
%xuprogpath;
```

### Step 5: Save log and lst files if route=YES
```sas
proc printto print = "&__lst_path.&II.&__p_name..lst" log = "&__log_path.&II.&__p_name..log" new;
run;
```

### Step 6: Display log and lst files if route=NO
```sas
proc printto log = log print=print;
run;
```

### Step 7: Convert notes of interest into warnings
```sas
%*Converting NOTE of interest into warnings;
```

### Step 8: Restore original options
```sas
options &__origopts.;
```

_code_link: [xumprint.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xumprint.sas)_