# Lambda表达式

### 函数式接口

Lambda表达式的语法：

([Lambda参数列表，即形参列表]) -> {Lambda体，即方法体}

拷贝小括号，写死右箭头，落地大括号，大括号中写上业务逻辑

@Functionalnterface

default

静态方法

特点：使用 "->"将参数和实现逻辑分离；( ) 中的部分是需要传入Lambda体中的参数；{ } 中部分，接收来自 ( ) 中的参数，完成一定的功能。


什么是函数式接口？

即SAM（Single Abstract Method ）接口，有且只有一个抽象方法的接口（可以有默认方法或者是静态方法和从Object继承来的方法，但是抽象方法有且只能有一个）。 JDK1.8之后，添加@FunctionalInterface表示这个接口是是一个函数式接口，因为有了@functionalInterface标记，也称这样的接口为Mark（标记）类型的接口。

只有函数式接口的变量和函数式接口才能赋值Lam表达