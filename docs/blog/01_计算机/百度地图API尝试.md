# 序  
使用百度地图API开发应用，首先需要申请一个百度地图API的密钥，点击[百度开发者平台](http://lbsyun.baidu.com/apiconsole/center#/home)进入到百度开发者平台，然后按照下列顺序一步一步的完成申请：  
1.创建一个应用，填写应用名称  
2.选择应用类型，这里使用的是浏览器端应用。  
相信很多人会不清楚自己应该选用什么类型，这里我给大家一些解释：  
服务端API为开发者提供http/https接口，即开发者通过http/https形式发起检索请求，获取返回json或xml格式的检索数据，不提供任何的图像和效果显示；  
浏览器端API是由JavaScript语言编写的应用程序接口，可以帮助你在网站中构建功能丰富、交互性强的地图应用，支持PC端和移动端基于浏览器的地图应用开发，且支持HTML5特性的地图开发。简单来说就是由js封装的接口，可以在html或js文件中直接调用。  
3.OK，解释完毕，下面勾选启用服务，一般全部打勾，以备不时之需。  
4.接下来就是白名单，什么是白名单？就是只有在白名单列表上的地址才可以访问这个服务。在本地开发调试的时候，直接填个*就可以了，表示任何地址都可以访问，正式上线的时候，做一下限制就好了。接下来点击提交就ok了。  
那么接下来如何调用百度地图API？  
1.在html文件的头部`<head>`里面写入以下代码：  
`<script type="text/javascript" src="http://api.map.baidu.com/api?v=接口版本&ak=你的密钥（ak）"></script>`  
2.在html中创建容器，如下所示：  
`<div style="width:680px;height:550px;border:#ccc solid 1px;font-size:12px" id="map"></div>`  
3.创建地图实例：在js文件或者html文件的`<script></script>`之间写入以下代码（示例）：  

```javascript
var map = new BMap.Map('map');//创建Map实例
var point = new BMap.Point(116,28);//创建坐标点
map.centerAndZoom(point,16);//初始化地图，并且设置中心点和地图缩放级别
```
OK，这就是使用百度地图API的第一步，如果有些地方不懂的，请多看一下html和js的知识补充一下就好了。  

## 1.使用百度地图api实现地点标注

在.html文件或者.js文件下的`<script>`和`</script>`块之间插入下列代码  

```javascript
var point = new BMap.Point(116.038624,28.695625);//创建标注的位置
var marker = new BMap.Marker(point);//创建标注
map.addOverlay(marker);// 将标注添加到地图中
var opts = {
   width : 200,     // 信息窗口宽度
   height: 100,     // 信息窗口高度
   title : "南昌工程学院" , // 信息窗口标题
   enableMessage:true,//设置允许信息窗发送短息
}
var infoWindow = new BMap.InfoWindow("地址：江西省南昌市南昌工程学院", opts);  // 创建信息窗口对象 
//标注增加鼠标事件监听
 marker.addEventListener("click", function(){          
 map.openInfoWindow(infoWindow,point); //开启信息窗口
 });
```
## 2.使用百度地图api实现输入框自动提示

前端html代码

```html
<html>
<head>
<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=你的ak"></script>
</head>
<body>
<h4 >增加地点</h4>
<form>
<div >
<label>地点名字</label>
<input id="suggestId" name="sendAddress" type="text"  placeholder="输入地名/建筑物名字">
</div>
<button type="submit" >Submit</button>
</form>
</body>
<script>
//这里插入js代码
</script>
</html>
```
js代码  
```javascript
loadMapAutocomplete("suggestId");

function loadMapAutocomplete(suggestId) {
    var checkValue;
    Ac = new BMap.Autocomplete( //建立一个自动完成的对象
        {
            "input": suggestId,
    });
    Ac.addEventListener("onconfirm", function(e) { //鼠标点击下拉列表后的事件
        var _value = e.item.value;
        checkValue = _value.province + _value.city + _value.district + _value.street + _value.business;
        //注：$("#")是jQuery用法，作用是根据标签id获取值或赋值，使用前请先引用jQuery
        $("#suggestId").val(checkValue);//点击地名后把地点信息显示到输入框上
    });
}
```
