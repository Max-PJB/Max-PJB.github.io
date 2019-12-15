---
title: Spring AOP 心得 
categories: 
  - Spring
  - Java
  - SpringBoot 
tags:
  - java
  - Spring
  - AOP
---

## 简介 
面向对象编程，也称为OOP（即Object Oriented Programming） 
最大的优点在于能够将业务模块进行封装，从而达到功能复用的目的。通过面向对象编程，不同的模板可以相互组装，从而实现更为复杂的业务模块，其结构形式可用下图表示：

![](/public/img/2019-07-17-1.PNG)

面向对象编程解决了业务模块的封装复用的问题，但是对于某些模块，其本身并不独属于摸个业务模块，而是根据不同的情况，贯穿于某几个或全部的模块之间的。例如登录验证，其只开放几个可以不用登录的接口给用户使用（一般登录使用拦截器实现，但是其切面思想是一致的）；再比如性能统计，其需要记录每个业务模块的调用，并且监控器调用时间。可以看到，这些横贯于每个业务模块的模块，如果使用面向对象的方式，那么就需要在已封装的每个模块中添加相应的重复代码，对于这种情况，面向切面编程就可以派上用场了。

面向切面编程，也称为AOP（即Aspect Oriented
Programming），指的是将一定的切面逻辑按照一定的方式编织到指定的业务模块中，从而将这些业务模块的调用包裹起来。如下是其结构示意图：

![](/public/img/2019-07-17-2.PNG)

## AOP的各个扮演者 

### AOP的主要角色

切面：使用切点表达式表示，指定了当前切面逻辑所要包裹的业务模块的范围大小；

Advice：也即切面逻辑，指定了当前用于包裹切面指定的业务模块的逻辑

### Advice 的主要类型

@Before：该注解标注的方法在业务模块代码执行之前执行，其不能阻止业务模块的执行，除非抛出异常；

@AfterReturning：该注解标注的方法在业务模块代码执行之后执行；

@AfterThrowing：该注解标注的方法在业务模块抛出指定异常后执行；

@After：该注解标注的方法在所有的Advice执行完成后执行，无论业务模块是否抛出异常，类似于finally的作用；

@Around：该注解功能最为强大，其所标注的方法用于编写包裹业务模块执行的代码，其可以传入一个ProceedingJoinPoint用于调用业务模块的代码，无论是调用前逻辑还是调用后逻辑，都可以在该方法中编写，甚至其可以根据一定的条件而阻断业务模块的调用；

## 先来看个例子

```java
package com.meicloud.ai.server.platform.config.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * @author Max_Pengjb
 * @Classname AopTest
 * @Description 使用@Aspect注解将此类定义为切面类
 * @Date 2019-7-17 10:55
 */
@Aspect
@Component
public class AopTest {
    private Logger log = LoggerFactory.getLogger(AopTest.class);

    /**
     * 定义一个切点
     *
     * @PointCut注解表示表示横切点，哪些方法需要被横切 value = 切点表达式
     */
    @Pointcut(value = "execution(public * com.meicloud.ai.server.platform.controller.IndexController.*(..)))")
    public void cutaspect() {
    }


    /**
     * @Before注解表示在具体的方法之前执行
     */
    @Before("cutaspect()")
    public void beforerun(JoinPoint joinPoint) {
        log.error("before");
    }

    /**
     * @Around 环绕执行
     */
    @Around("cutaspect()")
    public Object aroundrun(ProceedingJoinPoint pjp) throws Throwable {
        log.error("around 1111111111");
        Object result = pjp.proceed();
        log.error("around 22222222222");
        return result;
    }

    /**
     * @After 注解表示在方法执行之后执行
     */
    @After("cutaspect()")
    public void afterrun(JoinPoint jp) {
        log.error("after ");
    }


    /**
     * @AfterReturning 注解用于获取方法的返回值
     */
    @AfterReturning("cutaspect()")
    public void afterrunt(JoinPoint jp) {
        log.error("after returning ");
    }

    /**
     * @AfterThrowing 程序出错抛出异常执行
     */
    @AfterThrowing(value = "cutaspect()", throwing = "e")
    public void afterrunth(JoinPoint jp, Exception e) {
        log.error("after throwing ");
        log.error(e.getMessage());
        e.printStackTrace();
    }
}
```

> 可以看到我把@PointCut(×××)给单独拎出来了，省的在@Before、@After、@AfterReturn注解中重复写，而是直接用类似于@Before(value="cutaspect()")代替，减少重复性代码;
>
> cutaspect() @Before @After 等后面后面的方法名字可以随便取，没有关系

连接点：
```
@RequestMapping(value = "/", method = RequestMethod.GET)
public String page(@RequestParam(defaultValue = "1") Integer a) {

    log.error("aop yes");
    a = 1 / a;
    return "system/newFirstIndex";
}
```

需要注意的是：

>@Around 需要传入参数 ProceedingJoinPoint pjp 并且 需要执行 pjp.proceed()
>返回 pjp.proceed() 的结果给连接点，否则的话就相当于拦截了返回结果
>
>@AfterThrowing 如果要获取到 异常，需要指定 throwing = "e" 然后再函数中接收

```java
/**
 * @AfterThrowing 程序出错抛出异常执行
 */
@AfterThrowing(value = "cutaspect()", throwing = "e")
public void afterrunth(JoinPoint jp, Exception e) {
    log.error("after throwing ");
    log.error(e.getMessage());
    e.printStackTrace();
}
```

正常运行:

```
2019-07-17 15:19:51.479 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.platform.config.aop.AopTest      : around 1111111111
2019-07-17 15:19:51.480 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.platform.config.aop.AopTest      : before
2019-07-17 15:19:51.502 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.p.controller.IndexController     : aop yes
2019-07-17 15:19:51.502 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.platform.config.aop.AopTest      : around 22222222222
2019-07-17 15:19:51.503 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.platform.config.aop.AopTest      : after 
2019-07-17 15:19:51.503 ERROR 13032 --- [nio-8762-exec-1] c.m.a.s.platform.config.aop.AopTest      : after returning 
```
执行的顺序：

![](../public/img/2019-07-17-3.PNG)

异常运行： 前端传参

```log
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.platform.config.aop.AopTest      : around 1111111111
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.platform.config.aop.AopTest      : before
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.p.controller.IndexController     : aop yes
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.platform.config.aop.AopTest      : after 
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.platform.config.aop.AopTest      : after throwing 
2019-07-17 15:25:52.811 ERROR 13032 --- [nio-8762-exec-2] c.m.a.s.platform.config.aop.AopTest      : / by zero
```

执行顺序：

![](../public/img/2019-07-17-4.PNG)


如果同时有多个 Aspect 类拦截是什么样的呢？


![](../public/img/2019-07-17-5.PNG)

> 想象一下，大圈套小圈，不过至于是谁套谁呢？如果没有经过设置，这都是随机的

![](../public/img/2019-07-17-6.PNG)

如何指定每个 aspect 的执行顺序呢？
方法有两种：

实现`org.springframework.core.Ordered`接口，实现它的`getOrder()`方法

给aspect添加`@Order`注解，该注解全称为：`org.springframework.core.annotation.Order`

不管采用上面的哪种方法，都是值越小的 aspect 越先执行。

比如，我们为 apsect1 和 aspect2 分别添加 @Order 注解，如下：


```
@Order(5)
@Component
@Aspect
public class Aspect1 {
    // ...
}
 
@Order(6)
@Component
@Aspect
public class Aspect2 {
    // ...
}

```

这样就可以确保是 Aspect1 套着 Aspect2 了

>
>注意点
>
>如果在同一个 aspect 类中，针对同一个 pointcut，定义了两个相同的 advice(比如，定义了两个
>@Before)，那么这两个 advice 的执行顺序是无法确定的，哪怕你给这两个 advice 添加了 @Order
>这个注解，也不行。这点切记。


## 切点(pointcut)表达式

### 1. execution
   
   由于Spring切面粒度最小是达到方法级别，而execution表达式可以用于明确指定方法返回类型，类名，方法名和参数名等与方法相关的部件，并且在Spring中，大部分需要使用AOP的业务场景也只需要达到方法级别即可，因而execution表达式的使用是最为广泛的。如下是execution表达式的语法：
   
   ```
   execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
   ```

   这里问号表示当前项可以有也可以没有，其中各项的语义如下：

    modifiers-pattern：方法的可见性，如public，protected；
    
    ret-type-pattern：方法的返回值类型，如int，void等；
    
    declaring-type-pattern：方法所在类的全路径名，如com.spring.Aspect.aoptest；
    
    name-pattern：方法名类型，如buisinessService()；
    
    param-pattern：方法的参数类型，如java.lang.String；
    
    throws-pattern：方法抛出的异常类型，如java.lang.Exception；
       
例： 

```java
@Pointcut(value = "execution(public * com.meicloud.ai.server.platform.controller.IndexController.*(..)))")
```

上述切点表达式将会匹配使用public修饰，返回值为任意类型，并且是
`com.meicloud.ai.server.platform.controller.IndexController`
类中名称为任意方法，方法可以是任意参数。 

上述示例中我们使用了 ` .. ` 和 ` * ` 通配符

`*` 通配符和正则表达式类似，也可以当做前缀通配或者后缀通配符，如
`Package.Test*` 就表示 Package包下所有 以 Test 开头的类

`..` 通配符，有两种使用方法： 

1. 表示子目录   
  
    `@Pointcut(value = "execution(public *
    com.meicloud.ai.server.platform..IndexController.*(..)))")`
    
    比如这里把 Controller 改为 `..` 就会搜索 platform
    目录下和子目录下的任意的 IndexController类来匹配

2. 表示任意参数 

    上例中使用了 `..`表示任意的参数，
    需要注意，表示参数的时候可以在括号中事先指定某些类型的参数，而其余的参数则由`..`进行匹配：
    
    比如： `@Pointcut(value = "execution(public *
    com.meicloud.ai.server.platform..IndexController.*(java.lang.Integer,..,java.lang.String)))")`
    就表示匹配 以 Ingter 开头 String 结尾 中间任意参数的 args 接收

### 2. within

within表达式的粒度为类，其参数为全路径的类名（可使用通配符）.如下面代码的效果与上面excution的例子是一样的：

```java
@Pointcut(value = "within(com.meicloud.ai.server.platform.controller.IndexController)")
```

若要匹配包下的所有类，可以使用 `..`通配符： `within(com.ai..*)`

### 3. args

 args表达式的作用是匹配指定参数类型和指定参数数量的方法，无论其类路径或者是方法名是什么。这里需要注意的是，args指定的参数必须是全路径的。
 也可以使用通配符，但这里通配符只能使用..，而不能使用*。
 
 如下是使用通配符的实例，该切点表达式将匹配第一个参数为java.lang.String，最后一个参数为java.lang.Integer，并且中间可以有任意个数和类型参数的方法：
 
```
@Pointcut(value = "args(java.lang.String,..,java.lang.Integer)")
```

### 4. this和target

this用来匹配的连接点所属的对象引用是某个特定类型的实例，target用来匹配的连接点所属目标对象必须是指定类型的实例；那么这两个有什么区别呢？原来AspectJ在实现代理时有两种方式：
1、如果当前对象引用的类型没有实现自接口时，spring
aop使用生成一个基于CGLIB的代理类实现切面编程
2、如果当前对象引用实现了某个接口时，Spring
aop使用JDK的动态代理机制来实现切面编程
this指示符就是用来匹配基于CGLIB的代理类，通俗的来讲就是，如果当前要代理的类对象没有实现某个接口的话，则使用this；target指示符用于基于JDK动态代理的代理类，通俗的来讲就是如果当前要代理的目标对象有实现了某个接口的话，则使用target.：

```
 public class FooDao implements BarDao { ... }
```

比如在上面这段代码示例中，spring
aop将使用jdk的动态代理来实现切面编程，在编写匹配这类型的目标对象的连接点表达式时要使用target指示符，
如下所示： 

``` 
@Pointcut("target(org.baeldung.dao.BarDao)")
```

如果FooDao类没有实现任何接口，或者在spring
aop配置属性：proxyTargetClass设为true时，Spring
Aop会使用基于CGLIB的动态字节码技为目标对象生成一个子类将为代理类，这时应该使用this指示器：

``` 
@Pointcut("this(org.baeldung.dao.FooDao)")
```

### 5. @Target

这个指示器匹配指定连接点，这个连接点所属的目标对象的类有一个指定的注解:

``` 
@Pointcut("@target(org.springframework.stereotype.Repository)")
```

### 6. @args

这个指示符是用来匹配连接点的参数的，@args指出连接点在运行时传过来的参数的类必须要有指定的注解，假设我们希望切入所有在运行时接受实@Entity注解的bean对象的方法：

``` 
@Pointcut("@args(org.baeldung.aop.annotations.Entity)") 
public void methodsAcceptingEntities() {}
```

为了在切面里接收并使用这个被@Entity的对象，我们需要提供一个参数给切面通知：JointPoint:

``` 
@Before("methodsAcceptingEntities()") 
public void logMethodAcceptionEntityAnnotatedBean(JoinPoint jp) {
  logger.info("Accepting beans with @Entity annotation: " +
  jp.getArgs()[0]); }
```

### 7. @within

@within表示匹配带有指定注解的类,指定匹配必须包括某个注解的的类里的所有连接点：

``` 
@Pointcut("@within(org.springframework.stereotype.Repository)")
```

上面的切点跟以下这个切点是等效的： (下面这个还是少用，不好看）

```
@Pointcut("within(@org.springframework.stereotype.Repository *)")
```

### 8. @annotation

 @annotation的使用方式与@within的相似，表示匹配使用@annotation指定注解标注的方法，区别：这里是指定单个方法上的注解
 
定义一个注解：

```
// 注解类
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface FruitAspect {
}
```

定义aop 切点和环绕

```
@Aspect
public class MyAspect {
  @Around("@annotation(com.business.annotation.FruitAspect)")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("this is before around advice");
    Object result = pjp.proceed();
    System.out.println("this is after around advice");
    return result;
  }
}
```

目标类，连接点

```
// 目标类，将FruitAspect移到了方法上
public class Apple {
  @FruitAspect
  public void eat() {
    System.out.println("Apple.eat method invoked.");
  }
}
```

这里Apple.eat()方法使用FruitAspect注解进行了标注，因而该方法的执行会被切面环绕，其执行结果如下：

```
this is before around advice
Apple.eat method invoked.
this is after around advice
``` 

## 切点表达式组合

可以使用`&&、||、!`三种运算符来组合切点表达式，表示与或非的关系。

```
@Pointcut("@target(org.springframework.stereotype.Repository)")
public void repositoryMethods() {}

@Pointcut("execution(* *..create*(Long,..))")
public void firstLongParamMethods() {}

@Pointcut("repositoryMethods() && firstLongParamMethods()")
public void entityCreationMethods() {}
```
