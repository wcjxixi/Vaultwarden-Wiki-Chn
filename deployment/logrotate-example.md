# 7.转储示例

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Logrotate-example)
{% endhint %}

随着时间增长，Vaultwarden 日志文件的大小可能会增长到很大。使用 logrotate，我们可以定期转储日志。

```shell
sudo nano /etc/logrotate.d/vaultwarden
```

```shell
/var/log/vaultwarden/*.log {
    # 以 bitwarden 用户和群组的身份执行轮换
    su vaultwarden vaultwarden
    # 每天轮换
    daily
    # 当尺寸大于 5M 时轮换
    size 5M
    # 压缩旧的日志文件
    compress
    # 在删除或邮寄到 mail 指令中指定的地址之前，保留 4 个轮换的日志文件
    rotate 4
    # 把当前日志备份并截断
    copytruncate
    # 如果日志文件不存在，继续下一个操作
    missingok
    # 如果日志文件为空则不进行轮换
    notifempty
    # 在轮换的日志文件中添加数字格式的日期
    dateext
    # dateext 的日期格式
    dateformat -%Y-%m-%d-%s
}
```

无需手动解压缩而查看压缩的日志文件：

```shell
zcat logfile.gz
zless logfile.gz
zgrep -i keyword_search logfile.gz
```
