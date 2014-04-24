Android分享canvas图形的一点经验总结
---

20140424

最近想做一点Android开发，基本框架使用Android原生代码，其中曲线图功能选用了https://github.com/nnnick/Chart.js 这个canvas绘图库，在做分享功能时遇到了点问题，记录如下。

## 基本原理
* 使用ShareActionProvider实现原生的分享功能。
* 使用webView.draw进行浏览器截屏操作。
* 将截取到的Bitmap保存为临时文件。
* 将图形文件作为intent的参数传给ShareActionProvider，供分享使用。

## 遇到的坑
截图时页面中的canvas无法正常显示。

页面中其他元素都能正常显示，就canvas显示为空白。难道webkit默认的draw实现就是无法导出canvas？于是在页面中又添加了一个简单的canvas，结果可以显示。那就只能是Chart这个库的问题了。简单浏览了下源码，发现以下代码比较可疑：
```
	if (window.devicePixelRatio) {
		context.canvas.style.width = width + "px";
		context.canvas.style.height = height + "px";
		context.canvas.height = height * window.devicePixelRatio;
		context.canvas.width = width * window.devicePixelRatio;
		context.scale(window.devicePixelRatio, window.devicePixelRatio);
	}
```
这是视网膜屏中防止canvas显示模糊的特殊处理。简单注释掉看看，嗯，这下可以正常导出了。

导出的问题解决了，但是这样显示太模糊了，一点不友好，看来显示页面和导出页面得作不同处理才行。考虑到webView.draw可以获取即使不在视野内的页面内容，思路就有了。

## 新的解决方案：
* 初始WebView为invisible
* 使用js渲染模糊的canvas
* Android中截屏，保存为文件，设置好ShareActionProvider
* 使用js擦除之前渲染的canvas，渲染正常的canvas，这时可以自由选用动画来增强用户体验。
* 设置WebView为visible

只要管理好android和js的调用关系，实现起来不算困难。

## 其他问题
android中定义的js接口如果涉及界面更新，需要强制在主线程执行，否则会报错。
```
            runOnUiThread(new Runnable() {
                public void run() {

                }
            });
```

js中为了调试方便，可以在桌面浏览器中自行模拟一个android接口。但是模拟接口有可能为异步，而android接口一般为同步，无法做到接口的完美统一。

android定义的js接口无法通过类似extend的手段复制到js变量中，具体原因还没有研究。

canvas中如果width，height不一致，有可能也会导致无法导出。

js绘制完canvas调用android进行截图时，需要做个延时，否则得到的图形还是空白。

## 源代码
java文件：
https://github.com/stonelee/android-money/blob/00f91a96cc21827f7b13389942f0616b0e3706a2/app/src/main/java/info/stonelee/money/app/ChartActivity.java

js文件：
https://github.com/stonelee/android-money/blob/00f91a96cc21827f7b13389942f0616b0e3706a2/app/src/main/assets/main.js


