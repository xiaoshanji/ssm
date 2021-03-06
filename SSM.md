#### 

# **SSM(Spring + SpringMVC + Mybatis)**

# 一、Spring

## **1、核心理论：**

​		Spring核心容器就是一个超级大工厂，所有的对象(包括数据源、Hibernate  SessionFactory等基础性资源)都会被当成Spring核心容器管理的对象————Spring把容器中的一切对象统称为Bean。

​		所谓的Bean，与Java Bean不同，Java  Bean必须遵守一些特定的规范，而**Spring对Bean没有任何要求**，只要**是一个Java类，Spring就可以管理该Java类，并把它当成Bean处理**。(一切 Java对象都是Bean)。

```java
public class Axe
{
    public String chop()
    {
        return "使用斧头砍柴";
    }
}
public class Person
{
    private Axe axe;
    public void setAxe(Axe axe)
    {
        this.axe = axe;
    }
    public void useAxe()
    {
        System.out.println("我打算去砍点柴火！");
        /**
        *	一个对象需要调用另一个对象的方法的情形，称为依赖。
        */
        System.out.println(axe.chop());
    }
}
public class Main
{
    public static void main(String [] args)
    {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person p = ctx.getBean("person",Person.class);
        p.useAxe();
    }
}
```





```xml
<beans xmlns:xsi="....">
	<bean id="person" class="....Person">
    	<property name="axe" ref="axe" />
    </bean>
    <bean id="axe" class="....Axe" />
    <bean id="win" class="javax.swing.JFrame" />
    <bean id="date" class="java.util.Date" />
</beans>
```

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.1.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>5.1.8.RELEASE</version>
    </dependency>
```

#### 1.1、基于xml来管理容器中的Bean：

​		根元素<beans .../>，包含多个<bean .../>元素，每个<bean .../>元素定义一个Bean。<bean .../>元素默认驱动Spring在底层调用无参数的构造器创建对象。

​		解析时会执行类似如下的过程：

```java
String idStr = ...; //解析<bean .../>元素的id属性的到该字符串值
String classStr = ...; // 解析<bean .../>元素的class属性得到字符串值，所以当指定class属性时必须是					完整类名，不能是抽象类，接口(除非有特殊配置)；
Class clazz = Class.forName(classStr);
Object obj = clazz.newInstance();
//container代表Spring容器
container.put(idStr,obj);
```

​		**结论：**

​				在Spring配置文件中配置Bean时，**class属性的值必须是Bean实现类的完整类名(必须带包名)，不能		是接口，不能是抽象类(除非有特殊配置)，否则Spring无法使用反射创建该类的实例**。

​		**<property .../>元素通常用于作为<bean .../>元素的子元素**，它驱动Spring在底层以反射执行一个setter方法，其中**name属性决定执行哪个setter方法，而value或ref决定执行setter方法传入的参数**。如果**传入参数是基本类型及其包装类、String等类型**，则**使用value属性**指定传入参数。以**容器中其他Bean作为传入参数**，则**使用ref属性**指定传入参数。

​		此时会执行类似如下的代码：

```java
String  nameStr = ...; // 解析<property .../>的name属性得到该字符串值
String  refStr = ...;  //解析<property .../>的ref属性得到该字符串值
String setterName = "set" + nameStr.subString(0,1).toUpperCase() + nameStr.subString(1);
Object paramBean = container.get(refStr);
Method setter = clazz.getMethod(setterName,paramBean.getClass());
setter.invoke(obj,paramBean);
```

#### 1.2、获取Bean：

​		通过Spring容器来访问容器中的Bean。ApplicationContext是Spring容器最常用的接口，两个实现类：

​				1）、ClassPathXmlApplicationContext：从类加载路径搜索配置文件，并根据配置文件来创建Spring容					器(常用)。

​				2）、FileSystemXmlApplicationContext：从文件系统的相对路径或绝对路径下去搜索配置文件，并根					据配置文件来创建Spring容器。

```java
public class BeanTest
{
    public static void main(String[] args) throws Exception
    {
    	ApplicationContext ctx = new ClassPathXmlApplicationContext("....xml");
        Person p = ctx.getBean("key",Person.class);
        p.useAxe();
    }
}
```

![](image/QQ截图20190617153646.png)

​		获取Bean对象的两个方法：

​				Object  get(String  id)：根据容器中的Bean的id来获取指定的Bean，获取Bean之后需要进行强制类型转			换。

​				T  getBean(String  name,Class<T>  requiredType)：根据容器中的Bean的id来获取指定Bean，但该方			法带一个泛型参数，因此获取Bean之后无须进行强制类型转换。

​		使用Spring框架之后最大的改变之一是：**程序不再使用new调用构造器创建 Java对象那个，所有的 Java对	象都由Spring容器负责创建**。

## **2、依赖注入**

​		几乎所有的 Java应用中大量存在着A对象需要调用B对象方法的情形，这种情形被Spring称为依赖，即A对象依赖B对象。它们总是由一些互相调用的对象构成的，Spring把这种互相调用的关系称为依赖关系。

​		Spring的核心功能：

​				1）：Spring容器作为超级大工厂，负责创建、管理所有的 Java对象，这些 Java对象被称为Bean。

​				2）：Spring容器管理容器中Bean之间的依赖关系，Spring使用一种被称为“**依赖注入**”的方式管理Bean		之间的依赖关系。

​		使用依赖注入，不仅可以为Bean注入普通的属性值，还可以注入其他Bean的引用。通过这种依赖注入，应用中各种组件不需要以硬编码方式耦合在一起，甚至无须使用工厂模式。

​		当某个 Java对象(调用者)需要调用另一个 Java对象(被依赖对象)的方法时

​				传统模式：

​						原始做法：调用者**主动**创建被依赖对象，然后再调用被依赖对象的方法。需要用过“new  被依赖对				象构造器();”的代码创建对象。导致调用者与被依赖对象实现类的硬编码耦合。不利于项目维护。

​						简单工厂模式：调用者先找到被依赖对象的工厂，然后**主动通过工厂去获取被依赖对象**，最后再调				用被依赖对象的方法。把握三点：

​								1）：调用者面向被依赖对象的接口编程。

​								2）：将被依赖对象的创建交给工厂完成。

​								3）：调用者通过工厂来获得被依赖组件。

​								唯一缺点：调用组件需要主动通过工厂去获取被依赖对象，会带来调用组件与被依赖对象工厂						的耦合。

​		**控制反转（依赖注入）**：Spring之后，调用无须主动获取被依赖对象，调用者只要被动接受Spring容器为调用者的成员变量赋值即可（只要**配置一个<property .../>子元素，Spring就会执行对应的setter方法为调用者的成员变量赋值**）。

​		依赖注入两种方式：

​				1）：**设值注入**：Ioc容器使用**成员变量的setter方法**来注入被依赖对象。

​				2）：**构造注入**：Ioc容器**使用构造器**来注入被依赖对象。

​		Spring推荐面向接口编程，不管是调用者，还是被依赖对象，都应该为之定义接口，程序应该面向它们的接口，而不是面向实现类编程。

### 2.1、设值注入(推荐使用)

```java
public interface Axe
{
    public String chop();
}
public interface Person
{
    public void useAxe();
}

public class Chinese implements Person
{
    private Axe axe;

    public void setAxe(Axe axe) {
        this.axe = axe;
    }

    @Override
    public void useAxe() {
        System.out.println(axe.chop());
    }
}


public class SteelAxe implements Axe
{

    @Override
    public String chop() {
        return "钢斧砍柴真快";
    }
}

public class StoneAxe implements Axe
{

    @Override
    public String chop() {
        return "斧头砍柴好慢";
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="beans.Person">
        <property name="axe" ref="axe" />
    </bean>
    <bean id="axe" class="beans.Axe" />
    <bean id="chinese" class="beans.Chinese">
        <!--<property name="axe" ref="steelAxe"></property>-->
        <property name="axe" ref="stoneAxe"></property>
    </bean>
    <bean id="stoneAxe" class="beans.StoneAxe" />
    <bean id="steelAxe" class="beans.SteelAxe" />
</beans>
```

### 2.2、构造注入

​		<bean  .../>元素默认总是驱动Spring调用无参数的构造器来创建对象，而**<constructor-arg .../>子元素代表一个构造器参数**，如果<bean  .../>元素**包含N个<constructor-arg  .../>子元素**，就会驱动Spring**调用带N个元素的构造器来创建对象**。

```java
package beans;

import beans_interface.Axe;
import beans_interface.Person;

public class Chinese implements Person
{
    private Axe axe;

    public void setAxe(Axe axe) {
        this.axe = axe;
    }
    public  Chinese(Axe axe)
    {
        this.axe = axe;
    }
    

    @Override
    public void useAxe() {
        System.out.println(axe.chop());
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="beans.Person">
        <property name="axe" ref="axe" />
    </bean>
    <bean id="axe" class="beans.Axe" />
    <bean id="chinese" class="beans.Chinese">
        <!--<property name="axe" ref="steelAxe"></property>-->
        <!--<property name="axe" ref="stoneAxe"></property>-->
        <constructor-arg ref="steelAxe" />
    </bean>
    <bean id="stoneAxe" class="beans.StoneAxe" />
    <bean id="steelAxe" class="beans.SteelAxe" />
</beans>
```

## **3、使用Spring容器**

### 3.1、Spring容器

​		两个核心接口：BeanFactory和ApplicationContext。其中ApplicationContext是BeanFactory的子接口。

![](image/QQ截图20190618003405.png)

​		BeanFactory负责配置、创建、管理Bean，它有一个子接口：ApplicationContext，因此也被称为Spring上下文。

![](image/QQ截图20190617225243.png)

![](image/QQ截图20190618003607.png)

​	创建Spring容器的实例时，必须提供Spring容器管理的Bean的详细配置信息，Spring的配置信息通常采用xml配置文件来配置，因此，创建BeanFactory实例时，应该提供XML配置文件作为参数，XML配置文件通常使用Resource对象传入。Resource接口是Spring提供的资源访问接口，通过该接口，Spring能以简单、透明的方式访问磁盘、类路径以及网络上的资源。

​		获取BeanFactory：

```java
private static Resource isr;
    private static DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

    /**
     * 实例化BeanFactory
     * @param xmlPathName
     */
    public static void getBeanFactory(String xmlPathName)
    {
        isr = new ClassPathResource(xmlPathName);
//        isr = new FileSystemResource(xmlPathName);
        new XmlBeanDefinitionReader(beanFactory).loadBeanDefinitions(isr);
    }

    /**
     * 获取Spring容器
     * @param xmlPathName
     * @return
     */
    public static ApplicationContext getApplicationContext(String... xmlPathName)
    {
//        return new FileSystemXmlApplicationContext(xmlPathName);
        return new ClassPathXmlApplicationContext(xmlPathName);
    }
```

### 3.2、使用ApplicationContext

​		除了提供BeanFactory所支持的全部功能外，ApplicationContext还有如下功能：

![](image/QQ截图20190617232124.png)

​		当系统创建ApplicationContext容器时，默认会预初始化所有的  singleton  Bean。包括调用构造器创建Bean的实例，并根据<property  .../>元素执行setter方法。所以系统前期创建ApplicationContext时将有较大的系统开销。

```java
package beans;

import beans_interface.Axe;
import beans_interface.Person;

public class Chinese implements Person
{
    private Axe axe;

    public void setAxe(Axe axe)
    {
        System.out.println("正在设置axe");
        this.axe = axe;
    }
//    public  Chinese(Axe axe)
//    {
//        this.axe = axe;
//    }
    public Chinese()
    {
        System.out.println("正在初始化chinese");
    }


    @Override
    public void useAxe() {
        System.out.println(axe.chop());
    }
}


package test;

import beans.Chinese;
import beans_interface.Person;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main
{
    public static void main(String [] args)
    {
        /**
        *创建Spring容器
        */
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
          Person p = ctx.getBean("chinese", Chinese.class);
          p.useAxe();
    }
}
```

![](image/QQ截图20190617233131.png)

​		如果不想Spring容器预初始化  singleton  Bean，可以为<bean  .../>指定**lazy-init="true"**。此时，只有使用Spring容器的getBean方法时，Bean才会进行初始化。

​		Bean的初始化：分三步

![](image/QQ截图20190618004034.png)

### 3.3、ApplicationContext的国际化支持

​		ApplicationContext接口继承了MessageSource接口，因此具有国家化功能。

​		两个国际化方法：

![](image/QQ截图20190617233833.png)

​		当程序创建ApplicationContext容器时，Spring自动查找配置文件中名为messageSource的Bean实例，一旦找到这个Bean实例，上述两个方法的调用就被委托给该messageSource。如果没有该Bean，ApplicationContext会查找其父容器中的messageSource  Bean。如果找到，它将被作为messageSource  Bean使用，如果无法找到messageSource  Bean，系统就会创建一个空的StaticMessageSource  Bean。

```xml
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n.message</value>
            </list>
        </property>
    </bean>
```

message.properties

```properties
hello=欢迎你，{0}
now=现在时间是:{0}
```

message_en_US.properties

```properties
hello=welcome,{0}
now=now is : {0}
```

message_zh_CN.porperties

```properties
hello=欢迎你，{0}
now=现在时间是：{0}
```

```java
public static void main(String [] args)
    {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        String hello = ctx.getMessage("hello",new String[]{"小杉杉"}, Locale.getDefault(Locale.Category.FORMAT));
        String now = ctx.getMessage("now",new Object[]{new Date()},Locale.getDefault(Locale.Category.FORMAT));
        System.out.println(hello);
        System.out.println(now);
    }
```

![](image/QQ截图20190617235824.png)

### 3.4、ApplicationContext的事件机制

​		**ApplicationContext的事件机制是观察者设计模式的实现**，通过ApplicationEvent和ApplicationListener接口，可以实现ApplicationContext的事件处理。如果容器中有一个ApplicationListener  Bean，每当ApplicationContext发布ApplicationEvent时，ApplicationListener  Bean将自动被触发。

​		两个成员：

​				ApplicationEvent：容器事件，必须由ApplicationContext发布。

​				ApplicationListener  ：监听器，可由容器中的任何监听器Bean担任。

![](image/QQ截图20190618000518.png)

```java
public class EmailNotifier implements ApplicationListener
{
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        if(applicationEvent instanceof  EmailEvent)
        {
            EmailEvent emailEvent = (EmailEvent) applicationEvent;
            System.out.println("发送邮件的接收地址" + emailEvent.getAddress());
            System.out.println("发送邮件的正文" + emailEvent.getText());
        }
        else
        {
            System.out.println("其他事件：" + applicationEvent);
        }
    }
}

public class EmailEvent extends ApplicationEvent
{
    private String address;
    private String text;
    public EmailEvent(Object source) {
        super(source);
    }
    public EmailEvent(Object source,String address,String text)
    {
        super(source);
        this.address = address;
        this.text = text;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }
}


public class Main
{
    public static void main(String [] args)
    {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        EmailEvent ele = new EmailEvent("test","602345436@qq.com","this is a test");
        ctx.publishEvent(ele);
    }
}

```

```xml
<bean class="event.EmailNotifier" />
```

![](image/QQ截图20190618001414.png)

​		为Spring容器注册事件监听器，只要在Spring中配置一个实现ApplicationListener接口的Bean，Spring容器就会把这个Bean当成容器事件的事件监听器。

​		当系统创建Spring容器，加载Spring容器时会自动触发容器事件，容器事件监听器可以监听到这些事件。也可以调用ApplicationContext的pulishEvent()方法来主动触发容器事件。

​		监听器不仅监听到程序所触发的事件，也监听到容器内置的事件。

​		几个内置事件：

![](image/QQ截图20190618001929.png)

![](image/QQ截图20190618001954.png)

### 3.5、让Bean获取Spring容器

​		为了让Bean获取它所在的Spring容器，可以让Bean实现BeanFactoryAware接口：

​				setBeanFactory(BeanFactory beanFactory)：该方法有一个参数，该参数指向创建它的BeanFactory。

​		这个setter方法将由Spring调用，Spring调用该方法时会将Spring容器作为参数传入该方法。

## 4、Spring容器中的Bean

​		Spring框架的本质：通过XML配置来驱动 Java代码，这样可以把原本由 Java代码管理的耦合关系，提取到XML配置文件中管理。实现了系统中各组件的解耦。

### 4.1、Bean的基本定义和Bean别名

​		<beans  .../>根元素的属性：

![](image/QQ截图20190618223600.png)

​		<bean  .../>元素是<beans  .../>元素的子元素。每一个<bean  .../>子元素定义个Bean，每个Bean对应Spring容器中的一个 Java实例。

​		定义Bean时需要指定的两个属性：

![](image/QQ截图20190618224227.png)

​		**id属性是容器中Bean的唯一标识，必须遵循XML文档的id属性规则**。如果**值中需要包含特殊符号**，**可以采用name属性**，指定Bean的别名，通过访问Bean别名也可访问Bean实例。

​		指定别名的两种方式：

![](image/QQ截图20190618230104.png)

```xml
<bean id="person" class="beans.Person" name="styleperson">
        <property name="axe" ref="xiaoshanshan" />
    </bean>
    <bean id="axe" class="beans.Axe" name="xiaoshanshan" />
    <bean id="chinese" class="beans.Chinese" lazy-init="true">
        <property name="axe" ref="steelAxe"></property>
        <!--<property name="axe" ref="stoneAxe"></property>-->
        <!--<constructor-arg ref="steelAxe" />-->
    </bean>
```

```java
public static void main(String [] args)
    {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person p = ctx.getBean("styleperson",Person.class);
        p.useAxe();
    }
```

### 4.2容器中Bean的作用域

![](image/QQ截图20190618232106.png)

​		

​		**通过<bean  .../>的scope属性指定作用域**。在不指定作用域时，**默认为singleton作用域**，singleton作用的Bean，每次请求都会获得相同的实例，而prototype作用域，每次请求都会产生一个新的Bean的实例。

​		request和session作用域只在Web应用中才有效，并且必须在Web应用中将HTPP请求对象绑定到为该请求提供服务的线程上。

### 4.3配置依赖

​		不管是设值注入，还是构造注入，都视为Bean的依赖，接受Spring容器管理。要么是一个确定的值，要么就是Spring容器中其他Bean的引用。

​		通常**不建议使用配置文件管理Bean的基本类型的属性值**，通常只使用配置文件管理容器中Bean与Bean之间的依赖关系。

​		**value属性**用于指定基本类型及其包装，字符串类型的参数值。Spring使用XML解析器来解析出这些数据，然后利PropertyEditor完成类型转换。

​		**ref属性**指定Bean设置的属性值是容器中另一个Bean实例。

### 4.4自动装配

​	通过为<beans  .../>指定 default-autowire属性和<bean  .../>指定autowire属性完成。接受如下值：

![](image/QQ截图20190619000823.png)

​	**显示指定会覆盖自动装配。**

### 4.5注入嵌套Bean

​		将<bean  .../>作为<property  .../>或<constructor-args  .../>的子元素，此时该元素配置的Bean仅仅作为setter注入，构造注入的参数，但是容器不能获取，因此也无须指定id属性。

### 4.6注入集合值

![](image/QQ截图20190619003402.png)

![](image/QQ截图20190619003444.png)

![](image/QQ截图20190619003547.png)

![](image/QQ截图20190619003623.png)

​		**<map  .../>**

![](image/QQ截图20190619003723.png)

​		Spring中的Bean的几个原则：

​				1）：**尽量为每个Bean实现类提供无参数的构造器**。

​				2）：**接受设值注入的Bean，则应提供对应的setter方法，并不要求提供对应的getter方法**。

​				3）：**接受构造 注入的Bean，则应提供对应的、带参数的构造函数**

## 5、使用注解

### 1）：

```xml
<bean name="chinese" class="beans.Chinese">
        <property name="axe" ref="stoneAxe" />
    </bean>
    <bean name="stoneAxe" class="beans.StoneAxe" />
    <bean name="steelAxe" class="beans.SteelAxe" />
```

```java
@Configuration //修饰以Java配置类
public class StyleConfig
{
    @Value("小杉杉") //修饰一个字段，为该字段配置一个值，相当于配置一个变量
    String name;

    @Bean(name = "chinese") //修饰一个方法，将返回值定义成容器中的一个Bean
    public Person person()
    {
        Chinese chinese = new Chinese();
        chinese.setAxe(stoneAxe());
        return chinese;
    }

    @Bean(name = "stoneAxe")
    public Axe stoneAxe()
    {
        return new StoneAxe();
    }

    @Bean(name = "steelAxe")
    public Axe steelAxe()
    {
        return new SteelAxe();
    }
}



//创建容器：
ApplicationContext ctx = new AnnotationConfigApplicationContext(StyleConfig.class);
```

```properties
@Import：修饰一个Java配置类,用于向当前Java配置类中导入其他Java配置类
@Scope：用于修饰一个方法，指定该方法对饮得Bean的声明周期
@Lazy：用于修饰一个方法，指定该方法对应的Bean是否需要延迟初始化
@DependsOn：用于修饰一个方法，指定在初始化该方法对应的Bean之前初始化指定的Bean
@ImportResource：修饰一个Java配置类，用于向当前配置累中导入其他配置文件
```

### 2）：

![](image/QQ截图20190619221714.png)



##### 添加Bean：

​		@Configuration：修饰类，表明是一个配置类。

​		@Bean：修饰方法和类，修饰方法时将返回值作为一个Bean注册，类型为返回值类型，id默认是方法名。注册的Bean默认为单实例。

​		@ComponentScan：value：指定要扫描的包，excludeFilters：指定排除指定规则的Bean，includeFilters：指定扫描指定只包含的组件，要让其生效，需要指定useDefaultFilters=false，禁用默认的过滤规则。（**type：FilterType.ANNOTATION：按照注解；FilterType.ASSIGNABLE_TYPE：按照类型；FilterType.ASPECTJ：指定ASPECTJ表达式；FilterType.REGEX：按照正则表达式；FilterType.CUSTOM：自定义规则；**）

​					自定义规则：需要一个实现了TypeFilter接口的类。并重写match方法：其中MetadataReader类		型的参数代表当前正在扫描的类的信息，MetadataReaderFactory：用于获取其他类的信息。

​		@ComponentScans：指定多个@ComponentScan。		

​		@Scope：指定Bean的作用域。

​				ConfigurableBeanFactory.SCOPE_PROTOTYPE：prototype。ioc容器启动并不会去调用方法创建对象		放在容器中，每次获取的时候才会调用方法创建对象。每调一次就创建一个对象。

​				ConfigurableBeanFactory.SCOPE_SINGLETON：singleton。ioc容器启动会调用方法创建对象放到ioc		容器中，以后每次获取就是直接从容器中拿。

​				WebApplicationContext.SCOPE_REQUEST：request。

​				WebApplicationContext.SCOPE_SESSION：session。后两种在web环境下有效。

​		@Lazy：懒加载。针对单实例Bean。

​		@Conditional：按照一定的条件进行判断，满足条件的Bean才注册到ioc容器中。传入Condition数组。

​		@Import：快速给容器中导入一个组件。

##### Bean的生命周期：

​		由于多实例Bean，Ioc容器不会管理，所以只会执行初始化方法。

​		1、通过@Bean的initMethod和destroyMethod属性来指定初始化之后和销毁之前方法。

​		2、通过实现InitializingBean和DisposableBean来定义初始化之后和销毁之前逻辑。

​		3、通过@PostConstruct和@PreDestroy修饰方法来指定初始化之后和销毁之前逻辑。

​		4、通过加入后置处理器（实现BeanPostProcessor接口）来指定所有bean初始化之前和之后的逻辑。遍历的到容器中所有的BeanPostProcessor，挨个执行beforInitialization，一旦返回null，就跳出循环，不会执行后面的BeanPostProcessor.postProcessors()。

​		不管用那种方式，都是在 Bean 被实例化并且完成属性赋值之后，才会执行。

##### 属性赋值：

​		@Value：修饰类的字段，为其赋默认值。可以是基本数值，SpEL表达式，以及${}获取配置文件的值

​		@PropertySource：引入配置文件。

​		@Autowired：自动装配，可修饰方法、属性、构造器、参数   ，优先按照类型寻找，如果找到多个，再将属性名作为 id 去容器中查找。

​				修饰方法：@Bean + 方法参数，默认不写@Autowired，在 Spring 容器创建对象时，就会调用其修饰的方法，使用的参数，会从 IoC 容器中获取。@Bean 标注的方法创建对象时，方法参数的值从容器中获取。

​				修饰构造器：构造器的参数也是从 IoC 容器中获取。如果 Bean 只有一个有参构造器，参数的 @Autowired 可以省略，参数位置的组件还是可以从容器中自动获取。 

​				修饰参数：标注的参数从 IoC 容器中获取。

​				@Qualifier：指定装配给定的 id 的对象。其和 @Autowired 默认是一定要装配对应的 Bean，如果没有		找到，会报错。可指定 @Autowired 的属性 required 为 false ，来允许为空。

​				@Primary：修饰的 Bean 在进行依类型自动装配时，会优先使用。也可以继续使用 @Qualifier 来指定		具体装配哪一个 Bean。

​		@Resource：Java 规范中的注解，默认按照属性名进行装配。不支持 @Primary 和 @Autowired 的 reqiured   = false 的功能。

​		@Inject：需要导入 javax.injet 依赖。功能和特性和 @Autowired 一样。但 @Inject 没有属性，所以默认一定要装配成功。

​		自动装配的功能由 AutowiredAnnotationBeanPostProcessor 完成。



​		使用 Spring 容器底层的组件，只需要实现  xxxAware。比如：获取applicationContext，就可以实现ApplicationContextAware，IoC 容器会回调相应的方法，并将对应的组件实例作为参数。其实现原理是，对应的 xxxAwareProcessor。



​		Profile：根据当前环境，动态的激活和切换一系列的 Bean 的功能。

​				@Profile：被标注的组件，在其指定的环境生效时，才会被注册进容器或生效。

​							让其环境生效：

​									1、启动时，指定虚拟机参数：-Dspring.profile.active。

​									2、用代码：首先用用无参数的构造方法创建 IoC 容器，然后用 applicationContext.getEnvironment().setActiveProfile() 方法设置激活的环境，然后注册配置，在刷新容器。

##### AOP：

​		@Before：前置通知，在目标方法之前切入。

​		@After：后置通知，在目标方法运行结束之后切入，无论方法是正常结束还是异常结束。

​		@AfterReturning：返回通知，在目标方法正常返回之后切入，其 returning 属性可以一个变量名用来封装返回值等信息。

​		@AfterThrowing：异常通知，在目标方法出现异常之后切入，其 throwing 属性可指定一个变量名，来接收异常信息，通知为其标注的方法传递一个 Exception 类型的变量，变量名为指定的异常的变量名。

​		@Around：动态代理，手动推进目标方法切入。



​		在上面的注解标注的方法中，可以传入一个 JoinPoint 参数，并且此参数必须为第一个参数，此参数可以用来获取传递的参数，方法名等信息。

​		@Aspect：标注的类被指定为切面类。

```xml
<!-- 开启切面功能 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
或
@EnableAspectJAutoProxy：加在配置类上，启动切面功能：
	利用 AspectJAutoProxyRegistrar 自定义给容器中注入一个 AnnotationAwareAspectJAutoProxyCreator类型的 Bean。
		
```

​		AOP原理：

​			@EnableAspectJAutoProxy；

​					1、@EnableAspectJAutoProxy是什么？

​							@Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar利用					AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetioninternalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator

​							给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；

​					2、 AnnotationAwareAspectJAutoProxyCreator：

​							AnnotationAwareAspectJAutoProxyCreator

​									->AspectJAwareAdvisorAutoProxyCreator

​											->AbstractAdvisorAutoProxyCreator

​													->AbstractAutoProxyCreator   implements 																	SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware关注																	后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory



​				AbstractAutoProxyCreator.setBeanFactory()

​				AbstractAutoProxyCreator.有后置处理器的逻辑；



​				AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()



​				AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()

 

​				流程：

​						1）、传入配置类，创建ioc容器

​						2）、注册配置类，调用refresh（）刷新容器；

​						3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创				建；

​								1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor

​								2）、给容器中加别的BeanPostProcessor

​								3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；

​								4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；

​								5）、注册没实现优先级接口的BeanPostProcessor；

​								6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；

​										创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】

​										1）、创建Bean的实例

​										2）、populateBean；给bean的各种属性赋值

​										3）、initializeBean：初始化bean；

​												1）、invokeAwareMethods()：处理Aware接口的方法回调

​												2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的															postProcessBeforeInitialization（）

​												3）、invokeInitMethods()；执行自定义的初始化方法

​												4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的															postProcessAfterInitialization（）；

​										4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》													aspectJAdvisorsBuilder

​								7）、把BeanPostProcessor注册到BeanFactory中；

​											beanFactory.addBeanPostProcessor(postProcessor);

​			=======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========

​						AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor

​						4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单			实例bean

​								1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);

​										getBean->doGetBean()->getSingleton()->

​								2）、创建bean

​										【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，								InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】

​										1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使								用，否则再创建；

​												只要创建好的Bean都会被缓存起来

​										2）、createBean（）;创建bean；

​												AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回										bean的实例

​												【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】

​												【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理										器返回对象的】

​												1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation

​														希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不												能就继续

​														1）、后置处理器先尝试返回对象；

​																bean = applyBeanPostProcessorsBeforeInstantiation（）：

​																		拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;

​																		就执行postProcessBeforeInstantiation

​																if (bean != null) {
​																	bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
​																}



​												2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；										和3.6流程一样；

​												3）、



AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：

​		1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；关心MathCalculator和					LogAspect的创建

​				1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）

​				2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，

​		或者是否是切面（@Aspect）

​				3）、是否需要跳过

​						1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】每一个封				装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；判断每一个增强器是否是 				AspectJPointcutAdvisor 类型的；返回true

​						2）、永远返回false



​		2）、创建对象

​				postProcessAfterInitialization；

​						return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下

​						1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors

​								1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）

​								2、获取到能在bean使用的增强器。

​								3、给增强器排序

​						2）、保存当前bean在advisedBeans中；

​						3）、如果当前bean需要增强，创建当前bean的代理对象；

​								1）、获取所有增强器（通知方法）

​								2）、保存到proxyFactory

​								3）、创建代理对象：Spring自动决定

​										JdkDynamicAopProxy(config);jdk动态代理；

​										ObjenesisCglibAopProxy(config);cglib的动态代理；

​						4）、给容器中返回当前组件使用cglib增强了的代理对象；

​						5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知				方法的流程；





3）、目标方法执行	；

容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；

1）、CglibAopProxy.intercept();拦截目标方法的执行

2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；

List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

1）、List<Object> interceptorList保存所有拦截器 5

一个默认的ExposeInvocationInterceptor 和 4个增强器；

2）、遍历所有的增强器，将其转为Interceptor；

registry.getInterceptors(advisor);

3）、将增强器转为List<MethodInterceptor>；

如果是MethodInterceptor，直接加入到集合中

如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；

转换完成返回MethodInterceptor数组；



3）、如果没有拦截器链，直接执行目标方法;

拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）

4）、如果有拦截器链，把需要执行的目标对象，目标方法，

拦截器链等信息传入创建一个 CglibMethodInvocation 对象，

并调用 Object retVal =  mi.proceed();

5）、拦截器链的触发过程;

1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；

2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；

拦截器链的机制，保证通知方法与目标方法的执行顺序；



总结：

1）、  @EnableAspectJAutoProxy 开启AOP功能

2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator

3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；

4）、容器的创建流程：

1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象

2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean

1）、创建业务逻辑组件和切面组件

2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程

3）、组件创建完之后，判断组件是否需要增强

是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；

5）、执行目标方法：

1）、代理对象执行目标方法

2）、CglibAopProxy.intercept()；

1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）

2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；

3）、效果：

正常执行：前置通知-》目标方法-》后置通知-》返回通知

出现异常：前置通知-》目标方法-》后置通知-》异常通知



​		@Pointcut：抽取公共的切入点表达式。在本类引用，只需在上面的注解中写上被其标注的方法名加上圆括号。在外部引用，在上面的注解中写的是被其标注的方法名的包含包名、类名、方法名的全名加上圆括号。 @Pointcut( execution ( 切入表达式 ) )  

​		



## 6、创建Bean的方式

​		1）：调用构造器创建Bean。

​		2）：调用静态工厂方法创建Bean。

​		3）：调用实例工厂方法创建Bean。

### 6.1、使用构造器创建Bean实例

​		如果不采用构造注入，Spring底层会调用Bean类的无参数构造器来创建实例。对Bean实例的所有属性执行默认初始化，即所有**基本类型的值初始化为0或false，所有引用类型的值初始化为null。**然后BeanFactory会根据配置决定依赖关系，先实例化被依赖的Bean实例，然后为Bean注入依赖关系。

​		采用构造注入，Spring容器将使用带对应参数的构造器来创建Bean实例，Spring调用构造器传入的参数即可用于初始化Bean的实例变量。

### 6.2、使用静态工厂方法创建Bean

​		采用XML：

​		此时，class属性并不指定Bean实例的实现类，而是静态工厂类，Spring通过该属性知道由哪个工厂类来创建Bean实例。

​		除此以外，还需要指定factory-method属性来指定静态工厂方法，Spring 将调用指定方法返回一个Bean实例。如果需要参数，则使用<constructor-arg  .../>元素传入。

```xml
<bean name="dog" class="factory.BeingFactory" factory-method="getBing">
        <constructor-arg name="type" value="dog" />
        <property name="message" value="dog" />
    </bean>
    <bean name="cat" class="factory.BeingFactory" factory-method="getBing">
        <constructor-arg name="type" value="cat" />
        <property name="message" value="cat" />
    </bean>
```

```java
public class BeingFactory
{
    public static Being getBing(String type)
    {
        if(type.equalsIgnoreCase("dog"))
        {
            return new Dog();
        }
        else
        {
            return new Cat();
        }
    }
}
public class Dog implements Being
{
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public void testBeing()
    {
        System.out.println("狗:" + message);
    }
}
public static void main(String [] args)
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("configuration.xml");
        Being dog = context.getBean("dog",Being.class);
        dog.testBeing();
        Being cat = context.getBean("cat",Being.class);
        cat.testBeing();
    }

```

​		工厂类必须包含产生实例的静态工厂方法。

​		此时Spring会先解析配置文件，并根据配置文件指定的信息，通过反射调用静态工厂类的静态工厂方法，以返回值作为Bean实例。

### 6.3、调用实例工厂方法创建Bean

​		实例工厂和静态工厂唯一的不同：静态工厂方法只需使用工厂类即可，而调用实例工厂方法则需要工厂实例。因此此时需要使用factory-bean指定工厂实例。而无须指定class属性

```java
public class PersonFactory
{
    public FactoryPerson getPerson(String type)
    {
        if(type.equalsIgnoreCase("chin"))
        {
            return new FactoryChinese();
        }
        else
        {
            return new American();
        }
    }
}
```

```xml
<bean id="personFactory" class="factory.PersonFactory" />
    <bean id="chin" factory-bean="personFactory" factory-method="getPerson">
        <constructor-arg name="type" value="chin" />
    </bean>
    <bean id="american" factory-bean="personFactory" factory-method="getPerson">
        <constructor-arg name="type" value="ame" />
    </bean>

```

## 7、深入理解容器中的Bean

### 7.1、抽象Bean与子Bean

​		把多个<bean   .../>配置中相同的信息提取出来，集中成配置模版，但这个模版并不是真正的Bean，所以应该为<bean   .../>配置增加abstract="true"，这就是抽象Bean。由于Spring不会实例化该Bean，所以可以不指定class属性。

​		通过为<bean  .../>元素指定parent属性即可指定该Bean是一个子Bean，parent属性指定该Bean所继承的父Bean的id。

​		子Bean的配置信息会覆盖父Bean模版中相同的信息。如果父Bean指定了class属性，那么子Bean可不指定class属性，子Bean将采用与父Bean相同的实现类。

​		无法继承的属性：

```properties
depends-on
autowire
singleton
scope
lazy-init

depends-on：通过指定该属性，可以在初始化主调Bean之前，强制初始化一个或多个Bean。

这些属性总是从子Bean定义中获得，或者采用默认值
```

### 7.2、Bean继承与 Java继承的区别

![](image/QQ截图20190621214527.png)

### 7.3、容器中的工厂Bean

​		此时的工厂Bean并不是创建Bean的工厂，而是一种特殊的Bean，这种Bean必须实现FactoryBean接口。

​		FactoryBean接口是Bean的标准接口，把工厂Bean（实现FactoryBean接口的Bean）部署在容器中之后，程序通过getBean()方法来获取它时，容器返回的不是FactoryBean实现类的实例，而是返回FactoryBean的产品（即该工厂Bean的getObject方法的返回值）。

![](image/QQ截图20190621225409.png)

​		配置FactoryBean与配置普通Bean的定义没有什么区别，但当程序向Spring容器请求过去该Bean时，**容器返回该FactoryBean的产品，而不是返回该FactoryBean本身**。

​		Spring在检测容器中的所有Bean时，如果发现某个Bean实现类实现了FactoryBean接口，**Spring容器就会在实例化该Bean，根据<property .../>执行setter方法之后，额外调用该Bean的getObject方法，并将该方法的返回值作为容器中的Bean**。当程序需要获得FactoryBean本身时，并不直接请求Bean  id，而是在Bean  id前增加“&”符号，容器则返回FactoryBean本身，而不是其产品Bean。

### 7.4、获取Bean本身的id

​		通过Spring提供的BeanNameAware接口，可提前预知该Bean的配置id。

​		此接口提供一个setBeanName(String  name)，该方法的name参数就是Bean的id，实现该方法的Bean类可通过该方法来获得部署该Bean的id。这个方法是由Spring容器负责调用的。此时的name参数就是配置该Bean时，指定的id。

## 8、容器中Bean的声明周期

​		Spring可以管理singleton作用域的Bean的生命周期，对于prototype作用域的Bean，Spring容器仅仅负责创建，无法管理其生命周期。

### 8.1、依赖关系注入之后的行为

​		init-method属性：指定某个方法应在Bean全部依赖关系设置结束后自动执行。

​		InitializingBean接口：该接口提供一个afterPropertiesSet方法，可以达到上述同样的效果。

​		如果既配置Bean指定init-method属性，而且该Bean对应的类也实现了InitializingBean接口，那么Bean在配置完所有的依赖以后，**会先执行afterPropertiesSet，然后再执行init-method指定的方法**。

### 8.2、Bean销毁之前的行为

​		destory-method属性：指定某个方法在Bean销毁之前被自动执行。

​		实现DisposableBean接口：该接口提供一个destroy方法，可以达到与上述一样的效果。

​		同样，如果上述都被指定，也是先执行接口中的方法，在执行属性指定的方法。

**如果容器中的许多Bean都需要指定特定的生命周期行为，则可以在<beans  .../>元素上指定default-init-method和default-destroy-method属性，只要容器中的具有其指定的方法，那么该方法就会在Bean所有依赖设置完成后，或者销毁之前执行**。

![](image/QQ截图20190623225250.png)

### 8.3、协作作用域不同步的Bean

​		当singleton作用域的Bean依赖prototype作用的Bean，Spring容器，**会在初始化singleton作用域Bean之前，先初始化一个prototype作用域的Bean，并将这个Bean注入到singleton作用域的Bean中**，之后无论什么时候**通过singleton  Bean访问 prototype  Bean时，得到的永远都是最初创建的这个 prototype  Bean**，这就相当于singleton  Bean把它所依赖 prototype  Bean的作用域变成了singleton。

![](image/QQ截图20190623225923.png)

​			利用方法注入可以解决上述问题：

​			1）、将调用者Bean的实现类定义为抽象类，并定义一个抽象方法来获取被依赖的Bean。

​			2）、在< bean  .../>元素中添加<lookup-method  .../>子元素让Spring为调用者Bean的实现类实现指定的抽象方法。

```java
public class Dog implements Being
{
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

//    @Override
//    public String toString() {
//        return "Dog{" +
//                "message='" + message + '\'' +
//                '}';
//    }

    @Override
    public void testBeing()
    {
        System.out.println("狗:" + message);
    }
}

public abstract class CollaborationBean
{
    private Dog dog;
    public abstract Dog getDog();
    public void hunt()
    {
        System.out.println(getDog());
    }
}

```

```xml
<bean id="collaboration" class="beans.CollaborationBean">
        <lookup-method name="getDog" bean="guDog" />
    </bean>
    <bean id="guDog" class="beans.Dog" scope="prototype">
        <property name="message" value="小杉杉" />
    </bean>
```

​		lookup-method的name属性指定的抽象方法是由Spring容器来负责实现，当实现了该方法，那么其配置的Bean的类不再是抽象类，故可以初始化，Spring实现抽象方法是总是执行：**return   application.getBean(bean)**，故总是获取的新对象。但是此时也要**将被依赖的Bean的作用域设置为prototype**。

​		如果执行上述元素的Bean的类实现过接口，那么Spring会采用JDK动态代理来实现其抽象类，并为之实现抽象方法，如果没有实现过接口，那么Spring会采用cglib实现该抽象类，并为之实现抽象方法。

## 9、高级依赖关系配置

​		通常的建议：组件与组件之间的耦合，采用依赖注入管理，但基本类型的成员变量值，应直接在代码中设置。

### 9.1、获取其他Bean的属性值

​		PropertyPathFactoryBean用来获取目标Bean的属性值（实际上就是它的getter方法的返回值），获得的值可注入给其他Bean，也可直接定义成新的Bean。

​		需要指定一下信息：

​				调用那个对象：由PropertyPathFactoryBean得setTargetObject(Object  targetObject)方法指定。

​				调用那个getter方法：由PropertyPathFactoryBean得setPropertyPath(String  propertyPath)方法指		定。

```xml
<bean id="amer" class="beans.American">
        <property name="axe">
            <bean class="beans.Axe">
                <property name="message" value="小杉杉" />
            </bean>
        </property>
    </bean>
    <bean id="axe1" class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
        <property name="targetBeanName" value="amer" />
        <property name="propertyPath" value="axe" />
    </bean>

	<!-- 下面是简化配置 -->
	<util:property-path path="amer.axe" id="axe1" />
```

```java
public class American implements FactoryPerson
{

    private Axe axe;

    public Axe getAxe() {
        return axe;
    }

    public void setAxe(Axe axe) {
        this.axe = axe;
    }

    @Override
    public String sayHello(String name) {
        return name + "：Hello";
    }
}

public class Axe
{
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String chop()
    {
        return "使用斧头砍柴";
    }

    @Override
    public String toString() {
        return "Axe{" +
                "message='" + message + '\'' +
                '}';
    }
}

```

​		**此时配置的PropertyFactoryBean（工厂Bean）指定的id属性，并不是该Bean的唯一标识，而是用于指定属性表达式的值**。

​		对于指定的PropertyFactoryBean的getter方法的返回值，不仅可以定义成容器中的Bean实例，还可以注入另一个Bean中。

```xml
<bean id="axe2" class="beans.Axe">
      <property name="message">
         <bean id="amer.axe.message" PropertyFactoryBean工厂Bean负责获取容器另一个Bean的属性值 class="org.springframework.beans.factory.config.PropertyPathFactoryBean" />
      </property>
    </bean>

	<!-- 下面是简化配置 -->
<bean id="axe2" class="beans.Axe">
      <property name="message">
         <util:property-path path="amer.axe.message" />
      </property>
    </bean>
```

​		还可以为PropertyFactoryBean的setPropertyPath方法指定属性表达式，还支持符合舒心的形式。

```xml
<bean id="message" class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
        <property name="targetBeanName" value="amer" />
        <property name="propertyPath" value="axe.message" />
    </bean>
<!-- 下面是简化配置 -->
	<util:property-path path="amer.axe.message" id="message" />
```

​		目标Bean既可以是容器中已有的Bean实例，也可以是嵌套的Bean实例。

```xml
<bean id="message2" class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
        <property name="targetObject"> <!--这里是重点 -->
            <bean class="beans.Axe">
                <property name="message" value="shanji" />
            </bean>
        </property>
        <property name="propertyPath" value="message" />
    </bean>
```

### 9.2、获取Field值

​		通过FieldRetrievingFactoryBean类，可访问类的静态Field或对象的实例Field值。FieldRetrievingFactoryBean获得指定的Field的值之后，既可以将获得的值注入到其他Bean，也可以直接定义成新的Bean。

​		访问的Field是静态的：

​				调用哪个类：由FieldRetrievingFactoryBean的setTargetClass(String  targetClass)方法指定。

​				访问哪个Field：由FieldRetrievingFactoryBean的setTargetField(String  targetField)方法指定。

​		访问的Field是实例的：

​				调用那个对象：由FieldRetrievingFactoryBean的setTargetObject(Object  targetObject)方法指定。

​				访问哪个Field：由FieldRetrievingFactoryBean的setTargetField(Stringh  targetField)方法指定。

​		对于Field是实例的情况，很少遇到，原因是此时要求实例的Field用public修饰，而良好的封装原则是以private修饰。

​		将静态Field定义为容器中的Bean

```xml
<bean id="connectmessage" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">
        <property name="targetClass" value="java.sql.Connection" />
        <property name="targetField" value="TRANSACTION_SERIALIZABLE" />
    </bean>
```

​		还有一个setStaticField(String  staticField)方法，可以达到同样的效果。

```xml
<bean id="connectmessage2" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">
        <property name="staticField" value="java.sql.Connection.TRANSACTION_SERIALIZABLE" />
    </bean>
```

​		也可以将获取到的注入到其他Bean中。

```xml
<bean id="axe3" class="beans.Axe">
        <property name="message">
            <bean id="java.sql.Connection.TRANSACTION_SERIALIZABLE" Field表达式 class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean" />
            
            <!-- 简化 -->
            <util:constant static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE" />
            id:指定将静态Field的指定名为id的Bean实例。
            static-field：指定访问哪个类的哪个静态Field。
        </property>
    </bean>
```

### 9.3、获取方法的返回值

​		通过MethodInvokingFactoryBean可以调用任意类的类方法，也可以调用任意对象的实例方法，如果方法有返回值，即可将该返回值定义成容器中的Bean，也可以注入给其他Bean。

​		静态方法：

​				调用哪个类：由MethodInvokingFactoryBean的setTargetClass(String  targetClass)方法指定。

​				调用哪个方法：由MethodInvokingFactoryBean的setTargetMethod(String  targetMethod)方法指定。

​				调用方法的参数：由MethodInvokingFactoryBean的setArguments(Object[]  arguments)方法指定。

​		实例方法：

​				调用哪个对象：由MethodInvokingFactoryBean的setTargetObject(Object  targetObject)方法指定。

​				调用哪个方法：由MethodInvokingFactoryBean的setTargetMethod(String  targetMethod)方法指定。

​				调用方法的参数：由MethodInvokingFactoryBean的setArguments(Object[]  arguments)方法指定。

![](image/QQ截图20190624232758.png)

```xml
<bean id="win" class="javax.swing.JFrame">
        <constructor-arg value="小杉杉的窗口" type="java.lang.String" />
        <property name="visible" value="true" />
    </bean>

    <bean id="jta" class="javax.swing.JTextArea">
        <constructor-arg value="7" type="int" />
        <constructor-arg value="40" type="int" />
    </bean>

    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="targetObject" ref="win" />
        <property name="targetMethod" value="add" />
        <property name="arguments">
            <list>
                <bean class="javax.swing.JScrollPane">
                    <constructor-arg ref="jta" />
                </bean>
            </list>
        </property>
    </bean>

    <bean id="jp" class="javax.swing.JPanel" />

    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="targetObject" ref="win" />
        <property name="targetMethod" value="add" />
        <property name="arguments">
            <list>
                <ref bean="jp" />
                <util:constant static-field="java.awt.BorderLayout.SOUTH" />
            </list>
        </property>
    </bean>

    <bean id="jb1" class="javax.swing.JButton">
        <constructor-arg value="确定" type="java.lang.String" />
    </bean>

    <bean id="jb2" class="javax.swing.JButton">
        <constructor-arg value="取消" type="java.lang.String" />
    </bean>

    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="targetObject" ref="jp" />
        <property name="targetMethod" value="add" />
        <property name="arguments">
            <list>
                <ref bean="jb1" />
            </list>
        </property>
    </bean>

    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="targetObject" ref="jp" />
        <property name="targetMethod" value="add" />
        <property name="arguments">
            <list>
                <ref bean="jb2" />
            </list>
        </property>
    </bean>

    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="targetObject" ref="win" />
        <property name="targetMethod" value="pack" />
    </bean>
```

​		**总结：**

​				调用构造器创建对象：用<bean .../>元素.。

​				调用setter方法：用<property .../>元素。

​				调用getter方法：用PropertyPathFactoryBean或<util:property-path .../>元素。

​				调用普通方法：用MethodInvokingFactoryBean工厂Bean。

​				获取Field的值：用FieldRetrievingFactoryBean或<util:constant .../>元素。

## 10、基于XML Schema的简化配置

### 10.1、使用**p:**命名空间简化配置

​		**p:**命名空间不需要特定的Schema定义，它直接存在于Spring内核中，当导入p:命名空间之后，就可以直接在<bean .../>元素中使用属性来驱动执行setter方法。主要用于简化设值注入。

```java
public class Chinese implements Person, BeanNameAware, InitializingBean, DisposableBean
{
    private Axe axe;
    private String beanName;
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public void setAxe(Axe axe)
    {
        System.out.println("正在设置axe");
        this.axe = axe;
    }
```

```xml
<bean id="chinese" class="beans.Chinese" lazy-init="true" init-method="init" destroy-method="destorys">
        <property name="axe" ref="steelAxe" />
        <property name="message" value="xiaoshanshan" />
    </bean>

简化：
<bean id="simplyechinese" class="beans.Chinese" p:message="xiaoshanhs" p:axe-ref="steelAxe" />
```

​		但是**p:**空间没有标准的XML格式灵活，如果某个Bean的属性名是以 "-ref"结尾的，那么采用**p:**命名空间定义时就会发生冲突。

### 10.2、使用**c:**命名空间简化配置

​		**c:**命名空间则用于简化构造注入。

​		格式：c:构造器参数名= ” 值 “ 或 c:构造器参数名-ref= ” 其他Bean的id “。

```java
public class Chinese implements Person, BeanNameAware, InitializingBean, DisposableBean
{
    private Axe axe;
    private String beanName;
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public void setAxe(Axe axe)
    {
        System.out.println("正在设置axe");
        this.axe = axe;
    }
    public  Chinese(Axe axe)
    {
        this.axe = axe;
    }
```

```xml
<bean id="chinese" class="beans.Chinese" lazy-init="true" init-method="init" destroy-method="destorys">
        <property name="message" value="xiaoshanshan" />
        <constructor-arg ref="steelAxe" />
    </bean>

简化：
<bean id="simplyechinese1" class="beans.Chinese" c:axe-ref="stoneAxe" p:message="shanji" />
或者
<bean id="simplyechinese2" class="beans.Chinese" c:_0-ref="stoneAxe" p:message="shanji" /><!-- 索引方式  c:_N中的N代表第几个构造器参数  -->
```

### 10.3、使用**util:**命名空间简化配置

​		有以下元素：

​		constant：该元素用于获取指定类的静态Field的值，它是FieldRetrievingFactoryBean的简化配置。

​		property-path：该元素用于获取指定对象的getter方法的返回值，它是PropertyPathFactoryBean的简化配置。

​		list：该元素用于定义一个List  Bean，支持使用<value ../>，<ref .../>，<bean .../>等子元素来定义List集合元素，该标签支持三个属性：

​				id：该元素指定定义一个名为id的List  Bean实例。

​				list-class：该属性指定Spring使用哪个List实现类来创建Bean实例。默认使用ArrayList作为实现类。

​				scope：指定该List  Bean实例的作用域。

​		set：该元素用于定义一个Set  Bean，支持使用<value .../>，<ref .../>，<bean .../>等子元素来定义Set集合元素。该标签有三个属性：

​				id：该属性指定定义一个名为id的Set Bean实例。

​				set-class：该属性指定Spring使用哪个Set实现类来创建Bean实例，默认使用HshSet作为实现类。

​				scope：指定Set  Bean实例的作用域。

​		map：该元素用于定义一个Map  Bean，支持使用<entry ../>来定义Map的  key-value 对。该标签有三个属性。

​				id：该属性指定定义一个名为id的Map  Bean实例。

​				map-class：该属性指定Spring使用哪个Map实现类来创建Bean实例，默认使用HashMap作为实现类。

​				scope：指定该Map  Bean实例的作用域。

​		properties：该元素用于加载一份资源文件，并根据加载的资源文件创建一个Properties  Bean实例。该标签有三个属性：

​				id：该属性指定定义一个名为id的Properties Bean实例。

​				location：指定资源文件的位置。

​				scope：指定该Properties  Bean实例的作用域。

```java
public class Chinesed
{
    private beans_interface.Axe axe;
    private int age;
    private List schools;
    private Map scores;
    private Set axes;

```

```xml
<bean id="chinesed" class="beans.Chinesed" p:age-ref="chin.age" p:axe-ref="steelAxe" p:schools-ref="chin.schools" p:axes-ref="chin.axes" p:scores-ref="chin.scores" />
    <util:constant static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE" id="chin.age" />
    <util:properties id="conftest" location="classpath:i18n/message_zh_CN.properties" />
    <util:list id="chin.schools" list-class="java.util.LinkedList">
        <value>小学</value>
        <value>中学</value>
        <value>大学</value>
    </util:list>
    <util:set id="chin.axes" set-class="java.util.HashSet">
        <value>字符串</value>
        <bean class="beans.SteelAxe" />
        <ref bean="stoneAxe" />
    </util:set>

    <util:map id="chin.scores" map-class="java.util.TreeMap">
        <entry key="数学" value="87" />
        <entry key="英语" value="89" />
        <entry key="语文" value="82" />
    </util:map>
```

## 11、Spring3.0提供的表达式语言（SpEL）

​		Spring表达式语言(SpEL)是一种与JSP 2的EL功能类似的表达式语言。

### 11.1、使用Expression接口进行表达式求值

​		Spring的SpEL可以单独使用，可以使用SpEL对表达式计算，求值。

​		三个接口：

​				ExpressionParser：该接口的实例负责解析一个SpEL表达式，返回一个Expression对象。

​				Expression：该接口的实例代表一个表达式。

​				EvaluationContext：代表计算表达式值得上下文。这个Context对象可以包含多个对象，但只能有一个root（根）对象，当SpEL表达式中含有变量时，程序将使用该API来计算表达式的值。

​		Expression包含的计算方法：

![](image/QQ截图20190708144954.png)

```java
ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'HelloWorld'");
        System.out.println("'HelloWorld'的结果：" + exp.getValue());

        exp = parser.parseExpression("'HelloWorld'.concat('!')");
        System.out.println("'HelloWorld'.concat('!')的结果：" + exp.getValue());

        exp = parser.parseExpression("'HelloWorld'.bytes");
        System.out.println("'HelloWorld'.bytes的结果：" + exp.getValue());

        exp = parser.parseExpression("'HelloWorld'.bytes.length");
        System.out.println("'HelloWorld'.bytes.length的结果：" + exp.getValue());

        exp = parser.parseExpression("new String('helloworld').toUpperCase()");
        System.out.println("new String('helloworld').toUpperCase()的结果：" + exp.getValue(String.class));

        Persons persons = new Persons(1,"小杉杉",new Date());
        exp = parser.parseExpression("name");
        System.out.println("以persons为root,name表达式的值是：" + exp.getValue(persons,String.class));
        exp = parser.parseExpression("name == '小杉杉'");

        StandardEvaluationContext ctx = new StandardEvaluationContext();
        ctx.setRootObject(persons);
        System.out.println(exp.getValue(ctx,Boolean.class));

        List<Boolean> list = new ArrayList<>();
        list.add(true);

        EvaluationContext ctx2 = new StandardEvaluationContext();
        ctx2.setVariable("list",list);

        parser.parseExpression("#list[0]").setValue(ctx2,"false");
        System.out.println("list集合的第一个元素为：" + parser.parseExpression("#list[0]").getValue(ctx2));
```

![](image/QQ截图20190708151515.png)

​		往EvaluationContext里放入对象(SpEL称之为变量)，调用：

​				setVariable(String  name,Object  value)：向EvaluationContext中放入value对象，该对象名为name。		**访问：#name**

​		设置root对象：

​				setRootObject(Object  rootObject)

​		访问：

​				可省略root对象前缀：如：foo.bar 即访问rootObject的foo属性的bar属性。

​		使用Expression对象计算表达式的值时，也可以直接指定root对象，如：exp.getValue(persons,String.class)即：以persons对象为root对象计算表达式的值。

### 11.2、Bean定义中的表达式语言支持

​		SpEL的一个重要作用就是扩展Spring容器的功能，允许在Bean定义中使用SpEL。在XML配置文件和注解中都可以使用SpEL。在表达式外面增加**#{  }**即可。

```java
public class Author implements Person
{
    private Integer id;
    private String name;
    private List<String> books;
    private Axe axe;
```

```xml
<util:properties id="confTest" location="classpath:i18n/message_zh_CN.properties" />
<!-- 可以简化配置，使用SpEL可以在配置文件中调用方法，创建对象(此方式可以代替嵌套Bean语法)，访问其他Bean的属性... -->    
<bean id="author" class="beans.Author" p:name="#{T(java.lang.Math).random()}" p:axe="#{new beans.SteelAxe()}" p:books="#{{confTest.hello,confTest.now}}" />
```

### 11.3、SpEL语法详述

#### 1、直面量表达式

​		SpEL中最简单的表达式，直接量表达式就是在表达式中使用 Java语言支持的直面量，包括字符串，日期，数值，boolean和null。

```java
Expression exp = parser.parseExpression("'HelloWorld'");
System.out.println(exp.getValue(String.class));
```

#### 2、在表达式中创建数组

​		支持使用静态初始化，动态初始化两种语法来创建数组。

```java
exp = parser.parseExpression("new String[]{'java','struts','spring'}");
```

#### 3、在表达式中创建List集合

```java
exp = parser.parseExpression("{'java','struts','spring'}");
```

#### 4、在表达式中访问List、Map等集合元素

​		List：list[index]。

​		Map：map[key]。

```java
		List<String> lists = new ArrayList<>();
        lists.add("java");
        lists.add("spring");
        Map<String,Double> map = new HashMap<>();
        map.put("java",0.8);
        map.put("spring",0.9);
        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("list",list);
        context.setVariable("map",map);
        
        System.out.println(parser.parseExpression("#list[0]").getValue(context));
        
        System.out.println(parser.parseExpression("#map[java]").getValue(context));
```

#### 5、调用方法

​		与在 Java代码中调用方法没有任何区别。

```java
exp = parser.parseExpression("'HelloWorld'.concat('!')");
System.out.println(exp.getValue());

List<String> lists = new ArrayList<>();
lists.add("java");
lists.add("spring");

        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("list",lists);

        System.out.println(parser.parseExpression("#list.subList(0,1)").getValue(context));
```

#### 6、算术，比较，逻辑，赋值，三目等运算符

```java
List<String> lists = new ArrayList<>();
        lists.add("java");
        lists.add("spring");

        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("list",lists);

        parser.parseExpression("#list[0]='小杉杉'").getValue(context);
        System.out.println(lists.get(0));

        System.out.println(parser.parseExpression("#list.size() > 3 ? '大于3':'不大于3'").getValue(context));
```

#### 7、类型运算符

​		特殊的运算符：T()。这个运算符将运算符类的字符串当成“类”处理，避免Spring对其进行其他解析，尤其是调用某个类的静态方法时。

```java
System.out.println(parser.parseExpression("T(java.lang.Math).random()").getValue());
        System.out.println(parser.parseExpression("T(System).getProperty('os.name')").getValue());
```

​		如果只写类名，不写包名，SpEL使用StandardTypeLocator去定位，默认会在java.lang包下找这些类。

#### 8、调用构造器

​		允许在SpEL表达式中直接使用new 来调用构造器，这样可以创建一个对象。

```java
exp = parser.parseExpression("new String('helloworld').toUpperCase()");
```

#### 9、变量

​		SpEL允许通过EvaluationContext来使用变量，该对象包含一个setVariable(String  name,Object  value)方法，该方法用于设置一个变量。

​		两个特殊变量：

​				#this：引用SpEL当前正在计算的对象。

​				#root：引用SpEL的EvaluationContext的root对象。

#### 10、自定义函数

​		通过StandardEvaluationContext的registerFunction(String  name,Method  m)将m方法注册成自定义函数。该函数的名称为name，作用并不大，因为SpEL本身已经允许在表达式语言中调用方法。

#### 11、Elvis运算符

​		三目运算符的简写：

```java
name != null ? name : "newVal";
简写
name?:"newVal";
```

#### 12、安全导航操作

​		在进行形如 foo.bar可能会出现空指针异常，为了避免此种情况，使用 foo?.bar。此时如果foo为空，那么结果为null，而不会出现空指针异常。

#### 13、集合选择

​		SpEL允许直接对集合进行选择操作，可以对集合元素筛选。

​		语法：**collection.?[condition_expr]**

​				condition_expr是一个根据元素定义的表达式，只有当该表达式返回true时，对应的集合元素才会被筛		选出来。

```java
List<String> newlist = new ArrayList<>();
        newlist.add("疯狂java讲义");
        newlist.add("疯狂ajax讲义");
        newlist.add("疯狂ios讲义");
        newlist.add("经典java ee企业应用实战");

        EvaluationContext bookcontext = new StandardEvaluationContext();
        bookcontext.setVariable("list",newlist);
        exp = parser.parseExpression("#list.?[length() > 7]"); //length()：指list中值的长度
        System.out.println(exp.getValue(bookcontext));

        Map<String,Double> score = new HashMap<>();
        score.put("java",89.0);
        score.put("spring",82.0);
        score.put("english",75.0);
        bookcontext.setVariable("map",score);
        exp = parser.parseExpression("#map.?[value > 80]");
        System.out.println(exp.getValue(bookcontext));
```

#### 14、集合投影

​		依次迭代出每个集合元素，迭代时根据指定表达式对集合元素进行计算得到一个新的结果，依次将每个结果收集成新的集合，这个新的集合将作为投影运算的结果。

​		语法：**collection.![condition_expr]**

```java
List<String> list = new ArrayList<>();
        list.add("疯狂java讲义");
        list.add("疯狂ajax讲义");
        list.add("疯狂ios讲义");
        list.add("经典java ee企业应用实战");

        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("list",list);
        exp = parser.parseExpression("#list.![length()]");
        System.out.println(exp.getValue(context));

        List<Persons> list12 = new ArrayList<>();
        list12.add(new Persons(1,"山鸡",new Date()));
        list12.add(new Persons(2,"小山鸡",new Date()));
        list12.add(new Persons(3,"小杉杉",new Date()));

        context.setVariable("lists",list12);
        exp = parser.parseExpression("#lists.![name]");
        System.out.println(exp.getValue(context));
```

#### 15、表达式模版

​		本质是对**直接量表达式**的扩展，它允许在直接量表达式中插入一个或多个#{expr}，#{expr}将会被动态计算出来。

```java
Persons persons1 = new Persons(1,"山鸡",new Date());
        Persons persons2 = new Persons(1,"小山鸡",new Date());

        exp = parser.parseExpression("我的名字是#{name},录入日期是#{create}",new TemplateParserContext());
        System.out.println(exp.getValue(persons1));
        System.out.println(exp.getValue(persons2));
```

​		解析字符串模版时需要传入一个TemplateParserContext参数，该TemplateParserContext实现了ParserContext接口，它用于为表达式解析传入一些额外的信息。

## 12、两种后处理器

### 12.1、Bean后处理器

​		Bean后处理器是一种特殊的Bean，这种特殊的**Bean并不对外提供服务**，它甚至可以无须id属性，它主要负责对容器中的其他Bean执行后处理。**Bean后处理器会在Bean实例创建成功之后，对Bean实例进行进一步的增强处理**。

​		**Bean后处理器必须实现BeanPostProcessor接口**，两个方法：

​				postProcessBeforeInitalization(Object  bean,String  name) throws BeansException：该方法的第一		个参数是系统即将进行后处理的Bean实例，第二个参数是该Bean的配置id。

​				postProcessAfterInitialization(Object  bean,String  name) throws  BeansException：该方法的第一个		参数是系统即将进行后处理的Bean实例，第二个参数是该Bean配置的id。

​		这两个方法会对容器的中的Bean进行后处理，**会在目标Bean初始化之前，初始化之后被回调**，这两个方法用于对容器中的Bean实例进行增强处理。

```java
public class StyleBeanPostProcessor implements BeanPostProcessor
{
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Bean后处理器在住实话之前对" + beanName + "进行增强处理。。。。");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Bean后处理器在初始化之后对" + beanName + "进行增强处理。。。");
        if(bean instanceof Persons)
        {
            Persons person = (Persons) bean;
            person.setName("小杉杉");
        }
        return bean;
    }
}


public static void main(String [] args) throws Exception
    {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        System.out.println(applicationContext.getBean("persons"));
    }

```

```xml
<bean id="persons" class="beans.Persons" />
    <bean class="processor.StyleBeanPostProcessor" />
```

![](image/QQ截图20190709205448.png)

​		容器中一旦注册了Bean后处理器，Bean后处理器就会自动启动，在容器中每个Bean创建时自动工作，加入Bean后处理器需要完成的工作。

![](image/QQ截图20190709210142.png)

​		如果**使用BeanFactory作为Spring容器，则必须手动注册Bean后处理器**，程序必须获取Bean后处理器实例，然后手动注册，在这种情况下，程序可能需要在配置文件中为Bean处理器指定id属性，这样才能让Spring容器先获取Bean后处理器，然后注册它。

```java
Resource isr = new ClassPathResource("applicationContext.xml");
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        new XmlBeanDefinitionReader(beanFactory).loadBeanDefinitions(isr);
        BeanPostProcessor bp = (BeanPostProcessor)beanFactory.getBean("processor");
        beanFactory.addBeanPostProcessor(bp);//注册
        System.out.println(beanFactory.getBean("persons"));
```

### 12.2、Bean后处理器的用处

​		两个常用的Bean后处理器：

​				BeanNameAutoProxyCreator：根据Bean实例的name属性，创建Bean实例的代理。

​				DefaultAdvisorAutoProxyCreator：根据提供的Advisor，对容器中的所有Bean实例创建代理。

### 12.3、容器后处理器

​		Bean后处理器负责处理容器中的所有Bean实例，而**容器后处理器则负责处理容器本身**。

​		**容器后处理器必须实现BeanFactoryPostProcessor接口**，必须实现以下方法：

​				postProcessBeanFactory(ConfigurableListableBeanFactory  beanFactory)：该方法的方法体就是对		Spring 容器进行的处理，这种处理可以对Spring容器进行自定义扩展。

​		与Bean后处理器类似，ApplicationContext也可以自动检测容器中的容器后处理器，并自动注册，但使用BeanFactory作为Spring容器，同样也需要手动获取和注册。

​		Spring没有提供ApplicationContextPostProcessor，对于ApplicationContext容器，一样使用BeanFactoryPostProcessor作为容器后处理器。

​		常用的容器后处理器：

​				PropertyPlaceholderConfigurer：属性占位符配置器。

​				PropertyOverrideConfigurer：重写占位符配置器。

​				CustomAutowireConfigurer：自定义自动装配的配置器。

​				CustomScopeConfigurer：自定义作用域的配置器。

​		容器后处理通常用于对Spring容器进行处理，并且总是在容器实例化任何其他的Bean之前，读取配置文件的元数据，并有可能修改这些元数据。

​		程序可以配置多个容器后处理器，多个容器后处理器可设置order属性来控制容器后处理器的执行次序。

​		**为了给容器后处理器指定order属性，则要求容器后处理器必须实现Ordered接口，因此在实现BeanFactoryPostProcessor时，就应当考虑实现Ordered接口。**

### 12.4、属性占位符配置器

​		PropertyPlaceholderConfigurer，一个容器后处理器，负责读取Properties属性文件里的属性值，并将这些属性值设置成Spring配置文件的数据。

​		可以将部分相似的配置放在特定的属性文件中，如果只需要修改这部分配置，则无需修改Spring配置文件，修改属性文件即可。

```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssh
jdbc.username=root
jdbc.passowrd=179980
```

```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>db.properties</value>
            </list>
        </property>
    </bean>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close" p:driverClass="${jdbc.driverClass}" p:jdbcUrl="${jdbc.url}" p:user="${jdbc.username}" p:password="${jdbc.passowrd}" />
<!-- 简化 -->
<context:property-placeholder location="classpath:db.properties" />
```

### 12.5、重写占位符配置器

​		PropertyOverrideConfigurer的属性文件指定的信息可以直接覆盖Spring配置文件中的元数据。此时，可以认为Spring配置信息是XML配置文件和属性文件的总和，当XML配置文件和属性文件指定的元数据不一致时，属性文件的信息取胜。

​		属性文件格式：

​				beanId.property=value

​						beanId：试图覆盖Bean的id。Bean必须是容器中真实存在的Bean，否则会报错。

​						property：试图覆盖的属性名(对应于调用setter方法)。

```xml
<bean class="org.springframework.beans.factory.config.PropertyOverrideConfigurer">
        <property name="locations">
            <list>
                <value>dbcon.properties</value>
            </list>
        </property>
    </bean>
    <bean id="dataSources" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close" />

    <!-- 简化 -->
    <context:property-override location="dbcon.properties" />
```

```properties
dataSources.driverClass=com.mysql.cj.jdbc.Driver
dataSources.jdbcUrl=jdbc:mysql://localhost:3306/ssh
dataSources.user=root
dataSources.passowrd=179980
```

​		如果对同一属性进行多次覆盖，最后一次覆盖将会生效。

## 13、Spring的零配置支持

### 13.1、搜索Bean类

​		约定优于配置：要求将不同组件放在不同路径下。

​		**Spring没有采用“约定优于配置”的策略**，Spring通过使用一些特殊的注解来标注Bean类。

​				**@Component：标注一个普通的Spring  Bean类。**

​				**@Controller：标注一个控制器组件类。**

​				**@Service：标注一个业务逻辑组件类。**

​				**@Repository：标注一个DAO组件类。**

​		指定了某些类可作为Spring  Bean类使用后，最后还需要让Spring搜索指定路径，此时需要在Spring配置文件中导入context  Schema，并指定一个简单的搜索路径。

```java
public static void main(String [] args) throws Exception
    {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("annotation.xml");
        System.out.println("所有扫描到的Bean\n" + Arrays.toString(applicationContext.getBeanDefinitionNames()));
    }
```

```xml
<context:component-scan base-package="annotation" />
```

```java
@Component
//@Component("chin")  //指定Bean的id
public class Chinese implements Person
{
    private Axe axe;

    public Axe getAxe() {
        return axe;
    }

    public void setAxe(Axe axe) {
        this.axe = axe;
    }

    @Override
    public void useAxe() {

    }
}

@Component
public class SteelAxe implements Axe
{
    @Override
    public String chop() {
        return "钢斧砍柴真快";
    }
}

@Component
public class StoneAxe implements Axe
{
    @Override
    public String chop() {
        return "石斧砍柴真慢";
    }
}
```

![](image/QQ截图20190710175620.png)

​		如果在使用注解时，没有指定Bean的id，那么Spring采用约定的方式来为这些Bean实例指定名称，默认是Bean类的首字母小写，其余部分不变。

​		默认情况下，Spring会**自动搜索所有以@Component，@Controller，@Service，@Repository标注的 Java类，并将它们当成Spring  Bean来处理**。

​		还可以通过为<component-scan  .../>元素添加<include-filter  .../>或<execlude-filter  .../>子元素来指定Spring  Bean类，只要位于指定路径下的 Java类满足这种规则，即使这些 Java类没有使用任何注解标注，Spring一样会将它们当成Bean类来处理。

​		<include -filter  .../>元素用于满足该规则的 Java类会被当成Bean类处理，<exclude-filter  .../>指定满足该规则的 Java类不会被当成Bean类处理。

​		使用时需指定以下属性：

​				type：指定过滤器类型。

​				expression：指定过滤器所需要的表达式。

​		4种过滤器：

​				annotation：注解过滤器，该过滤器需要指定一个注解名。

​				assignable：类名过滤器，该过滤器直接指定一个 Java类。

​				regex：正则表达式过滤器，该过滤器指定一个正则表达式，匹配该正则表达式的 Java类将满足该过滤过		则。

​				aspectj：AspectJ过滤器。

```xml
<context:component-scan base-package="annotation">
        <context:include-filter type="regex" expression=".*Chinese" />
        <context:exclude-filter type="regex" expression=".*Axe" />
    </context:component-scan>
```

![](image/QQ截图20190710203854.png)

### 13.2、指定Bean的作用域

​		使用XML配置Bean时，通过scope属性来指定作用域，默认是singleton，注解使用@Scope来指定作用域。

​		在极端情况下，如果不想使用注解的方式来指定作用域，而是希望提供自定义的作用域解析器，那么可以自定义解析器，并实现ScopeMetadataResolver接口，并提供自定义的作用域解析策略，然后在配置扫描器时指定解析器的全限定类名即可。

### 13.3、使用@Resource配置依赖

​		Spring使用@Resource为目标Bean指定协作者Bean。其有一个name属性，在默认情况下，Spring将这个值解释为需要被注入的Bean实例的id，即与<property ../>元素的ref属性有相同的效果。

```java
@Component
public class Chinese implements Person
{
    private Axe axe;

    public Axe getAxe() {
        return axe;
    }

    @Resource(name = "stoneAxe")
    public void setAxe(Axe axe) {
        this.axe = axe;
    }

    @Override
    public void useAxe() {
        System.out.println(axe.chop());
    }
}
```

![](image/QQ截图20190710205558.png)

​		@Resource也可以修饰实例变量，此时Spring会直接使用 Java EE规范的Field注入，此时连setter方法都可以不要。

​		使用@Resource**修饰setter方法省略name属性时，name默认值为该setter方法去掉前面的set子串，首字母小写后得到的子串**。

​		使用@Resource**修饰实例变量省略name属性时，name默认值与该实例变量同名**。

### 13.4、使用@PostConstruct和@PreDestroy定制声明周期行为

​		@PostConstruct和@PreDestroy和<bean  .../>元素的init-method，destroy-method属性具有相似的作用。

​		@PostConstruct修饰的方法是Bean的初始化之后方法。

​		@PreDestroy修饰的方法是Bean销毁之前的方法。

## 13.5、Spring3.0新增的注解

​		@DependsOn：用于强制初始化其他Bean。修饰Bean类或方法，可以指定一个字符串数组作为参数，每个数组元素对应于一个强制初始化的Bean。

​		@Lazy：用于指定该Bean是否取消预初始化。修饰Bean类，可指定一个boolean类型的value属性，该属性决定是否要预初始化该Bean。

### 13.6、Spring4.0增强的自动装配和精确装配

​		Spring提供@AutoWired注解来指定自动装配，可以修饰setter方法，普通方法，实例变量和构造器。

​		**修饰setter方法时，默认采用byType自动装配策略**。此时Spring自动搜索容器中与setter方法参数类型匹配的Bean实例，如果**正好找到一个**，就**以该Bean作为参数执行setter方法**，如果**找到多个，那么会引发异常**，如果没有找到，就什么也不做，也不会引发异常。

​		**修饰带多个参数的普通方法时，Spring会自动到容器中寻找类型匹配的Bean，如果恰好为每个参数都找到一个类型匹配的Bean，Spirng会自动以这些Bean作为参数来调用该方法**。

​		**修饰一个实例变量时，与修饰setter方法一样**，找到一个就赋值，多个就会引发BeanCreateException异常。

​		当被修饰的实例变量为数组时，Spring会自动搜索容器中所有与数组元素类型匹配的Bean实例，并将这些实例作为数组元素来创建数组，最后将数组赋值给该数组变量。

​		当修饰的变量为集合类型，或者标注形参类型的集合方法时，与数组类型的处理是相同的。但是对于集合类型的参数而言，必须使用泛型。

​		@AutoWired还可以根据泛型进行自动装配。

​		为了实现精确的自动匹配，Spirng提供了@Qualifier注解，其允许根据Bean的id来执行自动装配。其还可以标注方法的形参。

![](image/QQ截图20190710213442.png)

## 14、资源访问

### 14.1、Resource实现类

​		Resource本身是一个接口，是具体资源访问策略的抽象，也是所有资源访问类所实现的接口。

![](image/QQ截图20190711213504.png)

​		Resource接口本身没有提供访问任何底层资源的实现逻辑，针对不同的底层资源，Spring提供不同的Resource实现类，不同的实现类负责不同的资源访问逻辑。其也可以作为资源访问的工具类使用。

​		UrlResource：访问网络资源的实现类。

​		ClassPathResource：访问类加载路径里资源的实现类。

​		FileSystemResource：访问文件系统里资源的实现类。

​		ServletContextResource：访问相对于ServletContext路径下的资源的实现类。

​		InputStreamResource：访问输入流资源的实现类。

​		ByteArrayResource：访问字节数组资源的实现类。

#### 1）、访问网络资源

​		UrlResource是URL类的包装，主要用于访问之前通过URL类访问的资源对象。URL资源通常应该提供标准的协议前缀。如：**file:用于访问文件系统；http:用于访问HTTP协议访问资源；ftp:用于通过FTP协议访问资源**等。

#### 2）、访问类加载路径下的资源

​		ClassPathResource用来访问类加载路径下的资源，相对于其他的Resource实现类，优势是方便访问类加载路径下的资源，尤其是对于Web应用，**ClassPathResource可自动所有位于WEB-INF/classes下的资源文件，无须使用绝对路径访问**。

​		**Spring的资源访问消除了底层资源访问的差异，允许程序以一致的方式来访问不同的底层资源。**

#### 3）、访问文件系统资源

​		FileSystemResource用于访问文件系统资源，相对于File类的优势就是，可以消除底层资源访问的差异，使用统一的Resource API来进行资源访问。

​		与前两种Resource进行资源访问的区别在于：资源字符串确定的资源，位于本地文件系统内，而且无须使用任何前缀。

#### 4）、访问应用相关资源

​		ServletContextResource类访问Web  Context下相对路径下的资源，ServletContextResource构造器接收一个代表资源位置的字符串参数，该资源位置是相对于Web应用跟路径的位置。

​		使用ServletContextResource访问的资源，也可以通过文件IO访问或URL访问，**通过File访问要求资源被解压缩，而且在本地文件系统中；使用ServletContextResource进行访问时无须关心资源是否被解压缩出来，或者直接存放在JAR文件中，总可通过Sservlet容器访问**。直通过**File来访问 Web  Context下相对路径下的资源时，应该先使用ServletContext的getRealPath()方法来取得资源绝对路径，在以该绝对路路径来创建File对象**。

#### 5）、访问字节数组资源

​		InputStreamResource来访问二进制输入流资源，InputStreamResource是这堆输入流的Resource实现，只有当没有合适的Resource实现时，才考虑使用该InputStreamResource，通常情况下，优先考虑使用ByteArrayResource，或者基于文件的Resource实现。

​		**InputStreamResource是一个总是被打开的Resource，所以isOpen()方法总是返回true，因此如果需要多次读取某个流，就不要使用InputStreamResource**，创建InputStreamResource实例时应该提供一个InputStream参数。

​		ByteArrayResource用于直接访问字节数组资源，可将字节数组包装成Resource使用。此时getFile和getFilename两个方法不可用。

### 14.2、ResourceLoader接口和ResourceLoaderAware接口

​		ResourceLoader接口和ResourceLoaderAware接口都是标志性接口。

​		ResourceLoader：该接口实现类的实例可以获得一个Resource实例。

![](image/QQ截图20190711223600.png)

​		ResourceLoaderAware：该接口实现类的实例将获得一个ResourceLoader的引用。

​		Spring将采用和ApplicationContext相同的策略来访问资源。如果ApplicationContext是FileSystemXmlApplicationContext，那么返回的就是FileSystemResource实例。如果ApplicationContext是ClassPathXmlApplicationContext，那么返回的就是ClassPathResource实例。如果ApplicationContext是XmlWebApplicationContext，那么返回的就是ServletContextResource实例。

​		也可以强制指定Resource实现类，可通过不同的前缀来指定：

![](image/QQ截图20190711225436.png)

​		ResourceLoaderAware接口提供了一个setResourceLoader()方法，该方法将由Spring容器负责调用，Spring容器会将一个ResourceLoader对象作为该方法的参数传入。

​		如果把实现ResourceLoaderAware接口的Bean类部署在Spring容器中，Spring容器会将自身当成ResourceLoader作为setResourceLoader方法的参数传入，由于ApplicationConetxt的实现类都实现了ResourceLoader接口，Spring容器自身完全可作为ResourceLoader使用。

```java
@Component("test")
public class TestBean implements ResourceLoaderAware
{
    private ResourceLoader resourceLoader;

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader)
    {
        this.resourceLoader = resourceLoader;
    }

    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }
}

public class Main
{
    public static void main(String [] args) throws Exception
    {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("annotation.xml");
        TestBean test = applicationContext.getBean("test",TestBean.class);
        ResourceLoader resourceLoader = test.getResourceLoader();
        System.out.println(applicationContext == resourceLoader);

    }
}

```

```xml
<context:component-scan base-package="annotation" />
```

![](image/QQ截图20190711230629.png)

### 14.3、使用Resource作为属性

​		程序中的Bean实例需要访问资源：

​				1、在代码中获取Resource实例。

​				2、使用依赖注入。				

​		使用第一种方式，无论使用那个Resource的实现类程序都必须提供资源位置，那么当资源位置变动，代码也要随之改动，所以推荐第二种。

### 14.4、在ApplicationContext中使用资源

​		不管以怎样的方式创建ApplicationConetxt实例，都需要为ApplicationContext指定配置文件，通常也是以Resource的方式来访问配置文件的。

​		ApplicationConetxt确定资源访问策略的两种方法：

​				1、使用ApplicationConetxt实现类指定访问策略。

​				2、使用前缀指定访问策略。

#### 1)、使用ApplicationConetxt实现类指定访问策略

![](image/QQ截图20190711232317.png)

#### 2）、使用前缀指定访问策略

​		在创建ApplicationConetxt时，在资源字符串中添加前缀，来指定资源访问策略。但是，此时指定的资源访问策略仅对访问有效，以后通过ApplicationConetxt实例访问资源时，还是会选择与ApplicationConetxt实例对应的资源访问策略。此时，也可以在进行资源访问的时候再次指定前缀，程序会根据前缀来确定资源访问策略。

![](image/QQ截图20190711233509.png)

![](image/QQ截图20190711233540.png)

![](image/QQ截图20190711233605.png)

![](image/QQ截图20190711233651.png)

![](image/QQ截图20190711233834.png)

## 15、Spring的AOP

​		AOP：面向切面编程，作为面向对象的一种补充，面向对象编程是从静态角度考虑程序结构，而面向切面编程则是从动态角度考虑程序运行过程。

​		AspectJ是一个基于 Java语言的AOP框架，提供了强大的AOP功能，Spring4.0的AOP与AspectJ进行了很好的继承。

​		AspectJ包含两部分：一部分定义了如何表达，定义AOP编程中的语法规则，通过这套语法规范，可以方便地用AOP来解决 Java语言中存在的交叉关注点的问题；另一部分是工具部分，包括编译器，调试工具等。

​		AOP要达到的效果是，保证在程序员不修改源代码的前提下，为系统中业务组件的多个业务方法添加某种通用功能。**本质：依然需要修改业务组件的多个业务方法的源代码，只是这个修改由AOP框架完成。**

​		两类（按AOP框架修改源码的时机）：

​				静态AOP实现：AOP框架在编译阶段对程序进行修改，即实现对目标类的增强，生成静态的AOP代理类	（生成的*.class文件已经被改掉了，需要使用特定的编译器）。AspectJ为代表。

​				动态AOP实现：AOP框架在运行阶段动态生成AOP代理(在内存中以JDK动态代理或cglib动态地生成AOP		代理类)，以实现对目标对象的增强。Spring  AOP为代表。

​		

### 15.1、AOP的基本概念

​		AOP从程序运行的角度考虑程序的流程，提取业务处理过程的切面。AOP面向的是程序运行中各个步骤，希望以更好的方式来组合业务处理的各个步骤。

​		AOP框架并不与特定的代码耦合，AOP框架能处理程序执行中特定的切入点，而不与某个具体类耦合。

​		两个特征：

​				1、各步骤之间的良好隔离性。

​				2、源代码无关性。

​		术语：

​				切面(Aspect)：切面用于组织多个Advice，Advice放在切面中定义。

​				连接点(Joinpoint)：程序执行过程中明确的点，在Spring  AOP中，连接点总是方法的调用。

​				增强处理(Advice)：AOP框架在特定的切入点执行的增强处理。处理有“around”，“before”，“after”等类		型。

​				切入点(Pointcut)：可以插入增强处理的连接点，简而言之，当某个连接点满足指定要求时，该连接点将		被添加增强处理，该连接点也就变成了切入点。

​				引入：将方法或字段添加到被处理的类中。Spring允许将新的接口引入到任何被处理的对象中。

​				目标对象：被AOP框架进行增强处理的对象，也被称为被增强的对象，如果AOP框架采用的是动态AOP		实现，那么该对象就是一个被代理的对象。

​				AOP代理：AOP框架创建的对象，代理就是对目标对象的加强。Spring中的AOP代理可以是JDK动态代		理，也可以是cglib代理，前者为实现接口的目标对象的代理，后者为不实现接口的目标对象的代理。

​				织入(Weaving)：将增强处理添加到目标对象中，并创建一个被增强的对象(AOP代理)的过程就是织入。		织入有两种实现方式：编译时增强和运行时增强。**Spring和其他纯 Java  AOP框架一样，在运行时完成织		入**。

​			AOP代理就是由AOP框架动态生成的一个对象，该对象可作为目标对象使用。AOP代理包含了目标对象的全部方法，但**AOP代理中的方法与目标对象的方法存在差异：AOP方法在特定切入点添加了增强处理，并回调目标对象的方法**。

![](image/QQ截图20190713163457.png)

### 15.2、Spring的AOP支持

​		Spring中的AOP代理由Spring的Ioc容器负责生成，管理，其依赖关系也由Ioc容器负责管理。因此，AOP代理可以直接使用容器中的其他Bean实例作为目标，Spring默认使用 Java动态代理来创建AOP代理，这样就可以为任何接口实例创建代理。

​		Spring也可以使用cglib代理，在需要代理类而不是代理接口的时候，Spring会自动切换为使用cglib代理。

​		Spring目前仅支持将方法调用作为连接点，如果需要把成员变量的访问和更新也作为增强处理的连接点，则可以考虑使用AspectJ。

​		AOP编程三步：

​				1、定义普通业余组件。

​				2、定义切入点，一个切入点可以横切多个业务组件。

​				3、定义增强处理，增强处理就是在AOP框架为普通业务组件织入的处理动作。

### 15.3、基于注解的零配置方式

​		Spring只是使用了和AspectJ 5一样的注解，但并没有使用AspectJ 的编译器或者织入器，底层依然使用的是Spring  AOP，依然是在运行时动态生成AOP代理。

​		为了启用Spring对@AspectJ切面配置的支持，并保证Spring容器中的目标Bean被一个或多个切面自动增强。需要使用下面的配置。

```xml
<aop:aspectj-autoproxy />
<!-- 或者 -->
<bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator" />
```

​		自动增强：Spring会判断一个或多个切面是否需要对指定Bean进行增强，并据此自动生成相应的代理，从而使得增强处理哎合适的时候被调用。

#### 1）、定义切面Bean

​		当启动了@AspectJ 支持后，只要在Spring容器中配置一个带@AspectJ注解的Bean。

​		切面类和其他类一样可以有方法，成员变量定义，还可能包含切入点，增强处理定义。

​		当使用@AspectJ来修饰一个Java类之后，Spring将不会把该Bean当成组件Bean处理，因此负责自动增强的后处理Bean将会略过该Bean，不会把对该Bean进行任何增强处理。

#### 2）、定义Before增强处理

​		在切面类使用@Before来修饰一个方法，该方法将作为Before增强处理，使用@Before修饰时，通常需要指定一个value属性值，该属性值定义一个切入点表达式(既可以时一个已有的切入点，也可以直接定义切入点表达式)，用于指定该增强处理将被织入哪些切入点。

![](image/QQ截图20190713211223.png)

![](image/QQ截图20190713212747.png)

#### 3）、定义AfterReturning增强处理

​		使用@AfterReturning可修饰AfterReturning增强处理，AfterReturning增强处理将在目标方法正常完成后被织入。

​		两个属性：

​				pointcut/value：这两个属性的作用一样，它们都用于指定该切入点对应的切入表达式。当指定了			pointcut属性值后，value属性将会被覆盖。

​				returning：该属性值指定一个形参名，用于表示Advice方法中可定义与此同名的形参，该形参可用于访		问目标方法的返回值，除此之外，在Advice方法中定义该形参(代表目标方法的返回值)时指定的类型，会限制		目标方法返回指定类型的值或没有返回值。

![](image/QQ截图20190713213453.png)

#### 4）、定义AfterThrowing增强处理

​		使用@AfterThrowing注解可修饰AfterThrowing增强处理，AfterThrowing主要用于处理程序中未处理的异常。

​		两个属性：

​				pointcut/value：指定该切入点对应的切入表达式，当指定pointcut属性值后，value属性值将会被覆		盖。

​				throwing：该属性指定一个形参名。该形参可用于访问目标方法抛出的异常，定义该形参时指定的类		型，会限制目标方法必须抛出指定类型的异常。

![](image/QQ截图20190713214145.png)

​		AfterThrowing虽然处理了异常，但不能完全处理该异常，该异常依然会传播到上一级调用者。

#### 5）、After增强处理

​		After增强处理不管目标方法如何结束（包括成功完成和遇到异常中止两种情况），它都会被织入。

​		需要指定一个value属性：用于指定增强处理被注入的切入点。

#### 6）、Around增强处理

​		Around近似等于Before，AfterReturning的总和。既可以在执行目标方法之前织入增强动作，也可在执行目标方法之后织入增强动作。但是其可以决定目标方法在什么时候执行，如何执行，甚至可以完全阻止目标方法的执行。也可以改变执行目标方法的参数值，也可以改变执行目标方法之后的返回值。·但是**通常需要在线程安全的环境下使用**。**需要目标方法之前和之后共享某种状态数据，应该考虑使用Around增强处理，尤其是需要改变目标方法的返回值时，则只能使用Around增强处理了**。

​		@Around注解需要指定一个value属性：指定该增强处理被织入的切入点。

​		定义一个Around增强处理方法时，至少包含一个形参，且第一个形参必须是ProceedingJoinPoint类型。这是Around增强处理的可以完全控制目标方法的执行时机，如何执行的关键，如果程序没有调用ProceedingJoinPoint的proceed方法，则目标方法不会被执行。调用proceed时，还可以传入一个Oject[]对象作为参数，该数组的值将被传入目标方法作为执行方法的实参。此时，如果传入的Object数组的长度或者元素类型不与目标方法匹配时，程序会出现异常。

#### 7、访问目标方法的参数

​		最简单的做法：定义增强处理方法时将第一个参数定义为JoinPoint类型。JoinPoint参数代表了织入增强处理的连接点。

![](image/QQ截图20190713223247.png)

​		进入连接点，高优先级将先被织入(Before，优先级高的限制性)，退出时，高优先级的最后被织入。(After，高优先级的后执行)。

​		指定优先级：

​				1、实现Ordered接口，实现getOrder()方法，返回的值越小，优先级越高。

​				2、使用@Order修饰切面类，value值越小，优先级越高。

​		同一个切面类里的两个相同类型的增强处理在同一个连接点被织入时，以随即的顺序来织入这两个增强处理。

![](image/QQ截图20190713224024.png)

​		args两个作用：

​				1、提供一种简单的方式来访问目标方法的参数。

​				2、对切入表达式增加额外的限制。

#### 8）、定义切入点

​		Spring  AOP只支持Spring  Bean的方法执行作为连接点，可以把切入点堪称所有能和切入点表达式匹配的Bean方法。

​		两个部分：

​				1、切入点表达式。

​				2、包含名字和任意参数的方法签名。

​		切入点签名采用一个普通的方法定义(方法体通常为空)来提供，且方法的返回值必须为void，切入点表达式需要使用@Pointcut注解来标注。一旦定义了，可以在本切面类，其他切面类，或者其他包的切面类使用，这取决于签名前面的访问控制符。如果要引用其他包的，必须添加类名前缀。

#### 9）、切入点指示符

![](image/QQ截图20190713225702.png)

![](image/QQ截图20190713225736.png)

![](image/QQ截图20190713225822.png)

![](image/QQ截图20190713225843.png)

![](image/QQ截图20190713225903.png)

![](image/QQ截图20190713225920.png)

![](image/QQ截图20190713225943.png)

#### 10）、组合切入点表达式

![](image/QQ截图20190713230126.png)

### 15.4、基于XML配置文件的管理方式

​		在Spring配置文件中，所有的切面，切入点和增强处理都必须定义在<aop:config .../>元素内部。<beans .../>元素可以包含多个<aop:config .../>元素，一个<aop:config  .../>可以包含pointcut，advisor，aspect元素，且这三个元素必须按照此顺序来定义。

![](image/QQ截图20190713203334.png)

#### 1）、配置切面

​		使用<aop:aspect .../>元素定义切面，实质是将一个已有的Spring  Bean转换成切面Bean，所以需要先定义一个普通的Spring  Bean。

​		三个属性：

​				id：定义该切面额标识名。

​				ref：用于将ref属性所引用的普通Bean转换成切面Bean。

​				order：指定该切面Bean的优先级，order属性值越小，该切面对应的优先级越高。

#### 2）、配置增强处理

​		使用XML配置增强处理分别依赖如下元素：

​				<aop:before .../>：配置Before增强处理。

​				<aop:after .../>“配置After增强处理。

​				<aop:after-returning .../>：配置AfterReturning增强处理。

​				<aop:after-throwing .../>：配置AfterThrowing增强处理。

​				<aop:around .../>配置Around增强处理。

​		属性：

​				pointcut：该属性指定一个切入表达式，Spring将在匹配该表达式的连接点时织入该增强处理。

​				pointcut-ref：该属性指定一个已经存在的切入点名称，通常pointcut和pointcut-ref两个属性只需使用		其中之一。

​				method：该属性指定一个方法名，指定将切面的该方法转换为增强处理。

​				throwing：该属性只对<ater-throwing .../>元素有效，用于指定一个形参名，AfterThrowing增强处理方		法可通过该形参访问目标方法所抛出的异常。

​				returning：该属性只对<after-returning .../>元素有效，用于指定一个形参名，AfterReturning增强处理		方法可通过该形参访问目标方法的返回值。

```xml
<context:component-scan base-package="annotation" />
    <aop:config>
        <aop:aspect id="fourAdviceAspect" ref="fourAdviceTest" order="2">
            <aop:after pointcut="execution(* annotation.*.*(..))" method="release" />
            <aop:before method="authority" pointcut="execution(* annotation.*.*(..))" />
            <aop:after-returning method="log" pointcut="execution(* annotation.*.*(..))" returning="rvt" />
            <aop:around method="processTx" pointcut="execution(* annotation.*.*(..))" />
        </aop:aspect>
        <aop:aspect id="secondAdviceTest" ref="secondAdviceTest" order="1">
            <aop:before method="authrity" pointcut="execution(* annotation.*.*(..)) and args(aa)" />
        </aop:aspect>
    </aop:config>
```

## 16、缓存机制

​		此种缓存机制可以与Spring容器无缝地整合在一起，可以对容器中的任意Bean或者Bean的方法增加缓存。与Hibernate的二级缓存相比，级别更高。

### 16.1、启用Spring缓存

​		为了启用Spring缓存，需要在配置文件中导入cache:命名空间。

​		然后进行下面两步：

​				1、在Spring配置文件中添加<cache:annotation-driven  cache-manager=""  />，该元素指定Spring根		据注解来启用Bean级别或者方法级别的缓存。默认值为cacheManager，即如果将容器中缓存管理器的ID设		为cacheManager，则可省略<cache:annotation-driven  .../>的cache-manager属性。

​				2、针对不同的缓存实现配置对应的缓存管理器。

#### 1）、Spring内置缓存实现的配置

​		Spring内置的缓存实现只是一种内存中的缓存，并非真正的缓存实现。不建议在实际项目中使用Spring内置的缓存实现。

​		Spring内置的缓存实现使用SimpleCacheManger作为缓存管理器，使用时直接在Spring容器中配置该Bean，然后通过<property  ../>驱动该缓存管理器执行setCaches()方法来设置缓存其即可。

​		SimpleCacheManager是一种内存中的缓存区，底层直接使用了JDK的ConcurrentMap来实现缓存，SimpleCacheManager使用了ConcurrentMapCacheFactoryBean作为缓存区，每个ConcurrentMapCacheFactoryBean配置一个缓存区。

```xml
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default" />
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="user" />
            </set>
        </property>
    </bean>
```

#### 2）、EhCache缓存实现的配置

​		为了使用EhCache，需要在类加载路径下添加一个ehcache.xml配置文件。

```xml
<dependency>
      <groupId>org.ehcache</groupId>
      <artifactId>ehcache</artifactId>
      <version>3.7.0</version>
    </dependency> 
```

```xml

<!-- ehcache3.x 与之前的版本有很大差距 -->
<config
        xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'
        xmlns='http://www.ehcache.org/v3'
        xsi:schemaLocation="http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core.xsd">
    <cache alias="default">
        <key-type>java.lang.String</key-type>
        <value-type>java.lang.String</value-type>
        <resources>
            <heap unit="entries">2000</heap>
            <offheap unit="MB">20</offheap>
        </resources>
    </cache>
    <cache alias="user">
        <key-type>java.lang.String</key-type>
        <value-type>java.lang.String</value-type>
        <resources>
            <heap unit="entries">2000</heap>
            <offheap unit="MB">20</offheap>
        </resources>
    </cache>
</config>
```

​		Spring使用EhCacheManager作为EhCache缓存实现的缓存管理器，只要该对象配置在Spring容器中，就可作为缓存管理器使用，但EhCacheManager底层需要依赖一个net.sf.ehcache.CacheManager作为实际的缓存管理器。

​		为了将CacheManager纳入Spring容器的管理之下，Spring提供了CacheManagerFactoryBean工厂Bean，其实现了FactoryBean<CacheManager>接口，当程序把CacheManagerFactoryBean部署在Spring容器中，并通过Spring容器请求该工厂Bean时，实际返回的是它的产品：CacheManager对象。

```xml
<bean id="ehCacheManager" class="org.springframework.cache.ehcache.EhcacheManagerFactoryBean" p:configLocation="classpath:ehcache.xml" p:shared="false" />
    <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cacheManager-ref="ehCacheManager" />
```

### 16.2、使用@Cacheable执行缓存

​		@Cacheable可用于修饰类和方法，当修饰类时，将在类级别上进行缓存，程序调用该类的实例的任何方法都需要缓存，共享同一个缓存区。修饰方法时，在方法级别上进行缓存，当程序调用方法时才需要缓存。

#### 1）、类级别的缓存

​		当使用类级别缓存时，当程序调用该类的任意方法时，只要传入的参数相同，Spring就会使用缓存，并不会执行相应的方法。

​		类级别的缓存默认以所有方法参数作为key来缓存方法返回的数据：即同一个类不管调用哪个方法，只要调用方法时传入的参数相同，Spring都会直接利用缓存区中的数据。

​		属性：

​				value：必须属性。可指定多个缓存区的名字，用于指定将方法返回值放入指定的缓存区。

​				key：通过SpEL表达式显式指定缓存的key。

​				condition：指定一个返回boolean的SpEL表达式，只有返回true时，Spring才会缓存方法返回值。

​				unless：返回一个boolean的SpEL表达式，返回true时，Spring不缓存方法返回值。

![](image/QQ截图20190715171210.png)

#### 2）、方法级别的缓存

​		当修饰方法时，在方法级别上进行缓存，当调用方法传入的参数相同时，Spring就会使用缓存。

### 16.3、使用@CacheEvict清除缓存

​		@CacheEvict修饰方法可用于清除缓存，属性：

​				value：必需属性：指定该方法清除那个缓存区的数据。

​				allEntries：指定是否清空整个缓存区。

​				beforeInvocation：指定是否在执行方法之前清除缓存。默认是在方法成功完成之后才清除缓存。

​				condition：指定一个SpEL表达式，当返回true时才清除缓存。

​				key：通过SpEL表达式显式指定缓存的key。

## 17、Spring的事务

### 17.1、Spring支持的事务策略

​		Java  EE应用的传统事务有两种策略：全局事务和局部事务。

​		全局事务：由应用服务器管理，需要底层服务器的JTA支持。

​		局部事务：与底层所采用的持久化技术有关，当采用JDBC持久化技术时，需要使用Connection对象来操作事务，而采用Hibernate持久化技术时，需要使用Session对象来操作事务。

​		全局事务可以跨多个事务性资源，局部事务，应用服务器不需要参与事务管理，因此不能保证跨多个事务性资源的事务的正确性。

![](image/QQ截图20190716171158.png)

​		Spring事务策略是通过PlatformTransactionManager接口体现的，该接口是Spring事务策略的核心。

![](image/QQ截图20190716173932.png)

​		getTransaction(TransactionDefinition  definition)返回的TransactionStatus对象，可能是一个新的事务，也可能是一个已经存在的事务对象。如果当前执行的线程已经处于事务管理下，则返回当前线程的事务对象，否则，系统将新建一个事务对象后返回。

​		TransactionDefinition 接口定义了一个事务规则。必须指定如下属性：

​				事务隔离：当前事务和其他事物的隔离程度。

​				事务传播：通常，在事务中执行的代码都会在当前事务中运行。但是，如果一个事务上下文已经存在，		有几个选项可指定该事务性方法的执行行为。

​				事务超时：事务在超时前能运行多久，也就是事务的最长持续时间。如果事务一直没有被提交或回滚，		将在超出该时间后，系统自动回滚事务。

​				只读状态：只读事务不修改任何数据，在某些情况下，只读事务是非常有用的优化。

​		TransactionStatus代表事务本身，提供了简单的控制事务执行和查询事务状态的方法，这些方法在所有的事务API中都是相同的。

![](image/QQ截图20190716175444.png)

​		Spring具体的事务管理由PlatformTransactionManager的不同实现类来完成。Spring容器中配置PlatformTransactionManager  Bean时，必须针对不同的环境提供不同的实现类。

​		采用不同的持久层访问环境的配置文件：

​				JDBC：

![](image/QQ截图20190717100422.png)

​				JTA：

![](image/QQ截图20190717100454.png)

​				Hibernate：

![](image/QQ截图20190717100545.png)

​				底层采用Hibernate，但事务采用JTA全局事务：

![](image/QQ截图20190717100641.png)

![](image/QQ截图20190717101635.png)

​		Spring的两种事务管理方式：

​				编程式事务管理：即使使用Spring的编程式事务，程序也可直接获取容器中的transactionManager  		Bean，该Bean总是PlantformTransactionManager的实例，所以可以通过该接口提供的三个方法来开始事		务，提交事务和回滚事务。

​				声明式事务：无须在 Java程序中书写任何事物操作代码，而是通过在XML文件中为业务组件配置事务代		理（AOP代理的一种），AOP为事务代理所织入的增强处理也由Spring提供，在目标方法执行之前，织入开		始事务，在目标方法执行之后，织入结束事务。

​		

​		Spring的编程式事务还可通过TransactionTemplate类来完成，该类提供了一个execute(TransactionCallback  action)方法，可以以更简捷的方式来进行事务操作。

### 17.2、使用XML  Schema配置事务策略

​		推荐使用声明式事务策略。

​		tx:命名空间来配置事务管理，其下提供了<tx:advice  .../>元素来配置事务增强处理，一旦使用该元素配置了事务增强处理，就可直接使用<aop:advisor  .../>元素启用自动代理了。

​		配置<ex:advice  .../>元素时除了需要transaction-manager属性指定事务管理器之外，还需要配置一个<attributes .../>子元素，该子元素里又可包含多个<method  .../>子元素。

![](image/QQ截图20190717104014.png)

​		配置<tx:advice  .../>元素的重点就是配置<method  .../>子元素。

​				属性：

![](image/QQ截图20190717104458.png)

​		<method .../>子元素的propagetion属性指定的事务传播行为的枚举值：

![](image/QQ截图20190717104646.png)

![](image/QQ截图20190717105646.png)

​		当采用<aop:advisor .../>元素将Advice和切入点绑定时，实际上是由Spring提供的Bean后处理器完成的。BeanNameAutoProxyCreator、DefaultAdvisorAutoProxyCreator两个Bean后处理器，它们都可以对容器中的Bean执行后处理（为它们织入切面中包含的增强处理）。当配置<aop:advisor .../>元素时传入一个txAdvice事务增强处理，所有Bean后处理器将为所有Bean实例里匹配切入点的方法织入事务操作的增强处理。

​		默认情况下，只有方法引发运行时异常和unchecked异常时，Spring事务机制才会自动回滚事务。可以使用rollback-for属性强制遇到checked异常时自动回滚事务。

![](image/QQ截图20190717110728.png)

### 17.3、使用@Transactional

​		@Transactional注解，既可以修饰Spring  Bean类，也可用于修饰Bean类中的某个方法。修饰类，事务将对整个Bean类其作用，修饰方法，只对该方法其作用。

​		属性：

![](image/QQ截图20190717111017.png)

​		还需要在配置文件中让Spring根据注解来配置事务代理。

![](image/QQ截图20190717111230.png)

## 18、Spring整合Struts  2

### 18.1、启动Spring容器

​		在Web应用中创建Spring容器有两种方式：

​				1、直接在web.xml文件中配置创建Spring容器。

​				2、利用第三方MVC框架的扩展点，创建Spring容器。

​		第一种创建Spring容器的方式更加常见，为了让Spring容器随Web应用的启动而自动启动，借助于ServletContextListener监听器即可完成，该监听器可以在Web应用启动时回调自定义方法，该方法就可以启动Spring容器。

​		Spring提供了一个ContextLoderListener，该监听器实现了ServletContextListener接口，该类可作为Listener使用，它会在创建时自动查找WEB-INF下的applicationContext.xml文件。如果有多个配置文件需要载入，则考虑使用<context-parm .../>元素来确定配置文件的文件名。参数名为contextConfigLocation。如果既没有指定contextConfigLocation参数，WEB-INF下也没有applicationContext.xml文件，Spring将无法正常初始化。

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```

​		Spring根据指定的配置文件创建WebApplicationContext对象，并将其保存在ServletContext中。

​		获取：

```java
WebApplicationContextUtils.getWebApplicationContext(servletContext);
```

​		也可以通过ServletContext的getAttribute方法获取ApplicationContext。**ApplicationContext在ServletContext中的属性名：WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE**。

### 18.2、MVC框架与Spring整合的思考

​		工厂模式顺序图：

![](image/QQ截图20190718164927.png)

​		控制器访问Spring容器中的业务逻辑组件：

​				1、Spring容器负责管理控制器Action，并利用依赖注入为控制器注入业务逻辑组件。

​				2、利用Spring的自动装配，Action将会自动从Spring容器中获取所需的业务逻辑组件。

### 18.3、让Spring管理控制器

![](image/QQ截图20190718213652.png)		在定义action时，class不再指定Action的实际实现类，而是指定为Spring容器中的Bean  ID。此时处理用户请求的Action由Spring插件负责创建，但创建Action实例时，并不是利用配置Action时指定的class属性来创建该Action实例，而是从Spring容器中取出对应的Bean实例完成创建的。

​		当使用Spring容器管理系统的Action，在struts.xml文件中配置该Action时，class属性并不是指向该Action的实现类，而是指定了Spring容器中Action实例的ID。

​		当Spring管理Struts 2的Action时，一定要配置scope属性，因为Action里包含了请求的状态信息，必须为每个请求对应一个Action，所以不能将该Action实例配置成singleton行为。

![](image/QQ截图20190722230633.png)

### 18.4、使用自动装配

![](image/QQ截图20190718225744.png)

## 19、Spring整合Hibernate

### 19.1、Spring提供的DAO支持

​		DAO模式：

​				所有的数据库访问都通过DAO组件完成，DAO组件封装了数据库的增，删，改等原子操作。业务逻辑组		件依赖于DAO组件提供的数据库原子操作，完成系统业务逻辑的实现。

![](image/QQ截图20190719153724.png)

### 19.2、管理Hibernate的SessionFactory

​		以声明式的方式将SessionFactory实例加入到Spring的IoC容器中。此时SessionFactory将随应用的启动而加载。

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close"
          p:driverClass="${jdbc.driver}"
          p:jdbcUrl="${jdbc.url}"
          p:user="${jdbc.username}"
          p:password="${jdbc.password}"
          p:maxPoolSize="40"
          p:minPoolSize="2"
          p:initialPoolSize="2"
          p:maxIdleTime="30" />
    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean" p:dataSource-ref="dataSource">
        <property name="annotatedClasses">
            <list>
                <value>com.shanji.login.entity.User</value>
            </list>
        </property>
        <property name="hibernateProperties">
            <props>
                <prop key="dialect">org.hibernate.dialect.MySQL8Dialect</prop>
                <prop key="hbm2ddl.auto">update</prop>
            </props>
        </property>
    </bean>
```

### 19.3、实现DAO组件的基类

​		所有的DAO组件都应该提供的方法：

​				1、根据ID加载持久化实体。

​				2、保存持久化实体。

​				3、更新持久化实体。

​				4、删除持久化实体，以及根据ID删除持久化实体。

​				5、获取所有的持久化实体。

### 19.4、传统的HibernateTemplate和HibernateDaoSupport

​		Spring为整合Hibernate3提供了HibernateTemplate和HibernateDaoSupport两个工具类。

​		HibernateTemplate提供了持久层访问模版化。只需传入一个SessionFactory，就可以执行持久化操作。

![](image/QQ截图20190720012848.png)

​		Spring为实现DAO组件提供了工具基类：HibernateDaoSupport。

![](image/QQ截图20190720013103.png)

![](image/QQ截图20190720014338.png)

### 19.5、使用IoC容器组装各种组件

![](image/QQ截图20190720095014.png)

## 20、Spring整合JPA

### 20.1、管理EntityManagerFactory

​		Spring可通过容器管理JPA的EntityManagerFactory，将JPA  EntityManagerFactory部署在Spring容器中。

​		好处：

​				以声明式方式管理EntityManagerFactory，避免在程序中手动创建EntityManagerFactory。

​				可以方便地将EntityManagerFactory作为基础资源注入其他组件中。

​		方式：

​				LocalEntityManagerFactoryBean。

​				LocalContainerEntityManagerFactoryBean。

#### 20.1.1、使用LocalEntityManagerFactoryBean

​		LocalEntityManagerFactoryBean可用于创建一个EntityManagerFactory。但是创建的EntityManagerFactory在很多情况都是受限的，它不能使用Spring容器中已有的DataSource，也不能切换到全局事务。只有需要使用JPA进行简单的数据访问，或者简单的测试才会采用LocalEntityManagerFactoryBean来配置EntityManagerFactory。

```xml
<bean id="emf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean" p:persistenceUnitName="user_pu" />
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence">
    <persistence-unit name="user_pu" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.shanji.login.entity.User</class>
        <properties>
            <property name="hibernate.connection.driver_class" value="com.mysql.cj.jdbc.Driver" />
            <property name="hibernate.connection.url" value="jdbc:mysql://localhost:3306/oa" />
            <property name="hibernate.connection.username" value="root" />
            <property name="hibernate.connection.password" value="179980" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

#### 20.1.2、使用LocalContainerEntityManagerFactoryBean

​		使用LocalContainerEntityManagerFactoryBean可以提供对EntityManagerFactory的全面控制，LocalContainerEntityManagerFactoryBean将根据persistence.xml文件创建PersistenceUnitInfo，并提供DataSourceLookup策略，因此它完全可以直接使用Spring容器中已有的数据源，并之际控制织入流程。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence">
    <persistence-unit name="user_pu" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.shanji.login.entity.User</class>
        <properties>
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

```xml
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean" p:dataSource-ref="dataSource">
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <property name="showSql" value="true" />
                <property name="database" value="MYSQL" />
            </bean>
        </property>
    </bean>
```

### 20.2、实现DAO组件基类

​		使用Spring容器管理EntityManagerFactory之后，Spring可以通过EntityManagerFactory获取线程安全的EntityManager，并将该EntityManager注入DAO组件。

![](image/QQ截图20190722150603.png)

​		applicationContext.xml配置

```xml
<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" />
```

### 20.3、使用声明式事务

​		Spring与JPA整合之后，所有的持久化操作都应该在事务环境中进行，否则对数据库所做的持久化操作都不会自动提交。

​		JPA的事务管理器：

```xml

```

# 二、设计模式

## 1、单例模式

​		如果一个类始终只能创建一个实例，则这个鳄梨被称为单例类，这种模式就被称为单例模式。

![](image/QQ截图20190727135742.png)

```java
public class Singleton
{
    private static Singleton instance;
    private Singleton()
    {

    }
    public static Singleton getInstance()
    {
        if(instance == null)
        {
            instance = new Singleton();
        }
        return instance;
    }
}

```

## 2、简单工厂

​		将多个类对象交给工厂类来生成的设计方式被称为简单工厂模式。

```java
public interface Output
{
    final int MAX_CACHE_LINE  = 10;
    void out();
    void getData(String message);
}
```

```java
public class Printer implements Output
{
    private String[] printData = new String[MAX_CACHE_LINE];

    private int dataNum = 0;

    @Override
    public void out()
    {
        while (dataNum > 0)
        {
            System.out.println("打印机打印：" + printData[0]);
            System.arraycopy(printData,1,printData,0,--dataNum);
        }
    }

    @Override
    public void getData(String message)
    {
        if(dataNum >= MAX_CACHE_LINE)
        {
            System.out.println("输出队列已满，添加失败");
        }
        else
        {
            printData[dataNum++] = message;
        }
    }
}
```

```java
public class OutputFactory
{
    public Output getOutput()
    {
        return new Printer();
    }
}
```

```java
public class Computer
{
    private Output out;
    public Computer(Output out)
    {
        this.out = out;
    }
    public void keyIn(String message)
    {
        out.getData(message);
    }

    public void print()
    {
        out.out();
    }
    public static void main(String [] args)
    {
        OutputFactory of = new OutputFactory();
        Computer c = new Computer(of.getOutput());
        c.keyIn("小杉杉");
        c.keyIn("小山鸡");
        c.print();
    }
}
```

简单的Ioc容器

```java
public class simplyIocContext implements ApplicationContext
{

    private Map<String,Object> objPool = Collections.synchronizedMap(new HashMap<String, Object>());

    private Document doc;

    private Element root;

    public simplyIocContext(String filePath) throws Exception
    {
        SAXReader reader = new SAXReader();
        doc = reader.read(new File(filePath));
        root = doc.getRootElement();
        initPool();
        initProp();
    }

    @Override
    public Object getBean(String name) throws Exception {
        Object target = objPool.get(name);
        if(target.getClass() != String.class)
        {
            return target;
        }
        else
        {
            String clazz = (String)target;
            return Class.forName(clazz).newInstance();
        }
    }
    private void initPool() throws Exception
    {
        for(Object obj : root.elements())
        {
            Element beanEle = (Element)obj;
            String beanId = beanEle.attributeValue("id");

            String beanClazz = beanEle.attributeValue("class");

            String beanScope = beanEle.attributeValue("scope");

            if(beanScope == null || beanScope.equals("singleton"))
            {
                objPool.put(beanId,Class.forName(beanClazz).newInstance());
            }
            else
            {
                objPool.put(beanId,beanClazz);
            }
        }
    }

    private void initProp() throws Exception
    {
        for(Object obj : root.elements())
        {
            Element beanEle = (Element)obj;
            String beanId = beanEle.attributeValue("id");

            String beanScope = beanEle.attributeValue("scope");

            if(beanScope == null || beanScope.equals("singleton"))
            {
                Object bean = objPool.get(beanId);
                for(Object prop : beanEle.elements())
                {
                    Element propEle = (Element)prop;

                    String propName = propEle.attributeValue("name");

                    String propValue = propEle.attributeValue("value");

                    String propRef = propEle.attributeValue("ref");

                    String propNameCamelize = propName.substring(0,1).toUpperCase() + propName.substring(1,propName.length());

                    if(propValue != null && propValue.length() > 0)
                    {
                        Method setter = bean.getClass().getMethod("set" + propNameCamelize,String.class);
                        setter.invoke(bean,propValue);
                    }

                    if(propRef != null && propRef.length() > 0)
                    {
                        Object target = objPool.get(propRef);
                        if(target == null)
                        {

                        }

                        Method setter = null;
                        for(Class superInterface : target.getClass().getInterfaces())
                        {
                            try
                            {
                                setter = bean.getClass().getMethod("set" + propNameCamelize,superInterface);
                                break;
                            }
                            catch (NoSuchMethodException ex)
                            {
                                continue;
                            }

                        }
                        if(setter == null)
                        {
                            setter = bean.getClass().getMethod("set" + propNameCamelize,target.getClass());
                        }
                        setter.invoke(bean,target);
                    }
                }
            }
        }
    }
}

```

## 3、工厂方法和抽象工程

​		为了解决客户端代码和不同工厂类耦合的问题，接着考虑在增加一个工厂类，该工厂类不制造具体的被调用对象，而是制造不同工厂对象，这个特殊的工厂类呗称为抽象工厂类，这种设计模式也被称为抽象工厂模式。

```java
public class OutputFactoryFactory
{
    public static OutputFactory getOuputFactory(String type)
    {
        if(type.equalsIgnoreCase("better"))
        {
            return new BetterPrinterFactory();
        }
        else
        {
            return new PrinterFactory();
        }
    }
}
```

## 4、代理模式

​		当客户端代码需要调用某个对象时，客户端实际上也不关心是否准确得到该对象，它只要一个能提供该功能的对象即可，此时就可返回对象的代理。此时系统会为某个对象提供一个代理对象，并由代理对象控制对源对象的引用。

```java
public interface Image
{
    void show();
}
```

```java
public class BigImage implements Image
{
    public BigImage()
    {
        try
        {
            Thread.sleep(3000);
            System.out.println("图片加载成功");
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }

    @Override
    public void show() {
        System.out.println("绘制实际的大图片");
    }
}
```

```java
public class ImageProxy implements Image
{
    private Image image;

    public ImageProxy(Image image)
    {
        this.image = image;
    }

    @Override
    public void show() {
        if(image == null)
        {
            image = new BigImage();
        }
        image.show();
    }
}
```

​		jdk动态代理：

```java
public class GunDog implements Dog
{
    @Override
    public void info() {
        System.out.println("我是一只猎狗");
    }

    @Override
    public void run() {
        System.out.println("我奔跑迅速");
    }
}
```

```java
public class TxUtil
{
    public void beginTx()
    {
        System.out.println("=======模拟开始事务");
    }
    public void endTx()
    {
        System.out.println("=======模拟结束书屋");
    }
}
```

```java
public class StyleInvokationHander implements InvocationHandler
{
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        TxUtil tx = new TxUtil();
        tx.beginTx();
        Object invoke = method.invoke(target, args);
        tx.endTx();
        return invoke;
    }
}
```

```java
public class StyProxyFactory
{
    public static Object getProxy(Object object)
    {
        StyleInvokationHander hander = new StyleInvokationHander();
        hander.setTarget(object);
        return Proxy.newProxyInstance(object.getClass().getClassLoader(),object.getClass().getInterfaces(),hander);
    }
}
```

## 5、命令模式

​		某个方法不仅要求参数可以变化，甚至要求方法执行体的代码页可以变化，需要能把处理行为作为一个参数传入该方法。

​		Java 8开始支持的Lambda表达式也可以实现这种功能。

​		在Java中，类才是一等公民，方法也不能独立存在，所以实际传入该方法的应该是一个对象，该对象通常是某个接口的匿名实现类的实例，该接口通常被称为命令接口，该设计方法也被称为命令模式。

```java
public interface Command
{
    void process(int[] target);
}
```

```java
public class ProcessArray
{
    public void each(int[] target,Command cmd)
    {
        cmd.process(target);
    }
}
```

```java
public static void main(String [] args)
    {
        ProcessArray pa  = new ProcessArray();
        int[] target = {3,-4,6,4};
        pa.each(target, new Command() {
            @Override
            public void process(int[] target) {
                for(int temp : target)
                {
                    System.out.println("迭代输出目标数组的元素：" + temp);
                }
            }
        });
    
    	/*  Lambda表达式
    		pa.each(target, array ->{
            for(int temp : array)
            {
                System.out.println("迭代输出目标数组的元素：" + temp);
            }
        });
    	*/
    	
        System.out.println("----------------------");
        pa.each(target, new Command() {
            @Override
            public void process(int[] target) {
                int sum = 0;
                for(int temp : target)
                {
                    sum += temp;
                }
                System.out.println("数组元素的总和是：" + sum);
            }
        });
    }
```

​		**就像是js中当某个按钮的点击事件中的编写的函数。**

## 6、策略模式

​		策略模式用于封装系列的算法，通常被封装与一个被称为Context的类中，客户端程序可以自由选择其中一种算法，或让Context为客户端选择一个最佳的算法。优势：为了支持算法的自由切换。

```java
public interface DiscountStrategy
{
    double getDiscount(double originPrice);
}

```

```java
public class VipDiscount implements DiscountStrategy
{
    @Override
    public double getDiscount(double originPrice) {
        System.out.println("使用vip折扣。。。。");
        return originPrice * 0.5;
    }
}
```

```java
public class OldDiscount implements DiscountStrategy
{

    @Override
    public double getDiscount(double originPrice) {
        System.out.println("使用旧书折扣。。。");
        return originPrice * 0.7;
    }
}

```

```java
public class DiscountContext
{
    private DiscountStrategy strategy;
    public DiscountContext(DiscountStrategy strategy)
    {
        this.strategy = strategy;
    }

    public double getDiscountProce(double price)
    {
        if(strategy == null)
        {
            strategy = new OldDiscount();
        }
        return this.strategy.getDiscount(price);
    }
    public void changeDiscount(DiscountStrategy strategy)
    {
        this.strategy = strategy;
    }
}

```

```java
public static void main(String [] args)
    {
        DiscountContext dc = new DiscountContext(null);
        double price = 79;
        System.out.println("默认打折：" + dc.getDiscountProce(price));
        dc.changeDiscount(new VipDiscount());
        double prices = 89;
        System.out.println("vip打折：" + dc.getDiscountProce(prices));
    }
```

## 7、门面模式

​		门面模式也被称为正面模式，外观模式，这种模式用于将一组负责的类包装到一个简单的外部接口中。

```java
public class PaymentImpl implements Payment
{
    @Override
    public String pay() {
        String food = "快餐";
        System.out.println("已支付，购买的东西为：" + food);
        return food;
    }
}
```

```java
public class CookImpl implements Cook
{
    @Override
    public String cook(String food) {
        System.out.println("正在烹调：" + food);
        return food;
    }
}

```

```java
public class WaiterImpl implements Waiter
{
    @Override
    public void serve(String food) {
        System.out.println("以上菜：" + food);
    }
}

```

```java
public class Facade
{
    Payment pay;
    Cook cook;
    Waiter waiter;
    public Facade()
    {
        this.pay = new PaymentImpl();
        this.cook = new CookImpl();
        this.waiter = new WaiterImpl();
    }
    public void serveFood()
    {
        String foog = pay.pay();
        foog = cook.cook(foog);
        waiter.serve(foog);
    }
}
```

```java
public class Customer
{
    public void haveDinner()
    {
        Facade f = new Facade();
        f.serveFood();
    }
}
```

## 8、桥接模式

​		桥接模式是一种结构型模式，它主要应对的是：由于实际的需要，某个类具有两个或两个以上的维度变化，如果只是使用继承将无法实现这种需要，或者使得设计变得相当臃肿。

​		做法：将变化部分抽象出来，使变化部分与主类分离开来，从而将多个纬度的变化彻底分离。最后提供一个管理类来组合不同维度上的变化，通过这种组合来满足业务的需要。

```java
public class PepperyStyle implements Peppery
{
    @Override
    public String style() {
        return "重辣";
    }
}
```

```java
public class PlainStyle implements Peppery
{
    @Override
    public String style() {
        return "清淡";
    }
}
```

```java
public abstract class AbstractNoodle
{
    protected Peppery style;
    public AbstractNoodle(Peppery style)
    {
        this.style = style;
    }
    public abstract void eat();
}
```

```java
public class PorkyNoodle extends AbstractNoodle
{
    public PorkyNoodle(Peppery style)
    {
        super(style);
    }

    @Override
    public void eat() {
        System.out.println("稍嫌油腻：" + super.style.style());
    }
}
```

```java
public class BeefMoodle extends AbstractNoodle
{
    public BeefMoodle(Peppery style)
    {
        super(style);
    }

    @Override
    public void eat() {
        System.out.println("牛肉:" + super.style.style());
    }
}
```

```java
public static void main(String [] args)
    {
        AbstractNoodle noodle1 = new BeefMoodle(new PepperyStyle());
        noodle1.eat();
        AbstractNoodle noodle2 = new BeefMoodle(new PlainStyle());
        noodle2.eat();
        AbstractNoodle noodle3 = new PorkyNoodle(new PepperyStyle());
        noodle3.eat();
        AbstractNoodle noodle4 = new PorkyNoodle(new PlainStyle());
        noodle4.eat();
    }
```

## 9、观察者模式

​		观察者模式定义了对象间一对多依赖关系，让一个或多个对象观察一个主题对象。当主题对象的状态发生变化时，系统能通知所有的依赖于此对象的观察者对象，从而使得观察者对象能够自动更新。

​		在观察者模式中，被观察的对象常常也被称为目标或主题，依赖的对象被称为观察者。

```java
public interface Observer
{
    void update(Observable o,Object args);
}
```

```java
public class NameObserver implements Observer
{
    @Override
    public void update(Observable o, Object args) {
        if(args instanceof String)
        {
            String name = (String) args;
            JFrame f = new JFrame("观察者");
            JLabel l = new JLabel("名称改变为：" + name);
            f.add(l);
            f.pack();
            f.setVisible(true);
            System.out.println("名称观察者：" + o + "物品名称已经改变为：" + name);
        }
    }
}
```

```java
public class PriceObserver implements Observer
{
    @Override
    public void update(Observable o, Object args) {
        if(args instanceof Double)
        {
            System.out.println("价格观察者：" + o + "物品价格已经改变为：" + args);
        }
    }
}
```

```java
public class Observable
{
    List<Observer> observers = new ArrayList<>();
    public void registObserver(Observer o)
    {
        observers.add(o);
    }

    public void removeObserver(Observer o)
    {
        observers.remove(o);
    }

    public void notifyObservers(Object value)
    {
        for(Observer o : observers)
        {
            o.update(this,value);
        }
    }
}
```

```java
public class Product extends Observable
{
    private String name;
    private double price;
    public Product(){}
    public Product(String name,double price)
    {
        this.name = name;
        this.price = price;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
        notifyObservers(name);
    }

    public double getPrice()
    {
        return price;
    }

    public void setPrice(double price)
    {
        this.price = price;
        notifyObservers(price);
    }
}
```

```java
Product p = new Product("电视机",176);
        NameObserver no = new NameObserver();
        PriceObserver po = new PriceObserver();
        p.registObserver(no);
        p.registObserver(po);

        p.setName("书桌");
        p.setPrice(345f);
```

![](image/QQ截图20190730210222.png)

# 三、MyBatis

​		MyBatis的核心组件：

​				SqlSessionFactoryBuilder （构造器）：它会根据配置或者代码来生成 。

​				SqISessionFactory ，采用的是分步构建的 Builder 模式。 

​				SqlSessionFactory （工厂接口）：依靠它来生成 Sq!Session 使用的是工厂模式。 

​				SqlSession 会话： 既可以发送 SQL 执行返回结果，也可以获取 Mapper 的接口。在现有的技术		中，一般我们会让其在业务逻辑代码中“消失”，而使用的是My Batis 提供的 SQLMapper 接口编程技术，它能提高代码的可读性和可维护性。

​				SQLMapper （映射器）： MyBatis 新设计存在的组件，它由 Java 接口和 XML文件（或注解）构成，		需要给出对应的 SQL 和映射规则。它负责发送 SQL 去执行，并返回结果。

![](image/QQ截图20190801205307.png)

## 1、SqlSessionFactory（工厂接口）

​		其两个实现类：SqlSessionManager和DefaultSqlSessionFactory。

​		一般，具体是由 DefaultSqlSessionFactory 去实现的，而 SqlSessionManager 使用在多线程的环境中，它的具体实现依靠 DefaultSqlSessionFactory。

![](image/QQ截图20190801205645.png)

​		SqlSessionFactory唯一的作用：生成MyBatis的核心接口对象 SqlSession，往往会采用单例模式处理它。

### 1.1、使用XML构建SqlSessionFactory

​		mybatis-config.xml：

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 定义完成后，可以用user代表User类 -->
    <typeAliases>
        <typeAlias alias="user" type="com.shanji.over.user.entity.User" />
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <!-- 事务管理器 -->
            <transactionManager type="JDBC" />
            
            <!-- 配置数据库  POOLED：代表采用Mybatis内部提供的连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/oa" />
                <property name="username" value="root" />
                <property name="password" value="179980" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 映射文件 -->
    </mappers>
</configuration>
```

​		生成SqlSessionFactory：

```java
	SqlSessionFactory SqlSessionFactory = null ; 
	String resource =”mybatis-config.xml”; 
	InputStream inputStream;
	try
	{
    	inputStream = Resources.getResourceAsStream (resource) ; 
		SqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
	}
	catch(IOException e)
    {
        e.printStackTrace();
    }
```

### 1.2、使用代码创建SqlSessionFactory

```java
//数据库连接池信息
        PooledDataSource dataSource = new PooledDataSource();
        dataSource.setDriver("com.mysql.cj.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("179980");
        dataSource.setUrl("jdbc:mysql://localhost:3306/oa");
        dataSource.setDefaultAutoCommit(false);

        //采用mybatis的jdbc事务方式
        TransactionFactory transactionFactory = new JdbcTransactionFactory();
        Environment environment = new Environment("development",transactionFactory,dataSource);
        
        //创建Configuration对象
        Configuration configuration = new Configuration(environment);
        
        //注册一个mybatis上下文别名
        configuration.getTypeAliasRegistry().registerAlias("user",User.class);
        
        //加入一个映射器
        configuration.addMapper(...);

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

## 2、SqlSession

​		SqlSession为核心接口，其类似于一个JDBC 中的 Connection 对象，代表着一个连接资源启用，两个实现类：

​				DefaultSqlSession：单线程使用。

​				SqlSessionManager：多线程环境下使用。

​		三个作用：

​				1、获取Mapper接口。

​				2、发送SQL给数据库。

​				3、控制数据库事务。

​		创建SqlSession（门面接口，实际工作类为：Executor）：

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```

​		控制事务：

```java
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
        SqlSession sqlSession = null;
        try
        {
            sqlSession = sqlSessionFactory.openSession();

        ...

            sqlSession.commit();
        }
        catch (Exception ex)
        {
            sqlSession.rollback();
        }
        finally 
        {
            if(sqlSession != null)
            {
                sqlSession.close();
            }
        }
```

## 3、映射器

​		映射器是MyBatis中最重要、最复杂的组件，它由一个接口和对应的XML文件（或注解组成）。可以配置的内容：

​				1、描述映射规则。

​				2、提供SQL语句，并可以配置SQL参数类型，返回类型，缓存刷新等信息。

​				3、配置缓存。

​				4、提供动态SQL。

​		作用：将SQL查询到的结果映射为一个POJO，或者将POJO的数据插入到数据库中，并定义一些关于缓存的重要内容。

​		开发只是一个接口，MyBatis运用了动态代理技术使得接口能运行起来。

### 3.1、用XML实现映射器

用XML定义映射器分为两个部分：接口和XML。

```xml
<!-- namespace所对应的是一个接口的全限定名 于是MyBatis上下文就可以通过它找到对应的接口。 -->
<mapper namespace="com.shanji.over.user.mapper.UserMapper">
    
    <!-- 表明是一条查询语句，id标识这条SQL，parameterType说明传递给SQL的参数的类型， resultType表明返回值得类型，(user为在mybatis-config.xml配置的别名)-->
    <select id="getUser" parameterType="String" resultType="user">
        <!-- #{id}代表传递的参数 -->
        select * from user where id=#{id}
    </select>
</mapper>
```

```xml
<mapper resource="com/shanji/over/user/sqlmap/UserMapper.xml" />
```

```java
public class User
{
    private String id;
    private String username;
    private String password;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

```java
public interface UserMapper
{
    public User getUser(String id);
}
```

### 3.2、注解实现映射器

```java
public interface UserMapper
{
    @Select(value = "select * from user where id=#{id}")
    public User getUser(String id);
}
```

```xml
<mapper class="com.shanji.over.user.mapper.UserMapper" />
```

​		如果同时定义注解和XML，XML方式将覆盖注解方式。MyBatis官方推荐XML方式。

### 3.3、SqlSession发送SQL

```java
public class MyBatisUtil
{
    private static SqlSessionFactory sessionFactory = null;
    private static String resource = "mybatis-config.xml";
    private static InputStream inputStream;
    static
    {
        try
        {
            inputStream = Resources.getResourceAsStream (resource) ;
            sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }
        catch (Exception ex)
        {
            ex.printStackTrace();
        }
    }
    public static SqlSession getSqlSession()
    {
        return sessionFactory.openSession();
    }
    public static void close(SqlSession session)
    {
        if(session != null)
        {
            session.close();
        }
    }
}
```

```java
User user = MyBatisUtil.getSqlSession().selectOne("com.shanji.over.user.mapper.UserMapper.getUser", "297ecb206c1e2f9c016c1e2fbdec0000");
```

​		第一个参数为String对象，由一个命名空间加上SQL id组合而成，它完全定位了一条SQL，这样MyBatis就会找到对应的SQL。

### 3.4、用Mapper接口发送SQL

```java
UserMapper mapper = MyBatisUtil.getSqlSession().getMapper(UserMapper.class);
User user = mapper.getUser("297ecb206c1e2f9c016c1e2fbdec0000");
System.out.println(user);
```

​		通过SqlSession的getMapper方法来获取一个Mapper接口，就可以调用它的方法了。因为XML文件或者接口注解定义的SQL都可以通过”类的全限定名+方法名“查找，所以MyBatis会启用对应的SQL进行运行，并返回结果。

### 3.5、对比两种发送SQL方式

​		建议使用SqlSession获取Mapper 的方式：

​				1、使用Mapper接口编程可以消除SqlSession的带来的功能性代码，提高可读性，而SqlSession发送		SQL，需要一个SQL  id去匹配SQL，使用Mapper接口，是完全面向对象的语言，更能体现业务的逻辑。

​				2、使用Mapper方式，会提示错误和检验，而使用SqlSession，只有在运行中才能知道是否会产生错		误。

## 4、生命周期

### 4.1、SqlSessionFactoryBuilder

​		SqlSessionFactoryBuilder的作用在于创建SqlSessionFactory，创建成功后，SqlSessionFactoryBuilder就失去了作用，所以它只能存在于创建SqlSessionFactory的方法中，而不要让其长期存在。

### 4.2、SqlSessionFactory

​		SqlSessionFactory可以被认为是一个数据库连接池，它的作用是创建SqlSession接口对象。因为MyBatis的本质就是Java对数据库的操作，所以SqlSessionFactory的声明周期存在于整个MyBatis的应用之中，所以一旦创建了SqlSessionFactory，就要长期保存它，直至不再使用MyBatis应用，所以可以认为SqlSessionFactory的声明周期就等同于MyBatis的应用周期。

### 4.3、SqlSession

​		SqlSession相当于一个数据库连接，可以在一个事务里面执行多条SQL，然后通过它的commit、rollback等方法，提交或者回滚事务。所以其应该存活在一个业务请求中，处理完这个请求后，应该关闭这条连接，让它归还给SqlSessionFactory，否则数据库资源就很快被耗费精光，系统就会瘫痪，所以用try...catch...finally语句来保证其正确关闭。

### 4.4、Mapper

​		Mapper是一个接口，它由SqlSession所创建，所以它的最大声明周期至多和SqlSession保持一致。Mapper代表的是一个请求中的业务处理，所以它应该在一个请求中，一旦处理完了相关的业务，就应该废弃它。



![](image/QQ截图20190818124214.png)

![](image/QQ截图20190821095214.png)

​		MybBatis配置项的顺序不能颠倒，如果颠倒了它们的顺序，那么在MyBatis启动阶段就会发生异常，导致程序无法运行。

## 5、properties属性

### 5.1、property子元素

```xml
    <properties>
        <property name="database.driver" value="com.mysql.cj.jdbc.Driver" />
        <property name="database.url" value="jdbc:mysql://localhost:3306/ssm?useUnicode=true&amp;characterEncoding=UTF-8" />
        <property name="database.username" value="root" />
        <property name="database.password" value="179980" />
    </properties>
    <typeAliases>
        <typeAlias alias="user" type="com.shanji.over.user.entity.User" />
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="${database.driver}" />
                <property name="url" value="${database.url}" />
                <property name="username" value="${database.username}" />
                <property name="password" value="${database.password}" />
            </dataSource>
        </environment>
    </environments>
```

### 5.2、使用properties文件

```properties
database.driver=com.mysql.cj.jdbc.Driver
database.url=jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=UTF-8
database.username=root
database.password=179980
```

```xml
    <properties resource="database.properties" /> <!-- 引入配置文件 -->
    <typeAliases>
        <typeAlias alias="user" type="com.shanji.over.user.entity.User" />
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="${database.driver}" />
                <property name="url" value="${database.url}" />
                <property name="username" value="${database.username}" />
                <property name="password" value="${database.password}" />
            </dataSource>
        </environment>
    </environments>
```

## 6、setting设置

​		配置项

![](image/QQ截图20190821103402.png)

![](image/QQ截图20190821103435.png)

## 7、typeAliases别名

​		由于类的全限定名称很长，需要大量使用的时候，不方便。在MyBatis中允许定义一个简写来代表这个类，这就是别名。分为系统定义别名和自定义别名。在MyBatis中别名由类TypeAliasRegistry去定义。**别名不区分大小写。**

### 7.1、系统定义别名

![](image/QQ截图20190822103226.png)

![](image/QQ截图20190822103301.png)

```java
public TypeAliasRegistry() {
        this.registerAlias("string", String.class);
        this.registerAlias("byte", Byte.class);
        this.registerAlias("long", Long.class);
        this.registerAlias("short", Short.class);
        this.registerAlias("int", Integer.class);
        this.registerAlias("integer", Integer.class);
        this.registerAlias("double", Double.class);
        this.registerAlias("float", Float.class);
        this.registerAlias("boolean", Boolean.class);
        this.registerAlias("byte[]", Byte[].class);
        this.registerAlias("long[]", Long[].class);
        this.registerAlias("short[]", Short[].class);
        this.registerAlias("int[]", Integer[].class);
        this.registerAlias("integer[]", Integer[].class);
        this.registerAlias("double[]", Double[].class);
        this.registerAlias("float[]", Float[].class);
        this.registerAlias("boolean[]", Boolean[].class);
        this.registerAlias("_byte", Byte.TYPE);
        this.registerAlias("_long", Long.TYPE);
        this.registerAlias("_short", Short.TYPE);
        this.registerAlias("_int", Integer.TYPE);
        this.registerAlias("_integer", Integer.TYPE);
        this.registerAlias("_double", Double.TYPE);
        this.registerAlias("_float", Float.TYPE);
        this.registerAlias("_boolean", Boolean.TYPE);
        this.registerAlias("_byte[]", byte[].class);
        this.registerAlias("_long[]", long[].class);
        this.registerAlias("_short[]", short[].class);
        this.registerAlias("_int[]", int[].class);
        this.registerAlias("_integer[]", int[].class);
        this.registerAlias("_double[]", double[].class);
        this.registerAlias("_float[]", float[].class);
        this.registerAlias("_boolean[]", boolean[].class);
        this.registerAlias("date", Date.class);
        this.registerAlias("decimal", BigDecimal.class);
        this.registerAlias("bigdecimal", BigDecimal.class);
        this.registerAlias("biginteger", BigInteger.class);
        this.registerAlias("object", Object.class);
        this.registerAlias("date[]", Date[].class);
        this.registerAlias("decimal[]", BigDecimal[].class);
        this.registerAlias("bigdecimal[]", BigDecimal[].class);
        this.registerAlias("biginteger[]", BigInteger[].class);
        this.registerAlias("object[]", Object[].class);
        this.registerAlias("map", Map.class);
        this.registerAlias("hashmap", HashMap.class);
        this.registerAlias("list", List.class);
        this.registerAlias("arraylist", ArrayList.class);
        this.registerAlias("collection", Collection.class);
        this.registerAlias("iterator", Iterator.class);
        this.registerAlias("ResultSet", ResultSet.class);
    }
```



​		如果需要使用对应类型的数组型，要看其是否能支持数组，如果支持只需要在别名后加"[]"即可。

​		通过代码注册时，首先通过Configuration获取TypeAliasRegistry类对象，其中有一个getTypeAliasRegistry方法可以获得别名，然后就可以通过registryAlias方法对别名注册了。

​		Configuration对象也对一些常用的配置项配置了别名：

![](image/QQ截图20190822103828.png)

```java
public Configuration() {
    	//事务方式别名
        this.typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
        this.typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
    	//数据源类型别名
        this.typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
        this.typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
        this.typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
    	//缓存策略别名
        this.typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
        this.typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
        this.typeAliasRegistry.registerAlias("LRU", LruCache.class);
        this.typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
        this.typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
    	//数据库标识别名
        this.typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
    	//语言驱动类别名
        this.typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
        this.typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);
    	//日志类别名
        this.typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
        this.typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
        this.typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
        this.typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
        this.typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
        this.typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
        this.typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
    	//动态代理别名
        this.typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
        this.typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);
    }
```

### 7.2、自定义别名

​		可以通过TypeAliasRegistry类的registryAlias方法注册，也可以采用配置文件或者扫描方式来定义它。

​		配置文件：

​				为某一个些类配置别名：

```xml
<typeAliases>
        <typeAlias alias="user" type="com.shanji.over.user.entity.User" />
</typeAliases>
```

​				扫描别名：

```xml
<typeAliases>
        <package name="com.shanji.over" />
    </typeAliases>
```

​		用扫描的方式，当扫描到类时，将第一个字母小写其余不变作为别名，出现不同包下命名相同的类时，扫描会报错，此时可以通过@Alias注解来修饰类来区分。

## 8、typeHandler类型转换器

​		在JDBC中，需要在PreparedStatement对象中设置那些已经预编译过的SQL语句的参数，执行SQL后，会通过ResultSet对象获取到数据库的数据。而这些MyBatis是通过typeHandler是来实现的。在typeHandler中，分为jdbcType和javaType，其中jdbcType用于定义数据库类型，而javaType用于定义Java类型，那么typeHandler的作用就是承担jdbcType和javaType之间的相互转换。

![](image/QQ截图20190822111120.png)

### 8.1、系统定义的typeHandler

![](image/QQ截图20190822111425.png)

![](image/QQ截图20190822111446.png)

​		所有的typeHandler都要实现接口TypeHandler。

```java
public interface TypeHandler<T> {
    void setParameter(PreparedStatement var1, int var2, T var3, JdbcType var4) throws SQLException;

    T getResult(ResultSet var1, String var2) throws SQLException;

    T getResult(ResultSet var1, int var2) throws SQLException;

    T getResult(CallableStatement var1, int var2) throws SQLException;
}
```

​		T：泛型，专指javaType。

​		setParameter：typeHandler通过PreparedStatement对象进行设置SQL参数的时候使用的具体方法，i是参数在SQL的下标，var3是参数，JdbcType是数据库类型。

​		其中三个getResult的方法，作用是从JDBC结果集中获取数据进行转换，要么使用列名，要么使用下标获取数据库的数据，其中最后一个getResult方法是存储过程专用的。

​		typeHandler的所有实现类都继承了BaseTypeHandler。

```java
public abstract class BaseTypeHandler<T> extends TypeReference<T> implements TypeHandler<T> {
    protected Configuration configuration;

    public BaseTypeHandler() {
    }

    public void setConfiguration(Configuration c) {
        this.configuration = c;
    }

    public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        if (parameter == null) {
            if (jdbcType == null) {
                throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
            }

            try {
                ps.setNull(i, jdbcType.TYPE_CODE);
            } catch (SQLException var7) {
                throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. Cause: " + var7, var7);
            }
        } else {
            try {
                this.setNonNullParameter(ps, i, parameter, jdbcType);
            } catch (Exception var6) {
                throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . Try setting a different JdbcType for this parameter or a different configuration property. Cause: " + var6, var6);
            }
        }

    }

    public T getResult(ResultSet rs, String columnName) throws SQLException {
        Object result;
        try {
            result = this.getNullableResult(rs, columnName);
        } catch (Exception var5) {
            throw new ResultMapException("Error attempting to get column '" + columnName + "' from result set.  Cause: " + var5, var5);
        }

        return rs.wasNull() ? null : result;
    }

    public T getResult(ResultSet rs, int columnIndex) throws SQLException {
        Object result;
        try {
            result = this.getNullableResult(rs, columnIndex);
        } catch (Exception var5) {
            throw new ResultMapException("Error attempting to get column #" + columnIndex + " from result set.  Cause: " + var5, var5);
        }

        return rs.wasNull() ? null : result;
    }

    public T getResult(CallableStatement cs, int columnIndex) throws SQLException {
        Object result;
        try {
            result = this.getNullableResult(cs, columnIndex);
        } catch (Exception var5) {
            throw new ResultMapException("Error attempting to get column #" + columnIndex + " from callable statement.  Cause: " + var5, var5);
        }

        return cs.wasNull() ? null : result;
    }

    public abstract void setNonNullParameter(PreparedStatement var1, int var2, T var3, JdbcType var4) throws SQLException;

    public abstract T getNullableResult(ResultSet var1, String var2) throws SQLException;

    public abstract T getNullableResult(ResultSet var1, int var2) throws SQLException;

    public abstract T getNullableResult(CallableStatement var1, int var2) throws SQLException;
}
```

​		BaseTypeHandler：抽象类，本身实现了typeHandler接口的4个方法，但是需要子类去实现其定义的4个抽闲方法。

​		getResult方法：非空结果集通过getNullableResult方法获取的。如果判断为空，则返回null。

​		setParameter方法：当参数parameter和jdbcType同时为空时，抛出异常。能明确jdbcType，则会进行空设置，参数不为空，将采用setNonNullParameter方法设置参数。

​		getNullableResult方法用于存储过程。

​		StringTypeHandler：用于字符串转换

```java
public class StringTypeHandler extends BaseTypeHandler<String> {
    public StringTypeHandler() {
    }

    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter);
    }

    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getString(columnName);
    }

    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex);
    }

    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex);
    }
}
```

​		javaType和jdbcType相互转换的注册。

```java
public TypeHandlerRegistry() {
        this.register((Class)Boolean.class, (TypeHandler)(new BooleanTypeHandler()));
        this.register((Class)Boolean.TYPE, (TypeHandler)(new BooleanTypeHandler()));
        this.register((JdbcType)JdbcType.BOOLEAN, (TypeHandler)(new BooleanTypeHandler()));
        this.register((JdbcType)JdbcType.BIT, (TypeHandler)(new BooleanTypeHandler()));
        this.register((Class)Byte.class, (TypeHandler)(new ByteTypeHandler()));
        this.register((Class)Byte.TYPE, (TypeHandler)(new ByteTypeHandler()));
        this.register((JdbcType)JdbcType.TINYINT, (TypeHandler)(new ByteTypeHandler()));
        this.register((Class)Short.class, (TypeHandler)(new ShortTypeHandler()));
        this.register((Class)Short.TYPE, (TypeHandler)(new ShortTypeHandler()));
        this.register((JdbcType)JdbcType.SMALLINT, (TypeHandler)(new ShortTypeHandler()));
        this.register((Class)Integer.class, (TypeHandler)(new IntegerTypeHandler()));
        this.register((Class)Integer.TYPE, (TypeHandler)(new IntegerTypeHandler()));
        this.register((JdbcType)JdbcType.INTEGER, (TypeHandler)(new IntegerTypeHandler()));
        this.register((Class)Long.class, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)Long.TYPE, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)Float.class, (TypeHandler)(new FloatTypeHandler()));
        this.register((Class)Float.TYPE, (TypeHandler)(new FloatTypeHandler()));
        this.register((JdbcType)JdbcType.FLOAT, (TypeHandler)(new FloatTypeHandler()));
        this.register((Class)Double.class, (TypeHandler)(new DoubleTypeHandler()));
        this.register((Class)Double.TYPE, (TypeHandler)(new DoubleTypeHandler()));
        this.register((JdbcType)JdbcType.DOUBLE, (TypeHandler)(new DoubleTypeHandler()));
        this.register((Class)Reader.class, (TypeHandler)(new ClobReaderTypeHandler()));
        this.register((Class)String.class, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.CHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.CLOB, (TypeHandler)(new ClobTypeHandler()));
        this.register((Class)String.class, JdbcType.VARCHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((Class)String.class, JdbcType.LONGVARCHAR, (TypeHandler)(new ClobTypeHandler()));
        this.register((Class)String.class, JdbcType.NVARCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((Class)String.class, JdbcType.NCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((Class)String.class, JdbcType.NCLOB, (TypeHandler)(new NClobTypeHandler()));
        this.register((JdbcType)JdbcType.CHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((JdbcType)JdbcType.VARCHAR, (TypeHandler)(new StringTypeHandler()));
        this.register((JdbcType)JdbcType.CLOB, (TypeHandler)(new ClobTypeHandler()));
        this.register((JdbcType)JdbcType.LONGVARCHAR, (TypeHandler)(new ClobTypeHandler()));
        this.register((JdbcType)JdbcType.NVARCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((JdbcType)JdbcType.NCHAR, (TypeHandler)(new NStringTypeHandler()));
        this.register((JdbcType)JdbcType.NCLOB, (TypeHandler)(new NClobTypeHandler()));
        this.register((Class)Object.class, JdbcType.ARRAY, (TypeHandler)(new ArrayTypeHandler()));
        this.register((JdbcType)JdbcType.ARRAY, (TypeHandler)(new ArrayTypeHandler()));
        this.register((Class)BigInteger.class, (TypeHandler)(new BigIntegerTypeHandler()));
        this.register((JdbcType)JdbcType.BIGINT, (TypeHandler)(new LongTypeHandler()));
        this.register((Class)BigDecimal.class, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.REAL, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.DECIMAL, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((JdbcType)JdbcType.NUMERIC, (TypeHandler)(new BigDecimalTypeHandler()));
        this.register((Class)InputStream.class, (TypeHandler)(new BlobInputStreamTypeHandler()));
        this.register((Class)Byte[].class, (TypeHandler)(new ByteObjectArrayTypeHandler()));
        this.register((Class)Byte[].class, JdbcType.BLOB, (TypeHandler)(new BlobByteObjectArrayTypeHandler()));
        this.register((Class)Byte[].class, JdbcType.LONGVARBINARY, (TypeHandler)(new BlobByteObjectArrayTypeHandler()));
        this.register((Class)byte[].class, (TypeHandler)(new ByteArrayTypeHandler()));
        this.register((Class)byte[].class, JdbcType.BLOB, (TypeHandler)(new BlobTypeHandler()));
        this.register((Class)byte[].class, JdbcType.LONGVARBINARY, (TypeHandler)(new BlobTypeHandler()));
        this.register((JdbcType)JdbcType.LONGVARBINARY, (TypeHandler)(new BlobTypeHandler()));
        this.register((JdbcType)JdbcType.BLOB, (TypeHandler)(new BlobTypeHandler()));
        this.register(Object.class, this.UNKNOWN_TYPE_HANDLER);
        this.register(Object.class, JdbcType.OTHER, this.UNKNOWN_TYPE_HANDLER);
        this.register(JdbcType.OTHER, this.UNKNOWN_TYPE_HANDLER);
        this.register((Class)Date.class, (TypeHandler)(new DateTypeHandler()));
        this.register((Class)Date.class, JdbcType.DATE, (TypeHandler)(new DateOnlyTypeHandler()));
        this.register((Class)Date.class, JdbcType.TIME, (TypeHandler)(new TimeOnlyTypeHandler()));
        this.register((JdbcType)JdbcType.TIMESTAMP, (TypeHandler)(new DateTypeHandler()));
        this.register((JdbcType)JdbcType.DATE, (TypeHandler)(new DateOnlyTypeHandler()));
        this.register((JdbcType)JdbcType.TIME, (TypeHandler)(new TimeOnlyTypeHandler()));
        this.register((Class)java.sql.Date.class, (TypeHandler)(new SqlDateTypeHandler()));
        this.register((Class)Time.class, (TypeHandler)(new SqlTimeTypeHandler()));
        this.register((Class)Timestamp.class, (TypeHandler)(new SqlTimestampTypeHandler()));
        if (Jdk.dateAndTimeApiExists) {
            this.register(Instant.class, InstantTypeHandler.class);
            this.register(LocalDateTime.class, LocalDateTimeTypeHandler.class);
            this.register(LocalDate.class, LocalDateTypeHandler.class);
            this.register(LocalTime.class, LocalTimeTypeHandler.class);
            this.register(OffsetDateTime.class, OffsetDateTimeTypeHandler.class);
            this.register(OffsetTime.class, OffsetTimeTypeHandler.class);
            this.register(ZonedDateTime.class, ZonedDateTimeTypeHandler.class);
            this.register(Month.class, MonthTypeHandler.class);
            this.register(Year.class, YearTypeHandler.class);
            this.register(YearMonth.class, YearMonthTypeHandler.class);
            this.register(JapaneseDate.class, JapaneseDateTypeHandler.class);
        }

        this.register((Class)Character.class, (TypeHandler)(new CharacterTypeHandler()));
        this.register((Class)Character.TYPE, (TypeHandler)(new CharacterTypeHandler()));
    }
```

### 8.2、自定义typeHandler

​		自定义的typeHandler一般不会使用代码注册，而是通过配置或扫描。

​		自定义的typeHandler需要实现TypeHandler接口，或者BaseTypeHandler。然后在配置文件中配置自定义的typeHandler。

```java
public class StyleTypeHandler implements TypeHandler<String>
{

    @Override
    public void setParameter(PreparedStatement preparedStatement, int i, String s, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i,s);
    }

    @Override
    public String getResult(ResultSet resultSet, String s) throws SQLException {
        return resultSet.getString(s);
    }

    @Override
    public String getResult(ResultSet resultSet, int i) throws SQLException {
        return resultSet.getString(i);
    }

    @Override
    public String getResult(CallableStatement callableStatement, int i) throws SQLException {
        return callableStatement.getString(i);
    }
}
```

```xml
<typeHandlers>
        <typeHandler handler="com.shanji.over.typehandler.StyleTypeHandler" jdbcType="VARCHAR" javaType="string" />
    
    <!--
		包扫描
		此时就没法指定jdbcType和javaType，可以使用注解来修饰类
		@MappedTypes(String.class)
		@MappedJdbcTypes(JdbcType.VARCHAR)
 		
		<package name="com.shanji.over.typehandler" />
	-->
    </typeHandlers>
```

​		配置完成后，当jdbcType和javaType能与其对应的时候，就会启动，还可以显示启用。

```xml
<resultMap id="userMapper" type="com.shanji.over.example.Role">
        <result property="id" column="id" />
        <result property="roleName" column="role_name" jdbcType="VARCHAR" javaType="string" />
        <result property="note" column="note" typeHandler="com.shanji.over.typehandler.StyleTypeHandler" />
    </resultMap>

    <select id="findRoles" parameterType="string" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where role_name like concat('%',#{roleName,jdbcType=VARCHAR,javaType=string},'%')
    </select>
    <select id="findRoless" parameterType="string" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where note like concat('%',#{note,typeHandler=com.shanji.over.typehandler.StyleTypeHandler},'%')
    </select>
```

​		要么指定了与自定义typeHandler一致的jdbcType和javaType，要么直接使用typeHandler指定具体的实现类，在一些因为数据库返回为空导致无法断定采用哪个typeHandler来处理，而又没有注册对应的javaType的typeHandler时，MyBatis无法知道使用哪个typeHandler转换数据。

### 8.3、枚举typeHandler

​		大多数情况，typeHandler因为枚举而使用，两个作为枚举类型支持的类：

​				EnumOrdinalTypeHandler。

​				EnumTypeHandler。

#### 8.3.1、EnumOrdinalTypeHandler

​		根据枚举数组下标索引的方式进行匹配，也是枚举类型的默认转换类，要求数据库返回一个证书作为其下标，会根据下标找到读音的枚举类型。

```java
public enum SexEnum
{
    //按id排序，否则会出错
    FEMALE(0,"女"),
    MALE(1,"男");
    private int id;
    private String name;
    /* getter and setter */
}
```

```java
public class User
{
    private Integer id;
    private String userName;
    private String password;
    private SexEnum sex;
    private String mobile;
    private String tel;
    private String email;
    private String note;
    /* getter and setter  */
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.shanji.over.typehandler.UserMappler">
    <resultMap id="userMapper" type="com.shanji.over.typehandler.User">
        <result property="id" column="id" />
        <result property="userName" column="user_name" />
        <result property="password" column="password" />
        <!-- 若是自定义typehandler，此处只对查询起作用 -->
        <result property="sex" column="sex" 
                typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler" />
        <result property="mobile" column="mobile" />
        <result property="tel" column="tel" />
        <result property="email" column="email" />
        <result property="note" column="note" />
    </resultMap>

    <select id="getUser" resultMap="userMapper" parameterType="int">
        select * from user where id=#{id}
    </select>
    <insert id="insertUser" parameterType="com.shanji.over.typehandler.User">
        insert into user(id,user_name,password,sex,mobile,tel,email,note) values (#{id},#{userName},#{password},
        #{sex} <!--自定义： #{sex,typeHandler=com.shanji.over.typehandler.SexEnumTypeHandler} -->
        ,#{mobile},#{tel},#{email},#{note})
    </insert>
</mapper>
```

```xml
<typeHandlers>
        <!-- 若不在mybatis-config文件中加入此配置，那么存入数据库的是枚举值中的字符串而不是对应的序号
  			如果使用EnumTypeHandler，不能有这个配置。自定义也不需要有此配置
-->
        <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="com.shanji.over.typehandler.SexEnum" />
    </typeHandlers>
```

#### 8.3.2、EnumTypeHandler

​		将使用的名称转化为对应的枚举。

```xml
<result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumTypeHandler" />
```

### 8.4、自定义typeHandler

​		以上个实例，将sex=0为男性，sex=1为女性

```java
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(value =SexEnum.class)
public class SexEnumTypeHandler extends BaseTypeHandler<SexEnum>
{
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, SexEnum sexEnum, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i,String.valueOf(sexEnum.getId()));
    }

    @Override
    public SexEnum getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return SexEnum.getSexById(Integer.parseInt(resultSet.getString(s)));
    }

    @Override
    public SexEnum getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return SexEnum.getSexById(Integer.parseInt(resultSet.getString(i)));
    }

    @Override
    public SexEnum getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return SexEnum.getSexById(Integer.parseInt(callableStatement.getString(i)));
    }
}
```

```xml
<mapper namespace="com.shanji.over.typehandler.UserMappler">
    <resultMap id="userMapper" type="com.shanji.over.typehandler.User">
        <result property="id" column="id" />
        <result property="userName" column="user_name" />
        <result property="password" column="password" />
        <!-- 此处只对查询有用 -->
        <result property="sex" column="sex"
                typeHandler="com.shanji.over.typehandler.SexEnumTypeHandler" />
        <result property="mobile" column="mobile" />
        <result property="tel" column="tel" />
        <result property="email" column="email" />
        <result property="note" column="note" />
    </resultMap>

    <select id="getUser" resultMap="userMapper" parameterType="int">
        select * from user where id=#{id}
    </select>
    <insert id="insertUser" parameterType="com.shanji.over.typehandler.User">
        insert into user(id,user_name,password,sex,mobile,tel,email,note) values (#{id},#{userName},#{password},#{sex,typeHandler=com.shanji.over.typehandler.SexEnumTypeHandler},#{mobile},#{tel},#{email},#{note})
    </insert>
</mapper>


或者

<!-- 此时上面的配置文件中的typeHandler都可以省略 -->
<typeHandlers>
        <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="com.shanji.over.typehandler.SexEnum" />
</typeHandlers>
```

### 8.5、文件操作

​		MyBatis对数据库的Blob字段也进行了支持，其提供了BlobTypeHandler，ByteArrayTypeHandler，后者不太常用。

## 9、对象工厂

​		当创建结果集，MyBatis会使用一个对象工厂来完成创建这个结果集实例，在默认的情况下，MyBatis会使用其定义的对象工厂——DefaultObjectFactory来完成相应的工作。

​		如果自定义，需要实现接口ObjectFactory，并配置。大多数情况下，会优先考虑继承已经实现好的DefaultObjectFactory，并通过改写来完成所需要的工作。

## 10、运行环境

​		运行环境主要的作用是配置数据库信息，可以配置多个数据库，但一般只需要配置其中的一个就可以了。其下面两个可配置的元素，事务管理器，数据源。

```xml
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="${database.driver}" />
                <property name="url" value="${database.url}" />
                <property name="username" value="${database.username}" />
                <property name="password" value="${database.password}" />
            </dataSource>
        </environment>
    </environments>
```

### 10.1、transactionManager(事务管理器)

​		transactionManager提供了两个实现类，需要实现接口Transaction。

```java
//主要工作：提交，回滚，关闭数据库的事务
public interface Transaction {
    Connection getConnection() throws SQLException;

    void commit() throws SQLException;

    void rollback() throws SQLException;

    void close() throws SQLException;

    Integer getTimeout() throws SQLException;
}
```

​		两个实现类：JdbcTransaction和ManagedTransaction。

![](image/QQ截图20190824085941.png)

​		两个实现类，对应两个工厂，JdbcTransactionFactory和MandgedTransactionFactory，工厂需要实现TransactionFactory接口。通过它们会生成对应的Transaction对象。

```xml
<!-- 两种事务管理器的配置 -->
<transactionManager type="JDBC" />
	<!-- 使用JdbcTransactionFactory生成的JdbcTransaction对象实现，以JDBC的方式对数据库的提交和回滚进行操作 -->
	
<!-- <transactionManager type="MANAGED" /> -->
	<!-- 使用MandgedTransactionFactory生成的ManagedTransaction对象实现。提交和回滚方法不同任何操作，而是把事务交给容器处理，默认情况下，会关闭连接如果不需要关闭，可将closeConnection属性设置为false来阻止其默认的关闭行为 -->

```

​		自定义：

```java
public class StyleTransactionFactory implements TransactionFactory
{
    @Override
    public void setProperties(Properties properties) {

    }

    @Override
    public Transaction newTransaction(Connection connection) {
        return new  StyleTransaction(connection);
    }

    @Override
    public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel transactionIsolationLevel, boolean b) {
        return new StyleTransaction(dataSource,transactionIsolationLevel,b);
    }
}
```

```java
public class StyleTransaction extends JdbcTransaction implements Transaction
{
    public StyleTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
        super(ds, desiredLevel, desiredAutoCommit);
    }

    public StyleTransaction(Connection connection) {
        super(connection);
    }
}
```

```xml
<transactionManager type="com.shanji.over.transaction.StyleTransactionFactory" />
```

### 10.2、environment数据源环境

​		environment主要的作为是配置数据库，数据库通过PooledDataSourceFactory，UnpooledDataSourceFactory和JndiDataSourceFactory三个工厂类来提供，前两者对应产生PooledDataSource，UnpooledDataSource类对象，而JndiDataSourceFactory则会根据JNDI的信息拿到外部容器实现的数据库连接对象，最后三个工厂类都会生成一个实现了DataSource接口的数据库连接对象。

```xml
<dataSource type="POOLED"></dataSource>
<dataSource type="UNPOOLED"></dataSource>
<dataSource type="JNDI"></dataSource>
```

#### 1、UNPOOLED

​		UNPOOLED采用非数据库池的管理方式，每次请求都会打开一个新的数据库连接，所以创建会比较慢。

​		可配置属性：

```properties
driver:数据库驱动名
url:连接数据库的URL
username:用户名
password:密码
defaultTransactionIsolationLevel:默认的连接事务隔离级别
```

​		传递属性给数据库驱动也是一个可选项，注意属性的前缀是"driver."。如driver.encoding=UTF-8。其会通过DriverManager.getConnection(url,driverProperties)方法传递值为UTF-8的encoding属性给数据库驱动。

#### 2、POOLED

​		数据源POOLED利用池的概念将JDBC的Connection对象组织起来，开始会有一些空置，并且已经连接好的数据库连接，所以请求时，无须再建立和验证，省去了创建新的连接实例时所必需的初始化和认证时间。其还可以控制最大连接数，避免过多的连接导致系统瓶颈。

​		可配置属性：

```properties
driver:数据库驱动名
url:连接数据库的URL
username:用户名
password:密码
defaultTransactionIsolationLevel:默认的连接事务隔离级别
poolMaximumActiveConnections:是在任意时间都存在的活动（也就是正在使用）连接数量，默认值为10
poolMaximumIdleConnections:是任意时间可能存在的空闲连接数
poolMaximumCheckoutTime:在被强制返回之前，池中连接被检出的时间，默认为20000，即20秒
poolTimeToWait:一个底层设置，如果获取连接花费相当长的时间，会给连接池打印状态日志，并重新尝试获取一个连接（避免在误配置的情况下一直失败），默认为20000，即20秒
poolPingQuery:为发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中，并准备接收请求。默认是"NO PING QUERY SET",这会导致多数数据库驱动失败时带有一个恰当的错误消息
poolPingEnabled:是否启动侦测查询，若开启，也必须使用一个可执行的SQL语句设置poolPingQuery属性，默认为false
poolPingConnectionsNotUsedFor:配置poolPingQuery的使用频度，可以被设置成匹配具体的数据库连接超时时间，来比吗inn不必要的侦测，默认为0
```

#### 3、JNDI

​		JNDI的实现是为了能在如EJB或应用服务器这类容器中使用，容器可以集中或在外部配置数据源。然后放置一个JNDI上下文的引用，只需配置两个属性：

```properties
initial_ context:用来在InitialContext中寻找上下文。可选属性，如果忽略，那么data_source属性将会直接从InitialContext中寻找
data_source:引用数据库实例位置上下文的路径，当提供initial_ context配置时，data_source会在其返回的上下文中进行查找，没有提供initial_ context时，data_source直接在InitialContext中查找。
```

​		与其他数据源配置类似，其可以通过添加前缀"env."直接把属性传递给初始上下文。

​		MyBatis也支持第三方数据源，此时需要提供一个自定义的DataSourceFactory。并配置它。

```java
public class StyleDataSourceFactory implements DataSourceFactory
{
    private Properties properties = null;

    @Override
    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    @Override
    public DataSource getDataSource() {
        DataSource dataSource = null;
        try
        {
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        }
        catch (Exception ex)
        {
            ex.printStackTrace();
        }
        return dataSource;
    }
}
```

```xml
<dataSource type="com.shanji.over.datasource.StyleDataSourceFactory">
                <property name="driver" value="${database.driver}" />
                <property name="url" value="${database.url}" />
                <property name="username" value="${database.username}" />
                <property name="password" value="${database.password}" />
</dataSource>
```

## 11、databaseIdProvider数据库厂商标识

### 11.1、使用系统默认的databaseIdProvider

​		使用databaseIdProvider要配置的属性：

```xml
<databaseIdProvider type="DB_VENDOR">
    	<!-- name是数据库的名称，可以使用JDBC创建其数据库连接对象Connection，然后通过connection.getMetaData().getDatabaseProductName()获取，value为别名 -->
        <property name="Oracle" value="oracle" />
        <property name="MySQL" value="mysql" />
        <property name="DB2" value="db2" />
</databaseIdProvider>
```

​		配置后，就可以在 xxxMappler.xml切换数据库类型了。

```xml
<!-- mysql -->
<select id="getUser" parameterType="String" resultType="user" databaseId="mysql">
        select * from user where id=#{id}
</select>

<!-- oracle -->
<select id="getUser" parameterType="String" resultType="user" databaseId="oracle">
        select * from user where id=#{id}
</select>
```

​		如果同时存在一条有databaseId和没有databaseId的标识，当databaseId属性被配置时，系统会优先取得和数据库配置一致的SQL，如果没有，则取没有databaseId的SQL，可以把它当作默认值，如果还是取不到，则报错。

### 11.2、不使用系统规则

​		使用自定义的规则，需要实现DatabaseIdProvider接口。

```java
public class StyleDatabaseIdProvider implements DatabaseIdProvider
{
    private static final String DATABASE_TYPE_DB2 = "DB2";
    private static final String DATABASE_TYPE_MYSQL = "MySQL";
    private static final String DATABASE_TYPE_ORACLE = "Oracle";

    Logger logger = LogManager.getLogger(StyleDatabaseIdProvider.class);




    @Override
    public void setProperties(Properties properties) {
        logger.info(properties);
    }

    @Override
    public String getDatabaseId(DataSource dataSource) throws SQLException {
        Connection connection = dataSource.getConnection();
        String dbbProductName = connection.getMetaData().getDatabaseProductName();
        if(StyleDatabaseIdProvider.DATABASE_TYPE_DB2.equals(dbbProductName))
        {
            return "db2";
        }
        else if(StyleDatabaseIdProvider.DATABASE_TYPE_MYSQL.equals(dbbProductName))
        {
            return "mysql";
        }
        else if(StyleDatabaseIdProvider.DATABASE_TYPE_ORACLE.equals(dbbProductName))
        {
            return "oracle";
        }
        else
        {
            return  null;
        }
    }
}
```

```xml
<databaseIdProvider type="com.shanji.over.databaseidprovider.StyleDatabaseIdProvider">
	<property name="..." value="..." />
</databaseIdProvider>
```

## 12、引入映射器

​		映射器定义命名空间的方法，命名空间对应的是一个接口的全路径，而不是实现类。

​		首先定义接口：

```java
public interface RoleMapper
{
    int insertRole(Role role);
    int deleteRole(Role role);
    int updateRole(Role role);
    Role getRole(Integer id);
    List<Role> findRoles(String roleName);
}
```

​		其次，定义xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace对应接口的全路径 -->
<mapper namespace="com.shanji.over.example.RoleMapper">
    <insert id="insertRole" parameterType="com.shanji.over.example.Role">
        insert into role(role_name,note) values (#{roleName},#{note})
    </insert>

    <delete id="deleteRole" parameterType="int">
        delete from role where id=#{id}
    </delete>

    <update id="updateRole" parameterType="com.shanji.over.example.Role">
        update role set role_name=#{roleName},note=#{note} where id=#{id}
    </update>

    <select id="getRole" parameterType="int" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where id=#{id}
    </select>


    <resultMap id="userMapper" type="com.shanji.over.example.Role">
        <result property="id" column="id" />
        <result property="roleName" column="role_name" jdbcType="VARCHAR" javaType="string" />
        <result property="note" column="note" typeHandler="com.shanji.over.typehandler.StyleTypeHandler" />
    </resultMap>

    <select id="findRoles" parameterType="string" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where role_name like concat('%',#{roleName,jdbcType=VARCHAR,javaType=string},'%')
    </select>
    <select id="findRoless" parameterType="string" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where note like concat('%',#{note,typeHandler=com.shanji.over.typehandler.StyleTypeHandler},'%')
    </select>
</mapper>
```

​		引入：

### 1、用文件路径引入

```xml
<mappers>
        <!-- 映射文件 -->
        <mapper resource="com/shanji/over/example/RoleMapper.xml" />

        <mapper resource="com/shanji/over/typehandler/UserMapper.xml" />

    </mappers>
```

### 2、包名引入

```xml
<package name="com.shanji.over" />
```

### 3、类注册引入

```xml
<mapper class="com.shanji.over.example.RoleMapper" />
```

### 4、userMappler.xml引入

```xml
<mapper url="file://..." />
```

## 13、select元素——查询语句

![](image/QQ截图20190826092422.png)





​		select元素代表SQL的select语句，用于查询。

​		配置：

![](image/QQ截图20190826092705.png)

![](image/QQ截图20190826092730.png)

### 13.1、简单的select元素的应用

```xml

<!--
id:与Mapper的全限定名，联合称为一个唯一的标识，用于标识这条SQL
parameterType：表示这条SQL接受的参数类型，可以是MyBatis系统定义或者自定的别名，也可以是类的全限定名
resultType：表示这条SQL返回的结果类型，与parameterType一样，可以是MyBatis系统定义或者自定的别名，也可以是类的全限定名
#{roleName}：被传递进去的参数
-->
<select id="roles" parameterType="string" resultType="int">
        select count(id) from role where role_name like concat('%',#{roleName},'%')
</select>
```

```java
Integer roles(String roleName);
```

### 13.2、自动映射和驼峰映射

​		默认情况下，自动映射功能是开启的，使用它的好处在于能有效减少大量的映射配置，从而减少工作量。

​		在setter元素中的autoMappingBeHavior和mapUnderscoreToCamelCase用来控制自动映射和驼峰映射的开关，一般情况下，自动映射会使用得多一些，因为可以通过SQL别名机制处理一些细节，比较灵活，而驼峰映射则要求比较严苛。

​		autoMappingBeHavior的取值范围：

​				NONE：不进行自动映射

​				PARTIAL：默认值，只对没有嵌套结果集进行自动映射

​				FULL：对所有的结果集进行自动映射，包括嵌套结果集

```xml

<!-- 此时列role_name的值映射到Role对象的roleName属性上 -->
<select id="getRole" parameterType="int" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where id=#{id}
</select>


<!-- 如果按照驼峰命名，此时要求数据字段和POJO属性名严格对应，那么,列role_name对应属性roleName，user_name对应userName，并且还需设置mapUnderscoreToCamelCase为true -->
```

### 13.3、传递多个参数

#### 1、使用map接口传递参数：不推荐

```xml
<select id="roles" parameterType="map" resultType="int">
        select count(id) from role where role_name like concat('%',#{roleName},'%')
</select>
```

```java
//此时要求，roleName作为param的键
Integer roles(Map<String,Object> param);
```

#### 2、使用注解传递多个参数

​		@Param，用于定义映射器的参数名称。

```java
List<Role> findRoles(@Param("roleName") String rolename);
```

```xml
<!-- 此时并不需要给出parameterType参数，MyBatis会自动探索 -->
<select id="findRoles" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where role_name like concat('%',#{roleName},'%')
    </select>
```

#### 3、通过Java Bean传递多个参数

```xml
<select id="roles" parameterType="com.shanji.over.example.Role" resultType="int">
        select count(id) from role where role_name like concat('%',#{roleName},'%')</select>
```

```java
Integer roles(Role role);
```

### 13.4、使用resultMap映射结果集

​		resultMap：支持复杂的映射

```xml
<!-- 定义一个roleMap，id表示它的标识，type代表使用哪个类作为其映射的类
	id:代表resultMap的主键，result代表其属性，其property代表POJO的属性名，column代表SQL的列名 
-->
<resultMap id="roleMap" type="com.shanji.over.example.Role">
    	<id property="id" column="id" />
        <result property="roleName" column="role_name"  />
        <result property="note" column="note" />
</resultMap>


<!-- resultMap：决定采用哪个resultMap作为其映射规则 -->
<select id="findRoless" parameterType="string" resultMap="roleMap">
        select id,role_name roleName,note from role where note like concat('%',#{note,typeHandler=com.shanji.over.typehandler.StyleTypeHandler},'%')
</select>
```

### 13.5、分页参数RowBounds

​		MyBatis内置处理分页的类：RowBounds。

```java
public class RowBounds {
    public static final int NO_ROW_OFFSET = 0;
    public static final int NO_ROW_LIMIT = 2147483647;
    public static final RowBounds DEFAULT = new RowBounds();
    
    //偏移量，即从第几行开始读取数据
    private final int offset;
    
    //限制条数
    private final int limit;

    public RowBounds() {
        this.offset = 0;
        this.limit = 2147483647;
    }

    public RowBounds(int offset, int limit) {
        this.offset = offset;
        this.limit = limit;
    }

    public int getOffset() {
        return this.offset;
    }

    public int getLimit() {
        return this.limit;
    }
}
```

​		使用时，给接口增加一个RowBounds参数即可，xml文件无需任何改动。

```java
List<Role> findRoles(String roleName, RowBounds rowBounds);
```

```xml
<select id="findRoles" parameterType="string" resultType="com.shanji.over.example.Role">
        select id,role_name roleName,note from role where role_name like concat('%',#{roleName},'%')
    </select>
```

​		MyBatis会自动识别RowBounds参数，并据此进行分页，但是它只能运用于一些小数据量的查询，原理是执行SQL的查询后，按照偏移量和限制条数返回查询结果，对于大量的数据查询，性能不佳。

## 14、insert元素——插入语句

​		属性：

![](image/QQ截图20190827065415.png)

​		在执行完一条insert语句后，会返回一个整数表示其影响记录数。

```xml
<insert id="insertRole" parameterType="com.shanji.over.example.Role">
        insert into role(role_name,note) values (#{roleName},#{note})
</insert>

<!-- 没有配置的属性将采用默认值 -->
```

### 14.1、主键回填

​		JDBC中的Statement对象在执行插入的SQL后，可以通过getGeneratedKeys方法获得数据库生成的主键（需要数据库驱动支持），这样就能达到获取主键的功能，在insert语句中有一个开关属性useGeneratedKeys，用来控制是否打开这个功能，默认为false，打开开关，还要配置其属性keyProperty或keyColumn，告诉系统把生成的主键放入哪个属性中，如果存在多个主键，就用逗号隔开。

```xml
<insert id="insertRole" parameterType="com.shanji.over.example.Role" useGeneratedKeys="true" keyProperty="id">
        insert into role(role_name,note) values (#{roleName},#{note})
</insert>
```

### 14.2、自定义主键

​		对于主键依赖于自定义规则的情况，依赖于selectKey元素进行支持，它允许自定义键值的生成规则。

```xml
<insert id="insertRole" parameterType="com.shanji.over.example.Role" useGeneratedKeys="true" keyProperty="id">
    	<!-- order="BEFORE"表示会在执行插入语句之前执行此SQL，keyProperty="id"指定采用POJO的哪个属性来作为主键 -->
        <selectKey keyProperty="id" resultType="int" order="BEFORE">
            select if(max(id) = null,1,max(id) + 3) from role
        </selectKey>
        insert into role(role_name,note) values (#{roleName},#{note})
</insert>
```

## 15、update元素和delete元素

​		它们执行完后也会返回一个整数，表示影响的条数

```xml
<delete id="deleteRole" parameterType="int">
        delete from role where id=#{id}
    </delete>

    <update id="updateRole" parameterType="com.shanji.over.example.Role">
        update role set role_name=#{roleName},note=#{note} where id=#{id}
    </update>
```

## 16、sql元素

​		sql元素的作用是可以定义一条SQL的一部分，方便后面的SQL引用它。

```xml
<!-- 定义 -->
<sql id="roleCols">
        id,role_name roleName,note
</sql>

<!-- 引用 -->
<select id="getRole" parameterType="int" resultType="com.shanji.over.example.Role">
        select <include refid="roleCols" /> from role where id=#{id}
</select>
```

​		其还支持变量传递

```xml
<!-- 此时引用符号为"$" -->
<sql id="roleCols">
        ${role}.id,${role}.role_name roleName,${role}.note
</sql>

<select id="getRole" parameterType="int" resultType="com.shanji.over.example.Role">
        select 
        <include refid="roleCols">
            <property name="role" value="r" />
        </include>
         from role r where id=#{id}
</select>
```

## 17、参数

### 17.1、存储过程参数支持

​		在存储过程中存在：输入参数（IN），输出参数（OUT），输入输出参数（INOUT）三种类型。

​		输入参数：外界需要传递给存储过程的。

​		输出参数：存储过程经过处理后返回的。

​		输入输出：一方面外界需要可以传递给它，另一方面在最后存储过程也会将它返回给调用者。

​		对于简单的输出参数可以使用POJO通过映射来完成，返回游标，MyBatis也提供了jdbcType为CURSOR对此进行了支持。

​		参数定义：

```properties
#{id,mode=IN}:输入参数
#{roleName,mode=OUT}:输出参数
#{note,mode=INOUT}:输入输出参数
```

### 17.2、特殊字符串的替换和处理（#和$）

​		对于动态列名，可以使用：

```sql
select ${columns} from tableName .....

最后编程
select col1,col2,col3 from tableName ....
```

​		此时MyBatis不会像处理普通参数一样处理他们，

## 18、resultMap元素

​		作用：

​				定义映射规则，级联的更新，定制类型转换器等。但现有版本，只支持resultMap查询，不支持更新或者		保存。

```xml
<resultMap id="" type="">
    	<!-- 配置构造方法 -->
        <constructor>
            <idArg />
            <arg />
        </constructor>
    	<!-- 配置主键列 -->
        <id />
    	<!-- 配置POJO到SQL列名的映射关系 -->
        <result />
        <association property="" />
        <collection property="" />
        <discriminator javaType="">
            <case value=""></case>
        </discriminator>
</resultMap>
```

![](image/QQ截图20190828110357.png)

### 18.1、使用POJO存储结果集

```xml
<resultMap id="userMapper" type="com.shanji.over.example.Role">
        <result property="id" column="id" />
        <result property="roleName" column="role_name"  />
        <result property="note" column="note"  />
</resultMap>



<!-- 此时配置了resultMap，就不能再配置resultType -->
<select id="findRoless" parameterType="string" resultMap="userMapper">
        select id,role_name roleName,note from role where note like concat('%',#{note,typeHandler=com.shanji.over.typehandler.StyleTypeHandler},'%')
</select>
```

## 19、级联

​		级联是resultMap中的配置，级联的好处是获取关键数据十分便捷，但是级联过多会增加系统的复杂度，同时降低系统的性能。当级联的层数超过3层时，就不要考虑使用级联了，这样会造成多个对象的关联，导致系统的耦合，复杂和难以维护。

### 19.1、MyBatis中的级联

​		3种：

​				1、鉴别器：根据某些条件决定采用具体实现类级联的方案。

​				2、一对一。

​				3、一对多。

​		MyBatis没有多对多级联，因为多对多级联比较复杂，使用困难，而且可以通过两个一对多级联进行替换。

```xml
<resultMap id="EmployeeTaskMap" type="com.shanji.over.cascade.EmployeeTask">
        <id column="id" property="id" />
        <result column="emp_id" property="empId" />
        <result column="task_name" property="taskName" />
        <result column="note" property="note" />
        <association property="task" column="task_id" select="com.shanji.over.cascade.TaskMapper.getTask" />
</resultMap>
```

​		association元素代表一对一级联的开始，property属性代表映射到POJO属性上，select配置是命名空间加上SQLid的形式，这样边可以指向对应Mapper的SQL，MyBatis就会通过对应的SQL将数据查新出来，column代表SQL的列，用作参数传递给select属性指定的SQL，如果有多个参数，用逗号隔开。

```xml
<mapper namespace="com.shanji.over.cascade.EmployeeMapper">

    <resultMap id="employee" type="com.shanji.over.cascade.Employee">
        <id column="id" property="id" />
        <result column="real_name" property="realName" />
        <result column="sex" property="sex" typeHandler="com.shanji.over.typehandler.SexEnumTypeHandler" />
        <result column="birthday" property="birthday" />
        <result column="mobile" property="mobile" />
        <result column="email" property="email" />
        <result column="position" property="position" />
        <result column="note" property="note" />
        <association property="workCard" column="id" select="com.shanji.over.cascade.WorkCardMapper.getWorkCardByEmpId" />

        <collection property="employeeTaskList" column="id" select="com.shanji.over.cascade.EmployeeTaskMapper.getEmployeeTaskByEmpid" />

        <discriminator javaType="long" column="sex">
            <case value="1" resultMap="maleHealthFormMapper" />
            <case value="2" resultMap="femaleHealtrhFormMapper" />
        </discriminator>

    </resultMap>


    <resultMap id="femaleHealtrhFormMapper" type="com.shanji.over.cascade.FemaleEmployee" extends="employee">
        <association property="femaleHealthForm" column="id" select="com.shanji.over.cascade.FemaleHealthFormMapper.getFemaleHealthForm" />
    </resultMap>

    <resultMap id="maleHealthFormMapper" type="com.shanji.over.cascade.MaleEmployee" extends="employee">
        <association property="maleHealthForm" column="id" select="com.shanji.over.cascade.MaleHealthFormMapper.getMaleHealthForm" />
    </resultMap>

    <select id="getEmployee" parameterType="long" resultMap="employee">
        select id,real_name realName,sex,birthday,mobile,email,position,note from t_employee where id=#{id}
    </select>
</mapper>
```

​		collection元素：一对多级联，其select元素指向SQL，将通过column指定的SQL字段作为参数进行传递，然后将结果返回给对应的POJO属性。

​		discriminator元素：鉴别器，column代表使用那个字段进行鉴别，case元素，则用于进行区分，resultMap属性表示采用哪个ResultMap区映射。

### 19.2、延迟加载

​		在MyBatis的settings配置中存在两个元素可以配置级联：

![](image/QQ截图20190904111334.png)

​		选项lazyLoadingEnabled决定是否开启延迟加载，而选项aggressiveLazyLoading则控制是否采用层级加载，但是它们都是全局性的配置，有时并不能解决所有的需求。

​		为了解决上述问题，在MyBatis中使用fetchType属性，可以处理全局定义无法处理的问题，进行自定义。

​		fetchType出现在级联元素（association、collection，注意，discriminator没有这个属性）中，两个值：

​				eager：获得当前POJO后立即加载对应的数据。

​				lazy：获得当前POJO后延迟加载对应的数据。

### 19.3、另一种级联

​			MyBatis提供的另一种级联方式，基于SQL表连接。

![](image/QQ截图20190905095931.png)

![](image/QQ截图20190905100014.png)

![](image/QQ截图20190905100042.png)

​		每一个级联元素中属性id的配置和POJO实体配置的id一一对应，形成级联。

​		在级联元素，association是通过javaType的定义去声明实体映射，而collection则是通过ofType进行声明的。

​		discriminator元素定义使用那种具体的resultMap进行级联。

### 19.4、多对多级联

![](image/QQ截图20190905101838.png)

![。](image/QQ截图20190905101859.png)

## 20、缓存

​		一级缓存是在SqlSession上的缓存，二级缓存是在SqlSessionFactory上的缓存。默认情况下，会开启一级缓存，一级缓存不需要POJO对象可序列化。

​		开启二级缓存，在对应的映射文件中加入<cache />，此时要求POJO是可序列化的。MyBatis就会将对应命名空间内所有select元素SQL查询结果进行缓存，而insert、delete、update语句在操作时会刷新缓存。

### 20.1、缓存配置项、自定义和引用

​		cache元素配置项：

![](image/QQ截图20190908093803.png)

​		使用自定义缓存，需要实现Cache接口：、

```java
public interface Cache {
    
    //获取缓存ID
    String getId();

    //保存对象，key为键，value为值
    void putObject(Object var1, Object var2);

    //获取缓存数据，key为键
    Object getObject(Object var1);

    //删除缓存key为键
    Object removeObject(Object var1);

    
    //清除缓存
    void clear();

    //获得缓存大小
    int getSize();

    //获取读/写锁，需要考虑多线程的场景
    ReadWriteLock getReadWriteLock();
}
```

```xml
<!-- 默认配置,flushCache代表是否刷训缓存，useCache代表是否需要使用缓存 -->
<select ... flushCache="false" useCache="true" />
<insert ... flushCache="true" />
<delete ... flushCache="true" />
<update ... flushCache="true" />
```

​		如果多个映射器需要采用同样的配置，可以使用：

```xml
<cache-ref namespace="..." />
```

## 21、存储过程

​		存储过程：数据库的一个概念，数据预先编译好，是放在数据库内存的一个程序片段。

​		三种类型 参数：

​				输入参数：外界给存储过程的参数。

​				输出参数：存储过程经过计算返回给程序的结果参数。

​				输入输出参数：一开始作为参数传递给存储过程，存储过程修改后将其返回的参数。

```xml
<select id="xxx" statementType="CALLABLE">
        {call xxx(#{xx,mode=in,jdbcType=varchar},#{xx,mode=out,jdbcType=varchar})}
 </select>
```

​		statementType指定为CALLABLE，表明在使用存储过程，否则会报错。

​		在调存储过程中放入参数，并在属性上通过mode设置了其输入或者输出参数，指定对应的jdbcType。

### 21.1、游标的使用

​		将jdbcType设为CURSOR，MyBatis会使用ResultSet对象处理对应的结果，只要设置映射关系，MyBatis就会把结果集映射出来。

## 22、动态SQL

![](image/QQ截图20190910102529.png)

### 22.1、if元素

​		if元素是最常用的判断语句，相当于java中的if语句，常常与test属性联合使用。 

```xml
<select id="roles" parameterType="com.shanji.over.example.Role" resultType="int">
        select count(id) from role where role_name
        <if test="roleName != null and roleName != ''">
            like concat('%',#{roleName},'%')
        </if>
    </select>
```

### 22.2、choose、when、otherwise元素

​		choose、when、otherwise元素类似于java中的switch、case、default关键字。

```xml
select count(id) from role where role_name like concat('%',#{roleName},'%')         
        <choose>
            <when test="id != null">
                and id = #{id}
            </when>
            <otherwise>
                limit 0,10
            </otherwise>
        </choose>
```

### 22.3、trim、where、set元素

​		where元素：

![](image/QQ截图20190910104541.png)

​		trim元素：

![](image/QQ截图20190910104620.png)

​				trim表示去掉一些特殊的字符串，prefix代表的是语句的前缀，prefixOverrides代表的是需要去掉哪种		字符串。

​		set元素：

![](image/QQ截图20190910104903.png)

### 22.4、foreach元素

​		循环语句，遍历结合，支持数组和List、Set接口的集合，对此提供遍历功能，往往作用于SQL中的in关键字。

![](image/QQ截图20190910105836.png)

​		collection：配置传递进来的参数名称，可以是一个数组、List、Set等集合。

​		item：配置循环中当前的元素。

​		index：配置当前元素在集合中的位置下标。

​		open、close：配置以什么符号将集合元素包装起来。

​		separator：各个元素的间隔符。

### 22.5、用test的属性判断字符串

​		test用于条件判断语句。相当于判断真假，大部分场景用以判断空和非空，有时需要判断字符串、数字和枚举等。对于枚举，取决于使用那种typeHandler。

### 22.6、bind元素

​		bind的作用是用过OGNL表达式去自定义一个上下文变量。

![](image/QQ截图20190910111904.png)

## 23、构建SqlSessionFactory过程

​		MyBatis运行过程：

​				1、读取配置文件缓存到Configuration对象，用以创建SqlSessionFactory。

​				2、SqlSession的执行过程。

​		SqlSessionFactory最重要的功能就是提供创建MyBatis的核心接口SqlSession，所以要先创建SqlSessionFactory，为此要提供配置文件和相关的参数。

​		MyBatis采用Builder模式去创建SqlSessionFactory，实际中可以通过SqlSessionFactoryBuilder去创建

​				1、通过XMLConfigBuilder解析配置的XML文件，读出所配置的参数，并将读取的内容存入Configuration类对象中，Configuration采用的是单例模式，几乎所有的MyBatis配置内容都会存放在这个单例对象中。

​				2、使用Configuration对象去创建SqlSessionFactory，MyBatis中的SqlSessionFactory是一个接口，而		不是一个实现类，为此MyBatis提供了一个默认的实现类DefaultSqlSessionFactory，在大部分情况下都没有必要自己去创建新的SqlSessionFactory实现类。

```java
//读取配置文件
private void parseConfiguration(XNode root) {
        try {
            this.propertiesElement(root.evalNode("properties"));
            Properties settings = this.settingsAsProperties(root.evalNode("settings"));
            this.loadCustomVfs(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
            this.settingsElement(settings);
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
}
```

```java
//配置的typeHandler会被注册到typeHandlerRegistry中
public BaseBuilder(Configuration configuration) {
        this.configuration = configuration;
        this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
        this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
    }
```

### 23.1、构建Configuration

​		Configuration作用：

​				1、读入配置文件，包括基础配置的XML和映射器XML（或注解）。

​				2、初始化一些基础配置。

​				3、提供单例，为后续创建SessionFactory服务，提供配置的参数。

​				4、执行一些重要对象的初始化方法。

​		Configuration是通过XMLConfigurationBuilder去构建的，首先会读出所有XML配置的信息，然后把它们解析并保存在Configuration单例中。

​		初始化：

​				1、properties全局参数。

​				2、typeAliases别名。

​				3、Plugins插件。

​				4、objectWrapperFactory对象包装工厂。

​				5、reflectionFactory反射工厂。

​				6、settings环境配置。

​				7、environments数据库环境。

​				8、databaseIdProvider数据库标识。

​				9、typeHandlers类型转换器。

​				10、Mappers映射器。

### 23.2、构建映射器的内部组成

​		当XMLConfigBuilder解析XML时，会将每一个SQL和其配置的内容保存起来。

​		MyBatis中一条SQL及其相关的配置信息由3个部分组成：

​				MappedStatement：保存一个映射器结点的内容，是一个类，包括许多配置的SQL，SQL的id，缓存信		息，resultMap，parameterType，resultType，resultMap，languageDriver等重要配置内容，还有一个		sqlSource，MyBatis通过它来获取某条SQL配置的所有信息。

​				SqlSource：提供BoundSql对象的地方，是MappedStatement的一个属性，是一个接口，实现类：		DynamicSqlSource，ProviderSqlSource，RawSqlSource，StaticSqlSource。其作用是根据上下文和参数解		析生成需要的SQL，其getBoundSql方法返回一个BoundSql对象。

​				BoundSql：结果对象，也就是SqlSource通过对SQL和参数的联合解析得到的SQL和参数，它是建立SQL		和参数的地方，3个常用属性：sql，parameterObject，parameterMappings。

![](image/QQ截图20190911211758.png)

​						1、parameterObject：参数本身，可以传递简单对象，POJO，或者Map，@Param注解的参数。

​								传递简单对象：包括基本数据类型等，此时会将基本类型转为对应的包装类型。

​								传递POJO或者Map：parameterObject就是传入的POJO或者Map。

​								传递多个参数：如果没有@Param注解，会把parameterObject变成一个Map<String,Object>						对象，其键值的关系是按顺序来规划的。所以可以在编写时可以使用 #{param1}或者#{1}去引用第						一个参数。

​								使用@Param注解：也会将parameterObject转为Map<String,Object>对象，只是将数字的键						值换成@Param注解键值。

​						2、parameterMappings：一个List，每一个元素都是ParameterMapping对象，对象会描述参数，				参数包括属性名称，表达式，javaType，jdbcType，typeHandler等重要信息，通过它可以实现参数和				SQL的结合，以便PreparedStatement能够通过它找到parameterObject对象的属性设置参数，使得程				序能准确运行。

​						3、sql：映射器中一条被SqlSource解析后的SQL。

### 23.3、构建SqlSessionFactory

​		使用Configuration对象构建SqlSessionFactory：

```java
//通过文件流先生成Configuration独享，进而构建SqlSessionFactory对象
sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

## 24、SqlSession运行过程

​		MyBatis只用Mapper接口就能运行：Mappler的XML文件的命名空间对应的是这个接口的全限定名，而方法就是那条SQL的id，这样MyBatis就可以根据全路径和方法名，将其和代理对象绑定起来，通过动态代理技术，让这个接口运行起来，而后采用命令模式，最后使用SqlSession接口的方法使得它能够执行对应的SQL。

### 24.1、SqlSession下的四大对象

​		映射器就是一个动态代理对象进入到了MapperMethod的execute方法，然后经过简答的判断进入SqlSession的delte、update、insert、select等方法。

​		Executor：执行器，由它调度StatementHandler，ParameterHandler，ResultSetHandler等来执行相应的SQL。

​		StatementHandler：使用数据库的Statement(PreparedStatement)执行操作，四大对象的核心。

​		ParameterHandler：用来SQL参数。

​		ResultSetHandler：进行数据集的封装返回处理。

#### 24.1.1、Executor——执行器

​		SqlSession只是一个门面，真正干活的是执行器，其是一个真正执行 Java和数据库交互的对象。

​		三种执行器：

​				SIMPLE：简易执行器，默认执行器。

​				REUSE：一种能够执行重用预处理语句的执行器。

​				BATCH：执行器重用语句和批量更新，批量专用的执行器。

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
        executorType = executorType == null ? this.defaultExecutorType : executorType;
        executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
        Object executor;
        if (ExecutorType.BATCH == executorType) {
            executor = new BatchExecutor(this, transaction);
        } else if (ExecutorType.REUSE == executorType) {
            executor = new ReuseExecutor(this, transaction);
        } else {
            executor = new SimpleExecutor(this, transaction);
        }

        if (this.cacheEnabled) {
            executor = new CachingExecutor((Executor)executor);
        }

        Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
        return executor;
    }
```

​		MyBatis根据Configuration来构建StatementHandler，然后使用prepareStatement方法，对SQL编译和参数进行初始化，其调用StatementHandler的prepare方法进行了预编译和基础的设置，然后通过StatementHandler的parameterize方法来设置参数，最后使用StatementHandler的query方法，把ResultHandler传递进去，使用它组织结果返回给调用者完成一次查询。

#### 24.1.2、StatementHandler——数据库会话器

​		顾名思义：专门处理数据库会话。

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
        StatementHandler statementHandler = (StatementHandler)this.interceptorChain.pluginAll(statementHandler);
        return statementHandler;
    }
```

​		真实对象为RoutingStatementHandler对象。其并不是真实的服务对象，通过适配模式来找到对应的StatementHandler来执行的。

​		SimpleStatementHandler：对应JDBC的Statement。

​		PreparedStatementHandler：对应PreparedStatement（预处理）。

​		CallableStatementHandler：对应CallableStatement（存储过程处理）。

```java
public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        switch(ms.getStatementType()) {
        case STATEMENT:
            this.delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case PREPARED:
            this.delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case CALLABLE:
            this.delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        default:
            throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
        }

    }
```

​		一条查询SQL的执行过程：

​					Executor先调用StatementHandler的prepare方法预编译SQL，同时设置一些基本运行的参数，然后		用parameterize方法启用ParameterHandler设置参数，完成预编译，执行查询。update方法也是这样。如		果是查询，MyBatis会使用ResultSetHandler封装结果返回给调用者。

#### 24.1.3、ParameterHandler——参数处理器

​		作用：完成对预编译参数的设置。

​		DefaultParameterHandler的setParameters方法：

```java
public void setParameters(PreparedStatement ps) {
        ErrorContext.instance().activity("setting parameters").object(this.mappedStatement.getParameterMap().getId());
        List<ParameterMapping> parameterMappings = this.boundSql.getParameterMappings();
        if (parameterMappings != null) {
            for(int i = 0; i < parameterMappings.size(); ++i) {
                ParameterMapping parameterMapping = (ParameterMapping)parameterMappings.get(i);
                if (parameterMapping.getMode() != ParameterMode.OUT) {
                    String propertyName = parameterMapping.getProperty();
                    Object value;
                    if (this.boundSql.hasAdditionalParameter(propertyName)) {
                        value = this.boundSql.getAdditionalParameter(propertyName);
                    } else if (this.parameterObject == null) {
                        value = null;
                    } else if (this.typeHandlerRegistry.hasTypeHandler(this.parameterObject.getClass())) {
                        value = this.parameterObject;
                    } else {
                        MetaObject metaObject = this.configuration.newMetaObject(this.parameterObject);
                        value = metaObject.getValue(propertyName);
                    }

                    TypeHandler typeHandler = parameterMapping.getTypeHandler();
                    JdbcType jdbcType = parameterMapping.getJdbcType();
                    if (value == null && jdbcType == null) {
                        jdbcType = this.configuration.getJdbcTypeForNull();
                    }

                    try {
                        typeHandler.setParameter(ps, i + 1, value, jdbcType);
                    } catch (TypeException var10) {
                        throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + var10, var10);
                    } catch (SQLException var11) {
                        throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + var11, var11);
                    }
                }
            }
        }

    }
```

​		从parameterObject对象中取到参数，然后使用typeHandler转换参数，如果有设置，会根据签名注册的typeHandler对参数进行处理。

#### 24.1.4、ResultSetHandler——结果处理器

​		作用：组装结果集返回。

```java
public interface ResultSetHandler {
    
    //包装结果集
    <E> List<E> handleResultSets(Statement var1) throws SQLException;

    <E> Cursor<E> handleCursorResultSets(Statement var1) throws SQLException;
    //处理存储过程输出参数
    void handleOutputParameters(CallableStatement var1) throws SQLException;
}
```

### 24.2、总结

![](image/QQ截图20190916210442.png)

## 25、插件

### 25.1、插件接口

​		在MyBatis中使用插件，必须实现接口Interceptor。

```java
public interface Interceptor {
    //插件的核心方法，将直接覆盖拦截对象原有的方法，通过var1可以反射调度原来对象的方法
    Object intercept(Invocation var1) throws Throwable;

    //var1：被拦截对象，作用是给被拦截对象生成一个代理对象，并返回它。在MyBatis中，提供了package org.apache.ibatis.plugin.Plugin中的wrap静态方法生成代理对象，一般情况下都会使用它来生成代理对象。也可以自定义。
    Object plugin(Object var1);

    //允许在plugin元素中配置所需参数，方法在插件初始化时就被调用了一次，然后把插件对象存入到配置中，以便后面再取出。
    void setProperties(Properties var1);
}
```

### 25.2、插件的初始化

​		插件的初始化是在MyBatis初始化时完成的。

```java
private void pluginElement(XNode parent) throws Exception {
        if (parent != null) {
            Iterator var2 = parent.getChildren().iterator();

            while(var2.hasNext()) {
                XNode child = (XNode)var2.next();
                String interceptor = child.getStringAttribute("interceptor");
                Properties properties = child.getChildrenAsProperties();
                Interceptor interceptorInstance = (Interceptor)this.resolveClass(interceptor).newInstance();
                interceptorInstance.setProperties(properties);
                this.configuration.addInterceptor(interceptorInstance);
            }
        }

    }
```

​		MyBatis的上下文初始化过程中，就开始读入插件节点和配置的参数，同时使用反射生成对应的插件实例，调用插件方法的setProperties方法，设置配置参数，将插件实例保存到配置对象中。

### 25.3、插件的代理和反射设计

​		插件用的是责任链模式。

```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
        ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
        parameterHandler = (ParameterHandler)this.interceptorChain.pluginAll(parameterHandler);
        return parameterHandler;
   }
```

```java
public Object pluginAll(Object target) {
        Interceptor interceptor;
        for(Iterator var2 = this.interceptors.iterator(); var2.hasNext(); target = interceptor.plugin(target)) {
            interceptor = (Interceptor)var2.next();
        }

        return target;
    }
```

​		plugin 方法是生成代理对象的方法 它是从 Configuration 对象中取出插件的。从第一个对象（四大对象中的一个）开始，将对象传递给了plugin 方法，然后返回一个代理。如果存在第二个插件，那么就拿到第一个代理对象，传递给plugin 方法，再返回第一个代理对象的代理……依次类推，有多少个拦截器就生成多少个代理对象。每 个插件都可以拦截到真实的对象。这就好比每一个插件都可以一层层处理被拦截的对象。

​		自定义代理类，MyBatis提供了一个常用的工具类Plugin，Plugin实现了InvocationHandler，采用JDK的动态代理。

```java
public class Plugin implements InvocationHandler {
    //生成动态代理对象
	public static Object wrap(Object target, Interceptor interceptor) {
        Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
        Class<?> type = target.getClass();
        Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
        return interfaces.length > 0 ? Proxy.newProxyInstance(type.getClassLoader(), interfaces, new Plugin(target, interceptor, signatureMap)) : target;
    }
    //如果此类为插件生成代理对象，那么代理对象在调用方法时会进入这个方法。
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            Set<Method> methods = (Set)this.signatureMap.get(method.getDeclaringClass());
            return methods != null && methods.contains(method) ? this.interceptor.intercept(new Invocation(this.target, method, args)) : method.invoke(this.target, args);
        } catch (Exception var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
    }
    ....
}
```

```java
public class Invocation {
    private final Object target;
    private final Method method;
    private final Object[] args;

    public Invocation(Object target, Method method, Object[] args) {
        this.target = target;
        this.method = method;
        this.args = args;
    }

    public Object getTarget() {
        return this.target;
    }

    public Method getMethod() {
        return this.method;
    }

    public Object[] getArgs() {
        return this.args;
    }

    //通过反射调用代理对象的真实方法
    public Object proceed() throws InvocationTargetException, IllegalAccessException {
        return this.method.invoke(this.target, this.args);
    }
}
```

### 25.4、常用的工具类-MetaObject

​		MetaObject：

![](image/QQ截图20191019154045.png)

### 25.5、插件开发过程和实例

#### 25.5.1、确定需要拦截的签名

​		MyBatis允许拦截四大对象中的任意一个对象。

​				1、确定拦截对象

![](image/QQ截图20191019154924.png)

![](image/QQ截图20191019155007.png)

​				2、拦截方法和参数

​		确定了拦截的对象，接着需要去定拦截什么方法及方法的参数。

​		以查询为例：查询的过程是通过Executor调度StatementHandler来完成的。调用其prepare方法预编译SQL。

```java
public interface StatementHandler {
    Statement prepare(Connection var1, Integer var2) throws SQLException;

    void parameterize(Statement var1) throws SQLException;

    void batch(Statement var1) throws SQLException;

    int update(Statement var1) throws SQLException;

    <E> List<E> query(Statement var1, ResultHandler var2) throws SQLException;

    <E> Cursor<E> queryCursor(Statement var1) throws SQLException;

    BoundSql getBoundSql();

    ParameterHandler getParameterHandler();
}
```

​				此时拦截器如下：

```java
/*
@Intercepts：表明其是一个拦截器
@Signature：注册拦截器签名的地方
type：拦截的对象
method：拦截的方法
args：方法的参数
*/

@Intercepts(@Signature(type = StatementHandler.class,method = "prepare",args = {Connection.class,Integer.class}))
public class StylePlugin implements Interceptor
{
    private Properties properties = null;


    //插件方法，替换StatementHandler的prepare方法
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        StatementHandler target = (StatementHandler) invocation.getTarget();

        //进行绑定
        MetaObject metaObject = SystemMetaObject.forObject(target);

        Object object = null;
        //分离代理对象链
        while (metaObject.hasGetter("h"))
        {
            object = metaObject.getValue("h");
            metaObject = SystemMetaObject.forObject(object);
        }
        target = (StatementHandler) object;
        String sql = (String) metaObject.getValue("delegate.boundSql.sql");
        Long value = (Long) metaObject.getValue("delegate.boundSql.parameterObject");
        System.out.println("执行的SQL：" + sql);
        System.out.println("参数：" + value);
        System.out.println("before ....");
        Object proceed = invocation.proceed();
        System.out.println("after ....");
        return proceed;
    }


    //生成代理对象
    @Override
    public Object plugin(Object o) {
        return Plugin.wrap(o,this);
    }

    //设置参数，MyBatis初始化时，会生成插件实例，并调用此方法
    @Override
    public void setProperties(Properties properties) {
        this.properties = properties;
    }
}
```

​				3、配置和运行

```xml
<plugins>
        <plugin interceptor="com.shanji.over.interceptor.StylePlugin">
            <property name="dbType" value="mysql" />
        </plugin>
    </plugins>
```

# 四、整合Spring、MyBatis

步骤：

1、配置数据源

2、配置SqlSessionTemplate，在同时配置 SqlSessionTemplate和SqlSessionFactory的情况下，优先采用SqlSessionTemplate。

3、配置Mapper，可以配置单个Mapper，也可以通过扫描的方法生成Mapper。此时注意maven的扫描根路径的配置

4、事务管理

# 五、Spring  MVC

![](image/QQ截图20191020164735.png)

![](image/QQ截图20191020165038.png)

![](image/QQ截图20191020170201.png)

## 1、Spring  MVC初始化

### 1.1、初始化Spring  IoC上下文

​		Java  Web容器为其生命周期提供了ServletContextListener，这个接口可以在Web容器初始化和结束中执行一定的逻辑，通过实现它可以使得在DispatcherServlet初始化前就可以完成Spring  IoC容器的初始化，也可以在结束前完成对Spring  IoC容器的销毁，只要实现ServletContextListener接口的方法就可以了。

```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        //初始化IoC容器，使用的是满足ApplicationContext接口的Spring Web IoC容器
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        //关闭Web IoC容器
        this.closeWebApplicationContext(event.getServletContext());
        //清除相关参数
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
```

### 1.2、初始化映射请求上下文

​		映射请求上下文是通过DispatcherServlet初始化的，与普通的Servlet也是一样的。

​		Spring  MVC的核心组件：

​				MultipartResolver：文件解析器，用于支持服务器的文件上传。

​				LocaleResolver：国际化解析器，可以提供国际化的功能。

​				ThemeResolver：主题解析器，类似于软件皮肤的转换功能。

​				HandlerMapping：包装用户提供一个控制器的方法和对它的一些拦截器，通过调用它就能够运行控制		器。

​				handlerAdapter：处理器适配器，因为处理器会在不同的上下文中运行，所以Spring  MVC会先找到合		适的适配器，然后运行处理器服务方法。

​				HandlerExceptionResolver：处理器异常解析器，处理器有可能产生异常，如果产生异常，则可以通过		异常解析器来处理它。

​				RequestToViewNameTranslator：视图逻辑名称转化器，有时候在控制器中返回一个视图的名称，通过		它可以找到实际的视图，当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名。

​				ViewResolver：视图解析器，当控制器返回后，通过视图解析器会把逻辑视图名成进行解析，然后定位		实际视图。

### 1.3、使用注解配置方式初始化

​		由于在Servlet3.0之后的规范允许取消web.xml配置，只使用注解方式便可以了。

​		首先继承AbstractAnnotationConfigDispatcherServletInitializer，然后实现它所定义的方法。

## 2、Spring MVC开发流程

​		注解方式：以一个注解@Controller标注，一般只需要通过扫描配置，就能够将其扫描出来，但往往还需要结合注解@RequestMapping去配置它。@RequestMapping可以配置在类或者方法上，作用是指定URI和哪个类或者方法作为一个处理请求的处理器，为了更加灵活，Spring MVC还定义了处理器中的拦截器，当启动Spring  MVC的时候，Spring  MVC就会去解析@Controller中的@RequestMapping的配置，在结合所配置的拦截器，组成多个拦截器和一个控制器的形式，存放到一个HandlerMapping中去。

### 2.1、配置@RequestMapping

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    //请求路径
    String name() default "";

    //请求路径，可以是数组
    @AliasFor("path")
    String[] value() default {};

    //请求路径，可以是数组
    @AliasFor("value")
    String[] path() default {};

   	//请求类型，GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS,TRACE
    RequestMethod[] method() default {};

    //请求参数，当请求带有配置的参数时，才匹配处理器
    String[] params() default {};

    //请求头，当HTTP请求头为配置项时，才匹配处理器
    String[] headers() default {};

    //请求类型为配置类型才匹配处理器
    String[] consumes() default {};

    //处理器之后的响应用户的结果类型，如：{"application/json;charset=UTF-8","text/plain","application/*"}
    String[] produces() default {};
}
```

### 2.2、控制器的开发

​		三步：

​				1、获取请求参数

​				2、处理业务逻辑

​				3、绑定模型和视图

​		接受参数时，建议不要使用Servlet容器所给予的API，此时控制器将会依赖于Servlet容器。

​		@RequestParam：获取指定参数值，并做相应的类型转换。此时要求参数不为空，否则会报错。

​					属性：required：布尔值，默认true，不允许参数为空。

​								defaultValue：默认值

​		@SessionAttribute：获取Session里指定的数据。

实现逻辑和绑定视图

​		配置数据库：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 使用注解驱动 -->
    <context:annotation-config />
    <!-- 引入配置文件 -->
    <context:property-override location="classpath:database.properties" />
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${database.driver}" />
        <property name="jdbcUrl" value="${database.url}" />
        <property name="user" value="${database.username}" />
        <property name="password" value="${database.password}" />
    </bean>

    <!-- 集成mybatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath*:com/shanji/**/*.xml" />
    </bean>

    <!-- 配置数据源事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 采用自动扫描方式创建mapperbean -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="mvc" />
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
        <property name="annotationClass" value="org.springframework.stereotype.Repository" />
    </bean>
</beans>
```

### 2.3、视图渲染

​		Spring  MVC默认使用JstlView进行渲染，即：将查询出来的模型绑定到JSTL（JSP标准标签库）模型中，通过JSTL就可以把数据模型在JSP中读出展示数据。

![](image/QQ截图20191024220432.png)

## 3、控制器接受各类请求参数

### 1、接受普通请求参数

​		如果传递过来的参数名称和HTTP的保存一致，那么无需任何注解也可以获取参数。

```java
	@RequestMapping("/commonParams")
    public ModelAndView commonParams(String roleName,String note)
    {
        System.out.println("roleName=>" + roleName);
        System.out.println("note=>" + note);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

​		在参数很多的情况下，使用一个POJO来管理这些参数。POJO属性和HTTP参数一一对应-0

### 2、使用@RequestParam

​		当前端的命名规则和后端不一样时，可以使用@RequestParam来接收参数。

```java
	@RequestMapping("/commonParams")
	//如果role_name为空，将会报错，此时可以修改属性required为false
    public ModelAndView commonParams(@RequestParam("role_name") String roleName, String note)
    {
        System.out.println("roleName=>" + roleName);
        System.out.println("note=>" + note);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

### 3、使用URL传递参数

​		需要使用@RequestParam与@PathVariable两个注解

```java
	//http://localhost:8080/mybatisspring/test/1.do
	@RequestMapping("/test/{id}")
    @ResponseBody
    public Map test(@PathVariable("id")Integer id)
    {
        Map role = roleService.getRole(id);
        return role;
    }
```

​		{id}：接受一个由URL组成的参数，@PathVariable获取该参数。

### 4、传递JSON数据

```java
	@RequestMapping("/commonParamsPojo")
    public ModelAndView commonParamsPojo(@RequestBody Role role)
    {
        System.out.println("roleName=>" + role.getRoleName());
        System.out.println("note=>" + role.getNote());
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

```javascript
$(document).ready(function () {
        var data = {
            roleName : "小杉杉",
            note : "测试",
            page : {
                start : 1,
                limit : 10
            }
        };

        $.ajax({
            //请求方式
            type : "POST",
            //请求的媒体类型
            contentType: "application/json;charset=UTF-8",
            //请求地址
            url : "<%=baseUrl%>/params/commonParamsPojo.do",
            //数据，json字符串
            data : JSON.stringify(data),
            //请求成功
            success : function(result) {
                console.log(result);
            },
            //请求失败，包含具体的错误信息
            error : function(e){
                console.log(e.status);
                console.log(e.responseText);
            }
        });
    })
```

​		传递的JSON数据需要和对应参数的POJO保持一致，其次，请求时请求的参数类型为JSON，最后传递的是字符串，然后用@RequestBody接收参数。

### 5、接收列表数据和表单序列化

​		列表：

```java
	@RequestMapping("/commonParamsPojo")
    public ModelAndView commonParamsPojo(@RequestBody List<Role> roleList)
    {
        System.out.println(roleList);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

```javascript
		var roleList = [
            {roleName : "小杉杉",note : "测试1"},
            {roleName : "小山鸡",note : "测试2"}
        ];

        $.ajax({
            //请求方式
            type : "POST",
            //请求的媒体类型
            contentType: "application/json;charset=UTF-8",
            //请求地址
            url : "<%=baseUrl%>/params/commonParamsPojo.do",
            //数据，json字符串
            data : JSON.stringify(roleList),
            //请求成功
            success : function(result) {
                console.log(result);
            },
            //请求失败，包含具体的错误信息
            error : function(e){
                console.log(e.status);
                console.log(e.responseText);
            }
        });
```

​		序列化：

```javascript
		$("#tijiao").click(function () {
            $.ajax({
                //请求方式
                type : "POST",
                //请求地址
                url : "<%=baseUrl%>/params/commonParams.do",
                //数据，json字符串
                data : $("#formtest").serialize(),
                //请求成功
                success : function(result) {
                    console.log(result);
                },
                //请求失败，包含具体的错误信息
                error : function(e){
                    console.log(e.status);
                    console.log(e.responseText);
                }
            });
        })
```

```java
	@RequestMapping("/commonParams")
    public ModelAndView commonParams(String roleName, String note)
    {
        System.out.println("roleName=>" + roleName);
        System.out.println("note=>" + note);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

## 4、重定向

​		Spring  MVC通过返回字符串来实现重定向功能，并可以附加一个Model的实例，此实例将实例化重定向的下一个中的对应实例。

```java
	@RequestMapping("/commonParams")
    public ModelAndView commonParams(String roleName, String note)
    {
        System.out.println("roleName=>" + roleName);
        System.out.println("note=>" + note);
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }

	@RequestMapping("/addRole")
    public String addRole(Model model,String roleName,String note)
    {
        model.addAttribute("roleName",roleName);
        model.addAttribute("note",note);
        //当返回字符串以redirect开头时，认为需要的是一个重定向，也可以返回一个ModelAndView的实例，将实例名设置为  redirect:./commonParams.do  也可实现相同的效果。
        return "redirect:./commonParams.do";
    }

	
	//附加一个对象
	@RequestMapping("/addRoleByPojo")
    public String addRoleByPojo(RedirectAttributes re, Role role)
    {
        //此时Spring  MVC会将对象暂时存入Session中，从定向后就会将其清除。
        re.addFlashAttribute(role);
        return "redirect:./commonParamsPojos.do";
    }
		
	@RequestMapping("/commonParamsPojos")
    public ModelAndView commonParamsPojos(Role roleList)
    {
        System.out.println(roleList);
//        System.out.println("roleName=>" + role.getRoleName());
//        System.out.println("note=>" + role.getNote());
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

![](image/QQ截图20191029214233.png)

## 5、保存并获取属性参数

​		从内置对象中获取参数：

​				request：使用@RequestAttribute(默认不能为空，可配置required属性为false，允许为空)，相当于调用 request.getAttribute("xxx")返回的是对象。注意获取		表单中的参数使用的是  request.getParameter("xxx")返回的是字符串。

​				session：使用@SessionAttribute，相当于调用 session.getAttribute("xxx")。

​				@SessionAttributes：可以给它配置一个字符串数组，这个数组对应的是数据模型对应的键值对，然后		将这些键值对保存到session中。

​		@SessionAttributes只能对类进行标注，不能对方法或者参数注解，它可以配置属性名称或者属性类型，它的作用是当这个类被注解后，Spring  MVC执行完控制器的逻辑后，将数据模型中对应的属性名称或者属性类型保存到session对象中。

```java
@Controller
@RequestMapping("/params")
@SessionAttributes(names = {"id"},types = {Role.class})
public class ParamsController
{
    @RequestMapping("/commonParamsPojos")
    public ModelAndView commonParamsPojos(Role roleList,@SessionAttribute(name = "role",required = false) Role role)
    {
        System.out.println(roleList);
//        System.out.println("roleName=>" + role.getRoleName());
//        System.out.println("note=>" + role.getNote());
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }

    @RequestMapping("/addRoleByPojo")
    public String addRoleByPojo(RedirectAttributes re, Role role)
    {
        re.addFlashAttribute(role);
        return "redirect:./commonParamsPojos.do";
    }
}
```

```jsp
<%@ page import="com.shanji.role.Role" %>
<%@page contentType="text/html" pageEncoding="utf-8" %>
<%@include file="base.jsp"%>
<html>
<body>
<h2>Hello World!</h2>
<%
    Role role = (Role) session.getAttribute("role");
    out.print(role);
%>
</body>
<script src="<%=baseUrl%>/webjars/jquery/3.4.1/dist/jquery.min.js"></script>
<script>
    
</script>
</html>

```

​		@@CookieValue与@RequestHeader用来从Cookie和HTTP请求头获取对应的信息，注意对于Cookie而言，用户是可以禁用的。

```java
	@RequestMapping("/testRequestHeader")
    public ModelAndView testRequestHeader(@RequestHeader(value = "User-Agent",required = false,defaultValue = "attribute")String userAgent,@CookieValue(value = "JSESSIONID",required = false,defaultValue = "xiaoshanshan")String sessionid)
    {
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
```

## 6、拦截器

​		拦截器可以在进入处理器之前或处理器完成之后进行操作，甚至在渲染视图后进行操作。

​		Spring  MVC在启动期间通过@RequestMapping的注解解析URI和处理器的对应关系，在运行时通过请求找到对应的HandlerMapping，然后构建HandlerExecutionChain对象，其是一个执行的责任链对象。

### 1、定义

​		拦截器需要实现HandlerInterceptor

```java
public interface HandlerInterceptor {
    //在处理器之前执行的前置方法,当返回false，不会再执行后面的逻辑
    boolean preHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;

    //在处理器之后执行的后置方法
    void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception;

    //无论是否产生异常都会在渲染视图后执行的方法
    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception;
}
```

### 2、执行流程

![](image/QQ截图20191031154317.png)

### 3、开发

​		在XML配置<mvc:annotation-driven />或者使用注解@EnableWebMvc时，系统就会初始化拦截器ConversionServiceExposingInterceptor，它是一个一开始就被Spring  MVC系统默认加载的拦截器，主要作用是根据配置在控制器上的注解来完成对应的功能。

![](image/QQ截图20191031160056.png)

​		如果只想实现拦截器中的部分方法时可以继承HandlerInterceptorAdapter。

```java
public class RoleInterceptor extends HandlerInterceptorAdapter
{
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

```xml
	<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**/**"/>  <!-- 拦截所有请求 -->
            <bean class="com.shanji.interceptror.RoleInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>
```

### 4、多个拦截器的执行顺序

​		当配置多个拦截器时：

​				正常情况下：按配置顺序从第一个拦截器开始进入前置方法，前置方法的执行顺序是配置顺序，然后运		行处理器，再运行后置方法，后置方法和完成方法。**后置方法和完成的顺序和配置顺序相反。**

​				非正常情况下（其中某一个preHandle方法返回false）：按配置顺序，后面的preHandle方法都不会执		行，控制器和所有的后置方法postHandle也不会运行。执行过preHandle方法且该方法返回true的完成方法		将按照与配置相反的顺序运行。

## 7、验证器

### 1、使用 JSR 303注解验证输入内容

Spring 提供了对Bean的功能校验，通过注解@Valid表明哪个Bean需要启动注解式的验证。

![](image/QQ截图20191102153311.png)

```java
@Controller
@RequestMapping("/verify")
public class VerifyController
{
    @RequestMapping("/annotation")
    public ModelAndView annotationValidate(@Valid Transaction trans, Errors errors)
    {
        if(errors.hasErrors())
        {
            List<FieldError> fieldErrors = errors.getFieldErrors();
            for(FieldError error : fieldErrors)
            {
                System.out.println("fied:" + error.getField() + "\tsg:" +error.getDefaultMessage());
            }
        }
        ModelAndView mv = new ModelAndView("index");
        return mv;
    }
}



public class Transaction
{
    @NotNull
    private Long productId;

    @NotNull
    private Long userId;

    @Future
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    @NotNull
    private Date date;

    @NotNull
    @DecimalMin(value = "0.1")
    private Double price;

    @Min(1)
    @Max(100)
    @NotNull
    private Integer quantity;

    @NotNull
    @DecimalMax("500000.00")
    @DecimalMin("1.00")
    private Double amount;

    @Pattern(regexp = "^([a-zA-Z0-9]*[-_]?[a-zA-Z0-9]+)*@([a-zA-Z0-9]*[-_]?[a-zA-Z0-9]+)+[\\.][A-Za-z]{2,3}([\\.][a-zA-Z]{2})?$",message = "不符合邮件格式")
    private String email;

    @Size(min = 0,max = 256)
    private String note;
	//getter  and  setter
}
```

```jsp
<%@page contentType="text/html" pageEncoding="utf-8" %>
<%@include file="base.jsp"%>
<html>
<body>
<form id="formtest" action="<%=baseUrl%>/verify/annotation.do">
    <table>
        <tr>
            <td>产品编号：</td>
            <td><input type="text" id="productId" name="productId" value="" /></td>
        </tr>
        <tr>
            <td>用户编号：</td>
            <td><input type="text" id="userId" name="userId" value="" /></td>
        </tr>
        <tr>
            <td>交易日期：</td>
            <td><input type="text" id="date" name="date" value="" /></td>
        </tr>
        <tr>
            <td>价格：</td>
            <td><input type="text" id="price" name="price" value="" /></td>
        </tr>
        <tr>
            <td>数量：</td>
            <td><input type="text" id="quantity" name="quantity" value="" /></td>
        </tr>
        <tr>
            <td>交易金额：</td>
            <td><input type="text" id="amount" name="amount" value="" /></td>
        </tr>
        <tr>
            <td>用户邮件：</td>
            <td><input type="text" id="email" name="email" value="" /></td>
        </tr>
        <tr>
            <td>备注：</td>
            <td>
                <textarea id="note" name="note" cols="20" rows="5"></textarea>
            </td>
        </tr>
        <tr>
            <td colspan="2" align="right">
                <input type="submit" value="提交" />
            </td>
        </tr>
    </table>
</form>
</body>
</html>

```

### 2、使用验证器

​		对于业务校验，Spring提供了Validator接口来实现校验，它将在进入控制器逻辑之前对参数的合法性进行校验。

```java
public interface Validator {
    
    //判断当前验证器是否用于检验clazz类型的POJO，return true启动检验，false则不在检验
    boolean supports(Class<?> var1);

    //检验POJO的合法性
    void validate(Object var1, Errors var2);
}
```

​		JSR 303注解方式和验证器方式不能同时使用。

## 8、数据模型

​		ModelAndView来定义视图类型，包括JSON属兔，也是用它来加载数据模型。ModelAndView有一个类型为ModelMap的属性model，而ModelMap继承了LinkedHashMap<String,Object>，因此它可以存放各种键值对，为了进一步定义数据模型功能，Spring还创建了类ExtendedModelMap，这个类实现了数据模型定义的Model接口，并且还在此基础上派生了关于数据绑定的类——BindingAwareModelMap。

![](image/QQ截图20191111190107.png)

​		在控制器方法中，可以把ModelAndView，Model，ModelMap作为参数。在Spring  MVC运行的时候，回自动初始化它们，因此可以选择ModelMap或者Model作为数据模型。事实上Spring  MVC创建的是一个BindingAwareModelMap实例。根据上图，可以通过强制转换把Model变成ModelMap，或者把ModelMap转换为Model。ModelAndView初始化后，Model属性为空，当调用它增加数据模型的方法后，会自动创建一个ModelMap实例，用以保存数据模型。

```java
	//这两个方法，并没有将数据模型绑定到属兔或者模型，此时Spring MVC将在完成控制器逻辑后，自动绑定
	@RequestMapping(value = "/getRoleByModelMap",method = RequestMethod.GET)
    public ModelAndView getRoleByModelMap(@RequestParam("id")Integer id, ModelMap modelMap)
    {
        Map role = roleService.getRole(id);
        ModelAndView mv = new ModelAndView();
        mv.setViewName("index");
        modelMap.addAttribute("role",role);
        return mv;
    }

    @RequestMapping(value = "/getRoleByModel",method = RequestMethod.GET)
    public ModelAndView getRoleByModel(@RequestParam("id")Integer id, Model model)
    {
        Map role = roleService.getRole(id);
        ModelAndView mv = new ModelAndView();
        mv.setViewName("index");
        model.addAttribute("role",role);
        return mv;
    }
```

## 9、视图和视图解析器

### 1、视图

​		在请求之后，Spring  MVC控制器获取了对应的数据，绑定到数据模型中，那么视图就可以展示数据模型的信息了。

​		Spring  MVC定义了多种视图，它们都满足视图的定义，接口——View。

```java
public interface View {
    //响应状态属性
    String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";
    //定义数据模型下取出变量路径
    String PATH_VARIABLES = View.class.getName() + ".pathVariables";
    //选择响应内容类型
    String SELECTED_CONTENT_TYPE = View.class.getName() + ".selectedContentType";
	//响应客户端类型，表明用户什么类型的文件响应，可以是HTML,JSON,PDF等
    String getContentType();
	//渲染方法，var1是数据模型，其余参数用于处理HTTP请求的各类问题
    void render(Map<String, ?> var1, HttpServletRequest var2, HttpServletResponse var3) throws Exception;
}
```

​		当控制器返回ModelAndView的时候，视图解析器就会解析它，然后将数据模型传递给render方法，这样就能够渲染视图了，通过各种视图类型的render方法，Spring  MVC可以将视图模型渲染成为各类视图，以满足各种需求。

![](image/QQ截图20191111193726.png)

​		配饰逻辑视图解析器：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/" p:suffix=".jsp" />
```

​		其根本就是创建一个视图解析器，通过前缀和后缀加上视图名称就能找到对应的jsp文件，然后将数据模型渲染到JSP文件中，这样便能展现视图给用户。

### 2、视图解析器

```java
public interface ViewResolver {
    //var2:国际化
    View resolveViewName(String var1, Locale var2) throws Exception;
}
```

![](image/QQ截图20191111194952.png)

​		有时候在控制器中并没有返回一个ModelAndView，而只是返回了一个字符串，也能够渲染视图，因为视图解析器生成了对应的视图。

### 3、实例

​		到处Excel：使用AbstractXlsView

```java
public abstract class AbstractXlsView extends AbstractView {
    public AbstractXlsView() {
        this.setContentType("application/vnd.ms-excel");
    }

    protected boolean generatesDownloadContent() {
        return true;
    }

    protected final void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        Workbook workbook = this.createWorkbook(model, request);
        this.buildExcelDocument(model, workbook, request, response);
        response.setContentType(this.getContentType());
        this.renderWorkbook(workbook, response);
    }

    protected Workbook createWorkbook(Map<String, Object> model, HttpServletRequest request) {
        return new HSSFWorkbook();
    }

    protected void renderWorkbook(Workbook workbook, HttpServletResponse response) throws IOException {
        ServletOutputStream out = response.getOutputStream();
        workbook.write(out);
        if (workbook instanceof Closeable) {
            workbook.close();
        }

    }

    /**
    创建excel文件
    var1：数据模型
    var2：POI workbook对象
    var3，var4：http请求，响应对象
    /
    protected abstract void buildExcelDocument(Map<String, Object> var1, Workbook var2, HttpServletRequest var3, HttpServletResponse var4) throws Exception;
}
```

```java
public class ExcelUtil
{
    @Autowired
    private RoleService roleService;

    public ModelAndView export()
    {
        ModelAndView mv = new ModelAndView();
        ExcelView ev = new ExcelView(exportService());
        ev.setFileName("所有角色.xlsx");
        //List<Role> roles = roleService.findRoles("1");
        mv.addObject("roleList",roles);
        mv.setView(ev);
        return mv;
    }

    private ExcelExportService exportService()
    {
        //导出表格的具体内容
        return (Map<String,Object> model,Workbook workbook) -> {
            List<Role> roleList = (List<Role>) model.get("roleList");
            Sheet sheet = workbook.createSheet("所有角色");
            Row title = sheet.createRow(0);
            title.createCell(0).setCellValue("编号");
            title.createCell(1).setCellValue("名称");
            title.createCell(2).setCellValue("备注");
            for(int i = 0,len = roleList.size(); i < len ; i++)
            {
                Role role = roleList.get(i);
                int rowIndex = i + 1;
                Row row = sheet.createRow(rowIndex);
                row.createCell(0).setCellValue(role.getId());
                row.createCell(1).setCellValue(role.getRoleName());
                row.createCell(2).setCellValue(role.getNote());
            }
        };
    }
}
```

```java
public class ExcelView extends AbstractXlsView
{

    private String fileName = null;
    private ExcelExportService excelExportService = null;

    public ExcelView(ExcelExportService excelExportService)
    {
        this.excelExportService = excelExportService;
    }

    public ExcelView(String viewName,ExcelExportService excelExportService)
    {
        this.setBeanName(viewName);
    }

    @Override
    protected void buildExcelDocument(Map<String, Object> map, Workbook workbook, HttpServletRequest request, HttpServletResponse response) throws Exception {
        if(excelExportService == null)
        {
            throw new RuntimeException("导出服务接口不能为null！");
        }
        if(!StringUtils.isEmpty(fileName))
        {
            String reqCharset = request.getCharacterEncoding();
            reqCharset = reqCharset == null ? "UTF-8" : reqCharset;
            fileName = new String(fileName.getBytes(reqCharset),"ISO8859-1");
            response.setHeader("Content-disposition","attachment;filename=" + fileName);
            excelExportService.makeWorkBook(map,workbook);
        }
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public ExcelExportService getExcelExportService() {
        return excelExportService;
    }

    public void setExcelExportService(ExcelExportService excelExportService) {
        this.excelExportService = excelExportService;
    }
}
```

```java
public interface ExcelExportService
{
    //用于回调，实现表格的具体内容
    public void makeWorkBook(Map<String,Object> model, Workbook workbook);
}
```

## 10、上传文件

​		Spring  MVC的文件上传是通过MultipartResolver(Multipart解析器)处理的，对于MultipartResolver而言，它只是一个接口，两个实现类：

​				1、CommonsMultipartResovler：依赖于Apache下的 jakrta Common FileUpload项目解析Multipart		请求，可以在Spring的各个版本中使用，只是它要依赖于第三方包才得以实现。

​				2、StandardServletMultipartResolver：是Spring  3.1版本后的产物，它依赖于Servlet3.0或者更高版本		的实现，它不用依赖第三方包。	

![](image/QQ截图20191112182749.png)	

### 1、MultipartResolver

​		配置MultipartResolver：

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver" />
```

​		配置其属性，如：限制单个文件上传大小：在web.xml中

```xml
	<servlet>
      <servlet-name>dispatcher</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <load-on-startup>2</load-on-startup>
        <!-- 配置文件上传的属性：注意需要servlet3.0及以上 -->
      <multipart-config>
        <location>/upload/image/</location>
        <max-file-size>5242880</max-file-size>
        <max-request-size>10485760</max-request-size>
        <file-size-threshold>0</file-size-threshold>
      </multipart-config>
    </servlet>
```

​		当使用的Spring或者servlet版本过低时，就只能使用CommonsMultipartResolver。此时需要引入第三方包

```java
	@Bean(name = "multipartResolver")
    public MultipartResolver initCommonsMultipartResolver()
    {
        String filepath = "/upload/image/";
        long singleMax = (long) (5 * Math.pow(2,20));
        long totalMax = (long) (10 * Math.pow(2,20));
        CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
        commonsMultipartResolver.setMaxUploadSizePerFile(singleMax);
        commonsMultipartResolver.setMaxUploadSize(totalMax);
        try
        {
            commonsMultipartResolver.setUploadTempDir(new FileSystemResource(filepath));
        }
        catch (Exception ex)
        {
            ex.printStackTrace();
        }
        return commonsMultipartResolver;
    }
```

​		在Spring  MVC中，对于MultipartResolver解析的调度是通过DispatcherServlet进行的。首先判断请求是否是一种 enctype="multipart/*"请求，如果是并且存在一个名称为multipartResolver的Bean定义，那么它将会把HttpServletRequest请求转换为MultipartHttpServletRequest请求对象。MultipartHttpServletRequest是一个Spring  MVC自定义的接口，它扩展了HttpServletRequest和关于文件的操作接口MultipartRequest。同样的，实现MultipartHttpServletRequest接口的是一个抽象的类，它就是AbstractMultipartHttpServletRequest，它提供了一个公共的实现，在这个类的基础上，根据MultipartResolver的不同，派生出StandardMultipartHttpServletRequest和DefaultMultipartHttpServletRequest，代表可以根据实现方式的不同进行选择。

![](image/QQ截图20191112190846.png)

### 2、提交上传文件表单

```html
<!-- 必须加上enctype="multipart/form-data"，否则会解析失败 -->
<form id="formtest" action="<%=baseUrl%>/file/uploadMultipartFile.do" enctype="multipart/form-data" method="post">
    <input type="file" name="file" value="选择上传的文件" />
    <input type="submit" value="上传" />
</form>
```

```java
	@RequestMapping("/uploadMultipartFile")
    @ResponseBody
    public Map uploadMultipartFile(MultipartFile file)
    {
        Map<String,Object> result = new HashMap<>();
        String fileName = file.getOriginalFilename();
        file.getContentType();
        File file2 = new File(fileName);
        try
        {
            file.transferTo(file2);
            result.put("success",true);
            result.put("msg","上传成功");
        }
        catch (Exception ex)
        {
            result.put("success",false);
            result.put("msg","上传失败");
            ex.printStackTrace();
        }
        return result;
    }
```

## 11、Spring  MVC的数据转换和格式化

​		当一个请求到达DispatcherServlet的时候，需要找到对应的HandlerMapping，然后根据HandlerMapping去找到对应的HandlerAdapter执行处理器，处理器在要调用的控制器之前，需要先获取HTTP发送过来的信息，然后将其转变为控制的各种不同类型的参数，这就是各类注解能够得到丰富类型参数的原因。

​		它首先用HTTP的消息转换器对消息转换，但这是一个比较原始的转换，它是String类型和文件类型比较简易的转换，还需哟啊进一步转换才能转换为POJO或者其他丰富的参数类型。

​		当处理器处理完了这些参数的转换，它就会进行验证。完成了这些内容，下一步就是调用开发者所提供的控制器了，将之前转换成功的参数传递进去，这样控制器就能够得到丰富的Java类型的支持。，进而完成控制器的逻辑，控制器完成了对应的逻辑，返回结果后，处理如果可以找到对应结果类型的HttpMessageConverter的实现类，它就会调用对应的HttpMessageConverter的实现类方法。对控制器返回的结果进行HTTP转换，这一步不是必须的，可以转化的前提是能够找到对应的转换器，做完这些处理的功能就完成了。

​		接下来就是关于视图解析和视图解析器。

![](image/QQ截图20191121203246.png)

​		对于Spring  MVC，在XML中配置了 <mvc:annotation-driven />，或者Java配置的注解上加入@EnableWebMvc的时候，Spring  IoC容器会自定义生成一个关于转换器和格式化器的类实例——FormattingConversionServiceFactoryBean，这样就可以从Spring  IoC容器中获取这个对象了，其是一个工厂，产品主要就是DefaultFormattingConversionService类对象，类对象继承了一些类，并实现了许多接口。

![](image/QQ截图20191121203953.png)

​		在Java类型转换之前，在Spring  MVC中，为了应对HTTP请求，还定义了HttpMessageConverter，它是一个总体的接口，通过它可以读入HTTP的请求内容，也就是说，在读取HTTP请求的参数和内容的时候会先用HttpMessageConverter读出，做一次简单转换为Java类型，主要是字符串，然后就可以使用各类转换器进行转换了，在逻辑业务处理完成后，还可以通过它把数据转换为响应给用户的内容。

​		转换器：两大类，它们都可以使用注册机注册。

​				1、由Converter接口所定义的。

​				2、GenericConverter。

### 11.1、HttpMessageConverter和JSON消息转换器

​		HttpMessageConverter是定义从HTTP接收请求信息和应答给用户的。

```java
public interface HttpMessageConverter<T> {
    //判断类型是否可读，var1是类别，var2是HTTP类型
    boolean canRead(Class<?> var1, MediaType var2);

    //判断类型是否可写，var1是类别，var2是HTTP类型
    boolean canWrite(Class<?> var1, MediaType var2);

    //MediaType是HTTP类型
    List<MediaType> getSupportedMediaTypes();

    //读取数据类型，进行转换，var1是类，var2是HTTP请求消息
    T read(Class<? extends T> var1, HttpInputMessage var2) throws IOException, HttpMessageNotReadableException;

    //消息写，var2是HTTP类型，var3是HTTP的应答消息
    void write(T var1, MediaType var2, HttpOutputMessage var3) throws IOException, HttpMessageNotWritableException;
}
```

​		其实现中用得最多的是：MappingJackson2HttpMessageConverter

​	![](image/QQ截图20191121205537.png)

​		注册 MappingJackson2HttpMessageConverter：

```java
	@Bean(name = "requestMappingHandlerAdapter")
    public HandlerAdapter initRequestMappingHandlerAdapter()
    {
        //创建RequestMappingHandlerAdapter适配器
        RequestMappingHandlerAdapter rmhd = new RequestMappingHandlerAdapter();
        //HTTP JSON转换器
        MappingJackson2HttpMessageConverter json = new MappingJackson2HttpMessageConverter();
        //http类型支持为JSON类型
        MediaType mediaType = MediaType.APPLICATION_JSON_UTF8;
        List<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(mediaType);
        //键入转换器的支持类型
        json.setSupportedMediaTypes(mediaTypes);
        //往适配器加入JSON转换器
        rmhd.getMessageConverters().add(json);
        return rmhd;
    }
```

```xml
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <ref bean="jsonConverter" />
            </list>
        </property>
    </bean>
    <bean id="jsonConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
        <property name="supportedMediaTypes">
            <list>
                <value>APPLICATION_JSON_UTF8</value>
            </list>
        </property>
    </bean>
```

​		注册后，当在控制器中加入注解@ResponseBody的时候，Spring  MVC便会将应带请求转变为关于JSON的类型，这样的一次转换，就意味着处理器会在控制器返回结果后，遍历其项目中定义的各类HttpMessageConverter实现类，由于MappingJackson2HttpMessageConverter定义为支持JSON数据的转换，它和@ResponseBody所定义的响应类型一致，因此Spring  MVC会将其转换为JSON数据集，此时控制器返回的ModelAndView为null，也就没有后面的视图渲染的过程了。

### 11.2、一对一转换器（Converter）

```java
/**
* 转换器接口，
* S：源类型
* T：目标类型
*/
public interface Converter<S, T> {
    T convert(S var1);
}
```

​		部分转换器：

![](image/QQ截图20191121211621.png)

​		自定义转换器：（转换器未生效，问题未解决）

```java
public class StringToRoleConverter implements Converter<String, Role>
{

    @Override
    public Role convert(String str)
    {
        if()
        {
            StringUtils.isEmpty(str)
            {
                return null;
            }
            if(str.indexOf("-") == -1)
            {
                return null;
            }
            String[] arr = str.split("-");
            if(arr.length != 3)
            {
                return null;
            }
            Role role = new Role();
            role.setId(Integer.parseInt(arr[0]));
            role.setRoleName(arr[1]);
            role.setNote(arr[2]);
            return role;
        }
    }
}
```

​		此时还不够，还需要注册：

```java
	private List<Converter> styleConverter = null;

	//需要通过这个工厂类来进行注册
	@Autowired
	private FormattingConversionServiceFactoryBean fcsfb = null;

    @Bean("styleConverter")
    public List<Converter> initStyleConverter()
    {
        if(styleConverter == null)
        {
            styleConverter = new ArrayList<>();
        }
        Converter roleConverter = new StringToRoleConverter();
        styleConverter.add(roleConverter);
        fcsfb.getObject().addConverter(roleConverter);
        return styleConverter;
    }
```

```xml
<!-- 指定转换服务类 -->
<mvc:annotation-driven conversion-service="conversionService" />
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.shanji.converter.StringToRoleConverter" />
            </list>
        </property>
    </bean>
```

### 11.3、数组和集合转换器 GenericConverter

​		Conver存在弊端：只能从一种类型转换成另一种类型，不能进行一对多的转换。

​		为了解决这个问题，Spring  Core项目加入了另一个转换器结构GenericConverter，它能够满足数组和集合转换的要求。

```java
public interface GenericConverter {
    //返回可接受的转换类型
    Set<GenericConverter.ConvertiblePair> getConvertibleTypes();

    //转换方法
    Object convert(Object var1, TypeDescriptor var2, TypeDescriptor var3);

    //可转换匹配类
    public static final class ConvertiblePair {
        //源类型
        private final Class<?> sourceType;
        //目标类型
        private final Class<?> targetType;

        public ConvertiblePair(Class<?> sourceType, Class<?> targetType) {
            Assert.notNull(sourceType, "Source type must not be null");
            Assert.notNull(targetType, "Target type must not be null");
            this.sourceType = sourceType;
            this.targetType = targetType;
        }

        public Class<?> getSourceType() {
            return this.sourceType;
        }

        public Class<?> getTargetType() {
            return this.targetType;
        }

        public boolean equals(Object other) {
            if (this == other) {
                return true;
            } else if (other != null && other.getClass() == GenericConverter.ConvertiblePair.class) {
                GenericConverter.ConvertiblePair otherPair = (GenericConverter.ConvertiblePair)other;
                return this.sourceType == otherPair.sourceType && this.targetType == otherPair.targetType;
            } else {
                return false;
            }
        }

        public int hashCode() {
            return this.sourceType.hashCode() * 31 + this.targetType.hashCode();
        }

        public String toString() {
            return this.sourceType.getName() + " -> " + this.targetType.getName();
        }
    }
}
```

​		为了进行类型匹配判断，还定义了另一个接口：ConditionalConverter

```java
public interface ConditionalConverter {
    //var1:源数据类型，var2目标数据类型，如果返回true，才进行下一步转换
    boolean matches(TypeDescriptor var1, TypeDescriptor var2);
}
```

​		为了整合原有的接口GenericConverter有了一个新的接口：ConditionalGenericConverter，它是最常用的集合转换器接口。由于继承了GenericConverter, ConditionalConverter，所以它既能判断，又能转换。

```java
public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```

​		基于此接口，Spring  Core给出了许多实现类，这些实现类都会注册到ConversionService对象中，通过ConditionalConverter的matches进行匹配，如果可以匹配，则会调用Conver方法进行转换，它能够提供各种对数组和集合的转换。

![](image/QQ截图20191121233245.png)

​		StringToArrayConverter源码：

```java
final class StringToArrayConverter implements ConditionalGenericConverter {
    //转换服务类
    private final ConversionService conversionService;

    public StringToArrayConverter(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    //可接收的类型
    public Set<ConvertiblePair> getConvertibleTypes() {
        //确定转换类型
        return Collections.singleton(new ConvertiblePair(String.class, Object[].class));
    }

    //查找是否存在Converter支持转换，如果不使用系统的，那么需要自己注册
    public boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType) {
        return this.conversionService.canConvert(sourceType, targetType.getElementTypeDescriptor());
    }

    //转换方法
    public Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
        if (source == null) {
            return null;
        } else {
            //源数据
            String string = (String)source;
            //逗号分隔字符串
            String[] fields = StringUtils.commaDelimitedListToStringArray(string);
            //转换目标
            Object target = Array.newInstance(targetType.getElementTypeDescriptor().getType(), fields.length);

            //转换数组
            for(int i = 0; i < fields.length; ++i) {
                String sourceElement = fields[i];
                //使用conversionService做类型转换，要求我们使用一个自定义或者Spring  Core的Converter
                Object targetElement = this.conversionService.convert(sourceElement.trim(), sourceType, targetType.getElementTypeDescriptor());
                Array.set(target, i, targetElement);
            }

            return target;
        }
    }
}
```

### 11.4、格式化器

​		为了支持数据格式化，Spring  Context提供了相关的Formatter。

![](image/QQ截图20191121234619.png)

​		通过print方法能将结果按照一定的格式输出字符串，通过parse方法能够将满足一定格式的字符串转换为对象。它的内部实际是委托给Converter机制去实现的。

​		@DateTimeFormat：标注日期参数。

​		@NumberFormar：标注数字参数。

```java
<form action="<%=baseUrl%>/params/format.do">
        <table>
            <tr>
                <td>日期</td>
                <td><input type="text" value="2019-11-21" name="date" /></td>
            </tr>
            <tr>
                <td>金额</td>
                <td><input type="text" value="123,000.00" name="amount" /></td>
            </tr>
            <tr>
                <td></td>
                <td align="right"><input type="submit" value="提交" /></td>
            </tr>
        </table>
</form>
```

```java
	@RequestMapping("/format")
    @ResponseBody
    public Map<String,Object> format(@RequestParam("date") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)Date date, @RequestParam("amount") @NumberFormat(pattern = "#,###.##")Double amont)
    {
        Map<String,Object> result = new HashMap<>();
        result.put("date",date);
        result.put("amount",amont);
        return result;
    }
```

## 12、为控制器添加通知

​		与Spring  AOP一样，Spring  MVC也能够给控制器加入通知。涉及4个注解：

​				@ControllerAdvice：主要作用于类，用以标注全局性的控制器的拦截器，它将应用与对应的控制器。

​				@InitBinder：一个允许构建POJO参数的方法，允许在构造控制器参数的时候，加入一定的自定义控		制。

​				@ExceptionHandler：通过它可以注册一个控制器异常，使用当控制器发生异常时，就会跳转到该方法		上。

​				@ModelAttribute：一种针对于数据模型的注解，它选育控制器方法运行，当标注方法返回对象时，它		会保存到数据模型中。

```java
@ControllerAdvice(basePackages = {"com.shanji.advice"})
public class CommonControllerAdvice
{
    @InitBinder
    public void InitBinder(WebDataBinder binder)
    {
        //针对日期类型的格式化，其中CustomDateEditor是客户自定义编辑器
        binder.registerCustomEditor(Date.class,new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"),false));
    }

    //处理数据模型，如果返回对象，则该对象会保存在数据模型中
    @ModelAttribute
    public void populateModel(Model model)
    {
        model.addAttribute("style","小杉杉");
    }

    //此返回值，在控制器抛出异常时，被当做逻辑视图名
    @ExceptionHandler(Exception.class)
    public String exception()
    {
        return "exception";
    }
}
```

```java
@Controller
@RequestMapping("/advice")
public class AdviceController
{
    @RequestMapping("/test")
    @ResponseBody
    public Map<String,Object> testAdvice(Date date, @NumberFormat(pattern = "##,###.00")BigDecimal amount, Model model)
    {
        Map<String,Object> result = new HashMap<>();
        result.put("style",model.asMap().get("style"));
        result.put("date", DateUtils.formatDate(date,"yyyy-MM-dd"));
        result.put("amount",amount);
        return result;
    }

    @RequestMapping("/exception")
    public void exception()
    {
        throw new RuntimeException("测试异常跳转");
    }
}
```

​		控制器也可以使用除@ControllerAdvice以外的其余的三个注解。此时如果控制器不在@ControllerAdvice标注的包下，那么它只对当前控制器有效。

​		@ModelAttribute：另外一个作用，在同一个控制器中，为不同方法传值。

```java
	@ModelAttribute("role")
    public Role initRole(@RequestParam(value = "id",required = false)Integer id)
    {
        if(id == null || id < 1)
        {
            return null;
        }
        Role role = roleService.getRole(id);
        return role;
    }

    @RequestMapping("/getRoleFromModelAttribute")
    @ResponseBody
    public Role getRoleFromModelAttribute(@ModelAttribute("role")Role role)
    {
        return role;
    }
```

## 13、处理异常

​		除了@ExceptionHandler可以处理异常，Spring会将自身产生的异常转换成合适的状态码。

![](image/QQ截图20191124002215.png)

```java
public enum HttpStatus {
    CONTINUE(100, "Continue"),
    SWITCHING_PROTOCOLS(101, "Switching Protocols"),
    PROCESSING(102, "Processing"),
    CHECKPOINT(103, "Checkpoint"),
    OK(200, "OK"),
    CREATED(201, "Created"),
    ACCEPTED(202, "Accepted"),
    NON_AUTHORITATIVE_INFORMATION(203, "Non-Authoritative Information"),
    NO_CONTENT(204, "No Content"),
    RESET_CONTENT(205, "Reset Content"),
    PARTIAL_CONTENT(206, "Partial Content"),
    MULTI_STATUS(207, "Multi-Status"),
    ALREADY_REPORTED(208, "Already Reported"),
    IM_USED(226, "IM Used"),
    MULTIPLE_CHOICES(300, "Multiple Choices"),
    MOVED_PERMANENTLY(301, "Moved Permanently"),
    FOUND(302, "Found"),
    /** @deprecated */
    @Deprecated
    MOVED_TEMPORARILY(302, "Moved Temporarily"),
    SEE_OTHER(303, "See Other"),
    NOT_MODIFIED(304, "Not Modified"),
    /** @deprecated */
    @Deprecated
    USE_PROXY(305, "Use Proxy"),
    TEMPORARY_REDIRECT(307, "Temporary Redirect"),
    PERMANENT_REDIRECT(308, "Permanent Redirect"),
    BAD_REQUEST(400, "Bad Request"),
    UNAUTHORIZED(401, "Unauthorized"),
    PAYMENT_REQUIRED(402, "Payment Required"),
    FORBIDDEN(403, "Forbidden"),
    NOT_FOUND(404, "Not Found"),
    METHOD_NOT_ALLOWED(405, "Method Not Allowed"),
    NOT_ACCEPTABLE(406, "Not Acceptable"),
    PROXY_AUTHENTICATION_REQUIRED(407, "Proxy Authentication Required"),
    REQUEST_TIMEOUT(408, "Request Timeout"),
    CONFLICT(409, "Conflict"),
    GONE(410, "Gone"),
    LENGTH_REQUIRED(411, "Length Required"),
    PRECONDITION_FAILED(412, "Precondition Failed"),
    PAYLOAD_TOO_LARGE(413, "Payload Too Large"),
    /** @deprecated */
    @Deprecated
    REQUEST_ENTITY_TOO_LARGE(413, "Request Entity Too Large"),
    URI_TOO_LONG(414, "URI Too Long"),
    /** @deprecated */
    @Deprecated
    REQUEST_URI_TOO_LONG(414, "Request-URI Too Long"),
    UNSUPPORTED_MEDIA_TYPE(415, "Unsupported Media Type"),
    REQUESTED_RANGE_NOT_SATISFIABLE(416, "Requested range not satisfiable"),
    EXPECTATION_FAILED(417, "Expectation Failed"),
    I_AM_A_TEAPOT(418, "I'm a teapot"),
    /** @deprecated */
    @Deprecated
    INSUFFICIENT_SPACE_ON_RESOURCE(419, "Insufficient Space On Resource"),
    /** @deprecated */
    @Deprecated
    METHOD_FAILURE(420, "Method Failure"),
    /** @deprecated */
    @Deprecated
    DESTINATION_LOCKED(421, "Destination Locked"),
    UNPROCESSABLE_ENTITY(422, "Unprocessable Entity"),
    LOCKED(423, "Locked"),
    FAILED_DEPENDENCY(424, "Failed Dependency"),
    UPGRADE_REQUIRED(426, "Upgrade Required"),
    PRECONDITION_REQUIRED(428, "Precondition Required"),
    TOO_MANY_REQUESTS(429, "Too Many Requests"),
    REQUEST_HEADER_FIELDS_TOO_LARGE(431, "Request Header Fields Too Large"),
    UNAVAILABLE_FOR_LEGAL_REASONS(451, "Unavailable For Legal Reasons"),
    INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
    NOT_IMPLEMENTED(501, "Not Implemented"),
    BAD_GATEWAY(502, "Bad Gateway"),
    SERVICE_UNAVAILABLE(503, "Service Unavailable"),
    GATEWAY_TIMEOUT(504, "Gateway Timeout"),
    HTTP_VERSION_NOT_SUPPORTED(505, "HTTP Version not supported"),
    VARIANT_ALSO_NEGOTIATES(506, "Variant Also Negotiates"),
    INSUFFICIENT_STORAGE(507, "Insufficient Storage"),
    LOOP_DETECTED(508, "Loop Detected"),
    BANDWIDTH_LIMIT_EXCEEDED(509, "Bandwidth Limit Exceeded"),
    NOT_EXTENDED(510, "Not Extended"),
    NETWORK_AUTHENTICATION_REQUIRED(511, "Network Authentication Required");
    ......
}
```

​		新增一个自定义异常映射码：

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND,reason = "找不到角色信息异常")
public class RoleException extends RuntimeException
{
    private static final long serialVersionUID = 6602340062281161155L;
}
```

```java
	@ModelAttribute("role")
    public Role initRole(@RequestParam(value = "id",required = false)Integer id)
    {
        if(id == null || id < 1)
        {
            return null;
        }
        Role role = roleService.getRole(id);
        if(role == null)
        {
            throw new RoleException();
        }
        return role;
    }

    @ExceptionHandler(RoleException.class)
    public String HandlerRoleException(RoleException e)
    {
        return "index";
    }
```

## 14、国际化

​		DispatcherServlet会解析一个LocaleResover接口对象，通过它来决定用户区域，读出对应用户系统设定的语言或者用户选择的语言，以确定国际化。但对于DispatcherServlet而言，只能够注册一个LocaleResover接口对象。

![](image/QQ截图20191124105719.png)

​		LocaleResolver主要作用是实现解析国际化，此外还会解析内容的问题，尤其是时区。所以在LocaleResolver的基础上，Spring 还扩展了LocaleContextResolver，它还能处理一些用户区域上的问题，包括语言和时区的问题。CookieLocalResolver主要是使用浏览的Cookie实现国际化的，而Cookie有时候需要服务器去写入到浏览器中，所以它会继承一个产生Cookie的类CookieGenerator。FixedLocaleResolver和SessionLocaleResolver有公共的方法，所以被抽象为AbstractLocaleContextResolver。它是一个能够提供语言和时区的抽象类，而它的语言功能则继承了AbstractLocaleResolver，而时区的实现则扩展了LocaleContextResolver接口。

​		1、AcceptHeaderLocaleResolver：Spring默认的区域解析器，它功过检验HTTP请求的accept-language头部来解析 区域。这个头部是由用户的web浏览器根据底层操作系统的区域设置进行设定。注意，这个区域解析器无法改变用户的区域，因为它无法修改用户操作系统的区域设置。

​		2、FixedLocaleResolver：使用固定Locale国际化，不可修改Locale。

​		3、CookieLocaleResolver：根据Cookie数据获取国际化数据，由于用户禁止Cookie或者没有设置，如果是这样，它会金桔accept-language HTTP头部来确定默认区域。

​		4、SeesionLocaleResolver：根据Session进行国际化，也就是根据用户Session的变量读取区域设置，可变，如果Session也没有设置，那么它也会使用开发者设置默认的值。

​		为了修改国际化，Spring  MVC还提供了一个国际化的拦截器——LocaleChangeInterceptor，通过它可以获取参数，然后既可以通过CookieLocaleResolver使用浏览器的Cookie来实现国际化，也可以用SessionLocaleResolver通过服务器的Session来实现国际化。

### 14.1、MessageSource接口

​		MessageSource接口是Spring  MVC为了加载消息所设置的接口，通过它来加载对应的国际化属性文件。

![](image/QQ截图20191124111659.png)

​		StaticMessageSource：静态消息源。

​		DelegatingMessageSource：实现的是一个代理的功能。

​		ResourceBundleMessageSource：使用的是JDK提供的ResourceBundle，它只能把文件放在对应的类路径下，不具备热加载的功能，也就是需要重启系统才能重新加载它。

​		ReloadableResourceBundleMessageSource：可以把属性文件放置在任何地方，可以在系统不重新启动的情况下重新加载属性文件，这样就可以在系统运行期修改对应的国际化文件，重新定制国际化的内容。

```xml
<!-- id="messageSource",Spring  MVC默认的名称，不能随意修改。basename可以传递一个文件名(带路径从classpath算起)的前缀，后缀通过Locale来确定 -->
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource" p:defaultEncoding="UTF-8" p:basename="msg" />

<!-- cacheSeconds：每隔相应的设置时间，探测属性文件的最后修改时间，如果被修改过则会重新加载，默认值为-1，表示永远缓存，系统运行期间不在修改，如果设置为0，每次访问国际化文件时都会探测属性文件的最后修改时间
	basename：可以是多个属性文件名组成。 
-->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource" p:defaultEncoding="UTF-8" p:basename="classpath:msg" p:cacheSeconds="3600" />
```

### 14.2、CookieLocaleResolver和SessionLocaleResolver

​		CookieLocaleResolver：创建时，需要设置两个属性。

​				cookieName：一个cookie变量的名称。

​				maxAge：cookie超时时间，单位为秒。

```xml
<!-- localeResolver:不能修改，此时设置了默认为中文简体，当cookie值无效时，就会使用简体中文 -->
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver" p:cookieName="lang" p:cookieMaxAge="1800" p:defaultLocale="zh_CN" />
```

​		SessionLocaleResolver：

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" p:defaultLocale="zh_CN" />
```

​		SessionLocaleResolver定义了两个静态公共常量：可以通过控制器读取 

​				1、LOCALE_SESSION_ATTRIBUTE_NAME：Session的Locale的键。

​				2、TIME_ZONE_SESSION_ATTRIBUTE_NAME：时区。

### 14.3、国际化拦截器

​		通过请求参数去改变国际化的值，可以使用Spring提供的拦截器LocaleChangeInterceptor，它继承了HandlerInterceptorAdapter，通过覆盖它的preHandle方法，然后使用系统所配置的LocaleResolver实现国际化。

```xml
<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**/**"/>
            <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
                <property name="paramName" value="language" />
            </bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

​		当请求到来，首先拦截器会监控有无language参数，有则获取它，然后通过使用系统所配置的LocaleResolver实现国际化。注意：获取不到参数，或者获取的参数的国际化并非系统能够支持，那么此时会采用默认值。

### 14.4、开发

```jsp
<%@ taglib prefix="s" uri="http://www.springframework.org/tags" %>
<%--
  Created by IntelliJ IDEA.
  User: 小杉杉
  Date: 2019/11/24
  Time: 12:10
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h2>
        <s:message code="welcome" />
    </h2>
Locale：${pageContext.response.locale}
</body>
</html>
```

# 六、Redis

​		一般而言Redis在 Java  Web应用中存在两个主要的场景：

​				1、缓存常用的数据。

​				2、在需要高速读/写的场合使用它快速读/写。

![](image/QQ截图20191125210119.png)

![](image/QQ截图20191125210227.png)

​		如果业务数据写次数远大于读次数没有必要使用Redis，如果是读次数远大于写次数，则使用Redis就有其价值了，因为写入Redis虽然要消耗一定的代价，但是其性能良好，相对数据库而言，几乎可以忽略不急。

![](image/QQ截图20191125210635.png)

​		当一个请求到达服务器，只是把业务数据先在Redis读/写，而没有进行任何对数据库的操作，系统仅仅是操作Redis缓存，而没有操作数据库，这个速度就比操作数据库要快得多，从而达到需要高速响应的效果，但是一般缓存不能持久化，或者所持久化的数据不太规范，因此需要吧这些数据存入数据库，所以在一个请求操作完Redis的读/写后，会去判断该高速读/写的业务是否结束。

​		简单使用：

```java
Jedis jedis = new Jedis("192.168.58.129",6379);
        jedis.auth("179980");
        int i = 0;
        try
        {
            long start = System.currentTimeMillis();
            while (true)
            {
                long end = System.currentTimeMillis();
                if(end - start >= 1000)
                {
                    break;
                }
                i++;
                jedis.set("test" + i,i + "");
            }
        }
        finally {
            jedis.close();
        }
        System.out.println(i);
```

​		Java  Redis的连接池提供了类 JedisPool用来创建Redis连接池对象。使用这个对象，需要使用 JedisPoolConfig对连接池进行配置。

```java
JedisPoolConfig poolConfig = new JedisPoolConfig();
        //最大空闲数
        poolConfig.setMaxIdle(50);
        //最大连接数
        poolConfig.setMaxTotal(100);
        //最大等待毫秒数
        poolConfig.setMaxWaitMillis(20000);
        //使用配置创建连接池
        JedisPool pool = new JedisPool(poolConfig,"192.138.58.129",6379);
        //从连接池中连接单个连接
        Jedis resource = pool.getResource();
        resource.auth("179980");
```

​		Spring中使用Redis：

​			Redis只能提供基于字符串型的操作。

```xml
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig" p:maxIdle="50" p:maxTotal="100" p:maxWaitMillis="20000" />
```

​		在使用Spring提供的RedisTemplate之前需要配置Spring所提供的连接工厂，在Spring Data Redis提供了工厂模型。

​				1、JedisConnectionFactory

​				2、LettuceConnectionFactory

​		配置 JedisConnectionFactory：

```xml
<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" p:hostName="192.138.58.129" p:port="6379" p:password="179980" c:poolConfig-ref="poolConfig" />
```

​		普通的连接使用没有办法把 Java对象直接存入Redis，而需要我们自己提供方案，这时往往就是将对象序列化，然后使用Reids进行存储，而取回序列化的内容后，在通过转换变为 Java对象，Spring中提供了封装的方案，在其内部提供了RedisSerializer接口和一些实现类。

![](image/QQ截图20191125220003.png)

​		Spring提供了实现RedisSerializer接口的序列化器：

​				1、GenericJackson2JsonRedisSerializer：通用的使用 Json2.jar的包，将Redis对象的序列化器。

​				2、Jackson2JsonRedisSerializer：通过 Jackson2.jar包提供的序列化进行转换。

​				3、JdkSerializationRedisSerializer：使用JDK的序列化器进行转化。

​				4、OxmSerializer：使用Spring  O/X对象Object和XML相互转化。

​				5、StringRedisSerializer：使用字符串进行序列化。

​				6、GenericToStringSerializer：通过通用的字符串序列化进行相互转换。

​		使用他们就能够将对象通过系列化存储到Redis中，也可以把Redis存储的内容转换为 Java对象，为此Spring提供的RedisTemplate还有两个属性：

​				1、keySerializer——键序列器

​				2、valueSerializer——值序列器

```xml
<bean id="jdkSerializationRedisSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
    <bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="valueSerializer" ref="stringRedisSerializer" />
    </bean>
```

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate bean = applicationContext.getBean(RedisTemplate.class);
        Role role = new Role();
        role.setId(1);
        role.setRoleName("小杉杉");
        role.setNote("测试");
        bean.opsForValue().set("role_1",role);
        Role role_1 = (Role) bean.opsForValue().get("role_1");
        System.out.println(role_1.getRoleName());
```

​		此时并不能保证每次使用RedisTemplate是操作同一个对Redis的连接。可能来自于同一个Redis连接池 的不同Redis的连接。为了使得所有的操作都来自于同一个连接。可以使用SessionCallback或者RedisCallback这两个接口，而RedisCallback是比较底层的封装，使用不是很友好，所以更到使用SessionCallback，通过这个接口就可以把多个命令放入到同一个Redis连接中去执行。

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate bean = applicationContext.getBean(RedisTemplate.class);
        Role role = new Role();
        role.setId(1);
        role.setRoleName("小杉杉");
        role.setNote("测试");
        SessionCallback callback = new SessionCallback<Role>() {
            @Override
            public Role execute(RedisOperations redisOperations) throws DataAccessException {
                redisOperations.boundValueOps("role_1").set(role);
                return (Role) redisOperations.boundValueOps("role_1").get();
            }
        };
        Role execute = (Role) bean.execute(callback);
        System.out.println(execute.getRoleName());
```

## 1、Redis的6中数据类型

​		Redis是一种基于内存的数据库，并且提供了一定的持久化能力，它是一种键值数据库，使用key作为索引找到当前缓存的数据，并且返回给程序调用者。

​		6种类型：

​				1、字符串（String）

​				2、列表（List）

​				3、集合（set）

​				4、哈希结构（hash）

​				5、有序集合（zset）

​				6、基数（HyperLogLog）

![](image/QQ截图20191125230653.png)

## 2、常用命令

### 1、字符串

​		字符串是Redis最基本的数据结构，它将一个键和值存储与Redis内部，犹如 Java 的 Map 结构，让Redis通过键去找到值。

![](image/QQ截图20191127222540.png)

![](image/QQ截图20191127222643.png)

![](image/QQ截图20191127222838.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        redisTemplate.opsForValue().set("key_one","value_one");
        redisTemplate.opsForValue().set("key_two","value_two");
        String value_one = (String) redisTemplate.opsForValue().get("key_one");
        System.out.println(value_one);
        redisTemplate.delete("key_one");
        Long length = redisTemplate.opsForValue().size("key_two");
        System.out.println(length);
        String oldValueTwo = (String)redisTemplate.opsForValue().getAndSet("key_two", "xiaoshanshan");
        System.out.println(oldValueTwo);
        String value_two = (String) redisTemplate.opsForValue().get("key_two");
        System.out.println(value_two);
        Integer key_two1 = redisTemplate.opsForValue().append("key_two", " is superman");
        System.out.println(key_two1);
        String value_two_append = (String) redisTemplate.opsForValue().get("key_two");
        System.out.println(value_two_append);
```

​		除此之外，Redis还对整数和浮点数的功能，如果字符串是数字，Redis还能支持简单的运算，目前版本只支持加法和减法。

![](image/QQ截图20191127231630.png)

![](image/QQ截图20191127231835.png)

​		值得注意的是：此时键和值得序列化器都是字符串序列化器，所以Redis保存的是字符串，如果采用其他的序列化器，Redis将不会按照原字符串保存，在进行简单运算时会有异常。并且所有的减法，原有值必须是整数。

```java
public static void main(String[] args)
    {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        redisTemplate.opsForValue().set("i","9");
        printCurrValue(redisTemplate,"i");
 
 //因为Redis不支持减法运算，所以需要用此方法转换
    redisTemplate.getConnectionFactory().getConnection().decr(redisTemplate.getKeySerializer().serialize("i"));
        printCurrValue(redisTemplate,"i");
        redisTemplate.getConnectionFactory().getConnection().decrBy(redisTemplate.getKeySerializer().serialize("i"),2);
        printCurrValue(redisTemplate,"i");
        redisTemplate.opsForValue().increment("i",2.3);
        printCurrValue(redisTemplate,"i");
    }

    public static void printCurrValue(RedisTemplate redisTemplate,String key)
    {
        Object o = redisTemplate.opsForValue().get(key);
        System.out.println(o);
    }
```

### 2、哈希

​		哈希结构如同 Java中的 map 一样，一个对象里有许多键值对，特别适合存储对象。

​		在Redis中，hash是一个String类型的field和value的映射表，因为存储的数据实际在Redis内存中都是一个个字符串而已。

![](image/QQ截图20191127234029.png)

​		其中，role_1代表这个hash结构在Redis内存中的key，通过它可以找到这个hash结构，而hash结构由一系列的field和value组成。

![](image/QQ截图20191127234322.png)

​		hash的键值对在内存中是一种无序的状态，hash结构命令：

![](image/QQ截图20191127234437.png)

​		Redis需要通过key索引到对应的hash结构，在通过field来确定使用hash结构的哪个键值对。

​		哈希结构的大小：如果哈希结构是个很大的键值对，那么hkeys，hgetall，hvals等返回所有哈希结构数据的命令，会造成大量数据的读取。

​		对于数字的操作命令hincrby而言，要求存储的也是整数型的字符串，对于hincrbyfloat而言，则要求使用浮点数或者整数，否则命令会失败。

![](image/QQ截图20191127235253.png)

```xml
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="defaultSerializer" ref="stringRedisSerializer" />
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="valueSerializer" ref="stringRedisSerializer" />
    </bean>
```

​		Spring对hash结构的操作中会设计map等其他类的操作，需要明确它的规则，这里只是指定默认的序列化器，如果想为hash结构指定序列化器，可以使用RedisTemplate的hashKeySerializer和hashValueSerializer两个属性，来为hash结构的field和value指定序列化器。

```java
public static void main(String[] args)
    {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        String key = "hash";
        Map<String,String> map = new HashMap<>();
        map.put("f1","value1");
        map.put("f2","value2");

        //相当于hmset命令
        redisTemplate.opsForHash().putAll(key,map);

        //相当于hset命令
        redisTemplate.opsForHash().put(key,"f3","6");
        printValueForHash(redisTemplate,key,"f3");

        //相当于hexists key field命令
        boolean exists = redisTemplate.opsForHash().hasKey(key, "f3");
        System.out.println(exists);

        //相当于hgetall命令
        Map keyValueMap = redisTemplate.opsForHash().entries(key);

        //相当于hincrby命令
        redisTemplate.opsForHash().increment(key,"f3",2);
        printValueForHash(redisTemplate,key,"f3");

        //相当于hincrbyfloat命令
        redisTemplate.opsForHash().increment(key,"f3",0.88);
        printValueForHash(redisTemplate,key,"f3");

        //相当于hvals命令
        List values = redisTemplate.opsForHash().values(key);

        //相当于hkeys命令
        Set keys = redisTemplate.opsForHash().keys(key);
        ArrayList<String> fieldList = new ArrayList<>();
        fieldList.add("f1");
        fieldList.add("f2");

        //相当于hmget命令
        List list = redisTemplate.opsForHash().multiGet(key, keys);

        //相当于hsetnx命令
        Boolean aBoolean = redisTemplate.opsForHash().putIfAbsent(key, "f4", "value4");
        System.out.println(aBoolean);

        //相当于hdel命令
        Long delete = redisTemplate.opsForHash().delete(key, "f1", "f2");
        System.out.println(delete);


    }

    private static void printValueForHash(RedisTemplate redisTemplate,String key,String field)
    {
        System.out.println(redisTemplate.opsForHash().get(key,field));
    }
```

​		hmset命令：在 Java的API中，是使用map保存多个键值对在先的。

​		hgetall：返回所有的键值对，并保存在一个map对象中。

​		hincrby和hincrbyfloat：都采用increment方法。

​		redisTemplate.opsForHash().values(key)：相当于hvals命令，返回所有的值，保存到List对象中。

​		redisTemplate.opsForHash().keys(key)：相当于hkeys命令，获取所有的键，保存到一个Set对象中。

### 3、链表

​		链表：存储多个字符串，有序，双向，即可以从左到右，也可以从右到左遍历。

​		与  Java 的链表类似，其优势在与插入删除，而非查询。

​		既然是双向，那么操作就有方向的区别：

![](image/QQ截图20191128002031.png)

​		上述命令是线程不安全的。为了克服这个问题，Redis提供了链表的阻塞命令，运行时，会给链表加锁，以保证链表的命令安全性。

![](image/QQ截图20191128003003.png)

![](image/QQ截图20191128003025.png)

![](image/QQ截图20191128003441.png)

```java
public static void main(String[] args)
    {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        try
        {
            redisTemplate.delete("list");
            redisTemplate.opsForList().leftPush("list","node3");
            ArrayList<String> nodeList = new ArrayList<>();
            for(int i = 2 ; i >= 1 ; i--)
            {
                nodeList.add("node" + i);
            }
            redisTemplate.opsForList().leftPushAll("list",nodeList);
            redisTemplate.opsForList().rightPush("list","node4");

            String list = (String) redisTemplate.opsForList().index("list", 0);

            Long size = redisTemplate.opsForList().size("list");

            String lpop = (String) redisTemplate.opsForList().leftPop("list");

            String rpop = (String) redisTemplate.opsForList().rightPop("list");

            //在节点之前插入
            redisTemplate.getConnectionFactory().getConnection().lInsert("list".getBytes("utf-8"), RedisListCommands.Position.BEFORE,"node2".getBytes("utf-8"),"before_node".getBytes("utf-8"));
            //在节点之后插入
            redisTemplate.getConnectionFactory().getConnection().lInsert("list".getBytes("utf-8"), RedisListCommands.Position.AFTER,"node2".getBytes("utf-8"),"after_node".getBytes("utf-8"));

            redisTemplate.opsForList().leftPushIfPresent("list","head");
            redisTemplate.opsForList().rightPushIfPresent("list","end");

            List list1 = redisTemplate.opsForList().range("list", 0, 10);
            nodeList.clear();
            for(int i =1 ; i <= 3 ;  i++)
            {
                nodeList.add("node");
            }
            redisTemplate.opsForList().leftPushAll("list",nodeList);
            //从左网友删除至多3个node节点
            redisTemplate.opsForList().remove("list",3,"node");
            redisTemplate.opsForList().set("list",0,"new_head_value");
        }
        catch (Exception ex)
        {
            ex.printStackTrace();
        }

        printList(redisTemplate,"list");
    }		

    private static void printList(RedisTemplate redisTemplate,String key)
    {
        Long size = redisTemplate.opsForList().size(key);
        List range = redisTemplate.opsForList().range(key, 0, size);
        System.out.println(range);
    }
```

​		阻塞命令：

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        ArrayList<String> nodeList = new ArrayList<>();
        for(int i = 1 ; i <= 5 ; i++)
        {
            nodeList.add("node" + i);
        }
        redisTemplate.opsForList().leftPushAll("list1",nodeList);
        redisTemplate.opsForList().leftPop("list1",1, TimeUnit.SECONDS);
        redisTemplate.opsForList().rightPop("list1",1, TimeUnit.SECONDS);
        nodeList.clear();
        for(int i = 1 ; i <= 3 ; i++)
        {
            nodeList.add("data" + i);
        }
        redisTemplate.opsForList().leftPushAll("list2",nodeList);
        redisTemplate.opsForList().rightPopAndLeftPush("list1","list2");
        redisTemplate.opsForList().rightPopAndLeftPush("list1","list2",1,TimeUnit.SECONDS);
        printList(redisTemplate,"list1");
        printList(redisTemplate,"list2");
```

### 4、集合

​		Redis的集合不是一个线性结构，而是一个哈希结构，内部会根据hash银子来存储和查找数据，插入，删除和查找的复杂度都是O(1)。注意事项：

​				1、对于集合而言，每一个元素都是不能重复的，当插入相同记录的时候都会失败。

​				2、集合石无序的。

​				3、集合的每一个元素都是String数据结构类型。

![](image/QQ截图20191128224927.png)

![](image/QQ截图20191128225000.png)

![](image/QQ截图20191128225552.png)

![](image/QQ截图20191128225954.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        Set set = null;
        redisTemplate.boundSetOps("set1").add("v1","v2","v3","v4","v5","v6");
        redisTemplate.boundSetOps("set2").add("v0","v2","v4","v6","v8");
        redisTemplate.opsForSet().size("set1");
        set = redisTemplate.opsForSet().difference("set1", "set2");

        set = redisTemplate.opsForSet().intersect("set1","set2");

        Boolean exists = redisTemplate.opsForSet().isMember("set1", "v1");

        set = redisTemplate.opsForSet().members("set1");

        String val = (String)redisTemplate.opsForSet().pop("set1");

        val = (String) redisTemplate.opsForSet().randomMember("set1");

        List list = redisTemplate.opsForSet().randomMembers("set1", 2L);

        redisTemplate.opsForSet().remove("set1","v1");
        redisTemplate.opsForSet().union("set1","set2");
        redisTemplate.opsForSet().differenceAndStore("set1","set2","diff_set");

        redisTemplate.opsForSet().intersectAndStore("set1","set2","inter_set");

        redisTemplate.opsForSet().unionAndStore("set1","set2","union_set");
```

### 5、有序集合

​		有序集合和集合类似，和无序集合的区别在于每一个元素除了只之外，还多了一个分数，分数为浮点数，在 Java中是使用双精度表示的，根据分数，Redis就可以支持对分数从小到大或者从大到小的排序。跟无序集合一样，值不能重复，但是分数可以一样。

![](image/QQ截图20191128231432.png)

![](image/QQ截图20191128231541.png)

​		Spring对Redis有序集合的元素的值和分数的范围和限制进行了封装。

​		TypedTuple，一个内部接口，是ZSetOperations接口的内部接口：

```java
public interface TypedTuple<V> extends Comparable<ZSetOperations.TypedTuple<V>> {
        //获取值
    	V getValue();

    //获取分数
        Double getScore();
    }
```

​		Spring-data-redis提供了一个默认的实现类：DefaultTypedTuple，在默认情况下Spring就会把带有分数的有序集合的值和分数封装到这个类中。

​		Spring不仅对有序集合元素封装，而且对范围也进行了封装，使用RedisZSetCommands下的内部类：Range封装，其有一个静态的range方法，使用它可以生成一个Range对象。

```java
//设置大于等于min
public RedisZSetCommands.Range gte(Object min) {
            Assert.notNull(min, "Min already set for range.");
            this.min = new RedisZSetCommands.Range.Boundary(min, true);
            return this;
        }

		//设置大于min
        public RedisZSetCommands.Range gt(Object min) {
            Assert.notNull(min, "Min already set for range.");
            this.min = new RedisZSetCommands.Range.Boundary(min, false);
            return this;
        }

		//设置小于等于max
        public RedisZSetCommands.Range lte(Object max) {
            Assert.notNull(max, "Max already set for range.");
            this.max = new RedisZSetCommands.Range.Boundary(max, true);
            return this;
        }

		//设置小于max
        public RedisZSetCommands.Range lt(Object max) {
            Assert.notNull(max, "Max already set for range.");
            this.max = new RedisZSetCommands.Range.Boundary(max, false);
            return this;
        }
```

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);

        Set<ZSetOperations.TypedTuple> set1 = new HashSet<>();
        Set<ZSetOperations.TypedTuple> set2 = new HashSet<>();
        int j = 9;
        for(int i = 1 ; i <= 9 ; i++ , j--)
        {
            Double score1 = Double.valueOf(i);
            String value1 = "x" + i;
            Double score2 = Double.valueOf(j);
            String value2 = j % 2 == 0 ? "y" + j : "x" + j;
            ZSetOperations.TypedTuple typedTuple = new DefaultTypedTuple(value1,score1);
            set1.add(typedTuple);
            ZSetOperations.TypedTuple typedTuples = new DefaultTypedTuple(value2,score2);
            set2.add(typedTuples);
        }
        redisTemplate.opsForZSet().add("zset1",set1);
        redisTemplate.opsForZSet().add("zset2",set2);

        Long size = redisTemplate.opsForZSet().zCard("zset1");

        size = redisTemplate.opsForZSet().count("zset1",3,6);

        Set set = null;
        set = redisTemplate.opsForZSet().range("zset1",1,5);
        printSet(set);

        set = redisTemplate.opsForZSet().rangeWithScores("zset1",0,-1);
        printTypedTuple(set);

        size = redisTemplate.opsForZSet().intersectAndStore("zset1","zset2","inter_zset");

        RedisZSetCommands.Range range = RedisZSetCommands.Range.range();
        range.lt("x8");
        range.gt("x1");

        set = redisTemplate.opsForZSet().rangeByLex("zset1",range);
        printSet(set);

        range.lte("x8");
        range.gte("x1");

        set = redisTemplate.opsForZSet().rangeByLex("zset1",range);
        printSet(set);

        RedisZSetCommands.Limit limit = RedisZSetCommands.Limit.limit();
        limit.count(4);
        limit.offset(5);
        set = redisTemplate.opsForZSet().rangeByLex("zset1",range,limit);
        printSet(set);

        Long rank = redisTemplate.opsForZSet().rank("zset1", "x4");
        System.out.println("rank = " + rank);

        size = redisTemplate.opsForZSet().remove("zset1","x5","x6");
        System.out.println("delete = " + size);

        size = redisTemplate.opsForZSet().removeRange("zset2",1,2);
        set = redisTemplate.opsForZSet().rangeWithScores("zset2",0 ,-1);
        printTypedTuple(set);

        size = redisTemplate.opsForZSet().remove("zset2","y5","y3");
        System.out.println(size);

        Double dbl = redisTemplate.opsForZSet().incrementScore("zset1", "x1", 11);

        redisTemplate.opsForZSet().removeRangeByScore("zset1",1,2);

        set = redisTemplate.opsForZSet().reverseRangeWithScores("zset2",1,10);
        printTypedTuple(set);

    }


    private static void printTypedTuple(Set<ZSetOperations.TypedTuple> set)
    {
        if(set == null || set.isEmpty())
        {
            return;
        }
        Iterator<ZSetOperations.TypedTuple> iterator = set.iterator();
        while (iterator.hasNext())
        {
            ZSetOperations.TypedTuple next = iterator.next();
            System.out.println("{value= " + next.getValue() + ", score= " + next.getScore() + "}");
        }
    }

    private static void printSet(Set set)
    {
        if(set == null || set.isEmpty())
        {
            return;
        }
        Iterator iterator = set.iterator();
        while (iterator.hasNext())
        {
            Object next = iterator.next();
            System.out.print(next + "\t");
        }
    }
```

### 6、基数——HyperLogLog

​		基数是一种算法。作用是评估大约需要准备多少个存储单元去存储数据，但是存在误差。

![](image/QQ截图20191129000049.png)

![](image/QQ截图20191129000331.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);

        redisTemplate.opsForHyperLogLog().add("h1","a","b","c","d","a");
        redisTemplate.opsForHyperLogLog().add("h2","a");
        redisTemplate.opsForHyperLogLog().add("h2","z");

        Long h1 = redisTemplate.opsForHyperLogLog().size("h1");
        System.out.println(h1);

        Long h2 = redisTemplate.opsForHyperLogLog().size("h2");
        System.out.println(h2);

        redisTemplate.opsForHyperLogLog().union("des_key","h1","h2");

        Long des_key = redisTemplate.opsForHyperLogLog().size("des_key");
        System.out.println(des_key);
```

## 3、事务

​		为了保证数据的一致性和安全性，Redis提供了事务方案，Redis的事务是使用 MULTI-EXEC的命令组合，使用它提供了两个重要的保证：

​				1、事务是一个被隔离的操作，事务中的方法都会被 Redis 进行序列化并按顺序执行，事务在执行的过程		中不会被其他客户端发生的命令所打断。

​				2、事务是一个原子性的操作，要么全部执行，要么什么都不会执行。

​		使用事务的三个过程：

​				1、开启事务

​				2、命令进入队列

​				3、执行事务

![](image/QQ截图20191129212713.png)

### 1、基础事务

​		Redis中开启事务是multi命令，而执行事务是exec命令。multi到exec命令之间的Redis命令将采取进入队列的形式，直至exec命令的出现，才会一次性发送队列里的命令去执行，而在执行这些命令的时候其他客户端就不能再插入任何命令了。

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);

        SessionCallback callback = new SessionCallback() {
            @Override
            public Object execute(RedisOperations ops) throws DataAccessException {
                ops.multi();
                ops.boundValueOps("key1").set("valu1");
                //此时并会得到返回值，此时所有的命令只是进入了队列，但是未被执行
                Object key1 = ops.boundValueOps("key1").get();
                System.out.println("value = " + key1);

                //执行队列的所有命令
                List exec = ops.exec();
                String key11 = (String) redisTemplate.opsForValue().get("key1");
                return key11;
            }
        };
        Object execute = redisTemplate.execute(callback);
        System.out.println(execute);
```

​		在执行事务命令的时候，在命令入队列时，Redis会检测事务的**命令是否正确**，如果不正确则会产生错误。无论之前和之后的命令都会被事务所回滚，就变为什么都没有执行。

​		当**命令格式正确**，因为数据结构引起的错误，则该命令执行出现错误，而其之前和之后的命令都会被正常执行。

### 2、监控事物

​		Redis中使用watch命令可以决定事务是执行还是回滚。

​		可以在multi命令之前使用watch命令监控某些键值对，然后使用multi命令开始事务，执行各类对数据结构进行操作的命令，这个时候这些命令就会进入队列。当Redis使用exec命令执行事务的时候，首先会去对比被watch命令所监控的键值对，如果没有发生变化，那么它会执行事务队列中的命令，提交事务；如果发生变化，那么它不会执行任何事物中的命令，而去事务回滚，无论事务是否回滚，Redis都会去取消执行事务前的watch命令。

![](image/QQ截图20191129220620.png)

​		乐观锁（CAS原理）：当一条线程去执行某些业务逻辑，但是这些业务逻辑操作的数据可能被其他线程共享了，这样会引发多线程中数据不一致的情况，为了克服这个问题，首先，在线程开始时读取这些多线程共享的数据，并将其保存到当前线程的副本中，成为旧值。watch命令就是这样的一个功能。然后，开始线程业务逻辑，有multi命令提供这一功能。在执行更新前，比较当前线程副本保存的旧值和当前线程共享的值是否一致，如果不一致，那么该苏剧已经被其他线程操作过，此次更新失败，为了保持一致，线程就不会更新任何值，而将事务回滚；否则就认为它没有被其他线程操作过，执行对应的业务逻辑。

​		ABA问题：上述原理造成的问题。

![](image/QQ截图20191129221737.png)

​				在处理复杂运算时，被线程2修改的X的值可能导致线程1的运算出错，而最后线程2将X的值修改为原来		的A，那么到了线程1匀速结束的时间顺序，将检测X的值是否发生变化，比对发现一致，于是提交事务，然后		再复杂计算的过程中X被线程2修改过了，这会导致线程1的运算出错，在这个过程中，对于线程2而言，X的值		变化为：A->B->A。

​				在Hibernate中，其会对缓存对象加入字段version，每当操作一次该POJO，则version加1，从而保证数		据的一致性。

​		

​		所以：Redis在执行事务的过程中，并不会阻塞其他连接的并发，而只是通过比较watch监控的键值对去保证数据的一致性，所以Redis多个事务完全可以在非阻塞的多线程环境中并发执行，而且Redis的机制是不会产生ABA问题，这样就有利于在保证数据一致的基础上，提高高并发系统的数据读/写性能。

![](image/QQ截图20191129222748.png)

## 4、流水线

​		流水线技术是为了解决网络延迟也产生的长时间的等待。其是一种通信协议。

![](image/QQ截图20191129223150.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);

        SessionCallback callback = new SessionCallback() {
            @Override
            public Object execute(RedisOperations ops) throws DataAccessException {
                for (int i = 0 ; i < 100000 ; i++)
                {
                    int j = i + 1;
                    ops.boundValueOps("pipeline_key_" + j).set("pipeline_value_" + j);
                    ops.boundValueOps("pipeline_key_" + j).get();
                }
                return null;
            }
        };
        long start = System.currentTimeMillis();
		//执行Redis的流水线命令
        List list = redisTemplate.executePipelined(callback);
        long l = System.currentTimeMillis();
        System.out.println(l - start);
```

## 5、发布订阅

​		发布消息模式首先需要消息源，也就是要有消息发布出来，当消息发出，订阅者就能收到这个消息进行处理。

![](image/QQ截图20191201193425.png)

![](image/QQ截图20191201195258.png)

​		Spring中的发布订阅模式，接受消息的类，是MessageListener接口的实现类。

​		Redis发布订阅监听类：

```java
public class RedisMessageListener implements MessageListener
{
    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void onMessage(Message message, byte[] bytes)
    {
        byte[] body = message.getBody();
        String msgBody = (String)redisTemplate.getValueSerializer().deserialize(body);
        byte[] channel = message.getChannel();
        String channelStr = (String)redisTemplate.getStringSerializer().deserialize(channel);

        System.out.println(channelStr);
        String s = new String(bytes);
        System.out.println(s);
    }
}
```

```xml
<bean id="redisMessageListener" class="com.shanji.messagelister.RedisMessageListener" />
```

​		此时Spring中定义了监听类。但是还不能测试，还需要一个监听容器。RedisMessageListenerContainer，它可用于监听Redis的发布订阅消息。

```xml
<bean id="topicContainer" class="org.springframework.data.redis.listener.RedisMessageListenerContainer" destroy-method="destroy">
    	<!-- Redis连接工厂 -->
        <property name="connectionFactory" ref="connectionFactory" />
        <!-- 连接池，这里只要线程池生存，才能继续监听 -->
    	<property name="taskExecutor">
            <bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler">
                <property name="poolSize" value="3" />
            </bean>
        </property>
    	<!-- 消息监听map -->
        <property name="messageListeners">
            <map>
                <!-- 配置监听者，key-ref和bean id定义一致 -->
                <entry key-ref="redisMessageListener">
                    <!-- 监听类 -->
                    <bean class="org.springframework.data.redis.listener.ChannelTopic">
                        <constructor-arg value="chat" />
                    </bean>
                </entry>
            </map>
        </property>
    </bean>
```

## 6、超时命令

​		Redis也有着自己的垃圾回收机制，对于Redis而言，del 命令可以删除一些键值对，与此同时，当内存空间满了之后，其还会按照回收机制去自动回收一些键值对。但是当垃圾进行回收的时候，又有可能执行回收而引发系统停顿，所以要选择适当的回收机制和时间将有利于系统性能的提高。

​		对于Redis而言，可以给对应的键值设置超时。

![](image/QQ截图20191201202209.png)

![](image/QQ截图20191201202639.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        redisTemplate.execute((RedisOperations ops) -> {
            ops.boundValueOps("key1").set("valu1");
            String keyValue = (String) ops.boundValueOps("key1").get();
            Long key1 = ops.getExpire("key1");
            System.out.println(key1);

            boolean b = false;
            b = ops.expire("key1", 120L, TimeUnit.SECONDS);
            b = ops.persist("key1");

            long l = 0L;
            l = ops.getExpire("key1");

            long now = System.currentTimeMillis();
            Date date = new Date();
            date.setTime(now + 120000);
            ops.expireAt("key", date);
            return null;
        });
```

​		Redis的key超时不会被其自动回收，它只会标识那些键值对超时了。好处在于：如果一个很大的键值对超时，对其回收需要很长的时间，如果采用超时回收，则可能产生停顿。坏处：超时的键值对会浪费比较多的空间。

​		两种方式回收超时键值对：定时回收和惰性回收

​				1、定时回收：指在确定的某个时间出发一段代码，回收超时的键值对。

​				2、惰性回收：当一个超时的键，被再次用get命令访问时，将出发Redis将其从内存中清空。

## 7、Lua语言

​		在Redis中，执行Lua语言是原子性的，也就是在执行Lua的时候是不会被中断的，具有原子性，这个特性有助于Redis对并发数据一致性的支持。

​		两种运行脚本的方法：

​				1、直接输入一些Lua语言的程序代码。

​				2、将Lua语言编写成文件。

​		一些简单的脚本 可以采用第一种方式，对于有一定逻辑的一般采用第二种方式。而对于采用简单脚本的，Redis支持缓存脚本，只是它会使用SHA-1算法对脚本进行签名，然后把SHA-1标识返回回来，只要通过这个标识运行就行了。

​		命令格式：

![](image/QQ截图20191201210521.png)

![](image/QQ截图20191201211137.png)

​		对于redis.call命令

![](image/QQ截图20191201211301.png)

​		KEYS[1]：代表读取传递给Lua脚本的第一个key参数，而ARGV[1]：代表第一个非key参数。上述只有一个key参数，所以key-num为1。

​		如果需要多次执行同样的一段脚本，可以使用Redis缓存脚本的功能。好处在于，如果脚本很长，从客户端传输可能需要很长的时间，那么使用标识字符串，则只需要传递32为字符串即可。

​		命令：script  load  script。

![](image/QQ截图20191201211844.png)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
        Jedis jedis = (Jedis) redisTemplate.getConnectionFactory().getConnection().getNativeConnection();

		//执行简单脚本
        String hello = (String) jedis.eval("return 'hello xiaoshanshan'");
        System.out.println(hello);

		//执行带参数脚本
        jedis.eval("redis.call('set',KEYS[1],ARGV[1])",1,"lua-key","lua-value");

        String s = jedis.get("lua-key");
        System.out.println(s);

		//缓存脚本，返回签名标识
        String s1 = jedis.scriptLoad("redis.call('set',KEYS[1],ARGV[1])");
		//通过标识执行脚本
        jedis.evalsha(s1,1,new String[]{"sha-key","sha-val"});

		//获取执行脚本后的数据
        String shaval = jedis.get("sha-key");
        System.out.println(shaval);
        jedis.close();
```

​		如果要存储对象，此时考虑使用Spring提供的RedisScript接口，其实现类：DefaultRedisScript

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
		
		//定义默认脚本封装类
        DefaultRedisScript<Role> redisScript = new DefaultRedisScript<>();

		//设置脚本
        redisScript.setScriptText("redis.call('set',KEYS[1],ARGV[1]) return redis.call('get',KEYS[1])");

		//定义操作的key列表
        ArrayList<String> keyList = new ArrayList<>();
        keyList.add("role1");

		//需要序列化保存和读取的对象
        Role role = new Role();

        role.setId(1);
        role.setRoleName("xiaoshanshan");
        role.setNote("ceshi");

		//获取标识字符串
        String sha1 = redisScript.getSha1();
        System.out.println(sha1);
		
		//设置返回结果类型，如果没有这句话，结果返回为空
        redisScript.setResultType(Role.class);

		//定义序列化器
        JdkSerializationRedisSerializer jdkSerializationRedisSerializer = new JdkSerializationRedisSerializer();

		//执行脚本，第一个RedisScript接口对象，第二个是参数序列化器，第三个是结果序列化器，第四个是Redis的key列表，最后是参数列表
        Role execute = (Role) redisTemplate.execute(redisScript, jdkSerializationRedisSerializer, jdkSerializationRedisSerializer, keyList, role);
        System.out.println(execute);
```

​		需要存在较多的逻辑。需要使用文件。

![](image/QQ截图20191201214314.png)

![](image/QQ截图20191201214331.png)

​		执行的命令键和参数是使用逗号分隔得，键之间用空格分开。并且逗号前后的空格不能省略。

​		Java中没有办法执行这样的文本脚本，可以考虑使用evalsha命令，因为evalsha可以缓存脚本，返回32为的shal标识。如果使用eval命令去执行文件里的字符串，一旦文件很大，那么就需要通过网络反复传递文件，此时效率不会很高。

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationConetxt.xml");
        RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);

        File file = new File("E:\\test.lua");
        byte[] bytes = getFileToByte(file);

        Jedis jedis = (Jedis) redisTemplate.getConnectionFactory().getConnection().getNativeConnection();

        byte[] shal = jedis.scriptLoad(bytes);

        Object evalsha = jedis.evalsha(shal, 2, "key1".getBytes(), "key2".getBytes(), "2".getBytes(), "4".getBytes());
        System.out.println(evalsha);
```

```txt
redis.call('set',KEYS[1],ARGV[1])
redis.call('set',KEYS[2],ARGV[2])

local n1 = tonumber(redis.call('get',KEYS[1]))
local n2  = tonumber(redis.call('get',KEYS[2]))

if n1 > n2 then
	return 1
end
if n1 == n2 then
	return 0
end
if n1 < n2 then
	return -1
end
```

## 8、备份

​		两种备份方式：

​				1、快照：备份当前瞬间Redis在内存中的数据记录。

​				2、追加文件：当Redis执行写命令后，在一定的条件下将执行过的写命令一次保存在Redis的文件中，		将来就可以一次执行那些保存的命令恢复Redis的数据。

​		快照备份，如果当前Redis的数据量大，备份可能造成Redis卡顿，但是恢复重启是比较快速的。对于追加文件而言，它只是追加写入 命令，所以备份一般不会造成Redis卡顿，但是恢复重启要执行更多的命令，备份文件可能也很大。

​		Redis允许使用其中的一种，同时使用两种，或者两者都不用。

![](image/QQ截图20191202213454.png)

​		默认配置：

​				当900秒执行一个写命令时，启动快照备份

​				当300秒执行10个写命令时，启动快照备份

​				当60秒内执行10000个写命令时，启动快照备份

​		Redis执行save命令的时候，将禁止写入命令。

​		stop-writes-on-bgsave-error  yes

​		bgsave：异步保存命令，系统会启动另外一条进程，把Redis的数据保存到对应的数据文件中，它和save命令最大的不同时它不会阻塞客户端的写入，也就是在执行bgsave的时候，允许客户端继续读/写Redis。在默认情况下，如果Redis执行bgsave失败后，Redis将停止接受写操作，这样以一种强硬的方式让用户知道数据不能正确的持久化到磁盘，某则就没人注意到灾难的发生。如果后台保存进程重新启动工作了，Redis也将自动允许写操作。



​		rdbchecksum  yes

​		这个命令是 是否对rdb文件进行检验，如果是将对rdb文件检验，从dbfilename的配置可以知道，rdb文件实际是Redis持久化的数据文件。



​		dbfilename  dump.rdb

​		它是数据文件，当采用快照模式备份（持久化）时，Redis将使用它保存数据，将来可以使用它恢复数据。



​		appendonly  no

​		如果appendonly配置为no，则不启动追加文件方式进行备份，如果appendonly配置为yes，则以追加文件方式备份Redis数据，那么此时Redis会按照配置，在特定的时候执行追加命令，用以备份数据。



​		appendfilename  "appendonly.aof"

​		定义追加的写入文件为 appendonly.aof，采用追加文件备份的时候命令都会写到这里。



​		appendfsync  always， appendfsync  everysec， appendfsync  no

​		追加文件和Redis命令是同步频率的，假设配置为always，其含义为当Redis执行命令的时候，则同步到追加文件，这样会使得Redis同步刷新追加文件，造成缓慢。

​		采用everysec则代表每秒同步一个命令到追加文件。

​		采用no，则有客户端调用命令执行备份，Redis本身不备份文件。

​		对于采用always配置的时候，每次命令都会持久化，好处在于安全，坏处每次的持久化将导致性能较差。采用everysec则每秒同步，安全性不如always，可能丢失1秒以内的命令，安全性尚可，性能也可以得到保障。采用no，性能有保障，但是失去备份将导致安全性较差。



​		no-appendfsync-on-rewrite  no

​		指定是否在后台追加文件rewrite（重写）期间调用fsync，默认为no ，表示要调用fsync（无论后台是否有子进程在刷盘）。Redis在后台写rdb文件或重写追加文件期间会存在大量磁盘I/O，此时，在某些Linux系统中，调用fsync可能会阻塞。



​		auto-aof-rewrite-percentage  100

​		指定Redis重写追加文件的条件，默认为100，表示与上次rewrite的追加文件大小相比，如果追加文件增长量超过上次追加文件大小的100%时，就会出发backgroundrewrite，若配置为0，则会禁用自动rewrite。



​		auto-aof-rewirte-min-size  64mb

​		指定触发rewrite的追加文件大小，若追加文件小于该值，即使当前文件的增量比例达到auto-aof-rewrite-percentage的配置值，也不会触发自动rewrite，即这两个配置项同时满足时，才会出发rewrite。



​		aof-load-truncated  yes

​		Redis在恢复时会忽略最后一跳可能存在问题的指令，默认为yes，即在追加文件写入时，可能存在指令写错的问题，这种情况下yes会log并继续，而no会直接回复失败。

## 9、内存回收策略

​		Redis也会因为内存不足而产生错误，也可能因为回收过久而导致系统长期的停顿。

​		当Redis的内存达到规定的最大值时，允许配置6中策略中的一种进行淘汰键值，并且将一些键值对进行回收。

![](image/QQ截图20191202214041.png)

​		volatile-lru：采用最近使用最少的淘汰策略，Redis将回收那么超时的（仅仅是超时的）键值对，也就是它只淘汰那些超时的键值对。

​		allkey-lru：采用淘汰最少使用的策略。Redis将对所有的（不仅仅是超时的）键值采用最近使用最少的淘汰策略。

​		volatile-random：采用随机淘汰策略删除超时的（仅仅是超时的）键值对。

​		allkeys-random：采用随机淘汰策略删除所有的（不仅仅是超时的）键值对，不常用。

​		volatile-ttl：采用删除存活时间最短的键值对策略。

​		noeviction：根本就不删除任何键值对，当内存已满，如果读操作，将正常工作，写操作，将返回错误。即只能读不能写。默认值

![](image/QQ截图20191202214737.png)

​		缺点：必须指明超时的键值对。对所有的键值对进行回收，有可能把正在使用的键值对删掉，增加了存储的不稳定性。

## 10、复制

### 1、读写分离

​		主从架构，大概思路：

​				1、在多台数据服务器中，只有一台主服务器，而主服务器只负责写入数据，不负责让外部程序读取数		据。

​				2、存在多台从服务器，从服务器不写入数据，只负责同步主服务器的数据，并让外部程序读取数据。

​				3、主服务器在写入数据后，即刻将写入数据的命令发送给从服务器，从而使得主从数据同步。

​				4、应用程序可以随机读取一台从服务器的数据，这样就分摊了读数据的压力。

​				5、当从服务器不能工作的时候，整个系统将不受影响；当主服务器不能工作的时候，可以方便地从从服		务器中选举一台来当主服务器。

​		![](image/QQ截图20191204202343.png)

### 2、读写分离配置

​		1、首先明确主机，确定后，关键的两个配置是 dir 和 dbfilename 选项，必须保证这个两个文件是可写的。对于 Redis的默认配置而言，dir 的默认值为 "./"，而对于dbfilename的默认值为 dump.rdb。即，采用 Redis当前目录的 dump.rdb 文件进行同步 。

​		2、明确主机后，还要配置 slaveof 选项。

​					格式： slaveof  server  port

​							server：主机

​							port：端口

​				此时，当从机Redis服务重启时，就会同步对应主机的数据。如果不想让从机继续复制主机的数据了，执行 slaveof  no  one。

​		在实际的Linux环境中，配置文件 redis.conf 中有一个 bind 配置，默认为 127.0.0.1，即只允许本机访问，将其修改为 bind 0.0.0.0，其他的服务器就能访问了。

### 3、读写分离的过程

![](image/QQ截图20191204204151.png)

​		1、无论如何要先保证主服务器的开启，开启主服务器后，从服务器通过命令或者重启配置项可以同步到主服务器。

​		2、当从服务器启动时，读取同步的配置，根据配置决定是否使用当前数据响应客户端，但后发送 SYNC命令。当主服务器接收到同步命令的时候，就会执行 bgsave 命令备份数据，但是主服务器并不会拒绝客户端的读/写，而是将来自客户端的写命令写入缓冲期。从服务器未收到主服务器备份的快照文件的时候，会根据其配置决定使用现有数据响应客户端或者拒绝。

​		3、当 bgsave 命令被主服务器执行完后，开始向从服务器发送备份文件，这个时候从服务器就会丢弃所有现有的数据，开始载入发送的快照文件。

​		4、当主服务器发送完备份文件后，从 服务器就会执行这些写入命令。此时就会把 bgsave 执行之后的缓存区内的写命令也发送给从服务器，从服务器完成备份文件解析，就开始像往常一样，接收命令，等待命令写入。

​		5、缓冲区的命令发送完成后，当主服务器执行一条写命令后，就同时往从服务器发送同步写入命令，从服务器酒喝主服务器保持一致了。而此时当从服务器完成主服务器发送的缓冲区命令后，就开始等待主服务器的命令了。

​		在主服务器同步到从服务器的过程中，需要备份文件。所以在配置的时候一般需要预留一些内存空间给主服务器，用以腾出空间执行备份命令。

![](image/QQ截图20191204205326.png)

## 11、哨兵模式

​		Redis可以存在多台服务器，并且实现了读写分离的功能，哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

![](image/QQ截图20191204210111.png)

​		作用：

​				1、通过发送命令，让Redis服务器返回监测其运行状态，包括主服务器和从服务器。

​				2、当哨兵监测到 master 宕机，会自动将 slave切换成 master，然后通过发布订阅模式通知到其他的从		服务器，修改配置文件，让它们切换主机。

​		

​		可以使用多个哨兵来进行多哨兵监控，多个哨兵不仅监控各个Redis服务器，而且哨兵之间相互监控，看看哨兵们是否还活着。

​		故障切换过程：当主服务器宕机，哨兵1先监测到这个结果，当时系统并不会马上进行 failover 操作，而仅仅是哨兵1主观地认为主机已经不可用，这个现象被称为主观下线。当后面的哨兵监测也监测到了主服务器不可用，并且有了一定数据的哨兵认为主服务器不可用，那么哨兵之间就会形成一次投票，投票的结果由一个哨兵发起，进行 failover 操作，在 failover 操作的过程中切换成功后，就会通过发布订阅方式，让各个哨兵把自己监控的服务器实现切换主机，这个过程称为：客观下线。对于客户端而言，一切都是透明的。

![](image/QQ截图20191204211156.png)

### 1、搭建

![](image/QQ截图20191204212342.png)

​		在从服务器的 redis.conf 配置文件中，修改：

![](image/QQ截图20191204212425.png)

​		配置哨兵，三个哨兵分别监控三个服务器：

![](image/QQ截图20191204212523.png)

​		上述配置完后，进入 Redis 的 src 目录：

![](image/QQ截图20191204212716.png)

​		启动顺序：首先启动主服务器，在启动从服务器，最后才启动哨兵进程。

### 2、Java中使用

```xml
<!-- Redis连接池 -->
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig" p:maxIdle="50" p:maxTotal="100" p:maxWaitMillis="20000" />

<!-- 连接池配置 -->
<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <constructor-arg name="sentinelConfig" ref="sentinelConfiguration" />
        <constructor-arg name="poolConfig" ref="poolConfig" />
        <property name="password" value="179980" />
    </bean>

<!-- JDK序列化器 -->
<bean id="jdkSerializationRedisSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
    
<!-- string 序列化器 -->
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />

<!-- 配置redisTemplate -->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="defaultSerializer" ref="stringRedisSerializer" />
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="valueSerializer" ref="stringRedisSerializer" />
    </bean>

<!-- 哨兵配置 -->
<bean id="sentinelConfiguration" class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
    <!-- 服务名称 -->
        <property name="master">
            <bean class="org.springframework.data.redis.connection.RedisNode">
                <property name="name" value="stylemaster" />
            </bean>
        </property>
    <!-- 哨兵服务IP和端口 -->
        <property name="sentinels">
            <set>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.11.128" />
                    <constructor-arg  name="port" value="26379" />
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.11.129" />
                    <constructor-arg  name="port" value="26379" />
                </bean>
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <constructor-arg name="host" value="192.168.11.130" />
                    <constructor-arg  name="port" value="26379" />
                </bean>
            </set>
        </property>
    </bean>
```

3、其他配置项：

![](image/QQ截图20191204214416.png)

![](image/QQ截图20191204214433.png)

​		其中 sentinel  down-after-milliseconds 配置项只是一个哨兵在超过其指定额毫秒数依旧没有得到回答消息后，会自己认为主机不可用，对于其他哨兵而言，并不会认为主机不可用。哨兵会记录这个消息，当拥有认为主观下线的哨兵到达  sentiel  monitor 所配置的数量的时候，就会发起一次新的投票，然后切换主机，此时哨兵会重写 Redis 的哨兵配置文件，以适应新场景的需要。

## 12、Redis和数据库的结合

### 1、Redis和数据库读操作

​		数据缓存往往会在Redis上设置超时时间，当设置Redis的数据超时后，Redis就没法读出数据了，这个时候就会触发程序读取数据库，然后将读取的数据库数据写入Redis（此时会给Redis重设超时时间），这样程序在读取的过程中就能按一定的时间间隔刷新数据了。

![](image/QQ截图20191205204040.png)

### 2、Redis和数据库写操作

​		写操作要考虑数据一致的问题，首先应该考虑从数据库中读取最新的数据，然后对数据进行操作，然后把数据写入Redis缓存中。

![](image/QQ截图20191205204238.png)

## 13、使用Spring缓存机制整合Redis

​		在Spring项目中提供了接口CacheManager来定义缓存管理器，这样各个不同缓存就可可以实现它来提供管理器的功能了。而在spring-data-redis.jar包中的实现CacheManager接口的则是RedisCacheManger。

### 1、缓存注解

![](image/QQ截图20191208164639.png)

​		这些注解既可以标注在类上，也可以标注在方法上。标注在类上，对所有的方法都有效，如果标注在方法上，则只是对方法有效。对于查询，考虑用@Cacheable；对于插入和修改，考虑用@CachePut；对于删除，考虑用@CacheEvict。

### 2、@Cacheable和@CachePut

​		属性：

![](image/QQ截图20191208165213.png)

​		value：数组，可以引用多个缓存管理器

​		Spring表达式和注解的约定：

![](image/QQ截图20191208165559.png)

### 3、@CacheEvict

​		这个注解主要是为了移除缓存对应的键值对，主要对于那些删除的操作。

![](image/QQ截图20191208171045.png)

​		如果将beforeInvocation属性设置为false，那么在方法内还是能读取到缓存服务器的数据，即在方法执行后删除缓存。如果设置为true，则会在方法执行前删除缓存。

# 七、高并发

​		负载均衡的功能：

​				1、对业务请求做初步的分析，决定分不分发请求到Web服务器。

​				2、提供路由算法。根据各个服务器的负载能力进行合理分发。

​				3、限流。

​		RPC（远程过程调用协议）：每一个服务都会暴露一些公共的接口给RPC服务，这样对于任何一个服务器都能够通过RPC服务获取其他服务器对应的接口去调度各个服务器的逻辑来完成功能，但是接口的相互调用也会造成一定的缓慢。

## 悲观锁

​		悲观锁是一种利用数据库内部机制提供的锁的方法，也就是对更新的数据加锁，这样在并发期间一旦有一个事务持有了数据库记录的锁，其他的线程就不能在对数据进行更新了。

```mysql
select id,user_id as uesrId,amount,send_date as sendDate,total,unit_amount as unitAmount,stock,version,note from t_red_packet where id = #{id} for update
```

​		对于使用主键查询，只会对对应的行加锁。

​		对于悲观锁来说，当一条线程抢占了资源后，其他的线程将得不到资源，那么这个时候，CPU就会将这些得不到资源的线程挂起，挂起的线程也会消耗CPU的资源，尤其是在高并发的请求中。

​		测试就会频繁的挂起，等待持有锁线程释放资源，一旦释放资源后，就开始抢夺，恢复线程，周而复始直至所有的资源抢完，大量的线程被挂起和恢复，将十分消耗资源。

​		有时，悲观锁也称独占锁，阻塞锁。

## 乐观锁

​		乐观锁是一种不会阻塞其他线程并发的机制，它不会使用数据库的锁进行实现，它的设计里面由于不阻塞其他线程，所以并不会引发线程频繁挂起和恢复，其也被称为非阻塞锁。

### CAS原理

![](image/QQ截图20191213202712.png)

​		CAS原理并不排斥并发，也不独占资源，只是在线程开始阶段就读入线程共享数据，保存为旧值，当处理完逻辑，需要更新数据时，会进行一次比较，即比较各个线程当前共享的数据是否和旧值保持一致，如果一致，就开始更新数据；如果不一致，则认为该数据已经被其他线程修改了，那么就不在更新数据，可以考虑重试或者放弃。

### ABA问题

![](image/QQ截图20191213203057.png)

​		T3时刻，线程2修改了X = B，此时线程1的业务逻辑依旧在执行，但是到了T5时刻，线程2又把X还原为A，那么到了T6时刻，使用CAS原理的旧值判断，线程1就会认为X值没有被修改过，于是执行了更新。此时难以判断在T4时刻，线程1在X = B的时候，对于线程1的业务逻辑是否正确的问题。

​		问题的发生，是因为业务逻辑存在回退的可能性，如果加入一个非业务逻辑的属性，并给予一定的约定，就可以解决问题。

![](image/QQ截图20191213203556.png)

​		但此时会因为非业务逻辑属性的原因，会有很多资源未得到使用，可以考虑充入机制，但过多的重入会造成大量的SQL执行。

​		两种限制：

​				1、按时间戳充入，也就是在一定时间内，不成功的会循环到成功为止，直至超过		时间戳，不成功才会退出，返回失败。

​				2、按次数。

