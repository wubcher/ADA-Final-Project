ADA Final Project – NHANES 2021–2022

Overview
This project analyzes data from the 2021–2022 National Health and Nutrition Examination Survey (NHANES) to examine:

- Primary objective: Association between serum 25-hydroxyvitamin D (vitamin D) categories and current depressive symptoms (PHQ-9 ≥10) among U.S. adults aged 18–45 years.
- Secondary objective: Association between poverty-income ratio categories and current depressive symptoms in the same population.

The analysis is implemented in an R Markdown file (ADA Final Project Analysis) and produces descriptive tables and unweighted logistic regression models.

Data sources
- NHANES 2021–2022 SAS transport files (.xpt):
  - DEMO_L.xpt: demographics, survey design, weights
  - VID_L.xpt: vitamin D measures
  - DPQ_L.xpt: PHQ-9 depression items
  - BMX_L.xpt: body measures (BMI)
  - DIQ_L.xpt: diabetes
  - BPQ_L.xpt: blood pressure/hypertension
  - MCQ_L.xpt: medical conditions
  - SMQ_L.xpt: smoking/tobacco
  - ALQ_L.xpt: alcohol use
  - UCPREG_L.xpt: urine pregnancy test

Key variables
- Outcome:
  - PHQ-9 total score (phq9_total)
  - Depressive symptoms (depressive_symptoms: TRUE if PHQ-9 ≥10, FALSE otherwise)
- Primary exposure:
  - Vitamin D in nmol/L (vitamin_d)
  - Vitamin D in ng/mL (vitamin_d_ngml)
  - Vitamin D category (vitamin_d_cat: Sufficient, Insufficient, Deficient)
- Secondary exposure:
  - Income-to-poverty ratio (income_poverty_ratio)
  - Poverty category (poverty_cat: ≥3.0 High, 1.3–<3.0 Middle, <1.3 Low)
- Covariates:
  - Age (age, age_cat)
  - Sex (sex)
  - Race/ethnicity (race_ethnicity)
  - Education (education)
  - BMI (bmi, bmi_cat, bmi_cat3)
  - Smoking status (smoking_status)
  - Alcohol use (alcohol_cat)
  - Chronic disease count and indicator (chronic_disease_count, any_chronic_disease)
  - Survey design variables (strata, psu, weight_phlebotomy)

Main steps in the R Markdown code
1) Load packages
   - haven, dplyr, tidyr, naniar, mice, survey, car, ggplot2, table1, flextable.

2) Import and merge data
   - Read all .xpt files.
   - Merge by SEQN into a single dataset (nhanes_merged).

3) Select variables and apply inclusion/exclusion
   - Keep adults aged 18–45 years.
   - Exclude pregnant participants based on exam status and urine pregnancy test.

4) Recode and derive variables
   - Rename NHANES variable names to analysis-friendly names.
   - Recode categorical variables (sex, race_ethnicity, education, smoking, alcohol).
   - Create:
     - PHQ-9 total and depressive_symptoms.
     - Vitamin D categories.
     - BMI and BMI category variables.
     - Poverty categories.
     - Chronic disease count and any_chronic_disease.
     - Smoking_status and alcohol_cat.
     - Age categories.

5) Create analytic sample and examine missingness
   - Restrict to participants with non-missing vitamin D, PHQ-9, and survey design variables.
   - Summarize missingness for key variables using naniar::miss_var_summary.

6) Base complete-case dataset and multiple imputation
   - Create nhanes_cc_base with complete data on all model covariates except education and income_poverty_ratio.
   - Construct impute_df and perform multiple imputation with mice:
     - education: multinomial logistic (polyreg)
     - income_poverty_ratio: predictive mean matching (pmm)
   - Inspect imputed values.

7) Descriptive statistics (Table 1)
   - Complete the first imputed dataset (imp1).
   - Create an overall Table 1 using table1 (vitamin D, demographics, BMI, behaviors, chronic disease, depressive symptoms).
   - Export Table 1 to PowerPoint using flextable::save_as_pptx.
   - Produce cross-tabulations of selected categorical variables with depressive_symptoms.

8) Unweighted logistic regression – primary objective
   - Build nhanes_simple with:
     - Outcome: dep_bin (0/1 from depressive_symptoms)
     - Exposure: vitamin_d_cat (reference = Sufficient)
     - Covariates: age, sex, race_ethnicity, education, income_poverty_ratio, bmi, smoking_status, alcohol_cat, chronic_disease_count.
   - Fit unweighted logistic regression (fit_logit) with glm(family = binomial).
   - Calculate odds ratios and 95% confidence intervals (or_table).

9) Unweighted logistic regression – secondary objective (poverty)
   - Build nhanes_pov with:
     - Outcome: dep_bin
     - Exposure: poverty_cat (reference = ≥3.0 High)
     - Covariates: age, sex, race_ethnicity, education, bmi, smoking_status, alcohol_cat, chronic_disease_count.
   - Fit logistic regression (fit_pov).
   - Calculate odds ratios and 95% confidence intervals (or_pov).

How to run the analysis
1) Place the Rmd file and all NHANES .xpt files in the same working directory.
2) Open the Rmd file in RStudio (or another R environment).
3) Knit to HTML:
   - Click “Knit” → “Knit to HTML”, or run rmarkdown::render("ADA-Final-Project.Rmd").
4) The HTML output will include:
   - Sample size and missingness summaries.
   - Table 1 and cross-tabulations.
   - Logistic regression output and odds ratio tables for the primary and secondary objectives.

Notes and assumptions
- Analyses are unweighted for the main models; survey design variables are used for sample definition and imputation but not in the final glm models.
- Education and income-to-poverty ratio are imputed using multiple imputation; other covariates use listwise deletion in nhanes_cc_base and nhanes_simple.
- Vitamin D and poverty are modeled as categorical exposures; linearity-in-the-logit is relevant only for continuous covariates such as age and (if used continuously) BMI or income_poverty_ratio.
- Results generalize to the analytic complete-case sample; they are not fully survey-weighted national estimates.

Contact / author
- Author: Yosef Chernet  
- Project: ADA Final Project – Association of vitamin D and depressive symptoms using NHANES 2021–2022