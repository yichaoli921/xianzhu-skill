# Step 0 · 题目还能不能做 — 操作清单

> skill：`xianzhu-skill` v2.0.1  
> 用途：进入 7 轮顺序**之前**必跑；不通过则**直接封存**，不进入显著性流程。  
> 与 `SKILL.md` 的 "Step 0" 段同步；本文档是它的**机械可执行版**。

---

## 为什么需要 Step 0

显著性试错的成本很高——一跑就是 7 轮。  
如果在 L1 阶段就已经知道数据没了，再往下跑就是在浪费：

- 数据可获取性 = L1
- 工具 / 计算环境 = L2
- 制度 / 法律 / 平台规则 = L3
- 伦理 / 风险 = L4

每一层都必须**显式确认通过**才能进入下一层；任一层失败 → **立即封存**，不进入显著性流程。

> ⚠️ **本约定是 v2.0.1 加入的硬纪律**：哪怕 7 轮顺序看起来已经触发，进 Step 0 时也要按 L1→L2→L3→L4 走。

---

## 4 层结构总览

| 层 | 检查目标 | 主要工具 | 失败处置 |
|---|---|---|---|
| **L1 数据可达** | 项目原始 / 中间 / 派生数据是否还在 | `ls`、`stat`、`wc -l`、`md5sum`、DB 登录 | 直接封存 |
| **L2 操作可达** | 跑数据的工具链是否就绪 | `which stata`、`mcp__stata-mcp__get_data_info`、`python3 -c "import ..."` | 直接封存 |
| **L3 制度 / 法律 / 平台规则** | 法律 / 政策 / 平台规则是否直接冲突 | `WebSearch` + `WebFetch` + 时间戳 | 询问用户，等明确意向 |
| **L4 伦理 / 风险** | 题目是否触及红线 / 风险不可调和 | 自我审阅 + 必要时回写用户确认 | 询问用户并要求文字确认 |

**执行顺序硬纪律**：L1 → L2 → L3 → L4，**不可乱序、不可跳过、不可合并**。

每一层都通过 → 进入 7 轮顺序。  
任一层失败 → 不再往下走，先去封存或回写用户。

---

## L1 · 数据可达

### 1.1 检查清单

- 项目根 / `raw/` / `tmp/` 目录存在
- `raw/**/*.dta`（或 `.csv` / `.sas7bdat`）每个关键文件**大小 > 0**
- 每个关键 `.dta` 用 `mcp__stata-mcp__get_data_info` / Python `pandas.read_stata` 能正确打开
- 行数与既有 `data_manifest.md` / `data_card.md` 一致
- 如有数据库账号（CSMAR / Wind / CNRDS / RESSET），最近一次下载日期距今 ≤ 90 天

### 1.2 命令模板（Bash）

```bash
# 1.2.1 项目根是否存在
test -d "/path/to/project" && echo "OK project root" || { echo "FAIL no project root"; exit 1; }

# 1.2.2 关键数据文件存在性 + 大小
for f in raw/xxx.dta raw/yyy.dta tmp/merged.dta; do
  if [ -f "/path/to/project/$f" ] && [ $(stat -f%z "/path/to/project/$f") -gt 0 ]; then
    echo "OK $f"
  else
    echo "FAIL $f 缺失或空"
  fi
done

# 1.2.3 样本审计：行数与既有 manifest 对齐
python3 - <<'PY'
import pandas as pd
from pathlib import Path
ROOT = Path("/path/to/project")
expected = {
    "raw/xxx.dta": 12345,
    "tmp/merged.dta": 12340,
}
for f, n in expected.items():
    try:
        df = pd.read_stata(ROOT / f)
        flag = "OK" if len(df) == n else f"FAIL expected={n} got={len(df)}"
    except Exception as e:
        flag = f"FAIL {e}"
    print(f"{ROOT/f}: {flag}")
PY

# 1.2.4 数据库账号快照（如果有 .env / config）
test -f ~/.config/csmar.json && jq -r '.last_download_at' ~/.config/csmar.json || echo "no DB config"
```

### 1.3 失败语义

- L1 失败 = **直接封存**，不进入 L2。
- "封存"动作：
  - 在项目根写 `STOP_step0_L1.md`（含时间戳 + 失败原因 + 涉及的命令输出）
  - 在 `agent_state.json` 写 `status=archived / failure_mark=hard_fail`
  - 在 `沟通记录.md` 追加一条 "Step 0 L1 失败" 记录
  - **不进入 7 轮顺序**

---

## L2 · 操作可达

### 2.1 检查清单

- Stata 可执行文件存在（`which stata` 或 `which StataMP`）
- Stata 许可证有效（`stata -q -e do` 能跑空脚本）
- MCP 通道可连（如果用了 `mcp__stata-mcp`）
- Python ≥ 3.10 且关键库装好：`pandas / numpy / linearmodels / pyfixest / statsmodels`
- 远程计算资源（如 SSH 到学校服务器）可达

### 2.2 命令模板（Bash）

```bash
# 2.2.1 Stata 可执行
which stata || which StataMP || { echo "FAIL no stata"; }

# 2.2.2 Stata 跑空命令
echo 'display 1+1' | stata -q -e do 2>&1 | tail -3

# 2.2.3 Python 与关键库
python3 - <<'PY'
import sys
print("python:", sys.version.split()[0])
mods = ["pandas", "numpy", "statsmodels", "linearmodels", "pyfixest"]
for m in mods:
    try:
        mod = __import__(m)
        print(f"OK {m} {getattr(mod, '__version__', 'unknown')}")
    except Exception as e:
        print(f"FAIL {m}: {e}")
PY

# 2.2.4 Stata MCP（如果用了）
mcp__stata-mcp__get_data_info data_path=/path/to/some.dta 2>&1 | tail -10
```

### 2.3 失败语义

- L2 失败 = **直接封存**（同上节 1.3 的"封存"动作，标 `STOP_step0_L2.md`）

---

## L3 · 制度 / 法律 / 平台规则（用 WebSearch / WebFetch）

> 这一层是显式允许使用 `WebSearch` 与 `WebFetch` 工具的场合。
> 调用规则：每个搜索查询必须**包含时间锚**（年份 + 月份 + 关键词），搜索结果要写进 `step0-audit.md` 的 L3 段，附 5 个以内最权威来源链接。

### 3.1 检查清单

按"题目相关程度"由近到远：

1. **直接相关政策**
   - 中央 / 部委官方文件
   - 关键词模板：<研究主题> + <实施年份> + "政策" OR "通知" OR "办法" OR "意见"
2. **数据源状态**
   - 数据库官方网站 / 公告（CSMAR、Wind、CNRDS、RESSET、CNINFO、HKEX）
   - 关键词模板：<数据库名> + "维护" OR "下线" OR "停售" OR "数据更新"
3. **平台规则**（如果用了爬虫 / 开放平台数据）
   - 微博、知乎、小红书、抖音、快手、B 站、微信公众号、百度
   - 关键词模板：<平台名> + "用户协议" OR "开放平台" OR "数据合规"
4. **法律修订**
   - 《统计法》《数据安全法》《个人信息保护法》《网络安全法》《反垄断法》《公司法》《证券法》
   - 关键词模板：法律名 + "修订" OR "施行" OR "发布"
5. **学术伦理 / 撤稿**
   - Retraction Watch、中国科学院文献情报中心
   - 关键词模板：<同领域代表人物/期刊/方法> + "撤稿" OR "更正"
6. **国际制裁 / 出口管制**（如果数据涉及境外实体）
   - 关键词模板：<国家/地区> + "制裁" OR "出口管制"

### 3.2 命令模板（WebSearch + WebFetch 调用规范）

```text
# 3.2.1 时间锚（每条搜索都包含）
SEARCH_ANCHOR="$(date '+%Y 年 %m 月')"   # 例如 "2026 年 08 月"

# 3.2.2 中央部委文件搜索（按部门）
WebSearch(query="${SEARCH_ANCHOR} 国务院/发改委/工信部/证监会 印发 关于 <研究主题> 的 通知/意见/办法")
WebFetch(url="https://www.gov.cn/zhengce/zhengceku/", prompt="列出最近 12 个月关于 <研究主题> 的 5 份核心政策文件，并指出每份文件的发布日期与是否在 <研究起始年份> 前后修改过")

# 3.2.3 数据源状态（按数据库）
WebSearch(query="CSMAR 数据 维护通知 2026")
WebFetch(url="https://www.gtdata.com.cn/", prompt="最近 3 个月是否有维护、停售、数据下架公告")

# 3.2.4 平台规则（按平台）
WebSearch(query="微博 开放平台 用户协议 修订 2026")
WebFetch(url="https://open.weibo.com/wiki/Policy", prompt="是否限制研究类数据访问")

# 3.2.5 法律修订
WebSearch(query="统计法 修订 2026")
WebFetch(url="http://www.npc.gov.cn/npc/c2/c30834/", prompt="最近 12 个月《统计法》《数据安全法》《个人信息保护法》是否有修订")
```

### 3.3 中国常见官方源速查

| 类别 | 名称 | URL | 备注 |
|---|---|---|---|
| 中央文件 | 中国政府网 | https://www.gov.cn/zhengce/ | 行政法规库、政策文件库 |
| 法律法规 | 国家法律法规数据库 | https://flk.npc.gov.cn/ | 人大系统立法 |
| 部委文件（发展改革） | 国家发改委 | https://www.ndrc.gov.cn/ | 重大政策、行业规划 |
| 部委文件（财税） | 财政部 / 税务总局 | https://www.mof.gov.cn/ / https://www.chinatax.gov.cn/ | 财政、税收 |
| 金融监管 | 国家金融监督管理总局 | https://www.nfra.gov.cn/ | 银保监 2023 改名 |
| 金融监管 | 中国证监会 | https://www.csrc.gov.cn/ | 资本市场政策 |
| 数据 / 信息化 | 国家数据局 | https://www.nda.gov.cn/ | 2023 成立，管"数据要素" |
| 经济数据 | 国家统计局 | https://www.stats.gov.cn/ | GDP、CPI、固投等 |
| 行业数据 | 工信部 | https://www.miit.gov.cn/ | 制造业、信息产业 |
| 上市公司公告 | 巨潮资讯 | https://www.cninfo.com.cn/ | A 股公告 |
| 上市公司公告 | 港交所披露易 | https://www.hkexnews.hk/ | 港股 |
| 上市公司公告 | SEC EDGAR | https://www.sec.gov/edgar | 美股 |
| 学术撤稿 | Retraction Watch | https://retractionwatch.com/ | 海外期刊 |
| 学术资料 | 中国知网 | https://www.cnki.net/ | 中文学术 |
| 学术资料 | Google Scholar | https://scholar.google.com/ | 跨语种 |
| 财经数据 | CSMAR / 国泰安 | https://www.gtdata.com.cn/ | 中国上市公司、股票 |
| 财经数据 | Wind | https://www.wind.com.cn/ | 多源金融数据 |
| 财经数据 | CNRDS | https://www.cnrds.com/ | 学术用注册研究系列 |

> L3 的搜索不是"必须有结论"，而是"必须有过尝试并留痕"。

### 3.4 失败语义

- L3 **发现冲突**（如数据已下线 / 法律禁止 / 方法已被撤稿）= **回写用户，等明确意向**
  - 在 `沟通记录.md` 追加一条 "Step 0 L3 冲突候选" 记录（含来源链接）
  - **不擅自决定是否继续**
  - 用户的回复必须是**显式**确认（"是的，继续" / "暂停" / "改数据源"）
- L3 **搜索结果不足**（找不到相关公告）= 进入 L4，**不视为失败**

---

## L4 · 伦理 / 风险

### 4.1 红线清单（任一打勾 → 询问用户并要求文字确认）

- 题目涉及**特定群体**的歧视性结论（性别 / 民族 / 地区 / 行业 / 信仰）
- 题目结论若成立，会触发**法律风险**（如证明某群体系统性违规）
- 数据来源**未经授权**或**明显违规获取**
- 与**已撤稿 / 已更正**的方法步骤实质重合
- 题目在**现行法规**下属于禁谈（如非法行业）
- 题目要求**绕过**已生效的隐私 / 个人信息保护规定

### 4.2 命令模板（无命令，纯自检）

```text
针对当前研究主题逐条检查 4.1 红线清单：
[ ] 4.1.1 特定群体歧视?
[ ] 4.1.2 法律风险?
[ ] 4.1.3 数据来源合规?
[ ] 4.1.4 方法重合已撤稿?
[ ] 4.1.5 现行法规禁谈?
[ ] 4.1.6 绕过隐私规定?

全部为否 → 通过 L4。
任一为是 → 询问用户并要求文字确认。
```

### 4.3 失败语义

- L4 红线打勾 = **询问用户并要求文字确认**
- 用户**拒绝**继续 = 封存（同 L1 / L2 "封存"动作）
- 用户**继续** = 进入 7 轮顺序，但在 `step0-audit.md` 保留"用户书面确认"快照作为留痕

---

## 输出格式：step0-audit.md

每跑一次 Step 0 都要在项目根（或 `output/审计/`）写一份：

```markdown
# Step 0 审计报告

- 项目：<项目名>
- 跑 Step 0 的时间：<ISO8601>
- 跑 Step 0 的一方：claude_code / codex / user
- 最终结果：通过 / L1 封存 / L2 封存 / L3 待用户决定 / L4 待用户确认 / 完全封存

## L1 数据可达
- 检了什么：
- 命令输出（30 行内摘录）：
- 结论：通过 / 失败

## L2 操作可达
- 检了什么：
- 命令输出：
- 结论：通过 / 失败

## L3 制度 / 法律 / 平台规则
- 时间锚：<日期>
- WebSearch 查询（最多 5 条）：
- WebFetch URL（最多 5 个）：
- 冲突候选（如有）：
- 结论：通过 / 待用户决定

## L4 伦理 / 风险
- 红线清单逐项打勾：
- 用户确认快照（如有）：
- 结论：通过 / 待用户确认

## 决策记录
- 接下来做什么：进入 7 轮顺序 / 封存 / 等用户回话
```

---

## 跨层硬纪律

1. **顺序硬性**：L1 → L2 → L3 → L4，不可乱序。
2. **不可跳过**：哪怕题目看起来"显然还在做"，也得跑完 L1 才能说通过。
3. **失败语义不混淆**：
   - L1 / L2 失败 = 直接封存，不询问用户
   - L3 / L4 失败 = 询问用户，等明确意向
4. **每层都必须留痕**：命令输出 / 搜索结果 / 用户确认必须写进 `step0-audit.md`
5. **`step0-audit.md` 不进审计目录**：随项目存档，归到 `output/审计/step0_<时间戳>.md`
6. **复用之前 Step 0 结果**：如果同一题目 30 天内跑过 Step 0 且 `failure_mark != hard_fail`，可跳过 L1/L2 但**L3 必跑**，L4 视情况

---

## Step 0 与 7 轮顺序的衔接

```
Step 0 通过 → 进入 7 轮顺序
Step 0 失败 → 封存
                              │
                              ▼
┌─────────────── 7 轮顺序 ───────────────┐
│ 第 1 轮 自然基准                          │
│ 第 2 轮 调 Y                             │
│ 第 3 轮 调 X                             │
│ 第 4 轮 调模型                           │
│ 第 5 轮 事件研究↔主回归一致性             │
│ 第 6 轮 上控制变量                       │
│ 第 7 轮 审计"显著是不是假的"             │
└────────────────────────────────────┘
```

7 轮顺序里**不重新跑 Step 0**——Step 0 是一锤子买卖，除非研究主题发生了显著变化（例如换了被解释变量、换了主样本、换了主处理变量定义）。

---

## 何时**重新跑 Step 0**

满足以下任一，重新跑：

- 研究主题变了
- 数据源换了（例如从 CSMAR 换到 Wind）
- 主要被解释 / 处理变量定义换了
- 法律法规 / 平台规则发生变化（特别是修订公告）
- 距上次 Step 0 跑过已经超过 90 天
