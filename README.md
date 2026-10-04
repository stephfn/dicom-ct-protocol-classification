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

## Repository Contents

```text
dicom-ct-protocol-classification/
│
├── 01_lidc_metadata_exploration.ipynb
├── README.md
└── .gitignore
