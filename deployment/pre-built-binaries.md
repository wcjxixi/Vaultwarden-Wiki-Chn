# 3.预构建二进制

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Pre-built-binaries)
{% endhint %}

Vaultwarden 目前并没有提供独立的二进制文件作为单独的下载，但您可以从基于 Alpine 的官方 Docker 镜像中提取独立的、静态链接的二进制文件。每个 Docker 镜像还包括一个匹配的网页密码库构建（与平台无关）。

## 在已安装 Docker 情况下提取二进制文件 <a href="#extracting-binaries-with-docker-installed" id="extracting-binaries-with-docker-installed"></a>

假设要为您运行的平台提取二进制文件：

```shell
docker pull docker.io/vaultwarden/server:latest-alpine
docker create --name vw docker.io/vaultwarden/server:latest-alpine
docker cp vw:/vaultwarden .
docker cp vw:/web-vault .
docker rm vw
```

如果您想获取不同平台的二进制文件（例如，您的 x86-64 机器上只安装了 Docker，但您想在 Raspberry Pi 上运行 Vaultwarden）， 将 `--platform` 选项添加到 `docker pull` 命令中：

```shell
docker pull --platform linux/arm/v7 docker.io/vaultwarden/server:latest-alpine
# 按照上面的方法运行其余的命令。
# 注意， `docker create` 命令可能会输出如下类似的信息：
#   WARNING: The requested image's platform (linux/arm/v7) does not match the detected host platform (linux/amd64)
#   and no specific platform was requested
# 这是预料之中的，不用担心。
```

## 在未安装 Docker 情况下提取二进制文件 <a href="#extracting-binaries-without-docker-installed" id="extracting-binaries-without-docker-installed"></a>

如果您不能或不想安装 Docker，您可以使用 [docker-imag-extract](https://github.com/jjlin/docker-image-extract) 脚本来拉取和提取 Docker 镜像。例如，要拉取和提取 x86-64 镜像：

```shell
$ mkdir vm-image
$ cd vm-image
$ wget https://raw.githubusercontent.com/jjlin/docker-image-extract/main/docker-image-extract
$ chmod +x docker-image-extract
$ ./docker-image-extract docker.io/vaultwarden/server:latest-alpine
Getting API token...
Getting image manifest for docker.io/vaultwarden/server:latest-alpine...
Downloading layer 801bfaa63ef2094d770c809815b9e2b9c1194728e5e754ef7bc764030e140cea...
Extracting layer...
Downloading layer c6d331ed95271d8005dea195449ab4ef943017dc97ab134a4426faf441ae4fa6...
Extracting layer...
Downloading layer bfd9ec32f740ca8c86ccde057595d29a31eb093aafd7619fcdd4b956c7bf95e3...
Extracting layer...
Downloading layer e9bfb5d92e4629b1dcb4a13a470c90f51b9edde4e184d8520afc589728b8b675...
Extracting layer...
Downloading layer 5757963c858ce72bc4a1874f4971d326d21d2a844f03063a3c99e312150adf95...
Extracting layer...
Downloading layer f705bf64e4315fea1830cc137d1deda194e825da03bd7822e41ac52457bc83e7...
Extracting layer...
Downloading layer 909b5deb38cbce9f83598918bf7f38b7c2194d385456cf7ef15eff47f8a63108...
Extracting layer...
Downloading layer 8516f4cd818630cd60fa18254b072f8d9c3748bdb56f6e2527dc1c204e8e017c...
Extracting layer...
Image contents extracted into ./output.
$ ls -ld output/{vaultwarden,web-vault}
-rwx------ 1 user user 22054608 Feb  6 21:46 output/vaultwarden
drwx------ 8 user user     4096 Feb  6 21:46 output/web-vault/
```

要拉取并提取其他平台的镜像：

* ARMv6：`./docker-image-extract -p linux/arm/v6 docker.io/vaultwarden/server:latest-alpine`
* ARMv7：`./docker-image-extract -p linux/arm/v7 docker.io/vaultwarden/server:latest-alpine`
* ARMv8 / AArch64：`./docker-image-extract -p linux/arm64 docker.io/vaultwarden/server:latest-alpine`

或使用 github actions 从[此存储库](https://github.com/czyt/vaultwarden-binary)自动提取二进制文件。
