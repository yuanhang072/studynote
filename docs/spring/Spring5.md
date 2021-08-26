# Spring5概念

是轻量级的开源的JavaEE框架，可以解决企业应用开发的复杂性。

优点：

- 方便解耦，简化开发
- 方便程序调试
- 方便和其他框架整合
- 方便事务操作

两个核心部分：

- IOC：控制翻转，把创建对象过程交给Spring管理。
- AOP：面向切面，不修改源代码进行功能增强。



# IOC容器

## 概念

> 什么是IOC ?  **IOC就是控制反转，把对象创建和对象调用的过程交给spring管理**
>
> 目的是什么？**目的是为了降低耦合度**
>
> 底层原理和思想是什么？**xml解析、工厂模式、反射**

把对象的创建交给对象工厂。

从原本的对象工厂直接new对象，现在采用解析xml文件后通过反射获取对应的类再创建对象

~~~java
//伪代码
public class BeanFactory {
    public User makeBean() throws Exception {
        //step1：通过xml解析获取对应的value值  com.yh.spring.demo.User
        String value = "com.yh.spring.demo.User";
        //step2：获取对应的Class
        Class aa = Class.forName(value);
        //step3：返回对应的对象
        return (User) aa.newInstance();
    }
}
~~~



## 使用IOC容器

1.配置xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.yh.com.spring5.demo1.UserDao"></bean>
</beans>
~~~

2.使用IOC接口读取xml文件获取对象

> 1.BeanFactory：加载配置文件的时候不会创建对象，内部开发使用，不建议对外使用
>
> 2.ApplicationContext：启动项目去加载配置文件的时候直接创建对象。

~~~java
ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
UserDao userDao  = (UserDao)context.getBean("user");
~~~



## IOC管理Bean

> Bean管理指的是2个操作，1、spring创建对象  2、spring属性注入
>
> Bean管理操作两种方式：1、基于xml实现  2、基于注解实现

基于xml实现：

1.创建对象

- 使用bean标签

- id属性：唯一标识

- class属性：类全路径

  创建对象时，调用**无参构造**方法

2.属性注入

> DI:依赖注入，指的就是注入属性

- 使用set方法注入  使用property标签

~~~java
public class UserDao {

    private String name;

    public void add(){
        System.out.println("name:"+name);
    }

    public void setName(String name) {
        this.name = name;
    }
}
~~~

~~~xml
<bean id="user" class="com.yh.com.spring5.demo1.UserDao">
     <property name="name" value="jhon"></property>
</bean>
~~~

- 使用有参方法注入：使用constructor-arg标签

~~~java
public class UserDao {

    private String name;

    public UserDao(String name) {
        this.name = name;
    }

    public void add(){
        System.out.println("name:"+name);
    }
}
~~~

~~~xml
<bean id="user" class="com.yh.com.spring5.demo1.UserDao">
    <constructor-arg name="name" value="ABC"></constructor-arg>
</bean>
~~~

- 注入null值和特殊符号

~~~xml
<bean id="user" class="com.yh.com.spring5.demo1.UserDao">
    <constructor-arg name="name">
        <null></null>
    </constructor-arg>
</bean>
<bean id="user" class="com.yh.com.spring5.demo1.UserDao">
    <constructor-arg name="name">
        <value><![CDATA[<<南京>>]]></value>
    </constructor-arg>
</bean>

~~~

- 内部bean   外部bean    级联赋值（两种方法）
- 注入集合：注入数组、List、Map、Set集合类型
- 集合中设置对象类型
- 提取集合中的注入部分作为共用属性，使用util空间





## IOC操作Bean管理（FactoryBean）

>Springbean 有两种类型的bean
>
>1.普通bean：配置文件中定义的bean类型就是返回类型
>
>2.工厂bean：配置文件中定义的bean类型可以和返回类型不一样

工厂bean实现：

step1：创建一个类，定义成bean，实现FactoryBean接口

step2：实现接口中的方法，在实现的方法里返回真正的bean类型

~~~java
public class Student implements FactoryBean {

    UserDao dao = new UserDao("aa");

    public void read(){
        System.out.println("reading");
    }

    @Override
    public Object getObject() throws Exception {
        return dao;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
~~~

~~~xml
<bean id="stu" class="com.yh.com.spring5.demo1.Student"></bean>
~~~

~~~java
@org.junit.Test
public void test02(){
    ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
    UserDao st = (UserDao) context.getBean("stu");
    st.add();
}
~~~



## IOC操作Bean管理（bean的作用域）

1.在spring里面，设置创建bean实例是单实例还是多实例。

2.默认情况下，bean是单实例对象

3.如何设置单实例还是多实例，通过scope属性来设置，singleton、prototype

4.singleton、prototype的区别

​	singleton，在加载配置文件时，会创建对象

​    prototype，在加载配置文件时，不会创建对象，在getBean时创建对象。

~~~xml
//单实例、也可以不写
<bean id="stu" class="com.yh.com.spring5.demo1.Student" scope="singleton"></bean>
//多实例
<bean id="stu" class="com.yh.com.spring5.demo1.Student" scope="prototype"></bean>
~~~



~~~java
public void test02(){
    ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
    Student st1 = (Student) context.getBean("stu");
    Student st2 = (Student) context.getBean("stu");
    System.out.println(st1);
    System.out.println(st2);
}
//单实例singleton输出结果：
com.yh.com.spring5.demo1.Student@7cbd213e
com.yh.com.spring5.demo1.Student@7cbd213e
    
//多实例输出结果：
com.yh.com.spring5.demo1.Student@2dde1bff
com.yh.com.spring5.demo1.Student@15bbf42f
~~~



## IOC操作Bean管理（bean的生命周期）

1.调用无参构造器

2.调用set方法

3.调用初始化方法

4.使用bean

5.调用销毁方法

bean的后置处理器，创建类，实现BeanPostProcessor接口，实现后置处理器。



## IOC操作Bean管理（xml自动装配）

根据指定装配规则autowire（ByName、ByType），spring自动将匹配的属性值注入

1、根据属性名称注入：一般使用这种  只要保证xml中bean的id和属性name一样即可注入

2、根据属性类型注入：不适用多个同类型的Bean

~~~xml
<bean id="tService" class="com.yh.com.spring5.demo2.TeacherService" autowire="byName"></bean>
<bean id="tdao" class="com.yh.com.spring5.demo2.TeacherDao"></bean>
~~~

~~~java
public class TeacherService {
    TeacherDao tdao;

    public void setTdao(TeacherDao tdao) {
        this.tdao = tdao;
    }

    public void teach(){
        tdao.teach();
    }

}

public class TeacherDao {

    public void teach(){
        System.out.println("Daoteach...");
    }
}

public class test {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
        TeacherService teacherService = (TeacherService)context.getBean("tService");
        teacherService.teach();
    }

}
//测试结果
Daoteach...
~~~





## IOC操作Bean管理（引入外部属性文件）

1.创建外部属性文件，properties格式文件，譬如数据库信息

2.在spring配置文件中，用context 引入外部属性文件，并用${}获取属性值

~~~properties
yh.userName=yuan
yh.psw=111111
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="jdbc.properties"></context:property-placeholder>

    <bean id="jdbc" class="com.yh.com.spring5.demo2.jdbc">
        <property name="userName" value="${yh.userName}"></property>
        <property name="psw" value="${yh.psw}"></property>
    </bean>
</beans>
~~~

~~~java
package com.yh.com.spring5.demo2;

/**
 * @author yuanh
 * @date 2021/8/10  -  16:44
 **/
public class jdbc {
    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setPsw(String psw) {
        this.psw = psw;
    }

    private String userName;
    private String psw;

    public void print(){
        System.out.println("userName"+userName+"&&&psw="+psw);
    }
}

~~~

~~~java
@Test
public void test02(){
    ApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
    jdbc jc = (jdbc)context.getBean("jdbc");
    jc.print();
}

//测试结果：userNameyuan&&&psw=111111
~~~





## IOC操作Bean管理（注解方式）

什么是注解：

1.注解是代码特殊标记、格式是@注解名称（属性名称=属性值，属性名称=属性值）

2.使用注解，注解可以使用在类上、方法上、属性上

3.使用注解的目的是为了简化xml配置



Spring中针对Bean管理中创建对象提供四种注解

1.@Component

2.@Service

3.@Controller

4.@Respository



**基于注解方式实现对象创建**

1.引入依赖，aop包

2.开启组件扫描，context component-scan

3.创建类 添加注解



**基于注解方式实现属性注入**

@Autowired：根据属性类型注入

step1：把service和dao对象创建、在service和dao上添加注解

step2：在service注入dao对象，添加dao类型属性，在属性上面添加注解



@Qualifier：根据属性名称注入

要和@Autowired一起使用，

一个接口，多个实现类的时候，就需要这个



@Resource：可以根据类型也可以根据名称注入，是javax包中的 不建议使用





@Value：注入普通类型属性



## 完全注解开发

（1）创建配置类，替代xml文件

（2）测试中加载配置类，替代配置文件





# AOP

## 概念和原理

面向切面编程，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的[耦合度](https://baike.baidu.com/item/耦合度/2603938)降低，提高程序的可重用性，同时提高了开发的效率。

原理：用动态代理实现

有两种情况使用动态代理：

1.有接口情况：使用JDK动态代理：创建接口类的代理对象，实现功能增强

2.没有接口情况：使用CGLIB动态代理：创建子类的代理对象，实现功能增强



## AOP术语

1.连接点：类里面的哪些方法可以被增强，这些方法被称为连接点

2.切入点：实际被真正增强的方法，称为切入点

3.通知（增强）：实际增强的逻辑部分，被称为通知。通知有多种类型：

- 前置通知
- 后置通知
- 环绕通知
- 异常通知
- 最终通知

4.切面：是一种动作

- 把通知应用到切入点的过程

## AOP操作

1.Spring一般基于AspectJ来实现AOP操作

AspectJ是独立的AOP框架，不属于Spring的组成部分，但是会一起组合使用实现AOP

使用XML或者注解来实现AOP操作

语法：

> execution([权限修饰符] [返回类型] [类全路径名称] [方法名] (参数列表))

## AOP实操

1.创建2个类

~~~java
@Component
public class Person {

    public void add(){
        System.out.println("add...");
    }
}

@Component
@Aspect
public class PersonProxy {

    @Before(value="execution(* com.yh.com.spring5.demo5.Person.*(..))")
    public void addStrong(){
        System.out.println("before...");
    }
}
~~~

配置文件：

~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.yh.com.spring5.demo5"></context:component-scan>
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
~~~

或者配置类：

~~~java
@Configuration
@ComponentScan(basePackages="com.yh.com.spring5.demo5")
@EnableAspectJAutoProxy
public class Demo5Config {
}
~~~



测试：

~~~java
public class test {
    public static void main(String[] args) {
        //使用xml方式
        //ApplicationContext con = new ClassPathXmlApplicationContext("bean6.xml");
        
        //完全使用注解方式
        ApplicationContext con = new AnnotationConfigApplicationContext(Demo5Config.class);
        
        Person p = (Person)con.getBean("person");
        p.add();
    }
}
before...
add...
~~~



## 事务

概念：事务是数据库操作最基本的单元，逻辑上的一组操作，要么都成功，要么都失败。

事务四大特性（ACID）

- 原子性

- 一致性
- 隔离性
- 持久性

在Spring中进行事务管理操作有2种实现方式：编程式事务管理和声明式事务管理

声明式事务管理底层原理基于AOP

有基于注解和基于xml实现
