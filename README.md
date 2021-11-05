<div align="center">
<h1 align="center">
BILIHELPER
</h1>

</div>



# 目录

- [目录](#目录)
  - [使用说明](#使用说明)
    - [获取运行所需的 Cookies](#获取运行所需的-cookies)
    - [一、使用 腾讯云函数](#一使用-腾讯云函数)
    - [二、使用 Docker](#二使用-docker)
    - [三、使用 Linux Crontab 方式](#三使用-linux-crontab-方式)
    - [自定义功能配置](#自定义功能配置)
  - [订阅执行结果](#订阅执行结果)
    - [Server 酱 Turbo 版](#server-酱-turbo-版)
    - [Telegram 订阅执行结果](#telegram-订阅执行结果)
    - [钉钉机器人](#钉钉机器人)
    - [PushPlus(Push+)](#pushpluspush)
  - [更新和帮助](#更新和帮助)
    - [使用 Pull APP［推荐］](#使用-pull-app推荐)
  - [免责声明](#免责声明)
  - [API 参考列表](#api-参考列表)
  - [基于本项目的衍生项目](#基于本项目的衍生项目)
  - [致谢](#致谢)
  - [License](#license)
  

## 使用说明

### 获取运行所需的 Cookies

1. **Fork 本项目**
2. **获取 Bilibili Cookies**
3. 浏览器打开并登录 [bilibili 网站]()
4. 按 F12 打开 「开发者工具」 找到 应用程序/Application -\> 存储 -\> Cookies
5. 找到 `bili_jct` `SESSDATA` `DEDEUSERID` 三项，并复制值，后面需要用到。

![图示](docs/IMG/20201012001307.png)


### 一、使用 腾讯云函数

请参考[腾讯云函数部署](https://github.com/JunzhouLiu/BILIBILI-HELPER/blob/main/docs/scf.md)

### 二、使用 Docker

请自行参阅 [Issues/75#issuecomment-731705657][28] 和[基于本项目的衍生项目](#基于本项目的衍生项目)。

[28]: https://github.com/JunzhouLiu/BILIBILI-HELPER/issues/75#issuecomment-731705657

### 三、使用 Linux Crontab 方式

1. 在 linux shell 环境执行以下命令，并按照提示输入 SESSDATA，DEDEUSERID，BILI_JCT，SCKEY 四个参数

```
wget https://raw.githubusercontent.com/JunzhouLiu/BILIBILI-HELPER/main/setup.sh && chmod +x ./setup.sh && sudo ./setup.sh
```

**ps：注意，如果使用自定义配置，请将`config.json`和 jar 包放置在同一目录(使用 setup.sh 安装则需要将`config.json`放置到`{HOME}/BILIBILI-HELPER`)，`v1.2.2`
之后的版本`release`中都会携带一份`config.json`。**

2. 除此之外，也可以通过点击 [BILIBILI-HELPER/release][30]，下载已发布的版本，解压后将 jar 包手动上传到 Linux 服务器，使用 crontab 完成定时执行，如果使用`crontab`
   请记得`source /etc/profile`和`source ~/.bashrc`,建议直接使用仓库提供的[`start.sh`][31]脚本,注意修改脚本的 jar 包路径和 cookies 参数。

[30]: https://github.com/JunzhouLiu/BILIBILI-HELPER/releases/latest
[31]: https://github.com/JunzhouLiu/BILIBILI-HELPER/blob/main/start.sh

**crontab 命令示例**

`30 10 * * * sh /home/start.sh`

| args              | 说明               |
| ----------------- | ------------------ |
| 30 10 \* \* \*    | `crontab` 定时时间 |
| sh /home/start.sh | `start.sh`的路径   |

```shell
#!/bin/bash
source /etc/profile
source ~/.bashrc
source ~/.zshrc #其他终端请自行引入环境变量
echo $PATH
java -jar /home/BILIBILI-HELPER.jar DEDEUSERID SESSDATA BILI_JCT SCKEY >> /var/log/bilibili-help.log
# 注意将jar包路径替换为实际路径。将参数修改该你自己的参数，cookies中含有等特殊字符需要转义。
```

**命令示例：**

```shell
# *如果Cookies参数中包含特殊字符，例如`%`请使用`\`转义*,如果不执行可在命令前增加 source /etc/profile
# m h  dom mon dow   command
30 10 * * * java -jar /home/BILIBILI-HELP.jar DEDEUSERID SESSDATA BILI_JCT >/var/log/cron.log &
```

### 自定义功能配置

配置文件示例：

```json
{
  "numberOfCoins": 5,
  "reserveCoins": 50,
  "selectLike": 0,
  "monthEndAutoCharge": true,
  "giveGift": true,
  "upLive": "0",
  "chargeForLove": "0",
  "devicePlatform": "ios",
  "coinAddPriority": 1,
  "skipDailyTask": true,
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0 Safari/605.1.15"
}
```

**Windows/Linux 用户使用 jar 包时，`release`包中会包含一份`config.json`配置文件，只需将其和`BILIBILI-HELP.jar`放在同一目录即可，执行时优先加载外部配置文件**

配置文件参数示意

| Key                | Value                | 说明                                                                     |
| ------------------ | -------------------- | ------------------------------------------------------------------------ |
| numberOfCoins      | [0,5]                | 每日投币数量,默认 5 ,为 0 时则不投币                                     |
| reserveCoins       | [0,4000]             | 预留的硬币数，当硬币余额小于这个值时，不会进行投币任务，默认值为 50      |
| selectLike         | [0,1]                | 投币时是否点赞，默认 0, 0：否 1：是                                      |
| monthEndAutoCharge | [false,true]         | 年度大会员月底是否用 B 币券给自己充电，默认 `true`，即充电对象是你本人。 |
| giveGift           | [false,true]         | 直播送出即将过期的礼物，默认开启，如需关闭请改为 false                   |
| upLive             | [0,送礼 up 主的 uid] | 直播送出即将过期的礼物，指定 up 主，为 0 时则随随机选取一个 up 主        |
| chargeForLove      | [0,充电对象的 uid]   | 给指定 up 主充电，值为 0 或者充电对象的 uid，默认为 0，即给自己充电。    |
| devicePlatform     | [ios,android]        | 手机端漫画签到时的平台，建议选择你设备的平台 ，默认 `ios`                |
| coinAddPriority    | [0,1]                | 0：优先给热榜视频投币，1：优先给关注的 up 投币                           |
| userAgent          | 浏览器 UA            | 用户可根据部署平台配置，可根据 userAgent 参数列表自由选取                |
| skipDailyTask      | [false,true]         | 是否跳过每日任务，默认`true`,如果关闭跳过每日任务，请改为`false`         |



userAgent 可选参数列表

| 平台      | 浏览器         | userAgent                                                                                                                           |
| --------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Windows10 | EDGE(chromium) | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36 Edg/86.0.622.69 |
| Windows10 | Chrome         | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36                 |
| masOS     | safari         | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0 Safari/605.1.15               |
| macOS     | Firefox        | Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:65.0) Gecko/20100101 Firefox/65.0                                                  |
| macOS     | Chrome         | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36            |

_ps：如果尝试给关注的 up 投币十次后（保不准你关注的是年更 up 主），还没完成每日投币任务，则切换成热榜模式，给热榜视频投币_

_投币数量代码做了处理，如果本日投币不能获得经验了，则不会投币，每天只投能获得经验的硬币。假设你设置每日投币 3 个，早上 7 点你自己投了 2 个硬币，则十点半时，程序只会投 1 个）_
tips:从1.4.1版本开始，随机视频投币有一定的概率会将硬币投给本项目的核心开发者，算是对核心开发者长期以来维护的回馈。
## 订阅执行结果

### Server 酱 Turbo 版

目前 Turbo 版本的消息通道支持以下渠道

- 企业微信应用消息
- Android，
- Bark iOS，
- 企业微信群机器人
- 钉钉群机器人
- 飞书群机器人
- 自定义微信测试号
- 方糖服务号

1. 前往 [sct.ftqq.com](https://sct.ftqq.com/sendkey)点击登入，创建账号。
2. 点击点[SendKey](https://sct.ftqq.com/sendkey) ，生成一个 Key。将其增加到 Github Secrets 中，变量名为 `SERVERPUSHKEY`
3. [配置消息通道](https://sct.ftqq.com/forward) ，选择方糖服务号，保存即可。

**
旧版推送渠道[sc.ftqq.com](http://sc.ftqq.com/9.version0) 即将与 4 月底下线，请前往[sct.ftqq.com](https://sct.ftqq.com/sendkey)生成`Turbo`版本的`Key`，注意，申请 Turbo 版 Key 后请配置消息通道，如果想沿用以前的推送方式，选择方糖服务号即可**

### Telegram 订阅执行结果

1.在 Telegram 中添加 BotFather 这个账号，然后依次发送/start /newbot 按照提示即可创建一个新的机器人。记下来给你生成的 token。

2.搜索刚刚创建的机器人的名字，并给它发送一条消息。

_特别注意：需要先与机器人之间创建会话，机器人才能下发消息，否则机器人无法主动发送消息，切记！_

3.在 Telegram 中搜索 userinfobot，并给它发送一条消息，它会返回给你 chatid。

4.在 Github Secrets 中删除 SERVERPUSHKEY，添加 TELEGRAMBOTTOKEN，TELEGRAMCHATID。

### 钉钉机器人

1.首先你得有个钉钉企业 [快速注册](https://oa.dingtalk.com/register.html)

2.[进入钉钉开放平台添加机器人](https://open-dev.dingtalk.com/#/corprobot)

3.添加自定义关键词：BILIBILI

4.在 Github Secrets 中的 SERVERPUSHKEY 中更新成机器人的 Webhook

例如:`https://oapi.dingtalk.com/robot/send?access_token=XXX`

5.完成

### PushPlus(Push+)

1.[前往 PushPlus 获取 Token](https://www.pushplus.plus/push1.html)

2.在 Github Secrets 中的 SERVERPUSHKEY 中更新成获取到的 Token

3.完成

## 更新和帮助

### 使用 Pull APP［推荐］

参阅 [Pull APP](https://github.com/apps/pull)


## 免责声明

1. 本工具源码仅在[JunzhouLiu/BILIBILI-HELPER](https://github.com/JunzhouLiu/BILIBILI-HELPER)开源。
2. 本项目遵守[MIT License](https://github.com/JunzhouLiu/BILIBILI-HELPER/blob/main/LICENSE)，请各位知悉。

**仓库地址：[JunzhouLiu/BILIBILI-HELPER](https://github.com/JunzhouLiu/BILIBILI-HELPER)**

**仓库地址：[JunzhouLiu/bilibili-match-prediction](https://github.com/JunzhouLiu/bilibili-match-prediction)**

## API 参考列表

- [SocialSisterYi/bilibili-API-collect](https://github.com/SocialSisterYi/bilibili-API-collect)
- [happy888888/BiliExp](https://github.com/happy888888/BiliExp)

## 基于本项目的衍生项目

- **基于本项目的 docker 封装项目：[SuperNG6/docker-bilibili-helper](https://github.com/SuperNG6/docker-bilibili-helper)**

- **基于本项目的 docker 镜像：[superng6/bilibili-helper](https://hub.docker.com/r/superng6/bilibili-helper)**

- **基于本项目的 runer 项目：[KurenaiRyu/bilibili-helper-runer](https://github.com/KurenaiRyu/bilibili-helper-runer)**

- **基于本项目的 k8s 项目：[yangyang0507/k8s-bilibili-helper](https://github.com/yangyang0507/k8s-bilibili-helper)**

## 致谢

感谢 JetBrains 对本项目的支持。

[![JetBrains](docs/IMG/jetbrains.svg)](https://www.jetbrains.com/?from=BILIBILI-HELPER)

## License

[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FJunzhouLiu%2FBILIBILI-HELPER.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2FJunzhouLiu%2FBILIBILI-HELPER?ref=badge_large)
