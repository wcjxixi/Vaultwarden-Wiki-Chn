# 3.Docker Traefik ModSecurity 设置

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Docker---Traefik---ModSecurity-Setup)
{% endhint %}

设置 ModSecurity 将通过 Web 应用程序防火墙 (WAF) 将所有请求代理到 Vaultwarden。这可能有助于通过过滤可疑请求（例如注入尝试）来缓解 Vaultwarden 中的未知漏洞。

## 前提条件 <a href="#pre-reqs" id="pre-reqs"></a>

* 设置了使用 Docker + Traefik 2.0 作为反向代理
* 正确设置了 Fail2Ban（[参阅此教程](fail2ban-setup.md#debian-ubuntu-raspian-pi-os)）&#x20;
* 这仅在 Debian 上进行了测试（但应该可以在 Ubuntu 或 Raspbian 等类似系统上运行）

## 安装 <a href="#installation" id="installation"></a>

bash：

```bash
touch /opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf && touch /opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

`/opt/docker/docker-compose.yml`：

```yaml
services:
  traefik:
    image: traefik:latest
    container_name: traefik
    command:
      - --providers.docker=true
      - --providers.docker.exposedByDefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.myresolver.acme.tlschallenge=true
      - --certificatesresolvers.myresolver.acme.email=you@domain.tld
      - --certificatesresolvers.myresolver.acme.storage=acme.json
      - --certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /opt/docker/le:/letsencrypt

  waf:
    image: owasp/modsecurity-crs:apache
    container_name: waf
    environment:
      PARANOIA: 1
      ANOMALY_INBOUND: 10
      ANOMALY_OUTBOUND: 5
      ALLOWED_METHODS: "GET POST PUT DELETE OPTIONS PATCH HEAD"
      PROXY: 1
      REMOTEIP_INT_PROXY: "172.20.0.1/16"
      BACKEND: "http://vaultwarden:80"
      BACKEND_WS: "ws://vaultwarden:80/notifications/hub"
      ERRORLOG: "/var/log/waf/waf.log"
    volumes:
     - /opt/docker/waf:/var/log/waf
     - /opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf:/etc/modsecurity.d/owasp-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
     - /opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf:/etc/modsecurity.d/owasp-crs/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
    labels:
      - traefik.enable=true
      - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
      - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
      - traefik.http.routers.vw-ui-https.rule=Host(`sub.domain.tld`)
      - traefik.http.routers.vw-ui-https.entrypoints=websecure
      - traefik.http.routers.vw-ui-https.tls=true
      - traefik.http.routers.vw-ui-https.service=vw-ui
      - traefik.http.routers.vw-ui-http.rule=Host(`sub.domain.tld`)
      - traefik.http.routers.vw-ui-http.entrypoints=web
      - traefik.http.routers.vw-ui-http.middlewares=redirect-https
      - traefik.http.routers.vw-ui-http.service=vw-ui
      - traefik.http.services.vw-ui.loadbalancer.server.port=80
      - traefik.http.routers.vw-websocket-https.rule=Host(`sub.domain.tld`) && Path(`/notifications/hub`)
      - traefik.http.routers.vw-websocket-https.entrypoints=websecure
      - traefik.http.routers.vw-websocket-https.tls=true
      - traefik.http.routers.vw-websocket-https.service=vw-websocket
      - traefik.http.routers.vw-websocket-http.rule=Host(`sub.domain.tld`) && Path(`/notifications/hub`)
      - traefik.http.routers.vw-websocket-http.entrypoints=web
      - traefik.http.routers.vw-websocket-http.middlewares=redirect-https
      - traefik.http.routers.vw-websocket-http.service=vw-websocket
      - traefik.http.services.vw-websocket.loadbalancer.server.port=3012

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      WEBSOCKET_ENABLED: "true"
      SENDS_ALLOWED: "true"
      PASSWORD_ITERATIONS: 500000
      SIGNUPS_ALLOWED: "true"
      SIGNUPS_VERIFY: "true"
      SIGNUPS_DOMAINS_WHITELIST: "yourdomain.tld"
      ADMIN_TOKEN: "some random string" #generate with openssl rand
      DOMAIN: "domain host name"
      SMTP_HOST: "smtp server"
      SMTP_FROM: "sender email e.g: you@domain.tld"
      SMTP_FROM_NAME: "sender name"
      SMTP_SECURITY: "starttls"
      SMTP_PORT: 587
      SMTP_USERNAME: "smtp username"
      SMTP_PASSWORD: "smtp password"
      SMTP_TIMEOUT: 15
      LOG_FILE: "/data/vaultwarden.log"
      LOG_LEVEL: "warn"
      EXTENDED_LOGGING: "true"
      TZ: "your time zone"
    volumes:
      - /opt/docker/vaultwarden:/data

networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.20.0.1/16
```

`/etc/fail2ban/filter.d/waf.conf`：

```systemd
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*\[client <ADDR>\] ModSecurity: Access denied with code 403 .*$
ignoreregex =
```

`/etc/fail2ban/jail.d/waf.conf`：

```systemd
[waf]
enabled = true
port = 80,443
filter = waf
action = iptables-allports[name=waf, chain=FORWARD]
logpath = /opt/docker/waf/waf.log
maxretry = 1
bantime = 14400
findtime = 14400
```

## 备注 <a href="#note" id="note"></a>

将 Fail2Ban 与 ModSecurity 集成将减缓/阻止对手的进一步利用/探测。这是设置为在第一次 ModSecurity 干预时禁止。

为了增加 ModSecurity 的攻击性，可以增加 `PARANOIA`（[在这里了解更多](https://coreruleset.org/20211028/working-with-paranoia-levels/)）和/或减少 `ANOMALY_INBOUND`（[了解更多](https://coreruleset.org/docs/concepts/anomaly\_scoring/)）的值。

准备好在 PARANOIA > 2 的情况下对 ModSecurity 进行严格的调整，以便在不禁用大量规则的情况下使 UI 能勉强工作。

以下文件将使您能够对 ModSecurity 进行调整（[教程](https://coreruleset.org/docs/concepts/false\_positives\_tuning/)）：

```
/opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
/opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

☁️ 值得深思 ☁️

如果您的数据非常敏感，以至于您正在考虑设置 `PARANOIA` > 1，那么请考虑不在公共端点上托管 Vaultwarden，并通过防火墙限制对主机本身的访问，并仅通过 VPN 连接授予用户访问权限。请记住，这并不能内部威胁，而内部威胁往往被低估，所以请记住这一点！
