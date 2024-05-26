# SAS Programming Macros for Clinical Data Analysis

## Steps:
1. Identify and list the global standard codes with descriptions.
2. Identify and list the study-specific codes with descriptions.

## Relevant Code Snippets:
### Global standard code
```sas
/* Global standard code */
%macro xucont;
/* Uploads the dataset and executes the contents procedure. */
%mend xucont;

%macro xufmt;
/* Uses a metadata file to create SAS formats. */
%mend xufmt;

%macro xuload;
/* Uploads datasets to SAS working directory and removes formats/informats. */
%mend xuload;

%macro xuloadcsv;
/* Loads external data files in CSV format into one dataset in the Work directory. */
%mend xuloadcsv;

%macro xumprint;
/* Calls xuprogpath and does all setup for the program, including creating libnames and saving .log and .lst files on the server. */
%mend xumprint;

%macro xuprogpath;
/* Creates global macro variables for the paths to the program and log files, and information about the task such as asset, protocol, task, level (development or final), subfolders, side (production or validation), type (SDTM, ADaM, TFL). */
%mend xuprogpath;

%macro xurnm;
/* Renames all variables in the dataset. */
%mend xurnm;

%macro xusave;
/* Applies the keep variables, sort order, dataset label, and saves the dataset permanently. */
%mend xusave;

%macro xcdtcdy;
/* Takes two --DTC dates and calculates a study day (--DY) variable. */
%mend xcdtcdy;

%macro xucomm;
/* Creates CO domain records from the dataset. */
%mend xucomm;

%macro xuepoch;
/* Derives EPOCH variable using the SE SDTM dataset. */
%mend xuepoch;

%macro xusupp;
/* Creates and adds variables to SUPP-- domain. */
%mend xusupp;

%macro xuvisit;
/* Derives VISIT/VISITNUM variables. */
%mend xuvisit;

%macro xcdtc2dt;
/* Converts character --DTC variable into numeric --DT --TM --DTM. Also calculates analysis --DY variables. */
%mend xcdtc2dt;

%macro xtcore;
/* Uses a metadata file to get the list of ADSL core variables, and then merges them from ADSL to the input dataset. */
%mend xtcore;

%macro xumrgcs;
/* Merges respective SUPP-- and CO records to the input SDTM dataset. */
%mend xumrgcs;

%macro xtmeta;
/* Uses a metadata file to create a zero-record dataset with all needed variables from spec and their attributes, plus a macro variable with a list of variables to keep in the final dataset. */
%mend xtmeta;

%macro xtorder;
/* Uses a metadata file to create a macro variable with the sorting sequence for the dataset. */
%mend xtorder;

%macro xuct;
/* Checks if a variable has only the values specified in the metadata file and checks values for compliance with CDISC Controlled terminology. */
%mend xuct;

%macro xuseq;
/* Creates --SEQ variable using the sorting sequence from the metadata file. */
%mend xuseq;

%macro xusplit;
/* Splits long text variable into multiple shorter sub-variables. */
%mend xusplit;

%macro xdalign;
/* Adds leading spaces to decimal-align variable values in data displays. */
%mend xdalign;
```

### Study-specific code
```sas
/* Study-specific code */
%macro jdtflstyle;
/* A style template macro for study PDF/RTF outputs. Opens ODS PDF/RTF destination and creates PDF/RTF file. Also, creates global macro variables with timestamps. */
%mend jdtflstyle;

%macro kdident;
/* Split long text into several lines with indentation, if required. */
%mend kdident;

%macro kqtlfcomp;
/* Nice and robust TFL datasets compare macro. */
%mend kqtlfcomp;

%macro kqucomp;
/* SDTM and ADaM datasets compare macro. */
%mend kqucomp;

%macro kudlen;
/* Set the variable length to the maximum length met for this variable within the dataset. */
%mend kudlen;

%macro kutitles;
/* Macro which assigns titles according to the external TFL_titles.csv file. */
%mend kutitles;
```

For more details, you can refer to the full code document [_here_](https://github.com/atorus-research/atorus-sas-macros/blob/dev/README.md).