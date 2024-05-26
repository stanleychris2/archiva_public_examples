# Macro for Deriving VISIT/VISITNUM Variables

## Steps:
1. Define macro xuvisit with input parameters
2. Check for required parameters
3. Upload SV and TV domain datasets
4. Sort datasets by visit
5. Merge datasets and create VISITNUM variable
6. Subset unscheduled visits
7. Create VISIT and VISITNUM for unscheduled visits
8. Delete temporary datasets if debug flag is set to N

## Relevant Code Snippets:
```sas
%macro xuvisit(inds /*Required. Name of the input dataset.*/,
               dtcdate /*Required. Name of the date variable.*/,
               outds=&inds. /*Default: &inds. Name of the output dataset.*/,
               debug=N /*Default: N. If N then deletes temporary datasets.*/);
```

```sas
%*Upload SV and TV domain;
%xuload(sv tv, sdtm);

proc sort data = tv nodupkey;
    by visit;
run;

proc sort data = &inds. out = __tmp_&sysmacroname._1;
    by visit;
run;
```

```sas
%*Subset unscheduled visits;
proc sort data = sv out = __tmp_&sysmacroname._3(keep = usubjid svstdtc visit visitnum rename = (visit = __tmp_visit_sv visitnum = __tmp_visitnum_sv) where = (index(lowcase(__tmp_visit_sv), "uns") > 0));
    by usubjid svstdtc;
run;
```

For more details, you can refer to the full code [here](https://github.com/atorus-research/atorus-sas-macros/blob/dev/sas/global/xuvisit.sas).