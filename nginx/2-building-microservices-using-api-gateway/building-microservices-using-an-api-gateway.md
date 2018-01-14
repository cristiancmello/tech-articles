# 2. Construindo Microservices: Usando um API Gateway
Suponha que iremos construir um aplicativo para Android de e-commerce. É
bem provável que precisaremos implementar uma página de detalhes do produto.

Exemplo abaixo, o diagrama mostra o que veremos ao percorrer os detalhes do
produto no aplicativo móvel da Amazon.

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/2-building-microservices-using-api-gateway/nginx-img-1.png"/>
</p>

Mesmo sendo um aplicativo para smartphone, a página de detalhes do produto
exibe muitas informações. Existem informações mais do que básicas que podem
ser visualizadas:

* Número de itens no carrinho de compras;
* Histórico de pedidos;
* Avaliação de Clientes;
* Opções de envio;
* e etc.

Ao se utilizar uma arquitetura de backend monolítico, um aplicativo móvel
recuperaria dados fazendo uma única chamada **REST**. Em seguida, um 
*load balancer* encaminha o pedido para uma das *N* instâncias da aplicação 
("clones").
    
O aplicativo, então, consulta várias tabelas do Banco de Dados
e retorna um JSON como resposta ao cliente por meio da rota HTTP
(URL) `GET api.company.com/productdetails/product-id`.

Por outro lado, utilizando-se a abordagem de Microservices, os dados na página
de detalhes do produto são de propriedade de vários microserviços. Alguns
microserviços com potencial de serem utilizados:

* Carrinho de Compras: número de itens no carrinho de compras;
* Serviço de Pedidos: histórico de encomendas;
* Serviço de Catálogo: informações básicas de produto, como nome, imagem 
e preço;
* e etc;

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/2-building-microservices-using-api-gateway/nginx-img-2.png"/>
</p>


## Comunicação direta entre cliente e microserviço
Em teoria, uma aplicação cliente poderia fazer pedidos para cada um dos
microserviços por meio do acesso a um *end-point* público como uma rota
HTTP. Essa URL seria mapeada para um load balancer, que iria distribuir
solicitações entre as instâncias disponíveis.

Em resumo, para recuperar os detalhes de produto, o aplicativo cliente faria
solicitações para cada um dos serviços listados anteriormente.

* **IMPORTANTE:** a abordagem de microserviços vai significar, neste cenário,
que somente para processar uma página, serão precisos fazer muitas solicitações
a diversos serviços - o que irá demandar mais tráfego de rede numa rede local
ou rede móvel. Além disso, o código do lado cliente irá ser tornar muito
complexo.

* Problema dos protocolos não-*web-friendly*: um serviço pode fazer chamada
RPC (procedimento remoto) que está sendo servido por um binário no lado
servidor. Assim, tem-se um entrave quando a aplicação cliente tenta chamar
esses serviços diretamente.

* **Problema da refatoração dos microserviços**: outra desvantagem de ter
clientes chamando diretamente cada serviço é na dificuldade em refatorar
serviços, ou seja, alterando o comportamento padrão deles, unindo com
outros serviços ou dividindo.

Devido a esses tipos de problemas, raramente faz sentido os clientes efetuarem
uma chamada direta aos microserviços.


## Usando um API Gateway
Um abordagem muito melhor é a utilização de um servidor central para roteamento
de APIs, o chamado **API Gateway**. Este é o único *entry-point* (ponto de 
entrada) no sistema.

Fica encarregado de encapsular a arquitetura interna do sistema, fornecendo
uma API adaptada para cada cliente. Também pode possuir outras
responsabilidades, tais como:

* Autenticação;
* Monitoramento;
* Load balancer;
* Armazenamento em cache;
* Tratamento de resposta estática;
* e etc.

<p align="center">
    <img src="https://raw.githubusercontent.com/cristiancmello/tech-articles/master/nginx/2-building-microservices-using-api-gateway/nginx-img-3.png"/>
</p>

* Logo, o API Gateway é responsável pelo roteamento de requisições, composição
e tradução de protocolos "hostis" à Web.

* Todas as requisições do lado cliente irão passar pelo API Gateway. Em
seguida, o mesmo encaminhará ao microserviço apropriado.

* Podemos também diferenciar cada serviço disponibilizado pelo API Gateway
a cada tipo de dispositivo, de modo a oferecer melhor adaptação às necessidades
intrínsecas.
    - Um ótimo exemplo disso é do API Gateway da Netflix. Inicialmente, a 
    Netflix tentou oferecer uma única API para todos os dispositivos.
    Entretanto, a empresa percebeu as necessidades exclusivas dos mesmos
    e a evolui para um API Gateway que fornece uma API adaptada para
    cada dispositivo. Em geral, um adaptador lida com cada requisição
    invocando em média de 6 a 7 serviços de back-end.


## Vantagens e Desvantagens de um API Gateway
* Uma das principais vantagens: encapsulamento da estrutura interna
da aplicação. Em vez de se ter que invocar serviços específicos, as
aplicações clientes se comunicam diretamente com o API Gateway. Isso
reduz o tráfego de rede entre o cliente e a aplicação. Também simplifica
o código do cliente.

* Uma desvantagem: é outro componente que irá precisar estar altamente
disponível e que deve ser desenvolvido, implantado e gerenciado. Há
um risco do API Gateway ser um gargalo no desenvolvimento da aplicação.

* É importante que o processo de atualização do API Gateway seja o mais
leve possível. Caso contrário, os desenvolvedores serão forçados a esperar
para atualizá-lo.

* Apesar dessas desvantagens, no entanto, para a maioria das aplicações
do mundo real, faz sentido usar um API Gateway.


## Implementando um API Gateway
Vamos analisar os problemas de design ao se projetar um API Gateway.

### Desempenho e Escalabilidade
Faz sentido construir o API Gateway em uma plataforma que suporte Input/Output 
assíncronos não-bloqueantes. Um artigo interessante tratando sobre
essa questão está [aqui](https://imasters.com.br/front-end/javascript/node-js-v8-single-thread-e-io-nao-bloqueante/?trace=1519021197&source=single).

Algumas tecnologias que oferecem bom desempenho e escalabilidade são:

* [Tyk](https://tyk.io/): API Gateway Open Source;
* [Amazon API Gateway](https://aws.amazon.com/pt/api-gateway/): gratuito para teste de 1 ano;
* [Netty](https://netty.io/);
* [Eclipse Vert.x](http://vertx.io/);
* [Project Reactor](https://projectreactor.io/);
* [Node.js](https://nodejs.org/en/): podemos utilizá-lo diretamente também;
* [NGINX Plus](https://www.nginx.com/solutions/api-gateway/): é oferecido um servidor Web, 
escalável e de alto desempenho, além de proxy reverso. É pago.

### Usando um modelo de programação reativa
O API Gateway lida com algumas solicitações simplesmente roteando-as para o
serviço de back-end apropriado. Ele lida com outras solicitações invocando
vários serviços de back-end e agregando resultados.

Algumas requisições, como a solicitação de detalhes de produto para serviços,
são independentes uma da outra. Para minimizar o tempo de resposta, o API
Gateway deverá executar as requisições independentes simultaneamente. Às
vezes, no entanto, há dependências entre solicitações. O API Gateway pode
primeiramente precisar validar a requisição para um serviço de back-end.
* Exemplo: para buscar informações sobre produtos na lista de desejos
de um cliente, o API Gateway deve primeiro recuperar o perfil do cliente
que contém essas informações e, em seguida, recuperar as informações de
cada produto.

* Escrever código de API usando a abordagem de retorno de chamada assíncrona
tradicional rapidamente nos leva ao inferno de retorno de chamada. Haverá
um emaranhado de códigos, complexidade de entendimento e propensão a erros
muito grande durante o desenvolvimento.

* Uma forma mais eficiente é escrever código de API Gateway num estilo
declarativo usando abordagem reativa (*Reactive Programming*). Exemplos:

- [Future](http://docs.scala-lang.org/overviews/core/futures.html) (em Scala);
- [CompletableFuture class](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) (em Java 8);
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) (em JavaScript);
- [RectiveX](http://reactivex.io/);

### Invocação de Serviço
Um aplicativo baseado em microserviços é um sistema distribuído e deve usar
um mecanismo de comunicação inter-processo. Existem dois estilos de comunicação
entre processos:
* Usar mecanismo assíncrono baseado em mensagens. Exemplos: 
**JMS (Java Message Service)**, protocolo 
**AMQP (Advanced Message Queuing Protocol)**, **ZeroMQ** e etc;

* Usar mecanismo síncrono em HTTP. Exemplo: **Apache Thrift**, **ASN.1**, 
**Protocol Buffers** e etc.
    
* **IMPORTANTE**: um sistema usará tipicamente estilos assíncronos e síncronos.
É possível até usar múltiplas implementações de cada estilo. Consequentemente,
o **API Gateway precisará suportar uma variedade de mecanismos de 
comunicação**.

### Descoberta de Serviço
* O API Gateway precisará saber o local (endereço IP e porta) de cada
microserviço com o qual ele se comunica. Num aplicativo simples, 
a descoberta dos serviços se dará de maneira trivial. Com o aumento
do uso de serviço, será *não-trivial*.

* Serviços de infraestrutura, como um **message broker**, geralmente
tem uma localização estática, que pode ser especificada através de
variáveis de ambiente do Sistema Operacional.
    - Serviços de aplicação têm atribuições dinamicamente localizadas.
    - Conjunto de instâncias ("clones") de um serviço muda dinamicamente
    por causa do *autoscaling* e atualizações.

* Logo, o API Gateway irá precisar de mecanismo de descoberta de serviço
de sistema.
    - [Server-side](http://microservices.io/patterns/server-side-discovery.html);
    - [Client-side](http://microservices.io/patterns/client-side-discovery.html).

* Posteriormente, a descoberta de serviço será tratada em detalhes. Por 
enquanto, vale notar que, se o sistema usa descoberta *Client-side*, o API
Gateway deve poder consultar o **Service Registry**, que é um banco de dados
onde as localizações de todas as instâncias de microserviços estão armazenadas.

### Tratamento de falhas parciais
Outro problema que teremos que resolver será de falhas que ocorrem quando
um serviço chama um outro que está demorando para responder ou pode estar
indisponível. O API Gateway não pode esperar indefinidamente por um serviço.

* Uma forma de lidar com essa falha: retornar ao cliente uma informação
relevante ao usuário enquanto o serviço procurado está fora do ar. Se,
no entanto, um serviço mais básico está fora do ar, o API Gateway deve 
retornar um erro ao cliente. Essa é uma forma de mascarar falhas nos
serviços de back-end, podendo-se fazer uso de caches.

* Uma ótima biblioteca para lidar com esse cenário de tolerância a falhas
é a [**Netflix Hystrix**](https://github.com/Netflix/Hystrix), fazendo uso
da JVM. Alternativas: 
    - [**Hystrix-Go**](https://github.com/afex/hystrix-go) (em Go);
    - [**Phystrix**](https://github.com/upwork/phystrix) (em PHP).
