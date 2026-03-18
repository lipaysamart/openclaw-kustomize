English | [简体中文](README.md)

## Project Description

This project provides a set of basic resources for kustomize to launch Openclaw services in K8s. The project comes with a pre-configured `dev` environment. However, it may not meet your specific needs, so you can create a `kustomization.yaml` file to customize environment-specific overrides for the base resources.

## Tracked Version

Currently tracking upstream openclaw version: `2026.3.11`

## Prerequisites

- kubectl
- kustomize

## Directory Structure

```
├── base/                                   # Base deployment resource definitions
│   ├── kustomization.yaml                  # Base configuration manifest
│   ├── deployment.yaml                     # Gateway main application deployment
│   ├── service.yaml                        # Service exposure
│   ├── pvc.yaml                            # Persistent volume claim
│   └── openclaw.json                       # OpenClaw configuration file
├── overlays/                               # Environment-specific configurations
│   └── development/                        # Development environment configuration
│       ├── kustomization.yaml              # Development environment customization
│       ├── ingress.yaml                    # Ingress rules configuration
│       ├── network-policies.yaml           # Network policies
│       └── patches/                        # Custom patches
├── .github/workflows/ci.yml                # CI test workflow
├── tests/                                  # End-to-end tests
│   └── e2e/openclaw-running-check/         # OpenClaw runtime status check test suite
├── README.md                               # Project documentation
└── README.en.md                            # English version of README
```

## Quick Start

#### 1. Preview generated YAML

Before deploying, preview the rendered output

```sh
kubectl kustomize overlays/development
```

#### 2. Deploy

```sh
# Deploy to development environment
kubectl apply -k overlays/development
```

#### 3. Check deployment status

```sh
# Check status of all resources
kubectl get all -n openclaw-dev

# View deployment logs
kubectl logs -f deployment/openclaw-gateway -n openclaw-dev
```

## Environment Comparison

| Feature               | Base (Default) | Development              |
| :-------------------- | :------------- | :----------------------- |
| **Configured Domain** | N/A            | openclaw-dev.example.com |
| **StorageClass**      | N/A            | Custom                   |
| **Namespace**         | default        | openclaw-dev             |
| **Configuration**     | Default        | N/A                      |
| **Network Policy**    | N/A            | Custom                   |
| **Image**             | Default        | Custom                   |

## Architecture Features

1. **Dual Container Design**: Integrated deployment of gateway main container and chromium headless shell container
2. **Security Hardening**: ReadOnlyRootFilesystem enabled, runs with non-root privileges, follows least privilege principle
3. **Skill Initialization**: Pre-install system dependencies (uv, pnpm) and default skills (weather, gog, github) through init containers
4. **Smart Config Initialization**: Supports CONFIG_MODE environment variable to control initialization strategy - `merge` mode deeply merges default config with existing config (preserving user customizations), `overwrite` mode overwrites with default config
5. **Data Persistence**: Persist user workspace data and configurations through PVC
6. **Health Checks**: Configure liveness and readiness probes to ensure application stability

## Security Features

- Containers run as non-root user (UID/GID 1000)
- ReadOnlyRootFilesystem enabled to enhance security
- Auto-dropping all privilege escalation capabilities (allowPrivilegeEscalation)
- Limit system call capabilities (drop ALL capabilities)

## Skill System

- During initialization, skills specified in default configuration are automatically installed (currently including `weather`, `gog`, `github`)
- Provides dependency installation mechanism for interfaces (Node.js, Python, Go, etc.)
- Supports downloading and installing new skills from ClawHub (https://clawhub.com)

## Maintenance Guide

### Updating Deployment

Directly modify configuration files in overlays, such as updating image versions:

```yaml
images:
  - name: ghcr.io/openclaw/openclaw:main-slim
    newName: ghcr.io/openclaw/openclaw
    newTag: main
```

### Adding Additional Skills

Add new skill names in the install_skill function of the init container in deployment.yaml.

## Acknowledgements

- [openclaw-helm](https://github.com/serhanekicii/openclaw-helm)
