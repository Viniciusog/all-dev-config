# Configurações de deploy com heroku e PostgreSQL database. (Spring)



OBS: É preciso ter Java 8+ instalado,  juntamente com Maven e Gradle, configurados nas variáveis de ambiente.

OBS: __Instalar Heroku CLI__. Digite no terminal: ```heroku -v``` para ver a versão do Heroku. Caso não esteja aparecendo, vá nas variáveis de ambiente e coloque em ```path``` o caminho \bin do local onde o Heroku foi instalado



### 1 - Adicionar dependências no pom.xml ou no build.gradle 

#### 1.1 - Adicionar dependência do PostgreSQL 

```xml
<dependency>
	<groupId>org.postgresql</groupId>
	<artifactId>postgresql</artifactId>
	<scope>runtime</scope>
</dependency>
```

#### 1.2 - Adicionar dependência do H2 Database

```xml
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<version>1.4.197</version>
	<scope>runtime</scope>
</dependency>
```

---



### 2 - Configuração de ambientes

#### 2.1 - application-test.properties com H2 Database

```properties
spring.jpa.open-in-view=false

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

#### 2.2 application-prod.properties

`````properties
#Esse DATABASE_URL estará configurado lá no HEROKU
spring.datasource.url=${DATABASE_URL}

# heroku -> configuration -> reveal config vars
#lá já terá DATABASE_URL configurado automaticamente
`````

#### 2.3 application.properties (Definir qual dos ambientes será executado

```properties
#--------------Heroku-----------------------------------------------------
#APP_PROFILE estará configurado lá no Heroku. Se não tiver configurado APP_PROFILE, etão
#o ambiente de test execuratá como padrão
spring.profiles.active=${APP_PROFILE:test}

#Configurar variáveis no site do heroku: heroku -> configuration -> reveal config vars
#Temos que adicionar a config var APP_PROFILE, valendo 'prod'. Para que o heroku saiba #que estamos executando em produção.

spring.jpa.open-in-view=false
```

#### 2.4 - system.properties (Na raiz do projeto)

```
java.runtime.version=11
```

---



### 3 - Conectar banco de dados local com banco de dados no site do Heroku

#### 3.1 - Criar banco de dados PostgreSQL local

Via terminal psql (psql no menu de navegação do windows)

```sh
Server [localhost]: NÃO PRECISA INSERIR
Database [postgres]: NÃO PRECISA INSERIR
Port [5432]: NÃO PRECISA INSERIR
Username [postgres]: NÃO PRECISA INSERIR
Password for user postgres: COLOCAR A SENHA
psql (12.6)
WARNING: Console code page (850) differs from Windows code page (1252)
         8-bit characters might not work correctly. See psql reference
         page "Notes for Windows users" for details.
Type "help" for help.

postgres=# CREATE DATABASE nomedobanco;
CREATE DATABASE
postgres=#  **Ok, aqui já sabemos que o banco foi criado**
```

Via pgAdmin4 (Digite pgAdmin no menu de navegação do Windows)

```
Servers -> PostgreSQL12 ==> (Botão direito) ==> (Create new Database)
**Insira o nome do banco e aperte em salvar**
O banco será criado em: PostgreSQL12 -_ NomeDoBancoCriado
Aperte no banco de dados criado para conectar.
Pronto!
```

#### 3.2 - HEROKU - Criar aplicação lá no heroku

- Criar app no Heroku
- Provisionar banco PostgreSQL
- Definir variável APP_PROFILE=prod
- Conectar ao banco via pgAdmin
- Criar seed do banco (Apenas caso queira ter inserções iniciais)



__1 - Criar app no Heroku__

```
Após ter criado a conta no heroku vá para: https://dashboard.heroku.com/apps
* Aperte em: 'New' ==> 'Create new App'
	- App name: Coloque o nome da aplicação
	- Choose a region: Pode deixar Estados Unidos mesmo
	- Aperte em 'Create App'
	Pronto! App foi criado!
```

__2 - Provisionar banco PostgreSQL__

```
Após ter criado um App, uma nova tela irá aparecer (Personal => NomeAppCriado). 
* Clique na aba 'Resources'
	- Em 'Add-ons' busque pela palavra 'PostgreSQL'
	- Selecione 'Heroku Postgres'
	- Um modal de preços irá aparecer. Selecione a opção grátis
	Pronto! Banco já foi criado!
```

__3 - Definir variável APP_PROFILE=prod__

```
Após ter criado o banco de dados PostgreSQL, observe as abas
* Clique na aba 'settings'
	- Procure o botão 'Reveal config vars' e clique nele
	- Observe que já tem uma variável 'DATABASE_URL' criada automaticamente
	- Em 'Key', coloque: 'APP_PROFILE'.
	- Em 'VALUE' (na mesma linha de Key), coloque: prod
	- Aperte em adicionar para salvar a nossa variável APP_PROFILE
	Pronto! Variável APP_PROFILE foi criada!
```

__4 - Conectar ao banco via pgAdmin__

```
Ainda na aba 'settings', vá no botão 'reveal config vars' e aperte-o. 
* Procure a variável DATABASE_URL e copie o valor dela.
	- Abra um editor de texto e cole o valor da variável DATABASE_URL
	- Observe que é uma URL muito grande, dividida por sinais.
	
* Divida a sua URL da seguinte forma: 

postgres://
paldsfgbiuegbdf --> É o nome de usuário definido pelo Heroku
:
51bsgsdgGsdgdfgDsssSs23sdfsndfwdsjfdfgnee101 --> É a senha definida pelo heroku
@
ec1-1-197-274-075.compute-1.amazonaws.com --> Host aonde o banco foi colocado pelo Heroku
:
5432 --> Porta para que possamos acessar o nosso banco de dados PostgreSQL do Heroku
/
dh3ldkfg4dhdi6 --> Nome da base de dados definido pelo Heroku

* Agora abra o pgAdmin para configurar acesso ao nosso banco PostgreSQL do Heroku
	- Servers ==> create ==> server (Clique em server)
	- General/Name: "QualquerNomeQueVocêQueira.Ex:HerokuSDS3"
	- Connection/Host: Pega o host da URL do DATABASE_URL
	- Connection/Maintenance database: Pega o nome da base de dados da URL DATABASE_URL
	- Connection/Username: Pega o nome de usuário da URL DATABASE_URL
	- Connection/Password: Pega a senha da URL DATABASE_URL
	- Connection/Save Password?: "Yes"
	- Advanced/DB restriction: Pega o nome da base de dados da URL DATABASE_URL
	Pronto! Aperte em save e o pgAdmin estará conectado com o banco de dados criado lá no Heroku. (Tudo que fizermos no banco de dados no pgAdmin será refletido no banco de dados no Heroku)
```

__5 - Enviar nossa aplicação para o Heroku__














