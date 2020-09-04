# Note to Create-React-App Handling CORS Cross-Domain

**仅针对2.x~3.x的一些情况**

**如何查看 Create-React-App 版本？package.json 中 react-scripts 的版本**

## 原因

当你控制台出现类似如下的错误：

> No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

那么恭喜你，遇到了 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 问题。
如何解决？告诉后端大哥，你遇到了 CORS 问题，需要后端大哥配置一下，然后发现错误不见了。这时候已经解决，不用往下了解。如何配置？我不懂后端，我不会。但是很多时候，后端大哥早早配置了 localhost ，我常常是感受不到这其中的差异。当然你如果只是遇到不会后端配置的问题，估计不会看到这个。

很多时候不能修改配置，那我们前端该怎么办，首先最方便的是装一个插件，去浏览器搜索 CORS 安装安装量最高的那个一般就没错。然而插件并不是万能的，因为插件的原理是“欺骗”浏览器，即给请求加上 [CORS 相关的头](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)，让浏览器认为你现在需要跨域，[这里能测试是否开启了CORS](https://webbrowsertools.com/test-cors/) 。但是由于插件通常设置的参数很多时候不能满足你的要求，比如你需要传递 cookie 但是很多 CORS 插件设置 ` Access-Control-Allow-Origin:*` 在全匹配时，并不能携带 cookie ，MDN说明如下：

> When responding to a credentialed request, the server **must** specify an origin in the value of the `Access-Control-Allow-Origin` header, instead of specifying the "`*`" wildcard.
>
> Because the request headers in the above example include a `Cookie` header, the request would fail if the value of the `Access-Control-Allow-Origin` header were "*". But it does not fail: Because the value of the `Access-Control-Allow-Origin` header is "`http://foo.example`" (an actual origin) rather than the "`*`" wildcard, the credential-cognizant content is returned to the invoking web content.

当你费劲心机时，还是不能传递 cookie ，于是你不得不亲自上阵，手写 CORS相关的头，[类似这位老哥](https://juejin.im/post/5c8faf386fb9a0710a1bbfbe) 。事实上当我感觉需要这么做的时候，我发现我做的是不是不太对了，因为这需要很多 if ，因为你需要判断是不是要被编译成产品。所以很多时候，不是很推荐直接这么干，可能我会先求完后端大哥，我才会这么做。

另一种处理方式，就是利用 chrome 自带的关闭安全模式。首先关闭浏览器，然后使用如下命令打开 chrome

> open -a Google\ Chrome --args --disable-web-security --user-data-dir="~/exmapledir"
> ps: 在 chrome 80 以前 --user-data-dir 可以为空("")，80以后不能为空必须是你电脑上的目录

上面的命令在 Windows 下是类似的，假如 Windows 中你常使用双击打开，那么在图标属性中就可以加入命令。这个时候，其实很多场景下都是够用了。基本上你能传输 cookie 和愉快的跨域。但是，显然这对于日常来说还是有一点点麻烦，特别对于把 chrome 作为主力的日常使用浏览器，首先，使用关闭安全策略的浏览器是不安全的，其次，很多网站不允许用这样的模式，会显示的非常奇怪，例如 gmail。

那么我该如何只针对当前项目进行自我配置的跨域呢？那么我只有选择在 create-react-app 中进行配置。

## Create-React-App中配置

在CRA(Create-React-App)配置的好处很多，因为你只需要你自己的项目能够跨域，而并非真要在跨域的环境下，毕竟浏览器这个安全策略目的还是保护用户，如果你经常跨域浏览器其他网站，那么你是非常危险的。这个原理是利用你的本地服务器进行请求代理，因为跨域仅存在浏览器，本地服务器是不存在跨域的，而当你 `npm start `时会在本地开启一个 `webpack-dev-server ` ，而配置跨域就是利用这个本地服务器去代理；配置跨域代理后，你会发现你直接请求的接口全部会去请求 localhost 服务器，而这是同源的不存在跨域，本地的开发服务器会去请求真正的请求并返回数据，于是这个项目看起来可以跨域一样，这样是可以携带cookie的。

##### 开启跨域

首先是最常见的，也是最简单的，就是在 package.json 加上 proxy 字段，类似如下：

```json
"proxy":"https://api.proxy.com"
```

但是实际上你很难会真的成了，因为CRA 2.0以后这方面就被限制了，首先不像之前 proxy 字段可以是对象，现在是必须是 string ，而你很多跨域的需要是必须要其他配置（例如 changeOrigin ），所以这配置虽然保留下来，实际上已经没用了。

那么现在该咋办？现在就需要用 [CRA 推荐的代理方式](https://create-react-app.dev/docs/proxying-api-requests-in-development/#configuring-the-proxy-manually) ，目前最新版本是 3.4.1 ，目前大概是这样的步骤：

```js
yarn add http-proxy-middleware --save

// 添加 src/setupProxy.js 文件 并写入如下的内容

const { createProxyMiddleware } = require('http-proxy-middleware');
module.exports = function(app) {
    app.use(
    '/api',
    createProxyMiddleware({
      target: 'https://api.proxy.com',
      changeOrigin: true,
    })
  );
};
```

这时候应该就可以代理成功了，看看是否代理。什么，还没好？

这里注意几点：

- 你和目标服务器都是 https 吗，如果目标是 https 而 localhost 是 http 那混合起来浏览器也是不允许，而且启动时会提示你代理失败。
- `/api`这样的需要代理的 path ，你需要填写多少个。
- 假如 src 目录下存在 setupProxy.js 那么必然会用到，如果避免代理，例如我在 http 情况下就不想代理

这几个问题都是非常恼人，这样完全跟着官方是不行的，必须根据实际情况选择代理，再次回去看 CRA 文档、http-proxy-middleware 模块文档、express 文档（本地服务器）,结合起来才能解决。

- 对于 https，需要启动时加入 https 的参数，使用 ` HTTPS=true react-scripts start` 启动
- 对于 router path 匹配，graphql 仅仅需要配置 `/graphql` 就可以，这个优势很大。其他的推荐[http-proxy-middleware 文档](https://www.npmjs.com/package/http-proxy-middleware) ，通用匹配是存在的。
- 对于不想走代理则需要对 express 中间件(middleware)的了解，了解 express 中间件，那么代理本质也是一个中间件。

于是最终的配置文件就是：

```js
const { createProxyMiddleware } = require('http-proxy-middleware');

const target = process.env.REACT_APP_API;
const HTTPS = process.env.HTTPS;

module.exports = (app) => {
  // 当不是 https 环境 则不走代理
    if (!HTTPS) {
        return app.use((res, req, next) => {
            next();
        });
    }
    return app.use(
    // paths 是数组
        paths,
        createProxyMiddleware({
            target,
            changeOrigin: true
        })
    );
};
```

现在终于完成。

### 总结

对于 CROS 最好的方法是后端进行配置，类似白名单，因为这是浏览器允许的，也是推荐的。在种种情况仅仅开发环境需要跨域，那么很多时候由于配置的修改和情况很少（大多并不需要你配置，对 nginx 可能就一行的事情），导致网络上种种跨域手段并不是那么好用，所以经历了这个跨域的过程，但是由于各种情况，配置一次后很可能就忘了，那么几个月后会再回来看看这个记录。跨域如果需要你自己想办法了，那么你就会遇到各种问题，是个棘手的问题，而且只有遇到才会想到去解决的。
