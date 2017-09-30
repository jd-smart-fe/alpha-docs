# 智能冰箱控制 H5 开发接入指南

## 1. 前端开发准备

请仔细阅读 [Alpha 前端开发准备](https://smartdev.jd.com/docCenterDoc/view/2/103/95500001#topMaoDot)，而除 “前端开发准备” 外，文档中其余部分不需要看。

冰箱控制 H5 会接入【京东微联】和【京东新鲜Go】两款 APP。

冰箱控制各种参数请在 【京东微联·开发者中心】>【产品管理】>【参数设置】中查看。

## 2. 数据流

PAD 端有任何数据更新，都会通过长连接上报给云端，云端再推送给 APP。

![fridge dataflow][1] 

## 3. 开始开发

### 3.1 引入 JSSDK

请将 `jdsmart-fridge-sdk.js` 文件放在其他 js 文件之前引入。

```html
<body>
  <div>Example</div>
</body>
<!-- jssdk 文件首先引入 -->
<script src="assets/jdsmart-fridge-sdk.js" charset="utf-8"></script>
<!-- 业务 js 文件放在后面 -->
<script src="assets/app.js" charset="utf-8"></script>
```

### 3.2 调用 JSSDK

#### 3.2.1 JSSDK初始化 JDSMART.ready

jssdk 初始化完毕后会调用 ready 方法。

ready 方法会接受一个回调函数作为参数。

```html
<script src="assets/jdsmart-fridge-sdk.js" charset="utf-8"></script>
<script type="text/javascript">
  window.JDSMART.ready(function() {
    alert('inited');
  });
</script>
```
#### 3.2.2 获取设备快照 JDSMART.getSnapshot

getSnapshot 方法可以获取到设备保存在云端的最新快照。

```html
<script src="assets/jdsmart-fridge-sdk.js" charset="utf-8"></script>
<script type="text/javascript">
  window.JDSMART.ready(function() {
    /**
    * @param id 设备id（feed_id/guid），对于冰箱控制项目来说，改参数直接填 null 即可。
    * @param callback 成功后的回调
    *
    * 因为【京东微联】和【京东新鲜Go】两款 APP 返回的 data 类型有所不同
    * 因此必须要加上下述代码进行字符串转换。
    */
    window.JDSMART.getSnapshot(null, function(rowData) {
      var data = typeof rowData === 'string'
        ? JSON.parse(rowData)
        : rowData;
      });
      data.result = typeof data.result === 'string'
        ? JSON.parse(data.result)
        : data.result;

      console.log(data);
  });
</script>
```

返回结果参考:

Success:

```js
{"status":0, "error":null, "result":{
  "streams": [     // 请求返回的设备快照
    {
      "stream_id": "smartmode",		
          "current_value": "0"      
    },{
      "stream_id": "fritemp",		
          "current_value": "3"      
    }
]}}
```

Error:

```js
{"status": !0, "error": {"errorCode":10000, "errorInfo":""}, "result" :{}}
```

#### 3.2.3 下发控制命令 JDSMART.controlDevice

```html
<script src="assets/jdsmart-fridge-sdk.js" charset="utf-8"></script>
<script type="text/javascript">
  window.JDSMART.ready(function() {
    /**
    * 开启智能模式
    */
    var command = getCommand({ smartmode: 1 });
    window.smartSDK.controlDevice(command, function(rowData) {
      var data = typeof rowData === 'string'
        ? JSON.parse(rowData)
        : rowData;
      });
      data.result = typeof data.result === 'string'
        ? JSON.parse(data.result)
        : data.result;

      console.log(data);
    });
  });

  function getCommand(obj) {
    var command = [];
    for (var item in obj) {
      if (obj.hasOwnProperty(item)) {
        var stream = {};
        stream.stream_id = item;
        stream.current_value = obj[item];
        command.push(stream);
      } else {
        return null;
      }
    }
    return command;
  }
</script>
```

返回结果：
请参考 `JDSMART.getSnapshot` 的返回结果

#### 3.2.3 设置【京东微联】抬头副标题 JDSMART.setNavigationConfig;

```html
<script src="assets/jdsmart-fridge-sdk.js" charset="utf-8"></script>
<script type="text/javascript">
  window.JDSMART.ready(function() {
    /**
     * 参数 config 配置:
     *{
     *  showBack: false,  // 返回按钮，false 是隐藏，true 是显示
     *  showShare: false,
     *  showMore: false,  // 更多按钮
     *  color: “#998877”, // title 栏颜色
     *  showOnline:false  // 在线提示 true-不在线，false-在线
     * }
     */
    var config  = { showOnline: true }  // 注意: true-不在线
    window.smartSDK.setNavigationConfig(config);  // 设置副标题为设备不在线
  });
</script>
```

### 3.3 从长连接获取数据推送

APP 客户端会跟云端建立长连接，而每次云端通过长连接向客户端推送数据的时候，APP 客户端便会调用 H5 的 `window.onRecive` 方法，并将长连接数据作为参数传入。

```html
<script type="text/javascript">
  window.onReceive = function(rowData) {
    var data = typeof rowData === 'string'
      ? JSON.parse(rowData)
      : rowData;
    if (!data) return;

    data.body.data.result = typeof data.body.data.result === 'string'
      ? JSON.parse(data.body.data.result)
      : data.body.data.result;

    console.log(data);
  };
</script>
```

返回结果参考:
```json
{
  "header": {
    "code": "snapshot",
    "seq": 58556,
    "session_id": "14c560db-02fe-46f3-b18d-985c88888888",
    "version": "1.0"
  },
  "body": {
    "feed_id": 150588719588888888,
    "data": {
      "result": {
        "streams": [
          {
            "stream_id": "fastfrimode",
            "current_value": "0"
          },
          {
            "stream_id": "smartmode",
            "current_value": "0"
          }
        ],
        "status": "1",
        "digest": "724462958"
      },
      "status": 0
    },
    "time": 1506073611142
  }
}
```

[1]: https://raw.githubusercontent.com/jd-smart-fe/alpha-docs/master/assets/fridge_dataflow.png



