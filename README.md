# CI/CD Pipeline para Oracle Cloud Infrastructure (OKE)

![Oracle Cloud](https://img.shields.io/badge/Oracle%20Cloud-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

> **Exemplo de pipeline CI/CD usando GitHub Actions para deploy em ambiente de DESENVOLVIMENTO no Oracle Kubernetes Engine (OKE)**

## ⚠️ Avisos Importantes

**🔴 Este é um projeto de exemplo educacional para ambiente de DESENVOLVIMENTO**

- ✅ **Ideal para**: Aprendizado, POCs, ambientes de desenvolvimento
- ❌ **NÃO recomendado para**: Ambientes de produção sem adaptações
- 📚 **Implementações para staging/production**: Devem seguir o modelo de governança e compliance da sua organização
- 🔒 **Segurança**: Políticas IAM e configurações de rede devem ser revisadas para cada ambiente

## 📋 Visão Geral

Este repositório demonstra um **pipeline de CI/CD para ambiente de desenvolvimento** que deploya aplicações containerizadas no **Oracle Kubernetes Engine (OKE)** usando **GitHub Actions**.

### 🎯 Escopo do Exemplo

- **Ambiente**: Apenas desenvolvimento (namespace `dev`)
- **Estratégia**: Rolling update simples
- **Foco**: Demonstrar integração GitHub Actions + OCI + OKE
- **Limitações**: Não inclui estratégias avançadas de deploy (Blue/Green, Canary), aprovações manuais ou testes de carga

**Para ambientes de staging e produção**, você deverá:
- Adaptar estratégias de deployment conforme governança da organização
- Implementar aprovações manuais (GitHub Environments)
- Adicionar testes adicionais (performance, segurança, compliance)
- Configurar políticas de rollback e disaster recovery
- Revisar e endurecer configurações de segurança

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

### 6️⃣ Configurar Políticas IAM Adicionais

**⚠️ IMPORTANTE**: Políticas IAM adicionais podem ser necessárias dependendo dos recursos utilizados.

#### A) Políticas para OCI Native Ingress Controller

O Ingress Controller precisa criar e gerenciar Load Balancers:

```hcl
# Dynamic Group para os nodes do OKE
# Regra: ALL {instance.compartment.id = 'ocid1.compartment.oc1..xxxxx'}

# Políticas necessárias:
Allow dynamic-group oke-nodes to manage load-balancers in compartment <compartment-name>
Allow dynamic-group oke-nodes to use virtual-network-family in compartment <compartment-name>
Allow dynamic-group oke-nodes to manage public-ips in compartment <compartment-name>
Allow dynamic-group oke-nodes to manage network-security-groups in compartment <compartment-name>
```

#### B) Políticas para External Secrets Operator (OCI Vault)

Para acessar secrets no OCI Vault usando Instance Principal:

```hcl
# Dynamic Group para os nodes do cluster OKE
# Regra: ALL {instance.compartment.id = 'ocid1.compartment.oc1..xxxxx'}

# Políticas necessárias:
Allow dynamic-group oke-nodes to read secret-family in compartment <compartment-name>
Allow dynamic-group oke-nodes to read vaults in compartment <compartment-name>
Allow dynamic-group oke-nodes to read keys in compartment <compartment-name>

# Ou mais específico para um vault:
Allow dynamic-group oke-nodes to read secret-bundles in compartment <compartment-name> where target.vault.id = 'ocid1.vault.oc1..xxxxx'
```

#### C) Políticas para CI/CD (GitHub Actions)

O usuário usado no GitHub Actions precisa de permissões para:

```hcl
# Para gerenciar cluster OKE
Allow group github-actions-users to use cluster-node-pools in compartment <compartment-name>
Allow group github-actions-users to use clusters in compartment <compartment-name>

# Para push de imagens no OCIR
Allow group github-actions-users to manage repos in tenancy

# Para ler configurações (opcional)
Allow group github-actions-users to read all-resources in compartment <compartment-name>
```

**📚 Documentação Oficial:**
- [OKE IAM Policies](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengpolicyconfig.htm)
- [OCI Vault Policies](https://docs.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults_topic-To_control_who_can_access_vaults_and_keys.htm)
- [Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingdynamicgroups.htm)

## 🔄 Fluxo de Trabalho

### Deploy para Development

**⚠️ Este workflow é simplificado para ambiente de desenvolvimento. Para staging/production, considere:**
- Aprovações manuais (GitHub Environments)
- Estratégias de deploy mais seguras (Blue/Green, Canary)
- Testes de performance e segurança adicionais
- Rollback plans e disaster recovery

```bash
# 1. Criar feature branch
git checkout -b feature/nova-funcionalidade

# 2. Fazer alterações e commit
git add .
git commit -m "feat: adiciona nova funcionalidade"

# 3. Push para develop (ou branch principal)
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
- [OCI IAM Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Helm Documentation](https://helm.sh/docs/)
- [External Secrets Operator](https://external-secrets.io/)

## ⚠️ Considerações para Produção

Se você planeja adaptar este exemplo para ambientes de produção, considere:

### Segurança
- [ ] Implementar Network Policies mais restritivas
- [ ] Habilitar Pod Security Standards/Admission
- [ ] Configurar mTLS entre serviços (Service Mesh)
- [ ] Implementar runtime security (Falco, OPA)
- [ ] Habilitar audit logs no OKE
- [ ] Usar Private Endpoints para API Server

### Observabilidade
- [ ] Configurar Prometheus/Grafana para métricas
- [ ] Implementar distributed tracing (Jaeger, Zipkin)
- [ ] Centralizar logs (ELK, Splunk, OCI Logging)
- [ ] Configurar alertas (PagerDuty, OpsGenie)
- [ ] Implementar SLOs e error budgets

### High Availability
- [ ] Multi-region deployment
- [ ] Disaster Recovery plan
- [ ] Backup automatizado de dados
- [ ] Testes de chaos engineering
- [ ] Pod Disruption Budgets configurados

### CI/CD Avançado
- [ ] Aprovações manuais para produção
- [ ] Estratégias de deploy: Blue/Green, Canary
- [ ] Feature flags para controle granular
- [ ] Testes de carga automatizados
- [ ] Análise de vulnerabilidades no pipeline
- [ ] Assinatura de imagens (Cosign, Notary)

### Governança
- [ ] Policies de compliance (OPA/Gatekeeper)
- [ ] Cost management e resource quotas
- [ ] RBAC granular por equipe/projeto
- [ ] GitOps com ArgoCD/FluxCD
- [ ] Documentação de runbooks

## 🤝 Contribuindo

Contribuições são bem-vindas! Este é um projeto de exemplo educacional.

## 📄 Licença

MIT License - veja [LICENSE](LICENSE) para detalhes.

---

**⚠️ DISCLAIMER**

Este é um **projeto de exemplo para fins educacionais e ambiente de desenvolvimento**. 

**NÃO use diretamente em produção sem:**
- Revisão completa de segurança
- Adequação às políticas de governança da sua organização
- Implementação de controles de compliance necessários
- Testes extensivos em ambientes não-produtivos
- Aprovação das equipes de segurança e infraestrutura

A Oracle, GitHub e os mantenedores deste projeto não se responsabilizam por uso inadequado ou problemas decorrentes da implementação em ambientes produtivos sem as devidas adaptações.
