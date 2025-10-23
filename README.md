# ollama-inference
# Llama Inference Helm Chart

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square)
![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)
![AppVersion: latest](https://img.shields.io/badge/AppVersion-latest-informational?style=flat-square)

A Helm chart for deploying Llama language models using Ollama in OpenShift/Kubernetes environments.

## Features

- 🚀 **Easy Deployment**: One-command deployment of Llama models
- 🔄 **Auto-Pull Models**: Automatically downloads and configures models on startup
- 💾 **Persistent Storage**: Models cached in persistent volumes for faster restarts
- 🔒 **OpenShift Ready**: Includes Route configuration with TLS support
- 📊 **Health Checks**: Built-in liveness and readiness probes
- ⚡ **Configurable Resources**: Easily adjust CPU/memory based on model size
- 🎯 **Multiple Model Support**: Switch between Llama 3.2, Llama 2, and other models

## Prerequisites

- Kubernetes 1.19+ or OpenShift 4.x+
- Helm 3.0+
- Persistent Volume provisioner support in the cluster
- At least 4GB of available memory (8GB+ recommended)
- 20GB+ of available storage

## Installation

### Quick Start

```bash
# Add the repository (if published)
helm repo add llama-inference https://your-repo-url
helm repo update

# Install with default settings (Llama 3.2 1B model)
helm install my-llama llama-inference/llama-inference
```

### Install from Source

```bash
# Clone the repository
git clone https://github.com/yourusername/llama-inference-helm.git
cd llama-inference-helm

# Install the chart
helm install my-llama ./llama-inference
```

### Install in a Specific Namespace

```bash
# Create namespace
kubectl create namespace ai-models

# Install
helm install my-llama ./llama-inference -n ai-models
```

## Configuration

### Basic Configuration Examples

#### Deploy Llama 2 7B Model

```bash
helm install llama2-7b ./llama-inference \
  --set model.name=llama2:7b \
  --set resources.limits.memory=16Gi \
  --set resources.requests.memory=8Gi \
  --set persistence.size=30Gi
```

#### Deploy Llama 3.2 3B Model

```bash
helm install llama3-3b ./llama-inference \
  --set model.name=llama3.2:3b \
  --set resources.limits.memory=12Gi \
  --set resources.requests.memory=6Gi
```

#### Deploy Without Auto-Pull (Manual Model Management)

```bash
helm install my-llama ./llama-inference \
  --set model.autoPull=false
```

#### Custom Values File

Create a `custom-values.yaml`:

```yaml
model:
  name: "llama2:13b"
  autoPull: true

resources:
  limits:
    cpu: 8
    memory: 32Gi
  requests:
    cpu: 4
    memory: 16Gi

persistence:
  size: 50Gi

route:
  enabled: true
  host: "llama.example.com"
```

Install with custom values:

```bash
helm install my-llama ./llama-inference -f custom-values.yaml
```

## Configuration Parameters

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full name | `""` |

### Image Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Ollama image repository | `docker.io/ollama/ollama` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### Model Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `model.name` | Llama model to deploy | `llama3.2:1b` |
| `model.autoPull` | Auto-pull model on startup | `true` |

**Available Models:**
- `llama3.2:1b` - Llama 3.2 1B (Smallest, ~2GB RAM)
- `llama3.2` or `llama3.2:3b` - Llama 3.2 3B (~6GB RAM)
- `llama2` or `llama2:7b` - Llama 2 7B (~8GB RAM)
- `llama2:13b` - Llama 2 13B (~16GB RAM)
- `llama2:70b` - Llama 2 70B (~64GB RAM)

### Service Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `11434` |
| `service.targetPort` | Container port | `11434` |

### Route Parameters (OpenShift)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `route.enabled` | Enable OpenShift Route | `true` |
| `route.host` | Custom hostname | `""` |
| `route.tls.enabled` | Enable TLS | `true` |
| `route.tls.termination` | TLS termination type | `edge` |

### Resource Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `4` |
| `resources.limits.memory` | Memory limit | `8Gi` |
| `resources.requests.cpu` | CPU request | `2` |
| `resources.requests.memory` | Memory request | `4Gi` |

### Persistence Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.storageClass` | Storage class name | `""` |
| `persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `persistence.size` | Volume size | `20Gi` |
| `persistence.mountPath` | Mount path | `/root/.ollama` |

### Health Check Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe.enabled` | Enable liveness probe | `true` |
| `livenessProbe.initialDelaySeconds` | Initial delay | `60` |
| `readinessProbe.enabled` | Enable readiness probe | `true` |
| `readinessProbe.initialDelaySeconds` | Initial delay | `30` |

## Usage

### Get Application URL

```bash
# For OpenShift
export ROUTE_HOST=$(oc get route my-llama-llama-inference -o jsonpath='{.spec.host}')
echo "API URL: https://$ROUTE_HOST"

# For Kubernetes (with port-forward)
kubectl port-forward svc/my-llama-llama-inference 11434:11434
echo "API URL: http://localhost:11434"
```

### Test the API

#### Generate Text

```bash
curl https://$ROUTE_HOST/api/generate -d '{
  "model": "llama3.2:1b",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

#### Chat Completion

```bash
curl https://$ROUTE_HOST/api/chat -d '{
  "model": "llama3.2:1b",
  "messages": [
    {
      "role": "user",
      "content": "What is the capital of France?"
    }
  ]
}'
```

#### List Available Models

```bash
curl https://$ROUTE_HOST/api/tags
```

#### Show Model Information

```bash
curl https://$ROUTE_HOST/api/show -d '{
  "name": "llama3.2:1b"
}'
```

### Monitor Deployment

```bash
# Watch pod status
kubectl get pods -w

# View logs
kubectl logs -f deployment/my-llama-llama-inference

# View model pull logs (init container)
kubectl logs deployment/my-llama-llama-inference -c pull-model
```

### Manually Pull Additional Models

```bash
# Exec into the pod
kubectl exec -it deployment/my-llama-llama-inference -- /bin/bash

# Pull additional models
ollama pull llama2:7b
ollama pull codellama

# List installed models
ollama list
```

## Upgrading

### Upgrade to a Different Model

```bash
helm upgrade my-llama ./llama-inference \
  --set model.name=llama2:7b \
  --set resources.limits.memory=16Gi \
  --reuse-values
```

### Upgrade with New Values File

```bash
helm upgrade my-llama ./llama-inference -f new-values.yaml
```

## Uninstallation

```bash
# Uninstall the release
helm uninstall my-llama

# Delete the PVC (if needed)
kubectl delete pvc my-llama-llama-inference-models
```

## Troubleshooting

### Pod Not Starting

**Check pod status:**
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

**Common issues:**
- Insufficient memory: Increase `resources.limits.memory`
- Storage issues: Check PVC status with `kubectl get pvc`
- Image pull errors: Check image repository and network connectivity

### Model Pull Failing

**Check init container logs:**
```bash
kubectl logs <pod-name> -c pull-model
```

**Solutions:**
- Ensure internet connectivity from cluster
- Verify model name is correct
- Check available disk space

### Out of Memory Errors

**Symptoms:**
- Pod gets OOMKilled
- Pod crashes during inference

**Solutions:**
```bash
# Increase memory limits
helm upgrade my-llama ./llama-inference \
  --set resources.limits.memory=16Gi \
  --set resources.requests.memory=8Gi
```

### Slow Inference

**Recommendations:**
- Use smaller models (1B or 3B parameters)
- Increase CPU allocation
- Consider GPU-enabled nodes if available
- Use quantized models (Q4, Q5)

## Model Size Reference

| Model | Parameters | RAM Required | Storage | Inference Speed |
|-------|-----------|--------------|---------|-----------------|
| Llama 3.2 1B | 1B | 2-4 GB | 1-2 GB | Fast |
| Llama 3.2 3B | 3B | 4-8 GB | 3-6 GB | Medium |
| Llama 2 7B | 7B | 8-16 GB | 7-14 GB | Medium-Slow |
| Llama 2 13B | 13B | 16-32 GB | 13-26 GB | Slow |
| Llama 2 70B | 70B | 64-128 GB | 70+ GB | Very Slow |

## API Documentation

Full Ollama API documentation: https://github.com/ollama/ollama/blob/main/docs/api.md

### Common Endpoints

- `POST /api/generate` - Generate completion
- `POST /api/chat` - Chat completion
- `POST /api/pull` - Pull a model
- `GET /api/tags` - List local models
- `POST /api/show` - Show model information
- `DELETE /api/delete` - Delete a model

## Examples

### Python Client

```python
import requests
import json

url = "https://your-route-host/api/generate"
data = {
    "model": "llama3.2:1b",
    "prompt": "Explain quantum computing in simple terms",
    "stream": False
}

response = requests.post(url, json=data)
result = response.json()
print(result['response'])
```

### cURL with Streaming

```bash
curl https://$ROUTE_HOST/api/generate -d '{
  "model": "llama3.2:1b",
  "prompt": "Write a short poem about clouds",
  "stream": true
}'
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/llama-inference-helm/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/llama-inference-helm/discussions)
- **Ollama Docs**: https://github.com/ollama/ollama

## Acknowledgments

- [Ollama](https://ollama.ai/) - For providing the excellent model serving platform
- [Meta AI](https://ai.meta.com/) - For developing the Llama models
- OpenShift/Kubernetes community

## Changelog

### Version 0.1.0 (Initial Release)
- Initial Helm chart release
- Support for Ollama-based Llama model deployment
- Auto-pull model functionality
- OpenShift Route support
- Persistent volume support
- Health checks and monitoring

---

**Made with ❤️ for the AI/ML community**
