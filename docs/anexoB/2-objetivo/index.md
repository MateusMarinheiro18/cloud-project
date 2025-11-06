# 1. Objetivos e Estratégia de DR

## 1.1 Objetivos de Recuperação por Serviço

### 1.1.1 Site Estático (S3 → Azure Blob Storage)

**RTO: 30 minutos**  
**RPO: 24 horas**  
**Criticidade: Média**

**Justificativa do RTO:** O site institucional é principalmente informativo. Uma indisponibilidade de 30 minutos é aceitável, pois não impacta diretamente o processamento de imagens. A recuperação é rápida via sincronização de blobs.

**Justificativa do RPO:** Conteúdo estático muda raramente (páginas institucionais). Sincronização diária é suficiente. Eventual perda de até 24h não compromete a operação.

**Estratégia:** Sincronização automatizada diária via AzCopy, com versionamento habilitado.

---

### 1.1.2 Função de Conversão (Lambda → Azure Functions)

**RTO: 1 hora**  
**RPO: N/A (stateless)**  
**Criticidade: Alta**

**Justificativa do RTO:** A função é crítica para o core business. 1 hora permite deployment validado e testado da Azure Function, sem precipitação que cause erros.

**Justificativa do RPO:** Funções são stateless (código versionado em Git). Não há perda de dados, apenas necessidade de redeployment.

**Estratégia:** Código mantido em repositório Git. Infraestrutura como código (Terraform/ARM) para provisionamento rápido. Azure Function App em modo Consumption pré-configurada (scaled to zero).

---

### 1.1.3 Banco de Dados (DynamoDB → Cosmos DB)

**RTO: 2 horas**  
**RPO: 24 horas**  
**Criticidade: Crítica**

**Justificativa do RTO:** É o componente mais crítico (contém todos os metadados e Base64). 2 horas permitem restauração completa e validação de integridade dos dados.

**Justificativa do RPO:** Backup diário via Export do DynamoDB para S3, com cópia para Azure Blob. Perda de até 24h de uploads é aceitável para um projeto piloto, considerando que o volume é baixo (5-10 uploads/dia).

**Estratégia:** 
- Backups diários automatizados (DynamoDB Export → S3 → Azure Blob)
- Scripts de transformação de dados (DynamoDB JSON → Cosmos DB JSON)
- Cosmos DB pré-provisionado com 400 RU/s (scaled down)

---

### 1.1.4 API/Área de Clientes (EC2/ASG → Azure VM/VMSS)

**RTO: 2 horas**  
**RPO: 24 horas (código)**  
**Criticidade: Alta**

**Justificativa do RTO:** A API é a interface principal dos clientes. 2 horas incluem: provisionamento de VMs, deployment da aplicação, configuração do Load Balancer e testes de fumaça.

**Justificativa do RPO:** Código versionado em Git (sem perda). Configuração de ambiente via variáveis. Eventual necessidade de reconfigurar variáveis de ambiente (conexões, secrets) em caso de drift não versionado.

**Estratégia:** 
- Imagens de VM pré-buildadas (golden image) com aplicação
- VM Scale Set pré-configurado (0 instâncias em standby)
- Scripts de inicialização automatizados

---

### 1.1.5 Balanceador de Carga (ALB → Azure Load Balancer)

**RTO: 1 hora**  
**RPO: N/A**  
**Criticidade: Alta**

**Justificativa do RTO:** Componente de entrada. Configuração via IaC permite provisionamento e validação em ~1 hora, incluindo testes de health checks.

**Estratégia:** Terraform/ARM templates versionados. Azure Load Balancer Standard pré-provisionado com backend pool vazio.

---

### 1.1.6 DNS e Certificados TLS

**RTO: 30 minutos**  
**RPO: N/A**  
**Criticidade: Crítica**

**Justificativa do RTO:** Mudança de DNS (TTL baixo de 300s) + validação de certificado. É o ponto de entrada de todo tráfego.

**Justificativa do RPO:** Registros DNS e certificados são configurações, não dados. Certificados gerenciados (AWS ACM → Azure Key Vault) são emitidos sob demanda.

**Estratégia:** 
- DNS com TTL de 5 minutos (facilita cutover)
- Certificados wildcard pré-emitidos no Azure Key Vault
- Runbook de cutover de DNS testado

---

## 1.2 Cenários de Falha Considerados

### Cenário 1: Indisponibilidade Regional da AWS

**Descrição:** Uma região inteira da AWS (ex: us-east-1) fica indisponível por mais de 4 horas devido a falha catastrófica.

**Probabilidade:** Baixa (histórico: ~1-2x por década)  
**Impacto:** Crítico (100% de indisponibilidade)  
**Decisão de ativação:** Após 4h de downtime confirmado pela AWS ou quando AWS declarar ETA > 8h

**Resposta:**
1. Ativar plano de DR (seção 3)
2. Sincronizar últimos backups disponíveis
3. Provisionar recursos no Azure (scale-up do Warm Standby)
4. Validar integridade dos dados
5. Cutover de DNS para Azure
6. Monitorar e validar operação

---

### Cenário 2: Falha Prolongada de Serviço Específico

**Descrição:** Um serviço específico (ex: Lambda, DynamoDB) apresenta degradação severa (>50% de erro) por mais de 2 horas, sem previsão de resolução.

**Probabilidade:** Média-Baixa  
**Impacto:** Alto (componente crítico indisponível)  
**Decisão de ativação:** Após 2h de degradação severa ou quando AWS declarar impacto prolongado

**Resposta:**
1. Avaliar se migração parcial é viável (ex: apenas Lambda → Azure Functions)
2. Se inviável, ativar DR completo
3. Priorizar restauração do componente crítico
4. Operar temporariamente com funcionalidades degradadas se necessário

---

### Cenário 3: Decisão Estratégica de Saída

**Descrição:** Decisão de negócio para migrar permanentemente para Azure (custos, estratégia multi-cloud, aquisição, etc.)

**Probabilidade:** Média (decisão de negócio)  
**Impacto:** Nenhum (planejado)  
**Decisão de ativação:** Planejada com antecedência mínima de 2 semanas

**Resposta:**
1. Executar migração planejada (blue-green deployment)
2. Sincronizar dados em modo de manutenção (minimizar RPO para < 1h)
3. Validar completamente no Azure antes do cutover
4. Executar cutover gradual (% do tráfego)
5. Monitorar por 48h antes de desprovisionar AWS
6. Manter AWS em standby por 1 semana após cutover

---

### Cenário 4: Exigência Regulatória ou Compliance

**Descrição:** Nova regulamentação exige que dados sejam armazenados em região/provedor específico (ex: LGPD, soberania digital).

**Probabilidade:** Baixa-Média  
**Impacto:** Crítico (bloqueio legal)  
**Decisão de ativação:** Conforme timeline legal (geralmente 30-90 dias)

**Resposta:**
1. Avaliar requirements legais (data residency, certificações)
2. Validar compliance do Azure (Azure Brazil South, ISO 27001, etc.)
3. Executar migração planejada com validação jurídica
4. Documentar chain of custody dos dados
5. Obter atestação de compliance pós-migração

---

## 1.3 Estratégia Warm Standby Detalhada

### Definição

**Warm Standby** mantém uma versão reduzida da infraestrutura executando continuamente na nuvem de destino, com dados sincronizados periodicamente. Em caso de ativação, os recursos são escalados para capacidade de produção.

### Componentes em Warm Standby (Azure)

| Componente Azure | Estado em Standby | Custo/mês | Tempo de Scale-Up |
|------------------|-------------------|-----------|-------------------|
| Blob Storage (site) | Sync ativo, blob público | $5 | 0 min (pronto) |
| Blob Storage (uploads) | Sync ativo, private | $15 | 0 min (pronto) |
| Cosmos DB | 400 RU/s (mínimo) | $25 | 10 min (scale) |
| Function App | Consumption plan | $0 | 5 min (cold start) |
| VM Scale Set | 0 instâncias | $0 | 15 min (provision) |
| Load Balancer | Provisionado | $5 | 5 min (config) |
| Key Vault | Provisionado | $2 | 0 min (pronto) |
| VNet/NSG | Provisionado | $0 | 0 min (pronto) |
| **Total** | | **~$52/mês** | **~30-45 min** |

### Sincronização de Dados

**Site Estático:**
- Frequência: Diária (3 AM UTC)
- Ferramenta: AzCopy via cron job
- Validação: Checksum MD5

**Uploads (Imagens):**
- Frequência: Diária (4 AM UTC)
- Ferramenta: AzCopy incremental
- Validação: Contagem de objetos + amostragem

**Metadados (DynamoDB → Cosmos DB):**
- Frequência: Diária (5 AM UTC)
- Ferramenta: Script Python (DynamoDB Export → transform → Cosmos DB Bulk Import)
- Validação: Contagem de documentos + hash de amostra

### Trade-offs da Estratégia Warm

**Vantagens:**
- RTO aceitável (2-4h) para o perfil do projeto
- Custo controlado (~$52/mês vs $800+ de Active-Active)
- Dados sincronizados (RPO 24h aceitável)
- Infraestrutura pré-validada (reduz risco de ativação)
- Pode ser usada para testes de DR periódicos

**Desvantagens:**
- RPO de 24h (perda de até 1 dia de uploads)
- RTO de 2-4h (janela de indisponibilidade)
- Requer sincronização contínua (overhead operacional)
- Risco de drift entre ambientes (mitigado por IaC)

**Comparação com Alternativas:**

**Cold Standby:** RTO > 6h, RPO > 48h, custo ~$20/mês → **Descartado** (RTO inaceitável)  
**Active-Active:** RTO < 1min, RPO near-zero, custo ~$1000/mês → **Descartado** (custo excessivo para piloto)

---

## 1.4 Matriz de Decisão de Ativação

| Condição | Tempo de Avaliação | Decisão | Responsável |
|----------|-------------------|---------|-------------|
| AWS region down > 4h | 4h | Ativar DR | Tech Lead |
| Serviço crítico degradado > 2h | 2h | Avaliar DR parcial | Tech Lead |
| AWS declara ETA > 8h | Imediato | Ativar DR | CTO |
| Exigência regulatória | Conforme prazo legal | Migração planejada | Legal + CTO |
| Decisão estratégica | Planejado | Migração planejada | C-Level |

**Autoridade Final:** CTO ou designado (Tech Lead em ausência)

---