# THAW
卡车之家weex开发--native与js通信



---
>     THAW 模块名 （truckhome and weex）在此模块下进行扩展和补充。
---

- [App传值](#App传值)
- [去登陆](#去登陆)
- [客户端响应物理返回键](#返回按钮)
- [主动获取客户端信息](#主动获取客户端信息)
- [拨打电话](#拨打电话)
- [拍照或从手机相册中选图接口](#拍照或从手机相册中选图接口)
- [获取经纬度](#获取经纬度)
- [根据经纬度显示地图](#根据经纬度显示地图)
- [url跳转](#url跳转)
- [webView标题](#webView标题)
- [地理位置提交](#地理位置提交)
- [分享](#分享)
- [分享到具体平台](#分享到具体平台)
- [显示正在加载弹层](#显示正在加载弹层)
- [隐藏正在加载弹层](#隐藏正在加载弹层)
- [关闭当前页面](#关闭当前页面)
- [如果请求失败，返回格式](#如果请求失败，返回格式)




# App传值
所有进入weex页面都必须传下面参数

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //App传的值从weex.config中获取

            {
                "status":"1" , //1表示已登录，0表示未登录
                "userId":"292972" ,//用户uid，status=0时，用户uid为0
                "auth":"&&&" //客户端登陆成功后，服务端返回的auth值，未登录传“”
                "deviceId":""//设备mac地址，数据统计用
            }

            //如果userId为0时 代表用户未登录 如果调取登录弹层，监听用户登录成功时，需要主动去更新用户的userId

            例:获取deviceId
            weex.config.deviceId

            注意：IOS中只有入口页才可以获取到 需要存储在storage中
                 安卓每个页面都可以直接获取到
        }
    }
</script>
```

# 客户端响应物理返回键

```
<!-- weex -->
<template>
  <div>
    <div class="header">
      <div class="back" @click="back">
        <div class="icon"></div>
      </div>
    </div>
</template>

<script>
let globalEvent = weex.requireModule('globalEvent');

  export default {
    created(){
      //监听用户点击安卓物理返回键
      globalEvent && globalEvent.addEventListener("onRespondNativeBack", e => {
        this.goBack() //返回上一个的方法
      })
    }
  }
</script>
```
*点击事件执行的时候给客户端传一个定好的值‘1’，告诉客户端需要做的操作*


# **主动获取客户端信息**


 在客户端打开页面的时候主动向native端传值1，module回调，通过onGetData方法通知，将得到的 信息进行操作


```
<script>
    export default {
        created () {
            var me = this;
            var thaw = weex.requireModule('thaw');
            thaw.onGetData('1',function(ret) {
                me.userid = ret.userid;
                me.userName = ret.userName;
                //执行操作
            }
        }
    }
</script>
```

*这里是进入每个页面后得到用户信息，格式json，{'userid':'123456','userName':'牛逼啦123'}*          
*这个操作在created中执行，且this必须在外定义，否则找不到*


# 拨打电话
接受一个字符串参数，为电话号码


```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        call (){
            //   native操作
            weex.requireModule('thaw').onGoCall("400-800-1616");
        }
    }
</script>
```
```

```
# 拍照或从手机相册中选图接口
点击按钮弹出选取图片或者拍照功能 传递两个参数

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        call (){
            //   native操作
            weex.requireModule('THAW').chooseImage({
               "type":"1",  //1表示图片，0表示拍照和图片。2表示拍照
               "bucket":"" , //图片上传到七牛的桶名，比如bbsimage
               "imgPath":"",  // 图片上传文件路径 比如 imga/nr/t/
               "hostUrl":"", //图片上传域名 比如 https://img9.kcimg.cn/
            },this.callBack.bind(this));
        },
        methods:{
            callBack(data){
                //返回数据
                {
                    'imageUpload':"图片地址",
                    'preview':'图片预览的base64'
                }
            }
        }
    }
</script>
```

# 获取经纬度
获取用户当前设备所在位置的经纬度。


```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').getLocation(this.callBack.bind(this));
        },
        methods:{
            callBack(data){
                //返回数据
                {
                    "state": "success", //成功是success，失败是error
                    "longitude": "", // 经度
                    "latitude": "", //纬度
                }
            }
        }
    }
</script>
```


# 根据经纬度显示地图
发送经纬度调取地图，标注位置点始终在所发送的经纬度，如果有第三个参数，第三个参数为"auto",那么标注位置始终为地图的中心点，并返回地图中心点的经纬度和地址名称
返回JSON：{state:"success":data{latitude:'',longitude:'',name:''}}

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').onShowMap({
                "type": "auto", // auto为标注物在地图中保持不动
                "longitude": "", // 经度
                "latitude": "", //纬度
            },this.callBack.bind(this));
        },
        methods:{
            callback(){
                //返回数据
                {
                    "state": "success", //成功是success，失败是error
                    "longitude": "", // 经度
                    "latitude": "", //纬度
                }
            }
        }
    }
</script>
```

# 去登陆
调起登陆弹层
    传递一个callbak，登录成功时，在callback里面操作后续方法

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').onGoLogin(this.callBack.bind(this));
        },
        methods(data){
            callback(){
                //返回数据
                {
                    "status":"1" ,     //1表示成功，0表示失败
                    "userId":"292972" ,//用户uid，status=0时，用户uid为0
                    "auth":"&&&"       //客户端登陆成功后，服务端返回的auth值
                }
            }
        }
    }
</script>
```

# url跳转
打开普通web页面

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            // 带标题跳转
            weex.requireModule('THAW').onOpenNewWeexPage({url:'需要跳转的url地址',title:'跳转的标题'})
            // 直接跳转
            weex.requireModule('THAW').goUrl(url:'需要跳转的url地址');
        }
    }
</script>
```

# webView标题
在m站中发送标题名称 app需要在头部使用该标题

```
<!--html-->
<body>
    
</body>

<!--js-->
<script>
    document.addEventListener('WebViewJavascriptBridgeReady', function(){
         WebViewJavascriptBridge.callHandler('onChangeWebTitle',{"changeWebTitle":'标题名称'});
    });
</script>
```

# 地理位置提交
点击按钮返回用户选择的经纬度和街道名称

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
let globalEvent = weex.requireModule('globalEvent');
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').onLocationCommit(this.callBack.bind(this))
        },
        methods:{
            callback(){
                // 返回参数
                {
                    "address": "北京市天安门",
                    "longitude": "", // 经度
                    "latitude": "", //纬度
                }
            }
        }
    }
</script>
```

# 分享

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').onShowShare({
                title: "", // 分享标题
                desc: "", // 分享描述
                link: "", // 分享链接
                imgUrl: "" // 分享图标
            },this.callBack.bind(this));
        },
        methods:{
            callback(){
                //返回数据
                {
                    "platform": "0",// 0-微信，1-朋友圈，2-QQ，3-qq空间，4-新浪
                    "status": "0", // 0成功，1失败 ，2取消
                }
            }
        }
    }
</script>
```

# 分享到具体平台

```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>
    export default {
        created (){
            //   native操作
            weex.requireModule('THAW').onShowSharePlatfrom({
                "platform": "0", // 分享到那个平台，0-微信，1-朋友圈，2-QQ，3-qq空间，4-新浪
                "title": "", // 分享标题
                "desc": "", // 分享描述
                "link": "", // 分享链接
                "imgUrl": "" // 分享图标
            },this.callBack.bind(this));
        },
        methods:{
            callback(){
                //返回数据
                {
                    "platform": "0",// 0-微信，1-朋友圈，2-QQ，3-qq空间，4-新浪
                    "status": "0", // 0成功，1失败 ，2取消
                }
            }
        }
    }
</script>
```

# 显示正在加载弹层
    弹出客户端正在加载弹层
```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>let
     let thaw = weex.requireModule('THAW')
     export default {
        call (){
            //   native操作
            thaw.onShowLoading();
        }
    }
</script>
```

# 隐藏正在加载弹层
    隐藏正在加载的弹层
```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>let
     let thaw = weex.requireModule('THAW')
     export default {
        call (){
            //   native操作
            thaw.onHideLoading();
        }
    }
</script>
```

# 关闭当前页面
  关闭当前页面(Weex中可直接使用navigator.pop())
```
<!--html-->
<template>
    <div @click="call" class="phone"></div>
</template>

<!--js-->
<script>let
     let thaw = weex.requireModule('THAW')
     export default {
        call (){
            //   native操作
            thaw.onGoBack();
        }
    }
</script>
```

# 如果请求失败，返回格式
JSON：{state:error,data,"失败原因"}
