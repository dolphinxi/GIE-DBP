# Robust Classification Decision Boundary Prediction via PDE-Governed Geometric Interface Evolution

This repository provides the software and reproducibility assets for the paper **"Robust Classification Decision Boundary Prediction via PDE-Governed Geometric Interface Evolution"**. The implementation is organized around two stages:

1. **Decision boundary inference**.
2. **Classification decision**.

The released software is distributed as Windows executables (`.exe`) together with all data, intermediate features, geometric prior dictionaries, and trained weights needed to reproduce the reported experiments.

## Repository Structure

```text
.
|-- boundary_inference/
|   |-- data_spatiotemporal_modeling.exe
|   |-- symbolic_distance_function.exe
|   |-- train.exe
|   |-- data/
|   |-- checkpoints/
|   `-- _internal/
|-- classification_decision_COVID/
|   |-- classifier_levset.exe
|   |-- potential_data/
|   `-- outputs/
|-- classification_decision_RSRAC/
|   |-- classifier_levset.exe
|   |-- potential_data/
|   `-- outputs/
|-- COVID-19 Datasets/
|   |-- EfficientNet_KAN/
|   |-- MLP/
|   |-- ResNet/
|   |-- ResNet_KAN/
|   `-- VIT/
`-- RSRAC Datasets/
    |-- RoMD=0/
    |-- RoMD=0.2/
    |-- RoMD=0.4/
    |-- RoMD=0.6/
    |-- RoMD=0.8/
    `-- test_data/
```

`boundary_inference` implements the decision boundary inference stage. `classification_decision_COVID` and `classification_decision_RSRAC` implement the classification decision stage for the COVID-19 and RSRAC experiments, respectively.

The `_internal` directory contains runtime dependencies required by the packaged executables. Keep it in place when moving or copying the executable files.

## Software Components

### 1. Boundary Inference

The `boundary_inference` module contains three executable programs.

| Program | Role | Input | Output |
| --- | --- | --- | --- |
| `data_spatiotemporal_modeling.exe` | Spatiotemporal association modeling | Latent classification features for one class | Spatiotemporal Correlation data |
| `symbolic_distance_function.exe` | Geometric prior dictionary construction | Spatiotemporal association data | Geometric prior dictionary containing spatiotemporal data, signed distance functions, boundary hierarchy, and global modeling information |
| `train.exe` | Boundary inference training | Geometric prior dictionary plus training parameters | Level-set model weights saved automatically as checkpoints |

Typical workflow:

```powershell
# 1. Generate spatiotemporal association data from latent classification features.
.\boundary_inference\data_spatiotemporal_modeling.exe "<absolute-path-to-class-feature-file>"

# 2. Build the geometric prior dictionary.
.\boundary_inference\symbolic_distance_function.exe "<absolute-path-to-es_data_with_time>" `
  --class-num <class-index> `
  --class-name <class-name> `
  --data-dim <feature-dimension> `
  --class-base-dir "<absolute-path-to-class-base-directory>"

# 3. Train the PDE-governed level-set boundary model.
.\boundary_inference\train.exe "<absolute-path-to-signed_distance_dict>" `
  --epochs 100 `
  --batch-size 64 `
  --data-dim <feature-dimension>
```

Available command-line options:

```powershell
.\boundary_inference\data_spatiotemporal_modeling.exe --help
.\boundary_inference\symbolic_distance_function.exe --help
.\boundary_inference\train.exe --help
```

### 2. Classification Decision

The classification decision programs take trained level-set weights and geometric prior dictionaries as input. For the RSRAC dataset, the classifier expects 8 class-specific level-set checkpoints and 8 corresponding signed-distance dictionaries. For the COVID-19 dataset, it expects 3 class-specific checkpoints and 3 corresponding signed-distance dictionaries.

If an independent validation set is available, the time parameter `t` is selected by maximizing validation accuracy. If no validation set is available, set `t = 0`. The released dataset folders include `Parameter t selection.txt` files for the reproduced experiments.

The executable argument is named `--time-multipliers`; provide one value per class in class order.

## Reproducing the Paper Experiments

All data required for reproducibility are included in the repository:

- `RSRAC Datasets/` contains RSRAC test data, intermediate features, geometric prior dictionaries, selected time parameters, and trained weights for RoMD settings `0`, `0.2`, `0.4`, `0.6`, and `0.8`.
- `COVID-19 Datasets/` contains COVID-19 test data, intermediate features, geometric prior dictionaries, selected time parameters, and trained weights for `EfficientNet_KAN`, `MLP`, `ResNet`, `ResNet_KAN`, and `VIT` feature backbones.

Use absolute paths when running the executables, especially because several dataset directory names contain spaces.

### RSRAC Example

The following example evaluates the RSRAC experiment for `RoMD=0`. Replace the time values with the values recorded in each class folder's `Parameter t selection.txt` file, or use `0` for each class if no validation set is used.

```powershell
$repo = (Get-Location).Path
$base = Join-Path $repo "RSRAC Datasets\RoMD=0"

$checkpoints = (0..7 | ForEach-Object {
  Join-Path $base "class_$_\checkpoints\checkpoint.pth"
}) -join ","

$signedDistance = (0..7 | ForEach-Object {
  Join-Path $base "class_$_\data\signed_distance_dict"
}) -join ","

.\classification_decision_RSRAC\classifier_levset.exe `
  --test-data (Join-Path $repo "RSRAC Datasets\test_data\potential_testdata") `
  --test-label (Join-Path $repo "RSRAC Datasets\test_data\potential_testlabel") `
  --checkpoints $checkpoints `
  --signed-distance $signedDistance `
  --time-multipliers "0,0,0,0,0,0,0,0" `
  --batch-size 64 `
  --output-dir (Join-Path $repo "classification_decision_RSRAC\outputs") `
  --device auto
```

You may also run the RSRAC classifier interactively:

```powershell
.\classification_decision_RSRAC\classifier_levset.exe --interactive
```

### COVID-19 Example

The following example evaluates the COVID-19 experiment using the `ResNet` feature backbone. Replace `ResNet` with `EfficientNet_KAN`, `MLP`, `ResNet_KAN`, or `VIT` to reproduce the corresponding experiment.

```powershell
$repo = (Get-Location).Path
$backbone = "ResNet"
$base = Join-Path $repo "COVID-19 Datasets\$backbone"

$checkpoints = (0..2 | ForEach-Object {
  Join-Path $base "class_$_\checkpoints\checkpoint.pth"
}) -join ","

$signedDistance = (0..2 | ForEach-Object {
  Join-Path $base "class_$_\data\signed_distance_dict"
}) -join ","

.\classification_decision_COVID\classifier_levset.exe `
  --test-data (Join-Path $base "test_data\potential_test_data") `
  --test-label (Join-Path $base "test_data\potential_test_label") `
  --checkpoints $checkpoints `
  --signed-distance $signedDistance `
  --time-multipliers "0,0,0" `
  --batch-size 64 `
  --output-dir (Join-Path $repo "classification_decision_COVID\outputs") `
  --device auto
```

You may also run the COVID-19 classifier interactively:

```powershell
.\classification_decision_COVID\classifier_levset.exe --interactive
```

## Output

The classification decision executables export evaluation results, including confusion-matrix results, to the specified `--output-dir`. If `--device auto` is used, CUDA is selected automatically when available; otherwise, the program falls back to CPU execution.

## Notes

- The executables are intended for Windows.
- The provided data files are serialized experiment artifacts and should be used through the released executables.
- To reproduce the paper results, provide the absolute paths to the corresponding dataset, checkpoint, and signed-distance dictionary files.
- Keep each class's checkpoint and `signed_distance_dict` paired with the same dataset setting or feature backbone.

