# Extract Task-Related Global Variables from File Path

## Steps:
1. Check if [_sasprogramfile](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuprogpath.sas) exists
```sas
%if %symexist(_sasprogramfile) %then %do;
```

2. Set global variables based on program path
```sas
%global __program_full_path __clnt __comp __prot __subfolders __task __level __side __type __p_name __p_path __log_path __lst_path;
```

3. Extract task information from path
```sas
%let __clnt = %scan(&projects_path., 1, &II.);
%let __comp = %scan(&projects_path., 2, &II.);
%let __prot = %scan(&projects_path., 3, &II.);
```

4. Determine level of program
```sas
%let __level = development;
%let __level = final;
```

5. Assign subfolders if applicable
```sas
%let __subfolders = %substr(&projects_path., %eval(%sysfunc(find(&projects_path., &__prot.)) + %length(&__prot.) + 1), %eval(%sysfunc(find(&projects_path., &__task.&II.&__level.)) - %eval(%sysfunc(find(&projects_path., &__prot.)) + %length(&__prot.) + 1) - 1)); 
```

6. Determine side of program
```sas
%let __side = %scan(&projects_path., %eval(%sysfunc(count(%substr(&projects_path., 1, %sysfunc(find(&projects_path., &__level.))), &II.)) + 2), &II.);
```

7. Determine type of program
```sas
%let __type = %scan(&projects_path., -2, &II.);
```

8. Set program name and path
```sas
%let __p_name = %qscan(%qscan(&__program_full_path., -1, &II.), -2, .);
%let __p_path = %substr(&__program_full_path., 1, %eval(%length(&__program_full_path.) - %length(&__p_name..sas) - 1));
```

9. Set log and lst paths
```sas
%let __log_path = %sysfunc(tranwrd(&__p_path.,program,log));
%let __lst_path = %sysfunc(tranwrd(&__p_path.,program,lst));
```

10. Display macro variables in log
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