
# Cinco considerações ao se utilizar containers para os processos de DevOps
Desde a publicação da tecnologia do Docker, em 2013, o uso de containers tem
sido objeto de experimentação e testes em montagem de ambientes de Integração
Contínua (CI).

No momento, precisaremos considerar **cinco** pontos importantes que devem
ser incluídos na discussão sobre a aplicação de containers em processos
de desenvolvimento.

## 1. Definir infraestrutura de suporte aos containers
Quando temos poucos colaboradores experimentando containers, na criação e 
armazenamento das imagens dos mesmos é de se esperar que sejam feitos 
em ambiente de desenvolvimento local (desktop ou notebook). Quando
a decisão é feita para utilizar containers em ambiente de produção,
contudo, importantes decisões precisam ser feitas em torno da criação e
armazenamento de imagens do Docker.

Antes de iniciar a jornada de implantação de ambiente de produção,
pergunte e responda as seguintes questões:

- **O processo será seguido ao se criar novas imagens?**
    - Como garantiremos que as imagens usadas sejam atualizadas e seguras?
    - Quem será responsável por garantir que as imagens sejam mantidas atualizadas
      e que as atualizações de segurança sejam aplicadas regularmente?

- **Onde serão armazenadas nossas imagens Docker?**
    - Elas são publicamente acessíveis no [Docker Hub](https://hub.docker.com/)/[Docker Store](https://store.docker.com/)?
    - Elas precisam ser mantidas em um repositório privado? 
      Em caso afirmativo, onde serão hospedadas?

- **Como lidaremos com o armazenamento de credenciais em cada imagem do Docker?
  Isso incluirá, mas não está limitado a:**
    - Credenciais para acessar outros recursos do sistema;
    - API para sistemas externos, como para monitoramento.

- **O nosso ambiente de produção precisa mudar?**
    - Nosso ambiente atual pode suportar uma abordagem baseada em container de forma eficaz?
    - Como administraremos nossos *deploys* (*implantações*) com container?
    - Será que uma abordagem baseada em containers terá um bom custo-benefício?

## 2. Não dispense nada do pipeline de Integração Contínua (CI)
Um dos melhores recursos trazidos pelo Docker é do fato de um container
poder funcionar de maneira mais razoável possível e equivalente
tanto num notebook de um desenvolvedor júnior quanto num servidor avançado
ou num datacenter. Portanto, as equipes de desenvolvimento podem ficar
tentadas a assumir que o teste localizado é bom o suficiente e que há um
valor limitado no uso de um pipeline de Integração Contínua (CI) completa.

Importante salientar que o pipeline CI fornece, em verdade, estabilidade e
segurança. Ao executar todas as mudanças de código através de um conjunto
automatizado de testes e garantias, a equipe de desenvolvimento pode
desenvolver uma maior confiança de que as mudanças no código foram testadas
completamente.

## 3. Siga um processo de Deploy (implantação)
Na era de DevOps e CI, temos uma oportunidade de entregar correções de erros,
atualizações e novos recursos para clientes de forma mais rápida e eficiente
do que nunca. Como desenvolvedores, vivemos para resolver problemas e oferecer
qualidade que as pessoas apreciam. No entanto, é importante definir e seguir um
processo que assegure que as etapas-chave não sejam esquecidas na "emoção" do
deploy (aquela ansiedade mortífera 😬).

Em um esforço para maximizar o tempo de atividade e a entrega de novas funcionalidades,
a adoção de um processo como deploy [Blue Green](https://martinfowler.com/bliki/BlueGreenDeployment.html)
é imperativa. A premissa, em relação aos containers, é coexistência de containers novos e
antigos em ambiente de produção. O uso de balanceamento de carga dinâmico para deslocamento
gradual e contínuo do tráfego de produção dos containers antigos para os novos, enquanto
o monitoramento de problemas potenciais, permite um retorno relativamente fácil se problemas 
forem observados nos novos containers.

## 4. Não relaxe no Teste de Integração
Os recipientes podem ser os mesmos, independentemente do sistema host, mas, à medida
que movemos containers de um ambiente para outro, corremos o risco de quebrar
nossas dependendências externas, sejam eles conexões a serviços de terceiros,
bancos de dados ou simplesmente diferenças na configuração de um ambiente para outro.
Por este motivo, é imperativo que executemos testes de integração sempre que uma
nova versão de um container seja implantada num novo ambiente ou quando mudanças num
ambiente possam afetar as interações dos containers internamente.

Os *testes de integração* devem ser executados como parte do processo de CI e novamente
como **etapa final no processo de deploy**. Se estivermos usando o modelo de deploy
*Blue Green* mencionado anteriormente, podemos executar testes de integração em relação
aos novos containers antes de configurar o proxy para incluí-los e, novamente,
uma vez que o proxy foi direcionado para apontar para os novos containers.

## 5. Certifiquemos de que o nosso ambiente de produção seja escalável
A facilidade com que os containers podem ser criados e destruídos é um benefício
definitivo dos containers, até que tenhamos que gerenciar esses containers em um
ambiente de produção. A tentativa de fazer isso manualmente com mais de 1 ou 2 containers
seria impraticável. Seria ainda mais impossível num cenário com diversos containers
diferentes, dimensionados em diferentes níveis.

Avaliar e decidir sobre uma solução de gerenciamento de container é uma das decisões
mais importantes que teremos que tomar em todo o processo de DevOps. Há varias soluções,
como:
- [Rancher](http://rancher.com/): traz por padrão o **Cattle**, gerenciador de cluster de containers em Docker.
Entretanto, pode criar **clusters de Kubernetes**, [Apache Mesos](http://mesos.apache.org/) e etc;
- [Kubernetes](https://kubernetes.io/): gerenciador de cluster de container em Docker, desenvolvido pela Google;
- [Juju](https://jujucharms.com/): semelhante ao Rancher, mas faz uso do LXC (outra tecnologia de container);
- [OpenShift Origin](https://www.openshift.org/): traz por padrão o Kubernetes.

Devemos, no entanto, tomar cuidado ao implementar, pois nem toda solução, por mais que seja rápida e eficiente,
vai nos livrar de falhas.

# Referências
1. [Introducing Containers into Your DevOps Processes: Five Considerations](http://rancher.com/introducing-containers-devops-processes-five-considerations/)
