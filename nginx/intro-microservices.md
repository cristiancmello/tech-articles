# Introdução aos Microservices
* Apesar de todo entusiasmo e ceticismo a cerca da ideia de Microservices, esse padrão arquitetural tem benefícios significativos - especialmente quando se trata de habilitar o desenvolvimento ágil e entrega de apps corporativos complexos.

* Neste artigo, são tratadas as diferenças básicas entre as arquiteturas de Microservices e a Monolítica, que é um padrão mais tradicional.

* Também serão tratadas as vantagens e desvantagens do padrão de Microservices.

## Construindo Aplicações Monolíticas
Vamos imaginar que iremos construir uma aplicação de táxi semelhante ao Uber. Após reuniões preliminares de coleta de requisitos, construiremos um novo projeto centralizado usando uma framework, como Laravel, Rails, Spring Boot entre outros.

Percebe-se que todas as funcionalidades irão ficar agregadas num mesmo repositório, expondo adapters (componentes de acesso ao BD, componentes de mensagens que produzem e consomem mensagens), APIs REST e Interfaces de Usuário. 

Apesar de ter uma estrutura logicamente modular, o aplicativo é empacotado e implantado como um monólito (algo único, coeso).

Aplicações escritas nessa forma são extremamente comuns. Elas são simples de se desenvolver uma vez que os ambientes se focam na construção de uma única aplicação. Os testes também são mais simples.

Aplicações monolíticas são mais simples de se implementar e implantar (Deploy). É necessário apenas copiar o pacote da aplicação para um servidor. Podemos também dimensionar o aplicativo executando várias cópias atrás de um balanceador de carga (load balancer). Inicialmente, essa arquitetura funciona bem.

## Marchando para o Monolithic Hell
A medida que a aplicação cresce ao longo do tempo, a abordagem monolítica começa a trazer muitas limitações. A cada Sprint, a equipe de desenvolvimento implementa mais histórias, o que, naturalmente, significa adicionar mais linhas de código. Após alguns anos, a aplicação se tornará um monstruoso monólito, havendo-se muitas dependências, riscos de migração elevados e etc.

Uma vez que a aplicação e tornou grande e complexa, a organização de desenvolvimento provavelmente estará cheia de atrasos e muito sacrifício. Quaisquer tentativas de desenvolvimento ágil e entrega serão a um grande custo. Entender o relacionamento entre as partes do sistema também será muito difícil, o que acarreta em correções de bugs mais demoradas e com milhares de efeitos colaterais indesejados.

Se a base de código for difícil de entender, as alterações não serão feitas corretamente. Logo, acabaremos numa enorme bola de lama.
