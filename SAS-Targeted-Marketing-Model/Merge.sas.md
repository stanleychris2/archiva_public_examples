# Data Manipulation and Aggregation for Census and Bank Branch Data

## Steps:
1. Check the data in storage.dmefzip
2. Create new variables in prefactor
3. Aggregate data in branch_fdic
4. Merge datasets in akash.branch_fi_comp_census
5. Output final data to out_branch_census.csv
6. Perform SQL query in branch_all_sql

### Step 1: Check the data in storage.dmefzip
```sas
/*Check the data*/
proc contents data=storage.dmefzip;
title "Contents of Zipcode Demographic File";
run;
proc print data=storage.dmefzip (obs=10);
run;
```

### Step 2: Create new variables in prefactor
```sas
/*Create new data*/
data prefactor;
set storage.dmefzip;

stname=zipstate(zipcode);
zip=input(zipcode,8.0);
zp=zipcode*1;

run;
```

### Step 3: Aggregate data in branch_fdic
```sas
/*Get the data*/
proc import datafile='/folders/myfolders/fdic_branch.csv' 
    out=branch_fdic dbms=csv replace;
run;
```

### Step 4: Merge datasets in akash.branch_fi_comp_census
```sas
/*merge dataset together
 keep only zipcodes where FI is located*/
data akash.branch_fi_comp_census;
merge branch_count_fi (drop=percent in=a) branch_count_comp (drop=percent in=b) prefactor (in=c);
by zip;
if a;
if a and ^b then num_branches_comp=0;
if a and ^c  then missing_census=1;
else missing_census=0;
run;
```

### Step 5: Output final data to out_branch_census.csv
```sas
/*Output the data*/
proc export data=akash.branch_fi_comp_census 
    outfile='/folders/myfolders/out_branch_census.csv' 
    dbms=csv replace;
run;
```

### Step 6: Perform SQL query in branch_all_sql
```sas
/*sql example*/
proc sql;
create table branch_all_sql as
select '589' as fi_uninum_nm, zip, servtype, sum(fi_uninum=589) as num_branches, sum(fi_uninum^=589) as num_comp_branches
from branch_fdic
where servtype=11 and zip in
    (select distinct zip from branch_fdic where fi_uninum=589 and servtype=11)
group by 2,3
;
quit;
proc print data=branch_all_sql;
TITLE "sql data QA";
run;
``` 

[Link to the code file](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase1/Merge.sas)