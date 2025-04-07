# Solução Arquitetural para Migração de Sistema Legado para Modelo Híbrido

Autor: Evandro da Costa Vieira\
Telefone: (61) 999357026\
Email: <evandro.costajr@gmail.com>\
<https://www.linkedin.com/in/evandro-da-costa-vieira-6160a3178/>

## Introdução

A proposta apresenta uma solução arquitetural para migrar um sistema legado para um modelo híbrido (on-premises + cloud), com foco em escalabilidade, alta disponibilidade e otimização de custos. 

Inicialmente, é feita uma análise dos objetivos de negócio e das motivações da migração, seguida por uma avaliação detalhada da infraestrutura atual, incluindo servidores, redes, bancos de dados e licenças. 

Com base nessa análise, será dimensionada a nova estrutura em nuvem, utilizando uma topologia genérica descrita no cenário “AS-IS” e representada com recursos da AWS na solução “TO-BE” — embora seja possível usar outras nuvens. A proposta inclui o uso de IaC (Infraestrutura como Código) com Terraform, para automação e melhor gestão da infraestrutura. Também foi feito o desenho da arquitetura de Disaster Recovery e algumas diretrizes de segurança importantes no projeto.

As escolhas tecnológicas foram justificadas com prós e contras ao longo do projeto.

## 1. Cenário Atual AS-IS - Objetivo do Desafio

O sistema atual proposto é composto por uma infraestrutura predominantemente on-premises, mas a empresa deseja migrar para um ambiente híbrido para aproveitar a escalabilidade e flexibilidade da nuvem sem abandonar investimentos existentes.

A arquitetura de uma solução de produto de fluxo de caixa atual é composta por dois serviços rodando em VMs onpremises:
- Serviço que faz o controle de lançamentos;
- Serviço do consolidado diário.

O sistema possui:

1. Infraestrutura:
    - Dificuldade em escalar horizontalmente e verticamente as VMs.
    - Manutenção dificultada.
    - Hospedagem dos VCenters em data centers locais, gerando:
        - Altos custos com manutenção
        - Alto consumo de energia
        - Ocupação de espaço físico
2. Armazenamento:
    - Armazenamento em discos on-premises.
3. Sistema Operacional
    - Dificuldade em subir de versão.
    - Aplicação manual de patches de SO.
4. Bancos de Dados
    - Relacionais: Ex: Oracle, SQL Server. 
    - NoSQL: Ex: MongoDB, Redis.

**Diagrama da Topologia da Solução Legada**

![Topologia da Solução - Cenário Atual](/Images/Topologia_da_Solução_OnPremise.png)

## 2. Arquitetura Proposta TO-BE - Requisitos obrigatórios

A arquitetura desenhada leva em consideração as premissas de escalabilidade e a otimização de custos como prioritárias para atender o cliente. Essas premissas podem mudar de acordo com a necessidade do cliente, porém a mudança delas pode impactar parte ou toda a solução proposta.

A estratégia de migração adotada dado uma infraestrutura híbrida será a Replatform (lift-tinker-and-shift) em que consideramos a migração da infraestrutura como está e faremos pequenas modificações para melhor performance. A ideia é que após a migração com a utilização da nuvem seja feito o redesenho da aplicação para aproveitar o máximo da nuvem, deixando toda a aplicação na nuvem.

**Objetivos**

\-Modernizar o sistema legado sem interrupção do serviço.\
\-Aproveitar os recursos da nuvem para escalabilidade, disponibilidade e eficiência operacional.\
\-Reduzir custos operacionais a longo prazo.\
\-Melhorar a segurança e o desempenho.\
\-Resiliência e Recuperação (DR).\
\-Automação e Governança.

### a. Dimensionamento de Recursos

Para o levantamento dos dados do ambiente atual, serão feitas medições reais de uso por um período mínimo de 30 dias, sendo o ideal de 90 dias. Como se trata de um ambiente legado, caso não exista uma ferramenta de estudo de capacidade implantada, é possível utilizar soluções como Zabbix, Nagios, Prometheus + Grafana, entre outras. Para máquinas virtuais, pode-se utilizar o vROps para levantar o consumo de memória, CPU e armazenamento.

\- **CPU:** Deve-se observar a média de uso e os picos, bem como os horários e dias da semana em que esses picos ocorrem. Utiliza-se a média com uma margem para picos (exemplo: se a média for 40% e o pico atingir 80%, estimar com 60–70% como margem de segurança). Também é possível utilizar ferramentas de Auto Scaling para reagir dinamicamente às variações de CPU.

\- **Memória:** Avaliar a memória alocada e a realmente utilizada, considerando requisitos específicos da aplicação. Algumas aplicações exigem uma quantidade mínima de memória alocada, mesmo que não utilizem toda essa capacidade. O mesmo raciocínio se aplica ao processamento e ao uso de disco.

\- **Armazenamento:** Avaliar o tamanho atual, o crescimento nos últimos meses, a projeção de crescimento e adicionar uma margem de segurança entre 20% e 30%, dependendo da criticidade da aplicação. Para armazenamento de bancos de dados, também é importante considerar os IOPS (operações de entrada e saída por segundo).

Os recursos dimensionados podem ser traduzidos para recursos em nuvem com base no perfil das máquinas. Tomando como exemplo as instâncias da AWS:

- **Uso geral:**
  - _Família t (ex: t3, t4g)_: Ideal para cargas variáveis, leves e de baixo custo.
  - _Família m (ex: m6i, m8g)_: Balanceadas entre CPU, RAM e rede. Indicadas para aplicações web, APIs e bancos de dados de pequeno porte.
- **Otimizado para CPU:**
  - _Família c (ex: c6i, c8g)_: Alta capacidade de CPU. Ideal para análise, CI/CD, microsserviços e processamento em lote.
  - _Família hpc (ex: hpc7g, hpc6id)_: Projetadas para computação de alto desempenho (HPC), como simulações científicas, engenharia e CFD.
- **Otimizado para memória:**
  - _Família r (ex: r7i, r8g)_: Indicado para bancos de dados em memória (Redis), SAP HANA e aplicações Java com uso intensivo de heap.
  - _Família x (ex: x8g, x2idn)_: Ainda mais memória que a família r. Foco em bancos de dados grandes e cargas corporativas críticas.
- **Otimizado para armazenamento (Caso a opção Cloud native não seja a melhor):**
  - _Família i (ex: i8g, i7ie)_: Otimizadas para armazenamento SSD (NVMe). Excelentes para bancos NoSQL, Elasticsearch e big data.
  - _Família d (ex: d3, d3en)_: Grande capacidade de armazenamento magnético (HDD). Ideais para dados com leitura sequencial em grande volume.

Escolher corretamente os recursos na criação das instâncias é essencial para estimar custos. Contudo, é possível alterar as famílias de instância conforme a necessidade. A própria AWS disponibiliza o AWS Compute Optimizer, que analisa o uso real de instâncias EC2, volumes EBS, funções Lambda, grupos de Auto Scaling e Kubernetes Fargate, sugerindo:

- Troca de tipo de instância;

- Aumento ou redução de recursos com base na utilização real;

- Otimização para evitar superdimensionamento.

Para realizar a **escalabilidade** de instâncias EC2, utiliza-se o Auto Scaling da AWS, que pode ser configurado para aumentar ou reduzir instâncias automaticamente com base em métricas como CPU, memória, uso de rede, fila de requisições ou métricas personalizadas via CloudWatch. O uso de Elastic Load Balancer (ELB) distribui o tráfego entre instâncias de forma inteligente, trabalhando junto com o Auto Scaling e sendo essencial para escalabilidade horizontal.

Serviços da AWS que escalam automaticamente e facilitam a gestão do ambiente:

- **AWS Lambda:** Escala automaticamente até milhares de execuções simultâneas. Ideal para cargas irregulares ou orientadas a eventos (serverless).

- **Amazon ECS / EKS com Fargate:** Para containers e kubernetes, o Fargate escala automaticamente os serviços e gerencia a infraestrutura (serverless).

- **Amazon ElastiCache for Redis:** É um serviço gerenciado, compatível com Redis, projetado para oferecer baixa latência e alto throughput, alta disponibilidade com suporte a múltiplas zonas de disponibilidade (Multi-AZ), replicação e persistência de dados.

### b. Metodologia de FinOps

Com a utilização de parte da infra na nuvem haverá uma redução de CAPEX, eliminando o custo de aquisição de equipamentos e melhor controle de OPEX, pois será pago apenas pelo que usa, facilitando o planejamento financeiro.

Manter um ambiente on-premises inclui os custos da infraestrutura on-premise e também os custos do pessoal capacitado e habilitado para dar suporte aquela infra.

**Monitoramento Contínuo:** Utilização de ferramentas como AWS Cost Explorer para análise de custos em tempo real. Com o AWS Budgets é possível definir orçamentos personalizados para custos, uso ou reserva, ele envia alertas por e-mail ou SNS quando os limites são atingidos. O AWS Cost Anomaly Detection é um serviço baseado em Machine Learning que detecta variações inesperadas de custo e envia alertas automaticamente ao identificar anomalias em padrões de uso.

**Direcionamento de Workloads:** Aumentar o uso de serviços serverless (Fargate e Lambda) e reduzir o uso de infraestrutura gerenciada. A AWS e demais nuvens tem muitas facilidades que possibilitam reduzir custos, desde que se façam boas escolhas.

**Uso de Instâncias Reservadas e Spot:** Para as instâncias EC2 e RDS pode ser feito o aproveitamento de instâncias com desconto para workloads previsíveis e instâncias spot EC2 para processos não críticos.

- Instâncias Spot são até 90% mais baratas e boa para cargas temporárias, não críticas.

- Savings Plans / Reserved Instances: economia de até 72% para uso previsível.

**Otimização de Armazenamento:** Implementação de políticas de lifecycle para mover dados menos acessados para classes de armazenamento mais baratas.

**Rightsizing, Auto Scaling e Compute Optimizer:** Ajuste dinâmico dos recursos computacionais para evitar superprovisionamento. Análise do uso real das suas instâncias EC2, volumes EBS, funções Lambda, Auto Scaling groups e EKS Fargate. Sugere as opções de economia de custo, melhor desempenho e equilíbrio entre os dois. Preferência pelo uso de máquinas EC2 com processadores Graviton da AWS.

**Distribuição Geográfica:** Os recursos da nuvem tem custos diferentes dependendo da região. Por exemplo nas regiões dos Estados Unidos os recursos são mais baratos que na região do Brasil. Nesse caso, para economia sem afetar a performance, utiliza-se os ambientes de não-produção na região de Norte Virgínia por exemplo e o ambiente de produção no Brasil por questão de latência.

**Automação via IaC:** A automação via IaC contribui para o FinOps ao otimizar custos, garantindo o provisionamento eficiente de recursos e evitando desperdícios. Ela permite a escalabilidade sob demanda, o desligamento de recursos não utilizados e a aplicação automática de tags para controle financeiro. Além disso, oferece previsibilidade orçamentária e facilita a governança e auditoria das mudanças de infraestrutura, alinhando TI e finanças.

### c. Diagrama da Topologia

![Topologia da Solução - Cenário Atual](/Images/Topologia_da_Solução_Proposta_D1.png)



### d. Justificativas

A arquitetura foi pensada para garantir alta disponibilidade, escalabilidade, performance, segurança, facilidade de migração e otimização de custos.

#### 1. Computação
O uso de serviços serverless (Fargate e Lambda) é sugerido preferencialmente pois com eles temos escalabilidade e no geral um menor custo se comparado com serviços gerenciados (EC2 e ECS).

- **Fargate:** serve para aplicações em containers/kubernetes, escala automaticamente e sem a necessidade de gerenciar servidores.
- **Lambda:** serve quando pode-se alterar a aplicação para usar funções pequenas, que tem eventos rápidos, workloads imprevisíveis ou intermitentes.

**Vantagens:** Ambos os serviços permitem uma diminuição considerável da carga operacional. Esse é o melhor cenário em termo de custo a longo prazo. No caso do Fargate escala rapidamente horizontal e verticalmente.

**Desvantagens:** Exige mais tempo e esforço na migração que pode acarretar em custo maior de curto prazo, pois será necessário fazer algum tipo de desenvolvimento por parte da aplicação para se adaptar aos novos serviços.

- **EC2:** Foi sugerida a utilização das instâncias EC2 com Auto Scaling Groups. As EC2 são usadas para a parte da aplicação que precisa de controle total da infra

**Vantagens:** Executa workloads pesados ou que não são adequadas para ambientes serverless ou containers. É o caso da maioria das aplicações legadas e monolíticas. Usada com o Auto Scaling permitem escalabilidade automática, alta disponibilidade e performance otimizada.

**Desvantagens:** exige gerenciamento da infraestrutura, o cliente é responsável por configurar, atualizar, aplicar patches e manter o sistema operacional. Existe uma sobrecarga operacional com backups, logs, monitoramento, rede, storage. No geral são mais caras do que infraestrutura serveless.

- **AWS Compute Optimizer:** será usado na análise de toda a parte de compute Fargate, Lambda e EC2. Identifica instâncias e recursos superdimensionados, ajudando a economizar ao recomendar tamanhos menores mais adequados. Ferramenta gratuita para os principais recursos.

Para a parte da infraestrutura legada que não for compatível com recursos nuvem, é possível migrar lift-shift do jeito que estão para recursos dedicado na nuvem. Para esses casos é recomendado deixar on-premise, pois no geral levá-las para a nuvem costuma ficar mais caro.

#### 2. Banco de Dados
A definição de qual banco na nuvem deve ser usado depende muito do funcionamento da aplicação, do tamanho do banco, IOPs etc.. Foram sugeridos os seguintes bancos na nuvem:

- **Amazon ElastiCache for Redis:** será usado para receber o banco Redis da aplicação.

**Vantagens:** É um serviço gerenciado pela AWS, possui backups automáticos, aplicação de patches, atualizações e replicação feitos automaticamente. Suporte a Multi-AZ, réplicas de leitura e Global Datastore para replicação entre regiões. Milhões de requisições por segundo com baixa latência (microsegundos). Integração com VPC, IAM, KMS (criptografia em trânsito e em repouso), e ACLs.

**Desvantagens:** Pode ser mais caro do que manter Redis autogerenciado, especialmente em ambientes pequenos ou não críticos. Atualizações maiores podem exigir intervenção manual e reinicializações.

- **RDS para Oracle e para SQL Server:** Foi sugerido para as aplicações que tem baixa latência e precisam que os bancos Oracle e SQLServer estejam na nuvem também.

**Vantagens:** É um serviço de gerenciamento simplificado em que a AWS cuida de patching, backups automáticos, replicação, failover e atualizações menores do banco. Suporte nativo à replicação e failover automático para maior disponibilidade

**Desvantagens:** Custo elevado devido ao licenciamento Oracle e SQL Server. No caso desses bancos é necessário avaliar o custo em outras nuvens como OCI e Azure, e verificar se não é melhor deixá-los on-premises.

#### 3. Integração
Serviço gerenciado de fila de mensagens que permite a comunicação assíncrona entre diferentes componentes de um sistema distribuído. Ele atua como um intermediário confiável que garante que mensagens enviadas por uma parte do sistema sejam entregues a outra parte, mesmo que essa segunda parte esteja temporariamente indisponível. Necessário para atender ao requisito não funcional. Além disso, é fundamental adotar práticas como escalabilidade horizontal, alta disponibilidade, circuit breakers para resiliência, e monitoramento proativo para garantir a continuidade e resposta rápida a falhas, mantendo a operação sem interrupções.

- **SQS:** Garante a entrega confiável de mensagens mesmo com falhas momentâneas. Armazena mensagens de forma redundante em várias zonas de disponibilidade (AZs), garantindo alta confiabilidade. Suporta milhões de mensagens por segundo sem necessidade de configuração manual.

#### 5. Infraestrutura on-premises
Algumas das aplicações legadas e monolíticas terão que ficar no ambiente on-premises. 

- **Instâncias de Computação:** Aplicações que estão obsoletas, com sistema operacional legado, que dependem de hardware específico ou tem restrições regulatórias ficarão no DataCenter.
- **Banco de Dados:** Banco de dados como o Oracle e SQLServer que possuem licenças proprietárias são mais vantajosos em on-premises. Pode ser necessário também por questão de desempenho intensivo de I/O local e requisitos de conformidade, em que dados sensíveis devem permanecer em datacenters próprios por exigência legal (LGPD, HIPAA, etc.).



### e. Estratégia de Automação

O uso de uma ferramenta de IaC é altamente recomendado, pois a Infraestrutura pode ser provisionada automaticamente e de forma consistente. Evita erros manuais e garante que ambientes sejam sempre criados da mesma forma (ex: dev, staging, prod).

É possível usar o Terraform para definir regras de escalabilidade como código (Infrastructure as Code - IaC), permitindo controle de versionamento e automação do dimensionamento na AWS (ou em outros provedores). A escolha do Terraform frente a outros serviços de IaC se dá por ele ser um provedor que se conecta com qualquer outro provedor de nuvem além da AWS. Ou seja, uma vez que os códigos forem feitos é possível criar a infra igual em qualquer outra nuvem.

A melhor ferramenta é a Terraform pois ela suporta múltiplos provedores (AWS, Azure, GCP, VMware, etc.), tem Linguagem declarativa (HCL) fácil de ler, possui grande comunidade e documentação. Dentre as vantagens de usar a IaC tem-se:

1. **Provisionamento Automatizado:** Utilização de código para criar, modificar e destruir recursos na nuvem e on-premises.
2. **Versionamento e Reutilização:** Manutenção do código em repositórios Git para controle de versão e reutilização de módulos.
3. **Segurança e Compliance:** Aplicação de políticas de segurança e conformidade automatizadas (Terraform Sentinel, AWS Config, Azure Policy).
4. **Escalabilidade Dinâmica:** Implementação de Auto Scaling Groups e Kubernetes HPA para ajustar recursos conforme a demanda.
5. **Monitoramento e Logging:** Integração com Prometheus, Grafana e CloudWatch para observabilidade contínua.
6. **Orquestração com CI/CD:** Uso de Terraform Cloud/Enterprise ou pipelines (GitHub Actions, GitLab CI, Jenkins) para automação de deploys.

## 3. Requisitos diferenciais
### a. Plano de Disaster Recovery (DR):

Ter uma infraestrutura de Disaster Recovery (DR) bem planejada traz diversas vantagens estratégicas e operacionais para a empresa. Os principais benefícios são:

1. **Continuidade dos negócios:** Minimiza o tempo de inatividade em caso de falhas, garantindo que os serviços essenciais continuem funcionando.
2. **Restauração rápida:** Permite recuperação rápida de dados e sistemas, reduzindo o impacto financeiro e operacional.
3. **Proteção contra perda de dados:** Garante que backups e réplicas estejam disponíveis para restaurar informações críticas.
4. **Redução de perdas financeiras:** Minimiza prejuízos relacionados à paralisação de sistemas, perda de vendas ou multas contratuais.
5. **Maior segurança e conformidade:** Atende a requisitos legais e regulatórios sobre continuidade de serviços e proteção de dados.
6. **Ambiente de testes e simulações:** DR permite testar atualizações e mudanças em um ambiente isolado, sem afetar a produção.

Para a infraestrutura de Disaster Recovery foi desenhada uma topologia Hot Site em que o ambiente de recuperação está em funcionamento constante, com dados e sistemas atualizados em tempo real. Nesse modelo tem-se o RTO (Objetivo de Tempo de Recuperação) e RPO (Objetivo de Ponto de Recuperação) muito baixos (quase imediatos). A escolha do modelo depende claro da criticidade de RTO e RPO necessários pela aplicação. Segue a topologia:

![Topologia da Solução - Cenário Atual](/Images/Topologia_da_Solução_Disaster_Recovery_Desafio.png)

### b. Monitoramento e Observabilidade
Ferramentas como CloudWatch e CloudTrail garantem monitoramento e auditoria centralizados.

- **CloudWatch:** Coleta métricas de uso (CPU, memória, disco, rede, etc.) de serviços AWS e aplicações. Cria alarmes para reagir automaticamente a eventos (ex: ativar Auto Scaling ou enviar alertas via SNS). É possível criar métricas personalizadas para serem usadas pelo Terraform.
- **CloudTrail:** Registra todas as chamadas de API feitas na conta, incluindo o que foi feito, por quem, e quando. Permite saber exatamente quem criou, alterou ou excluiu recursos (e de onde). É possível configurar via Terraform para enviar os logs para S3.

### c. Modelo OSI e Segurança

1. **Camada 1 – Física:**	Meio físico de transmissão (cabos, switches, energia, etc).	Segurança física dos data centers da nuvem e proteção contra acesso físico não autorizado. Refletido nos controles físicos das certificações ISO 27001, SOC 1, SOC 2 e SOC 3.

2. **Camada 2 – Enlace de Dados:**	Comunicação entre dispositivos na mesma rede.	Gerenciado pela provedor dos serviços cloud. Proteção lógica contra manipulação de frames e MAC spoofing. Usado em redes privadas virtuais (VPCs).

3. **Camada 3 – Rede:**	Endereçamento IP, roteamento.	Definição de IPs de origem/destino, rotas e segmentação de redes com VPCs e subnets. Segurança via ACLs (Access Control Lists), firewalls e controle de tráfego.

4. **Camada 4 – Transporte:**	Transporte confiável (TCP/UDP), portas.	Controle das portas e protocolos (ex: 443 para HTTPS). Segurança com firewalls do provedor da nuvem. Porém a definição dos security groups é feito pelo cliente.

5. **Camada 5 – Sessão:**	Estabelecimento e controle de sessões. Autenticação e controle de sessões entre serviços (por exemplo, IAM, autenticação com tokens, sessões seguras TLS).

6. **Camada 6 – Apresentação:**	Criptografia, compressão, tradução de dados. Uso de TLS/SSL para criptografia em trânsito. Utilização de KMS ou HSM para criptografia em repouso.

7. **Camada 7 – Aplicação:**	Interface de usuário e aplicações.	Proteção com AWS WAF, autenticação por IAM, regras de acesso por grupos, validação de entrada para prevenir XSS e SQL Injection. Também relacionada à definição de privilégios de usuários/grupos por serviço.

Definir claramente como será a comunicação entre os serviços para liberação dos acessos na Cloud e On-premise. Levantar as informações conforme abaixo:

| IP de Origem | IP de Destino | Protocolo | Porta / Serviço | Ambiente | Descrição |
| --- | --- | --- | --- | --- | --- |
| _Ex: 10.10.10.10_ | _Ex:_<br><br>_10.10.10.11_ | _Ex: TCP/UDP)_ | _Ex: 8080/443_ | _Ex: PRD/HML/DEV_ | _Ex: acesso web para o time XPTO_ |

Cada serviço deverá seguir a premissa do menor acesso, sendo assim deve ser definido quem irá ter acesso a cada serviço e o tipo de privilégio. Priorizando a utilização sempre de grupos de acessos ao invés de usuários.

| Serviço | Usuário/Grupo | Responsável | Tipo de Acesso |
| --- | --- | --- | --- |
| _Ex: EC2_ | _Ex: admso, web, deploy_ | _Ex: Suporte Unix, Windows_ | _Ex: Root, Web, Banco_ |

Vantagens do AWS WAF (Web Application Firewall):

1. **Proteção contra ameaças comuns na camada de aplicação:** O AWS WAF protege contra ataques como injeção de SQL, cross-site scripting (XSS), bots maliciosos e ataques de negação de serviço (DDoS) direcionados a aplicações.
2. **Regras personalizáveis:** É possível definir regras específicas com base em padrões de tráfego, como IPs de origem, strings nos headers, URIs, geolocalização, entre outros.
3. **Monitoramento em tempo real:** O AWS WAF fornece métricas e logs de acesso em tempo real por meio da integração com o Amazon CloudWatch, permitindo análises rápidas e detalhadas.
4. **Integração com serviços da AWS:** O WAF pode ser utilizado em conjunto com serviços como o Amazon CloudFront, oferecendo proteção ampla para aplicações distribuídas.
5. **Regras gerenciadas:** A AWS disponibiliza conjuntos de regras gerenciadas, mantidas e atualizadas automaticamente, que ajudam a proteger contra novas ameaças sem a necessidade de intervenção manual.

## Considerações Finais
A proposta apresentada resultou em uma solução arquitetural robusta para a migração de um sistema legado para um modelo híbrido (on-premises + cloud), atendendo aos principais objetivos definidos:  escalabilidade e flexibilidade.

A partir da compreensão dos objetivos de negócio e das motivações para a migração, foi possível estruturar uma arquitetura capaz de modernizar o ambiente tecnológico, proporcionando maior disponibilidade, escalabilidade, segurança, controle de acesso, otimização de custos, resiliência e recuperação (DR), automação e governança.

A avaliação da infraestrutura atual — incluindo servidores, redes, sistemas de armazenamento e bancos de dados — permitiu identificar os recursos mais adequados a serem migrados ou mantidos localmente. A análise das interdependências entre os componentes também foi essencial para garantir uma transição segura e eficiente.

Com base nessas informações, a arquitetura da solução foi dimensionada, considerando o consumo de recursos das máquinas atuais e comparando com os serviços disponíveis nas principais nuvens públicas. Embora tenha sido utilizada a AWS como referência para o desenho da topologia TO-BE, a solução permanece aplicável a outras plataformas, conforme as necessidades e restrições de cada organização. Caso o critério de escolha da nuvem seja o menor custo, o comparativo pode ser feito através da calculadora pública das nuvens existentes ([AWS](https://calculator.aws/), [Azure](https://azure.microsoft.com/en-us/pricing/calculator/), [OCI](https://cloud.oracle.com/calc), [Google Cloud](https://cloud.google.com/products/calculator), dentre outras).

A proposta também incluiu o uso de IaC com Terraform, viabilizando a automação da infraestrutura e tornando o gerenciamento mais ágil, seguro e reprodutível, tanto no ambiente local quanto na nuvem.

Em cada etapa do projeto, as escolhas tecnológicas foram devidamente justificadas, com análise de vantagens e desvantagens, contribuindo para uma solução arquitetural sólida, escalável e preparada para o futuro.

