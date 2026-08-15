# Dataset inventory

The paper evaluates CURE on sixteen public medical datasets spanning unpaired images, unpaired non-imaging data, paired medical images, and paired WSI–omics data. Dataset files are not redistributed by this repository.

## Benchmark groups

| Index | Dataset | Data family | Pairing / role | Representative task |
|---:|---|---|---|---|
| D1 | HAM10000 | Dermoscopy images | Unpaired imaging | Skin-lesion classification |
| D2 | SIPaKMeD | Pap-smear / cytology images | Unpaired imaging | Cervical-cell classification |
| D3 | PathMNIST | Histopathology images | Unpaired imaging | Tissue classification |
| D4 | OrganAMNIST | Abdominal CT images | Unpaired imaging | Organ classification |
| D5 | SARS-CoV-2 CT | Chest CT images | Unpaired imaging | COVID-19 classification |
| D6 | C-NMC 2019 | Hematology microscopy | Unpaired imaging | Leukemia-cell classification |
| D7 | Chest X-Ray Pneumonia | Chest radiography | Unpaired imaging | Pneumonia classification |
| D8 | TCGA-BRCA | Multi-omics | Unpaired non-imaging | Survival prediction |
| D9 | TCGA-UCEC | Multi-omics | Unpaired non-imaging | Survival prediction |
| D10 | TCGA-GBMLGG | Multi-omics | Unpaired non-imaging | Survival prediction |
| D11 | MIMIC-III | EHR / clinical records | Unpaired clinical data | Mortality and clinical-code prediction |
| D12 | MHEALTH | Wearable physiological signals | Unpaired time series | Human-activity recognition |
| D13 | UCI-HAR | Inertial sensor signals | Unpaired time series | Human-activity recognition |
| D14 | BraTS 2021 | Multi-sequence brain MRI | Paired imaging modalities | Multimodal brain-tumor prediction |
| D15 | TCGA-KIRP | Whole-slide images + omics | Paired multimodal | Survival prediction |
| D16 | TCGA-BLCA | Whole-slide images + omics | Paired multimodal | Survival prediction |

## Common preprocessing reported in the paper

- Images are resized to `128 × 128 × 3` where applicable.
- Non-imaging vectors are reshaped into pseudo-image tensors for a uniform fusion interface.
- An `80% / 10% / 10%` train/validation/test split is used where applicable.
- Results are aggregated over five random seeds.

## Augmentation in the released notebook

The released SIPaKMeD preprocessing cells add three training variants per image:

- fixed 20-degree rotation;
- 5-pixel translation;
- 3×3 Gaussian smoothing.

Treat these as implementation-specific notebook details unless the corresponding dataset experiment explicitly uses the same augmentation configuration.

## Current notebook subset

The notebook presently released in `Code/` demonstrates a four-stream example based on:

1. HAM10000 dermoscopy;
2. SIPaKMeD cytology;
3. TCGA-BRCA survival arrays;
4. MIMIC-III mortality arrays.

The remaining paper benchmarks require additional prepared inputs and experiment-specific task heads that are not currently packaged in the repository.

## Data governance

Medical datasets can carry separate licenses, credentialing requirements, data-use agreements, and restrictions on redistribution. Obtain each dataset from its original provider and follow all applicable terms. In particular, controlled-access clinical and genomic resources may require institutional approval or user credentialing.
