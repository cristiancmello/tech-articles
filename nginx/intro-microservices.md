# Introdução aos Microservices
* Apesar de todo o entusiasmo e ceticismo a cerca da ideia de Microservices, esse padrão arquitetural tem benefícios significativos - especialmente quando se trata em habilitar o desenvolvimento ágil e entrega de apps corporativos complexos.

* Neste artigo, são tratadas as diferenças básicas entre as arquiteturas de **Microservices** e a **Monolítica**, que é um padrão mais tradicional.

* Também serão tratadas as **vantagens** e **desvantagens** do padrão de **Microservices**.

## Construindo Aplicações Monolíticas
Vamos imaginar que iremos construir uma aplicação de táxi semelhante ao Uber. Após reuniões preliminares de coleta de requisitos, construiremos um novo projeto centralizado usando uma framework, como Laravel, Rails, Spring Boot entre outros.

Percebe-se que todas as funcionalidades irão ficar agregadas num mesmo repositório, expondo adapters (componentes de acesso ao BD, componentes de mensagens que produzem e consomem mensagens), APIs REST e Interfaces de Usuário. 

Apesar de ter uma estrutura logicamente modular, o aplicativo é empacotado e implantado como um monólito (algo único, coeso).

Aplicações escritas nessa forma são extremamente comuns. Elas são simples de se desenvolver uma vez que os ambientes se focam na construção de uma única aplicação. Os testes também são mais simples.

Aplicações monolíticas são mais simples de se implementar e implantar (Deploy). É necessário apenas copiar o pacote da aplicação para um servidor. Podemos também dimensionar o aplicativo executando várias cópias atrás de um **balanceador de carga** (**load balancer**). Inicialmente, essa arquitetura funciona bem.

## Marchando para o Monolithic Hell
À medida que a aplicação cresce ao longo do tempo, a abordagem monolítica começa a trazer muitas limitações. A cada Sprint (ciclo de desenvolvimento do Scrum), a equipe de desenvolvimento implementa mais histórias (funcionalidades), o que, naturalmente, significa adicionar mais linhas de código. Após alguns anos, a aplicação se tornará um monstruoso monólito, havendo-se muitas dependências, riscos de migração elevados e etc.

Uma vez que a aplicação e tornou grande e complexa, a organização de desenvolvimento provavelmente estará cheia de atrasos e muito sacrifício. Quaisquer tentativas de desenvolvimento ágil e entrega serão a um grande custo. Entender o relacionamento entre as partes do sistema também será muito difícil, o que acarreta em correções de bugs mais demoradas e com milhares de efeitos colaterais indesejados.

Se a base de código for difícil de entender, as alterações não serão feitas corretamente. Logo, acabaremos numa enorme **bola de lama**.

Outro problema da aplicação será a dimensão do consumo de recursos. Quanto maior o aplicativo, maior o tempo de inicialização. Em uma pesquisa recente ([link](https://plainoldobjects.com/2015/05/13/monstrous-monoliths-how-bad-can-it-get/)), alguns desenvolvedores relataram tempo de inicialização de até 12 minutos. Já se foi noticiado inicializações maiores do que 40 minutos. Isso acarreta em desperdício de tempo em razão da demora ao se operar regularmente a aplicação.

Em relação à **Implantação Contínua** (relacionada à **Integração Contínua**), a aplicação monolítica grande e complexa será um obstáculo. Hoje, o estado da arte para aplicações SaaS (Software como serviço) é realizar deploys em produção muitas vezes ao dia. Tomando como base uma aplicação monolítica complexa, isso sera extremamente difícil de fazer, requerendo, muita vezes, extensos testes manuais.

Outra problemática trazida pelas aplicações monolíticas, é na dificuldade em escalonamento quando se é preciso diferentes módulos que exigem recursos computacionais conflitantes. Exemplo: um módulo pode implementar um lógica de processamento de imagem com uso intensivo de CPU/GPU e idealmente ser implantado em instâncias otimizadas da AWS. Outro módulo pode ser um banco de dados em memória e mais adequado para instâncias EC2 otimizadas para a memória. Como os módulos foram implementados juntos, a escolha de hardware foi comprometida.

A confiabilidade em aplicações monolíticas também é outro problema. Como todos os módulos estão sendo executados no mesmo processo, um bug em qualquer módulo, como vazamento de memória, tem o poder de congelar a aplicação inteira.

Um último ponto desfavorável à abordagem de microarquitetura, é a dificuldade em adoção de novas tecnologias e frameworks mais modernas. Além disso, novos desenvolvedores necessitarão de um maior tempo de treinamento na tecnologia de implantação e encontrarão muitas resistências ao se propor mudanças na aplicação.

## Microservices - enfrentando a complexidade
Muitas organizações, como **Amazon**, **eBay** e **Netflix**, resolveram esse problema utilizando a abordagem do padrão do Microservices. Em vez de construir uma única aplicação monstruosa e monolítica, a ideia é dividir a aplicação em um conjunto de serviços menores e interconectados.

Um serviço tipicamente implementa um conjunto de funcionalidades, tais como:
* **Gestão de Clientes**;
* **Gestão de Pedidos**;
* ...

Cada microserviço é uma mini-aplicação que tem sua própria arquitetura monolítica que consiste em lógica de negócio juntamente com vários adaptadores. Alguns microservices exporiam uma API (Interface de Programação de Aplicação) que é consumida por outros microservices ou pelos clientes da aplicação. Outros microservices podem implementar uma interface da Web. Em tempo de execução, cada instância é geralmente uma nuvem em VM (Máquina Virtual) ou container do Docker.

Abaixo, há uma possível decomposição funcional da aplicação:

<centered>
  <img src="https://raw.githubusercontent.com/mrparty/tech-articles/master/nginx/nginx-article-1.png" width="500" height="500">
</centered>

Cada área funcional da aplicação é agora implementada por seu próprio microserviço. Além disso, a aplicação Web é dividida em um conjunto de aplicações Web mai simples (como uma para passageiros e outro para motoristas). Isso facilita a implantação de experiências distintas para usuários específicos, dispositivos ou casos de uso especializados.

Cada serviço de backend expõe uma API REST e a maioria dos serviços consome APIs fornecidas por outros serviços. Por exemplo, o **Gerenciamento de Drivers** usa o servidor de notificação para informar um driver disponível sobre uma possivel viagem. Os serviços de interface do usuário invocam os outro serviços para renderizar páginas Web. Os serviços também podem usar comunicação assíncrona baseada em mensagens. A comunicação entre serviços será abordada mais detalhadamente em outro capítulo.

Algumas APIs REST também são expostas aos aplicativos móveis usados pelos motoristas e passageiros. As aplicaçes, no entanto, não tem acesso direto aos serviços de backend. Em vez disso, a comunicação é intermediada por um componente chamado **API Gateway**. O API Gateway é responsável por tarefas como balanceamento de carga, cache, controle de acesso, medição de API e monitoramento, e **pode ser implementado efetivamente usando o NGINX**. Capítulos posteriores irão cobrir o API Gateway.

### Arquitetura de Microserviço e o Cubo de Escalabilidade
O padrão de **Arquitetura de Microserviços** corresponde ao **eixo Y** no **Cubo de Escalabilidade** (*Scale Cube*), que é um modelo 3D de escalabilidade do livro *The Art of Scalability*. Os outros eixos de dimensionamento são escalonamento de **eixo X**, que consiste em várias cópias idênticas do aplicativo por trás de um balanceador de carga e o escalonamento de **eixo Z** (ou particionamento de dados), onde um atributo da solicitação (como chave-primária) é  usado para encaminhar a solicitação para um servidor específico.

<img src="https://raw.githubusercontent.com/mrparty/tech-articles/master/nginx/nginx-article-3.png" width="500" height="500">

As aplicaçes costumam utilizar os três tipos de escalonamento em conjunto:
* **Eixo Y**: decompõe a aplicação em microserviços;
* **Eixo X**: em tempo de execução, executa várias instâncias (clones) de cada serviço atrás de um balanceador de carga (*load balancer*);
* **Eixo Z**: usado por algumas aplicações para particionar serviços, como Banco de Dados.

A seguir, um exemplo de serviço de **Gerência de Viagem** implementado com o Docker na AWS EC2.

<img src="https://raw.githubusercontent.com/mrparty/tech-articles/master/nginx/nginx-article-3.png" width="500" height="400">

### Relação da Arquitetetura de Microserviço e o Banco de Dados
A Arquitetura de Microserviço afeta significativamente a relação entre o aplicativo e o banco de dados. Em vez de compartilhar um único esquema do BD com outros serviços, cada serviço tem seu próprio esquema do Banco de Dados. Por um lado, esta abordagem está em desacordo com a ideia de um modelo de dados de toda a empresa. Além disso, muitas vezes resulta na duplicação de alguns dados. No entanto, ter um esquema de Banco de Dados por serviço é essencial se você quiser se beneficiar de microservices, porque garante acoplamento frouxo. O diagrama a seguir mostra a arquitetura do BD para o aplicativo de exemplo.

<img src="https://raw.githubusercontent.com/mrparty/tech-articles/master/nginx/nginx-article-4.png" width="500" height="400">

Cada um dos servços tem seu próprio Banco de Dados. Além disso, um serviço pode usar um tipo de BD que é mais adequado às suas necessidades, a chamada **Arquitetura de Persistência Polyglot**. Por exemplo, a Gestão de Corridas, que localiza drivers próximos a um passageiro em potencial, deve usar um Banco de Dados que ofereça suporte a geo-consultas eficientes.

Há também muita semelhança entre a Arquitetura de Microserviços com o **SOA** (**Orientado a Serviços**). Ambas as abordagens consistem num conjunto de serviços. A diferença entre as duas arquiteturas consiste que o SOA tem caráter comercial (geralmente atrelado a uma tecnologia específica fornecida por uma única empresa como um "pacote"), além de uma especificação como um Web Service (WSDL, SOAP e etc) e um ESB (Enterprise Service Bus). As aplicações baseadas em Microservices favorecem protocolos mais simples e leves, como o REST, ao invs de Web Service. O padrão de Microserviço rejeita outras partes do SOA, como o conceito deum esquema canônico.
