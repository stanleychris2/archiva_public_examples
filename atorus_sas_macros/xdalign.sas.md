# Number Alignment Macro

### Step 1: Define macro parameters with default values
```sas
%macro xdalign(invar /*Required. Input variable name.*/,
               type=RTF, /*Default: RTF. Type of the output, extension.*/
			   escapechar=$ /*Default: $. Escape character.*/
               len=6 /*Default: 6. Length of the whole variable in the output, including indentation.*/);
```

### Step 2: Check if required parameters are empty
```sas
%let params_to_check = invar type len;

%do i = 1 %to %sysfunc(countw(&params_to_check., %str( )));
```

### Step 3: Check if output type is PDF or RTF
```sas
%if %lowcase(&type.) = pdf %then %do;
```

### Step 4: Add spaces for PDF outputs based on length of integer part
```sas
if length(scan(&invar., 1, ".")) = 1 then &invar. = "&escapechar.{nbspace %eval(&len. - 1)}" || strip(&invar.);
```

### Step 5: Add spaces for RTF outputs based on length of integer part
```sas
%else %if %lowcase(&type.) = rtf %then %do;
``` 

For more details, you can refer to the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xdalign.sas)