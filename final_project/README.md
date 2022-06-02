# Final project

## Introduction

This project mimics an end-to-end study conducted on a clinical data warehouse and leading to the publication of a scientific article.

## Problem statement

Let’s imagine that we are in the year 2025 and that the scientific community does not know yet whether smoking is associated or not with a higher risk of developing a lung cancer. 
Answering this question may save millions of lives due to the high prevalence of smoking in the general population. 
A randomized controlled trial (RCT) would provide a high-level evidence but it does not appear feasible in that case. 
First, the delay between exposure and cancer is expected to be long (some years). 
Realizing a prospective study would consequently be extremely long whereas analyzing retrospective data may provide a rapid answer. 
Second, it would be ethically unacceptable and practically difficult to force a “treated” group to smoke and a "control" group not to smoke during 10 to 20 years. 
Conducting a retrospective observational study therefore appears as the best option to provide first evidences.

## Study design

Leveraging real world data collected in the Greater Fromage University Hospitals and centralized in its clinical data warehouse, 
you will realize a case-control study to determine whether developing a lung cancer is associated with a higher prevalence of smoking. 
Each case (patient with a diagnosed lung cancer) should be matched with a control (patient without lung cancer that shares similar 
demographic characteristics). We consider data that has been extracted on $t_{extract}=$ December 1st, 2025, and that has been collected 
after December 1st, 2022. 

The study will be composed of the following steps:
- **Data pre-processing**:
  - **Data cleaning** (identity deduplication, imputation of linkage keys, merge of contiguous visits, etc.)
  - **Data selection** (selection of medical units, selection of timespans, detection of flawed data leveraging of a priori knowledge, etc.)
      - Tip: the clinical software used to collect claim data (condition dataframe) may have been deployed at different dates in different hospitals. This deployment may lead to visits with no claim data in some hospitals and some periods. These visits shall be discarded. Moreover, including only some visits coming from hospitals with an heterogeneous availability of claim data may lead to biases, as these hospitals may be specialized in some pathologies/populations. Discarding all data coming from these problematic hospitals may be judicious.
  - **Variable extraction** (risk factors, cancer outcomes). Cancer outcomes will be defined using claim codes (condition data) attached to each visit. Smoking risk factor may be defined using either claim codes, variables that have already been extracted by a machine-learning-based NLP algorithm (not provided), or a new rule-based NLP algorithm.
- **Statistical analysis**
  - **Case-control matches**: a **case cohort** will be composed of patients with a diagnosis of lung cancer attached to a visit starting in the last year before the end of the study. A **control cohort** will be chosen among patients with a visit starting in the same period and not in the **case cohort** (cohort of eligible patients). Each patient of the **case cohort** will be matched with a patient in the eligible population to form a **control cohort** of the same size than the **case cohort**. The matching algorithm will select for each case patient a control patient of the same sex and with an age that is close to the case patient. Each patient of the eligible population can be matched to at most one patient of the case cohort.
  - **Hypothesis testing**: test whether the prevalence ratio of the suspected risk factor (*i.e.* history of smoking) differs significantly in the case and control cohorts.
  - **Sensitivity analysis**: assess whether the results are robust to alternative parameterization of the analysis pipeline
    - Tip: you can for instance compare the analysis conducted defining smoking either using only claim data, using only NLP-data or using both sources of data

## Dataset description:

To conduct the study, the following data is made available (freely inspired from the OMOP standard)

### Raw data
#### Patients' identities and demographic data

A first category of data concerns the identities of the patients and their demographic characteristics (males or females, dates of birth, etc.). Ideally each patient that has been treated in a hospital has one and only one identity related to him/her in the clinical information system. Identity and demographic data is made available in the *df_person* dataframe that contains the following columns:
- **person_id** : a unique identifier for each identity
- **birth_datetime**: date of birth
- **death_datetime**: date of death if the patient died during a hospitalization stay
- **gender_source_value**: gender as mentioned in the clinical software
- **cdm_source**: name of the clinical software

We assume that a deduplication algorithm designed for research has moreover been applied to all the identities that are registered in the clinical data warehouse to automatically detect duplicates. Its results are made available as a *df_dedup* dataframe that contains the following columns:
- **unique_person_id** : identifier of the unique identities obtained after deduplication
- **person_id** : identifier of a record before deduplication that corresponds to the same patient than the unique_person_identifier 

#### Administrative data related to patients' pathways

Administrative data is collected during a patient's hospitalization stay for many purposes among which the management of the hospital. Hospital managers indeed need to know where patients are, when they entered and left some medical units. The administrative description of a patient's pathway in a hospital may be used for research as it describes the temporality of a disease evolution and treatment. Many levels of description may be considered (hospitals, buildings, rooms, medical units, etc.), but we focus in this example to the simplest one: visits in hospitals. This administrative data is made available as a *df_visit* dataframe that contains the following columns:
- **visit_occurrence_id** : identifier of a visit
- **care_site_id** : identifier of a care site
- **visit_start_datetime** : date of the beginning of the visit
- **visit_end_datetime** : date of the end of the visit
- **person_id** : identifier of the patient
- **visit_source_value** : type of visit (here only hospitalisation stays)


#### Claim data related to patients' condition (diagnosis)

Claim data is collected in order for the hospital to be reimbursed when it treats a patient. Specific information systems have been developed and deployed to collect and send data to the relevant reimbursement agencies (*e.g.* in France the *Programme de Médicalisation du Système d'Information, PMSI*). Claim data should 1) describe the medical conditions of patients that have been admitted and 2) indicate the treatments and procedures that the hospital has provided. Patients' medical conditions are often described using the International Classification of Diseases (ICD, or *Classification Internationale des Maladies - CIM* in French). To be easily processed by reimbursement agencies, claim data is structured (*i.e.* it is collected as tabular data not as free text, images or signals). In this exercise, we consider the *df_condition* dataframe with data imported from the claim information system. The dateframe contains the following columns:
- **visit_occurrence_id** : identifier of the visit
- **condition_occurrence_id** : identifier of the condition
- **condition_source_value** : value of the condition

In this exercise, we assume that a lung cancer is coded as a condition with one of the following ICD-10 codes:
'C343', 'C341', 'C342', 'C34', 'C340'.

Smoking is coded as a secondary condition with one of the following ICD-10 codes:
'Z720', 'Z716', 'Z864', 'Z587'.


#### Clinical notes

In this project we consider two categories of data related to clinical notes:
- **free text** contained in clinical texts (*i.e.* raw data)
- variables that have been **extracted by a pre-defined NLP algorithm** (*i.e.* extracted data).

Free text is made available in a *df_note* dataframe that contains the following columns:
- **visit_occurrence_id** : identifier of the visit
- **note_id** : identifier of the medical note
- **note_text** : free text contained in the medical note
- **note_datetime** : date of text edition
- **cdm_source** : name of the clinical software

We consider moreover the *smoking* variable that has been extracted from the clinical texts by a pre-defined machine learning algorithm (not provided here). We assume that this algorithm has been developed, validated and deployed by another research team. The results of the algorithm are made available in the *df_note_nlp* dataframe that contains the following columns:
- **visit_occurrence_id** : identifier of the visit
- **note_id** : identifier of the medical note
- **extracted_concept_source_value** : name of the extracted concept
- **cdm_source** : name of the clinical software

Linkage keys are used to link records using common identifiers (*e.g* visit occurrence identifiers, patients identifiers, etc.). For the sake of simplicity, we consider only data related to the linkage of clinical notes to administrative visits and provided by a pre-defined deterministic algorithm. The results of the **deterministic algorithm** are made available in the *df_note_transco* dataframe that contains the following columns:
- **EHR1_visit_id** : visit_occurrence_id contained in the software "EHR 1"
- **EHRnote_visit_id** : visit_occurrence_id contained in the software "EHR note"

### A priori knowledge

In addition to raw and extracted data, engineers of the IT department have shared with you the following piece of information that may be leveraged in the analysis:
- *data collected in the Clinique de Comté-Vieux and collected in the period from the 1st January 2023 to the 13th April 2023 are not reliable due to technical problems in the clinical information system in this period*.

Moreover, each category of data requires a specific duration after the event it describes before being available in the clinical data warehouse (*i.e.* data timeliness). The operators of the clinical data warehouse have listed the following durations for each category of data:
- $\Delta t_{timeliness}^{(person)} = 1$ days
- $\Delta t_{timeliness}^{(visit)} = 1$ days 
- $\Delta t_{timeliness}^{(condition)} = 45$ days
- $\Delta t_{timeliness}^{(note)} = 1$ day
- $\Delta t_{timeliness}^{(note-nlp)} = 1$ day

We assume that condition and note data are manually collected at the admission of a patient (*i.e.* at the beginning date of a visit). Moreover, hospital managers indicate that the activity of hospitals may largely vary (the number of visits per month is not constant) but that this effect is expected and is not due to a problem in the information system.

## Deliverables
In this project, we expect a twofolds reporting of the study. First, a report shall present the study design, its results and discuss them. Second, the computer code used to realize the analysis shall be delivered to evaluators in order to ensure study reproducibility.

### Deliverable 1: Report
The study's report may be organized following the classical structure of a scientific article:

- **Introduction**
- **Methods**
- **Results**
- **Discussion**
- **Conclusion**
- **Annex**

In particular, one should discuss the strengths and limitations of this study in the discussion section. The report should be delivered as an HTML file in the /docs folder (see next section).

### Deliverable 2: Analysis code
The code developed to realize the analysis and generate the report shall be structured as:

- /data: raw and intermediate data
- /docs: final report and other documents
- /images: figures and tables
- /notebooks: notebook used to generate the final report
- /lung_cancer: functions used in the notebooks
- /tests: tests implemented in the code
- generate_report.sh: command to generate the HTML report out of the notebook
- README.md
- requirements.txt

Such a repository has been initiated and is made available with raw data. Various helper functions and analysis pipelines are in particular provided in the /lung_cancer folder. They may be used in the notebook /notebook/final_project.ipynb that is used to generate the final report when executing the generate_report.sh command. 

Remark: The scientific libraries *eds-nlp* and *eds-temporal-variability* may be used to facilitate the analysis. In the notebook /notebook/final_project.ipynb, we suggest using the *eds-temporal-variability* library. This use may be unnecessarily complex in this use case and you may code your own function to explore/select the judicious hospitals to correct for the bias caused by the heterogeneous deployment of clincal softwares. Feel free to delete this function and implement a new one.


### Expected work

In this project you have to finalize this study by 1) completing the /notebook/final_project.ipynb notebook that generates the final report and 2) if necessary add functions in the /lung_cancer folder to conduct this analysis. You will deliver a zip file containing all the repository files organized in the structure described above. Do not include in your zip file raw data.

Your evaluator will:

- 1) unzip your folder
- 2) copy raw data in the /data folder (same dataset than the one made available here)
- 3) execute all cells of the notebook
- 4) execute the generate_report.sh command
- 5) evaluate the report generated in the /docs folder
- 6) read the code that you will have added to the /lung_cancer folder and /notebook/final_project.ipynb file