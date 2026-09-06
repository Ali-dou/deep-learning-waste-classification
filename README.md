# Automatic Waste Image Classification

## Project overview

This project addresses the **Hugging Face Image Classification** task of assigning a waste image to one of seven classes:

- cardboard
- compost
- glass
- metal
- paper
- plastic
- trash

The project compares:

1. a compact **CNN trained from scratch**;
2. **ResNet18 with a frozen ImageNet-pretrained backbone**;
3. **ResNet18 fine-tuned** on the final residual block (`layer4`) and classification head.

The final model is evaluated on a **leakage-free test set** after removing exact test images that also occurred in the training data.

## Main result

| Model | Clean-test accuracy | Clean-test macro F1 |
|---|---:|---:|
| SimpleCNN from scratch | 0.5168 | 0.4680 |
| ResNet18 frozen | 0.8563 | 0.8389 |
| ResNet18 fine-tuned | **0.8694** | **0.8644** |

The final leakage-free test set contains **536 images**. The original test split contained 644 images, but 108 exact image duplicates were found in the training data and removed before reporting final performance.

## Repository structure

```text
.
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_cnn_baseline.ipynb
│   ├── 03_resnet18_transfer_learning.ipynb
│   └── 04_error_analysis.ipynb
├── outputs/
│   ├── figures/
│   ├── metrics/
│   └── models/
├── src/
├── requirements.txt
└── README.md
```

## Dataset

Dataset: [`rootstrap-org/waste-classifier`](https://huggingface.co/datasets/rootstrap-org/waste-classifier)

The download is approximately 1.08 GB and should **not** be committed to Git. In this project, it is stored outside the repository:

```text
D:\datasets\waste-classifier\dataset-splits-custom.zip
D:\datasets\waste-classifier\extracted\dataset_splits\
```

The archive contains:

```text
dataset_splits/
├── train/
│   ├── cardboard/
│   ├── compost/
│   ├── glass/
│   ├── metal/
│   ├── paper/
│   ├── plastic/
│   └── trash/
└── test/
    └── same class folders
```

## Environment setup (Windows + NVIDIA GPU)

```powershell
py -m venv .venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

Install a CUDA-enabled build of PyTorch that matches your system. The configuration used in this project was:

```powershell
python -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
```

Then install the remaining packages:

```powershell
python -m pip install -r requirements.txt
```

Select `.venv\Scripts\python.exe` as the VS Code Python interpreter and Jupyter kernel.

## Run order

1. `01_eda.ipynb` - dataset exploration, class distribution, image dimensions, preprocessing checks.
2. `02_cnn_baseline.ipynb` - CNN baseline training, validation, and evaluation.
3. `03_resnet18_transfer_learning.ipynb` - frozen ResNet18, fine-tuning, duplicate audit, and clean-test comparison.
4. `04_error_analysis.ipynb` - error ranking, top confusions, and qualitative inspection.

## Reproducibility choices

- Fixed random seed: `42`
- Stratified 80/20 train/validation split: 2,088 / 523 images
- Batch size: `32`
- Selection metric: validation macro F1
- Loss: class-weighted cross entropy
- Early stopping: enabled in all training experiments
- Final reported evaluation: 536-image test split with exact train/test duplicates removed

## Important data-quality finding

The supplied test set contained **108 exact image duplicates** already present in the training data:

- 100 duplicates had the same label;
- 8 duplicates had at least one different training label.

The final model comparison therefore uses the cleaned test split rather than the original 644-image test split.

## Results artefacts

The notebooks save:

- model checkpoints in `outputs/models/`;
- tables and predictions in `outputs/metrics/`;
- confusion matrices and plots in `outputs/figures/`.

Large data files and model checkpoints should remain ignored by Git. The trained models can optionally be uploaded to the Hugging Face Hub instead of GitHub.

## References

- He, K., Zhang, X., Ren, S., & Sun, J. (2016). *Deep Residual Learning for Image Recognition.* CVPR.
- rootstrap-org. (2024). *Waste Classifier* [Dataset]. Hugging Face.
- Paszke, A. et al. (2019). *PyTorch: An Imperative Style, High-Performance Deep Learning Library.* NeurIPS.
