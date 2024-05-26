# Style Template Macro with Timestamps

## Step 1: Define macro parameters and default values

```sas
%macro jdtflstyle(filename /*Required. Name of the output file.*/,
                  filepath=&__root.&__clnt_I.&II.&__comp.&II.&__prot.&__subfolders_I.&II.&__task.&II.&__level.&__side_I.&II.tfl /*Default: side/tlf. File save path.*/,
                  type=RTF /*Default: RTF. File extension. RTF/PDF.*/,
                  lmarg=1in /*Default: 1in. Left margin size.*/,
                  rmarg=1in /*Default: 1in. Right margin size.*/,
                  tmarg=0.5in /*Default: 0.5in. Top margin size.*/,
                  bmarg=1in /*Default: 1in. Bottom margin size.*/,
                  escapechar=$ /*Default: $. Escape character.*/);
```

## Step 2: Check for required parameters

```sas
%let params_to_check = filename filepath type lmarg rmarg tmarg bmarg escapechar;
```

## Step 3: Define styles using proc template

```sas
proc template;
    define style glst1;
        class body, data /
            fontfamily=courier
            fontsize=9pt
            backgroundcolor=white
            color=black;

        class header /
            fontfamily=courier
            fontweight = medium
            fontsize=9pt
            backgroundcolor=white
            color=black;

        style table /
            fontweight = medium
            fontstyle = roman
            color = black
            backgroundcolor = white
            borderspacing = 1
            cellpadding = 2;

        style header /
            fontfamily = courier
            fontsize = 9pt
            fontweight = medium
            fontstyle = roman
            color = black
            backgroundcolor = white
            bordertopcolor=black
            bordertopwidth=1
            borderbottomcolor=black
            borderbottomwidth=1
            borderspacing = 1
            cellpadding = 7;

        style systemtitle /
            fontfamily = courier
            fontsize = 9pt
            fontweight = medium;

        style systemfooter /
            fontfamily = courier
            fontsize = 9pt
            fontweight = medium;
    end;
run;
```

## Step 4: Create macro variables with timestamps

```sas
%global timestamp tmstmp_date tmstmp_time;

%let timestamp = %sysfunc(datetime(), e8601dt16.);
%let tmstmp_date = %scan(&timestamp, 1, T);
%let tmstmp_time = %scan(&timestamp, 2, T);;
```

## Step 5: Open output destination based on file type

```sas
ods _all_ close;
options orientation = landscape device = ACTXIMG nodate nonumber leftmargin = &lmarg. rightmargin = &rmarg. topmargin = &tmarg. bottommargin = &bmarg.;

%if %lowcase(&type.) = pdf %then %do;
    ods pdf file = "%sysfunc(compress(&filepath.&II.&filename..pdf))" style = glst1 nogtitle nogfootnote nobookmarkgen;
%end;
%if %lowcase(&type.) = rtf %then %do;
    ods rtf file = "%sysfunc(compress(&filepath.&II.&filename..rtf))" style = glst1 nogtitle nogfootnote nobodytitle;
%end;

ods escapechar = "&escapechar.";
``` 

[Link to code file](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/study_specific/jdtflstyle.sas)