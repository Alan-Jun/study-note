# 基本介绍

我们首先需要知道的是堆的结构是一种安全二叉树的结构。**它利用完全二叉树的结构来维护一组数据，可以细分为`最大堆`和`最小堆`**

* **最大堆也就是说它的每一个父* 节点都要大于它的子节点**

  **root 一定是真个堆集合中最大的**

  

* **最小堆也就是说它的每一个父节点都要小于它的子节点**

  **root 一定是真个堆集合中最小的**

既然说了堆就是一个完全二叉树，那么就需要了解它的性质

**若设二叉树的深度为h，除第h 层外，其它各层(0～h-1)的结点数都达到最大个数，第h 层所有的结点都连续集中在最左边，这就是完全二叉树。我们知道二叉树可以用数组模拟，堆自然也可以。**

* **具有n个结点的完全二叉树的深度为|log<sub>2</sub>n|+1**
* **已知一个完全二叉树的节点树等于 n， 那么他的非叶子节点数量就是 n/2的整数部分值**

* **其中如果完全二叉树用数组来存储，（如果数组从0位置开始存）那么具有以下性质**
1. 父节点的下标是`parent`，左孩子的下标就是`2parent+1`，右孩子下标就是`2*parent+2`.
  
2. `i=0`,那就是 `roo`t, `i>0` 那么他的双亲结点的编号=`(i-1)/2的结果的整数部分`，比如(2-1)/2 = 0，(4-1)/2=1,这个结果就是它的parent的index
  
3. 由上面的非叶子节点的数量的性质，可以知道在从零开始存放的数组中，非叶子节点的索引值都小于非叶子节点的数量值
  4. 如果 2i+1>n,则这个结点没有左孩子，否则他的左孩子编号 = 2i+1
  5. 如果 2i+2>n,则这个结点没有右孩子，否则他的左孩子编号 = 2i+2
  
* **如果从数组从1这位置开始存**

  1. 如果父节点的下标是 `index`,左孩子的下标就是`2*index`，右孩子下标就是`2*index+1`.

  2. `i =1` 那他就是`root`，`i>1`那么他的双亲结点的编号=`i/2的结果的整数部分` 比如 3/2=1，5/2=2
  3. 由上面的 非叶子节点的数量的性质，可以知道再从零开始存放的数组中，非叶子节点的索引值都小于等于非叶子节点的数量值
  4. 如果 2i>n,则这个结点没有左孩子，否则他的左孩子编号 = 2i
  5. 如果 2i+1>n,则这个结点没有右孩子，否则他的左孩子编号 = 2i+1

# 建堆

我们不管是建立最大堆还是最小堆，会有几种情况呢？

1. 堆集合中，数据插入，或则空堆进行数据插入
2. 堆集合的数据数据删除
3. 将已有集合变成堆

**堆集合中，数据插入，或则空堆进行数据插入** 

这种情况，举个例子比如我们现在要建立一个最小堆集合，比如我们按照 3，6，8，15，9，30，那么不需要做任何操作，他就是一个很好的最小堆，但是如果我们按照15，30，9，3，8，6 那么就需要做**上浮操作**了，比如当我们插入9的时候，这时候需要和parent也就是15做比较，9比15小，所以需要交换他们的位置，也就是上浮了；我们继续看，我们插入3，3的parent是30（怎么定位paren，上文中解释完全二叉树的性质的时候说了 ）,这时候3需要上浮，上浮之后发现，它比它的parent 8 还是要小，继续上浮，那么除了不需要上浮的判断情况，如果可以上浮的话，上浮到 root 的时候就可以结束了。

这样就可以得出我们的 插入函数，以及上浮函数，函数实现这里使用 java的`PriorityQueue` 的方法

```java
// 插入方法：
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)// 数组容量不够需要扩容
        grow(i + 1);
    size = i + 1;
    if (i == 0)// root节点不需做上浮的动作
        queue[0] = e;
    else
        siftUp(i, e);// 上浮操作
    return true;
}
// 上浮方法
private void siftUp(int k, E x) {
    // 根据是否传入了比较器，做上浮方法的选择，其实逻辑都一样，就是做比较器的选用罢了
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {// 极端情况如果需要一直上浮，到root位置也就会结束了
        int parent = (k - 1) >>> 1;// 获取parent 节点
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)// 如果新的值key 大于 parent 的e 那就不需要上浮
            break;
        queue[k] = e;// 如果新的值key 小于 parent 的e ， 将它当前的paren 的 e  和它现在的位置交换
        k = parent;// 上浮
    }
    queue[k] = key;// 将值放到最总上浮到的位置
}
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {// 极端情况如果需要一直上浮，到root位置也就会结束了
        int parent = (k - 1) >>> 1;// 获取parent 节点
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)// 这里默认是最小堆的实现，所以如果新的值x 大于 parent 的e 那就不需要上浮。如果要实现最大堆的话，我们只需要重写自己的 comparator 就可以直接实现最大堆了
            break;
        queue[k] = e;// 如果新的值key 小于 parent 的e ， 将它当前的paren 的 e  和它现在的位置交换
        k = parent; // 上浮
    }
    queue[k] = x;// 将值放到最总上浮到的位置
}
```

**堆集合的数据数据删除**

加入我们现在有 [3，6，8，15，9，30] 这样一个最小堆堆，如果我们删除3，最简单的操作就是将3和堆中最后一个位置交换位置，然后再删除3， 比如这里就是变成了 [30，6，8，15，9，3] ，删除3就是堆的最后一个叶子几点，也就是简化成了数组最后一个元素的删除。删除之后得到的就是 [30，6，8，15，9]，这时候不符合我们的堆的结构特性了，那么 30这个元素需要**下沉**, 因为这里是最小堆，所以需要先找到30的子节点中最小的节点，也就是 6 ， 然后让30个6交换位置得到 [6，30，8，15，9]， 30 还是大于他们子节点，同样的方式找到子节点中最小的，交换上来最终得到[6，9，8，15，30]。大致就是这样，但是再写代码的时候我们需要确定循环什么时候结束，下沉结束标志，除了不需要继续下沉，被直接break结束，还有就是会一直下沉到叶子节点的位置才能结束了。根据完全二叉树的性质，**已知一个完全二叉树的节点树等于 n， 那么他的非叶子节点数量就是 n/2的整数部分值 `non-leaf-num`，从零开始存放的数组中，叶子几点的索引值都小于`non-leaf-num`**

这样就可以得出我们的删除，以及下沉函数，函数实现这里使用 java的`PriorityQueue` 的方法

```java
public E poll() {
    if (size == 0)
        return null;
    int s = --size;// 获取尾部的索引
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    // 上面的三段代码就是将root和堆的最末尾的节点交换位置
    if (s != 0)
        siftDown(0, x); // 开始下沉操作
    return result;
}

@SuppressWarnings("unchecked")
private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;// 获取尾部的索引
    if (s == i) // removed last element
        queue[i] = null;
    else {
        E moved = (E) queue[s];// 获取尾部数据
        queue[s] = null;
        siftDown(i, moved);
        if (queue[i] == moved) {
            siftUp(i, moved);
            if (queue[i] != moved)
                return moved;
        }
    }
    return null;
}
// 下沉函数
private void siftDown(int k, E x) {
   if (comparator != null)
       siftDownUsingComparator(k, x);
   else
       siftDownComparable(k, x);
}

private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf， 得到非叶子节点的数量
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;// 右孩子
        if (right < size && // 操作size 那就是没有右孩子
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0) // 如果左孩子大于右孩子，那就用右孩子的值
            c = queue[child = right];
        // 上面的步骤都是为了找到最小的孩子的位置和值
        if (key.compareTo((E) c) <= 0)// key 比它的孩子都要小，就不需要下沉了
            break;
        queue[k] = c;//交换位置
        k = child;
    }
    queue[k] = key;// 最后key的位置处，放入key
}

@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}

```

**将已有集合变成堆**

那么它需要对每一个非叶子节点调用下沉操作函数，从末尾的叶子节点开始，一直到root，具体需不需要下沉交给我们的函数去判断， **时间复杂度近似 O(N)**

函数实现这里使用 java的`PriorityQueue` 的方法

```java
private void heapify() {
	//  (size >>> 1) - 1 拿到最末尾的一个非叶子节点，从这个节点开始，调用下沉操作函数siftDown，从末尾的叶子节点开始，一直到root
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}
```





```
class Solution {
 public int lengthOfLongestSubstring(String input) {
        if(input.isEmpty()){
        	return 0;
        }
        if(input.length() == 1 ){
        	return 1;
        }
        int [] set = new int[128];
        int left =0;
        int max = 0;
        for(int right=0;right<input.length();right++){
        	if(set[input.charAt(right)]!=0){
            	left= set[input.charAt(right)];
            }
        	max =Math.max(max,right-left+1);
        	set[input.charAt(i)]=i+1;
        }
        return max;
    }
}
```

