# Reproducibility guide

## Scope of the current release

The repository currently exposes one TensorFlow/Keras notebook containing the principal four-stream CURE experiment and implementations of the core hybrid-space fusion components. It is useful for understanding and adapting the architecture, but it is not yet a fully scripted reproduction package for all sixteen benchmarks.

## Environment

The notebook metadata records:

- Python 3.11.11;
- a GPU-enabled Kaggle environment;
- TensorFlow/Keras;
- NumPy, pandas, scikit-learn, OpenCV, matplotlib, seaborn, Plotly, and lifelines.

`requirements.txt` provides a compatibility-oriented reconstruction. Exact package versions from the original training environment were not preserved in the repository, so numerical results can differ across TensorFlow, CUDA, cuDNN, and hardware versions.

## Expected input files

The notebook uses fixed relative paths.

| Stream | Expected inputs |
|---|---|
| HAM10000 | `ham10000/x_train.npy`, `x_val.npy`, `x_test.npy`, `y_train.npy`, `y_val.npy`, `y_test.npy` |
| SIPaKMeD | `SIPAKMED/features.npy`, `SIPAKMED/labels.npy` |
| TCGA-BRCA | `X_train_img_BRCA_updated.npy`, `X_val_img_BRCA_updated.npy`, `X_test_img_BRCA_updated.npy`, and corresponding `y_*_tab_BRCA_updated.npy` files |
| MIMIC-III mortality | `X_train_MORT_MIMIC3_updated.npy`, `X_val_MORT_MIMIC3_updated.npy`, `X_test_MORT_MIMIC3_updated.npy`, and corresponding labels |

Before running, verify that all streams have compatible sample counts after the notebook's sampling and augmentation cells.

## Paper protocol versus released notebook

| Item | Paper | Current notebook |
|---|---|---|
| Epochs | 200 | 200 |
| Optimizer | Adam, initial LR `1e-3` | `optimizer='adam'` |
| Scheduler | ReduceLROnPlateau to minimum `1e-6` | factor 0.1, patience 50, minimum `1e-6` |
| Early stopping | Validation-driven | patience 60 with best-weight restoration |
| Checkpoint | Paper experiments retain trained models | `best1_model.keras`, selected by validation loss |
| Seeds | Five runs | Generates five random integer seeds at runtime |
| Batch size | Not specified in the manuscript | Primary `model.fit` call does not explicitly set one; Keras default applies |
| Tasks in current example | Heterogeneous multitask learning | Two categorical heads, one Cox survival head, one binary head |

## Recommended run procedure

1. Create the documented environment.
2. Prepare the four sets of NumPy arrays.
3. Confirm data ranges and labels before augmentation.
4. Replace runtime-generated seeds with a fixed explicit list.
5. Execute the notebook sequentially in a fresh kernel.
6. Record:
   - framework and CUDA versions;
   - random seeds;
   - exact data split identifiers;
   - batch size;
   - checkpoint selected;
   - metric aggregation code.
7. Repeat each configuration for the same five seeds and report mean ± standard deviation.

## Compatibility notes

### Keras output naming

The released model creates four output layers without explicit names while the metric configuration uses dictionary keys. Some Keras versions require the dictionary keys to match actual output names. When a metric-key error occurs, either:

- give each output layer an explicit name and use those names consistently in the loss/metric dictionaries; or
- pass losses and metrics as ordered lists.

This is an environment-compatibility adjustment and should not change the architecture.

### Reproducible seeds

The final evaluation cell generates random seed values using `np.random.randint`. Save and reuse the printed seed list—or define a fixed list manually—before comparing runs.

### Paper LLF versus notebook `LF`

The manuscript's LLF is content-aware and mask-aware, while the current notebook's `LF` class uses trainable scalar rescaling followed by concatenation. Use the manuscript formulation for missing-modality reproduction and record the modality mask used at evaluation.

### Dataset alignment

The notebook samples and augments streams separately. Check sample alignment carefully before using any genuinely paired data. For paired WSI–omics or paired MRI experiments, modality-specific records must refer to the same patient/sample after all filtering and splitting.

## Checklist for exact table-level reproduction

- [ ] Obtain the same dataset releases and cohort filters.
- [ ] Preserve patient-level rather than image-level separation where applicable.
- [ ] Recreate the exact five seeds.
- [ ] Match preprocessing, augmentation, and pseudo-image reshaping.
- [ ] Match modality ordering and order-invariance experiments.
- [ ] Match the backbone-specific CURE variant.
- [ ] Match task heads, loss weights, and metric definitions.
- [ ] Match TensorFlow/CUDA/cuDNN versions and hardware.
- [ ] Use the same parameter and FLOP counting convention.
- [ ] Report mean and standard deviation rather than a single favorable run.

## What is not included

- all sixteen prepared datasets;
- fixed split manifests;
- pretrained checkpoints for CURE-SN, CURE-18, CURE-V, CURE-IN, and CURE-50;
- scripts for every baseline;
- automated parameter/FLOP benchmarking;
- a deterministic end-to-end reproduction command.

These omissions should be stated when describing the current repository release.
