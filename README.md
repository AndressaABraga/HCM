# HCM
This repository contains the economic evaluation model developed to assess the cost-effectiveness of genetic screening in relatives of patients with dilated cardiomyopathy, from the perspective of the Brazilian Unified Health System (SUS).  
# Cost-Utility Model – Genetic Testing for Hypertrophic Cardiomyopathy (HCM)

This repository contains the economic model developed to assess the cost-utility of genetic testing versus standard clinical screening for hypertrophic cardiomyopathy (HCM) in the Brazilian Public Health System (SUS).

The model was implemented using **AMUA** and includes both deterministic and probabilistic components.

---

## Model Overview

| Component       | Description                                     |
|----------------|-------------------------------------------------|
| Model Type      | Decision tree + Markov model                    |
| Simulation      | Cohort-based (deterministic and probabilistic) |
| Initial Age     | 8 years                                         |
| Time Horizon    | Lifetime (62 years)                             |
| Cycle Length    | 1 year                                          |
| Discount Rate   | 5% annually (costs and benefits, from cycle 2) |

---

## Model Components

### Probabilities

- Probability of carrying a pathogenic variant
- Inheritance and phenotype expression
- Mortality rates (general and disease-specific)

### Utility Parameters

- QALYs based on age, clinical status, and genetic profile
- Utility modeled as disutility: `1 − Gamma(shape, rate)`

### Cost Parameters (in USD)

- Genetic testing (proband and cascade testing)
- Annual treatment and follow-up costs
- Genetic counseling

### Index Variables

Used internally in AMUA to manage table dimensions, time-series values, and PSA distributions.

---
##  Variable Overview – Probabilities

| Variable / Table   | Description                                                                               | Mean / Notes                              |
|:-------------------|:------------------------------------------------------------------------------------------|:------------------------------------------|
| p_prob             | Probability that the proband carries a pathogenic variant                                 | 0.41 (Literature-based estimate)          |
| p_sanger           | Probability that a first-degree relative inherits the variant (if proband is positive)    | 0.50 (Assumes Mendelian inheritance)      |
| p_fam_pos_fen      | Probability of developing HCM phenotype in genotype-positive relatives (by cycle)         | Time-dependent (Used in both models)      |
| p_fam_sem_tes_fen  | Probability of developing HCM phenotype in untested relatives or relatives of G− probands | Time-dependent (Applies to both groups)   |
| p_morte_g          | Background mortality from Brazilian population life tables                                | Time-dependent (IBGE 2023)                |
| p_morte_pac        | Mortality of individuals with HCM phenotype                                               | Time-dependent (Constant or time-varying) |

----
##  Utility Parameters

| Variable Name   | Description                                                      | Value          |
|:----------------|:-----------------------------------------------------------------|:---------------|
| u_pac           | Utility of individuals with HCM phenotype                        | Time-dependent |
| u_fam_gen_pos   | Utility of genotype-positive relatives in screening              | Time-dependent |
| u_fam _neg      | Utility of genotype-negative relatives discharged from follow-up | Time-dependent |
| u_fam_sem_teste | Utility of relatives without genetic test in screening           | Time-dependent |


----
##  Cost Parameters

| Variable Name        | Description                                                   | Value ($)       | Notes                                                         |
|:---------------------|:--------------------------------------------------------------|:----------------|:--------------------------------------------------------------|
| c_prob               | Cost of genetic test for proband                              | 286.71          | Local cost estimates                                          |
| c_sanger             | Cost of Sanger sequencing for relatives                       | 76.94           | Used in cascade testing                                       |
| c_tto                | Annual treatment cost for HCM patients                        | 65.53           | Echocardiogram, ECG, Holter consultation and hospitalization. |
| c_acons              | Cost of genetic counseling                                    | 51.95           | Per probando or relative                                      |
| c_rastreio_gen_pos   | Annual screening cost for genotype-positive relatives         | 48.24           | Echocardiogram, ECG, consultation                             |
| c_rastreio_sem_teste | Annual screening cost for untested relatives                  | 48.24           | Echocardiogram, ECG, consultation                             |
| eco_ras_gen_pos      | Number of screening per cycle for genotype-positive relatives | 0 or 1 per year | Echocardiogram, ECG, consultation                             |
| eco_ras_sem_teste    | Number of screening per cycle for untested relatives          | 0 or 1 per year | Echocardiogram, ECG, consultation                             |

---

##  Index Variables and Model Settings

| Variable Name     | Description                                      | Notes                     |
|:------------------|:-------------------------------------------------|:--------------------------|
| n_fam             | Number of relatives per proband                  | Base case: 4 (range: 1–6) |
| z_u_fam_neg       | Index for utility of genotype-negative relatives | Internal use in AMUA      |
| z_u_fam_gen_pos   | Index for utility of genotype-positive relatives | Internal use in AMUA      |
| z_u_fam_sem_teste | Index for utility of untested relatives          | Internal use in AMUA      |
| z_u_pac           | Index for utility of HCM patients                | Internal use in AMUA      |
| z_prob_gen        | Index for proband positivity probability         | Internal use in AMUA      |
| z_c_rastreio      | Index for screening cost tables                  | Internal use in AMUA      |
| z_eco_ras         | Index for number of echocardiograms              | Internal use in AMUA      |

---

## Methods

- **Deterministic and Probabilistic Simulations**
- **Time-dependent transitions** based on phenotype evolution and mortality
- **PSA inputs** modeled with Beta, Gamma, and Triangular distributions
- **WTP threshold**: US$ 7,421.15 per QALY (Brazilian guidelines)

---

All model outcomes were parameterized using specific dimensions with defined symbols and precision:
- Cost: Represented by the symbol $, with four decimal places.
- Utility: Represented by the symbol U, also with four decimal places.
- Screening: Represented by the symbol S, with one decimal place.
The benefit dimension in the cost-effectiveness analysis was defined as utility (QALYs),
and the willingness-to-pay (WTP) threshold was set at US$ 7,421.15 per QALY,
in accordance with Brazilian guidelines.

-----

## Standard Screening Diagnosis Equation

Given that genetic testing does not modify the penetrance of HCM, the probability of diagnosis under the standard screening strategy was estimated using a combined expression that accounts for both genotype-positive and genotype-negative probands. In AMUA, the formula was implemented as:

((p_prob * p_sanger) * p_fam_pos_fen[t, round(1)]) + ((1 - p_prob) * p_fam_sem_tes_fen[t, round(1)])

This reflects the probability that a family member is diagnosed in cycle t, either as a genotype-positive relative of a proband with an identified variant, or as a relative of a genotype-negative proband. The term for diagnosis in genotype-negative relatives with a positive proband was excluded, as it was assumed to be zero.


The probability of diagnosis under standard screening is estimated using:
((p_prob * p_sanger) * p_fam_pos_fen[t, round(1)]) + ((1 - p_prob) * p_fam_sem_tes_fen[t, round(1)])
This expression captures diagnoses among both genotype-positive and genotype-negative relatives.

---

## Notes

- All costs are in USD (US$) (Exchange rate: R$ 5.39 = US$ 1.00)
- Variables prefixed with `z_` are internal indices
- Full list of input variables is available in the documentation
- Cycle-dependent values were derived from time series tables using exact or truncated lookup.
- Parameters with dist_ prefix: Parameters beginning with dist_ represent probabilistic distributions used in the probabilistic sensitivity analysis (PSA). Utility values were modeled indirectly using 1 − Gamma(shape, rate) to represent disutility distributions by age group and clinical/genetic status. The probability of the proband carrying a pathogenic or likely pathogenic variant, as well as the number of relatives screened per proband, were modeled using Beta(α, β) distributions. The screening frequency was modeled using a triangular distribution. Aggregated variables such as dist_u_fam_gen_pos, dist_u_fam_sem_teste, dist_u_fam_neg, and dist_u_pac were defined using conditional expressions (iif) to apply the appropriate distribution based on the individual's age at each model cycle.
- This documentation covers both deterministic and probabilistic implementations.

---

## Tools Used
- AMUA (modeling platform)
- Microsoft Excel (parameter tables)
- R (for PSA scripts, if applicable)

---

## Intended Audience

Health economists, HTA researchers, policymakers, and public health professionals working with genetic testing and rare diseases.

---

## License

MIT License 
