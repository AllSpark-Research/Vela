# purchase_intent_ecom_goods_open_prediction：站内购买商品开放预测

## 一句话定义

不提供候选，直接预测未来窗口内真实站内购买的商品集合。

## 出题条件

- 对应选择题 `purchase_intent_ecom_goods` 已具备完整供给；
- gold 只取 `购买计数 > 0` 且 `商品名` 非空的电商记录，并收齐窗口内全部商品后去重；
- 模型最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个预测，不据此截断 gold。

## 题目怎么问

题面只给预测窗口与冻结历史，不展示未来商品或候选，要求直接返回真实购买商品集合。

## 可用信息

可以使用历史购买、浏览、收藏、加购、搜索和内容行为；弱动作只能作为线索，不能写成已经购买。

## 回答与评分

答案只输出一个 JSON 对象，`predictions` 单项格式为 `{"behavior_type":"ecom_purchase","target_type":"goods","target":"商品"}`。隐藏评分使用 `metric=semantic_set_f1`、`content_dimensions=["target"]`；总分为 `semantic_set_f1 * ({{OPEN_PREDICTION_CONTENT_WEIGHT}} + {{OPEN_PREDICTION_SUPPORT_WEIGHT}} * support_score)`。

## 可选变化

可以限定商品域，但不预测品牌、类目、店铺、金额或购买件数。

## 种子提示词

请根据题面给出的 <user behaviors>，预测指定未来窗口内更可能真实购买的商品。题面不提供候选；只有站内电商 `购买计数 > 0` 算真实购买。本题最多输出 {{OPEN_PREDICTION_TASK_LIMIT}} 个去重预测。

只输出一个 JSON 对象：`predictions` 中每项固定填写 `behavior_type="ecom_purchase"`、`target_type="goods"`，`target` 填商品；`evidence` 引用真实 B 编号及最短原文；`uncertainty` 说明证据缺口。不得凭空编造商品，也不得输出品牌、类目、店铺、金额或购买动机。

## 不出题/不合格

- 没有非空商品 gold；
- 泄漏 future 或候选；
- 混入其他目标层或把弱动作写成购买；
- 与 `purchase_intent_ecom_goods` 选择题同时保留。
