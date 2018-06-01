### 路由

当你在hapi中定义一个路由的时候，和其他框架一样，需要三个基本要素，`path`, `method`,`handler`，把这些作为一个对象传递给你的服务，向下面这样：

```js
server.route({
  method: 'GET',
  path: '/',
  handler: function (request, h) {
    return 'Hello!';
  }
});
```

### 方法(Methods)

上面的路由对字符串`Hello/`的`GET`请求做出响应。`method`选项可以是任何有效的HTTP方法或方法数组。比方说，当用户发送PUT或POST请求时，您需要相同的响应，您可以使用以下方法执行此操作：

```js
server.route({
  method: ['PUT', 'POST'],
  path: '/',
  handler: function (request, h) {
    return 'I did something!';
  }
});
```

### 路径(path)

path选项必须是字符串，尽管它可以包含命名参数。要命名路径中的参数，只需用{}包装即可。例如：

```js
server.route({
  method: 'GET',
  path: '/hello/{user}',
  handler: function (request, h) {
    return `Hello ${encodeURIComponent(request.params.user)}!`;
  }
});
```

正如你在上面看到的，我们在我们的路径中有字符串`{user}`，这意味着我们要求将该段路径分配给一个命名参数。这些参数存储在处理程序中的对象`request.params`中。由于我们将参数`user`命名，我们可以在URI编码后使用属性`request.params.user`访问值，以防止内容注入攻击。

### 可选参数

上面的例子中，user参数是必须的，像`/hello/bob`和`/hello/susan`这样的请求才能匹配，像`/hello`就不能匹配。为了使得参数可选，可以在参数名后面加一个问号，向下面这样：

```js
server.route({
  method: 'GET',
  path: '/hello/{user?}',
  handler: function (request, h) {
    const user = request.params.user ?
      encodeURIComponent(request.params.user) :
      'stranger';
    return `Hello ${user}!`;
  }
});
```

现在请求`/hello/mary`将会响应`hello mary!`，请求`/hello`将会响应`hello stranger`。需要注意的是只有最后一个参数可以设置成可选参数，这意味着`/{one?}/{two}/`是一条无效路径，因为在这种情况下，在可选路径之后还有另一个参数。您可能还有一个只覆盖部分路径段的命名参数，例如`/{filename}.jpg`是有效的,如果在它们之间存在非参数分隔符，则每个分段也可以有多个参数，这意味着`/{filename}.{ext}`有效，而`/{filename}{ext}`不是。

### 多个参数

除了可选的路径参数外，还可以允许匹配多个段的参数。为了做到这一点，我们使用星号和数字。例如：

```js
server.route({
  method: 'GET',
  path: '/hello/{user*2}',
  handler: function (request, h) {
    const userParts = request.params.user.split('/');
    return `Hello ${encodeURIComponent(userParts[0])} ${encodeURIComponent(userParts[1])}!`;
  }
});
```

使用此配置，对`/hello/john/doe`的请求将以字符串`Hello john doe`回复。这里需要注意的重要一点是该参数实际上是字符串`“john/doe”`。这就是为什么我们对这个角色进行了拆分以获得两个单独的部分.星号后面的数字表示应为参数分配多少个路径段。您也可以完全忽略该号码，并且该参数将匹配任意数量的可用分段。与可选参数一样，通配符参数（例如`/{files*}`）用于路径中的最后一个参数。

### 处理程序(Handler)方法

处理程序选项是一个接受两个参数，`request`和`h`的函数。

请求参数(request)是一个包含最终用户请求详细信息的对象，例如路径参数，关联的有效内容，认证信息，请求头信息等。有关请求对象包含的完整文档可以在[API参考](https://hapijs.com/api#request-properties)中找到。第二个参数`h`是响应对象，这是一个包含多种用于响应请求的方法的对象。正如你在前面的例子中看到的，如果你想用一些值来回应请求，你只需从处理程序中返回它。有效载荷可以是字符串string，buffer，JSON序列化对象，stream流或promise。

或者，您可以将相同的值传递给`h.response(value)`并从处理程序返回。这个调用的结果是一个响应对象，它可以与其他方法链接在一起，以在响应发送之前改变响应。例如，`h.response('created').code(201)`将发送创建的有效载荷，其HTTP状态码为201。您还可以设置`header`，`content-type`，`content-length`，发送重定向响应以及[API参考](https://hapijs.com/api#response-toolkit)中记录的许多其他内容。

### 配置项(options)

除了以上基本的三个要素以外，您还可以给每个路由添加一个可选的配置项options。这是配置参数验证([[validation](https://hapijs.com/tutorials/validation)])，验证([authentication](https://hapijs.com/tutorials/auth))，先决条件，有效负载处理和缓存配置等事情的地方。更多细节可以在链接教程中找到，也可以在[API参考](https://hapijs.com/api#route-options)中找到。

在这里，我们将看几个旨在帮助生成文档的选项。

```js
server.route({
  method: 'GET',
  path: '/hello/{user?}',
  handler: function (request, h) {
    const user = request.params.user ?
      encodeURIComponent(request.params.user) :
      'stranger';
    return `Hello ${user}!`;
  },
  options: {
    description: 'Say hello!',
    notes: 'The user parameter defaults to \'stranger\' if unspecified',
    tags: ['api', 'greeting']
  }
});
```

从功能上讲，这些选项没有任何作用，但是当使用像[lout](https://github.com/hapijs/lout)这样的插件为您的API生成文档时，它们可能非常有价值。这些数据与路由相关联，并可在稍后用于检查或显示。