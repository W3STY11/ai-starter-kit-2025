# ML Model Deployment Example

Production-ready deployment patterns for AI/ML models with Docker, Kubernetes, and cloud services.

## 🎯 Overview

This example covers:
- REST API with FastAPI
- Docker containerization
- Kubernetes deployment
- Model versioning and A/B testing
- Monitoring and logging
- Auto-scaling based on load
- CI/CD pipeline integration

## 📁 Project Structure

```
deployment/
├── api/
│   ├── app.py
│   ├── models.py
│   ├── middleware.py
│   └── utils.py
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.gpu
│   └── docker-compose.yml
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── hpa.yaml
├── monitoring/
│   ├── prometheus.yml
│   └── grafana-dashboard.json
├── tests/
│   ├── test_api.py
│   └── load_test.py
├── scripts/
│   ├── build.sh
│   ├── deploy.sh
│   └── rollback.sh
└── requirements.txt
```

## 🚀 Quick Start

### 1. Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally
uvicorn api.app:app --reload --port 8000

# Test endpoint
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [1.2, 3.4, 5.6]}'
```

### 2. Docker Deployment

```bash
# Build image
docker build -t ml-model:latest -f docker/Dockerfile .

# Run container
docker run -p 8000:8000 ml-model:latest

# Using docker-compose
docker-compose -f docker/docker-compose.yml up
```

### 3. Kubernetes Deployment

```bash
# Create namespace
kubectl create namespace ml-models

# Deploy application
kubectl apply -f kubernetes/ -n ml-models

# Check status
kubectl get pods -n ml-models
kubectl get svc -n ml-models
```

## 🔧 API Implementation

### FastAPI Application (api/app.py)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import numpy as np
from typing import List, Dict
import logging

app = FastAPI(title="ML Model API", version="1.0.0")
logger = logging.getLogger(__name__)

class PredictionRequest(BaseModel):
    features: List[float]
    model_version: str = "latest"

class PredictionResponse(BaseModel):
    prediction: float
    confidence: float
    model_version: str
    processing_time: float

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    start_time = time.time()
    
    try:
        # Load model based on version
        model = load_model(request.model_version)
        
        # Make prediction
        features = np.array(request.features).reshape(1, -1)
        prediction = model.predict(features)[0]
        confidence = model.predict_proba(features).max()
        
        processing_time = time.time() - start_time
        
        return PredictionResponse(
            prediction=prediction,
            confidence=confidence,
            model_version=request.model_version,
            processing_time=processing_time
        )
    
    except Exception as e:
        logger.error(f"Prediction error: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    return {"status": "healthy", "timestamp": datetime.utcnow()}

@app.get("/metrics")
async def metrics():
    return {
        "total_predictions": get_metric("predictions_total"),
        "avg_latency": get_metric("prediction_latency_avg"),
        "error_rate": get_metric("prediction_errors_rate")
    }
```

## 🐳 Docker Configuration

### Production Dockerfile

```dockerfile
# docker/Dockerfile
FROM python:3.9-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 mluser

# Set working directory
WORKDIR /app

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY api/ ./api/
COPY models/ ./models/

# Change ownership
RUN chown -R mluser:mluser /app

# Switch to non-root user
USER mluser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["uvicorn", "api.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### GPU Support Dockerfile

```dockerfile
# docker/Dockerfile.gpu
FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04

# Install Python and dependencies
RUN apt-get update && apt-get install -y \
    python3.9 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Rest similar to CPU Dockerfile...
```

## ☸️ Kubernetes Configuration

### Deployment (kubernetes/deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-model-api
  labels:
    app: ml-model-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-model-api
  template:
    metadata:
      labels:
        app: ml-model-api
    spec:
      containers:
      - name: ml-model
        image: ml-model:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        env:
        - name: MODEL_PATH
          value: "/models/latest"
        - name: LOG_LEVEL
          value: "INFO"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Auto-scaling (kubernetes/hpa.yaml)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ml-model-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ml-model-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## 📊 Monitoring Setup

### Prometheus Configuration

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'ml-model-api'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_label_app]
      action: keep
      regex: ml-model-api
```

### Custom Metrics

```python
# api/metrics.py
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
prediction_counter = Counter(
    'ml_predictions_total',
    'Total number of predictions made',
    ['model_version', 'status']
)

prediction_latency = Histogram(
    'ml_prediction_duration_seconds',
    'Prediction latency in seconds',
    ['model_version']
)

model_load_time = Gauge(
    'ml_model_load_time_seconds',
    'Model loading time in seconds',
    ['model_version']
)
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy ML Model

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run tests
      run: |
        pip install -r requirements.txt
        pytest tests/

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Build Docker image
      run: |
        docker build -t ${{ secrets.REGISTRY }}/ml-model:${{ github.sha }} .
        docker push ${{ secrets.REGISTRY }}/ml-model:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/ml-model-api \
          ml-model=${{ secrets.REGISTRY }}/ml-model:${{ github.sha }} \
          -n ml-models
```

## 🚦 A/B Testing

### Canary Deployment

```python
# api/middleware.py
import random

class ABTestingMiddleware:
    def __init__(self, app, model_weights):
        self.app = app
        self.model_weights = model_weights  # {"v1": 0.8, "v2": 0.2}
    
    async def __call__(self, request, call_next):
        # Randomly assign model version based on weights
        model_version = self._select_model()
        request.state.model_version = model_version
        
        response = await call_next(request)
        response.headers["X-Model-Version"] = model_version
        return response
    
    def _select_model(self):
        return random.choices(
            list(self.model_weights.keys()),
            weights=list(self.model_weights.values())
        )[0]
```

## 🧪 Load Testing

```python
# tests/load_test.py
import asyncio
import aiohttp
import time

async def make_request(session, url, data):
    async with session.post(url, json=data) as response:
        return await response.json()

async def load_test(url, num_requests=1000, concurrent=50):
    async with aiohttp.ClientSession() as session:
        data = {"features": [1.0, 2.0, 3.0]}
        
        start_time = time.time()
        tasks = []
        
        for _ in range(num_requests):
            task = make_request(session, url, data)
            tasks.append(task)
            
            if len(tasks) >= concurrent:
                await asyncio.gather(*tasks)
                tasks = []
        
        if tasks:
            await asyncio.gather(*tasks)
        
        duration = time.time() - start_time
        rps = num_requests / duration
        
        print(f"Completed {num_requests} requests in {duration:.2f}s")
        print(f"Requests per second: {rps:.2f}")

if __name__ == "__main__":
    asyncio.run(load_test("http://localhost:8000/predict"))
```

## 📈 Best Practices

1. **Model Versioning**: Always version your models and APIs
2. **Health Checks**: Implement comprehensive health endpoints
3. **Resource Limits**: Set appropriate CPU/memory limits
4. **Logging**: Use structured logging for better observability
5. **Security**: Use HTTPS, API keys, and rate limiting
6. **Caching**: Cache predictions when appropriate
7. **Graceful Shutdown**: Handle SIGTERM properly

## 🔗 Additional Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [MLOps Principles](https://ml-ops.org/)

---

<div align="center">
  <p>Production-ready ML deployment made simple!</p>
</div>