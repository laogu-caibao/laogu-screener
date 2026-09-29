# 数据源（实测日期：2026-09-29，北京时间）

> 实测方式说明：本机 `curl` 直连被沙箱出口代理拦截（datacenter-web/push2/qt.gtimg.cn/hq.sinajs.cn 均返回 000），
> 以下"可用"结论经抓取层直取真实返回验证（含完整 JSON 字段），用户侧 `curl` 按"请求要求"直接可用。
> push2 在本沙箱返回 502（与方法论 §13 已知坑一致），用户侧通常可用。

## 1. 东财 datacenter 通用接口（主力）

```
https://datacenter-web.eastmoney.com/api/data/v1/get?reportName={report}&columns={columns}&filter={filter}&pageSize={page_size}&pageNumber={page_number}&sortColumns={sort_col}&sortTypes={sort_type}&source=WEB&client=WEB
```

- 请求要求：桌面端 UA；服务端直连可通，无需 Referer；`columns=ALL` 必带（不带返回 `code:9501,"返回字段参数不能为空"`）
- 返回：`{result:{pages,data:[...],count},success,message,code}`；data 为对象数组
- 分页：`pageSize`/`pageNumber`；翻页取全
- 实测状态：**可用**（2026-09-29）

### 1a. RPT_SHAREBONUS_DET（分红送转 / 股息率）

- 用途：高股息策略全市场筛选；近 12 个月股息率
- 关键字段：`SECURITY_CODE` 代码、`SECURITY_NAME_ABBR` 名称、`PRETAX_BONUS_RMB` 每10股派息(元，税前)、`DIVIDENT_RATIO` **股息率（小数，×100 转百分比）**、`EX_DIVIDEND_DATE` 除权除息日、`IMPL_PLAN_PROFILE` 分红方案文本（如"10派1.80元(含税)"）、`ASSIGN_PROGRESS` 实施进度、`REPORT_DATE` 报告期
- 实测：`sortColumns=EX_DIVIDEND_DATE&sortTypes=-1` 返回最新分红（2026-09-29 实测首条：华泰证券 601688，DIVIDENT_RATIO=0.010089686099≈1.01%，EX_DIVIDEND_DATE=2026-10-23）
- 筛选示例：`filter=(EX_DIVIDEND_DATE>'2025-09-29')` 取近 12 个月分红 + 本地按 `DIVIDENT_RATIO` 排序（filter 不支持对计算字段直接比较，拉全量本地筛）
- 注意：A 股代码白名单过滤（B 股 900xxx 也会分红）；`DIVIDENT_RATIO` 取已实施口径，预案未实施的看 `ASSIGN_PROGRESS`
- 实测状态：**可用**（2026-09-29）

### 1b. RPT_F10_BASIC_ORGINFO（公司信息 / 行业 / 实控人）

- 用途：行业归属（消费/科技映射）、国企识别、剔除 ST
- 关键字段：`SECURITY_CODE`、`SECURITY_NAME_ABBR`、`EM2016` 东财三级行业（如"化石能源-石油天然气-石油天然气开采"）、`ACTUAL_HOLDER` 实际控制人、`SECURITY_TYPE`（含"ST"字样即 ST 股）、`LISTING_DATE` 上市日期（剔除上市<1 年次新股）
- 实测：`filter=(SECURITY_CODE="601857")&columns=SECURITY_CODE,SECURITY_NAME_ABBR,ACTUAL_HOLDER,EM2016` → 中国石油，ACTUAL_HOLDER="国务院国有资产监督管理委员会"（2026-09-29）
- 国企关键词清单（ACTUAL_HOLDER 含任一即判国企）：国资委、国有资产、财政部、中央汇金、国新、诚通
- 实测状态：**可用**（2026-09-29）

### 1c. RPT_DAILYBILLBOARD_DETAILSNEW（龙虎榜，可选增强）

- 用途：三档定级时查"提示风险"（近 3 个月是否频繁上榜/游资炒作）
- 关键字段见 laogu-lhb；本 skill 仅作定级辅助，不列入主流程
- 实测状态：**可用**（2026-09-29，兄弟 skill 同日验证）

## 2. 东财 push2 全市场快照（备选：PE/市值/换手率）

```
https://push2.eastmoney.com/api/qt/clist/get?pn={page}&pz={size}&po=1&np=1&ut=bd1d9ddb04089700cf9c27f6f3f5f080&fltt=2&invt=2&fid=f3&fs=m:0+t:6,m:0+t:80,m:1+t:2,m:1+t:23&fields=f12,f13,f14,f2,f8,f20,f21,f62,f115,f152
```

- 字段：`f12` 代码、`f14` 名称、`f2` 最新价、`f8` 换手率、`f20` 总市值、`f21` 流通市值、`f62` 振幅、`f115` **PE(TTM)**、`f152` 市盈率(动)
- `fs` 市场范围：`m:0+t:6,m:0+t:80` 深圳（主板+创业板）、`m:1+t:2,m:1+t:23` 上海（主板+科创板）；北交所 `m:0+t:81` 按需加
- `fid=f3&po=1` 按涨幅排序；改 `fid=f115` 可按 PE 排序取低 PE（配合 `po=1` 升序）
- 实测状态：**本沙箱 502（2026-09-29），已知坑**；用户侧通常可用，列为"备选"而非主力
- 降级：502 时改用下述网页搜索模板逐只核验 PE/市值

## 3. 新浪批量行情（备选）

```
https://hq.sinajs.cn/list={symbols}   # symbols 如 sh600519,sz000858（逗号分隔，一次 ≤ 50 只）
```

- 请求要求：必须带 `Referer: https://finance.sina.com.cn`，否则 403；返回 GBK 编码，转 UTF-8
- 字段（逗号分隔）：0 名称、1 今开、2 昨收、3 现价、4 最高、5 最低…30 日期、31 时间（无 PE/市值，仅作价格核验）
- 实测状态：**本环境未测通**（抓取层拒绝），按方法论已知配置记为备选；用户侧按上述要求可用

## 4. 腾讯批量行情（备选）

```
https://qt.gtimg.cn/q={codes}   # codes 如 sh600519,sz000858
```

- 字段：1 名称、3 现价、4 昨收、32 涨跌幅、38 换手率、39 PE(TTM)、46 市值（亿）
- 实测状态：**本沙箱连接失败**（000）；方法论记载部分网络超时，记为备选

## 5. 网页搜索模板（兜底：单只核验）

| 需求 | 搜索关键词模板 |
|---|---|
| PE/市值核验 | `{代码} {名称} 市盈率TTM 总市值 2026年9月` |
| ROE/营收增速 | `{代码} {名称} 2026半年报 ROE 营收同比` |
| 分红核验 | `{代码} {名称} 分红 派息 除权日` |
| 负面/问询函 | `{代码} {名称} 问询函 / 减持 / 立案调查` |
| 自然语言映射复核 | `{行业词} 东财行业分类 EM2016` |

- 搜索结果必须注明来源与日期；搜不到即标"未核验"

## 降级链总览

```
高股息/国企/行业：datacenter（主力，2026-09-29 已验证）
    → 全失败：网页搜索模板逐只（慢，仅 Top20）
PE/市值/换手：push2 clist（用户侧主力，本沙箱 502）
    → 腾讯 qt.gtimg.cn → 新浪 hq.sinajs.cn（Referer+GBK）
    → 网页搜索模板逐只核验
ROE/营收增速/现金流：无稳定全市场公开接口
    → 票池内（≤50 只）网页搜索/研报逐只核验，输出注明"票池核验"
K 线类策略（放量/趋势/MACD/布林）：push2his K 线（本 skill 未实测）
    → 降级为"给出票池 + 计算口径说明"，由宿主按口径自行计算并标注
```

## 已证伪 / 不要用

- `RPT_STOCK_VALUE`、`RPT_DCS_VALUE_A`、`RPT_VALUE_AGGREGATE`、`RPT_F10_MAIN_FINANCE`、`RPT_F10_FINANCE_MAININDEX`：2026-09-29 实测返回 `code:9501,"报表配置不存在"`，**不要用**
- 同花顺问财（iwencai）自然语言选股：反爬强（JS 加密动态参数），本 skill 的"自然语言→条件"由 LLM 映射表完成，**不直调问财**
- 北向"净流入"口径：2024-08-19 起已死，本 skill 不涉及
