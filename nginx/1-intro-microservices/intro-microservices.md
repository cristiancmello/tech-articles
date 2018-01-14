# 1. Introdução aos Microservices
* Apesar de todo o entusiasmo e ceticismo a cerca da ideia de Microservices, 
esse padrão arquitetural tem benefícios significativos - especialmente quando 
se trata em habilitar o desenvolvimento ágil e entrega de apps corporativos 
complexos.

* Neste artigo, são tratadas as diferenças básicas entre as arquiteturas de 
**Microservices** e a **Monolítica**, que é um padrão mais tradicional.

* Também serão tratadas as **vantagens** e **desvantagens** do padrão 
de **Microservices**.

## Construindo Aplicações Monolíticas
Vamos imaginar que iremos construir uma aplicação de táxi semelhante ao Uber.
Após reuniões preliminares de coleta de requisitos, construiremos um novo 
projeto centralizado usando uma framework, como Laravel, Rails, Spring Boot 
entre outros.

Percebe-se que todas as funcionalidades irão ficar agregadas num mesmo 
repositório, expondo adapters (componentes de acesso ao BD, componentes de 
mensagens que produzem e consomem mensagens), APIs REST e Interfaces de Usuário. 

Apesar de ter uma estrutura logicamente modular, o aplicativo é empacotado 
e implantado como um monólito (algo único, coeso).

Aplicações escritas nessa forma são extremamente comuns. Elas são simples de 
se desenvolver uma vez que os ambientes se focam na construção de uma única 
aplicação. Os testes também são mais simples.

Aplicações monolíticas são mais simples de se implementar e implantar 
(Deploy). É necessário apenas copiar o pacote da aplicação para um servidor. 
Podemos também dimensionar o aplicativo executando várias cópias atrás de 
um **balanceador de carga** (**load balancer**). Inicialmente, essa arquitetura 
funciona bem.

## Marchando para o Monolithic Hell
À medida que a aplicação cresce ao longo do tempo, a abordagem monolítica 
começa a trazer muitas limitações. A cada Sprint (ciclo de desenvolvimento 
do Scrum), a equipe de desenvolvimento implementa mais histórias 
(funcionalidades), o que, naturalmente, significa adicionar mais linhas 
de código. Após alguns anos, a aplicação se tornará um monstruoso monólito, 
havendo-se muitas dependências, riscos de migração elevados e etc.

Uma vez que a aplicação se tornou grande e complexa, a organização de 
desenvolvimento provavelmente estará cheia de atrasos e muito sacrifício. 
Quaisquer tentativas de desenvolvimento ágil e entrega serão a um grande 
custo. Entender o relacionamento entre as partes do sistema também será 
muito difícil, o que acarreta em correções de bugs mais demoradas e com 
milhares de efeitos colaterais indesejados.

Se a base de código for difícil de entender, as alterações não serão 
feitas corretamente. Logo, acabaremos numa enorme **bola de lama**.

Outro problema da aplicação será a dimensão do consumo de recursos. 
Quanto maior o aplicativo, maior o tempo de inicialização. 
Em uma pesquisa recente 
([link](https://plainoldobjects.com/2015/05/13/monstrous-monoliths-how-bad-can-it-get/)), 
alguns desenvolvedores relataram tempo de inicialização de até 12 minutos. 
Já se foi noticiado inicializações maiores do que 40 minutos. Isso acarreta 
em desperdício de tempo em razão da demora ao se operar regularmente a aplicação.

Em relação à **Implantação Contínua** (relacionada à **Integração Contínua**), 
a aplicação monolítica grande e complexa será um obstáculo. 
Hoje, o estado da arte para aplicações SaaS (Software como serviço) é realizar 
deploys em produção muitas vezes ao dia. Tomando como base uma aplicação 
monolítica complexa, isso será extremamente difícil de se fazer, requerendo, 
muita vezes, extensos testes manuais.

Outra problemática trazida pelas aplicações monolíticas é na dificuldade em 
escalonamento quando se é preciso diferentes módulos que exigem recursos 
computacionais conflitantes. Exemplo: um módulo pode implementar um lógica 
de processamento de imagem com uso intensivo de CPU/GPU e idealmente ser 
implantado em instâncias otimizadas da AWS (Amazon Web Services). Outro 
módulo pode ser um banco de dados em memória e mais adequado para instâncias 
EC2 otimizadas para a memória. Como os módulos foram implementados juntos, 
a escolha de hardware foi comprometida.

A confiabilidade em aplicações monolíticas também é um outro problema. 
Como todos os módulos estão sendo executados no mesmo processo, um bug 
em qualquer módulo, como vazamento de memória, tem o poder de congelar 
a aplicação inteira.

Um último ponto desfavorável à abordagem de microarquitetura é a dificuldade
em adoção de novas tecnologias e frameworks mais modernas. Além disso, novos 
desenvolvedores necessitarão de um maior tempo de treinamento na tecnologia 
de implantação e encontrarão muitas resistências ao se propor mudanças na 
aplicação.

## Microservices - enfrentando a complexidade
Muitas organizações, como **Amazon**, **eBay** e **Netflix**, resolveram esse 
problema utilizando a abordagem do padrão do Microservices. Em vez de construir 
uma única aplicação monstruosa e monolítica, a ideia é dividir a aplicação 
em um conjunto de serviços menores e interconectados.

Um serviço tipicamente implementa um conjunto de funcionalidades, tais como:
* **Gestão de Clientes**;
* **Gestão de Pedidos**;
* ...

Cada microserviço é uma mini-aplicação que tem sua própria arquitetura 
monolítica que consiste em lógica de negócio juntamente com vários 
adaptadores. Alguns microservices exporiam uma **API (Interface de Programação 
de Aplicação)** que é consumida por outros microservices ou pelos clientes 
da aplicação. Outros microservices podem implementar uma interface Web. 
Em tempo de execução, cada instância é geralmente uma nuvem em VM 
(Máquina Virtual) ou container do Docker.

Abaixo, há uma possível decomposição funcional da aplicação:

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/1-intro-microservices/nginx-article-1.png" width="500" height="500">
</p>

Cada área funcional da aplicação é agora implementada por seu próprio 
microserviço. Além disso, a aplicação Web é dividida em um conjunto de 
aplicações Web mais simples (como uma para passageiros e outro para 
motoristas). Isso facilita a implantação de experiências distintas 
para usuários específicos, dispositivos ou casos de uso especializados.

Cada serviço de backend (lado servidor) expõe uma API REST e a maioria 
dos serviços consome APIs fornecidas por outros serviços. Por exemplo, 
o **Gerenciamento de Corridas** usa o servidor de notificação para 
informar uma corrida disponível. Os serviços de interface do usuário 
invocam os outros serviços para renderizar páginas Web. Os serviços 
também podem usar comunicação assíncrona baseada em mensagens. 
A comunicação entre serviços será abordada mais detalhadamente em 
outro capítulo.

Algumas APIs REST também são expostas aos aplicativos móveis usados 
pelos motoristas e passageiros. As aplicações, no entanto, não tem 
acesso direto aos serviços de backend. Em vez disso, a comunicação 
é intermediada por um componente chamado **API Gateway**. 
O API Gateway é responsável por tarefas como balanceamento de carga, 
cache, controle de acesso, medição de API e monitoramento, e **pode 
ser implementado efetivamente usando o NGINX**. Capítulos posteriores 
irão cobrir o API Gateway.

### Arquitetura de Microserviço e o Cubo da Escalabilidade
O padrão de **Arquitetura de Microserviços** corresponde ao **eixo Y** 
no **Cubo da Escalabilidade** (*Scale Cube*), que é um modelo 3D de
escalabilidade do livro *The Art of Scalability*, dos autores 
Martin L. Abbott e Michael T. Fisher (2ª Ed., Addison-Wesley, 2015). 
Os outros eixos de dimensionamento são escalonamento de **eixo X**, 
que consiste em várias cópias idênticas do aplicativo por trás de 
um balanceador de carga e o escalonamento de **eixo Z** (ou 
particionamento de dados), onde um atributo da solicitação (como 
chave-primária) é usado para encaminhar a solicitação para um servidor 
específico.

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/1-intro-microservices/nginx-article-2.png" width="500" height="380">
</p>

As aplicações costumam utilizar os três tipos de escalonamento em conjunto:
* **Eixo Y**: decompõem a aplicação em microserviços;
* **Eixo X**: em tempo de execução, executam várias instâncias (clones) 
de cada serviço atrás de um balanceador de carga (*load balancer*);
* **Eixo Z**: usado por algumas aplicações para particionar serviços, como 
Banco de Dados.

A seguir, um exemplo de serviço de **Gerência de Viagem** implementado com 
o Docker na AWS EC2.

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/1-intro-microservices/nginx-article-3.png" width="500" height="500">
</p>

### Relação da Arquitetetura de Microserviço e o Banco de Dados
A Arquitetura de Microserviço afeta significativamente a relação entre 
o aplicativo e o Banco de Dados. Em vez de compartilhar um único 
esquema do BD com outros serviços, cada serviço tem seu próprio esquema 
do Banco de Dados. Por um lado, esta abordagem está em desacordo com a 
ideia de um modelo de dados para toda a empresa. Além disso, muitas vezes 
resulta na duplicação de alguns dados. No entanto, ter um esquema de 
Banco de Dados por serviço é essencial se você quiser se beneficiar 
de microservices, porque garante acoplamento frouxo. O diagrama a seguir 
mostra a arquitetura do BD para o aplicativo de exemplo.

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/1-intro-microservices/nginx-article-4.png" width="500" height="350">
</p>

Cada um dos serviços tem seu próprio Banco de Dados. Além disso, um 
serviço pode usar um tipo de BD que é mais adequado às suas necessidades, 
a chamada **Arquitetura de Persistência Polyglot**. Por exemplo, a Gestão 
de Corridas, que localiza corridas próximas a um passageiro em potencial, 
deve usar um Banco de Dados que ofereça suporte a geo-consultas eficientes.

Há também muita semelhança entre a Arquitetura de Microserviços com o 
**SOA** (**Orientado a Serviços**). Ambas as abordagens consistem 
num conjunto de serviços. A diferença entre as duas arquiteturas 
consiste que o SOA tem caráter comercial (geralmente atrelado a 
uma tecnologia específica fornecida por uma única empresa como um 
"pacote"), além de uma especificação como um Web Service (WSDL, 
SOAP e etc) e um ESB (Enterprise Service Bus). As aplicações baseadas 
em Microservices favorecem protocolos mais simples e leves, como o 
REST, ao invés de Web Service. O padrão de Microserviço rejeita outras 
partes do SOA, como o conceito de um esquema canônico.

## Os Benefícios dos Microserviços
O padrão de Microservices tem uma série de benefícios importantes. 
Primeiro, aborda o problema da complexidade. Ele decompõe uma monstruosa 
aplicação monolítica em conjunto de serviços. Embora a quantidade total 
de funcionalidades seja inalterada, o aplicativo foi dividido em blocos 
ou serviços gerenciáveis. Cada serviço tem um limite bem definido na forma 
de um RPC (Chamada à Procedimento Remoto) ou API orientada a mensagens. 
O padrão de arquitetura de Microservices impõe um nível de modularidade 
que na prática é extremamente difícil de alcançar com uma base de código 
monolítico. Consequentemente, os serviços individuais são muito mais rápidos 
para desenvolver e muito mais fáceis de entender e manter.

Outras considerações:
* Cada serviço pode ser desenvolvido por equipes independentes;
* Desenvolvedores são livres para escolher qualquer tecnologia, desde que 
honre o contrato da API;
* Como serviços são relativamente pequenos, torna-se possível reescrever 
um serviço antigo usando a tecnologia atual;
* Cada serviço pode ser implantado de forma independente;
* É possível realizar a implantação contínua de maneira mais rápida.
* **Cada serviço pode ser dimensionado de forma independente**. Isto 
é, pode ser satisfeito de acordo com um hardware específico que 
melhor corresponda aos requisitos de um serviço.

## Os inconvenientes dos microserviços
* *A Arquitetura de Microservices não é a solução para tudo*;
* Uma das desvantagens é na ênfase excessiva ao tamanho do serviço. Alguns 
desenvolvedores dimensionam de modo a serem bem pequenos.
    * **Importante lembrar que os microserviços são um meio para o fim e não o
    objetivo principal**;
* **Objetivo principal dos microserviços: decompor suficientemente a aplicação 
para facilitar o desenvolvimento e implantação de aplicativos ágeis**.

* Outro grande inconveniente: complexidade que surge do fato de que a aplicação
é um sistema distribuído (vários serviços interligados)
    * É muitas vezes preciso implementar mecanismos de comunicação entre processos
    baseados em mensagens ou RPC (Chamada a Procedimento Remoto);

    * Além disso, é preciso escrever código para lidar com falhas parciais,
    uma vez que o destino de uma solicitação pode ser lento ou estar indisponível.
    
* Outro desafio é a arquitetura particionada de Banco de Dados. Transações comerciais,
por exemplo, exigem atualização de várias entidades de negócio. Numa arquitetura 
monolítica, isso é trivial. Já numa aplicação de microserviços, é preciso atualizar
vários bancos de dados pertencentes a diferentes serviços.
    * Transações distribuídas, geralmente, não é uma opção, pois muitos Bancos de 
    Dados não irão suportá-las, como os NoSQL. Assim, por vezes é necessário adotar
    uma estratégia baseada em consistência, o que é ainda mais desafiador para a 
    codificação.

* O teste da aplicação de microserviços é também bastante complexo. Uma classe 
de teste semelhante para um serviço precisaria lançar o serviço e quaisquer
outros dependentes a fim de se montar uma cenário de teste.

* Implementar mudanças que abrangem vários serviços também é um grande desafio.
É comum pensar numa estratégia de rollout de alterações em cada um dos serviços.
No entanto, é raro uma grande coordenação para alterar alguns serviços.

* Deploy (implantação) da aplicação é mais complexa. Será preciso implementar
um mecanismo de descoberta de serviço (será discutido posteriormente) que
irá permitir a um serviço descobrir locais (hosts e portas) de quaisquer
outros serviços com os quais ela precisa se comunicar.
    - Abordagem baseada em *tickets* e operações manuais podem não escalar
    muito bem para este nível de complexidade;
    - Consequentemente, implementar com êxito um aplicativo de microserviços
    requer um maior controle dos métodos de implantação pelos desenvolvedores
    e um nível alto de automação.

### Abordagem PaaS (Plataforma como Serviço)
* Uma abordagem para automação da aplicação de microserviços é usar um *PaaS*,
como o **Cloud Foundry**.
    * Um PaaS fornece aos desenvolvedores uma maneira fácil de implantar e
    gerenciar seus microserviços. Ele os isola de preocupações como
    aquisição e configuração de recursos computacionais.

* Outra abordagem com PaaS é desenvolver um próprio, utilizando uma solução
de clustering, como o **Kubernetes**, em conjunto com o **Docker**
