# Leaflet.heat 热力图

> els的热力图使用的就是数据可视化的热力图 [可视化-热力图](/leaflet/dataview/heater)

!> 由于elastic search在数据库实时库df_real中值保存最新的数据,会淘汰覆盖历史数据,因此这个接口查询的永远是数据库中最新的数据.因此如果要做历史数据跟踪的话,需要在前端使用数组来保留历史数据或者使用下面的历史库df_histroy进行查询.

!> elastic search的历史库df_history中的数据是记录的过去的历史记录,因此该表的查询速度会比上面的实时库df_real要慢

------
## 中地内部封装接口
### post请求地址 该接口查询的是实时库df_real###
> http://192.168.13.193:8080/zondybigdatawebapp/RealTimeLocationAgg

#### 请求参数

|参数|类型|说明|
|:---|:---|:---|
|precision|数字|数据精度级别,一般对应瓦片金字塔模型的第几级 `1 ~ 12 超出该级别范围的请求是无效的`|


#### 返回数据
- `agg` 聚合属性标签, `这是一个数组`
  + count 该precision精度级别下的geohash范围内的`聚类数量`
  + lon 该geohash中心点的`经度`
  + lat 该geohash中心点的`纬度`


------

#### 代码示例
~~~ javascript
var param = {
      precision: 12 //有效范围 1- 12
    };
$.ajax({
      url: "http://192.168.13.193:8080/zondybigdatawebapp/RealTimeLocationAgg",
      type: "POST",
      dataType: "json",
      data: JSON.stringify(param),
      contentType: "application/x-www-from-urlencoded;charset=utf-8",
      success: function(data) {
        var heatPoints = [];
        for (var i = 0; i < data.agg.length; i++) {
          var point = data.agg[i];
          var count = data.agg[i].count;
          var latlon = [point.lat, point.lon, point.count];
          heatPoints.push(latlon);
        }
        var heaterLayer = L.heatLayer(heatPoints, {
          radius: 10,
          minOpacity: 1.0
        }).addTo(mymap);
      },
      error: function(xhr, status, p3, p4) {}
  })
~~~

## Elastic Search原生接口
