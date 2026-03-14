简体中文 | [English](README.en.md)

## 项目描述

本项目提供了一组 kustomize 的基础资源，用于在 K8s 启动 Openclaw 服务。项目预配置了 `dev` 环境。但它不一定满足你的需求，你可以根据需要来编写一个 `kustomization.yaml` 来定制你的环境覆盖 base 资源。

## 追踪版本

当前追踪上游 openclaw 版本：`2026.3.11`

## 前置要求

- kubectl
- kustomize

## 目录结构

```
├── base/                                   # 基础部署资源定义
│   ├── kustomization.yaml                  # 基础配置清单
│   ├── deployment.yaml                     # Gateway 主应用部署
│   ├── service.yaml                        # 服务暴露
│   ├── pvc.yaml                            # 持久化存储声明
│   └── openclaw.json                       # OpenClaw 配置文件
├── overlays/                               # 环境特定配置
│   └── development/                        # 开发环境配置
│       ├── kustomization.yaml              # 开发环境定制化配置
│       ├── ingress.yaml                    # 入口规则配置
│       ├── network-policies.yaml           # 网络策略
│       └── patches/                        # 定制化补丁
├── .github/workflows/ci.yml                # CI 测试流程
├── tests/                                  # 端到端测试
│   └── e2e/openclaw-running-check/         # OpenClaw 运行状态检查测试集
├── README.md                               # 项目说明文档
└── README.en.md                            # README 英文版
```

## 快速开始

#### 1. 预览最终生成的 YAML

在部署前，先查看渲染后的结果

```sh
kubectl kustomize overlays/development
```

#### 2. 执行部署

```sh
# 部署到开发环境
kubectl apply -k overlays/development
```

#### 3. 查看部署状态

```sh
# 查看所有资源状态
kubectl get all -n openclaw-dev

# 查看部署日志
kubectl logs -f deployment/openclaw-gateway -n openclaw-dev
```

## 环境对比

| 特性             | Base (默认) | Development              |
| :--------------- | :---------- | :----------------------- |
| **配置域名**     | N/A         | openclaw-dev.example.com |
| **StorageClass** | N/A         | 自定义                   |
| **命名空间**     | default     | openclaw-dev             |
| **配置文件**     | 默认        | N/A                      |
| **网络策略**     | N/A         | 自定义                   |
| **镜像**         | 默认        | 自定义                   |

## 架构特点

1. **双容器设计**：gateway 主容器与 chromium headless shell 容器的集成部署
2. **安全加固**：启用 ReadOnlyRootFilesystem，非 root 权限运行，最小权限原则
3. **技能初始化**：通过 init 容器预安装系统依赖（uv、pnpm）及默认技能（weather、gog、github）
4. **数据持久化**：通过 PVC 持久化用户工作区数据和配置
5. **健康检查**：配置健康存活探针确保应用稳定性

## 安全特性

- 容器以非 root 用户（UID/GID 1000）运行
- 启用 ReadOnlyRootFilesystem 以提高安全性
- 自动剔除所有权限提升能力（allowPrivilegeEscalation）
- 限制系统调用能力（drop ALL capabilities）

## 技能系统

- 初始化过程中会自动安装默认配置文件中的技能（目前包括 `weather`、`gog`、`github`）
- 提供接口依赖安装机制（Node.js、Python、Go 等）
- 支持从 ClawHub（https://clawhub.com）下载和安装新技能

## 维护指南

### 更新部署

直接修改 overlays 中的配置文件，如更新镜像版本：

```yaml
images:
  - name: ghcr.io/openclaw/openclaw:main-slim
    newName: ghcr.io/openclaw/openclaw
    newTag: main
```

### 添加额外技能

可在 deployment.yaml 中修改 init 容器的 install_skill 函数中添加新的技能名称。

## 感谢

- [openclaw-helm](https://github.com/serhanekicii/openclaw-helm)
