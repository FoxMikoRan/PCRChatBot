# PCRChatBot

由[HoshinoBot](https://github.com/Ice-Cirno/HoshinoBot)项目，搬运性质的添加安装收集到的部分插件（[来源](https://github.com/pcrbot)）而来的自用QQ机器人，对其中一些插件内容有修改。仅仅是自用，所以对于授权以及管理功能没有做添加和改动，不保证相关功能的稳定性和安全性。

由于之前是部署在本地Windows平台上，现有需求将其移植到CentOS7.6轻量级服务器并服务于自用公会战群，且其他部署教程均有不同程度的不适配，所以留档以备后续维护。

Hoshino与yobot已经是很成熟很好用的机器人，配置难度较低，如果需要一键配置或者bot租用，请参考 云星乃 平台。引用原文：

“加入星乃のお茶会(787493356)，发送“云星乃”就会有专门的女仆指引你获取招待券。
跟随提示，依次确认bot账号、要接入的群号，茶会女仆会将招待券(配置文件)私发给你。
将其复制保存为config.yml，放置于go-cqhttp.exe同目录下，运行登录即可接入！“

## 1.主要功能

### HoshinoBot项目原生功能

并未做太大改动，修改行为只是以稳定运行为目的进行参数修改，具体接口和实现方法均未作改动。只测试了一些常用的功能，后续功能需求均由第三方插件实现，功能冲突请阅读报错信息自行寻求解决方案。一些简易功能的实现是直接在原文件上进行的改动，如果有问题请联系我。

一些不常用的功能没有测试，请参考help和帮助命令。

## 2.部署方法

修改自[CentOS 下安装 HoshinoBot 和 yobot](http://cn.pcrbot.com/deploy-hoshinobot-on-centos/)，使用命令行依次执行即可。

### 1.环境配置

选择使用本地电脑、服务器、虚拟机之一创建centos 7.6系统。

服务器配置要求很低，腾讯云和阿里云都有不错的便宜服务器。（阿里云的服务器优惠可能不稳定）

[腾讯云](https://cloud.tencent.com/act/new)

[阿里云](https://www.aliyun.com/minisite/goods)

几十块一年的服务器还是挺实惠的，记得取消自动付费。

```bash
# 更新系统环境
yum -y update 

yum -y groupinstall "Development tools"

yum -y install wget zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc* libffi-devel make git vim screen

# 安装ffmpeg，后续发送语音可能报错，可能需要额外配置
yum install ffmpeg ffmpeg-devel 
```

### 2.安装Python3.8

```bash
# 安装依赖
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel gcc

# 解压
xz -d Python-3.8.0.tar.xz
tar -xvf Python-3.8.0.tar

# 创建文件夹
mkdir /usr/local/python3.8.0

#编译安装
cd Python-3.8.0
./configure --with-ssl --prefix=/usr/local/python3.8.0
$>make && make install

# 创建python3.8和pip3.8软链接
ln -s /usr/local/python3.8.0/bin/python3.8 /usr/bin/python3.8
ln -s /usr/local/python3.8.0/bin/pip3.8 /usr/bin/pip3.8

# 升级pip3
pip3.8 install --upgrade pip
```

后续报错不少都与Python3.8相关的环境配置有关，换用第三方插件可能需要更高的版本，兼容性请自行测试。

### 3.安装go-cqhttp

```bash
# 创建并进入 go-cqhttp 目录
cd ~&&mkdir go-cqhttp&&cd go-cqhttp

# 下载压缩包，最新版可能存在登陆问题
wget https://github.com/Mrs4s/go-cqhttp/releases/download/v1.0.0-rc1/go-cqhttp_linux_amd64.tar.gz
# 推荐使用镜像站进行下载
wget https://hub.fastgit.xyz/Mrs4s/go-cqhttp/releases/download/v1.0.0-rc1/go-cqhttp_linux_amd64.tar.gz

# 或者考虑回退一个版本
wget https://github.com/Mrs4s/go-cqhttp/releases/download/v1.0.0-beta8-fix2/go-cqhttp_linux_amd64.tar.gz
# 推荐使用镜像站进行下载
wget https://hub.fastgit.xyz/Mrs4s/go-cqhttp/releases/download/v1.0.0-beta8-fix2/go-cqhttp_linux_amd64.tar.gz

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
      reconnect-interval: 5000
      middlewares:
        <<: *default # 引用默认中间件
  # 9222 为 yobot 默认端口号，对yobot没有需求可以注释掉下面几行，以免出现连接反向服务器失败的报错
  - ws-reverse:
      universal: ws://127.0.0.1:9222/ws/
      api: ""
      event: ""
      reconnect-interval: 5000
      middlewares:
        <<: *default # 引用默认中间件
```

修改完相应的账号密码，输入 `nohup ./go-cqhttp &` 即可后台运行 go-cqhttp，默认输出日志文件为工作目录下的 `nohup.out` 。

推荐先使用`./go-cqhttp` 命令试运行排除报错，再使用 `nohup ./go-cqhttp &` 命令后台运行；`nohup.out`文件会逐渐增大，一般没有清理设定会占用巨量磁盘空间。 

验证码登录时，记得把命令行界面调大点不然显示不全无法扫码。

### 3.1 防止风控

默认的登陆模式很容易被风控，如果被风控了的话需要一些额外操作：

使用真实安卓手机登陆bot的qq号，开启设备锁，然后编辑config.yml

`vim config.yml`

```bash
account:
  uin: 1233456 # 你的 bot 的 QQ 账号
  password: '' # 被风控时填写密码登陆
```

编辑device.json

```bash
vim device.json
```

```bash
# 修改登录模式为安卓手机
"protocol": 1,
```

删除session.token

`rm session.token`

重新运行go-cqhttp，按提示登陆

`./go-cqhttp`

### 4.安装 HoshinoBot

**（二选一）原版安装如下，如果仅需要HoshinoBot原生功能可以减少调试工作量。**

推荐参考[HoshinoBot](https://github.com/Ice-Cirno/HoshinoBot)，原作者已经更新了更好用的原版部署教程（windows平台与Linux平台）。

```bash
cd ~

# github访问可能不稳定
git clone https://github.com/Ice-Cirno/HoshinoBot.git
# 推荐使用镜像站进行下载
git clone https://hub.fastgit.xyz/Ice-Cirno/HoshinoBot.git

cd HoshinoBot

pip3 install -r requirements.txt

cp -r hoshino/config_example/ hoshino/config/

# 编辑配置文件
vim hoshino/config/__bot__.py

# 需要修改的内容有：
SUPERUSERS = [10000]        # 填写超级用户的QQ号，可填多个用半角逗号","隔开，权限为 SUPERUSER = 999
NICKNAME = ''               # 机器人的昵称。呼叫昵称等同于@bot，可用元组配置多个昵称


# 后台运行 HoshinoBot
nohup python3 run.py &
```

还是推荐先使用`python3 run.py` 命令试运行排除报错，再使用 `nohup python3 run.py &` 命令后台运行。



**（二选一）本版安装如下，与上方原版二选一。**

```bash
cd ~

git clone https://github.com/FoxMikoRan/PCRChatBot.git
# 推荐使用镜像站进行下载
git clone https://hub.fastgit.xyz/FoxMikoRan/PCRChatBot.git

cd PCRChatBot

pip3 install -r requirements.txt

cp -r hoshino/config_example/ hoshino/config/

# 编辑配置文件
vim hoshino/config/__bot__.py

# 需要修改的内容有：
SUPERUSERS = [10000]        # 填写超级用户的QQ号，可填多个用半角逗号","隔开，权限为 SUPERUSER = 999
NICKNAME = ''               # 机器人的昵称。呼叫昵称等同于@bot，可用元组配置多个昵称


# 后台运行 HoshinoBot
nohup python3.8 run.py &
```

requirements.txt中已经加上了部分后续拓展插件的需求，并将常见报错的需求版本回退。

还是推荐先使用`python3 run.py` 命令试运行排除报错，再使用 `nohup python3 run.py &` 命令后台运行。

### 5.1 可选装的yobot

yobot 可独立进行安装，不依赖于 HoshinoBot，注意配置好步骤 3 中的反向 ws 端口号即可（默认为9222）。

部分插件也能达到简易的报刀需求，参见[插件索引](https://github.com/pcrbot/HoshinoBot-plugins-index)。

yobot的工作方式可以使独立的，也可以是作为插件模组嵌入式运行，拓展插件模式可能会对yobot的安装路径有要求，作为模组的标准安装路径可能是：client文件夹在modules/yobot/yobot/src/路径下。

```bash
cd ~

git clone https://github.com/yuudi/yobot.git
# 推荐使用镜像站进行下载
git clone https://hub.fastgit.xyz/yuudi/yobot.git

cd yobot/src/client

pip3 install -r requirements.txt

python3 main.py

# 后台运行 yobot
nohup sh yobotg.sh &
```

还是推荐先使用`sh yobotg.sh` 命令试运行排除报错，再使用 `nohup sh yobotg.sh &` 命令后台运行。



### 5.2 访问网页版 yobot

在服务器安全组中开启端口 9222，具体方法自行百度或[参照此文](https://developer.aliyun.com/article/767328)

之后在浏览器中输入公网 ip:9222 即可访问 yobot 网页端，更多指令，[参照 yobot 官网](https://yobot.win/)

## 3.其他

### 1.插件管理

额外插件的安装方法基本一致，通用方法是在modules文件夹中放入插件，对设置文件进行配置后，打开`hoshino/config/__bot__.py`，在`MODULES_ON = {}`中仿照格式填入modules文件夹中插件所属的文件夹名，在文件夹名前方#进行注释即可暂时屏蔽指定插件。（请逐个添加插件以免出现bug难以解决）

### 2.引用的插件设置

已经安装的部分插件需要额外配置：

#### 1. [pcrjjc2](https://github.com/cc004/pcrjjc2)

##### 配置

在modules/pcrjjc2/目录中，打开并更改`account.json`内的account和password为你的bilibili账号的用户名和密码, admin为管理员的qq，用来接受bilibili验证码进行登录；机器人登录需要验证码时会将链接形式私聊发给admin，这时你需要点进链接正确验证，如果成功，将会出现如下的内容，例如：
`validate=c721fe67f0196d7defad7245f6e58d62 seccode=c721fe67f0196d7defad7245f6e58d62|jordan`
此时，你需要将验证结果发给机器人，通过指令`/pcrval c721fe67f0196d7defad7245f6e58d62`即可完成验证

##### 命令

| 关键词             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| 竞技场绑定 uid     | 绑定竞技场排名变动推送，默认双场均启用，仅排名降低时推送     |
| 竞技场查询 uid     | 查询竞技场简要信息（绑定后无需输入uid）                      |
| 停止竞技场订阅     | 停止战斗竞技场排名变动推送                                   |
| 停止公主竞技场订阅 | 停止公主竞技场排名变动推送                                   |
| 启用竞技场订阅     | 启用战斗竞技场排名变动推送                                   |
| 启用公主竞技场订阅 | 启用公主竞技场排名变动推送                                   |
| 竞技场历史         | 查询战斗竞技场变化记录（战斗竞技场订阅开启有效，可保留10条） |
| 公主竞技场历史     | 查询公主竞技场变化记录（公主竞技场订阅开启有效，可保留10条） |
| 删除竞技场订阅     | 删除竞技场排名变动推送绑定                                   |
| 竞技场订阅状态     | 查看排名变动推送绑定状态                                     |
| 详细查询 uid       | 查询账号详细状态（绑定后无需输入uid）                        |
| 查询群数           | 查询bot所在群的数目                                          |
| 查询竞技场订阅数   | 查询绑定账号的总数量                                         |
| 查询头像框         | 查看自己设置的详细查询里的角色头像框                         |
| 更换头像框         | 更换详细查询生成的头像框，默认彩色                           |
| 清空竞技场订阅     | 清空所有绑定的账号(仅限主人)                                 |

#### 2. [setu_renew](https://github.com/pcrbot/setu_renew)

##### 配置

在modules/setu_renew/目录中， 打开并更改`config.json` , 修改该配置文件设置自己的apikey和其他选项, 除apikey以外都可保持默认值.

`lolicon`分组:

- `mode` : 模块模式, 0=关闭, 1=在线(不使用本地仓库), 2=在线(使用本地仓库), 3=离线(仅使用本地仓库), 默认模式为2.
- `r18` : 是否启用r18
- `use_thumb` : 是否发送大小更小的图
- `pixiv_direct` : 是否直连访问pixiv, 若使用代理请在下方配置代理，并将此项置否，大陆地区访问pixiv不稳定，请配合代理使用
- `proxy_site` : pixiv的代理站或镜像站 **(推荐使用`https://i.pixiv.re/`)**
- **(可以改用`https://i.pixiv.cat/`)**
- `local_proxy` : 本地代理地址, 不需要请留空.（根据本地代理的设置进行修改，一般为`https://127.0.0.1:1080/`）

##### 指令说明

- 来 [num] 张 [keyword] 涩/色/瑟图 : 来num张keyword的涩图 (不指定数量与关键字发送一张随机涩图)
- 涩/色/瑟图 : 发送一张随机涩图
- 本日涩图排行榜 [page] : 获取[第page页]p站排行榜(需开启acggov模块)
- 看涩图 [n] 或 [start end] : 获取p站排行榜[第n名/从start名到end名]色图(需开启acggov模块)
- 提取图片 pid : 获取指定pid的图片,若本地有则会直接发送,否则将给出原图链接.

##### 以下指令仅限超级用户使用

- setu 设置 模块 设置值 [群号]

  : 修改本群或指定群的设置

  设置项 - 取值 - 说明:

  - `acggov` : `开 / 关` 是否开启acggov模块
  - `lolicon` : `开 / 关` 是否开启lolicon模块
  - `lolicon_r18` : `开 / 关` 是否开启lolicon_r18模块
  - `撤回` : `n` 发出的图片在n秒后撤回, 设置为0表示不撤回

- `setu 状态 [群号]` : 查看本群或指定群的模块开启状态

- `setu 缓存` : 手动从api拉取一批图片存入本地仓库(插件每10分钟会自动拉取一次)

- `setu 仓库` : 查询本地仓库图片数量

- `setu 黑/白名单 新增/添加/移除/删除 群号` : 修改黑白名单

#### 3. [pcr_autocb](https://github.com/yoooowi/pcr_autocb)

##### 配置

请自行登录 `https://www.bigfun.cn/tools/pcrteam/` 并将该网站 Cookie 中的 `session-api` 填入modules/pcr_autocb/ `config.json`文件中"session-api": "" 内引号中

请在 `config.json` 中"group": 0 改为群号

公会战开始当日

请用自己手机上的 bigfun 客户端登录并打开一次 pcr 团队战工具，确保手机app上能正确显示内容。

请发送 `init` 初始化 bot

##### 指令

| 指令                        | 效果                                                 |
| --------------------------- | ---------------------------------------------------- |
| 作业                        | 获取作业表链接                                       |
| 状态                        | 获取公会当前排名、Boss 进度及血量                    |
| sl                          | 记录一次 sl                                          |
| sl?                         | 查询今日是否记录过 sl                                |
| 预约 1/2/3/4/5              | 预约 1/2/3/4/5 王                                    |
| 更新boss列表                | 更新当期的 BOSS 列表，在不能正确显示 BOSS 名称时使用 |
| gettime                     | 获取当期会战期间                                     |
| init@bot                    | （管理权限）初始化并重置预约表，更新BOSS列表         |
| 上树/挂树                   | 挂树，55分钟后提醒下树                               |
| 上树/挂树+数字              | 挂树，在X分钟后提醒下树                              |
| 下树                        | 下树并取消提醒                                       |
| 查树                        | 看看树上都有谁                                       |
| 昨日日报                    | 获取昨日 04:59 时的出刀数、排名及伤害                |
| 今日出刀                    | 获取今日公会成员出刀数量                             |
| 昨日出刀                    | 获取昨日公会成员出刀数量                             |
| 注册 + 游戏昵称             | 绑定自己的 QQ 账号与游戏角色                         |
| 注册 + @成员 + 游戏昵称     | （SU权限）绑定指定 QQ 账号与游戏角色                 |
| 查看注册信息                | 查看自己的绑定角色昵称                               |
| 查看注册信息 + @成员        | （SU权限）查看指定 QQ 账号的绑定信息                 |
| 更新注册 + 游戏昵称         | 更新绑定信息                                         |
| 更新注册 + @成员 + 游戏昵称 | （SU权限）更新指定 QQ 账号的绑定信息                 |
| 删除成员 + @成员/qq号       | （SU权限）删除绑定信息                               |

#### 4. [clanbattle_info](https://github.com/cc004/clanbattle_info)

与3功能近似但是更详细，暂时没有需求所以注释掉了，配置方法请参考上方链接

#### 5. [pcr_calculator_plus](https://github.com/watermellye/pcr_calculator_plus)

无需设置

##### 输入方法

###### 指令前缀

`合刀`, `cal`, `尾刀计算`。

###### 指令内容

- boss剩余血量和两刀伤害（可只输一刀或不输入）
- boss血量、造成伤害（缺省为boss血量）、打死时间、期望时间（可无）

时间输入后缀为s，范围[0,90)的正整数。

血量输入支持[|w|W|万]后缀；若无后缀，数值小于50000的将被10000；若输入表达式，将根据eval转换。

例：`cal 700w 400万 5000000` / `合刀 2000-87-49 2000 30s 61s`

#### 6. [Simple1kill2](https://github.com/CCA2878/Simple1kill2)

无需设置

##### 使用方法

群聊中输入：一穿二 剩余秒数 BOSS血量 目标补偿（非必需）。如：“一穿二 34 2000”或“一穿二 34 2000 56”。

#### 7. [pcr_almanac](https://github.com/azmiao/pcr_almanac)

无需设置

##### 功能

```
目前就一个命令：
[签到] 签到看黄历
```

#### 8. [tarot_hoshinot](https://github.com/haha114514/tarot_hoshino)

无需设置

##### 用法

群内发送 塔罗牌 即可使用。（改了登陆协议会导致不可用）

#### 9. [lifeRestart_bot](https://github.com/DaiShengSheng/lifeRestart_bot)

无需设置

##### 指令

| 命令                | 说明     | 例      |
| ------------------- | -------- | ------- |
| /remake 或 人生重来 | 触发指令 | /remake |

#### 10. [pcr_scrimmage](https://github.com/eggggi/pcr_scrimmage)

无需设置

##### 指令表

| 指令              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 大乱斗规则        | 查看大乱斗相关规则                                           |
| 大乱斗角色        | 查看所有可用角色                                             |
| 角色详情 [角色名] | 查看角色的基础属性和技能                                     |
| 结束大乱斗        | 可以强制结束正在进行的大乱斗游戏（该命令只有管理员和房主可用） |

> [ ] 表示内部的文字为意译, 实际发指令时请删去 [ ]

| 创建阶段指令 | 说明       |
| ------------ | ---------- |
| 创建大乱斗   | 创建大乱斗 |
| 加入大乱斗   | 加入大乱斗 |
| 开始大乱斗   | 开始大乱斗 |

| 选择角色阶段指令 | 说明                                   |
| ---------------- | -------------------------------------- |
| [角色名]         | 如：**凯露 / 黑猫** （名字和外号都行） |

> [ ] 表示内部的文字为意译, 实际发指令时请删去 [ ]

| 对战阶段指令   | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 丢色子         | 丢色子                                                       |
| [技能编号]@xxx | 如：1 @xxx (发送技能编号并@目标，如果这个技能不需要指定目标，直接发送技能编号即可) |
| 查看属性       | 可查看自己当前角色详细属性                                   |
| 投降 / 认输    | 投降 / 认输                                                  |

> [ ] 表示内部的文字为意译, 实际发指令时请删去 [ ]

##### 自定义

游戏内的跑道事件、角色、技能效果都可自定义，详情看 runway_case.py 和 role.py 的顶部注释

#### 11. [xcw](https://github.com/zangxx66/HoshinoBot-xcwRecord)

无需设置

##### 指令列表

@你的机器人 骂我	

仅支持群聊

#### 12. [buy_potion_reminder](https://github.com/pcrbot/HoshinoBuyPotionReminder)

无需设置

#### 13. [pcr_calendar](https://github.com/zyujs/pcr_calendar)

无需设置

##### 指令列表

日历 : 查看本群订阅服务器日历
[国台日]服日历 : 查看指定服务器日程
[国台日]服日历 on/off : 订阅/取消订阅指定服务器的日历推送
日历 time 时:分 : 设置日历推送时间
日历 status : 查看本群日历推送设置
日历 cardimage : (go-cqhttp限定)切换是否使用cardimage模式发送日历图片

## 4.报错归档

### NameError: name 'xxx' is not defined

在报错提示行加上（" "）或者（' '）

python版本问题

### ffmpeg: Executable not found on machine

将ffmpeg复制到python根目录下

检测机制的问题，执行时只会检测python根目录

### go-cqhttp网络环境报错

根据提示修改dns为114.114.114.114

```bash
vi /etc/resolv.conf

nameserver 114.114.114.114
```

### ImportError: cannot import name 'xxx' from 'markupsafe'

改用markupsafe2.0.1版本

新版本语法变化

### ModuleNotFoundError:No module named ‘xxx’

使用pip命令退回相关旧版本

新版本兼容问题

### Requirement already satisfied: pip in xxxxx/xxxx/

使用命令时额外添加`--target=报错路径`

未指定路径

### len out of range

使用旧版本go-cqhttp

新版go-cqhttp不适配



## 5.常用命令



```bash
# 去除代理环境变量
unset http_proxy 

# 查看后台
ps -def | grep "go-cqhttp"

# 结束进程
kill -9  进程号PID

# 编辑插件
vim /root/PCRChatBot/hoshino/config/__bot__.py

# 调试插件
python3.8 /root/PCRChatBot/run.py

# 使用代理运行
proxychains4 python3.8 run.py
```



