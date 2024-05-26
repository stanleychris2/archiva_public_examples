# Extract Program Path and Task-Related Global Variables

### Step 1: Check if global variable _sasprogramfile exists
```sas
%if %symexist(_sasprogramfile) %then %do;
```

### Step 2: Extract program full path
```sas
%let __program_full_path = %sysfunc(compress(&_sasprogramfile., "\'"));
```

### Step 3: Extract task-related global variables
```sas
%let __clnt = %scan(&projects_path., 1, &II.);
%let __comp = %scan(&projects_path., 2, &II.);
%let __prot = %scan(&projects_path., 3, &II.);
```

### Step 4: Display all macro variables in the log.
```sas
%put ******************************************************************************************************;
%put === Program being executed === ;
%put Program executed by: %sysfunc(compress(&_clientuserid., "\'"));
%put Program full path  : &__program_full_path.;
%put Program name       : &__p_name.;
%put Program path       : &__p_path.;
%put ******************************************************************************************************;
%put === Task Information === ;
%put Client    : &__clnt.;
%put Compound  : &__comp.;
%put Protocol  : &__prot.;
%put Subfolders: &__subfolders.;
%put Task      : &__task.;
%put Level     : &__level.;
%put Side      : &__side.;
%put Type      : &__type.;
%put ******************************************************************************************************;
%put === Logs and lst outputs path === ;
%put Log path: &__log_path.;
%put Lst path: &__lst_path.;
%put ******************************************************************************************************;
```

Code Link: [xuprogpath.sas](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuprogpath.sas)