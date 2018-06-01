### 日志

与任何服务器软件一样，日志记录非常重要。 hapi有一些内置的日志记录方法，以及一些显示这些日志的基本功能。

### 内置方法

hapi有两种日志记录方法，`server.log(tags，[data，[timestamp]])`和`request.log(tags，[data])`，只要您想在应用程序中记录事件。可以在路由处理器，请求生命周期扩展或认证方案中调用`request.log()`，可以在服务器启动之后或在插件的`register()`方法内使用`server.log()`

它们都接受相同的前两个参数，tags和data。

tags是一个字符串数组，里面的元素标识各个事件，把它们想象成日志级别，但更具表现力。例如，您可以标记从数据库中检索数据的错误，如下所示：

```js
server.log(['error', 'database', 'read']);
```

hapi在内部生成的任何日志事件都会有与之相关的hapi标签。

第二个参数data是一个可选字符串或对象，用于记录事件。这就是您要传递错误消息或其他任何希望与您的代码一起使用的细节的地方。

另外`server.log()`接受第三个时间戳参数。这个默认值为`Date.now()`，并且只有在需要由于某种原因而需要覆盖默认值时才能传入。

### 检索并显示日志

hapi的server对象为每个日志事件发出事件。您可以使用标准EventEmitter API来监听这些事件并根据需要显示它们。

```js
server.events.on('log', (event, tags) => {
  if (tags.error) {
    console.log(`Server error: ${event.error ? event.error.message : 'unknown'}`);
  }
});
server.events.on('request', (event, tags) => {
  if (tags.error) {
    console.log(`Request error: ${event.error ? event.error.message : 'unknown'}`);
  }
});
```

使用`server.log()`记录的事件将发出以上那个`log`事件，使用`request.log()`记录的事件将发出`request`事件。

您可以通过`request.logs`一次性检索特定请求的所有日志。这将是一个包含所有记录的请求事件的数组。您必须首先在路由上将`log.collect`选项设置为true，否则此数组将为空。

```js
server.route({
  method: 'GET',
  path: '/',
  options: {
    log: {
      collect: true
    }
  },
  handler: function (request, h) {
    return 'hello';
  }
});
```

### 调试模式（仅在开发模式下）

hapi有一个调试模式，这是一种让你的日志事件输出到控制台的方式，无需自己配置额外的插件或编写日志代码。

默认情况下，打印到控制台的唯一错误调试模式是用户代码中未捕获的错误，以及hapi API的错误实现导致的运行时错误。但是，您可以将服务器配置为基于标签打印请求事件。例如，如果您想在请求中打印任何错误，您可以按如下方式配置服务器：

```js
const server = Hapi.server({ debug: { request: ['error'] } });
```

### 日志插件

hapi提供的用于检索和打印日志的内置方法非常简单。为了获得更多功能丰富的日志记录体验，您可以考虑使用像
[good](https://github.com/hapijs/good)或[bucker](https://github.com/nlf/bucker)这样的插件。