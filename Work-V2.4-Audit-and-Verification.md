# Work V2.4 — 哈希稳定版｜审查与验证报告

审查基线：`Work-V2.1-Flat-Priority-Fallback-TEST.toml`；结果文件：`Work-V2.4-Hash-Stable-No-RoundRobin.toml`。

## 结论和改动范围

- **保留** `⚖️ Work 精选哈希`，算法 `consistent-hashing`、候选正则、健康检查网址 `https://www.gstatic.com/generate_204` 和间隔全部逐字段不变。
- **删除** `⚖️ Work 精选轮询` 策略组及 `🔀 非标端口` 中对该组的选项。
- **删除** `🔀 非标端口` 内的 `DIRECT` 手动出口选项，降低通过这个入口误选直连的风险；其他业务分流中的 DIRECT 原样保留。
- 非标端口仅保留 `🐟 遵循规则`（默认）及 `⚖️ Work 精选哈希`。
- **不改变**五个国家的 `fallback`、MiSub Provider 引用、其他组和 36 条规则集定义。
- **不改变**你在 OpenClash 覆写设置里添加的 `IP-CIDR,192.168.110.1/32,DIRECT,no-resolve`；这是 OpenClash 本地设置，而非此模板定义。

## 实际执行的离线检查

| 检查项 | 结果 |
|---|---|
| TOML 语法解析 | 通过 |
| 策略组数 | 42，原 43 减轮询 1 组 |
| 规则集引用 | 36，与基线完全相同 |
| 其他 41 组逐字段一致 | 通过（不含非标端口选择器） |
| 哈希组的算法、正则、地址、时间间隔 | 与 V2.1 完全相同 |
| 五个国家 fallback | 与 V2.1 完全相同 |
| `[]` 显式策略组引用 | 381 条，未出现悬空引用 |
| 显式组引用循环检测 | 未发现循环 |
| 历史订阅样本名称正则筛选 | 44 条：HK 15、SG 14、US 15；Deep 36、YC/YC2 8 |
| 选择器默认项目 | 仍为 `🐟 遵循规则` |

## 仍未验证、不能承诺的项目

1. 转换服务 `api.asailor.org` 接收 V2.4 外部 TOML 并生成 Mihomo YAML。
2. OpenClash 最终运行配置是否保持 HTTPS、国家组顺序和哈希算法。
3. 44 条历史候选节点中每条能否连接具体的远端 SOCKS5 网关；Mihomo HTTPS 测速无法代表这个能力。
4. AdsPower 环境最终公网出口、DNS/WebRTC 泄漏，以及 SOCKS5 断开时 AdsPower 是否允许直连。
5. 长时间并发吞吐、成功率和重启恢复；没有在线路由器或代理账号，无法真正执行这些实测。

## 值得持续关注的风险

- 一致性哈希按**目标地址**决定选择，不是按每个 AdsPower 环境固定，也不是自动选择最低延迟；当多个环境共享相同 SOCKS5 网关 IP 时仍可能共用机场线路。
- 当候选池变动、健康状态变化、Provider 更新或 OpenClash 重载时，目标与机场节点的映射可能变化；既有连接可能中断。
- `🐟 遵循规则` 会继续跟随 `🐟 漏网之鱼` 的选择；该后者保留原有 DIRECT 备选，此版不触碰其他业务策略。
- 整个 Provider/组完全为空时的 empty-fallback 风险没有在本版解决。必须另行验证默认出口，不可承诺“零直连”。
- 既有广义 `Custom_Port_Direct` 规则还可捕获其他应用的非常规端口。新版本没有新增端口规则，也没有隔离专属 AdsPower 流量。

## 安全部署与回退

1. 备份现有运行配置并保留 V2.1，**不要覆盖原文件**。
2. 将新 TOML 上传公共 GitHub 原始文件地址。模板不得包含订阅密钥或代理认证信息。
3. 在 OpenClash 中只为测试订阅设置新 Raw URL；生成 YAML，先不直接切换业务。
4. 检查 `⚖️ Work 精选哈希` 的 `strategy: consistent-hashing`、`use`、`filter`，且 `🔀 非标端口` 只有 `🐟 遵循规则` 和 `⚖️ Work 精选哈希`。
5. 验证 `/etc/openclash/working.yaml` 中相关测速地址仍为 HTTPS；OpenClash 优先自定义 DNS 直连规则仍在 `Custom_Port_Direct` 之前。
6. 空闲时应用并确认 `🐟 遵循规则` 正常；再以少数 AdsPower 环境测试哈希，确认真实出口是各自指定的家宽 SOCKS5 公网 IP，非预期直接出口要立即停止测试。
7. 若失败，先将 `🔀 非标端口` 切回 `🐟 遵循规则`，必要时重新选用原 V2.1 订阅配置。

## 公开技术资料

- Mihomo load-balance 文档：https://wiki.metacubex.one/en/config/proxy-groups/load-balance/
- Mihomo 社区负载均衡 health-check Bug 报告：https://github.com/MetaCubeX/mihomo/issues/2564
- Mihomo 社区嵌套 fallback 状态问题：https://github.com/MetaCubeX/mihomo/issues/2588
- SubConverter TOML 与策略组生成源码：https://github.com/tindy2013/subconverter/blob/master/src/generator/config/subexport.cpp
