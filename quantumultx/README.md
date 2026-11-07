# Quantumult X

**空段修复已合并、真机结果待确认。** 原候选在 iPhone 导入时报「配置文件缺少模块 [rewrite_remote]」，QX 版本/build 未知。修复后的真机导入结果尚未确认。由主配置 `yaml/ACL4SSR_Online_Full_WithIcon.yaml` 机械派生，适用于 Quantumult X；不是 Mihomo 覆写。

- [配置 raw URL](https://raw.githubusercontent.com/r404r/override-hub/main/quantumultx/ACL4SSR_Online_Full_WithIcon.quantumultx.conf)
- [CN GeoIP 规则 raw URL](https://raw.githubusercontent.com/r404r/override-hub/main/quantumultx/geoip-cn.list)

## 导入

1. **等本轮自动测速变更合并到 main 后再更新导入**。空段兼容修复与 geoip 资源已先行合入 main，但修复后的真机导入仍待验证；下载前确认两个 raw URL 均可访问。本轮合并前配置链接仍提供旧的手选行为。
2. 在 QX 备份当前配置和策略选择，通过「下载配置」导入配置 raw URL，或复制配置内容。它是一份完整配置；先保留自己的节点资料。
3. 在 QX 客户端添加自己的节点或订阅。文件中的 `[server_local]` / `[server_remote]` 均为空，不含节点、订阅 URL 或凭证。
4. **在 App 中关闭「分流匹配优化」**。官方 sample.conf 没有记录可在配置文件里表达这个开关的键，不能只靠导入自动关闭；不同版本菜单位置以 App 为准。
5. 更新远程资源，确认原有 37 个列表和最后的 `GeoIP-CN` 共 38 个都加载成功。后者必须只有 `geoip, cn, 全球直连` 一行，保持最后一个，不要移回本地。
6. 记录 QX 版本/build、iOS 版本与导入结果；期望不再报缺段，若出现其他错误，回传错误原文。核对每组节点数量、当前选择与默认路径；检查活动记录中广告、AI、国内站点、Telegram 的策略。此修复尚未完成 iPhone 导入、正则、空组、DNS、UDP 和内存验证；空段是否触发证书提示或后台行为也需观察。

## 与主配置的差距

- 主配置 49 组生成 48 策略；成员顺序和节点筛选正则原样映射。`GLOBAL` 是 Mihomo 专有且无规则引用，不生成。QX 对后顾环视、测速组正则、组嵌套、候选组与正则并存的支持仍需设备核对。
- 必须关优化：同一历史域名快照的静态模型测得，开优化 113 / 10392 个落点不同，关优化 0 / 10392。**这只覆盖支持的域名规则，不代表所有请求等价**，也不是网络或客户端实测。
- `geoip, cn` 已放到远程资源末尾，使用原生 QX 格式和 `force-policy=全球直连`。远程资源先后顺序的模型仍待真机确认。
- 4 条 Telegram IP 补充仍在 `[filter_local]`，`no-resolve` 无对应字段。关优化后它们与远程域名/IP 规则的真实优先级及解析行为待真机验证；没有据此声称 IP 分流等价。
- 37 个原远程列表保持与主配置同 URL，经 KOP-XIAO `resource-parser.js` 和 `opt-parser=true` 转换。第三方脚本会在设备端执行，跟随上游变化；其中 GitHub、Google、OpenAI 是 YAML provider，须核对转换后的规则数。新增 geoip 资源使用 `opt-parser=false`。
- `empty-fallback: REJECT`（20 组）和 `expected-status: 204`（17 组）不能映射。**空组是否拒绝尚未验证**，不要在缺少节点时假定失败即拒绝。17 个测速组沿用 QX 候选基线的 600 秒间隔和 `alive-checking=false`（主配置为 300 秒），测速只用一个全局 URL；可能改变自动重新选点的时机。
- `AI固定出口` 已随主配置由 `static` 改为 `url-latency-benchmark`，显式候选只保留 `纯净节点-USA`，删除两个 SPEC 组，节点正则保持不变；只在美区 pure 名称范围内测速，不再保证手选出口保持不动。无匹配节点时没有 SPEC 备选；QX 的空组是否拒绝仍待真机验证。需要 SPEC 时在 OpenAI、Gemini 等业务组手动选。具体选择、嵌套 benchmark 与正则并存和旧选择迁移均待真机验证，Mihomo 的解除固定 API 不适用于 QX。
- 快照内 26 条 `PROCESS-NAME`、9 条 `URL-REGEX`、1 条复杂 `DOMAIN-REGEX` 不在域名对照模型内；QX 无这些原生分流写法，不补偿性加宽规则。复杂 OpenAI Azure WebPubSub 正则无法照搬，相关流量可能落到微软服务。
- **主配置没有、QX 侧单独存在**：`[dns]` 的 `no-ipv6`、`no-system`、`doh-server = https://223.5.5.5/dns-query, https://120.53.53.53/dns-query`。本轮按决定原样保留；禁 AAAA 不等于禁用全部 IPv6，也不证明没有 DNS 泄露。蜂窝和公共 Wi-Fi 解析待验证。
- 含 `[rewrite_remote]`、`[rewrite_local]`、`[task_local]`、`[http_backend]`、`[mitm]` 5 个空段头，因为 App 导入时要求段存在（已观察到缺 `[rewrite_remote]` 的报错）；其余四段按官方 sample 一并补齐，不代表已经验证所有版本都要求它们。五段内只有注释，没有任何 rewrite、任务/后端脚本、主机名或 CA 设置，也没有 MITM 规则。既有远程资源解析器仍按上条说明使用。
- 沿用 `fallback_udp_policy=reject`、`excluded_routes`、`dns_exclusion_list` 和全局测速 URL。被排除网段绕过隧道；它们不是主配置的完整行为映射。不声称延迟、连通性或省电收益。

## 再生与回滚

生成工具位于研究库 `p-research-clash` 的 `2026-11-07-002` 记录中（从 `2026-11-07-001` 复制，仅修正风险头部的记录数与组数口径）；独立校验器仍来自 `2026-11-07-001`。配置头部保留源文件 SHA256。维护者在研究库运行：

```sh
python3 research/2026/2026-11-07-002-ai-auto-exit/tools/gen_quantumultx.py --config ../override-hub/yaml/ACL4SSR_Online_Full_WithIcon.yaml --out ../override-hub/quantumultx/ACL4SSR_Online_Full_WithIcon.quantumultx.conf
python3 research/2026/2026-11-07-001-quantumultx-publish/tools/verify_quantumultx.py --offline
```

生成器同时重出配置与 `geoip-cn.list`，不要手改产物。校验器默认用已留存的同一历史快照；结果不代表上游列表或设备端解析器当前输出。

空段兼容修复已先行合并，本轮自动测速变更保留五个空段。真机发现问题时先切回备份配置，再由维护者按本轮记录对两库成对 `git revert`；不要只改 geoip 地址或规则顺序。回滚本次修复会恢复已知缺段、导入失败的原候选，不能当作可用配置；若后续调整已合并，应先逆序回滚后续相关改动。
