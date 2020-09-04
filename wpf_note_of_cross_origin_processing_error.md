# Note of  Cross-Origin processing error

### 跨域(Cross-Origin)

跨域是现代浏览器对不同源(Origin)资源的访问控制的安全机制。产生跨域的条件是访问了不同源的资源，同源就差不多是[这篇文章写的](https://web.dev/same-site-same-origin/)，简单说属于下面其中一种情况及以上就属于跨域：

> - 不同协议( [scheme](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web#Scheme_or_protocol) )，例如：http 与 https
> - 不同域名(host)，例如： www.example.com 与 sub.example.com
> - 不同端口，例如： example.com:80 与 example.com:443

跨域的报错通常是类似下面这种输出：

> Fetch API cannot load http://localhost:4000/api/todos. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

### 发现问题

目标是获取一个非本网站（B网站）的资源，而且必须是通过一个 post 方法去获取，此时必然发生跨域。而我在本地开发时由于经常需要跨域，为了方便发开，我的[浏览器环境是关闭 web security](https://stackoverflow.com/questions/35432749/disable-web-security-in-chrome-48) , 这样我的浏览器与客户端一样的允许跨域。在一开始，我正常的在本地调用接口然后发送请求获取资源，因为本地允许跨域，于是我并没有发现这个跨域的问题，而且在测试服我没有硬编码进行测试。但是到了线上就出问题了，一直提示获取错误，而且大概率就是跨域的问题。这个时候就必须赶紧的解决跨域的问题。这也是这次写作的目的，记录下这次错误的解决过程。

一开始的报错是提示用户获取失败，当看到重现的提示时，我意识到是跨域，当时的提示是下图这样：

![withCredentials](https://raw.githubusercontent.com/wupengfei1/images-storage/master/images/Snipaste_2020-08-31_01-49-18.png)

一看到这个提示，就是跨域的问题，我对这方面的处理非常的不熟悉，因为这里面之前遇到很多hack技巧。

### 问题处理的过程

问题处理的前提是系统调用和生成都是好的，因为老版页面使用都是正常的，而我与老版的调用方式几乎是一样的。

因为上图，我意识到我的跨域配置可能是错的，但是我并不是特别了解所有的跨域的配置。于是首先查找跨域的头，并全部加上。于是我在请求的时候加上了下面的头，这个时候也被提醒到 withCredentials 设置错误。那么全部都要调整，这时候我只想着是否之前老版本在某个地方默默的加上了跨域的头（事后证明是我的问题），之前某个登陆遇到过这个情况。

```js
'Access-Control-Allow-Origin': '*',
'Access-Control-Allow-Methods': '*',
'Access-Control-Allow-Headers':
'Authorization,X-Application-Access-Token, X-Requested-With, Content-Type, Accept',
```



并把 withCredentials 设置为 false，这时候错误也变短了，此时错误提示如下：

```js
Request header field access-control-allow-origin is not allowed by Access-Control-Allow-Headers in preflight response.
```

这时我还侥幸认为的测试服和本地的 localhost 会不一样，然后硬编码放测试服希望能成功获取，而事实上看到报错一样后，我就意识到由于目标服务器允许跨域，而不是白名单那种（CROS），只要是 https 的网站那都是一样的，非 https 的本地环境会提示你混合内容，http与https内容混合在最新浏览器也是不允许的。从这开始我便本地测试。

但是这个报错提示我头不正确，我却无法设置正确的头，因为我无论是否加上或者删除头，不断的排列组合，也无法再突破了一般，报错信息一直都是一样的了。

此时吃饭时，我想到我与他的做法可谓是一模一样了，我得恢复成他的样子，我唯一不同的地方在于我曾经设置过axios默认值，那时候处理登录的跨域问题。那两个默认的配置是这样：

```js
axios.defaults.withCredentials = true;
axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
```

于是我立马把这两个头去掉，但是这也在捉弄我，这时候并没有效果，那时候我想明白然后立马去尝试唯一可能的突破时，还是不行，于是我又陷入了绝境。但是过了一个小时的调整，突然有一刻就好了，与此同时许哥也告诉我是这两个头的原因。而我之前由于去掉头也没能成，我还以为是promise的问题（开始怀疑自己），但是此时我恢复所有代码，然后去掉两个头，这样获取资源立刻一切正常了。

### 最后

为什么需要这两个请求头，首先`withCredentials` 是必要的，因为你需要传递 cookie，不允许则请求不传递 cookie 。而对于`X-Requested-With` 这个是很有历史原因的，甚至可能涉及到微软引入 ajax 时的设计，由于种种原因当时 ajax 默认发这个头，但这个头并不属于任何规范，后来JQuery 默认发送这个头加剧了不规范，现在规范已经不推荐这个头，而且浏览器对发送这个头的请求[视为不安全](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#Simple_requests), 而且现在标准请求已不带，所以需要断舍离。

### 总结

一开始的到后来的方向总体都是对的，但是中间不少插曲都是错误的，比如花了很多时间去排列组合跨域请求头。许哥后面帮助我明白了根源，这里突然本地不知道是什么原因没有刷新好还是其他导致误导我方向。还有由于设置默认，导致报错未能报错到底出在哪个请求头上，我又将 OPTION 请求给过滤了，chrome 认为 OPTION 请求不属于 xhr ， 事后也能明白 OPTION 是浏览器的机制，并不是我们理解的那种请求，虽然也会向服务器去请求，但是你无法主动发一个 OPTION ，或者主动阻止。

##### 对请求头的跨域解决方法

- 首先查看报错，类似这样的是明确的告诉你哪个头不对，那是很好办的，类似这样就很容易发现了：

  ![x-requested-with](https://raw.githubusercontent.com/wupengfei1/images-storage/master/images/Snipaste_2020-08-31_02-41-24.png)

- 假如你不知道哪个头合适或者不合适，那请用chrome浏览器，这时候浏览器会发送一个 OPTION 请求，在这个请求里面会告诉 你`access-control-allow-headers` (如下图)，而你的 request header里面是不是完整的对上，必须是不能多也不能少。没有发现 OPTION 也不要紧张，由于 OPTION 请求是存在消耗的，浏览器并不会每次都会先去 OPTION，chrome 大概在五分钟左右会再次发送。

  ![access-contal-aloww-headers](https://raw.githubusercontent.com/wupengfei1/images-storage/master/images/Snipaste_2020-08-31_02-44-34.png)

- 然后这时候你发现多了头或者还未能找到，一定要注意全局的设置，在所有设置过全局默认值的地方检查。

经过这些基本能确定问题和解决问题。总的来说，主要细心和慢慢寻找，网上有很多这方面的文章可以找到，对于跨域是一个棘手的问题，浏览器大多对 CROS 的机制都是相同的，对待小问题的处理上有点点差异。难点在于他不太好debbug，如果错误信息不完善（太多错误也会暴露服务），就不太好确定错误所在。


