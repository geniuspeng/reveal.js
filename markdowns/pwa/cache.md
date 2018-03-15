## Cache API介绍

Cache 接口提供缓存的 Request / Response 对象对的存储机制，例如作为ServiceWorker 生命周期的一部分。 Cache 接口像 workers 一样, 是暴露在 window 作用域下的。尽管它被定义在 service worker 的标准中,  但是它不必一定要配合 service worker 使用。


### Cache 和 CacheStorage
Cache 和 CacheStorage是两个不同的东西。Cache直接和请求打交道，CacheStorage和Cache对象打交道。通俗点来说，Cachestorage对象可以看作是当前域名下所有Cache对象的集合体，它提供了一系列接口去操作Cache对象，可以通过暴露在window中的caches属性来访问。例如，虽然API上显示的是CacheStorage.open()，但我们实际使用的时候，直接使用caches.open()。使用CacheStorage.open()打开一个Cache对象，就可以使用 Cache 对象的方法去处理缓存了。
Note:你需要定期地清理缓存条目, 因为每个浏览器都严格限制了一个域下缓存数据的大小。 浏览器尽其所能去管理磁盘空间，浏览器有可能去删除一个域下的缓存数据，确保你的代码可以安全地操作缓存。


### CacheStorage

- CacheStorage.open(cacheName):传入一个cacheName。
- CacheStorage.match(request, options):传入一个Request对象（或者url）。
- CacheStorage.has(cacheName) :检查是否存在指定名称的 Cache 对象。
- CacheStorage.keys():返回 CacheStorage 中所有 Cache 对象的 cacheName 列表。
- CacheStorage.delete(cacheName):删除指定 cacheName 的 Cache 对象。
Note:刚刚提到过CacheStorage主要用来管理Cache对象，在 W3C 规范中，CacheStorage 对应到内核的 ServiceWorkerCacheStorage 对象。它提供了很多JS接口用于操作Cache 对象，这些接口均返回一个Promise对.


### Cache

- Cache.match(request, options) 用于查找是否存在以 Request 为 key 的Cache 对象，resolve的结果是跟 Cache 对象匹配的第一个已经缓存的请求。
- Cache.matchAll(request, options) 用于查找是否存在一组以 Request 为 key 的 Cache 对象组，resolve的结果是跟 Cache 对象匹配的所有请求组成的数组。
- Cache.put(request, response)，同时抓取一个请求及其响应，并将其添加到给定的cache。


- Cache.add(request)用于抓取这个URL, 检索并把返回的response对象添加到给定的Cache对象.
- Cache.addAll(requests)，抓取一个URL数组，检索并把返回的response对象添加到给定的Cache对象。


- Cache.delete(request, options)，搜索key值为request的Cache 条目。如果找到，则删除该Cache 
条目，并且返回一个resolve为true的Promise对象；如果未找到，则返回一个resolve为false的Promise对象。
- Cache.keys(request, options)，返回一个Promise对象，resolve的结果是Cache对象key值组成的数组。
Note:一个域可以有多个 Cache 对象.  你将在你的代码中处理和更新缓存 (e.g. in a ServiceWorker) . 在 Cache 除非显示地更新缓存, 否则缓存将不会被更新; 缓存数据不会过期, 除非删除它. 使用 CacheStorage.open(cacheName) 打开一个Cache 对象, 再使用 Cache 对象的方法去处理缓存.
规范里 Cache 对应内核的 ServiceWorkerCache 对象，提供了已缓存的 Request / Response 对象体的存储管理机制。它提供了一系列管理存储的JS接口，除了add，addAll和put这三个用于添加cache的方法，其他接口接口同样统统返回一个Promise对象：


说到存储，那必然会涉及到存储的容量大小，Service Worker 规范并没有明确规定 ServiceWorkerCache 的容量限制，在 Chromium 50 以下版本的内核限制为 512M，Chromium 50 及以上版本内核不作限制（即为std::numeric_limits<int>::max）。当然，这只是 Service Worker 层面的限制，它还会受浏览器 QuotaManager 的限制。


关于它与http cache的区别，HTTP Cache 中，一个 URL 只能对应一个 Response，但 Cache API 规范要求同一 URL（不同的 Header）可以对应多个 Response，另外，HTTP Cache 没有使用容量管理系统（QuotaManager）而 Cache API 需要使用。当 Service Worker 从 Cache 拿不到资源时，就会去 http cache 查找，找不到才去请求网络。