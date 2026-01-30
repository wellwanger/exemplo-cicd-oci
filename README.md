# CI/CD Pipeline para Oracle Cloud Infrastructure (OKE)

![Oracle Cloud](https://img.shields.io/badge/Oracle%20Cloud-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

> **Exemplo completo de pipeline CI/CD usando GitHub Actions para deploy automatizado no Oracle Kubernetes Engine (OKE)**

## 📋 Visão Geral

Este repositório demonstra um pipeline completo de CI/CD para deployar aplicações containerizadas no **Oracle Kubernetes Engine (OKE)** usando **GitHub Actions**, seguindo as melhores práticas de DevOps e segurança.

### 🎯 Componentes Oracle Cloud Infrastructure (OCI)

| Serviço | Finalidade | Documentação |
|---------|-----------|--------------|
| **OKE (Oracle Kubernetes Engine)** | Cluster Kubernetes gerenciado | [Docs OKE](https://docs.oracle.com/en-us/iaas/Content/ContEng/home.htm) |
| **OCIR (Oracle Container Image Registry)** | Registry privado de imagens Docker | [Docs OCIR](https://docs.oracle.com/en-us/iaas/Content/Registry/home.htm) |
| **OCI Vault** | Gerenciamento de secrets e chaves | [Docs Vault](https://docs.oracle.com/en-us/iaas/Content/KeyManagement/home.htm) |
| **OCI Native Ingress Controller** | Load Balancer Layer 7 nativo | [Docs Ingress](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengsettingupingresscontroller.htm) |
| **VCN (Virtual Cloud Network)** | Rede virtual isolada | [Docs VCN](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm) |
| **IAM (Identity and Access Management)** | Controle de acesso e políticas | [Docs IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm) |

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                         GitHub Actions                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │  Build Image │───▶│  Push OCIR   │───▶│ Deploy OKE   │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
└────────────┬────────────────────┬─────────────────────┬─────────┘
             │                    │                     │
             ▼                    ▼                     ▼
    ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
    │  Docker Build  │   │      OCIR      │   │      OKE       │
    │   Multi-stage  │   │  Private Repo  │   │   Kubernetes   │
    └────────────────┘   └────────────────┘   └────────────────┘
                                                       │
                         ┌─────────────────────────────┼──────────┐
                         │                             │          │
                         ▼                             ▼          ▼
                 ┌───────────────┐           ┌──────────────┐  ┌─────────┐
                 │   OCI Vault   │           │ OCI Ingress  │  │   Pods  │
                 │    Secrets    │           │ Load Balancer│  │  Apps   │
                 └───────────────┘           └──────────────┘  └─────────┘
```

## 🚀 Funcionalidades

### Pipeline CI/CD
- ✅ Build automatizado de imagens Docker
- ✅ Push para Oracle Container Image Registry (OCIR)
- ✅ Deploy automatizado no OKE
- ✅ Gestão de secrets via OCI Vault + External Secrets Operator
- ✅ Health checks e smoke tests
- ✅ Rollback automático em caso de falha

### Segurança
- 🔒 Autenticação via OCI credentials
- 🔒 Secrets gerenciados no OCI Vault
- 🔒 Imagens privadas no OCIR
- 🔒 Network policies no Kubernetes
- 🔒 RBAC e service accounts

### Helm Charts
- 📦 Templates Kubernetes reutilizáveis
- 📦 Configurações por ambiente (dev, staging, prod)
- 📦 HPA (Horizontal Pod Autoscaler)
- 📦 Ingress com OCI Native Controller

## 📋 Pré-requisitos

### 1. Conta Oracle Cloud Infrastructure (OCI)
- Conta OCI ativa com créditos disponíveis
- Compartimento criado
- Usuário com permissões de administrador

### 2. Cluster OKE Configurado
```bash
# Criar cluster via Console ou Terraform
# Deve ter:
- Kubernetes version: 1.29+
- Node pool com pelo menos 2 nodes
- Load Balancer configurado
```

### 3. Ferramentas Locais
```bash
# OCI CLI
curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh | bash

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Helm 3
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## ⚙️ Setup

### 1️⃣ Configurar OCI Credentials

Obtenha as credenciais no Console OCI:

```bash
# Tenancy OCID
OCI Console → Governance → Tenancy Details → OCID

# User OCID
OCI Console → Identity → Users → Your User → OCID

# API Key
OCI Console → Identity → Users → Your User → API Keys → Add API Key
# Download private key e copie fingerprint
```

### 2️⃣ Configurar GitHub Secrets

Acesse: `Settings → Secrets and variables → Actions → Secrets`

**Secrets Obrigatórios:**
```
OCI_TENANCY_OCID       = ocid1.tenancy.oc1..xxxxx
OCI_USER_OCID          = ocid1.user.oc1..xxxxx
OCI_FINGERPRINT        = aa:bb:cc:dd:ee:ff:11:22:33:44
OCI_PRIVATE_KEY        = -----BEGIN PRIVATE KEY----- ...
OCIR_USERNAME          = <namespace>/<username>
OCIR_TOKEN             = <auth-token>
REGISTRY_NAMESPACE     = <namespace>
OKE_CLUSTER_ID         = ocid1.cluster.oc1.xxxxx
```

**Variables (não sensíveis):**
```
OCI_REGION             = sa-saopaulo-1  (ou us-ashburn-1)
OCI_REGISTRY           = sa-saopaulo-1.ocir.io
IMAGE_NAME             = myapp
```

### 3️⃣ Criar OCIR Repository

```bash
# Via Console
OCI Console → Developer Services → Container Registry → Create Repository

# Repository Name: myapp
# Access: Private
```

### 4️⃣ Configurar kubectl Local

```bash
# Obter kubeconfig
oci ce cluster create-kubeconfig \
  --cluster-id <cluster-ocid> \
  --file ~/.kube/config \
  --region <region>

# Testar conexão
kubectl get nodes
```

### 5️⃣ (Opcional) Instalar External Secrets Operator

Para usar OCI Vault para gerenciar secrets:

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  -n external-secrets-system --create-namespace
```

## 🔄 Fluxo de Trabalho

### Deploy para Development

```bash
# 1. Criar feature branch
git checkout -b feature/nova-funcionalidade

# 2. Fazer alterações e commit
git add .
git commit -m "feat: adiciona nova funcionalidade"

# 3. Push para develop
git checkout develop
git merge feature/nova-funcionalidade
git push origin develop

# 4. GitHub Actions executa automaticamente:
#    - Build da imagem Docker
#    - Push para OCIR
#    - Deploy no OKE namespace 'dev'
```

### Monitorar Deploy

```bash
# Via GitHub
Actions → CD - Deploy to Development → Ver logs

# Via kubectl
kubectl get pods -n dev
kubectl logs -f deployment/myapp -n dev
kubectl get ingress -n dev
```

## 📁 Estrutura do Projeto

```
myapp/
├── .github/
│   └── workflows/
│       └── cd-dev.yml              # Pipeline de deploy development
├── helm/
│   └── myapp/
│       ├── Chart.yaml              # Metadados do Helm chart
│       ├── values.yaml             # Valores padrão
│       ├── values-dev.yaml         # Valores para dev
│       └── templates/
│           ├── deployment.yaml     # Deployment Kubernetes
│           ├── service.yaml        # Service
│           ├── ingress.yaml        # Ingress (Load Balancer)
│           ├── hpa.yaml            # Horizontal Pod Autoscaler
│           ├── configmap.yaml      # ConfigMap
│           ├── external-secret.yaml # External Secrets (OCI Vault)
│           └── _helpers.tpl        # Funções auxiliares
├── LICENSE
└── README.md
```

## 🛠️ Comandos Úteis

### Verificar Deployment
```bash
kubectl get all -n dev
kubectl describe deployment myapp -n dev
kubectl logs -f -l app.kubernetes.io/name=myapp -n dev
```

### Escalar Aplicação
```bash
kubectl scale deployment myapp --replicas=5 -n dev
```

### Acessar Aplicação
```bash
# Obter URL do Ingress
kubectl get ingress -n dev

# Port-forward local
kubectl port-forward svc/myapp 8080:80 -n dev
curl http://localhost:8080/health
```

### Rollback
```bash
kubectl rollout undo deployment/myapp -n dev
kubectl rollout status deployment/myapp -n dev
```

## 🔍 Troubleshooting

### Pod não inicia
```bash
kubectl describe pod <pod-name> -n dev
kubectl logs <pod-name> -n dev
```

### Erro de autenticação OCIR
```bash
# Verificar secret
kubectl get secret ocir-secret -n dev -o yaml

# Recriar secret
kubectl delete secret ocir-secret -n dev
kubectl create secret docker-registry ocir-secret \
  --docker-server=<region>.ocir.io \
  --docker-username=<namespace>/<username> \
  --docker-password=<auth-token> \
  --namespace=dev
```

### External Secrets não sincroniza
```bash
# Verificar operator
kubectl get pods -n external-secrets-system

# Verificar ExternalSecret
kubectl describe externalsecret myapp-vault-secrets -n dev

# Ver logs do operator
kubectl logs -n external-secrets-system -l app.kubernetes.io/name=external-secrets
```

## 📚 Recursos Adicionais

- [OCI Documentation](https://docs.oracle.com/en-us/iaas/Content/home.htm)
- [OKE Best Practices](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengbestpracticesoverview.htm)
- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Helm Documentation](https://helm.sh/docs/)
- [External Secrets Operator](https://external-secrets.io/)

## 🤝 Contribuindo

Contribuições são bem-vindas! Este é um projeto de exemplo educacional.

## 📄 Licença

MIT License - veja [LICENSE](LICENSE) para detalhes.

---

**⚠️ Aviso:** Este é um projeto de exemplo para fins educacionais. Adapte as configurações de segurança e recursos conforme necessário para ambientes de produção.
