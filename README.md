<p align="center">
  <img src="icon.svg" width="160" />
</p>

# WLOC 虚拟定位：免越狱修改 iOS 网络定位

**简体中文** | [English](./README.en.md)

![iOS](https://img.shields.io/badge/iOS-Virtual%20Location-blue)
![No Jailbreak](https://img.shields.io/badge/No-Jailbreak-green)
![Surge](https://img.shields.io/badge/Surge-Module-orange)
![Quantumult X](https://img.shields.io/badge/Quantumult%20X-Module-orange)
![Loon](https://img.shields.io/badge/Loon-Plugin-orange)
![Stash](https://img.shields.io/badge/Stash-Override-orange)
![Shadowrocket](https://img.shields.io/badge/Shadowrocket-Module-orange)

WLOC 虚拟定位通过改写 Apple 网络定位服务（WLOC，即 WiFi / 基站定位）返回的坐标，实现 **iOS 免越狱改定位**：无需手动填写经纬度，在在线选点页面点一下地图，或在地图 App 里分享一个位置，就能切换定位。本仓库提供 Surge、Quantumult X、Loon、Stash、Shadowrocket 五个平台的现成模块、两支 MITM 脚本，以及配套的在线选点页面与地图链接解析服务。

## 目录

- [快速开始](#快速开始)
- [获取模块与交流](#获取模块与交流)
- [订阅地址](#订阅地址)
- [快捷指令（最推荐）](#快捷指令最推荐)
- [在线选点页面](#在线选点页面)
- [地图链接解析与隐私](#地图链接解析与隐私)
- [自部署选点页面](#自部署选点页面)
- [工作原理](#工作原理)
- [参数配置](#参数配置)
- [iOS 26 / 27 必看：定位缓存与重启](#ios-26--27-必看定位缓存与重启)
- [收藏位置](#收藏位置)
- [恢复真实定位](#恢复真实定位)
- [注意事项](#注意事项)
- [常见问题 FAQ](#常见问题-faq)
- [致谢](#致谢)
- [免责声明](#免责声明)

## 快速开始

三条路径，任选其一：

| 路径 | 适合人群 | 入口 |
| --- | --- | --- |
| ① 订阅模块 | 所有用户（必备前提） | [订阅地址](#订阅地址) |
| ② 快捷指令 | 想一键切换 / 恢复定位 | [快捷指令（最推荐）](#快捷指令最推荐) |
| ③ 在线选点 | 想在地图上慢慢挑位置 | https://wloc-pages.pages.dev/ |

无论走哪条路径，都需要先完成同一组前提：**代理已开启、模块已启用、MITM 已开启并信任证书、MITM 主机名包含 `gs-loc.apple.com`**。

## 获取模块与交流

> [!IMPORTANT]
> **获取模块与订阅地址的唯一方式：**
>
> 1. 加入 Telegram 群组 **中文搜索大师**（[@Sousuozh_cn](https://t.me/Sousuozh_cn)）
> 2. 在群里发送 **三条随机消息**（任意内容，共三条）
> 3. 再发送「**看简介**」三个字
> 4. 按回复提示即可获取模块与订阅地址
>
> 本仓库不直接提供模块下载与订阅链接；未加群、未发送三条随机消息、或未发送「看简介」，都无法获取，请勿在 Issues 里索取。

使用上的问题也可以在群里交流提问。

## 订阅地址

订阅地址不在此处公开提供。需要订阅地址，请加入 Telegram 群组 **中文搜索大师**（[@Sousuozh_cn](https://t.me/Sousuozh_cn)），先发送三条随机消息、再发送「看简介」获取，详见[获取模块与交流](#获取模块与交流)。

两点说明：

- **Stash** 用户请使用 `.stoverride` 覆写版，无需用 Script Hub 转换
- **Egern** 可直接使用 Surge 版模块

## 快捷指令（最推荐）

装好模块后，最方便的用法是 iOS 快捷指令：不用打开选点页面，在地图 App 里分享位置即可切换，清除也是一键完成。

- **wloc 设置地理位置**：https://www.icloud.com/shortcuts/a82717d8fdad4e6280866fcf911173f7
- **wloc 清理恢复位置**：https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f

**设置位置**

1. 在地图 App 里选好位置（长按地图选点）
2. 点「共享 / 分享」→ 选择「wloc 设置地理位置」
   - 苹果地图：选点 → 共享 →「wloc 设置地理位置」
   - 高德地图：选点 → 分享 → **更多** →「wloc 设置地理位置」

**恢复位置**：运行「wloc 清理恢复位置」即可回到真实定位。

目前快捷指令支持苹果地图与高德地图（含短链，会自动跟随跳转并做 GCJ-02→WGS84 坐标换算，原理见[地图链接解析与隐私](#地图链接解析与隐私)）。更完整的图文步骤见 [快捷指令使用指南](docs/shortcut-guide.md)。选点页面（Worker / Pages）方案同样保留，见下一节。

## 在线选点页面

公共选点页面：https://wloc-pages.pages.dev/ （建议添加到主屏幕，当 App 一样用）

使用步骤：

1. 订阅上方模块，并启用 MITM
2. 打开在线选点页面
3. 在地图上选位置 / 搜索地名 / 粘贴地图链接（支持 Apple Maps、Google Maps、高德、百度链接与纯坐标文本）
4. 点击「储存到设备」
5. 下次 Apple 定位被触发时，新坐标自动生效

选好位置后还可以一键收藏，方便在多个地点之间来回切换，见[收藏位置](#收藏位置)。

## 地图链接解析与隐私

快捷指令在处理地图链接时，会先把链接发给解析接口 `wloc-spoofer.wloc.workers.dev/api/parse`，由 worker 统一处理。原因是两类链接各有难点：

- **高德地图**：分享出来的是短链，真实坐标藏在 302 跳转响应的 `Location` 头里，而且是 GCJ-02 偏移坐标。快捷指令既读不到跳转头，也难以自己做坐标换算，所以交给 worker：跟随跳转 → 提取坐标 → GCJ-02 转 WGS84 → 返回经纬度。
- **苹果地图**：链接里直接带 `coordinate=纬度,经度`，但中国大陆的坐标同样是 GCJ-02 偏移，需要像高德一样先转换再返回；境外坐标会自动跳过换算（`out_of_china` 判断），原样返回。

统一走一个接口，还顺带处理了短链展开、文本链接、地点名称解码等杂活。

**隐私说明**：`/api/parse` 是纯转发解析——收到链接 → 跟随跳转 → 解析坐标 → 返回 JSON，全程不写入任何储存、不记录日志、不做缓存，处理完即丢弃。

解析逻辑与路由都在本仓库开源（[`worker/src/parse.js`](worker/src/parse.js)、[`worker/src/index.js`](worker/src/index.js)），不放心公共接口的话，可以按下一节自部署一份，再把快捷指令里的 `wloc-spoofer.wloc.workers.dev` 换成你自己的 worker 域名即可。

## 自部署选点页面

公共选点页面有请求上限，长期使用建议部署自己的实例。两个现成入口功能完全一致：

- **Workers**：https://wloc-spoofer.wloc.workers.dev/
- **Pages**：https://wloc-pages.pages.dev/

**一键部署（仅 Workers 模式）**

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/Armorys/wloc-ios-virtual-location/tree/main/worker)

点击按钮后按提示授权即可完成部署。

**手动部署（Workers）**

```bash
# 1. 克隆仓库
git clone https://github.com/Armorys/wloc-ios-virtual-location.git
cd wloc/worker

# 2. 安装依赖
npm install

# 3. 登录 Cloudflare（首次需要）
npx wrangler login

# 4. 部署
npm run deploy
```

部署成功后会得到你自己的 Worker 地址（如 `https://wloc-spoofer.<你的子域名>.workers.dev`），用这个地址打开选点页面即可。Cloudflare 免费账户每天 10 万次请求，个人使用完全够用。

<details>
<summary>高级：Pages 部署</summary>

Pages 部署不支持一键按钮，需要手动执行：

```bash
git clone https://github.com/Armorys/wloc-ios-virtual-location.git
cd wloc/worker
npm install
npx wrangler pages deploy dist --project-name <自定义项目名>
```

部署时会提示设置 production branch，输入 `main` 即可。部署成功后得到 `https://<项目名>.pages.dev` 地址。Pages 与 Workers 功能完全一致，按需选择即可。

</details>

## 工作原理

```text
选点页面把目标坐标发给 gs-loc.apple.com/wloc-settings/save
  → 代理模块拦截该请求，wloc-settings.js 把坐标写进 $persistentStore
  → 下一次 WLOC 定位请求触发时，wloc.js 读出坐标
  → 解析 /clls/wloc 的 protobuf 响应并替换其中的坐标后返回给系统
```

模块由两条脚本规则组成，都在 `dist/` 目录：

- **`wloc.js`**：拦截 `/clls/wloc` 响应，解析 protobuf 并替换坐标
- **`wloc-settings.js`**：拦截 `/wloc-settings/save` 请求，把选点页面提交的坐标写入持久化储存

也就是说，本方案只改写网络定位（WiFi / 基站）的结果，不碰 GPS 硬件定位。

## 参数配置

| 参数 | 说明 | 默认值 |
| --- | --- | --- |
| longitude | 目标经度（在线选点优先） | null（透传） |
| latitude | 目标纬度（在线选点优先） | null（透传） |
| accuracy | 精度（米） | 25 |
| logLevel | 日志级别 | info |

优先级：**在线选点储存 > 模块参数 > 默认值**。

## iOS 26 / 27 必看：定位缓存与重启

Apple 从 iOS 26 开始大幅强化了 `locationd` 的定位缓存：系统会把之前获取到的真实定位结果缓存在内存中并长时间复用。结果是——哪怕脚本已经成功改写了 WLOC 响应（日志显示「已修改」），系统仍可能继续用缓存里的旧坐标，看起来就像定位没变。

**解决方法：重启设备。** 重启会清空 `locationd` 的内存缓存，系统重新发起 WLOC 请求时就能拿到修改后的坐标。在 iOS 26+ 上，开飞行模式、关闭定位服务等方式都**无法**清除此缓存，必须重启；iOS 15~18 通常不需要重启即可生效。

**高版本系统推荐流程（成功率最高，二选一）**

方法一：

1. 先在选点页面选好目标位置并储存到设备
2. 开飞行模式 → 关闭定位服务 → 重启设备
3. 关闭飞行模式（WiFi 也要关）→ 连接代理工具（确认 VPN 图标出现）→ 打开定位服务
4. 打开地图验证

方法二：

1. 关闭定位服务
2. 在选点页面选好位置并储存到设备
3. 打开定位服务 → 弹出「允许访问位置信息」时选择**「下次询问或在我共享时」**
4. 打开地图验证

## 收藏位置

在线选点页面支持收藏多个位置，方便在几个常用地点之间来回切换：

- **添加收藏**：选好位置后点「收藏位置」，输入备注名称（支持中文 / 英文 / 数字，最多 30 字）保存
- **快速切换**：点收藏列表中的位置，地图自动跳转，再点「储存到设备」即可切换
- **当前生效标记**：与设备已保存坐标一致的收藏会显示「✓ 当前生效」
- **删除管理**：可单个删除（× 按钮）或清空全部
- **当前生效坐标**：页面会显示设备端持久化数据（`wloc_settings`），支持刷新查询和清除

**两种储存不要混淆**：

| 数据 | 存放在哪 | 说明 |
| --- | --- | --- |
| 收藏列表 | 浏览器 `localStorage` | 只是选点页面的 UI 辅助数据；清浏览器缓存或换浏览器后需重新收藏 |
| 生效坐标 | 代理工具的 `$persistentStore` | 脚本运行时实际读取的数据；不受浏览器缓存影响 |

## 恢复真实定位

**方法一：关闭或删除模块（推荐）**

模块关闭后，脚本不再拦截 WLOC 请求，系统自动恢复真实定位。iOS 26+ 另需重启设备清除定位缓存。

**方法二：清除持久化数据，进入透传模式**

清掉已保存的坐标后，脚本进入**透传模式**：不修改 WLOC 响应、原样放行，系统随之恢复真实 GPS 定位。

透传模式的触发条件是两个同时成立：**持久化数据为空（null），且模块参数保持默认值（113.94114, 22.544577）**。此时脚本判定用户没有自定义坐标，自动跳过修改。模块参数不用改，只清持久化数据即可。

在代理工具中删除名为 `wloc_settings` 的持久化数据：

- **Surge** — 在脚本编辑器运行：`$persistentStore.write(null, "wloc_settings")`
- **Quantumult X** — 运行：`$prefs.removeValueForKey("wloc_settings")`
- **Loon** — 运行：`$persistentStore.write(null, "wloc_settings")`

清除后重启设备即可恢复真实定位，无需关闭模块。

> 注意：如果你在模块参数里手动改过经纬度（不是默认的 113.94114, 22.544577），即使清除了持久化数据，脚本仍会按模块参数的坐标继续修改。只有参数保持默认时，清除持久化数据才会进入透传模式。

## 注意事项

- MITM 证书需要信任 `gs-loc.apple.com` 和 `gs-loc-cn.apple.com` 两个主机名
- 本方案仅修改网络定位（WiFi / 基站），不影响 GPS 硬件定位
- GPS 信号强时，iOS 可能忽略网络定位结果
- 以 WiFi 定位为主的室内场景效果最佳
- 选点页面需在代理模式下使用（Safari 流量走代理，才能拦截到储存请求）

## 常见问题 FAQ

**Q：需要越狱吗？**
A：不需要。整套方案基于代理工具的 MITM 与脚本能力，在普通 iOS 设备上即可使用。

**Q：支持哪些代理工具？**
A：Surge、Quantumult X、Loon、Stash、Shadowrocket 都有现成模块（见[订阅地址](#订阅地址)），Egern 可直接用 Surge 模块。

**Q：明明设置了，定位却没变化怎么办？**
A：先确认代理已连接、模块已启用、证书已信任且主机名包含 `gs-loc.apple.com`。如果你是 iOS 26 或更高版本，大概率是 `locationd` 缓存问题，按[iOS 26 / 27 必看](#ios-26--27-必看定位缓存与重启)一节重启设备并照推荐流程操作。

**Q：怎么恢复真实定位？**
A：最省事是运行「wloc 清理恢复位置」快捷指令；也可以关闭 / 删除模块，或清除 `wloc_settings` 持久化数据进入透传模式，详见[恢复真实定位](#恢复真实定位)。

**Q：我的位置数据会被上传吗？**
A：不会。地图链接解析接口 `/api/parse` 是纯转发解析，不写入储存、不记日志、不缓存；选点坐标只保存在你自己设备上代理工具的持久化储存里。不放心公共接口可以自部署 worker。

## 致谢

本项目的实现参考与受益于以下开源工作：

- [proxypin-wloc-spoofer](https://github.com/FFF686868/proxypin-wloc-spoofer) — 原始 WLOC 定位修改思路，by FFF686868
- [NSNanoCat/Util](https://github.com/NSNanoCat/util) — 跨平台脚本工具框架

获取模块或交流使用问题，也可加入 Telegram 群组「中文搜索大师」（[@Sousuozh_cn](https://t.me/Sousuozh_cn)），见[获取模块与交流](#获取模块与交流)。

## 免责声明

本项目仅供学习、开发测试与个人隐私保护使用。请遵守所在地法律法规及相关平台的服务条款，因使用本项目产生的一切后果由使用者自行承担。
