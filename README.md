# xianzhu-skill ✨

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:2F6BFF,100:5CC8FF&height=170&section=header&text=xianzhu-skill&fontSize=42&fontColor=ffffff&animation=fadeIn&fontAlignY=36&desc=Specification%20search%20for%20empirical%20economics&descAlignY=57&descAlign=50" alt="xianzhu-skill header" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-2.0.0-2F6BFF" alt="version" />
  <img src="https://img.shields.io/badge/license-MIT-111827" alt="license" />
  <img src="https://img.shields.io/badge/focus-empirical%20economics-0EA5E9" alt="focus" />
  <img src="https://img.shields.io/badge/workflow-Stata--friendly-2563EB" alt="workflow" />
  <img src="https://img.shields.io/badge/status-public%20release-16A34A" alt="status" />
</p>

> 📌 一个给 **Codex** 用的中国实证经济学家向 skill：
> **系统性、可审计、解释优先** 的规格搜索（specification search）协议。

`xianzhu-skill` 解决一个最常见的研究困境：
**研究问题已经定了，但回归结果不够稳、不够可信、或者不够显著，还没到能写进论文的程度。**

它不会鼓励盲目追求 p-value，
而是用一份**先检查能不能做、锁定边界、按层诊断、一轮只动一个东西、最后看"是不是真的稳"**的协议，
把"在同一个 `x → y` 关系附近继续试"这件事变成可审计的、有边界的工作流。

---

## 🌟 本版本更新亮点（v2.0.0）

> 每发一版就把这一段换成最新的；要查历史变更看 [CHANGELOG](CHANGELOG.md)。

相对于 v1.0.0，v2.0.0 **真正新增的不是 reference 文档数量，是工作流的结构性升级**：

| # | 结构性升级 | 体现在 |
|---|---|---|
| 1 | **多了 Step 0：先判断题目是否还值得做** | 制度 / 法律 / 伦理 / 平台规则 / 数据可获得性有任何重大冲突 → **直接封存，不进入显著性流程** |
| 2 | **多了数据口径突变 / 法律风险 / 平台规则审计** | 在固定识别和样本前先审计：法律改了？平台改版了？数据换口径了？变量断点在哪一年？ |
| 3 | **多了模型层面的固定顺序**，尤其**事件研究一致性** | TWFE / 交错 DID / 堆叠 DID / 事件窗口 / 聚类层级 / FE 结构 **按顺序试**；**主回归和事件研究绑在一起审**，图与表冲突立即停 |
| 4 | **多了样本收缩与 FE 吸收问题诊断** | merge / 控制变量 / 清洗规则不能让样本塌缩；FE 不能把 DID 交互项吸收到看不见 |
| 5 | **多了显式停机规则** | 见 [references/stop-rules.md](references/stop-rules.md) — 不再默认"无限试到显著"，机械式停机条件清楚列出 |

加上原有的合规底盘（边界 + 解释优先 + 失败留痕 + Stata-friendly），
形成**"先看能不能做 → 看什么被动了 → 一轮只动一处 → 必须真稳"**的完整流程。

---

## 🧭 什么时候用

用户说一句以下话，本 skill 就该被加载：

- "试到显著" / "换个口径试试"
- "结果方向对但不显著怎么办"
- "别换研究问题，围绕当前方程继续试"
- "为什么这个方程不显著"
- "调参 / 试模型 / 稳住结果"

如果用户只是想让 AI 替他**发明题目**或**硬显著**，本 skill **不会**用，请改用其他 skill。

---

## 🔄 完整工作流（v2.0 核心）

```mermaid
flowchart TD
    S0[Step 0 · 题目能否继续做<br/>制度/法律/伦理/平台规则/数据可获得性冲突?] -->|是| STOP[直接封存<br/>不进入显著性流程]
    S0 -->|否| L[锁定不可动项<br/>研究问题 / 识别逻辑 / 基准模型]
    L --> D[诊断问题在哪<br/>X / Y / 样本 / 模型]
    D --> FIX[先固定三样<br/>识别 + 样本定义 + 固定 X 或 Y]
    FIX --> R1[第 1 轮 · 自然基准<br/>最自然的 Y/分母/FE/聚类]
    R1 --> R2[第 2 轮 · 调 Y 口径<br/>只动 Y<br/>ln(y+1)/比例/强度/人均/分母重构]
    R2 --> R3[第 3 轮 · 调 X 口径<br/>只动 X<br/>处理组/实施年/treat·post/强度口径]
    R3 --> R4[第 4 轮 · 调模型<br/>只动模型<br/>TWFE/交错DID/堆叠DID/事件窗口/聚类/FE结构]
    R4 --> R5[第 5 轮 · 事件研究↔主回归一致性<br/>图与表冲突 → 立即停]
    R5 --> R6[第 6 轮 · 上控制变量<br/>低缺失→强含义→完整<br/>每加一层审计样本缩多少]
    R6 --> R7[第 7 轮 · 显著是否为假<br/>极短窗口/小样本/高维FE前/奇怪分母]
    R7 --> SEL[最终选用规则<br/>选最能解释·最能辩护·最稳定·最可复现的]
    SEL --> RET[进正文; 其他显著结果最多做补充]
    R1 -.不显著不是错.-> DOC[失败留痕进审计目录]
    R2 -.样本塌缩.-> STOP2[停]
    R4 -.FE吸收DID.-> STOP2
    R7 -.只在极端口径显著.-> STOP2
    STOP2 --> DOC
```

每一轮的硬纪律是**绝对不能同时换 X + Y + 模型 + 窗口**。

---

## 🪜 7 轮顺序详解（每轮只看一件事）

### 第 0 步：题目能否继续做
- 制度 / 法律 / 伦理 / 平台规则是否直接冲突
- 数据是否还能继续获得（数据源改版 / 论文撤稿 / 数据库下架）
- 题目本身是否有**不可调和的硬约束**（如法规明确禁止讨论）
- **任一项为真 → 直接封存，不进入显著性流程**

### 第 1 轮：自然基准
- 用最自然的 Y、最自然的分母、最自然的 FE、最自然的聚类
- **先看方向、量级、样本量、处理组数量**
- **不追显著**
- 这一轮跑出来不显著，**不是错**

### 第 2 轮：调 Y 口径（只动 Y）
- 候选：`ln(y+1)` / `asinh(y)` / 比例 / 强度 / 人均 / 分母重构
- **这一轮禁止动 X、动识别、动模型**
- 目标：让 Y 的量纲更稳、更合理，让残差更接近正态

### 第 3 轮：调 X 口径（只动 X）
- 候选：处理组定义 / 实施年份 / `treat × post` 构造 / 强度口径
- **这一轮禁止动 Y、动识别、动模型**
- 目标：让处理变量更可识别、更可辩护

### 第 4 轮：调模型（只动模型）
- 候选：TWFE / 交错 DID / 堆叠 DID / 事件窗口 / 聚类层级 / FE 结构
- **这一轮禁止动 X、动 Y、动样本**
- 目标：让**识别表达方式**更合适，不是为了瞎试显著

### 第 5 轮：事件研究 + 主回归绑一起审
- **主回归正，事件研究不能乱**
- 事件窗口**不能为显著乱缩**
- **图和表冲突，先停下来解释**
- 这一轮是给前面几轮"识别可信度"的最终体检

### 第 6 轮：上控制变量（每加一层都审计样本）
- 顺序：低缺失控制 → 经济含义强的控制 → 完整控制
- 每一层都审计：样本缩了多少？是谁被缩掉了？
- **控制变量主要是调精度，不该用来硬翻符号**

### 第 7 轮：审计"显著是不是假的"
- 是不是只在**极短窗口**才显著？
- 是不是只在**小样本**才显著？
- 是不是只在**高维 FE 吸收前**才显著？
- 是不是只在**某个奇怪分母**下才显著？
- 任一项打钩 → **回到上一轮调整，而不是继续往下扩**

### 最终选用规则
- **不是最显著的进正文**
- 选**最能解释、最能辩护、最稳定、最可复现**的那条
- 其他显著结果**最多进稳健性附录 / 异质性表**
- 进正文的那一条必须能在 Spec-log 里写出"为什么是它，不是别的"

---

## 🧠 核心立场

本 skill **故意做窄**：只在你已经有明确经济学问题时才有用。

### 1. 先定问题（Step 0）
研究问题**已经知道**，且**仍然能继续做**。本 skill **不**用来凭空发明题目。

### 2. 先动当前指标，再考虑同概念替代
先在**当前变量**周围做变换；若都不行，再考虑同概念的替代指标。

### 3. 解释优先于显著
更显著的系数**不等于更好的结果**——如果它让经济学解释变弱。

### 4. 失败必须留痕
失败的规格**不是要被藏起来的噪声**，而是经验记录的一部分。

### 5. 一轮只动一样
同时动 X+Y+模型+窗口是最常见的 p-hacking 路径，本 skill 直接禁止。

### 6. Stata-friendly
面向真实经济学项目：`do` 文件、log、输出表必须可复现、可投稿。

---

## 🧰 仓库结构

```text
xianzhu-skill/
├── SKILL.md                       ← Codex 加载的协议本体
├── README.md                      ← 本文件
├── LICENSE                        ← MIT
├── CHANGELOG.md
├── .gitignore
├── agents/
│   └── openai.yaml                ← Codex 侧子代理配置示例
├── evals/
│   └── evals.json                 ← 触发式测试用例
└── references/
    ├── attempt-ladder.md          ← 显著性尝试梯子
    ├── case-examples.md           ← 真实案例
    ├── method-menu.md             ← 容许的估计方法菜单
    ├── spec-log-template.md       ← 规格日志最小可追溯要求
    ├── stata-collab-template.md   ← Stata 协作输出规范
    ├── stata-output-conventions.md← Stata 文件 / do / 输出约定
    ├── stop-rules.md              ← 停机规则（七轮顺序里贯穿的硬条件）
    └── transform-playbook.md      ← 变换剧本（log / asinh / winsor / 离散化 ...）
```

---

## 📚 参考模块清单

仓库里以下文件供 Codex 在判定时直接读取：

- `references/stop-rules.md` ⭐  
  七轮顺序里"该停"的硬条件；遇到任意一条立即封存或回滚。
- `references/attempt-ladder.md` ⭐  
  显著性尝试的层级清单，从"要不要救这条结果"开始判断。
- `references/stata-collab-template.md` ⭐  
  Stata 协作输出硬约定：脚本名 / 变量名 / 输出路径 / 审计目录。
- `references/method-menu.md`  
  一份精简的"可用估计方法"清单。
- `references/transform-playbook.md`  
  `log` / `ln(x+1)` / `asinh` / winsor / 离散化 / 重表达 的实操指南。
- `references/spec-log-template.md`  
  规格搜索日志的最小可追溯要求（每一轮都写一行）。
- `references/stata-output-conventions.md`  
  命名、`do` 文件拆分、输出 / log 约定。
- `references/case-examples.md`  
  真实案例，展示工作流在实际项目里的样子。

⭐ 标注 = Step 0 / 7 轮顺序的"必读"配套。

---

## 🚫 这个 skill **不**做什么

本 skill **不会**：

- 凭空发明研究问题
- 替代文献综述
- 没有依据地伪造替代变量
- 隐藏不显著的结果
- 把"更显著"等同于"更可发表"
- 把激进 p-hacking 当成有效经验研究
- 容忍 X+Y+模型+窗口 同轮共改
- 让"通过了停机规则"成为继续试的通行证

---

## ✅ 推荐输出形态

跑完一轮 `xianzhu-skill`，通常会得到：

- 模式识别（`Step 0` 通过 / `Mode A` / `Mode B`）
- 锁定的"不可动项"清单
- 诊断结论（问题在 X / Y / 样本 / 模型）
- 7 轮顺序里哪几轮跑过、跑出什么样
- 当前推荐的保留规格
- **进正文的规格必须是"最能解释、最能辩护、最稳定、最可复现"的那一条**
- 被否定的规格与其被否理由
- 完整的实验日志（可复现）

---

## 🎯 适用对象

本仓库特别适合：

- 中国实证经济学科研工作者（中文为主）
- 经验类论文修改阶段
- Stata 优先的回归项目
- 在用 Codex、需要"有边界的显著性搜索"协议的论文复现流程
- 已经装好 [`cc-codex-collaboration-skill`](https://github.com/yichaoli921/cc-codex-collaboration-skill) 的双代理用户

---

## 🏷️ 推荐 GitHub About

仓库描述：

**中文**：中国实证经济学科研向 Codex skill：把"试到显著"变成可审计、可解释、有边界的规格搜索。

**English**：A Codex skill for systematic and auditable specification search in empirical economics.

Topics：
`codex-skill` `economics` `empirical-economics` `stata` `regression` `specification-search` `research-workflow` `经验研究` `显著性`

---

## 🔗 与本系列其他 skill 的关系

| Skill | 关系 | 仓库 |
|---|---|---|
| **cc-codex-collaboration-skill** | 上游：双代理协作协议（脚手架） | https://github.com/yichaoli921/cc-codex-collaboration-skill |
| **xianzhu-skill（本仓库）** | 方法类：显著性 / 规格搜索 | https://github.com/yichaoli921/xianzhu-skill |

推荐组合：先装 `cc-codex-collaboration-skill` 建协作环境，再装本 skill 做方法学任务。

---

## 📄 License

MIT
