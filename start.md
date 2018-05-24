### 开始

> 这个教程适用于hapi v17

### 安装hapi

新建文件夹`myproject`，然后按步骤

* 执行: `cd myproject`进入项目文件夹。
* 执行: `npm init`并根据提示一步一步，最终生成一个package.json文件。
* 执行: `npm install --save hapi@17.x.x`安装hapi作为项目的一个主要依赖。

就这样！你现在已经可以使用hapi创建一个server服务了。

### 创建web服务

最基本的server像这样：

```js
'use strict';
const Hapi = require('hapi');
const server = Hapi.server({
    port: 3000,
    host: 'localhost'
});
const init = async () => {
    await server.start();
    console.log(`Server running at: ${server.info.uri}`);
};
process.on('unhandledRejection', (err) => {
    console.log(err);
    process.exit(1);
});
init();
```

首先，我们引入hapi，然后我们创建一个hapi server实例，传入参数，包含服务要监听的host主机和port端口，最后我们运行server，并打印日志。
当创建一个server的时候，我们还可以提供主机名，IP地址甚至Unix套接字文件，或Windows命名管道来绑定服务器。

### 添加路由

接下来我们需要往我们的server里面添加一个或两个路由来实现一些api，让我们看看如何实现：

```js
'use strict';
const Hapi = require('hapi');
const server = Hapi.server({
    port: 3000,
    host: 'localhost'
});
server.route({
    method: 'GET',
    path: '/',
    handler: (request, h) => {

        return 'Hello, world!';
    }
});
server.route({
    method: 'GET',
    path: '/{name}',
    handler: (request, h) => {

        return 'Hello, ' + encodeURIComponent(request.params.name) + '!';
    }
});
const init = async () => {
    await server.start();
    console.log(`Server running at: ${server.info.uri}`);
};
process.on('unhandledRejection', (err) => {
    console.log(err);
    process.exit(1);
});
init();
```

将上面的代码保存成server.js文件，然后使用命令行`node server.js`执行，浏览器打开`http://localhost:3000`，你就能看到`Hello, world!`，如果你打开`http://localhost:3000/stimpy`你就会看到页面打印`Hello, stimpy!`
注意我们必须要使用URI编码传过来的参数，这是为了防止内容注入攻击。请记住，在不使用编码的情况下使用用户提供的数据绝不是一个好主意！
method参数，可以是合法的HTTP方法中的任意一个，或是一个星号以允许任何方法。路径参数定义了包含参数的路径。它可以包含可选参数，编号参数，甚至通配符。

### 配置静态页面和静态资源

我们已经证明，我们可以用我们的Hello World应用程序启动一个简单的hapi应用程序。接下来，我们将使用一个名为inert的插件来提供静态页面。在开始之前，用CTRL + C停止服务器。

请在命令行运行此命令：`npm install --save inert`,安装inert.

更新init方法

```js
const init = async () => {
    await server.register(require('inert'));
    server.route({
        method: 'GET',
        path: '/hello',
        handler: (request, h) => {

            return h.file('./public/hello.html');
        }
    });
    await server.start();
    console.log(`Server running at: ${server.info.uri}`);
};
```
 `server.register`方法将inert插件加入到hapi应用
 `server.route()`命令注册`/hello`路由，它告诉服务器接受`/hello`的GET请求并回复hello.html文件的内容。注册inert插件后，我们再进行路由注册。在注册插件后再运行依赖于插件的代码通常是明智的，这样在代码运行时可以绝对确定该插件是存在的。

执行`node server.js`启动服务器，然后在浏览器中转到`http://localhost:3000/hello`。不好！由于我们从未创建过hello.html文件，因此出现错误。您需要创建丢失的文件以摆脱此错误。

在目录的根目录下创建一个名为public的文件夹，其中包含名为hello.html的文件。在hello.html里面放入下面的HTML：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hapi.js is awesome!</title>
  </head>
  <body>
    <h1>Hello World.</h1>
  </body>
</html>
```

这是一个简单有效的HTML5文档。
现在在浏览器中重新加载页面。您应该看到“Hello World”
当请求发生时，inert可以提供保存在硬盘上的任何内容，这就是实现这种实时重新加载行为的原因。根据自己的喜好自定义/hello页面。此技术通常用于在Web应用程序中提供图像，样式表和静态页面。

### 使用插件

创建任何Web应用程序都需要的基本功能是访问日志。为了给我们的应用程序添加一些基本的日志记录，我们来加载hapi pino插件。
首先安装`hapi-pino`

```shell
npm install hapi-pino
```

然后更新server.js

```js
'use strict';
const Hapi = require('hapi');
const server = Hapi.server({
    port: 3000,
    host: 'localhost'
});
server.route({
    method: 'GET',
    path: '/',
    handler: (request, h) => {
        return 'Hello, world!';
    }
});
server.route({
    method: 'GET',
    path: '/{name}',
    handler: (request, h) => {
        // request.log(['a', 'name'], "Request name");
        // or
        request.logger.info('In handler %s', request.path);
        return `Hello, ${encodeURIComponent(request.params.name)}!`;
    }
});
const init = async () => {
    await server.register({
        plugin: require('hapi-pino'),
        options: {
            prettyPrint: false,
            logEvents: ['response']
        }
    });
    await server.start();
    console.log(`Server running at: ${server.info.uri}`);
};
process.on('unhandledRejection', (err) => {
    console.log(err);
    process.exit(1);
});
init();
```

现在当server运行时你将看到

```shell
[2017-12-03T17:15:45.114Z] INFO (10412 on box): server started
    created: 1512321345014
    started: 1512321345092
    host: "localhost"
    port: 3000
    protocol: "http"
    id: "box:10412:jar12y2e"
    uri: "http://localhost:3000"
    address: "127.0.0.1"
```

当你在浏览器中访问`http://localhost:3000/`时，控制台将输出具体的访问日志。
logger的更多功能可以在register的参数options中配置。

### 其他


