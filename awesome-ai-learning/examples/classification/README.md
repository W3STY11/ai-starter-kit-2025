# Image Classification Example

A clean, production-ready image classification implementation using PyTorch.

## 🎯 Overview

This example demonstrates:
- Clean code structure for image classification
- Best practices for data loading and augmentation
- Modular model architecture
- Proper training/validation loops
- Model export for deployment
- Performance optimization techniques

## 📁 Project Structure

```
classification/
├── data/
│   ├── train/
│   ├── val/
│   └── test/
├── models/
│   ├── __init__.py
│   ├── resnet.py
│   └── efficientnet.py
├── utils/
│   ├── data_loader.py
│   ├── transforms.py
│   └── metrics.py
├── configs/
│   └── default.yaml
├── train.py
├── evaluate.py
├── predict.py
└── requirements.txt
```

## 🚀 Quick Start

### 1. Setup Environment

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Prepare Data

```bash
# Download example dataset (CIFAR-10)
python scripts/download_data.py --dataset cifar10

# Or use your own data
# Organize as: data/{train,val,test}/class_name/image.jpg
```

### 3. Train Model

```bash
# Train with default settings
python train.py --config configs/default.yaml

# Train with custom settings
python train.py \
    --model resnet50 \
    --epochs 50 \
    --batch-size 64 \
    --learning-rate 0.001
```

### 4. Evaluate

```bash
# Evaluate on test set
python evaluate.py --checkpoint checkpoints/best_model.pth

# Generate detailed report
python evaluate.py --checkpoint checkpoints/best_model.pth --detailed
```

### 5. Make Predictions

```bash
# Single image prediction
python predict.py --image path/to/image.jpg --checkpoint checkpoints/best_model.pth

# Batch prediction
python predict.py --input-dir path/to/images/ --output results.csv
```

## 📊 Results

| Model | Dataset | Accuracy | Params | Inference Time |
|-------|---------|----------|---------|----------------|
| ResNet50 | CIFAR-10 | 94.5% | 25.6M | 23ms |
| EfficientNet-B0 | CIFAR-10 | 95.2% | 5.3M | 31ms |

## 🔧 Configuration

### Training Configuration (configs/default.yaml)

```yaml
model:
  name: resnet50
  num_classes: 10
  pretrained: true
  dropout: 0.5

data:
  dataset: cifar10
  data_dir: ./data
  num_workers: 4
  pin_memory: true

training:
  epochs: 100
  batch_size: 32
  learning_rate: 0.001
  optimizer: adamw
  scheduler: cosine
  
augmentation:
  train:
    - RandomCrop: {size: 32, padding: 4}
    - RandomHorizontalFlip: {p: 0.5}
    - ColorJitter: {brightness: 0.2, contrast: 0.2}
  val:
    - CenterCrop: {size: 32}
```

## 💡 Key Features

### 1. Modular Architecture

```python
# models/resnet.py
class ResNetClassifier(nn.Module):
    def __init__(self, num_classes, pretrained=True):
        super().__init__()
        self.backbone = models.resnet50(pretrained=pretrained)
        self.backbone.fc = nn.Linear(2048, num_classes)
        
    def forward(self, x):
        return self.backbone(x)
```

### 2. Efficient Data Loading

```python
# utils/data_loader.py
def create_data_loader(data_dir, batch_size, is_train=True):
    transform = get_transforms(is_train)
    dataset = ImageFolder(data_dir, transform=transform)
    
    loader = DataLoader(
        dataset,
        batch_size=batch_size,
        shuffle=is_train,
        num_workers=4,
        pin_memory=torch.cuda.is_available()
    )
    return loader
```

### 3. Mixed Precision Training

```python
# train.py
scaler = torch.cuda.amp.GradScaler()

for batch in train_loader:
    optimizer.zero_grad()
    
    with torch.cuda.amp.autocast():
        outputs = model(inputs)
        loss = criterion(outputs, labels)
    
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

## 🚀 Deployment

### Export to ONNX

```bash
python scripts/export_onnx.py --checkpoint checkpoints/best_model.pth
```

### Export to TorchScript

```bash
python scripts/export_torchscript.py --checkpoint checkpoints/best_model.pth
```

### Serve with FastAPI

```bash
# Start API server
python serve/app.py --model checkpoints/model.onnx --port 8000

# Make prediction via API
curl -X POST -F "file=@image.jpg" http://localhost:8000/predict
```

## 📈 Performance Tips

1. **Use Mixed Precision**: ~2x faster training with minimal accuracy loss
2. **Enable cuDNN Autotuner**: `torch.backends.cudnn.benchmark = True`
3. **Optimize Data Loading**: Use multiple workers and pin memory
4. **Gradient Accumulation**: Simulate larger batch sizes on limited GPU memory

## 🧪 Testing

```bash
# Run all tests
pytest tests/

# Run specific test
pytest tests/test_model.py::test_forward_pass

# Check coverage
pytest --cov=src tests/
```

## 📚 References

- [PyTorch Image Models (timm)](https://github.com/rwightman/pytorch-image-models)
- [Albumentations for augmentations](https://github.com/albumentations-team/albumentations)
- [Mixed Precision Training](https://pytorch.org/docs/stable/amp.html)

---

<div align="center">
  <p>Ready to adapt this example for your own use case!</p>
</div>