# creator_next_publish_topic_open_prediction：未来发布主题开放预测

## 一句话定义

不提供候选，直接预测用户本人在未来窗口内会发布的主题或内容类目集合。

## 出题条件

- 对应选择题 `creator_next_publish_topic` 已经具备完整供给。
- future 发布 Note 有完整标题和正文，并能在同一层级归一出主题或类目；收齐全部目标并去重。
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；不得据此截断 gold。
- 主题层级混乱或只有他人内容互动时不出题。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来作品或候选主题。要求一次返回完整发布主题集合，不复原未来标题或正文。

## 可用信息

主要使用用户本人此前发布的作品和明确创作线索；互动过的他人内容只能作为弱信号，不能直接当作创作计划。

## 回答与评分

答案只输出一个 JSON 对象：

```json
{
  "predictions": [
    {"behavior_type": "publish_note", "target_type": "topic/category", "target": "发布主题", "entities": ["核心实体"]}
  ],
  "evidence": [{"event_id": "B编号", "quote": "最短可见原文"}],
  "uncertainty": "证据不足之处"
}
```

隐藏评分配置使用 `metric=semantic_set_f1`，保存完整 `gold_items`，并令
`match_dimensions=["publish_note", "target_type", "target", "entities"]`、`content_dimensions=["target", "entities"]`。
Judge 分别给出 `target_score` 和 `entities_score`，Python 取平均作为单项匹配分；总分为
`semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。无效 JSON、混入他人行为或凭空加入热点时记 0 分。

## 可选变化

可以限定主题或类目层级；不预测标题原文、正文、发布时间或创作动机。

## 种子提示词

请根据题面给出的 <user behaviors>，预测用户本人在指定未来窗口内更可能发布哪些内容主题或类目。题面不提供候选，不要求复原未来作品原文。

本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；`predictions` 必须去重。

只输出一个 JSON 对象，不要在 JSON 前后添加分析、标题或 Markdown。各字段含义如下：

- `predictions`：预测结果列表，可以有 1 到题面上限项；
- `behavior_type`：固定填写 `publish_note`；
- `target_type`：按题面要求填写 `topic` 或 `category`；
- `target`：填写发布主题或内容类目；
- `entities`：填写主题中能明确点名的地点、品牌、商品、人物或作品，没有时写空数组；
- `evidence`：列出真实 B 编号和对应行为中的最短原文；
- `uncertainty`：用一句话说明主题层级或创作延续性上的证据缺口，没有明显缺口时写空字符串。

下面只演示格式，示例内容不是本题答案：

```
{
  "predictions": [
    {"behavior_type": "publish_note", "target_type": "topic", "target": "低脂家常菜", "entities": ["空气炸锅"]}
  ],
  "evidence": [
    {"event_id": "B000231", "quote": "用户此前连续发布低脂食谱内容"}
  ],
  "uncertainty": "能判断创作方向，但无法确定具体菜品"
}
```

不得补写外部热点，不得预测标题原文、正文、发布时间或创作动机。实际作答时，`evidence` 只能引用题面真实存在的 B 编号，不得照抄示例编号或证据。

## 不出题/不合格

- future 作品不可读或主题层级不稳定；
- 只有他人内容互动，没有本人创作线索；
- 题面泄漏未来作品；
- 与 `creator_next_publish_topic` 选择题同时保留。
