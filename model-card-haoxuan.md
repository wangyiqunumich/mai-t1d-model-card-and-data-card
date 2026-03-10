---
# For reference on model card metadata, see the spec: https://github.com/huggingface/hub-docs/blob/main/modelcard.md?plain=1
# Doc / guide: https://huggingface.co/docs/hub/model-cards
{{ card_data }}
---

[Monthly Data track](https://docs.google.com/spreadsheets/d/1h-XOwoo0j8yyUnCtUFVTH2C4Ep8eSjISmqHu2hRg6hw/edit?gid=0#gid=0), every 10th of each month.
 

# Model Card for {{ model_id | default("Model ID", true) }}

This model is an encoder-only Transformer for learning protein (ADT) representations from protein data. It supports:

Structured Masked Value Prediction for protein-value reconstruction under random masking and panel dropout (simulated missing-marker panels)

Supervised cell-type classification from a learned cell embedding (CLS token / pooled embedding)

## Model Details

### Model Description

**Inputs (per cell):**

* ADT count vector across **163 proteins** (CITE-seq antibody-derived tags)
* Optional donor/batch metadata used for dataset splitting (donor-level split during evaluation)

**Preprocessing:**

* Per-cell **CLR** transformation on ADT counts:

  * `x = log1p(count)`
  * `x = x - mean(x over proteins)`

**Tokenization / Representation:**

* Each protein is treated as a **token** (sequence length = number of proteins; + optional `[CLS]`)
* Hyper-token embedding components (Stage-0):

  * `E_protein(id)` (learned protein identity embedding)
  * `E_value(v)` (continuous value encoder on CLR values; implemented as an MLP in current version)
  * `E_mod(CITE)` (single modality embedding placeholder; for future multi-modality expansion)
  * (Optional) donor/batch embedding exists in the design, but **its use requires care** because embeddings may become donor-dominated; current benchmarks therefore report donor effects explicitly.

**Backbone:**

* Encoder-only Transformer

**Outputs:**

* **Cell embedding** (CLS token or pooled embedding) for downstream use
* **SMVP head**: predicts masked protein values (regression)
* **Classification head**: predicts cell-type label (cross-entropy)

**Objectives:**

* `L = L_smvp + λ * L_cls`

  * `L_smvp`: masked-value regression loss (weighted MSE over masked tokens)
  * `L_cls`: supervised cell-type classification loss

* **Developed by:** Liu Lab, University of Michigan (TODO: confirm exact lab naming)

* **Shared by:** Haoxuan Zeng

* **Model type:** Protein (ADT) encoder-only Transformer for single-cell representation learning

* **License:** Apache 2.0 (TODO: confirm)
* 
* **Finetuned from model:** Not applicable / trained from scratch (Stage-0)
### Model Sources [optional]

<!-- Provide the basic links for the model. -->

- **Repository:** {{ repo | default("[More Information Needed]", true)}}
- **Paper [optional]:** {{ paper | default("[More Information Needed]", true)}}
- **Demo [optional]:** {{ demo | default("[More Information Needed]", true)}}

## Uses

<!-- Address questions around how the model is intended to be used, including the foreseeable users of the model and those affected by the model. -->

### Direct Use

This model is intended for **research use** on ADT data.

Typical direct uses:

* **Cell embedding extraction** for clustering, visualization (UMAP), retrieval, and downstream ML
* **Cell-type classification** (supervised head provided)
* **Protein imputation / panel completion** under random masking and simulated panel dropout


### Downstream Use [optional]

Potential downstream uses:

* Cross-panel harmonization (different antibody panels)
* Cross-modality alignment (ADT ↔ RNA/ATAC/IMC/CODEX/flow) using shared anchors
* Donor-level phenotype prediction via pooled cell embeddings
* Clone-aware modeling when paired with TCR/BCR (if available in dataset)

### Out-of-Scope Use

<!-- This section addresses misuse, malicious use, and uses that the model will not work well for. -->

{{ out_of_scope_use | default("[More Information Needed]", true)}}

## Bias, Risks, and Limitations

<!-- This section is meant to convey both technical and sociotechnical limitations. -->

{{ bias_risks_limitations | default("[More Information Needed]", true)}}

### Recommendations

<!-- This section is meant to convey recommendations with respect to the bias, risk, and technical limitations. -->

{{ bias_recommendations | default("Users (both direct and downstream) should be made aware of the risks, biases and limitations of the model. More information needed for further recommendations.", true)}}

## How to Get Started with the Model

Use the code below to get started with the model.

{{ get_started_code | default("[More Information Needed]", true)}}

## Training Details

### Training Data

<!-- This should link to a Dataset Card, perhaps with a short stub of information on what the training data is all about as well as documentation related to data pre-processing or additional filtering. -->

{{ training_data | default("[More Information Needed]", true)}}

### Training Procedure

<!-- This relates heavily to the Technical Specifications. Content here should link to that section when it is relevant to the training procedure. -->

#### Preprocessing [optional]

{{ preprocessing | default("[More Information Needed]", true)}}


#### Training Hyperparameters

- **Training regime:** {{ training_regime | default("[More Information Needed]", true)}} <!--fp32, fp16 mixed precision, bf16 mixed precision, bf16 non-mixed precision, fp16 non-mixed precision, fp8 mixed precision -->

#### Speeds, Sizes, Times [optional]

<!-- This section provides information about throughput, start/end time, checkpoint size if relevant, etc. -->

{{ speeds_sizes_times | default("[More Information Needed]", true)}}

## Evaluation

<!-- This section describes the evaluation protocols and provides the results. -->

### Testing Data, Factors & Metrics

#### Testing Data

<!-- This should link to a Dataset Card if possible. -->

{{ testing_data | default("[More Information Needed]", true)}}

#### Factors

<!-- These are the things the evaluation is disaggregating by, e.g., subpopulations or domains. -->

{{ testing_factors | default("[More Information Needed]", true)}}

#### Metrics

<!-- These are the evaluation metrics being used, ideally with a description of why. -->

{{ testing_metrics | default("[More Information Needed]", true)}}

### Results

{{ results | default("[More Information Needed]", true)}}

#### Summary

{{ results_summary | default("", true) }}

## Model Examination [optional]

<!-- Relevant interpretability work for the model goes here -->

{{ model_examination | default("[More Information Needed]", true)}}

## Environmental Impact

<!-- Total emissions (in grams of CO2eq) and additional considerations, such as electricity usage, go here. Edit the suggested text below accordingly -->

Carbon emissions can be estimated using the [Machine Learning Impact calculator](https://mlco2.github.io/impact#compute) presented in [Lacoste et al. (2019)](https://arxiv.org/abs/1910.09700).

- **Hardware Type:** {{ hardware_type | default("[More Information Needed]", true)}}
- **Hours used:** {{ hours_used | default("[More Information Needed]", true)}}
- **Cloud Provider:** {{ cloud_provider | default("[More Information Needed]", true)}}
- **Compute Region:** {{ cloud_region | default("[More Information Needed]", true)}}
- **Carbon Emitted:** {{ co2_emitted | default("[More Information Needed]", true)}}

## Technical Specifications [optional]

### Model Architecture and Objective

{{ model_specs | default("[More Information Needed]", true)}}

### Compute Infrastructure

{{ compute_infrastructure | default("[More Information Needed]", true)}}

#### Hardware

{{ hardware_requirements | default("[More Information Needed]", true)}}

#### Software

{{ software | default("[More Information Needed]", true)}}

## Citation [optional]

<!-- If there is a paper or blog post introducing the model, the APA and Bibtex information for that should go in this section. -->

**BibTeX:**

{{ citation_bibtex | default("[More Information Needed]", true)}}

**APA:**

{{ citation_apa | default("[More Information Needed]", true)}}

## Glossary [optional]

<!-- If relevant, include terms and calculations in this section that can help readers understand the model or model card. -->

{{ glossary | default("[More Information Needed]", true)}}

## More Information [optional]

{{ more_information | default("[More Information Needed]", true)}}

## Model Card Authors [optional]

{{ model_card_authors | default("[More Information Needed]", true)}}

## Model Card Contact

{{ model_card_contact | default("[More Information Needed]", true)}}
