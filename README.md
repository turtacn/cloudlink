# CloudLink - The Unified Cloud Connector Kit

[![Go Report Card](https://goreportcard.com/badge/github.com/turtacn/cloudlink)](https://goreportcard.com/report/github.com/turtacn/cloudlink)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/turtacn/cloudlink/blob/main/LICENSE)

**CloudLink** is the digital foundation of Nexus Cloud, a "digital convenience store" hybrid cloud management platform. It serves as a lightweight, efficient, and neutral connector ecosystem, enabling standardized, real-time access and control of heterogeneous cloud resources across multi-cloud and hybrid environments. Unlike traditional IaC tools, CloudLink focuses on imperative operations for immediate management, bridging the gap between declarative orchestration and real-time CMP needs.

Project Repository: [https://github.com/turtacn/cloudlink](https://github.com/turtacn/cloudlink)

For the Chinese version, see [README-zh.md](README-zh.md).

## Key Pain Points and Core Value

In the era of multi-cloud and hybrid architectures, enterprises face challenges like resource heterogeneity, complex authentication, and API disparities. Traditional tools like Terraform (declarative IaC) excel in end-state orchestration but fall short in real-time control (e.g., querying live CPU usage or instant shutdowns). Crossplane ties resources to Kubernetes CRDs, limiting flexibility outside K8s ecosystems.

**Core Value**:
- **Connect Everything, Standardize Everything, Manage Everything**: Provides a standardized abstraction layer for real-time operations, reducing integration costs and enabling future-proof evolution.
- **Strategic Role**: Acts as Nexus Cloud's unified entry and intelligent steward, offering platform neutrality, rapid market adaptation, and ecosystem expansion.
- **Differentiation**: Imperative and real-time focus (vs. Terraform's declarative model) with pure Go flexibility (vs. Crossplane's K8s dependency), making it an evolvable "cloud connection OS kernel" for multi-tenant, cross-cloud scenarios.

## Main Features

- **Standardized Interfaces**: Go interfaces for resources like `VirtualMachine`, `StorageBucket`, etc., emphasizing real-time control.
- **Plugin-Based Providers**: Modular adapters for cloud vendors, supporting versioning, hot-plugging, and remote registration.
- **Unified Authentication**: Integrates with HashiCorp Vault for dynamic credentials, rotation, and multi-tenant isolation.
- **Real-Time Query and Control**: Supports live status fetching, operation cancellation via `context.Context`, and structured error handling.
- **Caching and Optimization**: Redis-backed caching for resource lists and metrics, with TTL and invalidation.
- **Observability**: Prometheus metrics and OpenTelemetry tracing for API calls, errors, and plugin health.
- **Security and Multi-Tenancy**: Runtime credential fetching, mTLS, and isolation by TenantID/Region.
- **Extensibility**: gRPC/HTTP remote plugins, Kubernetes sidecar support, and event hooks for workflows.
- **SaaS Integration**: Extendable to non-cloud resources like Salesforce or GitHub.

For detailed architecture, see [docs/architecture.md](docs/architecture.md).

## Architecture Overview

CloudLink adopts a layered, modular design:
- **Core Layer**: Standardized models and interfaces.
- **Provider Layer**: Pluggable adapters for vendors.
- **Infrastructure Layer**: Caching (Redis), secrets (Vault), observability.
- **Extension Layer**: Remote plugins and event-driven features.

This ensures low coupling, high cohesion, and easy testing. Refer to [docs/architecture.md](docs/architecture.md) for diagrams and in-depth design.

## Build and Run Guide

### Prerequisites
- Go 1.20.2 or later
- Redis (for caching)
- HashiCorp Vault (for secrets management)
- Optional: Docker for containerized deployment

### Installation
```bash
git clone https://github.com/turtacn/cloudlink.git
cd cloudlink
go mod tidy
````

### Build

```bash
go build -o cloudlink ./cmd/cloudlink
```

### Run

```bash
./cloudlink --config config.yaml
```

Sample `config.yaml`:

```yaml
providers:
  - name: aws
    version: v1.0.0
cache:
  redis: redis://localhost:6379
vault:
  addr: http://localhost:8200
```

### Testing

Run unit tests:

```bash
go test ./...
```

Use the built-in verifier:

```bash
go run tools/verify-provider --provider=aws
```

## Demonstration: Code Snippets

### Standardized Interface Example

Demonstrates real-time VM control:

```go:internal/core/virtualmachine.go
package core

import "context"

// VirtualMachine defines imperative operations for VMs.
type VirtualMachine interface {
    GetStatus(ctx context.Context, id string) (Status, error) // Real-time status query
    Shutdown(ctx context.Context, id string) error            // Immediate shutdown
}

type Status struct {
    CPUUsage float64 `json:"cpu_usage,omitempty"`
    Running  bool    `json:"running,omitempty"`
}
```

### Provider Registration Example

Shows plugin registration for an AWS provider:

```go:internal/providers/aws/provider.go
package aws

import (
    "context"
    "github.com/turtacn/cloudlink/internal/core"
    "github.com/turtacn/cloudlink/internal/registry"
)

type AWSProvider struct{}

func (p *AWSProvider) VirtualMachine() core.VirtualMachine {
    return &awsVM{} // Implements the interface
}

func init() {
    registry.Register("aws", &AWSProvider{})
}
```

### Real-Time Query Usage

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
        fmt.Println("Error:", err)
    } else {
        fmt.Printf("CPU Usage: %.2f%%\n", status.CPUUsage)
    }
}
```

These snippets showcase CloudLink's real-time, imperative capabilities, differentiating it from declarative tools like Terraform.

## Contribution Guide

We welcome contributions! Please follow these steps:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/new-feature`.
3. Commit changes: `git commit -m 'Add new feature'`.
4. Push to the branch: `git push origin feature/new-feature`.
5. Open a Pull Request.

See [CONTRIBUTING.md](CONTRIBUTING.md) for details. Ensure code adheres to Go standards and includes tests.

Licensed under Apache 2.0.