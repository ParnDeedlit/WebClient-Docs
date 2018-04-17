## 添加度量
前面的例子告诉我们每个桶里面的文档数量，这很有用。 但通常，我们的应用需要提供更复杂的文档度量。 例如，每种颜色汽车的平均价格是多少？

为了获取更多信息，我们需要告诉 Elasticsearch 使用哪个字段，计算何种度量。 这需要将度量 嵌套 在桶内， 度量会基于桶内的文档计算统计结果。

让我们继续为汽车的例子加入 average 平均度量：
``` javascript
GET /cars/transactions/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": {
               "avg": {
                  "field": "price"
               }
            }
         }
      }
   }
}
```

1. 为度量新增 aggs 层。

2. 为度量指定名字： avg_price 。

3. 最后，为 price 字段定义 avg 度量。

正如所见，我们用前面的例子加入了新的 aggs 层。这个新的聚合层让我们可以将 avg 度量嵌套置于 terms 桶内。实际上，这就为每个颜色生成了平均价格。

正如 颜色 的例子，我们需要给度量起一个名字（ avg_price ）这样可以稍后根据名字获取它的值。最后，我们指定度量本身（ avg ）以及我们想要计算平均值的字段（ price ）：

```json
{
...
   "aggregations": {
      "colors": {
         "buckets": [
            {
               "key": "red",
               "doc_count": 4,
               "avg_price": {
                  "value": 32500
               }
            },
            {
               "key": "blue",
               "doc_count": 2,
               "avg_price": {
                  "value": 20000
               }
            },
            {
               "key": "green",
               "doc_count": 2,
               "avg_price": {
                  "value": 21000
               }
            }
         ]
      }
   }
...
}
```

响应中的新字段 avg_price 。

尽管响应只发生很小改变，实际上我们获得的数据是增长了。之前，我们知道有四辆红色的车，现在，红色车的平均价格是 $32，500 美元。这个信息可以直接显示在报表或者图形中。

---
## 嵌套桶
在我们使用不同的嵌套方案时，聚合的力量才能真正得以显现。 在前例中，我们已经看到如何将一个度量嵌入桶中，它的功能已经十分强大了。

但真正令人激动的分析来自于将桶嵌套进 另外一个桶 所能得到的结果。 现在，我们想知道每个颜色的汽车制造商的分布：

``` javascript
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": {
               "avg": {
                  "field": "price"
               }
            },
            "make": {
                "terms": {
                    "field": "make"
                }
            }
         }
      }
   }
}
```


1. 注意前例中的 avg_price 度量仍然保持原位。

2. 另一个聚合 make 被加入到了 color 颜色桶中。

3. 这个聚合是 terms 桶，它会为每个汽车制造商生成唯一的桶。

> 这里发生了一些有趣的事。 首先，我们可能会观察到之前例子中的 avg_price 度量完全没有变化，还在原来的位置。 一个聚合的每个 层级 都可以有多个度量或桶， avg_price 度量告诉我们每种颜色汽车的平均价格。它与其他的桶和度量相互独立。

这对我们的应用非常重要，因为这里面有很多相互关联，但又完全不同的度量需要收集。聚合使我们能够用一次数据请求获得所有的这些信息。

!> 另外一件值得注意的重要事情是我们新增的这个 make 聚合，它是一个 terms 桶（嵌套在 colors 、 terms 桶内）。这意味着它 会为数据集中的每个唯一组合生成（ color 、 make ）元组。

让我们看看返回的响应（为了简单我们只显示部分结果）：
``` json
{
...
   "aggregations": {
      "colors": {
         "buckets": [
            {
               "key": "red",
               "doc_count": 4,
               "make": {
                  "buckets": [
                     {
                        "key": "honda",
                        "doc_count": 3
                     },
                     {
                        "key": "bmw",
                        "doc_count": 1
                     }
                  ]
               },
               "avg_price": {
                  "value": 32500
               }
            },

...
}
```

1. 正如期望的那样，新的聚合嵌入在每个颜色桶中。

2. 现在我们看见按不同制造商分解的每种颜色下车辆信息。

3. 最终，我们看到前例中的 avg_price 度量仍然维持不变。

** 响应结果告诉我们以下几点： **

1. 红色车有四辆。
2. 红色车的平均售价是 $32，500 美元。
3. 其中三辆是 Honda 本田制造，一辆是 BMW 宝马制造。