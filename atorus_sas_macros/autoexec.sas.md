# Autoexec Macro for Setting Defaults

### Step 1: Define macro %init
```sas
%macro init;

    %*This option allows using macro "in" operator.;
    options minoperator mindelimiter=",";

    %*Variables declaration;
    %global __root II __sponsor_level __prod_qc_separation;
    %*Root folder path.;
    %let __root = ;
    %*Left or right slash depends on OS type.;
    %let II = ;
    %*Defines if a client level folders should exist. Should be Y or N.;
    %let __sponsor_level = Y;
    %*Tells if there is an extra level under development/final area segregating prod vs QC. Should be Y or N.;
    %let __prod_qc_separation = Y; 

    %*Based on the OS type assign root path and a "slash" - left or right.;
    %*Put error to the log if OS type was not described here.;
    %if &sysscp=WIN %then %do;
        %let __root = F:\\projects;
        %let II = \\;
    %end;
    %else %if &sysscp in (SUN 4, SUN 64, LIN X64) %then %do;
        %let __root = /sambaShare/projects;
        %let II = /;
    %end;
    %else %do;
        %put ERROR: autoexec.sas - operating system &sysscp in not defined.;
    %end;

%mend init;
```

### Step 2: Set options for minoperator and mindelimiter
```sas
options minoperator mindelimiter=",";
```

### Step 3: Declare global variables
```sas
%global __root II __sponsor_level __prod_qc_separation;
```

### Step 4: Determine root folder path based on OS type
```sas
%if &sysscp=WIN %then %do;
    %let __root = F:\\projects;
    %let II = \\;
%end;
%else %if &sysscp in (SUN 4, SUN 64, LIN X64) %then %do;
    %let __root = /sambaShare/projects;
    %let II = /;
%end;
%else %do;
    %put ERROR: autoexec.sas - operating system &sysscp in not defined.;
%end;
```

### Step 5: Set sponsor_level and prod_qc_separation variables
```sas
%let __sponsor_level = Y;
%let __prod_qc_separation = Y;
```

### Step 6: Assign root path and slash based on OS type
```sas
%let __root = ;
%let II = ;
```

### Step 7: Log error if OS type is not defined
```sas
%else %do;
    %put ERROR: autoexec.sas - operating system &sysscp in not defined.;
%end;
```

### Step 8: Call the macro %init
```sas
%init;
```

### Step 9: Setup path to global macros library
```sas
options mautosource sasautos=("&__root.&II.utils&II.func", sasautos) validvarname=any;
```

**[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/autoexec.sas)**