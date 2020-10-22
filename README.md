# Estudo docker

### Breve resumo de entendimento do docker

*Docker é praticamente responsável por gerenciar maquinas criadas em cima de um sistema operacional com uma configuração especifica geralmente para um serviço especifico como por exemplo: uma maquina que roda uma uma instância do banco de dados postgres, garantindo assim que as configurações usadas para que o mesmo possa executar em uma maquina sejam as mesmas usadas para que o mesmo possa rodar em outra maquina, garantido assim que não irá haver diferenças da maquina docker que roda em um maquina para outra*

---
### Inciando o uso do docker

* Imagem: É uma maquina com configurações ja definidas para um determinado tipo de serviço.

* Docker Hub: Site do próprio docker onde podemos encontrar imagem ja prontas de serviços específicos como exemplificado no exemplo acima podemos conseguir uma imagem especifica referente ao serviço de banco de dados postgres.

* Dockerfile: Arquivo que precisamos criar no inicio do uso de  criação de uma maquina Docker para definimos as configurações que queremos na nossa maquina e a imagem que usaremos para a nossa maquina docker.

#### Exemplo de configuração de um dockerfile:

1. criar pasta api/db
2. criar arquivo Dockerfile na pasta db
3. adicionar o seguinte conteúdo:
    
    > FROM mysql

    > ENV MYSQL_ROOT_PASSWORD root

    * *FROM mysql* faz referencia a imagem **mysql** que está no DockerHub.
    * *ENV MYSQL_ROOT_PASSWORD root* define uma configuração específica referente as variáveis de ambiente que podem ser configuradas para imagem 
    **mysql**.

4. Comando de construção de uma imagem:

    > docker build -t mysql-image -f api/db/Dockerfile .

    * *docker build* : usado para construir a imagem
    * *-t* : especifica um nome para a imagem que esta sendo construída.
    * *-f* : especifica a localização do arquivo Dockerfile para gerar a imagem.
    * ( *.* ) : No final significa que o contexto para gerar a imagem sera o da pasta onde esta sendo executado o comando.

5. Comando para ver as imagens docker disponíveis para uso.

    > docker images

6. Comando para executar a imagem:

    > docker run -d --rm --name mysql-container mysql-image

    * *docker run* : executa a imagem
    * *-d* : significa **detach** em português **separar**, especifica que a execução do processo não ficará preso ao terminal
    * *rm* : especifica que se houver o container ja existir ele deve remove-lo antes de criar o novo container.
    * *-name* : define o nome do container
    * mysql-image : representa o nome da imagem docker que sera executada dentro container.

7. Criar aquivo de script sql que sera executado no container

    > Crete database if not exists mysql-bd;

    > Use mysql-bd;

    > Create table if not exists user(id INT(11) AUTO INCREMENT, name VARCHAR(255), PRIMARY KEY (id));

8. Comando para executar script sql dentro do container mysql-container:

    > docker exec -i mysql-container mysql -uroot -root < api/db/script.sql

    * *docker exec* : significa que será executado um comando dentro de um container que está rodando.
    * *-i* : significa que estamos executando uma comando no modo interativo, serve para execução de processos interativos como um shell, como se a tag

9. Comando para executar bash/terminal de dentro do container

    > docker exec -it mysql-container /bin/bash

    * *-it* : especifica o o TTY ou um terminal onde poderão ser executados comandos
    * *mysql-container* : container onde será aberto o terminal.
    * /bin/bash : tipos de comando que serão executados no terminal, nesse cao comando do tipo bash.

**OBS.: Quando paramos a execução de um container docker tudo que é feito dentro desse container é perdido, e as vezes isso não é muito bom! mas existe uma forma de resolver isso!**

10. Ao executar o comando do item 6 devemos adicionar a tag *-v* seguido do caminho onde queremos que seja criado o volume com os dados de configurações a ser usados pelo container:

    > docker run -d -v $(pwd)/api/db/data:/var/lib/mysql --rm --name mysql-container mysql-image

    * *$(pwd)* - retorna o diretório atual
    * *$(pwd)*/api/db/data - Especifica o diretório onde esta os dados da maquina local
    * */var/lib/mysql* - Especifica onde esta a pasta mysql dentro do container
    * $(pwd)/api/db/data:/var/lib/mysql - especifica a referencia entre os dados da pasta da maquina e para onde devem ser enviados os dados na criação do container, dessa forma é como as duas ficam linkadas enquanto o o container estiver rodando logo toda configuração que for realizada na maquina container sera refletida nas pasta de dados da maquina, sendo assim quando o container for executado novamente os dados podem ser recuperados e usados.

11. Comando para visualizar dados referente ao container

    > docker inspect nome_container
    
### Curiosidades

*O docker cria uma rede que permite que uma maquina docker possa ser visualizada por outra maquina docker através dos seus IPs, mas é possível também fazer com que uma maquina visualize outra através do nome do container utilizando o comando **-link** seguido do nome do container do comando do item 6 que executa uma maquina que queremos que nosso container acesse através do nome*

*É possível mapear uma porta da maquina docker para uma porta da maquina do SO*, para que consigamos ter acesso por exemplo a uma aplicação que esta executando dentro de uma maquina docker em uma porta específica, apenas precisamos usar a tag **-p 9000:9001** na execução do comando do item 6, onde devemos especificar a porta da maquina local e a porta da maquina docker que ficarão linkadas, como no exemplo do comando acima é possível acessarmos a porta 9001 da maquina docker através da porta 9000 da maquina local*.

**OBS.: Cada imagem docker que é criada precisa do seu arquivo Dockerfile e isso em um projeto relativamente grande onde varias imagens com serviços diferentes precisam ser usadas pode ser muito trabalhoso por ter que estar mantendo essas configurações todas isoladas, mas para isso existe o docker-compose, com ele podemos configurar apenas uma arquivo *docker-compose.yml* para especificar as configurações das imagens docker que serão utilizadas no projeto, com esse arquivo inda podemos especificar todos os comandos que serão executados em terminal para cada maquina docker e sua ordem de execução com seus dados específicos, sendo assim é possível executar todo um projeto onde precisaremos executar o arquivos e todas os containers serão criados com suas respectivas configurações e ja serão executados diretamente**

## Docker Compose

*O docker-composer funciona dentro do docker como um gerenciador de containers, onde é possível criar novos containers, configura-los e executa-los em um único lugar com uma sintaxe própria, como falado anteriormente isso é util para o caso onde temos um projeto com varias maquinas com serviços separados, pois isso ajuda a poder subir o projeto de uma forma mais simples*

### Configurando um docker-compose.yml

1. Criar um arquivo docker-compose.yml geralmente esse arquivo fica na raiz do projeto.
    * Esse arquivo irá conter toda configuração com relação aos containers dockers pertencentes ao projeto.

2. A primeira linha do arvo contem a versão do docker-compose que será utilizada.

    > version "3.7

3. Logo depois especificamos os serviços == CONTAINERS, para isso utilizados a palavra reservada **services:** e logo abaixo de forma indentada criaremos os containers como um nome de sua escolha **nome_container:** e logo abaixo seguindo o padrão de indentação segue as configurações relacionadas ao container como no exemplo abaixo:

        version "3.7
        services:
            nome_container:
                < Config_container >

**OBS.: Na parte de configuração de um container dentro de um docker-compose.yml podemos referenciar um Dockerfile especifico de uma imagem docker que criamos ou podemos criar a configuração de imagem diretamente dentro do .yml**

4. Clausulas de configuração de de um container:

    * *image:* : define a imagem que será usada do DockerHub para subir o container.

    * *container-name:* : define o nome do container a ser criado para a imagem

    * *environment:* : Abaixo dessa para reservada seguindo o padrão de indentação deve ficar as variáveis de ambiente que podem ser configuradas para a imagem.

    * *volumes:* : Com essa palavra reservada podemos especificar os volumes onde ficarão salvos as configurações feitas na maquina logo abaixo dessa palavra reservada de forma indentada cada nova linha iniciada co o caractere ( **-** ) devem ser referenciados os locais de dados que ficarão lincados da maquina local e do container por exemplo **- ./api/db/data:/var/lib/mysql**, utilizada para manter um estado da maquina sempre que a mesma for reiniciada.

    * *restart:* :  É uma palavra reservada que indica quando container deve ser restartado, nesse caso quando seguido da palavra 'always' ele sempre será reiniciado caso ocorra algum problema com o container.

    * *build:* É uma palavra reservada usada no inicio da configuração de um container para especificar uma configuração Dockerfile que esta em um local da maquina local específica, logo apos a palavra reservada deve ser indicada o local entre aspas de onde se encontra a configuração Dockerfile.
        
        * caso o arquivo Dockerfile esteja nomeado de forma diferente podemos usar a palavra reservada **context: nome_dockerfile**
    * *ports:* : a palavra reservada seguido de uma indentação sendo que cada linha inicia com o caractere ( **-** ) + porta_maquina_local:porta_container, isso estabelece uma ligação entre as portas do container e as portas da maquina local onde podemos acessar os serviços que estão rodando no container pela porta da maquina local.

    * *depends_on:* : a palavra reserva seguida da indentação sendo que cada linha inicia com o caractere ( **-** ) seguido do nome do container do qual o container que esta sendo criado depende. Essa configuração diz ao docker-compose que o container y depende do container x, sendo assim o container y so será executado quando o container y ja estiver em execução, é possível especificar que um container depende de vários containers.