# 11.更改 worker 数量

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Changing-the-number-of-workers)
{% endhint %}

> \[**译者注**]：worker 相当于工人，就是干活的人。也不知如何翻译才准确，就不翻译这个词了。「Master-Worker 模式是常用的并行设计模式。核心思想是，系统由两个角色组成：Master 和 Worker。Master 负责接收和分配任务，Worker 负责处理子任务。任务处理过程中，Master 还负责监督任务进展和 Worker 的健康状态；Master 将接收 Client 提交的任务，并将任务的进展汇总反馈给 Client。」

当 Vaultwarden 运行时，默认它会产生 `2 * <cpu 核心数>` 个 worker 来处理请求。在某些系统上，这可能会由于 worker 数量太少，从而导致性能降低，因此在 docker 镜像中更改为默认产生 10 个线程。您可以通过设置 `ROCKET_WORKERS` 变量来增加或减少 worker 数量以覆盖此默认设置。

在下面的示例中，我们设置为 20 个 worker：

```shell
docker run -d --name vaultwarden \
  -e ROCKET_WORKERS=20 \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
