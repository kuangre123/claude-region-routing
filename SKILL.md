---
name: claude-region-routing
description: Configure or troubleshoot per-domain routing for Claude web, Desktop, and direct Anthropic API traffic through verified US or Singapore proxy exits. Use for requests such as “Claude 走美国/新加坡” or “Claude 同地区自动切换” in Clash/Mihomo, Surge, sing-box, or another rule-capable proxy client.
---

# Claude 地区分流

让当前设备上发往 Claude 官方服务的流量走用户指定的出口地区。有多个可用节点时，优先使用只含该地区节点的自动切换组。

- 将美国、US、美区归一为 `US`，将新加坡、Singapore、SG 归一为 `SG`。诊断现有配置时无需先选地区。配置目标不明确且无法从上下文推断时，先识别客户端和候选节点，再问目标地区。
- 识别实际运行的客户端、内核及版本、活动配置、规则模式和可持久保存的覆盖层。区分浏览器代理、系统代理、TUN、终端代理环境变量及单独网关；以实际 Claude 请求的路由为准。客户端适配见 [客户端参考](references/clients.md)，选线和实测见 [验证参考](references/verification.md)。
- 使用域名后缀匹配官方服务：`claude.ai`、`claude.com`、`anthropic.com`、`claude.app`、`claudeusercontent.com`、`claudemcpcontent.com`。需要核对新功能时查 [Claude Desktop 网络要求](https://code.claude.com/docs/en/desktop#network-access-requirements) 和 [Claude Code 网络配置](https://code.claude.com/docs/en/network-config)。第三方登录、连接器站点和自定义 API 网关按实际目标单独处理。Anthropic 云端发起的连接不经过本机代理。
- 节点名称只用来找候选。逐条确认 Claude 网页和 API 可达，并核对服务看到的出口国家。只有通过两项验证的同地区节点才能入组。优先放入至少两条；只有一条时说明没有自动切换能力。没有合格候选时先报告缺口；用户要求严格分流时，可以按其授权明确拒绝该流量。任何情况下都不偷偷加入其他地区、直连或不受地区约束的默认组。
- 以当前配置里的节点名或出站标签建立专用策略，将 Claude 规则放在可能抢先命中的规则之前。重复运行时更新原有专用规则和组；切换 US/SG 后检查旧规则不会抢先命中。保持其他域名的分流规则。只有诊断或要求示例时输出建议；用户要求应用时，先准备可回退的小范围改动，通过配置校验后使用客户端支持的方式应用。不得把只读订阅、账号会话或整机出口当作规则覆盖层。
- 应用后用新连接核对规则命中、节点、网页和 API 出口地区。有条件时在隔离配置中模拟首选节点失效，确认备用出口仍在同一地区。验证失败时回退本次变更或报告未生效的具体部分。报告实际验证范围、当前选中的线路及故障切换能力。Claude 的旧会话位置可能与实时出口不同。
