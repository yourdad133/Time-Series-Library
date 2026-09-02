# TimeMixer ETTm1 Smoke Test

Date: 2026-09-02

## Final Status

`FAIL_RUNTIME`

The smoke test failed before data loading and before the first training epoch. No model implementation, data loading logic, experiment code, dependencies, or training parameters were changed.

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
- Git commit before run: `e3bb0697c6e0a8c2bdb527576972cbd5437197b6`
- Worktree status before run: clean
- Recorded start time: `2026-09-02T13:01:41.0496332+08:00`
- Recorded end time: `2026-09-02T13:02:48.5766537+08:00`
- Recorded attempt window: approximately `67.53 seconds`
- Training process duration: approximately `4.31 seconds`

## Command

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

## Runtime Environment

The interpreter path provided without the separator before `.conda` was checked and does not exist locally. The run used the existing TSLib Conda interpreter under the user Conda environment, not Anaconda base Python.

- Python executable: `<user-home>\.conda\envs\tslib\python.exe`
- Python: `3.11.16 | packaged by Anaconda, Inc. | MSC v.1942 64 bit (AMD64)`
- PyTorch: `2.6.0+cu124`
- `torch.cuda.is_available()`: `true`
- `torch.version.cuda`: `12.4`
- `torch.backends.cudnn.version()`: `90100`
- GPU 0: `NVIDIA GeForce RTX 3050 Ti Laptop GPU`
- GPU total memory: `4095 MiB` by PyTorch, `4096 MiB` by `nvidia-smi`

CUDA compute was previously confirmed in this environment with a `512 x 512` CUDA matrix multiplication on `cuda:0`, producing finite results.

Observed GPU memory:

- Before run: `0 MiB / 4096 MiB`
- After run: `0 MiB / 4096 MiB`
- Peak memory during the short failed run was not accurately captured. The run failed before model construction completed, so these before/after snapshots do not represent training memory use.

## Actual Parameters Reached

`run.py` parsed and printed the following relevant parameters before failing:

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

The process printed `Using GPU` and `Use GPU: cuda:0`, so the entry point selected `cuda:0` before the runtime failure.

## Dataset Split Counts

Not reached.

The failure occurred during experiment/model construction, before `_get_data(flag='train')`, `_get_data(flag='val')`, or `_get_data(flag='test')` was called. Therefore the train, validation, and test sample counts were not printed by this run.

Expected ETTm1 dataset lengths for this configuration, based on the repository data split formula and `seq_len=96`, `pred_len=96`, are:

- train: `34465`
- validation: `11425`
- test: `11425`

These are calculated from the code path but were not observed from the failed smoke run output.

## Metrics

Not available.

- Epoch training loss: not reached
- Validation loss: not reached
- Test MSE: not reached
- Test MAE: not reached

## Error

Failure type: runtime error.

The process failed while constructing `Exp_Long_Term_Forecast`, before model construction completed and before data loading or training.

Relevant traceback:

```text
Traceback (most recent call last):
  File "<repo>\run.py", line 205, in <module>
    exp = Exp(args)  # set experiments
  File "<repo>\exp\exp_long_term_forecasting.py", line 20, in __init__
    super(Exp_Long_Term_Forecast, self).__init__(args)
  File "<repo>\exp\exp_basic.py", line 23, in __init__
    self.model = self._build_model().to(self.device)
  File "<repo>\exp\exp_long_term_forecasting.py", line 23, in _build_model
    model = self.model_dict[self.args.model](self.args).float()
  File "<repo>\exp\exp_basic.py", line 96, in __getitem__
    print(f"\U0001f680 Lazy Loading: {key} ...")
UnicodeEncodeError: 'gbk' codec can't encode character '\U0001f680' in position 0: illegal multibyte sequence
```

Cause:

- The Windows PowerShell console encoding used by this run could not encode the rocket emoji printed by `LazyModelDict.__getitem__` in `exp/exp_basic.py`.
- This is not a CUDA OOM.
- This is not a missing dependency.
- This is not a TimeMixer model logic failure.
- It is a console output encoding failure triggered before the model class was returned by lazy loading.

## Generated Files

No checkpoint, result, test result, model weight, or full console log is being committed.

`git status` after the failed run showed no tracked or untracked changes before this `SMOKE_TEST.md` report was created, indicating the run did not leave visible Git-managed artifacts at that point.

## Recommendation

Do not proceed to the full reproduction run until the smoke test can pass.

The next smoke attempt should use the same model and data parameters, but the execution environment must ensure UTF-8-capable console output before calling `run.py`. This report does not change code or retry with environment changes because the instruction for non-OOM runtime errors was to record the error and stop.
