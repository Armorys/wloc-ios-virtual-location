# WLOC 虚拟定位 · Web 控制台（worker）

**简体中文** | [English](./README.en.md)

本目录是 WLOC 项目的 Web 控制台与地图链接解析 API：在地图上选好目标位置，一键写入代理工具，即可修改 iOS 定位，无需越狱。模块本体、订阅地址与完整教程见仓库根目录的 [README](../README.md)。

- 🌐 在线演示：https://wloc-pages.pages.dev/
- 📦 所属仓库：https://github.com/Armorys/wloc-ios-virtual-location（本目录 `worker/`）

## 功能

- 🗺️ **地图点选定位**：基于 Leaflet 的地图界面，点击地图即可选定目标位置
- 🎯 **扰动半径**：可设置偏移半径（米），每次定位在目标点周围随机偏移，`0` = 关闭
- ⭐ **收藏位置**：把常用地点收藏起来，随取随用
- 🔗 **粘贴地图链接解析**：支持 Apple Maps、Google Maps、高德、百度地图链接与纯坐标文本，短链接会自动展开
- 🔍 **地点搜索**：按名称搜索地点并快速定位
- 💾 **持久化写入**：选好位置后点击「储存到设备」，把目标坐标以 `wloc_settings` 形式写入代理工具，设备端持久保存

## 工作原理

控制台页面本身不直接改定位，它负责把你选定的目标坐标写成 `wloc_settings` 数据，供 iOS 代理工具（Surge、Quantumult X、Shadowrocket 等）中的 **WLOC 定位模块**读取；模块拦截 Apple 定位服务请求并返回虚拟坐标。

模块生效需要满足以下条件（控制台页面顶部也会给出提示）：

1. 已安装并启用 WLOC 定位模块
2. MITM 已开启且已信任证书
3. MITM 主机名包含 `gs-loc.apple.com`
4. 当前网络流量已走代理

## 部署

### Cloudflare Pages（推荐）

1. 在 Cloudflare Pages 中连接 Armorys/wloc-ios-virtual-location 仓库，项目根目录设置为 `worker`
2. 构建输出目录设置为 `dist`（也可直接使用本目录的 `wrangler.pages.jsonc`）
3. 部署后 `functions/[[route]].js` 会作为 Pages Functions 入口自动生效，控制台页面与 `/api/parse` 接口均可使用

线上实例即由 Cloudflare Pages 托管：https://wloc-pages.pages.dev/

### 本地开发

```bash
npm install
npm run pages:dev   # 本地预览 http://localhost:8787
```

### 独立 Worker 版

本目录的 `wloc-worker.js` 是独立的 Worker 版入口，可单独部署到 Cloudflare Workers（部署配置见同目录 `wrangler.jsonc`）：

```bash
npm run deploy
```

## API

### GET /api/parse

把地图链接或坐标文本解析为经纬度，主要供 iOS 快捷指令等场景调用。

**请求参数**

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| `u` | 是 | 地图链接（Apple Maps / Google Maps / 高德 / 百度，含短链）或 `纬度,经度` 纯文本 |
| `format` | 否 | `json` 时返回 JSON；省略时返回纯文本片段 |
| `cs` | 否 | `gcj` = 强制按 GCJ-02 转换；`none` = 强制不转换；省略 = 按链接来源自动判断 |

**坐标系规则**

- 高德、苹果地图（中国大陆）的坐标为 GCJ-02，默认自动转换为 WGS84 后返回（迭代反算，亚米级精度）
- 中国境外坐标不做偏移，原样返回
- 返回的经纬度保留 6 位小数

**返回示例（JSON）**

```text
GET /api/parse?u=31.230416,121.473701&format=json
```

```json
{ "lat": 31.230416, "lon": 121.473701, "name": "" }
```

其中 `name` 为链接中携带的地点名称（纯坐标输入时为空）。

**返回示例（纯文本，省略 `format=json`）**

```text
GET /api/parse?u=31.230416,121.473701
```

```text
lat=31.230416&lon=121.473701
```

解析失败时返回 HTTP 422 与 `{ "error": "..." }`。

## 项目结构

```text
worker/
├── src/
│   ├── index.js          # Hono 应用入口：GET / 控制台页面、GET /api/parse 解析接口
│   ├── page.js           # 控制台页面（Leaflet 地图、选点、收藏、搜索、写入设置）
│   └── parse.js          # 地图链接解析与 GCJ-02 ⇄ WGS84 坐标转换
├── functions/
│   └── [[route]].js      # Cloudflare Pages Functions 入口
├── dist/
│   └── _routes.json      # Pages 路由配置
├── wloc-worker.js        # 独立 Worker 版入口（单文件）
├── wrangler.jsonc        # Worker 部署配置
├── wrangler.pages.jsonc  # Pages 部署配置（输出目录 dist）
└── package.json
```

## 获取 WLOC 定位模块

本仓库只是 Web 控制台 + 链接解析 API。需要 WLOC 定位模块本体，请先私聊分发机器人 [@wlocios_bot](https://t.me/wlocios_bot) 登记，加入 Telegram 群组 **中文搜索大师**（[@souqings](https://t.me/souqings)）并**在群里发送三条随机消息**，再回到机器人发送 **wloc** 或 **定位模块** 领取。


## 相关项目

- WLOC 主项目（模块订阅、教程）：https://github.com/Armorys/wloc-ios-virtual-location
- WLOC 定位模块上游仓库：https://github.com/ios151/wloc-ios

## 免责声明

本项目仅供学习、开发测试与个人隐私保护使用。请遵守所在地法律法规及相关平台的服务条款，因使用本项目产生的一切后果由使用者自行承担。
