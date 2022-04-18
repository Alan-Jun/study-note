# BitMap、RoaringBitmap与JavaEWAH](http

本文主要介绍BitMap的算法思想，以及开源工具类JavaEWAH、RoaringBitmap的简单用法。

## 一、BitMap

#### 介绍

BitMap使用`bit位`，来标记元素对应的Value。该算法能够`节省存储空间`。

假设一个场景，要存0-7以内的数字[3,5,6,1,2]，尽可能的节省空间。
一种思路就是单纯使用数组存储，但如果数据量放大百万倍甚至千万倍呢，数组的所占用的内存会非常大。
另一种思路是`使用BitMap`。

表示[3,5,7,1,2]，我们可以用8bit的空间来存储，每个数字都在对应的位置中以1的方式表示。

| 位置7 | 位置 6 | 位置 5 | 位置 4 | 位置 3 | 位置 2 | 位置 1 | 位置 0 |
| ----- | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| 1     | 0      | 1      | 0      | 1      | 1      | 1      | 0      |

若将上述BitMap看作是存储`用户的标签`，如`信用卡逾期`标签，位置看成`用户ID`，则若需要查询哪些用户有信用卡逾期的行为（标签），就非常容易查询统计了。

所谓的Bit-map就是用一个bit位来标记某个元素对应的Value， 而Key即是该元素。由于采用了Bit为单位来存储数据，因此在存储空间方面，可以大大节省。
如果说了这么多还没明白什么是Bit-map，那么我们来看一个具体的例子，假设我们要对0-7内的5个元素(4,7,2,5,3)排序（**这里假设这些元素没有重复**）。那么我们就可以采用Bit-map的方法来达到排序的目的。

要表示8个数，我们就只需要8个Bit（1Bytes），首先我们开辟1Byte的空间，将这些空间的所有Bit位都置为0
然后遍历这5个元素，首先第一个元素是4，那么就把4对应的位**置为1**,然后别的位置也是一样
然后我们现在遍历一遍Bit区域，将该位是一的位的编号输出（2，3，4，5，7），这样就达到了排序的目的。
其实就是把计数排序用的统计数组的每个单位缩小成bit级别的布尔数组

**当然具体实现还可以参考 java.util.BitSet 的类的实现，里面是使用的long[] 来实现，源码逻辑不是很复杂，但是位运算需要好好理解一下**

[java.util.BitSet 简单使用demo](https://gitee.com/dyyx/hellocode/blob/master/src/dyyx/test/BitsetTest.java)

## 二、RoaringBitmap

## 文档中怎么说？

#### 文档中怎么说？[#](https://www.cnblogs.com/fonxian/p/10937882.html#2160232400)

> Bitsets, also called bitmaps, are commonly used as fast data structures. Unfortunately, they can use too much memory. To compensate, we often use compressed bitmaps.

BitMap通常被用作快速查询的数据结构，但它太占内存了。解决方案是，`对BitMap进行压缩`。

> Roaring bitmaps are compressed bitmaps which tend to outperform conventional compressed bitmaps such as WAH, EWAH or Concise. In some instances, roaring bitmaps can be hundreds of times faster and they often offer significantly better compression. They can even be faster than uncompressed bitmaps.

Roaring bitmaps是一种超常规的压缩BitMap。它的速度比`未压缩的BitMap`快上百倍。

* [原理讲解的博客](https://blog.csdn.net/yizishou/article/details/78342499)

* [git项目链接](https://github.com/RoaringBitmap/RoaringBitmap)

#### 简单使用

**引入依赖**

```xml

        <dependency>
            <groupId>org.roaringbitmap</groupId>
            <artifactId>RoaringBitmap</artifactId>
            <version>0.8.11</version>
        </dependency>
```

**测试代码**

```java
Copy
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestRoaringbitmap {

    @Test
    public void test(){
        
        //向rr中添加1、2、3、1000四个数字
        RoaringBitmap rr = RoaringBitmap.bitmapOf(1,2,3,1000);
        //创建RoaringBitmap rr2
        RoaringBitmap rr2 = new RoaringBitmap();
        //向rr2中添加10000-12000共2000个数字
        rr2.add(10000L,12000L);
        //返回第3个数字是1000，第0个数字是1，第1个数字是2，则第3个数字是1000
        rr.select(3); 
        //返回value = 2 时的索引为 1。value = 1 时，索引是 0 ，value=3的索引为2
        rr.rank(2); 
        //判断是否包含1000
        rr.contains(1000); // will return true
        //判断是否包含7
        rr.contains(7); // will return false
        
        //两个RoaringBitmap进行or操作，数值进行合并，合并后产生新的RoaringBitmap叫rror
        RoaringBitmap rror = RoaringBitmap.or(rr, rr2);
        //rr与rr2进行位运算，并将值赋值给rr
        rr.or(rr2); 
        //判断rror与rr是否相等，显然是相等的
        boolean equals = rror.equals(rr);
        if(!equals) throw new RuntimeException("bug");
        // 查看rr中存储了多少个值，1,2,3,1000和10000-12000，共2004个数字
        long cardinality = rr.getLongCardinality();
        System.out.println(cardinality);
        //遍历rr中的value
        for(int i : rr) {
            System.out.println(i);
        }
        //这种方式的遍历比上面的方式更快
        rr.forEach((Consumer<? super Integer>) i -> {
            System.out.println(i.intValue());
        });

    }

}
```

## 三、JavaEWAH

**引入依赖**

```xml

            <dependency>
                <groupId>com.googlecode.javaewah</groupId>
                <artifactId>JavaEWAH</artifactId>
                <version>1.1.6</version>
            </dependency>
```

**测试代码**

```java
Copy
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestJavaEWAH {

    @Test
    public void test(){
    
        EWAHCompressedBitmap ewahBitmap1 = EWAHCompressedBitmap.bitmapOf(0, 2, 55, 64, 1 << 30);
        EWAHCompressedBitmap ewahBitmap2 = EWAHCompressedBitmap.bitmapOf(1, 3, 64,1 << 30);
        //bitmap 1: {0,2,55,64,1073741824}
        System.out.println("bitmap 1: " + ewahBitmap1);
        //bitmap 2: {1,3,64,1073741824}
        System.out.println("bitmap 2: " + ewahBitmap2);

        //是否包含value=64，返回为true
        System.out.println(ewahBitmap1.get(64));

        //获取value的个数，个数为5
        System.out.println(ewahBitmap1.cardinality());
        
        //遍历所有value
        ewahBitmap1.forEach(integer -> {
            System.out.println(integer);
        });


        //进行位或运算
        EWAHCompressedBitmap orbitmap = ewahBitmap1.or(ewahBitmap2);
        //返回bitmap 1 OR bitmap 2: {0,1,2,3,55,64,1073741824}
        System.out.println("bitmap 1 OR bitmap 2: " + orbitmap);
        //memory usage: 40 bytes
        System.out.println("memory usage: " + orbitmap.sizeInBytes() + " bytes");

        //进行位与运算
        EWAHCompressedBitmap andbitmap = ewahBitmap1.and(ewahBitmap2);
        //返回bitmap 1 AND bitmap 2: {64,1073741824}
        System.out.println("bitmap 1 AND bitmap 2: " + andbitmap);
        //memory usage: 32 bytes
        System.out.println("memory usage: " + andbitmap.sizeInBytes() + " bytes");

        //序列化与反序列化
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ewahBitmap1.serialize(new DataOutputStream(bos));
            EWAHCompressedBitmap ewahBitmap1new = new EWAHCompressedBitmap();
            byte[] bout = bos.toByteArray();
            ewahBitmap1new.deserialize(new DataInputStream(new ByteArrayInputStream(bout)));
            System.out.println("bitmap 1 (recovered) : " + ewahBitmap1new);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}
```