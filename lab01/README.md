## Lab 01 - Criar um projeto FIS (SpringBoot)

Na IDE Jboss Developer Studio (JBDS), crie um novo projeto utilizando o click direito do mouse no painel project explorer, selecione: **New** -> **Fuse Integration Project**

![01-FIS-project.png](./img/01-FIS-project.png)

Para o nome do projeto utilize **myfuselab** , e clique em *next*

IMPORTANTE : Selecione a versão **2.18.1.redhat-000012** do Camel !

Em *select target runtime*, clique em *next*

![02-runtime.png](./img/02-runtime.png)

Em *Advance project setup*, escolha **Use a predefined template** e selecione **Fuse on OpenShift** -> **SprintBoot on OpenShift** e clique em *finish*

![03-template.png](./img/03-template.png)

O JBDS irá perguntar se você quer alterar a perspectiva para Fuse, informe que *sim*.

No diretório *src/main/resources* teremos dois arquivos de propriedades **application.properties** and e **application-dev.properties**, nós utilizaremos o com prefixo *-dev* no desenvolvimento deste laboratório.

![04-devproperties.png](./img/04-devproperties.png)

Adicione o trecho abaixo nas configurações de datasource no arquivo: **application-dev.properties**

```
#Database configuration
spring.datasource.url = jdbc:h2:mem:mydb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username = sa
spring.datasource.password = 
spring.datasource.driver-class-name = org.h2.Driver
spring.datasource.platform = h2
```

*NOTA: Utilizaremos o banco de dados em memória para testes. E graças ao recurso de autowiring do SpringBoot, este datasource já estará carregado e pronto para uso como datasource padrão no Camel Context.*

Crie um novo arquivo em *src/main/resources*, clicando com o mouse direito no diretório no painel "project explorer" e selecione **New** -> **Others**

![05-newfile.png](./img/05-newfile.png)

Em "Select a wizard", escolha **File** e clique em next, 
em File, insira **schema.sql** como nome para o arquivo, e se certifique que o mesmo estará no diretório *src/main/resources* e selecione "finish".

![06-schemasql.png](./img/06-schemasql.png)

Adicione o seguinte SQL no arquivo **schema.sql**

```
CREATE TABLE customerdemo (
	customerID varchar(10) NOT NULL,
	vipStatus varchar(10) NOT NULL ,
	balance integer NOT NULL
);

INSERT INTO customerdemo (customerID,vipStatus,balance) VALUES ('A01','Diamond',1000);
INSERT INTO customerdemo (customerID,vipStatus,balance) VALUES ('A02','Gold',500);
```

![07-sql.png](./img/07-sql.png)

Duplo clique no arquivo **camel-context.xml** em **Camel Contexts**, agora você estará vendo uma rota camel. Remova a rota existente do canvas.

![08-deleteroute.png](./img/08-deleteroute.png)

Crie uma nova rota arrastando o componente **ROUTE** da palheta *Routing* no canto direito da tela. Insira o nome **customer** na rota, preenchendo o atributo *ID* na seção de propriedade.

![09-route.png](./img/09-route.png)

Utilize o componente timer arrastando da palheta de componentes a direita para o canvas. Configure o timer para executar apenas uma vez, em *Properties*-> *Advance* tab -> *Consumer* , insira o valor **1** para o atríbuto **Repeat Count**

![10-timer.png](./img/10-timer.png)

To read data from the datasource, select **SQL** component from the *Components* palette on the right to the route. Under *Properties*-> *Advance* tab -> *Path*, set **Query** to **select * from customerdemo** 

![11-sqlcomponent.png](./img/11-sqlcomponent.png)

and in *Common* tab set **Data Source** to **dataSource**

![12-datasource.png](./img/12-datasource.png)

And lastly select **LOG** component from the *Components* palette to the end of route. Under *Properties*-> *Detail* tab, set **Message** to **${body}**

![13-log.png](./img/13-log.png)

Before we kick start the application, add the database driver dependency in the **pom.xml** file

```
...
<properties>
  ...
  <run.profiles>dev</run.profiles>
</properties>
...

<dependencies>
	...
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>
    ...
</dependencies>
```
Right click on the **myfuselab** in the project explorer panel, select **Run As..** -> **Maven build...** 

![14-mavenrun.png](./img/14-mavenrun.png)

in the pop-up windown enter **spring-boot:run** in *Goals* and select **Skip Tests**.

![15-springbootrun.png](./img/15-springbootrun.png)

In your log console, verify that customer data is printed.
```
customer - [{CUSTOMERID=A01, VIPSTATUS=Diamond, BALANCE=1000}, {CUSTOMERID=A02, VIPSTATUS=Gold, BALANCE=500}]
```
