# Leaflet.heat 热力图
[Bug提交地址,如有问题请提交问题说明,我们会及时响应](https://github.com/ParnDeedlit/WebClient-Leaflet/issues)

---
!> 版本号: 5885ec0ec2442876b67725c64dc537cb12c52c73

> github 最后更新时间:2016-09-30 18:49:32

这个是`官方插件`,应该没什么大问题.一个小巧轻便的热力图Leaflet插件 [Leaflet plugin](http://leafletjs.com) .该插件是从 [simpleheat 插件](https://github.com/mourner/simpleheat) 演化而来.

## 示例

- [10,000 points &rarr;](http://leaflet.github.io/Leaflet.heat/demo)
- [动态添加点](http://leaflet.github.io/Leaflet.heat/demo/draw.html)


## 基本使用

```js
var heat = L.heatLayer([
	[50.5, 30.5, 0.2], // lat, lng, intensity
	[50.6, 30.4, 0.5],
	...
], {radius: 25}).addTo(map);
```

> 传统第三方引入

> 直接将源码dist目录下的 `leaflet-heat.js` 文件引入即可

```html
<script src="leaflet-heat.js"></script>
```

> 中地内部版本引入方式

> `include`标签属性中添加一个`heater`属性即可,还可以添加`heater,cluster,ant-path`多个标签

```html
<script include="heater" src="../../libs/zondyclient/include-leaflet.js"></script>
```

### 完整示例
#### js 代码
``` html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">

<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <title>leaflet显示</title>
  <script include="jquery" src="../../libs/zondyclient/include-lib.js"></script>
  <script include="leaflet,heater" src="../../libs/zondyclient/include-leaflet.js"></script>
  <style>
    #mapid {
      height: 600px
    }
  </style>
</head>

<body>
  <div id="mapid"></div>
  <script>
    var mymap = L.map('mapid').setView([34.25, 108.95], 10);
    L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=' +
      'sk.eyJ1IjoiY2hlbmdkYWRhIiwiYSI6ImNqZDFjaGo0ZjFzcnoyeG54enoxdnNuZHUifQ.hTWXXBUQ0wdGeuDF3GWeUw', {
        attribution: '<a href="#">MapBox</a>',
        maxZoom: 18,
        id: 'mapbox.streets'
      }).addTo(mymap);
    var param = {
      precision: 12 //有效范围 1- 12
    };
    $.getJSON('../../data/dataview/heater-data.json', function(data) {
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
    });
  </script>
</body>

</html>
```

#### 数据说明
~~~ json
{"agg":[{"count":1,"lat":39.76900493726134,"lon":116.26720380038023},{"count":1,"lat":37.46901288628578,"lon":120.39447974413633},{"count":1,"lat":35.52809486165643,"lon":117.61087987571955},{"count":1,"lat":34.71326993778348,"lon":113.95750969648361},{"count":1,"lat":34.099193941801786,"lon":114.12192977964878},{"count":1,"lat":34.17148584499955,"lon":112.84415390342474},{"count":1,"lat":33.94231991842389,"lon":112.86824986338615},{"count":1,"lat":29.822511915117502,"lon":121.50082994252443},{"count":1,"lat":33.15971989184618,"lon":114.73482493311167},{"count":1,"lat":33.66343690082431,"lon":113.43467976897955},{"count":1,"lat":23.284247908741236,"lon":114.46811981499195}]}
~~~

## 参考说明

#### L.heatLayer(latlngs, options)

1 **latlngs** `point数组`

2 **options** 配置参数

	2 **minOpacity** - 热力图的最小透明度
  - **maxZoom** - 这个表示热力聚类效果最大到多少级,如果6级后就热力聚类,则会出现6级以后没有聚类热力的效果显示,但是还是会有红色的热力点, `maxZoom`默认与地图的最大显示级别一样,`一般最好不要设置这个选项`
  - **max** - 最大点密度,与	`[50.6, 30.4, 0.5]`中第三个数0.5的值一一对应, `1.0` by default
  - **radius** - 热力点聚类半径, `25` by default
  - **blur** - 模糊程度, `15` by default
  - **gradient** - 颜色梯度设置, e.g. `{0.4: 'blue', 0.65: 'lime', 1: 'red'}`

输入数组中的每个点都可以是这样的类型`[50.5, 30.5, 0.5]`,第三个表示密度值
or a [Leaflet LatLng object](http://leafletjs.com/reference.html#latlng).

除非`max`选项倍特别设定,否则第三个参数最好是0.0-1.0之间;如果你的数据是1-200的分布,则密度值`max`设置为200即可.

> 老实说,密度值 `max` 这个值设不设其实对显示效果的影响并不大.


#### Methods

- **setOptions(options)**: 设置新的参数并重新绘制
- **addLatLng(latlng)**: 添加一个新点并重新绘制
- **setLatLngs(latlngs)**: 重新设置整个图层并重新绘制
- **redraw()**: 重绘热力图

## 最新修改日志 Changelog

### 0.2.0 &mdash; Oct 26, 2015

- Fixed intensity to work properly with `max` option.
- Fixed zoom animation on Leaflet 1.0 beta 2.
- Fixed tiles and point intensity in demos.
