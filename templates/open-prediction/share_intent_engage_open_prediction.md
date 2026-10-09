# share_intent_engage_open_prediction：分享目标开放预测

## 一句话定义

不提供候选，直接预测未来窗口内会被用户分享或转发的内容主题与核心实体集合。

## 出题条件

- 对应选择题 `share_intent_engage` 已经具备完整供给。
- gold 只来自真实分享动作；对应 Note 有完整标题和正文，并可归一为稳定主题或核心实体。
- 收齐完整窗口内全部可读语义目标并去重。
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；不得据此截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来互动 Note 或候选，不要求复原 note_id、标题或正文。要求一次返回完整分享目标集合。

## 可用信息

可以使用历史分享、内容口味和传播倾向。浏览、长看、点赞、收藏或评论都不能直接升级为分享。

## 回答与评分

答案只输出一个 JSON 对象：

```json
{
  "predictions": [
    {"behavior_type": "share", "target_type": "note_topic", "target": "内容主题", "entities": ["核心实体"]}
  ],
  "evidence": [{"event_id": "B编号", "quote": "最短可见原文"}],
  "uncertainty": "证据不足之处"
}
```

隐藏评分配置使用 `metric=semantic_set_f1`，保存完整 `gold_items`，并令
`match_dimensions=["share", "note_topic", "target", "entities"]`、`content_dimensions=["target", "entities"]`。
Judge 分别给出 `target_score` 和 `entities_score`，Python 取平均作为单项匹配分，再按 soft F1 评分，
总分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。无效 JSON、混入其他动作或动作升级时记 0 分。

## 可选变化

可以限定某个内容域；不预测分享对象、渠道、话术、次数或精确 Note。

## 种子提示词

请根据题面给出的 <user behaviors>，预测用户在指定未来窗口内更可能分享或转发哪些内容主题。题面不提供候选，不要求预测具体 Note 或分享对象。

判断时优先使用历史分享行为、内容口味和传播倾向，区分“喜欢看”与“愿意转发”。浏览、长看、点赞、收藏或评论都只能作为线索，不能直接当作分享。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；`predictions` 必须去重。

只输出一个 JSON 对象，不要在 JSON 前后添加分析、标题或 Markdown。各字段含义如下：

- `predictions`：预测结果列表，可以有 1 到题面上限项；
- `behavior_type`：固定填写 `share`；
- `target_type`：固定填写 `note_topic`；
- `target`：填写可能被分享或转发的内容主题；
- `entities`：填写主题中能明确点名的地点、品牌、商品、人物或作品，没有稳定实体时写空数组；
- `evidence`：列出真实 B 编号和对应行为中的最短原文；
- `uncertainty`：用一句话说明分享倾向或内容主题上的证据缺口，没有明显缺口时写空字符串。

下面只演示格式，示例内容不是本题答案：

```
{
  "predictions": [
    {"behavior_type": "share", "target_type": "note_topic", "target": "周末亲子出游攻略", "entities": ["某博物馆"]}
  ],
  "evidence": [
    {"event_id": "B000155", "quote": "用户曾多次分享亲子出游和实用攻略内容"}
  ],
  "uncertainty": "能判断分享主题，但无法确定具体目的地"
}
```

不得凭空编造 Note、作者、分享对象、渠道或经历。实际作答时，`evidence` 只能引用题面真实存在的 B 编号，不得照抄示例编号或证据。

## 不出题/不合格

- 没有真实分享 gold，或 Note 不可读；
- 要求预测精确 Note、分享对象或渠道；
- 把赞藏评升级为分享；
- 与 `share_intent_engage` 选择题同时保留。
