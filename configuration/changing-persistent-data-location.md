# 9.更改持久性数据位置

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Changing-persistent-data-location)
{% endhint %}

## /data 前缀 <a href="#data-prefix" id="data-prefix"></a>

默认情况下，所有持久性数据都保存在 `/data` 下，您可以通过设置 `DATA_FOLDER` 环境变量来覆盖此路径：

```python
docker run -d --name vaultwarden \
  -e DATA_FOLDER=/persistent \
  -v /vw-data/:/persistent/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，您需要相应地调整您的卷挂载。

## 数据库名称和位置 <a href="#database-name-and-location" id="database-name-and-location"></a>

默认值为 `$DATA_FOLDER/db.sqlite3`，您可以使用 `DATABASE_URL` 变量专门为数据库更改路径：

```python
docker run -d --name vaultwarden \
  -e DATABASE_URL=/database/vaultwarden.sqlite3 \
  -v /vw-data/:/data/ \
  -v /vw-database/:/database/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，如果数据库和其他持久性数据在不同的位置，记得为他们挂载卷。

## 附件位置 <a href="#attachments-location" id="attachments-location"></a>

默认值为 `$DATA_FOLDER/attachments`，您可以使用 `ATTACHMENTS_FOLDER` 变量更改路径：

```python
docker run -d --name vaultwarden \
  -e ATTACHMENTS_FOLDER=/attachments \
  -v /vw-data/:/data/ \
  -v /vw-attachments/:/attachments/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，如果附件和其他持久性数据在不同的位置，记得为他们挂载卷。

## 图标缓存位置 <a href="#icons-cache" id="icons-cache"></a>

默认值为 `$DATA_FOLDER/icon_cache`，您可以使用 `ICON_CACHE_FOLDER` 变量更改路径：

```python
docker run -d --name vaultwarden \
  -e ICON_CACHE_FOLDER=/icon_cache \
  -v /vw-data/:/data/ \
  -v /icon_cache/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，在上面的示例中，我们没有在本地挂载该卷，这意味着在升级过程中将不会保留该卷，除非您用 `--volumes-from` 使用中间数据容器。这会影响性能，因为 Vaultwarden 在重新启动时必须重新下载图标。但由于图标不会被自动清除，因此可免于在缓存中保留过时的图标。
