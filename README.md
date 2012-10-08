#《Real World Scala》

_by fujohnwang_

你可以认为它是一本入门书，或者是把它当作了解Scala世界里当前主流框架的一个入口，但多少你应该了解一些基础的Scala概念，否则本人不保证你能看懂，：）

本人也不保证这本书可以持续更新甚至书写完成，本人更不保证会一直把它挂在github上，总之，爱看不看，哈哈

## 都讲了些啥？

1. 使用SBT构建Scala应用
2. 通过Play构建Web应用
3. 走进Akka的(并发)世界
4. 使用Slick进行数据访问
5. 使用ScalaTest进行单元测试
6. 使用Scalaz强化函数式编程
7. pending...

## 全是markdown文件，怎么看啊？

安装[pandoc](http://johnmacfarlane.net/pandoc/)，然后运行:

```
$ pandoc -s -N --toc -c css/default.css *.markdown > real_world_scala.html
```

css可以根据自己的胃口调和， 输出的文件名也不用非得叫real_world_scala.html，自己喜欢输出成什么文件名就叫什么文件名， 点开你的输出文件就可以看了， easy as ABC~



