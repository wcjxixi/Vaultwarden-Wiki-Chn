# 通用（非 docker）

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/General-\(not-docker\))
{% endhint %}

## 备份 <a href="#backup" id="backup"></a>

需要包含在备份中的内容：

* 启动 Vaultwarden 时使用的环境文件
* `data` 目录
* Vaultwarden 数据库
  * SQLite 数据库默认存放在 `data` 目录下
  * 使用 MariaDB/PostgreSQL/MySQL 的数据库备份功能创建备份

确保您记录了备份存储的过程和位置！

## 还原 <a href="#restore" id="restore"></a>

* 安装 Vaultwarden
* （不适用于 SQLite）从备份中恢复数据库
* 恢复环境文件
* 恢复您的 `data` 目录到正确的位置

## 特定平台 <a href="#platform-specific" id="platform-specific"></a>

### FreeBSD 端口 <a href="#freebsd-port" id="freebsd-port"></a>

| 项目          | 位置                                     |
| ----------- | -------------------------------------- |
| Environment | `/usr/local/etc/rc.conf.d/vaultwarden` |
| Data        | `/usr/local/www/vaultwarden/data`      |

### MariaDB / MySQL <a href="#mariadb-mysql" id="mariadb-mysql"></a>

参阅 [MariaDB - 备份和还原概述](https://mariadb.com/kb/en/backup-and-restore-overview/)
