# CloudLink

[![Go Report Card](https://goreportcard.com/badge/github.com/turtacn/cloudlink)](https://goreportcard.com/report/github.com/turtacn/cloudlink)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/turtacn/cloudlink/blob/main/LICENSE)

**CloudLink** 是 Nexus Cloud 的数字底座，这是一个“数字便利店”式的混合云管理平台。它作为一个轻量、高效、中立的连接器生态系统，实现对多云和混合环境中异构云资源的标准化、实时访问和控制。与传统 IaC 工具不同，CloudLink 聚焦于命令式操作，用于即时管理，填补了声明式编排与实时 CMP 需求之间的鸿沟。

项目仓库：[https://github.com/turtacn/cloudlink](https://github.com/turtacn/cloudlink)

英文版请参阅 [README.md](README.md)。

## 主要痛点与核心价值

在多云与混合架构时代，企业面临资源异构、认证复杂、API 差异等挑战。传统工具如 Terraform（声明式 IaC）擅长终态编排，但不适合实时控制（如查询实时 CPU 使用率或即时关机）。Crossplane 将资源绑定到 Kubernetes CRD，限制了非 K8s 生态的灵活性。

**核心价值**：
- **连接一切，标准化一切，管理一切**：提供标准化抽象层，支持实时操作，降低集成成本，并确保未来演进性。
- **战略作用**：作为 Nexus Cloud 的统一入口和智能管家，提供平台中立性、快速市场响应和生态扩展。
- **差异化**：命令式实时焦点（相较 Terraform 的声明式模型）与纯 Go 灵活性（相较 Crossplane 的 K8s 依赖），使其成为可演化的“云连接操作系统内核”，适用于多租户、跨云场景。

## 主要功能特性

- **标准化接口**：Go 接口用于资源如 `VirtualMachine`、`StorageBucket` 等，强调实时控制。
- **插件式 Provider**：模块化适配器，支持版本管理、热插拔和远程注册。
- **统一认证**：集成 HashiCorp Vault，支持动态凭证、轮换和多租户隔离。
- **实时查询与控制**：支持实时状态拉取、通过 `context.Context` 取消操作，以及结构化错误处理。
- **缓存与优化**：基于 Redis 的缓存，用于资源列表和指标，支持 TTL 和失效机制。
- **可观测性**：Prometheus 指标和 OpenTelemetry 追踪，用于 API 调用、错误和插件健康。
- **安全与多租户**：运行时凭证拉取、mTLS 和按 TenantID/Region 隔离。
- **扩展性**：gRPC/HTTP 远程插件、Kubernetes sidecar 支持，以及事件钩子用于工作流。
- **SaaS 集成**：可扩展到非云资源如 Salesforce 或 GitHub。

详细架构请参阅 [docs/architecture.md](docs/architecture.md)。

## 架构概览

CloudLink 采用分层、模块化设计：
- **核心层**：标准化模型和接口。
- **Provider 层**：可插拔的厂商适配器。
- **基础设施层**：缓存（Redis）、秘密管理（Vault）、可观测性。
- **扩展层**：远程插件和事件驱动特性。

这确保了低耦合、高内聚和易测试性。参阅 [docs/architecture.md](docs/architecture.md) 获取图表和深入设计。

## 构建与运行指南

### 前置条件
- Go 1.20.2 或更高版本
- Redis（用于缓存）
- HashiCorp Vault（用于秘密管理）
- 可选：Docker 用于容器化部署

### 安装
```bash
git clone https://github.com/turtacn/cloudlink.git
cd cloudlink
go mod tidy
````

### 构建

```bash
go build -o cloudlink ./cmd/cloudlink
```

### 运行

```bash
./cloudlink --config config.yaml
```

示例 `config.yaml`：

```yaml
providers:
  - name: aws
    version: v1.0.0
cache:
  redis: redis://localhost:6379
vault:
  addr: http://localhost:8200
```

### 测试

运行单元测试：

```bash
go test ./...
```

使用内置验证工具：

```bash
go run tools/verify-provider --provider=aws
```

## 演示：代码片段

### 标准化接口示例

演示实时 VM 控制：

```go:internal/core/virtualmachine.go
package core

import "context"

// VirtualMachine 定义了 VM 的命令式操作。
type VirtualMachine interface {
    GetStatus(ctx context.Context, id string) (Status, error) // 实时状态查询
    Shutdown(ctx context.Context, id string) error            // 即时关机
}

type Status struct {
    CPUUsage float64 `json:"cpu_usage,omitempty"`
    Running  bool    `json:"running,omitempty"`
}
```

### Provider 注册示例

展示 AWS Provider 的插件注册：

```go:internal/providers/aws/provider.go
package aws

import (
    "context"
    "github.com/turtacn/cloudlink/internal/core"
    "github.com/turtacn/cloudlink/internal/registry"
)

type AWSProvider struct{}

func (p *AWSProvider) VirtualMachine() core.VirtualMachine {
    return &awsVM{} // 实现接口
}

func init() {
    registry.Register("aws", &AWSProvider{})
}
```

### 实时查询使用示例

```go:main.go
package main

import (
    "context"
    "fmt"
    "github.com/turtacn/cloudlink/internal/registry"
)

func main() {
    provider := registry.Get("aws")
    vm := provider.VirtualMachine()
    status, err := vm.GetStatus(context.Background(), "vm-123")
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Printf("CPU 使用率: %.2f%%\n", status.CPUUsage)
    }
}
```

这些片段展示了 CloudLink 的实时、命令式能力，与 Terraform 等声明式工具的区别。

## 贡献指南

欢迎贡献！请遵循以下步骤：

1. Fork 仓库。
2. 创建特性分支：`git checkout -b feature/new-feature`。
3. 提交变更：`git commit -m '添加新特性'`。
4. 推送分支：`git push origin feature/new-feature`。
5. 打开 Pull Request。

详情见 [CONTRIBUTING.md](CONTRIBUTING.md)。确保代码符合 Go 标准并包含测试。

Apache 2.0 许可。