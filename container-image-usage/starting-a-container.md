# 2.启动容器

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Starting-a-Container)
{% endhint %}

请注意，`docker run` 命令的名字略有误导性，因为它不仅会创建一个容器，它还会启动此容器。当在仅停止容器而不删除容器后使用 `docker run` 命令时，会导致发生冲突。为了一个简单的开始，请接着往下看。

## 创建容器 <a href="#creating-the-container" id="creating-the-container"></a>

持久性数据存储在容器内的 `/data` 下，因此使用 Docker 进行持久性部署的唯一要求是在路径上挂载持久性卷：

```python
# 使用 Docker:
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
# 使用 Podman as non-root:
podman run -d --name vaultwarden -v /vw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 vaultwarden/server:latest
# 使用 Podman as root:
sudo podman run -d --name vaultwarden -v /vw-data:/data/:Z -p 80:80 vaultwarden/server:latest
```

所有持久性数据将保存在 `/vw-data/` 路径下，您可以根据自己的需要调整此路径。

该服务将暴露在主机的 80 或 8080 端口上。默认情况下，非 root 容器不允许使用特权端口（<1024），因此需要通过端口映射来传递 `ROCKET_PORT` 环境变量以更改 Vaultwarden 的监听端口。

对于非 x86 硬件或要运行特定版本，可以[选择其他镜像](which-container-image-to-use.md)。

如果您的 docker/vaultwarden 运行在具有固定 IP 的设备上，则可以将主机端口绑定到该 IP 地址，从而避免将主机端口暴露到网络上。如下所示，将 IP 地址（例如 192.168.0.2）添加到主机端口和容器端口前面：

```python
# 使用 Docker:
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 192.168.0.2:80:80 vaultwarden/server:latest
```

## 启动容器 <a href="#starting-the-container" id="starting-the-container"></a>

如果运行了 `docker stop vaultwarden` 命令，或重启，亦或任何其他原因，容器停止了，则可以使用以下命令将其启动：

```python
docker start vaultwarden
```

## 自定义容器启动 <a href="#customizing-container-startup" id="customizing-container-startup"></a>

如果你想在容器启动时运行自定义启动脚本，你可以将 `/etc/vaultwarden.sh` 作为单个脚本和/或将 `/etc/vaultwarden.d` 作为脚本目录挂载到容器中。对于后一种情况，只有扩展名为 .sh 的文件才会运行，所以其他扩展名的文件（例如，data/config 文件）则可以驻留在同一个目录中（具体的工作方式请参见 [start.sh](https://github.com/dani-garcia/vaultwarden/blob/main/docker/start.sh)）。

自定义启动脚本对于修补网页密码库文件或安装额外的包、CA 证书等非常有用，因为可以让您不必构建和维护您自己的 Docker 镜像。

### 示例 <a href="#example" id="example"></a>

假设你的脚本名为 `init.sh`，其包含以下内容：

```python
echo "starting up"
```

您可以像这样在启动时运行脚本：

```python
docker run -d --name vaultwarden -v $(pwd)/init.sh:/etc/vaultwarden.sh <other docker args...> vaultwarden/server:latest
```

如果你运行 `docker logs vaultwarden`，现在你应该能看到 `starting up` 作为输出的第一行。

请注意，每次容器启动时都会运行初始化脚本（而不仅仅是第一次），所以这些脚本通常应该是幂等的（即，你可以多次运行这些脚本而不会出现不良/异常）。如果你的脚本天然没有此属性，你可以这样做：

```python
if [ ! -e /.init ]; then
  touch /.init

  # 运行您的初始化步骤...
fi
```
