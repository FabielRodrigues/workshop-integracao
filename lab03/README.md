## Lab 03 - Deploy no Openshift

1. Acesse o Openshift em https://console.ocp.rhbrlab.com:8443

![00-openshift.png](./img/00-ocp.png)

Agora que temos o Openshift em execução não precisamos continuar testando nossa aplicação com o banco de dados em memória H2, agora podemos executar com uma base de dados real para isso utilizaremos o MYSQL. Substitua a configuração do banco H2 pelo seguinte trecho de código no arquivo **application.properties** que se encontra no diretório *src/main/resources*.

    #mysql specific
    mysql.service.name=mysql
    mysql.service.database=sampledb
    mysql.service.username=dbuser
    mysql.service.password=password

    #Database configuration
    spring.datasource.url = jdbc:mysql://${${mysql.service.name}.service.host}:${${mysql.service.name}.service.port}/${mysql.service.database}
    spring.datasource.username = ${mysql.service.username}
    spring.datasource.password = ${mysql.service.password}

Como estamos utilizando o banco de dados MYSQL, iremos adicionar o driver como dependência no arquivo **pom.xml**

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

Abra o **Openshift Explorer View**, no menu do topo selecione **Window -> Show view -> others**. Uma janela irá aparecer, digite openshift no campo de busca e selecione **Openshift Explorer**

![00-view.png](./img/00-view.png)

![00-openshiftexplorer02.png](./img/00-openshiftexplorer.png)

No **Openshift Explorer**, clique com o direito em **connection** e crie um novo projeto **NEW** -> **Project**

![01-newproject.png](./img/01-newproject.png)

**Note:** 'Caso não tenha criado a conexão anteriormente

1. Clique em **New Connection Wizard...** Para configurar o Openshift. Em **Server** Insira a URL do console web Openshift (https://console.ocp.rhbrlab.com:8443) e clique em  **retrieve** para obter o token de acesso.
1. Nova nova janela faça login como desenvolvedor usando as credenciais entregue no inicio do workshop.

    ![05-token1.png](./img/05-token1.png)

1. Clique em **Close**
1. **DESMARQUE** o botão *Save token* e clique em **Finish**

Então utilizando o projeto que foi criado para você (workshop-user-#)iremos criar um banco de dados MYSQL para nossa aplicação. No JBoss Developer Studio clique direito no projeto **myfuseproject** e então **New** -> **Application**

![03-newapp.png](./img/03-newapp.png)

Na aba **Server application source**, selecione  **mysql-ephemeral(database, mysql) - openshift** eclique em next.

![04-mysql.png](./img/04-mysql.png)

Se certifique de utilizar os seguintes parâmetros

```
MYSQL_PASSWORD = password
MYSQL_USER = dbuser
```
![05-param.png](./img/05-param.png)

Clique em Finish, e agora você deve ver uma instância do MYSQL em execução no **Openshift Explorer**.

![06-mysqlcreated.png](./img/06-mysqlcreated.png)

Agora nós podemos finalmente fazer o deploy da nossa aplicação no Openshift, clicando com o botão direito no projeto e selecionando **Run As** -> **Run Configurations...**

![07-runmvn.png](./img/07-runmvn.png)

No menu popup, selecione **Deploy myfuselab on Openshift** no painel da esquerda. Agora vá na aba **JRE** e em **VM arguments**, atualize a variávels kubernetes.master com o o endereço do seu openshift **https://console.ocp.rhbrlab.com:8443** e kubernetes.namespace com **workshop-user-X**  e finalmente usuário/senha com workshop-user-X/<seu-password> e clique em **RUN**.

![08-runconfig.png](./img/08-runconfig.png)

Na aba **Main** troque o comando para 

    fabric8:deploy

Para ver tudo em execução, abra o seu navegador em *https://console.ocp.rhbrlab.com:8443/console/* faça o login com as credenciais developer/developer, selecione o projeto **MY Fuse Project**. E você verá as duas aplicações em execução na aba Overview.

Alternativamente, você pode simplesmente executar o comando maven

    mvn fabric8:deploy 

![09-overview.png](./img/09-overview.png)

Para que essa aplicação possa ser acessada por uma URL externa, é necessário ir em **Application** -> **Service** no menu da esquerda e clique em **camel-ose-springboot-xml**.

![10-service.png](./img/10-service.png)

Clique em **Create route**.

![11-createroute.png](./img/11-createroute.png)

Não altere nada e clique em **Create**.

Acesse a API que criamos através clicando na rota informada no Openshift.

Para verificar a sua rota Camel em ação, no seu console Openshift, vá em **Application** -> **pod** e selecione o pod **camel-ose-springboot-xml-1-xxxxx**

![12-podlist.png](./img/12-podlist.png)

Clique em **Open Java Console**, isso te levará para um console individual que mostra o que a sua rota Camel está fazendo

![13-pod.png](./img/13-pod.png)

Clique em **Route Diagram** e execute as APIs algumas vezes para ver o que acontece

![14-javaconsole.png](./img/14-javaconsole.png)

Para aqueles que querem ver o que está acontecendo no banco de dados, logue na base de dados MYSQL através da linha de comando e execute

```
oc project workshop-user-X

oc get pods
NAME                                   READY     STATUS    RESTARTS   AGE
camel-ose-springboot-xml-s2i-1-build   1/1       Running   0          15s
mysql-1-xxxxx                          1/1       Running   0          2m

oc rsh mysql-1-xxxxx

sh-4.2$ mysql -udbuser -p sampledb
Enter password:

mysql> select * from customerdemo;
+------------+-----------+---------+
| customerID | vipStatus | balance |
+------------+-----------+---------+
| A01        | Diamond   |    1000 |
| A02        | Gold      |     500 |
+------------+-----------+---------+
2 rows in set (0.00 sec)
```
