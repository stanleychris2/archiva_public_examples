# Macro for Creating --SEQ Variable

### Step 1: Define local macro variables
```sas
%local end_xuseq params_to_check i param_name param_value pref;
```

### Step 2: Check for required parameters
```sas
%let params_to_check = inds sortvars;
```

### Step 3: Check for optional parameters
```sas
%let pref = &prefix.;
```

### Step 4: Sort the dataset by sorting order
```sas
proc sort data = &inds. out = &inds.;
    by &sortvars.;
run;
```

### Step 5: Create --SEQ variable
```sas
data &inds. %if &debug. = N %then %do; (drop = __tmp_n) %end;;
    set &inds.;
    by &sortvars.;

    retain __tmp_n;

    if first.usubjid then __tmp_n = 1;
    else __tmp_n + 1;

    &pref.seq = __tmp_n;
run;
```

### Step 6: Handle debug flag
```sas
%if &end_xuseq. = Y %then %goto endmac;
```

### Step 7: Stop macro if requirements are not met
```sas
%macro xuseq(inds /*Required. Name of the input dataset.*/,
            sortvars /*Required. Sort order for the dataset.*/,
            prefix= /*Optional. Prefix for the --SEQ variable name.*/,
            debug=N /*Default: N. If N then deletes temporary variables.*/);
``` 

For the full code, please refer to the [link](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuseq.sas).