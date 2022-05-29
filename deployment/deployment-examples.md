# 5.部署示例

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Deployment-examples)
{% endhint %}

本页面是独立部署示例的索引。如果要添加新的示例，请在适当的时候创建一个新的类别，并在总体上保持有序。

## Google Cloud

* [https://github.com/dadatuputi/bitwarden\_gcloud](https://github.com/dadatuputi/bitwarden\_gcloud)

针对 Google Cloud 的「永远免费」的 f1-micro 计算实例进行了优化的 Vaultwarden 安装。

## Kubernetes

* [https://github.com/icicimov/kubernetes-bitwarden\_rs](https://github.com/icicimov/kubernetes-bitwarden\_rs)

在 Kubernetes 上以 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details\_for\_Elastic\_Load\_Balancing\_Products) 作为后端设置一个功能齐全且安全的 Vaultwarden 应用程序。它提供的不仅仅是简单的部署，还可以根据您的需要和设置使用全部或部分功能。

* [https://github.com/Skeen/helm-bitwarden\_rs](https://github.com/Skeen/helm-bitwarden\_rs)

在 Kubernetes 上以您选择的 nginx 控制器作为后端设置一个功能齐全且安全的 Vaultwarden 应用程序。它运行良好，并已使用 [microk8s](https://microk8s.io/) 设置进行了测试。而且支持通过 [cert-manager](https://github.com/jetstack/cert-manager) 生成 SSL 证书。

* [https://github.com/guerzon/vaultwarden](https://github.com/guerzon/vaultwarden)

使用 [Helm](https://helm.sh/zh/docs/) 将 Vaultwarden 部署到 Kubernetes 集群。它支持重要的自定义，例如提供图像标签和自定义注册表值、使用​​外部 MySQL 或 PostgreSQL 数据库、使用入口控制器（如 [nginx-ingress](https://kubernetes.github.io/ingress-nginx/deploy/) 和 [AWS LB 入口控制器](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/)）、使用服务帐户、配置 SMTP，以及配置存储选项。它拥有充分的记录，未来将继续引入更多配置选项。

## Raspberry Pi

* [https://github.com/martient/vaultwarden-ansible](https://github.com/martient/vaultwarden-ansible)

raspberry pi 上的 Vaultwarden Ansible 部署。要从以前的配置迁移，请遵循本指南 [https://martient.medium.com/migrate-from-bitwarden-rs-to-vaultwarden-199aeb6927a3](https://martient.medium.com/migrate-from-bitwarden-rs-to-vaultwarden-199aeb6927a3)

## 共享主机 <a href="#shared-hosting" id="shared-hosting"></a>

* [https://github.com/jjlin/vaultwarden-shared-hosting](https://github.com/jjlin/vaultwarden-shared-hosting)

在 [DreamHost](https://www.dreamhost.com/) 上运行 Vaultwarden 的配置示例，但应该也适用于许多其他共享主机服务。

* [https://lab.uberspace.de/guide\_vaultwarden.html?highlight=bitwarden](https://lab.uberspace.de/guide\_vaultwarden.html?highlight=bitwarden)

如何从源代码安装以及如何在 [Uberspace](https://uberspace.de/en/) 共享托管服务提供商上运行的说明。

## NixOS (by tklitschi)

这里是一个针对 NixOS 上的 Vaultwarden 配置的示例。它不是很复杂，有您想使用的数据库类型的后端选项、用于系统服务专用备份的备份目录、启用它的选项以及配置选项。对于配置选项，你只需[从 .env 模板](https://github.com/dani-garcia/bitwarden\_rs/blob/1.13.1/.env.template)传递 .env 变量到 nix 语法中即可。密码 (SMTP\_PASSWORD,... ) 存储在 /nix/store 之外的另一个 .env 文件中，并被 [services.vaultwarden.environmentFile](https://search.nixos.org/options?channel=21.11\&show=services.vaultwarden.environmentFile\&from=0\&size=50\&sort=relevance\&type=packages\&query=vaultw) 包含。请参阅[代理示例](proxy-examples.md)以了解 nixos-nginx 的配置示例。

<details>

<summary>配置示例</summary>

```yaml
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
      SMTP_SSL = true;
#     SMTP_USERNAME = (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
#     SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
    environmentFile = "/etc/nixos/secret/bitwarden.env";
  };
}
```

如果你有任何关于这部分的问题，请随时联系我。我在 matrix 的 @litschi:litschi.xyz 、以及 IRC（hackint 和 freenode）的 litschi，或简单地在 matrix.org 的 Vaultwarden 频道中询咨询我。

</details>

## QNAP NAS (ARM 和 x86) <a href="#qnap-nas-arm-and-x-86" id="qnap-nas-arm-and-x-86"></a>

* [https://github.com/umireon/vaultwarden-qnap](https://github.com/umireon/vaultwarden-qnap)

您可以使用 Let's Encrypt 将 Vaultwarden 安装到您的安全网络附加存储 (NAS) 中。但由于 QNAP 内置的 HTTP(S) 服务器，您不能在标准的 HTTP(S) 端口 (80/443) 上发布 Vaultwarden。
