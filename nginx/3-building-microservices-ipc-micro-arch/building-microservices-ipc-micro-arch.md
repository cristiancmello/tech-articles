# Building Microservices: Inter-Process Communication in a Microservices Architecture

Em um aplicação monolítica, os componentes invocam uns aos outros através
de chamadas de função ou de nível de linguagem. Em contraste, um aplicação
baseada em microservices é um sistema distribuído executado em várias
máquinas. Cada instâncias de serviço é tipicamente um processo. Para interagir
com os serviços, podemos utilizar um **mecanismo de comunicação entre 
processos** (**IPC**).

<p align="center">
    <img src="https://cdn.rawgit.com/mrparty/tech-articles/master/nginx/3-building-microservices-ipc-micro-arch/nginx-article-0.png" width="800px"/>
</p>

Futuramente, iremos abordar as tecnologias de IPC específicas. Neste artigo,
serão discutidos problemas de design.

## Estilos de Interação
É útil pensar sobre como os serviços interagem. Há ma variedade de estilos
de interação *Cliente* <=> *Servidor*.

Os estilos de interação podem ser categorizados em 2 dimensões:

* 1ª Dimensão:
    - **Interação 1:1**: cada requisição de cliente é processada por exatamente
    1 instância de serviço;
    
    - **Interação 1:N**: cada requisição é processada por várias instâncias de
    serviço.

* 2ª Dimensão:
    - **Interação Síncrona**: o *cliente* **espera** uma **resposta** 
    oportuna do serviço e **pode até bloquear enquanto aguarda**;

    - **Interação Assíncrona**: o *cliente* **não bloqueia** *enquanto* 
    **aguarda uma resposta**. Além disso, **não precisa enviar imediatamente 
    a resposta**.
    
A tabela a seguir mostra os vários estilos de interação.

<table style="width:100%">
  <tr>
    <th></th>
    <td><b>Interação 1:1</b></td>
    <td><b>Interação 1:N</b></td>
  </tr>
  <tr>
    <th>Síncrono:</th>
    <td>Requisição/Resposta</td>
    <td>-</td>
  </tr>
  <tr>
    <th rowspan="2">Assíncrono:</th>
    <td>Notificação</td>
    <td>Publicar/Assinar</td>
  </tr>
  <tr>
    <td>Resposta de Solicitação/Resposta Assíncrona</td>
    <td>Publicar/Respostas Assíncronas</td>
  </tr>
</table>

* Tipos de **interações 1:1**:
    - **Requisição/Resposta**: um cliente solicita um serviço e aguarda uma
    resposta. O cliente espera que a resposta chegue em tempo hábil. Num
    aplicativo baseado em thread, o segmento que faz a solicitação pode
    até bloquear enquanto espera;

    - **Notificação**: também conhecido como uma solicitação unidirecional,
    um cliente envia uma solicitação para um serviço, mas nenhuma resposta
    é esperada ou enviada;
    
    - **Requisição/Resposta assíncrona**: um cliente envia uma solicitação
    para um serviço, que responde de forma assíncrona. O cliente não bloqueia
    enquanto espera e é projetado com o pressuposto de que a resposta pode não
    chegar a tempo.
    
* Tipos de **interações 1:N**:
    - **Publicar/Subscrever**: um cliente publica uma mensagem de 
    notificação, que é consumida por zero ou mais serviços interessados.

    - **Publicar/Respostas assíncronas**: um cliente publica uma mensagem de
    solicitação e, em seguida, aguarda certo tempo para respostas de serviços
    interessados.

* Cada serviço geralmente usa uma combinação desses estilos de interação.
* Para alguns serviços, um **único mecanismo IPC é suficiente**.

O diagrama abaixo mostra como os serviços de um aplicativo de táxi podem
interagir quando o usuário solicita uma viagem.

<p align="center">
    <img src="https://cdn.rawgit.com/mrparty/tech-articles/master/nginx/3-building-microservices-ipc-micro-arch/nginx-article-1.png" width="800px"/>
</p>

* Os serviços usam uma combinação de notificações, requisição/resposta e
publicar/assinar. Por exemplo, o smartphone do passageiro envia uma
notificação para o serviço **Trip Management** para solicitar um
veículo. O serviço **Trip Management** verifica se a conta do passageiro
está ativa, usando requisição/resposta para invocar o **Passenger 
Management**. O serviço **Trip Management** cria a viagem e usa 
publicar/assinar para notificar outros serviços, incluindo o **Dispatcher**, 
que localiza uma corrida disponível.

## Definindo APIs
* Uma API é um contrato entre um serviço e seus clientes. Independente da 
escolha do mecanismo de **comunicação entre processos** (**IPC**), é importante
definir com precisão a API através de algum tipo de linguagem de definição
de interface (IDL).
    - Projetar a definição de API e apresentar aos desenvolvedores das 
    aplicações clientes antes de implementar significa aumentar as
    nossas chances de construir um serviço que atenda as necessidades
    efetivamente.

* **IMPORTANTE**: a natureza da definição de API depende do mecanismo de IPC
adotado. Se estivermos usando HTTP, por exemplo, a API poderá ser definida
através de URLs e nos formatos de dados de requisição/resposta.

## APIs em evolução
* API de um serviço muda invariavelmente ao longo do tempo. Em um aplicativo
monolítico, é geralmente simples mudar a API e atualizar todos os *callers*.

* Numa aplicação de microserviços, há muitos problemas em atualizar a API
devido à compatibilidade que os consumidores dos serviços esperam.

* Logo, **é importante ter uma estratégia para lidar com essas questões**.
  - Os clientes que usam uma API mais antiga devem continuar a trabalhar
  com a nova versão. O serviço fornece valores-padrão para os atributos de
  solicitação ausentes e os clientes ignoram quaisquer atributos de resposta
  extra.

  - **É importante usar um mecanismo de comunicação entre processos (IPC)
  e um formato de mensagens que permitam evoluir facilmente as APIs**.
  
  - No entanto, ocorrerá necessidades de se fazer alterações importantes
  e incompatíveis para uma API. **Como você não pode forçar os clientes a
  atualizarem imediatamente**, um serviço deverá oferecer suporte a versões
  mais antigas por um período de tempo.

  - Se estivermos usando um mecanismo baseado em HTTP, como REST, uma
  abordagem é incorporar o número da versão na URL. Exemplo:
  `https://site.com/api/v1/persons` (API versão 1).

  - Cada instância de serviço pode manipular várias versões simultaneamente.
  Como alternativa, podemos implantar diferentes instâncias e cada uma
  com uma versão específica.

## Tratamento de falhas parciais
* Há sempre um risco de falhas parciais na comunicação entre serviços, seja
devido à manutenção, desativação ou sobrecarga.

* Exemplo: considere a tela de *Detalhes de Produto*. Ele faz uso do
**Serviço de Recomendação** e imaginemos que esse serviço pare de
funcionar.
  - Uma implementação ingênua de um cliente pode bloquear indefinidamente
  aguardando uma resposta. Isso geraria uma experiência de usuário ruim,
  além de consumir tempo de processamento precioso da aplicação.

  - Eventualmente, a aplicação mataria threads em tempo de execução e
  ficaria sem resposta.

<p align="center">
  <img src="https://cdn.rawgit.com/mrparty/tech-articles/master/nginx/3-building-microservices-ipc-micro-arch/nginx-article-2.png"/>
</p>

* Para evitar esse problema, é essencial projetarmos os serviços
tendo em vista cenários de falhas.

### Estratégias da Netflix para lidar com falhas parciais
Através da biblioteca [Netflix Hystrix](https://github.com/Netflix/Hystrix),
são implementadas as estratégias a seguir e outros padrões.

* **Timeouts de rede**: nunca bloqueie indefinidamente e use sempre 
tempos-limite ao esperar por uma resposta. O uso de tempos-limite garante que
os recursos nunca serão "travados" por tempo indeterminado.

* **Limite de número de requisições pendentes**: consiste em impor um limite 
superior sobre o número de requisições pendentes que um cliente pode ter com um
determinado serviço. Se o limite for atingido, provavelmente é inútil
fazer requisições adicionais, sendo que as mesmas precisarão falhar
imediatamente.

* **Circuit Breaker Pattern**: rastreia o número de solicitações bem-
sucedidas e com falha. Se a taxa de erro exceder um limite configurado,
dispare um *circuit breaker* para que outras tentativas falhem imediatamente.
Se um grande número de solicitações estão falhando, isso sugere que o serviço 
não está disponível e que enviar requisições é inútil. Após um período de
tempo-limite, o cliente deve tentar novamente e, se bem-sucedido, fechar o
*circuit breaker*.

* **Fornecimento de Fallbacks**: executa a lógica de fallback quando uma
solicitação falha. Por exemplo, retornar dados armazenados em cache ou um
valor padrão, como um conjunto vazio de recomendações, por exemplo.

## Tecnologias de IPC
Há muitas tecnologias de comunicação entre processos (IPC):

* **Mecanismos de comunicação síncronos baseados em requisição/resposta**: 
**HTTP REST** e **Apache Thrift** baseado em HTTP;

* **Mecanismos de comunicação assíncronos baseados em mensagens**: **AMQP** 
ou **STOMP**;

* Formatos de mensagens: 
  - Baseados em **texto**: **JSON** ou **XML**;
  - Baseados em **formato binário**: **Apache Avro** ou **Protocol Buffers**;

### Comunicação assíncrona baseada em mensagens
Ao usar mensagens, os processos se comunicam pela troca de mensagens de modo
assíncrono. Um cliente faz um pedido a um serviço enviando-lhe uma mensagem.
Se o serviço é esperado para responder, ele faz isso enviando outra mensagem
separada de volta ao cliente. Devido à natureza assíncrona da requisição, o
cliente é criado assumindo que a resposta não será recebida imediatamente.

* Mensagem = cabeçalho (como metadado de remetente) + corpo.
* As mensagens são trocadas por *canais*.
  - Qualquer quantidade de produtores pode enviar mensagens para 1 canal.
  Da mesma forma, qualquer quantidade de consumidores pode receber mensagens
  de 1 canal. Existem 2 tipos de canais:
    - Canal **Point-to-point**: entrega uma mensagem para exatamente um dos
    consumidores que está ouvindo o canal; os serviços que usam canais 
    point-to-point são dos **estilos de interação 1:1**;

    - Canal **Publish-Subscribe**: 1 canal entrega cada mensagem a todos os
    consumidores conectados; os serviços usam canais publish-subscribe para
    os **estilos de interação 1:N**.
    
O diagrama abaixo mostra como a aplicação de solicitação de táxi pode
usar canais **Publish-Subscribe**.

<p align="center">
  <img src="https://cdn.rawgit.com/mrparty/tech-articles/master/nginx/3-building-microservices-ipc-micro-arch/nginx-article-3.png" width="700px"/>
</p>

* O serviço Trip Management notifica os serviços interessados, como o 
Dispatcher, sobre uma nova viagem, escrevendo a mensagem Trip Created
para um canal de **Publish-Subscribe**. O Dispatcher localiza uma corrida
disponível e notifica outros serviços escrevendo uma mensagem de Corrida
Disponível para um canal de **Publish-Subscribe**.

* Existem vários sistemas de mensagens. Alguns:
  - Suportam protocolos padronizados: 
    - [AMQP](https://www.amqp.org/)
    - [STOMP](https://stomp.github.io/);
  - Outros sistemas Open Source:
    - [RabbitMQ](https://www.rabbitmq.com/);
    - [Apache Kafka](http://kafka.apache.org/);
    - [Apache ActiveMQ](http://activemq.apache.org/);
    - [NSQ](https://github.com/nsqio/nsq).

* Vantagens em se usar mensagens:
  - **Desacopla o cliente do serviço**: um cliente faz uma requisição
  simplesmente enviando uma mensagem para o canal apropriado. O cliente
  está completamente inconsciente das instâncias de serviço. Ele não
  precisa usar um mecanismo de descoberta para determinar o local de uma
  instância de serviço;

  - **Buffer de mensagens**: ao se usar um protocolo síncrono de
  requisição/resposta, como um HTTP, podemos enfileirar as mensagens
  gravadas num canal até que elas possam ser processadas pelo consumidor.
  Isso significa, por exemplo, que um e-commerce pode aceitar pedidos de
  clientes mesmo quando o sistema de atendimento de pedidos está lento
  ou indisponível.

  - **Interações flexíveis com o cliente**: sistemas de mensagens suportam
  todos os estilos de interação descritos anteriormente.

  - **Comunicação explícita entre processos**: os mecanismos baseados em
  RPC tentam fazer com que a chamada de um serviço remoto pareça o mesmo
  que chama um serviço local. No entanto, por causa das leis da física e
  das possibilidades de fracasso parcial, eles são de fato muito diferentes.
  As mensagens tornam essas diferenças muito explícitas, de modo que os
  desenvolvedores não são confundidos com um falso senso de segurança.

* Desvantagens em se usar mensagens:
  - **Complexidade operacional adicional**: o sistema de mensagens será
  outro componente a ser instalado, configurado e operado. É essencial
  que o *message broker** seja altamente disponível, caso contrário,
  a confiabilidade do sistema é impactada;

  - **Complexidade da implementação de requisição/resposta**: esse
  estilo requer algum trabalho a ser implementado. Cada mensagem de pedido
  deve conter um identificador de canal de resposta e um identificador
  de correlação. O serviço grava uma mensagem de resposta contendo
  o ID de correlação para o canal de resposta. O cliente utiliza
  o ID de correlação para corresponder à resposta com a solicitação. Muitas
  vezes, é mais fácil usar um mecaniso de IPC que suporte diretamente
  requisição/resposta.

### IPC baseado em requisição/resposta síncrona
* Basicamente, um mecanismo baseado em requisição/resposta consiste no
envio de uma solicitação por parte do cliente para um serviço. O serviço
processa a requisição e envia uma resposta.

* O cliente assume que a resposta à requisição chegará em tempo hábil.

* 2 protocolos populares:
    - **REST**;
    - **Apache Thrift**.

#### REST
- Está muito em voga desenvolver APIs ao estilo **RESTful**;
- O REST é um mecanismo IPC que quase sempre usa HTTP;
- Conceito-chave em REST: **resource**, que normalmente representa o
  *objeto do negócio*, como Cliente ou Produto, ou *coleção de objetos
  do negócios*.
  - Utiliza verbos HTTP (GET, POST, PUT/PATCH, DELETE) para manipular
  resources.

Diagrama abaixo mostrar uma das maneiras pelas quais o aplicativo de
táxi pode usar REST.

<p align="center">
  <img src="https://cdn.rawgit.com/mrparty/tech-articles/master/nginx/3-building-microservices-ipc-micro-arch/nginx-article-4.png" width="700px"/>
</p>

- Smartphone do passageiro solicita uma viagem fazendo um **POST** para
resource **/trips** do serviço de Gerenciamento de Viagem;

- Em seguida, o Gerenciamento de Viagem processa o pedido enviando um
**GET** ao serviço de Gerenciamento de Passageiros;

- Depois de verificar que o passageiro está autorizado a criar uma viagem,
o serviço de Gerenciamento de Viagem cria um viagem e retorna **201**
ao smartphone.

Muitos desenvolvedores afirmam que suas APIs baseadas em HTTP são RESTful.
No entanto, Fielding descreve neste 
[post](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)
que nem todos são.

Leonard Richardson define um 
[modelo](https://martinfowler.com/articles/richardsonMaturityModel.html)
de maturidade útil para REST que consiste nos seguintes níveis:

* **Nível 0**: os clientes de uma API de nível 0 invocam o serviço fazendo
solicitações **POST** num único *end-point* de URL. Cada requisição
especifica a ação a executar, o destino da ação (o objeto de negócios, 
por exemplo) e quaisquer parâmetros;

* **Nível 1**: suporte ao conceito de **resources**. Para executar uma ação
num resource, um cliente faz uma requisição POST que especifica a ação a
executar e quaisquer parâmetros;

* **Nível 2**: uso de **verbos HTTP** para executar ações. Notáveis:
  - **GET**: recuperar;
  - **POST**: criar;
  - **PUT/PATCH**: atualização (comumente utilizado, mas não em definitivo);
  - **DELETE**: destruir.

* **Nível 3**: baseado no princípio **HATEOAS** (Hypertext as Engine Of 
Application State). A ideia básica é a representação de um **resource**
e **links para ações permitidas**, o que faz com que o cliente não
precise adivinhar quais ações podem ou não ser executadas em num
resource em seu estado atual.

Há inúmeros benefícios ao se utilizar um protocolo baseado em HTTP:
- HTTP é mais simples e familiar;
- Facilidade ao se testar uma API HTTP através de um navegador web
com extensão (como o 
[Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop))
ou o utilitário de linha de comando, como o **curl**;
- Ele suporta diretamente a comunicação de requisição/resposta;
- HTTP é naturalmente *firewall-friendly*;
- Não requer um *intermediate broker*, o que simplifica a arquitetura do
sistema.

Há algumas desvantagens para se usar HTTP:
- Ele suporta diretamente o estilo de interação requisição/resposta. Podemos
usá-lo para notificações, mas o servidor sempre deve enviar uma resposta HTTP;

- Tanto o cliente quanto o servidor precisam estar em execução durante o
período de comunicação;

- O cliente deve saber o local (ou seja, a URL) de cada instância de serviço.
Este é um problema não-trivial em um aplicação moderna. Como relatado no artigo
anterior, os clientes devem usar um mecanismo de descoberta de serviço para
localizar instâncias de serviço.

A comunidade de desenvolvedores redescobriu o valor de uma linguagem de 
definição de interface para um API RESTful. Algumas opções são:
- [RAML](http://raml.org/);
- [Swagger](http://swagger.io/);
- [Apiary](https://apiary.io/).

Há uma especificação de API RESTful baseada em JSON bastante adotada, 
chamada de [json:api](http://jsonapi.org/).

#### Apache Thrift
É uma alternativa interessante ao REST. É uma framework para escrever
cliente/servidor baseado em RPC (chamada à procedimento remoto) com
suporte a muitas linguagens (ex.: C/C++, Python, PHP, Node.js, ...).

Uma interface Thrift consiste em um ou mais serviços. Uma definição
de serviço é análoga a uma interface Java. É uma coleção de métodos
fortemente tipados. Os métodos Thrift podem retornar um valor
ou podem ser definidos como *one-way* (mão única). Os métodos que retornam
um valor implementam o estilo de interação de requisição/resposta.
O cliente aguarda uma resposta e pode lançar uma exceção. Os métodos
unidirecionais (*one-way*) correspondem ao estilo de interação de notificação.
O servidor, assim, não envia uma resposta.

Thrift suporta vários formatos de mensagem:
- **JSON** (humanamente mais amigável);
- **Binário** (mais eficiente do que JSON porque é mais rápido para 
decodificar);
- **Binário compactado** (mais eficiente em termos de espaço).

**IMPORTANTE**: Thrift permite a escolha de protocolos de transporte.
Podemos utilizar ou **TCP** (formato mais bruto do que HTTP e portanto 
mais rápido) ou **HTTP** (formato mais amigável para firewalls, navegadores
e humanamente inteligível).

### Formatos de mensagem
Se utilizarmos um sistema de mensagens ou REST, podemos escolher um
formato de mensagem. Outros mecanismos de IPC, como o Thrift, podem
suportar apenas um pequeno número de formatos de mensagem, ou
talvez apenas um formato. Em ambos os casos, é importante usar
algum formato de mensagem para várias linguagens.

Existem 2 tipos principais de formatos de mensagem: **texto** e **binário**.

* *Texto*: como exemplo JSON e XML. Possuem uma boa legibilidade e
são auto-descritivos. Entretanto, possuem uma desvantagem de se exigir
análise sintática do texto, o que pode sobrecarregar a aplicação dependendo
da dimensão das mensagens e recursos computacionais;

* *Binário*: como exemplo o Thrift binário. Se fizermos a escolha do formato
da mensagem, iremos precisar de escolher alguma tecnologia. As opções populares 
disponíveis são o 
[Protocol Buffers](https://developers.google.com/protocol-buffers/docs/overview)
e [Apache Avro](https://avro.apache.org/). A respeito da evolução da API,
é mais fácil desenvolver com o formato de mensagem do **Protocol Buffers**.

* Uma [postagem interessante](http://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html) 
com a comparação entre Avro, Protocol Buffers e Thrift.