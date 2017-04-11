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

## Tratamento de falhas parciais

## Tecnologias de IPC

### Comunicação assíncrona baseada em mensagens

### IPC baseado em requisição / resposta síncrona

#### REST

#### Apache Thrift

### Formatos de mensagem