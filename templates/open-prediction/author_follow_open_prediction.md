# author_follow_open_prediction：关注作者开放预测

## 一句话定义

不提供候选，只预测未来窗口内用户会关注的作者内容方向集合，不预测具体作者身份。

## 出题条件

- 对应选择题 `author_follow` 已经具备完整供给。
- 每位 future 作者都有完整转粉 Note 卡，并能从作者类目或作品内容归一出稳定内容方向；收齐全部方向并去重。
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；不得据此截断 gold。
- 只有作者 ID、无法得到稳定内容方向时不出题。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来作者卡或候选。要求一次返回可能关注的作者内容方向，不允许输出作者昵称、作者 ID 或具体账号。

## 可用信息

可以使用历史作者互动、内容口味和关注倾向。浏览、点赞或作者与兴趣相似都不等于已经关注。

## 回答与评分

答案只输出一个 JSON 对象：

```json
{
  "predictions": [
    {"behavior_type": "follow_author", "target_type": "author_topic", "target": "作者内容方向"}
  ],
  "evidence": [{"event_id": "B编号", "quote": "最短可见原文"}],
  "uncertainty": "证据不足之处"
}
```

隐藏评分配置使用 `metric=semantic_set_f1`，保存完整 `gold_items`，并令
`match_dimensions=["follow_author", "author_topic", "target"]`、`content_dimensions=["target"]`。内容方向完整对齐为 1，只命中明确上位方向
使用配置中的中间档分数；总分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。输出具体作者、无效 JSON 或混入其他行为时记 0 分。

## 可选变化

可以改变内容方向的类目粒度；不预测作者身份、关注时间、粉丝量或关注原因。

## 种子提示词

请根据题面给出的 <user behaviors>，预测用户在指定未来窗口内更可能关注哪些作者内容方向。题面不提供候选，也不要求猜具体作者。

先从可见历史中归纳用户反复消费、深度互动或持续搜索的内容方向，再判断这些方向是否足以支持“关注作者”，而不只是一次浏览或点赞。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；`predictions` 必须去重。

只输出一个 JSON 对象，不要在 JSON 前后添加分析、标题或 Markdown。各字段含义如下：

- `predictions`：预测结果列表，可以有 1 到 K 项；每项代表一种作者内容方向；
- `behavior_type`：固定填写 `follow_author`；
- `target_type`：固定填写 `author_topic`；
- `target`：填写具体内容方向，例如“家居收纳”“职场成长”“数码测评”；
- `evidence`：列出支持预测的可见行为，每项包含真实 B 编号和该条行为中的最短原文；
- `uncertainty`：用一句话说明仍不确定的方向或证据缺口，没有明显缺口时填写空字符串。

下面只演示格式，示例内容不是本题答案：

```
{
  "predictions": [
    {"behavior_type": "follow_author", "target_type": "author_topic", "target": "家居收纳"},
    {"behavior_type": "follow_author", "target_type": "author_topic", "target": "数码测评"}
  ],
  "evidence": [
    {"event_id": "B000123", "quote": "用户多次浏览并深度互动家居收纳内容"}
  ],
  "uncertainty": "可见历史能支持内容方向，但不能确定具体作者"
}
```

不得输出作者昵称、作者 ID、具体账号，也不得凭空编造作者。浏览、点赞或兴趣相似只能作为线索，不能写成用户已经关注。实际作答时，`evidence` 只能引用题面真实存在的 B 编号，不得照抄示例编号或示例证据；没有可靠证据时可以减少预测项，并在 `uncertainty` 中说明原因。

## 不出题/不合格

- future 作者没有可归一的内容方向；
- 只有作者 ID 或缺少完整转粉 Note；
- 题目要求预测具体作者、昵称或账号；
- 浏览、点赞被升级为关注；
- 与 `author_follow` 选择题同时保留。
