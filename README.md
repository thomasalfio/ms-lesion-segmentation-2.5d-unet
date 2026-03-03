# 2.5D U-Net for Multiple Sclerosis Lesion Segmentation

Automatic segmentation of Multiple Sclerosis (MS) lesions from brain FLAIR MRI scans using a lightweight 2.5D U-Net, trained and evaluated on the [MSLesSeg dataset](https://springernature.figshare.com/articles/dataset/MSLesSeg_baseline_and_benchmarking_of_a_new_Multiple_Sclerosis_Lesion_Segmentation_dataset/27919209) (Guarnera et al., *Scientific Data*, 2025).

## Approach

Instead of processing slices independently (2D) or using memory-heavy 3D convolutions, the **2.5D approach** stacks 3 adjacent axial FLAIR slices as input channels — similar to RGB in natural images — and predicts a binary lesion mask for the central slice. This captures inter-slice context while staying within free-tier Google Colab GPU limits (~15 GB on a T4).

| | 2D | **2.5D (ours)** | 3D |
|---|---|---|---|
| Inter-slice context | None | Adjacent slices | Full volume |
| GPU memory | Low | Low | Very high |
| Architecture | Standard 2D conv | Standard 2D conv | 3D conv |

## Architecture

- **Encoder**: 4 levels of double 3x3 conv + BatchNorm + ReLU, with max-pooling
- **Bottleneck**: 512-channel feature maps
- **Decoder**: 4 levels with transposed convolutions and skip connections
- **Output**: 1x1 conv + sigmoid (binary lesion probability)
- **Parameters**: ~7.8M | **Model size**: ~31 MB

## Results

Evaluated on **22 held-out test patients** (completely separate from training):

| Metric | Score |
|--------|-------|
| **Dice Score** | **0.666** |
| IoU | 0.508 |
| Precision | 0.698 |
| Recall | 0.673 |

Generalization gap (val vs test Dice): 0.01 — indicating good generalization with no overfitting.

## Dataset

The [MSLesSeg dataset](https://doi.org/10.1038/s41597-024-04208-w) contains 115 FLAIR MRI scans from 75 patients:
- **Training**: 93 scans (53 patients, multiple timepoints)
- **Test**: 22 scans (22 patients)

Train/validation split is done at the **patient level** to prevent data leakage.

## Training Details

- **Loss**: Binary Cross-Entropy
- **Optimizer**: Adam (lr=1e-4)
- **Epochs**: 20
- **Input size**: 256 x 256
- **Augmentation**: random flips, 90-degree rotations, small rotations (+-15 deg), brightness/contrast shifts, Gaussian noise
- **Environment**: Google Colab (T4 GPU)

## Usage

Open the notebook in Google Colab:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

1. Upload the MSLesSeg dataset to your Google Drive
2. Update `DATA_ROOT` in the notebook to point to your dataset location
3. Run all cells

## Project Structure

```
├── unet2_5_with_augmentation_by_patient.ipynb   # Full pipeline (training + evaluation)
├── report.pdf                                    # Project report
└── README.md
```

## References

- Guarnera, F. et al. "MSLesSeg: a public benchmark for MS lesion segmentation." *Scientific Data*, 2025. [DOI: 10.1038/s41597-024-04208-w](https://springernature.figshare.com/articles/dataset/MSLesSeg_baseline_and_benchmarking_of_a_new_Multiple_Sclerosis_Lesion_Segmentation_dataset/27919209)
- Ronneberger, O. et al. "U-Net: Convolutional Networks for Biomedical Image Segmentation." MICCAI, 2015.

## License

This project is for educational and research purposes.
