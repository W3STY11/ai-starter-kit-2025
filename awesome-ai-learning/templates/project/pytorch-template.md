# PyTorch Project Template

A production-ready template for PyTorch deep learning projects.

## 🚀 Quick Start

```bash
# Create project structure
mkdir my-pytorch-project && cd my-pytorch-project

# Copy this template structure
cp -r /path/to/template/* .

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## 📁 Project Structure

```
my-pytorch-project/
├── README.md
├── requirements.txt
├── setup.py
├── .gitignore
├── LICENSE
│
├── config/
│   ├── config.yaml
│   ├── model_config.yaml
│   └── training_config.yaml
│
├── data/
│   ├── raw/
│   ├── processed/
│   ├── external/
│   └── README.md
│
├── models/
│   ├── checkpoints/
│   ├── production/
│   └── README.md
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_model_experiments.ipynb
│   └── README.md
│
├── src/
│   ├── __init__.py
│   ├── data/
│   │   ├── __init__.py
│   │   ├── dataset.py
│   │   ├── dataloader.py
│   │   └── transforms.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base_model.py
│   │   ├── resnet.py
│   │   └── custom_model.py
│   ├── training/
│   │   ├── __init__.py
│   │   ├── trainer.py
│   │   ├── losses.py
│   │   └── optimizers.py
│   ├── evaluation/
│   │   ├── __init__.py
│   │   ├── metrics.py
│   │   └── evaluator.py
│   └── utils/
│       ├── __init__.py
│       ├── logger.py
│       ├── visualization.py
│       └── helpers.py
│
├── scripts/
│   ├── train.py
│   ├── evaluate.py
│   ├── predict.py
│   └── preprocess_data.py
│
├── tests/
│   ├── __init__.py
│   ├── test_data.py
│   ├── test_models.py
│   └── test_training.py
│
└── deployment/
    ├── docker/
    │   ├── Dockerfile
    │   └── docker-compose.yml
    ├── api/
    │   ├── app.py
    │   └── requirements.txt
    └── scripts/
        └── export_model.py
```

## 📄 File Templates

### `requirements.txt`
```txt
# Core ML
torch>=2.0.0
torchvision>=0.15.0
numpy>=1.24.0
scipy>=1.10.0

# Data Processing
pandas>=2.0.0
scikit-learn>=1.2.0
pillow>=9.5.0

# Visualization
matplotlib>=3.7.0
seaborn>=0.12.0
tensorboard>=2.13.0

# Utilities
tqdm>=4.65.0
pyyaml>=6.0
wandb>=0.15.0

# Development
pytest>=7.3.0
black>=23.3.0
flake8>=6.0.0
mypy>=1.3.0
```

### `config/config.yaml`
```yaml
# Project Configuration
project:
  name: "my-pytorch-project"
  seed: 42
  device: "cuda"
  
# Data Configuration  
data:
  root_dir: "./data"
  batch_size: 32
  num_workers: 4
  train_split: 0.8
  val_split: 0.1
  test_split: 0.1

# Model Configuration
model:
  name: "resnet50"
  pretrained: true
  num_classes: 10
  dropout: 0.5

# Training Configuration
training:
  epochs: 100
  learning_rate: 0.001
  weight_decay: 0.0001
  early_stopping_patience: 10
  checkpoint_dir: "./models/checkpoints"
  
# Logging Configuration
logging:
  level: "INFO"
  tensorboard_dir: "./logs/tensorboard"
  wandb_project: "my-pytorch-project"
```

### `src/models/base_model.py`
```python
"""Base model class for PyTorch models."""
from abc import ABC, abstractmethod
from typing import Dict, Tuple, Optional
import torch
import torch.nn as nn


class BaseModel(nn.Module, ABC):
    """Abstract base class for all models."""
    
    def __init__(self, config: Dict):
        """Initialize the model.
        
        Args:
            config: Model configuration dictionary
        """
        super().__init__()
        self.config = config
        
    @abstractmethod
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """Forward pass of the model.
        
        Args:
            x: Input tensor
            
        Returns:
            Output tensor
        """
        pass
    
    def save(self, path: str) -> None:
        """Save model checkpoint.
        
        Args:
            path: Path to save the checkpoint
        """
        torch.save({
            'model_state_dict': self.state_dict(),
            'config': self.config
        }, path)
        
    def load(self, path: str) -> None:
        """Load model checkpoint.
        
        Args:
            path: Path to the checkpoint
        """
        checkpoint = torch.load(path)
        self.load_state_dict(checkpoint['model_state_dict'])
        self.config = checkpoint['config']
        
    def count_parameters(self) -> int:
        """Count the number of trainable parameters.
        
        Returns:
            Number of trainable parameters
        """
        return sum(p.numel() for p in self.parameters() if p.requires_grad)
```

### `scripts/train.py`
```python
#!/usr/bin/env python
"""Training script for PyTorch models."""
import argparse
import logging
from pathlib import Path
import yaml
import torch
from torch.utils.data import DataLoader

from src.data.dataset import CustomDataset
from src.models.resnet import ResNet50
from src.training.trainer import Trainer
from src.utils.logger import setup_logger


def parse_args():
    """Parse command line arguments."""
    parser = argparse.ArgumentParser(description="Train a PyTorch model")
    parser.add_argument(
        "--config",
        type=str,
        default="config/config.yaml",
        help="Path to configuration file",
    )
    parser.add_argument(
        "--resume",
        type=str,
        default=None,
        help="Path to checkpoint to resume from",
    )
    return parser.parse_args()


def main():
    """Main training function."""
    args = parse_args()
    
    # Load configuration
    with open(args.config, 'r') as f:
        config = yaml.safe_load(f)
    
    # Setup logging
    logger = setup_logger(config['logging']['level'])
    logger.info(f"Starting training with config: {args.config}")
    
    # Set device
    device = torch.device(config['project']['device'])
    logger.info(f"Using device: {device}")
    
    # Set seed for reproducibility
    torch.manual_seed(config['project']['seed'])
    
    # Create data loaders
    train_dataset = CustomDataset(
        root=config['data']['root_dir'],
        split='train',
    )
    train_loader = DataLoader(
        train_dataset,
        batch_size=config['data']['batch_size'],
        shuffle=True,
        num_workers=config['data']['num_workers'],
    )
    
    val_dataset = CustomDataset(
        root=config['data']['root_dir'],
        split='val',
    )
    val_loader = DataLoader(
        val_dataset,
        batch_size=config['data']['batch_size'],
        shuffle=False,
        num_workers=config['data']['num_workers'],
    )
    
    # Create model
    model = ResNet50(config['model'])
    model = model.to(device)
    logger.info(f"Model has {model.count_parameters():,} trainable parameters")
    
    # Create trainer
    trainer = Trainer(
        model=model,
        config=config,
        train_loader=train_loader,
        val_loader=val_loader,
        device=device,
    )
    
    # Resume from checkpoint if specified
    if args.resume:
        trainer.load_checkpoint(args.resume)
        logger.info(f"Resumed from checkpoint: {args.resume}")
    
    # Train the model
    trainer.train()
    
    logger.info("Training completed!")


if __name__ == "__main__":
    main()
```

### `.gitignore`
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
env/
*.egg-info/

# Data
data/raw/*
data/processed/*
data/external/*
!data/*/README.md

# Models
models/checkpoints/*
models/production/*
!models/*/README.md

# Logs
logs/
*.log
wandb/

# Jupyter
.ipynb_checkpoints/
*.ipynb_checkpoints

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Environment
.env
.env.local
```

## 🔧 Development Workflow

### 1. Data Preparation
```bash
# Download and prepare your data
python scripts/preprocess_data.py --input data/raw --output data/processed
```

### 2. Model Development
```bash
# Train model
python scripts/train.py --config config/config.yaml

# Evaluate model
python scripts/evaluate.py --checkpoint models/checkpoints/best_model.pth

# Make predictions
python scripts/predict.py --checkpoint models/production/model.pth --input data/test
```

### 3. Testing
```bash
# Run all tests
pytest tests/

# Run specific test
pytest tests/test_models.py -v

# Check code coverage
pytest --cov=src tests/
```

### 4. Code Quality
```bash
# Format code
black src/ scripts/ tests/

# Lint code
flake8 src/ scripts/ tests/

# Type checking
mypy src/ scripts/
```

## 🚀 Deployment

### Docker Deployment
```bash
# Build Docker image
docker build -t my-pytorch-model -f deployment/docker/Dockerfile .

# Run container
docker run -p 8000:8000 my-pytorch-model
```

### Model Export
```bash
# Export to ONNX
python deployment/scripts/export_model.py --format onnx

# Export to TorchScript
python deployment/scripts/export_model.py --format torchscript
```

## 📝 Best Practices

1. **Version Control**
   - Use meaningful commit messages
   - Tag releases with model versions
   - Track experiments with git branches

2. **Documentation**
   - Document all functions and classes
   - Keep README updated
   - Log all experiments

3. **Testing**
   - Write unit tests for critical functions
   - Test data pipelines thoroughly
   - Validate model outputs

4. **Reproducibility**
   - Set random seeds
   - Log all hyperparameters
   - Version control data and models

## 📚 Additional Resources

- [PyTorch Documentation](https://pytorch.org/docs/stable/)
- [PyTorch Tutorials](https://pytorch.org/tutorials/)
- [PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/) - For simpler training loops
- [Hydra](https://hydra.cc/) - For advanced configuration management