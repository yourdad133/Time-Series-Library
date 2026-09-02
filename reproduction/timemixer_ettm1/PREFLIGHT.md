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

Checked with pandas using the TSLib Conda interpreter.

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
- `from models import TimeMixer` succeeded using the TSLib Conda interpreter.
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
- `run.py --help` succeeded when executed with the TSLib Conda interpreter.
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

Important correction:

- The previous report used the wrong Python environment, Anaconda base Python, and therefore produced false dependency-missing conclusions.
- The requested path as written without the separator before `.conda` was checked and does not exist on this machine.
- The TSLib Conda interpreter that exists and was used for this corrected check is the user Conda environment at `<user-home>\.conda\envs\tslib\python.exe`.
- Every Python command in this corrected pass was executed by explicitly calling that interpreter.

Python and package checks from the TSLib Conda interpreter:

- Python: `3.11.16 | packaged by Anaconda, Inc. | MSC v.1942 64 bit (AMD64)`
- Platform: Windows 11
- PyTorch: `2.6.0+cu124`
- PyTorch CUDA available: `true`
- PyTorch CUDA version: `12.4`
- cuDNN version: `90100`
- CUDA GPU count: `1`
- GPU 0: `NVIDIA GeForce RTX 3050 Ti Laptop GPU`
- GPU 0 total memory: `4095 MiB`

CUDA compute check:

- A `512 x 512` CUDA matrix multiplication was executed on `cuda:0`.
- Result shape: `[512, 512]`
- Result finite: `true`
- Conclusion: CUDA is not merely detectable; it can execute tensor computation in this environment.

Module import checks from the TSLib Conda interpreter:

| Package | Status |
| --- | --- |
| `numpy` | importable, `2.1.2` |
| `pandas` | importable, `2.3.3` |
| `sklearn` | importable, `1.7.2` |
| `scipy` | importable, `1.16.3` |
| `matplotlib` | importable, `3.10.8` |
| `einops` | importable, `0.8.1` |
| `reformer_pytorch` | importable, `1.4.4` |
| `sktime` | importable, `0.40.1` |
| `datasets` | importable, `4.5.0` |
| `huggingface_hub` | importable, `0.36.0` |
| `patoolib` | importable, `4.0.3` |

Repository import checks from the TSLib Conda interpreter:

| Import | Status |
| --- | --- |
| `import data_provider.data_loader` | succeeded |
| `import data_provider.data_factory` | succeeded |
| `from exp.exp_long_term_forecasting import Exp_Long_Term_Forecast` | succeeded |
| `from models import TimeMixer` | succeeded |

`run.py --help` executed successfully with the TSLib Conda interpreter, confirming the formal entry point can load without the previously reported dependency failure.

No dependency installation, removal, or upgrade was performed.

## 8. Smoke Test Feasibility

Requested smoke test shape:

- `pred_len=96`
- `train_epochs=1`
- TimeMixer
- ETTm1
- long-term forecasting

Conclusion: the corrected TSLib Conda environment is ready for a minimal smoke test.

Reason:

- The formal long-term forecasting experiment class imports successfully.
- The data provider modules import successfully.
- TimeMixer imports successfully.
- The official entry point, `run.py --help`, loads successfully.
- CUDA is available and passed a real CUDA matrix multiplication test.
- The official TimeMixer ETTm1 pred_len=96 model configuration is small (`75,497` parameters when constructed directly).

The smoke test itself was not executed because the preflight instructions explicitly said not to start training.

## 9. Blocking Issues

Blocking:

1. No blocking issue is currently confirmed in the corrected TSLib Conda environment for starting a `pred_len=96`, `train_epochs=1` smoke test.

Previously reported but now classified as false positives caused by checking the wrong Python environment:

1. `patoolib` missing.
2. `sktime` missing.
3. `datasets` missing.
4. `huggingface_hub` missing.
5. `einops` missing.
6. `reformer_pytorch` missing.

Still not blocking:

1. Current branch is correct.
2. `origin` and `upstream` are configured.
3. `ETTm1.csv` exists and has no missing values.
4. `models/TimeMixer.py` exists, imports successfully, and can construct the target model configuration directly.
5. Official TimeMixer ETTm1 script exists and contains the expected pred_len runs.

## 10. Suggested Minimum Smoke Command

Use the TSLib Conda interpreter explicitly.

```powershell
& "<user-home>\.conda\envs\tslib\python.exe" -u run.py `
  --task_name long_term_forecast `
  --is_training 1 `
  --root_path ./dataset/ETT-small/ `
  --data_path ETTm1.csv `
  --model_id ETTm1_96_96_smoke `
  --model TimeMixer `
  --data ETTm1 `
  --features M `
  --seq_len 96 `
  --label_len 0 `
  --pred_len 96 `
  --e_layers 2 `
  --enc_in 7 `
  --c_out 7 `
  --des Smoke `
  --itr 1 `
  --d_model 16 `
  --d_ff 32 `
  --batch_size 16 `
  --learning_rate 0.01 `
  --train_epochs 1 `
  --patience 3 `
  --down_sampling_layers 3 `
  --down_sampling_method avg `
  --down_sampling_window 2
```
