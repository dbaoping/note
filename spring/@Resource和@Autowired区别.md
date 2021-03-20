## 1. 前言

`@Resource`和`@Autowired`注解都可以在**Spring Framework**应用中进行声明式的依赖注入。而且面试中经常涉及到这两个注解的知识点。今天我们来总结一下它们。

## 2. @Resource

全称`javax.annotation.Resource`,它属于**JSR-250**规范的一个注解，包含**Jakarta EE**（**J2EE**）中。**Spring**提供了对该注解的支持。

**该注解使用在成员属性和setter方法上。默认情况下`@Resource`按照名称注入，如果没有显式声明名称则按照变量名称或者方法中对应的参数名称进行注入。**



![Resource注解流程](https://user-gold-cdn.xitu.io/2020/6/8/17291ffc5b4f28a5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果我们希望在目标Bean中体现多态我们可以这样写：

```java
/**
 * 多态的体现.
 */
@Component
public class ResourceTest {
    @Resource
    private ApplicationRunner applicationRunner;    
    @Resource
    private ApplicationRunner runner;
    // ...
}
```

> Qualifier 约束参见 [Spring 注解 @Qualifier 详细解析](https://felord.cn/spring-qualifier.html)

## 3. @Autowired

`@Autowired`通常适用于构造函数，成员变量以及方法上。它的机制是这样的：



![Autowired流程](https://user-gold-cdn.xitu.io/2020/6/8/172918772129d803?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这个注解我们是需要好好聊聊的，日常使用频率相当高。

### 3.1 标注在构造上

通过在目标**Bean**的构造函数上标注就可以注入对应的**Bean**。

```java
package cn.felord;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class AutowiredTest {
    
	private final ApplicationRunner applicationRunner;

    @Autowired
    public AutowiredTest(ApplicationRunner applicationRunner) {
        this.applicationRunner = applicationRunner;
    }
    
}
```

> 从**Spring Framework 4.3**开始，`@Autowired`如果目标Bean只定义一个构造函数，则不再需要在该构造函数上添加`@Autowired`注解。如果目标Bean有几个构造函数可用，并且没有主/默认构造函数，则必须至少有一个构造函数被`@Autowired`标记，以指示Spring IoC容器使用哪个构造函数。

### 3.2 标注在成员变量上

和`@Resource`一样，`@Autowired`也可以标注到目标**Bean**的成员变量上。

```java
@Component
public class AutowiredTest {
    
    @Autowired
    private ApplicationRunner applicationRunner;
    
    // ...
}
```

### 3.3 标注到方法上

一般setter方法上使用的比较多。而且一个 `@Autowired` 支持注入多个参数。

```java
@Component
public class AutowiredTest {

    private ApplicationRunner applicationRunner;
    
    private EmployeeMapper employeeMapper;
    
    private DepartmentMapper departmentMapper;

    /**
     * 注入applicationRunner 
     * 单个参数
     * 
     * @param applicationRunner the application runner
     */
    @Autowired
    public void setApplicationRunner(ApplicationRunner applicationRunner) {
        this.applicationRunner = applicationRunner;
    }

    /**
     * 支持多参数
     *
     * @param employeeMapper   the employee mapper
     * @param departmentMapper the department mapper
     */
    @Autowired
    public void prepare(EmployeeMapper employeeMapper, DepartmentMapper departmentMapper) {
        this.employeeMapper = employeeMapper;
        this.departmentMapper = departmentMapper;
    }

}
```

你以为这就完了？下面这种方式估计大多数人并没有在意过。

```java
@Component
public class AutowiredTest {
    
    // 注入 数组
    @Autowired
    private MovieCatalog[] movieCatalogs;
    
    private Map<String, Movie> movies;
    
    private Set<CustomerPreferenceDao> customerPreferenceDaos;
    
    // 注入 set
    @Autowired
    public MovieRecommender(Set<CustomerPreferenceDao> customerPreferenceDaos) {
        this.customerPreferenceDaos = customerPreferenceDaos;
    }
            
    // 注入 map 
    @Autowired
    public void setMovieCatalogs(Map<String, Movie> movies) {
        this.movies = movies;
    }
   
    // ...
}
```

可以把Bean注入目标Bean的数组、集合容器中去。默认情况下，当给定注入点没有匹配的候选Bean可用时，自动装配将失败。至少应有一个匹配元素。

> 如果您希望元素按照特定顺序排序，则元素Bean可以实现`org.springframework.core.Ordered`接口或者对应注解`@Order`或标准`@Priority`。基于某些机制不建议使用注解方式来排序，否则无法达到预期期望，推荐使用接口`Ordered`。

### 3.4 装配可选

`@Resource`没有提供可选择装配的特性，一旦无法装配则会抛出异常；而`@Autowired`提供了`required`属性（默认值为`true`）以避免这种情况，设置`@Autowired`为`false`。

```java
@Component
public class AutowiredTest {
    // 一旦找不到 movieFinder  不会异常  而初始化为 null
    @Autowired(required = false)
    private MovieFinder movieFinder;
    // ...
}
```

这里也有骚操作，你可以忽略`required`属性。通过 **Java 8**的 `java.util.Optional`来表明候选Bean可选。

```java
@Component
public class AutowiredTest {
public class SimpleMovieLister {
    // 使用 Optional 表明候选Bean可选
    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
     //   ...
    }
}
```

从**Spring 5.0**开始，您还可以使用`@Nullable`注解，这个注解可以你自己实现检测逻辑或者直接使用 **JSR-305**提供的`javax.annotation.Nullable`。

```java
@Component
public class AutowiredTest {
public class SimpleMovieLister {
    // 使用 @Nullable 注解表明候选Bean可选
    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
      //  ...
    }
}
```

## 4. @Inject

从**Spring 3.0**开始，**Spring**提供对**JSR-330**标准注解（依赖注入）的支持。 你需要引入依赖：

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

然后你就可以使用相关的注解来进行依赖注入了，其中主要注解为`@javax.inject.Inject`。大部分情况下该注解都可以代替`@Autowired`使用，但`@Inject`没有`required`属性，不过它也可以与`java.util.Optional`或使用`@Nullable`来达到同样的效果。

> 大部分情况下没有人喜欢额外引入**Jakarta EE**依赖来使用一个已经拥有的功能，**Spring**堵死了**Jakarta EE**依赖注入的生态。

## 5. 总结

`@Resource`和`@Autowired`的优先级顺序不同（参见上图），另外`@Resource`属于 **Jakarta EE**规范而`@Autowired`属于`Spring`范畴，`@Resource`无法使用在构造参数中，`@Autowired`支持`required`属性。从面向对象来说，`@Resource`更加适用于多态性的细粒度注入，而`@Autowired`更多专注于多态的单例注入。`@Inject` 则没必要过多讨论。