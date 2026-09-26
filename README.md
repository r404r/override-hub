## Mihomo Party 常用覆写脚本/示例

### 使用方法

1. 复制对应文件 raw 直连，如 `https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/javascript/%E5%B8%83%E4%B8%81%E7%8B%97%E7%9A%84%E8%AE%A2%E9%98%85%E8%BD%AC%E6%8D%A2.js`。
2. 打开 Mihomo Party，左侧导航栏打开“覆写”页面，粘贴链接后导入，即可看到对应的覆写脚本/配置。
3. 左侧导航栏打开“订阅管理”，点击需要覆写的订阅右上角的三个点，选择“编辑信息”。
4. 在打开的对话框中最后一项“覆写”，选择刚刚导入的覆写脚本/配置，保存即可。

### YAML

[布丁狗的订阅转换.yaml](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/yaml/%E5%B8%83%E4%B8%81%E7%8B%97%E7%9A%84%E8%AE%A2%E9%98%85%E8%BD%AC%E6%8D%A2.yaml)

[ACL4SSR_Online_Full.yaml](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/yaml/ACL4SSR_Online_Full.yaml)

[ACL4SSR_Online_Full_WithIcon.yaml](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/yaml/ACL4SSR_Online_Full_WithIcon.yaml)

[添加直连规则.yaml](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/yaml/%E6%B7%BB%E5%8A%A0%E7%9B%B4%E8%BF%9E%E8%A7%84%E5%88%99.yaml)

### sing-box

[ACL4SSR_Online_Full_WithIcon.sing-box.json](https://raw.githubusercontent.com/r404r/override-hub/main/yaml/ACL4SSR_Online_Full_WithIcon.sing-box.json)

由 `ACL4SSR_Online_Full_WithIcon.yaml` 派生的 sing-box 订阅转换模板，策略组、组成员和规则顺序与之一致（不含图标）。它不是可直接运行的配置：需要用 [sing-box-subscribe](https://github.com/Toperlock/sing-box-subscribe) 把订阅节点填入 `{all}`，并按 `filter` 正则分组。例如在其 `providers.json` 中把 `config_template` 设为上面的链接，`emoji` 设为 `0`，`prefix` 留空（地区和 `^pure-` 筛选依赖原始节点名），`auto_set_outbounds_dns` 留空。节点名不能与 `DIRECT`、`REJECT` 或策略组同名。

- 使用 sing-box 1.14 配置格式，已用 sing-box 1.14.1 与 sing-box-subscribe `4782237` 验证。入站为 TUN（`auto_route`、`strict_route`；Linux 可自行加 `auto_redirect`）和 `127.0.0.1:7890` mixed。clash_api 监听 `127.0.0.1:9090`，未设 `secret`，CORS 只允许本地来源；使用网页面板时请设置 `secret` 并加入面板来源。`7890` 与 Mihomo Party 默认 mixed 端口相同，两者的 TUN 也会互相冲突，不要同时启用。
- DNS：`geosite-cn`、`geosite-private` 等国内与局域网列表的域名经 223.5.5.5 DoH 返回真实 IP（`nas.lan` 这类局域网主机名公共 DNS 解析不了）。其余域名的 A/AAAA 查询返回 fake-ip（`198.18.0.0/15`、`fc00::/18`，TTL 1 秒），HTTPS/SVCB 查询返回空应答。
- 这些 fake-ip 域名的连接：
  - 按规则直连的 TCP 连接用 223.5.5.5 解析；按规则走代理的 TCP 连接把域名交给节点。
  - UDP 连接，以及未命中列表、需要按 `GEOIP,CN` 判断的连接，先经 `自动选择` 用 8.8.8.8 DoH 解析，再以 IP 发出。
  - 其他类型的查询也经 `自动选择` 用 8.8.8.8 解析。`自动选择` 没有可用节点时，这些解析失败，对应连接直接断开，即使最终会走直连；只有切到 Direct 模式能恢复 UDP，Global 模式不行。
  - 在国内列表中、却被规则分到代理的域名，节点收到的是 223.5.5.5 解析出的 IP。
- 停止 sing-box 后，系统或应用缓存的 fake-ip 会短暂失效；删除 `cache.db` 会重新分配 fake-ip。
- `REJECT` 是指向本机未监听端口的占位出站，用于替代 Mihomo 的 REJECT，所以可能为空的组末尾都会带一个 `REJECT` 选项。筛选不到节点的组、`广告拦截` 与 `应用净化` 会连接失败，而不是回退直连。
- 规则集从 jsDelivr 镜像直连下载（KaringX/karing-ruleset、senshinya/singbox_ruleset、MetaCubeX/meta-rules-dat），并缓存到 `cache.db`；首次启动时镜像不可达会导致启动失败。`BanAD` 内联了 ACL4SSR `70d11f1` 版本的快照，不会自动更新。`Download`、`AI-Transit`、`SharedServices`、`UnBan-Lowercase`、`OneDrive-Process` 也为内联。
- 与 Mihomo 版的已知差异：
  - 不支持 URL-REGEX；Android 包名形式的进程规则不生效。
  - `ProxyGFWlist` 的转换源缺少 `ip138.com`；`OneDrive` 使用 geosite，会多出少量域名，例如 `sharepoint.cn`。
  - 测速不检查 204 状态码；UDP 流量会跳过不支持 UDP 的节点，没有可用节点时落到 `REJECT`。
  - TCP 连接中，IP 规则（最后的 `GEOIP,CN` 除外）只匹配目标已经是 IP 的连接。
  - fake-ip 域名的 UDP 连接在匹配规则前就已解析，所有规则集里的 IP 段都会生效，例如 `WeChat` 包含腾讯云国际 AS132203，`Apple` 包含 `17.0.0.0/8`，还有局域网段。这类 UDP（如 QUIC）可能先被这些规则直连，与同域名的 TCP 以及 Mihomo 的 `no-resolve` 行为不同；解析失败的域名，其 UDP 连接会直接断开。
- 其他：`LocalAreaNetwork` 包含 `100.64.0.0/10`，Tailscale 等 CGNAT 地址会直连；`strict_route` 在 Windows 上可能影响虚拟机网络。

### 塔台（Tower）

[ACL4SSR_Online_Full_WithIcon.tower.ini](https://raw.githubusercontent.com/r404r/override-hub/main/yaml/ACL4SSR_Online_Full_WithIcon.tower.ini)

[塔台](https://github.com/pengchujin/tower)规则方案（subconverter INI 语法），以塔台内置的“ACL4SSR 全分组”（ACL4SSR `75f0101`）为底：内置规则一行未改，内置分组的内容除下面列出的 5 个选择组新增可选项外保持原样，仅分组的列出顺序被重排。在此基础上只插入 `ACL4SSR_Online_Full_WithIcon.yaml` 的专属规则，并追加以下专属分组：`纯净节点`、`纯净节点-USA`、`AI固定出口`、`OpenAI`、`Gemini`、`Claude`、`AI中转`、`AI-MIX`、`共享服务`、`Google`、`Google直连例外`、`GitHub`、`日韩台自动`、`纯日韩台自动`。

- **导入**：在塔台“规则方案”中通过上面的链接或复制文本导入，节点来自塔台里的订阅。导入时塔台会下载全部规则列表，任一下载失败都会中止导入；其中 ACL4SSR 列表使用 raw.githubusercontent.com 固定版本，网络受限时请在可访问 GitHub 的环境下导入。
- **沿用塔台内置方案的部分**（与 Mihomo 覆写不同）：
  - 组名带 emoji；地区组正则较宽松；
  - `Ⓜ️ 微软服务` 默认直连，并使用 ACL4SSR 的 Microsoft 列表；
  - `🎥 奈飞视频` 默认 `🎥 奈飞节点`；
  - 没有 `GLOBAL` 组；保留 `💬 Ai平台`，但 AI 域名会先被专属规则匹配；
  - 地区组筛选不到节点时会被塔台改为直连。
- **失败即拒绝**：`纯净节点`、`纯净节点-USA`、`日韩台自动`、`纯日韩台自动` 以 `REJECT` 结尾，保证没有匹配节点时被拒绝而不是直连；sing-box 中 `REJECT` 是 `block` 出站。`日韩台自动` 直接按严格的日本、台湾、韩国节点名正则测速，不再嵌套内置地区组，因为塔台会把没有节点的地区组改为直连。`纯日韩台自动` 在此之上再要求节点名以 `pure-` 开头，正则与 Mihomo 覆写中的同名组一致。
- **追加的可选项**：`🚀 节点选择` 的第二、三项为 `日韩台自动` 和 `纯日韩台自动`，末尾加了两个 pure 组；`📹 油管视频`、`🌍 国外媒体`、`📢 谷歌FCM` 末尾加了两个 pure 组；`💬 Ai平台` 的首项加了 `AI固定出口`（该组默认项因此变为 `AI固定出口`）。其余组的默认项不变。
- **分组顺序**：`🚀 节点选择`、`🚀 手动切换`、`♻️ 自动选择` 在最前，其后是 AI 相关组，再是其他服务组与直连/拦截组，节点类分组（地区、pure、奈飞、`日韩台自动`、`纯日韩台自动`）排在最后。顺序只影响列表显示。
- **`💬 Ai平台`**：保留内置分组与其两条规则。它的两个列表共 52 条规则，其中 51 条会先被前面的专属规则匹配；已知例外是 `challenges.cloudflare.com` 的子域（专属规则用的是精确域名，内置列表用的是后缀），这类请求仍由 `💬 Ai平台` 处理，没有 pure 节点时会被拒绝。
- **导出 sing-box 时**：
  - 打开塔台导出页高级选项里的“优先使用规则集”后，塔台**内置的那 32 个 ACL4SSR 列表**会换成预编译规则集（sing-box 启动时下载）；该开关默认关闭，关闭时所有列表都会内联进配置。
  - 进程名、URL-REGEX、IP-ASN 规则会被丢弃。`GEOIP,CN` 也会被丢弃，因此在它前面加入了 MetaCubeX 的国内 IP 列表 `geo/geoip/classical/cn.yaml`（9741 条，含 IPv4 与 IPv6）。没有它时，按 IP 发起、嗅探不到域名的国内连接（微信自有协议就是这样）会被送去代理，表现为微信提示网络不稳定。
  - 该列表不在塔台内置的 32 个列表中，所以导出 sing-box 时，无论“优先使用规则集”开或关都会内联进配置，不依赖这个开关。它与 Mihomo geodata 模式下 `GEOIP,CN` 的数据同源（已比对 `geoip.dat` 的 CN 段，9741 条完全一致）。
  - 导出到 Clash、Surge 等目标时这一行是冗余的：这些客户端本来就保留 `GEOIP,CN`，各自使用自带的数据库。另外该列表的 IPv6 条目写作 `IP-CIDR` 而非 `IP-CIDR6`，Surge、Loon、Quantumult X 可能忽略或报错；Clash 系与 sing-box 不受影响。
  - 体积：导出 sing-box 时，开启“优先使用规则集”的 lite 由约 146 KB 增至约 379 KB，默认关闭时由约 225 KB 增至约 458 KB。本机模拟（120 节点）显示内存增加约 1 MiB 量级，与多次运行的波动同量级。
  - 该列表的地址未固定版本，跟随上游更新；导入时塔台会下载它（任一列表下载失败会中止导入）。导出 Clash 系目标并开启“优先使用规则集”时，它会成为一条每天自动更新的远程规则集。
- **代理集合**：为 Clash、Surge、Loon、Quantumult X、Egern 导出时不要开启塔台的“代理集合”，否则以 `REJECT` 结尾的那 4 个组（`纯净节点`、`纯净节点-USA`、`日韩台自动`、`纯日韩台自动`）会把 `REJECT` 排在订阅节点之前；sing-box 导出不受影响。
- **分组选择会被记住**：sing-box 按组名保存手动选择，并可能在不同配置之间沿用。导入后请在 sing-box 的分组页确认 `🐟 漏网之鱼`、`🚀 节点选择` 和各 AI 组的当前选择，特别是之前用过同名分组的配置时。
- **验证范围**：按塔台 1.0.21 源码，在本机用模拟塔台行为的脚本和 sing-box 1.14.1 验证；未在塔台 App 中实际导入和导出。

#### 塔台内存优化版

[ACL4SSR_Online_Full_WithIcon.tower-lite.ini](https://raw.githubusercontent.com/r404r/override-hub/main/yaml/ACL4SSR_Online_Full_WithIcon.tower-lite.ini)

由上面的塔台方案派生，供 iOS 上 sing-box 内存紧张时使用。iOS 的 sing-box 网络扩展内存上限为 50 MB，启动时、每次网络切换时，以及内存告警触发网络重置时，所有自动测速组都会同时测试各自的节点。

- **与原方案的差异**：
  - `🇭🇰 香港节点` 等 6 个地区组和 `纯净节点` 改为手动选择，不再自动测速，默认为组内第一个匹配的节点，没有故障切换。无节点时 `纯净节点` 仍为 `REJECT`，地区组仍会被塔台改为直连。
  - 自动测速组从 11 个减为 4 个：`♻️ 自动选择`、`纯净节点-USA`、`日韩台自动`、`纯日韩台自动`。`🚀 节点选择` 和 AI 组的默认路径不变。`纯日韩台自动` 保持自动测速——它本身就是「自动」组，改成手动会失去意义；代价是本方案的测速组比加入该组之前多一个。
  - `📺 巴哈姆特` 默认的 `🇨🇳 台湾节点` 从自动测速变为该组第一个节点。
  - ACL4SSR `ProxyGFWlist` 换为 `ProxyLite`：`ProxyGFWlist` 中有、`ProxyLite` 中没有的约 5,500 个域名改由最后的 `🐟 漏网之鱼` 决定，默认同样走 `🚀 节点选择`。如果 `🐟 漏网之鱼` 被选为直连，这些域名也会直连。约 20 个同时在国内列表中的域名改为直连。
- **导入后请检查**：
  - 地区组沿用塔台内置的宽松正则，并且不区分大小写，第一个匹配的节点可能是“剩余流量”之类的信息节点，或名字里碰巧带 US、台等字样的其他地区节点。建议在塔台中开启“过滤订阅节点信息”，导入后在 sing-box 分组页确认各地区组的当前节点。
  - sing-box 按组名记住手动选择，从原方案切换过来时，原来的选择可能沿用。
- **效果**：在 Linux 上用 sing-box 1.14.1 模拟 120 个节点、GOGC 与 iOS 相同（Go 内存上限按 50 MiB，iOS 为 40 MiB），启动时进程内存峰值约减少 4 MiB；节点更少时减少得更少。这不是 iOS 实测，实际效果以设备为准。导出的 sing-box 配置约为原方案的四分之三（开启“优先使用规则集”时本机实测 368 KiB vs 503 KiB）——早先“约为一半”的说法基于 2026-09-20-001 的体积，此后两套方案都内联了约 233 KB 的国内 IP 列表，比例已经变了。
- **可能的其他因素**：订阅节点数量、自动测速组中的 Hysteria2/TUIC 等 QUIC 节点，影响大小尚未测量。如需定位，可在 sing-box App 的“工具 → 内存不足报告”（OOM Report）中导出报告；分享时请关闭“附带配置”（With Configuration，含节点凭证）。
- 若不需要 `日韩台自动` 或 `纯日韩台自动`，删掉其中任一个可各少一个测速组（早先对 `日韩台自动` 的模拟中约减少 2 MiB；`纯日韩台自动` 的代价未单独测量）。

### JavaScript

[布丁狗的订阅转换.js](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/javascript/%E5%B8%83%E4%B8%81%E7%8B%97%E7%9A%84%E8%AE%A2%E9%98%85%E8%BD%AC%E6%8D%A2.js)

[防止dns泄露(雾).js](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/javascript/%E9%98%B2%E6%AD%A2dns%E6%B3%84%E9%9C%B2(%E9%9B%BE).js)

### 开发说明

仓库约定见 [AGENTS.md](AGENTS.md)，Claude Code 入口见 [CLAUDE.md](CLAUDE.md)。
