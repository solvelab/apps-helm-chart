# ☸️ Apps ArgoCD Template

Bem-vindo ao repositório de **Templates Helm Universais**.
Este projeto fornece uma estrutura de Helm Chart padronizada e flexível, projetada para implantar diversos tipos de cargas de trabalho no Kubernetes via ArgoCD, apenas ajustando o arquivo `values.yaml`.

---

## 🚀 Funcionalidades

Este template suporta uma ampla gama de objetos Kubernetes, permitindo a implantação de:

*   **📦 Workloads**:
    *   `Deployment` (Aplicações Stateless)
    *   `StatefulSet` (Aplicações Stateful/Banco de Dados)
    *   `DaemonSet` (Agentes por nó)
    *   `CronJob` (Tarefas agendadas)
*   **🌐 Rede & Acesso**:
    *   `Service` (ClusterIP, NodePort, Headless)
    *   `Ingress` (Exposição HTTP/HTTPS)
    *   `HostAliases` (Mapeamento de DNS externo)
*   **🔐 Configuração & Segurança**:
    *   `ConfigMap` & `Secret`
    *   `ServiceAccount` & `RBAC`
    *   `PersistentVolumeClaim`

---

## 📂 Estrutura do Projeto

```plaintext
.
├── Chart.yaml          # Metadados do Helm Chart
├── templates/          # Arquivos de manifesto YAML (Go templates)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ...
└── test/               # Arquivos de valores para teste e validação
    ├── values-deployment.yaml
    ├── values-services-external.yaml
    └── README.MD       # Guia de testes local
```

---

## 🛠️ Como Usar

### No ArgoCD
Aponte sua `Application` do ArgoCD para este repositório e caminho, e forneça um `helm.values` específico para sua aplicação.

### Testando Localmente
Você pode validar seus arquivos de valores localmente usando o `helm template`.
Consulte o guia detalhado em **[test/README.MD](test/README.MD)**.

```bash
# Exemplo de teste rápido (Dry-run)
helm template my-release . -f test/values-deployment.yaml
```

---

## ✨ Boas Práticas

1.  **Mantenha Simples**: Use apenas os valores necessários no seu `values.yaml`.
2.  **Valide Sempre**: Antes de commitar, teste a renderização do template.
3.  **Headless Services**: Para mapear hosts externos, prefira usar Services `ClusterIP: None` (Headless).

---
*Mantido pela equipe de WorkOps/DevOps.*
