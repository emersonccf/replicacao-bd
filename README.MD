# Database Replication

## Replicação de Banco de Dados com SGDB MariaDB usando Docker

Trabalho apresentado na matérias Arquitetura de Software (INF016) sobre requisito
não funcional (Database Replication) Replicação de banco de dados para garantir:

- Disponibilidade;
- Confiabilidade; e
- Redundância de dados.

Fonte, documentação da imagem Docker: https://hub.docker.com/r/bitnami/mariadb

Utilizando o Docker version 20.10.17, build 100c701 e o Docker Compose version v2.6.1

### Execute os comandos dentro da pastas onde está presente o arquivo `docker-compose.yml`.

Para baixar a imagem e criar os containers Main e Backup, os volumes e a rede, em segundo plano:
`docker-compose up -d`

====

### Criando uma tabela e inserindo dados

Acessa o bash do container main como nome de serviço mariadb-main:
`docker-compose exec -it mariadb-main bash`

Dentro do bash do serviço mariadb-main, digite o comando abaixo para acessar o console do MariaDB:
`mysql -u root -pmain_root_password`

No console do MariaDB, digite para exibir as bases de dados existentes:
`show databases;`

Selecione uma base de dados:
`use inf016_db;`

Exibir tabelas existentes na base de dados selecionada:
`show tables;`

Crie uma tabela:
`CREATE TABLE pessoa ( id int(11) NOT NULL AUTO_INCREMENT, nome varchar(255) DEFAULT NULL, idade int(11) DEFAULT NULL, PRIMARY KEY (id) );`

Inserindo registros na tabela:
`INSERT INTO pessoa (nome, idade) VALUES ('Carlos dos Santos', 20), ('Paula Correia', 15);`

Exibe todos os registros da tabela:
`SELECT * FROM pessoa;`

Sai do console do MariaDB:
`exit`

Sai do bash do container dos service mariadb-main, retornado para o terminal do host hospedeiro:
`exit`

====

### Verificando se os dados foram replicados no backup

Acessa o bash do container backup como nome de serviço mariadb-backup:
`docker-compose exec -it mariadb-backup bash`

Dentro do bash do serviço mariadb-backup, digite o comando abaixo para acessar o console do MariaDB:
`mysql -u repl_user -prepl_password`

No console do MariaDB, digite para exibir as bases de dados existentes:
`show databases;`

Selecione uma base de dados:
`use inf016_db;`

Exibir tabelas existentes na base de dados selecionada:
`show tables;`

- O usuário de replicação não tem privilégios para realizar select na base de backup
  justamente porque o alvo das leituras e escritas deve ser na base main.

Sai do console do MariaDB:
`exit`

Sai do bash do container dos service mariadb-backup, retornado para o terminal do host hospedeiro:
`exit`

====

### Parando a funcionalidade de replicação no servidor de backup

Parar o servidor de backup/réplica (não é parar o serviço do container há diferença: cuidado!):
`docker-compose exec mariadb-backup mysqladmin -u root -pmain_root_password stop-slave`

Fazer cópia de segurança em cima do backup:
`docker-compose exec mariadb-backup mysqldump -u root -pmain_root_password inf016_db > inf016_db.sql`

====

### Para testar se a replicação ocorrerá no servidor de backup quando ele for iniciado, sincronizando com os dados do main faremos algumas inserções no banco main:

Acessa o bash do container main como nome de serviço mariadb-main:
`docker-compose exec -it mariadb-main bash`

Dentro do bash do serviço mariadb-main, digite o comando abaixo para acessar o console do MariaDB:
`mysql -u root -pmain_root_password`

Selecione uma base de dados:
`use inf016_db;`

Inserindo registros na tabela:
`INSERT INTO pessoa (nome, idade) VALUES ('Afonso Santos', 30), ('Bruno Sampaio', 44);`

Exibe todos os registros da tabela:
`SELECT * FROM pessoa;`

Sai do console do MariaDB:
`exit`

Sai do bash do container dos service mariadb-backup, retornado para o terminal do host hospedeiro:
`exit`

====

### Verificando o backup parado (não está fazendo a replicação de dados)

Acessa o bash do container backup como nome de serviço mariadb-backup:
`docker-compose exec -it mariadb-backup bash`

Dentro do bash do serviço mariadb-backup, digite o comando abaixo para acessar o console do MariaDB:
`mysql -u root -pmain_root_password`

Selecione uma base de dados:
`use inf016_db;`

Exibe todos os registros da tabela, e vai perceber que os últimos dois registros inseridos não foram sincronizados na replica de backup:
`SELECT * FROM pessoa;`

Sai do console do MariaDB:
`exit`

Sai do bash do container dos service mariadb-backup, retornado para o terminal do host hospedeiro:
`exit`

====

### Reativando o servidor/réplica mariadb-backup e verificando a sincronização após a reativação da funcionalidade no SGBD:

Reiniciar o servidor de backup:
`docker-compose exec mariadb-backup mysqladmin -u root -pmain_root_password start-slave`

Acessa o bash do container backup como nome de serviço mariadb-backup:
`docker-compose exec -it mariadb-backup bash`

Dentro do bash do serviço mariadb-backup, digite o comando abaixo para acessar o console do MariaDB:
`mysql -u root -pmain_root_password`

Selecione uma base de dados:
`use inf016_db;`

Exibe todos os registros da tabela, perceber que os últimos dois registros inseridos FORAM SINCRONIZADOS na replica de backup:
`SELECT * FROM pessoa;`

Sai do console do MariaDB:
`exit`

Sai do bash do container dos service mariadb-backup, retornado para o terminal do host hospedeiro:
`exit`

====

### Limpando o ambiente:

Para eliminar os containers Main e Backup, a rede entre eles, os volumes e as imagens que deram origem aos mesmos:
`docker-compose down -v --rmi all`
