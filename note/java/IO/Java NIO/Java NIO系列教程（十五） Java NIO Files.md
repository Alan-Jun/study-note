## Java NIO系列教程（十六） Java NIO Files

[原文地址 ](http://tutorials.jenkov.com/java-nio/files.html) 译者：章筱虎

转载自[并发编程网 – ifeve.com](https://ifeve.com/)**本文链接地址:** [Java NIO系列教程（十六） Java NIO Files](https://ifeve.com/java-nio系列教程（十六）-java-nio-files/)



java NIO Files类(java.nio.file.Files) 提供了操作文件的相关方法。本篇文章将会覆盖大多数常用的方法。Files类包含了很多方法，如果你需要的功能在文中没有提及，需要自己查阅JavaDoc文档确认，也许Files类提供了相应方法(译者注：但本文中没有涉及)

java.nio.file.Files类需要和java.nio.file.Path一起使用，在学习Files类前，你需要掌握Path类的相关用法。



### Files.exists()

Files.exists()方法用来检查文件系统中是否存在某路径。

Path实例对应的路径可能在文件系统中并不存在。例如，如果打算新建一个文件夹，首先需要创建一个对应的Path实例，然后才能创建对应路径下的文件夹。

因为Path实例对应的路径在文件系统的存在性不确定，可以使用Files.exists()方法确认Path对应的路径是否存在 (也就是开发需要自己显式的去调用该方法确认)。

如下是Files.exists()的示例：

```
Path path = Paths.get("data/logging.properties");

boolean pathExists =
 Files.exists(path,
 new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

示例中首先创建了一个Path。然后，通过调用Files.exists方法并将path作为第一个参数确认path对应的路径是否存在。
注意下Files.exist()方法的第二个参数。第二个参数数组是评判路径是否存在时使用的规则。示例中，数组包含LinkOption.NOFOLLOW_LINKS枚举类型，表示Files.exists不会跟进到路径中有连接的下层文件目录。表示path路径中如果有连接，Files.exists方法不会跟进到连接中去

