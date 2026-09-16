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
| `target_cloud` | Cloud (`aws` \| `azure`) used to choose between clusters of the same environment. Ignored when `target_cluster` is set; inferred from the runner when omitted. | ❌ No | — |
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
| `cloud` | Selected cluster's cloud (`aws` \| `azure`). Empty when the `env_map` does not declare one. |

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

## 🧭 Which cluster is selected

In order, first match wins:

1. **`target_cluster`** — names one cluster, searched across every environment.
   Nothing else is consulted. If `target_cloud` was also given and disagrees, it
   is ignored with a warning rather than failing: the cluster was named
   explicitly, so it is what the caller meant.
2. **`target_cloud`** — keeps the environment's clusters in that cloud. A
   cluster whose entry declares no `cloud` is never discarded by this filter, so
   an `env_map` written before the field existed resolves exactly as it did
   before. If the environment has clusters and none are in that cloud, the run
   fails and says which clouds it does have.
3. **The runner's own cloud**, detected via
   [`detect-cloud`](https://github.com/gitopsmanager/detect-cloud) — used **only
   to break a tie** between clusters of one environment, and only when nothing
   above resolved it. It is never a veto: a single-cluster environment is
   selected without consulting it at all, so a runner deploying across a VPN to
   the other cloud is unaffected. When it does decide, it says so with a
   warning.

If a choice still remains, the run fails and lists the candidates with their
clouds.

---

## 📦 Example `env_map` JSON

```json
{
  "prod": {
    "clusters": [
      {
        "cluster": "aks-prod-weu",
        "cloud": "azure",
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
        "cloud": "azure",
        "dns_zone": "staging.affinity7software.com",
        "container_registry": "acrstaging.azurecr.io"
      }
    ]
  }
}
```

---


## 🔢 Versioning Policy — Official Release

Starting with this release, all future versions follow the new **stable semantic tagging policy**.

Previously, tags like `v1`, `v1.4`, and `v1.4.7` might have moved unpredictably.  
From now on, they will always follow these simple, reliable rules:

| Tag | Moves When | Purpose |
|------|-------------|----------|
| **`v1`** | Any new release in the `v1.x.x` series | Always points to the latest stable release in the major version line. |
| **`v1.4`** | A new patch in the same minor version (e.g. `v1.4.7 → v1.4.8`) | Stays within that feature line. Receives only bug fixes and optimizations — no breaking changes. |
| **`v1.4.7`** | Never | Fully immutable and reproducible. |

### How to choose
- **`@v1`** → Always up to date with the newest non-breaking changes.  
- **`@v1.4`** → Stable feature line with fixes only.  
- **`@v1.4.7`** → Frozen snapshot for exact reproducibility.

All tags will now **increment forward permanently** — no re-use or re-tagging of old versions.  
This marks the **official release** of the action under the **Affinity7 Consulting** stable versioning model.

## 🧩 Dependencies

| Action | Description | Repository |
|--------|--------------|-------------|
| **github-script** | Executes JavaScript for environment parsing and JSON transformation. | [`actions/github-script`](https://github.com/actions/github-script) |

---

## 📄 License

MIT License © Affinity7 Consulting Ltd
