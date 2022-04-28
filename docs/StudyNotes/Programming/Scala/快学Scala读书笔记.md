

## 控制结构和函数

### if表达式也有值

```scala
val = if (x > 0) 1 else 2
```

scala的公共超类为Any

```scala
if (x>0) 1 else ()
```

这里的()代表“无有用值”的展位符

Unit可以视作Java中的void而非NULL

### 块也有值

块的值就是块中最后一个表达式的值

一个赋值动作本身的也是有值的，类型为Unit，这个类型只有一个值即()

```scala
{r=r*n;n-=1}
x=y=1
```

上述几个值都为()

### Scala中带有局部变量的遮盖效应

![image-20220427153220726](https://images.shrugging.cn/image-20220427153220726.png)

### 高级for循环

* for循环中可以提供多个生成式

  ```scala
  for(i<-1 to3;j<-1 to 3)
  ```

* 每个生成式都可以带上自己的守卫，守卫就是针对前面的生成式的过滤条件

* 如果for循环体以yield开始则会构造出一个集合

* 函数参数可以使用=来指定默认值

### 变长参数

![image-20220427164548803](https://images.shrugging.cn/image-20220427164548803.png)

### 过程

如果函数参数列表后面跟=，代表函数有具体的返回值；如果函数参数列表后面跟{}，代表函数是一个过程，类型为Unit。

### 异常

```scala
val url = "http://www.asdasbaidu.com"
try {
       val httpClient = HttpClients.createDefault()
       val get = new HttpGet(url)
       val response = httpClient.execute(get)
       println(response.getAllHeaders.mkString)
}
catch {
       case _: java.net.UnknownHostException => println("error")
       case ex: Exception => ex.printStackTrace()
}
finally {
       println("finally")
}
```

如果不需要捕获的异常对象，可以使用_代替变量名



```scala
try {
            import collection.JavaConverters._
            val doc: Document = Jsoup.parse(html) // 获取原始doc
            val headElems: Elements = doc.head().getAllElements // 获取head中所有element
            val bodyElems: Elements = doc.body().getAllElements // 获取body所有element
            val bodySize: Int = doc.body().toString.size
            val bodyScriptElems: Elements = doc.body().getElementsByTag("script") // 获取body中script element
            val keywords = Array("location.href = \\\"https://www.baidu.com\\\"",
                "www.hao123.com",
                "location.href= \\\"http://qq.com\\\"",
                "wanwang.aliyun.com/nametrade/",
                "api.share.baidu.com/s.gif",
                "没有找到站点", "安全加密检测", "不支持访问", "微信客户端", "微信里打开", "Welcome to nginx", "404 Not Found", "403 Forbidden", "502 Bad Gateway", "503 Service Temporarily Unavailable")
            val keyWordFlags = ArrayBuffer[Boolean]()
            keywords.foreach(keyWordFlags += html.contains(_))

            // 拼接body中非script的所有元素各自的text
            val textWithoutScript: Set[String] = Set()
            for (elem <- bodyElems.asScala if elem.tagName() != "script") textWithoutScript += elem.ownText()
            val bodyTextWithoutScript: String = textWithoutScript.reduce(_ + _)

            // 获取body中所有元素及其数量集合
            var bodyTagsNum: Map[String, Int] = Map()
            for (elem <- bodyElems.asScala) {
                bodyTagsNum += (elem.tagName() -> (bodyTagsNum.getOrElse(elem.tagName(), 0) + 1))
            }

            val nodExtractV2Flag = {
                if ((for (elem <- headElems.asScala if elem.tagName() == "script")
                    yield elem.hasAttr("src"))
                        .contains(false)
                ) true
                else {
                    bodyScriptElems match {
                        case _ if !bodyScriptElems.isEmpty =>
                            var hasSrcCounter = 0
                            for (elem <- bodyScriptElems.asScala if elem.hasAttr("src")) hasSrcCounter = hasSrcCounter + 1
                            val hasSrcCount = hasSrcCounter
                            bodyScriptElems.size() match {
                                case `hasSrcCount` =>
                                    if ((bodyTagsNum - ("body")).values.reduce(_ + _) <= 10) true
                                    else if ((bodyTagsNum - ("body") - ("script")).size == 1) true
                                    else if (bodyTextWithoutScript.size >= 20 && bodyTextWithoutScript.size <= 300) true
                                    else false
                                case _ => true
                            }
                        case _ => false

                    }
                }
            }
            nodExtractV2Flag match {
                case false => false
                case true => (bodyTagsNum, bodySize, keyWordFlags, html) match {
                    case _ if bodyTagsNum.getOrElse("script", 0) > 10 || bodyTagsNum.values.exists(_ > 20) => false
                    case _ if bodySize > 8000 || bodySize < 100 => false
                    case _ if keyWordFlags.exists(_ == true) => false
                    case _ if {
                        val bytes: Array[Byte] = html.getBytes
                        val i: Int = bytes.length //i为字节长度
                        val j: Int = html.length //j为字符长度
                        i == j
                    } => false
                    case _ => true
                }
            }
        } catch {
            // 捕获所有因为body解析异常而导致的	NullPointerException
            case ex: NullPointerException => false
        }
```

