使用node-webkit加载远程页面并调用本地程序
---

20131001

最近接到这么个需求:

系统登录后，自动最大化到整个桌面，用户通过任意方式都无法关闭系统。点击键盘指定热键，弹出解锁窗口，通过验证后窗口恢复正常。然后可以再次整个桌面最大化。

对于这种需求，传统的BS受限于浏览器，是无能为力的。无奈之下，只能自己选择浏览器内核，通过封装浏览器外壳的方式来实现了。

# 方案选择

锁定桌面可以使用其他语言单独实现，然后通过命令行进行调用。

浏览器内核基于性能、功能，以及开发方便考虑，首选webkit内核。

最近`node-webkit`相当之火，其宣称可以将node和js结合到一起，看上去非常不错。而且团队没有时间精力去折腾内核封装，最终定下了这一方案。

# 遇到的问题

## 全屏最大化

node-webkit已经提供了这项功能，在`package.json`中设置`kiosk:true`即可，也可以通过api在js文件中调用。 

## 调用远程页面

node-webkit的目的是基于HTML5和node来开发桌面程序，所以对于一般的开发来说，所有代码逻辑是放到本地的。但是我们的项目需要放在服务器端，这就需要使用`iframe`将远程页面嵌到本地页面中。

## 避免远程页面通过top直接调用本地脚本

我们的项目内部已经嵌套了很多iframe，而且很多是通过top来引用项目主页面脚本。如果直接通过iframe引入项目，需要进行大量的修改。

经查找发现，node-webkit为这个问题提供了贴心的设置，在iframe上设置`nwdisable nwfaketop`即可。

具体缘由详见 https://github.com/rogerwang/node-webkit/issues/534

## 远程页面调用本地程序

但是如果这样设置，远程页面无法通过传统的top来通知本地页面，这就无法调用本地扩展了。怎么办捏？

> node-webkit禁止远程页面访问本地，但是没有禁止本地页面访问远程。

这样反过来一想就豁然开朗了。

本地页面监听iframe窗体消息中：
```
frames[0].addEventListener('message', function(){}, false);
```
远程页面触发：
```
window.postMessage();
```
 
## 远程页面302跳转的处理

远程项目登录后会302跳转到新的页面，这时本地页面监听的iframe窗体实际上就变化了。因此需要对跳转进行监测。
```
var url = appFrame.location.href;
setInterval(function() {
     if (url !== appFrame.location.href) {
          url = appFrame.location.href;

          frames[0].addEventListener('message', function(){}, false);
     }
}, 1000);
```
 
## 最大化的处理

本来没什么好说的，只需要响应事件maximize即可
```
win.on('maximize', function() {
     win.enterKioskMode();
});
```
 但是在测试过程中发现，windows下`leaveKioskMode`也会导致触发`maximize`事件。因此做了个标志位，只在手工点击最大化按钮时进入kiosk模式。

# 总结

项目总体进展比较顺利，node-webkit比较成熟，许多功能已经内置。只要从思路上理清其运作机制，开发类似程序还是挺方便的。更重要的是，node-webkit能够极大的扩充前端开发人员的能力范围，对于浏览器没有提供的接口，可以通过自己开发浏览器外壳的方式予以补充，这对企业开发来说非常重要。

# PS

一般的远程调用可以直接在`package.json`中，通过配置`node-remote`为true，`main`为远程路径来实现。但是在测试过程中发现这样调用的远程页面，可以调用node-webkit的接口，如`require('nw.gui')`等，但是对于本地node环境，尤其是本地调用其他外部程序，貌似还是有问题。

 
