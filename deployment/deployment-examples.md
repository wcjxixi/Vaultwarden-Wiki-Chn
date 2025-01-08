# 5.部署示例

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Deployment-examples)
{% endhint %}

本页面是独立部署示例的索引。如果要添加新的示例，请酌情创建一个新的类别，并保持总体有序。

## 自托管 <a href="#self-hosted" id="self-hosted"></a>

本节介绍了在您**自己的硬件**或主要**由您自己管理**的任何基础设施上托管 Vaultwarden 的不同选项。

### 使用 Ansible 进行高可用性 Vaultwarden 部署 <a href="#highly-available-vaultwarden-deployment-with-ansible" id="highly-available-vaultwarden-deployment-with-ansible"></a>

* [https://github.com/sudoix/vaultwarden-ansible](https://github.com/sudoix/vaultwarden-ansible)

此 Ansible 部署使用以下组件设置高可用的 Vaultwarden 集群：

**主要特点：**

* **Nginx**：处理 SSL 卸载和负载平衡，以获得最佳性能和安全性。
* **Certbot**：自动生成和管理 SSL 证书以实现安全通信。
* **Vaultwarden**：作为密码管理的主要后端。
* **Keepalived**：提供虚拟IP和冗余以实现高可用性。
* **PostgreSQL**：使用外部数据库来存储数据。
* **Docker** 和 **Docker Compose**：使用 docker compose 部署所有服务。

### Ansible

* [https://github.com/guerzon/ansible-role-vaultwarden](https://github.com/guerzon/ansible-role-vaultwarden)

目前支持 EL8 和 EL9 发行版的 Ansible 角色。在积极开发和支持下，已经有一个可用的 MVP 版本。

### Raspberry Pi

* [https://github.com/martient/vaultwarden-ansible](https://github.com/martient/vaultwarden-ansible)

Raspberry Pi 上的 Vaultwarden Ansible 部署。要从以前的配置迁移，请按照页面上链接的指南进行操作。

* [https://dietpi.com/](https://dietpi.com/)

[DietPi](https://dietpi.com/) 是一个轻量级的基于 Debian 的发行版（镜像），适用于各种设备，例如 Raspberry Pi、Odroid、NanoPi 等。它提供了一个软件脚本，用于安装包括 Vaultwarden 在内的各种程序。这样可以让用户免去对安装命令的苦恼。

要在 DietPi 上安装 Vaultwarden，只需在命令行中键入 `dietpi-software install 183` 即可。有关在 DietPi 上的安装步骤和首次访问 Vaultwarden 的更多信息，请访问 [https://dietpi.com/docs/software/cloud/#vaultwarden](https://dietpi.com/docs/software/cloud/#vaultwarden)

* [https://mijo.remotenode.io/posts/tailscale-caddy-docker/](https://mijo.remotenode.io/posts/tailscale-caddy-docker/)

使用 Tailscale 和 Caddy 确保安全访问 Vaultwarden 的演练指南。所有服务均使用 Docker Compose 进行容器化管理，并托管在 Raspberry Pi 上。

* [https://github.com/AlphanAksoyoglu/vaultwarden-rpi/](https://github.com/AlphanAksoyoglu/vaultwarden-rpi/)

基于 docker-compose 的、模块化的、自托管的 Vaultwarden 部署。

选项：

* 仅 LAN，或 LAN + Tailscale（通过 VPN 从任何地方访问）
* 您的域名 (Cloudflare) 或 DuckDNS 域名
* 可选的不依赖第三方容器的备份服务
* 可选的 UFW 和 IPTABLES 强化

配有方便的安装程序：

* 只需运行 `install.sh --init` 和 `install.sh --install`

还有一个广泛的自述文件。

### 共享主机 <a href="#shared-hosting" id="shared-hosting"></a>

* [https://github.com/jjlin/vaultwarden-shared-hosting](https://github.com/jjlin/vaultwarden-shared-hosting)

在 [DreamHost](https://www.dreamhost.com/) 上运行 Vaultwarden 的配置示例，但应该也适用于许多其他共享主机服务。

* [https://lab.uberspace.de/guide\_vaultwarden.html?highlight=bitwarden](https://lab.uberspace.de/guide_vaultwarden.html?highlight=bitwarden)

如何从源代码安装以及如何在 [Uberspace](https://uberspace.de/en/) 共享托管服务提供商上运行的说明。

### NixOS (by tklitschi)

这里是一个针对 NixOS 上的 Vaultwarden 配置的示例。它不是很复杂，有您想使用的数据库类型的后端选项、用于系统服务专用备份的备份目录、启用它的选项以及配置选项。对于配置选项，你只需[从 .env 模板](https://github.com/dani-garcia/bitwarden_rs/blob/1.13.1/.env.template)传递 .env 变量到 nix 语法中即可。密码 (SMTP\_PASSWORD,... ) 存储在 /nix/store 之外的另一个 .env 文件中，并被 [services.vaultwarden.environmentFile](https://search.nixos.org/options?channel=21.11\&show=services.vaultwarden.environmentFile\&from=0\&size=50\&sort=relevance\&type=packages\&query=vaultw) 包含。请参阅[代理示例](proxy-examples.md)以了解 nixos-nginx 的配置示例。

<details>

<summary>配置示例</summary>

```nginx
{ pkgs, ... }:
{
  services.bitwarden_rs = {
    enable = true;
    backupDir = "/mnt/bitwarden";
    config = {
      WEB_VAULT_FOLDER = "${pkgs.bitwarden_rs-vault}/share/bitwarden_rs/vault";
      WEB_VAULT_ENABLED = true;
      LOG_FILE = "/var/log/bitwarden";
      WEBSOCKET_ENABLED = true;
      WEBSOCKET_ADDRESS = "0.0.0.0";
      WEBSOCKET_PORT = 3012;
      SIGNUPS_VERIFY = true;
#     ADMIN_TOKEN = (import /etc/nixos/secret/bitwarden.nix).ADMIN_TOKEN;
      DOMAIN = "https://exmaple.com";
#     YUBICO_CLIENT_ID = (import /etc/nixos/secret/bitwarden.nix).YUBICO_CLIENT_ID;
#     YUBICO_SECRET_KEY = (import /etc/nixos/secret/bitwarden.nix).YUBICO_SECRET_KEY;
      YUBICO_SERVER = "https://api.yubico.com/wsapi/2.0/verify";
      SMTP_HOST = "mx.example.com";
      SMTP_FROM = "bitwarden@example.com";
      SMTP_FROM_NAME = "Bitwarden_RS";
      SMTP_PORT = 587;
      SMTP_SECURITY = starttls;
#     SMTP_USERNAME = (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
#     SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
    environmentFile = "/etc/nixos/secret/bitwarden.env";
  };
}
```

如果您有任何关于这部分的问题，请随时联系我。我在 matrix 的 @litschi:litschi.xyz 、以及 IRC（hackint 和 freenode）的 litschi，或简单地在 matrix.org 的 Vaultwarden 频道中询咨询我。

</details>

### QNAP NAS (ARM 和 x86) <a href="#qnap-nas-arm-and-x-86" id="qnap-nas-arm-and-x-86"></a>

* [https://github.com/umireon/vaultwarden-qnap](https://github.com/umireon/vaultwarden-qnap)

您可以使用 Let's Encrypt 将 Vaultwarden 安装到您的安全网络附加存储 (NAS) 中。但由于 QNAP 内置的 HTTP(S) 服务器，您不能在标准的 HTTP(S) 端口 (80/443) 上发布 Vaultwarden。

### Kubernetes Manifests

* [https://github.com/icicimov/kubernetes-bitwarden\_rs](https://github.com/icicimov/kubernetes-bitwarden_rs)

在 Kubernetes 上以 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products) 作为后端设置一个功能齐全且安全的 Vaultwarden 应用程序。它提供的不仅仅是简单的部署，还可以根据您的需要和设置使用全部或部分功能。

### Helm charts

* [https://github.com/Skeen/helm-bitwarden\_rs](https://github.com/Skeen/helm-bitwarden_rs)

在 Kubernetes 上以您选择的 nginx 控制器作为后端设置一个功能齐全且安全的 Vaultwarden 应用程序。它运行良好，并已使用 [microk8s](https://microk8s.io/) 设置进行了测试。而且支持通过 [cert-manager](https://github.com/jetstack/cert-manager) 生成 SSL 证书。

* [https://github.com/guerzon/vaultwarden](https://github.com/guerzon/vaultwarden)

使用 [Helm](https://helm.sh/zh/docs/) 将 Vaultwarden 部署到 Kubernetes 集群。它支持重要的自定义，例如提供图像标签和自定义注册表值、使用​​外部 MySQL 或 PostgreSQL 数据库、使用入口控制器（如 [nginx-ingress](https://kubernetes.github.io/ingress-nginx/deploy/) 和 [AWS LB 入口控制器](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/)）、使用服务账户、配置 SMTP，以及配置存储选项。

此 Helm chart 目前正在积极开发和支持中。

## PaaS 托管 <a href="#paas-hosting" id="paas-hosting"></a>

本节介绍了**在云端**或使用平台即服务 (PaaS) 提供商托管 Vaultwarden 的不同选项。

> \[**译者注**]：[PaaS](https://cloud.google.com/learn/what-is-paas?hl=zh-cn)：Platform as a Service，平台即服务。PaaS 是一种云计算服务模型，提供灵活的可扩缩云平台来开发、部署、运行和管理应用。PaaS 为开发者提供了开发应用所需的所有功能，而不必费心考虑操作系统和开发工具更新或者硬件维护。整个 PaaS 环境（或平台）而是由第三方服务提供商通过云提供。

### AWS EKS

* [https://medium.com/@sreafterhours/deploy-vaultwarden-to-amazon-eks-using-terraform-terragrunt-and-helm-69a0a7396625](https://medium.com/@sreafterhours/deploy-vaultwarden-to-amazon-eks-using-terraform-terragrunt-and-helm-69a0a7396625)

使用 Terraform 和 Infrastructure-as-Code 在亚马逊 EKS 中部署 Vaultwarden。

### Sealos

[![Deploy on Sealos](https://raw.githubusercontent.com/labring-actions/templates/main/Deploy-on-Sealos.svg)](https://cloud.sealos.io/?openapp=system-fastdeploy%3FtemplateName%3Dvaultwarden)

使用完全免费的插件在 Sealos 上安装 Vaultwarden。安装大约需要 1 分钟。优雅地处理高并发并提供动态可扩展性。

### Google Cloud

* [https://github.com/dadatuputi/bitwarden\_gcloud](https://github.com/dadatuputi/bitwarden_gcloud)

针对 Google Cloud 的「永远免费」的 f1-micro 计算实例进行了优化的 Vaultwarden 安装。

* [~~https://medium.com/@sreafterhours/terraform-helm-external-dns-cert-manager-nginx-and-vaultwarden-on-gke-5080f3b4909f~~](https://medium.com/@sreafterhours/terraform-helm-external-dns-cert-manager-nginx-and-vaultwarden-on-gke-5080f3b4909f)

~~针对 Google Kubernetes Engine 的详细的 Vaultwarden 安装，包括基础设施和集群配置。~~

### Heroku

* [https://github.com/davidjameshowell/vaultwarden\_heroku](https://github.com/davidjameshowell/vaultwarden_heroku)

使用完全免费的插件在 Heroku 上安装 Vaultwarden。安装大约需要 15 分钟。

### Fly.io

* [https://github.com/nosovk/vaultwarden-fly-io/blob/main/fly.toml](https://github.com/nosovk/vaultwarden-fly-io/blob/main/fly.toml)

使用 SQLite 数据库安装 Vaultwarden。但是您需要为数据库创建卷：`flyctl volumes create vaultwarden_data -a [your app name] -s 1`

* [https://github.com/arthurgeek/vaultwarden-fly-template](https://github.com/arthurgeek/vaultwarden-fly-template)

在 Fly.io 上部署 Vaultwarden 的模板，具有 websockets 支持（带有 caddy）和使用 Restic 的 sqlite 每小时备份功能。

### Dokku

这是一个脚本，使用上传到 DockerHub 的 docker 镜像自动设置 Vaultwarden，并创建一个 Dokku 应用程序。该脚本假设您已经设置了一个全局域名（即存在 `/home/dokku/VHOST` 文件）。遵循提示进行设置。

```batch
#!/usr/bin/env bash

set -euo pipefail

APPNAME=""

read -rp "Enter the name of the app: " APPNAME

# 检查应用名称是否为空
if [ -z "$APPNAME" ]; then
    echo "App name empty. Using default name: vaultwarden"
    APPNAME="vaultwarden"
fi

# 检查 dokku 插件是否存在
if ! dokku plugin:list | grep letsencrypt; then
    sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
fi
# 检查是否设置了用于 letsencrypt 的全局电子邮件
if ! dokku config:get --global DOKKU_LETSENCRYPT_EMAIL; then
    read -rp "Enter email address for letsencrypt: " EMAIL
    dokku config:set --global DOKKU_LETSENCRYPT_EMAIL="$EMAIL"
fi

# 拉取最新版的镜像
IMAGE_NAME="vaultwarden/server"
docker pull $IMAGE_NAME
image_sha="$(docker inspect --format='{{index .RepoDigests 0}}' $IMAGE_NAME)"
echo "Calculated image sha: $image_sha"
dokku apps:create "$APPNAME"
dokku storage:ensure-directory "$APPNAME"
dokku storage:mount "$APPNAME" /var/lib/dokku/data/storage/"$APPNAME":/data
dokku domains:add $APPNAME $APPNAME."$(cat /home/dokku/VHOST)"
dokku letsencrypt:enable "$APPNAME"
dokku proxy:ports-add "$APPNAME" http:80:80
dokku proxy:ports-add "$APPNAME" https:443:80
dokku proxy:ports-remove "$APPNAME" http:80:5000
dokku proxy:ports-remove "$APPNAME" https:443:5000
dokku git:from-image "$APPNAME" "$image_sha"
```

将上面的脚本复制到您的 Dokku 主机然后运行它。脚本运行成功后，即可通过 `https://$APPNAME.dokku.me` 访问网页密码库。

要更新您的 Vaultwarden 服务器，请运行以下命令（记得将 `$APP_NAME` 替换为您的应用程序的名称）：

```batch
docker rmi -f vaultwarden/server
docker pull vaultwarden/server:latest
image_sha="$(docker inspect --format='{{index .RepoDigests 0}}' vaultwarden/server)"
dokku git:from-image $APP_NAME $image_sha
```

### Azure

* [https://github.com/adamhnat/vaultwarden-azure](https://github.com/adamhnat/vaultwarden-azure)

针对具有数据文件共享的 Azure 容器应用程序服务进行了优化的 Vaultwarden 安装。

### Digital Ocean

* [https://github.com/HarrisonLeach1/vaultwarden\_digitalocean](https://github.com/HarrisonLeach1/vaultwarden_digitalocean)

Digital Ocean 最便宜的 Droplet 的 Vaultwarden 安装。通过 Terraform 设置资源。

***

{% hint style="info" %}
2023-09-25：我们不认可这种托管 Vaultwarden 的方式，因此移除了代管式托管部分。
{% endhint %}

## ~~代管式托管~~ <a href="#managed-hosting" id="managed-hosting"></a>

~~最后，本节展示了代管式 Vaultwarden 托管的不同提供商和选项，如果您根本不想自己去关心配置和管理的话。~~

### ~~Server.Camp~~

* [~~https://server.camp/product/vaultwarden~~](https://server.camp/product/vaultwarden)

~~为开发人员、初创公司和中小型企业提供的基于欧盟且符合 GDPR 的 Vaultwarden 托管。15% 的收入将捐赠给开源社区。~~
