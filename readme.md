# Minhas anotações sobre o curso "Descomplicando o Docker"


# Agradecimento

Deixo aqui meu agradecimento ao [Jeferson](https://twitter.com/badtux_) da [LinuxTips](www.linuxtips.io) pelas iniciativas que ele vem promovendo na comunidade. Graças a [uma dessas iniciativas](https://youtu.be/RtQj19BYu-0) tive acesso gratuito a esse curso fantástico. Espero um dia poder retribuir ajudando a comunidade de alguma forma.


# Sobre este material

Criei este material com o objetivo de registrar pontos importantes abordados no curso, já que minha memória vive dando stackoverflow. 

Como não poderia deixar de ser, a maior parte do conteúdo foi baseada nas aulas do curso, embora alguns assuntos tenham sido complementados com material externo (desculpem os autores, mas esqueci de anotar as referências), além da própria documentação do Docker.

Importante dizer que procurei descrever alguns pontos a partir do meu entendimento sobre o assunto, então provavelmente deve haver erros/imprecisões. 

Se você chegou até aqui em busca de um bom material de referência, indico o livro "Descomplicando o Docker", que recentemente virou um projeto opensource e está disponível no [GitHub](https://github.com/badtuxx/DescomplicandoDocker). 

---
# Tabela de Conteúdos

- [O que é Docker?](#o-que-é-docker)
  - [Instalando o docker](#instalando-o-docker)
- [Imagens Docker](#imagens-docker)
  - [Camadas de uma imagem](#camadas-de-uma-imagem)
  - [Principais comandos](#principais-comandos)
- [Containers](#containers)
  - [Linha do tempo dos containers](#linha-do-tempo-dos-containers)
  - [Fluxo de criação de um container](#fluxo-de-criação-de-um-container)
  - [Principais comandos](#principais-comandos)
  - [Limitando CPU e RAM dos containers](#limitando-cpu-e-ram-dos-containers)
- [Docker commit](#docker-commit)
- [Volumes](#volumes)
  - [Tipo Bind](#tipo-bind)
    - [Comando](#comando)
  - [Tipo Volume](#tipo-volume)
    - [Comandos](#comandos)
  - [Forma antiga de usar volumes: criando containers data-only](#forma-antiga-de-usar-volumes-criando-containers-data-only)
  - [Desafio: Simulando um backup manual do postgres](#desafio-simulando-um-backup-manual-do-postgres)
- [Dockerfiles](#dockerfiles)
  - [Estrutura de um Dockerfile](#estrutura-de-um-dockerfile)
  - [Construindo uma imagem a partir de um Dockerfile](#construindo-uma-imagem-a-partir-de-um-dockerfile)
  - [Instruções do dockerfile](#instruções-do-dockerfile)
    - [Criando um Dockerfile que gera uma imagem que não executa nada](#criando-um-dockerfile-que-gera-uma-imagem-que-não-executa-nada)
    - [Adicionando as instruções ENTRYPOINT e CMD](#adicionando-as-instruções-entrypoint-e-cmd)
    - [A instrução COPY](#a-instrução-copy)
    - [Outras instruções disponíveis](#outras-instruções-disponíveis)
  - [MultiStage](#multistage)
- [Docker Hub](#docker-hub)
  - [Como usar?](#como-usar)
- [Docker Registry](#docker-registry)
  - [Principais comandos](#principais-comandos)
- [Docker Machine](#docker-machine)
  - [Principais comandos](#principais-comandos)
  - [Variáveis de ambiente](#variáveis-de-ambiente)
- [Docker Swarm](#docker-swarm)
  - [Managers e Workers](#managers-e-workers)
  - [Iniciando um cluster Swarm](#iniciando-um-cluster-swarm)
  - [Outros comandos](#outros-comandos)
- [Docker Services](#docker-services)
  - [Comandos](#comandos)
- [Docker Network](#docker-network)
  - [Comandos](#comandos)
- [Docker Secrets](#docker-secrets)
  - [Principais comandos](#principais-comandos)
- [Automatizando serviços Docker](#automatizando-servios-docker)
  - [Compose files](#compose-files)
    - [Exemplo de compose file para o docker stack:](#exemplo-de-compose-file-para-o-docker-stack)
    - [Exemplo de compose file para o docker-compose:](#exemplo-de-compose-file-para-o-docker-compose)
  - [Docker Stack](#docker-stack)
    - [Comandos](#comandos)
  - [Docker-compose](#docker-compose)
    - [Principais comandos](#principais-comandos)

## O que é Docker?

Criado pela empresa dotCloud (atual Docker inc), o docker é uma tecnologia que permite isolar e gerenciar processos e recursos de máquina via containers. Foi baseado no [LXC](https://linuxcontainers.org/#LXC) e teve seu código aberto em 2013, tendo sua 1ª release 15 meses depois.

### Instalando o docker

O passo a passo para a instalação do Docker pode variar dependendo do sistema operacional, portanto é sempre bom consultar a [documentação oficial](https://docs.docker.com/get-docker/). Para ambientes linux existe um [script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script) que facilita este processo de instalação:

```bash
$ curl -fsSL https://get.docker.com | bash
```

Após a instalação é necessário conceder permissões especiais para que o Docker daemon possa executar os comandos, conforme mostra a [documentação](https://docs.docker.com/engine/install/linux-postinstall/).


## Imagens Docker

É um conjunto de instruções que permitem a criação de um container. Fazendo um paralelo com a programação, a imagem seria a classe e o container a sua instância.


### Camadas de uma imagem

![](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

Imagens docker são contruídas a partir de uma série de camadas empilhadas, em que todas, exceto a última, são marcadas como "read-only". Isso garante que nenhuma instrução gravada na camada anterior possa ser desfeita, forçando a criação de uma nova camada a cada instrução adicional, mantendo um controle sobre as modificações realizadas na imagem.

Quando um container é criado uma nova camada "read-write" é gerada e todas as modificações do container passam a ser feitas nela. Ao deletar o container esta camada é eliminada junto com ele, mantendo as camadas da imagem inalteradas.

![](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)

Esse conceito é conhecido como "[copy-on-write](https://pt.wikipedia.org/wiki/C%C3%B3pia_em_grava%C3%A7%C3%A3o)" e é o que permite a reutilização de uma mesma imagem para a criação de diversos containers.

### Principais comandos

```bash
# Lista imagens disponíveis no host
$ docker image ls

# Cria uma imagem a partir de um Dockerfile presente na pasta atual (.)
$ docker image build .

# Atribui uma nova tag à imagem
$ docker image tag hello-world (imagem original) hello-terra (nova imagem)

# Exibe o histórico de instruções utilizadas para a criação da imagem
$ docker image history [REPOSITÓRIO ou IMAGE ID]

# Exibe informações sobre a imagem
$ docker image inspect [REPOSITÓRIO ou IMAGE ID]

# Baixa uma imagem de um registry (Docker Hub, por exemplo)
$ docker image pull [REPOSITÓRIO ou IMAGE ID]

# Envia uma imagem para um registry
$ docker image push [REPOSITÓRIO ou IMAGE ID]

# Exclui uma ou mais imagens
$ docker image rm [REPOSITÓRIO ou IMAGE ID, ...]

# Remove TODAS as imagens que não estão sendo utilizadas por containers
$ docker image prune

```


## Containers

São uma forma de isolamento de recursos em um sistema. Esse isolamento pode ser lógico (processos, usuários, rede, etc), de responsabilidade do namespaces, ou físico (cpu, ram, io, etc), controlado pelo cgroup.


### Linha do tempo dos containers

*   1979 - Chroot (Unix V7)
*   2000 - FreeBSD Jails
*   2004 - Solaris Containers
*   2005 - OpenVZ (Open Virtuzzo)
*   2008 - LXC (LinuX Container)
*   2013 - Docker

### Fluxo de criação de um container

1. O Docker client entra em contato com o Docker daemon;
2. O Docker daemon faz o pull da imagem do DockerHub;
3. O Docker daemon cria o novo container baseado na imagem baixada;
4. O container criado entra em execução;


### Principais comandos

```bash
# Lista containers ativos
$ docker container ls

# Lista todos os containers (ativos e inativos)
$ docker container ls -a

# Cria/executa um container
$ docker container run -ti (terminal interativo) [IMAGEM]

# Sai do container matando o processo
$ [ctrl + d]

# Sai do container sem matar o processo ("minimizar")
$ [ctrl + p + q]

# Volta ao container que foi "minimizado"
$ docker container attach [CONTAINER ID ou NAME]

# Executa o container como daemon (em 2º plano)
$ docker container run -d (daemon) [IMAGEM]

# Executa um comando no container
$ docker container exec -ti [CONTAINER ID ou NAME] [COMANDO]

# Para a execução do container
$ docker container stop [CONTAINER ID ou NAME]

# Inicia o container
$ docker container start [CONTAINER ID ou NAME]

# Reinicia o container
$ docker container restart [CONTAINER ID ou NAME]

# Exibe informações sobre o container
$ docker container inspect [CONTAINER ID ou NAME]

# Interrompe o container sem finalizar o processo
$ docker container pause [CONTAINER ID ou NAME]

# Retoma o container interrompido
$ docker container unpause [CONTAINER ID ou NAME]

# Exibe logs do container
$ docker container logs -f [CONTAINER ID ou NAME]

# Exclui um container inativo
$ docker container rm [CONTAINER ID ou NAME]

# Força a exclusão de um container que está ativo
$ docker container rm -f (força) [CONTAINER ID ou NAME]

# Força a exclusão de todos os containers inativos
$ docker container prune

# Exibe estatísticas de consumo de CPU, RAM, etc de um container:
$ docker container stats [CONTAINER ID ou NAME]

# Exibe os processos em execução no container:
$ docker container top [CONTAINER ID ou NAME]
```

DICA:
É possível manipular um container específico sem ter que digitar a ID completa. Apenas uma parte do código já é suficiente para o docker fazer a identificação.


### Limitando CPU e RAM dos containers
```bash
# Cria/executa container com limite de memória e CPU
$ docker container run -d -m (memória) 128M (128MB de limite) --cpus 0.5 (50% de 1 core de CPU) nginx (imagem)

# Atualiza o limite de CPU do container
$ docker container update --cpus 0.2 [CONTAINER ID ou NAME]

```


## Docker commit
 
O Docker commit pode ser útil para salvar modificações de um container (instalações de pacotes, alteração de configurações durante execução, etc) e transformá-las em uma nova imagem.
 
```bash
# A estrutura do comando se assemelha ao commit do git 
$ docker commit -m "mensagem descrevendo as alterações" [CONTAINER ID] [REPOSITÓRIO:TAG]
 
```
Apesar da praticidade do commit, é sempre uma boa prática criar um novo Dockerfile, gerando as modificações necessárias de forma transparente.



## Volumes

![](https://docs.docker.com/storage/images/types-of-mounts-bind.png)

 
São espaços que permitem salvar dados gerados pelos containers. Eles são armazenados diretamente na máquina host, o que garante a preservação dos dados, mesmo que os containers sejam destruídos. Um único volume pode, inclusive, ser compartilhado entre vários containers. Podem ser do tipo *bind* ou do tipo *volume*.
  
 
### Tipo Bind
 
Uma pasta da máquina host é montada no container como uma simples pasta compartilhada, que não é gerida pelo Docker.
 
 
#### Comando
 
```bash
# Cria um bind mount:
$ docker container run -ti --mount type=bind,src=/opt/giropops,dst=/giropops debian
```
 
### Tipo Volume
 
Uma nova pasta é criada dentro da pasta do Docker storage (/var/lib/docker/volumes), na máquina host e seu gerenciamento passa a ser feito pelo Docker via comandos.
 
 
#### Comandos
 
```bash
# Cria um novo volume
$ docker volume create dbdados
 
# Vincula um volume a um novo container
$ docker container run -d --name psql1 -p 5432:5432 --mount type=volume,src=dbdados,dst=/var/lib/postgresql/data -e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker -e POSTGRES_DB=docker postgres
 
# Vincula um volume somente leitura a um novo container
$ docker container run -d --mount type=volume,src=[VOLUME],dst=[PASTA DESTINO],ro [IMAGEM]
 
# Lista volumes
$ docker volume ls
 
# Exclui um ou mais volumes
$ docker volume rm [VOLUME NAME VOLUME NAME...]
 
# Exclui todos os volumes não vinculados a containers
$ docker volume prune
 
# Inspeciona os detalhes do volume
$ docker volume inspect [VOLUME NAME]
```
 
 
### Forma antiga de usar volumes: criando containers data-only
 
```bash
# Cria volume como um container
$ docker create -v /data --name dbdados debian
 
# Cria/executa container e vincula ao container de volume
$ docker run -d --name pgsql -p 5432:5432 --volumes-from dbdados -e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker -e POSTGRES_DB postgres
```
 
 
### Desafio: Simulando um backup manual do postgres
 
```bash
$ docker container run -it --mount type=volume,src=dbdados,dst=/data --mount type=bind,src=/opt/backup,dst=/backup debian tar -cvf /backup/bkp-banco.tar /data
```


## Dockerfiles


São arquivos que contêm todos os comandos necessários para a construção de imagens Docker.


### Estrutura de um Dockerfile

```dockerfile
# Conteúdo do arquivo Dockerfile

# Imagem base, disponível no Dockerhub
FROM debian

# Adiciona metadados à imagem
LABEL app="Dockerfile01"

# Adiciona uma variável de ambiente
ENV CURSO="Descomplicando o Docker"

# Executa comandos durante a criação da imagem
RUN apt-get update && apt-get install -y stress && apt-get clean

# Executa comandos após a criação da imagem
CMD stress --cpu 1 --vm-bytes 64M --vm 1
```


### Construindo uma imagem a partir de um Dockerfile

```bash
$ docker image build -t (tag) teste:1.0 (REPOSITÓRIO:TAG) . (CAMINHO DO DOCKERFILE)
```
 
 
### Instruções do dockerfile
 
 
#### Criando um Dockerfile que gera uma imagem que não executa nada
 
```dockerfile
# Conteúdo do Dockerfile incompleto
FROM debian
 
LABEL description="Webserver"
 
RUN apt-get update && apt-get install -y apache2 && apt-get clean
 
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"
 
VOLUME /var/www/html/
EXPOSE 80
```
Ao criar um container baseado nessa imagem, apesar do apache ser instalado e configurado com sucesso, seu principal processo, responsável por iniciar o servidor, não é executado.
 
 
#### Adicionando as instruções ENTRYPOINT e CMD
 
As duas instruções são responsáveis por definir um comando a ser executado ao instanciar um container. Podem ser usadas em conjunto (neste caso o CMD receberia os parâmetros do comando informado no ENTRYPOINT) ou isoladamente. A forma de execução também varia, podendo ser no "modo exec", com comando e parâmetros informados numa lista de strings, semelhante a uma lista no python (preferível, de acordo com a documentação do Docker), ou no "modo shell", da mesma forma que se usaria no bash.
 
```dockerfile
# Conteúdo do novo Dockerfile 
FROM debian
 
LABEL description="Webserver"
 
RUN apt-get update && apt-get install -y apache2 && apt-get clean
 
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"
 
VOLUME /var/www/html/
EXPOSE 80
 
ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]
```
Nesse novo Dockerfile a instrução ENTRYPOINT informa o comando apachectl para executar o processo padrão da imagem, que inicia o servidor Apache e o CMD informa os parâmetros desse comando, -D e FOREGROUND, responsáveis por manter a execução do processo em primeiro plano.
 
 
#### A instrução COPY
 
É responsável por copiar arquivos ou pastas da origem (local no host) e adicioná-los no destino (container).
 
```dockerfile
# Conteúdo do Dockerfile 
FROM debian
 
LABEL description="Webserver"
 
RUN apt-get update && apt-get install -y apache2 && apt-get clean
 
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"
 
COPY index.html /var/www/html/
 
VOLUME /var/www/html/
EXPOSE 80
 
ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]
```
O arquivo acima, além de iniciar o servidor apache, copia o arquivo index.html do host (criado na mesma pasta do Dockerfile) para o caminho /var/www/html/, dentro do container. Com isso, ao iniciar o servidor Apache, é possível acessar a página copiada:
 
```bash
# Cria a imagem do apache com base no Dockerfile
$ docker image build -t meu_apache:1.0 .
 
# Constrói um container a partir da imagem gerada
$ docker container run -d -p 8080:80 meu_apache:1.0
 
# Acessa o index.html na porta 8080 do servidor
$ curl localhost:8080
```
 
 
#### Outras instruções disponíveis
 
```dockerfile
# Conteúdo do Dockerfile 
FROM debian
 
LABEL description="Webserver"
 
RUN apt-get update && apt-get install -y apache2 && apt-get clean
 
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"
 
ADD index.html /var/www/html/
 
USER root
 
WORKDIR /var/www/html/
 
VOLUME /var/www/html/
EXPOSE 80
 
ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]
```
 
##### Instrução ADD
 
Funciona de forma similar ao COPY, mas possui funcionalidades extras:
 
*   Ao manipular arquivos .tar, ele faz a descompactação e salva no destino o conteúdo já descompactado;
*   Permite o uso de arquivos remotos (links) como origem, fazendo o download e enviando posteriormente o arquivo baixado para a pasta de destino.
 
Para evitar confusões, o ideal é usar o ADD apenas nessas situações específicas.
 
 
##### Instrução USER
 
Utilizada quando se quer mudar o usuário que executará as instruções na imagem. Por padrão o usuário é sempre o root (não recomendável manter).
 
 
##### Instrução WORKDIR
 
Define o diretório de trabalho padrão da imagem.
 
##### Instrução HEALTHCHECK
 
É utilizada quando se quer testar um container e checar de tempos em tempos se ele continua funcionando. O resultado da checagem pode ser acompanhado na coluna STATUS da listagem de containers ativos, sendo exibido "healthy" enquanto estiver tudo funcionando bem ou "unhealthy" quando algum erro ocorrer.
 
Apesar de ser possível utilizar o HEALTHCHECK diretamente no Dockerfile, é preferível configurá-lo em um docker-compose.
 
 
```dockerfile
# Conteúdo do Dockerfile 
FROM debian
 
LABEL description="Webserver"
 
RUN apt-get update && apt-get install -y apache2 curl && apt-get clean
 
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"
 
COPY index.html /var/www/html/
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost || exit 1
 
VOLUME /var/www/html/
EXPOSE 80
 
ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]
```
 
---
### MultiStage
 
Com o multistage é possível mesclar o build de várias imagens, com cada uma delas sendo responsável por uma etapa da construção da imagem final.
 
```dockerfile
# Conteúdo do Dockerfile
FROM golang as builder
 
ENV GO111MODULE=auto
 
WORKDIR /projeto
COPY . /projeto
 
RUN go build -o app
 
 
FROM alpine
 
WORKDIR /binario
COPY --from=builder /projeto/app /binario
 
ENTRYPOINT ./app
```
No exemplo acima, a 1ª imagem (builder) é usada para compilar um projeto em Go. Em seguida a 2ª imagem realiza a cópia do binário gerado e o disponibiliza para execução. Nesse processo houve uma redução significativa do tamanho da imagem final (de ~800mb para ~8mb), pois a 1ª imagem, que precisou de todo o ambiente necessário para a compilação do projeto, pôde ser descartada para dar lugar a uma imagem mais leve, responsável apenas pela execução do binário.
 


## Docker Hub
 
É uma plataforma web que permite o compartilhamento de imagens entre usuários Docker (uma espécie de Github do Docker).
 
 
### Como usar?
 
Antes de tudo, é obrigatório que o usuário tenha uma conta ativa no [Docker Hub](https://hub.docker.com/). Para que seja possível compartilhar uma imagem com outros usuários é necessário fazer o login no Docker Hub através do terminal:
 
```bash
# Serão solicitadas as credenciais do Docker Hub (usuário e senha)
$ docker login
```
 
O próximo passo é vincular a imagem ao usuário do Docker Hub adicionando o nome do usuário como prefixo ([USUÁRIO]/) ao nome da imagem:
 
```bash
# Gera uma nova tag que vincula uma imagem local ao usuário do Docker Hub
$ docker image tag apache_customizado:1.0 diogodev/apache_customizado:1.0
```
 
A última etapa é subir a imagem para o Docker Hub:
 
```bash
# Ao fim da execução a imagem estará disponível no site
$ docker image push diogodev/apache_customizado:1.0
```
 
 
---
## Docker Registry
 
É um servidor opensource que permite armazenar e compartilhar imagens Docker. O registry é uma alternativa ao Docker Hub e pode ser bastante útil para integrações de CI/CD.
 
 
### Principais comandos
 
```bash
# Cria/sobe um servidor registry
$ docker container run -d -p 5000:5000 --restart=always --name registry registry:2
 
# Loga no registry, caso ele tenha uma autenticação configurada (por padrão não tem)
$ docker login [ENDEREÇO DO REGISTRY]
 
# Gera uma nova tag que vincula uma imagem ao endereço do registry
$ docker image tag apache_customizado:1.0 localhost:5000/apache_customizado:1.0
 
# Envia uma imagem para o registry
$ docker image push localhost:5000/apache_customizado:1.0
 
# Lista as imagens no registry
$ curl localhost:5000/v2/_catalog
 
# Lista as tags de uma imagem no registry
$ curl localhost:5000/v2/apache_customizado/tags/list
 
# Baixa uma imagem do registry
$ docker image pull localhost:5000/apache_customizado:1.0
 
# Para o registry e exclui os dados salvos
$ docker container stop registry && docker container rm -v registry
```
 
 
## Docker Machine
 
![](https://docs.docker.com/machine/img/machine.png)
 
É uma ferramenta para gerenciamento distribuído de Docker hosts. Com ela é possível acessar e gerenciar hosts remotos através de [drivers](https://docs.docker.com/machine/drivers/) específicos para cada plataforma (AWS, Azure, GCP, VirtualBox, etc). Os detalhes para sua instalação podem ser encontrados na [documentação oficial](https://docs.docker.com/machine/install-machine/).
 
 
 
### Principais comandos
 
```bash
# Verifica a versão do docker-machine
$ docker-machine version
 
# Cria uma nova Docker machine
$ docker-machine create --driver virtualbox vb-machine
 
# Lista as máquinas criadas
$ docker-machine ls
 
# Exibe o ip da máquina
$ docker-machine ip vb-machine
 
# Conecta à máquina
$ docker-machine ssh vb-machine
 
# Exibe informações sobre a máquina
$ docker-machine inspect vb-machine
 
# Interrompe a execução da máquina
$ docker-machine stop vb-machine
 
# Verifica o status atual da máquina (se executando ou parada)
$ docker-machine status vb-machine
 
# Inicia a execução de uma máquina
$ docker-machine start vb-machine
 
# Remove uma máquina
$ docker-machine rm vb-machine
```
 
 
### Variáveis de ambiente
 
É possível setar através do shell um grupo de variáveis de ambiente para informar ao Docker a qual host ele deverá se conectar. 
 
O docker-machine possui o seguinte comando para exibição das variáveis da máquina selecionada:
 
```bash
# Exibe as variáveis da docker machine "vb-machine"
$ docker-machine env vb-machine
```
 
Ele vai retornar uma lista de variáveis, que podem ser copiadas e coladas no shell:
 
```bash
export DOCKER_TLS_VERIFY="1" # para habilitar ou não a verificação TLS
export DOCKER_HOST="tcp://172.16.62.130:2376" # endereço do host
export DOCKER_CERT_PATH="/Users/<yourusername>/.docker/machine/machines/vb-machine" # path dos certificados que foram copiados para a máquina
export DOCKER_MACHINE_NAME="vb-machine" # nome da máquina
```
 
Também é possível usar o eval para fazer isso automaticamente:
```bash
# Executa os exports resultantes do comando 'docker-machine env':
$ eval $(docker-machine env vb-machine)
```
 
Dessa forma, qualquer comando executado no docker client será repassado para o host remoto.
 
É possível remover as variáveis setadas com o seguinte comando:
```bash
$ eval $(docker-machine env -u)
```
 

## Docker Swarm
 
Docker Swarm é uma ferramenta nativa do Docker para orquestração de containers que permite subir e gerenciar vários containers em diferentes docker hosts.
 
 
### Managers e Workers
 
![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)
Um cluster Swarm é composto por vários docker hosts (nós) rodando no "modo swarm". Esses hosts são divididos em ***managers***, cujas funções são gerenciar e delegar tasks para os membros do cluster, e ***workers***, responsáveis por executar os serviços no cluster. Os nós podem, inclusive, assumir as duas funções. A [documentação](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#manager-nodes) do docker recomenda o número máximo de 7 managers por cluster.
 
Para assegurar a resiliência e disponibilidade do cluster é preciso sempre manter **mais da metade** dos managers em funcionamento. Um cluster com N managers deve ter uma tolerância de perda de no máximo (N-1)/2 managers. Exemplos: um cluster com 3 managers só pode perder 1 deles ((3-1)/2=1), um com 4 também pode perder apenas 1 e um com 5 managers apenas 2. Uma [dica](https://docs.docker.com/engine/swarm/admin_guide/#add-manager-nodes-for-fault-tolerance) é sempre optar por números ímpares de managers para manter um maior "quórum".
 
 
### Iniciando um cluster Swarm
 
```bash
# Inicia um cluster swarm. O parâmetro indica qual interface de rede usar, caso exista mais de uma.
$ docker swarm init --advertise-addr 192.168.0.8
```
Ao executar o comando acima, o nó onde ele foi executado vira um manager 'Leader' do cluster. Como resultado também é gerado um comando para rodar em cada host a ser adicionado ao cluster:
 
 ```bash
# Por padrão o token gerado adiciona os nós como workers.
$ docker swarm join --token SWMTKN-1-1u0vvlt0zjrxpkp3z7b8syyi67efycs7szsz6l011dt62sqv4i-7ua1in4xro3c6n33yen1pmvmo 192.168.0.8:2377
```
 
Também é possível gerar o token para um tipo específico de nó: 
```bash
# Comando responsável por gerar o token de acordo com cada tipo de nó
$ docker swarm join-token [manager ou worker]
 ```
 
 
 ### Outros comandos
 
 ```bash
# Lista todos os nós do cluster
$ docker node ls
 
# Exibe informações sobre um nó
$ docker node inspect [NÓ] 
 
# Promove um nó a manager
$ docker node promote [NÓ]
 
# Rebaixa um nó a worker
$ docker node demote [NÓ]
 
# Força a exclusão de um nó (um manager só pode ser excluído após virar worker)
$ docker node rm -f [NÓ]
 
# Atualiza disponibilidade do nó
# ative = disponível para execução de novos containers
# pause = impede a inclusão de novos containers
# drain = encerra qualquer container executando (outros nós assumem as tasks)
$ docker node update --availability [ative | pause | drain]
 
# Remove um nó worker do cluster
$ docker swarm leave
 
# Remove um nó manager do cluster
$ docker swarm leave -f
 ```
 
DICA: é possível simular a criação de um cluster swarm [neste site](https://labs.play-with-docker.com/).
 
 
 
## Docker Services
 
![](https://docs.docker.com/engine/swarm/images/services-diagram.png)
 
São serviços do docker swarm especializados em gerenciar e distribuir tarefas entre os nós do cluster.
 
 
### Comandos
 
```bash
# Cria um novo serviço com 3 réplicas (containers)
$ docker service create --name webserver --replicas 3 -p 8080:80 --mount type=volume,src=teste,dst=/app  nginx
 
# Lista os serviços ativos
$ docker service ls
 
# Lista os containers do serviço
$ docker service ps webserver
 
# Exibe informações sobre o serviço (o pretty melhora a forma de visualização)
$ docker service inspect webserver --pretty
 
# Redimensiona a quantidade de containers do serviço
$ docker service scale webserver=10
 
# Exibe os logs de todos os containers do serviço
$ docker service logs -f webserver
 
# Remove o serviço do cluster
$ docker service rm -f webserver
```
 
IMPORTANTE: O docker service não sincroniza os volumes dos nós. Para que isso seja possível é necessário usar outras ferramentas, como o NFS.
 
 

## Docker Network
 
É o comando responsável por criar redes virtuais, possibilitando o isolamento entre diferentes serviços docker. Uma vantagem das docker networks é a capacidade de nomeá-las, dispensando a necessidade de manipular ips.
 
### Comandos
 
```bash
# Cria uma nova rede overlay que interconecta todos os containers de determinado serviço 
$ docker network create -d overlay giropops
 
# Lista todas as redes disponíveis
$ docker network ls
 
# Exibe informações sobre uma rede específica
$ docker network inspect giropops
 
# Remove uma rede
$ docker network rm giropops
```
 
 
## Docker Secrets
 
Secrets são utilizados para gerenciar dados sensíveis e necessários aos containers, como senhas e tokens, evitando que sejam expostos em arquivos, por exemplo. O conteúdo dos secrets fica acessível apenas dentro dos próprios containers, no caminho */run/secrets/*.
 
***É importante ressaltar que os secrets estão disponíveis apenas em serviços do docker swarm, não sendo possível a utilização em containers fora de um cluster swarm.***
 
 
### Principais comandos
 
```bash
 
# Cria um novo secret a partir da saída do terminal
$ echo "ISTO É UM SEGREDO" | docker secret create frase -
 
# Cria um novo secret a partir de um arquivo
$ docker secret create frase2 ./frase.txt
 
# Lista secrets
$ docker secret ls
 
# Exibe informações sobre o secret
$ docker secret inspect frase
 
# Exclui o secret
$ docker secret rm frase
 
# Cria um service setando um secret
$ docker service create --name webserver -p 8080:80 --secret frase nginx
 
# Atualiza o service adicionando outro secret
$ docker service update --secret-add frase2 webserver
 
# Cria um service setando um secret com permissões restritas derivado de outro secret
$ docker service create --name webserver -p 8080:80 --secret src=frase,target=novo-secret,uid=200,gid=200,mode=0400 nginx
 
# Atualiza o service removendo um secret
$ docker service update --secret-rm frase webserver
```
 
 
## Automatizando serviços Docker
 
### Compose files
 
São arquivos YAML utilizados para configurar e executar serviços docker. A partir da versão 3 eles passaram a ser utilizados tanto pelo docker-compose quanto pelo docker stack, embora algumas instruções sejam [específicas](https://docs.docker.com/compose/compose-file/compose-file-v3/) para cada comando.
 
#### Exemplo de compose file para o docker stack:
```yaml
version: "3.7"
services:
  web:
    image: nginx
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
    - "8080:80"
    networks:
    - webserver
networks:
  webserver:
```
 

#### Exemplo de compose file para o docker-compose:

```yaml
version: '3.7'
services:
  web:
    build: .
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db/postgres
      - DEBUG=1
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - 8000:8000
    depends_on:
      - db

  db:
    image: postgres:11
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres

volumes:
  postgres_data:
```


### Docker Stack
 
É uma ferramenta escrita em go, builtin do docker (não precisa ser instalada), usada para criar e gerenciar serviços do Docker Swarm.
 
#### Comandos 
 
```bash
# Faz o deploy de uma stack ou atualiza uma já existente
$ docker stack deploy -c docker-compose.yml my_app
 
# Lista as stacks
$ docker stack ls
 
# Lista os serviços da stack
$ docker stack services my_app
 
# Lista as tarefas da stack em execução
$ docker stack ps my_app
 
# Remove uma ou mais stacks
$ docker stack rm my_app
```
 
 
### Docker-compose
 
É uma ferramenta Docker escrita em python, utilizada para criação e integração de vários containers numa única aplicação. Sua principal diferença em relação ao docker stack é a capacidade de "buildar" imagens, o que o torna ideal para ambientes de desenvolvimento. 
 
Como o Docker Compose não é uma ferramenta builtin do Docker, precisa ser instalada. Todo o processo de instalação pode ser conferido na [documentação oficial do Docker](https://docs.docker.com/compose/install/).
 
 
#### Principais comandos

```bash
# Inicia os containers em 2º plano
$ docker-compose up -d

# Escala um serviço:
$ docker-compose up --scale web=3

# Para a execução dos containers do compose:
$ docker-compose stop

# Inicia a execução dos containers do compose:
$ docker-compose start

# Verifica os logs dos containers do compose:
$ docker-compose logs

# Remove os container e também seus volumes
$ docker-compose down --volumes
```