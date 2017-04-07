# Introdução aos Microservices
* Apesar de todo entusiasmo e ceticismo a cerca da ideia de Microservices, esse padrão arquitetural tem benefícios significativos - especialmente quando se trata de habilitar o desenvolvimento ágil e entrega de apps corporativos complexos.

* Neste artigo, são tratadas as diferenças básicas entre as arquiteturas de Microservices e a Monolítica, que é um padrão mais tradicional.

* Também serão tratadas as vantagens e desvantagens do padrão de Microservices.

## Construindo Aplicações Monolíticas
Vamos imaginar que iremos construir uma aplicação de táxi semelhante ao Uber. Após reuniões preliminares de coleta de requisitos, construiremos um novo projeto centralizado usando uma framework, como Laravel, Rails, Spring Boot entre outros.

Percebe-se que todas as funcionalidades irão ficar agregadas num mesmo repositório, expondo adapters (componentes de acesso ao BD, componentes de mensagens que produzem e consomem mensagens), APIs REST e Interfaces de Usuário. 

Apesar de ter uma estrutura logicamente modular, o aplicativo é empacotado e implantado como um monólito (algo único, coeso).

Aplicações escritas nessa forma são extremamente comuns. Elas são simples de se desenvolver uma vez que os ambientes se focam na construção de uma única aplicação. Os testes também são mais simples de se desenvolver 
