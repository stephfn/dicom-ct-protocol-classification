# DICOM CT Protocol Classification

Classifying inconsistent DICOM CT protocol metadata using rule-based text matching and a large language model.

## Project Overview

CT examinations can be described differently across institutions, vendors, and imaging systems. Terms such as `CHEST`, `Thx_Native`, `CT Chest Routine`, and `LOW DOSE THROUGH LUNGS` may represent related examinations while using very different naming conventions. This inconsistency makes it difficult to group imaging studies reliably for research, analytics, and other downstream applications.

This project explores whether descriptive DICOM metadata can be used to automatically standardize CT series into a smaller reference taxonomy. I compared an interpretable rule-based classifier with a zero-shot large language model (LLM) approach.

## Research Questions

1. Can a compact reference taxonomy be created from metadata in a public CT collection?
2. How accurately can an interpretable rule-based classifier reproduce the reference taxonomy?
3. Can a pretrained LLM classify the same metadata without access to the manually assigned reference label, and how do its errors differ from the rule-based approach?

## Data

The project uses CT series metadata from the **LIDC-IDRI** collection available through **The Cancer Imaging Archive (TCIA)**.

- 1,018 CT series retrieved
- 573 series contained usable protocol-related descriptive metadata
- Image pixels were not used for classification

The analysis incorporated available information from:

- `SeriesDescription`
- `ProtocolName`
- `StudyDesc`
- `BodyPartExamined`
- `Manufacturer`

These fields were combined to preserve as much descriptive information as possible when individual fields were incomplete.


### Data Coverage and Selection Bias

Of the 1,018 retrieved CT series, 573 (56.3%) contained sufficient descriptive metadata for the classification experiment. The remaining 445 series (43.7%) were excluded because their available metadata did not support reliable assignment to the working protocol taxonomy.

This exclusion is an important limitation. Series with complete or informative metadata may differ systematically from those with incomplete descriptions. As a result, the reported classification performance applies to the retained subset and may overestimate performance across the full collection or in environments with less consistent metadata documentation.

A future evaluation should compare included and excluded records across available metadata characteristics and report both classification performance and the proportion of records that can be classified.


## Reference Taxonomy

Exploratory analysis of the metadata was used to develop eight working protocol classes:

- `LUNG_SCREENING`
- `CHEST_NONCONTRAST`
- `CHEST_CONTRAST`
- `CHEST_UNSPECIFIED`
- `CAP`
- `CTA_PE`
- `CT_GUIDED_BIOPSY`
- `UNKNOWN`

The taxonomy was developed specifically for this exploratory study and should not be interpreted as a universal CT protocol standard.

## Methods

### Rule-Based Classifier

The baseline classifier uses Boolean conditions and regular-expression pattern matching to identify terminology associated with each reference class.

Rule precedence was important because specific concepts such as pulmonary embolism or CT-guided biopsy needed to be evaluated before broader chest terminology.

### LLM Classifier

A zero-shot classification experiment was performed using **OpenAI GPT-5.6 Luna** through the Responses API.

The model received:

- Combined DICOM metadata
- Definitions of the eight protocol classes
- Instructions to use only the supplied metadata
- Instructions to return `UNKNOWN` when the evidence was insufficient

The manually assigned reference label was not provided to the model.

## Evaluation

Both approaches were evaluated against the same reference labels using:

- Accuracy
- Precision
- Recall
- F1 score
- Macro and weighted averages
- Confusion matrices
- Per-class error analysis

## Results

| Approach | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| Rule-Based | 97.9% | 0.947 | 0.977 |
| GPT-5.6 Luna | 89.9% | 0.828 | 0.892 |

The rule-based classifier correctly classified **561 of 573 series**, while the LLM correctly classified **515 of 573 series** according to the reference labels.

The error analysis was particularly informative. The LLM sometimes used contextual information more aggressively than the deterministic rules. For example, ACRIN and low-dose terminology led the model to infer lung screening in records that had been assigned more conservative reference labels. Other disagreements led to reconsideration of some records originally labeled `UNKNOWN`.

These findings demonstrated that disagreement with a reference label does not necessarily represent a random model failure. It can also expose ambiguity in the metadata, limitations in the taxonomy, or uncertainty in the reference standard itself.


## Key Takeaways

The rule-based approach performed extremely well when terminology explicitly matched the reference taxonomy. Its decisions were also transparent and reproducible.

The LLM achieved lower overall agreement but provided a different form of value by interpreting contextual relationships that were not always represented in the deterministic rules.

The results suggest that future work could explore a hybrid workflow in which interpretable rules handle explicit terminology while LLM-assisted analysis and human review help investigate ambiguous or previously unseen cases.

## Limitations

This was an exploratory study with several important limitations:

- Reference labels were assigned by a single reviewer.
- Class sizes were imbalanced.
- Some protocol categories contained very few examples.
- LIDC-IDRI represents thoracic CT and does not represent the full range of CT examinations.
- Results have not been validated across multiple institutions or vendors.
- The project evaluates metadata classification only and does not evaluate diagnosis, image quality, or clinical appropriateness.

The reference taxonomy should therefore be considered a working taxonomy rather than a finalized clinical or operational standard.


## Ethical Assessment

Although this project uses a publicly available research dataset and focuses on descriptive CT metadata rather than diagnostic image interpretation, several ethical and operational risks remain relevant.

The following assessment identifies potential concerns, estimates their likelihood and impact, and proposes mitigation measures. These ratings are qualitative assessments for an exploratory research workflow, not validated estimates of clinical risk.

| Ethical Concern | Likelihood | Impact | Proposed Mitigation |
|---|---|---|---|
| Privacy and external API transmission | Possible | High | Verify that all transmitted metadata are appropriately de-identified and permitted for external processing. Minimize the information included in API requests and review applicable data-sharing policies. |
| Reference-label validity | Likely | Moderate to High | Obtain independent review from qualified imaging professionals, document disagreements, and establish a process for adjudicating ambiguous labels. |
| Selection bias from missing metadata | Likely | Moderate to High | Analyze differences between included and excluded records, report classification coverage, and evaluate performance on additional datasets. |
| Class imbalance | Likely | Moderate | Report per-class precision, recall, F1, and support rather than relying on overall accuracy alone. Expand evaluation of underrepresented categories. |
| Misuse or overinterpretation | Possible | High | Clearly identify the system as exploratory research, avoid claims of clinical validation, and require human oversight and independent validation before operational use. |

### Privacy and Data Governance

The source collection is publicly available through The Cancer Imaging Archive (TCIA), but public availability does not eliminate the need to assess privacy and data-sharing risks.

DICOM metadata can contain identifying or sensitive information, particularly in free-text fields. Because the LLM experiment used an external API, the privacy implications of transmitting metadata are relevant even though image pixels were not used.

Before extending this workflow to institutional or patient-level data, a formal review of de-identification procedures, applicable data-sharing agreements, API data-handling policies, and institutional requirements would be necessary.

### Data Validity and Representation

The analysis excluded 445 series with insufficient descriptive metadata. This filtering decision was necessary for the exploratory comparison but may have introduced selection bias.

The reference taxonomy was also developed from the available data, and labels were assigned by a single reviewer. These decisions influence what the models were evaluated against and limit the strength of conclusions about classification correctness.

The results should therefore be interpreted as agreement with a working reference taxonomy rather than evidence of independently validated clinical accuracy.

### Responsible Interpretation and Use

The project is intended to investigate metadata standardization for research and analytics. It does not determine whether a CT examination was clinically appropriate, whether imaging findings are correct, or whether a protocol should be ordered.

A production implementation would require additional validation, documented governance procedures, and human review of ambiguous cases. Automated classifications should not be treated as authoritative clinical decisions without such safeguards.

## Future Work

Future research could include:

- Independent labeling by multiple imaging experts
- Adjudication of ambiguous reference labels
- Validation on additional institutions and CT datasets
- Evaluation of additional vendors and naming conventions
- Few-shot or more structured LLM prompting
- Human review of disagreements between classifiers
- Development of a hybrid rule-based and LLM-assisted workflow
- Iterative refinement of rules using validated contextual findings


## Reproducibility

The analysis was developed in a Jupyter Notebook using Python. The notebook documents the process of retrieving CT series metadata, examining missing values, developing a working reference taxonomy, applying classification methods, and evaluating their results.

### Software and Dependencies

The notebook uses the following Python packages:

- `pandas` for data preparation and analysis
- `tcia_utils` for retrieving TCIA/NBIA series metadata
- `scikit-learn` for classification metrics and confusion matrices
- `matplotlib` for visualizations
- `openai` for the LLM classification experiment

Python's built-in `re`, `os`, `time`, and `getpass` modules are also used.

The repository does not currently provide a pinned dependency environment. Package versions and compatibility should be verified before attempting to reproduce the analysis.

### Data Access

The source data are retrieved programmatically from the public LIDC-IDRI collection through TCIA/NBIA.

The notebook requests CT series metadata and constructs a pandas DataFrame from the returned records. It then examines metadata completeness and retains series containing at least one of the following descriptive fields:

- `SeriesDescription`
- `ProtocolName`
- `StudyDesc`

The original analysis retrieved 1,018 CT series and retained 573 for classification. Because the source is accessed through an external service, future retrievals may differ if the collection or API responses change.

### Running the Notebook

1. Clone or download this repository.
2. Set up a Python environment with Jupyter Notebook and the required dependencies.
3. Open `01_lidc_metadata_exploration.ipynb`.
4. Review the notebook's data retrieval, filtering, and classification steps.
5. Create a local `data/processed/` directory before running cells that export CSV files.
6. Run the notebook cells in sequence, reviewing the intermediate outputs and reference-labeling steps.
7. Compare the resulting classification metrics with those reported in this README.

The notebook contains exploratory steps used to develop the working taxonomy. These steps should be reviewed rather than treated as a fully automated or independently validated labeling process.

### Reproducing the LLM Experiment

The LLM experiment uses the OpenAI Responses API and requires a valid API key.

The notebook requests the API key interactively using `getpass`, rather than requiring the key to be stored directly in the notebook.

Reproducing the LLM experiment may incur API charges. Results may also differ because of model availability, model updates, and variation in generated responses.

The reported LLM results represent the original experimental evaluation and should not be interpreted as evidence of reproducibility across repeated runs.

### Reproducibility Limitations

The repository contains the analysis notebook but does not currently provide a fully automated, version-pinned reproduction environment.

The reference taxonomy was developed through exploratory analysis, and reference labels were assigned by a single reviewer. Reproducing the exact reference-labeling decisions may therefore require additional documentation or review.

The reported evaluation also applies only to the 573 retained series. Results may differ when using updated metadata, alternative reference labels, or additional datasets.

## Proposed Implementation Plan

A future implementation could use a staged approach:

1. **Metadata intake and validation:** Collect permitted DICOM metadata, verify de-identification where applicable, and identify missing or inconsistent fields.
2. **Rule-based classification:** Apply documented classification rules to records containing explicit protocol terminology.
3. **Ambiguity detection:** Flag records with insufficient information, conflicting terminology, or classifications requiring further review.
4. **LLM-assisted investigation:** Use an approved LLM workflow to investigate selected ambiguous records, subject to appropriate privacy and governance controls.
5. **Human review:** Have qualified reviewers evaluate disagreements and approve any changes to reference labels or classification rules.
6. **Ongoing evaluation:** Monitor classification coverage, per-class performance, error patterns, and changes in metadata terminology.

This workflow is a proposed extension of the exploratory study and has not been implemented or clinically validated.

## Repository Contents

```text
dicom-ct-protocol-classification/
│
├── 01_lidc_metadata_exploration.ipynb
├── README.md
└── .gitignore
