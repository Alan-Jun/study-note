标记接口 `RandomAccess` 他的作用其实是用来利用`instanceof`来判断哪一个类实现了`RandomAccess`

比如 `ArrayList` 和 `LinedList` 其中`ArrayList` 就实现了这个`RandomAccess` 接口，表示它的随机访问快，因为是数组实现的，这时候比如我们在做便利的时候就能根据这个值来判断了，`RandomAccess` 使用for(int i=0;i.....)循环 比较快，非`RandomAccess`使用它的迭代器便利会更快，比如`LinedList`,要是使用fori这种循环，通过get(i)访问那就很慢，因为每一次get(i)都需要在链表中做链路查找