# Snyk Broker (Code Agent Mode) - GitOps Deployment

This repository enables GitOps-based deployment of the **Snyk Broker** in **code agent mode** using **Helm** and **Argo CD** on OpenShift.

## 📦 Components

- **Helm chart**: Deploys Snyk Broker
- **Argo CD integration**: Automates syncing from this repository
- **values.yaml**: Defines configuration (e.g., broker token, image, env vars)

---

## 🚀 Deployment Instructions

### 1. Pre-Requisites

- An OpenShift cluster with Argo CD installed (e.g., via Red Hat GitOps Operator)
- GitHub repo access configured in Argo CD
- A Snyk Broker token (for your GitHub integration)
- Namespace created (e.g., `snyk`)
- Kubernetes secret created in the namespace:

  ```bash
  oc create ns snyk

  oc create secret generic snyk-broker-secret \
    --from-literal=broker-token=\<your_token_here\> \
    -n snyk
  ```

---

### 2. Repo Structure

```
snyk-broker-gitops/
├── helm-chart/                # Optional if using external Helm chart
│   └── Chart.yaml
│   └── templates/
├── values.yaml                # Overrides for the broker deployment
└── README.md
```

---

### 3. Helm Values

Edit `values.yaml` with your desired config:

```yaml
broker:
  client:
    id: snyk-broker
    token: \{\{ .Values.broker.client.token \}\}
  container:
    image:
      repository: snyk/broker
      tag: latest
    mode: code
  env:
    - name: BROKER_TOKEN
      valueFrom:
        secretKeyRef:
          name: snyk-broker-secret
          key: broker-token
```

> Note: The Helm chart can be referenced directly from the official Snyk repo if preferred.

---

### 4. Argo CD Setup

- Create a new Argo CD Application via the UI or CLI.
- Point it to this repo.
- Specify the path to your Helm chart or `values.yaml`.

#### Example Argo CD App (YAML)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: snyk-broker
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/joeaudio/snyk-broker-gitops
    targetRevision: main
    path: .
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: snyk
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## ✅ Syncing and Updating

- Edit values in `values.yaml`
- Commit changes to GitHub
- Argo CD will auto-sync (if automated sync is enabled), or you can sync manually

---

## 🔐 Secrets Management

This repo does **not** store secrets. Secrets like `BROKER_TOKEN` must be created manually or managed via:
- Sealed Secrets
- External Secrets Operator
- OpenShift Pipelines (optional)

---

## 📎 References

- [Snyk Broker (Code Agent Mode)](https://github.com/snyk/broker#code-agent-mode)
- [Snyk Broker Helm Chart](https://github.com/snyk/broker-helm)
- [Argo CD Docs](https://argo-cd.readthedocs.io/)

