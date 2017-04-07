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

<center>
  <img src="https://raw.githubusercontent.com/mrparty/tech-articles/master/nginx/nginx-article-0.png" width="500" height="500">
</center>

