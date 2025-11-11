# 3. Procedimentos e Runbooks de DR

## 3.1 Visão Geral

Este capítulo detalha os procedimentos passo a passo para ativação do Disaster Recovery, organizados em fases sequenciais:

1. **Preparação** - Ações executadas antes de qualquer crise (durante Warm Standby)
2. **Ativação** - Procedimentos de emergência ao detectar falha
3. **Validação** - Testes de integridade pós-migração
4. **Rollback** - Retorno à AWS caso Azure falhe
5. **Comunicação** - Fluxos de informação interna e externa

**Tempo Total Estimado (RTO):** 2-4 horas do início da ativação ao serviço restaurado.

---

## 3.2 Fase 1: Preparação (Pré-Crise)

### Objetivo
Manter ambiente Azure em Warm Standby operacional, com dados sincronizados e infraestrutura validada.

### Frequência
Execução contínua (automações diárias + revisões mensais).

---

#### 3.2.1 Sincronização de Dados

**Responsável:** DevOps Engineer  
**Frequência:** Diária (3-5 AM UTC)  
**SLA:** Sync completo em < 2 horas

A sincronização de dados é uma das atividades centrais do modo **Warm Standby**. Seu objetivo é garantir que o ambiente Azure mantenha uma cópia atualizada de todos os componentes críticos da aplicação hospedados originalmente na AWS — incluindo o site estático, as imagens de upload e os metadados armazenados no DynamoDB.

O processo é dividido em três fluxos principais:

**Site Estático (S3 → Blob Storage)**

O conteúdo do site institucional hospedado no Amazon S3 é replicado diariamente para o container público do Azure Blob Storage configurado para Static Website Hosting. Essa tarefa é agendada como um cron job diário, que realiza uma sincronização recursiva entre os buckets da AWS e a conta de armazenamento da Azure, substituindo automaticamente arquivos alterados e removendo versões obsoletas.

Ao final da execução, o DevOps valida a integridade da sincronização por meio de uma contagem de arquivos e verificação de tamanho total, assegurando que a quantidade de objetos no Blob coincida com o S3. O sucesso é considerado alcançado quando não há divergências e todas as páginas estáticas respondem com código HTTP 200 no endpoint do Azure.

**Uploads de Imagens (S3 → Blob Storage Privado)**
As imagens enviadas pelos usuários, originalmente armazenadas em um bucket privado no S3, também são sincronizadas com o Blob Storage no Azure.
O processo é semelhante ao do site estático, porém sem exclusão automática de objetos no destino, evitando perda acidental de versões recentes. Após a sincronização, o sistema executa uma validação por amostragem, verificando um conjunto de arquivos (geralmente os últimos 10 envios) para confirmar que os hashes MD5 ou ETags são idênticos entre origem e destino.
A sincronização é considerada bem-sucedida quando 100% dos arquivos amostrados apresentam integridade confirmada.

**Metadados (DynamoDB → Cosmos DB)**
Este é o processo mais sensível, pois envolve dados estruturados essenciais ao funcionamento da aplicação.
Diariamente, é realizado um backup incremental do DynamoDB através de exportação automática para um bucket S3 dedicado. Essa exportação contém os metadados das imagens (IDs, timestamps, índices e fragmentos Base64).
Após o backup, o conjunto exportado é copiado para o ambiente de staging no Azure, onde é convertido do formato DynamoDB JSON para Cosmos DB JSON. A transformação adapta as chaves e índices, criando identificadores únicos compatíveis com o modelo de dados do Cosmos DB.

Em seguida, ocorre a importação em lote (bulk import) para o container de destino no Cosmos, mantendo a estrutura de particionamento original. A etapa final consiste em uma verificação de integridade, comparando o número total de documentos e validando amostras de 10% dos registros por meio de hashes de verificação.

O procedimento é considerado concluído com sucesso quando a contagem e os hashes de amostra coincidem com o backup da AWS.

**Monitoramento e Alertas:**
O processo é acompanhado tanto pela AWS quanto pela Azure. Alarmes do CloudWatch notificam o time de DevOps caso a exportação ou sincronização falhe por dois dias consecutivos, enquanto o Azure Monitor coleta logs de execução e erros das ferramentas de sincronização, permitindo rápida identificação de falhas de conectividade ou autenticação.

Com essa rotina, garante-se que o ambiente de standby mantenha uma cópia quase atualizada dos dados produtivos, suportando o RPO de até 24 horas definido no plano de DR e permitindo ativação imediata em caso de incidente na AWS.

---

#### 3.2.2 Validação de Infraestrutura (Mensal)

**Responsável:** Tech Lead  
**Frequência:** Primeira segunda-feira de cada mês  
**Duração:** Aproximadamente 1 hora

A validação mensal da infraestrutura tem como objetivo confirmar que todos os componentes do ambiente de *Warm Standby* no Azure permanecem íntegros, atualizados e prontos para serem ativados imediatamente em caso de desastre na AWS. Essa verificação periódica garante que nenhum recurso essencial tenha sido desconfigurado, expirado ou desprovisionado inadvertidamente, mantendo a confiabilidade do plano de recuperação.

Durante o procedimento, o Tech Lead segue uma rotina estruturada de checagem, percorrendo cada camada da arquitetura:

1. **Verificação do Blob Storage:** o responsável acessa o endereço público do site estático hospedado no Azure Blob Storage, confirmando que a página principal (index.html) responde corretamente e sem erros HTTP. Essa etapa assegura que o conteúdo do site e as permissões de acesso do container estão corretas, garantindo que o front-end possa ser disponibilizado sem intervenção adicional.

2. **Validação do Cosmos DB:** é testada a conectividade com o banco de dados de standby, confirmando que a conta Cosmos DB está ativa, o container de imagens acessível e as permissões de leitura e escrita válidas. Essa verificação demonstra que os metadados replicados do DynamoDB podem ser consultados e atualizados normalmente, condição indispensável para a ativação rápida do ambiente.

3. **Checagem das Azure Functions:** o Tech Lead confirma que as funções de processamento (principalmente a de conversão de imagens) estão corretamente implantadas e listadas no Function App. Essa etapa valida que o código do Lambda convertido foi sincronizado e está pronto para execução, inclusive com os *triggers* de Blob configurados.

4. **Inspeção do VM Scale Set (VMSS):** verifica-se que a definição do conjunto de máquinas virtuais da API encontra-se configurada e funcional, porém com capacidade zero de instâncias, conforme previsto no modo *Warm Standby*. Isso confirma que as imagens base e scripts de inicialização permanecem disponíveis para *scale-up* imediato em caso de ativação.

5. **Status do Application Gateway:** o Tech Lead acessa o painel ou usa o comando de gerenciamento para confirmar que o Application Gateway está provisionado, em estado operacional e pronto para receber novas configurações de *backend pools* e regras de roteamento. Esse gateway será responsável pelo tráfego de produção durante a operação de DR.

6. **Validação do Certificado TLS no Key Vault:** é verificado se o certificado wildcard usado para comunicação segura está armazenado no Key Vault e se sua validade é superior a 30 dias. Caso o certificado esteja prestes a expirar, inicia-se imediatamente o processo de renovação e revalidação do vínculo com o Application Gateway.

7. **Verificação do DNS de Staging:** o Tech Lead confirma que o registro DNS de contingência (por exemplo, `dr.demay.com`) está devidamente configurado e aponta para o endereço do Application Gateway em Azure. Essa checagem garante que o *failover* poderá ser realizado sem necessidade de criação manual de registros durante uma crise.

Ao final do processo, o Tech Lead registra o resultado de cada etapa no relatório mensal de prontidão de DR. Qualquer falha identificada deve ser corrigida imediatamente e retestada na mesma janela de manutenção. Caso a falha não possa ser resolvida no mesmo dia, o incidente é escalado ao CTO para priorização e acompanhamento.

Essa rotina de validação preventiva é fundamental para evitar surpresas no momento de um *cutover* real, garantindo que a infraestrutura de contingência esteja sempre em conformidade com o estado esperado e possa ser promovida a ambiente produtivo em menos de 2 horas.

---

## 3.3 Fase 2: Ativação (Durante Crise)

### Objetivo
Migrar tráfego de produção de AWS para Azure no menor tempo possível.

---

#### 3.3.1 Declaração de Desastre e Mobilização

**Responsável:** Tech Lead ou CTO  
**Duração:** 5 minutos

A fase de **declaração de desastre** marca o início oficial da ativação do plano de Disaster Recovery. É o momento em que a equipe de tecnologia reconhece formalmente uma falha grave ou uma interrupção prolongada na infraestrutura principal da AWS e decide iniciar a migração temporária para o ambiente de contingência no Azure.

Assim que a decisão é tomada, o **Tech Lead** ou, na ausência dele, o **CTO**, deve emitir uma comunicação imediata em todos os canais internos oficiais da organização. Essa mensagem deve conter informações claras sobre o cenário de falha, o motivo da ativação do DR, o horário de início do procedimento e a referência ao runbook correspondente. O objetivo é eliminar ambiguidades e garantir que toda a equipe envolvida saiba que o processo de recuperação está em andamento.

Em seguida, é convocada uma War Room virtual, um espaço de reunião exclusivo para gerenciamento do incidente. Essa sala de coordenação deve incluir obrigatoriamente os papéis críticos do processo: o Tech Lead, o engenheiro DevOps responsável pela execução técnica no Azure, o SRE encarregado de monitoramento e validação de métricas, e o CTO, que participa como autoridade de decisão para aprovar ações críticas e avaliar riscos. O Product Manager também pode ser incluído para centralizar a comunicação com usuários e stakeholders externos.

Após a definição de papéis, o Incident Commander realiza um **checkpoint**, certificando-se de que todos compreendem o cenário atual, o plano de ação e as prioridades imediatas. Esse alinhamento é essencial para evitar sobreposição de tarefas e minimizar erros durante a execução do runbook.

---

#### 3.3.2 Sincronização Final de Dados

**Responsável:** DevOps Engineer  
**Duração:** 30-60 minutos

**Objetivo:** Garantir que todos os dados da AWS estejam atualizados no Azure antes do redirecionamento do tráfego, reduzindo ao mínimo possível a perda de dados (RPO).

Nesta fase, o engenheiro de DevOps executa a última sincronização completa entre os ambientes, assegurando que nenhuma informação relevante fique apenas na origem antes do cutover. Trata-se de uma etapa crítica e irreversível, pois após o redirecionamento do DNS, a AWS deixa de ser a origem ativa dos dados.

O processo inicia com a suspensão temporária de novos uploads na AWS, se necessário, aplicando uma política de bloqueio no bucket de uploads. Essa medida evita divergências entre as versões dos arquivos durante a sincronização emergencial.

Em seguida, ocorre a sincronização forçada dos dados, que abrange dois fluxos:

**Uploads de imagens:** é realizada uma cópia direta dos objetos armazenados no S3 para o Blob Storage no Azure. O DevOps acompanha o progresso e registra logs de eventuais falhas de transferência.

**Metadados das imagens:** o DynamoDB é exportado rapidamente para o S3 e, em seguida, os arquivos são transferidos e importados para o Cosmos DB após transformação no formato apropriado. Esse processo é executado de forma automatizada por scripts previamente testados.

Ao término, o DevOps realiza uma validação de integridade, comparando contagens e amostras de registros entre os dois ambientes. Se ao menos 95% dos dados estiverem confirmadamente sincronizados, o Incident Commander autoriza o prosseguimento. Percentuais inferiores a 90% exigem avaliação executiva imediata pelo CTO, que decidirá entre aguardar uma nova tentativa ou aceitar a perda residual de dados, conforme o impacto operacional.

---

#### 3.3.3 Scale-Up da Infraestrutura Azure

**Responsável:** DevOps Engineer  
**Duração:** 30-45 minutos

**Objetivo:** Ativar e dimensionar os recursos do ambiente Azure até atingir a capacidade total de produção, garantindo que todos os componentes essenciais estejam operacionais e em estado “saudável”.

Nesta fase, o DevOps expande o ambiente de Warm Standby para produção. O processo começa ajustando o Cosmos DB, elevando a taxa de requisições (RU/s) para suportar a carga esperada. Esse aumento é monitorado para confirmar que o banco responde sem latência excessiva ou erros de throughput.

Em seguida, o VM Scale Set (API) é escalado para duas instâncias, ativando as máquinas virtuais e verificando se estão em estado “Running”. Após o boot, o DevOps confirma que os serviços da API foram iniciados corretamente e que os health checks retornam status 200.

Com o backend ativo, é verificado o Application Gateway, garantindo que todos os backends apareçam como “Healthy”. Caso algum esteja inoperante, os logs das VMs são revisados imediatamente para correção.

Por fim, realiza-se o aquecimento das Azure Functions, acionando um evento de teste para reduzir o tempo de resposta inicial. Logs no Application Insights confirmam a execução bem-sucedida.

---

#### 3.3.4 Cutover de DNS

**Responsável:** Tech Lead  
**Duração:** 10-15 minutos

**Objetivo:** Redirecionar todo o tráfego de usuários para o ambiente Azure de forma controlada e segura.

Após a confirmação de que todos os componentes do Azure estão “Healthy” e o Incident Commander autorizar o prosseguimento, o Tech Lead executa o cutover de DNS, ou seja, a troca dos registros que definem para onde os usuários serão direcionados.

Primeiro, verifica-se se o TTL (Time to Live) dos registros DNS já está configurado para um valor baixo — idealmente 300 segundos ou menos — para permitir uma propagação rápida. Se ainda estiver alto, o TTL é reduzido antes da troca e aguarda-se o tempo necessário para a atualização se espalhar entre os servidores de nomes públicos.

Em seguida, os registros A ou CNAME do domínio principal e do subdomínio da API são atualizados, apontando para o endereço público do Application Gateway hospedado no Azure. Essa alteração efetiva o redirecionamento do tráfego dos clientes para o novo ambiente. Caso o site estático também precise ser movido, seu registro é atualizado para o endpoint público do Blob Storage, garantindo a continuidade do front-end.

Após aplicar as mudanças, o Tech Lead monitora a propagação do DNS a partir de diferentes resolvers públicos (como Google, Cloudflare e OpenDNS) até confirmar que todos retornam o novo endereço IP do Azure. Esse processo normalmente leva poucos minutos e é acompanhado em tempo real pela equipe.

O corte de DNS é considerado concluído quando o domínio principal e o da API resolvem corretamente para o ambiente Azure em pelo menos três resolvers distintos, confirmando que os usuários estão acessando a infraestrutura de contingência de forma estável. A partir desse momento, a operação passa oficialmente a rodar no ambiente de recuperação.

---

## 3.4 Fase 3: Validação

**Responsável:** SRE + Tech Lead  
**Duração:** 30-60 minutos  
**Objetivo:** Confirmar que o sistema no ambiente Azure está completamente operacional após o failover e que todos os dados foram sincronizados corretamente.

---

### 3.4.1 Testes de Fumaça (Smoke Tests)

**Propósito:** Validar rapidamente se os principais componentes do ambiente Azure respondem de forma esperada.

Assim que o tráfego é direcionado para o Azure, o SRE executa uma série de verificações simples, conhecidas como smoke tests, que confirmam o funcionamento básico da aplicação. O objetivo não é testar a lógica completa do sistema, mas assegurar que os serviços principais — API, site estático, balanceador e DNS — estão operando como esperado.

O primeiro passo é verificar a saúde da API, acessando o endpoint de monitoramento /health. A resposta deve indicar o status “healthy”, confirmando que as instâncias do VM Scale Set estão ativas e a aplicação em execução.

Em seguida, o SRE acessa o site estático hospedado no Blob Storage, conferindo se o domínio principal responde com o código HTTP 200. Essa checagem simples garante que o front-end foi corretamente replicado e está acessível ao público.

O próximo teste envolve o Application Gateway, onde é validado se os backends — as máquinas virtuais que hospedam a API — estão com o estado “Healthy”. Caso algum apareça como “Unhealthy”, a equipe revisa rapidamente logs e configurações de rede antes de prosseguir.

Por fim, é feita a verificação de propagação DNS, consultando diferentes servidores públicos para confirmar que o domínio da aplicação (como api.demay.com) já está resolvendo para o endereço IP do Azure. Isso comprova que o redirecionamento do tráfego foi concluído com sucesso.

---

### 3.4.2 Teste End-to-End (Upload → Conversão → Consulta)

**Propósito:** Garantir integridade total do fluxo principal da aplicação.

Após a aprovação dos testes de fumaça, o time de SRE realiza um teste end-to-end com uma imagem de exemplo. Esse procedimento confirma que todas as integrações entre os serviços estão ativas e que o processamento ocorre sem falhas.

Primeiro, é gerada uma imagem de teste e enviada manualmente para o container de uploads do Blob Storage usando um link de acesso temporário (SAS URL). Esse upload simula a ação de um usuário real enviando uma nova imagem para processamento.

Logo em seguida, o time monitora a execução da Azure Function, que é automaticamente acionada pelo evento de criação do arquivo no Blob. Essa função deve processar a imagem, convertê-la em Base64 e registrar seus metadados no banco de dados Cosmos DB. O acompanhamento é feito pelo painel do Application Insights, onde se verifica a ocorrência de logs indicando o processamento da imagem de teste.

Depois, é realizada uma verificação direta no Cosmos DB, assegurando que o item correspondente à imagem foi inserido corretamente na coleção de dados. A validação inclui a presença do imageId, o chunkIndex e o conteúdo codificado, confirmando a integridade da operação.

---

### 3.4.3 Validação de Infraestrutura Complementar

**Objetivo:** Confirmar que todos os serviços de suporte e segurança estão íntegros.

Após a execução dos testes de fumaça e do teste end-to-end, o time de SRE realiza uma **validação complementar da infraestrutura**, com foco nos serviços que garantem estabilidade, segurança e observabilidade do ambiente. Essa verificação assegura que nenhum elemento de suporte crítico foi negligenciado durante o processo de failover.

A primeira checagem envolve o certificado **TLS armazenado no Key Vault**, verificando se o certificado wildcard utilizado nas comunicações seguras está válido e com prazo de expiração superior a 30 dias. Isso garante que o tráfego HTTPS permanecerá funcional e confiável durante o período de operação no Azure.

Em seguida, são analisados os logs e métricas do Azure Monitor e do Application Insights para confirmar que o monitoramento está ativo e recebendo dados em tempo real. A equipe verifica que a maioria das requisições registradas retornam código HTTP 200, sem ocorrência de erros 4xx ou 5xx significativos. Essa análise serve como indicador direto da saúde geral do sistema.

Também é verificada a disponibilidade e performance do Cosmos DB, assegurando que o throughput configurado (geralmente 4000 RU/s durante a operação ativa) está corretamente aplicado e que o banco responde a consultas sem latência anormal. Essa etapa é essencial para garantir que o armazenamento de metadados e o processamento das imagens não sofram lentidão sob carga.

Por fim, a equipe confirma a **sincronização e acessibilidade do Blob Storage**, validando se o container de uploads permanece íntegro e livre de falhas de autenticação. A simples consulta à contagem de objetos e à capacidade do bucket serve como indicador de consistência e conectividade entre os serviços.

---

## 3.5 Fase 4: Rollback

A fase de rollback representa o processo reverso do failover, em que o ambiente Azure, temporariamente promovido a produção, é gradualmente desativado e o tráfego volta a ser roteado para a infraestrutura original da AWS. Esse procedimento deve ser realizado somente após confirmação de estabilidade plena na AWS, garantindo que o retorno não cause perda de dados nem inconsistências.

A fase de rollback representa o processo reverso do failover, em que o ambiente Azure, temporariamente promovido a produção, é gradualmente desativado e o tráfego volta a ser roteado para a infraestrutura original da AWS. Esse procedimento deve ser realizado somente após confirmação de estabilidade plena na AWS, garantindo que o retorno não cause perda de dados nem inconsistências.

### Etapas Principais do Rollback

---

1. **Avaliação e Aprovação**

O Tech Lead confirma, junto ao CTO, que o ambiente AWS está pronto para receber novamente o tráfego. São revisados relatórios de monitoramento e logs para garantir que não há falhas regionais ou degradações de desempenho.

2. **Sincronização de Dados de Retorno (Azure → AWS)**

- O DevOps executa a sincronização inversa, exportando os dados mais recentes processados no Azure e reinserindo-os na AWS.
- Os arquivos armazenados no Blob Storage são copiados de volta para o bucket S3 de origem.
- Os metadados registrados no Cosmos DB são exportados e importados novamente no DynamoDB.
- A integridade é validada por amostragem (comparação de hashes e contagem de itens).
Esse passo assegura que nenhuma atualização ou imagem processada durante o período de contingência seja perdida no retorno.

3. **Revalidação de Serviços AWS**

O time SRE executa testes de disponibilidade em todos os componentes da AWS garantindo que o ambiente original está pronto para retomar o processamento. Health checks do Load Balancer e endpoints críticos devem retornar HTTP 200.

4. **Reconfiguração do DNS (Cutback)**

Após a confirmação de integridade, o Tech Lead realiza o cutback do DNS, apontando novamente os registros principais para o ALB da AWS. O TTL reduzido, definido durante o failover, permite que essa propagação ocorra rapidamente. Durante esse período, o tráfego pode ser dividido temporariamente entre as nuvens até a total convergência dos resolvers públicos.

5. **Desativação Gradual do Azure**

Assim que o tráfego for 100% revertido, o DevOps reduz a escala do VM Scale Set para zero instâncias, ajusta o throughput do Cosmos DB para o valor mínimo e confirma que nenhuma operação adicional está sendo processada. O ambiente Azure é então mantido em standby, pronto para futuras ativações.

--- 

Após o rollback, o ambiente Azure volta ao estado de Warm Standby, mantendo apenas recursos essenciais provisionados para futuras ativações. Um relatório pós-rollback deve ser gerado em até 24 horas, descrevendo o tempo total de retorno, eventuais divergências detectadas e melhorias a implementar no próximo ciclo de testes.

## 3.6 Fase 5: 

**Responsável:** Product Manager + Tech Lead + CTO
**Duração Estimada:** Contínua durante todo o ciclo de DR
**Objetivo:** Garantir uma comunicação clara, tempestiva e consistente entre as equipes técnicas, gestores e usuários, antes, durante e após o processo de Disaster Recovery.

A comunicação é um pilar fundamental do plano de DR, pois reduz incertezas, mantém o alinhamento entre os times e preserva a confiança dos usuários e stakeholders. Todo o processo segue um fluxo previamente definido, com responsabilidades claras e canais oficiais de contato.

### Comunicação Interna

Assim que o incidente é confirmado, o Tech Lead ou o Incident Commander emite uma mensagem de alerta interno, notificando as equipes de engenharia, produto e suporte. Essa comunicação inicial deve incluir o contexto do incidente, o status da infraestrutura afetada, a decisão de ativação do DR e o horário estimado para as próximas atualizações.

A War Room virtual é criada imediatamente servindo como canal central de coordenação técnica. Nela, apenas mensagens operacionais e decisões críticas são permitidas. O Product Manager atua como ponto focal de informação, atualizando os gestores e sintetizando o progresso das etapas.

Durante todo o processo, o Incident Commander publica atualizações de status curtas e regulares, informando o avanço das atividades e quaisquer desvios relevantes do plano.

### Comunicação Externa

O Product Manager, com aprovação do CTO, conduz a comunicação com usuários e partes externas. As mensagens devem ser objetivas, evitando termos técnicos excessivos e priorizando clareza e transparência.
As comunicações seguem três marcos principais:

1. **Aviso inicial de incidente** – notifica sobre a instabilidade e informa que o plano de recuperação foi acionado, com previsão estimada de restabelecimento.
2. **Atualizações durante o processo** – comunicados regulares com o progresso da recuperação.
3. **Confirmação de retorno** – mensagem final informando a normalização do serviço e agradecendo a compreensão dos usuários.

Todos os comunicados devem ser publicados nos canais oficiais da aplicação, com registro das datas e horários de envio para auditoria.

---

### Pós-Incidente e Encerramento

Após a conclusão do DR   é emitido um comunicado de estabilização confirmando o restabelecimento total dos serviços. Em até 48 horas, a equipe publica um relatório pós-incidente resumido, destacando causa raiz, tempo total de recuperação (RTO real) e medidas corretivas adotadas.

Em paralelo, o Tech Lead atualiza o documento de DR com as lições aprendidas e a lista de ações de melhoria, enquanto o Product Manager conduz uma reunião de retrospectiva com as equipes envolvidas, reforçando boas práticas de resposta e comunicação.