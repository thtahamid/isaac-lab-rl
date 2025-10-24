# Isaac Lab — Installation & Quick Start

This guide captures common steps to prepare a machine for running Isaac Lab / Isaac Sim, create a Python 3.11 virtual environment, install dependencies, and run example training/playback commands.

Official docs:

- Isaac Lab / Isaac Sim installation (pip): https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html

---

## Prerequisites

- Ubuntu 22.04 or a supported OS (or Windows for local development).
- Python 3.11 installed (system or pyenv).
- NVIDIA drivers and CUDA (if you plan to use GPU acceleration). Verify with `nvidia-smi`.
- Git and a GitHub account configured.

## 1) System checks

Check Python and GPU driver:

```bash
python3.11 --version
nvidia-smi
```

Configure git (if not already):

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## 2) Create and activate a Python 3.11 virtual environment

Linux / macOS:

```bash
python3.11 -m venv env_isaaclab
source env_isaaclab/bin/activate
```

Windows (PowerShell):

```powershell
python3.11 -m venv env_isaaclab
.\env_isaaclab\Scripts\Activate.ps1
```

Upgrade pip inside the venv:

```bash
pip install --upgrade pip
```

## 3) Install PyTorch (CUDA 12.8 wheel used in this repo)

Install the pinned PyTorch and torchvision versions used by the project:

```bash
pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
```

If you need a CPU-only build or a different CUDA version, consult PyTorch's installation page and adjust the index URL and package versions accordingly.

## 4) Install Isaac Sim / Isaac Lab pip package

Install the Isaac Sim pip package (example pinned version shown — replace with the version you require):

```bash
pip install "isaacsim[all,extscache]==5.0.0" --extra-index-url https://pypi.nvidia.com
```

Note: The extra index URL points to NVIDIA's PyPI repository where Isaac Sim binary wheels are hosted.

## 5) Clone Isaac Lab repository and run installer

Clone the repo and run the provided installer script for your platform.

```bash
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab-main
# Windows
.\isaaclab.bat --install
# Linux / macOS
./isaaclab.sh --install
```

Follow the install script output and any prompts. The installer resolves additional assets and dependencies required by Isaac Lab.

## 6) Run an instance / demo

After installation, follow the repository docs to start a local Isaac Lab instance or run one of the demos. See the official installation link above for platform-specific startup commands.

## 7) Training / running example policies

Train example policies (example command shown; adjust task and flags as needed):

```bash
python scripts/reinforcement_learning/rsl_rl/train.py \
  --task=Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs=5000 \
  --headless
```

Run/playback policies:

```bash
python scripts/reinforcement_learning/rsl_rl/play.py \
  --task=Isaac-Velocity-Rough-Anymal-C-v0 \
  --num_envs=20
```

## 8) SSH examples (connect to cloud/EC2 instance)

macOS / Linux (example):

```bash
ssh -i /Users/th/.ssh/isaac-sim-lab.pem ubuntu@ec2-3-28-126-137.me-central-1.compute.amazonaws.com
```

Windows (PowerShell example):

```powershell
ssh -i .\\.ssh\\isaac-sim-lab.pem ubuntu@ec2-3-28-126-137.me-central-1.compute.amazonaws.com
```

## Notes & links

- Official pip installation guide: https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html
- Replace pinned versions (Python, PyTorch, Isaac Sim) if your environment requires different versions.
- If you encounter GPU or driver issues, check `nvidia-smi`, your driver/CUDA compatibility, and the Isaac Sim troubleshooting docs.

If you want, I can:

- add the original `.txt` as a backup in the repo,
- or include example `tools/` files (systemd or verification scripts) for common tasks.
