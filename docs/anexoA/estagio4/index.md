## Anexo A  
---
### Estágio 4 — Projeto Avançado (Segurança, SLA e Custo) (Obj. 4)
---
#### Regular (2)

O Estágio 4 tem como objetivo consolidar toda a infraestrutura e os componentes do projeto, apresentando um **relatório final** que descreve as decisões de arquitetura, as práticas de segurança aplicadas e uma visão de custo e disponibilidade dos serviços utilizados.

Nesta fase, são avaliadas as boas práticas de **segurança**, **organização de recursos**, **uso eficiente dos serviços AWS** e a **comunicação clara da arquitetura** final do sistema.

---

##### Arquitetura Geral
A arquitetura final do sistema está baseada em serviços totalmente gerenciados pela **AWS**, visando escalabilidade, alta disponibilidade e segurança. O diagrama geral abaixo ilustra a integração dos componentes principais:

![Arquitetura](assets/arquitetura.png)  

---

##### Escolhas de Serviços e Justificativas

| Componente | Função no Sistema | Justificativa de Uso |
|-------------|----------------------|----------------------|
| **Amazon S3** | Armazenamento de arquivos estáticos (site institucional) e uploads de imagens | Alta durabilidade, baixo custo e integração nativa com Lambda |
| **AWS Lambda** | Conversão automática de imagens em Base64 e gravação no DynamoDB | Execução serverless e escalabilidade automática, sem custo ocioso |
| **Amazon DynamoDB** | Armazenamento dos metadados e Base64 das imagens | Banco NoSQL totalmente gerenciado, otimizado para alta disponibilidade |
| **Amazon EC2 (ASG)** | Hospedagem da área de clientes e API | Permite instâncias dinâmicas conforme demanda, com controle de custo |
| **Application Load Balancer (ALB)** | Balanceamento de carga e roteamento HTTPS | Garantia de distribuição uniforme, integração com ACM para HTTPS |
| **AWS Certificate Manager (ACM)** | Emissão de certificado TLS interno | Criptografia de tráfego HTTPS sem custo adicional |

---

##### Políticas de Segurança Aplicadas

1. **Security Groups Restritos:**
   - O SG do ALB permite apenas **porta 80 (HTTP)** e **443 (HTTPS)** de entrada pública.
   - O SG das instâncias EC2 aceita tráfego **somente do SG do ALB**, reforçando o isolamento interno.

2. **IAM Roles e Permissões Mínimas:**
   - **LambdaS3DynamoRole:** permissões limitadas a `s3:GetObject` e `dynamodb:PutItem`.
   - **EC2-ImageApp-Role:** acesso controlado apenas aos recursos necessários (S3, DynamoDB e CloudWatch Logs).

3. **HTTPS Ativo via ACM:**
   - Certificado privado `projeto-cloud25b.local` instalado no Listener 443 do ALB.
   - Tráfego entre cliente e load balancer criptografado (TLS 1.2+).

4. **Buckets S3 Privados:**
   - Uploads via **URL pré-assinada**, garantindo segurança e controle de acesso temporário.

---

##### Aspectos de Custo e Disponibilidade

O projeto foi desenvolvido com base em **serviços de baixo custo** e escalabilidade automática:

- **EC2 (ASG):** instâncias *t3.micro* sob demanda, ajustando a capacidade conforme CPU > 50%.
- **Lambda:** modelo *pay-per-use*, sem custo fixo quando inativo.
- **DynamoDB:** modo *On-Demand*, elimina necessidade de provisionamento de throughput.
- **S3:** custo próximo a US$ 0.023/GB/mês, apenas para armazenamento de objetos.

**Evidência — AWS Pricing Calculator**

![Custo AWS](assets/aws_calculator_estimate.png)  
*Figura 3 — Estimativa de custo mensal gerado via AWS Pricing Calculator, contemplando EC2, S3, DynamoDB e Lambda.*

---

##### Considerações de SLA e Resiliência

| Serviço | SLA Oficial AWS | Justificativa |
|-----------|-----------------|----------------|
| **S3** | 99.99% de disponibilidade | Garantia de durabilidade de 11 9s e replicacão entre AZs |
| **Lambda** | 99.95% | Alta resiliência por execução distribuída |
| **DynamoDB** | 99.999% | Multi-AZ e replicado automaticamente |
| **EC2 + ASG** | 99.99% | Escalabilidade e substituição automática de instâncias com falha |
| **ALB** | 99.99% | Balanceamento automático entre múltiplas zonas |

---