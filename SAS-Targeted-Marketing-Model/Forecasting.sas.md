# Logistic Regression Model Building and Evaluation

## Steps:
1. Create test and training data
2. Run logistic model
3. Create data for gains chart
4. Check for overfitting
5. Explain the model to business partner
6. Save the model for execution

---

### Step 1: Create test and training data

```sas
libname storage '/folders/myfolders/';

data storage.model_ds;
set  storage.model_vars;

rand=ranuni(092765);
testdata=0;
if rand <=.7 then Resptest=.;
else if rand  >.7 then do;
    testdata=1;
    Resptest=Resp;
    Resp=.;
end;
run;

proc print data=storage.model_ds (obs=10);
run;
```

---

### Step 2: Run logistic model

```sas
proc logistic data=storage.model_ds descending;
model resp=
mostyp2
mostyp3 
mostyp4 
...
amotsc
APERSA
/selection=stepwise;
output out=scored p=pred;
run;

proc print data=scored (where=(testdata=1) obs=10);
run;
```

---

### Step 3: Create data for gains chart

```sas
proc sort data=scored;
by testdata;
run;

proc rank data=scored groups=100 ties=high out=scoredrank;
by testdata;
var pred;
ranks mscore;
run;

proc sql;
create table tt as
select ((100-mscore)/100) as rank, ((100-mscore)/100) as random, sum(resp) as resp_development, sum(resptest) as resp_test
from scoredrank
group by 1,2
order by 1,2
;
quit;
```

---

### Step 4: Check for overfitting

```sas
proc logistic data=storage.model_ds descending;
model resp= mostyp25 mostyp34 moshoo1 moshoo2
            mrelge2 mskc2 paanha2 amotsc MAUT12;
output out=scored_chk p=pred;
run;
quit;

proc sort data=scored_chk;
by testdata;
run;

proc rank data=scored_chk groups=100 ties=high out=scoredrank_chk;
by testdata;
var pred;
ranks mscore;
run;

proc sql;
create table tt_chk as
select ((100-mscore)/100) as rank, ((100-mscore)/100) as random, sum(resp) as resp_development, sum(resptest) as resp_test
from scoredrank_chk
group by 1,2
order by 1,2
;
quit;
```

---

### Step 5: Explain the model to business partner

```sas
proc reg data=storage.model_ds;
model resp= mostyp25 maut12 mostyp34 moshoo1 moshoo2
            mrelge2 mskc2 paanha2 amotsc;
run;
quit;
```

---

### Step 6: Save the model for execution

```sas
proc logistic data=storage.model_ds descending
            outmodel=storage.log_resp_model;
model resp= mostyp25 maut12 mostyp34 moshoo1 moshoo2
            mrelge2 mskc2 paanha2 amotsc;
run;
quit;
```

---

For the complete code, you can refer to the [Forecasting.sas](https://github.com/akashagte/SAS-Targeted-Marketing-Model/blob/master/Phase4/Forecasting.sas) file.