## Lab 02 - Criando APIs

Para expor um endpoint HTTP (API), primeiramente temos que injetar um *Servlet* dentro do *Camel Contexto*, para isso abra o arquivo **camel-context** exibido debaixo da opção **Camel Contexts**, abra a tab *source* e adicione o trecho o seguinte trecho de código antes da tag `<camelContext..>`.

```
    ...
    <bean class="org.apache.camel.component.servlet.CamelHttpTransportServlet" id="camelHttpTransportServlet"/>
    <bean class="org.springframework.boot.web.servlet.ServletRegistrationBean" id="servlet">
        <property name="name" value="CamelServlet"/>
        <property name="servlet" ref="camelHttpTransportServlet"/>
        <property name="urlMappings" value="/*"/>
    </bean>
    ...
```

No mesmo arquivo agora embaixo da tag `<camelcontext..>` adicione o seguinte trecho de código para configurar o endpoint REST. Dessa forma ele estará o Servlet que injetamos no último passo

```
    ...
       <restConfiguration apiContextPath="api-docs" bindingMode="json"
            component="servlet" contextPath="/">
            <apiProperty key="cors" value="true"/>
            <apiProperty key="api.title" value="My First Camel API Lab"/>
            <apiProperty key="api.version" value="1.0.0"/>
        </restConfiguration>
	<!-- Right above route id="customer" -->    
	...
```

Agora iremos expor um único API endpoint, logo após **restConfiguration** adicione

```
    ...
        <rest path="/customer">
            <get uri="all">
            	<description>Retrieve all customer data</description>
                <to uri="direct:getallcustomer"/>
            </get>
        </rest>
    ...
```

Agora invés de chamar um *select* na base de dados através do componente timer como havíamos feito, iremos fazer com que essa interação seja via uma chamada a uma API. Na rota Camel, troque o componente **Timer** pelo componente **Direct**.

Troque

```
<from id="time1" uri="timer:timerName?repeatCount=1"/>
```

por

```
<from id="direct1" uri="direct:getallcustomer"/>
```

Agora vamos adicionar todas as dependências requeridas no arquivo **pom.xml**

```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-servlet-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-jackson-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-swagger-java-starter</artifactId>
    </dependency>
```

Agora com um click direito no projeto **myfuselab** no painel *project explorer*, selecione **Run As..** -> **Maven build** para inicializar a aplicação novamente. E execute o seguinte comando no terminal de linha comando do sistema operacional

```
curl -i http://localhost:8080/myfuselab/customer/all
```

Verifique se o retorno é a lista de *customers* no formato JSON.

```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000},{"CUSTOMERID":"A02","VIPSTATUS":"Gold","BALANCE":500}]
```

Para verificar a documentação swagger

```
curl -i http://localhost:8080/myfuselab/api-docs
```

Pare a aplicação. Tente adicionar outro REST endpoint que receba como parâmetro o ID do Customer e retorne o Customer com o ID equivalente.

Tente fazer sem olhar :)

Dicas: 

```
uri="{custid}"
```

```
select * from customerdemo where customerID=:#custid
```

Verifique a documentação Swagger e teste as chamadas de API, se certificando que o customer retornado é o A01 no formato JSON.

```
curl -i http://localhost:8080/api-docs
curl -i http://localhost:8080/customer/A01
```

```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000}]
```
