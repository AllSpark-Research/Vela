# feed_hide_open_prediction：不感兴趣开放预测

## 一句话定义

不提供候选，直接预测未来窗口内会被用户明确点击“不感兴趣”的内容主题与核心实体集合。

## 出题条件

- 对应选择题 `feed_hide` 已经具备完整供给。
- gold 只来自明确 hide 动作；对应 Note 有完整标题和正文，并能归一为稳定主题或核心实体。
- 收齐完整窗口内全部可读语义目标并去重。
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；不得据此截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来 Note 或候选，不要求预测 note_id。要求一次返回完整的负反馈内容语义集合。

## 可用信息

可以使用历史明确负反馈、重复内容疲劳和内容口味冲突。未点击、短停留、没有互动和一般低兴趣都不能写成“不感兴趣”。

## 回答与评分

答案只输出一个 JSON 对象：

```json
{
  "predictions": [
    {"behavior_type": "feed_hide", "target_type": "note_topic", "target": "内容主题", "entities": ["核心实体"]}
  ],
  "evidence": [{"event_id": "B编号", "quote": "最短可见原文"}],
  "uncertainty": "证据不足之处"
}
```

`target` 写内容主题，`entities` 只写该主题下能明确点名的具体对象。例如：

- 旅游避坑内容：`target="旅游避坑"`，`entities=["某旅游目的地", "某酒店"]`；
- 手机测评内容：`target="手机测评"`，`entities=["某品牌手机", "某手机品牌"]`；
- 游戏攻略内容：`target="游戏攻略"`，`entities=["某手游", "某游戏角色"]`。

没有稳定核心实体时，`entities` 必须写空数组；不能为了填字段凭空编造地点、品牌、商品、人物或作品。

隐藏评分配置使用 `metric=semantic_set_f1`，保存完整 `gold_items`，并令
`match_dimensions=["feed_hide", "note_topic", "target", "entities"]`、`content_dimensions=["target", "entities"]`。
Judge 分别给出 `target_score` 和 `entities_score`，Python 取平均作为单项匹配分；
总分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。无效 JSON、混入其他动作或把未互动当 hide 时记 0 分。

## 可选变化

可以限定某个内容域，但不预测 note_id、作者身份、负反馈原因或次数。

## 种子提示词

请根据题面给出的 <user behaviors>，预测用户在指定未来窗口内更可能对哪些内容主题明确点击“不感兴趣”。题面不提供候选，不要求预测具体 Note。

判断时优先使用历史明确负反馈、重复内容疲劳和持续出现的内容口味冲突。未点击、短停留、没有互动或一般低兴趣都不等于明确的“不感兴趣”。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；`predictions` 必须去重。

只输出一个 JSON 对象，不要在 JSON 前后添加分析、标题或 Markdown。各字段含义如下：

- `predictions`：预测结果列表，可以有 1 到题面上限项；
- `behavior_type`：固定填写 `feed_hide`；
- `target_type`：固定填写 `note_topic`；
- `target`：填写可能被明确负反馈的内容主题；
- `entities`：填写该主题下能明确点名的对象，没有稳定实体时写空数组；
- `evidence`：列出真实 B 编号和对应行为中的最短原文；
- `uncertainty`：用一句话说明负反馈证据不足或主题边界不清之处，没有明显缺口时写空字符串。

下面只演示格式，示例内容不是本题答案：

```
{
  "predictions": [
    {"behavior_type": "feed_hide", "target_type": "note_topic", "target": "夸张减肥偏方", "entities": []}
  ],
  "evidence": [
    {"event_id": "B000087", "quote": "用户曾多次对夸张减肥内容点击不感兴趣"}
  ],
  "uncertainty": "只能确定用户排斥夸张内容，无法确定具体商品"
}
```

`target` 写内容主题，例如“旅游避坑”“手机测评”或“游戏攻略”。`entities` 只写可见历史中能明确点名的地点、品牌、商品、人物或作品。不得凭空编造 Note、作者或经历。实际作答时，`evidence` 只能引用题面真实存在的 B 编号，不得照抄示例编号或证据。

## 不出题/不合格

- 没有明确 hide gold，或 Note 不可读；
- 要求预测精确 note_id 或标题原文；
- 把未点击、短停留或无互动当作 hide；
- 与 `feed_hide` 选择题同时保留。
