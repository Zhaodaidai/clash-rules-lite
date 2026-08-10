# 审计发现

## 配置定位

- `SuperClash.ini` 是 subconverter 的 `[custom]` 远程配置模板。
- 当前包含 22 条规则集映射、8 个策略组和 2 个生成开关。

## 初步问题

- 同一 GitHub Raw 地址混用 `/main/` 与 `/refs/heads/main/`，可读性不一致。
- 第三方规则混用了 Clash、Loon、Shadowrocket 目录，需确认规则语法是否可直接被转换器消费。
- `🚀️ 下载节点` 的筛选正则重复 `大流量`。
- `GPT` 名称偏口语化，且节点匹配只覆盖中英文中的美国、日本、韩国缩写，需结合现有用途判断是否更名与补全。
- 策略组排序未按规则命中顺序或依赖关系组织，维护时不易扫描。
- README 描述的是旧仓库用途，与当前 `SuperClash.ini` 只能作为有限上下文，不能直接据此改写业务策略。

## 待确认

- 仓库自有四个列表均使用 Clash classical 规则语法；`dns.list` 内存在两个重复域名，但不属于本次 `SuperClash.ini` 修改范围。
- 最近历史修改意图以及是否存在可用的本地验证工具。
- 官方规则端点的存在性及内容格式。

## 官方语法核验

- subconverter 官方 `pref.example.ini` 说明 `url-test` 参数格式为 `interval[,timeout][,tolerance]`。
- 当前 `1200,50` 和 `180,50` 只设置了 50 秒超时，没有设置容差；官方示例使用 `300,5,100`。
- `ruleset` 未写类型时默认按 Surge 文本规则解析；当前 `.list` 内容为兼容的逐行文本规则，不应误标为 `clash-classic` YAML。
- blackmatrix7 当前同时提供 PikPak、Telegram 的 Clash `.list`，HTTP 状态均为 200，可替换现有 Shadowrocket/Loon 路径。
- OpenAI、Civitai、China、ChinaMedia 的当前 Clash 路径均返回 200。

## 重构方向

- 保留现有业务分流范围，不擅自新增服务。
- 规则按“拦截/专用服务/自定义/国内直连/通用代理/兜底”整理，保证专用规则早于通用规则。
- 策略组名称使用服务真实名称 `OpenAI`，健康检查使用 HTTPS 和显式三参数。
- 删除重复筛选词，统一 GitHub Raw URL 写法。
- 现有 18 个第三方及自维护远程端点均返回 HTTP 200。
- `ban.list` 只有注释和空白，不产生规则；其远程引用属于无效配置，应删除。
- 广告规则当前位于中国直连规则之后，部分国内广告域名可能先命中直连；广告规则应前置。
- 远程规则显式设置 86400 秒更新间隔，避免依赖转换器默认值。
- 自维护 `direct.list` 是有效业务输入，必须保留且置于通用国内规则之前。
