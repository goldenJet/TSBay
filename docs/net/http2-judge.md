#### 方法一
使用 Chrome，在 Console 中输入如下指令：window.chrome.loadTimes()

输出的 connectionInfo 和 npnNegotiatedProtocol 是 h2 就说明使用的是 http2

```
commitLoadTime: (...)
connectionInfo: "h2"
finishDocumentLoadTime: (...)
finishLoadTime: (...)
firstPaintAfterLoadTime: (...)
firstPaintTime: (...)
navigationType: (...)
npnNegotiatedProtocol: "h2"
requestTime: (...)
startLoadTime: (...)
wasAlternateProtocolAvailable: (...)
wasFetchedViaSpdy: (...)
wasNpnNegotiated: (...)
```


#### 方法二
或者执行这个：
```
(function(){
// 保证这个方法只在支持loadTimes的chrome浏览器下执行
if(window.chrome && typeof chrome.loadTimes === 'function') {
    var loadTimes = window.chrome.loadTimes();
    var spdy = loadTimes.wasFetchedViaSpdy;
    var info = loadTimes.npnNegotiatedProtocol || loadTimes.connectionInfo;
    // 就以 「h2」作为判断标识
    if(spdy && /^h2/i.test(info)) {
        return console.info('本站点使用了HTTP/2');
    }
}
console.warn('本站点没有使用HTTP/2');})();
```

