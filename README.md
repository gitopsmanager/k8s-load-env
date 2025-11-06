# Load Environment Config  — *Open Source* & Stable `@v1`

**Parse and export cluster configuration from an env_map JSON definition.**  
This action selects the target cluster, exports the CD_ROOT path, and flattens the UAMI map (uami_name → client_id) for use by downstream steps such as ArgoCD application deployment or cluster restore workflows.

---

## 🧩 Overview

This action is part of the **GitOps Manager** automation suite.  
It standardizes how Kubernetes environments are resolved from an `env_map` JSON file or from an **environment variable (`ENV_MAP`)** defined on **self-hosted runners**, ensuring consistent handling of cluster names, DNS zones, container registries, and UAMI identities across workflows.

**What it does:**
1. Parses an inline or environment-based `env_map` JSON configuration.  
2. If running on a self-hosted runner, automatically checks for an environment variable named `ENV_MAP` containing the JSON definition.  
3. Selects the appropriate cluster for a given environment or explicit cluster name.  
4. Exports environment context (`cluster`, `dns_zone`, `container_registry`, and `cd_root`).  
5. Flattens the `uami_map` array into a JSON mapping of `uami_name → client_id`.  
6. Makes all outputs available for use in later steps or composite workflows.

---

## 🌐 GitOps Manager™ Enterprise Platform

[**GitOps Manager™ Enterprise**](https://gitopsmanager.io) is the full platform that powers this open-source workflow.  
It’s a **turnkey GitOps automation platform** for AWS and Azure — combining open-source GitHub Actions, Kubernetes infrastructure automation, and global-scale CI/CD.

**Highlights:**
- Secure, opinionated **multi-cloud GitOps automation** for Kubernetes workloads.  
- Deep integration with **ArgoCD**, **Argo Workflows**, **Traefik**, **ECK**, and **Kubernetes Dashboard**.  
- Built for **high availability**, **autoscaling**, and **managed upgrades**.  
- Supports **Workload Identity**, **Pod Identity**, and **private, network-isolated clusters**.  
- Enables **global deployments**, **secret management**, and **production-grade infrastructure** with **zero vendor lock-in**.

🔗 Learn more: [https://gitopsmanager.io](https://gitopsmanager.io)

---

## 🚀 Inputs

| Name | Description | Required | Default |
|------|--------------|-----------|----------|
| `env_map` | Inline JSON `env_map` defining environments, clusters, DNS zones, and UAMI maps. | ❌ No | — |
| `target_environment` | Logical environment name (e.g., prod, qa). | ✅ Yes | — |
| `target_cluster` | Optional cluster override to directly select a cluster. | ❌ No | — |
| `namespace` | Kubernetes namespace for the deployment context. | ✅ Yes | — |
| `delete_only` | Skip UAMI export if true. | ❌ No | `false` |

---

## 📤 Outputs

| Name | Description |
|------|--------------|
| `cluster` | Selected cluster name. |
| `dns_zone` | DNS zone for the selected cluster. |
| `container_registry` | Container registry associated with the cluster. |
| `uami_vars` | Flattened JSON map of `uami_name → client_id`. |
| `cd_root` | Continuous-deployment repo path for the selected cluster and namespace. |

---

## 🧠 Example Usage

```yaml
jobs:
  env:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Load Environment Config
        id: env
        uses: ./.github/actions/load-env
        with:
          env_map: ${{ inputs.env_map }}
          target_environment: prod
          target_cluster: aks-prod-weu
          namespace: web

      - name: Show extracted environment info
        run: |
          echo "Cluster: ${{ steps.env.outputs.cluster }}"
          echo "DNS Zone: ${{ steps.env.outputs.dns_zone }}"
          echo "CD Root: ${{ steps.env.outputs.cd_root }}"
          echo "UAMI Vars: ${{ steps.env.outputs.uami_vars }}"
```

---

## ⚙️ Typical Workflow Integration

This action is used by higher-level workflows such as:
- **k8s-deploy** → Selects the target cluster and UAMI context for ArgoCD app deployments.  
- **k8s-restore-cluster** → Resolves both source and target cluster info during restore operations.  
- **k8s-upgrade** → Loads environment configuration before upgrade orchestration.

---

## 📦 Example `env_map` JSON

```json
{
  "prod": {
    "clusters": [
      {
        "cluster": "aks-prod-weu",
        "dns_zone": "prod.affinity7software.com",
        "container_registry": "acrprod.azurecr.io",
        "uami_map": [
          { "uami_name": "aks-prod-weu-agent", "client_id": "b44b..." },
          { "uami_name": "aks-prod-weu-monitor", "client_id": "82aa..." }
        ]
      }
    ]
  },
  "staging": {
    "clusters": [
      {
        "cluster": "aks-staging-weu",
        "dns_zone": "staging.affinity7software.com",
        "container_registry": "acrstaging.azurecr.io"
      }
    ]
  }
}
```

---

## 🧩 Dependencies

| Action | Description | Repository |
|--------|--------------|-------------|
| **github-script** | Executes JavaScript for environment parsing and JSON transformation. | [`actions/github-script`](https://github.com/actions/github-script) |

---

## 📄 License

MIT License © Affinity7 Consulting Ltd
