---
title: WebFlux中mongo操作-Aggregation
comments: true
categories: [程序猿]
tags: [Java, Reactive, 事务, mongo, 数据库, Aggregation]
toc: true
visible: show
indexing: true
date: 2020-01-17 15:12:53
---
<br>
<!--more-->

switch

```java
ConditionalOperators.Switch.CaseOperator cond = ConditionalOperators.Switch.CaseOperator.when(
                BooleanOperators.And.and(
                        ComparisonOperators.Eq.valueOf("channelBillStatus1").equalToValue("已结算"),
                        ComparisonOperators.Eq.valueOf("channelBillStatus2").equalToValue("已结算")
                )
        ).then("已结清");

        Aggregation aggregation = Aggregation.newAggregation(
                Aggregation.project("channelBillStatus1", "channelBillStatus2")
                        .and(ConditionalOperators.switchCases(cond).defaultTo("未结清")).as("channelBillStatus")
        );
        
        reactiveMongoTemplate.aggregate(aggregation, PlatformBillItem.class, PlatformBillBo.class);
```

lookup及id类型转换

```java
//把_id转成String并赋值给id
        Aggregation.project("internalId", "name", "isAvailable", "isCanAdd", "fitGender", "fitAge", "fitMaritalStatus", "price", "sortNo", "createdAt")
          			//如果需要把String转Object使用ConvertOperators.ToObjectId.toObjectId()
                .and(ConvertOperators.ToString.toString("$_id")).as("id");
        //用当前表的id值去匹配chn_section表的sectionId字段值，并把结果存入chnSections数组
        Aggregation.lookup("chn_section", "id", "sectionId", "chnSections");
        //如有需要，把chnSections数组拆出来，chnSections数组有几个元素，当前这条数据就会被拆成多少条，chnSections值会变成元素值而不再是原来的数组
        //如果chnSections数组无值，默认会丢弃这条数据，如果要保留设置preserveNullAndEmptyArrays=true
        Aggregation.unwind("chnSection", true);
        //只输出这些字段
        Aggregation.project("internalId", "name", "isAvailable", "isCanAdd", "fitGender", "fitAge", "fitMaritalStatus", "price", "sortNo", "createdAt", "chnSections");
        reactiveMongoTemplate.aggregate(aggregation, PlatformBillItem.class, PlatformBillBo.class);
```

如果lookup时，如果要对匹配的数据进行筛选（参考链接：https://stackoverflow.com/questions/51107626/spring-data-mongodb-lookup-with-pipeline-aggregation）

```java
//原始mongo
//{
//   $lookup:
//     {
//       from: <collection to join>,
//       let: { <var_1>: <expression>, …, <var_n>: <expression> },
//       pipeline: [ <pipeline to execute on the collection to join> ],
//       as: <output array field>
//     }
//}
//自定义一个AggregationOperation类
public class CustomProjectAggregationOperation implements AggregationOperation {
    private String jsonOperation;

    public CustomProjectAggregationOperation(String jsonOperation) {
        this.jsonOperation = jsonOperation;
    }

    @Override
    public Document toDocument(AggregationOperationContext aggregationOperationContext) {
        return aggregationOperationContext.getMappedObject(Document.parse(jsonOperation));
    }
}

private static String getJsonOperation() {
        return "{" +
                "    $lookup: " +
                "    {" +
                "        from: 'chn_set_meal'," +
                "        let: {" +
                "            id: { $toString: '$_id' }" +
                "        }," +
                "        pipeline: [" +
                "            {" +
                "                $match: " +
                "                {" +
                "                    $expr: " +
                "                    {" +
                "                        $and: " +
                "                        [" +
                "                            {" +
                "                                $eq: ['$setMealId', '$$id']" +
                "                            }," +
                "                            {" +
                "                                $eq: ['$cooperationState', '合作中']" +
                "                            }" +
                "                        ]" +
                "                    }" +
                "                }" +
                "            }," +
                "            {" +
                "                $project: {" +
                "                    channelId: 1," +
                "                    channelName: 1" +
                "                    cooperationState: 1" +
                "                }" +
                "            }" +
                "        ]," +
                "        as: 'channels'" +
                "    }" +
                "}}";
    }

AggregationOperation aggregationOperation = new CustomProjectAggregationOperation(getJsonOperation());
        return reactiveMongoTemplate.aggregate(Aggregation.newAggregation(aggregationOperation), SetMeal.class, SetMealListBo.class);
```

group

```java
//背景：查询交易表，订单和交易一对多
Aggregation.group("orderNo")
  //单一组的金额汇总
  .sum("amount").as("totalAmount")
  //组的最后一个订单号
  .last("orderNo").as("orderNo")
  //组里数据条数
  .count().as("tradeCount")
  //把一组数据里每条数据的状态放到一个statuses数组里
  .addToSet("status").as("statuses")
  //把一组数据里的一些字段信息重新组装成一个对象放到billItems的对象数组里
  .push(new BasicDBObject("tradeContent", "$tradeContent")
        .append("tradeNo", "$tradeNo")
        .append("amount", "$amount")
       ).as("billItems");
```