# Quantumult X

**主线策略引用修复已合并，本轮 AI 自动测速候选尚无导入结果、待真机验证。** 补齐空段后的 main 候选在 iPhone 导入时报「该类型策略不可引用其它策略」，指向「日本节点」←「日韩台自动」；QX 版本/build 未知。本候选保持自动测速，改用节点正则，尚未真机复测。由主配置 `yaml/ACL4SSR_Online_Full_WithIcon.yaml` 机械派生，适用于 Quantumult X；不是 Mihomo 覆写。

历史回报需核对：13:45 JST 曾回报成功（iOS 27.2 (24B119)，QX 1.8.1-915）；15:06 JST 又回报上述策略引用错误。两次回报相互矛盾，原因未知，不能把任一次当作已验证结果；本候选尚未真机验证，也不能据前一次回报认定段结构、语法或 `[dns]` 无 `server=` 已被接受。

- [配置 raw URL](https://raw.githubusercontent.com/r404r/override-hub/main/quantumultx/ACL4SSR_Online_Full_WithIcon.quantumultx.conf)
- [CN GeoIP 规则 raw URL](https://raw.githubusercontent.com/r404r/override-hub/main/quantumultx/geoip-cn.list)

## 导入

1. **等本轮 AI 自动测速变更合并到 main 后再更新导入**。空段修复、geoip 资源和策略引用修复均已合入主线；本轮合并前 raw URL 仍提供 AI固定出口 的手选行为。本候选尚无导入结果，不能用历史回报认定可导入；下载前确认两个 raw URL 可访问。
2. 在 QX 备份当前配置和策略选择，通过「下载配置」导入配置 raw URL，或复制配置内容。它是一份完整配置；先保留自己的节点资料。
3. 在 QX 客户端添加自己的节点或订阅。文件中的 `[server_local]` / `[server_remote]` 均为空，不含节点、订阅 URL 或凭证。
4. **在 App 中关闭「分流匹配优化」**。官方 sample.conf 没有记录可在配置文件里表达这个开关的键，不能只靠导入自动关闭；不同版本菜单位置以 App 为准。
5. 更新远程资源，确认原有 37 个列表和最后的 `GeoIP-CN` 共 38 个都加载成功。后者必须只有 `geoip, cn, 全球直连` 一行，保持最后一个，不要移回本地。
6. 记录 QX 版本/build、iOS 版本与导入结果；期望不再出现「该类型策略不可引用其它策略」，若出现其他错误，回传错误原文。确认 `日韩台自动` 仍是自动节点测速，核对成员等于日本、台湾、韩国三组筛选的并集，选择它后观察自动测速结果。核对每组节点数量、当前选择与默认路径；检查活动记录中广告、AI、国内站点、Telegram 的策略。此候选尚未完成 iPhone 导入、正则、空组、DNS、UDP 和内存验证；空段是否触发证书提示或后台行为也需观察。

## 与主配置的差距

- 主配置 49 组生成 48 策略（31 个 `static`、17 个 `url-latency-benchmark`，由生成器统计）；除下述非 static 策略引用转换外，其它成员顺序和节点正则原样映射。`GLOBAL` 是 Mihomo 专有且无规则引用，不生成。真机已证实 `url-latency-benchmark` 引用其它策略会报「该类型策略不可引用其它策略」，本次指向日韩台自动引用日本节点。`static` 引用其它策略仍是「官方 sample 有示例，这次导入没有报这条」，不能写成已验证或已否证。后顾环视、测速组正则、static 候选与正则并存仍需设备核对。
- **按指定行为保持自动测速**：`日韩台自动` 仍为 `url-latency-benchmark`，保留图标和测速参数，候选中删除三个地区策略名，`server-tag-regex` 改成日本节点、台湾节点、韩国节点三条筛选正则的逻辑或。测的是三个地区中全部匹配的节点，**不是三个区域组当前选中的那一个节点**。主 YAML 没有改，仍比较子组当前出口；这是本候选明确采用的行为差异。其它非 static 策略也按同样方式递归展开策略候选到节点正则；已有自身筛选也纳入并集。所有 static 策略及 `纯日韩台自动` 的筛选范围保持原样，不打平 static 组。
- `AI固定出口` 随主配置由 `static` 改为 `url-latency-benchmark`；用 003 轮生成器将原显式 `纯净节点-USA` 与自身节点筛选展开、去重为同一条美区 pure 节点正则，行内不再有任何策略名。节点范围与原 filter 匹配集合相同，不再保证手选出口保持不动；名字不证明实际出口地区。无匹配节点时没有 SPEC 备选，QX 空组行为仍待真机验证。需要 SPEC 时到 OpenAI、Gemini 等业务组手动选择；Mihomo 的解除固定 API 不适用于 QX。
- 必须关优化：同一历史域名快照的静态模型测得，开优化 113 / 10392 个落点不同，关优化 0 / 10392。**这只覆盖支持的域名规则，不代表所有请求等价**，也不是网络或客户端实测。
- `geoip, cn` 已放到远程资源末尾，使用原生 QX 格式和 `force-policy=全球直连`。远程资源先后顺序的模型仍待真机确认。
- 4 条 Telegram IP 补充仍在 `[filter_local]`，`no-resolve` 无对应字段。关优化后它们与远程域名/IP 规则的真实优先级及解析行为待真机验证；没有据此声称 IP 分流等价。
- 37 个原远程列表保持与主配置同 URL，经 KOP-XIAO `resource-parser.js` 和 `opt-parser=true` 转换。第三方脚本会在设备端执行，跟随上游变化；其中 GitHub、Google、OpenAI 是 YAML provider，须核对转换后的规则数。新增 geoip 资源使用 `opt-parser=false`。
- `empty-fallback: REJECT`（20 组）和 `expected-status: 204`（17 组）不能映射。**空组是否拒绝尚未验证**，不要在缺少节点时假定失败即拒绝。17 个测速组沿用 QX 候选基线的 600 秒间隔和 `alive-checking=false`（主配置为 300 秒），测速只用一个全局 URL；可能改变自动重新选点的时机。
- 快照内 26 条 `PROCESS-NAME`、9 条 `URL-REGEX`、1 条复杂 `DOMAIN-REGEX` 不在域名对照模型内；QX 无这些原生分流写法，不补偿性加宽规则。复杂 OpenAI Azure WebPubSub 正则无法照搬，相关流量可能落到微软服务。
- **主配置没有、QX 侧单独存在**：`[dns]` 的 `no-ipv6`、`no-system`、`doh-server = https://223.5.5.5/dns-query, https://120.53.53.53/dns-query`。本轮按决定原样保留；禁 AAAA 不等于禁用全部 IPv6，也不证明没有 DNS 泄露。蜂窝和公共 Wi-Fi 解析待验证。
- 含 `[rewrite_remote]`、`[rewrite_local]`、`[task_local]`、`[http_backend]`、`[mitm]` 5 个空段头，因为 App 导入时要求段存在（已观察到缺 `[rewrite_remote]` 的报错）；其余四段按官方 sample 一并补齐，不代表已经验证所有版本都要求它们。五段内只有注释，没有任何 rewrite、任务/后端脚本、主机名或 CA 设置，也没有 MITM 规则。既有远程资源解析器仍按上条说明使用。
- 沿用 `fallback_udp_policy=reject`、`excluded_routes`、`dns_exclusion_list` 和全局测速 URL。被排除网段绕过隧道；它们不是主配置的完整行为映射。不声称延迟、连通性或省电收益。

## 再生与回滚

当前生成器与校验器来自研究库 `p-research-clash` 已合入主线的 `2026-11-07-003` 记录；`2026-11-07-002` 的旧生成器副本不再用于当前产物；配置头部保留源文件 SHA256。维护者在研究库运行：

```sh
python3 research/2026/2026-11-07-003-qx-policy-reference/tools/gen_quantumultx.py --config ../override-hub/yaml/ACL4SSR_Online_Full_WithIcon.yaml --out ../override-hub/quantumultx/ACL4SSR_Online_Full_WithIcon.quantumultx.conf
python3 research/2026/2026-11-07-003-qx-policy-reference/tools/verify_quantumultx.py --offline --structure-only
```

生成器同时重出配置与 `geoip-cn.list`，不要手改产物。上述校验只检查结构，不运行历史域名落点模型；规则快照仍为研究库已留存的历史语料，结果不代表上游列表或设备端解析器当前输出。

主线策略引用修复与五个空段均已合并，本轮在此基础上生成 AI固定出口 的自动测速配置。真机发现问题时先切回备份，再由维护者对两库成对 `git revert` 本轮实际合并提交；只回滚本轮会回到包含策略引用修复和五个空段的主线版本。不要连带 revert 更早的策略引用修复或空段修复，否则会恢复已知导入错误；不要只改 geoip 地址或规则顺序。后续依赖若已合并，应先逆序回滚。
