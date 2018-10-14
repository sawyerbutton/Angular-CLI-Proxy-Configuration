# Angular-CLI-Proxy-Configuration

## Angular CLI 反向代理配置

> 在使用Angular进行本地开发时，与一个或多个后端服务进行链接是非常常见的状况

> 而使用nginx或Kubernetes Ingress这样的反向代理将请求路由到同一域`(domain)`上不同路径后端服务则是应对此种开发场景的常见方案

> 但是因为有了CLI的存在，可以通过简单的配置解决上述问题

## 从例子说起

> 假设存在一个前端坐落于路径`http://mydomain.com/catlog`上

> `Catalog Server`负责提供静态文件`.js，.css，.html`，并提供一个返回环境配置内容的接口给前端

![frontEnd and back server](./assets/1.png)

> 还有两个rest 接口, `Video API’`和`Library API`分别坐落于同一`domain`的`/video/`和`/library/`路径上, 这两个接口提供目录页面期望展示的书籍和电影信息

> 再默认的情况下，Angular CLI 假设前端将会服务在基本路径`/`上，比如 CLI 默认在index.html插入`<base href="/">`作为基础路径

> 比如当浏览器加载`https://mydomain.com/catalog/index.html`时, 浏览器将从`https://mydomain.com/main.bundle.js`请求`main.bundle.js`文件，但是这将会导致一个404错误，因为`main.bundle.js`的文件存在于路径`/catalog`上

> 这个问题可以通过指定`base`锚点(`/catlog`)并将其部署与package.json中即可

```json
{
    "start": "ng serve --base-href /catalog/ --deploy-url /catalog/",
    "build": "ng build --prod --base-href /catalog/ --deploy-url /catalog/"
}
```

_通过在`enviornment.ts`和`environment.prod.ts`中配置额外的属性可以解决某些业务API引用的问题`但并不能解决根本的状况`所以并`不推荐`_

> 如果运行`npm run start`可以生成一个带有`<base href ="/ catalog /">`的index.html文件，并可以通过访问`http://localhost:4200/catalog/`访问该页面

> 类似的状况包括对于css文件的引用，如果在使用CSS引用字体或图像等资源时遇到问题可以通过类似的方式尝试设置

> 至此项目已经更加符合期望的服务和前端在生产中的部署结构,但是本地已经开始处理静态文件的Angular CLI还是不清楚如何处理类似于下述请求
- `GET /video/films`
- `GET /library/books`
- `GET /catalog/config`

> 为了处理这些额外的请求必须配置Angular CLI以将其代理至能够理解并提供响应的服务器

> 需要在Angular-CLI项目的根目录中创建一个文件`proxy.conf.json`，内容如下

```json
{
  "/library/*": {
    "target": "http://localhost:10000",
    "secure": false,
    "logLevel": "debug",
    "changeOrigin": true,
    "pathRewrite": {
      "^/library": ""
    }
  },
  "/video/*": {
    "target": "http://localhost:10001",
    "secure": false,
    "logLevel": "debug",
    "changeOrigin": true,
    "pathRewrite": {
      "^/video": ""
    }
  },
  "/catalog/api/*": {
    "target": "http://localhost:5000",
    "secure": false,
    "logLevel": "debug",
    "changeOrigin": true,
    "pathRewrite": {
      "^/catalog": ""
    }
  }
}
```

> 除此之外，还需要更新`package.json`以在本地运行时引用此文件

```json
{
    "start": "ng serve --base-href /catalog/ --deploy-url /catalog/ --proxy-config proxy.conf.json",
}
```