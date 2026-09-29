# 多策略选股

`laogu-screener`

从全市场 A 股按条件筛出候选名单：给定 PE/股息率/市值/ROE/营收增速等多维条件，或直接说一句人话（"低PE高分红的国企"），自动转成结构化筛选条件。内置 11 种经典策略（高股息/低估值/高ROE/费雪成长/现金流质量/长期低位/近期放量/趋势跟踪/MACD底背离/布林带下轨/筹码集中），可单筛可组合。每只入选个股给一句话入选理由 + 强制三档定级（值得跟踪/保持观望/提示风险），只做逻辑陈述、不做买卖推荐。

## 文件结构

- `SKILL.md` — 主流程（平台中立，纯 Markdown，可移植）
- `references/sources.md` — 数据源：每个接口实测状态 + 降级链（2026-09-29 实测）
- `MARKET.md` — 市场调研：供给/需求/槽点/差异化
- `mcp-config.json` — laogu-mcp 热更新配置（config_version 1.0.0）

## 明确排除

只做"从全市场筛出候选"，不做个股诊断：已给定具体股票代码/名称的分析请用 `laogu-fundamentals`；资金流向解读请用 `laogu-moneyflow`。

## 一键安装

```bash
npx skills add laogu-caibao/laogu-screener
```

```bash
# 方式一：clone 仓库
git clone https://github.com/laogu-caibao/laogu-screener.git

# 方式二：下载 ZIP
https://github.com/laogu-caibao/laogu-screener/archive/refs/heads/main.zip
```

- Claude Code：放到 `~/.claude/skills/laogu-screener/`
- 豆包工作 / Workbuddy：按平台流程导入（zip 根目录已有 SKILL.md，直接上传）
- 扣子：扣子编程 → 技能面板 → 创建技能 → 本地上传（页面要求 `.skill` 后缀时由扣子导入后自动生成，不要只改 zip 扩展名）
- Trae：设置 → 技能 → 上传技能；或手动放到 `~/.trae/skills/laogu-screener/`（国区版 `~/.trae-cn/skills/`）
- 一次装全：`uvx laogu-mcp`（MCP 版，数据抓取走 tool，解读仍按本 skill 的 Output Contract）

---
## English

**laogu-screener — Multi-strategy A-share screener.** Filter the whole market by your conditions, including natural-language queries, with results in a structured Chinese table. Install: `npx skills add laogu-caibao/laogu-screener`.

## FAQ

**Q：laogu-screener 有什么用？**
适合的场景：想按自己的条件（估值、市值、财务指标等）从全市场筛 A 股，直接用大白话描述筛选条件。

**Q：数据可靠吗？会荐股吗？**
数字必须来自可核验的公开来源（上市公司公告、交易所公开数据、公开网页），取不到就标「未核验」，绝不编造；只做结构化整理与解读，不构成投资建议。

**Q：怎么安装？支持哪些 AI 平台？**
```bash
npx skills add laogu-caibao/laogu-screener
```
平台中立 Markdown，Claude Code、Codex、豆包智能体、Workbuddy、扣子 Coze、Trae 等环境均可用；数据能力可用 [laogu-mcp](https://github.com/laogu-caibao/laogu-mcp)（`uvx laogu-mcp`）一次装齐。更多 skill 见[老谷拆财报组织主页](https://github.com/laogu-caibao)。
---

## 出品

**老谷拆财报** —— 以数据为刃，剖市场真相

- 抖音 / 微信视频号 / 今日头条 / 快手：搜索「老谷拆财报」

财经科普、财报解读。个人观点，仅供参考，不构成投资建议。
