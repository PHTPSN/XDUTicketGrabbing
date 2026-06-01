# XDUTicketGrabbing

![Platform](https://img.shields.io/badge/Platform-Node%2Ejs-green)
![Deployment](https://img.shields.io/badge/Deployment-fly%2Eio-purple)
![Notification](https://img.shields.io/badge/Notification-Server%20酱-orange)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/PHTPSN/XDUTicketGrabbing)](https://github.com/PHTPSN/XDUTicketGrabbing/releases)

> ⚠️ 当前验证码平台的登录状态提示可能不准确
>
> 近期学校 IDS / 体育馆系统登录流程疑似存在响应延迟或状态返回变化。
> 因此，验证码平台在提交验证码后可能仍显示“登录失败”，但这并不一定代表实际登录失败。
>
> 根据近期测试，即使页面提示登录失败，后续抢票流程仍可能正常执行并成功预约。
>
> 当前版本的“登录成功判断”可能存在误判，建议以最终抢票结果或体育馆系统订单状态为准。
> 如果抢票失败，请再检查账号密码、验证码、部署日志以及是否能通过浏览器正常登录 IDS。
>
> 相关讨论见 [Issue #1](https://github.com/PHTPSN/XDUTicketGrabbing/issues/1)

**XDU 体育馆自动抢票助手 | Automated Ticket Grabber for Xidian University**

一个为西安电子科技大学学生开发的自动抢票工具，支持定时抢票、自动登录、微信通知和在线验证码平台。

**关键特性**: 自动登录 | 定时抢票 | 微信通知 | 验证码处理 | Node.js | 轻量服务器部署

**Keywords**: 抢票, 预约, 自动化, 体育馆, 西电, XDU, ticket grabbing, appointment, automation

众所周知，西电人的体育热情非常高，体育场馆席位常常在中午 12:00 放票的一瞬间就被一抢而空。即使你相信自己的手速，也无法保证每天都能记得抢票。本项目旨在自动化抢票流程，让你无需蹲点抢票，你只需要在收到微信里抢票成功通知后于 30 分钟内进入体育场馆预约系统完成支付即可。

本项目能维持校园账户登录，对于验证码的问题，项目会维持一个在线的验证码输入平台，如果登录信息过期，会通过微信提示你前往该平台输入验证码完成登录。

⚠️ **温馨提示**：本项目为个人学习研究用途开发，请合理使用，抢到票后请及时付款或取消订单，避免过度占用公共资源，影响其他同学正常抢票。

## 功能特性

- ✅ 自动登录校园统一认证系统
- ✅ 定时抢票，无需人工蹲点
- ✅ 微信通知抢票结果
- ✅ 在线验证码输入平台，方便手动处理验证码
- ✅ 支持多种体育场馆类型
- ✅ 可配置的抢票时间和参数

## 使用指南

本例在 **fly.io** 上部署我们的应用。fly.io 是一个云服务器提供商，提供免费额度，像我们这种轻量级的项目可以免费部署。本例使用 **Server 酱**来发送微信通知，一天有 5 次的免费额度，对于我们的项目也完全够用。

你也可以使用其他的云服务器和通知方式。前者不需要修改项目主体，但如果使用其他的通知方式，`/src/utils/wechat-notifier.js` 模块需要重写。

### 1. 配置

我们将通用配置放在 `config.json` 中，而将对安全性有要求，或是可能需要修改的配置放在环境变量中。

### 1.1 通用配置

在项目根目录下创建名为 `config.json` 的文件，内容如下，其中 `location` 项待定，之后应输入你在下一步部署的云服务器地址（如 `https://your-app-name-123.fly.dev`）：

```json
{
  "ticket": {
    "dateadd": 2,
    "TimePeriod": 2,
    "VenueNo": "01",
    "FieldTypeNo": "006",
    "targetTime": "12:00",
    "stopTime": "13:00",
    "stopTimeDateAdd": 0,
    "retryInterval": 2000
  },
  "notification": {
    "SendKey": null,
    "title": "抢票系统通知",
    "enable": false
  },
  "server": {
    "location": "" 
  }
}
```

#### 1.2 环境变量

在 fly.io 中，**当你完成部署后**，将以上变量输入到项目 Dashboard 的 Secrets 中即可，也可以使用命令行，如

```bash
fly secrets set NOTIFICATION_ENABLE=true
fly secrets set SEND_KEY=your_send_key_here
fly secrets set IDS_USERNAME=your_username
fly secrets set IDS_PASSWORD=your_password
```

注意所有的环境变量都不需要加引号。

必须设置的环境变量：

- `NOTIFICATION_ENABLE`：建议设置为`true`，用于 Server 酱发送微信通知
- `SEND_KEY`：Server 酱的凭证
- `IDS_USERNAME`：统一认证账号
- `IDS_PASSWORD`：统一认证密码

可选择设置的环境变量，如不设置为默认值：
- `STOP_TIME`：停止抢票时间，默认为 `13:00`
- `FIELD_TYPE_NO`：场馆类型，默认为 `006`，场馆对应编号如下

|场馆|编号|
|--|--|
|羽毛球（远望谷）|001|
|乒乓球（远望谷）|002|
|篮球（远望谷）|003|
|健身房（远望谷）|006|
|北校区羽毛球|007|
|北校区乒乓球|008|
|北校区篮球|009|

备注：场地类型信息可能会更新，可以在浏览器中进入体育馆系统页面，进入开发者模式，选择 Network 项，打开对应场地，找到开头是 `https://tybsouthgym.xidian.edu.cn/Field/GetVenueStateNew?` 的请求，其中 `FieldTypeNo` 后面的值就是场地编号。

考虑到晚上的票比较难抢，而周末白天的票比较容易抢到这一情况，目前仅支持提前两天抢晚上的票，如有需求，可联系作者增加功能。

### 2. 云服务器部署

#### 2.1 准备工作

1. 注册 [fly.io](https://fly.io/) 账号
1. 安装 flyctl 命令行工具：
   ```bash
   # macOS/Linux
   curl -L https://fly.io/install.sh | sh
   
   # Windows (使用 PowerShell)
   iwr https://fly.io/install.ps1 -useb | iex
   ```
1. 登录 fly.io：
   ```bash
   fly auth login
   ```

#### 2.2 创建应用

1. 在项目根目录下创建 `fly.toml` 文件，内容如下，其中 `app` 项是你自定义的应用名称，建议加入一些随机字符，避免重名的同时增加安全性：

    ```toml
    app = ''
    primary_region = 'iad'

    [build]
      dockerfile = 'Dockerfile'

    [env]
      NODE_ENV = 'production'

    [http_service]
      internal_port = 8080
      force_https = true
      auto_stop_machines = 'off'
      auto_start_machines = false

      min_machines_running = 1
      
      # 更宽松的健康检查
      [[http_service.checks]]
        interval = '30s'
        timeout = '5s'
        grace_period = '60s'
        method = 'GET'
        path = '/health'

    [[vm]]
      memory = '1gb'
      cpu_kind = 'shared'
      cpus = 1
    ```

1. 部署应用：
    ```bash
    fly launch
    ```
    按照提示完成部署，选择免费计划。

1. 在 Dashboard 中查看你的应用地址，或使用命令行：
    ```bash
    fly status
    ```

    更新 `config.json` 和环境变量（见[配置](#1-配置)）。

1. 启动服务，并将实例数量设置为 1：

    ```bash
    fly deploy
    fly scale count 1
    ```

### 3. 微信通知设置

1. 访问 [Server 酱官网](https://sct.ftqq.com/)
1. 根据提示完成微信登录、关注服务号和测试等操作
1. 将你的 `SendKey`（开头是 `SCT`）写入环境变量

### 4. 验证码平台使用

当登录信息过期时，系统会通过微信发送通知，提示你前往验证码输入平台。点击通知中的链接或直接访问你的应用地址，根据提示完成验证码输入即可。应用当前设置了在抢票前半小时检查登录状态，不过还是建议在部署完应用时就前往验证码平台完成首次登录。

### 5. 确认订单

通知提醒抢到票后，在体育馆系统中，打开“我的-场地订单”，支付或取消订单，注意 30 分钟的订单有效时间。

### 6. 监控和维护

查看日志：

```bash
fly logs
```

应用运行时会有心跳指示。


重启应用：

```bash
fly apps restart
```

修改代码后重新部署：

```bash
fly deploy
```

### 注意事项

- 登录时需要等待响应，请耐心等待 3 秒左右
- 登录偶尔不稳定，重新尝试即可
- 任何时候都可以重新登录
