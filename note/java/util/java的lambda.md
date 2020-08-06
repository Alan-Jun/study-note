https://www.cnblogs.com/gu-bin/p/11225749.html

lambda表达式可以说是Java 8最大的卖点，它将函数式编程引入了Java。Lambda允许把函数作为一个方法的参数，或者把代码看成数据（对于这一点会用js的人应该不陌生）。

## 语法

* (parameters) -> expression

```java
list.forEach(item -> System.out.println(item));
```

* (parameters) ->{ expressions}

```java
list.forEach(item -> {
  int numA = item.getNumA();
  int numB = item.getNumB();
      System.out.println(numA + numB);
});
```

总结起来就是：左侧指定 lambda 表达式需要的参数，右侧指定 lambda 方法体

## 重要特征

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

```java
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```

## sorted

https://www.cnblogs.com/kuanglongblogs/p/11230250.html

// order by id,age desc

```java
package com.kinson.stream;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SortedDemo {

    public static void main(String[] args) {
        ArrayList<Student> students = new ArrayList<>(8);
        Student student = new Student(1,28);
        Student student1 = new Student(1,24);

        Student student2 = new Student(3,27);
        Student student3 = new Student(3,23);

        Student student4 = new Student(4,31);
        Student student5 = new Student(4,25);
        Student student6 = new Student(4,20);

        students.add(student4);
        students.add(student6);
        students.add(student5);
        students.add(student2);
        students.add(student3);
        students.add(student);
        students.add(student1);
        // order by id,age desc
        List<Student> collect = students.stream().sorted(Comparator.comparingInt(Student::getId).thenComparing(Student::getAge,Comparator.reverseOrder())).collect(Collectors.toList());
        System.out.println(collect);
    }


}

class Student{

    private int id;

    private int age;

    public Student(int id, int age) {
        this.id = id;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", age=" + age +
                '}';
    }
}
```