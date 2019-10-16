## mpsdk支持mgc平台示例

以下是为了让mpsdk支持mgc平台的一些修改建议, 若有理解不对的地方请包涵.

mgc平台基本兼容微信api, 只有登录部分需要做一些调整, 所以基本思路是直接继承`MinaPlatform`的大部分代码, 然后做少量调整

### 判断是否为mgc平台

* 检查全局`mgc`是否定义, 如果有定义则是mgc平台: `if(window.mgc != null) { 是mgc平台 } `

### 得到平台实例

增加一个mgc平台实例:

```
Object.defineProperty(t, "instance", {
get: function() {
 if (!this._instance) switch (this.platformType) {
 // 添加mgc平台 
 case "mgc":
    this._instance = new e.MGCPlatform, e.log("识别到MGC平台");
   break;
   // 添加代码结束
   
  case "bk":
   this._instance = new e.BricksPlatform, e.log("识别到玩一玩/厘米游戏平台");
   break;
  case "qq":
   this._instance = new e.QQminiPlatform, e.log("识别到QQ平台");
   break;
  case "tt":
   this._instance = new e.TTPlatform, e.log("识别到字节跳动平台");
   break;
  case "wx":
   this._instance = new e.MinaPlatform, e.log("识别到微信平台");
   break;
  case "egret":
   this._instance = new e.EgretPlatform, e.log("识别到Egret引擎");
   break;
  case "cocos":
   this._instance = new e.CocosPlatform, e.log("识别到Cocos引擎");
   break;
  case "laya":
   this._instance = new e.LayaPlatform, e.log("识别到Laya引擎");
   break;
  default:
   e.log("未能识别平台类型，默认当前处于H5环境"), this._instance = new e.H5Platform
 }
 return this._instance
},
```

### 让MGCPlatform继承MinaPlatform的大部分代码, 仅自定义`getUserAccountOnce`和`login`

* getUserAccountOnce参考代码: mgc在`getUserInfo`里面直接返回了`openId`, 所以不需要再调用`login`

```
t.prototype.getUserAccountOnce = function(t) {
  var a = this;
  return new Promise(function(n, i) {
   var o = (new Date).getTime();
   e.Report.reportNewUserLog(e.constant.NewUserLogEnum.ACTION_MGC_LOGIN_BEGIN), mgc.login({
    success: function(r) {
      mgc.getUserInfo({
        success: res => {
         // 由于mpsdk使用的openid是全小写, mgc返回的字段里面有大写, 所以转换一下
          res.userInfo.openid = res.userInfo.openId
          res.userInfo.unionid = res.userInfo.unionId

          // ok
          n(res.userInfo)
        },
        fail: e => {
          i(e)
        }
      })
    },
    fail: function(t) {
     e.log("MGC登录失败: ", t), e.Report.reportNewUserLog(e.constant.NewUserLogEnum.ACTION_MGC_LOGIN_END, e.constant.NewUserLogStatusEnum.STATUS_REPORT_FAILS, "MGC登录失败:" + t), i(t)
    }
   })
  })
 }
```

* login参考代码: 由于mgc已经在`getUserInfo`返回了`openId`, 所以可以直接使用getUserInfo. 如果`login`没有在其它地方用到的话, 直接置为空函数应该也可以

```
t.prototype.login = function(t, a) {
  return new Promise(function(a, r) {
    mgc.getUserInfo({
      success: res => {
          // 由于mpsdk使用的openid是全小写, mgc返回的字段里面有大写, 所以转换一下
          res.userInfo.openid = res.userInfo.openId
          res.userInfo.unionid = res.userInfo.unionId

          // 回调
          a(res)
      },
      fail: e => {
        r(e)
      }
    })
  })
 }
```

### login方法修改的其它方案

* 如果仍然需要调用`getOpenId2.action`, 则`login`的代码不需要修改, 而在服务器端调用mgc平台的`jscode2session`接口来获得`openid`, 具体方式参考mgc平台的接入文档