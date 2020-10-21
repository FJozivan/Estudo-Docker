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

### Curiosidades

*O docker cria uma rede que permite que uma maquina docker possa ser visualizada por outra maquina docker através dos seus IPs, mas é possível também fazer com que uma maquina visualize outra através do nome do container utilizando o comando **-link** seguido do nome do container do comando do item 6 que executa uma maquina que queremos que nosso container acesse através do nome*

*É possível mapear uma porta da maquina docker para uma porta da maquina do SO*, para que consigamos ter acesso por exemplo a uma aplicação que esta executando dentro de uma maquina docker em uma porta específica, apenas precisamos usar a tag **-p 9000:9001** na execução do comando do item 6, onde devemos especificar a porta da maquina local e a porta da maquina docker que ficarão linkadas, como no exemplo do comando acima é possível acessarmos a porta 9001 da maquina docker através da porta 9000 da maquina local*.

**OBS.: Cada imagem docker que é criada precisa do seu arquivo Dockerfile e isso em um projeto relativamente grande onde varias imagens com serviços diferentes precisam ser usadas pode ser muito trabalhoso por ter que estar mantendo essas configurações todas isoladas, mas para isso existe o docker-compose, com ele podemos configurar apenas uma arquivo *docker-compose.yml* para especificar as configurações das imagens docker que serão utilizadas no projeto, com esse arquivo inda podemos especificar todos os comandos que serão executados em terminal para cada maquina docker e sua ordem de execução com seus dados específicos, sendo assim é possível executar todo um projeto onde precisaremos executar o arquivos e todas os containers serão criados com suas respectivas configurações e ja serão executados diretamente**

## Docker Compose ...