# 2. Mapeamento de Serviços AWS → Azure

## 2.1 Tabela de Equivalências

| AWS Service | Azure Equivalent | Grau de Portabilidade | Observações Críticas |
|-------------|------------------|----------------------|---------------------|
| **S3 (site estático)** | **Blob Storage + Static Website** | Alta | Suporta hosting de site estático. URLs mudam (`.blob.core.windows.net`). CORS configurado em conta. |
| **S3 (uploads)** | **Blob Storage** | Alta | SAS tokens equivalem a pre-signed URLs. Mesma hierarquia de prefixos. Versionamento suportado. |
| **Lambda** | **Azure Functions** | Média | Runtimes similares (Python/Node). Event triggers diferentes (Event Grid vs S3 Events). Requer reescrita de código de setup. |
| **DynamoDB** | **Cosmos DB (NoSQL API)** | Média-Baixa | Schema-less compatível. Partition key equivale a PK. Índices secundários diferentes (GSI → Composite Index). Migração de dados requer transformação. |
| **EC2 (instâncias)** | **Virtual Machines** | Alta | Sizing similar (2 vCPU/4GB equivale a B2s). User-data → Custom Script Extension. Imagens Linux idênticas. |
| **Auto Scaling Group** | **VM Scale Set** | Alta | Políticas de scaling similares (CPU-based, schedule). Health probes equivalentes. |
| **Application Load Balancer** | **Azure Load Balancer (Standard)** | Alta | Layer 7 LB. Path-based routing suportado. Health checks equivalentes. Session affinity configurável. |
| **VPC** | **Virtual Network (VNet)** | Alta | CIDR, subnets, route tables equivalentes. |
| **Security Groups** | **Network Security Groups (NSG)** | Alta | Regras de inbound/outbound idênticas. Stateful. |
| **IAM Roles/Policies** | **Managed Identities + RBAC** | Média | Conceito similar (identidade para recursos). Sintaxe de políticas diferente (IAM JSON vs Azure RBAC). Requer remapeamento completo. |
| **ACM (certificados)** | **Key Vault + App Gateway** | Alta | Certificados TLS gerenciados. Renovação automática (Let's Encrypt via App Gateway). |
| **Route 53** | **Azure DNS** | Alta | Zonas DNS, A/CNAME records idênticos. TTL configurável. |
| **CloudWatch Logs** | **Azure Monitor + Log Analytics** | Alta | Logs centralizados. Query language diferente (CloudWatch Insights vs KQL). Dashboards equivalentes. |
| **CloudWatch Metrics** | **Azure Monitor Metrics** | Alta | Métricas de recursos. Alertas configuráveis. |

**Legenda de Portabilidade:**
- Alta: Migração direta com configuração mínima
- Média: Requer reescrita/adaptação de código/config
- Baixa: Mudança arquitetural significativa

---

## 2.2 Detalhamento por Componente

### 2.2.1 Site Estático: S3 → Blob Storage

**AWS:**
```
S3 Bucket: demay-site-estatico
- Static Website Hosting enabled
- Index: index.html
- Error: 404.html
- Public access (website endpoint)
```

**Azure Equivalente:**
```
Storage Account: demaysiteestatico
- Static Website enabled
- Index: index.html
- Error: 404.html
- Public blob access
- URL: https://demaysiteestatico.z13.web.core.windows.net/
```

**Mudanças Necessárias:**
- Atualizar URLs de assets (se hardcoded)
- Reconfigurar CORS (definido na Storage Account, não no container)
- Atualizar DNS para apontar ao endpoint do Blob (ou usar CDN)

**Riscos:**
- URLs diferentes podem quebrar links externos
- Comportamento de cache pode diferir levemente

**Mitigação:**
- Usar Azure CDN para URLs customizadas
- Configurar redirects 301 se necessário
- Testar todos os assets antes do cutover

---

**Mudanças Necessárias:**
- Substituir SDK boto3 por azure-storage-blob
- Adaptar lógica de geração de URL pré-assinada → SAS token
- Atualizar variáveis de ambiente (account name, keys)

**Compatibilidade de API:**
- HTTP PUT para upload (idêntico)
- Content-Type headers (idêntico)
- Timeout/expiração (equivalente)

**Riscos:**
- Formato de URL diferente (pode quebrar parsers de URL)
- Permissões de SAS mais granulares (ajustar testes)

---

### 2.2.3 Conversão: Lambda → Azure Functions

**Riscos:**
- Query patterns diferentes (DynamoDB Query vs Cosmos SQL API)
- Custos imprevisíveis se RU/s mal dimensionado
- Limite de documento de 2MB (DynamoDB: 400KB) - compatível com chunks

**Mitigação:**
- Testar queries em ambiente de staging antes do cutover
- Monitorar Request Units (RU/s) e ajustar conforme necessário
- Manter estratégia de chunking (já resolve limite de tamanho)

---

### 2.2.5 API/Aplicação: EC2 → Azure VMs

**AWS:**
```
Instance Type: t3.small (2 vCPU, 2GB RAM)
Auto Scaling Group: 2-4 instâncias
AMI: Ubuntu 22.04 + Node.js app
User Data: git clone + npm install + pm2 start
```

**Azure:**
```
VM Size: B2s (2 vCPU, 4GB RAM)
VM Scale Set: 2-4 instâncias
Image: Ubuntu 22.04 LTS
Custom Script Extension: git clone + npm install + pm2 start
```

**Mudanças na Aplicação:**
- Variáveis de ambiente (AWS → Azure credentials)
- SDK calls (boto3 → azure-storage-blob, azure-cosmos)
- Healthcheck endpoint (manter GET /health)
- Logs (stdout → Azure Monitor se configurado)

**Riscos:**
- Preço ligeiramente diferente (B2s ~$30/mês vs t3.small ~$15/mês)
- Tempo de boot pode variar (testar cold start)
- Managed Identity setup diferente de IAM Role

**Mitigação:**
- Criar golden image com app pré-instalada (reduz boot time)
- Configurar System-Assigned Managed Identity com acesso a Blob + Cosmos
- Testar scale-out/scale-in antes do go-live

---

### 2.2.6 Balanceamento: ALB → Azure Load Balancer

**AWS ALB:**
```
Type: Application Load Balancer (Layer 7)
Listeners: HTTP (80) → redirect 443, HTTPS (443)
Target Group: api-tg (health check: GET /health)
Rules: /* → target group
```

**Azure Equivalente:**
```
Type: Azure Load Balancer Standard (Layer 4) 
      + Application Gateway (Layer 7 - se precisar path routing)
Backend Pool: api-backend (VMs do VMSS)
Health Probe: HTTP /health (interval: 15s)
Load Balancing Rules: Port 443 → Backend Pool
```

**Nota:** Para funcionalidade completa de Layer 7 (path-based routing, SSL offload), usar **Application Gateway** em vez de Load Balancer Standard.

**Application Gateway (recomendado):**
```
SKU: Standard_v2
Listeners: HTTP (80) redirect, HTTPS (443)
Backend Pool: api-backend
HTTP Settings: protocol HTTP, port 8080, path /health
Rules: /* → backend pool
SSL Certificate: Key Vault integration
```

**Mudanças:**
- Health check configurado no backend HTTP settings
- Certificado TLS migrado para Key Vault
- URLs permanecem idênticas após DNS cutover

**Riscos:**
- Application Gateway mais caro (~$125/mês vs ALB ~$20/mês)
- Load Balancer Standard não faz SSL offload (necessita App Gateway)

**Mitigação:**
- Usar Application Gateway para paridade completa com ALB
- Configurar auto-scaling do App Gateway (v2 suporta)

---

### 2.2.7 Rede: VPC → VNet

**AWS VPC:**
```
CIDR: 10.0.0.0/16
Subnets:
  - Public 1: 10.0.1.0/24 (us-east-1a)
  - Public 2: 10.0.2.0/24 (us-east-1b)
  - Private 1: 10.0.11.0/24 (us-east-1a)
  - Private 2: 10.0.12.0/24 (us-east-1b)
Route Tables:
  - Public: 0.0.0.0/0 → IGW
  - Private: 0.0.0.0/0 → NAT Gateway
```

**Azure VNet:**
```
Address Space: 10.0.0.0/16
Subnets:
  - Public: 10.0.1.0/24
  - Private: 10.0.11.0/24
  - AppGateway: 10.0.3.0/24 (required for App Gateway)
Route Tables:
  - Public: 0.0.0.0/0 → Internet (default)
  - Private: 0.0.0.0/0 → NAT Gateway (optional)
```

**Diferenças:**
- Azure não tem conceito de "Availability Zones" em subnets (VMs escolhem AZ)
- Application Gateway requer subnet dedicada
- Route tables equivalentes

**Mapeamento de CIDR (manter compatível):**
- Usar mesmo CIDR 10.0.0.0/16 para facilitar VPN/peering futuro

---

### 2.2.8 Segurança: Security Groups → NSG

**AWS Security Group (EC2):**
```
Inbound:
  - TCP 8080 from ALB Security Group
  - TCP 22 from Bastion (maintenance)
Outbound:
  - All traffic (default)
```

**Azure NSG (VMSS):**
```
Inbound:
  - TCP 8080 from Application Gateway subnet (10.0.3.0/24)
  - TCP 22 from Bastion subnet (10.0.4.0/24)
  - Deny all others
Outbound:
  - HTTPS 443 to Internet (Blob, Cosmos APIs)
  - Deny all others (principle of least privilege)
```

**Mudanças:**
- Referenciar por subnet CIDR em vez de Security Group ID
- NSG aplicado a subnet ou NIC (escolher subnet para simplicidade)

**Compatibilidade:**
- Lógica de firewall idêntica (stateful)
- Allow/Deny rules equivalentes

---

### 2.2.9 Identidade: IAM → Managed Identity + RBAC


**Mudanças:**
- **Remapear todas as políticas IAM para RBAC**
- Usar Managed Identity (sem keys no código)
- Configurar role assignments via portal ou ARM template

**Riscos:**
- Sintaxe totalmente diferente (requer análise manual)
- Azure RBAC tem menos granularidade (ações predefinidas)

**Mitigação:**
- Documentar mapeamento IAM → RBAC em planilha
- Testar permissões em ambiente de dev antes do DR

---

### 2.2.10 Certificados: ACM → Key Vault

**AWS ACM:**
```
Certificate: *.demay.com (wildcard)
Validation: DNS (Route 53)
Auto-renewal: Gerenciado pela AWS
```

**Azure Key Vault + App Gateway:**
```
Key Vault: demay-keyvault
Certificate: wildcard-demay-com (importado ou Let's Encrypt)
App Gateway: SSL certificate from Key Vault
Auto-renewal: Configurar via Key Vault + App Gateway integration
```

**Processo de Migração:**
1. Exportar certificado privado do ACM (ou re-emitir)
2. Importar para Azure Key Vault
3. Conceder acesso ao App Gateway (Managed Identity)
4. Configurar HTTPS listener no App Gateway

**Riscos:**
- Downtime durante troca de certificado (se não planejar)
- Certificado Let's Encrypt tem renewal manual no Azure (sem ACM automation)

**Mitigação:**
- Emitir certificado wildcard com validade longa (1 ano)
- Configurar alerta de expiração no Key Vault
- Usar Azure App Service Managed Certificate se aplicável (grátis, auto-renewal)

---

## 2.3 Comparação de Custos (Warm Standby)

| Componente | AWS (Atual) | Azure (Standby) | Azure (Produção) |
|------------|-------------|-----------------|------------------|
| Storage (site) | S3: $2/mês | Blob: $2/mês | Blob: $2/mês |
| Storage (uploads) | S3: $12/mês | Blob: $15/mês | Blob: $15/mês |
| Database | DynamoDB On-Demand: $8/mês | Cosmos 400 RU: $25/mês | Cosmos 4000 RU: $240/mês |
| Functions | Lambda: $3/mês | Functions: $0 (consumption) | Functions: $5/mês |
| Compute | EC2 2x t3.small: $30/mês | VMSS 0 instances: $0 | VMSS 2x B2s: $60/mês |
| Load Balancer | ALB: $20/mês | App Gateway: $5 (minimal) | App Gateway: $125/mês |
| Networking | Data transfer: $5/mês | VNet: $0 | Data transfer: $8/mês |
| **Total** | **$80/mês** | **$52/mês** | **$455/mês** |

**Observações:**
- Azure Standby é 65% do custo AWS atual
- Azure Produção seria 5.7x mais caro (principalmente App Gateway + Cosmos)
- Otimização possível: usar Load Balancer Standard ($18/mês) em vez de App Gateway se não precisar Layer 7

---

## 2.4 Matriz de Riscos de Portabilidade

| Componente | Risco | Impacto | Probabilidade | Mitigação |
|------------|-------|---------|---------------|-----------|
| Lambda → Functions | Cold start mais lento | Médio | Alta | Testar performance; considerar Premium plan |
| DynamoDB → Cosmos | Query patterns incompatíveis | Alto | Média | Validar queries em staging; ajustar índices |
| IAM → RBAC | Permissões mal configuradas | Alto | Média | Checklist de permissões; testes de segurança |
| S3 URLs → Blob URLs | Links quebrados | Baixo | Baixa | Usar CDN com domínio customizado |
| ACM → Key Vault | Expiração de certificado | Médio | Baixa | Alertas de expiração; documentar renewal |
| Logs CloudWatch → Monitor | Perda de visibilidade | Médio | Alta | Configurar Application Insights imediatamente |

---