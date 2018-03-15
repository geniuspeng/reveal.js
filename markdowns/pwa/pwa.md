### PWA
PWA的核心技术包括：

- Web App Manifest – 在主屏幕添加app图标，定义手机标题栏颜色之类
- Service Worker – 缓存，离线开发，以及地理位置信息处理等
- App Shell – 先显示APP的主结构，再填充主数据，更快显示更好体验
- Push Notification – 消息推送，之前有写过“简单了解HTML5中的Web Notification桌面通知”
Note:有此可见，Service Worker仅仅是PWA技术中的一部分，但是又独立于PWA。也就是虽然PWA技术面向移动端，但是并不影响我们在桌面端渐进使用Service Worker，考虑到自己厂子用户70%~80%都是Chrome内核，收益或许会比预期的要高。


#### APP Manifest 与添加到主屏幕

- 和 APP 一样在用户桌面(主屏幕)上存在图标
- 有 splash screen 启动屏
- 可以控制控制屏幕方向
- 可以设置全屏显示，没有浏览器的搜索框、导航栏

Note:
通过添加到主屏幕功能可以在用户桌面(主屏幕)上创建一个站点的快捷图标，实现和 Native APP 近似的使用体验。通过该图标打开的页面和浏览器中直接访问的体验有些不同，它在体验上更像 Native 应用：
要实现这一点的前提条件是：站点注册成功了 Service Worker；定义了一个 manifest 文件，并在页面中申明。


1. 创建 manifest.json 文件，并将它放到你的站点目录中。
2. 在所有页面引入该文件。
3. 在页面 head 区域添加如下内容：

```html
<link rel="manifest" href="/manifest.json">
```


#### 设置 IOS Safari 上的添加至主屏幕元素

```html
<!-- Add to home screen for Safari on iOS -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-title" content="Weather PWA">
<link rel="apple-touch-icon" href="images/icons/icon-152x152.png">
```
#### 设置 window10 贴片图标

```html
<meta name="msapplication-TileImage" content="images/icons/icon-144x144.png">
<meta name="msapplication-TileColor" content="#2F3BA2">
 ```


 ### 需要注意的地方

 - 缓存取决于为每次更改更新缓存键
 - 注意缓存优先策略在生产环境中的使用
 - app shell缓存与数据缓存的相关策略
 Note:我们的应用使用缓存优先策略，这会导致在返回任何已缓存内容的副本时都不会查询网络。尽管缓存优先策略易于实现，却可能在日后带来挑战。一旦缓存了主机页面和Service Worker注册的副本，更改Service Worker的配置可能变得极为困难（因为配置取决于其定义位置），并且您可能发现，自己部署的网站极难更新！


 - Service Worker取消注册后，它可能仍存在于列表内，直至包含它的浏览器窗口关闭为止。
 - 如果您的应用有多个窗口处于打开状态，则新Service Worker在这些窗口都已重新加载并更新到最新Service Worker后才会生效。
 - 将Service Worker取消注册并不会清除缓存，因此如果缓存名称尚未发生变化，您获得的可能仍然是旧数据。