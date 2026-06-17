# CLAUDE.md — a-stock-data（lcy fork）

> Claude Code 打开本目录时**自动加载**本文件。这是 [simonlin1212/a-stock-data](https://github.com/simonlin1212/a-stock-data) 的 fork（[taicilang-lcy/a-stock-data](https://github.com/taicilang-lcy/a-stock-data)），版本 `3.2.2-lcy.1`。

## 这是什么

A股全栈数据 **Claude Code Skill**（单文件 `SKILL.md`）。7 层数据源 / 27 端点：行情、研报、信号、资金面、新闻、基础数据、公告。完整用法见 `SKILL.md`，完整变更见 `CHANGELOG.md`。

## lcy fork 相对 upstream 改了什么（v3.2.2-lcy.1）

- **P0**：`download_pdf` 路径穿越修复（#3）；新增 `requirements.txt` 锁 `mootdx<0.11`（#26）。
- **P1**：`em_get` 连接级自动重试（urllib3 Retry）；腾讯字段停牌容错（`_to_float`）；`full_valuation` EPS 按列名解析（不再靠固定 iloc）；`eastmoney_reports` 暴露 `q_type=1`（行业研报）。
- **复审补**：requirements 补 `lxml`/`urllib3`；`full_valuation` 残留裸 float 修复；`eastmoney_reports` code guard；`download_pdf` 正则补控制字符。
- **有意未改**：iwencai 函数（核对源码 `_claw_headers()` 头齐全、无 bug）；异常处理（防御式 `print+默认值` 是 agent skill 的合理设计，保留）；渐进式披露拆分（P0-4，82KB 单文件→scripts/+references/，多日重构，留后续 phase）。

> 逐条证据见 `CHANGELOG.md` 的 `v3.2.2-lcy.1`。

## 配置（首次使用）

```bash
cd /Users/lcy/ai_workspace/ai_toolset/a-stock-data
pip install -r requirements.txt   # mootdx<0.11, requests, pandas, lxml, urllib3, stockstats
```

- **iwencai 语义搜索**（可选）：需 `IWENCAI_API_KEY`。代码只读**环境变量**（`os.environ.get`，不读 `.env` 文件），所以二选一：
  - 推荐：`echo 'export IWENCAI_API_KEY="你的key"' >> ~/.zshrc`（全局，所有项目生效）
  - 或：写进目标项目的 `.claude/settings.json` 的 `env` 字段（仅该项目 CC 子进程生效）
  - ⚠️ **别放项目 `.env`**——代码不会加载它（iwencai 永远 401）；且容易被误提交泄露凭证。
  其余数据源全部免费无 key。
- **网络**：mootdx 走国内通达信 TCP 7709，**海外 IP 不可用**；其余 HTTP 接口海外可用。

## 研报拉取（本项目主要用途）—— 两条路

| 路 | 接口 | key | 调用 | 擅长 |
|---|---|---|---|---|
| 主力 | 东财 reportapi | 无 | `eastmoney_reports(q_type=1, industry="通用设备", begin="2026-03-17", end="2026-06-17")`；`q_type=0`=个股 | 板块整体（周报/月报，量大） |
| 精准 | iwencai 语义 | 需 | `iwencai_search("减速器 丝杠 灵巧手", channel="report")`；channel 还支持 announcement/news | 子环节专题精准命中 |

- 东财频率限制：并发 <10 / <200 次·分 / <300 次·5分 → 触发封禁。`em_get` 已内置 1s 串行节流 + 重试，**抄代码即自带防封**。`pageSize=100`，上千份研报 = 几十次分页。
- iwencai 须 8 个请求头（`Authorization: Bearer` + `Content-Type` + 6 个 `X-Claw-*`，其中 `X-Claw-Trace-Id` 必须 64 字符）—— `_claw_headers()` 已正确生成，**不要删/改这些头**，否则 `401 当前 Skill 版本过低`。

## 开发约定

- **所有改动先在本 fork**（origin = taicilang-lcy），不直接改 upstream。
- **上游同步**：`git fetch upstream` → 按需 **cherry-pick 单个端点修复**（不要直接 merge master——全内联 vs 拆分会冲突）。
- **版本号**走独立线 `-lcy.N`。
- **改完自检**：① 对 SKILL.md 所有 ` ```python ` 块跑 `ast` 语法校验；② 关键数据改动用真实公开 API 实测返回非空。

## 注意 / 已知坑

- `SKILL.md` 是 82KB 单文件（渐进式披露未拆，已知技术债，P0-4 待办）。
- iwencai key 是凭证，**勿提交 git**，走环境变量。
- 大陆住宅 IP 对东财有间歇风控（`HTTP 000`/空），非代码 bug，换网络/重试即可。
- 早期调研曾误判"iwencai 缺请求头"——那是 grep 截断窗口的错觉，源码实际头齐全。勿重复踩。
