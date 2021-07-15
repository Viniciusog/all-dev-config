# Configurações instalação Java e banco local



## 1 - Requisitos Gerais e básicos no Windows

- JDK 11+ (zulu) -> Instalar e configurar JAVA_HOME e \bin em variável path

- Maven -> Configurar MVN_HOME ou MAVEN_HOME (Algo assim)

- Gradle -> Configurar GRADLE_HOME

  

## 2 - Configurações de properties

### application.properties

```properties
#--------------Produção---------------------------------------------------
spring.profiles.active=prod
spring.jpa.open-in-view=false
#--------------Desenvolvimento-------------------------------------------- 
spring.profiles.active=test
spring.jpa.open-in-view=false
#--------------Testes-----------------------------------------------------
spring.profiles.active=test
spring.jpa.open-in-view=false
#--------------Heroku-----------------------------------------------------
#APP_PROFILE estará configurado lá no Heroku. Se não tiver configurado APP_PROFILE, estão
# o ambiente de test execuratá como padrão
spring.profiles.active=${APP_PROFILE:test}

# heroku -> configuration -> reveal config vars
#temos que adicionar a config var APP_PROFILE, valendo 'prod'

spring.jpa.open-in-view=false
#-------------------------------------------------------------------------
```



### Ambiente Dev (application-dev.properties)

```properties
#Colocar as configurações do banco de dados para ambiente de desenvolvimento

#Truque para criar o SQL em um arquivo no nosso projeto
#spring.jpa.properties.javax.persistence.schema-generation.create-source=metadata
#spring.jpa.properties.javax.persistence.schema-generation.scripts.action=create
#spring.jpa.properties.javax.persistence.schema-generation.scripts.create-target=create.sql
#spring.jpa.properties.hibernate.hbm2ddl.delimiter=;

spring.datasource.url=jdbc:postgresql://localhost:5432/nomedobancodedados
spring.datasource.username=postgres
spring.datasource.password=1234567

spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
spring.jpa.hibernate.ddl-auto=none
```

### Ambiente de Teste com H2 (application-test.properties)

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

### Ambiente de produção com Heroku (application-prod.properties)

```properties
#Esse DATABASE_URL estará configurado lá no HEROKU
spring.datasource.url=${DATABASE_URL}

# heroku -> configuration -> reveal config vars
#lá já terá DATABASE_URL configurado automaticamente
```
