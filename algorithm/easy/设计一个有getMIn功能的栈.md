# 申明

这个题目是我在《程序员代码面试指南 IT名企算法与数据结构题目最优解 左程云著》

# 题目

实现一个特殊的栈，在实现栈功能的基础上，在实现返回栈中最小元素的操作。

# 要求

1.  pop，push，getMin操纵的时间复杂度都是O(1)
2. 设计的栈类型可以使用程序语言已有的栈结构

# 解决方案

申明中所说的书中说的解决方案已经很棒了，也很简单，这也是我把他放在简单算法中的原因，解决方案很好理解，**但是看完之后我们有必要做一些自己的思考，有没有办法做优化**

书中提出了两种解决方案，本质是一样的，不过就是基于时间换空间以及空间换时间的一些考量。

**两种方案设计上都使用了两个栈，一个用来存放当前栈中的元素，记作`stackData`,另一个用来记录保存操作每一步的最小值，记作`stackMin`。**

## 第一种方案

### push规则

![1542337471009](assets\1542337471009.png)

### pop规则

![1542338411029](assets\1542338411029.png)

### getMin

上述规则确定之后，我们可以知道`stackMin`栈顶一直存着当前`stackData`栈中的最小值.只需要调用`stackMin`的`peek`就可以了。`peek`方法是获取不移除这个元素的方法

### 代码

```java
package com.stu.algorithm.stack;

import java.util.Stack;

public class MyStack1 {
    private Stack<Integer> stackData;
    private Stack<Integer> stackMin;

    public MyStack1() {
        this.stackData = new Stack<Integer>();
        this.stackMin = new Stack<Integer>();
    }

    public void push(int num) {
        this.stackData.push(num);
        if (this.stackMin.isEmpty()) {
            this.stackMin.push(num);
        } else if (num <= this.getMin()) {
            this.stackMin.push(num);
        }
    }

    public int pop() {
        if(this.stackData.isEmpty()){
            throw new RuntimeException("this stack is empty");
        }
        int value = this.stackData.pop();
        if(value == this.getMin()){
            this.stackMin.pop();
        }
        return value;
    }

    public int getMin() {
        if(this.stackMin.isEmpty()){
            throw new RuntimeException("this stack is empty");
        }
        return this.stackMin.peek();
    }
}
```

## 第二种方案

这种方案就是牺牲空间来减少pop时候的判断动作，换来时间，下面介绍有区别的地方

### push规则

![1542341131521](assets\1542341131521.png)

### pop规则

![1542341580357](assets\1542341580357.png)

### 代码

```java
package com.stu.algorithm.stack;

import java.util.Stack;

public class MyStack2 {
    private Stack<Integer> stackData;
    private Stack<Integer> stackMin;

    public MyStack2() {
        this.stackData = new Stack<Integer>();
        this.stackMin = new Stack<Integer>();
    }

    public void push(int num) {
        if (this.stackMin.isEmpty()) {
            this.stackMin.push(num);
        } else if (num <= this.getMin()) {
            this.stackMin.push(num);
        } else {
            Integer peek = this.stackMin.peek();
            this.stackMin.push(peek);
        }
        this.stackData.push(num);
    }

    public int pop() {
        if (this.stackData.isEmpty()) {
            throw new RuntimeException("this stack is empty");
        }
        this.stackMin.pop();
        return this.stackData.pop();
    }

    public int getMin() {
        if (this.stackMin.isEmpty()) {
            throw new RuntimeException("this stack is empty");
        }
        return this.stackMin.peek();
    }
}

```

# 总结

上述两种方案，我们知道了他们在做的是空间时间取舍，事实上我们在做大多数的算法设计的时候，都是在做这样的取舍，当然互联网时代，我们更愿意用空间去换来时间的减少。因为性能的提高可以比喻成生产力的提高一样。



