# Building Microservices: Inter-Process Communication in a Microservices 
Architecture

Em um aplicação monolítica, os componentes invocam uns aos outros através
de chamadas de função ou de nível de linguagem. Em constraste, um aplicação
baseada em microservices é um sistema distribuído executado em várias
máquinas. Cada instâncias de serviço é tipicamente um processo. Para interagir
com os serviços, podemos utilizar um **mecanismo de comunicação entre 
processos** (**IPC**).

<p align="center">
    <img src="..."/>
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
    <td>Requisição / Resposta</td>
    <td>-</td>
  </tr>
  <tr>
    <th rowspan="2">Assíncrono:</th>
    <td>Notificação</td>
    <td>Publicar / Assinar</td>
  </tr>
  <tr>
    <td>Resposta de Solicitação / Resposta Assíncrona</td>
    <td>Publicar / Respostas Assíncronas</td>
  </tr>
</table>

* Tipos de **interações 1:1**:
    - **Requisição / Resposta**: um cliente solicita um serviço e aguarda uma
    resposta. O cliente espera que a resposta chegue em tempo hábil. Num
    aplicativo baseado em thread, o segmento que faz a solicitação pode
    até bloquear enquanto espera;

    - **Notificação**: também conhecido como uma solicitação unidirecional,
    um cliente envia uma solicitação para um serviço, mas nenhuma resposta
    é esperada ou enviada;
    
    - **Requisição / Resposta assíncrona**: um cliente envia uma solicitação
    para um serviço, que responde de forma assíncrona. O cliente não bloqueia
    enquanto espera e é projetado com o pressuposto de que a resposta pode não
    chegar a tempo.
    
* Tipos de **interações 1:N**:
    - **Publicar / Subscrever**: um cliente publica uma mensagem de 
    notificação, que é consumida por zero ou mais serviços interessados.

    - **Publicar / Respostas assíncronas**: um cliente publica uma mensagem de
    solicitação e, em seguida, aguarda certo tempo para respostas de serviços
    interessados.

* Cada serviço geralmente usa uma combinação desses estilos de interação.
* Para alguns serviços, um **único mecanismo IPC é suficiente**.

O diagrama abaixo mostra como os serviços de um aplicativo de táxi podem
interagir quando o usuário solicita uma viagem.

<p align="center">
    <img src="..."/>
</p>

* Os serviços usam uma combinação de notificações, requisição / resposta e
publicar / assinar. Por exemplo, um smartphone do passageiro envia uma
notificação para o serviço **Trip Management** para solicitar um
veículo. O serviço **Trip Management** verifica se a conta do passageiro
está ativa, usando requisição / resposta para invocar o **Passenger 
Management**. O serviço **Trip Management** cria a viagem e usa publicar / 
assinar para notificar outros serviços, incluindo o **Dispatcher**, que
localiza uma corrida disponível.

## Definindo APIs
* Uma API é um contrato entre um serviço e seus clientes. Independente da 
escolha do mecanismo de comunicação entre processos (IPC), é importante
definir com precisão a API através de algum tipo de linguagem de definição
de interface (IDL).
    - Projetar a definição de API e apresentar aos desenvolvedores das 
    aplicações clientes antes de implementar significa aumentar as
    nossas chances de construir um serviço que atenda as necessidades
    efetivamente.

* **IMPORTANTE**: a natureza da definição de API depende do mecanismo de IPC
adotado. Se estivermos usando HTTP, por exemplo, a API poderá ser definida
através de URLs e nos formatos de dados de requisição / resposta.

## APIs em evolução
* API de um serviço muda invariavelmente ao longo do tempo. Em um aplicativo
monolítico, é geralmente simples mudar a API e atualizar todos os *callers*.

* Numa aplicação de microserviços, há muitos problemas em atualizar a API
devido a compatibilidade que os consumidores dos serviços esperam.

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
devido a manutenção, desativação ou sobrecarga.

* Exemplo: considere a tela de *Detalhes de Produto*. Ele faz uso do
**Serviço de Recomendação** e imaginemos que esse serviço pare de
funcionar.
  - Uma implementação ingênua de um cliente pode bloquear indefinidamente
  aguardando uma resposta. Isso geraria uma experiência de usuário ruim,
  além de consumir tempo de processamento precioso da aplicação.

  - Eventualmente, a aplicação mataria threads em tempo de execução e
  ficaria sem resposta.

<p align="center">
  <img src="..."/>
</p>

* Para evitar esse problema, é essencial projetarmos os serviços
tendo em vista cenários de falhas.

### Estratégias da Netflix para lidar com falhas parciais
Através da biblioteca [Netflix Hystrix](https://github.com/Netflix/Hystrix),
são implementadas as estratégias a seguir e outros padrões.

* **Timeouts de rede**: nunca bloqueie indefinidamente e use sempre tempos
limite ao esperar por uma resposta. Usando tempos limite garante que os
recursos nunca serão "travados" por tempo indeterminado.

* **Limite de número de requisições pendentes**: impor um limite superior
sobre o número de requisições pendentes que um cliente pode ter com um
determinado serviço. Se o limite for atingido, provavelmente é inútil
fazer requisições adicionais, sendo que as mesmas precisam falhar
imediatamente.

* **Circuit Breaker Pattern**: rastreia o número de solicitações bem-
sucedidas e com falha. Se a taxa de erro exceder um limite configurado,
dispare um *circuit breaker* para que outras tentativas falhem imediatamente.
Se um grande número de solicitações estão falhando, isso sugere que o serviço 
não está disponível e que enviar requisições é inútil. Após um período de
tempo limite, o cliente deve tentar novamente e, se bem-sucedido, fechar o
*circuit breaker*.

* **Fornecimento de Fallbacks**: executa a lógica de fallback quando uma
solicitação falha. Por exemplo, retornar dados armazenados em cache ou um
valor padrão, como um conjunto vazio de recomendações.

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
    
O diagrama abaixo mostrar como a aplicação de solicitação de táxi pode
usar canais **Publish-Subscribe**.

<p align="center">
  <img src="..."/>
</p>

* O serviço Trip Management notifica os serviço interessados, como o 
Dispatcher, sobre um nova viagem, escrevendo uma mensagem Trip Created
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
  simplesmente enviando uma menasgem para o canal apropriado. O cliente
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
  outro componentes a ser instalado, configurado e operado. É essencial
  que o *message broker** seja altamente disponível, caso contrário,
  a confiabilidade do sistema é impactada;

  - **Complexidade da implementação de requisição/resposta**: esse
  estilo requer algum trabalho a ser implementado. Cada mensagem de pedido
  deve conter um identificador de canal de resposta e um identificador
  de correlação. O serviço grava uma mensagem de resposta contendo
  contendo o ID de correlação para o canal de resposta. O clien utiliza
  o ID de correlação para corresponder à resposta com a solicitação. Muitas
  vezes, é mais fácil usar um mecaniso de IPC que suporte diretamente
  requisição/resposta.

### IPC baseado em requisição / resposta síncrona


#### REST

#### Apache Thrift

### Formatos de mensagem