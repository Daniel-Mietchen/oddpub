# Open Data Detection in Publications (ODDPub)

[![Build Status](https://travis-ci.com/quest-bih/oddpub.svg?branch=master)](https://travis-ci.com/quest-bih/oddpub)
[![codecov](https://codecov.io/gh/quest-bih/oddpub/branch/master/graph/badge.svg)](https://codecov.io/gh/quest-bih/oddpub)


ODDPub is a text mining algorithm that parses a set of publications and detects
which publications disseminated Open Data or Open Code together with the
publication. It is tailored towards biomedical literature.

## Authors

Nico Riedel (nico.riedel@bihealth.de), Miriam Kip, Evgeny Bobrov - QUEST Center for Transforming Biomedical Research, Berlin Institute of Health

## Installation

The latest version of the algorithm is structured as an R package and can easily be installed with the following command:

``` r
# install.packages("devtools") # if devtools currently not installed
devtools::install_github("quest-bih/oddpub")
```

## Description

The algorithm searches for several categories of similar keywords in each 
sentence. Multiple categories have to match for a single sentence to trigger a 
detection. Among keyword categories are categories for specific biomedical
databases as well as their corresponding accession numbers (as regular
expressions), general-purpose repositories or different file formats typically
used to distribute raw data in the supplement.

Additionally, Open Code dissemination is detected using keywords categories
for source code or code repositories.

## Usage

The package exposes four functions that allow the following workflow:

``` r
oddpub::pdf_convert(PDF_folder, output_folder)
```
Converts PDFs contained in one folder to txt-files and saves them into the output folder.
**Requires external poppler library for the conversion.**

``` r
PDF_text_sentences <- oddpub::pdf_load(pdf_text_folder)
```
Loads all text files from given folder.

``` r
open_data_results <- oddpub::open_data_search(PDF_text_sentences)
```
Actual Open Data detection. Returns for each file if Open Data or Open Code is detected.

``` r
open_data_sentences_detected <- oddpub::open_data_sentences(PDF_text_sentences)
```
Function that returns the detected Open Data sentences as well. Does not work for several combined keyword categories that do not search on the sentence level ('Supplemental table or data', 'Supplementary raw/full data with specific file format', 'Data availability statement'). For those categories only returns if there was a hit in this category or not.

The algorithm was developed on an institutional publication record (N=8654). On an additional training dataset of 868 random Pubmed publications it achieved a sensitivity of 82% and a specificity of 98%. An independent validation sample is currently work in progress.

## Detailed description of the keywords

In the following, we give an overview over the different parts of the keywords used in the algorithm

Those are the combined keyword categories that are searched in the full text. If a hit was detected in any of these combined categories, the paper is flagged as Open Data. For definitions of the individual keyword categories, see below.

| Combined Keyword Category | Keywords |
|---------------------------|----------|
| Field-specific databases        | FIELD_SPECIFIC_DB NEAR ACCESSION_NR NEAR (AVAILABLE NOT (NOT_AVAILABLE OR WAS_AVAILABLE)) |
| General-purpose databases | REPOSITORIES NEAR (AVAILABLE NOT (NOT_AVAILABLE OR WAS_AVAILABLE)) |
| Dataset |	(DATASET_NAME OUTER_SYM DATASET_NUMBER) OR SUPPLEMENTAL_DATASET |
| Supplemental table or data |	SUPPLEMENTAL_TABLE NEAR_WD(10) (FILE_FORMATS OR ALL_DATA) |
| Supplementary raw/full data with specific file format | (ALL_DATA NOT NOT_DATA) NEAR_WD(10) FILE_FORMATS |
| Data availability statement |	DATA_AVAILABILITY NEAR_WD(30) (“doi” OR ACCESSION_NR OR REPOSITORIES) |
| Dataset on Github |	DATA NEAR GITHUB NEAR (AVAILABLE NOT (NOT_AVAILABLE OR WAS_AVAILABLE)) |


Additionally, the detection of Open Code statements is done with the following keywords:

| Combined Keyword Category | Keywords |
|---------------------------|----------|
| Source-code availability  | SOURCE_CODE NEAR AVAILABLE NOT (NOT_AVAILABLE OR WAS_AVAILABLE OR UPON_REQUEST) |
| Supplementary Source-code | SOURCE_CODE NEAR SUPPLEMENT |
| All Open Code keywords combined | SOURCE_CODE NEAR (SUPPLEMENT OR (AVAILABLE NOT (NOT_AVAILABLE OR WAS_AVAILABLE OR UPON_REQUEST)) |

Individual keyword categories:

| Definitions    | Description  |  Keywords  |
|----------------|--------------|------------|
| x NEAR y       | Are the two keywords (or groups of keywords) x and y in the same sentence? | |
| x NEAR_WD(n) y | Second definition of NEAR. This time counts how many words are between the two keywords. If the number lies below a cutoff value (e.g. 10 words), the two input words are considered "near". This additional definition is needed for cases like "S2 Table. Raw data. https://doi.org/10.1371/journal.pone.0158039.s002 (XLS)" | |
| NOT y          | y is not included in the same sentence | |
| AVAILABLE      | Abbreviation for a set of words that frequently occur to denote that the data have been made available in some way | ("included" OR "deposited" OR "released" OR "is provided" OR "are provided" OR "contained in" OR "available" OR "reproduce" OR "accessible" OR "can be accessed" OR "submitted" OR "can be downloaded" OR "reported in" OR "uploaded" OR "are public on") |
| WAS_AVAILABLE  | Set of words that indicate that data is not made available in the paper, but instead that data from a different source was used | ("was provided" OR  "were provided" OR  "was contained in" OR  "were contained in" OR  "was available" OR  "were available" OR  "was accessible" OR  "were accessible" OR  "deposited by" OR  "were reproduced") |
| NOT_AVAILABLE  | Set of negated availability phrases | ("not included" OR  "not deposited" OR  "not released" OR  "not provided" OR  "not contained in" OR  "not available" OR  "not accessible" OR  "not submitted")|
| UPON_REQUEST   | Phrase describing that data are only available upon request | ("upon request" OR "on request" OR "upon reasonable request") |
| ALL_DATA       | Set of words describing all data or raw data | ("all data" OR  "all array data" OR  "raw data" OR  "full data set" OR  "full dataset" OR  "crystallographic data" OR  "subject-level data") |
| NOT_DATA       | Set of negations of the data phrases | ("not all data" OR "not all array data" OR "no raw data" OR "no full data set" OR "no full dataset") |
| FIELD_SPECIFIC_DB        | Set of names and abbreviations of field specific databases | ("GEO" OR "Gene Expression Omnibus" OR "European Nucleotide Archive" OR "National Center for Biotechnology Information" OR "European Molecular Biology Laboratory" OR "EMBL-EBI" OR "BioProject" OR "Sequence Read Archive" OR "SRA" OR "ENA" OR "MassIVE" OR "ProteomeXchange" OR "Proteome Exchange" OR "ProteomeExchange" OR "MetaboLights" OR "Array-Express" OR "ArrayExpress" OR "Array Express" OR "PRIDE" OR "DNA Data Bank of Japan" OR "DDBJ" OR "Genbank" OR "Protein Databank" OR "Protein Data Bank" OR "PDB" OR "Metagenomics Rapid Annotation using Subsystem Technology" OR "MG-RAST" OR "metabolights" OR "OpenAgrar" OR "Open Agrar" OR "Electron microscopy data bank" OR "emdb" OR "Cambridge Crystallographic Data Centre" OR "CCDC" OR "Treebase" OR "dbSNP" OR "dbGaP" OR "IntAct" OR "ClinVar" OR "accession number" OR "accession code" OR "accession numbers" OR "accession codes") |
| ACCESSION_NR   | Set of regular expressions that represent the accession number formats of different (biomedicine-related) databases | ("GSE[[:digit:]]{2,8}" OR "GSM[[:digit:]]{2,8}" OR "GDS[[:digit:]]{2,8}" OR "GPL[[:digit:]]{2,8}" OR "(E\|D\|S)RP[:digit:]{6,}" OR "PRJ(E\|D\|N\|EB\|DB\|NB)[:digit:]+" OR "ERS[:digit:]{6,}" OR "SAM(E\|D\|N)[A-Z]?[:digit:]+" OR "[A-Z]{1}[:digit:]{5}\\.[:digit:]+" OR "[A-Z]{2}[:digit:]{6}\\.[:digit:]+" OR "[A-Z]{4}S?[:digit:]{8,9}\\.[:digit:]+" OR "[A-Z]{3}[:digit:]{5}\\.[:digit:]+" OR "[A-Z]{5}[:digit:]{7}\\.[:digit:]+" OR "[A-Z]{1}[:digit:]{5}" OR "[A-Z]{2}[:digit:]{6}" OR "[A-Z]{3}[:digit:]{5}" OR "[A-Z]{4}[:digit:]{8}" OR "[A-Z]{5}[:digit:]{7}" OR "[A-Z]{6}[:digit:]{9}" OR "(E\|D\|S)RX[:digit:]{6,}" OR "(E\|D\|S)RR[:digit:]{6,}" OR "GCA_[:digit:]{9}\\.[:digit:]+" OR "PRJNA[[:digit:]]{3,8}" OR "SRP[[:digit:]]{3,8}" OR "SRR[[:digit:]]{3,8}" OR "SRX[[:digit:]]{3,8}" OR "SRS[[:digit:]]{3,8}" OR "SRZ[[:digit:]]{3,8}" OR "MSV[[:digit:]]{7,9}" OR "PDX[[:digit:]]{5,6}" OR "E-[A-Z]{4}-[:digit:]{1,6}" OR "PXD[[:digit:]]{5,6}" OR "PRD[[:digit:]]{5,6}" OR "DRA[[:digit:]]{6}" OR "ERA[[:digit:]]{6}" OR "DRP[[:digit:]]{6}" OR "ERP[[:digit:]]{6}" OR "DRX[[:digit:]]{6}" OR "DRS[[:digit:]]{6}" OR "DRR[[:digit:]]{6}" OR "[:digit:]{1}[A-Z]{1}[[:alnum:]]{2}" OR "MTBLS[[:digit:]]{2,4}" OR "10.17590" OR "10.5073" OR "EMD-[[:digit:]]{4,5}" OR "[[:digit:]]{7}" OR "[A-Z]{2}_[:digit:]{6,7}" OR "[A-Z]{2}-[:digit:]{4,5}") |
| REPOSITORIES   | Set of names of general-purpose repositories | ("figshare" OR "dryad" OR "zenodo" OR "dataverse" OR "DataverseNL" OR "osf" OR "open science framework" OR "mendeley data" OR "GIGADB" OR "GigaScience database" OR "OpenNeuro") |
| FILE_FORMATS   | Set of file formats | ("csv" OR "zip" OR "xls" OR "xlsx" OR "sav" OR "cif" OR "fasta") |
| GITHUB |	Github for data has to be treated differently, as we need additional information that data and not only code was shared on Github |	(“github”) |
| DATA |	Data keywords only used for the Github category |	("data" OR "dataset" OR "datasets") |
| ALL_DATA |	Set of words describing all data or raw data |	("all data" OR  "all array data" OR  "raw data" OR  "full data set" OR  "full dataset" OR  "crystallographic data" OR  "subject-level data") |
| NOT_DATA |	Set of negations of the data phrases |	("not all data" OR  "not all array data" OR  "no raw data" OR  "no full data set" OR  "no full dataset") |
| DATA_AVAILABILITY	| Set of headings for data availability section |	("Data sharing" OR  "Data Availability Statement" OR  "Data Availability" OR  "Data deposition" OR  "Deposited Data" OR  "Data Archiving" OR  "Availability of data and materials" OR  "Availability of data" OR  "Data Accessibility" OR  "Accessibility of data") |
| x OUTER y |	makes all possible pairwise combinations of strings x and y	| |
| x OUTER_SYM y |	symmetrical outer product with both orderings of vectors	| |
| SUPPLEMENTAL_TABLE_NAME |	Set of keywords denoting supplemental tables or files |	("supplementary table" OR   "supplementary tables" OR  "supplemental table" OR   "supplemental tables" OR   "table", "tables" OR    "additional file" OR  "file", "files") |
| SUPPLEMENTAL_TABLE_NUMBER |	Possible numbers of supplemental tables or files |("S[[:digit:]]", "[[:digit:]]", "[A-Z]{2}[[:digit:]]") |
| SUPPLEMENTAL_TABLE |	Numbered supplemental table or file |	SUPPLEMENTAL_TABLE_NAME OUTER SUPPLEMENTAL_TABLE_NUMBER |
| SUPPLEMENTAL_DATASET |	Numbered supplemental dataset |	("supplementary data [[:digit:]]{1,2}" OR "supplementary dataset [[:digit:]]{1,2}" OR "supplementary data set [[:digit:]]{1,2}" OR "supplemental data [[:digit:]]{1,2}" OR "supplemental dataset [[:digit:]]{1,2}" OR "supplemental data set [[:digit:]]{1,2}") |
| DATASET_NAME	| Names for datasets |	("data" OR "dataset" OR "datasets" OR "data set" OR "data sets") |
| DATASET_NUMBER |	Number for dataset |	("S[[:digit:]]{1,2}") |
| DATA_JOURNAL_DOIS |	Set of Open Data Journal DOIs for which the publication DOI is checked (from filename, not part of actual keyword search) |	("10.1038/s41597-019-", "10.3390/data", "10.1016/j.dib") |
| SUPPLEMENT     | Set of expression describing the supplement of an article | ("supporting information" OR "supplement" OR "supplementary data") |
| SOURCE_CODE    | Set of expressions describing source code | ("source code" OR "analysis script" OR "github" OR "SAS script" OR "SPSS script" OR "R script" OR "R code" OR "python script" OR "python code" OR "matlab script" OR "matlab code") |


## License

ODDPub is available under the MIT license. See the [LICENSE](https://github.com/quest-bih/oddpub/blob/master/LICENSE) file for more info.
