# \*Kubernetes 部署

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Kubernetes-deployment)
{% endhint %}

在 Kubernetes 上部署有两种方式：

* 本地部署
* 通过 [Helm](https://helm.sh/) 部署

## 本地部署： <a href="#natively" id="natively"></a>

请查看 [kubernetes-bitwarden\_rs](https://github.com/icicimov/kubernetes-bitwarden_rs) 存储库，以获取在 Kubernetes 中部署的示例。

它将在 Kubernetes 中的 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products) 后面设置一个功能完整且安全的 `vaultwarden` 应用程序。它提供的不仅仅是简单的部署，您可以根据自身需求和环境，选择使用全部或部分配置文件。

## 通过 Helm 部署： <a href="#via-helm" id="via-helm"></a>

请查看 [guerzon/vaultwarden](https://github.com/guerzon/vaultwarden/tree/main/charts/vaultwarden) 存储库，以获取在 Kubernetes 中部署的示例。

另一个具有同等甚至更高灵活性的选项是：[https://github.com/gissilabs/charts/tree/master/vaultwarden](https://github.com/gissilabs/charts/tree/master/vaultwarden)
