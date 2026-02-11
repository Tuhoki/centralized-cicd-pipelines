# Centralized CI/CD Pipelines

Reusable GitHub Actions workflows for building, testing, and deploying containerized applications to Amazon EKS.

## Available Workflows

### build-push.yml
Builds Docker images and pushes them to Amazon ECR.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `service_name` | Yes | - | Name of the service |
| `service_path` | Yes | - | Path to service directory |
| `dockerfile_path` | No | `Dockerfile` | Path to Dockerfile |
| `ecr_repository` | Yes | - | ECR repository name |
| `aws_region` | No | `us-east-1` | AWS region |
| `build_args` | No | - | Docker build arguments |

**Secrets:**
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

**Outputs:**
- `image_tag`: The Docker image tag
- `image_uri`: Full ECR image URI

### deploy-eks.yml
Deploys Kubernetes manifests to EKS cluster.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `environment` | Yes | - | Target environment |
| `cluster_name` | Yes | - | EKS cluster name |
| `aws_region` | No | `us-east-1` | AWS region |
| `namespace` | No | `quickbite` | K8s namespace |
| `manifests_path` | Yes | - | Path to K8s manifests |
| `image_tag` | Yes | - | Docker image tag |

### test-java.yml
Runs Maven tests for Java services.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `service_path` | Yes | - | Path to Java service |
| `java_version` | No | `17` | Java version |

### test-node.yml
Runs npm tests for Node.js services.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `service_path` | Yes | - | Path to Node.js service |
| `node_version` | No | `20` | Node.js version |

## Usage Example

```yaml
name: My App CI/CD

on:
  push:
    branches: [main]

jobs:
  test:
    uses: YOUR_ORG/centralized-cicd-pipelines/.github/workflows/test-java.yml@main
    with:
      service_path: my-service

  build:
    needs: test
    uses: YOUR_ORG/centralized-cicd-pipelines/.github/workflows/build-push.yml@main
    with:
      service_name: my-service
      service_path: my-service
      ecr_repository: my-app
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    needs: build
    uses: YOUR_ORG/centralized-cicd-pipelines/.github/workflows/deploy-eks.yml@main
    with:
      environment: dev
      cluster_name: my-cluster-dev
      namespace: my-app
      manifests_path: k8s/base
      image_tag: ${{ github.sha }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## For Private Repositories

If this repository is private, calling repositories need to:
1. Be in the same organization
2. Have workflow permissions enabled

Go to **Settings → Actions → General → Access** and enable:
- "Accessible from repositories in the organization"
