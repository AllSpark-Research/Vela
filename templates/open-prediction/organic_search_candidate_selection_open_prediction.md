# organic_search_candidate_selection_open_prediction：自然搜索点击开放预测

## 一句话定义

不提供未来 query 或候选结果，直接预测用户下一次自然搜索点击内容的主题与核心实体集合。

## 出题条件

- 对应选择题 `organic_search_candidate_selection` 已经具备完整同会话供给。
- gold 只来自同一自然搜索中的真实点击 Note；Note 有完整标题和正文，并可归一为稳定内容语义。
- 收齐同次搜索全部可读点击语义并去重。
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；不得据此截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来 query、曝光结果或候选 Note。要求一次返回该次搜索真实点击内容的完整语义集合，不预测 note_id。

## 可用信息

可以使用历史搜索、点击内容、近期需求和内容口味。曝光未点击不等于不喜欢，点击也不等于满意、深度兴趣或购买。

## 回答与评分

答案只输出一个 JSON 对象：

```json
{
  "predictions": [
    {"behavior_type": "organic_search_click", "target_type": "note_topic", "target": "点击内容主题", "entities": ["核心实体"]}
  ],
  "evidence": [{"event_id": "B编号", "quote": "最短可见原文"}],
  "uncertainty": "证据不足之处"
}
```

`target` 写用户可能点击内容所回答的主题，`entities` 只写内容中能明确点名的对象。例如：

- 海边昆虫防护内容：`target="海边昆虫识别与防护"`，`entities=["某海滨公园", "某昆虫"]`；
- 短剧观看信息：`target="短剧观看渠道"`，`entities=["题面历史中出现的短剧名"]`；
- 手机选购测评：`target="手机选购与测评"`，`entities=["某品牌手机", "题面历史中出现的具体型号"]`。

没有稳定核心实体时，`entities` 必须写空数组；不能为了提高命中率凭空编造 query、地点、品牌、商品、人物或作品。

隐藏评分配置使用 `metric=semantic_set_f1`，保存完整 `gold_items`，并令
`match_dimensions=["organic_search_click", "note_topic", "target", "entities"]`、
`content_dimensions=["target", "entities"]`。Judge 分别给出 `target_score` 和 `entities_score`，Python 按
`0.5 * target_score + 0.5 * entities_score` 计算单项匹配分；只答对主题或实体中的一个时得 0.5。
总分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。无效 JSON、混入
广告点击或其他行为类型时记 0 分。

## 可选变化

可以限定内容域；不预测未来 query 原文、候选排名、精确 note_id 或点击后的深度行为。

## 种子提示词

请根据题面给出的 <user behaviors>，预测用户下一次自然搜索中更可能点击哪些内容主题。题面不会提供未来 query、搜索结果候选或排名，不要求预测具体 Note。

先结合历史搜索主题、搜索后的点击内容、近期需求变化和内容口味，判断用户在下一次自然搜索中可能选择的内容方向。曝光未点击只表示该次没有选择，不等于不喜欢；点击也不等于满意、深度兴趣或购买。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测；`predictions` 必须去重。

只输出一个 JSON 对象，不要在 JSON 前后添加分析、标题或 Markdown。各字段含义如下：

- `predictions`：预测结果列表，可以有 1 到题面上限项；
- `behavior_type`：固定填写 `organic_search_click`；
- `target_type`：固定填写 `note_topic`；
- `target`：填写用户下一次自然搜索中可能点击内容所回答的主题；
- `entities`：填写内容中能明确点名的地点、品牌、商品、人物或作品，没有稳定实体时写空数组；
- `evidence`：列出真实 B 编号和对应行为中的最短原文；
- `uncertainty`：用一句话说明搜索需求或点击方向上的证据缺口，没有明显缺口时写空字符串。

下面只演示格式，示例内容不是本题答案：

```
{
  "predictions": [
    {"behavior_type": "organic_search_click", "target_type": "note_topic", "target": "海边昆虫识别与防护", "entities": ["某昆虫"]}
  ],
  "evidence": [
    {"event_id": "B000066", "quote": "用户搜索过海蚊子并点击相关内容"}
  ],
  "uncertainty": "无法确定用户下一次搜索的具体地点"
}
```

`target` 写点击内容所回答的主题，例如“海边昆虫识别与防护”“短剧观看渠道”或“手机选购与测评”。不得补写未来 query、Note 或排名，也不得凭空编造实体。实际作答时，`evidence` 只能引用题面真实存在的 B 编号，不得照抄示例编号或证据。

## 不出题/不合格

- 没有完整同会话点击供给，或 Note 不可读；
- 题面泄漏未来 query、候选或排名；
- 要求预测精确 note_id；
- 与 `organic_search_candidate_selection` 选择题同时保留。
