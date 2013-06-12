# 通过Play构建Web应用

## Play框架简介
Play打破Java界web开发的戒律， 抛弃servlet模型， 转而效仿ROR并且自己实现所有东西包括server， 协议处理，变成模型等，总的来说，提高了开发效率，同时生产环境性能也很不错。

I am not like following things in Play:

1. template engine handling and default template syntax;
	- the template should be replacable, but currently it's bound to only one impl.
	- even the template solution is aimed to be treated as scala code, they are not looks like scala code
	- template is template, the framework should be view-specific, context and merge part can be made flexible, I think
2. if sbt is used, why don't follow the convention of it by putting souce code and resources under src/main(although configuration can be adjusted to do that)?
3. it's maybe contradiction, but I am still think a framework should give people opportunities to choose their preferred tools, say, Data access framework.

but I should say that `WS`, `Promise`, `OpenId` APIs are cool.


## Play的安装与配置

下载，并解压到任何目录， 将解压目录加到PATH环境变量中， DONE！

## 开始Play
	$ play new [first_app]
	$ cd [first_app]
	$ play run

## Play基础篇

### Action

Action（play.api.mvc.Action）的概念很简单，可以简单理解为只是一个Request=>Result的函数， 即接收到客户端HTTP请求之后，将其转换成某种处理结果并返回。

当然， 具体要复杂一些， Action的定义如下：

```scala
trait Action[A] extends (Request[A] => Result) {
  def parser: BodyParser[A]
}
trait Request[+A] extends RequestHeader {
  def body: A
}
```

Action拥有一个BodyParser用来将具体的HTTP请求体(request body)转换为指定的类型A，然后再构建成强类型的Request[A]以传给开发者使用，一般情况下，开发者不需要关注请求体是如何解析的，除非有特殊需求，这个时候才需要自己实现相应的BodyParser。 



### Result

Result有许多现成的实现类或者helper companion object可以用，比如Ok， TODO， BadRequest等.

这些实现类可以根据传入的参数自动设置HTTP Response的Headers， 比如Content-Type， 但我们也可以通过相应的方法调用自定义设置相应的Headers， 比如:

```scala
val htmlResult = Ok(<h1>Hello World!</h1>).as("text/html")
val htmlResult = Ok(<h1>Hello World!</h1>).as(HTML)
Ok("Hello World!").withHeaders(
  CACHE_CONTROL -> "max-age=3600", 
  ETAG -> "xx"
)
Ok("Hello world").withCookies(
  Cookie("theme", "blue")
)
```

### Routes

Routes是一堆Router的集合， 而每一个Router的任务也很简单，也是做函数转换，即：

	(HTTP Method Type, Request URI) => Action

也就是说，每一个route规则都会根据HTTP请求的Method类型加上请求的路径， 将当前HTTP请求转给相应Action进行处理（每个Action定义了具体的处理逻辑， remember？）

HTTP Method包括：

1. GET
2. POST
3. PUT
4. DELETE
5. HEAD

请求路径格式：

1. 静态路径(Static Path)
	- `GET   /clients/all              controllers.Clients.list()`
2. 动态路径(Dynamic Path) - 以`:`, `*`, `$`作为起始标志
	- `GET   /clients/:id          controllers.Clients.show(id: Long)`
	- `GET   /files/*name          controllers.Application.download(name)`
	- `GET   /clients/$id<[0-9]+>  controllers.Clients.show(id: Long)`

Action的调用部分， 如果action方法的参数是String类型，则不需要声明参数类型； 如果声明了方法的参数类型，比如id:Long，则Play将根据声明的类型进行类型转换。

如果请求的路径中没有声明相应的参数，而action方法有调用参数，则该参数将从QueryString中查找并转换。

Route可以从HTTP Method类型+请求的URI转换为相应的Action，Route也可以从相应的Action转换成对应的HTTP Method类型+请求URI，这称之为reverse route， 比如：

```scala
// Redirect to /hello/Bob
def helloBob = Action {
    Redirect(routes.Application.hello("Bob"))    
}
```

### Session And Flash 

都以Cookie形势存储，所以有存储限制（4k），同时意味着也可以操纵Headers的形势来操纵它们， 比如：

```scala
Ok("Welcome!").withSession(
  "connected" -> "user@gmail.com"
)
Ok("Hello World!").withSession(
  session + ("saidHello" -> "yes")
)
Ok("Theme reset!").withSession(
  session - "theme"
)
```

```scala
def index = Action { implicit request =>
  Ok {
    flash.get("success").getOrElse("Welcome!")
  }
}
def save = Action {
  Redirect("/home").flashing(
    "success" -> "The item has been created"
  )
}
```

其中， Flash与Session的区别在于， Flash的生命周期只延续到下一个请求，而Session则跨越多个请求。一般只用Flash来简单的传递某些成功或者错误信息。

### 模版引擎与模版
TBD


## Play进阶篇
1. 异步化
2. 插件化
3. 其他

### Play中的异步化HTTP编程(Async HTTP Programming in Play)

简单点儿来说， 异步化编程就是从原来的直接同步执行逻辑并返回Result的Action实现，转向可以异步执行处理逻辑并返回AsyncResult的Action实现。

AsyncResult可以通过`Async{…}`方法，从某个`Promise[Result]`实例来构建。 一个Promise从字面上来理解就是它会保证将来不一定某个时间点会返回你期望的处理结果， 并且Promise与Promise之间可以组合以串联执行：

```scala
val promiseOfPIValue: Promise[Double] = computePIAsynchronously()
val promiseOfResult: Promise[Result] = promiseOfPIValue.map { pi =>
  Ok("PI value computed: " + pi)    
}
```
而创建一个Promise最常用的方式就是直接将处理逻辑丢出去给另外一个线程去处理，比如通过Akka:

```scala
val promiseOfInt: Promise[Int] = Akka.future {
  intensiveComputation()
}
```

综上， 一个简单的异步Action实现就看起来是如此的样子：

```scala
def index = Action {
  val promiseOfInt = Akka.future { intensiveComputation() }
  Async {
    promiseOfInt.map(i => Ok("Got result: " + i))
  }
}
```
fucking simple!



> __有关异步话题的延伸__
> 
> 将一个线程内的执行逻辑丢给另一个线程去处理的异步化方式，实际上，从本质上来讲不会带来太多的收益（原因自己想）， 只有整个执行的pipeline内所有处理步骤都异步化，并且结合IO多路复用等措施，才会带来可观的收益(比如最直接的收益就是可以提高单机的硬件资源利用率，在大集群的情况下，可以大幅削减硬件以及运维开销)，否则，做的异步化很多时候是在干拆东墙补西墙的勾当，当然啦，笔者并不排除有些处理调度规划恰当的情况下的异步化的合理性。

### Streaming HTTP Programming
TBD

## 问题
1. 资源的管理，服务实例的注册与使用在Play中通常是如何做的？ 推荐什么样的实践方式？！
2. Play中所有controller中的action方法定义为static的目的是啥？！



## Play扩展篇
	实例待定， 构建基于JMX的监控web应用？
	外汇交易系统雏形？！
	Finance Desk Dashboard？！

