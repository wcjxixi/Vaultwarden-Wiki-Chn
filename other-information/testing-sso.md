# \*测试 SSO

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Testing-SSO/)
{% endhint %}

## 用于测试 SSO 的开发设置 <a href="#development-setup-to-test-sso" id="development-setup-to-test-sso"></a>

Vaultwarden 的 SSO 支持目前[正在开发中](https://github.com/dani-garcia/vaultwarden/pull/3154)。以下内容描述了基于 docker-compose 的用于本地测试这些更改的设置。

{% hint style="danger" %}
**仅用于测试 SSO，这些设置并不安全！**
{% endhint %}

## 设置 <a href="#setup" id="setup"></a>

* 查看 SSO 分支
*   使用以下内容创建 `docker-compose.yml`：

    ```batch
    services:
      vaultwarden:
        build: .
        environment:
          DOMAIN: "http://localhost:8000"
          I_REALLY_WANT_VOLATILE_STORAGE: "true"
          SSO_ENABLED: "true"
          SSO_CLIENT_ID: "client"
          SSO_CLIENT_SECRET: "clientsecret"
          SSO_AUTHORITY: "http://auth.test:8080/mock"
        ports:
          - 127.0.0.1:8000:80

      mock-oauth2:
        image: ghcr.io/navikt/mock-oauth2-server:0.5.10
        hostname: "auth.test"
        ports:
          - 127.0.0.1:8080:8080
    ```
* 将 `auth.test` 添加到您的系统 host 文件中：`echo "127.0.0.1 auth.test" | sudo tee -a /etc/hosts`
* 构建 Vaultwarden：`docker compose build`

## 测试 <a href="#testing" id="testing"></a>

* 启动服务：`docker compose up`
* 转到 [http://localhost:8000/#/sso](http://localhost:8000/#/sso)，输入任意字符串作为标识符，点击「登录」
* 在模拟 Auth2 服务器登录页面上，输入任意字符串作为用户/主题，并在声明字段中添加要测试的电子邮件，像这样：`{"email": "user@example.com"}`
* 如果一切按计划进行，您将会被要求输入主密码
