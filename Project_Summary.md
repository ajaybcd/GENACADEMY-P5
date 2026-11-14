IT Helpdesk Ticket Router Fine-Tuning Project Summary
Workflow Summary & Evaluation Report

# 1. Project Objectives & Architecture
The objective of this project was to automate IT ticket routing across seven business queues to reduce manual triaging overhead, latency, and cost at scale. Instead of relying on expensive frontier APIs (e.g., GPT-4o), we adapted a lightweight local foundation model to run efficiently in resource-constrained production environment parameters.

# 2. Data Engineering & Synthetic Augmentation
Our original training split derived from a stratified 80/20 train/validation breakdown of historical ticket logs. Initial testing highlighted an acute data imbalance challenge: the high-priority 'Active Directory' category was significantly underrepresented (only 39 instances in the original training set), leading to a dismal recall rate of 22.2%.
* Data Augmentation: We programmatically generated 50 synthetic, high-quality, domain-specific tickets representing Active Directory workflows (such as login lockout issues, provisioning, offboarding, and security group permissions).
* Dataset Balance: The final training set grew to 518 structured records compiled into standard ShareGPT conversational schemas (TRAIN.json), registered in dataset_info.json.

# 3. Fine-Tuning Workflow & Weight Fusion
Fine-tuning was executed locally within Google Colab's T4 GPU environment using the LLaMA-Factory interface with the following parameters:
* Precision and Optimizations: FP16 half-precision, cosine scheduler learning decay, 8-bit quantized baseline projection matrices.
* Weight Fusion (Merging): Following successful training convergence (final training loss dropped to 0.1399), we detached the PEFT wrapper and mathematically fused the isolated LoRA adapter weights back into the primary base model tensors using .merge_and_unload(). This eliminated all runtime overhead.
* Standalone Output Directory: The fully consolidated standalone model was exported to /content/qwen3_merged, allowing for immediate production packaging.

# 4. Performance Evaluation Summary

Next Steps: To increase Active Directory recall above our SLA-mandated >85% target threshold, future iterations should expand classification training bounds or enforce positive bias weightings on the token logit index during runtime inference.

| Parameter/Aspect | Project Selection |
| --- | --- |
| Foundation Base Model | Qwen/Qwen3-1.7B-Base |
| Sourcing Pipeline | Downloaded dynamically from Hugging Face Hub directly into local cached directory (/content/) |
| Adaptation Method | Parameter-Efficient Fine-Tuning (PEFT) using LoRA (Rank=16, Alpha=32) over frozen foundation weights |


| Model Stage | Overall Validation Acc | O365 F1-Score | Active Directory Recall |
| --- | --- | --- | --- |
| Base (Zero-Shot) | 31.6% | 15.4% | 0.0% |
| LoRA Merged Fine-Tuned | 76.9% | 100% | 41.7% / 22.2% (Raw Indexing) |
