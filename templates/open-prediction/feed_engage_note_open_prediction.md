# feed_engage_note_open_prediction：深度互动内容开放预测

## 一句话定义

不提供候选 Note，直接预测未来深度互动内容的主题与核心实体集合。

## 出题条件

- 对应选择题 `feed_engage_note` 已具备完整供给；
- gold 来自真实深度互动 Note 的完整内容，可稳定归一主题与核心实体；
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测，不截断 gold。

## 题目怎么问

题面不展示未来 Note 或候选，要求预测内容主题和核心实体，不预测 note_id、标题原文或作者。

## 可用信息

可以使用历史内容主题、深度互动和近期需求变化；曝光、点击只能作为弱线索。

## 回答与评分

预测项固定为 `behavior_type="feed_deep_engagement"`、`target_type="note_topic"`，包含 `target` 和 `entities`。隐藏评分使用 `content_dimensions=["target", "entities"]`；Judge 分别给 `target_score`、`entities_score`，Python 取平均后计算 soft F1。最终分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。

## 可选变化

可以限定内容域，不预测具体 Note 或深度互动动作种类。

## 种子提示词

请根据题面给出的 <user behaviors>，预测指定未来窗口内用户更可能深度互动的内容主题与核心实体。题面不提供候选，本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个去重预测。

只输出一个 JSON 对象；每项固定填写 `behavior_type="feed_deep_engagement"`、`target_type="note_topic"`，`target` 填主题，`entities` 填核心实体或空数组；`evidence` 引用真实 B 编号；`uncertainty` 说明证据缺口。


下面只演示格式,示例内容不是本题答案:

```
{
  "predictions": [
    {"behavior_type": "feed_deep_engagement", "target_type": "note", "target": "笔记内容方向"},
    {"behavior_type": "feed_deep_engagement", "target_type": "note", "target": "笔记内容方向"}
  ],
  "evidence": [
    {"event_id": "B000123", "quote": "用户在可见行为中与该目标相关的最短原文"}
  ],
  "uncertainty": "证据不足之处,无明显缺口时填空字符串"
}
```

## 不出题/不合格

- Note 内容不可读或无法形成主题 gold；
- 泄漏 future Note 或候选；
- 要求精确 note_id、标题或作者；
- 与 `feed_engage_note` 选择题同时保留。
