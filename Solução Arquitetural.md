**Solução Arquitetural para Migração de Sistema Legado para Modelo Híbrido**

Autor: Evandro da Costa Vieira
Telefone: (61) 999357026
Email: <evandro.costajr@gmail.com>
<https://www.linkedin.com/in/evandro-da-costa-vieira-6160a3178/>

**Introdução**
Esta proposta apresenta uma solução arquitetural para a migração de um sistema legado para um modelo híbrido (on-premises + cloud).

Para o desenvolvimento do desenho da arquitetura, é imprescindível, inicialmente, a compreensão dos objetivos de negócio, bem como a análise das motivações para a migração de parte do sistema para a nuvem. As razões mais comuns para essa migração incluem a redução de custos, aumento de escalabilidade, flexibilidade, modernização de sistemas e melhoria no desempenho. Neste projeto, foram considerados como principais objetivos a escalabilidade e a otimização de custos.

Será realizada uma avaliação detalhada dos recursos e da arquitetura existentes, com o objetivo de compreender como a infraestrutura local está configurada, incluindo servidores, sistemas de armazenamento, aplicativos, bancos de dados, redes, monitoramento, backup, entre outros. Além disso, serão identificadas as dependências e interconexões entre os aplicativos, bancos de dados, redes e sistemas, bem como um levantamento das licenças e dos custos relacionados à infraestrutura atual.

Somente após a avaliação da infraestrutura existente será possível dimensionar os recursos a serem criados na nuvem e definir a topologia da solução que atenderá aos requisitos do sistema. Para este trabalho, foi considerada uma arquitetura genérica, que está descrita na seção "**Cenário Atual**".

Com a topologia definida, será realizado o **dimensionamento dos recursos** a serem provisionados na nuvem. Será avaliado o consumo de CPU, memória e armazenamento das máquinas existentes fazendo um comparativo com os serviços disponíveis na nuvem. Para a definição do serviço da nuvem a ser usado, é importante ter definido a melhor nuvem que atenderá a solução. Caso o critério de escolha da nuvem seja o menor custo, o comparativo pode ser feito através da calculadora pública das nuvens existentes ([AWS](https://calculator.aws/), [Azure](https://azure.microsoft.com/en-us/pricing/calculator/), [OCI](https://cloud.oracle.com/calc), [Google Cloud](https://cloud.google.com/products/calculator), dentre outras).

Neste projeto foi considerado os recursos da nuvem AWS para representação na **topologia da solução**, porém eles podem ser substituídos pelos recursos equivalentes de qualquer uma das outras nuvens. Tal escolha foi feita para facilitar a representação e permitir também uma visão dos custos do projeto para definição das melhores **estratégias de FinOps** afim de otimizar os custos.

Por fim, foi abordada também a utilização de **automação via IaC** (Infraestrutura como Código) usando o Terraform. O uso de IaC permite a gestão automatizada de recursos tanto na nuvem quanto on-premises, por meio de código. No projeto, será detalhado onde e como a IaC pode ser aplicada para melhorar o gerenciamento da infraestrutura. Em cada seção do projeto, foram justificadas as escolhas tecnológicas, apresentando suas vantagens e desvantagens.

**Cenário Atual**
O sistema atual proposto é composto por tecnologias legadas, que foram projetadas e implantadas há um longo período. Esses sistemas apresentam diversas dificuldades, especialmente em termos de flexibilidade, escalabilidade, manutenção e custos.

A aplicação utiliza servidores físicos antigos, com hardware desatualizado, apresentando limitada capacidade de escalabilidade e dificultando a manutenção. As máquinas dedicadas e compartilhadas da aplicação estão hospedadas em data centers locais, o que resulta em elevados custos de manutenção, energia e espaço físico. O armazenamento também é realizado localmente, em unidades de armazenamento obsoletas.

O sistema é predominantemente composto por servidores Linux, mas também inclui servidores Windows. Possui um ambiente de produção, homologação e desenvolvimento sendo os ambientes de não produção equivalente a 20% do tamanho da produção.

Os bancos de dados relacionais (ex.: Oracle e SQL Server) estão hospedados em servidores físicos dedicados e, por questões de licenciamento e conformidade regulatória, deverão ser mantidos no data center local. Já os bancos de dados NoSQL (ex.: MySQL e PostgreSQL) também estão em servidores físicos dedicados, mas não possuem restrições de conformidade regulatória.

**Desafios da arquitetura atual:**
Processos manuais de configuração: A dependência de processos manuais dificulta a automação e o gerenciamento eficiente de grandes volumes de servidores e componentes.

Falta de orquestração: A ausência de orquestração limita a flexibilidade e a capacidade de escalar a infraestrutura de forma eficiente.

Suporte limitado a tecnologias legadas: Os fornecedores de hardware e software frequentemente descontinuam produtos antigos, o que torna a solução mais difícil de manter e atualizações se tornam mais complexas.

Elevados custos de infraestrutura: O uso de múltiplos servidores físicos resulta em custos altos de manutenção e operação.

Vulnerabilidades de segurança: Sistemas legados tornam-se obsoletos, dificultando a integração com novas tecnologias e atualizações, o que eleva os riscos de segurança.


![Topologia da Solução - Cenário Atual](/Topologia_da_Solução_CenarioAtual.png)

Dimensionamento de Recursos

**Objetivos**
\-Modernizar o sistema legado sem interrupção do serviço.
\-Aproveitar os recursos da nuvem para escalabilidade, disponibilidade e eficiência operacional.
\-Reduzir custos operacionais a longo prazo.
\-Melhorar a segurança e o desempenho.

**Arquitetura Proposta**
A arquitetura desenhada leva em consideração as premissas de escalabilidade e a otimização de custos como prioritárias para atender o cliente. Essas premissas podem mudar de acordo com a necessidade do cliente, porém a mudança delas pode impactar parte ou toda a solução proposta.A estratégia de migração adotada dado uma infraestrutura híbrida será a Replatform (lift-tinker-and-shift) em que consideramos a migração da infraestrutura como está e faremos pequenas modificações para melhor performance. A ideia é que após a migração com a utilização da nuvem seja feito o redesenho da aplicação para aproveitar o máximo da nuvem, deixando toda a aplicação na nuvem.
```
> Written with [StackEdit](https://stackedit.io/).
