# 客户端适配

示例的 `SG` 可换为 `US`。替换 `SG A` 和 `SG B` 为在当前客户端实际验证通过的节点名。示例以两个节点展示；策略成员不得包含默认代理、直连或其他地区。域名列表用 SKILL.md 中的六个后缀。

## Clash Verge Rev、Clash/Mihomo

先确认内核版本、规则模式及当前配置。在 Clash Verge Rev 的活动订阅「编辑规则」「编辑代理组」前置区保存；这两层独立于订阅原文。其他 Clash 客户端优先使用受支持的覆写，或者编辑用户拥有的本地配置。下面是完整配置的片段。使用前置覆盖时，分别把列表项目放入客户端对应的前置区。只有候选节点对 Anthropic API 根路径的 HEAD 请求都实测返回 404 时，才沿用示例的探测设置。

```yaml
proxy-groups:
  - name: Claude-SG
    type: fallback
    include-all-proxies: true
    filter: '^(?:SG A|SG B)$'
    empty-fallback: REJECT
    url: https://api.anthropic.com/
    expected-status: '404'
    interval: 60
    timeout: 5000
    lazy: false
    max-failed-times: 1
rules:
  - 'OR,((DOMAIN-SUFFIX,claude.ai),(DOMAIN-SUFFIX,claude.com),(DOMAIN-SUFFIX,anthropic.com),(DOMAIN-SUFFIX,claude.app),(DOMAIN-SUFFIX,claudeusercontent.com),(DOMAIN-SUFFIX,claudemcpcontent.com)),Claude-SG'
```

- 正则只匹配已经验证的精确名称；转义正则和 YAML 中的特殊字符。示例的 `include-all-proxies` 适用于配置内的实际节点。使用 `proxy-providers` 时，按实际来源指定 `use` 并验证最终成员，避免匹配任意未经验证的供应商节点。节点重命名后重新验证；`empty-fallback: REJECT` 防止筛空时走直连。
- 对不支持 `OR` 的版本，前置六条 `DOMAIN-SUFFIX,<域名>,Claude-SG`。不支持安全筛选和空组拒绝时，用明确列出的实际节点，校验节点移除或组不可用时的行为；无法保证时不要声称严格分流。
- 使用已安装的 Mihomo `-t -d <配置目录> -f <候选文件>` 或客户端校验。优先由客户端重载。如果必须用本机控制接口，先确认运行配置与源配置一致，保存持久覆盖后再热加载，验证 `/rules`、`/proxies` 和 `/connections`，失败时恢复改动。只改生成文件或只做运行时热加载会在客户端重建时丢失。保留现有控制接口的监听和权限。
- UDP 需要验证：Mihomo 可能跳过选中节点不支持 UDP 的代理规则。必要时验证专用的 Claude UDP 拒绝规则，防止 QUIC 流量落入后面的默认规则。

[Mihomo 代理组](https://wiki.metacubex.one/en/config/proxy-groups/) · [路由规则](https://wiki.metacubex.one/en/config/rules/) · [本地 API](https://wiki.metacubex.one/en/api/)

## Surge

在可编辑配置的 `[Proxy Group]` 中加 `Claude-SG = fallback, SG A, SG B, interval=60, evaluate-before-use=true`。在 `[Rule]` 里前置六条 `DOMAIN-SUFFIX,<域名>,Claude-SG`。模块可加规则；代理组需要在可编辑配置里建立。检测 URL 由节点的 `test-url` 或现有全局测试设置决定，保持全局设置，按实际版本核对每条线路。空组和所有节点失效时需要验证退出路径。

[Surge Fallback](https://manual.nssurge.com/policy-groups/fallback.html) · [Surge 模块](https://manual.nssurge.com/profile/module.html)

## Stash

使用 `.stoverride` 覆盖，为 `proxy-groups` 增加 `type: fallback`、明确的 `proxies` 和 `interval`。保留节点已配置的测速设置，必要时按该版本核对 `benchmark-url`。为 `rules` 前置六条 `DOMAIN-SUFFIX,<域名>,Claude-SG`。核对覆写合并顺序、同名策略与失效行为。Stash 空组可能走直连；不要使用可能筛空的动态组来承诺严格分流。

[Stash 覆写](https://stash.wiki/en/configuration/override) · [代理组](https://stash.wiki/en/proxy-protocols/proxy-groups)

## sing-box

在 `outbounds` 增加类似 `{"type":"urltest","tag":"claude-sg","outbounds":["SG-A","SG-B"],"interval":"1m","interrupt_exist_connections":false}` 的出站。成员是现有出站标签。根据所用版本验证健康检测 URL 和成功条件；`urltest` 按延迟选节点，无法保留顺序时应告诉用户。添加 `{"domain_suffix":["claude.ai","claude.com","anthropic.com","claude.app","claudeusercontent.com","claudemcpcontent.com"],"action":"route","outbound":"claude-sg"}` 到 `route.rules`。放在必要的 sniff/DNS 处理之后、一般路由之前。旧版语法以实际内核为准；使用 `sing-box check -c <候选文件>`。确认 `route.final` 不会接管失败的 Claude 流量。

[sing-box URLTest](https://sing-box.sagernet.org/configuration/outbound/urltest/) · [路由规则](https://sing-box.sagernet.org/configuration/route/rule/)

## Loon、Quantumult X、Shadowrocket

| 客户端 | 配置入口 |
| --- | --- |
| Loon | 在 `[Proxy Group]` 加 `Claude-SG = fallback,SG A,SG B,url=<已实测探测URL>,interval=60`；在 `[Rule]` 前置六条 `DOMAIN-SUFFIX,<域名>,Claude-SG`。 |
| Quantumult X | 在 `[policy]` 加 `available = Claude-SG, SG A, SG B`；在 `[filter_local]` 前置六条 `host-suffix, <域名>, Claude-SG`。 |
| Shadowrocket | 用当前版本的配置界面或导出格式创建地区策略组和域名规则；确认当前版本确有可用性切换。版本不支持时可用手动地区组，明确没有自动切换。 |

这些客户端用真实节点名或标签；核对规则模式、全部节点不可用时的行为和 UDP 支持。Quantumult X 的全局 `fallback_udp_policy` 可能把不支持的 UDP 流量交给其他地区，检查该设置并保持全局设置，必要时报告限制。未能检查实际客户端时交付可审查的片段，注明未应用或未验证的项目。

[Loon 策略组](https://nsloon.app/docs/Policy/policygroup/) · [Quantumult X 官方示例](https://github.com/crossutility/Quantumult-X/blob/master/sample.conf) · [Shadowrocket 官方功能页](https://apps.apple.com/app/shadowrocket/id932747118)

## Xray、v2rayN 及其他

- 对用户拥有的原生 Xray JSON，使用 `routing.rules` 的 `domain:claude.ai` 等后缀匹配，指定 `balancerTag`。`leastPing` 需要 observatory；为已验证节点设置唯一且专用的标签前缀（选择器是前缀匹配），明确把 `fallbackTag` 指向 blackhole 出站。核对真实匹配成员，使用当前版本的配置测试。v2rayN 等前端的生成文件应通过前端支持的自定义配置或路由入口维护。
- 有按域名规则但没有故障切换时，使用用户选定且实测的同地区节点或手动组。只有全局 VPN 模式时，没有 Claude 专用分流；报告限制。若用户明确接受整机走指定地区，可给出相应设置和影响。不要在尚未确定客户端能力时套用其他软件的格式。

[Xray 路由与负载均衡](https://xtls.github.io/en/config/routing)
