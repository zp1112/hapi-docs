### 日志

与任何服务器软件一样，日志记录非常重要。 hapi有一些内置的日志记录方法，以及一些显示这些日志的基本功能。

### 内置方法

有两种几乎完全相同的日志记录方法，server.log（tags，[data，[timestamp]]）和request.log（tags，[data]），只要您想在应用程序中记录事件。