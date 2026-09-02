# TimeMixer ETTm1 Smoke Test

Date: 2026-09-02

## Final Status

`PASS`

Attempt 2 passed after setting UTF-8 output handling in the current PowerShell process. The model, data loading logic, experiment code, dependencies, and training parameters were not modified.

## Target

- Task: `long_term_forecast`
- Model: `TimeMixer`
- Dataset: `ETTm1`
- Forecast horizon: `pred_len=96`
- Epochs: `train_epochs=1`
- Number of experiments: `itr=1`
- Official script not used; only the single `pred_len=96` command was run.

## Pre-Run Checks

- Branch: `codex/timemixer-ettm1-reproduction`
- Data exists: `./dataset/ETT-small/ETTm1.csv`
- Git commit before Attempt 1: `e3bb0697c6e0a8c2bdb527576972cbd5437197b6`
- Git commit before Attempt 2: `8a549e72abc411cce0b95170e5258e7f28104ae2`
- Worktree status before Attempt 2: clean
- The interpreter path written without the separator before `.conda` was checked and did not exist locally.
- The existing TSLib Conda interpreter under the user Conda environment was used explicitly. Anaconda base Python was not used.

## Runtime Environment

- Python executable: `<user-home>\.conda\envs\tslib\python.exe`
- Python: `3.11.16 | packaged by Anaconda, Inc. | MSC v.1942 64 bit (AMD64)`
- PyTorch: `2.6.0+cu124`
- `torch.cuda.is_available()`: `true`
- `torch.version.cuda`: `12.4`
- `torch.backends.cudnn.version()`: `90100`
- GPU 0: `NVIDIA GeForce RTX 3050 Ti Laptop GPU`
- GPU total memory: `4095 MiB` by PyTorch, `4096 MiB` by `nvidia-smi`

## Attempt 1: FAIL_RUNTIME

Attempt 1 failed before data loading and before the first training epoch.

- Final status: `FAIL_RUNTIME`
- Recorded start time: `2026-09-02T13:01:41.0496332+08:00`
- Recorded end time: `2026-09-02T13:02:48.5766537+08:00`
- Training process duration: approximately `4.31 seconds`
- MSE / MAE: not available

Cause:

- Windows PowerShell used GBK output encoding.
- `exp/exp_basic.py` printed a rocket emoji during lazy model loading.
- The console could not encode the emoji and raised:

```text
UnicodeEncodeError: 'gbk' codec can't encode character '\U0001f680' in position 0: illegal multibyte sequence
```

Attempt 1 confirmed the entry point selected `cuda:0` before failing, but it did not reach data loading, training, validation, or testing.

## Attempt 2: PASS

Attempt 2 kept the same model, data, and training parameters as Attempt 1. Only the current PowerShell process output environment was changed to UTF-8 before launching `run.py`.

### UTF-8 Setup

The following settings were applied in the same PowerShell process before running the smoke command:

```powershell
chcp 65001 | Out-Null
[Console]::InputEncoding = [System.Text.UTF8Encoding]::new()
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
$env:PYTHONUTF8 = "1"
$env:PYTHONIOENCODING = "utf-8"
```

Encoding verification succeeded before training:

```text
stdout encoding: utf-8
🚀 UTF-8 test passed
```

### Command

Executed from the project root with the TSLib Conda interpreter. The local user-home prefix is redacted in this committed report.

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

### Timing

- Attempt 2 start time: `2026-09-02T13:10:37.5810135+08:00`
- Attempt 2 end time: `2026-09-02T13:14:21.4679369+08:00`
- Total wall-clock runtime: approximately `223.89 seconds`
- Epoch 1 training phase time reported by the script: `79.13863563537598 seconds`

### Actual Parameters

`run.py` parsed and printed the following relevant parameters:

- `task_name`: `long_term_forecast`
- `is_training`: `1`
- `model_id`: `ETTm1_96_96_smoke`
- `model`: `TimeMixer`
- `data`: `ETTm1`
- `root_path`: `./dataset/ETT-small/`
- `data_path`: `ETTm1.csv`
- `features`: `M`
- `target`: `OT`
- `freq`: `h`
- `seq_len`: `96`
- `label_len`: `0`
- `pred_len`: `96`
- `enc_in`: `7`
- `dec_in`: `7`
- `c_out`: `7`
- `d_model`: `16`
- `n_heads`: `8`
- `e_layers`: `2`
- `d_layers`: `1`
- `d_ff`: `32`
- `moving_avg`: `25`
- `factor`: `1`
- `dropout`: `0.1`
- `embed`: `timeF`
- `activation`: `gelu`
- `num_workers`: `10`
- `itr`: `1`
- `train_epochs`: `1`
- `batch_size`: `16`
- `patience`: `3`
- `learning_rate`: `0.01`
- `des`: `Smoke`
- `loss`: `MSE`
- `lradj`: `type1`
- `use_amp`: `false`
- `use_gpu`: `true`
- `gpu`: `0`
- `use_multi_gpu`: `false`
- `devices`: `0,1,2,3`
- `down_sampling_layers`: `3`
- `down_sampling_method`: `avg`
- `down_sampling_window`: `2`

The process printed `Using GPU` and `Use GPU: cuda:0`, confirming the run selected `cuda:0`.

### Dataset Split Counts

Observed from the run output:

- train: `34369`
- validation: `11425`
- test before training: `11425`
- test during final testing: `11425`

### Losses and Metrics

Epoch summary:

- Steps: `2149`
- Train Loss: `0.2956966`
- Validation Loss: `0.4267094`
- Test Loss during validation pass: `0.3645226`

Final test metrics:

- Test MSE: `0.3647920787334442`
- Test MAE: `0.3872981071472168`
- DTW: `Not calculated`

Output shapes:

- predictions: `(11425, 96, 7)`
- ground truth: `(11425, 96, 7)`

### GPU Memory Observations

Available snapshots:

- During early training: `83 MiB / 4096 MiB`
- During later training / validation: `189 MiB / 4096 MiB`
- After completion: `0 MiB / 4096 MiB`

These are sampled `nvidia-smi` observations, not a precise peak-memory trace.

## Generated Files

The smoke test generated local artifacts in Git-ignored paths, including:

- `checkpoints/long_term_forecast_ETTm1_96_96_smoke_TimeMixer_ETTm1_ftM_sl96_ll0_pl96_dm16_nh8_el2_dl1_df32_expand2_dc4_fc1_ebtimeF_dtTrue_Smoke_0/checkpoint.pth`
- `results/long_term_forecast_ETTm1_96_96_smoke_TimeMixer_ETTm1_ftM_sl96_ll0_pl96_dm16_nh8_el2_dl1_df32_expand2_dc4_fc1_ebtimeF_dtTrue_Smoke_0/`
- `test_results/long_term_forecast_ETTm1_96_96_smoke_TimeMixer_ETTm1_ftM_sl96_ll0_pl96_dm16_nh8_el2_dl1_df32_expand2_dc4_fc1_ebtimeF_dtTrue_Smoke_0/`
- `result_long_term_forecast.txt`
- Python `__pycache__` files

These files are local run artifacts and are not committed.

Plain `git status --short --branch` after the successful smoke run was clean before this report update, confirming only ignored run artifacts were produced.

## Recommendation

The UTF-8 retry passed. There is no newly confirmed blocker for proceeding from the minimal smoke test to the planned TimeMixer ETTm1 reproduction workflow.
