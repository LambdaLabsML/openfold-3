# OpenFold3 Training Quick Start

> A joint guide from the **OpenFold consortium** and **Lambda** for setting up and training
> [OpenFold3](https://github.com/aqlaboratory/openfold-3) across different compute
> configurations — from single-GPU debugging to distributed multi-node training.

- [Setup](#setup)
- [Prepare Training Data](#prepare-training-data)
- [Training](#training)
- [Fine-tuning](#fine-tuning)
- [Checkpointing and Resuming](#checkpointing-and-resuming)
- [Inference](#inference)

---

## Setup

### Install Pixi

OpenFold3 provides pre-configured Pixi environments for different hardware and CUDA
versions. First, install Pixi:

```bash
curl -fsSL https://pixi.sh/install.sh | sh
```

Restart your shell after installation so that the `pixi` command is available.

### Clone the repository

```bash
git clone git@github.com:LambdaLabsML/openfold-3.git
cd openfold-3
```

### Set up the OpenFold3 environment

For CUDA 12:

```bash
pixi run -e openfold3-cuda12 setup_openfold
```

Follow the interactive prompts to configure the OpenFold cache directory, parameter
download directory, model parameters, and optional integration tests.

For CUDA 13, replace `openfold3-cuda12` with `openfold3-cuda13`.

> **Use the same Pixi environment consistently** throughout data preparation and training.

---

## Prepare Training Data

For a quick training experiment, generate a subset of the PDB training data:

```bash
pixi run -e openfold3-cuda12 generate-pdb-subset -- --train-size 1000 --seed 42
```

Download the corresponding data:

```bash
pixi run -e openfold3-cuda12 download-pdb-subset -- --train-size 1000 --workers 24
```

> `generate-pdb-subset` and `download-pdb-subset` are Pixi *tasks*, so arguments are
> passed after `--`. Both write to `./datasets` relative to the project root.

This produces a dataset configuration YAML at `datasets/train_pdb_subset.yaml`, ready to
use for training. The default `max_epochs` is `2`.

---

## Training

### Scheme 1: Single-GPU training (H100)

```bash
pixi run -e openfold3-cuda12 run_openfold train \
  --runner-yaml datasets/train_pdb_subset.yaml
```

### Scheme 2: Multi-GPU training

To train on multiple GPUs within a single node, configure your YAML:

```yaml
pl_trainer_args:
  devices: 8      # GPUs per node
  num_nodes: 1
```

For multi-node distributed training:

```yaml
pl_trainer_args:
  devices: 8      # GPUs per node
  num_nodes: 32   # Total number of nodes
```

Then launch training with:

```bash
pixi run -e openfold3-cuda12 run_openfold train \
  --runner-yaml datasets/train_pdb_subset.yaml
```

---

## Fine-tuning

To fine-tune from a pre-trained checkpoint, specify the checkpoint path in the config YAML:

```yaml
experiment_settings:
  mode: train
  output_dir: ./finetune_output
  seed: 42
  restart_checkpoint_path: /path/to/pretrained.ckpt
```

---

## Checkpointing and Resuming

Checkpoints are saved to `{output_dir}/checkpoints/`. Checkpointing is configured in the
config YAML:

```yaml
checkpoint_config:
  every_n_epochs: 1               # Save checkpoint every N epochs
  auto_insert_metric_name: false  # Don't add metric to filename
  save_last: true                 # Always save 'last.ckpt'
  save_top_k: -1                  # Keep all checkpoints (-1) or top K
```

To resume from the last checkpoint:

```yaml
experiment_settings:
  restart_checkpoint_path: last
```

---

## Inference

### Scheme 1: Inference with the default checkpoint

Several modes are available for inference. Using the **ColabFold MSA Server** generally
provides the best prediction accuracy. In this mode, MSAs are generated automatically
through the ColabFold server, with only protein sequences sent to the server. This mode is
recommended when predicting a small number of structures.

```bash
pixi run -e openfold3-cuda12 run_openfold predict \
  --query-json examples/example_inference_inputs/query_protein_ligand.json
```

| Argument | Required | Description |
| --- | --- | --- |
| `--query-json` | **Yes** | Path to the input query JSON file. |
| `--inference-ckpt-path` | No | Path to a model checkpoint file (`.pt`). If omitted, the default checkpoint is used and downloaded automatically on the first inference run. |
| `--output-dir` | No | Directory where inference outputs are written. Defaults to `test_train_output/`. |

> Every flag also accepts an underscore spelling — `--query_json` works the same as
> `--query-json`. Additional example inputs are in
> [`examples/example_inference_inputs/`](examples/example_inference_inputs/).

---

## About this repository

This repository carries the full upstream history of
[aqlaboratory/openfold-3](https://github.com/aqlaboratory/openfold-3) and is maintained by
Lambda. The upstream source tree is unmodified — only documentation has been added. See
[UPSTREAM.md](UPSTREAM.md) for the base commit and full list of changes.

Upstream's original README is preserved as [README_UPSTREAM.md](README_UPSTREAM.md), and
full OpenFold3 documentation lives at
[openfold-3.readthedocs.io](https://openfold-3.readthedocs.io/en/latest/).

Licensed under Apache-2.0, same as upstream. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
