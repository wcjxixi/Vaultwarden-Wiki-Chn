# Table of contents

* [关于](README.md)
* [首页](home.md)
* [FAQ](faq/README.md)
  * [1.FAQ](faq/faqs.md)
  * [2.审计](faq/audits.md)
  * [3.支持上游的发展](faq/supporting-upstream-development.md)
  * [故障排除](faq/troubleshooting/README.md)
    * [1.日志记录](faq/troubleshooting/logging.md)
    * [2.Bitwarden Android 故障排除](faq/troubleshooting/bitwarden-android-troubleshooting.md)
* [容器镜像的使用](container-image-usage/README.md)
  * [1.容器镜像的选择](container-image-usage/which-container-image-to-use.md)
  * [2.启动容器](container-image-usage/starting-a-container.md)
  * [3.使用 Docker Compose](container-image-usage/using-docker-compose.md)
  * [4.使用 Podman](container-image-usage/using-podman.md)
  * [5.更新 Vaultwarden 镜像](container-image-usage/updating-the-vaultwarden-image.md)
* [代理](reverse-proxy/README.md)
  * [1.代理示例](reverse-proxy/proxy-examples.md)
  * [2.使用备用基本目录](reverse-proxy/using-an-alternate-base-dir.md)
  * [HTTPS](reverse-proxy/https/README.md)
    * [1.启用 HTTPS](reverse-proxy/https/enabling-https.md)
    * [2.使用 Let's Encrypt 证书运行私有 Vaultwarden 实例](reverse-proxy/https/running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md)
* [配置](configuration/README.md)
  * [1.配置概述](configuration/configuration-overview.md)
  * [2.启用管理页面](configuration/enabling-admin-page.md)
  * [3.SMTP 配置](configuration/smtp-configuration.md)
  * [4.禁用新用户注册](configuration/disable-registration-of-new-users.md)
  * [5.禁用邀请](configuration/disable-invitations.md)
  * [6.启用 WebSocket 通知](configuration/enabling-websocket-notifications.md)
  * [7.启用移动客户端推送通知](configuration/enabling-mobile-client-push-notification.md)
  * [8.其他配置](configuration/other-configuration.md)
  * [数据库](configuration/database/README.md)
    * [1.使用 MariaDB (MySQL) 后端](configuration/database/using-the-mariadb-mysql-backend.md)
    * [2.使用 PostgreSQL 后端](configuration/database/using-the-postgresql-backend.md)
    * [3.在未启用 WAL 的情况下运行](configuration/database/running-without-wal-enabled.md)
    * [4.从 MariaDB (MySQL) 迁移到 SQLite](configuration/database/migrating-from-mariadb-mysql-to-sqlite.md)
  * [安全](configuration/security/README.md)
    * [1.强化指南](configuration/security/hardening-guide.md)
    * [2.显示密码提示](configuration/security/password-hint-display.md)
    * [3.启用 U2F 和 FIDO2 WebAuthn 身份验证](configuration/security/enabling-u2f-and-fido2-webauthn-authentication.md)
    * [4.启用 YubiKey OTP 身份验证](configuration/security/enabling-yubikey-otp-authentication.md)
    * [5.Fail2ban 设置](configuration/security/fail2ban-setup.md)
    * [6.Docker Traefik ModSecurity 设置](configuration/security/docker-traefik-modsecurity-setup.md)
  * [性能](configuration/performance/README.md)
    * [1.更改 API 请求大小限制](configuration/performance/changing-the-api-request-size-limit.md)
    * [2.更改 worker 数量](configuration/performance/changing-the-number-of-workers.md)
* [自定义](customization/README.md)
  * [1.翻译电子邮件模板](customization/translating-the-email-templates.md)
  * [2.翻译管理页面](customization/translating-admin-page.md)
  * [3.自定义 Vaultwarden CSS](customization/customize-vaultwarden-css.md)
  * [4.使用自定义网站图标](customization/using-custom-website-icons.md)
  * [5.禁用或覆盖密码库接口托管](customization/disabling-or-overriding-the-vault-interface-hosting.md)
* [备份](backup/README.md)
  * [1.通用（非 docker）](backup/general-not-docker.md)
  * [2.备份您的密码库](backup/backing-up-your-vault.md)
* [部署](deployment/README.md)
  * [1.构建二进制](deployment/building-binary.md)
  * [2.构建您自己的 Docker 镜像](deployment/building-your-own-docker-image.md)
  * [3.Git hooks](deployment/git-hooks.md)
  * [4.与上游 API 实现的区别](deployment/differences-from-the-upstream-api-implementation.md)
* [替代部署](alternative-deployments/README.md)
  * [1.预构建二进制](alternative-deployments/pre-built-binaries.md)
  * [2.设置为 systemd 服务](alternative-deployments/creating-a-systemd-service.md)
  * [3.第三方包](alternative-deployments/third-party-packages.md)
  * [4.部署示例](alternative-deployments/deployment-examples.md)
  * [5.禁用管理令牌](alternative-deployments/disable-the-admin-token.md)
* [其他](other-information/README.md)
  * [1.从 Keepass 或 KeepassX 导入数据](other-information/importing-data-from-keepass-or-keepassx.md)
  * [2.更改持久性数据位置](other-information/changing-persistent-data-location.md)
  * [3.从 LDAP 同步用户](other-information/syncing-users-from-ldap.md)
  * [4.使用 Cloudflare DNS 的 Caddy 2.x](other-information/caddy-2.x-with-cloudflare-dns.md)
  * [5.转储示例](other-information/logrotate-example.md)
  * [\*使用非 root 用户运行 docker 容器](other-information/running-docker-container-with-non-root-user.md)
  * [\*使私有 CA 和自签名证书兼容 Chrome](other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md)
  * [\*测试 SSO](other-information/testing-sso.md)
  * [\*使用 systemd docker 运行](other-information/running-with-systemd-docker.md)
