# Benchmarking Fine-Tuning Strategies for Pretrained Protein Language Models

**Virginia Tech — NeurIPS 2024 Course Project**  
Shoumik Bisoi (`shoumik77@vt.edu`) · Shruti Narayanan (`shruti22@vt.edu`)

---

## Overview

Protein sequence databases now contain over 200 million sequences, yet only a small fraction have been experimentally characterized. This project benchmarks multiple fine-tuning strategies for pretrained protein language models (PLMs) on **enzyme commission (EC) classification** — predicting what enzymatic function a protein performs directly from its amino acid sequence.

We compare linear probing, full fine-tuning, and LoRA across two PLM backbones of different scales: ESM-2 (8M parameters) and ProtT5-XL (1.2B parameters), evaluating the tradeoff between predictive performance and computational efficiency.

---

## Results

| Model | Strategy | Val Accuracy | Macro F1 | Runtime |
|-------|----------|:---:|:---:|:---:|
| ESM-2 (8M) | Linear Probing | 81.2% | 0.801 | 31.3s |
| ESM-2 (8M) | Full Fine-Tuning | 84.5% | 0.835 | 95.9s |
| ProtT5-XL (1.2B) | Linear Probing | 91.6% | 0.929 | 3.2s |
| ProtT5-XL (1.2B) | LoRA | **98.0%** | **0.981** | 4612s |

**Key finding:** ProtT5-XL linear probing (frozen encoder, 3s training) outperforms ESM-2 full fine-tuning (all weights updated, 96s training), suggesting model scale matters more than fine-tuning strategy. LoRA achieves the best performance overall while training only 0.32% of ProtT5-XL's parameters.

---

## Repo Structure

```
protein-ec-classification/
├── README.md
├── references.bib              # BibTeX references
├── paper/                      # NeurIPS-style paper (Overleaf)
├── data/
│   └── split_info.json         # Dataset split metadata
├── shoumik/
│   └── protT5_experiments.ipynb   # ProtT5-XL: linear probing + LoRA
└── shruti/
    └── esm2_experiments.ipynb     # ESM-2: linear probing + full fine-tuning
```

---

## Dataset

We use the [`lightonai/SwissProt-EC-leaf`](https://huggingface.co/datasets/lightonai/SwissProt-EC-leaf) dataset from HuggingFace, derived from the Swiss-Prot section of UniProt.

- **Full dataset:** 178,302 train / 23,010 val / 22,183 test sequences
- **Benchmark subset:** Top-20 most frequent EC classes, filtered to 5,000 train / 1,000 val / 1,000 test
- **Task:** Single-label EC classification (first label used for multi-label sequences)
- **Classes:** 20 (remapped to integer IDs 0–19)

---

## Models

### Shoumik — ProtT5-XL (`Rostlab/prot_t5_xl_uniref50`)
- 1.2 billion parameter T5-based encoder
- Sequences space-separated at amino acid level (`M K T A Y...`)
- Rare amino acids (U, Z, O, B) replaced with X
- Max sequence length: 256 tokens
- Loaded in fp16 to fit on free-tier GPU

### Shruti — ESM-2 (`facebook/esm2_t6_8M_UR50D`)
- 8 million parameter RoBERTa-based encoder
- Standard tokenization, no preprocessing needed
- Max sequence length: 256 tokens
- Trained via HuggingFace Trainer API

---

## Fine-Tuning Strategies

| Strategy | Description | Models |
|----------|-------------|--------|
| **Linear Probing** | Freeze entire encoder, train only classifier head | ESM-2, ProtT5-XL |
| **Full Fine-Tuning** | Update all encoder + classifier parameters end-to-end | ESM-2 |
| **LoRA** | Insert low-rank adapters (r=16) into attention layers, train 0.32% of params | ProtT5-XL |

> **Note:** Full fine-tuning of ProtT5-XL was infeasible on free-tier hardware (15GB VRAM) due to the model's 1.2B parameter size.

---

## Setup

### Requirements
```bash
pip install transformers datasets sentencepiece peft scikit-learn torch
```

### Running ProtT5-XL experiments (Shoumik)
Open `shoumik/protT5_experiments.ipynb` in Google Colab with a T4 GPU runtime and run all cells in order. The notebook covers:
1. Environment setup and GPU check
2. Dataset loading and filtering to top-20 EC classes
3. ProtT5-XL loading and sequence preprocessing
4. Exploratory data analysis
5. Embedding pre-computation and caching
6. Linear probing (10 epochs)
7. LoRA fine-tuning (3 epochs, requires Kaggle or Colab Pro for memory)

### Running ESM-2 experiments (Shruti)
Open `shruti/esm2_experiments.ipynb` in Google Colab with a T4 GPU runtime and run all cells in order. The notebook covers:
1. ESM-2 loading and tokenization
2. Dataset loading and filtering
3. Linear probing (2 epochs)
4. Full fine-tuning (2 epochs)

---

## Evaluation Metrics

- **Accuracy** — fraction of correctly classified sequences
- **Macro F1** — unweighted mean F1 across all 20 EC classes (primary metric due to class imbalance)
- **Runtime** — wall-clock training time in seconds

---

## Hardware

All experiments run on Google Colab (free tier) with an NVIDIA T4 GPU (15GB VRAM). LoRA experiments were run on Kaggle with a P100 GPU due to memory constraints with ProtT5-XL.

---

## Citation

If you use this code or results, please cite the key works this project builds on:

```bibtex
@article{lin2023esm,
  author  = {Lin, Zeming and others},
  title   = {Evolutionary-Scale Prediction of Atomic-Level Protein Structure with a Language Model},
  journal = {Science},
  year    = {2023}
}

@article{elnaggar2021prottrans,
  author  = {Elnaggar, Ahmed and others},
  title   = {{ProtTrans}: Towards Cracking the Language of Life's Code},
  journal = {IEEE Transactions on Pattern Analysis and Machine Intelligence},
  year    = {2021}
}

@misc{hu2021lora,
  author = {Hu, Edward J. and others},
  title  = {{LoRA}: Low-Rank Adaptation of Large Language Models},
  year   = {2021},
  eprint = {2106.09685}
}
```

---

## License

This project was completed as a course assignment at Virginia Tech. Code is provided for educational purposes.
