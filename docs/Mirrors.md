# 镜像

本页收录了本站的镜像站列表。

有关自行搭建本站的镜像站的方法，请参见 [建立镜像站](#建立镜像站) 。

### 镜像站列表

- [主站](https://hustoj.wiki/) *由 [宝硕小站](https://www.baoshuo.ren/) 提供和维护*
- [官方镜像站](https://wiki.hustoj.baoshuo.ren/) *由 [宝硕小站](https://www.baoshuo.ren/) 提供和维护*


### 建立镜像站

把大象装进冰箱需要几步我忘了，反正建立本站的镜像站只需要两步：

一、拉取仓库

此处命令将把仓库拉取到 `/var/www/hustoj-wiki` 目录下，可根据实际情况修改。

```bash
git clone -b master https://github.com/renbaoshuo/hustoj-wiki.git /var/www/hustoj-wiki
```

二、配置网站

根据你的网站运行程序，设置网站运行目录。

以 `nginx` 为例：

```nginx
server {
    listen      80;
    server_name wiki.hustoj.baoshuo.ren;
    index       index.html;
    root        /var/www/hustoj-wiki/docs;

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
        expires      30d;
        error_log    off;
        access_log   /dev/null;
    }
    
    location ~ .*\.(js|css)?$ {
        expires      12h;
        error_log    off;
        access_log   /dev/null; 
    }
    access_log  /var/logs/hustoj-wiki.log;
    error_log   /var/logs/hustoj-wiki.error.log;
}
```

此处 `server_name` 请根据自己的**实际域名**进行修改。

网站目录根据先前将仓库拉取到的目录而定。如果把仓库拉取到 `/var/www/hustoj-wiki` 目录下，则实际网站为 `/var/www/hustoj-wiki/docs` 。

额外步骤：自动更新

如果你想要将文档与本项目实时同步，请使用定时任务执行以下命令：

```bash
cd /var/www/hustoj-wiki && git pull
```

其中， `/var/www/hustoj-wiki` 请替换为你自己实际的路径。

