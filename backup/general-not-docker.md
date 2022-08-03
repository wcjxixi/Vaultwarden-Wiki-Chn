# 通用（非 docker）

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/General-\(not-docker\))
{% endhint %}

## 备份 <a href="#backup" id="backup"></a>

需要包含在备份中的内容：

* 启动 Vaultwarden 时使用的环境文件
* `data` 目录
* Vaultwarden 数据库
  * 使用 MariaDB/PostgreSQL/MySQL 的数据库备份功能创建备份

确保您记录了备份存储的过程和位置！

## 还原 <a href="#restore" id="restore"></a>

* 安装 Vaultwarden
* 从备份中恢复数据库
* 恢复环境文件
* 恢复您的 `data` 目录到正确的位置

## 特定平台 <a href="#platform-specific" id="platform-specific"></a>

### FreeBSD 端口 <a href="#freebsd-port" id="freebsd-port"></a>

| 项目          | 位置                                     |
| ----------- | -------------------------------------- |
| Environment | `/usr/local/etc/rc.conf.d/vaultwarden` |
| Data        | `/usr/local/www/vaultwarden/data`      |

### 数据库备份 <a href="#database-backups" id="database-backups"></a>

参阅 [MariaDB - 备份和还原概述](https://mariadb.com/kb/en/backup-and-restore-overview/)
