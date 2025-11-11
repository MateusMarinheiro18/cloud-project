# Rede, Segurança e Identidade

## Introdução

Esta seção apresenta o mapeamento completo de rede, identidade e segurança da solução na nuvem de destino (Azure), assegurando que os mesmos padrões de isolamento, governança e confidencialidade da arquitetura original na AWS sejam mantidos. O foco está na equivalência entre recursos, nas políticas de menor privilégio e nos mecanismos de gestão de certificados e segredos, conforme as boas práticas de um plano de Disaster Recovery de nível corporativo. O diagrama a seguir ilustra a topologia lógica da infraestrutura, incluindo sub-redes, balanceadores, gateways e integrações de identidade.

---

## Diagrama lógico 

![Desenho Lógico](assets/desenho_logico.png)

---

## Estrutura de Rede e Roteamento

A infraestrutura de rede no Azure reproduz a topologia lógica da AWS, utilizando uma Virtual Network principal dividida em sub-redes públicas e privadas. A subnet pública hospeda o Application Gateway com WAF habilitado, responsável por terminar o tráfego HTTPS e aplicar políticas de segurança de camada 7. Já as sub-redes privadas contêm as aplicações (em um VM Scale Set ou App Service Plan) e os serviços de backend, como o CosmosDB, o Storage Account e o Key Vault. O tráfego de saída é roteado de forma controlada por meio de um NAT Gateway, garantindo visibilidade e segurança nos acessos externos. As Network Security Groups (NSGs) substituem os Security Groups da AWS e controlam detalhadamente o fluxo entre sub-redes e recursos, mantendo apenas as portas estritamente necessárias abertas.

## Identidade e Controle de Acesso

O modelo de identidade é baseado em Azure Active Directory (AAD) e Managed Identities. As aplicações que anteriormente utilizavam papéis IAM no EC2 ou Lambda passam a usar identidades gerenciadas no Azure, garantindo autenticação segura sem necessidade de credenciais fixas. O princípio de menor privilégio é aplicado rigorosamente, de modo que cada componente (VMSS, Functions, Pipelines) possui apenas as permissões necessárias para executar suas funções. Administradores acessam o ambiente por meio de contas corporativas com autenticação multifator, e todas as ações são auditadas pelo Azure Monitor e Defender for Cloud, assegurando rastreabilidade total das operações.

## Certificados e Gestão de Segredos

Os certificados TLS são emitidos e armazenados no Azure Key Vault, que substitui o AWS Certificate Manager. O processo de rotação é automatizado e ocorre anualmente para certificados e a cada 90 dias para segredos. O Application Gateway referencia diretamente o certificado armazenado no Key Vault, eliminando a necessidade de cópias manuais. Os segredos sensíveis, como chaves de API e cadeias de conexão, são acessados apenas pelas identidades gerenciadas das aplicações, com auditoria e retenção de logs por 90 dias. Em ambientes de maior exigência, o Key Vault Premium pode ser utilizado com módulos HSM para proteção criptográfica de chaves privadas.

## Segregação de Ambientes

A arquitetura adota uma separação rigorosa entre ambientes de produção, homologação e recuperação (DR), cada um hospedado em Resource Groups e VNets distintas. O tráfego entre ambientes é bloqueado por padrão, sendo permitido apenas mediante firewall ou peering controlado. O uso de Azure Policy reforça a aplicação de convenções de nomenclatura, tags e restrições de rede, garantindo coerência e segurança operacional.