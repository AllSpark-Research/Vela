# feed_engage_brand_open_prediction：深度互动品牌开放预测

## 一句话定义

不提供候选，直接预测未来深度互动内容所关联的品牌集合。

## 出题条件

- 对应选择题 `feed_engage_brand` 已具备完整供给；
- gold 只取真实深度互动 feed 记录中的非空 `品牌账号名`，收齐后去重；
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测，不截断 gold。

## 题目怎么问

题面不展示未来品牌或候选，要求直接返回深度互动内容关联品牌集合。

## 可用信息

可以使用历史品牌互动、内容主题、搜索和电商行为；一般品牌兴趣只能作为线索。

## 回答与评分

预测项固定为 `behavior_type="feed_deep_engagement"`、`target_type="brand"`，`target` 填品牌；隐藏评分使用 `semantic_set_f1` 和 `content_dimensions=["target"]`。最终分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。

## 可选变化

可以限定品牌所属内容域，不预测商品、类目、Note 或具体互动动作。

## 种子提示词

请根据题面给出的 <user behaviors>，预测指定未来窗口内用户更可能深度互动的内容所关联的品牌。题面不提供候选，本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个去重预测。

只输出一个 JSON 对象；每项固定填写 `behavior_type="feed_deep_engagement"`、`target_type="brand"`，`target` 填品牌；`evidence` 引用真实 B 编号；`uncertainty` 说明证据缺口。不得凭空编造品牌。


下面只演示格式,示例内容不是本题答案:

```
{
  "predictions": [
    {"behavior_type": "feed_deep_engagement", "target_type": "brand", "target": "品牌账号名"},
    {"behavior_type": "feed_deep_engagement", "target_type": "brand", "target": "品牌账号名"}
  ],
  "evidence": [
    {"event_id": "B000123", "quote": "用户在可见行为中与该目标相关的最短原文"}
  ],
  "uncertainty": "证据不足之处,无明显缺口时填空字符串"
}
```

## 不出题/不合格

- future 品牌字段为空；
- 泄漏 future 或候选；
- 混入商品、类目或具体 Note；
- 与 `feed_engage_brand` 选择题同时保留。
