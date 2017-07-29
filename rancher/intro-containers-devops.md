
# Cinco considerações ao se utilizar containers para os processos de DevOps
Desde a publicação da tecnologia do Docker, em 2013, o uso de containers tem
sido objeto de experimentação e testes em montagem de ambientes de Integração
Contínua (CI).

No momento, precisaremos considerar **cinco** pontos importantes que devem
ser incluídos na discussão sobre a aplicação de containers em processos
de desenvolvimento.

## 1. Definir infra-estrutura de suporte aos containers
Quando temos poucos colaboradores experimentando containers, a criação e 
armazenamento das imagens dos mesmos é de se esperar que sejam feitos 
em ambiente de desenvolvimento local (desktop ou notebook). Quando
a decisão é feita para utilizar containers em ambiente de produção,
contudo, importantes decisões precisam ser seitas em torno da criação e
armazenamento de imagens do Docker.

Antes de iniciar a jornada de implantação de ambiente de produção,
pergunte e responda as seguintes questões:

- **O processo será seguido ao se criar novas imagens?**
    - Como garantiremos que as imagens usadas sejam atualizadas e seguras?
    - Quem será responsável por garantir que as imagens sejam mantidas atualizadas
      e que as atualizações de segurança sejam aplicadas regularmente?

- **Onde serão armazenadas nossas imagens Docker?**
    - Elas são publicamente acessíveis no [Docker Hub](https://hub.docker.com/)/[Docker Store](https://store.docker.com/)?
    - Elas precisam ser mantidas em um repositório privado? Em caso afirmativo, onde serão hospedadas?

- **Como lidaremos com o armazenamento de credenciais em cada imagem do Docker?
  Isso incluirá, mas não está limitado a:**
    - Credenciais para acessar outros recursos do sistema
    - API para sistemas externos, como para monitoramento

- **O nosso ambiente de produção precisa mudar?**
    - Nosso ambiente atual pode suportar uma abordagem baseada em container de forma eficaz?
    - Como administraremos nossos *deploys* (*implantações*) com container?
    - Será que uma abordagem baseada em containers terá um bom custo-benefício?

## 2. Não dispense nada do pipeline de Integração Contínua (CI)
Um dos melhores recursos trazidos pelo Docker é do fato de um container
poder funcionar de maneira mais razoável possível e equivalente
tanto num notebook de um desenvolvedor junior quanto num servidor avançado
ou num datacenter. Portanto, as equipes de desenvolvimento podem ficar
tentadas a assumir que o teste localizado é bom o suficiente e que há um
valor limitado no uso de um pipeline de Integração Contínua (CI) completa.

Importante salientar que o pipeline CI fornece, em verdade, estabilidade e
segurança. Ao executar todas as mudanças de código através de um conjunto
automatizado de testes e garantias, a equipe de desenvolvimento pode
desenvolver uma maior confiança de que as mudanças no código foram testadas
completamente.



# Referências
1. [Introducing Containers into Your DevOps Processes: Five Considerations](http://rancher.com/introducing-containers-devops-processes-five-considerations/)
