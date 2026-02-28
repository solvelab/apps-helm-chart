# Apps Helm Chart

Universal Helm Chart template for deploying diverse Kubernetes workloads via ArgoCD. A single chart handles all resource types -- just adjust `values.yaml` per application.

---

## Visao Geral

O **Apps Helm Chart** (`apps-solvelab`) fornece uma estrutura de Helm Chart padronizada e flexivel, projetada para deployar qualquer tipo de workload no Kubernetes via ArgoCD. Em vez de manter charts separados por aplicacao, este template universal cobre todos os resource types comuns e e configurado inteiramente por `values.yaml`.

### Caracteristicas

- **Single chart** para todos os tipos de workload: Deployment, StatefulSet, DaemonSet, CronJob
- **Networking**: Service (ClusterIP, NodePort, Headless) com Endpoints opcionais para servicos externos
- **Ingress**: Exposicao HTTP/HTTPS com suporte a TLS
- **Configuration**: Gerenciamento de ConfigMap e Secret
- **Storage**: Provisionamento de PersistentVolumeClaim
- **Security**: RBAC (Role, RoleBinding, ClusterRole, ClusterRoleBinding) e ServiceAccount
- **HostAliases**: Mapeamento de DNS externo
- **ArgoCD-native**: Configuracao totalmente declarativa via values

---

## Quick Start

### Pre-requisitos

- Helm 3 instalado
- kubectl configurado com acesso ao cluster
- (Opcional) ArgoCD para deployment GitOps

### Renderizacao Local

```bash
# Renderize templates localmente
helm template my-release . -f test/values-deployment.yaml

# Dry-run para validacao
helm install my-release . --dry-run -f test/values-deployment.yaml

# Lint do chart
helm lint .
```

### Deploy via ArgoCD

Aponte seu `Application` do ArgoCD para este repositorio:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: <this-repo-url>
    path: .
    helm:
      valueFiles:
        - my-app-values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: my-namespace
```

---

## Arquitetura

```
+------------------------------------------------------------------+
|                        values.yaml                               |
|  (app, component, deployment, services, ingress, configmap, ...) |
+------+-----------------------------------------------------------+
       |
       v
+------+-----------------------------------------------------------+
|                     Helm Template Engine                          |
|                                                                  |
|  +---------------+  +----------------+  +------------------+     |
|  | deployment    |  | statefulset    |  | daemonset        |     |
|  +---------------+  +----------------+  +------------------+     |
|  +---------------+  +----------------+  +------------------+     |
|  | cronjob       |  | services       |  | ingress          |     |
|  +---------------+  +----------------+  +------------------+     |
|  +---------------+  +----------------+  +------------------+     |
|  | configmap     |  | secret         |  | pvc              |     |
|  +---------------+  +----------------+  +------------------+     |
|  +---------------+  +------------------+                         |
|  | rbac          |  | serviceaccount   |                         |
|  +---------------+  +------------------+                         |
+------------------------------------------------------------------+
       |
       v
+------------------------------------------------------------------+
|              Kubernetes Cluster (via ArgoCD)                      |
+------------------------------------------------------------------+
```

### Templates (11 total)

| Template | Resource Types |
|----------|---------------|
| `deployment.yaml` | Deployment (aplicacoes stateless) |
| `statefulset.yaml` | StatefulSet (apps com estado, databases) |
| `daemonset.yaml` | DaemonSet (agentes por node) |
| `cronjob.yaml` | CronJob (tarefas agendadas) |
| `services.yaml` | Service + Endpoints opcionais |
| `ingress.yaml` | Ingress (exposicao HTTP/HTTPS com TLS) |
| `configmap.yaml` | ConfigMap (configuracao nao-sensivel) |
| `secret.yaml` | Secret (dados sensiveis, base64) |
| `pvc.yaml` | PersistentVolumeClaim (requisicoes de storage) |
| `rbac.yaml` | Role, RoleBinding, ClusterRole, ClusterRoleBinding |
| `serviceaccount.yaml` | ServiceAccount |

---

## Configuracao

Toda configuracao e fornecida via `values.yaml`. O chart usa chaves top-level para metadata e arrays para cada resource type.

### Values Globais

| Key | Tipo | Descricao |
|-----|------|-----------|
| `app` | string | Label do nome da aplicacao |
| `component` | string | Label do componente |
| `core` | string | Label do grupo core (opcional) |
| `version` | string | Versao da aplicacao |
| `portNameMaxLen` | int | Comprimento maximo para nomes de porta |
| `serviceNameMaxLen` | int | Comprimento maximo para nomes de servico |

### Resource Arrays

Cada resource type e configurado como array ou objeto:

```yaml
# Workloads
deployment:          # Objeto com replicas, ports, env, command, resources, affinity
statefulset:         # Similar a deployment com volumeClaimTemplates
daemonset:           # Workload per-node
cronjob:             # Schedule, jobTemplate, concurrencyPolicy

# Networking
services:            # Array de definicoes de Service
  - name: svc-my-app
    type: ClusterIP
    selector: { app: my-app }
    ports: [...]

ingress:             # Array de regras Ingress com TLS

# Configuration
configmap:           # Array de ConfigMaps
secret:              # Array de Secrets

# Storage
pvc:                 # Array de PersistentVolumeClaims

# Security
rbac:                # Role/ClusterRole e bindings
serviceaccount:      # Definicoes de ServiceAccount
```

### Exemplo: Deployment

```yaml
app: my-api
component: backend
version: 1.2.0

deployment:
  replicas: 2
  ports:
    - name: http
      containerPort: 8080
      protocol: TCP
  env:
    - name: APP_ENV
      value: production
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "1.0"

services:
  - name: svc-my-api
    type: ClusterIP
    selector:
      app: my-api
      component: backend
    ports:
      - name: http
        port: 80
        targetPort: 8080
        protocol: TCP
```

---

## Desenvolvimento

### Estrutura do Projeto

```
apps-helm-chart/
├── Chart.yaml              # Metadata (name: apps-solvelab, version: 0.1.0)
├── templates/
│   ├── deployment.yaml     # Deployment workload
│   ├── statefulset.yaml    # StatefulSet workload
│   ├── daemonset.yaml      # DaemonSet workload
│   ├── cronjob.yaml        # CronJob workload
│   ├── services.yaml       # Service + Endpoints
│   ├── ingress.yaml        # Ingress rules
│   ├── configmap.yaml      # ConfigMap
│   ├── secret.yaml         # Secret
│   ├── pvc.yaml            # PersistentVolumeClaim
│   ├── rbac.yaml           # RBAC (Roles + Bindings)
│   └── serviceaccount.yaml # ServiceAccount
├── test/
│   ├── values-deployment.yaml
│   ├── values-services-external.yaml
│   └── README.MD           # Guia de testes locais
└── README.md
```

### Testando Templates

```bash
# Renderizar Deployment
helm template test-release . -f test/values-deployment.yaml

# Renderizar Service com endpoints externos
helm template test-release . -f test/values-services-external.yaml

# Lint do chart
helm lint .

# Validar contra o cluster (dry-run)
helm install test-release . --dry-run --debug -f test/values-deployment.yaml
```

### Boas Praticas

1. **Mantenha simples** -- use apenas os valores necessarios no `values.yaml`
2. **Valide sempre** -- rode `helm template` ou `helm lint` antes de commitar
3. **Headless Services** -- para mapear hosts externos, use Services com `ClusterIP: None`
4. **Secrets** -- use `secretKeyRef` nos env vars do deployment para referenciar Kubernetes Secrets

---

## Stack Tecnologica

| Componente | Tecnologia |
|------------|------------|
| **Chart Engine** | Helm 3 |
| **Orquestracao** | Kubernetes |
| **GitOps** | ArgoCD |
| **Template** | Go templates |
| **Chart Type** | Application |
| **Chart Version** | 0.1.0 |

---

## License

Private project -- All rights reserved.
