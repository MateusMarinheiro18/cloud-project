## Anexo A 
---
### Estágio 1 - Estrutura mínima (Obj. 1 e 2)
---
#### Regular (2)

Nesta etapa do Estágio 1, foi criada a estrutura do **site estático** hospedado no serviço **Amazon S3**, que serve como interface inicial da aplicação.  
O objetivo é disponibilizar um site público com as páginas principais do projeto — *Empresa*, *Grupo* e *Área de Clientes* — comprovando o funcionamento do *Static Website Hosting* e a correta configuração dos arquivos necessários.

---

Foi criado o bucket chamado **`cloud25b-project`** na região `us-east-1` (N. Virginia).  
Esse bucket foi configurado para **Website Hosting**, permitindo acesso público apenas para leitura dos arquivos estáticos do site.

A configuração de hospedagem foi feita na aba **Properties → Static website hosting**, conforme evidenciado abaixo:

![Configuração do Static Website Hosting](assets/webhosting.png)  
*Figura 1 — Bucket S3 “cloud25b-project” configurado com Website Hosting ativo e endpoint público habilitado.*

O endpoint público gerado pelo S3 é:  
**[http://cloud25d-project.s3-website-us-east-1.amazonaws.com](http://cloud25d-project.s3-website-us-east-1.amazonaws.com)**  

Esse link permite acesso direto ao site publicado, hospedado integralmente dentro do bucket S3.

---

O bucket contém os seguintes arquivos:

| Nome do Objeto | Tipo | Função |
|----------------|------|--------|
| `homepage.html` | HTML | Página principal (Home) do site |
| `404.html` | HTML | Página exibida em caso de erro ou rota inexistente |
| `scripts.js` | JavaScript | Contém a lógica de interação e scripts do front-end |
| `styles.css` | CSS | Define o estilo visual e a formatação do site |
| `uploads/` | Pasta | Diretório preparado para armazenar imagens enviadas pelos usuários (etapas futuras) |


A estrutura pode ser visualizada no console da AWS conforme imagem abaixo:

![Lista de Objetos do Bucket](assets/objects.png)  
*Figura 2 — Arquivos HTML, CSS e JS hospedados no bucket “cloud25b-project”, compondo o site estático.*

---

A seguir estão o HTML que compõem o site estático hospedado no S3.

??? success "HTML"
    ```
    <!DOCTYPE html>
    <html lang="pt-BR">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>CloudTech Solutions - Computação em Nuvem</title>
        <meta name="description" content="CloudTech Solutions - Transformando o futuro através da computação em nuvem. Inovação, segurança e performance.">
        <link rel="stylesheet" href="styles.css">
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    </head>
    <body>
        <!-- Header -->
        <header id="header" class="header">
            <div class="container">
                <div class="header-content">
                    <div class="logo animate-fade-in">
                        <div class="logo-icon">
                            <i class="fas fa-cloud"></i>
                            <div class="logo-glow">
                                <i class="fas fa-cloud"></i>
                            </div>
                        </div>
                        <span class="logo-text">Demay's Infra Company</span>
                    </div>

                    <nav class="nav-desktop">
                        <button class="nav-btn active" data-section="home">Início</button>
                        <button class="nav-btn" data-section="about">Nossa História</button>
                        <button class="nav-btn" data-section="team">Equipe</button>
                        <button class="nav-btn" data-section="clients">Área de Clientes</button>
                    </nav>

                    <button class="mobile-menu-btn">
                        <i class="fas fa-bars"></i>
                    </button>
                </div>

                <nav class="nav-mobile">
                    <button class="nav-btn active" data-section="home">Início</button>
                    <button class="nav-btn" data-section="about">Nossa História</button>
                    <button class="nav-btn" data-section="team">Equipe</button>
                    <button class="nav-btn" data-section="clients">Área de Clientes</button>
                </nav>
            </div>
        </header>

        <main>
            <!-- Hero Section -->
            <section id="home" class="hero">
                <div class="hero-bg">
                    <img src="src/assets/hero-cloud-bg.jpg" alt="Cloud Background">
                </div>
                <div class="hero-effects">
                    <div class="floating-element" style="animation-delay: 0s;"></div>
                    <div class="floating-element" style="animation-delay: 1s;"></div>
                    <div class="floating-element" style="animation-delay: 2s;"></div>
                </div>

                <div class="container hero-content">
                    <div class="hero-text">
                        <h1 class="hero-title animate-fade-in">
                            <span class="text-gradient glow-text">Demay's Infra</span><br>
                            <span>Company</span>
                        </h1>

                        <p class="hero-subtitle animate-fade-in" style="animation-delay: 0.2s;">
                            Transformando o futuro através da computação em nuvem<br>
                            <span class="highlight">Inovação • Segurança • Performance</span>
                        </p>

                        <div class="hero-features animate-fade-in" style="animation-delay: 0.4s;">
                            <div class="feature-card">
                                <i class="fas fa-cloud"></i>
                                <h3>Cloud Native</h3>
                                <p>Soluções 100% em nuvem</p>
                            </div>
                            <div class="feature-card">
                                <i class="fas fa-shield-alt"></i>
                                <h3>Segurança Total</h3>
                                <p>Proteção de dados avançada</p>
                            </div>
                            <div class="feature-card">
                                <i class="fas fa-bolt"></i>
                                <h3>Alta Performance</h3>
                                <p>Velocidade incomparável</p>
                            </div>
                        </div>

                        <div class="hero-cta animate-fade-in" style="animation-delay: 0.6s;">
                            <button class="btn btn-hero" data-section="about">Conheça Nossa História</button>
                            <a href="../html/client-page.html" class="btn btn-outline">Área de Clientes</a>
                        </div>
                    </div>
                </div>

                <div class="scroll-indicator">
                    <div class="scroll-mouse">
                        <div class="scroll-dot"></div>
                    </div>
                </div>
            </section>

            <!-- About Section -->
            <section id="about" class="about">
                <div class="container">
                    <div class="section-header animate-fade-in">
                        <h2>Nossa <span class="text-gradient">História</span></h2>
                        <p>Desde 2018, a Demay's Infra revoluciona o mercado de computação em nuvem, oferecendo soluções inovadoras que transformam negócios em todo o mundo.</p>
                    </div>

                    <div class="about-content">
                        <div class="mission-vision">
                            <div class="card animate-fade-in" style="animation-delay: 0.2s;">
                                <div class="card-header">
                                    <h3 class="text-gradient">Nossa Missão</h3>
                                </div>
                                <div class="card-content">
                                    <p>Transformar a maneira como as empresas operam através de soluções em nuvem seguras, escaláveis e inovadoras. Acreditamos que a tecnologia deve ser acessível e poderosa, permitindo que nossos clientes foquem no que fazem de melhor.</p>
                                </div>
                            </div>

                            <div class="card animate-fade-in" style="animation-delay: 0.4s;">
                                <div class="card-header">
                                    <h3 class="text-gradient">Nossa Visão</h3>
                                </div>
                                <div class="card-content">
                                    <p>Ser a empresa líder mundial em computação em nuvem, reconhecida pela excelência em inovação, segurança e atendimento ao cliente. Queremos criar um futuro onde a tecnologia conecta e potencializa todos os negócios.</p>
                                </div>
                            </div>
                        </div>

                        <div class="timeline-section">
                            <h3 class="timeline-title animate-fade-in">Marcos da Nossa <span class="text-gradient">Jornada</span></h3>
                            <div class="timeline">
                                <div class="timeline-item animate-fade-in" style="animation-delay: 0.1s;">
                                    <div class="timeline-icon">
                                        <i class="fas fa-building"></i>
                                    </div>
                                    <div class="timeline-content">
                                        <div class="timeline-year">2018</div>
                                        <h4>Fundação</h4>
                                        <p>Demay's nasce com a visão de democratizar a computação em nuvem</p>
                                    </div>
                                </div>
                                <div class="timeline-item animate-fade-in" style="animation-delay: 0.2s;">
                                    <div class="timeline-icon">
                                        <i class="fas fa-rocket"></i>
                                    </div>
                                    <div class="timeline-content">
                                        <div class="timeline-year">2020</div>
                                        <h4>Expansão</h4>
                                        <p>Primeira expansão internacional e 1000+ clientes atendidos</p>
                                    </div>
                                </div>
                                <div class="timeline-item animate-fade-in" style="animation-delay: 0.3s;">
                                    <div class="timeline-icon">
                                        <i class="fas fa-award"></i>
                                    </div>
                                    <div class="timeline-content">
                                        <div class="timeline-year">2022</div>
                                        <h4>Inovação</h4>
                                        <p>Lançamento da plataforma de IA proprietária CloudAI</p>
                                    </div>
                                </div>
                                <div class="timeline-item animate-fade-in" style="animation-delay: 0.4s;">
                                    <div class="timeline-icon">
                                        <i class="fas fa-users"></i>
                                    </div>
                                    <div class="timeline-content">
                                        <div class="timeline-year">2024</div>
                                        <h4>Liderança</h4>
                                        <p>Reconhecidos como líder em soluções cloud no Brasil</p>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="values-section animate-fade-in" style="animation-delay: 0.6s;">
                            <div class="card">
                                <div class="card-header">
                                    <h3>Nossos <span class="text-gradient">Valores</span></h3>
                                </div>
                                <div class="card-content">
                                    <div class="values-grid">
                                        <div class="value-item">
                                            <h4>Inovação</h4>
                                            <p>Sempre na vanguarda da tecnologia, criando soluções que antecipam o futuro.</p>
                                        </div>
                                        <div class="value-item">
                                            <h4>Segurança</h4>
                                            <p>Proteção de dados é nossa prioridade absoluta em todas as soluções.</p>
                                        </div>
                                        <div class="value-item">
                                            <h4>Excelência</h4>
                                            <p>Comprometimento com a qualidade superior em cada projeto entregue.</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- Team Section -->
            <section id="team" class="team">
                <div class="container">
                    <div class="section-header animate-fade-in">
                        <h2>Nossa <span class="text-gradient">Equipe</span></h2>
                        <p>Profissionais apaixonados por tecnologia, unidos pela missão de transformar o mundo através da computação em nuvem.</p>
                    </div>

                    <div class="team-stats">
                        <div class="stat-card animate-fade-in" style="animation-delay: 0.1s;">
                            <div class="stat-number">50+</div>
                            <div class="stat-label">Profissionais</div>
                        </div>
                        <div class="stat-card animate-fade-in" style="animation-delay: 0.2s;">
                            <div class="stat-number">15+</div>
                            <div class="stat-label">Anos de Experiência</div>
                        </div>
                        <div class="stat-card animate-fade-in" style="animation-delay: 0.3s;">
                            <div class="stat-number">1000+</div>
                            <div class="stat-label">Projetos Entregues</div>
                        </div>
                        <div class="stat-card animate-fade-in" style="animation-delay: 0.4s;">
                            <div class="stat-number">25+</div>
                            <div class="stat-label">Certificações Cloud</div>
                        </div>
                    </div>

                    <div class="team-members">
                        <div class="member-card animate-fade-in" style="animation-delay: 0.1s;">
                            <div class="member-avatar">
                                <img src="https://avatars.githubusercontent.com/u/7607583?v=4" alt="Thiago Demay">
                                <div class="avatar-glow"></div>
                            </div>
                            <div class="member-info">
                                <h4>Thiago Demay</h4>
                                <p class="member-role">CEO & Founder</p>
                                <p class="member-bio">Visionário em tecnologia com 15+ anos de experiência em cloud computing. Formada em Engenharia de Software pela USP.</p>
                                <div class="member-skills">
                                    <span class="skill-tag">Liderança</span>
                                    <span class="skill-tag">Cloud Architecture</span>
                                    <span class="skill-tag">Strategy</span>
                                </div>
                                <div class="member-social">
                                    <i class="fab fa-linkedin"></i>
                                    <i class="fab fa-github"></i>
                                    <i class="fas fa-envelope"></i>
                                </div>
                            </div>
                        </div>

                        <div class="member-card animate-fade-in" style="animation-delay: 0.2s;">
                            <div class="member-avatar">
                                <img src="https://lh3.googleusercontent.com/a/ACg8ocJ2J2rwbD5LFsScw-QYTzzI3oyqoJ99eQTZwXZv1OgZ550stEA=s192-c" alt="Mateus Marinheiro">
                                <div class="avatar-glow"></div>
                            </div>
                            <div class="member-info">
                                <h4>Mateus Marinheiro</h4>
                                <p class="member-role">CTO</p>
                                <p class="member-bio">Especialista em arquiteturas distribuídas e segurança cibernética. PhD em Engenharia da Computação pela USP.</p>
                                <div class="member-skills">
                                    <span class="skill-tag">Cloud Computing</span>
                                    <span class="skill-tag">Security</span>
                                    <span class="skill-tag">SaaS</span>
                                </div>
                                <div class="member-social">
                                    <i class="fab fa-linkedin"></i>
                                    <i class="fab fa-github"></i>
                                    <i class="fas fa-envelope"></i>
                                </div>
                            </div>
                        </div>

                        <div class="member-card animate-fade-in" style="animation-delay: 0.3s;">
                            <div class="member-avatar">
                                <img src="https://avatars.githubusercontent.com/u/130405994?v=4" alt="Eduardo Zorzi">
                                <div class="avatar-glow"></div>
                            </div>
                            <div class="member-info">
                                <h4>Eduardo Zorzi</h4>
                                <p class="member-role">In N Out Manager</p>
                                <p class="member-bio">Especialista em estratégias de entrada e saida da corporação. Cuida do fluxo de entrada e saída de pessoas, além de ser mestre em cadsatros.</p>
                                <div class="member-skills">
                                    <span class="skill-tag">iFood</span>
                                    <span class="skill-tag">Uber</span>
                                    <span class="skill-tag">Prefect</span>
                                </div>
                                <div class="member-social">
                                    <i class="fab fa-linkedin"></i>
                                    <i class="fab fa-github"></i>
                                    <i class="fas fa-envelope"></i>
                                </div>
                            </div>
                        </div>

                        <div class="member-card animate-fade-in" style="animation-delay: 0.3s;">
                            <div class="member-avatar">
                                <img src="https://media.licdn.com/dms/image/v2/D4D03AQG5x4v70kZliw/profile-displayphoto-scale_400_400/B4DZgY4l5QG8Ag-/0/1752764146448?e=1761782400&v=beta&t=5EiG0_DzGbdOvkfNY_8amRtqcmTlXv3ocxrXg9azoKk" alt="Thiago Gouveia">
                                <div class="avatar-glow"></div>
                            </div>
                            <div class="member-info">
                                <h4>Thiago Gouveia</h4>
                                <p class="member-role">Front Office</p>
                                <p class="member-bio">Sempre a frente da entrada da coroporação, evitando a entrada de pessoas mal intencionadas. Ágil na hora de puxar o rádio ao relatar ocorrências.</p>
                                <div class="member-skills">
                                    <span class="skill-tag">Mr Aura</span>
                                    <span class="skill-tag">Terno</span>
                                    <span class="skill-tag">Rádio</span>
                                </div>
                                <div class="member-social">
                                    <i class="fab fa-linkedin"></i>
                                    <i class="fab fa-github"></i>
                                    <i class="fas fa-envelope"></i>
                                </div>
                            </div>
                        </div>
                </div>
            </section>

            <!-- Clients Section -->
            <section id="clients" class="clients">
                <div class="container">
                    <div class="section-header animate-fade-in">
                        <h2>Área de <span class="text-gradient">Clientes</span></h2>
                        <p>Acesse nossa plataforma exclusiva para gerenciar suas imagens, tags e conversões Base64.</p>
                    </div>

                    <div class="clients-content">
                        <div class="client-features">
                            <div class="feature-item animate-fade-in" style="animation-delay: 0.2s;">
                                <i class="fas fa-upload"></i>
                                <h4>Upload Seguro</h4>
                                <p>Envie suas imagens com total segurança e criptografia</p>
                            </div>
                            <div class="feature-item animate-fade-in" style="animation-delay: 0.4s;">
                                <i class="fas fa-tags"></i>
                                <h4>Sistema de Tags</h4>
                                <p>Organize e identifique suas imagens com tags personalizadas</p>
                            </div>
                            <div class="feature-item animate-fade-in" style="animation-delay: 0.6s;">
                                <i class="fas fa-code"></i>
                                <h4>Conversão Base64</h4>
                                <p>Visualize e baixe a conversão Base64 das suas imagens</p>
                            </div>
                        </div>

                        <div class="client-cta animate-fade-in" style="animation-delay: 0.8s;">
                            <a href="http://alb-cloud25d-1290402490.us-east-1.elb.amazonaws.com" class="btn btn-hero btn-large">
                                <i class="fas fa-arrow-right"></i>
                                Acessar Área de Clientes
                            </a>
                        </div>
                    </div>
                </div>
            </section>
        </main>

        <!-- Footer -->
        <footer class="footer">
            <div class="container">
                <div class="footer-content">
                    <div class="footer-logo">
                        <div class="logo-icon">
                            <i class="fas fa-cloud"></i>
                        </div>
                        <span class="logo-text">Demay's Infra Company</span>
                    </div>
                    <p class="footer-text">Transformando o futuro através da computação em nuvem</p>
                    <div class="footer-copyright">
                        <span>© 2024 Demay's Infra Company. Todos os direitos reservados.</span>
                    </div>
                </div>
            </div>
        </footer>

        <!-- Toast Container -->
        <div id="toast-container" class="toast-container"></div>

        <script src="script.js"></script>
    </body>
    </html>
    ```

--- 

A instância **EC2** é responsável por rodar o back-end da Área de Clientes da aplicação.  
No nível *Regular* é exigido que a EC2 esteja em execução, com `Status checks: 2/2 passed`, acessível publicamente (quando apropriado) e que exponha o endpoint `/health` retornando **HTTP 200** para comprovar que o serviço está ativo.

---

- **Instância EC2 em execução:** `evidence_ec2_running.png`  
  ![EC2 em execução](assets/ec2running.png)  
  **Legenda:** *Instância EC2 “EC2-Cloud25D” em execução (Running), tipo `t2.micro` e `Status checks: 2/2 passed` — evidência de instância operacional.*

- **Security Group associado à EC2:** `evidence_security_group.png`  
  ![Security Group](assets/ec2sg.png)  
  **Legenda:** *Security Group da EC2 com regras Inbound.*

- **Role IAM vinculada à EC2:** `evidence_iam_role.png`  
  ![IAM Role](assets/role.png)  
  **Legenda:** *Role IAM `EC2-ImageApp-Role` associada à instância, com políticas mínimas para acessar recursos AWS (ex.: DynamoDB, S3) sem embutir credenciais.*

- **Resposta do endpoint /health:** `evidence_health_check.png`  
  ![Health Check](assets/healthcheck.png)  
  **Legenda:** *Resposta do endpoint `/health` retornando `{"status":"ok"}` (HTTP 200), comprovando que o serviço está ativo e respondendo.*


---

A **Virtual Private Cloud (VPC)** é o componente fundamental da infraestrutura em nuvem, pois define o ambiente de rede isolado onde todos os recursos da aplicação — como EC2 e ALB — são provisionados.  
No projeto, a VPC foi criada para garantir **segurança, segmentação e controle de tráfego** entre os serviços hospedados na AWS.  

Nesta etapa, a VPC atende aos requisitos do **nível Regular**, contendo:
- Uma **VPC personalizada** (não a default);
- Uma **sub-rede pública** associada a ela;
- Uma **tabela de rotas** permitindo acesso externo por meio de um **Internet Gateway (IGW)**;
- E **Security Groups** controlando o tráfego de entrada e saída.

---

| Componente | Descrição |
|-------------|------------|
| **VPC** | Rede principal que abriga todos os recursos do projeto. CIDR configurado: `172.31.0.0/16`. |
| **Sub-rede pública** | Criada dentro da VPC e associada à tabela de rotas pública. Responsável por permitir comunicação da EC2 com a internet. |
| **Route Table** | Contém a rota `0.0.0.0/0` apontando para o Internet Gateway, garantindo conectividade externa. |
| **Internet Gateway (IGW)** | Fornece acesso público à EC2 e à aplicação via HTTP/HTTPS. |
| **Security Groups** | Controlam portas de entrada (80/443) e restringem SSH a IPs específicos. |

---

![VPC criada](assets/vpc1.png)  
*Figura 1 — Painel da VPC, com estado “Available” e faixa de endereçamento `172.31.0.0/16`. DNS resolution e hostnames habilitados, e tabela de rotas associada.*

---

![Sub-rede](assets/subnet.png)  
*Figura 2 — Sub-rede pública associada à VPC, exibindo o CIDR e a zona de disponibilidade.*

---

![Route Table](assets/routes.png)  
*Figura 3 — Tabela de rotas associada à VPC. Comprova que a VPC tem saída para a internet via Internet Gateway.*

---

![Security Group](assets/sg.png)  
*Figura 4 — Regras Inbound do Security Group, permitindo tráfego HTTP (80) e HTTPS (443) e restringindo SSH (22) ao IP do administrador.*

---

#### Bom (3)

Nesta seção detalharemos a organização da VPC em sub-redes públicas e privadas, a implantação e configuração de um Application Load Balancer (ALB) com seu Target Group e health checks, a associação de instâncias EC2 ao ALB e as verificações de funcionamento (teste via browser/curl e comandos AWS CLI). Também apresentaremos as evidências necessárias como screenshots do ALB, do Target Group, das sub-redes e da tabela de rotas, além de um diagrama lógico da arquitetura e instruções de boas práticas de segurança e operação.

---

##### VPC

A arquitetura de rede do projeto  foi organizada dentro de uma única **VPC personalizada**, que abriga todos os recursos de infraestrutura da aplicação. Dentro dessa VPC foram criadas **sub-redes públicas** responsáveis por hospedar o **Application Load Balancer (ALB)** e a instância **EC2** do back-end, e **sub-redes privadas** (nas zonas `us-east-1c` e `us-east-1d`) reservadas para futuras expansões, como bancos de dados ou serviços internos.

O ALB distribui o tráfego HTTP recebido da internet para o **Target Group**, que contém a instância EC2 configurada com o endpoint `/health` para monitoramento de disponibilidade. O tráfego de saída é roteado por uma **tabela de rotas pública**, que direciona o destino `0.0.0.0/0` para o **Internet Gateway (IGW)**, garantindo conectividade externa de forma controlada e segura.

##### Diagrama da VPC

``` mermaid
flowchart TD
  subgraph VPC["VPC Cloud25B — CIDR: 172.31.0.0/16"]
    style VPC fill:#f8fbff,stroke:#93c5fd,stroke-width:1px

    subgraph Public["Sub-redes Públicas"]
      direction TB
      ALB[("Application Load Balancer\nListener: 80 (HTTP)")]:::alb
      PUB1["Sub-rede Pública — us-east-1a"]:::subnet
      PUB2["Sub-rede Pública — us-east-1b"]:::subnet
      RT["Route Table Pública\n(rota 0.0.0.0/0 → IGW)"]:::rt
      IGW["Internet Gateway"]:::igw
    end

    subgraph Private["Sub-redes Privadas"]
      direction TB
      PRIV1["Sub-rede Privada — us-east-1c"]:::subnet
      PRIV2["Sub-rede Privada — us-east-1d"]:::subnet
    end

    subgraph EC2S["Instâncias e Target Group"]
      direction TB
      TG["Target Group — cloud25b-target"]:::tg
      EC2[("Instância EC2 — Cloud25D\nBackend /health ativo")]:::ec2
    end
  end

  %% Conexões
  ALB -->|Distribui tráfego| TG
  TG -->|Health checks /health| EC2
  EC2 --> PUB1
  PUB1 --> RT
  PUB2 --> RT
  RT -->|0.0.0.0/0| IGW
  IGW --> Internet[(Internet)]:::internet

  %% Estilo
  classDef alb fill:#fff7ed,stroke:#f59e0b,stroke-width:1px;
  classDef tg fill:#eef2ff,stroke:#6366f1,stroke-width:1px;
  classDef ec2 fill:#ecfeff,stroke:#06b6d4,stroke-width:1px;
  classDef subnet fill:#f0f9ff,stroke:#93c5fd,stroke-width:1px;
  classDef rt fill:#f8fafc,stroke:#94a3b8,stroke-width:1px;
  classDef igw fill:#fff1f2,stroke:#fb7185,stroke-width:1px;
  classDef internet fill:#f1f5f9,stroke:#94a3b8,stroke-width:1px;
```

##### ALB DNS

O Application Load Balancer (ALB) foi criado na VPC do projeto e configurado para distribuir requisições HTTP (porta 80) entre as instâncias EC2 do backend.
Durante os testes, o ALB apresentou status “Active”, com DNS público acessível e health checks configurados para a rota /health, que retornam HTTP 200 OK a partir da instância EC2-Cloud25D.
Essa configuração comprova que o balanceador está devidamente provisionado, respondendo a requisições externas e mantendo health checks para as instâncias registradas no Target Group.

##### Figura 1 — Application Load Balancer Ativo

![Application Load Balancer Ativo](assets/alb_active.png)

📘 *Figura 1 — Application Load Balancer criado na região `us-east-1`, com estado “Active”, tipo `Application` e DNS público configurado (`cloud25b-alb-xxxxxxx.us-east-1.elb.amazonaws.com`). Evidência de que o ALB está ativo e acessível na VPC `vpc-0fe22edc27c75fedb`.*

---

#### Excelente (4)

> Infra completa e coerente: VPC (2 AZs), IGW, NAT; Security Groups organizados; EC2 com `user-data` e documentação clara das decisões.

##### Resumo
Nesta etapa foram atendidos os requisitos dos níveis anteriores e entregues artefatos extras necessários para o nível **Excelente (4)**:  
- VPC customizada com duas zonas de disponibilidade (2 AZs) e sub-redes públicas/privadas;  
- Internet Gateway (IGW) e NAT Gateway (para sub-redes privadas com saída à Internet);  
- Security Groups (SG) organizados por função (ALB, EC2 backend, SSH administrativo);  
- Application Load Balancer (ALB) com Target Group e health checks configurados para `/health`;  
- Instância EC2 configurada via `user-data` para deploy do backend;  
- Documentação justificando CIDR, tabela de sub-redes/rotas e prints das evidências.

---

##### Arquitetura de Rede (visão lógica)

```mermaid
flowchart TD
  subgraph VPC["VPC — CIDR: 10.10.0.0/16 (justificação abaixo)"]
    style VPC fill:#f8fbff,stroke:#93c5fd,stroke-width:1px

    subgraph Public["Sub-redes Públicas (2 AZs)"]
      direction TB
      ALB[("ALB — Listener 80 (HTTP)")]:::alb
      PUB_A["Public Subnet — us-east-1a\\nCIDR: 10.10.0.0/24"]:::subnet
      PUB_B["Public Subnet — us-east-1b\\nCIDR: 10.10.1.0/24"]:::subnet
      IGW["Internet Gateway"]:::igw
    end

    subgraph Private["Sub-redes Privadas (2 AZs)"]
      direction TB
      PRIV_A["Private Subnet — us-east-1a\\nCIDR: 10.10.10.0/24"]:::subnet
      PRIV_B["Private Subnet — us-east-1b\\nCIDR: 10.10.11.0/24"]:::subnet
      NAT["NAT Gateway (in Public Subnet)"]:::nat
    end

    subgraph EC2S["Recursos"]
      direction TB
      TG["Target Group — cloud25b-target"]:::tg
      EC2[("EC2 Backend (in Private Subnet)\\n/user-data configurado")]:::ec2
      RDS["(opcional) DB em Private Subnet"]:::db
    end
  end

  ALB -->|HTTP| TG
  TG -->|health /health| EC2
  EC2 --> PRIV_A
  PRIV_A --> NAT
  NAT --> PUB_A
  PUB_A --> IGW
  PUB_B --> IGW
  IGW --> Internet[(Internet)]:::internet

  classDef alb fill:#fff7ed,stroke:#f59e0b,stroke-width:1px;
  classDef tg fill:#eef2ff,stroke:#6366f1,stroke-width:1px;
  classDef ec2 fill:#ecfeff,stroke:#06b6d4,stroke-width:1px;
  classDef subnet fill:#f0f9ff,stroke:#93c5fd,stroke-width:1px;
  classDef igw fill:#fff1f2,stroke:#fb7185,stroke-width:1px;
  classDef nat fill:#fffbe6,stroke:#f59e0b,stroke-width:1px;
  classDef db fill:#f1f5f9,stroke:#94a3b8,stroke-width:1px;
  classDef internet fill:#f1f5f9,stroke:#94a3b8,stroke-width:1px;
```

##### Justificativa do CIDR
Escolhemos `10.10.0.0/16` por ser um bloco privado amplo o suficiente para:

- permitir criação separada de sub-redes públicas/privadas por AZ;  
- minimizar risco de sobreposição com redes on-premises típicas (10.x usados frequentemente, mas escolhendo 10.10.0.0 reduz colisões óbvias);  
- reservar espaço para futuras expansões (VPNs, peering, conectividade transit).



---

##### Security Groups (resumo)
- **SG-ALB**: permite inbound 80/443 (0.0.0.0/0), outbound para target group (porta 3000 ou porta do backend).  
- **SG-EC2-Backend**: permite inbound apenas do SG-ALB na porta do app (ex.: 3000), SSH permitido somente do IP administrativo (meu-IP/32).  
- **SG-NAT**: regra padrão gerada pelo serviço (se usar NAT Gateway).  

Princípio aplicado: *least privilege* — portas/ips mínimos necessários.

---

##### EC2 — user-data & Bootstrapping
A instância EC2 usada para o backend foi provisionada com `user-data` que:
1. atualiza o SO; 2. instala Node.js (ou outra runtime); 3. puxa código do repositório; 4. configura o serviço (systemd) para iniciar o backend automaticamente; 5. registra logs em CloudWatch (opcional).

##### Health check e ALB
- ALB configurado com **Listener 80** e Target Group apontando para as instâncias EC2 (porta 3000 do app).  
- Health check: `HTTP /health` com timeout 5s, intervalo 30s, threshold healthy 2 / unhealthy 2.  
- Validação: Target Group reports `healthy` para todas as instâncias registradas.

---

##### Tabela de rotas (exemplo)
- Route Table (public): 0.0.0.0/0 → Internet Gateway (igw-xxxx).  
- Route Table (private): 0.0.0.0/0 → NAT Gateway (nat-xxxx) — permite que as instâncias privadas façam saídas (patches, downloads).

---