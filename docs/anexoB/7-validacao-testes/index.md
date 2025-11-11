# B.8 — Validação e Testes de DR

## Introdução

Esta seção apresenta o plano de testes do cenário de Disaster Recovery (DR) simulado, com o objetivo de comprovar a efetividade da estratégia de migração entre nuvens e garantir que todos os serviços essenciais da aplicação sejam restaurados de forma íntegra e funcional no ambiente de destino (Azure). O foco está em validar o fluxo completo — do upload até a recuperação da imagem convertida em Base64 —, assegurando que o RTO e o RPO definidos anteriormente sejam atendidos.

---

## Escopo e Objetivo dos Testes

Os testes de DR abrangem todo o ciclo operacional da solução migrada, incluindo armazenamento, funções serverless, APIs e banco de dados. O objetivo é garantir que a aplicação no ambiente Azure execute corretamente as operações críticas após um evento de migração ou falha simulada na AWS.

- **Serviços testados:** Storage Account (upload), Function App (conversão Base64), CosmosDB (armazenamento), e API hospedada no VMSS (listagem e leitura).  
- **Cenário de referência:** migração da AWS para Azure com failover de DNS para o Application Gateway.  
- **Tipo de teste:** simulação de DR planejado (cutover controlado).  

---

## Casos de Teste

### Caso 1 — Upload de Imagem
**Objetivo:** Validar a capacidade de upload e armazenamento em Storage Account via URL pré-assinada.  
**Procedimento:**
1. Gerar URL de upload pelo endpoint da aplicação no Azure.
2. Realizar upload de arquivo de teste (ex.: `dr-test.png`).
3. Confirmar armazenamento no container `uploads/` do Storage Account.  
**Critério de sucesso:** HTTP 200 retornado; objeto visível no portal do Azure.

### Caso 2 — Conversão e Gravação no Banco (Function App)
**Objetivo:** Verificar se a função é acionada automaticamente após o evento de upload.  
**Procedimento:**
1. Confirmar trigger `BlobCreated` no Azure Function Logs.
2. Validar que a imagem foi convertida em Base64 e registrada no CosmosDB.  
**Critério de sucesso:** item salvo em CosmosDB com campo `base64_chunk_000` presente.

### Caso 3 — Recuperação via API (VMSS)
**Objetivo:** Testar a capacidade da API em recuperar e recompor a imagem convertida.  
**Procedimento:**
1. Executar requisição GET `/api/images/{id}` no endpoint público da API.
2. Validar resposta JSON contendo string Base64 e metadados.  
**Critério de sucesso:** resposta HTTP 200, latência < 800 ms, integridade Base64 confirmada.

### Caso 4 — DNS Cutover e Acesso Público
**Objetivo:** Confirmar que o redirecionamento DNS pós-falha aponta corretamente para o Application Gateway no Azure.  
**Procedimento:**
1. Atualizar registro CNAME no Route53 apontando para o FQDN do Application Gateway.
2. Testar acesso ao domínio original e verificar resolução via `nslookup`.  
**Critério de sucesso:** resolução concluída; resposta 200 na rota `/health` do ambiente Azure.

### Caso 5 — Teste de Reversão (Rollback)
**Objetivo:** Validar que o rollback para a AWS é possível em caso de falha na ativação do DR.  
**Procedimento:**
1. Reverter DNS para o ALB original da AWS.
2. Verificar restabelecimento dos serviços na origem.  
**Critério de sucesso:** resposta 200 do ambiente AWS dentro de 10 minutos após rollback.

---

## Critérios de Sucesso e Validação

Os critérios gerais de aceitação para o plano de testes são:
- **Disponibilidade:** todos os endpoints principais respondem com código HTTP 200 após o failover.  
- **Integridade:** amostragem de 5 imagens convertidas retorna strings Base64 idênticas às originais (checksum validado).  
- **Desempenho:** latência média inferior a 1 segundo no endpoint `/api/images/{id}`.  
- **DNS Cutover:** propagação concluída em menos de 15 minutos.  
- **Monitoramento:** métricas do Application Gateway e logs do Function App registram transações com sucesso > 95%.

---

## Evidências Coletadas

Durante a execução dos testes simulados, foram registradas as seguintes evidências:
- **Captura de logs do Function App** mostrando triggers de upload e conversão concluída.  
- **Prints do CosmosDB** com itens criados após o evento.  
- **Relatórios do Application Gateway** com métricas de latência e taxa de sucesso.  
- **Consulta DNS (`nslookup`)** confirmando o apontamento para o novo FQDN.  
- **Verificação de imagem reconstituída** a partir do Base64 (comparação de hash MD5 idêntico ao original).  

---

## Checklist de Aceite

| Item | Descrição | Status |
|------|------------|--------|
| Upload realizado com sucesso | Bucket Azure acessível e objeto criado | ✅ |
| Função acionada e conversão registrada | Logs de execução no Azure Monitor | ✅ |
| Recuperação via API (GET /images/{id}) | Base64 íntegro e resposta < 800 ms | ✅ |
| DNS cutover testado e resolvido | Novo endpoint ativo | ✅ |
| Rollback funcional | DNS revertido para AWS | ✅ |
| Evidências anexadas no relatório final | Prints e logs incluídos | ✅ |

---

