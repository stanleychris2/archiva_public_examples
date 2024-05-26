# Macro for Creating --SEQ Variable

### Step 1: Define macro xuseq with input parameters
```sas
%macro xuseq(inds /*Required. Name of the input dataset.*/,
            sortvars /*Required. Sort order for the dataset.*/,
            prefix= /*Optional. Prefix for the --SEQ variable name.*/,
            debug=N /*Default: N. If N then deletes temporary variables.*/);
```

### Step 2: Check for required parameters
```sas
%let params_to_check = inds sortvars;
```

### Step 3: Check for optional parameters
```sas
%if %symexist(domain) = 0 and &prefix. = %then %do;
    %put %str(ERR)%str(OR: xuseq.sas - prefix = %nrstr(&domain) by default, but %nrstr(&domain) is not resolved. Either resolve %nrstr(&domain) variable before macro call, or specify the value in prefix parameter);
    %let end_xuseq = Y;
%end;
```

### Step 4: Sort the input dataset
```sas
proc sort data = &inds. out = &inds.;
    by &sortvars.;
run;
```

### Step 5: Create --SEQ variable based on sort order
```sas
data &inds. %if &debug. = N %then %do; (drop = __tmp_n) %end;;
    set &inds.;
    by &sortvars.;

    %*Retain temporary variable __tmp_n;
    retain __tmp_n;

    %*Create sequence;
    if first.usubjid then __tmp_n = 1;
    else __tmp_n + 1;

    %*Copy values from __tmp_n to --SEQ variable;
    &pref.seq = __tmp_n;

run;
```

### Step 6: Retain temporary variable and create sequence
```sas
%*Retain temporary variable __tmp_n;
retain __tmp_n;

%*Create sequence;
if first.usubjid then __tmp_n = 1;
else __tmp_n + 1;
```

### Step 7: Copy values to --SEQ variable
```sas
%*Copy values from __tmp_n to --SEQ variable;
&pref.seq = __tmp_n;
```

_link: [xuseq.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuseq.sas)_