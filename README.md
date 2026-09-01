# GSSPO: Group Sentence-Specified Sequence Policy Optimization for LLM Personality Alignment

[![Paper](https://img.shields.io/badge/Findings%20of%20EMNLP-2026-b31b1b)](https://2026.emnlp.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue)](LICENSE)

Official repository for **"GSSPO: Group Sentence-Specified Sequence Policy Optimization for LLM Personality Alignment"**, accepted to **Findings of the Association for Computational Linguistics: EMNLP 2026**.

Shurui Zhang\* (HKUST) · Zikai Li\* (CUHK) &nbsp;&nbsp;<sub>\*Equal contribution.</sub>

> **Status.** This repository currently hosts the project description only. The core training code, the PACF construction pipeline, and the evaluation scripts are being prepared for the camera-ready release — see [Release plan](#release-plan).

---

## Abstract

Large Language Models (LLMs) have demonstrated strong reasoning capabilities, yet achieving nuanced personality alignment for simulating human conversation remains challenging. This difficulty stems from two bottlenecks: the scarcity of scalable, privacy-respecting training data and the limited ability of conventional reinforcement learning (RL) rewards to capture subtle personality-relevant signals. In this work, we introduce a unified framework to address these issues. First, we propose the **Personality Alignment Choice Forge (PACF)**, an automatic pipeline that transforms existing conversational datasets into structured, multiple-choice benchmarks, constraining the model's action space and enabling scalable evaluation grounded in observed human choices. Second, we present **Group Sentence-Specified Sequence Policy Optimization (GSSPO)**, an RL algorithm designed to address the coarse credit assignment problem in methods that apply a uniform reward across all tokens. By integrating stable sequence-level updates with differentiated, rubric-guided token-level feedback, GSSPO assigns reward signals to more localized spans of the response. This targeted approach mitigates optimization instabilities caused by undifferentiated reward signals, leading to more effective training in our experiments.

## Method

The framework has two complementary components.

**PACF — Personality Alignment Choice Forge.** A dataset-agnostic pipeline that reformulates open-ended conversational alignment as a discrete multiple-choice decision problem grounded in observed human selections. It runs in three stages: (A) conversation preprocessing and structural compression, (B) candidate response generation and distractor-bank construction, and (C) multiple-choice packaging with in-domain and held-out-persona splits. Constraining the effective action space makes the reward automatically verifiable and the evaluation scalable.

**GSSPO — Group Sentence-Specified Sequence Policy Optimization.** A hybrid RL objective that keeps the stability of sequence-level updates while adding localized, rubric-guided token-level credit:

$$
\hat{A}^{*}_{i} = \underbrace{\frac{\hat{A}_i}{|y_i|}}_{\text{uniform base}} + \underbrace{\alpha_t (\mathbf{b}_i \cdot \mathbf{r})}_{\text{token-level correction}}
$$

An external rubric grader scores each sampled response against a criteria set; the resulting criterion vector $\mathbf{b}_i \in \lbrace -1, 0, 1 \rbrace^n$ is mapped back to token spans through a three-tier localization cascade (exact tokenizer matching → fuzzy text matching → uniform fallback), giving $+r$ on satisfied criteria of correct responses and $-r$ on violated criteria of incorrect ones. The blend coefficient $\alpha_t$ follows a three-phase schedule — **Full Guidance** (rapid rule acquisition) → **Guidance Decay** (reduce dependence on the external judge) → **Free Exploration** (refine general reasoning without token-level constraints).

## Benchmarks

Three Choice-Alignment (CA) benchmarks. PRISM-CA and Big5Chat-CA are built by applying PACF to [PRISM](https://arxiv.org/abs/2404.16019) and [Big5-Chat](https://arxiv.org/abs/2410.16491); Twin-2K-CA is built directly from [Twin-2K-500](https://arxiv.org/abs/2505.17479), keeping the original survey options and each participant's recorded answer as ground truth.

| Dataset | Train | Validation | Test |
|---|---|---|---|
| PRISM-CA | 5,704 | 545 | 5,798 |
| Big5Chat-CA | 2,397 | 100 | 540 |
| Twin-2K-CA | 3,302 | 825 | 1,658 |

Twin-2K-CA's test set uses a held-out-persona split for profile-level generalization.

## Results

On the Qwen3 model family, GSSPO achieves the best personality-alignment accuracy on every benchmark and scale in the SFT+RL regime, and leads the other RL methods on five of six in the SFT-Free regime. A cross-family replication on Llama-3.2-3B and Llama-3.1-8B shows the same ordering, locating the advantage in the credit-assignment objective rather than in a particular backbone. The paper additionally reports pairwise open-ended evaluation with an LLM judge, cross-domain transfer (Medical, Instruction Following, STEM), judge-agreement analysis, and token-localization reliability.

Full tables and analysis are in the paper.

## Release plan

- [ ] GSSPO trainer and rubric reward worker, documented against Section 3.2 / Algorithm 1 of the paper
- [ ] PACF construction scripts and dataset statistics
- [ ] Training and evaluation configs, multi-GPU quickstart
- [ ] The three CA benchmarks (PRISM-CA, Big5Chat-CA, Twin-2K-CA)

## Citation

```bibtex
@inproceedings{zhang2026gsspo,
  title     = {{GSSPO}: Group Sentence-Specified Sequence Policy Optimization for {LLM} Personality Alignment},
  author    = {Zhang, Shurui and Li, Zikai},
  booktitle = {Findings of the Association for Computational Linguistics: EMNLP 2026},
  year      = {2026}
}
```

The ACL Anthology entry will replace this once available.

## License

Code in this repository is released under the [Apache License 2.0](LICENSE).

The CA benchmarks are derived artifacts and are released for **non-commercial research use only**, following the terms of their sources: PRISM is dual-licensed (human-written text CC-BY-4.0, model responses CC-BY-NC-4.0) and prohibits re-identification; Big5-Chat is Apache-2.0; Twin-2K-500 is CC-BY-4.0. Model-generated distractors are additionally subject to the generating providers' terms.

## Contact

- Shurui Zhang — `szhangfa@connect.ust.hk`
- Zikai Li — `zikaili@link.cuhk.edu.hk`
