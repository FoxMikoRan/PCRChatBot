# 公会战数据查询及自动报刀插件

本插件改自[clanbattle_info](https://github.com/zyujs/clanbattle_info)，实现了app端协议

## 使用方法
1. 按原有项目配置好项目，cookie留空
2. **root**过的手机/模拟器打开bigfun登录b站并授权工会战信息获取
3. 打开/data/data/cn.bigfun/shared_prefs/BF_DATE.xml，将BF-Client-Data和device_number里面的内容复制到配置文件的对应值
4. 打开sqlite3数据库/data/data/cn.bigfun/databases/bf-db，找到表user中**access_key**字段，将值填入cookie中

# 以下为原项目简介

# 本项目已停止维护

现在打开毕方网页版团队战工具的boss报表页面, 你会在页面下方看到一行字:

>跨日刀会影响到统计数据的精准性，每日0点更新数据，实时数据请在bigfunAPP内查看

由于网页端出刀数据改为只在每日0时更新一次, 本项目已经没有实际使用价值, 所以就此停止维护更新.

请使用本项目的公会转为手动报刀或使用毕方客户端, 当然, 也可以忍受每天0点一口气上报全天的出刀数据.

万事万物总有始终, 感谢各位咸鱼公会的一路陪伴.

愿天堂没有公会战.

R.I.P.

# clanbattle_info

本项目为公主连结国服公会战信息管理插件,适用于HoshinoBot v2. 使用官方团队战工具 (https://www.bigfun.cn/tools/pcrteam/) 数据源.

本项目地址 https://github.com/zyujs/clanbattle_info

## 功能

- 公会战信息查询
- 出刀信息推送
- 自动报刀(目前仅支持yobot)

## 注意事项

本插件的功能和配置过程相对复杂, 不建议初级用户使用. 作者不提供任何针对本插件的使用指导, 如有使用上的疑问, 请自行阅读源代码.

由于cookie的安全性需要,本插件**仅限于bot维护者在自己的服务器中使用**,切勿在任何非本人完全可控的环境中使用本插件. 对于滥用导致的cookie泄露, 作者不承担任何责任. 同时由于上述原因, 请勿在共用/出租Bot中使用本插件.

官方团队战工具数据每5分钟更新一次, 因此本插件的报刀数据相对游戏内会存在数分钟的延迟, 在开启自动报刀的群内手动报刀前务必注意报刀数据是否有冲突, 手动报刀的伤害必须与游戏内完全一致, 以免插件无法同步记录.

## 安装方法

1. 在HoshinoBot的插件目录modules下clone本项目 `git clone https://github.com/zyujs/clanbattle_info.git`
1. 在 `config/__bot__.py`的模块列表里加入 `clanbattle_info`
1. 按照下一节说明创建群配置文件
1. 重启HoshinoBot

## 配置方法

请将项目根目录的 `config` 文件夹中 `template.json` 模板文件复制一份并重命名为 `群号.json`, 然后根据说明修改其中内容.

- `cookie` 使用游戏账号登录 [公主连结团队战工具](https://www.bigfun.cn/tools/pcrteam/) 生成的文本格式cookie, 需使用浏览器开发者工具获取
- `push_challenge` 出刀信息推送开关
- `report_mode` 自动出刀模式,取值可以为 `yobot_standalone` : 独立yobot模式, `yobot_embedded` : 嵌入式yobot模式, `yobot_plugin`：yobot作为hoshinobot的插件工作，`disable` : 关闭自动报刀
- `yobot_api` 如果使用 `yobot_standalone` 模式,需填入公会yobotAPI网址

## 自动报刀模式说明

### `yobot_standalone` 独立yobot模式

本模式应用场景: HoshinoBot和Yobot为两个独立的机器人, 它们各自拥有独立的QQ号码, 并且可能运行在不同的主机上. 在该模式下, 本插件使用群内消息报刀并使用yobotAPI同步出刀数据.

### `yobot_embedded` 嵌入式yobot模式

本模式应用场景: yobot使用插件模式运行在HoshinoBot内部,即缝合模式. 在该模式下, 本插件将调用yobot内部接口直接报刀并同步出刀数据. 该模式不需要设置yobot_api.

**注意**: 嵌入模式的yobot安装位置必须为标准路径, 即yobot的`__init__.py`文件位置为 `hoshino\modules\yobot\yobot\__init__.py`

### `disable` 关闭

关闭自动报刀功能

## 自动报刀功能使用流程

1. 依照配置方法小结配置群组设置
1. 在群内使用命令 `cbi 初始化`, 等待插件发送初始化完成信息.
1. 使用命令 `cbi 检查成员` 检查公会成员昵称是否能对应群内成员qq号码, 如果插件提示有会员找不到对应qq, 请~~杀掉~~使用命令 `cbi 绑定` 手动绑定.
1. 使用命令 `cbi 继续报刀` 开始自动报刀.
1. 出现异常时可以使用 `cbi 状态` 查询插件内部状态, 如有异常请重新初始化.

## 指令列表

本插件的全部命令都有前缀 `cbi`

- `cbi 帮助` : 获取帮助信息
- `cbi 总表` : 查询公会总表
- `cbi 日总表 [n]` : 查询公会日表, n表示会战第n天, 不带n参数默认为本日.
- `cbi 日出刀表 [n]` : 查询公会日表, n表示会战第n天, 不带n参数默认为本日.
- `cbi boss出刀表 [n]` : 查询boss出刀表, n为boss编号, 取值为1-5.
- `cbi 个人出刀表 昵称` : 查询指定成员的个人出刀记录.
- `cbi boss状态` : 查询boss状态
- `cbi 预约 n [@某人]` : 预约boss, n为boss编号, 取值为1-5.
- `cbi 取消预约 n [@某人]` : 取消预约boss
- `cbi 查看预约` : 查看boss预约情况
- `cbi 状态` : 查看插件当前状态
- `cbi 检查成员` : 检查公会成员是否全部有对应QQ号码.
- `cbi 绑定 游戏昵称 [@某人]` : 将指定游戏昵称绑定本qq号码或者指定qq号码, 适用于游戏昵称和群名片不匹配的情况.
- `cbi 绑定bot 游戏昵称` : 同上, 区别为将指定昵称绑定给bot自己的qq号码.
- `cbi 解除绑定 游戏昵称` : 解除绑定
- `cbi 查看绑定` : 查看当前的绑定列表
- `cbi 绑定未知成员 bot/@某人` : 将指定所有找不到对应qq的游戏昵称绑定给指定qq号码, 适用于部分成员不在群内的情况, bot表示将该昵称绑定为bot自己的qq号码,使用该功能必须参照下方常见问题解除yobot出刀限制.
- `cbi 解除绑定未知成员` : 禁用上述功能
- `cbi 继续报刀` : 继续自动报刀, 当自动报刀因错误暂停时, 请在排除错误后使用该命令继续报刀.
- `cbi 暂停报刀` : 暂停自动报刀
- `cbi 重置推送进度` : 重置推送进度, 下一次将从第一条出刀记录重新开始推送.
- `cbi 重置报刀进度` : 重置报刀进度, 从第一条出刀记录重新同步报刀进度, 适用于清空yobot记录后需要重新开始报刀的情况.
- `cbi 初始化` : 重新加载群数据, 修改config文件后可以使用本指令重新加载数据.
- `cbi 生成会战报告 游戏昵称` : (需要安装[clanbattle_report](https://github.com/zyujs/clanbattle_report)插件) 生成指定玩家的会战报告.
- `cbi 生成离职报告 游戏昵称` : 同上, 生成离职报告.

**注意**: 使用自动报刀的公会, 不能有重名的成员, 插件无法区分同名会员. 如果插件提示未知游戏昵称, 需要使用绑定指令为指定游戏昵称绑定成员qq. 本插件不限制每个qq号码绑定的游戏昵称数量, 但请注意yobot本身的每账号每天3刀的限制, 有特殊需求请自行修改Yobot相关代码.

## 常见问题

### 独立模式下,yobot不响应本插件的报刀指令

yobot默认不响应未加入公会的QQ号码的报刀信息, 需要使用yobot指令 `加入公会` 将hoshino机器人加入公会.

### 群内会员的名字与游戏昵称不一致

建议修改群名片与游戏昵称一致, 或使用指令 `cbi 绑定 游戏昵称` 为游戏昵称指定QQ号码.

### 有会员没有加群但还想使用自动报刀

可以使用 `cbi 绑定未知成员 bot` 指令将未加群的游戏昵称全部指定为bot所有, 并修改yobot代码解除每日3刀限制. 修改方法见 `yobot修改` 小结.

### 需要距离会战第一天超过2天甚至会战结束后重建全部报刀记录

首先修改yobot文件解除每天3刀限制, 然后在本插件群配置文件中加入隐藏选项 `"debug": true` 以关闭日期检查.

## yobot修改

为了配合本插件某些功能, 可能需要对yobot进行一些修改.

**请注意**: 修改yobot可能需要一定的编程基础, 修改前请备份原文件.

### 解除每天最多出3刀的限制

`yobot\src\client\ybplugins\clan_battle\battle.py` 第409行, 将 `if finished >= 3:` 中的数字3改为100或更大数字.

### 减小yobotAPI缓存时间以加快报刀速度

`yobot\src\client\ybplugins\clan_battle\battle.py` 第997行, 将 `@timed_cached_func(max_len=64, max_age_seconds=10, ignore_self=True)` 中的数字10改为1.

## 许可

本项目采用AGPLv3协议开源
