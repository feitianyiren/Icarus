# Icarus

![](misc/screenshot.png)

> A opensource forum project write with python3 and vue.js

[更多截图(旧版)](SCREENSHOT.md)


## 特性

* 用户系统

    * 注册、登录

    * 邮件激活

    * 邮箱找回密码

    * 修改个人信息

    * 上传头像（七牛云）

    * 每日签到

    * 个人提醒

* 论坛

    * 扁平化的内容展示

    * 创建和管理板块

    * 板块主题颜色

    * 发表和编辑主题

    * 文章页自动生成快捷导航

    * @功能

* 百科

    * 自定义侧边栏和主页

    * 文章的创建和编辑

    * 全部文章列表

    * 文章历史

    * 随机页面

* 管理后台

    * 提供对板块、主题、用户、评论的管理

    * 管理日志

* 安全机制

    * 前端密码加密，后端不取得用户的初始密码，最大限度降低了中间人攻击和数据库泄露的危害

    * 后端再次加密，sha512加盐迭代十万次后储存密码

    * 密码相关API均有防爆破，可设置IP请求间隔和账号请求间隔，分别提升批量撞库和单点爆破的难度

    * 隐私数据，例如IP地址脱敏后才可存入数据库

* 其他

    * 实时在线人数

    * 宽屏支持


## 如何部署

首先 clone 项目。

```bash
git clone https://github.com/fy0/Icarus.git
```

下面逐项照做即可。

迫于前端安装node_modules等待时间长，在环境安装完成后，前端篇和后端篇可以一起做，以节省时间。


## 环境依赖篇

### 1. Python 3.6+

Windows上直接使用Anaconda3或者官方版本。

Linux上部分发行版（例如ArchLinux）天然满足要求。

这里只说我的个人环境，Debian/Ubuntu 解决方案：

```bash
# 这个 deadsnakes 的 python 源并非最流行的那一款 Python3.6 第三方源
# 但是细节要比流行的源强得多，举例来说 ubuntu lts 现在内置 Python 3.5
# 安装了那个源之后会发生 3.5 和 3.6 的 pip 互相打架无法并存的情况
# 这个源这点就做的比较好，而且长期维护，很稳定
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install -y python3.6 python3.6-dev python3.6-venv
sudo su -c "curl https://bootstrap.pypa.io/get-pip.py | python3.6"
```


### 2. NodeJS

建议使用LTS版本的 nodejs，通过包管理器安装：

https://nodejs.org/en/download/package-manager/

```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```


### 3. PostgreSQL

官方提供了一系列操作系统的安装解决方案：https://www.postgresql.org/download/

Windows下你可以下载安装包，主流Linux可以使用包管理器添加软件源。

还是以ubuntu举例：https://www.postgresql.org/download/linux/ubuntu/

```bash
# 为 ubuntu 16.04 添加 PG 源
sudo su -c "echo 'deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo apt-get update

# 安装PostgreSQL，需要9.6以上版本
sudo apt-get install -y postgresql-10
```

装好之后做一些配置
```bash
sudo su postgres
createdb icarus
createuser icarus
psql
# 进入 PQ Shell
GRANT ALL ON DATABASE icarus TO icarus;
ALTER USER icarus WITH PASSWORD 'IcaruStest123';
```

### 4. Redis

一般直接使用包管理器安装就可以了。

Windows上可以使用[微软提供的二进制版本](https://github.com/MicrosoftArchive/redis/releases)


## 后端篇

运行 backend 目录下的 `main.py`，初次运行会创建自动 `private.py` 并退出。

这个文件里的内容会覆盖 `config.py` 中的配置。

然后逐项对内容进行修改，使其符合你机器的实际情况即可。

建议使用 pipenv 进行部署。

```bash
sudo pip3.6 install pipenv
pipenv install
```

又或者使用requirements.txt进行比较传统的安装。

> 特别地，在Windows上安装aioredis库时可能会遇到依赖的hiredis包无法安装的问题  
> 如果是anaconda用户，那么可以使用：  
> ```bash
> conda config --append channels conda-forge  
> conda install hiredis
> ```  
> 来进行安装。  
> 如果不是anaconda用户，那么可以直接访问
> https://anaconda.org/conda-forge/hiredis/files  
> 直接下载.tar.bz2压缩包把里面的site-packages/hiredis解压出来放到自己的site-packages即可


环境装完之后，这样启动服务：
```bash
pipenv shell
python main.py
```

不过有个问题就是 pipenv 太慢，总是在 Locking，给个venv的方案：

```bash
cd Icarus/backend
python3.6 -m venv .
source bin/activate
pip3.6 install requirements.txt
python3.6 main.py
```

这样也能启动服务。

另外与前端相似的，如果你觉得安装太慢，一样可以使用国内源：
```bash
mkdir -p ~/.config/pip
echo -e '[global]\nindex-url = https://mirrors.ustc.edu.cn/pypi/web/simple\nformat = columns' > ~/.config/pip/pip.conf
```

## 前端篇

```bash
# 安装项目依赖
cd Icarus
npm install
```

迫于安装时间过长，国内可以使用cnpm：
```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install
```

如果只是开发环境下看看效果，那么：
```bash
npm run serve
```
然后在浏览器中查看即可。


如果需要配置外部访问，那么注意：

在 Icarus 目录下新建一个 private.js，并按照以下几例进行填写

```js
// 单端口方案
var host = window.location.host

export default {
    remote: {
        API_SERVER: '//' + host,
        WS_SERVER: 'ws://' + host + '/ws',
    },
    qiniu: {
        server: 'http://upload.qiniu.com',
        // host: '//test-bucket.myrpg.cn'
    }
}
```

```js
// 双端口方案
var host = window.location.hostname

export default {
    remote: {
        API_SERVER: '//' + host + ':9002',
        WS_SERVER: 'ws://' + host + ':9002/ws',
    },
    qiniu: {
        server: 'http://upload.qiniu.com',
        // host: '//test-bucket.myrpg.cn'
    }
}
```

```bash
npm run build
```
生成dist目录备用。


## 扩展篇：Nginx部署

首先安装nginx，再复制配置文件模板并做简单修改。

这里使用的是单端口模板，即使用 9001 向外网提供服务。

```bash
sudo apt install nginx
cd Icarus
sudo cp misc/icarus-1port.conf /etc/nginx/conf.d/
```

随后编辑 /etc/nginx/conf.d/icarus-1port.conf，将
```
# root /home/{user}/Icarus/dist;
```
修改为正确的路径，重启服务：

```bash
sudo service nginx restart
```

访问服务器IP的9001端口，就可以看到最上面截图中的画面了。

第一个注册的用户将自动成为管理员。

## 升级指南

首先停止服务并更新源码。

然后请寻找 `backend/misc/upgrade` 目录下对应的升级文件，例如1.1升级1.2使用`u11-u12.py`。

复制到 `backend` 目录执行后删除即可。

注意如果使用了 pipenv 或其他虚拟环境，要在项目对应环境中完成这个操作。

然后分别升级前端项目(根目录)和后端项目(backend目录)的项目依赖。

如该版本无特别的升级说明，此时直接重新开启服务即可。
