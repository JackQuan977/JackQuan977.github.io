---
layout:     post
title:      Comparable和Comparator接口
subtitle:   概述
date:       2021-06-06
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - JavaSE
---

# Comparable和Comparator接口

- Comparable和Comparator都是用来实现集合中元素的比较、排序的。

- Comparable是在类内部定义的方法实现的排序，位于java.lang下。
- Comparator是在类外部实现的排序，位于java.util下。
- 实现Comparable接口需要覆盖compareTo方法，实现Comparator接口需要覆盖compare方法。

### Comparable接口

​	此接口强行对实现它的每个类的对象进行整体排序。类实现Comparable接口，重写 int compareTo(To)，这个方法 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数。

​	定义一个实体类：

~~~java
public class Person implements Comparable<Person>{
    private String name;
    private int age;

    public Person(){}

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int compareTo(Person o) {
        return this.getAge() - o.getAge();
    }
}
~~~

​	如果想从小到大排就当前减去要比较的，如果想从大到小排就要比较的减去当前的

测试排序：

~~~java
public class Test02 {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("quanli", 24));
        list.add(new Person("quanli2", 18));
        list.add(new Person("quanli3", 17));
        list.add(new Person("quanli4", 15));
        Collections.sort(list);
        System.out.println(list);
    }
}
~~~

输出结果就会按照年龄从小到大排序

### Comparator接口

​	Comparable接口将比较代码嵌入需要进行比较的类的自身代码中，而**Comparator接口在一个独立的类中实现比较。**

​	重写int compare(T o1, T o2)方法

​	可以在sort方法传入一个比较器，可以用lambda表达式简写

原始写法：

~~~java
public class Test02 {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("quanli", 24));
        list.add(new Person("quanli2", 18));
        list.add(new Person("quanli3", 17));
        list.add(new Person("quanli4", 15));
        Collections.sort(list, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return o1.getAge() - o2.getAge();
            }
        });
        System.out.println(list);
    }
}
~~~

精简写法：

~~~java
public class Test02 {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<>();
        list.add(new Person("quanli", 24));
        list.add(new Person("quanli2", 18));
        list.add(new Person("quanli3", 17));
        list.add(new Person("quanli4", 15));
        Collections.sort(list,(a,b) -> a.getAge() - b.getAge());
        System.out.println(list);
    }
}
~~~

### 总结

 	两种方法各有优劣，用Comparable 简单，只要实现Comparable接口的对象直接就成为一个可以比较的对象，但是需要修改源代码；**用Comparator 的好处是不需要修改源代码，而是另外实现一个比较器，当某个自定义的对象需要作比较的时候，把比较器和对象一起传递过去就可以比大小了。**
