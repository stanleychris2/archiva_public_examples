# Data Manipulation and Clustering Analysis

## Steps:
1. Print data from storage.census_factors

```sas
libname storage '/folders/myfolders/';

proc print data=storage.census_factors (obs=10);
run;
```

2. Standardize data from storage.dmefzip and sort by zipcode

```sas
proc standard data=storage.dmefzip replace out=prefactor;
run;
proc sort data=prefactor;
by zipcode;
run;
```

3. Run a macro loop for clustering

```sas
options pageno=1;
%macro loop (c);
options mprint ;

%do s=1 %to 10;
%let rand=%eval(100*&c+&s);
proc fastclus data=storage.census_factors out=clus&Rand cluster=clus maxclusters=&c
converge=0 maxiter=100 replace=random random=&Rand;
ods output pseudofstat=fstat&Rand (keep=value);
var factor1--factor5;
title1 "Clusters=&c, Run &s";
run;
title1;

proc freq data=clus&Rand noprint;
tables clus/out=counts&Rand;
where clus>.;
run;
proc summary data=counts&Rand;
var count;
output out=m&Rand min=;
run;

data  Stats&Rand;
label count=' ';
merge fstat&rand
      m&rand (drop= _type_ _freq_)
	  ;
Iter=&rand;
Clusters=&c;
rename count=minimum value=PseudoF;
run;

proc append base=ClusStatHold data=Stats&Rand;
run;
%end;
options nomprint;
%Mend Loop;
```

4. Delete ClusStatHold dataset and run an outer loop

```sas
%Macro OuterLoop;
proc datasets library=work;
delete ClusStatHold;
run;
%do clus=4 %to 8;
%Loop (&clus);
%end;
%Mend OuterLoop;

%OuterLoop;
```

5. Generate a ggplot for visualization

```sas
proc ggplot data=ClusStatHold;
plot pseudoF*minimum/haxis=axis1;
symbol value=dot color=blue pointlabel=("#clusters" color=black);
axis offset=(5,5)pct;
title "F by Min for Clusters";
run;
title;
quit;
```

6. Merge and summarize data for descriptive statistics

```sas
%let varlist=INCMINDX PRCHHFM PRCRENT PRC55P PRC65P HHMEDAGE PRCUN18 PRC200K OOMEDHVL PRCWHTE;

/*descriptive stats for clusters*/

proc sort data=clus610;
by zipcode;
run;

data cluster_vars;
merge clus610 (keep=zipcode factor1--factor5 clus in=a) prefactor (in=b);
by zipcode;
run;

proc print data=cluster_vars (obs=10);
run;

proc summary data=cluster_vars nway;
class clus;
var &varlist;
output out=ClusStats mean=;
run;

proc summary data=cluster_vars nway;
where clus>.;
var &varlist;
output out=OverallStats mean=;
run;

proc print data=clusStats;
run;
proc print data=OverallStats;
run;

data  stats;
set ClusStats
      OverallStats
	 ;
run;

proc print data=stats;
run;
``` 

[Click here to view the full code](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase2/ClusterLoop%20w%20factors.sas)