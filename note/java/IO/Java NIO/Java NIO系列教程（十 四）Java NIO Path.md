# Java NIO系列教程（十四）Java NIO Path

[原文链接](http://tutorials.jenkov.com/java-nio/path.html) 译者：章筱虎

转载自[并发编程网 – ifeve.com](https://ifeve.com/)**本文链接地址:** [Java NIO系列教程（十 五）Java NIO Path](https://ifeve.com/java-nio-path-2/)



Path接口是java NIO2的一部分。首次在java 7中引入。Path接口在java.nio.file包下，所以全称是java.nio.file.Path。 java中的Path表示文件系统的路径。可以指向文件或文件夹。也有相对路径和绝对路径之分。绝对路径表示从文件系统的根路径到文件或是文件夹的路径。相对路径表示从特定路径下访问指定文件或文件夹的路径。相对路径的概念可能有点迷糊。不用担心，我将在本文的后面详细介绍相关细节。

不要将文件系统的path和操作系统的环境变量path搞混淆。java.nio.file.Path接口和操作系统的path环境变量没有任何关系。

在很多方面，java.nio.file.Path接口和[java.io.File](http://tutorials.jenkov.com/java-io/file.html)有相似性，但也有一些细微的差别。在很多情况下，可以用Path来代替File类。

**创建Path实例**

为了使用java.nio.file.Path实例，必须首先创建它。可以使用Paths 类的静态方法Paths.get()来产生一个实例。以下是示例：

```
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {

public static void main(String[] args) {

Path path = Paths.get("c:\\data\\myfile.txt");

}
}
```

请注意例子开头的两个import语句。想要使用Paths类和Path接口，必须首先引入相应包。其次，注意Paths.get(“c:\\data\\myfile.txt”)的用法。其使用了Paths.get方法创建了Path的实例。它是一个工厂方法。

### 创建绝对路径Path

调用传入绝对路径当做参数的Paths.get()工厂方法，就可以生成绝对路径Path。示例如下：

```
Path path = Paths.get("c:\\data\\myfile.txt");
```

示例中的绝对路径是c:\data\myfile.txt。有两个\字符的原因是第一个\是转义字符，表示紧跟着它的字符需要被转义。\\表示需要向字符串中写入一个\字符。

上文示例的path是windows下的路径。在Unix系统(Linux,MacOS,FreeBSD等)中，上文中的path是这样的：

```
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

/home/jakobjenkov/myfile.txt就称作绝对路径。

如果把以/开头path的格式运行在windows系统中，系统会将其解析为相对路径。例如：

```
/home/jakobjenkov/myfile.txt
```

将会被解析为路径是在C盘。对应的绝对路径是：

C:/home/jakobjenkov/myfile.txt

### 创建相对路径Path

相对路径指从一个已确定的路径开始到某一文件或文件夹的路径。将确定路径和相对路径拼接起来就是相对路径对应的绝对路径地址。

java NIO Path类也能使用相对路径。可以通过Paths.get(basePath, relativePath)创建一个相对路径Path。示例如下：

```
Path projects = Paths.get("d:\\data", "projects");

Path file = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一个例子创建了一个指向d:\data\projects文件夹的实例。第二个例子创建了一个指向 d:\data\projects\a-project\myfile.txt 文件的实例。

当使用相对路径的时候，可以使用如下两种特别的符号。它们是：

- .
- ..

.表示当前路径。例如，如果以如下方式创建一个相对路径：

```
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

创建的Path实例对应的路径就是运行这段代码的项目工程目录。

如果.用在路径中，则其表示的就是当前路径下。示例：

Path currentDir = Paths.get(“d:\\data\\projects\.\a-project”);
对应的就是如下路径
d:\data\projects\a-project

..表示父类目录。示例：

Path parentDir = Paths.get(“..”);

Path对应的路径是当前运行程序目录的上级目录。

如果在path中使用..，表示上级目录的含义。例如：

```
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```

对应的绝对路径地址为：

d:\data\projects\another-project

在a-project目录后面的..符号，将指向的目录修改为projects目录，因此，最终path指向another-project目录。

.和..都可以在Paths.get()的双形参方法中使用。示例：

```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project",
 "..\\another-project");
```

下面介绍NIO 的Path类有关相对路径的其他使用方法。

### Path.normalize()

Path 的normalize()方法可以标准化路径。标准化的含义是路径中的.和..都被去掉，指向真正的路径目录地址。下面是Path.normalize()示例：

```
String originalPath =
 "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

上文示例，首先创建了一个包含..字符的路径地址。之后输出此路径。

之后，调用normalize方法，返回一个新的path对象。输出新对象的路径。

输出结果如下：

```
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```

如你所见，标准化后的路径不再包含 a-project\..部分，因为它是多余的。

