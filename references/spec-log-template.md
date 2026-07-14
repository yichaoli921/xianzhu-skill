# 实验日志模板

每一轮规格搜索都要至少记录以下字段。

## 单次尝试记录

- `spec_id`
- `mode`
  - `identify_then_search` 或 `direct_experiment`
- `focus_side`
  - `x` / `y` / `both`
- `base_variable`
- `transformation`
- `model`
- `sample_rule`
- `controls`
- `fixed_effects`
- `coefficient`
- `std_error`
- `p_value`
- `n_obs`
- `direction`
- `keep_or_drop`
- `reason`

## 推荐结论模板

### 推荐保留

- 规格：
- 变量定义：
- 变换：
- 模型：
- 原因：
  - 经济含义
  - 与故事一致性
  - 方向稳定性
  - 显著性表现

### 不推荐保留

- 规格：
- 原因：
  - 经济含义变弱
  - 模型不匹配
  - 样本过窄
  - 结果不稳定
  - 只是显著但不可解释

## 汇报原则

- 不要只报“显著的”
- 失败规格至少按类别汇总
- 若存在多个可用规格，优先推荐最可解释的一项
