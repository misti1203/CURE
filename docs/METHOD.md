# CURE method guide

This guide maps the terminology used in the paper to the implementation and explains the data flow at a level suitable for readers approaching the repository for the first time.

## 1. Problem setting

CURE accepts a collection of heterogeneous modalities

```text
X = {x1, x2, ..., xm}
```

where an input may be an image tensor or a non-imaging vector such as multi-omics or EHR data. Non-imaging vectors are reshaped into pseudo-image tensors so that all modalities can be processed through a common fusion interface.

The objective is to learn a shared representation that is:

- expressive enough to preserve modality-specific and cross-modal structure;
- efficient enough for resource-constrained medical AI;
- reusable across multiple downstream tasks;
- robust to modality order and missing inputs.

## 2. Two-phase architecture

### Multimodal Shared Information Learning (MSIL)

MSIL performs sequential integration. The first HyFuse layer combines the first two modalities. Its shared output is paired with the next modality and passed to the next HyFuse layer. The process continues until all modalities have been incorporated.

Conceptually:

```python
shared = None
for index, modality in enumerate(modalities):
    if index == 0:
        previous = modality
        continue
    shared = hyfuse(previous if shared is None else shared, modality)
return shared
```

The implementation is more detailed because it retains intermediate SIR features and late-fusion outputs, but the central principle is the same: **incremental shared-representation learning without one full parallel encoder per modality**.

### Heterogeneous Modality-Specific Multitask Learning (HMML)

HMML pools the final shared features and applies task-specific output heads. In the released four-stream notebook, these heads correspond to two categorical classification tasks, one survival-risk head, and one binary clinical-prediction head.

## 3. HyFuse components

### 3.1 EMRC — Efficient Multimodal Residual Convolution

EMRC extracts multi-scale patterns with efficient residual and grouped/multi-kernel convolutional operations. Its purpose is to enrich each modality before expensive cross-modal reasoning while controlling parameter and FLOP growth.

### 3.2 HySAM — Hybrid-Space Aware Attention Mixer

HySAM learns shared attention in complementary representation spaces.

Its two main blocks are:

1. **HQMGA — Hyperbolic Quantum Mutual Guidance Attention**
   - **MHDGA:** builds dual-geometry hyperbolic attention through Lorentz and Poincaré representations.
   - **MQIA:** constructs quantum-inspired interactions using trainable real/imaginary state components.
   - The streams mutually guide one another so that hierarchical geometry and high-dimensional interaction cues co-evolve.
2. **MAFG — Multimodal Attention Fusion Gating**
   - fuses the hyperbolic and quantum-inspired attention outputs;
   - produces refined attention-aware shared features for each modality pair.

In the released notebook, the central HySAM layer is implemented by the `MACFusion` class and invoked twice in the `PCMFA` function.

### 3.3 LLF — Learnable Late Fusion

LLF combines the refined outputs with sample-dependent learnable weights rather than a fixed arithmetic average or concatenation. In the paper formulation, unavailable streams are masked before weight normalization, enabling omics-only, WSI-only, partially observed, and fully observed evaluations.

> **Implementation note:** the released notebook's `LF` class is a compact learned scalar-rescaling and concatenation layer. Reproducing the manuscript's missing-modality experiments requires the more general content-aware, mask-aware LLF formulation described in the paper.

### 3.4 SIR — Shared Information Refinement

SIR is the intermediate-fusion pathway. It uses efficient multi-kernel grouped convolution to refine shared features at successive channel widths and retains intermediate information that would otherwise be lost in a purely late-fusion cascade.

In the released notebook, the `SIR`, `info_fusion`, and `info_fusion1` functions implement this progressive refinement.

## 4. End-to-end data flow

```text
Heterogeneous modalities
        │
        ▼
EMRC: multi-scale modality features
        │
        ▼
HySAM: hyperbolic + quantum-inspired cross-modal attention
        │
        ├────────► SIR intermediate refinement
        │
        ▼
LLF: adaptive shared representation
        │
        ▼
Next HyFuse stage + next modality
        │
        ▼
Final modality-order-invariant shared representation
        │
        ▼
HMML task-specific heads
```

## 5. Losses and metrics

The paper uses cross-entropy objectives for classification and a negative log-likelihood objective for survival prediction. The released notebook implements the survival objective as a Cox-style negative partial-likelihood, alongside:

- categorical cross-entropy for two classification heads;
- a custom Cox negative log partial likelihood for the survival head;
- binary cross-entropy for the mortality/clinical head;
- Accuracy and AUC for classification;
- C-index for survival.

## 6. CURE variants

| Variant | Backbone | Parameters | GFLOPs |
|---|---|---:|---:|
| CURE-SN | ShuffleNet | 3.1M | 0.29 |
| CURE-18 | ResNet18 | 7.71M | 0.59 |
| CURE-V | ViT-Tiny | 10.8M | 1.25 |
| CURE-IN | Inception-v3 | 13.7M | 2.82 |
| CURE-50 | ResNet50 | 14.4M | 1.83 |

The repository's current notebook is a research-oriented implementation of the core cascaded fusion logic rather than five separate packaged model classes.
