# 🌐 CloudLink - The Unified Cloud Connector Kit

CloudLink is a pluggable and lightweight Go library for building cloud-agnostic connectors. Designed for real-time control and visibility across hybrid, multi-cloud, and SaaS environments, CloudLink provides a unified interface for resource management with strong security and low integration overhead.

> **Built for CMPs (Cloud Management Platforms), not IaC.**
> Real-time operations. Standardized interfaces. One API to connect them all.

---

## ✨ Features

- 🧩 **Pluggable Architecture** — Add new cloud providers or SaaS systems as independent plugins
- 🛠 **Standardized Interfaces** — Unified Go interfaces for VMs, networks, storage, and more
- 🔐 **Secure Credential Handling** — Integrated with HashiCorp Vault, supports AK/SK, OAuth2, STS
- 📈 **Real-Time Control** — Built for operational tasks, not just desired state management
- ⚙️ **Low Overhead** — Lightweight core with no external dependencies, embeddable in any app
- 🧪 **Built-in Validation Toolkit** — Automated plugin compliance tests
- 📦 **Redis Caching** — Reduce rate limits and latency from frequent cloud API calls
- ☁️ **Vendor Neutrality** — Works equally well with public clouds, private IaaS, and SaaS APIs

---

## 📦 Use Cases

- Build your own **Cloud Management Platform (CMP)**
- Connect new cloud providers quickly to existing DevOps systems
- Provide real-time resource dashboards and control panels
- Automate operations across hybrid/multi-cloud environments

---

## 🚀 Getting Started

### 🔧 Installation

```bash
go get github.com/turtacn/cloudlink
````

### 💡 Basic Usage

```go
import (
    "context"
    "github.com/turtacn/cloudlink/core"
    "github.com/turtacn/cloudlink/provider/aws"
)

func main() {
    ctx := context.Background()

    // Initialize the provider
    p := aws.NewProvider(core.ProviderConfig{
        AccessKey: "AKIA...",
        SecretKey: "*****",
        Region:    "us-west-1",
    })

    // Use the standard interface
    vms, err := p.ListVirtualMachines(ctx)
    if err != nil {
        log.Fatal(err)
    }

    for _, vm := range vms {
        fmt.Println(vm.Name, vm.State)
    }
}
```

---

## 🔌 Writing Your Own Provider Plugin

Each provider must implement a set of standard interfaces like `VirtualMachineProvider`, `StorageProvider`, etc.

📖 See the [Plugin Developer Guide](docs/plugin_dev.md)

---

## 📊 Architecture Overview

```
             ┌──────────────────────────────────────────┐
             │              CloudLink Core              │
             ├──────────────┬──────────────┬────────────┤
             │  VM Provider │ Storage Prov │ Network Prov
             └────┬─────────┴────┬─────────┴────┬───────┘
                  │              │              │
     ┌────────────┴─────┐ ┌──────┴────────┐ ┌───┴──────────┐
     │    AWS Plugin     │ │ Azure Plugin │ │ VMware Plugin │
     └───────────────────┘ └──────────────┘ └───────────────┘
```

---

## ✅ Contributing

We welcome contributions! Please check our [Contributing Guide](CONTRIBUTING.md) and run the test suite before submitting a PR.

### 🧪 Run Plugin Compatibility Tests

```bash
go run tools/verify-provider --provider=aws
```

---

## 🔐 Security

We take credential security seriously.

* 🔐 Cloud credentials are never stored in plaintext
* 🔐 Integration with Vault for dynamic secret injection
* 🔐 Context-based timeout and cancellation for all API calls

---

## 📄 License

This project is licensed under the [Apache 2.0 License](LICENSE).

---

## 💬 Community & Support

* 📢 Issues: [GitHub Issues](https://github.com/turtacn/cloudlink/issues)
* 🤝 Discussions: [GitHub Discussions](https://github.com/turtacn/cloudlink/discussions)
* 📬 Contact: [cloudlink-dev@yourcompany.com](mailto:cloudlink-dev@yourcompany.com)

---

> CloudLink — Connect everything. Manage securely. Operate in real time.

