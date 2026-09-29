# istoreos-apk

A custom APK package feed for iStoreOS / OpenWrt 25.12+ (apk-tools v3, ADB index format), built and published automatically by GitHub Actions.

自建 APK 软件源，面向 iStoreOS / OpenWrt 25.12+（apk-tools v3）。每天由 GitHub Actions 自动从各上游 release 抓取最新包、重建签名索引并发布到 GitHub Pages。

## 提供哪些包

| 包 | 来源 |
|---|---|
| `luci-app-passwall` / `luci-i18n-passwall-zh-cn` | Openwrt-Passwall 官方 release |
| `smartdns` / `luci-app-smartdns` | pymumu 上游 release |
| `sing-box`、`xray-core`、`chinadns-ng`、`geoview`、`hysteria`、`dns2socks`、`tcping`、`v2ray-geoip`、`v2ray-geosite` | CloudRun 离线包解包 |
| `luci-i18n-adguardhome-zh-cn` | OpenWrt SNAPSHOT 源 |

## 工作方式

- 定时任务（cron `18 22 * * *` UTC）抓取各上游最新 release；构建逻辑见 `.github/workflows/build-apk-feed.yml`
- 文件名统一规整为 `名称-版本.apk`（与索引元数据严格一致，避免 fetch 404）
- 同名包只保留版本最高的一个（多渠道抓取时版本可能短暂错位）
- `apk mkndx --sign-key` 生成签名索引 `packages.adb`（EC P-256），本仓库根目录的 `public-key.pem` 为对应公钥
- `SHA256SUMS` 提供全部文件的校验和

## 路由器接入（iStoreOS 25.12+）

```sh
# 1. 信任本源公钥
#    ⚠️ 固件已在 /etc/apk/keys/ 放了名为 public-key.pem 的固件构建密钥，切勿覆盖
wget -O /etc/apk/keys/myfeed.pem https://apple4105.github.io/istoreos-apk/public-key.pem

# 2. 添加源（带 @custom 标记，便于钉包）
echo '@custom https://apple4105.github.io/istoreos-apk/packages.adb' \
  >> /etc/apk/repositories.d/customfeeds.list

# 3. 刷新索引并安装
apk update
apk add luci-app-passwall@custom luci-i18n-passwall-zh-cn@custom
apk add smartdns@custom luci-app-smartdns@custom sing-box@custom
```

### 为什么建议用 `@custom`

本源与官方源存在同名包（如 smartdns），且两边版本方案不同——官方 feed 用 `46.x` 这类 release 序号，上游预编译包用 `1.20xx.xx.xx-rXXXX` 日期式版本号。apk 按数字段逐位比较版本，会把官方源的旧包判为"更新"。给源行打上 `@custom` 标记后，`apk add <pkg>@custom` 即可把这些包钉到本源，不被官方源顶掉。

## 说明

- 所有包均从上游 release 原样转发/解包，内容未做修改；本仓库与上游项目无隶属关系。
- 索引每天重建，只保留各包最新版本，不归档旧版。
- 仅适用于 apk-tools v3（OpenWrt 24.10 及更早版本使用 opkg，不兼容本源）。
