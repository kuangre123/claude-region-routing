# Claude 地区分流 · Claude Region Routing

一个 Codex Skill，帮助你把 Claude 网页、桌面版和 Claude Code 直连官方 API 的流量分流到美国或新加坡。它会检查节点实际出口，在客户端支持的情况下，只在通过验证的同地区节点之间自动切换。

## 安装

首次安装时，把仓库克隆到 Codex 的技能目录：

```sh
git clone https://github.com/kuangre123/claude-region-routing.git "${CODEX_HOME:-$HOME/.codex}/skills/claude-region-routing"
```

如果该目录已经有同名技能，先比较文件，再按需更新。如果技能没有立即出现在 Codex 中，重启 Codex 后再试。

## 使用

- `$claude-region-routing 让 Claude 走美国，线路故障时只切美国节点`
- `$claude-region-routing 让 Claude 走新加坡，先给我看配置`
- `$claude-region-routing 检查 Claude 网页和 API 当前走哪个地区`

技能会识别当前代理客户端，检查分流规则和实际出口。你可以先看配置，再决定是否应用。需要你已有可用的代理线路。

已用 Mihomo 校验美国和新加坡配置模板，并验证空组拒绝行为。仓库也提供 Surge、Stash、sing-box、Loon、Quantumult X、Shadowrocket 和 Xray 的配置参考。其他客户端需要按实际版本验证规则和线路。只支持全局 VPN 的客户端无法单独为 Claude 分流。

详见 [技能说明](SKILL.md)、[客户端适配](references/clients.md) 和 [出口验证](references/verification.md)。

## English

A Codex skill for routing Claude service domains through verified US or Singapore proxy exits. Where the client supports it, failover stays within the selected country. Bring your own proxy client and nodes. Install with the command above, then ask Codex: `$claude-region-routing Route Claude through Singapore and preview the configuration first.`

[MIT License](LICENSE)
