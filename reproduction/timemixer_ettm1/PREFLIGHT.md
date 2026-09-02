# TimeMixer ETTm1 Reproduction Preflight

Date: 2026-09-02

## Target

- Project: Time-Series-Library / TSLib
- Task: long-term forecasting
- Model: TimeMixer
- Dataset: ETTm1
- Expected branch: `codex/timemixer-ettm1-reproduction`
- Data path: `./dataset/ETT-small/ETTm1.csv`
- Constraint observed: no training was started, no dependencies were installed or upgraded, and model implementation files were not modified.

## 1. Git State

- Current branch: `codex/timemixer-ettm1-reproduction`
- Branch tracking:
  - remote: `origin`
  - merge ref: `refs/heads/codex/timemixer-ettm1-reproduction`
- Latest local commit before this report:
  - `4e938a1 Update README.md`
- Working tree before generating this report:
  - clean
- Remotes:
  - `origin`: `https://github.com/yourdad133/Time-Series-Library.git`
  - `upstream`: `https://github.com/thuml/Time-Series-Library.git`

## 2. Dataset Existence

`./dataset/ETT-small/ETTm1.csv` exists.

- Repository-relative path: `dataset/ETT-small/ETTm1.csv`
- Size: 10,360,719 bytes
- Last modified: 2026-09-01 20:51:11

## 3. ETTm1 Data Summary

Checked with pandas.

- Shape: `(69680, 8)`
- Columns:
  - `date`
  - `HUFL`
  - `HULL`
  - `MUFL`
  - `MULL`
  - `LUFL`
  - `LULL`
  - `OT`
- Missing values:
  - total: `0`
  - by column:
    - `date`: `0`
    - `HUFL`: `0`
    - `HULL`: `0`
    - `MUFL`: `0`
    - `MULL`: `0`
    - `LUFL`: `0`
    - `LULL`: `0`
    - `OT`: `0`
- Date range:
  - start: `2016-07-01 00:00:00`
  - end: `2018-06-26 19:45:00`
- Date order: monotonically increasing

## 4. `models/TimeMixer.py` Check

`models/TimeMixer.py` exists and was inspected.

Main observations:

- Defines `DFT_series_decomp`.
- Defines `MultiScaleSeasonMixing`.
- Defines `MultiScaleTrendMixing`.
- Defines `PastDecomposableMixing`.
- Defines `Model(nn.Module)`.
- Supports `long_term_forecast` and `short_term_forecast` via `forecast`.
- Also includes branches for `imputation`, `anomaly_detection`, and `classification`.
- Uses:
  - `layers.Autoformer_EncDec.series_decomp`
  - `layers.Embed.DataEmbedding_wo_pos`
  - `layers.StandardNorm.Normalize`
- `from models import TimeMixer` succeeded.
- A TimeMixer model object for the official ETTm1 pred_len=96 configuration was constructed without training:
  - parameters: `75,497`

The model implementation was not modified.

## 5. Official Script Check

`scripts/long_term_forecast/ETT_script/TimeMixer_ETTm1.sh` exists and was inspected.

The script sets:

- `CUDA_VISIBLE_DEVICES=0`
- `model_name=TimeMixer`
- `seq_len=96`
- `e_layers=2`
- `down_sampling_layers=3`
- `down_sampling_window=2`
- `learning_rate=0.01`
- `d_model=16`
- `d_ff=32`
- `batch_size=16`

The script launches four long-term forecasting runs:

| pred_len | model_id |
| --- | --- |
| 96 | `ETTm1_96_96` |
| 192 | `ETTm1_96_192` |
| 336 | `ETTm1_96_336` |
| 720 | `ETTm1_96_720` |

Common command arguments:

- `--task_name long_term_forecast`
- `--is_training 1`
- `--root_path ./dataset/ETT-small/`
- `--data_path ETTm1.csv`
- `--model TimeMixer`
- `--data ETTm1`
- `--features M`
- `--seq_len 96`
- `--label_len 0`
- `--e_layers 2`
- `--enc_in 7`
- `--c_out 7`
- `--des Exp`
- `--itr 1`
- `--d_model 16`
- `--d_ff 32`
- `--batch_size 16`
- `--learning_rate 0.01`
- `--down_sampling_layers 3`
- `--down_sampling_method avg`
- `--down_sampling_window 2`

Arguments not set by the script and therefore inherited from `run.py` defaults:

- `--train_epochs`: `10`
- `--patience`: `3`
- `--num_workers`: `10`
- `--loss`: `MSE`
- `--lradj`: `type1`
- `--dropout`: `0.1`
- `--embed`: `timeF`
- `--freq`: `h`
- `--n_heads`: `8`
- `--d_layers`: `1`
- `--moving_avg`: `25`
- `--factor`: `1`
- `--decomp_method`: `moving_avg`
- `--use_norm`: `1`
- `--channel_independence`: `1`
- `--use_gpu`: default enabled
- `--gpu`: `0`
- `--gpu_type`: `cuda`
- `--augmentation_ratio`: `0`
- `--seed`: `2`

Random seed handling:

- `run.py` hard-codes `fix_seed = 2021` and applies it to Python `random`, PyTorch, and NumPy at startup.
- `run.py` also defines `--seed` with default `2`, mainly for augmentation-related code.
- `TimeMixer_ETTm1.sh` does not pass `--seed`.

## 6. `run.py` and Long-Term Forecast Experiment Check

`run.py` exists and was inspected.

Important behavior:

- Uses `argparse`.
- Selects `Exp_Long_Term_Forecast` when `--task_name long_term_forecast`.
- If `--is_training 1`, it constructs the experiment, trains, then tests.
- If `--is_training 0`, it constructs the experiment and calls `test(setting, test=1)`.
- `python run.py --help` succeeded.
- `run.py` itself can parse the expected TimeMixer/ETTm1 arguments.

`exp/exp_long_term_forecasting.py` exists and was inspected.

Important behavior:

- Imports `data_provider` at module import time.
- Builds the model through `Exp_Basic` and the lazy model dictionary.
- Uses `optim.Adam` with `args.learning_rate`.
- Uses `nn.MSELoss`.
- Training writes checkpoints under `args.checkpoints`.
- Testing writes:
  - `./test_results/<setting>/`
  - `./results/<setting>/`
  - `result_long_term_forecast.txt`

No training or testing command was started during this preflight.

## 7. Runtime Environment

Python and package checks:

- Python: `3.12.7 | packaged by Anaconda, Inc. | MSC v.1929 64 bit (AMD64)`
- Python executable: Anaconda Python on local machine
- Platform: Windows 11
- PyTorch: `2.5.1+cu121`
- PyTorch CUDA available: `true`
- PyTorch CUDA version: `12.1`
- cuDNN version: `90100`
- CUDA GPU count: `1`
- GPU class: NVIDIA laptop GPU

Installed/importable package versions observed:

| Package | Status |
| --- | --- |
| `torch` | `2.5.1+cu121` |
| `numpy` | `1.26.4` |
| `pandas` | `2.2.2` |
| `scikit-learn` / `sklearn` | `1.5.1` |
| `matplotlib` | `3.9.2` |
| `scipy` | `1.13.1` |
| `sympy` | `1.13.1` |
| `PyWavelets` / `pywt` | `1.7.0` |
| `tqdm` | `4.66.5` |
| `PyYAML` / `yaml` | `6.0.1` |
| `einops` | not importable |
| `reformer_pytorch` | not importable |
| `patoolib` | not importable |
| `patool` | not importable |
| `sktime` | not importable |
| `datasets` | not importable |
| `huggingface_hub` | not importable |
| `transformers` | not importable |
| `lightning` | not importable |
| `gluonts` | not importable |

Repository `requirements.txt` asks for newer/different versions of several packages, including:

- `einops==0.8.1`
- `local-attention==1.11.2`
- `reformer-pytorch==1.4.4`
- `numpy==2.1.2`
- `scipy==1.16.3`
- `scikit-learn==1.7.2`
- `pandas==2.3.3`
- `matplotlib==3.10.8`
- `sktime==0.40.1`
- `datasets==4.5.0`
- `patool==4.0.3`
- `transformers==4.57.3`
- `huggingface_hub==0.36.0`
- `gluonts==0.16.2`
- `lightning==2.6.0`

No dependency installation or upgrade was performed.

## 8. Smoke Test Feasibility

Requested smoke test shape:

- `pred_len=96`
- `train_epochs=1`
- TimeMixer
- ETTm1
- long-term forecasting

Conclusion: the current environment cannot run the smoke test yet.

Reason:

- Importing `Exp_Long_Term_Forecast` fails before training starts.
- Failure observed:

```text
ModuleNotFoundError: No module named 'patoolib'
```

Import chain:

```text
exp/exp_long_term_forecasting.py
  -> data_provider.data_factory
  -> data_provider.data_loader
  -> data_provider.m4
  -> import patoolib
```

This happens even though the target dataset is ETTm1, because `data_provider.data_loader` imports M4/UEA-related dependencies globally at module import time.

Additional missing packages that are likely to block subsequent imports or other repository paths:

- `sktime`
- `datasets`
- `huggingface_hub`
- `einops`
- `reformer_pytorch`

Hardware and model-size assessment:

- CUDA is available.
- The GPU is detected.
- The official TimeMixer ETTm1 pred_len=96 model configuration is small (`75,497` parameters when constructed directly).
- After resolving the missing import dependencies, a one-epoch pred_len=96 smoke run should be technically reasonable on this machine, but this was not executed because the current dependency state blocks the official training entry point.

## 9. Blocking Issues

Blocking:

1. `patoolib` is missing, causing `from exp.exp_long_term_forecasting import Exp_Long_Term_Forecast` to fail before any training can start.
2. Other repository dependencies are also missing or mismatched relative to `requirements.txt`, notably `sktime`, `datasets`, `huggingface_hub`, `einops`, and `reformer_pytorch`.

Not blocking:

1. Current branch is correct.
2. `origin` and `upstream` are configured.
3. `ETTm1.csv` exists and has no missing values.
4. `models/TimeMixer.py` exists, imports successfully, and can construct the target model configuration directly.
5. Official TimeMixer ETTm1 script exists and contains the expected pred_len runs.

## 10. Suggested Minimum Smoke Command After Dependencies Are Fixed

Do not run until the dependency blocker is resolved.

```bash
python -u run.py \
  --task_name long_term_forecast \
  --is_training 1 \
  --root_path ./dataset/ETT-small/ \
  --data_path ETTm1.csv \
  --model_id ETTm1_96_96_smoke \
  --model TimeMixer \
  --data ETTm1 \
  --features M \
  --seq_len 96 \
  --label_len 0 \
  --pred_len 96 \
  --e_layers 2 \
  --enc_in 7 \
  --c_out 7 \
  --des Smoke \
  --itr 1 \
  --d_model 16 \
  --d_ff 32 \
  --batch_size 16 \
  --learning_rate 0.01 \
  --train_epochs 1 \
  --patience 3 \
  --down_sampling_layers 3 \
  --down_sampling_method avg \
  --down_sampling_window 2
```
