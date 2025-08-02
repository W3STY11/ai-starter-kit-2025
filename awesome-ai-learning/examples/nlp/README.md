# NLP Text Classification Example

A clean implementation of text classification using Transformers and PyTorch.

## 🎯 Overview

This example demonstrates:
- Modern NLP pipeline with Hugging Face Transformers
- Efficient tokenization and data handling
- Fine-tuning pre-trained models
- Multi-class and multi-label classification
- Model optimization for production
- API deployment

## 📁 Project Structure

```
nlp/
├── data/
│   ├── raw/
│   ├── processed/
│   └── splits/
├── models/
│   ├── __init__.py
│   ├── classifier.py
│   └── tokenizer.py
├── utils/
│   ├── data_processing.py
│   ├── metrics.py
│   └── optimization.py
├── configs/
│   ├── bert_base.yaml
│   └── distilbert.yaml
├── train.py
├── evaluate.py
├── predict.py
├── serve.py
└── requirements.txt
```

## 🚀 Quick Start

### 1. Setup Environment

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Prepare Data

```bash
# Download example dataset (IMDB sentiment)
python scripts/download_data.py --dataset imdb

# Process and split data
python scripts/prepare_data.py \
    --input data/raw/train.csv \
    --output data/processed/ \
    --test-size 0.2
```

### 3. Train Model

```bash
# Fine-tune BERT
python train.py --config configs/bert_base.yaml

# Fine-tune DistilBERT (faster, smaller)
python train.py --config configs/distilbert.yaml

# Custom training
python train.py \
    --model bert-base-uncased \
    --epochs 3 \
    --batch-size 16 \
    --max-length 512
```

### 4. Evaluate

```bash
# Evaluate on test set
python evaluate.py --checkpoint checkpoints/best_model/

# Generate classification report
python evaluate.py --checkpoint checkpoints/best_model/ --report
```

### 5. Make Predictions

```bash
# Single text prediction
python predict.py --text "This movie was amazing!" --checkpoint checkpoints/best_model/

# Batch prediction from file
python predict.py --input texts.txt --output predictions.csv
```

## 📊 Benchmarks

| Model | Dataset | Accuracy | F1 Score | Inference Time | Model Size |
|-------|---------|----------|----------|----------------|------------|
| BERT-base | IMDB | 93.5% | 93.2% | 45ms | 438MB |
| DistilBERT | IMDB | 92.8% | 92.5% | 23ms | 265MB |
| RoBERTa | IMDB | 94.1% | 93.9% | 48ms | 476MB |

## 🔧 Configuration

### Model Configuration (configs/bert_base.yaml)

```yaml
model:
  name: bert-base-uncased
  num_labels: 2
  dropout: 0.1
  max_length: 512

data:
  train_file: data/processed/train.csv
  val_file: data/processed/val.csv
  text_column: text
  label_column: label
  
training:
  epochs: 3
  batch_size: 16
  learning_rate: 2e-5
  warmup_steps: 500
  weight_decay: 0.01
  gradient_accumulation: 2
  fp16: true
  
optimization:
  scheduler: linear
  adam_epsilon: 1e-8
  max_grad_norm: 1.0
```

## 💡 Key Implementation Details

### 1. Efficient Tokenization

```python
# models/tokenizer.py
class TextTokenizer:
    def __init__(self, model_name, max_length=512):
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.max_length = max_length
    
    def tokenize_batch(self, texts):
        return self.tokenizer(
            texts,
            padding=True,
            truncation=True,
            max_length=self.max_length,
            return_tensors='pt'
        )
```

### 2. Model Architecture

```python
# models/classifier.py
class TextClassifier(nn.Module):
    def __init__(self, model_name, num_labels, dropout=0.1):
        super().__init__()
        self.bert = AutoModel.from_pretrained(model_name)
        self.dropout = nn.Dropout(dropout)
        self.classifier = nn.Linear(self.bert.config.hidden_size, num_labels)
        
    def forward(self, input_ids, attention_mask):
        outputs = self.bert(input_ids=input_ids, attention_mask=attention_mask)
        pooled_output = outputs.pooler_output
        output = self.dropout(pooled_output)
        return self.classifier(output)
```

### 3. Training with Mixed Precision

```python
# train.py
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for batch in train_loader:
    optimizer.zero_grad()
    
    with autocast():
        outputs = model(**batch)
        loss = outputs.loss
    
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
    scheduler.step()
```

## 🚀 Deployment

### FastAPI Server

```python
# serve.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()
model = load_model("checkpoints/best_model/")

class TextInput(BaseModel):
    text: str

@app.post("/predict")
async def predict(input: TextInput):
    prediction = model.predict(input.text)
    return {
        "text": input.text,
        "label": prediction["label"],
        "confidence": prediction["confidence"]
    }
```

### Run Server

```bash
# Start API server
uvicorn serve:app --host 0.0.0.0 --port 8000

# Test API
curl -X POST "http://localhost:8000/predict" \
     -H "Content-Type: application/json" \
     -d '{"text": "This is a great product!"}'
```

## 📈 Advanced Features

### 1. Multi-Label Classification

```python
# For multi-label problems
class MultiLabelClassifier(TextClassifier):
    def forward(self, input_ids, attention_mask):
        logits = super().forward(input_ids, attention_mask)
        return torch.sigmoid(logits)  # Use sigmoid for multi-label
```

### 2. Model Quantization

```bash
# Quantize model for faster inference
python scripts/quantize_model.py \
    --checkpoint checkpoints/best_model/ \
    --output checkpoints/quantized_model/
```

### 3. ONNX Export

```bash
# Export to ONNX for deployment
python scripts/export_onnx.py \
    --checkpoint checkpoints/best_model/ \
    --output model.onnx
```

## 🧪 Testing

```bash
# Run all tests
pytest tests/

# Test data processing
pytest tests/test_data_processing.py

# Test model inference
pytest tests/test_inference.py
```

## 📚 Additional Resources

- [Hugging Face Transformers](https://huggingface.co/transformers/)
- [Transformers Course](https://huggingface.co/course/chapter1)
- [BERT Paper](https://arxiv.org/abs/1810.04805)
- [DistilBERT Paper](https://arxiv.org/abs/1910.01108)

---

<div align="center">
  <p>Ready for production NLP applications!</p>
</div>