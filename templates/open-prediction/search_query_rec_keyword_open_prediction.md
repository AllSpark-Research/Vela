# search_query_rec_keyword_open_prediction：未来搜索词开放预测

## 一句话定义

不提供候选，直接预测未来窗口内用户可能真实搜索的 query 集合。

## 出题条件

- 对应选择题 `search_query_rec_keyword` 已具备完整供给；
- gold 收齐窗口内真实搜索事件中的有效 `搜索词` 后去重；
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测，不截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来 query 或候选，要求直接返回可能搜索的 query。

## 可用信息

可以使用历史搜索、近期需求、内容点击和商品行为；内容浏览或系统推荐不能写成已经搜索。

## 回答与评分

预测项固定为 `behavior_type="search"`、`target_type="keyword"`，`target` 填 query。隐藏评分使用 `semantic_set_f1`、`content_dimensions=["target"]`。最终分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。

## 可选变化

可以限定搜索领域，但不预测排序、搜索结果或点击后的行为。

## 种子提示词

请根据题面给出的 <user behaviors>，预测指定未来窗口内用户更可能实际搜索的 query。题面不提供候选，本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个去重预测。

只输出一个 JSON 对象；每项固定填写 `behavior_type="search"`、`target_type="keyword"`，`target` 填预测 query；`evidence` 引用真实 B 编号；`uncertainty` 说明证据缺口。不得照抄或编造未来记录。


下面只演示格式,示例内容不是本题答案:

```
{
  "predictions": [
    {"behavior_type": "search", "target_type": "keyword", "target": "搜索词"},
    {"behavior_type": "search", "target_type": "keyword", "target": "搜索词"}
  ],
  "evidence": [
    {"event_id": "B000123", "quote": "用户在可见行为中与该目标相关的最短原文"}
  ],
  "uncertainty": "证据不足之处,无明显缺口时填空字符串"
}
```

## 不出题/不合格

- future 没有有效搜索词 gold；
- 泄漏 future query 或候选；
- 混入类目、结果 Note 或点击行为；
- 与 `search_query_rec_keyword` 选择题同时保留。
