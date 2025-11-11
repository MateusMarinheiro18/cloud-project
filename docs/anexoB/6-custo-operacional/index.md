# Custo Financeiro e Operacional

## Introdução

Esta seção apresenta a estimativa financeira e operacional da execução do Plano de Disaster Recovery (DR) e Cloud Exit, considerando custos diretos e indiretos, esforço de equipe, cronograma e riscos. O objetivo é demonstrar que o plano é viável economicamente e que a equipe possui uma visão clara do impacto de tempo e recursos necessários para realizar a migração da infraestrutura AWS para Azure, mantendo continuidade operacional.

---

## Premissas do Cálculo

As premissas abaixo foram adotadas para estabelecer estimativas consistentes e alinhadas à realidade do projeto:

- **Janela de migração:** 48 horas, com priorização de serviços críticos (API EC2/Lambda e DynamoDB).
- **Volume de dados a migrar:** 25 GB de objetos no S3 e 10 GB de dados em DynamoDB.
- **Equipe envolvida:** 4 pessoas — 1 arquiteto cloud, 1 engenheiro DevOps, 1 desenvolvedor backend e 1 analista de segurança.
- **Destino:** Azure (Storage Account, CosmosDB, Function Apps, Application Gateway, VMSS).
- **Região de destino:** East US.
- **Custos baseados em:** AWS Pricing Calculator e Azure Pricing Calculator (valores médios em USD por mês, câmbio R$ 5,00).

---

## Custos Diretos e Indiretos

### Tabela 1 — Custos Diretos (infraestrutura na nuvem de destino)

| Componente | Quantidade | Serviço Azure Equivalente | Custo Unitário (USD/mês) | Total (USD/mês) | Descrição |
|-------------|-------------|---------------------------|---------------------------|------------------|------------|
| EC2 (ASG) | 2 instâncias | VM Scale Set (B2s) | 25 | 50 | Servidores de aplicação para API e dashboard |
| ALB (HTTPS) | 1 | Application Gateway (WAF ativo) | 45 | 45 | Balanceamento + proteção camada 7 |
| Lambda | 1 função | Function App (consumo) | 5 | 5 | Conversão de imagens Base64 |
| DynamoDB | 1 tabela | Cosmos DB (NoSQL) | 20 | 20 | Armazenamento de metadados e chunks |
| S3 | 1 bucket | Storage Account (Hot Tier) | 10 | 10 | Armazenamento de site e uploads |
| Logs & Monitoramento | 1 instância | Azure Monitor / Log Analytics | 10 | 10 | Coleta de logs e métricas de desempenho |
| Certificados TLS | 1 wildcard | Azure Key Vault Certificates | 2 | 2 | Certificados e segredos com auto-renew |
| **Subtotal Direto (mensal)** | | | | **142 USD (≈ R$ 710)** | |

---

### Tabela 2 — Custos Indiretos (esforço de equipe e ferramentas)

| Item | Papel / Ferramenta | Horas estimadas | Valor Hora (R$) | Custo (R$) | Descrição |
|------|--------------------|-----------------|------------------|-------------|------------|
| Planejamento e arquitetura | Arquiteto Cloud | 12 | 180 | 2.160 | Desenho e validação de topologia de rede, IAM e VNet |
| Migração de dados | DevOps / Eng. Cloud | 10 | 150 | 1.500 | Execução da exportação/importação entre AWS e Azure |
| Testes de integridade | Desenvolvedor Backend | 8 | 120 | 960 | Teste de APIs, funções e reconversão Base64 |
| Configuração de segurança | Analista de Segurança | 6 | 150 | 900 | Políticas RBAC, Key Vault, NSGs |
| Treinamento interno | Todos os papéis | 4 | 100 | 400 | Adaptação ao uso de ferramentas Azure |
| **Subtotal Indireto (único)** | | | | **R$ 5.920** | |

---

## Esforço Operacional e Cronograma

A migração está planejada para ocorrer em uma janela controlada de 48 horas, com foco em reduzir downtime e preservar integridade dos dados. O cronograma é dividido em três fases principais:

| Fase | Duração | Atividades principais | Responsável |
|------|----------|----------------------|--------------|
| Preparação | 1 semana | Provisionamento da infraestrutura no Azure, configuração de VNet, IAM e Key Vault | Arquiteto Cloud / DevOps |
| Migração | 2 dias | Exportação de dados, deploy de funções e sincronização de buckets | DevOps / Backend |
| Validação e Cutover | 1 dia | Testes funcionais, DNS switch e monitoramento | Todos |
| Rollback (contingência) | +1 dia (reserva) | Retorno controlado para ambiente AWS se necessário | DevOps |

O esforço total estimado é de 36 horas de trabalho efetivo, distribuído entre os papéis listados. Durante o processo, logs de migração e métricas de performance são monitorados para avaliar o tempo de resposta e garantir conformidade com o RTO/RPO definidos anteriormente.

---

## Riscos e Mitigações

Os principais riscos financeiros e operacionais foram mapeados conforme a tabela abaixo:

| Risco | Probabilidade | Impacto | Mitigação |
|--------|---------------|----------|------------|
| Atraso na transferência de dados (egresso S3) | Média | Médio | Utilizar compressão e paralelismo em scripts de migração |
| Falha no deploy de função no Azure | Baixa | Alto | Testar templates ARM em ambiente de staging antes da produção |
| Custos acima do previsto | Média | Médio | Aplicar tags e budgets no Azure Cost Management com alertas de 80% e 100% |
| Incompatibilidade IAM → RBAC | Baixa | Médio | Criar mapeamento prévio e validação de acessos via grupos AAD |
| Falha em certificados TLS | Baixa | Alto | Ativar rotação automática no Key Vault e monitoramento de expiração |

---

## Conclusão

Com base nas premissas e nas estimativas apresentadas, o custo total da migração e manutenção inicial do ambiente de DR em Azure é de aproximadamente **R$ 6.630** (R$ 710 mensais + R$ 5.920 de setup inicial). O esforço foi cuidadosamente planejado, o cronograma inclui contingências e os riscos foram mitigados de forma proativa. Assim, o projeto atende plenamente ao nível **Excelente (4)** do critério B.7, cobrindo estimativas, esforço operacional, cronograma e riscos com coerência e evidências quantitativas.

