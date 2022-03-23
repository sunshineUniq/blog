---
title: class extends and implements 的区别
date: 2022-03-10 10:59:41
tags: class, extends and implements
---

## 定义

extends: es6中的关键字，是继承的意思，一般子类继承父类，可以复用父类的属性和方法

implements: 可以理解成当前定义的类通过关键字implements声明自己使用一个或者多个接口。 接口可以是一个或多个

## 使用

```react
class Parent {
    sayHi(): string {
        return `sayHi to you!`
    }
}

class Child extends Parent {
    
}

class Child1 implements Parent {
    sayHi(): string { // 在parent中定义的属性和方法， child1必须有
        return '1'
    }
    
}
```

extends 是继承某个类, 继承之后可以使用父类的方法, 也可以重写父类的方法;
implements 是实现多个接口, 接口的方法一般为空的, 必须重写才能使用.

## 区别

extends 是继承父类，只要那个类不是声明为final或者那个类定义为abstract的就能继承，
继承只能继承一个类，但implements可以实现多个接口，用逗号分开就行了

```react
class A extend B implements C,D,E {

}
```

extends可以覆盖父类的属性和方法； implements，实现父类，子类不可以覆盖父类的方法或者变量。即使子类定义与父类相同的变量或者函数，也会被父类取代掉。

## 接口实现的注意点

a.实现一个接口就是要实现该接口的所有的方法(抽象类除外)

b.接口中的方法都是抽象的。  

c.多个无关的类可以实现同一个接口，一个类可以实现多个无关的接口。

## 总结

需要实现，不可以修改implements
定义接口需要具体实现，或者可以被修改扩展性好，用extends
