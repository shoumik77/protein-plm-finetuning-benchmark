Benchmarking Fine-Tuning Strategies for Protein Language Models
This project benchmarks adaptation strategies for pretrained protein language models (PLMs) on enzyme commission (EC) classification using the SwissProt EC dataset.
Models evaluated:

ESM-2 (facebook/esm2_t6_8M_UR50D) — linear probing and full fine-tuning
ProtT5-XL (Rostlab/prot_t5_xl_uniref50) — linear probing and LoRA

Key results:
ModelStrategyVal AccuracyMacro F1ESM-2Linear Probing81.2%0.801ESM-2Full Fine-Tuning84.5%0.835ProtT5-XLLinear Probing91.6%0.929ProtT5-XLLoRA98.0%0.981
Dataset: lightonai/SwissProt-EC-leaf (top-20 EC classes, 5,000 training samples)
Authors: Shoumik Bisoi & Shruti Narayanan — Virginia Tech

