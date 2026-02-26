---
# For reference on model card metadata, see the spec: https://github.com/huggingface/hub-docs/blob/main/modelcard.md?plain=1
# Doc / guide: https://huggingface.co/docs/hub/model-cards
{{ card_data }}
---

[Monthly Data track](https://docs.google.com/spreadsheets/d/1h-XOwoo0j8yyUnCtUFVTH2C4Ep8eSjISmqHu2hRg6hw/edit?gid=0#gid=0), every 10th of each month.

# Model Card for KRONOS

<!-- Provide a quick summary of what the model is/does. -->

KRONOS is a panel-agnostic foundation model for spatial proteomics, pretrained via DINO self-supervised learning on 690k image patches spanning 55 protein markers, 3 tissue types, and 2 imaging platforms. It enables label-efficient cell phenotyping, unsupervised tissue analysis, patient stratification, spatial biomarker discovery, and cross-cohort tissue search.

## Model Details

### Model Description

KRONOS is built on a Vision Transformer (ViT) backbone and introduces two key innovations for multiplex spatial proteomics data: (1) a shared channel-wise patch embedding stem that processes each protein marker channel independently, and (2) sinusoidal marker-identity embeddings indexed by marker ID, making the model natively generalizable to arbitrary, unseen marker panels without retraining. The model was pretrained using the DINO self-supervised framework and supports inference on high-dimensional multiplex images from platforms such as CODEX and IMC.

- **Developed by:** Yicheng Tao and Zhengjia Sun.
- **Funded by:** National Institutes of Health (NIH).
- **Shared by:** Liu Lab (UMich)
- **Model type:** Vision Transformer (ViT-S/16 and ViT-L/16) with channel-wise patch embedding and sinusoidal marker-identity embeddings; pretrained with DINO self-supervised learning.
- **Language(s) (NLP):** N/A (image-based model)
- **License:** CC-BY-NC-ND-4.0 — Non-commercial academic research use only. Commercial use or creation of derivative models requires prior approval.
- **Finetuned from model:** DINOv2 architecture (Meta AI); pretrained from scratch on spatial proteomics data.

### Model Sources

- **Repository:** https://github.com/yctao7/BioSpatialFM/tree/mim-dev
<!-- - **Paper:** https://arxiv.org/abs/2506.03373 -->
- **Demo:** See tutorials at https://github.com/yctao7/BioSpatialFM/tree/mim-dev/tutorials

## Uses

### Direct Use

KRONOS can be used out-of-the-box to extract patch-level, marker-level, and token-level embeddings from multiplex spatial proteomics images. These embeddings can be used directly for:

- Unsupervised cell clustering and phenotyping
- Tissue region classification and artifact detection
- Cross-cohort tissue search and retrieval
- Patient stratification and biomarker discovery

No fine-tuning is required for these exploratory analyses.

### Downstream Use

KRONOS embeddings can serve as feature representations for supervised downstream tasks including:

- Label-efficient cell type classification (e.g., with a linear probe or k-NN classifier)
- Treatment response prediction
- Survival analysis and patient subgroup discovery
- Integration with spatial transcriptomics or other multi-modal data

The model can also be fine-tuned on domain-specific datasets using the provided DINO fine-tuning script (`finetune_kronos.py`).

### Out-of-Scope Use

- **Commercial use** is not permitted without prior written approval from the authors.
- **Creation of derivative models** (including further fine-tuning, distillation, or adaptation for new products) requires prior approval.
- The model is not designed for natural RGB images and should not be used as a general-purpose image encoder outside the spatial proteomics domain.
- The model does not perform clinical diagnosis and should not be used for direct patient care decisions.

## Bias, Risks, and Limitations

- **Data bias:** KRONOS was trained on data from 5 institutions and 8 imaging platforms. Performance may degrade on data from institutions, tissue types, or platforms not represented in the training set.
- **Marker coverage:** The model supports up to 512 distinct marker IDs. Markers with IDs outside this range or markers not represented in pretraining may yield lower-quality embeddings.
- **Platform-specific artifacts:** Imaging artifacts vary across platforms (e.g., CODEX vs. IMC). The model was exposed to diverse artifact types during training, but novel artifact types may not be handled robustly.
- **Normalization sensitivity:** Performance depends on proper per-marker normalization using the mean and standard deviation values from the training distribution (provided in `marker_metadata.csv`). Using arbitrary normalization values may significantly reduce embedding quality.
- **Not a diagnostic tool:** KRONOS is a research tool. Its outputs should not be used as the sole basis for clinical decisions.

### Recommendations

Users should normalize input images using the marker-specific mean and standard deviation values provided in `marker_metadata.csv`. For markers not present in the pretraining set, users should assign unused marker IDs and evaluate embedding quality empirically. Fine-tuning on in-domain data is recommended for high-stakes downstream tasks. Users should be aware that model outputs reflect patterns in the training data, which may not generalize to all tissue types or clinical contexts.

## How to Get Started with the Model

```python
from kronos import create_model_from_pretrained
import torch

# Load pretrained model (requires HuggingFace access request)
model, precision, embedding_dim = create_model_from_pretrained(
    checkpoint_path="hf_hub:MahmoodLab/kronos",
    cache_dir="./model_assets",
)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model.to(device)

# Example: batch of 2 images with 10 markers, 224x224 pixels
batch = torch.randn(2, 10, 224, 224).to(device)
mean = torch.randn(10).to(device)
std = torch.randn(10).to(device)
batch = (batch - mean[None, :, None, None]) / std[None, :, None, None]

with torch.no_grad():
    patch_embeddings, marker_embeddings, token_embeddings = model(batch)

# patch_embeddings: [B, D] — one vector per tissue patch
# marker_embeddings: [B, C, D] — per-marker spatial average
# token_embeddings: [B, C, H', W', D] — full spatial token grid per marker
```

See the [tutorials](https://github.com/yctao7/BioSpatialFM/tree/mim-dev/tutorials) for full-scale examples on cell phenotyping, tissue search, patient stratification, and more.

## Training Details

### Training Data

KRONOS was pretrained on **690k image patches** extracted from multiplex spatial proteomics images. The training dataset spans:

- **55 protein markers** (including structural, immune, and functional markers)
- **3 tissue types** (pancreas head, body, and tail tissues)
- **2 imaging platforms** (CODEX and IMC)

Each patch was extracted at 256×256 pixels and cropped to 224×224 for training. Per-marker mean and standard deviation statistics were computed across the training set and are provided in `marker_metadata.csv`.

### Training Procedure

#### Preprocessing

Raw multiplex TIFF images were tiled into 256×256 pixel patches. Each marker channel was normalized independently using per-marker mean and standard deviation values. Empty or low-signal patches were filtered. Patches were stored as HDF5 files for efficient loading.

#### Training Hyperparameters

- **Training regime:** fp32
- **Framework:** PyTorch, DINO self-supervised learning
- **Architecture:** ViT-S/16 (384-dim) and ViT-L/16 (1024-dim)
- **Patch size:** 16×16 pixels
- **Optimizer:** AdamW
- **Learning rate:** 0.0005 with cosine decay and linear warmup
- **Weight decay:** 0.04 → 0.4 (cosine schedule)
- **Batch size:** 16 per GPU
- **Epochs:** 100
- **EMA momentum:** 0.996 (teacher network)
- **DINO output dim:** 65536
- **Local crops:** 8 (96×96), Global crops: 2 (224×224)
- **Register tokens:** 16
- **Block chunks:** 4 (for FSDP-compatible chunking)

#### Speeds, Sizes, Times

- **ViT-S/16 checkpoint size:** ~85 MB
- **ViT-L/16 checkpoint size:** ~300 MB
- **Inference speed:** Runs on a single consumer GPU (e.g., NVIDIA RTX 3090)
- **Training hardware:** Multi-GPU cluster (NVIDIA H200s)

## Evaluation

### Testing Data, Factors & Metrics

#### Testing Data

KRONOS was evaluated on three lymphoma datasets:

- **cHL** (Classical Hodgkin Lymphoma)
- **DLBCL-1** (Diffuse Large B-Cell Lymphoma, cohort 1)
- **DLBCL-2** (Diffuse Large B-Cell Lymphoma, cohort 2)

Evaluation used 4-fold cross-validation.

#### Factors

Evaluations were disaggregated by dataset (tissue type and cohort) and compared across model architectures. All models were evaluated under the same linear probe protocol.

#### Metrics

- **F1-Score** (macro)
- **Balanced Accuracy**
- **Average Precision**
- **AUROC**

### Results

Cell phenotyping benchmark (mean ± std over 4 folds):

| Dataset  | Model   | F1-Score            | Balanced Accuracy   | AUROC               |
|----------|---------|---------------------|---------------------|---------------------|
| cHL      | DINO-v2 | 0.5493 ± 0.0160     | 0.6210 ± 0.0121     | 0.9565 ± 0.0007     |
|          | UNI     | 0.4793 ± 0.0152     | 0.5570 ± 0.0136     | 0.9377 ± 0.0020     |
|          | CA-MAE  | 0.4553 ± 0.0105     | 0.5331 ± 0.0123     | 0.9271 ± 0.0048     |
|          | **KRONOS**  | **0.6807 ± 0.0066** | **0.7358 ± 0.0089** | **0.9758 ± 0.0010** |
| DLBCL-1  | DINO-v2 | 0.1932 ± 0.0316     | 0.2664 ± 0.0201     | 0.6623 ± 0.0161     |
|          | UNI     | 0.4073 ± 0.0529     | 0.5077 ± 0.0333     | 0.8474 ± 0.0191     |
|          | CA-MAE  | 0.3992 ± 0.0498     | 0.5041 ± 0.0314     | 0.8455 ± 0.0179     |
|          | **KRONOS**  | **0.6669 ± 0.0492** | **0.7402 ± 0.0309** | **0.9638 ± 0.0045** |
| DLBCL-2  | DINO-v2 | 0.2045 ± 0.0077     | 0.2980 ± 0.0226     | 0.6938 ± 0.0194     |
|          | UNI     | 0.4295 ± 0.0164     | 0.5511 ± 0.0377     | 0.8759 ± 0.0190     |
|          | CA-MAE  | 0.4231 ± 0.0185     | 0.5503 ± 0.0368     | 0.8748 ± 0.0193     |
|          | **KRONOS**  | **0.6912 ± 0.0162** | **0.7969 ± 0.0125** | **0.9759 ± 0.0023** |

#### Summary

KRONOS consistently outperforms general-purpose vision foundation models (DINOv2, UNI) and the spatial proteomics-specific baseline CA-MAE across all three datasets and all metrics, with particularly large gains on DLBCL-1 and DLBCL-2 where marker-aware representations are most critical.

## Model Examination

KRONOS produces interpretable marker-level embeddings (`marker_embeddings`) and spatial token grids (`token_embeddings`) that can be used for attention visualization and spatial biomarker mapping. The sinusoidal marker-identity embeddings can be inspected to understand how the model differentiates between protein markers. See Tutorial 6 (Tissue Search) and Tutorial 5 (Unsupervised Tissue Phenotyping) for examples of embedding space visualization.

## Environmental Impact

Carbon emissions can be estimated using the [Machine Learning Impact calculator](https://mlco2.github.io/impact#compute) presented in [Lacoste et al. (2019)](https://arxiv.org/abs/1910.09700).

- **Hardware Type:** NVIDIA H200 GPUs
- **Hours used:** [More Information Needed]
- **Cloud Provider:** [More Information Needed]
- **Compute Region:** [More Information Needed]
- **Carbon Emitted:** [More Information Needed]

## Technical Specifications

### Model Architecture and Objective

**Backbone:** Vision Transformer (ViT-S/16 or ViT-L/16) with the following modifications:
- **Channel-wise patch embedding:** A single shared Conv2D (1→D, kernel 16×16, stride 16) is applied independently to each marker channel; all marker patch tokens are concatenated before the transformer.
- **Sinusoidal marker-identity embeddings:** Each marker is assigned an integer ID; a sinusoidal embedding (matching the patch embedding dimension) is looked up by marker ID and added to the corresponding patch tokens, encoding protein identity without learned per-marker parameters.
- **Register tokens:** 16 register tokens are prepended after the CLS token to absorb global artifacts.
- **Block chunking:** Transformer blocks are split into 4 chunks for FSDP-compatible distributed training.

**Pretraining objective:** DINO (self-distillation with no labels). A student network is trained to match the output distribution of an exponential moving average (EMA) teacher network using a cross-entropy loss between multi-crop views. The teacher's output is centered and sharpened with a temperature schedule.

**Outputs at inference:**
- `patch_embeddings` — CLS token; shape `[B, D]`
- `marker_embeddings` — spatially averaged token features per marker; shape `[B, C, D]`
- `token_embeddings` — full spatial token grid per marker; shape `[B, C, H', W', D]`

### Compute Infrastructure

#### Hardware

- Pretraining: Multi-GPU cluster with NVIDIA H200 GPUs
- Inference: Any single consumer-grade GPU (e.g., NVIDIA RTX 3090 or better); CPU inference also supported

#### Software

- Python 3.10
- PyTorch ≥ 2.0.0
- xFormers ≥ 0.0.18 (for memory-efficient attention; falls back to standard attention if unavailable)
- huggingface_hub ≥ 0.28.1
- Ubuntu 22.04

## Citation

<!-- **BibTeX:**

```bibtex
@article{shaban2024foundation,
  title        = {A Foundation Model for Spatial Proteomics},
  author       = {Muhammad Shaban and Yuzhou Chang and Huaying Qiu and Yao Yu Yeo and Andrew H. Song and Guillaume Jaume and Yuchen Wang and Luca L. Weishaupt and Tong Ding and Anurag Vaidya and Abdallah Lamane and Daniel Shao and Mohammed Zidane and Yunhao Bai and Paige McCallum and Shuli Luo and Wenrui Wu and Yang Wang and Precious Cramer and Chi Ngai Chan and Pierre Stephan and Johanna Schaffenrath and Jia Le Lee and Hendrik A Michel and Caiwei Tian and Cristina Almagro-Perez and Sophia J. Wagner and Sharifa Sahai and Ming Y. Lu and Richard J. Chen and Andrew Zhang and Mark Edward M Gonzales and Ahmad Makky and Joey Lee and Hao Cheng and Maximilian Haist and Darci Phillips and Yuqi Tan and Garry P Nolan and W. Richard Burack and Jacob D Estes and Jonathan T.C. Liu and Toni K Choueiri and Neeraj Agarwal and Marc Barry and Scott J Rodig and Long Phi Le and Georg Gerber and Christian M. Schürch and Fabian J. Theis and Youn H Kim and Joe Yeong and Sabina Signoretti and Brooke Howitt and Lit-Hsin Loo and Qin Ma and Sizun Jiang and Faisal Mahmood},
  year         = {2025},
  note         = {Preprint},
  howpublished = {\url{https://arxiv.org/abs/2506.03373}},
}
```

**APA:**

Shaban, M., Chang, Y., Qiu, H., Yeo, Y. Y., Song, A. H., Jaume, G., ... & Mahmood, F. (2025). *A Foundation Model for Spatial Proteomics*. Preprint. https://arxiv.org/abs/2506.03373 -->

## Glossary

- **Spatial proteomics:** High-dimensional imaging techniques that measure the expression of dozens to hundreds of proteins at single-cell resolution within intact tissue sections.
- **Multiplex imaging:** Imaging modalities (e.g., CODEX and IMC) that simultaneously capture multiple protein markers in a single tissue section.
- **Marker ID:** An integer index assigned to each protein marker, used to look up sinusoidal marker-identity embeddings.
- **Panel-agnostic:** The ability of the model to process images with any combination of protein markers without retraining, enabled by marker-identity embeddings rather than fixed channel weights.
- **DINO:** Self-DIstillation with NO labels — a self-supervised learning framework where a student network learns to match the output of an EMA teacher network.
- **CLS token:** A learnable classification token prepended to the patch token sequence; its final representation serves as the global patch embedding.
- **Register tokens:** Additional learnable tokens that absorb global, non-local information and reduce artifacts in patch token representations.

## More Information

- Step-by-step tutorials for all major use cases are available at: https://github.com/yctao7/BioSpatialFM/tree/mim-dev/tutorials
- For questions about commercial licensing or derivative model use, contact the authors directly.

## Model Card Authors

Yicheng Tao

## Model Card Contact

- Yicheng Tao: yctao@umich.edu
