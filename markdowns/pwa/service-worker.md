## Web Workers介绍
Service Worker是Web Workers大家族中的一员，所以这里还是首先介绍一下Web Workers。
Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面。此外，他们可以使用XMLHttpRequest执行 I/O  (尽管responseXML和通道属性总是为空)。一旦创建， 一个worker 可以将消息发送到创建它的JavaScript代码, 通过将消息发布到该代码指定的事件处理程序 (反之亦然)。
Note:实际上是一种多线程的机制，worker与用户页面的JS是相互独立的两个线程，所以能够互不干扰，worker有自己的作用域（如专用Worker的作用域DedicatedWorkerGlobalScope），worker内可以使用大多数JavaScript特性，包括Navigator，XMLHttpRequest，Math，Date等等。需要注意的是，worker内的代码不能操作DOM，也无法影响页面外观，综合整理worker大致有如下特点：


- 全局对象就是worker对象本身，就是说self和this都指向worker对象。
- 最小化的navigator对象，有online，appName，appVersion，userAgent和platFrom属性。
- 只读的location
- WindowTimers.setTimeout 和 WindowTimers.setInterval
- XMLHttpRequest
- Array, Date, Math, String
Note:在一个worker中最主要的你不能做的事情就是直接影响父页面。包括操作父页面的节点以及使用页面中的对象。比如通过PostMessage方法。


目前主要的web worker大致有以下几种：
- Worker（专用worker）：正如名字一样，就是一个正常的worker
- SharedWorker（共享worker）：一个共享worker可以被多个脚本使用——即使这些脚本正在被不同的window、iframe或者worker访问
- ServiceWorker：本文详细介绍。
- Chrome Workers： 一种仅适用于firefox的worker。详情请参阅[ChromeWorker](https://developer.mozilla.org/en-US/docs/Web/API/ChromeWorker)。
- Audio Workers（音频worker）：配合Web Audio API使用，使得在web worker上下文中直接完成脚本化音频处理成为可能。


## Service Worker
在2014年，W3C公布了service worker的草案，service worker提供了很多新的能力，使得web app拥有与native app相同的离线体验、消息推送体验。
Service Worker是PWA的核心。谷歌给以 Service Worker API 为核心实现的 web 应用取了个高大上的名字：Progressive Web Apps（PWA，渐进式增强 WEB 应用），并且在其主要产品上进行了深入的实践。
Note:Service Worker做为Worker一种，具有上述提到的worker的所有特性，同时还提供的自己独有的一些功能。例如：在页面中注册并安装成功后，运行于浏览器后台，不受页面刷新的影响，可以监听和截拦作用域范围内所有页面的 HTTP 请求。
基于 Service Worker API 的特性，结合 Fetch API、Cache API、Push API、postMessage API 和 Notification API，可以在基于浏览器的 web 应用中实现如离线缓存、消息推送、静默更新等 native 应用常见的功能，以给 web 应用提供更好更丰富的使用体验。


#### Service Worker兼容性
参考[http://caniuse.com/#feat=serviceworkers](http://caniuse.com/#feat=serviceworkers)
![img](https://raw.githubusercontent.com/geniuspeng/image-storage/master/blog/service-worker/sw-caniuse.png)
Note:当前，Edge已经竖起了小绿旗（默认不支持但可以手动开启），只剩下苹果的Safari还是一片红，虽然如此，不过最近似乎也开始在默默地搞起来了，支持sw只是时间的问题，至于IE嘛，不想多说。


<!-- .slide: data-background="#eaeaea" -->
#### Service Worker生命周期
服务工作线程的生命周期完全独立于网页，如下

install -> installed -> actvating -> Active -> Activated -> Redundant

[图片](https://raw.githubusercontent.com/geniuspeng/image-storage/master/blog/service-worker/sw-lifecycle.png)
Note:服务工作线程的生命周期完全独立于网页.
要为网站安装服务工作线程，您需要先在页面的 JavaScript 中注册。 注册服务工作线程将会导致浏览器在后台启动服务工作线程安装步骤。
在安装过程中，您通常需要缓存某些静态资产。如果所有文件均已成功缓存，那么服务工作线程就安装完毕。如果任何文件下载失败或缓存失败，那么安装步骤将会失败，服务工作线程就无法激活（也就是说，不会安装）。 如果发生这种情况，不必担心，它下次会再试一次。 但这意味着，如果安装完成，您可以知道您已在缓存中获得那些静态资产。
安装之后，接下来就是激活步骤，这是管理旧缓存的绝佳机会，我们将在服务工作线程的更新部分对此详加介绍。
激活之后，服务工作线程将会对其作用域内的所有页面实施控制，不过，首次注册该服务工作线程的页面需要再次加载才会受其控制。服务工作线程实施控制后，它将处于以下两种状态之一：服务工作线程终止以节省内存，或处理获取和消息事件，从页面发出网络请求或消息后将会出现后一种状态。
使用Service Worker有几个需要注意的地方，首先，要对一些前置的基础知识有一些了解，主要是[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise),以及前文提到的Fetch API，Cache API。
其次，由于一些不可描述的原因，Service Worker只能跑在HTTPS的服务器上，当然，Chrome等浏览器都对localhost跳过了安全认证，我们可以使用本地localhost进行调试。
这里主要介绍Service Worker的注册，以及相关事件的监听处理。


##### 注册Service Worker
Service Worker的注册不同于其他web workers，但是也很简单，只需要使用一个register方法即可，例如注册/sw/sw.js文件：

```js
if ('serviceWorker' in navigator) {
navigator.serviceWorker
         .register('/sw/sw.js', {scope: '/'})
         .then(function() { console.log('Service Worker Registered'); });
}
```
Note:注意这里有一个作用域的问题，如果你的sw.js写在/sw/路径下，不带第二个scope参数，Service worker只能监听/sw/路径下的fetch事件，而无法监听此目录外的fetch，使用scope可以改变这个作用域到根目录，对整个站点生效。另外Service Worker 没有页面作用域的概念，作用域范围内的所有页面请求都会被当前激活的 Service Worker 所监控。


##### install事件
只有注册是在页面js内实现，而sw所有的事件处理都写在worker的js下。首先是install事件，当前脚本被安装时，会触发 install 事件，在这个事件里，通常会做一些缓存的添加处理。

```js
let cacheName = 'my-cache';
let filesToCache = [
  '/index.html',
  '/js/test.js',
  '/css/test.css'
];
self.addEventListener('install', (e) => {
  console.log('[ServiceWorker] Install');
  e.waitUntil(
    caches.open(cacheName).then((cache) => {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll(filesToCache);
    })
  )
});
```


这里的event参数，实际上是一个[InstallEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/InstallEvent)实例，它继承了父类[ExtendableEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/ExtendableEvent)和Event的方法。
```
Event <- ExtendableEvent <- InstallEvent
```
Note:而ExtendableEvent有一个独有的方法waitUnti(),这个方法接收一个Promise（或者async function），会阻塞当前事件的行为而优先执行该Promise，直到Promise结束，再继续进行当前事件。所以在install事件里一般在安装之前先把重要的app shell进行缓存，确切的说一般缓存的是用于离线展示的相关资源。需要注意的是，filesToCache全部缓存之后，才会执行install事件，也就是说如果waitUntil内的Promise没有成功，install事件将不会触发而导致Service worker安装失败而进入redundant (废弃)状态，所以这里应尽量减少filesToCache中的缓存资源。
安装成功后，即进入等待(waiting)或激活(active)状态。在激活状态可通过监听各种事件，实现更为复杂的逻辑需求。


#### activate事件
当安装完成后并进入激活状态，会触发 activate 事件。通过监听 activate 事件你可以做一些预处理，如对于旧版本的更新、对于无用缓存的清理等。

```js
self.addEventListener('activate', function(e) {
  console.log('[ServiceWorker] Activate');
  e.waitUntil(
    caches.keys().then(function(keyList) {
      return Promise.all(keyList.map(function(key) {
        if (key !== cacheName) {
          console.log('[ServiceWorker] Removing old cache', key);
          return caches.delete(key);
        }
      }));
    })
  );
  return self.clients.claim();
});
```
Note:这里的event同样继承了[ExtendableEvent]，在这里对CacheStorage对象进行遍历，将过期的cache进行一个移除操作，及时清理CacheStorage的存储空间。
waitUntil阻塞activate事件，这可以确保清理操作会在第一次 fetch 事件之前完成。
在激活时执行clients.claim 方法，更新所有客户端上的 Service Worker。


#### fetch事件
当浏览器发起请求时，会触发 fetch 事件。

```js
self.addEventListener('fetch', function(e) {
  console.log('[Service Worker] Fetch', e.request.url);
  e.respondWith(
    caches.match(e.request).then(function(response) {
      return response || fetch(e.request);
    })
  );
});
```
Note:Service Worker 安装成功并进入激活状态后即运行于浏览器后台，可以通过 fetch 事件可以拦截到当前作用域范围内的 http/https 请求，并且给出自己的响应。结合 Fetch API ，可以简单方便地处理请求响应，实现对网络请求的控制。这个功能十分强大，可以说是PWA的核心功能。


此处的参数是一个FetchEvent的实例，[FetchEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/FetchEvent)继承了Event的属性和方法，同时拥有自己的方法respondWith。
FetchEvent 接口的 respondWith() 方法旨在包裹代码，这些代码为来自页面的request生成自定义的response。这些代码通过返回一个 Response 、 network error 或者 Fetch的方式resolve。


#### push事件
push 事件是为推送准备的。不过首先你需要了解一下 Notification API 和 PUSH API。

通过 PUSH API，当订阅了推送服务后，可以使用推送方式唤醒 ServiceWorker 以响应来自系统消息传递服务的消息，即使用户已经关闭了页面。


#### sync事件
sync事件由background sync发出，是一种后台同步事件。background sync 是 Google 配合 SW 推出的 API，用于为 SW 提供一个可以实现注册和监听同步处理的方法。但它还不在 W3C WEB API 标准中。在 Chrome 中这也只是一个实验性功能，需要访问 chrome://flags/#enable-experimental-web-platform-features ，开启该功能，然后重启生效。


在页面注册sync事件：

```js
// Register your service worker:
navigator.serviceWorker.register('/sw.js');

// Then later, request a one-off sync:
navigator.serviceWorker.ready.then(function(swRegistration) {
  return swRegistration.sync.register('myFirstSync');
});
```
在sw.js中监听sync事件

```js
self.addEventListener('sync', function(event) {
  if (event.tag == 'myFirstSync') {
    event.waitUntil(doSomeStuff());
  }
});
```
Note:一个很酷的地方在于，sync事件可以在离线时发出，ServiceWorker会记住发出的事件，并且在重新联网时做出响应，这可以解决我们实际生活中很多不必要的“时间浪费”，比如我们提交某个表单，提交的时候可能网络不好等了好久，最后还没提交成功又要重新填写表单数据，有了sync时间我们完全不必考虑网络情况，让后台同步解决就可以了。


#### message事件
前面说过，ServiceWorker同其他Worker一样，运行于一个独立的沙盒中，无法访问DOM等页面相关的信息，但我们可以通过 postMessage
API，实现Service Worker与页面之间的通信。


##### 页面向SW发消息
首先在sw注册之后，才可以navigator.serviceWorker.controller句柄。

```js
const sendMessageToSW = (msg) => {
  navigator.serviceWorker.controller && navigator.serviceWorker.controller.postMessage(msg);
} 
if ('serviceWorker' in navigator) {
  navigator.serviceWorker
           .register('/service-worker.js')
           .then(function() { console.log('Service Worker Registered'); })
           .then(() => sendMessageToSW('hello sw!'));
}
```
在sw.js监听message事件

```js
self.addEventListener('message', function(event) {
  console.log(event.data);
});
```


#### online/offline事件
当网络状态发生变化时，会触发 online 或 offline 事件。结合这两个事件，可以与 Service Worker 结合实现更好的离线使用体验，例如当网络发生改变时，替换/隐藏需要在线状态才能使用的链接导航等。

```js
window.addEventListener('offline', function(event) {
  console.log('离线了')
  Notification.requestPermission().then((grant) => {
    if (grant !== 'granted') return;
    const notification = new Notification("网络挂掉了哦~", {
      body: '虽然离线了，但是我还可以访问',
      icon: 'http://si1.go2yd.com/get-image/0Gp8bLbHOkK'
    });

  })
});
```


### Service Worker调试
在 Chrome 中，service worker 的信息显示在 Application -> Service Workers 中，就像这样
![img](https://raw.githubusercontent.com/geniuspeng/image-storage/master/blog/service-worker/sw-debug.png)


顶部有四个选项：
- Offline： 选中则当前站点即断开网络状态
- Update on reload： 选中则当刷新页面时强制更新
- Bypass for network： 选中则资源加载和更新绕过 Service Worker，都要从网络获取
- show all：显示所有的sw


下面还有四个选项：
- Update：(更新)按钮执行指定的Service workers 一次性更新。
- Push： (推送)按钮会模拟推送一个没有 payload 的通知(会触发 push 事件)，也称为 tickle。
- Synch： (同步)按钮模拟一个后台同步事件(会触发 sync 事件)。
- Unregister：(注销)按钮注销指定的Service Worker。查看Clear Storage以注销Service Workers，并通过单击按钮清除所有缓存和存储。