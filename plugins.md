### 插件

hapi拥有一个广泛而强大的插件系统，可让您轻松将应用程序分解为独立的业务逻辑和可重复使用的实用程序。

### 创建一个插件

插件是非常容易编写的。他们的核心代码为一个对象，有一个register属性，register是一个`async function (server, options)`这样的async函数，另外，插件对象必须要有name属性和几个可选属性，包括版本version。

一个非常简单的插件像下面这样：

```js
'use strict';

const myPlugin = {
  name: 'myPlugin',
  version: '1.0.0',
  register: async function (server, options) {
    // Create a route for example
    server.route({
      method: 'GET',
      path: '/test',
      handler: function (request, h) {
          return 'hello, world';
      }
    });
    // etc ...
    await someAsyncMethods();
  }
};
```

或者当作为外部模块写入时，你可以加一个pkg属性。

```js
'use strict';

exports.myPlugin = {
  name: 'myPlugin',
  version: '1.0.0',
  register: async function (server, options) {
    // Create a route for example
    server.route({
      method: 'GET',
      path: '/test',
      handler: function (request, h) {
          return 'hello, world';
      }
    });
    // etc ...
    await someAsyncMethods();
  }
};
```

注意在第一个例子中，我们需要明确写上name和version属性，然而在第二个例子中，我们用package.json的内容作为它的值来设置一个pkg参数。两种方法都可以接受。

当作为模块写入时，插件可以是顶级模块导出，即`module.exports = {register，name，version}`，或者如果您希望模块导出的不仅仅只有一个hapi插件，它可以导出为`ex​​ports.plugin = {register，name，version}`。此外，插件对象可以包含`multiple`属性，当设置为`true`时，告诉hapi在同一个server中多次注册插件。

另一个可选属性是`onece`。当设置为`true`时，意味着hapi会忽略同一插件的多次注册而不会引发错误。

### 注册方法

正如我们上面所看到的，注册方法接受两个参数，`server`和`options`。

`options`参数只是在调用`server.register(server, options)`时用户传递给插件的任何选项。不做任何更改，并将对象直接传递给您的register方法。

register应该是一个异步函数，一旦插件完成了注册所需的任何步骤，它就会return返回。或者，如果注册插件时发生错误，您的注册插件应该会引发错误。

server对象就是您当前正开启的server对象的引用。

### 加载一个插件

插件可以一次加载一个，也可以一次加载数组形式的多个插件，使用server.register方法，比如：

```js
const start = async function () {
  // load one plugin
  await server.register(require('myplugin'));
  // load multiple plugins
  await server.register([require('myplugin'), require('yourplugin')]);
};
```

为了把参数options传递给插件，我们可以使用object的形式将`plugin`和`options`作为属性，例如像这样。

```js
const start = async function () {
  await server.register({
    plugin: require('myplugin'),
    options: {
      message: 'hello'
    }
  });
};
```

这些对象也可以通过数组来传递：

```js
const start = async function () {
  await server.register([{
    plugin: require('plugin1'),
    options: {}
  }, {
    plugin: require('plugin2'),
    options: {}
  }]);
};
```

### 注册选项

您也可以将第二个可选参数传递给`server.register()`。这个对象的文档可以在[API参考](https://hapijs.com/api#-await-serverregisterplugins-options)中找到。

options对象由hapi使用，不传递给正在加载的插件。它允许您将vhost或prefix应用于插件注册的任何路由。

例如，我们有一个插件是这样的：

```js
'use strict';

exports.plugin = {
  pkg: require('./package.json'),
  register: async function (server, options) {
    server.route({
      method: 'GET',
      path: '/test',
      handler: function (request, h) {
        return 'test passed';
      }
    });
    // etc...
    await someAsyncMethods();
  }
};
```

通常，当这个插件被加载时，它会在`/test`处创建一个GET路由。这可以通过在`options`中使用`prefix`设置进行更改，这将在插件中创建的所有路由前添加一个字符串。

```js
const start = async function () {
  await server.register(require('myplugin'), {
    routes: {
      prefix: '/plugins'
    }
  });
};
```

现在，当插件被加载时，由于设置了`prefix`选项，插件将会创建一个`GET`请求的`/plugins/test`路径。

类似地，`options.routes.vhost`属性将为正在加载的插件创建的任何路由分配默认的虚拟主机配置，更多细节可以[参考这个API说明](https://hapijs.com/api#-serverrouteroute)