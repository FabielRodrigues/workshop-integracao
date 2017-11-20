## Lab 04 - Gerenciamento de APIs

### Conectando a API Customers ao Red Hat 3Scale

Para isso precisamos seguir 3 passos:

1. Acesse o 3scale Admin Portal e crie o seu primeiro serviço
1. Customize o Developer Portal
1. Integre a sua API com o 3Scale usando o API Gateway

### Step 0: Pré reqs

Antes de provisionar o API gateway on-premise, você deverá verificar o seguinte na sua conta 3Scale.

1. 3scale Domain
    * Você precisa saber qual é o domínio da sua conta 3Scale SaaS.

    * O 3Scale domain é referenciado na URL do seu portal administrativo do 3Scale SaaS, ex:  ie: https://&lt;YOURDOMAIN&gt;-admin.3scale.net/p/admin/dashboard.

1. 3scale Access Token
    * Para obter um token de acesso navegue em:

        `Gear Icon in top right corner -> Personal Settings -> Tokens -> Add Access Token`

        ![00-accesstoken-a.png](./img/00-accesstoken-a.png)

        ![00-accesstoken-b.png](./img/00-accesstoken-b.png)

        ![00-accesstoken-c.png](./img/00-accesstoken-c.png)

        ![00-accesstoken-d.png](./img/00-accesstoken-d.png)

    + O escopo do seu access token deverá ser: *Account Management API*.

    * Também se certifique de que o seu token tenha a permissão *Read Only*

        ![00-accesstoken-e.png](./img/00-accesstoken-e.png)

    > **Note:** Não se esqueça de copiar o token em um lugar seguro, esse será o único lugar que você poderá recuperá-lo. 
    
### Step 1: Deploy do APIcast via template no OpenShift


1. Caso você já esteja logado no Openshift CLI com o usuário developer, vá para o próximo passo

    Caso contrário se autentique no Openshift utilizando o comando `oc login` utilizando as credenciais *username = "developer"* and *password = "developer"*:

    ```
    oc login https://127.0.0.1:8443 -u developer
    ```

2. Crie um novo secret no Openshift contendo as informações de configuração do seu portal de administração.

    ```
    oc secret new-basicauth apicast-configuration-url-secret --password=https://<ACCESS_TOKEN>@<DOMAIN>-admin.3scale.net
    ```

    O **&lt;ACCESS_TOKEN&gt;** é um Access Token não confunda com o Service Token para o 3Scale Account Management API, e **&lt;DOMAIN&gt;-admin.3scale.net** é a URL do seu 3Scale Admin Portal.

    > **Nota:** Você criou esse access token e o domínio na seção de Pré req.

    A resposta que você obterá deverá ser similar a essa:

    ```
    secret/apicast-configuration-url-secret
    ```

3. Crie o APIcast Gateway a partir de um template

    ```
    oc new-app -f https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/2.0.0.GA-redhat-2/apicast-gateway/apicast.yml
    ```

    You should see the following messages at the bottom of the output:

    ```
    --> Creating resources ...
      deploymentconfig "apicast" created
      service "apicast" created
    --> Success
      Run 'oc status' to view your app.
    ```


4. No projeto *myfuseproject* você deverá ver a tab *Overview* no menu lateral esquerdo

    Cada instância do APIcast, durante o start faz o download dos arquivos de configuração requerido do 3Scale utilizando as configurações que você providenciou na página **Integration** do portal de administração do 3Scale. Para habilitar que o APIcast Gateway possa receber tráfego, você deverá criar uma rota. Para isso clique em **Create Route**

    ![20-openshift-create-route.png](./img/20-openshift-create-route.png)

    Adicione o mesmo host que você atribuiu no 3Scale anteriormente em **Staging Public Base URL** (sem o http:// e sem a porta) por exemplo: 
    `customer-api-staging.127.0.0.1.nip.io`, então clique no botão **Create**

    ![21-openshift-route-config.png](./img/21-openshift-route-config.png)

1. Agora adicione a rota de produção. Dessa vez selecione `Applications -> Routes` do menu lateral esquerdo

    ![22-applications-routes.png](./img/22-applications-routes.png)

1. Clique no botão `Create Route`

    ![23-create-route.png](./img/23-create-route.png)

1. Agora preencha as informações

    **Name:** `apicast-production`

    **Hostname:** `customer-api-production.127.0.0.1.nip.io`

    ![24-production-route.png](./img/24-production-route.png)

1.  Clique no botão `Create` no fim da página para salvar a rota de produção.

    Os API Gateways estão agora prontos para receber requisições. O openshift toma conta de fazer o load balance dos requests entre as duas instâncias do APIcast em execução

    Caso você queira verificar os logs, clique em **Applications > Pods**, selecione o POD que desejar e finalmente clique em **Logs**


### Passo 2: Defina sua API

1. Login no portal de administração http://&lt;YOURDOMAIN&gt;-admin.3scale.net

    ![01-login.png](./img/01-login.png)

2. Caso seja a primeira vez que você acessa o portal 3scale, clique em *activate* da sua caixa de email, agora feche o wizard clicando no **X** na parte superior na direita da sua tela

    ![01a-wizard.png](./img/01a-wizard.png)

3. A primeira página que você verá é a página de API. Daqui podemos criar a nossa API. Clique no link `Integration`

    ![02-api-integration.png](./img/02-api-integration.png)

4. Clique em `edit integration settings` para editar as configurações da API para o gateway

    ![03-edit-settings.png](./img/03-edit-settings.png)

5. Selecione a opção **APIcast self-managed** como opção para deploy do Gateway.

    ![04-apicast.png](./img/04-apicast.png)

6. Mantenha **API Key (user_key)** Authentication

    ![05-authentication.png](./img/05-authentication.png)

7. Clique em **Update Service**

8. Clique no botão **add the Base URL of your API and save the configuration**

9. Expanda a seção **mapping rules** que define os métodos (HTTP) que serão expostos por essa API

    > **Nota:** O mapeamento padrão é root ("/") of our API resources, something that we might want to avoid.

    ![07b-mapping-rules.png](./img/07b-mapping-rules.png)

10. Clique em **Metric or Method (Define)** 

    ![07b-mapping-rules-define.png](./img/07b-mapping-rules-define.png)

11. Clique em **New Method** na seção *Methods*

    ![07b-new-method.png](./img/07b-new-method.png)

12. Preencha as informações

    **Friendly name:** `Get Customers`

    **System name:** `customers_all`

    **Description:** `Method to return all customers`

    ![07b-new-method-data.png](./img/07b-new-method-data.png)

13. Clique em **Create Method**

14. **Optional:** Adicione o método `Get Customer` caso você tenha feito as intruções para implementar a busca do customer por `{id}`. Então nomeie para `customer_get`

15. Clique em **Add mapping rule** 

    ![07b-add-mapping-rule.png](./img/07b-add-mapping-rule.png)

16. Clique para editar este mapping rule

    ![07b-edit-mapping-rule.png](./img/07b-edit-mapping-rule.png)

17. Insira `/myfuselab/customer/all` para o Pattern

18. Selecione `customers_all` como Method

    ![07b-getall-rule.png](./img/07b-getall-rule.png)

19. *Optional::* Clique no botão **Add Mapping Rule** para adicionar o `customer_get` como method mapping.

20. Preencha as informações para acessar sua API:

    **Private Base URL:** `http://camel-ose-springboot-xml:80`

    **Staging Public Base URL:** `http://customer-api-staging.127.0.0.1.nip.io:80`

    **Production Public Base URL:** `http://customer-api-production.127.0.0.1.nip.io:80`

    ![07-baseurl-configuration.png](./img/07-baseurl-configuration.png)

    > **Note:** Nós estamos utilizando um serviço de API interno porque estamos utilizando tudo dentro do mesmo cluster Openshift

21. Mais abaixo clique em **API Test GET request**

22. Informe `/myfuselab/customer/all`

23. Clique em **Update the Staging Environment** para salvar as alterações e clique em **Back to Integration & Configuration** para voltar para página das integrações

    ![08-update-staging.png](./img/08-update-staging.png)

24. Clique no botão **Promote v.1 to Production** para promover suas configurações de homologação para produção

    ![08a-promote-production.png](./img/08a-promote-production.png)

25. Pronto! Agora a sua camada de gerenciamento de API só irá permitir que chamadas autenticadas cheguem até o seu backend de APIs (Fuse).

### Passo 3: Registre uma nova conta no Developer Portal 

The focal point of your developers’ experience is the API developer portal, and the level of effort you put into it will determine the level of decreased support costs and increased developer engagement. 3scale provides a built-in, state-of-the-art CMS portal, making it very easy to create your own branded hub with a custom domain to manage developer interactions and increase API adoption.

You can customize the look and feel of the entire Developer Portal to match your own branding. You have complete control over every element of the portal, so you can make it as easy as possible for developers to learn how to use your API.

The Developer Portal's CMS consists of a few elements:
* Horizontal menu in the Admin Portal with access to content, redirects, and changes
* The main area containing details of the sections above
* CMS mode, accessible through the preview option

![09-developer-portal.png](./img/09-developer-portal.png)

Liquid is a simple programming language used for displaying and processing most of the data from the 3scale system available for API providers. In the 3scale platform, it is used to expose server-side data to your API developers, greatly extending the usefulness of the CMS while maintaining a high level of security.

1. Click on the `Developer Portal` tab to access the developer portal settings.

    ![10-developer-portal.png](./img/10-developer-portal.png)

1. Click on the `Visit Developer Portal` to take a look of how your developer portal looks like.

    ![11-visit-devportal.png](./img/11-visit-devportal.png)

    You can see there is a default portal with information of your API and how to signup. Unfortunately the API information is incorrect.

    ![12-devportal-overview.png](./img/12-devportal-overview.png)

    > **Note:** We will edit our portal to update it with the correct information and to add the shadowman cool logo.

1. Go back to your admin portal browser tab and search the content sub-sections for the `Layouts `and select the `</> Main layout`

    ![12-main-layout.png](./img/12-main-layout.png)

1. First look for the `navbar` section of the main HTML. Replace the `{{ provider.name }}` for the shadowman image link:

    `<img src="https://www.redhat.com/profiles/rh/themes/redhatdotcom/img/logo.png" alt="{{ provider.name }}">`

    ![13-custom-logo.png](./img/13-custom-logo.png)

1. Click on `Publish` button at the bottom of the editor to save the changes and made them available in the site.

    ![14-publish-devportal.png](./img/14-publish-devportal.png)

1. Go back to browse the top of the content sub-sections and find the `Homepage` section.

    ![15-homepage-devportal.png](./img/15-homepage-devportal.png)

1. Change all the code `Echo` references in the homepage code for `Customer`.

1. Update the API call examples to reflect your real Fuse API calls.

    > **Note:** Use your production base url and add your defined methods. Dont worry if you don't have the "real" output, it won't affect the rest of the lab.

1. Refresh your Developer Portal's browser tab to check the changes. Your Developer Portal should look like this:

    ![16-updated-devportal.png](./img/16-updated-devportal.png)

1. Take the place of one of your developers and signup for the **Basic** plan.

    ![16a-signup-limited.png](./img/16a-signup-limited.png)

1. Fill in your information and your email to register as a developer. Click on the `Sign up` button.

    ![16b-signup-form.png](./img/16b-signup-form.png)

    > **Note:** Use an email address you can actually access.

1. Check your email and click on the `activate` link.

    ![16c-activate-account.png](./img/16c-activate-account.png)

1. As your portal is not currently public, you will need your portal code to finish the registration. You can get the code in your admin portal navigating to: `Settings > Developer Portal > Domains & Access`.

    ![16d-access-portal.png](./img/16d-access-portal.png)

1. Type your portal code to finish the account activation.

    ![16e-ingress-code.png](./img/16e-ingress-code.png)

1. Now that your developer account is active, sign in the portal.

    ![16f-dev-signin.png](./img/16f-dev-signin.png)

1. You will land in the developers homepage, where you will be able to check your developers settings and retrieve your `User Key`.

    ![16g-user-key.png](./img/16g-user-key.png)

    > **Note:** Copy down this key as it is used to authenticate yourself to the managed API.


### Step 4: Test APIcast

1. Test that APIcast authorizes a valid call to your API, by executing a curl command with your valid developer's `user_key` to the `hostname` that you configured in the previous step:

    ```
    curl -i "http://customer-api-production.<OPENSHIFT-SERVER-IP>.nip.io:80/myfuselab/customer/all?user_key=YOUR_USER_KEY" --insecure
    ```
    You should see the following messages:

    ```
    HTTP/1.1 200 OK
    Server: openresty/1.11.2.2
    Date: Tue, 30 May 2017 20:13:33 GMT
    Content-Type: application/json
    Transfer-Encoding: chunked
    X-Application-Context: application:dev
    accept: */*
    breadcrumbId: ID-traveler-laptop-rh-mx-redhat-com-45222-1496169770755-0-16
    forwarded: for=192.168.42.1;host=customer-api-staging.192.168.42.100.nip.io;proto=http
    user-agent: curl/7.29.0
    user_key: c13de99abb137810df23ce011d2a948a
    x-3scale-proxy-secret-token: Shared_secret_sent_from_proxy_to_API_backend_71cfe31d89d8cf53
    x-forwarded-for: 192.168.42.1
    x-forwarded-host: customer-api-staging.192.168.42.100.nip.io
    x-forwarded-port: 80
    x-forwarded-proto: http
    x-real-ip: 172.17.0.1
    Set-Cookie: e286b151c44656235d8bdca6ee183477=e58d9930d57779957bf1695b6c805dcd; path=/; HttpOnly
    Cache-control: private

    [{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000},{"CUSTOMERID":"A02","VIPSTATUS":"Gold","BALANCE":500}]
    ```

    The last line is the same output as when calling the API directly.

2. Test that APIcast does not authorize an invalid call to your API.

    ```
    curl -i "http://customer-api-production.<OPENSHIFT-SERVER-IP>.nip.io:80/myfuselab/customer/all?user_key=INVALID_KEY" --insecure
    ```

    When calling the API endpoint with an invalid key, the following messages appear:

    ```
    HTTP/1.1 403 Forbidden
    Server: openresty/1.11.2.2
    Date: Tue, 30 May 2017 20:17:19 GMT
    Content-Type: text/plain; charset=us-ascii
    Transfer-Encoding: chunked
    Set-Cookie: e286b151c44656235d8bdca6ee183477=e58d9930d57779957bf1695b6c805dcd; path=/; HttpOnly
    ```

    The *HTTP/1.1 403 Forbidden* response code indicates that our user_key was wrong or we don't have permisson to access this API endpoint.

3. You have sucessfully configured 3scale API Management and Gateway to access your API.
