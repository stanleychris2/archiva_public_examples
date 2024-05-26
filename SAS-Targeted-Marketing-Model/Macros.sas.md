# Log Odds Graph Maker

## Steps:
1. Calculate missing values for independent variable
```sas
%Macro DissGraphMakerLogOdds(dsn,groups,indep,dep);
proc summary data=&dsn;
var &indep;
output out=Missing&indep nmiss=;
run;

data Missing&indep;
set  Missing&indep;
PctMiss=100*(&indep/_freq_);
rename &indep=NMiss;
run;

data _null_;
set  Missing&indep;
call symput ('Nmiss',Compress(Put(Nmiss,6.)));
call symput ('PctMiss',compress(put(PctMiss,4.)));
run;
```

2. Calculate percentage of missing values
```sas
proc summary data=RankedFile nway missing;
class Ranks&indep;
var &dep &indep;
output out=GraphFile mean=;
run;
```

3. Rank data based on independent variable
```sas
proc rank data=&dsn groups=&groups out=RankedFile;
var &indep;
ranks Ranks&indep;
run;
```

4. Calculate log odds
```sas
data graphfile;
set  graphfile;
logodds=log(&dep/(1-&dep));
run;
```

5. Separate data based on missing values in independent variable
```sas
data graphfile setaside;
set  graphfile;
if &indep=. then output setaside;
else             output graphfile;
run;
```

6. Plot log odds against independent variable
```sas
proc plot data=graphfile;
plot LogOdds*&indep=' ' $_FREQ_ /vpos=20;
title "&dep by &groups Groups of &indep NMiss=&Nmiss PctMiss=&PctMiss%  LogOdds in Miss=&LogOdds"
;
run;
title;
quit;
%Mend DissGraphMakerLogOdds;
```

**Code Link:** [_link](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase3/Macros.sas)