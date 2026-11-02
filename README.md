Your Poetry skeleton is fine, but switch away from Python 3.14. The book’s official project currently supports Python `>=3.11,<3.14`, and Géron recommends Python 3.12. Your existing environment uses Python 3.14.4, which may cause dependency failures. [Official project configuration](https://github.com/ageron/handson-mlp/blob/main/pyproject.toml) · [Installation guide](https://github.com/ageron/handson-mlp/blob/main/INSTALL.md)

## 1. Use Python 3.12

You already have `pyenv`, so install a current Python 3.12 release, then select it:

```bash
pyenv install --list | rg '^\s+3\.12\.[0-9]+$' | tail
pyenv install 3.12.x
pyenv local 3.12.x

poetry env use "$(pyenv which python)"
poetry run python --version
```

Replace `3.12.x` with the newest version displayed by the first command.

Change this line in `pyproject.toml`:

```toml
requires-python = ">=3.12,<3.14"
```

## 2. Use a chapter-oriented layout

I recommend:

```text
ML_B2A/
├── notebooks/
│   ├── 00_tools/
│   ├── 01_ml_landscape/
│   ├── 02_end_to_end_project/
│   ├── 03_classification/
│   └── ...
├── notes/
│   ├── 01_ml_landscape/
│   │   ├── README.md
│   │   └── concepts.md
│   ├── 02_end_to_end_project/
│   └── ...
├── data/
│   ├── raw/              # Original, immutable data
│   ├── interim/          # Partially processed data
│   └── processed/        # Model-ready data
├── models/               # Saved models and checkpoints
├── outputs/
│   ├── figures/
│   └── reports/
├── src/ml_b2a/
│   ├── __init__.py
│   ├── data/
│   ├── models/
│   ├── visualization/
│   └── utils/
├── tests/
├── pyproject.toml
├── poetry.lock
└── README.md
```

Use notebooks for exploration and learning. Move reusable functions into `src/ml_b2a/` once they become useful in more than one notebook.

## 3. Install the initial dependencies

Start with the core stack rather than installing every library used across all 19 chapters:

```bash
poetry add \
  numpy \
  pandas \
  scipy \
  matplotlib \
  seaborn \
  scikit-learn \
  jupyterlab \
  ipykernel \
  ipywidgets \
  tqdm \
  joblib
```

Add development tools:

```bash
poetry add --group dev \
  pytest \
  ruff \
  black \
  nbdime
```

Install PyTorch when you reach the neural-network chapters:

```bash
poetry add torch torchvision torchaudio torchmetrics
```

On an Apple Silicon Mac, the standard PyTorch installation supports MPS acceleration; you do not need a CUDA build.

You can verify it later with:

```python
import torch

device = (
    "mps" if torch.backends.mps.is_available()
    else "cuda" if torch.cuda.is_available()
    else "cpu"
)
print(device)
```

## 4. Register the Poetry environment with Jupyter

```bash
poetry install

poetry run python -m ipykernel install \
  --user \
  --name ml-b2a \
  --display-name "Python (ML B2A)"
```

Start JupyterLab:

```bash
poetry run jupyter lab
```

Choose **Python (ML B2A)** as the kernel in every notebook.

## 5. Configure Git appropriately

A suitable `.gitignore` is:

```gitignore
# Python
__pycache__/
*.py[cod]
.pytest_cache/
.ruff_cache/
.mypy_cache/

# Virtual environments
.venv/

# Jupyter
.ipynb_checkpoints/

# macOS
.DS_Store

# Data generated or downloaded locally
data/raw/*
data/interim/*
data/processed/*
!data/**/.gitkeep

# Models and generated output
models/*
outputs/*
!models/.gitkeep
!outputs/**/.gitkeep

# Environment secrets
.env
```

Commit `poetry.lock`; it makes your learning environment reproducible. Avoid committing large datasets, trained models, API keys, or notebook checkpoints.

For readable notebook diffs, enable `nbdime`:

```bash
poetry run nbdime config-git --enable --global
```

## 6. Add packages chapter by chapter

Later chapters may need packages such as:

```bash
# Classical ML
poetry add xgboost statsmodels optuna

# NLP and transformers
poetry add transformers datasets sentence-transformers sentencepiece accelerate

# Computer vision and generative models
poetry add diffusers ultralytics pillow

# Reinforcement learning
poetry add gymnasium pygame stable-baselines3
```

This incremental approach keeps early installation fast and makes it clear which chapter introduced each dependency.

One small naming suggestion: use `ml_b2a` only as the importable Python package name, but describe the repository in the README as something like **Hands-On ML Study Notes and Experiments**. That will make its purpose much clearer six months from now.
