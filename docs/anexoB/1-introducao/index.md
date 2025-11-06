# Plano de Disaster Recovery e Cloud Exit

## Visão Geral Executiva

Este documento apresenta o **Plano Oficial de Disaster Recovery (DR) e Cloud Exit** da plataforma de conversão de imagens desenvolvida pela Demay's Infra Company, atualmente hospedada na Amazon Web Services (AWS).

## Contexto

A plataforma permite que usuários façam upload de imagens, que são automaticamente convertidas para formato Base64 e armazenadas para recuperação posterior. Embora a AWS ofereça alta disponibilidade, este plano garante continuidade do negócio em cenários críticos:

- **Falha regional prolongada** da AWS
- **Decisão estratégica** de mudança de provedor
- **Exigências regulatórias** ou contratuais
- **Otimização de custos** de longo prazo

## Escopo do Plano

Este plano documenta a migração completa da solução AWS para Microsoft Azure, incluindo:

- Todos os componentes da arquitetura (S3, Lambda, DynamoDB, EC2, ALB)
- Dados em repouso (imagens e metadados)
- Configurações de rede, segurança e identidade
- Procedimentos operacionais e testes de validação

### Estratégia Escolhida: Warm Standby

Adotamos a estratégia **Warm Standby** por oferecer o melhor equilíbrio entre custo e tempo de recuperação:

| Critério | Cold Standby | **Warm Standby** | Active-Active |
|----------|--------------|------------------|---------------|
| RTO | 6-24 horas | **2-4 horas** | < 1 minuto |
| RPO | 24-48 horas | **4-24 horas** | Near-zero |
| Custo mensal | $20-30 | **$50-100** | $800-1200 |
| Complexidade | Baixa | **Média** | Alta |

**Justificativa:** Para um projeto piloto com tráfego moderado (50-100 req/h), um RTO de 2-4 horas é aceitável e o custo de $50-100/mês em standby cabe no orçamento, enquanto Active-Active seria super-dimensionado.

### Objetivos de Recuperação

| Componente | RTO | RPO | Criticidade |
|------------|-----|-----|-------------|
| Site Estático (S3) | 30 min | 24h | Média |
| API/Área Clientes (EC2) | 2h | 24h | Alta |
| Conversão (Lambda) | 1h | N/A | Alta |
| Dados (DynamoDB) | 2h | 24h | Crítica |
| Balanceador (ALB) | 1h | N/A | Alta |
| DNS/Certificados | 30 min | N/A | Crítica |

### Estrutura do Documento

1. **Objetivos e Estratégia** - RTO/RPO detalhados e cenários de falha
2. **Mapeamento de Serviços** - Equivalências AWS ↔ Azure
3. **Procedimentos e Runbooks** - Passo a passo de ativação
4. **Rede, Segurança e Identidade** - Configurações técnicas
5. **Custos** - Estimativas financeiras e operacionais
6. **Validação e Testes** - Plano de testes de DR
7. **Riscos e Compliance** - Análise de riscos e LGPD
8. **Cronograma e Responsáveis** - Timeline e matriz RACI

---