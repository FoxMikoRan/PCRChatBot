# PCRChatBot

由[HoshinoBot](https://github.com/Ice-Cirno/HoshinoBot)源生项目，添加安装搜集到的部分插件（[来源](https://github.com/pcrbot)）而来的自用QQ机器人，对其中一些插件内容有修改。

由于之前是部署在本地Windows平台上，现有需求将其移植到CentOS7.6轻量级服务器并服务于自用公会战群，且其他部署教程均有不同程度的不适配，所以留档以备后续维护。

## 1.主要功能

### HoshinoBot项目原生功能

并未做太大改动，修改行为只是以稳定运行为目的进行参数修改，具体接口和实现方法均未作改动。只测试了一些常用的功能，后续功能需求均由第三方插件实现，功能冲突请阅读报错信息自行寻求解决方案。

## 2.部署方法

修改自[CentOS 下安装 HoshinoBot 和 yobot](http://cn.pcrbot.com/deploy-hoshinobot-on-centos/)，使用命令行依次执行即可。

### 1.环境配置

```bash
# 更新系统环境
yum -y update 

yum -y groupinstall "Development tools"

yum -y install wget zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc* libffi-devel make git vim screen

# 安装ffmpeg，后续发送语音可能报错，需要额外配置
yum install ffmpeg ffmpeg-devel 
```

### 2.安装Python3.7.1

先使用cd命令进到合适的路径。

```bash
# 使用华为镜像下载 python3.7.1
wget https://mirrors.huaweicloud.com/python/3.7.1/Python-3.7.1.tgz

# 解压
tar xf Python-3.7.1.tgz

# 进入文件夹
cd Python-3.7.1

# 编译安装
./configure

make&&make install

pip3 install --upgrade pip

#可能用到的命令
/usr/local/bin/pip3 install --upgrade pip
```

后续报错不少都与Python3.7.1相关的环境配置有关，换用第三方插件可能需要更高的版本，兼容性请自行测试。

### 3.安装go-cqhttp

```bash
# 创建并进入 go-cqhttp 目录
cd ~&&mkdir go-cqhttp&&cd go-cqhttp

# 下载最新版的压缩包，最新版可能存在登陆问题，可以考虑回退一个版本
wget https://github.com/Mrs4s/go-cqhttp/releases/download/v1.0.0-rc1/go-cqhttp_linux_amd64.tar.gz

#解压
tar xf go-cqhttp_linux_amd64.tar.gz

#授权
chmod +x go-cqhttp

#第一次运行
./go-cqhttp
```

第一次运行后需生成配置文件，通讯模式一般情况下选择3，回车确定后会生成 config.yml 配置文件。

```bash
# 备份配置文件
mv config.yml config.yml.bak

# 执行下面的指令后点击 i 进入 insert/插入 模式
vi config.yml

# 也可以用使用vim命令，按 a 进入插入模式
vim config.yml
```

主要需要修改的地方为：

```yml
account:
  uin: 1233456 # 你的 bot 的 QQ 账号，有风控危险，建议开启设备锁扫码登录多挂几天
  password: '' # 密码为空时使用扫码登录，建议留空，有时密码登录会产生很麻烦的问题
```

```yml
servers:
  # 反向WS设置
  # 8080 为 HoshinoBot 默认端口号，一般功能主要由HoshinoBot实现
  - ws-reverse:
      universal: ws://127.0.0.1:8080/ws/
      api: ""
      event: ""
      reconnect-interval: 3000
      middlewares:
        <<: *default # 引用默认中间件
  # 9222 为 yobot 默认端口号，对yobot没有需求可以注释掉下面几行，以免出现连接反向服务器失败的报错
  - ws-reverse:
      universal: ws://127.0.0.1:9222/ws/
      api: ""
      event: ""
      reconnect-interval: 3000
      middlewares:
        <<: *default # 引用默认中间件
```

修改完相应的账号密码，输入 `nohup ./go-cqhttp &` 即可后台运行 go-cqhttp，默认输出日志文件为工作目录下的 `nohup.out` 。

推荐先使用`./go-cqhttp` 命令试运行排除报错，再使用 `nohup ./go-cqhttp &` 命令后台运行；`nohup.out`文件会逐渐增大，一般没有清理设定会占用巨量磁盘空间。 

验证码登录时，记得把命令行界面调大点不然显示不全无法扫码。

### 4.安装 HoshinoBot

**原版安装如下，如果仅需要HoshinoBot原生功能可以减少调试工作量。**

```bash
cd ~

git clone https://github.com/Ice-Cirno/HoshinoBot.git

cd HoshinoBot

pip3 install -r requirements.txt

cp -r hoshino/config_example/ hoshino/config/

# 这一步启用或关闭模组
vi hoshino/config/__bot__.py

# 后台运行 HoshinoBot
nohup python3 run.py &
```

还是推荐先使用`python3 run.py` 命令试运行排除报错，再使用 `nohup python3 run.py &` 命令后台运行。



