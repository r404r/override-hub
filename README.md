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

由 `ACL4SSR_Online_Full_WithIcon.yaml` 派生的[塔台](https://github.com/pengchujin/tower)规则方案（subconverter INI 语法），策略组、组成员和规则顺序与之一致。在塔台“规则方案”中通过上面的链接导入；raw 链接不可达时可改用 jsDelivr 镜像 `https://testingcf.jsdelivr.net/gh/r404r/override-hub@main/yaml/ACL4SSR_Online_Full_WithIcon.tower.ini`（更新可能滞后），或复制文件内容以文本导入。节点来自塔台里的订阅，再由塔台导出 sing-box 等客户端配置。

- 按塔台 1.0.21 源码编写，并在本机用模拟塔台行为的脚本和 sing-box 1.14.1 验证；尚未在塔台 App 中实际导入和导出。
- 导入时塔台会下载全部 38 个规则列表（jsDelivr 镜像），任一下载失败都会中止导入。
- 为适配塔台所做的调整：
  - 塔台会把没有节点的组改为直连，所以原本空组拒绝的组都以 `REJECT` 作为最后一个成员，保持无节点时拒绝连接。它始终是成员：`手动切换`、`奈飞节点` 中会多一个 `REJECT` 选项，各地区测速组也会包含它。sing-box 中 `REJECT` 是 `block` 出站。
  - 塔台导出 sing-box 时会丢弃 `GEOIP,CN`，所以在它之前加入了 ACL4SSR `ChinaIp.list`（仅 IPv4）。
  - `UnBan` 中大小写混合的后缀补充了小写版本。
- 导出 sing-box 时的限制：
  - 进程名、URL-REGEX、IP-ASN 规则会被丢弃。
  - 规则列表全部内联进配置，按模拟结果约 0.9 MB。列表更新后需要在塔台刷新并重新导出。
  - 塔台的 sing-box 配置先解析所有连接再匹配规则：无法解析的域名连接会失败；IP 规则对所有连接生效，不同于 Mihomo 的 `no-resolve`。
  - 图标和测速 `expected-status` 不会保留。
- 为 Clash、Surge、Loon、Quantumult X、Egern 导出时不要开启塔台的“代理集合”：开启后 `REJECT` 会排在订阅节点之前，`手动切换`、`奈飞节点` 会默认选中 `REJECT`。sing-box 导出不受影响。

### JavaScript

[布丁狗的订阅转换.js](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/javascript/%E5%B8%83%E4%B8%81%E7%8B%97%E7%9A%84%E8%AE%A2%E9%98%85%E8%BD%AC%E6%8D%A2.js)

[防止dns泄露(雾).js](https://raw.githubusercontent.com/mihomo-party-org/override-hub/main/javascript/%E9%98%B2%E6%AD%A2dns%E6%B3%84%E9%9C%B2(%E9%9B%BE).js)

### 开发说明

仓库约定见 [AGENTS.md](AGENTS.md)，Claude Code 入口见 [CLAUDE.md](CLAUDE.md)。
