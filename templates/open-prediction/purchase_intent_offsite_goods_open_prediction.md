# purchase_intent_offsite_goods_open_prediction：站外成交商品开放预测

## 一句话定义

不提供候选，直接预测未来窗口内真实站外成交的商品集合。

## 出题条件

- 对应选择题 `purchase_intent_offsite_goods` 已具备完整供给；
- gold 只取真实站外成交且商品字段非空的记录，收齐窗口内全部商品后去重；
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测，不据此截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来商品或候选，要求直接返回真实成交商品集合。

## 可用信息

可以使用历史成交、商品互动、搜索和内容行为；弱动作只能作为线索，不能写成已经成交。

## 回答与评分

答案只输出一个 JSON 对象，`predictions` 单项固定为 `behavior_type="external_purchase"`、`target_type="goods"`，`target` 填商品。隐藏评分使用 `semantic_set_f1` 和 `content_dimensions=["target"]`。最终分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。

## 可选变化

可以限定商品域，但不预测品牌、金额、平台、支付方式或动机。

## 种子提示词

请根据题面给出的 <user behaviors>，预测指定未来窗口内更可能发生站外真实成交的商品。题面不提供候选。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个去重预测。

只输出一个 JSON 对象：`predictions` 中固定填写 `behavior_type="external_purchase"`、`target_type="goods"`；`evidence` 引用真实 B 编号及最短原文；`uncertainty` 说明证据缺口。不得凭空编造商品或声称成交已经发生。


下面只演示格式,示例内容不是本题答案:

```
{
  "predictions": [
    {"behavior_type": "external_purchase", "target_type": "goods", "target": "商品名"},
    {"behavior_type": "external_purchase", "target_type": "goods", "target": "商品名"}
  ],
  "evidence": [
    {"event_id": "B000123", "quote": "用户在可见行为中与该目标相关的最短原文"}
  ],
  "uncertainty": "证据不足之处,无明显缺口时填空字符串"
}
```

## 不出题/不合格

- 没有非空商品 gold；
- 泄漏 future 或候选；
- 混入品牌、金额或其他行为；
- 与 `purchase_intent_offsite_goods` 选择题同时保留。
