# Complete AI/ML Development Environment Setup Guide

This guide walks you through setting up a professional AI/ML development environment from scratch.

## 📋 Table of Contents

1. [System Requirements](#system-requirements)
2. [Python Environment](#python-environment)
3. [GPU Setup (Optional)](#gpu-setup-optional)
4. [Essential Tools](#essential-tools)
5. [IDE Setup](#ide-setup)
6. [Version Control](#version-control)
7. [Docker Setup](#docker-setup)
8. [Cloud Setup](#cloud-setup)
9. [Troubleshooting](#troubleshooting)

---

## 💻 System Requirements

### Minimum Requirements
- **OS**: Ubuntu 20.04+, macOS 11+, or Windows 10/11
- **RAM**: 8GB (16GB recommended)
- **Storage**: 50GB free space
- **CPU**: 4 cores (8 recommended)

### Recommended for Deep Learning
- **GPU**: NVIDIA GPU with 8GB+ VRAM
- **CUDA**: 11.8 or higher
- **RAM**: 32GB+
- **Storage**: 256GB+ SSD

---

## 🐍 Python Environment

### 1. Install Python

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-dev
```

#### macOS (using Homebrew)
```bash
brew install python@3.10
```

#### Windows
Download from [python.org](https://www.python.org/downloads/) or use:
```powershell
winget install Python.Python.3.10
```

### 2. Virtual Environment Setup

```bash
# Create project directory
mkdir my-ml-project && cd my-ml-project

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
# Linux/macOS:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Upgrade pip
python -m pip install --upgrade pip
```

### 3. Essential Python Packages

Create `requirements-base.txt`:
```txt
# Core Scientific Computing
numpy>=1.24.0
scipy>=1.10.0
pandas>=2.0.0
matplotlib>=3.7.0
seaborn>=0.12.0

# Machine Learning
scikit-learn>=1.3.0
xgboost>=1.7.0
lightgbm>=4.0.0

# Deep Learning (choose one)
torch>=2.0.0  # PyTorch
# tensorflow>=2.13.0  # TensorFlow

# Jupyter & Development
jupyter>=1.0.0
jupyterlab>=4.0.0
ipython>=8.14.0

# Utilities
tqdm>=4.65.0
python-dotenv>=1.0.0
pyyaml>=6.0
requests>=2.31.0

# Development Tools
black>=23.3.0
flake8>=6.0.0
pytest>=7.3.0
mypy>=1.3.0
pre-commit>=3.3.0
```

Install packages:
```bash
pip install -r requirements-base.txt
```

---

## 🎮 GPU Setup (Optional)

### NVIDIA GPU Setup

#### 1. Install NVIDIA Drivers

**Ubuntu:**
```bash
# Add NVIDIA PPA
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

# Install driver (replace 535 with latest version)
sudo apt install nvidia-driver-535

# Reboot
sudo reboot

# Verify installation
nvidia-smi
```

**Windows:**
Download from [NVIDIA Drivers](https://www.nvidia.com/download/index.aspx)

#### 2. Install CUDA Toolkit

```bash
# Ubuntu (CUDA 12.1)
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-12-1
```

#### 3. Install cuDNN

1. Download from [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)
2. Extract and copy files:
```bash
sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

#### 4. Install PyTorch with GPU

```bash
# For CUDA 11.8
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# For CUDA 12.1
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Verify GPU is available:
```python
import torch
print(torch.cuda.is_available())  # Should return True
print(torch.cuda.get_device_name(0))  # Should show your GPU
```

---

## 🛠️ Essential Tools

### 1. Git Configuration

```bash
# Install Git
sudo apt install git  # Ubuntu
brew install git      # macOS

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set up SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
cat ~/.ssh/id_ed25519.pub  # Copy this to GitHub/GitLab
```

### 2. Conda (Alternative to venv)

```bash
# Install Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# Create environment
conda create -n myenv python=3.10
conda activate myenv
```

### 3. Poetry (Advanced Package Management)

```bash
# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Initialize project
poetry new my-project
cd my-project
poetry add numpy pandas torch
```

---

## 💡 IDE Setup

### VS Code (Recommended)

1. Download from [code.visualstudio.com](https://code.visualstudio.com/)

2. Install essential extensions:
```bash
# Install via command line
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-toolsai.jupyter
code --install-extension github.copilot
code --install-extension eamodio.gitlens
```

3. Configure settings.json:
```json
{
    "python.linting.enabled": true,
    "python.linting.flake8Enabled": true,
    "python.formatting.provider": "black",
    "python.formatting.blackArgs": ["--line-length", "88"],
    "editor.formatOnSave": true,
    "python.testing.pytestEnabled": true,
    "python.terminal.activateEnvironment": true
}
```

### PyCharm Professional

1. Download from [JetBrains](https://www.jetbrains.com/pycharm/)
2. Configure interpreter to use virtual environment
3. Enable scientific mode for data visualization

---

## 🔄 Version Control

### Pre-commit Hooks

Create `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=1000']
      
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
        language_version: python3.10
        
  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        args: ['--max-line-length=88', '--extend-ignore=E203']
```

Install pre-commit:
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files  # Test run
```

---

## 🐳 Docker Setup

### Install Docker

```bash
# Ubuntu
sudo apt update
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER
newgrp docker

# Test installation
docker run hello-world
```

### ML-specific Dockerfile Template

```dockerfile
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-runtime

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project files
COPY . .

# Expose ports
EXPOSE 8888 6006

CMD ["python", "app.py"]
```

---

## ☁️ Cloud Setup

### AWS Setup

```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure
aws configure
```

### Google Cloud Setup

```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Initialize
gcloud init
```

### Azure Setup

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login
```

---

## 🔧 Troubleshooting

### Common Issues

#### 1. CUDA/GPU Not Detected
```bash
# Check NVIDIA driver
nvidia-smi

# Check CUDA version
nvcc --version

# Verify PyTorch CUDA
python -c "import torch; print(torch.cuda.is_available())"
```

#### 2. Package Conflicts
```bash
# Create fresh environment
python -m venv venv_fresh
source venv_fresh/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

#### 3. Memory Issues
```bash
# Clear pip cache
pip cache purge

# Clear conda cache
conda clean --all

# Monitor memory usage
htop  # or top
```

#### 4. Permission Issues
```bash
# Fix permissions
sudo chown -R $USER:$USER ~/.cache/pip
sudo chown -R $USER:$USER ~/my-project
```

---

## 📚 Next Steps

1. **Set up experiment tracking**: See [Experiment Tracking Guide](../best-practices/experiment-tracking.md)
2. **Configure data pipeline**: See [Data Pipeline Setup](data-pipeline.md)
3. **Learn debugging techniques**: See [Debugging ML Code](../development/debugging.md)
4. **Set up monitoring**: See [Monitoring Guide](../deployment/monitoring.md)

---

## 🔗 Additional Resources

- [NVIDIA CUDA Installation Guide](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)
- [PyTorch Get Started](https://pytorch.org/get-started/locally/)
- [TensorFlow Installation](https://www.tensorflow.org/install)
- [Docker for Data Science](https://docker-curriculum.com/)

---

<div align="center">
  <p>Need help? Open an issue in our <a href="https://github.com/W3STY11/awesome-ai-learning/issues">GitHub repository</a></p>
</div>