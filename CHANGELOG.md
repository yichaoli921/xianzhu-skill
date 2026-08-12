# Changelog

本 skill 的所有重要变更会在这里追加。

格式参考 [Keep a Changelog](https://keepachangelog.com/)，
版本号遵循 [SemVer](https://semver.org/)。

---

## [2.0.1] - 2026-08-12

### Added
- **Step 0 4 层硬纪律**：把"先判断题目还能不能做"拆成
  L1 数据可达 / L2 操作可达 / L3 制度·法律·平台规则 / L4 伦理·风险，
  并写入 `SKILL.md` 的 Step 0 段；顺序硬性、不可乱序/跳过/合并。
- **新增 `references/step0-checklist.md`**：
  4 层动作清单（含 Bash 命令模板）+ 中国常见官方源速查表 +
  `step0-audit.md` 输出模板。
- **L3 显式允许 `WebSearch` / `WebFetch`**：每条搜索必须包含**时间锚**，
  结果写进 `step0-audit.md`，附 5 个以内最权威来源链接。
- **失败语义分级**：
  L1/L2 失败 → 直接封存（写 `STOP_step0_L<1|2>.md` + `hard_fail`）；
  L3/L4 失败 → 询问用户、等明确意向。
- **Step 0 审计报告**：`output/审计/step0_<时间戳>.md`，
  30 天内可复用 L1/L2 但 L3 必跑。
- **重跑 Step 0 触发条件**：研究主题变了 / 数据源换了 /
  主要变量定义换了 / 法规/平台规则变化 / 距上次 > 90 天。

### Changed
- `SKILL.md` frontmatter `version: 2.0.0 → 2.0.1`，标题同步。
- `README.md` 徽章 `2.0.0 → 2.0.1`，"本版本更新亮点"段改成
  v2.0.1 增量 + v2.0.0 基线两段并列保留。
- `README.md` 仓库结构与参考模块清单加入 `step0-checklist.md`，标 ⭐。

### Notes
- v2.0.1 是**向前兼容补丁**：旧调用方式"先判断题目能不能做"
  依然成立，只是现在能**逐层机械跑完**。
- 与 [`cc-codex-collaboration-skill v2.0.0`](https://github.com/yichaoli921/cc-codex-collaboration-skill) 的
  命名顺序 / 状态词 / 文件夹架构保持一致。

---

## [2.0.0] - 2026-08-11

### Added
- **Step 0**：先判断题目是否还值得做（制度 / 法律 / 伦理 / 平台规则 /
  数据可获得性）— 任一冲突直接封存，不进入显著性流程。
- **数据口径突变 / 法律风险 / 平台规则审计**：在固定识别和样本前先审计。
- **模型层面的固定顺序**，尤其**事件研究一致性**：
  TWFE / 交错 DID / 堆叠 DID / 事件窗口 / 聚类层级 / FE 结构按顺序试；
  主回归与事件研究绑在一起审，图与表冲突立即停。
- **样本收缩与 FE 吸收问题诊断**：merge / 控制变量 / 清洗规则不能让样本塌缩；
  FE 不能把 DID 交互项吸收到看不见。
- **显式停机规则**：`references/stop-rules.md` —
  不再默认"无限试到显著"，机械式停机条件清楚列出。
- 三类新参考：
  - `references/attempt-ladder.md`（搜索顺序）
  - `references/stop-rules.md`（停机规则）
  - `references/stata-collab-template.md`（Stata 协作输出规范）

### Changed
- `SKILL.md` 标题改为"经济学显著 Skill 2.0.0"。
- `README.md` 重写为中文，含 mermaid 工作流图。

---

## [1.0.0] - 2026-08-11

### Added
- 协议本体 `SKILL.md`：触发短语清单、模式判定（A 识别前整理 / B 已有方程）、
  标准工作流（8 步）、输出格式、执行提醒、投稿实战规则、常见卡点与降级处理。
- 工程化：`README.md` / `LICENSE` (MIT) / `.gitignore` / 初始化 git 仓库。
- 参考模块：
  - `references/case-examples.md`
  - `references/method-menu.md`
  - `references/transform-playbook.md`
  - `references/spec-log-template.md`
  - `references/stata-output-conventions.md`
  - `references/mode-selection.md`
- 触发式测试用例：`evals/evals.json`。