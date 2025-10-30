# 1.代理示例

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples)
{% endhint %}

以下是用户收集的不同[反向代理](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)配置示例列表。我们建议使用反向代理终止 TLS/SSL 连接（最好是 443 端口，即 HTTPS 的标准端口），而不是使用 Vaultwarden 内置的 HTTPS 功能。有关更多信息，请参阅[启用 HTTPS](https/enabling-https.md)。

在本文档中，我们假设您在与反向代理相同的主机上运行 Vaultwarden。如果您将 Vaultwarden 作为一个容器运行，假设您已经向主机发布了 `-p 127.0.0.1:8000:80`，因为容器内的 Vaultwarden 已经被配置为监听所有 IPv4 接口（`ROCKET_ADDRESS=0.0.0.0`）上的 `80` 端口（通过 `ROCKET_PORT`）。

示例还假定您没有加密反向代理和 Vaultwarden 之间的连接，即没有配置 `ROCKET_TLS`，因此反向代理可以直接通过 `http://127.0.0.1:8000` 连接到 Vaultwarden。

如果您使用 [Docker Compose](https://docs.docker.com/compose/) 将容器化的服务（例如，Vaultwarden 与反向代理）链接在一起，您必须调整示例代码以考虑到这一点（例如，将 `127.0.0.1:8000` 改为 `service_name:80`，而 `service_name` 通常是 `vaultwarden`）。请参阅[使用 Docker Compose](../container-image-usage/using-docker-compose.md) 了解这方面的示例。

{% hint style="info" %}
可以使用 Mozilla 的 [SSL Configuration Generator](https://ssl-config.mozilla.org/) 来生成 Web 服务器的安全 TLS 协议和密码配置。已知所有受支持的浏览器和移动应用程序都可以使用这种「现代化的」配置。
{% endhint %}

<details>

<summary>Caddy 2.x</summary>

在大多数情况下 Caddy 2 会自动启用 HTTPS，参考[此文档](https://caddyserver.com/docs/automatic-https#activation)。

在 Caddyfile 语法中，`{$VAR}` 表示环境变量 `VAR` 的值。如果您喜欢，也可以直接指定一个值，而不是用一个环境变量的值来代替。

```nginx
# 取消注释以下语句以及取消注释 import admin_redir 语句，以仅允许从本地网络访问管理界面
# {
#        servers {
#                trusted_proxies static private_ranges
#                client_ip_headers X-Forwarded-For X-Real-IP
#                # client_ip_headers CF-Connecting-IP X-Forwarded-For X-Real-IP
#                # If using Cloudflare proxy, insert CF-Connecting-IP as first priority
#                # since Cloudflare doesn't prevent X-Forwarded-For spoofing.
#        }
# }
# (admin_redir) {
#        @admin {
#                path /admin*
#                not remote_ip private_ranges
#        }
#        redir @admin /
# }

{$DOMAIN} {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 如果你想通过 ACME（Let's Encrypt 或 ZeroSSL）获获取证书，请取消注释
  # tls {$EMAIL}

  # 或者如果您提供自己的证书，请取消注释
  # 如果您在 Cloudflare 后面运行，您也会使用此选项
  # tls {$SSL_CERT_PATH} {$SSL_KEY_PATH}

  # 此设置可能会在某些浏览器上出现兼容性问题（例如，在 Firefox 上下载附件）
  # 如果遇到问题，请尝试禁用此功能
  encode zstd gzip
  
  # 取消注释以提高安全性（警告：只有在您了解其影响的情况下才能使用！）
  # 如果您想使用 FIDO2 WebAuthn，请将 X-Frame-Options 设置为 "SAMEORIGIN"，否则浏览器将阻止这些请求
  # header / {
  #	# 启用 HTTP Strict Transport Security (HSTS)
  #	Strict-Transport-Security "max-age=31536000;"
  #	# 禁用 cross-site filter (XSS)
  #	X-XSS-Protection "0"
  #	# 禁止在框架内呈现网站 (clickjacking protection)
  #	X-Frame-Options "DENY"
  #	# 阻止搜索引擎建立索引（可选）
  #	X-Robots-Tag "noindex, nofollow"
  #	# 禁止嗅探 X-Content-Type-Options
  #	X-Content-Type-Options "nosniff"
  #	# 服务器名称移除
  #	-Server
  #	# 移除 X-Powered-By，虽然这不应该是一个问题，但最好移除
  #	-X-Powered-By
  #	# 移除 Last-Modified，因为 etag 相同并且同样有效
  #	-Last-Modified
  # }
  
  # 取消注释以仅允许从本地网络访问管理界面
  # import admin_redir
  
  # 取消注释以只允许从指定的转发 IP（例如 Cloudflare 代理）访问管理界面
  # @not_allowed_admin {
  #     path /admin*
  #     Trusted IPs one and two
  #     not client_ip xx.xx.xx.xx/32 xx.xx.xx.xx/32
  #     # remote_ip's forwarded mode is deprecated; client_ip matcher with global options client_ip_headers and trusted_proxies
  # }

  # respond @not_allowed_admin "401 - {http.request.header.Cf-Connecting-Ip} is not an allowed IP." 401

  # 将所有代理到 Rocket
  # 如果位于子路径中，则 reverse_proxy 行将如下所示：
  # reverse_proxy /vault/* 127.0.0.1:8000
  reverse_proxy 127.0.0.1:8000 {
       # 把真实的远程 IP 发送给 Rocket，以便 Vaultwarden 将其放入日志中，
       # 这样 fail2ban 就可以阻止正确的 IP 了
       header_up X-Real-IP {remote_host}
       # 如果您使用 Cloudflare 代理，请将 remote_host 替换为 http.request.header.Cf-Connecting-Ip
       # 如果使用全局选项 'client_ip_headers CF-Connecting-IP' 则不需要
       # 请参阅 https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/
       # 以及 https://caddy.community/t/forward-auth-copy-headers-value-not-replaced/16998/4
  }
}
```

</details>

<details>

<summary><del>lighttpd (by forkbomb9)</del></summary>

```nginx
erver.modules += ( "mod_proxy" )

$HTTP["host"] == "vault.example.net" {
    $HTTP["url"] == "/notifications/hub" {
       # WebSocket proxy
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 3012 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = (
           "https-remap" => "enable",
           "upgrade" => "enable",
           "connect" => "enable"
       )
    } else {
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 4567 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = ( "https-remap" => "enable" )
    }
}
```

在 Vaultwarden 环境中，您必须将 `IP_HEADER` 设置为 `X-Forwarded-For` 而不是 `X-Real-IP`。

</details>

<details>

<summary>lighttpd with sub-path (by FlakyPi)</summary>

在这个示例中，通过 [https://shared.example.tld/vault/](https://shared.example.tld/vault/) 访问 Vaultwarden。如果您想使用其他子路径，如 `vaultwarden` 或 `secret-vault`，则应更修改下面示例中的 `vault` 以匹配。

您还需要在 `DOMAIN` 环境变量的值中包含子路径（例如 `DOMAIN: "https://shared.example.tld/vault"`），才能使代理正常工作。

```nginx
server.modules += (
"mod_openssl",
"mod_redirect"
)

$SERVER["socket"] == ":443" {  
    ssl.engine   = "enable"   
    ssl.pemfile  = "/etc/letsencrypt/live/shared.example.tld/fullchain.pem"
    ssl.privkey  = "/etc/letsencrypt/live/shared.example.tld/privkey.pem"
}

# Redirect HTTP requests (port 80) to HTTPS (port 443)
$SERVER["socket"] == ":80" {  
        $HTTP["host"] =~ "shared.example.tld" {  
         url.redirect = ( "^/(.*)" => "https://shared.example.tld/$1" )  
         server.name = "shared.example.tld"   
        }  
}

server.modules += ( "mod_proxy" )

$HTTP["host"] == "shared.example.tld" {
    $HTTP["url"] =~ "/vault" {
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "127.0.0.1", "port" => 8000 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = (
           "https-remap" => "enable",
           "upgrade" => "enable",
           "connect" => "enable"
       )
    }
}
```

您必须将 Vaultwarden 环境配置中的 `IP_HEADER` 设置为 `X-Forwarded-For` 而不是默认的 `X-Real-IP`。

</details>

<details>

<summary>Nginx (by <a href="https://github.com/BlackDex">@BlackDex</a>)</summary>

```nginx
# 'upstream' 指令确保你有一个 http/1.1 连接
# 这里启用了 keepalive 选项并拥有更好的性能
#
# 此处定义服务器的 IP 和端口。
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8000;
  keepalive 2;
}

# 要支持 websocket 连接的话才需要
# 参阅：https://nginx.org/en/docs/http/websocket.html
# 我们不发送上述链接中所说的 "close"，而是发送一个空值。
# 否则所有的 keepalive 连接都将无法工作。
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# 将 HTTP 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name vaultwarden.example.tld;

    return 301 https://$host$request_uri;
}

server {
    # 对于旧版本的 nginx，在 ssl 后面的 listen 行中加入 http2，并移除 'http2 on'
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name vaultwarden.example.tld;

    # 根据需要指定 SSL 配置
    #ssl_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #add_header Strict-Transport-Security "max-age=31536000;";

    client_max_body_size 525M;

    location / {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    # 如果您使用 Cloudflare 代理，请将 $remote_addr 替换为 $http_cf_connecting_ip
    # 参阅 https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/#nginx-1
    # 或者使用 ngx_http_realip_module
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    location / {
      proxy_pass http://vaultwarden-default;
    }

    # 除了 ADMIN_TOKEN 之外，还可以选择添加额外的身份验证
    # 删除下面的 '#' 注释并创建 htpasswd_file 以使其处于活动状态
    #
    #location /admin {
    #  # 参阅：https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    #  auth_basic "Private";
    #  auth_basic_user_file /path/to/htpasswd_file;
    #
    #  proxy_http_version 1.1;
    #  proxy_set_header Upgrade $http_upgrade;
    #  proxy_set_header Connection $connection_upgrade;
    #
    #  proxy_set_header Host $host;
    #  proxy_set_header X-Real-IP $remote_addr;
    #  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #  proxy_set_header X-Forwarded-Proto $scheme;
    #
    #  proxy_pass http://vaultwarden-default;
    #}
}
```

如果遇到 504 Gateway Timeout（网关超时）故障，可以通过在 `server {` 部分添加更长的超时时间来告诉 nginx 等待 Vaultwarden 的时间，例如：

```nginx
  proxy_connect_timeout       777;
  proxy_send_timeout          777;
  proxy_read_timeout          777;
  send_timeout                777;
```

</details>

<details>

<summary>Nginx with sub-path (by <a href="https://github.com/BlackDex">@BlackDex</a>)</summary>

在这个示例中，Vaultwarden 的访问地址为 `https://shared.example.tld/vault/`，如果您想使用任何其他的子路径，比如 `vaultwarden` 或 `secret-vault`，您需要更改下面示例中相应的地方。

为此，您需要配置 `DOMAIN` 变量以使其匹配，它应类似于：

```systemd
; 添加子路径！否则将无法正常工作！
DOMAIN=https://shared.example.tld/vault/
```

```nginx
# 'upstream' 指令确保你有一个 http/1.1 连接
# 这里启用了 keepalive 选项并拥有更好的性能
#
# 此处定义服务器的 IP 和端口。
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8000;
  keepalive 2;
}

# 要支持 websocket 连接的话才需要
# 参阅：https://nginx.org/en/docs/http/websocket.html
# 我们不发送上述链接中所说的 "close"，而是发送一个空值。
# 否则所有的 keepalive 连接都将无法工作。
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# 将 HTTP 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name shared.example.tld;

    location /vault/ { # <-- 替换为所需的子路径
        return 301 https://$host$request_uri;
    }
    
    # 如果您想对整个域名而不是只对 Vaultwarden 强制 HTTPS，
    # 那么您可以使用下面的内容代替上面的 location 块：
    return 301 https://$host$request_uri;
}

server {
    # 对于旧版本的 nginx，在 ssl 后面的 listen 行中加入 http2，并移除 'http2 on;'
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name shared.example.tld;

    # 根据需要指定 SSL 配置
    #ssl_certificate /path/to/certificate/letsencrypt/live/shared.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/shared.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/shared.example.tld/fullchain.pem;
    #add_header Strict-Transport-Security "max-age=31536000;";

    client_max_body_size 525M;

    ## 使用子路径配置
    # 到你的安装的 root 目录的路径
    # 一定要加上尾部的 /，否则您会遇到问题
    # 但仅限于这个位置，所有其他位置都不应该添加这个。
    location /vault/ {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_pass http://vaultwarden-default;
    }

    # 除了 ADMIN_TOKEN 之外，还可以选择添加额外的身份验证
    # 删除下面的 '#' 注释并创建 htpasswd_file 以使其处于活动状态
    #
    # 不要添加尾部的/，否则您会遇到问题
    #location /vault/admin {
    #  # 参阅：https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    #  auth_basic "Private";
    #  auth_basic_user_file /path/to/htpasswd_file;
    #
    #  proxy_http_version 1.1;
    #  proxy_set_header Upgrade $http_upgrade;
    #  proxy_set_header Connection $connection_upgrade;
    #
    #  proxy_set_header Host $host;
    #  proxy_set_header X-Real-IP $remote_addr;
    #  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #  proxy_set_header X-Forwarded-Proto $scheme;
    #
    #  proxy_pass http://vaultwarden-default;
    #}
}
```

</details>

<details>

<summary><del>Nginx configured by Ansible/DebOps (by ypid)</del></summary>

使用 [DebOps](https://debops.org) 配置 nginx 作为 Vaultwarden 的反向代理的清单示例。我选择在 URL 中使用 PSK 以获得额外的安全性，从而不会将 API 暴露给 Internet 上的每个人，因为客户端应用程序尚不支持客户端证书（我测试过）。 参考[强化指南 - 隐藏在子目录下](../configuration/security/hardening-guide.md#hiding-under-a-subdir)

```nginx
vaultwarden__fqdn: 'vault.example.org'
vaultwarden__http_psk_subpath_enabled: True
vaultwarden__http_psk_subpath: '{{ lookup("password", secret + "/vaultwarden/" +
                                     inventory_hostname + "/config/subpath chars=ascii_letters,digits length=23")
                                   if vaultwarden__http_psk_subpath_enabled | bool
                                   else "" }}'

nginx__upstreams:

  - name: 'vaultwarden-default'
    type: 'default'
    enabled: True
    server: 'localhost:8000'

  - name: 'vaultwarden-ws'
    type: 'default'
    enabled: True
    server: 'localhost:3012'

nginx__servers:

  - name: '{{ vaultwarden__fqdn }}'
    filename: 'debops.vaultwarden'
    by_role: 'debops.vaultwarden'
    favicon: False
    # root: '/usr/share/vaultwarden/web-vault'

    location_list:

      - pattern: '/'
        options: |-
          deny all;

      - pattern: '= /{{ vaultwarden__http_psk_subpath }}'
        options: |-
          return 307 $scheme://$host$request_uri/;

      ## All the security HTTP headers would then need to be set by nginx as well.
      # - pattern: '/{{ vaultwarden__http_psk_subpath }}/'
      #   options: |-
      #     alias /usr/share/vaultwarden/web-vault/;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/'
        options: |-
          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-default;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/notifications/hub/negotiate'
        options: |-
          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-default;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/notifications/hub'
        options: |-
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade;

          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-ws;

      # Do not use the icons features as long as it reveals all domains from
      # our credentials to the server.
      - pattern: '/{{ vaultwarden__http_psk_subpath }}/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```

</details>

<details>

<summary>Nginx (NixOS) (by tklitschi, samdoshi)</summary>

NixOS Nginx 配置示例。关于 NixOS 部署的更多信息，请参阅[部署示例](../alternative-deployments/deployment-examples.md)页面。

```nginx
{ config, ... }:
{
  security.acme = {
    defaults = {
      acceptTerms = true;
      email = "me@example.com";
    };
    certs."vaultwarden.example.tld".group = "vaultwarden";
  };

  services.nginx = {
    enable = true;

    recommendedGzipSettings = true;
    recommendedOptimisation = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;

    virtualHosts = {
      "vaultwarden.example.ltd" = {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          proxyPass = "http://127.0.0.1:8000";
          proxyWebsockets = true;
        };
      };
    };
  };
}
```

</details>

<details>

<summary>Nginx with proxy_protocol in front (by dionysius)</summary>

在这个例子中，有一个下游代理在[这个 nginx 前面的 proxy\_protocol](https://docs.nginx.com/nginx/admin-guide/load-balancer/using-proxy-protocol/) 中进行通信（例如，[启用了 proxy\_protocol 的 LXD 代理设备](https://linuxcontainers.org/lxd/docs/master/reference/devices_proxy/)）。Nginx 需要从这里设置正确使用协议和要转发的标头。标有 `# <---` 的行与 blackdex 的示例内容不同。

参考这个 LXD 下游代理设备配置：

```nginx
devices:
  http:
    connect: tcp:[::1]:80
    listen: tcp:[::]:80
    proxy_protocol: "true"
    type: proxy
  https:
    connect: tcp:[::1]:443
    listen: tcp:[::]:443
    proxy_protocol: "true"
    type: proxy
```

<pre class="language-nginx"><code class="lang-nginx"># proxy_protocol 相关:

set_real_ip_from ::1; # 要信任哪个下游代理，请在前面输入您的代理地址
real_ip_header proxy_protocol; # 可选，如果您希望 nginx 使用来自 proxy_protocol 的信息覆盖 remote_addr。 取决于您在日志模板和服务器或流块中使用的关于远程地址的变量。

# 以下基于 blackdex 的示例:

# `upstream` 指令确保您有一个 http/1.1 连接
# 这启用了 keepalive 选项和更好的性能
#
# 这里定义服务器 IP 和端口.
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8000;
  keepalive 2;
}
# 需要这些以支持 websocket 连接
# 参阅：https://nginx.org/en/docs/http/websocket.html
# 我们发送的是空值，而不是上述链接中的 "close"
# 否则所有 keepalive 连接都将失效
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# HTTP 重定向到 HTTPS
server {
    listen 80 proxy_protocol; # &#x3C;---
    listen [::]:80 proxy_protocol; # &#x3C;---
    server_name shared.example.tld;
    
    return 301 https://$host$request_uri;
}

server {
<strong>    listen 443 ssl proxy_protocol; # &#x3C;---
</strong>    listen [::]:443 ssl proxy_protocol; # &#x3C;---
    http2 on;
    server_name shared.example.tld;

    # 需要时指定 SSL Config
    #ssl_certificate /path/to/certificate/letsencrypt/live/shared.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/shared.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/shared.example.tld/fullchain.pem;

    client_max_body_size 525M;

    ## 使用子路径 Config
    # 您的安装的根目录路径
    # 请务必添加尾随 /，否则您可能会遇到问题
    # 但仅限于此位置，所有其他位置不应添加这些内容
    location /vault/ {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }
}
</code></pre>

</details>

<details>

<summary>Apache (by fbartels)</summary>

请记得启用 `mod_proxy_http`，例如使用：`a2enmod proxy_http`。这要求 Apache >= 2.4.47。

```apacheconf
<VirtualHost *:443>
    SSLEngine on
    ServerName vaultwarden.example.tld

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/vaultwarden-error.log
    CustomLog \${APACHE_LOG_DIR}/vaultwarden-access.log combined

    ProxyPass / http://127.0.0.1:8000/ upgrade=websocket

    ProxyPreserveHost On
    ProxyRequests Off
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    # 如果您的 url 属性报告为 http://... ，请添加此行：
    # RequestHeader add X-Forwarded-Proto https
</VirtualHost>
```

</details>

<details>

<summary>Apache in a sub-location (by <a href="https://github.com/agentdr8">@agentdr8</a> &#x26; <a href="https://github.com/NoseyNick">@NoseyNick</a>)</summary>

修改 docker 启动以包含 sub-location。

```systemd
; 添加子位置！否则将不起作用！
DOMAIN=https://shared.example.tld/vault/
```

请记得启用 `mod_proxy_http`，例如使用：`a2enmod proxy_http`。这要求 Apache >= 2.4.47。

```apacheconf
<VirtualHost *:443>
    SSLEngine on
    ServerName $hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Location /vault> # 如果需要，调整此处
        ProxyPass http://127.0.0.1:8000/vault/ upgrade=websocket

        ProxyPreserveHost On
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    </Location>
</VirtualHost>
```

</details>

<details>

<summary><del>Apache 2.4.47 (or later) in a sub-location (by</del> <a href="https://github.com/NoseyNick"><del>@NoseyNick</del></a><del>)</del></summary>

现在，常规的 `mod_proxy` 支持使用 `upgrade=websocket` 升级到 WebSocket，而不需要 `mod_proxy_wstunnel`。

复制上面的说明，除非使用更简单的...

```apacheconf
<VirtualHost *:443>
  [ blah blah ]
  <Location /vault> #adjust here if necessary
    ProxyPass http://127.0.0.1:8000/vault/ upgrade=websocket
    ProxyPreserveHost On
    ProxyRequests Off # ... is the default, but as a safety-net
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
  </Location>
</VirtualHost>
```

</details>

<details>

<summary><del>Traefik v1 (docker-compose 示例)</del></summary>

```yaml
labels:
    - traefik.enable=true
    - traefik.docker.network=traefik
    - traefik.web.frontend.rule=Host:vaultwarden.example.tld
    - traefik.web.port=80
```

</details>

<details>

<summary>Traefik v2 (docker-compose 示例 by hwwilliams, gzfrozen)</summary>

#### 将 Traefik v1 标签迁移到 Traefik v2 <a href="#traefik-v-1-labels-migrated-to-traefik-v2" id="traefik-v-1-labels-migrated-to-traefik-v2"></a>

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.vaultwarden.rule=Host(`vaultwarden.example.tld`)
  - traefik.http.routers.vaultwarden.service=vaultwarden
  - traefik.http.services.vaultwarden.loadbalancer.server.port=80
```

#### 迁移的标签加上 HTTP 到 HTTPS 重定向 <a href="#migrated-labels-plus-http-to-https-redirect" id="migrated-labels-plus-http-to-https-redirect"></a>

这些标签假定 Traefik 中为端口 80 和 443 定义的入口点分别是「web」和「websecure」。

这些标签还假定您已经在 Traefik 中定义了默认的证书解析器。

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
  - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
  - traefik.http.routers.vaultwarden-https.rule=Host(`vaultwarden.domain.tld`)
  - traefik.http.routers.vaultwarden-https.entrypoints=websecure
  - traefik.http.routers.vaultwarden-https.tls=true
  - traefik.http.routers.vaultwarden-https.service=vaultwarden
  - traefik.http.routers.vaultwarden-http.rule=Host(`vaultwarden.domain.tld`)
  - traefik.http.routers.vaultwarden-http.entrypoints=web
  - traefik.http.routers.vaultwarden-http.middlewares=redirect-https
  - traefik.http.routers.vaultwarden-http.service=vaultwarden
  - traefik.http.services.vaultwarden.loadbalancer.server.port=80
```

</details>

<details>

<summary>HAproxy (by <a href="https://github.com/BlackDex">@BlackDex</a>)</summary>

将这些行添加到您的 HAproxy 配置中。

```yaml
frontend vaultwarden
    bind 0.0.0.0:80
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend vaultwarden_http

backend vaultwarden_http
    # 启用压缩（如果您需要）
    # 压缩算法 gzip
    # 压缩类型 text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    # Vaultwarden 不支持 forwarded 头，但您可以启用它
    # option forward
    # 添加 x-forwarded-for 头
    option forwardfor
    # 在 `X-Real-IP` 头中设置来源 IP
    http-request set-header X-Real-IP %[src]
    # 将流量发送到本地实例
    server vwhttp 0.0.0.0:8000alpn http/1.1
```

</details>

<details>

<summary><del>HAproxy - before v1.29.0 (by</del> <a href="https://github.com/williamdes"><del>@williamdes</del></a><del>)</del></summary>

将这些行添加到您的 HAproxy 配置中。

```yaml
backend static-success-default
  mode http
  errorfile 503 /usr/local/etc/haproxy/static/index.static.default.html
  errorfile 200 /usr/local/etc/haproxy/static/index.static.default.html

frontend http-in
    bind *:443 ssl crt /acme.sh/domain.tld/domain.tld.pem alpn h2,http/1.1
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend static-success-default

    # 定义主机
    acl host_vaultwarden_domain_tld hdr(Host) -i vaultwarden.domain.tld

    ## 谋划要使用哪一个
    use_backend vaultwarden_http if host_bitwarden_domain_tld

backend vaultwarden_http
    # 启用压缩（如果您需要）
    # 压缩算法 gzip
    # 压缩类型 text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    # 如果您在 docker-compose 中使用 haproxy，则可以使用容器主机名
    server vw_http 0.0.0.0:8080 alpn http/1.1
```

</details>

<details>

<summary><del>HAproxy inside PfSense (by</del> <a href="https://github.com/RichardMawdsley"><del>@RichardMawdsley</del></a><del>)</del></summary>

作为 GUI 设置，下面的详细信息\说明供您在需要的地方添加。

* 假设您已经设置好了基本的 HTTP > HTTPS 重定向设置。[基本设置](https://blog.devita.co/pfsense-to-proxy-traffic-for-websites-using-pfsense/)

### 后端创建

后端 1：

```
Mode	  Name	                     Forwardto	    Address	      Port	 Encrypt(SSL)	 SSL checks	  Weight	 Actions
active 	Vaultwarden                Address+Port:  IPADDRESSHERE 80     no            no
```

后端 2：

```
Mode	  Name	                     Forwardto	    Address	      Port	 Encrypt(SSL)	 SSL checks 	Weight	Actions
active 	Vaultwarden-Notifications  Address+Port:  IPADDRESSHERE 3012   no            no
```

### 前端创建-1-域名 <a href="#frontend-creation-1-domain" id="frontend-creation-1-domain"></a>

**ACCESS CONTROL LIST**

```yaml
ACL00
Host matches:
no
no
FQDN.com     -  注意：这需要是您的根域名。
 	
ACL00
Path starts with:
no
yes
/big-ass-randomized-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up

ACL01
Host matches:
no
no
VAULTWARDEN.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
```

**ACTIONS-1-Domain**

```yaml
http-request allow
See below
ACL01

http-request deny
See below
ACL00
```

### 前端创建-2-VaultWarden <a href="#frontend-creation-2-vaultwarden" id="frontend-creation-2-vaultwarden"></a>

**ACCESS CONTROL LIST**

```yaml
ACL1
Path starts with:
no
yes
/notifications/hub  
 	
ACL2
Path starts with:
no
no
/notifications/hub/negotiate  
 	
ACL3
Path starts with:
no
no
/notifications/hub  
 	
ACL4
Path starts with:
no
yes
/notifications/hub/negotiate

ACL5
Path starts with:
no
no
/admin
```

**ACTIONS - 2 - VaultWarden**

```yaml
Use Backend
See below
ACL1  
backend: VaultWarden
 	
Use Backend
See below
ACL2  
backend: VaultWarden
 	
Use Backend
See below
ACL3  
backend: VaultWarden-Notifications
 	
Use Backend
See below
ACL4
backend: VaultWarden-Notifications

http-request deny
See below
ACL5
```

#### **更新记录** <a href="#updates" id="updates"></a>

```
Updated above 30/07 - 我在第一次配置后意识到，因为 ACL1-4 有 'Not'，他们正在将任何内容与他们的动作相匹配。所以 BlahBlahMcGee.FQDN.com 通过了。这不是故意的，所以上面添加了 ACL5 来解决这个问题，它还移除了对默认后端的需要。
Updated again 30/07 - ^ 是的，没用。这一切都源于 HaProxy 不允许在 ACL 中使用 'AND'。唉。现在有了上面的内容，您可以为根域配置一个前端。这有一个否认本身，以及任何未指定的内容。因此，如果您要通过多个其他子域，则需要将它们全部添加到 ACL01 下。现在一切正常了！
```

#### 重要提示 <a href="#important-notes" id="important-notes"></a>

```
1) 您必须使域名前端与允许列表中的任何其他子域名保持同步
2) 在域名前端，ACL01 必须位于 Actions 表的顶部 - 或至少在 ACL00 的上面
3) ACL 名称的重复使用是故意的。是的，我没有打错它们。ACL00、ACL01 等等
```

#### 可选 <a href="#optional" id="optional"></a>

```
上面的 ACL5 拒绝访问 /admin 门户。我不是特别喜欢没有任何形式的 2FA 且只有密码的管理门户。因此，当我不使用它时，我只是拒绝访问。如果我需要它，请取消阻止，完成所需的工作并重新阻止。
```

完成！可以去做测试了！

反过来，可以将下面的等效项添加到您的配置中（请注意，这是一个示例摘要）。

```yaml
acl			ACL00	var(txn.txnhost) -m str -i VAULTWARDEN.MYDOMAIN.COM
acl			ACL00	var(txn.txnpath) -m beg -i /big-ass-randomised-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up
acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM
acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
acl			ACL1	var(txn.txnpath) -m beg -i /notifications/hub
acl			ACL2	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
acl			ACL3	var(txn.txnpath) -m beg -i /notifications/hub
acl			ACL4	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
acl			ACL5	var(txn.txnpath) -m beg -i /admin

http-request allow  if  ACL01 
http-request deny   if  !ACL00 
http-request deny   if  !ACL5 
http-request deny   if  ACL5 
use_backend VaultWarden_ipvANY  if  !ACL1 
use_backend VaultWarden_ipvANY  if  ACL2 
use_backend VaultWarden-Notifications_ipvANY  if  ACL3 
use_backend VaultWarden-Notifications_ipvANY  if  !ACL4 
```

为了进行测试，如果您在浏览器中导航到 /notifications/hub，那么您应该会看到一个页面，上面写着「WebSocket Protocol Error: Unable to parse WebSocket key.」（WebSocket 协议错误：无法解析 WebSocket 密钥。） ……这意味着它可以正常工作！ - 所有其他子页面都应该出现 Rocket 错误。

</details>

<details>

<summary>HAproxy Kubernetes Ingress(by <a href="https://github.com/devinslick">@devinslick</a>)</summary>

控制器安装详情可在此处找到：[https://www.haproxy.com/documentation/kubernetes-ingress/community/installation/on-prem/](https://www.haproxy.com/documentation/kubernetes-ingress/community/installation/on-prem/)。请注意，仅当您使用 Cloudflare 时才需要 CF-Connecting-IP 标头

添加以下资源定义：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vaultwarden
  namespace: default
  annotations:
    haproxy.org/forwarded-for: "true"
    haproxy.org/compression-algo: "gzip"
    haproxy.org/compression-type: "text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript"
    haproxy.org/http2-enabled: "true"
spec:
  ingressClassName: haproxy
  tls:
  - hosts:
    - vaultwarden.example.tld
  rules:
  - host: vaultwarden.example.tld
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vaultwarden-http
            port:
              number: 80
```

</details>

<details>

<summary>Istio k8s (by <a href="https://github.com/asenyaev">@asenyaev</a>)</summary>

```javascript
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: vaultwarden-gateway
  namespace: vaultwarden
spec:
  selector:
    istio: ingressgateway-internal # use Istio default gateway implementation
  servers:
  - hosts:
    - vw.k8s.prod
    port:
      number: 80
      name: http
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - vw.k8s.prod
    port:
      name: https-443
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: vw-k8s-prod-tls
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: vaultwarden-vs
  namespace: vaultwarden
spec:
  hosts:
  - vw.k8s.prod
  gateways:
  - vaultwarden-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: vaultwarden
```

</details>

<details>

<summary><del>Istio k8s - before v1.29.0 (by</del> <a href="https://github.com/dpoke"><del>@dpoke</del></a><del>)</del></summary>

```javascript
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: vaultwarden-gateway
  namespace: vaultwarden
spec:
  selector:
    istio: ingressgateway-internal # use Istio default gateway implementation
  servers:
  - hosts:
    - vw.k8s.prod
    port:
      number: 80
      name: http
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - vw.k8s.prod
    port:
      name: https-443
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: vw-k8s-prod-tls
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: vaultwarden-vs
  namespace: vaultwarden
spec:
  hosts:
  - vw.k8s.prod
  gateways:
  - vaultwarden-gateway
  http:
  - match:
    - uri:
        exact: /notifications/hub
    route:
    - destination:
        port:
          number: 3012
        host: vaultwarden-ws
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: vaultwarden
```

</details>

<details>

<summary><del>relayd on openbsd (by olliestrickland)</del></summary>

经测试可正常运行（包括 websockets） - /etc/relayd.conf - 在 openbsd 7.2 上使用来自 OpenBSD Ports 的 Vaultwarden - [https://openports.se/security/vaultwarden](https://openports.se/security/vaultwarden)

此配置取决于 tls 的正确设置 - 我使用 [https://man.openbsd.org/acme-client](https://man.openbsd.org/acme-client)

```nginx
table <vaultwarden-default-host> { localhost }
table <vaultwarden-websocket-host> { localhost }

# 带有 tls 的 Vaultwarden 协议定义

http protocol vaultwarden-https {
        # 添加 Vaultwarden 所需要的标头
        match request header append "X-Real-IP" value "$REMOTE_ADDR"

        # 添加一些 Vaultwarden 可能不需要的标头
        match request header append "Host" value "$HOST"
        match request header append "X-Forwarded-For" value "$REMOTE_ADDR"
        match request header append "X-Forwarded-By" value "$SERVER_ADDR:$SERVER_PORT"

        # 最普通的规则 - 转发到 Vaultwarden Rocket
        match request path "/*" forward to <vaultwarden-default-host>

        # 将用于 websocket 的路径转发到 Vaultwarden websocket 端口
        match request path "/notifications/hub" forward to <vaultwarden-websocket-host>

        # 将最具体的路径保存在最后 - 此路径不应转发到 websocket 服务器
        match request path "/notifications/hub/negotiate" forward to <vaultwarden-default-host>

        # 各种 TCP 选项
        tcp { nodelay, sack, backlog 128 }

        # tls 配置
        tls keypair bitwarden.example.tld
        tls { no tlsv1.0, ciphers HIGH }

        # 允许 websockets - 这很好，它可以处理连接升级，而无需手动编辑标头
        http websockets
}

# Vaultwarden 的中继定义 - 将出口接口上的入站 443 tls 转发到默认 8000 端口上的 rocket 和 3012 上的 websocket

relay vaultwarden-https-relay {
        listen on egress port 443 tls
        protocol vaultwarden-https
        forward to <vaultwarden-default-host> port 8000
        forward to <vaultwarden-websocket-host> port 3012
}
```

</details>

<details>

<summary><del>CloudFlare - before v1.29.0 (by</del> <a href="https://github.com/williamdes"><del>@williamdes</del></a><del>)</del></summary>

按照下面的截图创建新的规则。用于查找此设置的示例仪表板 URL：`https://dash.cloudflare.com/xxxxxx/example.org/rules/origin-rules/new`

<img src="https://user-images.githubusercontent.com/7784660/251004005-e27d9152-219b-4b6a-bf96-dcfce30ebd73.png" alt="" data-size="original">

</details>

<details>

<summary>CloudFlare Tunnel (by <a href="https://github.com/calvin-li-developer">@calvin-li-developer</a>)</summary>

`docker-compose.yml`：

```yaml
version: '3'

services:
  vaultwarden:
    container_name: vaultwarden
    image: vaultwarden/server:latest
    restart: unless-stopped
    environment:
      DOMAIN: "https://vaultwarden.example.ltd"  # 您的域名；vaultwarden 需要知道您的域名是 https，才能正常处理附件
    volumes:
      - ./vw-data:/data
    networks:
      - vaultwarden-network

  cloudflared:
    image: cloudflare/cloudflared:2024.1.2
    container_name: vaultwarden-cloudflared
    restart: unless-stopped
    read_only: true
    volumes:
      - ./cloudflared-config:/root/.cloudflared/
    command: [ "tunnel", "run", "${TUNNEL_ID}" ]
    user: root
    depends_on:
      - vaultwarden
    networks:
      - vaultwarden-network
networks:
  vaultwarden-network:
    name: vaultwarden-network
    external: false
```

`cloudflared-config` 文件夹中的内容：

```
config.yml  aaaaa-bbbb-cccc-dddd-eeeeeeeee.json
```

请使用[本指南](https://thedxt.ca/2022/10/cloudflare-tunnel-with-docker/)找出您的 cloudflare 账户的以下内容/值。注意：`aaaaa-bbbb-cccc-dddd-eeeeeeeee` 只是一个随机的 tunnelID，请使用真实的 ID。

`config.yml`：

```yaml
tunnel: aaaaa-bbbb-cccc-dddd-eeeeeeeee
credentials-file: /root/.cloudflared/aaaaa-bbbb-cccc-dddd-eeeeeeeee.json

originRequest:
  noHappyEyeballs: true
  disableChunkedEncoding: true
  noTLSVerify: true

ingress:
  - hostname: vault.example.com # 更改为您自己的域名
    service: http_status:404
    path: admin
  - hostname: vault.example.com # 更改为您自己的域名
    service: http://vaultwarden
  - service: http_status:404
```

`aaaaa-bbbb-cccc-dddd-eeeeeeeee.json`：

```
{"AccountTag":"changeme","TunnelSecret":"changeme","TunnelID":"aaaaa-bbbb-cccc-dddd-eeeeeeeee"}
```

</details>

<details>

<summary>Pound</summary>

```
Alive		15

ListenHTTP
	Address 127.0.0.1
	Port    80
	xHTTP 3
	HeadRemove "X-Forwarded-For"
	Service
		Host "vaultwarden.example.tld"
		Redirect 301 "https://vaultwarden.example.tld"
	End
End

ListenHTTPS
	Address 127.0.0.1
	Port    443
	Cert    "/path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem"
	xHTTP 3
	AddHeader "Front-End-Https: on"
	RewriteLocation 0
	HeadRemove "X-Forwarded-Proto"
	AddHeader "X-Forwarded-Proto: https"
End

Service
	Host "vaultwarden.example.tld"
	BackEnd
		Address 127.0.0.1
		Port    8000
	End
End
```

</details>
