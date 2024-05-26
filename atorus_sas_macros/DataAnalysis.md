# Table of Contents
- [SAS Programming Macros for Clinical Data Analysis](#sas-programming-macros-for-clinical-data-analysis)
- [Operating System Default Setup Macro](#operating-system-default-setup-macro)
- [Convert Character DTC Variable into Analysis Date Variables](#convert-character-dtc-variable-into-analysis-date-variables)
- [SDTM --DY Calculation Macro](#sdtm--dy-calculation-macro)
- [Number Alignment Macro](#number-alignment-macro)
- [Macro for Adding ADSL Core Variables to Analysis Dataset](#macro-for-adding-adsl-core-variables-to-analysis-dataset)
- [Dataset Creation and Metadata Handling](#dataset-creation-and-metadata-handling)
- [Macro for Creating Global Macro Variable from Dataset Metadata](#macro-for-creating-global-macro-variable-from-dataset-metadata)
- [CO Domain Record Creation Macro](#co-domain-record-creation-macro)
- [SAS Macro for Executing Contents Procedure](#sas-macro-for-executing-contents-procedure)
- [CDISC SDTM Controlled Terminology and Codelists Validation](#cdisc-sdtm-controlled-terminology-and-codelists-validation)
- [Macro for Deriving EPOCH Variable](#macro-for-deriving-epoch-variable)
- [SAS Format Creation from Codelist Metadata](#sas-format-creation-from-codelist-metadata)
- [SAS Dataset Upload and Processing Macro](#sas-dataset-upload-and-processing-macro)
- [Combine Multiple Raw .csv Datasets into One Dataset](#combine-multiple-raw-csv-datasets-into-one-dataset)
- [SAS Macro for Log and Lst File Management](#sas-macro-for-log-and-lst-file-management)
- [SDTM Dataset Merge Macro](#sdtm-dataset-merge-macro)
- [Extract Task-Related Global Variables from File Path](#extract-task-related-global-variables-from-file-path)
- [Variable Renaming Macro](#variable-renaming-macro)
- [Dataset Saving Macro with Sorting and Labeling](#dataset-saving-macro-with-sorting-and-labeling)
- [Macro for Creating --SEQ Variable](#macro-for-creating--seq-variable)
- [Text Variable Splitter Macro](#text-variable-splitter-macro)
- [Macro for Creating and Adding Variables to SUPP-- Domain](#macro-for-creating-and-adding-variables-to-supp--domain)
- [Macro for Deriving VISIT/VISITNUM Variables](#macro-for-deriving-visitvisitnum-variables)
- [Style Template Macro with Timestamps](#style-template-macro-with-timestamps)
- [Character Variable Splitting Macro](#character-variable-splitting-macro)
- [TLF Dataset Comparison Macro](#tlf-dataset-comparison-macro)
- [Dataset Comparison Macro](#dataset-comparison-macro)
- [Variable Length Adjustment Macro](#variable-length-adjustment-macro)
- [Macro for Assigning Titles from TFL_titles.csv](#macro-for-assigning-titles-from-tfl_titles.csv)

---

## SAS Programming Macros for Clinical Data Analysis
- **Description:** Listing of global and study-specific SAS codes with descriptions.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/README.md.md)**

## Operating System Default Setup Macro
- **Description:** Macro for setting defaults based on operating system before program execution.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/autoexec.sas.md)**

## Convert Character DTC Variable into Analysis Date Variables
- **Description:** Convert character --DTC variable into analysis -DT -TM -DTM and create analysis -DY variable.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xcdtc2dt.sas.md)**

## SDTM --DY Calculation Macro
- **Description:** Calculate SDTM --DY variable from two SDTM character --DTC dates.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xcdtcdy.sas.md)**

## Number Alignment Macro
- **Description:** Macro for proper number alignment in PDF or RTF files.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xdalign.sas.md)**

## Macro for Adding ADSL Core Variables to Analysis Dataset
- **Description:** Adds ADSL core variables to an analysis dataset.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xtcore.sas.md)**

## Dataset Creation and Metadata Handling
- **Description:** Creates a zero record dataset based on metadata and creates a global macro variable.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xtmeta.sas.md)**

## Macro for Creating Global Macro Variable from Dataset Metadata
- **Description:** Creates a global macro variable based on dataset metadata.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xtorder.sas.md)**

## CO Domain Record Creation Macro
- **Description:** Macro for creating CO domain records from dataset.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xucomm.sas.md)**

## SAS Macro for Executing Contents Procedure
- **Description:** Macro for executing contents procedure on input datasets.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xucont.sas.md)**

## CDISC SDTM Controlled Terminology and Codelists Validation
- **Description:** A SAS program to check variable values against CDISC SDTM Controlled Terminology and Codelists tab in spec.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuct.sas.md)**

## Macro for Deriving EPOCH Variable
- **Description:** Macro to derive EPOCH variable using SE SDTM dataset.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuepoch.sas.md)**

## SAS Format Creation from Codelist Metadata
- **Description:** Create SAS formats from spec Codelist tab.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xufmt.sas.md)**

## SAS Dataset Upload and Processing Macro
- **Description:** Macro for uploading datasets to SAS working directory, sorting, and processing formats/informats.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuload.sas.md)**

## Combine Multiple Raw .csv Datasets into One Dataset
- **Description:** Create one dataset from multiple raw .csv datasets based on specified parameters.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuloadcsv.sas.md)**

## SAS Macro for Log and Lst File Management
- **Description:** Macro for saving log and lst files and converting notes of interest into warnings.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xumprint.sas.md)**

## SDTM Dataset Merge Macro
- **Description:** Macro for merging SUPP-- and CO datasets to the input SDTM dataset.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xumrgcs.sas.md)**

## Extract Task-Related Global Variables from File Path
- **Description:** Extract task-related global variables from the currently executing file path.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuprogpath.sas.md)**

## Variable Renaming Macro
- **Description:** Macro to rename variables in a dataset based on specified mode, prefix, and characters to remove.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xurnm.sas.md)**

## Dataset Saving Macro with Sorting and Labeling
- **Description:** Macro for saving datasets with sorting, labeling, and optional .xpt file creation.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xusave.sas.md)**

## Macro for Creating --SEQ Variable
- **Description:** Macro for creating --SEQ variable in a dataset based on sort order.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuseq.sas.md)**

## Text Variable Splitter Macro
- **Description:** Macro to split long text variable into multiple shorter sub-variables.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xusplit.sas.md)**

## Macro for Creating and Adding Variables to SUPP-- Domain
- **Description:** Macro for creating and adding variables to SUPP-- domain.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xusupp.sas.md)**

## Macro for Deriving VISIT/VISITNUM Variables
- **Description:** Macro for deriving VISIT/VISITNUM variables using TV and SV domains.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/xuvisit.sas.md)**

## Style Template Macro with Timestamps
- **Description:** Macro for creating a style template for study PDF/RTF outputs with timestamps.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/jdtflstyle.sas.md)**

## Character Variable Splitting Macro
- **Description:** Macro for splitting character variables for output in table/listing.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/kdident.sas.md)**

## TLF Dataset Comparison Macro
- **Description:** Macro for comparing TLF datasets with various customizable options.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/kqtlfcomp.sas.md)**

## Dataset Comparison Macro
- **Description:** Macro for comparing datasets with optional SUPP-- and comments dataset comparison.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/kqucomp.sas.md)**

## Variable Length Adjustment Macro
- **Description:** Change variable length to the maximum length of value in it.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/kudlen.sas.md)**

## Macro for Assigning Titles from TFL_titles.csv
- **Description:** Macro assigns titles according to TFL_titles.csv file.
- **[Document Link](https://github.com/stanleychris2/archiva_public_examples/blob/master/atorus_sas_macros/kutitles.sas.md)**