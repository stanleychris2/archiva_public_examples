# Operating System Default Setup Macro

## Step 1: Define macro 'init'
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

## Step 2: Set options for macro usage
```sas
    options minoperator mindelimiter=",";
```

## Step 3: Declare global variables
```sas
    %global __root II __sponsor_level __prod_qc_separation;
```

## Step 4: Determine root folder path based on operating system
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

## Step 5: Call the macro 'init'
```sas
%init;
```

## Step 6: Setup path to global macros library
```sas
options mautosource sasautos=("&__root.&II.utils&II.func", sasautos) validvarname=any;
```

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/autoexec.sas)