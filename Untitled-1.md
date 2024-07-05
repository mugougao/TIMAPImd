# TIM API接口文档

## 文档说明
TIM API是一套JavaScript编程接口, 用于连接Web页面和51在线渲染平台。通过该API, 可以在网页上创建HTML5 UI元素, 并使其与云端渲染的3D场景进行双向交互。它兼容主流的前端框架如React、Vue、Angular等, 简化了开发工作。使用TIM API, 开发者可以开发集成高质量3D图形的Web应用, 无需强大的本地硬件。将渲染计算转到云端, 可以实现Web端实时高质量3D图形, 不占用本地设备资源。TIM API简化了开发集成云端渲染3D图形的Web应用的难度, 便于开发者构建网页3D交互体验。它消除了开发者在Web端集成高保真3D图形的障碍。

本文档中的API接口分为两个部分："基础API" 和 "行业API"，其中"基础API"属于"wdpapi" SDK，"行业API"部分属于"51timapi" SDK。

## 技术架构
三维场景通过51WORLD云端渲染, 将实时视频流推送到网页端, 并且能够实现前端到后端的交互同步。

像素流送以插件形式集成在引擎中, 插件会对主机服务器的图形流信息进行编码, 然后通过即时通信发送给位于接收端的浏览器和设备, 通过在高性能主机系统上运行渲染引擎,用户能在所有终端设备上享受到与主机相同的画质及所有功能。像素流插件在主机服务器上与客户端进行通信, 可以通过单台服务器运行, 也可以通过动态扩展并提供足够硬件的GPU云环境运行。

TIM API提供可在WEB端进行调用的方法, 以便用户从前端页面直接向51WORLD云渲染平台程序发送指令。同时通过注册函数监听51WORLD云渲染程序发送的事件, 用户再跟据事件类型, 在WEB端中对此类事件做出响应。
![avatar](https://wdpapi.51aes.com/img/architecture.09831275.svg)

## 开始第一个应用

### 一、初识渲染地址和渲染口令

三维场景通过云渲染服务, 采用实时视频流推送到网页端, 并且能够实现前端到后端的交互同步, 进入指定场景需要用到两个关键标识, 分别是渲染地址和渲染口令


渲染地址: 指部署云渲染服务(安装了51WDP并正常运行)机器的IP地址, 端口8889

***例: "http://172.31.19.235:8889/Renderers/Any/order"***


渲染口令: 由WDP数字孪生平台服务根据账号生成的一串唯一序列, 是开启渲染界面的凭证

***例: 7915cCb9***

### 二、获取渲染地址和渲染口令

1、部署WDP数字孪生平台

2、获取云渲染地址：通过部署服务器IP 和端口号8890，访问云渲染后台，获取渲染地址和渲染口令

***例： http:// 10.66.8.99:8890***
![_17200798037401.png](https://s2.loli.net/2024/07/04/WM7NX5z8Z92EAyS.png)

### 二、搭建第一个属于你的数字孪生应用

#### API 安装引用
 ```
npm install wdpapi
npm install 51timapi
```
``` javascript
import WdpApi from "wdpapi"
import TimApi from '51timapi';
```
**1. 创建DOM:**

需要先定义一个div, id="player", 作为渲染3D场景窗口的DOM节点.
``` javascript
<div id="player" style="width:100vw;height:100vh;"></div>
``` 
注意:不要在里面加任何自定义开发的元素, 特别是video; 也不要使用VUE自身的v-show.

**2. 创建云渲染对象:**

初始化一个实例
``` javascript
const App = new WdpApi({
    "id": "player", //渲染容器dom id
    "url": "http://172.31.19.235:8889/Renderers/Any/order", //[可选] 渲染服务地址
    "order": "7915cCb9", //[可选] 场景order
    "resolution": [1920,1080], //[可选] 场景输出分辨率
    "debugMode": "normal", //[可选] none:不打印日志, normal:普通日志
    "keyboard": { //[可选]
        "normal": false, //[可选] 键盘事件(wasd方向)开启关闭
        "func": false //[可选] 浏览器F1~F12功能键开启关闭
    }
});

// 实例化wdpapi对象
const App = new WdpApi(config);

// 绑定TimApi功能到wdpapi对象
const res = await App.Plugin.Install(TimApi);
console.log(res.result.id)

```

**3、启动云渲染:**
```javascript
await App.Renderer.Start().then(res => {
    if (res.success) {
        // to do
    }
}).catch(err => {
    console.log(err)
});
```
**4、事件监听**
```javascript
App.Renderer.RegisterSceneEvent([
    {
        name: 'OnWdpSceneIsReady', func: async function (res) {
            if(res.result.progress === 100) {
                // 场景加载完成
            }
        }
    },
    {
        name: 'OnEntityClicked', func: async function (res) {
            // Entity被点击事件回调; 包含数据信息与实体对象
        }
    }
])
```
更多的监听细节请浏览 事件监听

**5、使用API在场景中添加POI覆盖物**
```javascript
async function AddPOI() {
  const poi = new App.Poi({
    "location": [121.50007292,31.22579403,30],
    "poiStyle": {
      "markerNormalUrl": "http://wdpapi.51aes.com/doc-static/images/static/markerNormal.png",
      "markerActivateUrl": "http://wdpapi.51aes.com/doc-static/images/static/markerActive.png",
      "markerSize": [100,228],
      "labelBgImageUrl": "http://wdpapi.51aes.com/doc-static/images/static/LabelBg.png",
      "labelBgSize": [200,50],
      "labelBgOffset": [50,200], //x>0,y>0 向右,上偏移(x,y 单位:像素)
      "labelContent": [" 文本内容A","beff93ff","24"]
    },
    "entityName": "myName",
    "customId": "myId1",
    "customData": {
      "data": "myCustomData"
    }
  })

  // 向场景中添加实体
  const res = await App.Scene.Add(poi,{
    "calculateCoordZ": {  //[可选] 最高优先级
      "coordZRef": "surface",//surface:表面;ground:地面;altitude:海拔
      "coordZOffset": 50 //高度(单位:米)
    }
  });
}
```

**6、完整代码**
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://wdpapi.51aes.com/sdk/wdpApi.min.js"></script>
    <title">Hello 51WORLD!</title>
</head>
<body>
    <div id="player" style="width:100vw;height:100vh;"></div>
    <button class="btn" onclick="AddPOI()">添加点</button>

<script>
    const App = new WdpApi({
        "id": "player", //渲染容器dom id
        "url": "http://172.31.19.235:8889/Renderers/Any/order", //[可选] 渲染服务地址
        "order": "b96285A3", //[可选] 场景order
        "resolution": [1920,1080], //[可选] 场景输出分辨率
        "debugMode": "normal", //[可选] none:不打印日志, normal:普通日志
        "keyboard": { //[可选]
            "normal": false, //[可选] 键盘事件(wasd方向)开启关闭
            "func": false //[可选] 浏览器F1~F12功能键开启关闭
        }
    });

    async function start() {
        await App.Renderer.Start().then(res => {
            if (res.success) {
                App.Renderer.RegisterEvent([
                    {
                        name: 'OnWdpSceneIsReady', func: async function (res) {
                            if(res.result.progress === 100) {
                                // 场景加载完成
                            }
                        }
                    }
                ])
            }
        })
    }
    start()

    async function AddPOI() {
        const poi = new App.Poi({
            "location": [1054.44748524,-1255.58078392,61],
            "poiStyle": {
                "markerNormalUrl": "http://wdpapi.51aes.com/doc-static/images/static/markerNormal.png",
                "markerActivateUrl": "http://wdpapi.51aes.com/doc-static/images/static/markerActive.png",
                "markerSize": [100,228],
                "labelBgImageUrl": "http://wdpapi.51aes.com/doc-static/images/static/LabelBg.png",
                "labelBgSize": [200,50],
                "labelBgOffset": [50,200], //x>0,y>0 向右,上偏移(x,y 单位:像素)
                "labelContent": [" 文本内容A","beff93ff","24"]
            },
            "entityName": "myName",
            "customId": "myId1",
            "customData": {
                "data": "myCustomData"
            }
        })

        // 向场景中添加实体
        const res = await App.Scene.Add(poi,{
            "calculateCoordZ": {  //[可选] 最高优先级
                "coordZRef": "surface",//surface:表面;ground:地面;altitude:海拔
                "coordZOffset": 50 //高度(单位:米)
            }
        });
    }
</script>
</body>
</html>
```

## 基础API

### 事件监听

#### 云渲染事件

```javascript
App.Renderer.RegisterEvent([
    {
        name: 'onStopedRenderCloud', func: function (res) {
            // io client disconnect
            // 渲染服务中断 todo
        }
    },
    {
        name: 'onVideoReady', func: function (res) {
            // 视频流链接成功 todo
        }
    }
])
```

#### 场景事件

```javascript
App.Renderer.RegisterSceneEvent([
  {
    name: 'OnWdpSceneIsReady', func: async function (res) {
      // { "event_name": "OnWdpSceneIsReady", "result": { "progress": 100 } }
      if(res.result.progress === 100) {
          // 场景加载完成
      }
    }
  },
  {
    name: 'OnWdpSceneChanged', func: async function (res) {
      // 实体对象操作后回调；
      // res.result --> {added[object]，updated[object]，removed[object]}
    }
  },
  {
    name: 'OnMouseEnterEntity', func: async function (res) {
      // 鼠标滑入实体事件回调; 包含数据信息与实体对象
    }
  },
  {
    name: 'OnMouseOutEntity', func: async function (res) {
      // 鼠标滑出实体事件回调; 包含数据信息与实体对象
    }
  },
  {
    name: 'OnEntityClicked', func: async function (res) {
      // 覆盖物被点击事件回调; 包含数据信息与实体对象
    }
  },
  {
    name: 'OnWebJSEvent', func: async function (res) {
      // 接收widnow内嵌页面发送的数据
      // { "event_name": "OnWebJSEvent", "result": { "name": "自定义name", "args": "自定义数据" }}
    }
  },
  {
    name: 'MeasureResult', func: async function (res) {
      // 测量工具数据回调
    }
  },
  {
    name: 'OnMoveAlongPathEndEvent', func: async function (res) {
      // 覆盖物移动结束信息回调
    }
  },
  {
    name: 'OnCameraMotionStartEvent', func: async function (res) {
      // 相机运动开始信息回调
    }
  },
  {
    name: 'OnCameraMotionEndEvent', func: async function (res) {
      // 相机运动结束信息回调
    }
  },
  {
    name: 'PickPointEvent', func: async function (res) {
      // 取点工具取点数据回调
    }
  },
  {
    name: 'OnEntitySelectionChanged', func: async function (res) {
      // 实体被选取[框选]、数据回调
    }
  },
  {
    name: 'OnEntityNodeSelectionChanged', func: async function (res) {
      // 模型node选择状态变化数据回调
    }
  },
  {
     name: 'OnEntityReady', func: async function (res) {
         // 3DTilesEntity，WMSEntity，WMTSEntity 加载完成;
         // {success: true, message: '', result: { object: 对象, progress: 100 }}
      }
   },
  {
     name: 'OnCreateGeoLayerEvent', func: async function (res) {
         // 用于GisApi； WMS,WMTS 添加 报错回调
      }
   },
  {
     name: 'OnGeoLayerFeatureClicked', func: async function (res) {
         // 用于GisApi；点击实体回调
      }
   }
])
```

#### 场景事件注册/获取/删除

```javascript
App.Renderer.RegisterSceneEvents([
    {
        name: 'OnEntityClicked', func: async function (res) {
            console.log(res)
        }
    }
])
const res = await App.Renderer.GetRegisterSceneEvents();
console.log(res);
App.Renderer.UnRegisterSceneEvents(['OnEntityClicked'])
```


#### 开启/关闭场景事件发送

```javascript
await App.System.ToggleAPIEventChannel([
    {
        eventName: 'OnMouseEnterEntity', bopen: false // true, false
    },
    {
        eventName: 'OnMouseOutEntity', bopen: false // true, false
    }
]);
```

#### 设置Debug Log细度模式

```javascript
App.Debug.SetLogMode("high");  // none, normal, high
```

#### 键盘事件

```javascript
App.System.SetDefaultKeyboard(false);  // true, false  是否开启键盘事件
App.System.SetDefaultBrowserFunctionKeyboard(false);  // true, false
//开启键盘事件时是否开启浏览器F1-F12功能键(默认关闭)
```
#### 关闭云渲染

```javascript
App.Renderer.Stop()
```


### 场景相机
#### 设置相机初始状态
```javascript
async function CameraStart() {
  // 获取相机初始状态对象
  const { result: { CameraStart } } = await App.Scene.GetCameraStart();
  console.log(await CameraStart[0].Get());

  const jsondata = {
    "location": [121.48645883,31.23493598,1500],
    "locationLimit": [ //设置相机位置区域(至少三个坐标点,三角区域)[选填]
        [121.46690952,31.23621652],
        [121.47179065,31.22312573],
        [121.46505159,31.2407234]
     ], 
    "rotation": {
      "pitch": -35, //俯仰角, 参考(-90~0)
      "yaw": 0 //偏航角, 参考(-180~180)
    },
    "pitchLimit": [-90,0], //俯仰角, 参考(-90~0)
    "yawLimit": [-180,180], //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
    "viewDistanceLimit": [1,100000],
    "fieldOfView": 90, //相机视锥横向视角[0, 120]
    "controlMode": "RTS", //RTS (飞行模式); FPS (第一人称模式); TPS (第三人称模式)
  }

  // 更新相机初始状态
  const res = await CameraStart[0].Update(jsondata);
  console.log(res);

}
```
#### 设置相机模式
```javascript
async function SetCameraMode() {

  const res = await App.CameraControl.SetCameraMode('RTS');
  console.log(res);

  /*
    RTS (飞行模式)
    FPS (第一人称模式)
    TPS (第三人称模式)
  */
}
```
RTS

![_17200798037401.png](http://wdpapi-admin.51aes.com/public/content/f4698b70-af56-11ee-8838-2519b4aec160.webp)

FPS

![_17200798037401.png](http://wdpapi-admin.51aes.com/public/content/08f53fd0-af57-11ee-91cc-63f70a0848dc.webp)

TPS

![_17200798037401.png](http://wdpapi-admin.51aes.com/public/content/115988c0-af57-11ee-b3e3-1fc369141930.webp)

#### 获取相机位置
```javascript
async function GetCameraPose() {

  const res = await App.CameraControl.GetCameraPose();
  console.log(res);

}
```
#### 设置相机位置
```javascript
async function SetCameraPose() {
  const jsondata = {
    "location": [121.48537621,31.23840069,900],
    "rotation": {
      "pitch": -35, //俯仰角, 参考(-90~0)
      "yaw": 0 //偏航角, 参考(-180~180)
    },
    "flyTime": 1 //过渡时长(单位:秒)
  }

  const res = await App.CameraControl.SetCameraPose(jsondata);
  console.log(res);

}
```
#### 重置相机位置
```javascript
async function ResetCameraPose() {
  //相机初始状态效果若不理想，可以对相机初始状态进行调整

  const jsondata = {
    "state": 'Default', //Default: 相机初始状态; Last: 相机最后一次位置
    "flyTime": 1, //过渡时长(单位:秒)
  }

  const res = await App.CameraControl.ResetCameraPose(jsondata);
  console.log(res);

}
```
#### 获取相机Limit值
```javascript
async function GetCameraLimit() {

  const res = await App.CameraControl.GetCameraLimit();
  console.log(res);

}
```
#### 设置相机Limit值
```javascript
async function SetCameraLimit() {
  const jsondata = {
    "locationLimit": [ //设置相机位置区域(至少三个坐标点,三角区域)[选填]
      [121.47095414,31.22534628],
      [121.47264982,31.23423431],
      [121.49467492,31.24871524]
    ],
    "pitchLimit": [-80,0], //俯仰角; 取值范围[-90~0]
    "yawLimit": [-100,100], //偏航角; 取值范围[-180~180]
    "viewDistanceLimit": [100,1000] //相机距离实体距离范围
  }

  const res = await App.CameraControl.SetCameraLimit(jsondata);
  console.log(res);

}
```
#### 设置相机固定Limit值

```javascript
async function SetCameraLockLimit() {
  const jsondata = {
    "locationLimit": 100, // 鼠标拖动场景时, 相机前后左右移动[-100,100]范围(单位:米)
    "pitchLimit": 10, // 当前pitch(俯仰角)可移动的[-10,0]范围; 取值范围[0~90]
    "yawLimit": 20, // 当前yaw(偏航角)可移动的[-20,20]范围; 取值范围[0~180]
    "viewDistanceLimit": 100 // 鼠标滚轮时, 当前视距的[-100,100]范围(单位:米)
  }

  const res = await App.CameraControl.SetCameraLockLimit(jsondata);
  console.log(res);

}

```
#### 重制相机Limit值
```javascript
async function ResetCameraLimit() {

  const res = await App.CameraControl.ResetCameraLimit('Default');
  // Default: 相机初始Limit; Free: 无Limit限制

  console.log(res);

}
```
### 相机通用行为
#### 获取相机信息
```javascript
App.CameraControl.GetCameraInfo()
```
#### 更新相机
```javascript
const jsondata = {
    location: [121.48940131,31.25135281,500],
    locationLimit: [],  //设置相机位置区域(至少三个坐标点,三角区域)[选填]
    rotation: { pitch: -30, yaw: 0 },
    pitchLimit: [-90,0], // 俯仰角, 参考(-90~0)
    yawLimit: [-180,180], // 偏航角, 参考(-180~180)
    viewDistanceLimit: [500,600],
    fieldOfView: 90, // 相机视锥横向视角[0, 120]
    controlMode: 'RTS', // 控制模式; RTS (飞行模式); TPS (第三人称模式); FPS (第一人称模式)
    flyTime: 1 // 过渡时长(单位:秒)
}

await App.CameraControl.UpdateCamera(jsondata)
```
#### 相机移动
```javascript
App.CameraControl.Move({
    direction: 'right',
    velocity: 10
})
```
#### 相机旋转
```javascript
App.CameraControl.Rotate({
    direction: 'right',
    velocity: 10
})
```
#### 场景旋转
```javascript
App.CameraControl.Around({
    direction: 'clockwise',
    velocity: 10
})
```
#### 停止移动、旋转
```javascript
await App.CameraControl.Stop()
```

### 相机Step行为
#### 相机step移动
```javascript
App.CameraControl.CameraStepMove({
  moveDirection: 'E_Forward', // 移动方向
  step: 0.5, //速度的倍率 -1 ~ 1
  bContinuous: true //是否连续
});

/*
moveDirection: 移动方向
  E_Forward: 前;
  E_Backward: 后;
  E_Left: 左;
  E_Right: 右;
  E_Up: 上;
  E_Down: 下
*/
```
#### 相机step旋转
```javascript
App.CameraControl.CameraStepRotate({
  rotateDirection: 'E_Pitch', //E_Pitch: 俯仰角, E_Yaw: 偏航角
  step: 0.5, //速度的倍率 -1 ~ 1
  bContinuous: true //是否连续
});
```
#### 相机step缩放
```javascript
App.CameraControl.CameraStepZoom({
  step: 0.5, //速度的倍率 -1 ~ 1
  bContinuous: true //是否连续
});
```
#### 停止移动、旋转、缩放
```javascript
App.CameraControl.StopCameraStepUpdate();
```

####  相机聚焦到坐标点
```javascript
const jsondata = {
    "targetPosition": [121.48533665,31.24164246,30],
    "rotation": {
        "pitch": -30, //俯仰角, 参考(-90~0)
        "yaw": 0, //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
    },
    "distance": 500, //距离(单位:米)
    "flyTime": 1 //过渡时长(单位:秒)
}

await App.CameraControl.FlyTo(jsondata)
```

#### 相机聚焦到实体
```javascript
const jsondata = {
    "rotation": {
        "pitch": -30, //俯仰角, 参考(-90~0)
        "yaw": 0 //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
    },
    "distanceFactor": 0.4, //参数范围[0.1~1]; 实体占满屏幕百分比
    "flyTime": 1, //过渡时长(单位:秒)
    "entity": [pathObject] //实体对象
}
```
await App.CameraControl.Focus(jsondata)
#### 聚焦全部实体
```javascript
App.CameraControl.FocusToAll({
    types: ['Poi', 'Path'],  // 实体类型
    bFilterForExclude: false  // 是否排除指定的类型
})

// types (注意大小写)
/*
  RealTimeVideo  实时视频
  Window  窗口
  Poi  POI
  Particle  特效
  Text3D  3D文字
  Viewshed  可视域
  Path  路径
  Parabola  迁徙图
  Range  区域轮廓
  HeatMap  热力图
  ColumnarHeatMap  柱状热力图
  SpaceHeatMap  点云热力图
  RoadHeatMap  路径热力图
*/
```
#### 相机跟随实体
```javascript
const jsondata = {
    "followRotation": {
        "pitch": -20, //俯仰角, 参考(-90~0)
        "yaw": 20 //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
    },
    "useRelativeRotation": true, //相对实体进行offset
    "distance": 200,
    "entity": followParticle //实体对象
}

await App.CameraControl.Follow(jsondata)
```

#### 停止相机跟随
```javascript
await App.CameraControl.Stop()wqeqwd
```

### 相机漫游
#### 相机漫游
```javascript
const jsondata = {
  "frames": [
    {
      "location": [121.49067713,31.11991912,300],
      "rotation": {
        "pitch": -25, //俯仰角, 参考(-90~0)
        "yaw": 0 //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
      },
      "time": 20 //镜头到下一帧的时间(单位:秒)
    },
    {
      "location": [121.49060113,31.11431200,300],
      "rotation": {
        "pitch": -15,
        "yaw": 80
      },
      "time": 20
    },
    {
      "location": [121.49687008,31.13777349,300],
      "rotation": {
        "pitch": -20,
        "yaw": 160
      },
      "time": 15
    },
    {
      "location": [121.49441582,31.13728981,300],
      "rotation": {  // 最后一帧 镜头停止后的姿态
        "pitch": -15,
        "yaw": 240
      }
    }
  ]
}

const entityObj = new App.CameraRoam(jsondata);
const res = await App.Scene.Add(entityObj);

// 开启相机漫游
const args = {
    "state": "play", //play:漫游; pause:暂停; stop:结束
    "progressRatio": 0, //镜头位置切换到整体漫游比例,范围[0,1]
    "speedRatio": 1, //相机漫游移动倍率
    "bReverse": false //是否反向
}

await App.CameraControl.PlayCameraRoam(entityObj, args);
```
#### 更新相机漫游
```javascript
const jsondata = {
  "frames": [
    {
      "location": [121.48216421,31.10446008,300],
      "rotation": {
        "pitch": -25, //俯仰角, 参考(-90~0)
        "yaw": -90, //偏航角, 参考(-180~180; 0:东; 90:南; -90:北)
      },
      "time": 20 //镜头到下一帧的时间(单位:秒)
    },
    {
      "location": [121.48405533,31.12949274,300],
      "rotation": {
        "pitch": -15,
        "yaw": 0
      },
      "time": 20
    },
    {
      "location": [121.48880933,31.13466789,300],
      "rotation": {
        "pitch": -25,
        "yaw": 90
      }
    }
  ]
}

// roamObj 为 new App.CameraRoam({...}) 时创建的对象;
await roamObj.Update(jsondata);

const args = {
    "state": "play", //play:漫游; pause:暂停; stop:结束
    "progressRatio": 0, //镜头位置切换到整体漫游比例,范围[0,1]
    "speedRatio": 1, //相机漫游移动倍率
    "bReverse": false //是否反向
}

await App.CameraControl.PlayCameraRoam(entityObj, args);
```
#### 获取相机漫游信息
``` javascript
// roamObj 为 new App.CameraRoam({...}) 时创建的对象;
const _res = await roamObj.Get();
console.log(_res);
```
#### 停止相机漫游
```javascript
await App.CameraControl.Stop()
```

### 通用基础属性
#### Scene & Geometry
##### Add (往场景里添加entity)
```javascript
await App.Scene.Add(obj, {
  "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
    "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
    "coordZOffset": 50   // 高度(单位:米)
  }
});



返回:
{
    "success": true, // true, false
    "message": '',
    "result": {
        "object": {}, // "objects": [],
        "sceneChangeInfo": {}
    }
}
```

##### Update (多个相同类型对象统一更新)
```javascript
await App.Scene.Update([obj, obj, ...],
    { poiStyle: { ... } }, {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
    }
});
返回:
{
    "success": true, // true, false
    "message": '',
    "result": {
        "sceneChangeInfo": {}
    }
}
```
##### Create (批量添加相同分类的entity)
以下调用详情见：实体通用行为
```javascript
await App.Scene.Create({
    ... // 默认属性值
}, [{
   ... // 批量属性值
}], {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
      }
   }
);
```
##### Creates (批量添加不同分类的entity)
```javascript
await App.Scene.Creates([{
	... // 批量属性值
}], {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
      }
   }
);
```

##### GetAll (获取场景中所有entity对象)
```javascript
await App.Scene.GetAll();
```
##### GetByEids (通过eid获取对象)
``` javascript
await App.Scene.GetByEids(['-9151314316185345952', '-9151314316965221260', ...]);
```
##### GetByEntityName (通过entityName获取entity)
``` javascript
await App.Scene.GetByEntityName(['name01', 'name02', ...]);
```
##### GetByCustomId (通过customId获取entity)
``` javascript
await App.Scene.GetByCustomId(['cuId01', 'cuId02', ...]);
```
##### GetByTypes (通过类型获取entity)
``` javascript
await App.Scene.GetByTypes(['Poi', 'Static', ...]);
```
##### Delete (批量删除entity)
``` javascript
await App.Scene.Delete([obj, obj, ...]);
```
##### ClearByTypes (批量删除entity)
``` javascript
await App.Scene.ClearByTypes(['xxx', 'xxx', ...]);
/*
  Static           静态模型
  Skeletal         骨骼模型
  Hierarchy        结构模型
  Effects          粒子特效
*/

/*
  RealTimeVideo    实时视频
  Window           窗口
  Poi              POI
  Particle         特效
  Effects          粒子特效
  Light            灯光
  Text3D           3D文字
  Viewshed         可视域
  Path             路径
  Parabola         迁徙图
  Range            区域轮廓
  HeatMap          热力图
  ColumnarHeatMap  柱状热力图
  SpaceHeatMap     点云热力图
  RoadHeatMap      路径热力图
  Raster           栅格图
  HighlightArea    高亮区域
*/
```
##### ClearByObjects (批量删除entity)
``` javascript
await App.Scene.ClearByObjects([obj, obj, ...]);
```
##### Covering.Clear (删除场景中所有覆盖物)
``` javascript
await App.Scene.Covering.Clear();
```
##### ClearByCustomId (通过customId批量删除)
``` javascript
await App.Scene.ClearByCustomId(['xxx', 'xxx', ...]);
``` 
##### ClearByEntityName (通过entityName批量删除)
``` javascript
await App.Scene.ClearByEntityName(['xxx', 'xxx', ...]);
```
##### UpdateByCustomId (通过customId批量更新同类entity)
``` javascript
await App.Scene.UpdateByCustomId(['xxx', 'xxx', ...], { poiStyle: { ... } }, {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
    }
});
```
##### UpdateByCustomIds (通过customId批量更新不同类entity)
``` javascript
await App.Scene.UpdateByCustomId([
        { customId: 'cuId01', poiStyle: { ... } },
        { customId: 'cuId02', pathStyle: { ... } }
    ], {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
    }
});
```
##### UpdateByEntityName (通过entityName批量更新同类entity)
``` javascript
await App.Scene.UpdateByEntityName(['xxx', 'xxx', ...], { poiStyle: { ... } }, {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
    }
});
```
##### UpdateByEntityNames (通过entityName批量更新不同类entity)
``` javascript
await App.Scene.UpdateByEntityNames([
        { entityName: 'name01', poiStyle: { ... } },
        { entityName: 'name02', pathStyle: { ... } }
    ], {
    "calculateCoordZ": {   // 坐标类型及坐标高度; [可选] 最高优先级
        "coordZRef": "surface",   // surface: 表面;  ground: 地面;  altitude: 海拔
        "coordZOffset": 50   // 高度(单位:米)
    }
});
```
##### SetVisibleByObjects (批量显隐entity)
``` javascript
await App.Scene.SetVisibleByObjects([obj, obj, ...], false);
```
##### SetVisible (批量显隐entity)
``` javascript
await App.Scene.SetVisible([obj, obj, ...], false);
```
##### SetLocation (多个对象移动到同一个位置)
``` javascript
await App.Scene.SetLocation([obj, obj, ...], { x: 121.50796384, y: 31.23267352, z: 50 });
```
##### SetLocations (多个对象移动到不同位置)
``` javascript
await App.Scene.SetLocations([
    { object: obj1, location: { x: 121.50796384, y: 31.23267352, z: 50 } },
    { object: obj2, location: { x: 121.52796384, y: 31.25267352, z: 50 } },
]);
```
##### SetRotator (多个对象旋转到同一角度)
``` javascript
await App.Scene.SetRotator([obj, obj, ...], { pitch: 70, yaw: 20, roll: 80 });
``` 
##### SetRotators (多个对象旋转到不同角度)
``` javascript
await App.Scene.SetRotators([
    { object: obj1, rotator: { pitch: 70, yaw: 20, roll: 80 } }
    { object: obj2, rotator: { pitch: 50, yaw: 30, roll: 70 } }
]);
```
##### SetScale3D (多个对象缩放相同倍数)
``` javascript
await App.Scene.SetScale3D([obj, obj, ...], { x: 10, y: 50, z: 50 });
``` 
##### SetScale3Ds (多个对象缩放不同倍数)
``` javascript
await App.Scene.SetScale3Ds([
    { object: obj, scale3d: { x: 10, y: 50, z: 50 } }
]);
```
##### SetLocked (多个对象同时锁定解锁)
``` javascript
await App.Scene.SetLocked([obj, obj, ...], false);
```

#### 基础与自定义属性
基础与自定义属性
3D文字Text3D为示例, 适用所有实体覆盖物
``` javascript
const obj = new App.Text3D({
    ... ...
    // 基础属性，所有实例全部具有
    "bLocked": true, //添加的实体是否锁定, 不可点击、框选等操作(true/false) [可选]
    "bVisible": true, //添加的实体是否可见(true/false) [可选]

    // 自定义属性，所有实例全部具有; 按业务需求 自行定义内容
    "entityName": "myName", //[可选]
    "customId": "myId", //[可选]
    "customData": { //[可选]
        "data": "myCustomData"
    }

})
```

#### 属性设置
以字母b开始的key; Get/Set属性省略字母b, 且首字母大写
``` javascript
// 获取 getter:
// 方式一:
console.log("bLocked:: ", obj.bLocked);
console.log("bVisible:: ", obj.bVisible);
console.log("entityName:: ", obj.entityName);
console.log("customId:: ", obj.customId);

// 方式二:
console.log("GetLocked:: ", await obj.GetLocked());
console.log("GetVisible:: ", await obj.GetVisible());
console.log("GetEntityName:: ", await obj.GetEntityName());
console.log("GetCustomId:: ", await obj.GetCustomId());
// 设置 setter:
// 方式一:
obj.bLocked = false;
obj.bVisible = false;
obj.entityName = 'newName';
obj.customId = 'newId';

// 方式二:
await obj.SetLocked(false);
await obj.SetVisible(false);
await obj.SetEntityName('newName');
await obj.SetCustomId('newId');

```
接收一：对象点击事件
```javascript
text3d.onClick(async ev => {
    const obj = ev.result.object;
    console.log(await obj.Get());
})
```
接收二：事件监听回调
```javascript
App.Renderer.RegisterSceneEvents([
  {
    name: 'OnEntityClicked', func: async function (res) {
      // 覆盖物被点击事件回调; 包含数据信息与实体对象
      if (res.result.object.entityName === "myName") {
        const jsondata = {
          "text3DStyle": {
            "text": "更新3D文字",
            "color": "a421ffff",
            "type": "plain",
            "outline": 0.2,
            "portrait": false,
            "space": 0.2
          }
        }
        const newObj = res.result.object;
        newObj.Update(jsondata, {
          "calculateCoordZ": {  //[可选] 最高优先级
            "coordZRef": "surface",//surface:表面;ground:地面;altitude:海拔
            "coordZOffset": 0 //高度(单位:米)
          }
        });

        const info = await newObj.Get();
        console.log(info)
      };
    }
  }
])
```

#### 实体扩展属性
其它实例属性参照各实例属性字段
```javascript
// particleObj 为 new App.Particle({...}) 时创建的对象;

// 获取Particle属性
async function getParticleAttr () {
    // 方式一:
    console.log("location:: ", particleObj.location);
    console.log("type:: ", particleObj.particleType);
    console.log("rotator:: ", particleObj.rotator);
    console.log("scale3d:: ", particleObj.scale3d);

    // 方式二:
    console.log("location:: ", await particleObj.GetLocation());
    console.log("type:: ", await particleObj.GetParticleType());
    console.log("rotator:: ", await particleObj.GetRrotator());
    console.log("scale3d:: ", await particleObj.GetScale3d());
}
getParticleAttr();


// 设置Particle属性
async function setParticleAttr () {
    // 方式一:
    particleObj.location = [121.46141528,31.23360944,86];
    particleObj.particleType = "vehicle_car_white";
    particleObj.rotator = {
        "pitch": 0, "yaw": 40, "roll": 0
    };
    particleObj.scale3d = [200, 200, 200];

    // 方式二:
    await particleObj.SetLocation([121.46141528,31.23360944,86]);
    await particleObj.SetParticleType("vehicle_car_white");
    await particleObj.SetRotator({
        "pitch": 0, "yaw": 40, "roll": 0
    });
    await particleObj.SetScale3d([200, 200, 200]);
}
setParticleAttr();

```
实例中属性字段key是type时，设置属性时映射的新字段为sType
```javascript
// pathObj 为 new App.Path({...}) 时创建的对象;

// 获取Path属性
async function getPathAttr (attr) {
    // 方式一:
    console.log("coordinates:: ", pathObj.coordinates);
    console.log("sType:: ", pathObj.sType);
    console.log("width:: ", pathObj.width);
    console.log("color:: ", pathObj.color);
    console.log("passColor:: ", pathObj.passColor);

    // 方式二:
    console.log("coordinates:: ", await pathObj.GetCoordinates());
    console.log("sType:: ", await pathObj.GetsType());
    console.log("width:: ", await pathObj.GetWidth());
    console.log("color:: ", await pathObj.GetColor());
    console.log("passColor:: ", await pathObj.GetPassColor());
}
getPathAttr();


// 设置Path属性
async function setPathAttr (attr) {
    // 方式一:
    pathObj.coordinates = [
      [121.50056782,31.22792919,23],
      [121.49728647,31.22611933,90],
      [121.48236809,31.23146931,60]
    ];
    pathObj.sType = "solid";
    pathObj.width = 50;
    pathObj.color = "ff4b3dff";
    pathObj.passColor = "affff2ff";

    // 方式二:
    await pathObj.SetCoordinates([
      [121.50056782,31.22792919,23],
      [121.49728647,31.22611933,90],
      [121.48236809,31.23146931,60]
    ]);
    await pathObj.SetsType("solid");
    await pathObj.SetWidth(50);
    await pathObj.SetColor("ff4b3dff");
    await pathObj.SetPassColor("affff2ff");
}
setPathAttr();
```
#### 实体通用方法
##### 实体成员函数
示例: Text3D 成员函数
```javascript
const obj = new App.Text3D({ ... });
obj.Update(json);
obj.Get/SetLocation(json);
obj.Get/SetRotator(json);
obj.Get/SetScale3d(json);
obj.Get/SetLocked(boolean);
obj.Get/SetVisible(boolean);
obj.Get/SetEntityName(string);
obj.Get/SetCustomId(string);
obj.Get/SetCustomData(json);
obj.Get();
obj.oType;  //get
obj.bRemoved;  //get
obj.bLocked = boolean;   //get/set
obj.location = [121.49328325, 31.23863899, 10];   //get/set
obj.rotator = {pitch: 0, yaw: 60, roll: 0}   //get/set
obj.scale3d = [5,5,5];   //get/set
obj.bVisible = boolean;   //get/set
obj.entityName = '';   //get/set
obj.customId = '';   //get/set
obj.customData = {};   //get/set
obj.Delete();
obj.onClick(ev => {
    const newObj = ev.result.object;
    console.log(ev);
})
obj.onMouseEnter()(ev => {
    console.log(ev);
})
obj.onMouseOut()(ev => {
    console.log(ev);
})
```
#### 实体轮廓/高亮
##### 设置实体轮廓/高亮样式
```javascript
async function SetVisualStyle() {
  // 自定义轮廓、高亮样式(styleName, hexa颜色; alpha: 高亮有效)
  App.Setting.SetVisualColorStyle('styleName', "0f5dff4c")

  const style = await App.Setting.GetVisualColorStyle();
  console.log(style);


  // 设置轮廓线宽
  App.Setting.SetOutlineThickness(2);  //最小值: 2

  const thickness= await App.Setting.GetOutlineThickness();
  console.log(thickness);

}
```
##### 设置实体轮廓
```javascript
async function SetEntityOutline() {
  // 添加的实体覆盖物/模型对象
  const entityObj1 = cache.get('text3d'),
        entityObj2 = cache.get('particle');

  entityObj1.SetEntityOutline({
    bHighlight: true,
    styleName: "Blue"
  })


  App.Scene.SetEntityOutline({
    entities: [entityObj1,entityObj2],
    bOutline : true,
    styleName: "Blue"
  });

}
```

##### 设置实体高亮
```javascript
async function SetEntityHighlight() {
  // 添加的实体覆盖物/模型对象
  const entityObj1 = cache.get('text3d'),
        entityObj2 = cache.get('particle');

  entityObj1.SetEntityHighlight({
    bHighlight: true,
    styleName: "Blue"
  })

  App.Scene.SetEntityHighlight({
    entities: [entityObj1,entityObj2],
    bHighlight: true,
    styleName: "Blue"
  })

}
```