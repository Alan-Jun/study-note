BIO , NIO ，AIO

**同步IO:**

因为不管是BIO,还是NIO（new I/O ）人也不叫做非阻塞IO, NIO只是说是使用了监听事件的方式来实现同步这个概念，本质上它还是需要忙寻不断的等待事件（数据）的触发，和BIO等待数据是一个道理。所以：在**IO上，同步和非阻塞是互斥的，所以不存在同步非阻塞IO。同步IO一定是阻塞的，NIO的实现只是说它不会在一个IO操作上阻塞，他在等待一系列监听事件上是阻塞的，拿到可用事件开始处理数据的时候也是阻塞的**

**异步IO:**

1. 在等待数据的过程中，用户线程继续执行，在拷贝数据的过程中，线程在阻塞，这就是**异步阻塞IO**。
2. 在等待数据的过程中，和拷贝数据的过程中，用户线程都在继续执行，这就是**异步非阻塞IO**。

第一种情况是，用户线程没有参与数据等待的过程，所以它是异步的。但用户线程参与了数据拷贝的过程，所以它又是阻塞的。合起来就是异步阻塞IO。

第二种情况是，用户线程既没有参与等待过程也没有参与拷贝过程，所以它是异步的。当它接到通知时，数据已经准备好了，它没有因为IO数据而阻塞过，所以它又是非阻塞的。合起来就是异步非阻塞IO。



NIO 提供了“非阻塞” 和多路复用的同步IO处理方式

1. 非阻塞

   ```java
   public class NioServer {
       private static List<SocketChannel> channelList;
       public static void main(String[] args) throws IOException {
           ServerSocketChannel channel=ServerSocketChannel.open();
           channel.socket().bind(new InetSocketAddress(9000));
           channel.configureBlocking(false);//非阻塞
           System.out.println("server启动成功");
           while (true){
               SocketChannel socketChannel=channel.accept();
               if(socketChannel!=null){
                   System.out.println("有客户端连接");
                   socketChannel.configureBlocking(false);//设置读取为非阻塞
                   channelList.add(socketChannel);
               }
               Iterator<SocketChannel> i=channelList.iterator();
               while (i.hasNext()) {
                   SocketChannel sc= i.next();
                   ByteBuffer byteBuffer=ByteBuffer.allocate(128);
                   int len=sc.read(byteBuffer);
                   if(len>0){
                       System.out.println("接收到消息"+new String(byteBuffer.array()));
                   }else if(len==-1){
                       i.remove();
                       System.out.println("客户端断开连接");
                   }
               }
           }
       }
   }
   ```

2. 多路复用的同步IO处理方式

   ```java
   public class NioSelectorServer {
       public static void main(String[] args) throws IOException {
           ServerSocketChannel channel=ServerSocketChannel.open();
           channel.socket().bind(new InetSocketAddress(9000));
           channel.configureBlocking(false);//非阻塞
           Selector selector=Selector.open();
           channel.register(selector, SelectionKey.OP_ACCEPT);//selector注册连接事件
           System.out.println("server启动成功");
           while (true){
               selector.select();//阻塞等待事件发生
               Set<SelectionKey> selectionKeys=selector.selectedKeys();//获取全部事件
               Iterator<SelectionKey> iterator=selectionKeys.iterator();
               while (iterator.hasNext()){//遍历处理所有事件
                   SelectionKey selectionKey=iterator.next();
                   if(selectionKey.isAcceptable()){//有客户端连接
                       SocketChannel socketChannel=((ServerSocketChannel)selectionKey.channel()).accept(); // 拿到新的客户端段
                       socketChannel.configureBlocking(false);
                       socketChannel.register(selector,SelectionKey.OP_READ);//注册监听该客户端的数据读取事件
                   }else if(selectionKey.isReadable()){//接收到消息
                       SocketChannel sc= (SocketChannel)selectionKey.channel();
                       ByteBuffer byteBuffer=ByteBuffer.allocate(128);
                       int len=sc.read(byteBuffer);
                       if(len>0){
                           System.out.println("接收到消息"+new String(byteBuffer.array()));
                       }else if(len==-1){
                           sc.close();
                           System.out.println("客户端断开连接");
                       }
                   }
                   iterator.remove();//去掉处理过的事件
               }
           }
       }
   }
   ```

多路复用IO和“非阻塞” IO的区别是什么呢？ 

1. “非阻塞IO” 他在忙寻中每次循环都需要自己去检查哪些是就绪的，如果没有则跳过，有就处理

2. 多路复用是监听一些列事件，他在忙寻的过程中获取到的是已经就绪的事件，也就是可以直接处理数据读

   也就是说多路复用IO,他在操作系统层面帮助你将上面的找就绪事件这件事做了，并且epoll 还是使用的mmp 技术，性能会高很多



**有了多路复用IO,我们可以降低整体的一个线程数量，一定程度的缩减了我们的线程数量，避免服务端的线程数被耗光**

这一点我们的 BIO 是办不到的，它如果只是用一个线程，那么整个服务的吞吐量会降低很多，因为accept ，read， write 都是阻塞的，单线程处理的话会造成大量的阻塞：A的read需要等待B的read,

然后如果我们给 read，write 使用多线程, 那也是每一个 read,wirte

# 我觉得的BIO和NIO的区别我想到的一个点是：

1. NIO 被称为非阻塞IO的本质应该是在使用非阻塞模式的时候，不会向BIO一样。一个IO连接的accept ，需要等待前一个IO的accept，这样导致了我们处理连接的能力成为了严重的性能瓶颈，同时也不需要在应用层去new线程来处理 read造成的阻塞问题，来快速accept。

   NIO的的非阻塞模型可以在read 的时候快速返回，数据未就绪直接返回失败，就绪再处理，这样即使我们不采取创建线程的方式，也能够避免read的阻塞带来的处理性能下降问题。

   NIO的多路复用IO是基于事件模型的，他可以一次监听很多连接事件（accept），同时处理多个连接，不会出现一个accept 阻塞另一个这样的场景，极大的提高我们的客户端的链接的处理速度

2. 我们可以对selector : service channel 的监听事件做1对多的方式（每个selector 监听100个链接，甚至还能对读写事件做再拆分），然后实现降低线程数的目标，因为cpu在这个场景并不是性能瓶颈，这样做可以一定程度的缩减我们的线程数量，避免服务端的线程数被耗光

3. 使用多路复用IO 一次可以监听多个连接的事件意味着他可以减少很多的系统调用（多个文件描述符只有一次 select 的系统调用 + n 次就绪状态的文件描述符的 read 系统调用） ，BIO 每个连接都是一次就绪的系统调用+read的系统调用

4. NIO 提供了对直接内存的使用，也提供了mmap,sendFile 的支持可以在特定场景极大的提高我们的IO性能

> **NIO 也是同步IO , 同步的IO其实就一定阻塞的，因为在真实处理数据（read,write）的时候他是阻塞的，我们的NIO本身也不叫 NO blocking IO,而是 new IO ；  现在人们说的NIO非阻塞主要是说的在数据未就绪的时候它可以快速返回响应，不会让线程在这里阻塞等待**



BIO ,NIO demo https://blog.csdn.net/dingruibao/article/details/121198976

IO多路复用：https://mp.weixin.qq.com/s/CMWlDywI1zbgJSoeGTBmuw

