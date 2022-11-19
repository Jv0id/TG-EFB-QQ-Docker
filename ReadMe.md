# TG-EFB-QQ-Docker

EFB 和 Go-CQHTTP 的 Docker Compose 部署方式

## 这是什么

使用 Bot 在 Telegram 及 QQ 间转发消息，基本可以做到去 QQ 化，仅在 Telegram 上与 QQ 好友/群组互动

本项目使用 Docker Compose 简化了 [Telegram Bot](https://github.com/ehForwarderBot/ehForwarderBot) 和 [QQ Bot](https://github.com/Mrs4s/go-cqhttp) 的安装与配置，仅需要 Docker Compose 与流畅的国际互联网连接即可使用

## 使用项目

- [TG-EFB-QQ-Docker](https://github.com/xzsk2/TG-EFB-QQ-Docker)

  - [EFB-Docker](https://github.com/xzsk2/EFB-Docker)
    - [EH Forwarder Bot](https://github.com/ehForwarderBot/ehForwarderBot)
    - [Telegram Master](https://github.com/ehForwarderBot/efb-telegram-master)
    - [QQ Slave](https://github.com/milkice233/efb-qq-slave)
    - [QQ Plugin Mirai](https://github.com/milkice233/efb-qq-plugin-mirai)
    - [QQ Plugin go-cqhttp](https://github.com/XYenon/efb-qq-plugin-go-cqhttp)
    - [efb-filter-middleware](https://github.com/xzsk2/efb-filter-middleware)

  - [Go-CQHttp-Docker](https://github.com/xzsk2/Go-CQHTTP-Docker)
    - [go-cqhttp](https://github.com/Mrs4s/go-cqhttp)

## 使用

### 克隆项目

```bash
# 克隆
git clone -b go-cqhttp https://github.com/xzsk2/TG-EFB-QQ-Docker.git
# 进入文件夹
cd TG-EFB-QQ-Docker
```

### 修改配置文件

可参考 [Telegram收发QQ信息-EFB和GO-CQHTTP的Docker部署教程](https://sakari.top/2021/11/15/tg-qq-gocq/) 及各项目的文档

1. 修改 `blueset.telegram/config.yaml` 内的 `token` 及 `admins`，如不能访问Telegram则需要在此配置代理

2. 编辑 `gocq/config.yml` 配置文件

    ```bash
    account:         # 账号相关
      uin: 000000000 # QQ 账号
      password: ''   # QQ 密码，为空时使用扫码登录
    ```

3. （可选）修改登陆协议，运行如下命令，待提示生成 `device.json` 后 `ctrl+c` 退出，编辑 `gocq/device.json`，参考 [设备信息](https://docs.go-cqhttp.org/guide/config.html#%E8%AE%BE%E5%A4%87%E4%BF%A1%E6%81%AF)

    ```bash
    docker run --rm -it --name="gocq" -v $PWD/gocq:/data xzsk2/gocqhttp-docker:latest
    ```

### 运行

```bash
docker-compose up -d
```

如需扫码登陆输入 `docker logs gocq` 查看二维码

### 停止

```bash
docker-compose down
```

### 自动更新

```
docker run -d \
    --name watchtower \
    --restart unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower -c \
    --interval 3600 \
    efb gocq
```

若Docker镜像未及时跟进上游项目更新，欢迎提交 Issue

## 其他问题

### 所有消息挤在一个对话如何分开

参考 [绑定会话](https://github.com/ehForwarderBot/efb-telegram-master/blob/master/readme_translations/zh_CN.rst#link%E7%BB%91%E5%AE%9A%E4%BC%9A%E8%AF%9D)

### VPS上登陆异常

`当前设备网络不稳定或处于复杂网络环境,为了你的帐号安全,建议将两个设备连接同一网络或将被扫描设备连接你的手机热点后,重新扫码登录。`

在本地直接下载使用 [go-cqhttp](https://github.com/Mrs4s/go-cqhttp/releases/tag/v1.0.0-rc3) 配置运行，登陆成功后将生成的 `device.json` 上传到远程服务器对应位置重新登陆

### ⚠️配置文件更新相关⚠️

[go-cqhttp](https://github.com/Mrs4s/go-cqhttp) 在 [v1.0.0-rc2 ](https://github.com/Mrs4s/go-cqhttp/releases/tag/v1.0.0-rc2) 修改了部分配置

- [f63c59f](https://github.com/Mrs4s/go-cqhttp/commit/f63c59f1a40a4210bea6a59b10a658257722a2e0) `HTTP和正向WS使用了新配置文件格式(保留了对老版本的兼容)`

新版配置文件兼容旧版，但为了避免旧配置文件兼容的废弃，可以参考 [95f3890](https://github.com/xzsk2/TG-EFB-QQ-Docker/commit/95f3890f8a23f65365a089d7425dca4e3a5ca8aa) 更新配置文件

<details>
  <summary>2021/11/21</summary>
  
>如果你在 2021/11/21 前即 [9a84c3f](https://github.com/xzsk2/TG-EFB-QQ-Docker/commit/9a84c3f5850366c642cd3801e34068b182562b07) 前拉取过本项目且正在使用，请进行如下配置文件的修改，否则 go-cqhttp 更新后将无法正常使用
>
>如果你未使用或正准备本项目请略过本段，目前的仓库已经应用了新的配置文件，你可以直接使用本项目即可

[go-cqhttp](https://github.com/Mrs4s/go-cqhttp) 从 [v1.0.0-beta8](https://github.com/Mrs4s/go-cqhttp/releases/tag/v1.0.0-beta8) 开始修改了部分配置文件，请检查你的 `gocq/config.yml` 文件的最后一段是否为

```
      post:
      #- url: '' # 地址
      #  secret: ''           # 密钥
      - url: 127.0.0.1:8000 # 地址
        secret: ''          # 密钥
```

修改第四行的url如下

```
      post:
      #- url: '' # 地址
      #  secret: ''           # 密钥
      - url: http://127.0.0.1:8000/ # 地址
        secret: ''          # 密钥
```

完整的配置文件参考 [config.yml](gocq/config.yml) 或 [9a84c3f](https://github.com/xzsk2/TG-EFB-QQ-Docker/commit/9a84c3f5850366c642cd3801e34068b182562b07)
</details>
