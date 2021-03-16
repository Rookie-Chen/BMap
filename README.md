

# 百度地图API学习

* 需求：类似地图找房功能

* 最终demo效果

  * 行政区域级别

    ![](C:\Users\Administrator\Desktop\demo\img\1.jpg)

  * 行政区域下一级

     ![](C:\Users\Administrator\Desktop\demo\img\2.jpg)

  * 最后一级详情

    ![](C:\Users\Administrator\Desktop\demo\img\3.jpg)

* 准备工作：申请百度开放平台ak

* 初始化地图

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1,user-scalable=no">
    <title>百度api</title>
    <script type="text/javascript" src="//api.map.baidu.com/api?v=2.0&ak=自己申请的ak"></script>
  </head>
  <body>
    <div id="myMap"></div>
  </body>
  </html>
  ```

  ```javascript
  function createMap() {
    var map = new BMap.Map("myMap");//在百度地图容器中创建一个地图
    var point = new BMap.Point(106.557165, 29.570997);//定义一个中心点坐标,城市经纬度
    map.centerAndZoom(point, 12);//设定地图的中心点和坐标以及显示级别*/
    map.setCurrentCity("重庆"); // 设置地图显示的城市 此项是必须设置的
    window.map = map;//将map变量存储在全局  
  }
  //地图事件设置函数： 
  function setMapEvent() {
    map.enableScrollWheelZoom();//启用地图滚轮放大缩小
    map.enableKeyboard();//启用键盘上下左右键移动地图
    map.setMinZoom(10); // 设置地图最小缩放级别
    //map.setMaxZoom(10); // 设置地图最大缩放级别
  }
  //地图控件添加函数：
  function addMapControl() {
    //向地图中添加缩放控件
    var ctrl_nav = new BMap.NavigationControl({ anchor: BMAP_ANCHOR_TOP_RIGHT, type: BMAP_NAVIGATION_CONTROL_SMALL });
    map.addControl(ctrl_nav);
    //向地图中添加缩略图控件  
    var ctrl_ove = new BMap.OverviewMapControl({ anchor: BMAP_ANCHOR_BOTTOM_RIGHT, isOpen: 1 });
    map.addControl(ctrl_ove);
    //向地图中添加比例尺控件
    var ctrl_sca = new BMap.ScaleControl({ anchor: BMAP_ANCHOR_BOTTOM_LEFT });
    map.addControl(ctrl_sca);
  }
  // 初始化地图
  function initMap() {
    createMap();
    setMapEvent();
    addMapControl();
  }
  ```

* 行政区域绘制：

  ```javascript
  function getBoundary(name) {
    var bdary = new BMap.Boundary();
    bdary.get(name, function (rs) {       //获取行政区域
      map.clearOverlays();        //清除地图覆盖物       
      var count = rs.boundaries.length; //行政区域的点有多少个
      if (count === 0) {
        alert('未能获取当前输入行政区域');
        return;
      }
      for (var i = 0; i < count; i++) {
        var ply = new BMap.Polygon(rs.boundaries[i], {
            strokeWeight: 2,
            strokeColor: "#ff0000" 
        }); //建立多边形覆盖物
        map.addOverlay(ply);  //添加覆盖物
      }
    });
  }
  ```

* 绘制行政区域下一级（目前百度API只支持行政区域的边界绘制，下一级的需要自己给出边界点集合）

  * 可以使用百度地图插件DrawingManager
  * 插件api：http://api.map.baidu.com/library/DrawingManager/1.4/docs/symbols/BMapLib.DrawingManager.html#constructor

  ```javascript
  /* 根据坐标绘制 */
  function defineDraw(arr) {
    var count = arr.length;
    for (var i = 0; i < count; i++) {
      var ply = new BMap.Polygon(arr[i], {
        strokeWeight: 3,
        strokeColor: "#ef1b1c",
        fillColor: "rgba(239,27,28,0.2)",
      });
      map.addOverlay(ply);
    }
  }
  ```

* 最后一级就比较简单了，只需要提供坐标点

* 添加覆盖物

  ```javascript
  // 添加label覆盖物
  function addLabel(arr) {
    for (var i = 0; i < arr.length; i++) {
      var txt = arr[i].title;
      var num = arr[i].num;
      var pintx = arr[i].point.split('|')[0];
      var pinty = arr[i].point.split('|')[1];
      var str =
        `<p class="title">${txt}</p>
        <p class="num">${num}个小区</p>
        `
      var myLabel = new BMap.Label(str, {
        position: new BMap.Point(pintx, pinty),
        enableMassClear: false  // 确保使用 map.clearOverlays()时label覆盖物不会被清除
      });
      map.addOverlay(myLabel);
    }
  }
  ```

  