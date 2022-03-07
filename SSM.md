# **SSM(Spring + SpringMVC + Mybatis)**

# 一、Spring

## **1、核心理论：**

​		`Spring`核心容器就是一个超级大工厂，所有的对象`(`包括数据源、`Hibernate  SessionFactory`等基础性资源`)`都会被当成`Spring`核心容器管理的对象。

`Spring`把容器中的一切对象统称为`Bean`。

​		所谓的`Bean`，与`Java Bean`不同，`Java  Bean`必须遵守一些特定的规范，而**Spring对Bean没有任何要求**，只要**是一个Java类，Spring就可以管理该Java**

**类，并把它当成Bean处理**。`(`一切`Java`对象都是`Bean)`。

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
      <artifactId>spring-context</artifactId>
      <version>5.1.8.RELEASE</version>
    </dependency>
```

#### 1.1、基于xml来管理容器中的Bean：

​		根元素`<beans .../>`，包含多个`<bean .../>`元素，每个`<bean .../>`元素定义一个`Bean`。`<bean .../>`元素默认驱动`Spring`在底层调用无参数的构造器

创建对象。

​		解析时会执行类似如下的过程：

```java
String idStr = ...; //解析<bean .../>元素的id属性的到该字符串值
String classStr = ...; // 解析<bean .../>元素的class属性得到字符串值，所以当指定class属性时必须是完整类名，不能是抽象类，接口(除非有特殊配置)；
Class clazz = Class.forName(classStr);
Object obj = clazz.newInstance();
//container代表Spring容器
container.put(idStr,obj);
```

​		**结论：**

​				在`Spring`配置文件中配置`Bean`时，**class属性的值必须是Bean实现类的完整类名(必须带包名)，不能是接口，不能是抽象类(除非有特殊配置)，否则**

​		**Spring无法使用反射创建该类的实例**。

​		**<property .../>元素通常用于作为<bean .../>元素的子元素**，它驱动`Spring`在底层以反射执行一个`setter`方法，其中**name属性决定执行哪个setter方法，而**

**value或ref决定执行setter方法传入的参数**。如果**传入参数是基本类型及其包装类、String等类型**，则**使用value属性**指定传入参数。以**容器中其他Bean作为传**

**入参数**，则**使用ref属性**指定传入参数。

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

​				1）、ClassPathXmlApplicationContext：从类加载路径搜索配置文件，并根据配置文件来创建Spring容器(常用)。

​				2）、FileSystemXmlApplicationContext：从文件系统的相对路径或绝对路径下去搜索配置文件，并根据配置文件来创建Spring容器。

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

​				Object  get(String  id)：根据容器中的Bean的id来获取指定的Bean，获取Bean之后需要进行强制类型转换。

​				T  getBean(String  name,Class<T>  requiredType)：根据容器中的Bean的id来获取指定Bean，但该方法带一个泛型参数，因此获取Bean之后无须进行强制类型转换。

​		使用Spring框架之后最大的改变之一是：**程序不再使用new调用构造器创建 Java对象那个，所有的 Java对象都由Spring容器负责创建**。

## **2、依赖注入**

​		几乎所有的 Java应用中大量存在着A对象需要调用B对象方法的情形，这种情形被Spring称为依赖，即A对象依赖B对象。它们总是由一些互相调用的对象构成的，Spring把这种互相调用的关系称为依赖关系。

​		Spring的核心功能：

​				1）：Spring容器作为超级大工厂，负责创建、管理所有的 Java对象，这些 Java对象被称为Bean。

​				2）：Spring容器管理容器中Bean之间的依赖关系，Spring使用一种被称为“**依赖注入**”的方式管理Bean之间的依赖关系。

​		使用依赖注入，不仅可以为Bean注入普通的属性值，还可以注入其他Bean的引用。通过这种依赖注入，应用中各种组件不需要以硬编码方式耦合在一起，甚至无须使用工厂模式。

​		当某个 Java对象(调用者)需要调用另一个 Java对象(被依赖对象)的方法时

​				传统模式：

​						原始做法：调用者**主动**创建被依赖对象，然后再调用被依赖对象的方法。需要用过“new  被依赖对象构造器();”的代码创建对象。导致调用者与被依赖对象实现类的硬编码耦合。不利于项目维护。

​						简单工厂模式：调用者先找到被依赖对象的工厂，然后**主动通过工厂去获取被依赖对象**，最后再调用被依赖对象的方法。把握三点：

​								1）：调用者面向被依赖对象的接口编程。

​								2）：将被依赖对象的创建交给工厂完成。

​								3）：调用者通过工厂来获得被依赖组件。

​								唯一缺点：调用组件需要主动通过工厂去获取被依赖对象，会带来调用组件与被依赖对象工厂的耦合。

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

​		两个核心接口：`BeanFactory`和`ApplicationContext`。其中`ApplicationContext`是`BeanFactory`的子接口。

![](image/QQ截图20190618003405.png)

​		`BeanFactory`负责配置、创建、管理`Bean`，它有一个子接口：`ApplicationContext`，因此也被称为`Spring`上下文。

![](image/QQ截图20190617225243.png)

![](image/QQ截图20190618003607.png)

​	创建`Spring`容器的实例时，必须提供`Spring`容器管理的`Bean`的详细配置信息，`Spring`的配置信息通常采用`xml`配置文件来配置，因此，创建`BeanFactory`

实例时，应该提供`XML`配置文件作为参数，`XML`配置文件通常使用`Resource`对象传入。`Resource`接口是`Spring`提供的资源访问接口，通过该接口，`Spring`能以

简单、透明的方式访问磁盘、类路径以及网络上的资源。

​		获取`BeanFactory`：

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

​		除了提供`BeanFactory`所支持的全部功能外，`ApplicationContext`还有如下功能：

![](image/QQ截图20190617232124.png)

​		当系统创建`ApplicationContext`容器时，默认会预初始化所有的`singleton Bean`。包括调用构造器创建`Bean`的实例，并根据`<property  .../>`元素执行

`setter`方法。所以系统前期创建`ApplicationContext`时将有较大的系统开销。

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

​		如果不想`Spring`容器预初始化`singleton  Bean`，可以为`<bean  .../>`指定**lazy-init="true"**。此时，只有使用`Spring`容器的`getBean`方法时，`Bean`才会进

行初始化。

​		`Bean`的初始化：分三步

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

​				3）：**接受构造注入的Bean，则应提供对应的、带参数的构造函数**

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
@Scope：用于修饰一个方法，指定该方法对饮得Bean的生命周期
@Lazy：用于修饰一个方法，指定该方法对应的Bean是否需要延迟初始化
@DependsOn：用于修饰一个方法，指定在初始化该方法对应的Bean之前初始化指定的Bean
@ImportResource：修饰一个Java配置类，用于向当前配置类中导入其他配置文件
```

### 2）：

![](image/QQ截图20190619221714.png)



##### 添加Bean：

​		@Configuration：修饰类，表明是一个配置类。

​		@Bean：修饰方法和类，修饰方法时将返回值作为一个Bean注册，类型为返回值类型，id默认是方法名。注册的Bean默认为单实例。

​		@ComponentScan：value：指定要扫描的包，excludeFilters：指定排除指定规则的Bean，includeFilters：指定扫描指定只包含的组件，要让其生效，需要指定useDefaultFilters=false，禁用默认的过滤规则。（**type：FilterType.ANNOTATION：按照注解；FilterType.ASSIGNABLE_TYPE：按照类型；FilterType.ASPECTJ：指定ASPECTJ表达式；FilterType.REGEX：按照正则表达式；FilterType.CUSTOM：自定义规则；**）

​					自定义规则：需要一个实现了TypeFilter接口的类。并重写match方法：其中MetadataReader类型的参数代表当前正在扫描的类的信息，MetadataReaderFactory：用于获取其他类的信息。

​		@ComponentScans：指定多个@ComponentScan。		

​		@Scope：指定Bean的作用域。

​				ConfigurableBeanFactory.SCOPE_PROTOTYPE：prototype。ioc容器启动并不会去调用方法创建对象放在容器中，每次获取的时候才会调用方法创建对象。每调一次就创建一个对象。

​				ConfigurableBeanFactory.SCOPE_SINGLETON：singleton。ioc容器启动会调用方法创建对象放到ioc容器中，以后每次获取就是直接从容器中拿。

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

​				@Qualifier：指定装配给定的 id 的对象。其和 @Autowired 默认是一定要装配对应的 Bean，如果没有找到，会报错。可指定 @Autowired 的属性 required 为 false ，来允许为空。

​				@Primary：修饰的 Bean 在进行依类型自动装配时，会优先使用。也可以继续使用 @Qualifier 来指定具体装配哪一个 Bean。

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

​							@Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetioninternalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator

​							给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；

​					2、 AnnotationAwareAspectJAutoProxyCreator：

​							AnnotationAwareAspectJAutoProxyCreator

​									->AspectJAwareAdvisorAutoProxyCreator

​											->AbstractAdvisorAutoProxyCreator

​													->AbstractAutoProxyCreator   implements 																	SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory



​				AbstractAutoProxyCreator.setBeanFactory()

​				AbstractAutoProxyCreator.有后置处理器的逻辑；



​				AbstractAdvisorAutoProxyCreator.setBeanFactory()-》initBeanFactory()



​				AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()

 

​				流程：

​						1）、传入配置类，创建ioc容器

​						2）、注册配置类，调用refresh（）刷新容器；

​						3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；

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

​												2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization()

​												3）、invokeInitMethods()；执行自定义的初始化方法

​												4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization()；

​										4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》													aspectJAdvisorsBuilder

​								7）、把BeanPostProcessor注册到BeanFactory中；

​											beanFactory.addBeanPostProcessor(postProcessor);

​			=======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========

​						AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor

​						4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean

​								1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);

​										getBean->doGetBean()->getSingleton()->

​								2）、创建bean

​										【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】

​										1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；

​												只要创建好的Bean都会被缓存起来

​										2）、createBean（）;创建bean；

​												AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例

​												【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】

​												【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】

​												1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation

​														希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续

​														1）、后置处理器先尝试返回对象；

​																bean = applyBeanPostProcessorsBeforeInstantiation（）：

​																		拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;

​																		就执行postProcessBeforeInstantiation

​																if (bean != null) {
​																	bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
​																}



​												2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；

​												3）、



AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：

​		1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；关心MathCalculator和LogAspect的创建

​				1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）

​				2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，或者是否是切面（@Aspect）

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

​		此时的工厂`Bean`并不是创建`Bean`的工厂，而是一种特殊的`Bean`，这种`Bean`必须实现`FactoryBean`接口。

​		`FactoryBean`接口是`Bean`的标准接口，把工厂`Bean(`实现`FactoryBean`接口的`Bean)`部署在容器中之后，程序通过`getBean()`方法来获取它时，容器返回的

不是`FactoryBean`实现类的实例，而是返回`FactoryBean`的产品`(`即该工厂`Bean`的`getObject`方法的返回值`)`。

![](image/QQ截图20190621225409.png)

​		配置FactoryBean与配置普通Bean的定义没有什么区别，但当程序向Spring容器请求获取该Bean时，**容器返回该FactoryBean的产品，而不是返回该FactoryBean本身**。

​		Spring在检测容器中的所有Bean时，如果发现某个Bean实现类实现了FactoryBean接口，**Spring容器就会在实例化该Bean，根据<property .../>执行setter方法之后，额外调用该Bean的getObject方法，并将该方法的返回值作为容器中的Bean**。当程序需要获得FactoryBean本身时，并不直接请求Bean  id，而是在Bean  id前增加“&”符号，容器则返回FactoryBean本身，而不是其产品Bean。

### 7.4、获取Bean本身的id

​		通过Spring提供的BeanNameAware接口，可提前预知该Bean的配置id。

​		此接口提供一个setBeanName(String  name)，该方法的name参数就是Bean的id，实现该方法的Bean类可通过该方法来获得部署该Bean的id。这个方法是由Spring容器负责调用的。此时的name参数就是配置该Bean时，指定的id。

## 8、容器中Bean的生命周期

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

​				调用那个getter方法：由PropertyPathFactoryBean得setPropertyPath(String  propertyPath)方法指定。

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

​				postProcessBeforeInitalization(Object  bean,String  name) throws BeansException：该方法的第一个参数是系统即将进行后处理的Bean实例，第二个参数是该Bean的配置id。

​				postProcessAfterInitialization(Object  bean,String  name) throws  BeansException：该方法的第一个参数是系统即将进行后处理的Bean实例，第二个参数是该Bean配置的id。

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

​				postProcessBeanFactory(ConfigurableListableBeanFactory  beanFactory)：该方法的方法体就是对Spring 容器进行的处理，这种处理可以对Spring容器进行自定义扩展。

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

​				动态AOP实现：AOP框架在运行阶段动态生成AOP代理(在内存中以JDK动态代理或cglib动态地生成AOP代理类)，以实现对目标对象的增强。Spring  AOP为代表。

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

​				增强处理(Advice)：AOP框架在特定的切入点执行的增强处理。处理有“around”，“before”，“after”等类型。

​				切入点(Pointcut)：可以插入增强处理的连接点，简而言之，当某个连接点满足指定要求时，该连接点将被添加增强处理，该连接点也就变成了切入点。

​				引入：将方法或字段添加到被处理的类中。Spring允许将新的接口引入到任何被处理的对象中。

​				目标对象：被AOP框架进行增强处理的对象，也被称为被增强的对象，如果AOP框架采用的是动态AOP实现，那么该对象就是一个被代理的对象。

​				AOP代理：AOP框架创建的对象，代理就是对目标对象的加强。Spring中的AOP代理可以是JDK动态代理，也可以是cglib代理，前者为实现接口的目标对象的代理，后者为不实现接口的目标对象的代理。

​				织入(Weaving)：将增强处理添加到目标对象中，并创建一个被增强的对象(AOP代理)的过程就是织入。织入有两种实现方式：编译时增强和运行时增强。**Spring和其他纯 Java  AOP框架一样，在运行时完成织		入**。

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

​				pointcut/value：这两个属性的作用一样，它们都用于指定该切入点对应的切入表达式。当指定了pointcut属性值后，value属性将会被覆盖。

​				returning：该属性值指定一个形参名，用于表示Advice方法中可定义与此同名的形参，该形参可用于访问目标方法的返回值，除此之外，在Advice方法中定义该形参(代表目标方法的返回值)时指定的类型，会限制目标方法返回指定类型的值或没有返回值。

![](image/QQ截图20190713213453.png)

#### 4）、定义AfterThrowing增强处理

​		使用@AfterThrowing注解可修饰AfterThrowing增强处理，AfterThrowing主要用于处理程序中未处理的异常。

​		两个属性：

​				pointcut/value：指定该切入点对应的切入表达式，当指定pointcut属性值后，value属性值将会被覆盖。

​				throwing：该属性指定一个形参名。该形参可用于访问目标方法抛出的异常，定义该形参时指定的类型，会限制目标方法必须抛出指定类型的异常。

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

​		进入连接点，高优先级将先被织入(Before，优先级高的先执行)，退出时，高优先级的最后被织入。(After，高优先级的后执行)。

​		指定优先级：

​				1、实现Ordered接口，实现getOrder()方法，返回的值越小，优先级越高。

​				2、使用@Order修饰切面类，value值越小，优先级越高。

​		同一个切面类里的两个相同类型的增强处理在同一个连接点被织入时，以随机的顺序来织入这两个增强处理。

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

​				pointcut-ref：该属性指定一个已经存在的切入点名称，通常pointcut和pointcut-ref两个属性只需使用其中之一。

​				method：该属性指定一个方法名，指定将切面的该方法转换为增强处理。

​				throwing：该属性只对<ater-throwing .../>元素有效，用于指定一个形参名，AfterThrowing增强处理方法可通过该形参访问目标方法所抛出的异常。

​				returning：该属性只对<after-returning .../>元素有效，用于指定一个形参名，AfterReturning增强处理方法可通过该形参访问目标方法的返回值。

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

​				1、在Spring配置文件中添加<cache:annotation-driven  cache-manager=""  />，该元素指定Spring根据注解来启用Bean级别或者方法级别的缓存。默认值为cacheManager，即如果将容器中缓存管理器的ID设为cacheManager，则可省略<cache:annotation-driven  .../>的cache-manager属性。

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

​				事务传播：通常，在事务中执行的代码都会在当前事务中运行。但是，如果一个事务上下文已经存在，有几个选项可指定该事务性方法的执行行为。

​				事务超时：事务在超时前能运行多久，也就是事务的最长持续时间。如果事务一直没有被提交或回滚，将在超出该时间后，系统自动回滚事务。

​				只读状态：只读事务不修改任何数据，在某些情况下，只读事务是非常有用的优化。

​		TransactionStatus代表事务本身，提供了简单的控制事务执行和查询事务状态的方法，这些方法在所有的事务API中都是相同的。

![](image/QQ截图20190716175444.png)

​		Spring具体的事务管理由PlatformTransactionManager的不同实现类来完成。Spring容器中配置PlatformTransactionManagerBean时，必须针对不同的环境提供不同的实现类。

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

​				编程式事务管理：即使使用Spring的编程式事务，程序也可直接获取容器中的transactionManagerBean，该Bean总是PlantformTransactionManager的实例，所以可以通过该接口提供的三个方法来开始事务，提交事务和回滚事务。

​				声明式事务：无须在 Java程序中书写任何事物操作代码，而是通过在XML文件中为业务组件配置事务代理（AOP代理的一种），AOP为事务代理所织入的增强处理也由Spring提供，在目标方法执行之前，织入开始事务，在目标方法执行之后，织入结束事务。

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

​		@Transactional注解，既可以修饰Spring  Bean类，也可用于修饰Bean类中的某个方法。修饰类，事务将对整个Bean类起作用，修饰方法，只对该方法其作用。

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



## 源码

### 容器的基本实现

#### 两个重要类

##### DefaultListableBeanFactory

​		`DefaultListableBeanFactory`是整个`bean`加载的核心部分，是`Spring`注册及加载`bean`的默认实现，`DefaultListableBeanFactory`继承了

`AbstractAutowireCapableBeanFactory`并实现了`ConfigurableListableBeanFactory`以及`BeanDefinitionRegistry`接口。

![](image/QQ截图20220111094611.png)

|                类名                |                             描述                             |
| :--------------------------------: | :----------------------------------------------------------: |
|           AliasRegistry            |               定义对`alias`的简单增删改等操作                |
|        SimpleAliasRegistry         | 主要使用`map`作为`alias`的缓存，并对接口`AliasRegistry`进行实现 |
|       SingletonBeanRegistry        |                    定义对单例的注册及获取                    |
|            BeanFactory             |               定义获取`bean`及`bean`的各种属性               |
|    DefaultSingletonBeanRegistry    |          对接口`SingletonBeanRegistry`各函数的实现           |
|      HierarchicalBeanFactory       | 继承`BeanFactory`，也就是在`BeanFactory`定义的功能的基础上增加了对`parentFactory`的支持 |
|       BeanDefinitionRegistry       |            定义对`BeanDefinition`的各种增删改操作            |
|     FactoryBeanRegistrySupport     | 在`DefaultSingletonBeanRegistry`基础上增加了对`FactoryBean`的特殊处理功能 |
|      ConfigurableBeanFactory       |                 提供配置`Factory`的各种方法                  |
|        ListableBeanFactory         |               根据各种条件获取`bean`的配置清单               |
|        AbstractBeanFactory         | 综合`FactoryBeanRegistrySupport`和`ConfigurableBeanFactory`的功能 |
|     AutowireCapableBeanFactory     |   提供创建`bean`、自动注入、初始化以及应用`bean`的后处理器   |
| AbstractAutowireCapableBeanFactory | 综合`AbstractBeanFactory`并对接口`AutowireCapableBeanFactory`进行实现 |
|  ConfigurableListableBeanFactory   |         `BeanFactory`配置清单，指定忽略类型及接口等          |
|     DefaultListableBeanFactory     |         综合上面所有功能，主要是对`bean`注册后的处理         |

​		`XmlBeanFactory`对`DefaultListableBeanFactory`类进行了扩展，主要用于从`XML`文档中读取`BeanDefinition`，对于注册及获取`bean`都是使用从父类

`DefaultListableBeanFactory`继承的方法去实现，而唯独与父类不同的个性化实现就是增加了`XmlBeanDefinitionReader`类型的`reader`属性。在`XmlBeanFactory`

中主要使用`reader`属性对资源文件进行读取和注册。



##### XmlBeanDefinitionReader

​		`XmlBeanDefinitionReader`负责操作`XML`配置文件，包括资源文件读取、解析及注册等。

![](image/QQ截图20220111102623.png)

|             类名             |                             描述                             |
| :--------------------------: | :----------------------------------------------------------: |
|        ResourceLoader        | 定义资源加载器，主要应用于根据给定的资源文件地址返回对应的`Resource` |
|     BeanDefinitionReader     |    主要定义资源文件读取并转换为`BeanDefinition`的各个功能    |
|      EnvironmentCapable      |                  定义获取`Environment`方法                   |
|        DocumentLoader        |          定义从资源文件加载到转换为`Document`的功能          |
| AbstractBeanDefinitionReader | 对`EnvironmentCapable`、`BeanDefinitionReader`类定义的功能进行实现 |
| BeanDefinitionDocumentReader |         定义读取`Document`并注册`BeanDefinition`功能         |
| BeanDefinitionParserDelegate |                 定义解析`Element`的各种方法                  |

​		处理流程：

​				1、通过继承自`AbstractBeanDefinitionReader`中的方法，来使用`ResourLoader`将资源文件路径转换为对应的`Resource`文件。

​				2、通过`DocumentLoader`对`Resource`文件进行转换，将`Resource`文件转换为`Document`文件。

​				3、通过实现接口`BeanDefinitionDocumentReader`的`DefaultBeanDefinitionDocumentReader`类对`Document`进行解析，并使用

​		`BeanDefinitionParserDelegate`对`Element`进行解析。



#### XmlBeanFactory

```java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
```

​		在`Java`中，将不同来源的资源抽象成`URL`，通过注册不同的`handler(URLStreamHandler)`来处理不同来源的资源的读取逻辑，一般`handler`的类型使用不同

前缀来识别，然而`URL`没有默认定义相对`Classpath`或`ServletContext`等资源的`handler`，`Spring`对其内部使用到的资源实现了自己的抽象结构：`Resource`接

口封装底层资源。

```java
public interface InputStreamSource {
	InputStream getInputStream() throws IOException;
}

public interface Resource extends InputStreamSource {
	boolean exists();
	default boolean isReadable() {
		return true;
	}
	default boolean isOpen() {
		return false;
	}
	default boolean isFile() {
		return false;
	}
	URL getURL() throws IOException;
	URI getURI() throws IOException;
	File getFile() throws IOException;
	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}
	long contentLength() throws IOException;
	long lastModified() throws IOException;
	Resource createRelative(String relativePath) throws IOException;
	String getFilename();
	String getDescription();
}
```

​		对不同来源的资源文件都有相应的`Resource`实现：

![](image/QQ截图20220111134406.png)

​		当通过`Resource`相关类完成了对配置文件进行封装后配置文件的读取工作就全权交给`XmlBeanDefinitionReader`来处理了：

```java
//BeanFactory factory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
public class XmlBeanFactory extends DefaultListableBeanFactory {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		/*
		public AbstractAutowireCapableBeanFactory() {
			super();
			// ignoreDependencyInterface:忽略给定接口的自动装配功能
			ignoreDependencyInterface(BeanNameAware.class); // BeanNameAware：注入 bean 对应的 name
			ignoreDependencyInterface(BeanFactoryAware.class); // BeanFactoryAware：注入 BeanFactory
			ignoreDependencyInterface(BeanClassLoaderAware.class); // BeanClassLoaderAware：注入 bean 对应的类加载器
		} 
		*/
        super(parentBeanFactory); 
	    this.reader.loadBeanDefinitions(resource); // 此时将工作交给 XmlBeanDefinitionReader 来完成
	}
}
```

![](image/QQ截图20220111135325.png)

​		整个过程：

​				1、封装资源文件。当进入`XmLBeanDefinitionReader`后首先对参数`Resource`使用`EncodedResource`类进行封装。`EncodedResource`主要用于对资源文件

​		进行编码处理。

```java
public class EncodedResource implements InputStreamSource {
    ...
	public Reader getReader() throws IOException {
		if (this.charset != null) {
			return new InputStreamReader(this.resource.getInputStream(), this.charset);
		}
		else if (this.encoding != null) {
			return new InputStreamReader(this.resource.getInputStream(), this.encoding);
		}
		else {
			return new InputStreamReader(this.resource.getInputStream());
		}
	}
    ...
}
```

​				2、获取输入流。从`Resource`中获取对应的`InputStream`并构造`InputSource`。

​				3、通过构造的`InputSource`实例和`Resource`实例继续调用函数`doLoadBeanDefinitions`。

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource);
		}
		
    	// 通过属性来记录已经加载的资源
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
             // 从 encodedResource 中获取 inputStream
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
                  // InputSource 来源于 org.xml.sax，并不来自于 Spring
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
                  // 真正的处理部分
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
```

```java
protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
	return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
                                            getValidationModeForResource(resource), isNamespaceAware());
}

protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) throws BeanDefinitionStoreException {
		try {
             /**

             */
			Document doc = doLoadDocument(inputSource, resource);
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```

​		`doLoadBeanDefinitions`，忽略其中的异常处理，整个逻辑做了三件事情：

​				1、获取对`XML`文件的验证模式。

​				2、加载`XML`文件，并得到对应的`Document`。

​				3、根据返回的`Document`注册`Bean`信息。



#### XML的校验模式

​		`XML`文件的验证模式保证了`XML`文件的正确性，而比较常用的验证模式有两种：`DTD`和`XSD`。

​		`DTD`即文档类型定义，是一种`XML`约束模式语言，是`XML`文件的验证机制，属于`XML`文件组成的一部分。`DTD`是一种保证`XML`文档格式正确的有效方法，可

以通过比较`XML`文档和`DTD`文件来看文档是否符合规范，元素和标签使用是否正确。一个`DTD`文档包含：元素的定义规则，元素间关系的定义规则，元素可使用的

属性，可使用的实体或符号规则。要使用`DTD`验证模式的时候需要在XML文件的头部声明：

```xml
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">
```

​		`XML Schema`语言就是`XSD`。`XML Schema`描述了`XML`文档的结构。可以用一个指定的`XML Schema`来验证某个`XML`文档，以检查该`XML`文档是否符合其要求。

文档设计者可以通过`XML Schema`指定`XML`文档所允许的结构和内容，并可据此检查`XML`文档是否是有效的。`XML Schema`本身是`XML`文档，它符合`XML`语法结构。

可以用通用的`XML`解析器解析它。在使用`XML Schema`文档对`XML`实例文档进行检验，除了要声明名称空间外，还必须指定该名称空间所对应的`XML Schema`文档的

存储位置。通过`schemaLocation`属性来指定名称空间所对应的`XML Schema`文档的存储位置，它包含两个部分，一部分是名称空间的`URI`，另一部分就是该名称空

间所标识的`XML Schema`文件位置或`URL`地址：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

```java
protected int getValidationModeForResource(Resource resource) {
		int validationModeToUse = getValidationMode();
    	// 手动指定了验证模式则使用指定的验证模式，通过调用 XmlBeanDefinitionReader#setValidationMode 方法来设置
		if (validationModeToUse != VALIDATION_AUTO) {
			return validationModeToUse;
		}
    	 // 未指定则自动检测
		int detectedMode = detectValidationMode(resource);
		if (detectedMode != VALIDATION_AUTO) {
			return detectedMode;
		}
		return VALIDATION_XSD;
}
```

```java
protected int detectValidationMode(Resource resource) {
		if (resource.isOpen()) {
			throw new BeanDefinitionStoreException(
					"Passed-in Resource [" + resource + "] contains an open stream: " +
					"cannot determine validation mode automatically. Either pass in a Resource " +
					"that is able to create fresh streams, or explicitly specify the validationMode " +
					"on your XmlBeanDefinitionReader instance.");
		}

		InputStream inputStream;
		try {
			inputStream = resource.getInputStream();
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"Unable to determine validation mode for [" + resource + "]: cannot open InputStream. " +
					"Did you attempt to load directly from a SAX InputSource without specifying the " +
					"validationMode on your XmlBeanDefinitionReader instance?", ex);
		}

		try {
			return this.validationModeDetector.detectValidationMode(inputStream);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("Unable to determine validation mode for [" +
					resource + "]: an error occurred whilst reading from the InputStream.", ex);
		}
}
```

```java
// XmlValidationModeDetector 类
public int detectValidationMode(InputStream inputStream) throws IOException {
		// Peek into the file to look for DOCTYPE.
		BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
		try {
			boolean isDtdValidated = false;
			String content;
			while ((content = reader.readLine()) != null) {
				content = consumeCommentTokens(content);
                 // 读取到的行是空或者是注释则跳过
				if (this.inComment || !StringUtils.hasText(content)) {
					continue;
				}
				if (hasDoctype(content)) {
					isDtdValidated = true;
					break;
				}
                 // 读取到 < 开始符号，并且 < 符号后面是小写字母，则为 xsd 验证模式，验证模式一定会在开始符号之前
				if (hasOpeningTag(content)) {
					// End of meaningful data...
					break;
				}
			}
			return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
		}
		catch (CharConversionException ex) {
			// Choked on some character encoding...
			// Leave the decision up to the caller.
			return VALIDATION_AUTO;
		}
		finally {
			reader.close();
		}
}
```

​		`Spring`用来检测验证模式的办法就是判断是否包含`DOCTYPE`，如果包含就是`DTD`，否则就是`XSD`。



#### 获取Document

​		经过了验证模式准备的步骤就可以进行`Document`加载了，`XmlBeanFactoryReader`类委托给了`DocumentLoader`去执行，真正调用的是

`DefaultDocumentLoader`：

```java
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
			ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

		DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
		if (logger.isDebugEnabled()) {
			logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
		}
		DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
		return builder.parse(inputSource);
}
```

​		通过`SAX`解析`XML`文档，首先创建`DocumentBuilderFactory`，再通过`DocumentBuilderFactory`创建`DocumentBuilder`，进而解析`inputSource`来返回

`Document`对象。

​		`EntityResolver`的作用是项目本身就可以提供一个如何寻找`DTD`声明的方法，即由程序来实现寻找`DTD`声明的过程。`Spring`中通过`getEntityResolver()`

方法对`EntityResolver`的获取，`Spring`中使用`DelegatingEntityResolver`类为`EntityResolver`的实现类：

```java
public class DelegatingEntityResolver implements EntityResolver {
	...
    /**
     XSD：
    	<beans xmlns="http://www.springframework.org/schema/beans"
		       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
		</beans>
		publicId：null
		systemId：http://www.springframework.org/schema/beans/spring-beans.xsd
	
	
	 DTD:
	 	<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">
	 	publicId：-//SPRING//DTD BEAN 2.0//EN
	 	systemId：http://www.springframework.org/dtd/spring-beans-2.0.dtd
    */
	public InputSource resolveEntity(@Nullable String publicId, @Nullable String systemId)
			throws SAXException, IOException {

		if (systemId != null) {
			if (systemId.endsWith(DTD_SUFFIX)) {
                 // BeansDtdResolver
				return this.dtdResolver.resolveEntity(publicId, systemId);
			}
			else if (systemId.endsWith(XSD_SUFFIX)) {
                  // PluggableSchemaResolver
				return this.schemaResolver.resolveEntity(publicId, systemId);
			}
		}

		// Fall back to the parser's default behavior.
		return null;
	}
    ...
}
```

​		加载`DTD`类型的`BeansDtdResolver`的`resolveEntity`是直接截取`systemId`最后的`xx.dtd`然后去当前路径下寻找，而加载`XSD`类型的

`PluggableSchemaResolver`类的`resolveEntity`是默认到`META-INF/Spring.schemas`文件中找到`systemid`所对应的`XSD`文件并加载。



#### 解析注册BeanDefinitions

​		文件转换为`Document`后，就将进行提取及注册`bean`。

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
         // 使用 DefaultBeanDefinitionDocumentReader 实例化 BeanDefinitionDocumentReader
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    	// 在实例化 BeanDefinitionReader 时会将 BeanDefinitionRegistry传入，默认使用继承自DefaultListableBeanFactory的子类
        // 记录统计前 BeanDefinition的加载个数
		int countBefore = getRegistry().getBeanDefinitionCount();
    	// 加载及注册 bean
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    	// 记录本次加载的 BeanDefinition 个数
		return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

```java
// DefaultBeanDefinitionDocumentReader 类
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement(); // 提取 root
		doRegisterBeanDefinitions(root);
}

protected void doRegisterBeanDefinitions(Element root) {
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

    	// 处理 beans 标签的 profile 属性
		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isInfoEnabled()) {
						logger.info("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}

		preProcessXml(root); // 模板设计模式，解析前处理，留给子类实现
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root); // 模板设计模式，解析后处理，留给子类实现

		this.delegate = parent;
}

protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    	// 判断是否是默认标签
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
                          // 处理默认标签
						parseDefaultElement(ele, delegate);
					}
					else {
                          // 处理自定义标签
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
             // 处理自定义标签
			delegate.parseCustomElement(root);
		}
}
```

​		对于根节点或者子节点如果是默认命名空间的话则采用`parseDefaultElement`方法进行解析，否则使用`delegate.parseCustomElement`方法对自定义命名空间

进行解析。而判断是否默认命名空间还是自定义命名空间的办法其实是使用`node.getNamespaceURI()`获取命名空间，并与`Spring`中固定的命名空间

`http:/www.springframework.org/schema/beans`进行比对。如果一致则认为是默认，否则就认为是自定义。





### 标签解析

#### 默认标签

​		默认标签的解析是在`DefaultBeanDefinitionDocumentReader#parseDefaultElement`中进行的：

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    	// 处理 import 标签
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
    	// 处理 alias 标签
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
    	// 处理 bean 标签
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
    	// 处理 beans 标签
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
}
```



##### bean标签

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
}
```

​		处理过程：

​				1、首先委托`BeanDefinitionDelegate`类的`parseBeanDefinitionElement`方法进行元素解析，返回`BeanDefinitionHolder`类型的实例`bdHolder`，经过这

​		个方法后，`bdHolder`实例已经包含我们配置文件中配置的各种属性了。

​				2、当返回的`bdHolder`不为空的情况下若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。

​				3、解析完成后，需要对解析后的`bdHolder`进行注册，同样，注册操作委批给`BeanDefinitionReaderUtils`的`registerBeanDefinition`方法。

​				4、最后发出响应事件，通知相关的监听器，这个`bean`已经加载完成了。

![](image/QQ截图20220112092546.png)



​		首先进行`BeanDefinition`的解析，即：

```java
BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
```

```java
// BeanDefinitionParserDelegate 类
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
}

public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
		String id = ele.getAttribute(ID_ATTRIBUTE); // 解析 id 属性
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE); // 解析 name 属性

   		// 分割 name 属性
		List<String> aliases = new ArrayList<>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}

		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean); // 解析标签中的其他属性，并生成BeanDefinition
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
                    // 如果不存在 beanName 则根据 Spring 中提供的命名规则为当前 bean 生成对应的 beanName
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
}
```

​		上述代码的整体逻辑：

​				1、提取元素中的`id`以及`name`属性。

​				2、进一步解析其他所有属性并统一封装至`GenericBeanDefinition`类型的实例中。

​				3、如果检测到`bean`没有指定`beanName`，那么使用默认规则为此`bean`生成`beanName`。

​				4、将获取到的信息封装到`BeanDefinitionHolder`的实例中。

​		解析其他属性：

```java
// BeanDefinitionParserDelegate 类
public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, @Nullable BeanDefinition containingBean) {

		this.parseState.push(new BeanEntry(beanName));

    	// 解析 class 属性
		String className = null;
		if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
			className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
		}
    	// 解析 parent 属性
		String parent = null;
		if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
			parent = ele.getAttribute(PARENT_ATTRIBUTE);
		}

		try {
             // 创建用于承载属性的 AbstractBeanDefinition 类型的 GenericBeanDefinition
			AbstractBeanDefinition bd = createBeanDefinition(className, parent);
			// 硬编码解析默认 bean 的各种属性
			parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
             // 提取 description
			bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
			// 解析元数据
			parseMetaElements(ele, bd);
             // 解析 lookup-method 属性
			parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
             // 解析 replaced-method 属性
			parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
			// 解析构造函数
			parseConstructorArgElements(ele, bd);
             // 解析 property 子元素
			parsePropertyElements(ele, bd);
             // 解析qualifier 子元素
			parseQualifierElements(ele, bd);

			bd.setResource(this.readerContext.getResource());
			bd.setSource(extractSource(ele));

			return bd;
		}
		catch (ClassNotFoundException ex) {
			error("Bean class [" + className + "] not found", ele, ex);
		}
		catch (NoClassDefFoundError err) {
			error("Class that bean class [" + className + "] depends on not found", ele, err);
		}
		catch (Throwable ex) {
			error("Unexpected failure during bean definition parsing", ele, ex);
		}
		finally {
			this.parseState.pop();
		}

		return null;
}
```

​		1、创建`BeanDefinition`。`BeanDefinition`是一个接口，在`Spring`中存在三种实现：`RootBeanDefinition、ChildBeanDefinition、`

`GenericBeanDefinition`。三种实现均继承了`AbstractBeanDefiniton`，其中`BeanDefinition`是配置文件`<bean>`元素标签在容器中的内部表示形式。`<bean>`元素

标签拥有`class、scope、lazy-init`等配置属性，`BeanDefinition`则提供相应的`beanclass、 scope、lazyInit`属性，`BeanDefinition`和`<bean>`中的属性是一一对

应的。`RootBeanDetinItlon`是最常用的实现类，它对应一般性的`<bean>`元素标签，`GenericBeanDefinition`是自`2.5`版本以后新加入的`bean`文件配置属性定义

类，是一站式服务类。

​		在配置文件中可以定义父`<bean>`和子`<bean>`，父`<bean>`用`RootBeanDefinition`表示，而子`<bean>`用`ChildBeanDefiniton`表示，而没有父`<bean>`的

`<bean>`就使用`RootBeanDefinition`表示。`AbstractBeanDefinition`对两者共同的类信息进行抽象。

​		`Spring`通过`BeanDefinition`将配置文件中的`<bean>`配置信息转换为容器的内部表示，并将这些`BeanDefiniton`注册到`BeanDefinitonRegistry`中。`Spring`

容器的`BeanDefinitionRegistry`就像是`Spring`配置信息的内存数据库，主要是以`map`的形式保存，后续操作直接从`BeanDefinitionRegistry`中读取配置信息。

![](image/QQ截图20220112100034.png)

```java
protected AbstractBeanDefinition createBeanDefinition(@Nullable String className, @Nullable String parentName)
			throws ClassNotFoundException {

		return BeanDefinitionReaderUtils.createBeanDefinition(
				parentName, className, this.readerContext.getBeanClassLoader());
}
```

```java
// BeanDefinitionReaderUtils 类
public static AbstractBeanDefinition createBeanDefinition(
			@Nullable String parentName, @Nullable String className, @Nullable ClassLoader classLoader) throws ClassNotFoundException {

		GenericBeanDefinition bd = new GenericBeanDefinition();
    	// parentName 可能为空
		bd.setParentName(parentName);
		if (className != null) {
			if (classLoader != null) {
                 // 如果 classLoader 不为空，则使用传入的 classLoader 加载类对象，否则只是记录 className
				bd.setBeanClass(ClassUtils.forName(className, classLoader));
			}
			else {
				bd.setBeanClassName(className);
			}
		}
		return bd;
}
```

​		2、解析属性：

```java
public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
			@Nullable BeanDefinition containingBean, AbstractBeanDefinition bd) {
		// 解析 singleton 属性
		if (ele.hasAttribute(SINGLETON_ATTRIBUTE)) {
			error("Old 1.x 'singleton' attribute in use - upgrade to 'scope' declaration", ele);
		}
    	// 解析 scope 属性
		else if (ele.hasAttribute(SCOPE_ATTRIBUTE)) {
			bd.setScope(ele.getAttribute(SCOPE_ATTRIBUTE));
		}
		else if (containingBean != null) {
			// Take default from containing bean in case of an inner bean definition.
             // 在嵌入 beanDifinition 情况下且没有单独指定 scope 属性则使用父类默认的属性
			bd.setScope(containingBean.getScope());
		}
		// 解析 abstract 属性
		if (ele.hasAttribute(ABSTRACT_ATTRIBUTE)) {
			bd.setAbstract(TRUE_VALUE.equals(ele.getAttribute(ABSTRACT_ATTRIBUTE)));
		}
		// 解析 lazy-init 属性
		String lazyInit = ele.getAttribute(LAZY_INIT_ATTRIBUTE);
		if (isDefaultValue(lazyInit)) {
			lazyInit = this.defaults.getLazyInit();
		}
    	// 没有设置或设置为其他字符都会被设置为 false
		bd.setLazyInit(TRUE_VALUE.equals(lazyInit));
		// 解析 autowire 属性
		String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
		bd.setAutowireMode(getAutowireMode(autowire));
		// 解析 depends-on 属性
		if (ele.hasAttribute(DEPENDS_ON_ATTRIBUTE)) {
			String dependsOn = ele.getAttribute(DEPENDS_ON_ATTRIBUTE);
			bd.setDependsOn(StringUtils.tokenizeToStringArray(dependsOn, MULTI_VALUE_ATTRIBUTE_DELIMITERS));
		}
		// 解析 autowire-candidate 属性
		String autowireCandidate = ele.getAttribute(AUTOWIRE_CANDIDATE_ATTRIBUTE);
		if (isDefaultValue(autowireCandidate)) {
			String candidatePattern = this.defaults.getAutowireCandidates();
			if (candidatePattern != null) {
				String[] patterns = StringUtils.commaDelimitedListToStringArray(candidatePattern);
				bd.setAutowireCandidate(PatternMatchUtils.simpleMatch(patterns, beanName));
			}
		}
		else {
			bd.setAutowireCandidate(TRUE_VALUE.equals(autowireCandidate));
		}
		// 解析 primary 属性
		if (ele.hasAttribute(PRIMARY_ATTRIBUTE)) {
			bd.setPrimary(TRUE_VALUE.equals(ele.getAttribute(PRIMARY_ATTRIBUTE)));
		}
		// 解析 init-method 属性
		if (ele.hasAttribute(INIT_METHOD_ATTRIBUTE)) {
			String initMethodName = ele.getAttribute(INIT_METHOD_ATTRIBUTE);
			bd.setInitMethodName(initMethodName);
		}
		else if (this.defaults.getInitMethod() != null) {
			bd.setInitMethodName(this.defaults.getInitMethod());
			bd.setEnforceInitMethod(false);
		}
		// 解析 destroy-method 属性
		if (ele.hasAttribute(DESTROY_METHOD_ATTRIBUTE)) {
			String destroyMethodName = ele.getAttribute(DESTROY_METHOD_ATTRIBUTE);
			bd.setDestroyMethodName(destroyMethodName);
		}
		else if (this.defaults.getDestroyMethod() != null) {
			bd.setDestroyMethodName(this.defaults.getDestroyMethod());
			bd.setEnforceDestroyMethod(false);
		}
		// 解析 factory-method 属性
		if (ele.hasAttribute(FACTORY_METHOD_ATTRIBUTE)) {
			bd.setFactoryMethodName(ele.getAttribute(FACTORY_METHOD_ATTRIBUTE));
		}
    	// 解析 factory-bean 属性
		if (ele.hasAttribute(FACTORY_BEAN_ATTRIBUTE)) {
			bd.setFactoryBeanName(ele.getAttribute(FACTORY_BEAN_ATTRIBUTE));
		}

		return bd;
}
```

​		3、解析子元素`meta`：

```java
public void parseMetaElements(Element ele, BeanMetadataAttributeAccessor attributeAccessor) {
    	// 获取当前节点的所有子元素
		NodeList nl = ele.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
             // 提取 meta
			if (isCandidateElement(node) && nodeNameEquals(node, META_ELEMENT)) {
				Element metaElement = (Element) node;
				String key = metaElement.getAttribute(KEY_ATTRIBUTE);
				String value = metaElement.getAttribute(VALUE_ATTRIBUTE);
                  // 使用 key、value 构造 BeanMetadataAttribute
				BeanMetadataAttribute attribute = new BeanMetadataAttribute(key, value);
				attribute.setSource(extractSource(metaElement));
                  // 记录信息
				attributeAccessor.addMetadataAttribute(attribute);
			}
		}
}
```

​		4、解析子元素`lookup-method`：`lookuo-method`，获取器注入，是一种特殊的方法注入，将一个方法生命为某种类型的`bean`，但实际要返回的`bean`是在配

置文件中配置的。

```java
public void parseLookupOverrideSubElements(Element beanEle, MethodOverrides overrides) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
             // 仅当在 Spring 默认 bean 的子元素下且为 <lookup-method 时有效
			if (isCandidateElement(node) && nodeNameEquals(node, LOOKUP_METHOD_ELEMENT)) {
				Element ele = (Element) node;
                 // 获取要修饰的方法
				String methodName = ele.getAttribute(NAME_ATTRIBUTE);
                 // 获取配置返回的 bean
				String beanRef = ele.getAttribute(BEAN_ELEMENT);
				LookupOverride override = new LookupOverride(methodName, beanRef);
				override.setSource(extractSource(ele));
				overrides.addOverride(override);
			}
		}
}
```

​		5、解析子元素`replaced-method`：`replaced-method`，方法替换，可以在运行时用新的方法替换现有的方法，不仅可以动态替换返回实体`bean`，还能动态更

改原有方法的逻辑。

```java
public void parseReplacedMethodSubElements(Element beanEle, MethodOverrides overrides) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			// 仅当在 Spring 默认 bean 的子元素下且为 <lookup-method 时有效
			if (isCandidateElement(node) && nodeNameEquals(node, REPLACED_METHOD_ELEMENT)) {
				Element replacedMethodEle = (Element) node;
                 // 提取要替换的旧方法
				String name = replacedMethodEle.getAttribute(NAME_ATTRIBUTE);
                 // 提取对应的新的替换方法
				String callback = replacedMethodEle.getAttribute(REPLACER_ATTRIBUTE);
				ReplaceOverride replaceOverride = new ReplaceOverride(name, callback);
				// Look for arg-type match elements.
				List<Element> argTypeEles = DomUtils.getChildElementsByTagName(replacedMethodEle, ARG_TYPE_ELEMENT);
				for (Element argTypeEle : argTypeEles) {
                     // 记录参数
					String match = argTypeEle.getAttribute(ARG_TYPE_MATCH_ATTRIBUTE);
					match = (StringUtils.hasText(match) ? match : DomUtils.getTextValue(argTypeEle));
					if (StringUtils.hasText(match)) {
						replaceOverride.addTypeIdentifier(match);
					}
				}
				replaceOverride.setSource(extractSource(replacedMethodEle));
				overrides.addOverride(replaceOverride);
			}
		}
}
```

​		6、解析子元素`constructor-arg`：

```java
public void parseConstructorArgElements(Element beanEle, BeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, CONSTRUCTOR_ARG_ELEMENT)) {
                 // 解析 constructor-arg
				parseConstructorArgElement((Element) node, bd);
			}
		}
}
```

```java
public void parseConstructorArgElement(Element ele, BeanDefinition bd) {
    	// 提取 index 属性
		String indexAttr = ele.getAttribute(INDEX_ATTRIBUTE);
    	// 提取 type 属性
		String typeAttr = ele.getAttribute(TYPE_ATTRIBUTE);
    	// 提取 name 属性
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
		if (StringUtils.hasLength(indexAttr)) {
			try {
				int index = Integer.parseInt(indexAttr);
				if (index < 0) {
					error("'index' cannot be lower than 0", ele);
				}
				else {
					try {
						this.parseState.push(new ConstructorArgumentEntry(index));
                          // 解析 ele 对应的属性元素
						Object value = parsePropertyValue(ele, bd, null);
						ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
						if (StringUtils.hasLength(typeAttr)) {
							valueHolder.setType(typeAttr);
						}
						if (StringUtils.hasLength(nameAttr)) {
							valueHolder.setName(nameAttr);
						}
						valueHolder.setSource(extractSource(ele));
                          // 不允许重复指定相同参数
						if (bd.getConstructorArgumentValues().hasIndexedArgumentValue(index)) {
							error("Ambiguous constructor-arg entries for index " + index, ele);
						}
						else {
							bd.getConstructorArgumentValues().addIndexedArgumentValue(index, valueHolder);
						}
					}
					finally {
						this.parseState.pop();
					}
				}
			}
			catch (NumberFormatException ex) {
				error("Attribute 'index' of tag 'constructor-arg' must be an integer", ele);
			}
		}
		else {
             // 没有 index 属性则忽略此属性，自动寻找
			try {
				this.parseState.push(new ConstructorArgumentEntry());
				Object value = parsePropertyValue(ele, bd, null);
				ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
				if (StringUtils.hasLength(typeAttr)) {
					valueHolder.setType(typeAttr);
				}
				if (StringUtils.hasLength(nameAttr)) {
					valueHolder.setName(nameAttr);
				}
				valueHolder.setSource(extractSource(ele));
				bd.getConstructorArgumentValues().addGenericArgumentValue(valueHolder);
			}
			finally {
				this.parseState.pop();
			}
		}
}
```

​		配置了`index`属性：

​				1、解析`constructor-arg`的子元素。

​				2、使用`ConstructorArgumentValues.ValueHolder`类型来封装解析出来的元素。

​				3、将`type、name、index`属性一并封装在`ConstructorArgumentValues.ValueHolder`类型中并添加至当前`BeanDefinition`的`constructorArgumentValues`

​		的`indexedArgumentValues`属性中。

​		没有配置`index`属性：

​				1、解析`constructor-arg`的子元素。

​				2、使用`ConstructorArgumentValues.ValueHolder`类型来封装解析出来的元素。

​				3、将`type、name、index`属性一并封装在`ConstructorArgumentValues.ValueHolder`类型中并添加至当前`BeanDefinition`的`constructorArgumentValues`

​		的`genericArgumentValues`属性中。

​		解析`constructor-arg`中的各个属性：

```java
public Object parsePropertyValue(Element ele, BeanDefinition bd, @Nullable String propertyName) {
		String elementName = (propertyName != null ?
				"<property> element for property '" + propertyName + "'" :
				"<constructor-arg> element");

		// Should only have one child element: ref, value, list, etc.
    	// 一个属性只能对应一种类型
		NodeList nl = ele.getChildNodes();
		Element subElement = null;
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
            // 对应 description 或者 meta 不处理
			if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT) &&
					!nodeNameEquals(node, META_ELEMENT)) {
				// Child element is what we're looking for.
				if (subElement != null) {
					error(elementName + " must not contain more than one sub-element", ele);
				}
				else {
					subElement = (Element) node;
				}
			}
		}
		// 解析 constructor-arg 上的 ref 属性
		boolean hasRefAttribute = ele.hasAttribute(REF_ATTRIBUTE);
    	// 解析 constructor-arg 上的 value 属性
		boolean hasValueAttribute = ele.hasAttribute(VALUE_ATTRIBUTE);
		if ((hasRefAttribute && hasValueAttribute) ||
				((hasRefAttribute || hasValueAttribute) && subElement != null)) {
            /**
            	1、既有 ref 属性又有 value 属性
            	2、存在 ref 属性或 value 属性且还有子元素
            */
			error(elementName +
					" is only allowed to contain either 'ref' attribute OR 'value' attribute OR sub-element", ele);
		}

		if (hasRefAttribute) {
            // ref 属性的处理，使用 RuntimeBeanReference 封装对应的 ref 名称
			String refName = ele.getAttribute(REF_ATTRIBUTE);
			if (!StringUtils.hasText(refName)) {
				error(elementName + " contains empty 'ref' attribute", ele);
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName);
			ref.setSource(extractSource(ele));
			return ref;
		}
		else if (hasValueAttribute) {
            // value 属性的处理，使用 TypedStringValue 封装
			TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
			valueHolder.setSource(extractSource(ele));
			return valueHolder;
		}
		else if (subElement != null) {
            // 解析子元素
			return parsePropertySubElement(subElement, bd);
		}
		else {
			// Neither child element nor "ref" or "value" attribute found.
            // 即没有 ref 属性，也没有 value 属性，也没有子元素
			error(elementName + " must specify a ref or value", ele);
			return null;
		}
}
```

​		解析`constructor-arg`的子元素：

```java
public Object parsePropertySubElement(Element ele, @Nullable BeanDefinition bd) {
		return parsePropertySubElement(ele, bd, null);
}

public Object parsePropertySubElement(Element ele, @Nullable BeanDefinition bd, @Nullable String defaultValueType) {
		if (!isDefaultNamespace(ele)) {
			return parseNestedCustomElement(ele, bd);
		}
		else if (nodeNameEquals(ele, BEAN_ELEMENT)) {
			BeanDefinitionHolder nestedBd = parseBeanDefinitionElement(ele, bd);
			if (nestedBd != null) {
				nestedBd = decorateBeanDefinitionIfRequired(ele, nestedBd, bd);
			}
			return nestedBd;
		}
		else if (nodeNameEquals(ele, REF_ELEMENT)) {
			// A generic reference to any name of any bean.
			String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
			boolean toParent = false;
			if (!StringUtils.hasLength(refName)) {
				// A reference to the id of another bean in a parent context.
                 // 解析 parent
				refName = ele.getAttribute(PARENT_REF_ATTRIBUTE);
				toParent = true;
				if (!StringUtils.hasLength(refName)) {
					error("'bean' or 'parent' is required for <ref> element", ele);
					return null;
				}
			}
			if (!StringUtils.hasText(refName)) {
				error("<ref> element contains empty target attribute", ele);
				return null;
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName, toParent);
			ref.setSource(extractSource(ele));
			return ref;
		}
    	// 解析 idref 元素
		else if (nodeNameEquals(ele, IDREF_ELEMENT)) {
			return parseIdRefElement(ele);
		}
    	// 解析 value 子元素
		else if (nodeNameEquals(ele, VALUE_ELEMENT)) {
			return parseValueElement(ele, defaultValueType);
		}
    	// 解析 null 子元素
		else if (nodeNameEquals(ele, NULL_ELEMENT)) {
			// It's a distinguished null value. Let's wrap it in a TypedStringValue
			// object in order to preserve the source location.
			TypedStringValue nullHolder = new TypedStringValue(null);
			nullHolder.setSource(extractSource(ele));
			return nullHolder;
		}
    	// 解析 array 子元素
		else if (nodeNameEquals(ele, ARRAY_ELEMENT)) {
			return parseArrayElement(ele, bd);
		}
    	// 解析 list 子元素
		else if (nodeNameEquals(ele, LIST_ELEMENT)) {
			return parseListElement(ele, bd);
		}
    	// 解析 set 子元素
		else if (nodeNameEquals(ele, SET_ELEMENT)) {
			return parseSetElement(ele, bd);
		}
    	// 解析 map 子元素
		else if (nodeNameEquals(ele, MAP_ELEMENT)) {
			return parseMapElement(ele, bd);
		}
    	// 解析 props 子元素
		else if (nodeNameEquals(ele, PROPS_ELEMENT)) {
			return parsePropsElement(ele);
		}
		else {
			error("Unknown property sub-element: [" + ele.getNodeName() + "]", ele);
			return null;
		}
}
```



​		7、解析子元素`property`：

```java
public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
				parsePropertyElement((Element) node, bd);
			}
		}
}

public void parsePropertyElement(Element ele, BeanDefinition bd) {
    	// 获取配置元素中 name 的值
		String propertyName = ele.getAttribute(NAME_ATTRIBUTE);
		if (!StringUtils.hasLength(propertyName)) {
			error("Tag 'property' must have a 'name' attribute", ele);
			return;
		}
		this.parseState.push(new PropertyEntry(propertyName));
		try {
             // 不允许多次对同一属性配置
			if (bd.getPropertyValues().contains(propertyName)) {
				error("Multiple 'property' definitions for property '" + propertyName + "'", ele);
				return;
			}
			Object val = parsePropertyValue(ele, bd, propertyName);
			PropertyValue pv = new PropertyValue(propertyName, val);
			parseMetaElements(ele, pv);
			pv.setSource(extractSource(ele));
			bd.getPropertyValues().addPropertyValue(pv);
		}
		finally {
			this.parseState.pop();
		}
}
```



​		8、解析子元素`qualifier`：

```java
public void parseQualifierElements(Element beanEle, AbstractBeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ELEMENT)) {
				parseQualifierElement((Element) node, bd);
			}
		}
}

public void parseQualifierElement(Element ele, AbstractBeanDefinition bd) {
		String typeName = ele.getAttribute(TYPE_ATTRIBUTE);
		if (!StringUtils.hasLength(typeName)) {
			error("Tag 'qualifier' must have a 'type' attribute", ele);
			return;
		}
		this.parseState.push(new QualifierEntry(typeName));
		try {
			AutowireCandidateQualifier qualifier = new AutowireCandidateQualifier(typeName);
			qualifier.setSource(extractSource(ele));
			String value = ele.getAttribute(VALUE_ATTRIBUTE);
			if (StringUtils.hasLength(value)) {
				qualifier.setAttribute(AutowireCandidateQualifier.VALUE_KEY, value);
			}
			NodeList nl = ele.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ATTRIBUTE_ELEMENT)) {
					Element attributeEle = (Element) node;
					String attributeName = attributeEle.getAttribute(KEY_ATTRIBUTE);
					String attributeValue = attributeEle.getAttribute(VALUE_ATTRIBUTE);
					if (StringUtils.hasLength(attributeName) && StringUtils.hasLength(attributeValue)) {
						BeanMetadataAttribute attribute = new BeanMetadataAttribute(attributeName, attributeValue);
						attribute.setSource(extractSource(attributeEle));
						qualifier.addMetadataAttribute(attribute);
					}
					else {
						error("Qualifier 'attribute' tag must have a 'name' and 'value'", attributeEle);
						return;
					}
				}
			}
			bd.addQualifier(qualifier);
		}
		finally {
			this.parseState.pop();
		}
}
```

​		到此，`XML`文档就转换成了`GenericBeanDefinition`，`XML`中所有的配置都可以在`GenericBeanDefinition`找到对应的属性。但大部分通用的属性还是在

`AbstractBeanDefinition`中：

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor implements BeanDefinition, Cloneable {

	// bean 的作用范围
	private String scope = SCOPE_DEFAULT;
	// 是否是抽象
	private boolean abstractFlag = false;
	// 是否延迟加载
	private boolean lazyInit = false;
	// 自动注入模式，对应属性 autowire
	private int autowireMode = AUTOWIRE_NO;
	// 依赖检查，3.0 之后弃用了这个属性
	private int dependencyCheck = DEPENDENCY_CHECK_NONE;
	
	// 表示一个 bean 的实例化依赖另一个 bean 先实例化，对应属性 depend-on
	private String[] dependsOn;
	// 如果设置为 false ，容器在查找自动装配对象时，将不考虑该 bean ，即它不会被考虑作为其他 bean 自动装配的候选者，但该 bean 本身还是可以使用自动装配来注入其他 bean 的。对应属性 autowire-candidate
	private boolean autowireCandidate = true;
	// 自动装配时，当出现多个候选者时，将作为首选者，对应属性 primary
	private boolean primary = false;
	// 用于记录 Qualifier 对应子元素 qualifier
	private final Map<String, AutowireCandidateQualifier> qualifiers = new LinkedHashMap<>();

	@Nullable
	private Supplier<?> instanceSupplier;
	// 允许访问非公开的构造器和方法，程序设置
	private boolean nonPublicAccessAllowed = true;
    
	private boolean lenientConstructorResolution = true;

	// 对应属性 factory-bean
	private String factoryBeanName;

	// 对应属性 factory-method
	private String factoryMethodName;

	// 记录构造函数注入属性，对应属性 constructor-arg
	private ConstructorArgumentValues constructorArgumentValues;

	// 普通属性集合
	private MutablePropertyValues propertyValues;
	// 方法重写的持有者，记录 lookup-method、replaced-method 元素
	private MethodOverrides methodOverrides = new MethodOverrides();

	// 初始化方法，对应属性 init-method
	private String initMethodName;

	// 销毁方法，对应属性 destory-method
	private String destroyMethodName;
	// 是否执行 init-method ，程序设置
	private boolean enforceInitMethod = true;
	// 是否执行 destory-method ,程序设置
	private boolean enforceDestroyMethod = true;
	// 是否是用户定义的而不是应用程序本身定义的，创建 AOP 时为 true，程序设置
	private boolean synthetic = false;
	// 定义这个 bean 的应用, APPLICATION:用户，INFRASTRUCTURE:完全内部使用，与用户无关，SUPPORT:某些复杂配置的一部分，程序设置
	private int role = BeanDefinition.ROLE_APPLICATION;

	// bean 的描述信息
	private String description;
	// 这个 bean 定义的资源
	private Resource resource;
    ...
}
```

​		`bean`默认标签的解析与提取到此就结束了，但如果其中的子元素使用自定义的配置，这个配置并不是`bean`的形式出现，而是以属性的方式：

```java
// DefaultBeanDefinitionDocumentReader 类
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder); // 解析自定义子元素
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
}
```

```java
// BeanDefinitionParserDelegate 类
public BeanDefinitionHolder decorateBeanDefinitionIfRequired(Element ele, BeanDefinitionHolder originalDef) {
		return decorateBeanDefinitionIfRequired(ele, originalDef, null); // 第三个属性是父 bean，当对某个嵌套配置进行解析时传递，传递这个参数作用时使用其 scope 属性，以备子类在没有配置 scope 时默认使用父类的属性
}

public BeanDefinitionHolder decorateBeanDefinitionIfRequired(
			Element ele, BeanDefinitionHolder originalDef, @Nullable BeanDefinition containingBd) {

		BeanDefinitionHolder finalDefinition = originalDef;

		// Decorate based on custom attributes first.
		NamedNodeMap attributes = ele.getAttributes();
    	// 遍历所有属性，检查是否有适用于修饰的属性
		for (int i = 0; i < attributes.getLength(); i++) {
			Node node = attributes.item(i);
			finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
		}

		// Decorate based on custom nested elements.
		NodeList children = ele.getChildNodes();
    	// 遍历所有子节点，检查是否有适用于修饰的子元素
		for (int i = 0; i < children.getLength(); i++) {
			Node node = children.item(i);
			if (node.getNodeType() == Node.ELEMENT_NODE) {
				finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
			}
		}
		return finalDefinition;
}

/**
	默认标签的处理直接略过，只对自定义的标签或者说对 bean 的自定义属性感兴趣。在方法中实现了寻找自定义标签并根据自定义标签寻找命名空间处理器，并进行进一步的解析。
*/
public BeanDefinitionHolder decorateIfRequired(
			Node node, BeanDefinitionHolder originalDef, @Nullable BeanDefinition containingBd) {
		// 获取自定义标签的命名空间
		String namespaceUri = getNamespaceURI(node);
    	// 对于非默认标签进行修饰
		if (namespaceUri != null && !isDefaultNamespace(namespaceUri)) {
            // 根据命名空间找到对应的处理器
			NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
			if (handler != null) {
                // 进行修饰
				BeanDefinitionHolder decorated =
						handler.decorate(node, originalDef, new ParserContext(this.readerContext, this, containingBd));
				if (decorated != null) {
					return decorated;
				}
			}
			else if (namespaceUri.startsWith("http://www.springframework.org/")) {
				error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", node);
			}
			else {
				// A custom namespace, not to be handled by Spring - maybe "xml:...".
				if (logger.isDebugEnabled()) {
					logger.debug("No Spring NamespaceHandler found for XML schema namespace [" + namespaceUri + "]");
				}
			}
		}
		return originalDef;
}

public String getNamespaceURI(Node node) {
		return node.getNamespaceURI();
}

public boolean isDefaultNamespace(@Nullable String namespaceUri) {
    	// BEANS_NAMESPACE_URI = "http://www.springframework.org/schema/beans";
		return (!StringUtils.hasLength(namespaceUri) || BEANS_NAMESPACE_URI.equals(namespaceUri));
}
```

​		到此，配置文件所有的解析都已经完成，下面进入注册`BeanDefinition`：

```java
// DefaultBeanDefinitionDocumentReader 类
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry()); // 注册
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder)); // 通知监听器解析和注册完成，目前 Spring 对此事件未做处理
		}
}
```

```java
// BeanDefinitionReaderUtils 类
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
    	// 使用 beanName 做唯一标识注册
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition()); // 通过 beanName 注册

		// Register aliases for bean name, if any.
    	// 注册所有别名
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias); // 通过别名注册
			}
		}
}
```

​		`BeanDefinition`的注册，分为了两部分：

​				1、通过`beanName`注册：

```java
// DefaultListableBeanFactory 类
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
                /**
                	注册前的最后一次校验，主要是对于 AbstractBeanDefinition 中的 MethodOverrides 属性校验。
                	校验 MethodOverrides 是否与工厂方法并存或者 MethodOverrides 对应的方法根本不存在
                */
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
    	// 处理已经注册过的 beanName 情况
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
						"': There is already [" + existingDefinition + "] bound.");
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isWarnEnabled()) {
					logger.warn("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isInfoEnabled()) {
					logger.info("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
                // beanDefinitionMap 是全局变量，可能存在并发问题
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					if (this.manualSingletonNames.contains(beanName)) {
						Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
						updatedSingletons.remove(beanName);
						this.manualSingletonNames = updatedSingletons;
					}
				}
			}
			else {
				// Still in startup registration phase
                // 注册 beanDefinition 
				this.beanDefinitionMap.put(beanName, beanDefinition);
                // 记录 beanName
				this.beanDefinitionNames.add(beanName);
				this.manualSingletonNames.remove(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
            // 重置所有 beanName 对应的缓存
			resetBeanDefinition(beanName);
		}
}
```

​		2、通过别名注册：

```java
// SimpleAliasRegistry 类
public void registerAlias(String name, String alias) {
		Assert.hasText(name, "'name' must not be empty");
		Assert.hasText(alias, "'alias' must not be empty");
		synchronized (this.aliasMap) {
            // 如果 alias 与 beanName 相同，则不记录，并删除对应的 alias
			if (alias.equals(name)) {
				this.aliasMap.remove(alias);
				if (logger.isDebugEnabled()) {
					logger.debug("Alias definition '" + alias + "' ignored since it points to same name");
				}
			}
			else {
				String registeredName = this.aliasMap.get(alias);
				if (registeredName != null) {
					if (registeredName.equals(name)) {
						// An existing alias - no need to re-register
						return;
					}
                    // 如果 alias 不允许被覆盖，则抛出异常
					if (!allowAliasOverriding()) {
						throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +
								name + "': It is already registered for name '" + registeredName + "'.");
					}
					if (logger.isInfoEnabled()) {
						logger.info("Overriding alias '" + alias + "' definition for registered name '" +
								registeredName + "' with new target name '" + name + "'");
					}
				}
                // 当 A -> B 存在时，若再次出现 A -> C -> B 时抛出异常
				checkForAliasCircle(name, alias);
				this.aliasMap.put(alias, name);
                // 注册 alias
				if (logger.isDebugEnabled()) {
					logger.debug("Alias definition '" + alias + "' registered for name '" + name + "'");
				}
			}
		}
}
```



##### alias标签

​		`alias`标签的作用和`bean`标签中的`name`属性是一样的。

```java
// DefaultBeanDefinitionDocumentReader 类
protected void processAliasRegistration(Element ele) {
    	// 获取 beanName
		String name = ele.getAttribute(NAME_ATTRIBUTE);
    	// 获取 alias
		String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
		boolean valid = true;
		if (!StringUtils.hasText(name)) {
			getReaderContext().error("Name must not be empty", ele);
			valid = false;
		}
		if (!StringUtils.hasText(alias)) {
			getReaderContext().error("Alias must not be empty", ele);
			valid = false;
		}
		if (valid) {
			try {
                // 注册 alias
				getReaderContext().getRegistry().registerAlias(name, alias);
			}
			catch (Exception ex) {
				getReaderContext().error("Failed to register alias '" + alias +
						"' for bean with name '" + name + "'", ele, ex);
			}
            // 别名注册后通知监听器做处理
			getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
		}
}
```



##### import标签

​		如果将所有的配置都配置在一个文件中，文件会极其庞大，难以维护。可以将其拆开放置在不同的位置，然后在主配置文件中通过`import`标签引入。

```java
protected void importBeanDefinitionResource(Element ele) {
    	// 获取 resource 属性
		String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
		if (!StringUtils.hasText(location)) {
			getReaderContext().error("Resource location must not be empty", ele);
			return;
		}

		// Resolve system properties: e.g. "${user.dir}"
    	// 解析系统属性
		location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

		Set<Resource> actualResources = new LinkedHashSet<>(4);

		// Discover whether the location is an absolute or relative URI
    	// 判断 location 是绝对 URI 还是相对 URI
		boolean absoluteLocation = false;
		try {
			absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
		}
		catch (URISyntaxException ex) {
			// cannot convert to an URI, considering the location relative
			// unless it is the well-known Spring prefix "classpath*:"
		}

		// Absolute or relative?
    	// 绝对 URI 直接根据地址加载对应的配置文件
		if (absoluteLocation) {
			try {
				int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
				if (logger.isDebugEnabled()) {
					logger.debug("Imported " + importCount + " bean definitions from URL location [" + location + "]");
				}
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error(
						"Failed to import bean definitions from URL location [" + location + "]", ele, ex);
			}
		}
		else {
			// No URL -> considering resource location as relative to the current file.
            // 相对 URI 则根据相对地址计算出绝对地址
			try {
				int importCount;
                /**
                	尝试使用 Resource 的子类进行解析
                */
				Resource relativeResource = getReaderContext().getResource().createRelative(location);
				if (relativeResource.exists()) {
					importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
					actualResources.add(relativeResource);
				}
				else {
                    // 如果解析不成功，则使用默认的解析器 ResourcePatternResolver 进行解析
					String baseLocation = getReaderContext().getResource().getURL().toString();
					importCount = getReaderContext().getReader().loadBeanDefinitions(
							StringUtils.applyRelativePath(baseLocation, location), actualResources);
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Imported " + importCount + " bean definitions from relative location [" + location + "]");
				}
			}
			catch (IOException ex) {
				getReaderContext().error("Failed to resolve current resource location", ele, ex);
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to import bean definitions from relative location [" + location + "]",
						ele, ex);
			}
		}
    	// 解析后进行监听器激活处理
		Resource[] actResArray = actualResources.toArray(new Resource[0]);
		getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
}
```



##### 嵌入式beans

​		对于嵌入式`beans`标签来讲，与单独的配置文件并没有太大的差别，即递归调用`beans`的解析过程。



#### 自定义标签

​		当需要在`Spring`中配置较复杂的配置时，仅仅使用`Spring`提供的配置是远远不够的。`Spring`提供了可扩展`Schema`的支持`(`前提是要把`Spring`的`Core`包

加入项目中`)`：

​				1、创建一个需要扩展的组件。

​				2、定义一个`XSD`文件描述组件内容。

​				3、创建一个文件，实现`BeanDefinitionParser`接口，用来解析`XSD`文件中的定义和组件定义。

​				4、创建一个`Handler`文件，扩展自`NamespaceHandlerSupport`，目的是将组件注册到`Spring`容器。

​				5、编写`Spring.handlers`和`Spring.schemas`文件。



```java
// DefaultBeanDefinitionDocumentReader 类
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele); // 解析自定义标签
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root); // 解析自定义标签
		}
}

```

```java
// BeanDefinitionParserDelegate 类
public BeanDefinition parseCustomElement(Element ele) {
		return parseCustomElement(ele, null);
}

public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
    	// 获取对应的命名空间
		String namespaceUri = getNamespaceURI(ele);
		if (namespaceUri == null) {
			return null;
		}
    	// 根据命名空间定位对应的 NamespaceHandler
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
    	// 调用自定义的 NamespaceHandler 进行解析
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

​		在获取自定义标签处理器时，`readerContext`初始化的时候其属性`namespaceHandlerResolver`已经被初始化为了`DefaultNamespaceHandlerResolver`的实例：

```java
// DefaultNamespaceHandlerResolver 类
public NamespaceHandler resolve(String namespaceUri) {
    	// 获取已经配置的 handler 映射
		Map<String, Object> handlerMappings = getHandlerMappings();
    	// 根据命名空间找到对应的信息
		Object handlerOrClassName = handlerMappings.get(namespaceUri);
		if (handlerOrClassName == null) {
			return null;
		}
		else if (handlerOrClassName instanceof NamespaceHandler) {
            // 已经做过解析的情况，直接从缓存读取
			return (NamespaceHandler) handlerOrClassName;
		}
		else {
            // 没有做过解析，则返回类路径
			String className = (String) handlerOrClassName;
			try {
                // 使用反射转化为类
				Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
				if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
					throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
							"] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
				}
                // 初始化
				NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
                // 调用自定义 NamespaceHandler 的初始化方法
				namespaceHandler.init();
                // 记录到缓存
				handlerMappings.put(namespaceUri, namespaceHandler);
				return namespaceHandler;
			}
			catch (ClassNotFoundException ex) {
				throw new FatalBeanException("Could not find NamespaceHandler class [" + className +
						"] for namespace [" + namespaceUri + "]", ex);
			}
			catch (LinkageError err) {
				throw new FatalBeanException("Unresolvable class definition for NamespaceHandler class [" +
						className + "] for namespace [" + namespaceUri + "]", err);
			}
		}
}
```

​		得到了解析器以及要分析的元素后，`Spring`就可以将解析工作委托给自定义解析器去解析了：

```java
return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
```

​		但通常实现的自定义命名空间处理器中并没有实现`parse`方法，所以这个方法是父类中的实现：

```java
// NamespaceHandlerSupport
public BeanDefinition parse(Element element, ParserContext parserContext) {
    	// 寻找解析器
		BeanDefinitionParser parser = findParserForElement(element, parserContext);
    	// 解析元素
		return (parser != null ? parser.parse(element, parserContext) : null);
}
```

```java
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
    	// 获取元素名称
		String localName = parserContext.getDelegate().getLocalName(element);
    	// 找到对应的解析器
		BeanDefinitionParser parser = this.parsers.get(localName);
		if (parser == null) {
			parserContext.getReaderContext().fatal(
					"Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
		}
		return parser;
}
```

```java
// AbstractBeanDefinitionParser 类
public final BeanDefinition parse(Element element, ParserContext parserContext) {
		AbstractBeanDefinition definition = parseInternal(element, parserContext);
		if (definition != null && !parserContext.isNested()) {
			try {
				String id = resolveId(element, definition, parserContext);
				if (!StringUtils.hasText(id)) {
					parserContext.getReaderContext().error(
							"Id is required for element '" + parserContext.getDelegate().getLocalName(element)
									+ "' when used as a top-level tag", element);
				}
				String[] aliases = null;
				if (shouldParseNameAsAliases()) {
					String name = element.getAttribute(NAME_ATTRIBUTE);
					if (StringUtils.hasLength(name)) {
						aliases = StringUtils.trimArrayElements(StringUtils.commaDelimitedListToStringArray(name));
					}
				}
                // 将 AbstractBeanDefinition 转换为 BeanDefinitionHolder 并注册
				BeanDefinitionHolder holder = new BeanDefinitionHolder(definition, id, aliases);
				registerBeanDefinition(holder, parserContext.getRegistry());
				if (shouldFireEvents()) {
                    // 通知监听器进行处理
					BeanComponentDefinition componentDefinition = new BeanComponentDefinition(holder);
					postProcessComponentDefinition(componentDefinition);
					parserContext.registerComponent(componentDefinition);
				}
			}
			catch (BeanDefinitionStoreException ex) {
				String msg = ex.getMessage();
				parserContext.getReaderContext().error((msg != null ? msg : ex.toString()), element);
				return null;
			}
		}
		return definition;
}
```

```java
// AbstractSingleBeanDefinitionParser 类
protected final AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
		BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
		String parentName = getParentName(element);
		if (parentName != null) {
			builder.getRawBeanDefinition().setParentName(parentName);
		}
    	// 获取自定义标签中的 class 此时会调用自定义解析器
		Class<?> beanClass = getBeanClass(element);
		if (beanClass != null) {
			builder.getRawBeanDefinition().setBeanClass(beanClass);
		}
		else {
            // 若子类没有重写 getBeanClass ，则尝试调用子类的 getBeanClassName 方法
			String beanClassName = getBeanClassName(element);
			if (beanClassName != null) {
				builder.getRawBeanDefinition().setBeanClassName(beanClassName);
			}
		}
		builder.getRawBeanDefinition().setSource(parserContext.extractSource(element));
		BeanDefinition containingBd = parserContext.getContainingBeanDefinition();
		if (containingBd != null) {
			// Inner bean definition must receive same scope as containing bean.
            // 若存在父类则使用父类的 scope 属性
			builder.setScope(containingBd.getScope());
		}
		if (parserContext.isDefaultLazyInit()) {
			// Default-lazy-init applies to custom bean definitions as well.
            // 配置延迟加载
			builder.setLazyInit(true);
		}
    	// 调用子类的 doParse 方法
		doParse(element, parserContext, builder);
		return builder.getBeanDefinition();
}
```



### bean的加载

```java
Init init = (Init) factory.getBean("init");
```

```java
// AbstractBeanFactory
public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
}

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		// 提取对应的 beanName
		final String beanName = transformedBeanName(name);
		Object bean;

		/**
			创建 bean 时对于依赖注入，为了避免循环依赖，Spring 创建 bean 的原则是不等 bean 创建完成就会将创建 bean 的 ObjectFactory 提前曝光，即将 ObjectFactory 加入到缓存中，下个 bean 需要依赖上个 bean 时直接使用ObjectFactory
		*/
    	// 直接尝试从缓存获取或者 singletonFactories 中的 ObjectFactory 中获取
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			if (logger.isDebugEnabled()) {
				if (isSingletonCurrentlyInCreation(beanName)) {
					logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
							"' that is not fully initialized yet - a consequence of a circular reference");
				}
				else {
					logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
				}
			}
            // 存在例如 FactoryBean 的情况，并不直接返回实例本身而是返回指定方法返回的实例
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
            // 单例模式下才会尝试解决循环依赖
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

			// Check if bean definition exists in this factory.
			BeanFactory parentBeanFactory = getParentBeanFactory();
            // 如果 beandefinitionMap（所有已经加载的类） 中没有 beanName，则尝试从 parentBeanFactory 中检测
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent.
				String nameToLookup = originalBeanName(name);
				if (parentBeanFactory instanceof AbstractBeanFactory) {
					return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
							nameToLookup, requiredType, args, typeCheckOnly);
				}
				else if (args != null) {
					// Delegation to parent with explicit args.
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else {
					// No args -> delegate to standard getBean method.
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
			}

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}

			try {
                // 将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition,如果指定 BeanName 是子 Bean 的话同时会合并父类的相关属性
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// Guarantee initialization of beans that the current bean depends on.
				String[] dependsOn = mbd.getDependsOn();
                // 若存在依赖则需要递归实例化依赖的bean
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
                        // 缓存依赖调用
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}

				// Create bean instance.
                //实例化依赖的 bean 后便可以实例化当前 bean 本身
                //singleton 模式的创建
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							// Explicitly remove instance from singleton cache: It might have been put there
							// eagerly by the creation process, to allow for circular reference resolution.
							// Also remove any beans that received a temporary reference to the bean.
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					// It's a prototype -> create a new instance.
                     // prototype 模式的创建 (new)
					Object prototypeInstance = null;
					try {
						beforePrototypeCreation(beanName);
						prototypeInstance = createBean(beanName, mbd, args);
					}
					finally {
						afterPrototypeCreation(beanName);
					}
					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
				}

				else {
                    // 指定的 scope 上实例化 bean
					String scopeName = mbd.getScope();
					final Scope scope = this.scopes.get(scopeName);
					if (scope == null) {
						throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
					}
					try {
						Object scopedInstance = scope.get(beanName, () -> {
							beforePrototypeCreation(beanName);
							try {
								return createBean(beanName, mbd, args);
							}
							finally {
								afterPrototypeCreation(beanName);
							}
						});
						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
						throw new BeanCreationException(beanName,
								"Scope '" + scopeName + "' is not active for the current thread; consider " +
								"defining a scoped proxy for this bean if you intend to refer to it from a singleton",
								ex);
					}
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}

		// Check if required type matches the type of the actual bean instance.
   		// 检查需要的类型是否符合 bean 的实际类型
		if (requiredType != null && !requiredType.isInstance(bean)) {
			try {
				T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
				if (convertedBean == null) {
					throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
				}
				return convertedBean;
			}
			catch (TypeMismatchException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to convert bean '" + name + "' to required type '" +
							ClassUtils.getQualifiedName(requiredType) + "'", ex);
				}
				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
			}
		}
		return (T) bean;
}
```



#### 缓存中获取单例bean

```java
// DefaultSingletonBeanRegistry 类
public Object getSingleton(String beanName) {
		return getSingleton(beanName, true); // true 表示允许早期依赖
}

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    	// 检查缓存中是否存在实例
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            // 如果为空，则锁定全局变量并进行处理
			synchronized (this.singletonObjects) {
                // 如果 bean 正在加载则不处理
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
                    // 当某些方法需要提前初始化的时候则会调用 addSingletonFactory 方法将对应的 0bjectFactory 初始化策略存储在 singletonFactories
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
                        // //调用预先设定的 getObject 方法
						singletonObject = singletonFactory.getObject();
                        // //记录在缓存中，earlySingleton0bjects 和 singletonFactories 互斥
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
}
```

​		`singletonObjects`：用于保存`beanName`和`bean`实例之间的关系，`bean name -->beaninstance`。

​		`singletonFactories`：用于保存`beanName`和创建`bean`的工厂之间的关系，`bean name --> ObjectFactory`。

​		`earlySingletonObjects`：也是保存`beanName`和`bean`实例之间的关系，与`singletonObjects`的不同之处在于，当一个单例`bean`被放到这里面后，那么当

`bean`还在创建过程中，就可以通过`getBean`方法获取到了，其目的是用来检测循环引用。

​		`registeredSingletons`：用来保存当前所有已注册的`bean`。



#### FactoryBean中获取bean

​		`getObjectForBeanInstance`是个高频率使用的方法，无论是从缓存中获得`bean`还是根据不同的`scope`策略加载`bean`。总之，得到`bean`的实例后要做的第一

步就是调用这个方法来检测一下正确性，其实就是用于检测当前`bean`是否是`FactoryBean`类型的`bean`，如果是，那么需要调用该`bean`对应的`FactoryBean`实例

中的`getObject()`作为返回值。

```java
protected Object getObjectForBeanInstance(
			Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

		// Don't let calling code try to dereference the factory if the bean isn't a factory.
    	// 如果指定的 name 以 & 为前缀，并且 beanInstance 又不是 FactoryBean 类型，则校验不通过
		if (BeanFactoryUtils.isFactoryDereference(name)) {
			if (beanInstance instanceof NullBean) {
				return beanInstance;
			}
			if (!(beanInstance instanceof FactoryBean)) {
				throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
			}
		}

		// Now we have the bean instance, which may be a normal bean or a FactoryBean.
		// If it's a FactoryBean, we use it to create a bean instance, unless the
		// caller actually wants a reference to the factory.
    	// 处理 name 以 & 为前缀的情况
		if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
			return beanInstance;
		}
		// 加载 FactoryBean
		Object object = null;
		if (mbd == null) {
            // 尝试从缓存中加载 bean
			object = getCachedObjectForFactoryBean(beanName);
		}
		if (object == null) {
			// Return bean instance from factory.
            // 到这里 beanInstance 一定是 FactoryBean 类型
			FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
			// Caches object obtained from FactoryBean if it is a singleton.
            // containsBeanDefinition 检测 beanDefinitionMap 中也就是在所有已经加载的类中检测是否定义 beanName
			if (mbd == null && containsBeanDefinition(beanName)) {
                // 将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition,如果指定 beanName是子 Bean 的话同时会合并父类的相关属性
				mbd = getMergedLocalBeanDefinition(beanName);
			}
            // 是否是用户定义的而不是应用程序本身定义的
			boolean synthetic = (mbd != null && mbd.isSynthetic());
			object = getObjectFromFactoryBean(factory, beanName, !synthetic);
		}
		return object;
}
```

```java
// FactoryBeanRegistrySupport 类
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
    	// 如果是单例模式
		if (factory.isSingleton() && containsSingleton(beanName)) {
			synchronized (getSingletonMutex()) {
				Object object = this.factoryBeanObjectCache.get(beanName);
				if (object == null) {
					object = doGetObjectFromFactoryBean(factory, beanName);
					// Only post-process and store if not put there already during getObject() call above
					// (e.g. because of circular reference processing triggered by custom getBean calls)
					Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
					if (alreadyThere != null) {
						object = alreadyThere;
					}
					else {
						if (shouldPostProcess) {
							if (isSingletonCurrentlyInCreation(beanName)) {
								// Temporarily return non-post-processed object, not storing it yet..
								return object;
							}
							beforeSingletonCreation(beanName);
							try {
								object = postProcessObjectFromFactoryBean(object, beanName);
							}
							catch (Throwable ex) {
								throw new BeanCreationException(beanName,
										"Post-processing of FactoryBean's singleton object failed", ex);
							}
							finally {
								afterSingletonCreation(beanName);
							}
						}
						if (containsSingleton(beanName)) {
							this.factoryBeanObjectCache.put(beanName, object);
						}
					}
				}
				return object;
			}
		}
		else {
			Object object = doGetObjectFromFactoryBean(factory, beanName);
			if (shouldPostProcess) {
				try {
                    // 调用 objectFactory 的后处理器
					object = postProcessObjectFromFactoryBean(object, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
				}
			}
			return object;
		}
}

private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
			throws BeanCreationException {

		Object object;
		try {
            // 需要权限验证
			if (System.getSecurityManager() != null) {
				AccessControlContext acc = getAccessControlContext();
				try {
					object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
                // 直接调用 getObject 方法
				object = factory.getObject();
			}
		}
		catch (FactoryBeanNotInitializedException ex) {
			throw new BeanCurrentlyInCreationException(beanName, ex.toString());
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
		}

		// Do not accept a null value for a FactoryBean that's not fully
		// initialized yet: Many FactoryBeans just return null then.
		if (object == null) {
			if (isSingletonCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(
						beanName, "FactoryBean which is currently in creation returned null from getObject");
			}
			object = new NullBean();
		}
		return object;
}
```

```java
// AbstractAutowireCapableBeanFactory 类
protected Object postProcessObjectFromFactoryBean(Object object, String beanName) {
		return applyBeanPostProcessorsAfterInitialization(object, beanName);
}

public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
}
```

​		`Spring`获取`bean`的规则中有：尽可能保证所有`bean`初始化后都会调用注册的`BeanPostProcessor`的`postProcessAfterInitialization`方法进行处理。



#### 获取单例

​		如果缓存中没有，则需要从头开始`bean`的加载过程：

```java
// DefaultSingletonBeanRegistry 类
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
            // 首先检查对应的 bean 是否已经加载过
			Object singletonObject = this.singletonObjects.get(beanName);
            // 为空才可以进行 singleton 的 bean 的初始化
			if (singletonObject == null) {
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
                    // 初始化 bean
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// Has the singleton object implicitly appeared in the meantime ->
					// if yes, proceed with it since the exception indicates that state.
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
					afterSingletonCreation(beanName);
				}
                // 加入缓存
				if (newSingleton) {
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
}

//这个函数中做了一个很重要的操作:记录加载状态，也就是通过 this.singletonsCurrentlyInCreation.add(beanName) 将当前正要创建的 bean 记录在缓存中，这样便可以对循环依赖进行检测。
protected void beforeSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}
}
// 移除缓存中 bean 的正在加载状态的记录
protected void afterSingletonCreation(String beanName) {
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
			throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
		}
}
// 将结果记录至缓存并删除加载 bean 过程中所记录的各种辅助状态。
protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.put(beanName, singletonObject);
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
}
```



#### 准备创建bean

```java
// AbstractAutowireCapableBeanFactory 类
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Creating instance of bean '" + beanName + "'");
        }

        RootBeanDefinition mbdToUse = mbd;
    	// 锁定 class ，根据设置的 class 属性或者根据 className 来解析 Class
        Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
            mbdToUse = new RootBeanDefinition(mbd);
            mbdToUse.setBeanClass(resolvedClass);
        }

        try {
            // 验证及准备覆盖的方法
            mbdToUse.prepareMethodOverrides();
        } catch (BeanDefinitionValidationException var9) {
            throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var9);
        }

        Object beanInstance;
        try {
            // 给 BeanPostProcessors 一个机会来返回代理来替代真正的实例
            beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
            // 短路判断，AOP 的前置通知基于此
            if (beanInstance != null) {
                return beanInstance;
            }
        } catch (Throwable var10) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var10);
        }

        try {
            beanInstance = this.doCreateBean(beanName, mbdToUse, args);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Finished creating instance of bean '" + beanName + "'");
            }

            return beanInstance;
        } catch (ImplicitlyAppearedSingletonException | BeanCreationException var7) {
            throw var7;
        } catch (Throwable var8) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", var8);
        }
}


protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		if (logger.isTraceEnabled()) {
			logger.trace("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

    	// 锁定 class ，根据设置的 class 属性或者根据 className 来解析 Class
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
            // 验证及准备覆盖的方法
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}

		try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
            // 给 BeanPostProcessors 一个机会来返回代理来替代真正的实例
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
            // 短路判断，AOP 的前置通知基于此
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
		}
}
```



##### 处理override属性

​		`Spring`配置中是存在`lookup-method`和`replace-method`，这两个配置的加载其实就是将配置统一存放在`BeanDefinition`中的`methodOverrides`属性里。

```java
// AbstractBeanDefinition 类
public void prepareMethodOverrides() throws BeanDefinitionValidationException {
        if (this.hasMethodOverrides()) {
            this.getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);
        }

}
protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {
    	// 获取对应类中对应方法名的个数
        int count = ClassUtils.getMethodCountForName(this.getBeanClass(), mo.getMethodName());
        if (count == 0) {
            throw new BeanDefinitionValidationException("Invalid method override: no method with name '" + mo.getMethodName() + "' on class [" + this.getBeanClassName() + "]");
        } else {
            if (count == 1) {
                // 标记 MethodOverride 暂未被覆盖，避免参数类型检查的开销
                mo.setOverloaded(false);
            }

        }
}
```



##### 前置处理

```java
// AbstractAutowireCapableBeanFactory 类
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
        Object bean = null;
    	// 尚未被解析
        if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
            if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
                Class<?> targetType = this.determineTargetType(beanName, mbd);
                if (targetType != null) {
                    // 对后处理器中的所有 InstantiationAwareBeanPostProcessor 类型的后处理器进行 postProcessBeforeInstantiation 方法调用
                    bean = this.applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                    if (bean != null) {
                        // BeanPostProcessor 的 postProcessAfterInitialization 方法的调用
                        bean = this.applyBeanPostProcessorsAfterInitialization(bean, beanName);
                    }
                }
            }

            mbd.beforeInstantiationResolved = bean != null;
        }

        return bean;
}

public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
}

public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
}
```



#### 循环依赖

​		1、构造器循环依赖

​				表示通过构造器注入构成的循环依赖，此依赖是无法解决的，只能抛出`BeanCurrentlyInCreationException`异常表示循环依赖。`Spring`容器将每一个正

​		在创建的`bean`标识符放在一个`"`当前创建`bean`池`"`中，`bean`标识符在创建过程中将一直保持在这个池中，因此如果在创建`bean`过程中发现自己已经在池

​		里时，将抛出`BeanCurrentlyInCreationException`异常表示循环依赖；而对于创建完毕的`bean`将从池中清除掉。

​		2、`setter`循环依赖

​				表示通过`setter`注入方式构成的循环依赖。对于`setter`注入造成的依赖是通过`Spring`容器提前暴露刚完成构造器注入但未完成其他步骤的`bean`来完

​		成的，而且只能解决单例作用域的`bean`循环依赖。通过提前暴露一个单例工厂方法，从而使其他`bean`能引用到该`bean`。

```java
// AbstractAutowireCapableBeanFactory 类
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		...
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)); // 暴露单例工厂
		}
		...
}
```

​		3、`prototype`范围的依赖处理

​				对于`prototype`作用域`bean`，`Spring`容器无法完成依赖注入，因为`Spring`容器不进行缓存`prototype`作用域的`bean`，因此无法提前暴露一个创建中

​		的`bean`。





#### 创建bean

​		当经历过`resolveBeforeInstantiation`方法后，程序有两个选择，如果创建了代理或者说重写了`InstantiationAwareBeanPostProcessor`的

`postProcessBeforeInstantiation`方法并在方法`postProcessBeforeInstantiation`中改变了`bean`，则直接返回就可以了，否则需要进行常规`bean`的创建。而这常

规`bean`的创建就是在`doCreateBean中`完成的。

```java
// AbstractAutowireCapableBeanFactory 类
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
            // 根据指定 bean 使用对应的策略创建新的实例
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
                    // 应用 MergedBeanDefinitionPostProcessors，Autowired 注解通过此方法实现
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
    	// 是否需要提前曝光：单例 & 允许循环依赖 & 当前 bean 正在创建中，检测循环依赖
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
            // 为避免后期循环依赖，可以在 bean 初始化完成前将创建实例的 ObjectFactory 加入工厂
            // getEarlyBeanReference ,AOP 在这里将 advice 动态织入 bean 中，若没有则直接返回 bean，不做任何处理
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
            // 对 bean 进行填充，将各个属性值注入。可能存在依赖其他 bean 的属性，则会递归初始化依赖 bean
			populateBean(beanName, mbd, instanceWrapper);
            // 调用初始化方法，比如 init-method
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
            // earlySingletonReference 只有在检测到有循环依赖的情况下才会不为空
			if (earlySingletonReference != null) {
                
                // 如果 exposedObject 没有在初始化方法中被改变，即没有被增强
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
                        // 检测依赖
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
                   	/** 因为 bean 创建后其依赖的 bean 一定是已经创建的,actualDependentBeans 不为空则表示当前 bean 创建后其依赖的 bean 却没有全部创					建完，也就是存在循环依赖
                    */
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
            // 根据 scope 注册 bean
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
}
```



##### 创建 bean 的实例

```java
                                                                      -// AbstractAutowireCapableBeanFactory 类
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
    	// 解析 class
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}
		// 如果工厂方法不为空则使用工厂方法初始化策略
		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// Shortcut when re-creating the same bean...
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
                // 一个类有多个构造函数，每个构造函数都有不同的参数，所以调用前需要先根据参数锁定构造函数或对应的工厂方法
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
    	// 如果已经解析过则使用解析好的构造函数不需要再次锁定
		if (resolved) {
			if (autowireNecessary) {
                // 构造函数自动注入
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
                // 使用默认构造函数构造
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
    	// 需要根据参数解析构造函数
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
            // 构造函数自动注入
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		// No special handling: simply use no-arg constructor.
    	// 使用默认构造函数
		return instantiateBean(beanName, mbd);
}
```



###### 带有参数的构造函数实例化

```java
// ConstructorResolver 类
public BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd,
			@Nullable Constructor<?>[] chosenCtors, @Nullable Object[] explicitArgs) {

		BeanWrapperImpl bw = new BeanWrapperImpl();
		this.beanFactory.initBeanWrapper(bw);

		Constructor<?> constructorToUse = null;
		ArgumentsHolder argsHolderToUse = null;
		Object[] argsToUse = null;
		// explicitArgs 通过 getBean 方法传入，如果 getBean 方法调用的时候指定方法参数，那么直接使用
		if (explicitArgs != null) {
			argsToUse = explicitArgs;
		}
		else {
            // 如果在 getBean 方法时候没有指定，则尝试从配置文件中解析
			Object[] argsToResolve = null;
            // 尝试从缓存中获取
			synchronized (mbd.constructorArgumentLock) {
				constructorToUse = (Constructor<?>) mbd.resolvedConstructorOrFactoryMethod;
				if (constructorToUse != null && mbd.constructorArgumentsResolved) {
					// Found a cached constructor...
                    // 从缓存中取
					argsToUse = mbd.resolvedConstructorArguments;
					if (argsToUse == null) {
                        // 配置的构造函数参数
						argsToResolve = mbd.preparedConstructorArguments;
					}
				}
			}
            // 如果缓存中存在
            
			if (argsToResolve != null) {
                // 解析参数类型，如给定方法的构造函数A(int,int)则通过此方法后就会把配置中的//( "1"，"1")转换为（ 1,1 )
			   // 缓存中的值可能是原始值也可能是最终值
				argsToUse = resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve, true);
			}
		}
    	
    	// 没有被缓存
		if (constructorToUse == null || argsToUse == null) {
			// Take specified constructors, if any.
			Constructor<?>[] candidates = chosenCtors;
			if (candidates == null) {
				Class<?> beanClass = mbd.getBeanClass();
				try {
					candidates = (mbd.isNonPublicAccessAllowed() ?
							beanClass.getDeclaredConstructors() : beanClass.getConstructors());
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Resolution of declared constructors on bean Class [" + beanClass.getName() +
							"] from ClassLoader [" + beanClass.getClassLoader() + "] failed", ex);
				}
			}

			if (candidates.length == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
				Constructor<?> uniqueCandidate = candidates[0];
				if (uniqueCandidate.getParameterCount() == 0) {
					synchronized (mbd.constructorArgumentLock) {
						mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
						mbd.constructorArgumentsResolved = true;
						mbd.resolvedConstructorArguments = EMPTY_ARGS;
					}
					bw.setBeanInstance(instantiate(beanName, mbd, uniqueCandidate, EMPTY_ARGS));
					return bw;
				}
			}

			// Need to resolve the constructor.
			boolean autowiring = (chosenCtors != null ||
					mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
			ConstructorArgumentValues resolvedValues = null;

			int minNrOfArgs;
			if (explicitArgs != null) {
				minNrOfArgs = explicitArgs.length;
			}
			else {
                // 提起配置文件中的配置的构造函数参数
				ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
                // 用于承载解析后的构造函数参数的值
				resolvedValues = new ConstructorArgumentValues();
                // 能解析到的参数个数
				minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
			}
			// 排序给定的构造函数， public 构造函数优先、参数数量降序，非 public 构造函数参数数量降序
			AutowireUtils.sortConstructors(candidates);
			int minTypeDiffWeight = Integer.MAX_VALUE;
			Set<Constructor<?>> ambiguousConstructors = null;
			LinkedList<UnsatisfiedDependencyException> causes = null;

			for (Constructor<?> candidate : candidates) {

				int parameterCount = candidate.getParameterCount();

				if (constructorToUse != null && argsToUse != null && argsToUse.length > parameterCount) {
					// Already found greedy constructor that can be satisfied ->
					// do not look any further, there are only less greedy constructors left.
                    // 如果已经找到选用的构造函数或者需要的参数个数小于当前的构造函数参数个数则终止，因为已经按照参数个数降序排列
					break;
				}
				if (parameterCount < minNrOfArgs) {
                    // 参数个数不相等
					continue;
				}

				ArgumentsHolder argsHolder;
				Class<?>[] paramTypes = candidate.getParameterTypes();
				if (resolvedValues != null) {
                    // 有参数则根据值构造对应参数类型的参数
					try {
						String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, parameterCount);
						if (paramNames == null) {
                            // 注解上获取参数名称
							ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
							if (pnd != null) {
                                // 获取指定构造函数的参数名称
								paramNames = pnd.getParameterNames(candidate);
							}
						}
                        // 根据名称和数据类型创建参数持有者
						argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
								getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
					}
					catch (UnsatisfiedDependencyException ex) {
						if (logger.isTraceEnabled()) {
							logger.trace("Ignoring constructor [" + candidate + "] of bean '" + beanName + "': " + ex);
						}
						// Swallow and try next constructor.
						if (causes == null) {
							causes = new LinkedList<>();
						}
						causes.add(ex);
						continue;
					}
				}
				else {
					// Explicit arguments given -> arguments length must match exactly.
					if (parameterCount != explicitArgs.length) {
						continue;
					}
                    // 构造函数没有参数的情况
					argsHolder = new ArgumentsHolder(explicitArgs);
				}
				// 探测是否有不确定性的构造函数存在，例如不同构造函数的参数是父子关系
				int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
						argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
				// Choose this constructor if it represents the closest match.
                // 如果它代表这当前最接近的匹配，则选择作为构造函数
				if (typeDiffWeight < minTypeDiffWeight) {
					constructorToUse = candidate;
					argsHolderToUse = argsHolder;
					argsToUse = argsHolder.arguments;
					minTypeDiffWeight = typeDiffWeight;
					ambiguousConstructors = null;
				}
				else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
					if (ambiguousConstructors == null) {
						ambiguousConstructors = new LinkedHashSet<>();
						ambiguousConstructors.add(constructorToUse);
					}
					ambiguousConstructors.add(candidate);
				}
			}

			if (constructorToUse == null) {
				if (causes != null) {
					UnsatisfiedDependencyException ex = causes.removeLast();
					for (Exception cause : causes) {
						this.beanFactory.onSuppressedException(cause);
					}
					throw ex;
				}
				throw new BeanCreationException(mbd.getResourceDescription(), beanName,
						"Could not resolve matching constructor " +
						"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities)");
			}
			else if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) {
				throw new BeanCreationException(mbd.getResourceDescription(), beanName,
						"Ambiguous constructor matches found in bean '" + beanName + "' " +
						"(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " +
						ambiguousConstructors);
			}

			if (explicitArgs == null && argsHolderToUse != null) {
                // 将解析的构造函数加入缓存
				argsHolderToUse.storeCache(mbd, constructorToUse);
			}
		}

		Assert.state(argsToUse != null, "Unresolved constructor arguments");
    	// 将构建的实例加入 BeanWapper 中
		bw.setBeanInstance(instantiate(beanName, mbd, constructorToUse, argsToUse));
		return bw;
}
```



###### 不带参数的构造函数实例化

```java
// AbstractAutowireCapableBeanFactory 类
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
		try {
			Object beanInstance;
			final BeanFactory parent = this;
			if (System.getSecurityManager() != null) {
				beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
						getInstantiationStrategy().instantiate(mbd, beanName, parent),
						getAccessControlContext());
			}
			else {
				beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
			}
			BeanWrapper bw = new BeanWrapperImpl(beanInstance);
			initBeanWrapper(bw);
			return bw;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
		}
}
```



###### 实例化策略

```java
// SimpleInstantiationStrategy 类
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
		// Don't override the class with CGLIB if no overrides.
    	// 如果有需要覆盖或者动态替换的方法则当然需要使用 cglib 进行动态代理,因为可以在创建代理的同时将动态方法织入类中
	    // 但是如果没有需要动态改变得方法，为了方便直接反射就可以了
		if (!bd.hasMethodOverrides()) {
			Constructor<?> constructorToUse;
			synchronized (bd.constructorArgumentLock) {
				constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
				if (constructorToUse == null) {
					final Class<?> clazz = bd.getBeanClass();
					if (clazz.isInterface()) {
						throw new BeanInstantiationException(clazz, "Specified class is an interface");
					}
					try {
						if (System.getSecurityManager() != null) {
							constructorToUse = AccessController.doPrivileged(
									(PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
						}
						else {
							constructorToUse = clazz.getDeclaredConstructor();
						}
						bd.resolvedConstructorOrFactoryMethod = constructorToUse;
					}
					catch (Throwable ex) {
						throw new BeanInstantiationException(clazz, "No default constructor found", ex);
					}
				}
			}
			return BeanUtils.instantiateClass(constructorToUse);
		}
		else {
			// Must generate CGLIB subclass.
			return instantiateWithMethodInjection(bd, beanName, owner);
		}
}
```

```java
// CglibSubclassingInstantiationStrategy 类
public Object instantiate(@Nullable Constructor<?> ctor, Object... args) {
			Class<?> subclass = createEnhancedSubclass(this.beanDefinition);
			Object instance;
			if (ctor == null) {
				instance = BeanUtils.instantiateClass(subclass);
			}
			else {
				try {
					Constructor<?> enhancedSubclassConstructor = subclass.getConstructor(ctor.getParameterTypes());
					instance = enhancedSubclassConstructor.newInstance(args);
				}
				catch (Exception ex) {
					throw new BeanInstantiationException(this.beanDefinition.getBeanClass(),
							"Failed to invoke constructor for CGLIB enhanced subclass [" + subclass.getName() + "]", ex);
				}
			}
			// SPR-10785: set callbacks directly on the instance instead of in the
			// enhanced class (via the Enhancer) in order to avoid memory leaks.
			Factory factory = (Factory) instance;
			factory.setCallbacks(new Callback[] {NoOp.INSTANCE,
					new LookupOverrideMethodInterceptor(this.beanDefinition, this.owner),
					new ReplaceOverrideMethodInterceptor(this.beanDefinition, this.owner)});
			return instance;
}
```

​		首先判断如果没有使用`replace`或者`lookup`的配置方法，那么直接使用反射的方式，简单快捷，但是如果使用了这两个特性，在直接使用反射的方式创建实

例就不妥了，因为需要将这两个配置提供的功能切入进去，所以就必须要使用动态代理的方式将包含两个特性所对应的逻辑的拦截增强器设置进去，这样才可以保

证在调用方法的时候会被相应的拦截器增强，返回值为包含拦截器的代理实例。



##### 记录创建 bean 的ObjectFactory

```java
// AbstractAutowireCapableBeanFactory 类的 doCreateBean 方法片段
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
            // 为避免后期循环依赖，可以在 bean 初始化完成前将创建实例的 ObjectFactory 加人工厂
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

```java
// AbstractAutowireCapableBeanFactory 类
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
}
```



##### 属性注入

```java
// AbstractAutowireCapableBeanFactory 类
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		if (bw == null) {
			if (mbd.hasPropertyValues()) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
			}
			else {
				// Skip property population phase for null instance.
                // 没有可填充的属性
				return;
			}
		}

		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
    	// 给 InstantiationAwareBeanPostProcessors 最后一次机会在属性设置前来改变 bean，可以用来支持属性注入的类型
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						return;
					}
				}
			}
		}

		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
            // 根据名称自动注入
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
                // 根据类型自动注入
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}
		// 后处理器已经初始化
		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
    	// 需要依赖检查
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
                        // 对所有需要依赖检查的属性进行后处理
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
            // 依赖检查，对应 depends-on 属性，3.0已经弃用
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
            // 将属性应用的 bean 中
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
}
```



###### 根据名称自动注入

```java
// AbstractAutowireCapableBeanFactory 类
protected void autowireByName(
			String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
		// 获取需要依赖注入的属性
		String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
		for (String propertyName : propertyNames) {
			if (containsBean(propertyName)) {
                // 递归初始化相关的 bean
				Object bean = getBean(propertyName);
				pvs.add(propertyName, bean);
                // 注册依赖
				registerDependentBean(propertyName, beanName);
				if (logger.isTraceEnabled()) {
					logger.trace("Added autowiring by name from bean name '" + beanName +
							"' via property '" + propertyName + "' to bean named '" + propertyName + "'");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Not autowiring property '" + propertyName + "' of bean '" + beanName +
							"' by name: no matching bean found");
				}
			}
		}
}
```



###### 根据类型自动注入

```java
// AbstractAutowireCapableBeanFactory 类
protected void autowireByType(
			String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

		TypeConverter converter = getCustomTypeConverter();
		if (converter == null) {
			converter = bw;
		}

		Set<String> autowiredBeanNames = new LinkedHashSet<>(4);
    	// 获取需要依赖注入的属性
		String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
		for (String propertyName : propertyNames) {
			try {
				PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
				// Don't try autowiring by type for type Object: never makes sense,
				// even if it technically is a unsatisfied, non-simple property.
				if (Object.class != pd.getPropertyType()) {
                    // 探测指定属性的 set 方法
					MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
					// Do not allow eager init for type matching in case of a prioritized post-processor.
					boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);
					DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);
                    // 解析指定 beanName 的属性所匹配的值，并把解析到的属性名称存储在 autowiredBeanNames 中，当属性存在多个封装bean 时如: @Autowired private List<A> aList;将会找到所有匹配 A 类型的 bean 并将其注入
                    // 对于非集合属性注入，autowiredBeanNames 并无用处
					Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);
					if (autowiredArgument != null) {
						pvs.add(propertyName, autowiredArgument);
					}
					for (String autowiredBeanName : autowiredBeanNames) {
                        // 依赖注册
						registerDependentBean(autowiredBeanName, beanName);
						if (logger.isTraceEnabled()) {
							logger.trace("Autowiring by type from bean name '" + beanName + "' via property '" +
									propertyName + "' to bean named '" + autowiredBeanName + "'");
						}
					}
					autowiredBeanNames.clear();
				}
			}
			catch (BeansException ex) {
				throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, ex);
			}
		}
}
```

```java
// DefaultListableBeanFactory 类
public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,
			@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

		descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());
		if (Optional.class == descriptor.getDependencyType()) {
			return createOptionalDependency(descriptor, requestingBeanName);
		}
		else if (ObjectFactory.class == descriptor.getDependencyType() ||
				ObjectProvider.class == descriptor.getDependencyType()) {
            // ObjectFactory 类注入的特殊处理
			return new DependencyObjectProvider(descriptor, requestingBeanName);
		}
		else if (javaxInjectProviderClass == descriptor.getDependencyType()) {
            // javaxInjectProviderClass 类注入的特殊处理
			return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);
		}
		else {
			Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(
					descriptor, requestingBeanName);
			if (result == null) {
                // 通用处理
				result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
			}
			return result;
		}
}

public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
			@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

		InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
		try {
			Object shortcut = descriptor.resolveShortcut(this);
			if (shortcut != null) {
				return shortcut;
			}

			Class<?> type = descriptor.getDependencyType();
            // 用于支持 @Value 注解
			Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
			if (value != null) {
				if (value instanceof String) {
					String strVal = resolveEmbeddedValue((String) value);
					BeanDefinition bd = (beanName != null && containsBean(beanName) ?
							getMergedBeanDefinition(beanName) : null);
					value = evaluateBeanDefinitionString(strVal, bd);
				}
				TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
				try {
					return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
				}
				catch (UnsupportedOperationException ex) {
					// A custom TypeConverter which does not support TypeDescriptor resolution...
					return (descriptor.getField() != null ?
							converter.convertIfNecessary(value, type, descriptor.getField()) :
							converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
				}
			}

			Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
			if (multipleBeans != null) {
				return multipleBeans;
			}
			// 根据属性类型找到 beanFacotry 中所有类型的匹配bean,返回值的构成为: key = 匹配的beanName, value = beanName对应的实例化后的 bean(通过 getBean (beanName)返回)
			Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
			if (matchingBeans.isEmpty()) {
                // 如果 autowire 的 require 属性为 true 而找到的匹配项却为空则只能抛出异常
				if (isRequired(descriptor)) {
					raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
				}
				return null;
			}

			String autowiredBeanName;
			Object instanceCandidate;

			if (matchingBeans.size() > 1) {
				autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
				if (autowiredBeanName == null) {
					if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
						return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
					}
					else {
						// In case of an optional Collection/Map, silently ignore a non-unique case:
						// possibly it was meant to be an empty collection of multiple regular beans
						// (before 4.3 in particular when we didn't even look for collection beans).
						return null;
					}
				}
				instanceCandidate = matchingBeans.get(autowiredBeanName);
			}
			else {
				// We have exactly one match.
                // 确定只有一个匹配项
				Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
				autowiredBeanName = entry.getKey();
				instanceCandidate = entry.getValue();
			}

			if (autowiredBeanNames != null) {
				autowiredBeanNames.add(autowiredBeanName);
			}
			if (instanceCandidate instanceof Class) {
				instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
			}
			Object result = instanceCandidate;
			if (result instanceof NullBean) {
				if (isRequired(descriptor)) {
					raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
				}
				result = null;
			}
			if (!ClassUtils.isAssignableValue(type, result)) {
				throw new BeanNotOfRequiredTypeException(autowiredBeanName, type, instanceCandidate.getClass());
			}
			return result;
		}
		finally {
			ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
		}
}
```



​		到这里已经完成了对所有注入属性的获取，应用到实例化的`bean`中：

```java
// AbstractAutowireCapableBeanFactory 类
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
		if (pvs.isEmpty()) {
			return;
		}

		if (System.getSecurityManager() != null && bw instanceof BeanWrapperImpl) {
			((BeanWrapperImpl) bw).setSecurityContext(getAccessControlContext());
		}

		MutablePropertyValues mpvs = null;
		List<PropertyValue> original;

		if (pvs instanceof MutablePropertyValues) {
			mpvs = (MutablePropertyValues) pvs;
            //如果 mpvs 中的值已经被转换为对应的类型那么可以直接设置到 beanwapper 中
			if (mpvs.isConverted()) {
				// Shortcut: use the pre-converted values as-is.
				try {
					bw.setPropertyValues(mpvs);
					return;
				}
				catch (BeansException ex) {
					throw new BeanCreationException(
							mbd.getResourceDescription(), beanName, "Error setting property values", ex);
				}
			}
			original = mpvs.getPropertyValueList();
		}
		else {
            //如果 pvs 并不是使用 MutablePropertyValues 封装的类型，那么直接使用原始的属性获取方法
			original = Arrays.asList(pvs.getPropertyValues());
		}

		TypeConverter converter = getCustomTypeConverter();
		if (converter == null) {
			converter = bw;
		}
    	// 获取对应的解析器
		BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);

		// Create a deep copy, resolving any references for values.
		List<PropertyValue> deepCopy = new ArrayList<>(original.size());
		boolean resolveNecessary = false;
    	// 遍历属性，将属性转换为对应类的对应属性的类型
		for (PropertyValue pv : original) {
			if (pv.isConverted()) {
				deepCopy.add(pv);
			}
			else {
				String propertyName = pv.getName();
				Object originalValue = pv.getValue();
				if (originalValue == AutowiredPropertyMarker.INSTANCE) {
					Method writeMethod = bw.getPropertyDescriptor(propertyName).getWriteMethod();
					if (writeMethod == null) {
						throw new IllegalArgumentException("Autowire marker for property without write method: " + pv);
					}
					originalValue = new DependencyDescriptor(new MethodParameter(writeMethod, 0), true);
				}
				Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
				Object convertedValue = resolvedValue;
				boolean convertible = bw.isWritableProperty(propertyName) &&
						!PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
				if (convertible) {
					convertedValue = convertForProperty(resolvedValue, propertyName, bw, converter);
				}
				// Possibly store converted value in merged bean definition,
				// in order to avoid re-conversion for every created bean instance.
				if (resolvedValue == originalValue) {
					if (convertible) {
						pv.setConvertedValue(convertedValue);
					}
					deepCopy.add(pv);
				}
				else if (convertible && originalValue instanceof TypedStringValue &&
						!((TypedStringValue) originalValue).isDynamic() &&
						!(convertedValue instanceof Collection || ObjectUtils.isArray(convertedValue))) {
					pv.setConvertedValue(convertedValue);
					deepCopy.add(pv);
				}
				else {
					resolveNecessary = true;
					deepCopy.add(new PropertyValue(pv, convertedValue));
				}
			}
		}
		if (mpvs != null && !resolveNecessary) {
			mpvs.setConverted();
		}

		// Set our (possibly massaged) deep copy.
		try {
			bw.setPropertyValues(new MutablePropertyValues(deepCopy));
		}
		catch (BeansException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Error setting property values", ex);
		}
}
```



##### 初始化 bean

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            // 对特殊的 bean 处理:Aware、BeanclassLoaderAware、BeanFactoryAware
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            // 应用后处理器
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
            // 激活用于自定义的 init 方法
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
            // 应用后处理器
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
}
```

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
}

public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
}

public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
}
```



​		自定义初始化方法除了使用`init-method`外，还可以实现`InitializingBean`接口：执行顺序是先执行实现的接口方法，后执行`init-method`指定的方法。

```java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
    	// 首先会检查是否是 InitializingBean，如果是的话需要调用 afterPropertiesSet 方法
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
                // 属性初始化后的处理
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
                // 调用自定义初始化方法
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
}
```



##### 注册 DisposableBean

​		`Spring`中不但提供了对于初始化方法的扩展入口，同样也提供了销毁方法的扩展入口，对于销毁方法的扩展，除了配置属性`destroy-method`方法外，还可以

注册后处理器`DestructionAwareBeanPostProcessor`来统一处理`bean`的销毁方法，代码如下：

```java
// AbstractBeanFactory 类
protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
		AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
		if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
			if (mbd.isSingleton()) {
				// Register a DisposableBean implementation that performs all destruction
				// work for the given bean: DestructionAwareBeanPostProcessors,
				// DisposableBean interface, custom destroy method.
                 /*
				* 单例模式下注册需要销毁的bean，此方法中会处理实现 DisposableBean 的 bean
				* 并且对所有的 bean 使用 DestructionAwareBeanPostProcessors 处理
			    */
				registerDisposableBean(beanName,
						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
			}
			else {
				// A bean with a custom scope...
                /**
                	自定义 scope 处理
                */
				Scope scope = this.scopes.get(mbd.getScope());
				if (scope == null) {
					throw new IllegalStateException("No Scope registered for scope name '" + mbd.getScope() + "'");
				}
				scope.registerDestructionCallback(beanName,
						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
			}
		}
}
```



### ApplicationContext

​		`ApplicationContext`和`BeanFacotry`两者都是用于加载`Bean`的，但是相比之下，`ApplicationContext`提供了更多的扩展功能，简单一点说：

`ApplicationContext`包含`BeanFactory`的所有功能。

```java
public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {
    @Nullable
    private Resource[] configResources;
    
    public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[]{configLocation}, true, (ApplicationContext)null);
    }

    public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
        // 设置配置路径
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
    ...
}
```

```java
// AbstractRefreshableConfigApplicationContext 类
public void setConfigLocations(@Nullable String... locations) {
		if (locations != null) {
			Assert.noNullElements(locations, "Config locations must not be null");
			this.configLocations = new String[locations.length];
			for (int i = 0; i < locations.length; i++) {
				this.configLocations[i] = resolvePath(locations[i]).trim(); // 解析给定路径
			}
		}
		else {
			this.configLocations = null;
		}
}
```

```java
// AbstractApplicationContext 类
// 这个函数几乎包含了 ApplicationContext 提供的全部功能
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
            // 准备刷新上下文环境，对系统属性或者环境变量进行准备及验证。
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
            // 初始化BeanFactory，并进行 XML 文件读取，复用 BeanFactory 中的配置文件读取解析及其他功能，经过这一步之后，已经包含了 BeanFactory 所提		供的功能
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
            // 对 BeanFactory 进行各种功能填充，@Qualifier 与 @Autowired 在这一步骤中增加的支持
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
                // 子类覆盖方法做额外的处理
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
                // 激活各种 BeanFactory 处理器
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                // 注册拦截 Bean 创建的 Bean 处理器，这里只是注册，真正的调用在 getBean 
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
                // 为上下文初始化 Message 源，即不同语言的消息体，国际化处理
				initMessageSource();

				// Initialize event multicaster for this context.
                // 初始化应用消息广播器，并放入 applicationEventMulticaster bean 中
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                // 留给子类来初始化其它的 Bean
				onRefresh();

				// Check for listener beans and register them.
                // 在所有注册的 bean 中查找 Listener bean，注册到消息广播器中
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                // 初始化剩下的单实例（非惰性的)
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
                // 完成刷新过程，通知生命周期处理器 lifecycleProcessor 刷新过程，同时发出 ContextRefreshEvent 通知别人
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				destroyBeans();

				cancelRefresh(ex);

				throw ex;
			}

			finally {
				resetCommonCaches();
			}
		}
}
```



#### 环境准备

```java
// AbstractApplicationContext 类
protected void prepareRefresh() {
		// Switch to active.
		this.startupDate = System.currentTimeMillis();
		this.closed.set(false);
		this.active.set(true);

		if (logger.isDebugEnabled()) {
			if (logger.isTraceEnabled()) {
				logger.trace("Refreshing " + this);
			}
			else {
				logger.debug("Refreshing " + getDisplayName());
			}
		}

		// Initialize any placeholder property sources in the context environment.
    	// 留给子类覆盖
		initPropertySources();

		// Validate that all properties marked as required are resolvable:
		// see ConfigurablePropertyResolver#setRequiredProperties
    	// 验证需要的属性文件是否都已经放入环境中，可以调用 getEnvironment.setRequiredProperties 添加验证要求
		getEnvironment().validateRequiredProperties();

		// Store pre-refresh ApplicationListeners...
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
}
```



#### 加载 BeanFactory

​		`obtainFreshBeanFactory`方法是获取`BeanFactory`。`ApplicationContext`是对`BeanFactory`的功能上的扩展，不但包含了`BeanFactory`的全部功能更在其基

础上添加了大量的扩展应用，那么`obtainFreshBeanFactory`正是实现`BeanFactory`的地方，也就是经过了这个函数后`ApplicationContext`就已经拥有了

`BeanFactory`的全部功能。

```java
// AbstractApplicationContext 类
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    	// 初始化 BeanFactory，并进行 XML 文件读取，并将得到的 BeanFacotry 记录在当前实体的属性中
		refreshBeanFactory();
    	// 返回当前实体的 beanFactory 属性
		return getBeanFactory();
}
```

```java
// AbstractRefreshableApplicationContext 类
protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
            // 创建 DefaultListableBeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
            // 为了序列化指定 id，如果需要的话，让这个 BeanFactory 从 id 反序列化到 BeanFactory 对象
			beanFactory.setSerializationId(getId());
            // 定制 beanFactory，设置相关属性，包括是否允许覆盖同名称的不同定义的对象以及循环依赖以及
            // 设置 @Autowired 和 @Qualifier 注解解析器 QualifierAnnotationAutowireCandidateResolver
			customizeBeanFactory(beanFactory);
            // 初始化 DodumentReader，并进行XML文件读取及解析
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
}
```



##### 定制 BeanFactory

​		在基本容器的基础上，增加了是否允许覆盖、是否允许扩展的设置并提供了注解`@Qualifier`和`@Autowired`的支持。

```java
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    	// 如果属性 allowBeanDefinitionOverriding 不为空，设置给 beanFactory 对象相应属性
    	// 此属性的含义：是否允许覆盖同名称的不同定义的对象
		if (this.allowBeanDefinitionOverriding != null) {
			beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
    	// 如果属性 allowCircularReferences 不为空，设置给 beanFactory 对象相应属性
    	// 此属性的含义:是否允许 bean 之间存在循环依赖
		if (this.allowCircularReferences != null) {
			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
		}
}
```



##### 加载 BeanDefinition

​		在已经初始化`DefaultListableBeanFactory`后，还需要`XmlBeanDefinitionReader`来读取`XML`，在这个步骤中首先要做的就是初始化

`XmlBeanDefinitionReader`。

```java
// AbstractXmlApplicationContext 类
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
    	// //为指定 beanFactory 创建 XmlBeanDefinitionReader
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
    	// 对 beanDefinitionReader 进行环境变量的设置
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
	    // 对 BeanDefinitionReader 进行设置,可以覆盖
		initBeanDefinitionReader(beanDefinitionReader);
		loadBeanDefinitions(beanDefinitionReader);
}

protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
}
```

​		在`XmlBeanDefinitionReader`中已经将之前初始化的`DefaultListableBeanFactory`注册进去了，所以`XmIBeanDefinitionReader`所读取的

`BeanDefinitionHolder`都会注册到`DefaultListableBeanFactory`中，也就是经过此步骤，类型`DefaultListableBeanFactory`的变量`beanFactory`已经包含了所有

解析好的配置。



#### 功能扩展

​		`prepareBeanFactory`前，`Spring`已经完成了对配置的解析，而`ApplicationContext`在功能上的扩展也由此展开。

```java
// AbstractApplicationContext 类
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
    	// 设置 beanFactory 的 classLoader 为当前 context 的 classLoader
		beanFactory.setBeanClassLoader(getClassLoader());
    	// 设置 beanFactory 的表达式语言处理器，Spring 3 增加了 SpEL 表达式语言的支持
    	// 默认可以使用 #{bean.xxx} 的形式来调用相关属性值。
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    	// 为 beanFactory 增加了一个默认的 propertyEditor，这个主要是对 bean 的属性等设置管理的一个工具，即属性编辑器
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
    	// 添加BeanPostProcessor
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    	/**
    		设置了几个忽略自动装配的接口，这几个接口都是被 bean 实现的，即实现这几个接口的 bean 已经不是普通的 bean，所以依赖注入时需要忽略
    	*/
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
    	/**
    		设置了几个自动装配的特殊规则，注册之后，在做依赖注入时，如果发现属性的类型是以下类型，则将对应的实例注入进去
    	*/
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
    	// 增加对 AspectJ 的支持
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
    	// 添加默认的系统环境 bean
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
}
```



##### SpEL 表达式

​		在源码中通过代码`beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver())`注册语言解析器，就可以对`SpEL`进行解析了，`Spring`

在`bean`进行初始化的时候会有属性填充的一步，而在这一步中`Spring`会调用`AbstractAutowireCapableBeanFactory`类的`applyPropertyValues`函数来完成功能。

就在这个函数中，会通过构造`BeanDefinitionValueResolver`类型实例`valueR	esolver`来进行属性值的解析。同时，也是在这个步骤中一般通过

`AbstractBeanFactory`中的`evaluateBeanDefinitionString`方法去完成`SpEL`的解析。应用语言解析器的调用主要是在解析依赖注入`bean`的时候，以及在完成

`bean`的初始化和属性获取后进行属性填充的时候。



##### 增加属性注册编辑器

​		在`Spring DI`注入的时候可以把普通属性注入进来，但是像`Date`类型就无法被识别。解决方案：

​				1、使用自定义属性编辑器，通过继承`PropertyEditorSupport`，重写`setAsText`方法，在配置文件中引人类型为

​		`org.Springframework.beans.factory.config.CustomEditorConfigurer`的`bean`，并在属性`customEditors`中加入自定义的属性编辑器，其中`key`为属性编辑

​		器所对应的类型。

​				2、通过注册`Spring`自带的属性编辑器`CustomDateEditor`。

```java
// ResourceEditorRegistrar 类
public void registerCustomEditors(PropertyEditorRegistry registry) {
		ResourceEditor baseEditor = new ResourceEditor(this.resourceLoader, this.propertyResolver);
		doRegisterEditor(registry, Resource.class, baseEditor);
		doRegisterEditor(registry, ContextResource.class, baseEditor);
		doRegisterEditor(registry, InputStream.class, new InputStreamEditor(baseEditor));
		doRegisterEditor(registry, InputSource.class, new InputSourceEditor(baseEditor));
		doRegisterEditor(registry, File.class, new FileEditor(baseEditor));
		doRegisterEditor(registry, Path.class, new PathEditor(baseEditor));
		doRegisterEditor(registry, Reader.class, new ReaderEditor(baseEditor));
		doRegisterEditor(registry, URL.class, new URLEditor(baseEditor));

		ClassLoader classLoader = this.resourceLoader.getClassLoader();
		doRegisterEditor(registry, URI.class, new URIEditor(classLoader));
		doRegisterEditor(registry, Class.class, new ClassEditor(classLoader));
		doRegisterEditor(registry, Class[].class, new ClassArrayEditor(classLoader));

		if (this.resourceLoader instanceof ResourcePatternResolver) {
			doRegisterEditor(registry, Resource[].class,
					new ResourceArrayPropertyEditor((ResourcePatternResolver) this.resourceLoader, this.propertyResolver));
		}
}

private void doRegisterEditor(PropertyEditorRegistry registry, Class<?> requiredType, PropertyEditor editor) {
		if (registry instanceof PropertyEditorRegistrySupport) {
			((PropertyEditorRegistrySupport) registry).overrideDefaultEditor(requiredType, editor);
		}
		else {
			registry.registerCustomEditor(requiredType, editor);
		}
}
```

​		通过查看`registerCustomEditors`方法的调用层次结构，在`bean`的初始化后会调用`ResourceEditorRegistrar`的`registerCustomEditors`方法进行批量的通用

属性编辑器注册。注册后，在属性填充的环节便可以直接让`Spring`使用这些编辑器进行属性的解析了。`Spring`中用于封装`bean`的是`BeanWrapper`类型，而它又

间接继承了`PropertyEditorRegistry`类型，其实大部分情况下都是`BeanWrapper`，对于`BeanWrapper`在`Spring`中的默认实现是`BeanWrapperImpl`，而

`BeanWrapperImpl`除了实现`BeanWrapper`接口外还继承了`PropertyEditorRegistrySupport`。

![](image/QQ截图20220306113437.png)

```java
// PropertyEditorRegistrySupport 类
private void createDefaultEditors() {
		this.defaultEditors = new HashMap<>(64);

		// Simple editors, without parameterization capabilities.
		// The JDK does not contain a default editor for any of these target types.
		this.defaultEditors.put(Charset.class, new CharsetEditor());
		this.defaultEditors.put(Class.class, new ClassEditor());
		this.defaultEditors.put(Class[].class, new ClassArrayEditor());
		this.defaultEditors.put(Currency.class, new CurrencyEditor());
		this.defaultEditors.put(File.class, new FileEditor());
		this.defaultEditors.put(InputStream.class, new InputStreamEditor());
		this.defaultEditors.put(InputSource.class, new InputSourceEditor());
		this.defaultEditors.put(Locale.class, new LocaleEditor());
		this.defaultEditors.put(Path.class, new PathEditor());
		this.defaultEditors.put(Pattern.class, new PatternEditor());
		this.defaultEditors.put(Properties.class, new PropertiesEditor());
		this.defaultEditors.put(Reader.class, new ReaderEditor());
		this.defaultEditors.put(Resource[].class, new ResourceArrayPropertyEditor());
		this.defaultEditors.put(TimeZone.class, new TimeZoneEditor());
		this.defaultEditors.put(URI.class, new URIEditor());
		this.defaultEditors.put(URL.class, new URLEditor());
		this.defaultEditors.put(UUID.class, new UUIDEditor());
		this.defaultEditors.put(ZoneId.class, new ZoneIdEditor());

		// Default instances of collection editors.
		// Can be overridden by registering custom instances of those as custom editors.
		this.defaultEditors.put(Collection.class, new CustomCollectionEditor(Collection.class));
		this.defaultEditors.put(Set.class, new CustomCollectionEditor(Set.class));
		this.defaultEditors.put(SortedSet.class, new CustomCollectionEditor(SortedSet.class));
		this.defaultEditors.put(List.class, new CustomCollectionEditor(List.class));
		this.defaultEditors.put(SortedMap.class, new CustomMapEditor(SortedMap.class));

		// Default editors for primitive arrays.
		this.defaultEditors.put(byte[].class, new ByteArrayPropertyEditor());
		this.defaultEditors.put(char[].class, new CharArrayPropertyEditor());

		// The JDK does not contain a default editor for char!
		this.defaultEditors.put(char.class, new CharacterEditor(false));
		this.defaultEditors.put(Character.class, new CharacterEditor(true));

		// Spring's CustomBooleanEditor accepts more flag values than the JDK's default editor.
		this.defaultEditors.put(boolean.class, new CustomBooleanEditor(false));
		this.defaultEditors.put(Boolean.class, new CustomBooleanEditor(true));

		// The JDK does not contain default editors for number wrapper types!
		// Override JDK primitive number editors with our own CustomNumberEditor.
		this.defaultEditors.put(byte.class, new CustomNumberEditor(Byte.class, false));
		this.defaultEditors.put(Byte.class, new CustomNumberEditor(Byte.class, true));
		this.defaultEditors.put(short.class, new CustomNumberEditor(Short.class, false));
		this.defaultEditors.put(Short.class, new CustomNumberEditor(Short.class, true));
		this.defaultEditors.put(int.class, new CustomNumberEditor(Integer.class, false));
		this.defaultEditors.put(Integer.class, new CustomNumberEditor(Integer.class, true));
		this.defaultEditors.put(long.class, new CustomNumberEditor(Long.class, false));
		this.defaultEditors.put(Long.class, new CustomNumberEditor(Long.class, true));
		this.defaultEditors.put(float.class, new CustomNumberEditor(Float.class, false));
		this.defaultEditors.put(Float.class, new CustomNumberEditor(Float.class, true));
		this.defaultEditors.put(double.class, new CustomNumberEditor(Double.class, false));
		this.defaultEditors.put(Double.class, new CustomNumberEditor(Double.class, true));
		this.defaultEditors.put(BigDecimal.class, new CustomNumberEditor(BigDecimal.class, true));
		this.defaultEditors.put(BigInteger.class, new CustomNumberEditor(BigInteger.class, true));

		// Only register config value editors if explicitly requested.
		if (this.configValueEditorsActive) {
			StringArrayPropertyEditor sae = new StringArrayPropertyEditor();
			this.defaultEditors.put(String[].class, sae);
			this.defaultEditors.put(short[].class, sae);
			this.defaultEditors.put(int[].class, sae);
			this.defaultEditors.put(long[].class, sae);
		}
}
```

​		通过这个方法在`Spring`中定义了上面一系列常用的属性编辑器可以方便地进行配置。只有定义的 bean 中的某个属性的类型不在上面的常用配置中的话，才

需要我们进行个性化属性编辑器的注册。



##### 添加 ApplicationContextAwareProcessor 处理器

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor {

	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
				bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
				bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)){
			return bean;
		}

		AccessControlContext acc = null;

		if (System.getSecurityManager() != null) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareInterfaces(bean);
				return null;
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof EnvironmentAware) {
			((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
		}
		if (bean instanceof EmbeddedValueResolverAware) {
			((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
		}
		if (bean instanceof ResourceLoaderAware) {
			((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
		}
		if (bean instanceof ApplicationEventPublisherAware) {
			((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
		}
		if (bean instanceof MessageSourceAware) {
			((MessageSourceAware) bean).setMessageSource(this.applicationContext);
		}
		if (bean instanceof ApplicationContextAware) {
			((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
		}
	}
}
```

​		通过`postProcessBeforeInitialization`中的`if`判断可以看出：实现这些`Aware`接口的`bean`在被初始化之后，可以取得一些对应的资源。



#### BeanFactory的后处理

##### 激活注册的BeanFactoryPostProcessor

​		`BeanFactoryPostProcessor`接口跟`BeanPostProcessor`类似，可以对`bean`的定义`(`配置元数据`)`进行处理。也就是说，`Spring loC`容器允许

`BeanFactoryPostProcessor`在容器实际实例化任何其他的`bean`之前读取配置元数据，并有可能修改它。可以配置多个`BeanFactoryPostProcessor`。还能通过设

置`order`属性来控制`BeanFactoryPostProcessor`的执行次序`(`仅当`BeanFactoryPostProcessor`实现了`Ordered`接口时才可以设置此属性，因此在实现

`BeanFactoryPostProcessor`时，就应当考虑实现`Ordered`接口`)`。

​		想改变实际的`bean`实例，那么最好使用`BeanPostProcessor`。同样地，`BeanFactoryPostProcessor`的作用域范围是容器级的。它只和所使用的容器有关。如

果在容器中定义一个`BeanFactoryPostProcessor`，它仅仅对此容器中的`bean`进行后置处理。`BeanFactoryPostProcessor`不会对定义在另一个容器中的`bean`进行

后置处理，即使这两个容器都是在同一层次上。

```java
// PostProcessorRegistrationDelegate 类
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		
		Set<String> processedBeans = new HashSet<>();
		// 对 BeanDefinitionRegistry 类型的处理
		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

            // 硬编码注册的后处理器
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
                    // 对于 BeanDefinitionRegistryPostProcessor 类型，在 BeanFactoryPostProcessor 的基础上还有自己定义的方法,需要先调用
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
				else {
                    // 记录常规 BeanFactoryPostProcessor
					regularPostProcessors.add(postProcessor);
				}
			}

			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

			// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
            // 对于配置中读取的 BeanFactoryPostProcessor 的处理
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            // 对后处理器进行分类
			for (String ppName : postProcessorNames) {
                // 已经处理过
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
			currentRegistryProcessors.clear();

			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
			currentRegistryProcessors.clear();

			// Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
				currentRegistryProcessors.clear();
			}

			// Now, invoke the postProcessBeanFactory callback of all processors handled so far.
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// Finally, invoke all other BeanFactoryPostProcessors.
    	// 无序，直接调用
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
}
```

​		对于`BeanFactoryPostProcessor`的处理主要分两种情况进行，一个是对于`BeanDefinitionRegistry`类的特殊处理，另一种是对普通的

`BeanFactoryPostProcessor`进行处理。而对于每种情况都需要考虑硬编码注入注册的后处理器以及通过配置注入的后处理器。

​		对于硬编码注册的后处理器的处理，主要是通过`AbstractApplicationContext`中的添加处理器方法`addBeanFactoryPostProcessor`进行添加。添加后的后处理

器会存放在`beanFactoryPostProcessors`中，而在处理`BeanFactoryPostProcessor`时候会首先检测`beanFactoryPostProcessors`是否有数据。当然，

`BeanDefinitionRegistryPostProcessor`继承自`BeanFactoryPostProcessor`，不但有`BeanFactoryPostProcessor`的特性，同时还有自己定义的个性化方法，也需要

在此调用。所以，这里需要从`beanFactoryPostProcessors`中挑出`BeanDefinitionRegistryPostProcessor`的后处理器，并进行其

`postProcessBeanDefinitionRegistry`方法的激活。



##### 注册 BeanPostProcessor

```java
// AbstractApplicationContext 类
public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

		// Register BeanPostProcessorChecker that logs an info message when
		// a bean is created during BeanPostProcessor instantiation, i.e. when
		// a bean is not eligible for getting processed by all BeanPostProcessors.
    	/*
		* BeanPostProcessorChecker 是一个普通的信息打印，可能会有些情况，
		* 当 Spring 的配置中的后处理器还没有被注册就已经开始了bean的初始化时
		* 便会打印出 BeanPostProcessorChecker 中设定的信息
		*/
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

		// Separate between BeanPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
    	// 使用 Priorityordered 保证顺序
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    	// 使用 ordered 保证顺序
		List<String> orderedPostProcessorNames = new ArrayList<>();
    	// 无序 BeanPostProcessor
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, register the BeanPostProcessors that implement PriorityOrdered.
	    // 第 1 步，注册所有实现 Priorityordered 的 BeanPostProcessor
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

		// Next, register the BeanPostProcessors that implement Ordered.
    	// 第 2 步，注册所有实现 ordered 的 BeanPostProcessor
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// Now, register all regular BeanPostProcessors.
    	// 第 3 步，注册所有无序的 BeanPostProcessor
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// Finally, re-register all internal BeanPostProcessors.
    	// 第 4 步，注册所有 MergedBeanDefinitionPostProcessor 类型的 BeanPostProcessor ，并非重复注册，
		// 在 beanFactory.addBeanPostProcessor 中会先移除已经存在的 BeanPostProcessor
		sortPostProcessors(internalPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// Re-register post-processor for detecting inner beans as ApplicationListeners,
		// moving it to the end of the processor chain (for picking up proxies etc).
	    // 添加 ApplicationListener 探测器
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}
```



##### 初始化消息资源

```java
// AbstractApplicationContext 类
protected void initMessageSource() {
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
            // 如果在配置中已经配置了 messageSource，那么将 messageSource 提取并记录在 this.messagesource 中
			this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
			// Make MessageSource aware of parent MessageSource.
			if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
				HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
				if (hms.getParentMessageSource() == null) {
					// Only set parent context as parent MessageSource if no parent MessageSource
					// registered already.
					hms.setParentMessageSource(getInternalParentMessageSource());
				}
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Using MessageSource [" + this.messageSource + "]");
			}
		}
		else {
			// Use empty MessageSource to be able to accept getMessage calls.
            // 如果用户并没有定义配置文件，那么使用临时的 DelegatingMessageSource 以便于作为调用 getMessage 方法的返回
			DelegatingMessageSource dms = new DelegatingMessageSource();
			dms.setParentMessageSource(getInternalParentMessageSource());
			this.messageSource = dms;
			beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
			if (logger.isTraceEnabled()) {
				logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
			}
		}
}
```

​		在`initMessageSource`中的方法主要功能是提取配置中定义的`messageSource`，并将其记录在`Spring`的容器中，也就是`AbstractApplicationContext`中。当

然，如果未设置资源文件的话，`Spring`中也提供了默认的配置`DelegatingMessageSource`。在`initMessageSource`中获取自定义资源文件的方式为

`beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class)`，在这里`Spring`使用了硬编码的方式硬性规定了子定义资源文件必须为`message`，否则

便会获取不到自定义资源配置。



##### 初始化 ApplicationEventMulticaster

​		`initApplicationEventMulticaster`的方式比较简单，无非考虑两种情况：

​				1、如果用户自定义了事件广播器，那么使用用户自定义的事件广播器。

​				2、如果用户没有自定义事件广播器，那么使用默认的`ApplicationEventMulticaster`。

```java
// AbstractApplicationContext 类
protected void initApplicationEventMulticaster() {
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
			this.applicationEventMulticaster =
					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
			if (logger.isTraceEnabled()) {
				logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
			}
		}
		else {
			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
			if (logger.isTraceEnabled()) {
				logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
						"[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
			}
		}
}
```

```java
// SimpleApplicationEventMulticaster 类
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
}

protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
		ErrorHandler errorHandler = getErrorHandler();
		if (errorHandler != null) {
			try {
				doInvokeListener(listener, event);
			}
			catch (Throwable err) {
				errorHandler.handleError(err);
			}
		}
		else {
			doInvokeListener(listener, event);
		}
}

private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
			listener.onApplicationEvent(event); // 调用事件监听器的方法处理事件
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass()) ||
					(event instanceof PayloadApplicationEvent &&
							matchesClassCastMessage(msg, ((PayloadApplicationEvent) event).getPayload().getClass()))) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception.
				Log loggerToUse = this.lazyLogger;
				if (loggerToUse == null) {
					loggerToUse = LogFactory.getLog(getClass());
					this.lazyLogger = loggerToUse;
				}
				if (loggerToUse.isTraceEnabled()) {
					loggerToUse.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
}
```



##### 注册监听器

```java
// AbstractApplicationContext 类
protected void registerListeners() {
		// Register statically specified listeners first.
    	// 硬编码方式注册的监听器处理
		for (ApplicationListener<?> listener : getApplicationListeners()) {
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
    	// 配置文件注册的监听器处理
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}

		// Publish early application events now that we finally have a multicaster...
		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
		this.earlyApplicationEvents = null;
		if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
				getApplicationEventMulticaster().multicastEvent(earlyEvent);
			}
		}
}
```



#### 初始化非延迟加载单例

​		完成`BeanFactory`的初始化工作，其中包括`ConversionService`的设置、配置冻结以及非延迟加载的`bean`的初始化工作。

```java
// AbstractApplicationContext 类
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
    	// ConversionService：类型转换
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Register a default embedded value resolver if no BeanFactoryPostProcessor
		// (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
		// at this point, primarily for resolution in annotation attribute values.
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// Allow for caching all bean definition metadata, not expecting further changes.
	    // 冻结所有的 bean 定义，说明注册的 bean 定义将不被修改或任何进一步的处理
		beanFactory.freezeConfiguration();

		// Instantiate all remaining (non-lazy-init) singletons.
	    // 初始化剩下的单实例（非惰性的)
		beanFactory.preInstantiateSingletons();
}
```

```java
// DefaultListableBeanFactory 类
public void freezeConfiguration() {
		this.configurationFrozen = true;
		this.frozenBeanDefinitionNames = StringUtils.toStringArray(this.beanDefinitionNames);
}
```

​		`ApplicationContext`实现的默认行为就是在启动时将所有单例`bean`提前进行实例化。提前实例化意味着作为初始化过程的一部分，`ApplicationContext`实

例会创建并配置所有的单例`bean`。通常情况下这是一件好事，因为这样在配置中的任何错误就会即刻被发现`(`否则的话可能要花几个小时甚至几天`)`。而这个实

例化的过程就是在`finishBeanFactoryInitialization中`完成的。

```java
// DefaultListableBeanFactory 类
public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged(
									(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				StartupStep smartInitialize = this.getApplicationStartup().start("spring.beans.smart-initialize")
						.tag("beanName", beanName);
				SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
				smartInitialize.end();
			}
		}
}
```



#### finishRefresh

​		在`Spring`中还提供了`Lifecycle`接口，`Lifecycle`中包含`start/stop`方法，实现此接口后`Spring`会保证在启动的时候调用其`start`方法开始生命周期，并

在`Spring`关闭的时候调用`stop`方法来结束生命周期，通常用来配置后台程序，在启动后一直运行。而`ApplicationContext`的初始化最后正是保证了这一功能的

实现。

```java
// AbstractApplicationContext 类
protected void finishRefresh() {
		// Clear context-level resource caches (such as ASM metadata from scanning).
		clearResourceCaches();

		// Initialize lifecycle processor for this context.
		// 当 ApplicationContext 启动或停止时，它会通过 LifecycleProcessor 来与所有声明的 bean 的周期做状态更新
    	// 而在 LifecycleProcessor  的使用前首先需要初始化。
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
    	// 启动所有实现了 Lifecycle 接口的 bean
		getLifecycleProcessor().onRefresh();

		// Publish the final event.
		// 当完成 ApplicationContext 初始化的时候,要通过 Spring 中的事件发布机制来发出 ContextRefreshedEvent 事件
    	// 以保证对应的监听器可以做进一步的逻辑处理。
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		if (!NativeDetector.inNativeImage()) {
			LiveBeansView.registerApplicationContext(this);
		}
}
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

​				定义映射规则，级联的更新，定制类型转换器等。但现有版本，只支持resultMap查询，不支持更新或者保存。

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



### 注意

​		在使用`resultMap`进行类型转换时，不能出现字段无法找到对应实体属性的情况，在这种情况下，会报错。而在使用`resultType`的 情况下，就算字段没有找

到对应的实体属性，其也不会报错。



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

​				trim表示去掉一些特殊的字符串，prefix代表的是语句的前缀，prefixOverrides代表的是需要去掉哪种字符串。

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

# 逆向工程

​		通过数据库的表，生成出对应的实体类，mapper接口，mapper.xml 文件。

依赖

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.7</version>
</dependency>
```

编写配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    
    <!-- 用来指定外部的属性元素，最多配置一个  -->
    <properties resource="classpath:database.properties" />
    
    <!-- 用于指定驱动的路径 -->
    <classPathEntry location=""/>
    
    <!-- context 是逆向工程的主要配置信息 -->
    <!-- id：起个名字 -->
    <!-- targetRuntime：设置生成的文件适用于那个 mybatis 版本 -->
    <context id="default" targetRuntime="MyBatis3">
        <!--optional,指在创建class时，对注释进行控制-->
        <commentGenerator>
            <property name="addRemarkComments" value="true"/>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--jdbc的数据库连接 wg_insert 为数据库名字-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/ssm?useUnicode=true"
                        userId="root"
                        password="179980">
        </jdbcConnection>
        <!--非必须，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <!-- 默认情况下数据库中的 decimal，bigInt 在 Java 对应是 sql 下的 BigDecimal 类 -->
            <!-- 不是 double 和 long 类型 -->
            <!-- 使用常用的基本类型代替 sql 包下的引用类型 -->
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- targetPackage：生成的实体类所在的包 -->
        <!-- targetProject：生成的实体类所在的硬盘位置 -->
        <javaModelGenerator targetPackage="com.***pot.pojo"
                            targetProject="src/main/java">
            <!-- 是否允许子包 -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对modal添加构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否清理从数据库中查询出的字符串左右两边的空白字符 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立modal对象是否不可改变 即生成的modal对象不会有setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>
        <!-- targetPackage 和 targetProject：生成的 mapper 文件的包和位置 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources">
            <!-- 针对数据库的一个配置，是否把 schema 作为字包名 -->
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>
        <!-- targetPackage 和 targetProject：生成的 interface 文件的包和位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.****ot.mapper" targetProject="src/main/java">
            <!-- 针对 oracle 数据库的一个配置，是否把 schema 作为字包名 -->
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>
        <!-- tableName是数据库中的表名，domainObjectName是生成的JAVA模型名，后面的参数不用改，要生成更多的表就在下面继续加table标签 -->
        <!--1-->
        <table tableName="role" domainObjectName="Role"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
    </context>
</generatorConfiguration>
```

生成

```java
		List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("src/main/resource/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
```

context标签：

​		defaultModelType：取值：

​					conditional：默认值，如果一个表的主键只有一个字段，那么不会为该字段生成单独的实体类，而是会将该字段合并到基本实体类中。

​					flag：该模型只为每张表生成一个实体类，这个实体类包含表中的所有字段。

​					hierarchical：如果表有主键，那么该模型会产生一个单独的主键实体类，如果表还有BLOB字段，则会为表生成一个包含所有BLOB字段的单独的实体		类，然后为所有其他字段另外生成一个单独的实体类。

​		targetRuntime：此属性用于指定生成的代码的运行时环境。

​					MyBatis3：默认值

​					MyBatis3Simple：这种情况不会生成与Example相关的方法。

​		introspectedColumnImpl：该参数可以指定扩展org.mybatis.generator.api.Introspected Column类的实现类。



property标签：

​		当SQL中有数据库关键字时，使用分隔符括住关键字，可以避免数据库产生错误。

​		autoDelimitKeywords：自动给关键字添加分隔符。

​		beginningDelimiter：配置前置分隔符

​		endingDelimiter：配置后置分隔符。

​		javaFileEncoding：设置要使用的 Java 文件的编码，默认使用当前运行环境的编码。

```xml
		<property name="autoDelimitKeywords" value="true"/>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
```



plugin标签：

​		用来定义一个插件，用于扩展或修改通过生成器生成的代码。将按照配置中的配置顺序执行。

​		缓存插件：此插件可以在生成的SQL XML 映射文件中增加一个cache标签，只有当targetRuntime为MyBatis3，该插件才有效。

```xml
		<plugin type="org.mybatis.generator.plugins.CachePlugin">
            <property name="cache_eviction" value="LRU"/>
            <property name="cache_size" value="1024"/>
        </plugin>
```

​		除了缓存插件外，还默认包含了序列化差价，RowBounds插件，ToString插件等。



commentGenerator标签：

​		此标签有一个可选属性 type，指定用户的实现类，该类要实现 org.mybatis.generator.api.CommentGenerator接口，而且必须有一个默认空的构造方法。type 属性接收的默认的特殊值 DEFAULT，使用默认的实现类 org.mybatis.generator.internal.DefaultCommentGenerator。

​		默认的实现类中提供了三个可选属性，通过 property属性进行配置

​				suppressAllComments：阻止生成注释，默认为 falst。

​				suppressDate：阻止生成的注释包含时间戳，默认为 false。

​				addRemarkComments：注释是否添加数据库表的备注信息，默认为 false。



jdbcConnection标签：

​		此标签用于指定要连接的数据库信息。如果JDBC驱动不在 classpath下，就要通过 classpathEntry 标签引入 jar 包。

​		driverClass：访问数据库的 JDBC 驱动程序的完全限定类名。

​		connectionURL：访问数据库的 JDBC 链接 URL。

​		userId：访问数据库的用户 ID。

​		password：访问数据库的密码。

​		此外，还可以添加多个 property 子标签，配置的属性值都会添加到 JDBC 驱动的属性中。（使用 property 标签的 name 属性反射赋值）。



javaTypeResolver标签：

​		指定 JDBC 类型和 Java 类型如何转换。

​		forceBigDecimals：控制是否强制将 DECIMAL 和 NUMERIC 类型的 JDBC 字段转换为 Java 类型的 BigDecimal ，默认为 false。



javaModelGenerator标签：

​		控制生成的实体类，根据 context 标签中配置的 defaultModelType 属性值的不同，一个表可能会对应生成多个不同的实体类。

​		targetPackage：生成实体类存放的报名，一般都是放在该包下，实际还会受到其他配置的影响。

​		targetProject：指定目标项目路径，可以使用相对路劲或绝对路径。

​		通过 property 子标签可以配置的属性

​				constructorBased：该属性只对 MyBatis3 有效，如果为 true 就会使用构造方法入参，如果为 false 就会使用 setter方式。默认为 false。

​				enableSubPackages：如果为 true。会根据 catalog 和schema 来生成子包。如果为 false 就会直接使用 targetPackage 属性。默认为 false。

​				immutable：用来配置实体类属性是否可变。如果设置为 true，那么 constructorBased 不管设置成什么。都会使用构造方法入参，并且不会生成 setter 		方法，如果为 false ，实体类属性就可以改变，默认为 false。

​				rootClass：设置所有实体类的基类。如果设置，则需要使用类的全限定名称。并且，如果能够加载 rootClass ，那么不会覆盖和父类中完全匹配的属		性。

​				trimStrings：判断是否对数据库查询结果进行 trim 操作，默认值为 false。如果为 true 则，生成代码如下：

```java
public void setRoleName(String roleName) 
{
        this.roleName = roleName == null ? null : roleName.trim();
}
```

 

sqlMapGenerator标签：

​		用于配置 SQL 映射生成器（mapper.xml）的属性。

​		如果 targetRuntime 设置为 MyBatis3 ，则只有当 javaClientGenerator 配置需要 XML 时，该标签才必须配置一个，如果没有配置 javaClientGenerator：

​				如果指定了一个 sqlMapGenerator，那么将只生成 XML 的SQL映射文件和实体类。

​				如果没有指定 sqlMapGenerator，那么将只生成实体类。

​		targetPackage：生成SQL映射文件存放的报名。

​		targetProject：指定目标项目路径，可以使用相对路径或绝对路径。

​		可以通过 property 子标签配置 eanbleSubPackages，如果为 true，会根据 catalog 和schema 来生成子包，如果为 false 就会直接用 targetPackage 属性，默认为 false。



javaClientGenerator标签：

​		配置 Java 客户端生成 （mapper 接口）的属性。

​		type：用于选择客户端代码生成器，可以自定义实现，需要继承 AbstractJavaClientGenerator 类，必须有一个默认空的构造方法。提供了一下预设的代码生成器，首先根据 context 的 targetRuntime 分成两类：

​				MyBatis3：

​						ANNOTATEDMAPPER：基于注解的 Mapper 接口，不会有对应的 XML 映射文件。

​						MIXEDMAPPER：XML 和注解的混合形式。

​				MyBatis3Simple：

​						ANNOTATEDMAPPER：基于注解的 Mapper 接口，不会有对应的XML映射文件。

​						XMLMAPPER：所有的方法都在XML中，接口调用依赖XML文件。

​		targetPackage：生成Mapper接口存放的包名。

​		targetProject：指定目标项目路径。

​		implementationPackage：如果指定该属性，Mapper接口的实现类就会生成在这个属性指定的包中。



table标签：

​		拥有配置需要通过生辰的数据库的表，只有在其中配置过的表，才能生成最终的代码。

​		必选属性 tableName：指定要生成的表名，可以使用通配符匹配多个表。如果需要匹配全部表：

```xml
<table tableName="%" />
```

​		schema：数据库的schema，可以使用 SQL 通配符匹配，如果设置了改制，生成的 SQL 的表名会变成 schame.tableName 的形式。

​		catalog：数据库的 catalog，如果设置了该值，生成 SQL 的表名会变成 catalog.tableName 的形式。

​		alias：如果指定，这个值会用在生成的 select查询SQL表的别名和列名上。

​		domainObjectName：生成对象的基本名称。如果没有指定，会自动根据表明来生成名称。

​		enableXXX：XXX代表多种 SQL 方法，该属性用来指定是否生成对应的 XXX 语句。

​		selectByPrimaryKeyQueryId：DBA跟踪工具中会用到。

​		selectByExampleQueryId：DBA跟踪工具中会用到。

​		modelType：和 context 的 defaultModelType 含义一样，这里可以针对表配置，会覆盖 context 的 defaultModelType。

​		escapeWildCards：查询列是否对 schema 和表名中的 SQL 通配符 进行转义。对于某些驱动，当 schema 或表名中包含 SQL 通配符时，转义是必须的。有一些驱动则需要将下划线进行转义。

​		delimitIdentifiers：是否给标识符增加分隔符。默认为 false，当 catalog、schema、或tableName 中包含空白时，默认为 true。

​		delimitAllcolumns：是否对所有列添加分隔符，默认为 false。

table 下的子标签：

property：

​		constructorBased：和 javaModelGenerator 中的属性含义一样。

​		ignoreQualifiersAtRuntime：生成的 SQL 中的表名将不会包含 schema 和 catalog 前缀。

​		immutable：和 javaModelGenerator 中的属性含义一样。

​		modelOnly：配置是否值生成实体类，如果设置为 true ，就不会有 Mapper 接口，同时还会覆盖属性中的 enableXXX 方法，并且不会生成任何 CRUD 方法。如果配置了 sqlMapGenerator，并且 modelOnly 为 true ，那么 XML映射文件中只有实体对象的映射标签（resultMap）。

​		rootClass：和 javaModelGenerator 中的属性含义一样。

​		rootInterface：和 javaModelGenerator 中的属性含义一样。

​		runtimeCatalog：运行时的 catalog ，当生成表和运行环境表的 catalog 不一样时，可以使用该属性进行配置。

​		runtimeSchema：运行时的 schema，当生成表和运行环境表的 schema 不一样时，可以使用该属性进行配置。

​		runtimeTableName：运行时的 tableName，当生成表和运行环境表的 tableName 不一样时，可以使用该属性进行配置。

​		selectAllOrderByClause：该属性值会追加到 selectAll 方法后的 SQL 中，直接与 order by 拼接后添加到 SQL 末尾。

​		useActualColumnNames：如果设置为 true，那么会使用从数据库元数据获取的列名作为生成的实体对象的属性。如果为 false，将会尝试将返回的名称转换为驼峰实行。在这两种情况下，可以通过 columnOverride 标签显示指定，此时将会忽略该属性。

​		useColumnIndexes：如果为 true，生成resultMaps 时会使用列的索引，而不是结果中列名的顺序。

​		useCompoundPropertyName：如果为 true ，生成属性名的时候会将列名和列备注连接起来。



generatedKey（0或1个）：指定自动生成主键的属性。如果指定这个标签，生成的 insert 的 SQL 映射文件中插入一个 selctKey 标签。

​		column：生成列的列名。

​		sqlStatement：返回新值的 SQL 语句，如果这是一个 identity 列，则可以使用其中一个预定义的特殊值（Cloudscape、DB2、DB2_MF、Derby、HSQLDB、Informix、MySQL、SQL  Server、SYBASE、JDBC：使用这个值时，MyBatis会使用 JDBC 标准接口来获取值，这是一个独立于数据库获取标识列中的值的方法）

​		identity：当设置为 true 时，该列会被标记为 identity 列，并且 selectKey 标签会被插入到 insert后面，如果为 false，selectKey 会被插入到 insert之前。即使 type 属性指定为 post ，仍然需要将 identity 列设置为 true，这会作为从插入列表中删除该列的标志。该属性默认值为 false。

​		type：type = post 且 identity = true 时，生成的 selectKey 中的 order = AFTER。type = pre 时， identity只能为 false，生成的 selectKey 中 order = before。自动增长的列只有插入到数据库后才能获取ID，所以是 after，使用序列时，只有先获取序列之后才能插入数据库，所以是before。



columnRenamingRule：最多配置一个，在生成列之前对列进行重命名。

​		searchString：定义将要被替换的字符串对策正则表达式。

​		replaceString：替换搜索字符串列每一个匹配项的字符串。如果没有指定，就是用空字符串。



columnOverride：将某些默认计算的属性值更改为指定的值。可以配置多个。

​		column：重写的列名

​		property：要使用的 Java 属性的名称。如果没有指定，会根据列名生成。

​		javaType：列的属性值为完全限定的 Java 类型。如果需要，可以覆盖由 JavaTypeResolver计算出的类型。

​		jdbcType：列的 JDBC 类型

​		typeHandler：根据用户定义的需要用来处理列的类型处理器。必须是一个继承自 TypeHandler 接口的全限定的类名。如果没有指定或者是空白，会用默认的类型处理器来处理类型。

​		delimitedColumnName：是否应在生成的 SQL 的列名称上增加分隔符。如果列名中包含空格，会自动添加分隔符。只有当列名需要被强制为一个合适的名字或者列名是数据库中的保留字时，才是必要的。



ignoreColumn：用来屏蔽不需要生成的列。

​		column：要忽略的列名。

​		delimitedColumnName：标识匹配列名的时候是否区分大小写。



​		



​		



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

# SSM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:c="http://www.springframework.org/schema/c"
       xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:property-placeholder location="classpath:database.properties" />
    <context:annotation-config />
    <mvc:annotation-driven />


    <context:component-scan base-package="com.shanji.*" />
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/" p:suffix=".jsp" />

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**/**"/>
            <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
                <property name="paramName" value="language" />
            </bean>
        </mvc:interceptor>
    </mvc:interceptors>
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8" />
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="clone">
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="driverClassName" value="${database.driver}" />
            <property name="url" value="${database.url}" />
            <property name="filters" value="${database.filters}"/>

            <!-- 最大并发连接数 -->
            <property name="maxActive" value="${database.maxActive}"/>

            <!-- 初始化连接数量 -->
            <property name="initialSize" value="${database.initialSize}"/>

            <!-- 配置获取连接等待超时的时间 -->
            <property name="maxWait" value="${database.maxWait}"/>

            <!-- 最小空闲连接数 -->
            <property name="minIdle" value="${database.minIdle}"/>

            <!-- 最大空闲连接数 -->
            <property name="maxIdle" value="${database.maxIdle}"/>

            <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
            <property name="timeBetweenEvictionRunsMillis" value="${database.timeBetweenEvictionRunsMillis}"/>

            <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
            <property name="minEvictableIdleTimeMillis" value="${database.minEvictableIdleTimeMillis}"/>

            <property name="validationQuery" value="${database.validationQuery}"/>
            <property name="testWhileIdle" value="${database.testWhileIdle}"/>
            <property name="testOnBorrow" value="${database.testOnBorrow}"/>
            <property name="testOnReturn" value="${database.testOnReturn}"/>
            <property name="maxOpenPreparedStatements" value="${database.maxOpenPreparedStatements}"/>

            <!-- 超过时间限制是否回收 -->
            <property name="removeAbandoned" value="${database.removeAbandoned}"/>

            <!-- 1800 秒，也就是 30 分钟 -->
            <property name="removeAbandonedTimeout" value="${database.removeAbandonedTimeout}"/>

            <!-- 关闭 abanded 连接时输出错误日志 -->
            <property name="logAbandoned" value="${database.logAbandoned}"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath*:com/shanji/**/*.xml" />
    </bean>


    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <tx:annotation-driven transaction-manager="transactionManager" />

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg ref="sqlSessionFactory" />
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.shanji" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
        <property name="annotationClass" value="org.springframework.stereotype.Repository" />
    </bean>

    <bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver" />

</beans>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="3.0"
        xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
  <display-name>Archetype Created Web Application</display-name>
  
  <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationConetxt.xml</param-value>
    </context-param>

    <!-- 加载log4j的配置文件log4j.properties -->
    <context-param>
      <param-name>log4jConfigLocation</param-name>
      <param-value>classpath:log4j2.properties</param-value>
    </context-param>

    <!-- 设定刷新日志配置文件的时间间隔，这里设置为10s -->
    <context-param>
      <param-name>log4jRefreshInterval</param-name>
      <param-value>10000</param-value>
    </context-param>


    <filter>
      <filter-name>encodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
      </init-param>
      <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
      </init-param>
    </filter>
    <filter-mapping>
      <filter-name>encodingFilter</filter-name>
      <url-pattern>*.do</url-pattern>
    </filter-mapping>



    <!-- 加载Spring框架中的log4j监听器Log4jConfigListener -->
    <listener>
      <listener-class>
        org.springframework.web.util.Log4jConfigListener
      </listener-class>
    </listener>

    <!--<listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>-->
    <servlet>
      <servlet-name>dispatcher</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationConetxt.xml</param-value>
      </init-param>
      <load-on-startup>2</load-on-startup>
      <multipart-config>
        <location>E:\upload\mybatisspring</location>
        <max-file-size>5242880</max-file-size>
        <max-request-size>10485760</max-request-size>
        <file-size-threshold>0</file-size-threshold>
      </multipart-config>
    </servlet>
    <servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>

    <servlet-mapping>
      <servlet-name>dispatcher</servlet-name>
      <url-pattern>*.do</url-pattern>
    </servlet-mapping>

    <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
      <welcome-file>/qrcode.jsp</welcome-file>
    </welcome-file-list>
</web-app>
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

​		链表的底层并不是一个简单的`LinkedList`，而是一个被称为快速链表的结构，当元素较少的情况，会使用一块连续的内存存储，这个结构被称为`ZipList`，

它将所有元素彼此紧挨着一起存储。当元素比较多的时候才会改成快速链表，因为普通的链表需要附加的指针空间太大，会浪费空间，加重内存的碎片化，所有

`Redis`将链表和`ZipList`结合起来组成快速链表，也就是将多个`ZipList`使用双向指针串起来使用。

![](image/QQ截图20211209161322.png)









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

​		有序集合和集合类似，和无序集合的区别在于每一个元素除了值之外，还多了一个分数，分数为浮点数，在 Java中是使用双精度表示的，根据分数，Redis就可以支持对分数从小到大或者从大到小的排序。跟无序集合一样，值不能重复，但是分数可以一样。

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



## 基数——HyperLogLog

​		`HyperLogLog`提供**不精确**的去重技术方案。

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

​		在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时，它才会一次性转变为稠密矩

阵，占用`12KB`的空间。



## 布隆过滤器

​		`HyperLogLog`提供了不精确的去重统计功能，但如果想查询一个元素是否在这个去重集合中，`HyperLogLog`就不能实现了。

​		布隆过滤器，在去重的基础上，提供了对元素进行查询的功能，但是这个查询时不精确的，存在一定的误差。当查询时，如果结果为存在，可能元素在布隆过

滤器中并不存在，但如果查询结果为不存在，那么元素一定不在布隆过滤器中。

​		指令：

​				`bf.add`：添加元素。一次只能添加一个元素。

​				`bf.exists`：查询元素是否存在。一次只能查询一个元素。

​				`bf.madd`：一次性添加多个元素。

​				`bf.mexists`：一次性查询多个元素。

​		如果使用默认参数的布隆过滤器，它在第一个`add`时会自动创建，`Redis`也提供了自定义参数的布隆过滤器。需要在`add`之前使用`bf.reserve`指令显示创

建。如果对应的`key`已经存在，`bf.reserve`会报错。

​		`bf.reserve`的参数：

​				`error_rate`：错误率，其值越低，所需要的空间越大。默认为`0.01`。

​				`key`：布隆过滤器名称。

​				`initial_size`：预计放入的元素数量，当实际数量超过这个数值时，误判率会上升。默认为`100`。

​		每个布隆过滤器中有一个大型的位数组和多个`hash`函数`(`计算的`hash`值比较均匀，能够比较随机的将元素映射的数组中`)`。在添加元素时，会使用多个

`hash`函数对值进行`hash`，得到的值在与数组长度取模运算得到一个位置，每个`hash`函数都会得到一个位置。然后在将位数组中的这几个位置的值设置为`1`，完

成`add`操作。



​		在查询值是否存在时，也需要先将值与多个`hash`进行运算，得到多个位置，然后判断位数组中的这几个位置是否都为`1`，只要有一个位为`0`，则说明值不存

在，但如果**都为`1`**，也**不能说明值一定存在**。有可能是其他值在进行`hash`运算以后得到的位置是相同的。







## 3、事务

​		为了保证数据的一致性和安全性，`Redis`提供了事务方案，`Redis`的事务是使用`MULTI-EXEC`的命令组合，使用它提供了两个重要的保证：

​				1、事务是一个被隔离的操作，事务中的方法都会被`Redis`进行序列化并按顺序执行，事务在执行的过程中不会被其他客户端发生的命令所打断。

​				2、事务是一个原子性的操作，要么全部执行，要么什么都不会执行。

​		使用事务的三个过程：

​				1、开启事务

​				2、命令进入队列

​				3、执行事务

![](image/QQ截图20191129212713.png)

### 1、基础事务

​		`Redis`中开启事务是`multi`命令，而执行事务是`exec`命令。`mult`i到`exec`命令之间的`Redis`命令将采取进入队列的形式，直至`exec`命令的出现，才会一次

性发送队列里的命令去执行，而在执行这些命令的时候其他客户端就不能再插入任何命令了。

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

​		在执行事务命令的时候，在命令入队列时，`Redis`会检测事务的**命令是否正确**，如果不正确则会产生错误。无论之前和之后的命令都会被事务所丢弃，就变

为什么都没有执行。

​		当**命令格式正确**，因为数据结构引起的错误，则该命令执行出现错误，而其之前和之后的命令都会被正常执行。



### 2、监控事务

​		`Redis`中使用`watch`命令可以决定事务是执行还是回滚。

​		可以在`multi`命令之前使用`watch`命令监控某些键值对，然后使用`multi`命令开始事务，执行各类对数据结构进行操作的命令，这个时候这些命令就会进入

队列。当`Redis`使用`exec`命令执行事务的时候，首先会去对比被`watch`命令所监控的键值对，如果没有发生变化，那么它会执行事务队列中的命令，提交事务；

如果发生变化，那么它不会执行任何事务中的命令，而选择丢弃，然后`Redis`会去取消执行事务前的`watch`命令，此时会返回`NULL`回复给客户端告知事务执行失

败，不同的客户端对于这种情况有不同的处理，通常是抛出异常或直接返回`NULL`。可以利用此种方式实现事务的回滚。

![](image/QQ截图20191129220620.png)

​		乐观锁`(CAS`原理`)`：当一条线程去执行某些业务逻辑，但是这些业务逻辑操作的数据可能被其他线程共享了，这样会引发多线程中数据不一致的情况，为了

克服这个问题，首先，在线程开始时读取这些多线程共享的数据，并将其保存到当前线程的副本中，成为旧值。`watch`命令就是这样的一个功能。然后，开始线程

业务逻辑，有`multi`命令提供这一功能。在执行更新前，比较当前线程副本保存的旧值和当前线程共享的值是否一致，如果不一致，那么该数据已经被其他线程操

作过，此次更新失败，为了保持一致，线程就不会更新任何值，而将事务回滚；否则就认为它没有被其他线程操作过，执行对应的业务逻辑。

​		`ABA`问题：上述原理造成的问题。

![](image/QQ截图20191129221737.png)

​		在处理复杂运算时，被线程`2`修改的`X`的值可能导致线程`1`的运算出错，而最后线程`2`将`X`的值修改为原来的`A`，那么到了线程`1`运算结束的时间顺序，将

检测`X`的值是否发生变化，比对发现一致，于是提交事务，然后在复杂计算的过程中`X`被线程`2`修改过了，这会导致线程`1`的运算出错，在这个过程中，对于线

程`2`而言，`X`的值变化为：`A->B->A`。

​		可以对缓存对象加入字段`version`，每当操作一次该`POJO`，则`version`加`1`，从而保证数据的一致性。

​		

​		`Redis`在执行事务的过程中，并不会阻塞其他连接的并发，而只是通过比较`watch`监控的键值对去保证数据的一致性，所以`Redis`多个事务完全可以在非阻

塞的多线程环境中并发执行，而且`Redis`的机制是不会产生`ABA`问题，这样就有利于在保证数据一致的基础上，提高高并发系统的数据读`/`写性能。

![](image/QQ截图20191129222748.png)



​		为什么Redis不支持回滚：

​				`Redis`命令只会因为错误的语法而失败`(`并且这些问题不能在入队时发现`)`，或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，

​		失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。

​				因为不需要对回滚进行支持，所以`Redis`的内部可以保持简单且快速。

​		`Redis`主要认为失败都是使用者造成的所以就没有回滚操作。

​		在`Redis`中除了没有原子性外，`Redis`对事务也没有隔离级别的概念`(Redis`是单线程程序`)`，所以就不会产生使用关系型数据库需要关注的脏读，幻读，重

复读的问题。







## 4、管道

​		管道技术是为了解决网络延迟也产生的长时间的等待。其是一种通信协议。

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

​		管道本身并不是`Redis`服务器直接提供的技术，本质上该技术是由客户端提供的。

​		当使用客户端对`Redis`进行一次操作时，客户端需要将请求传送给服务器，服务器处理完毕后，在将响应恢复给客户端，需要花费一个网络数据包来回的时

间。当连续执行多条指令，就会花费多个网络数据包来回的时间。

![](image/QQ截图20211220155045.png)

![](image/QQ截图20211220155146.png)

​		当连续执行多条指令，客户端需要经历多次写`-`读才能完整的执行多条命令。如果将读写顺序调整，先执行所有的写操作，再执行所有的读操作，这样多条

指令同样可以正常完成，但只会花费很少的网络来回开销，就好像连续的写操作合并了，连续的读操作也合并了一样。这就是管道的本质。

​		写操作只负责将数据写到本地操作系统内核的发送缓存中然后就返回了，剩下的事交给操作系统内核异步将数据送到目标机器，但如果发送缓冲满了，那么就

需要等待缓冲空出空闲空间来，这就是写操作`IO`操作的真正耗时。

​		读操作只负责将数据从本地操作系统内核的接收缓冲中取出来就完事了，如果缓冲区是空的，那么就需要等待数据到来，这就是读操作`IO`操作的真正耗时。

​		对于一条`Redis`命令，写操作几乎没有耗时，直接写到发送缓冲中就返回了，而读操作就比较耗时，需要等待消息经过网络路由到目标机器处理后的响应信

息，在回送到当前的内核读缓冲才可以返回，这才是一个网络来回的真正开销。

​		对于管道而言，连续的写操作根本就没有耗时，之后第一个读操作会等待一个网络的来回开销，然后所有的响应信息就都已经送回到内核的读缓冲了，后续的

读操作直接从缓冲中拿结果，瞬间就返回了。













## 5、发布订阅

​		发布消息模式首先需要消息源，也就是要有消息发布出来，当消息发出，订阅者就能收到这个消息进行处理。

![](image/QQ截图20191201193425.png)

![](image/QQ截图20191201195258.png)

​		消息多播允许生产者只生产一次消息，由中间件负责将消息复制给多个消息队列，每个消息队列由响应的消费组进行消费，是分布式系统常用的一种解耦方

式，用于将多个消费组的逻辑进行拆分。

![](image/QQ截图20211220162225.png)



​		`Redis`单独使用了一个模块支持消息多播：`PubSub`。

​		发布订阅模式在实现时非常简单，它没有基于任何数据类型，也没有做任何的数据存储，它只是单纯地为生产者、消费者建立数据转发通道，把符合规则的数

据，从一端转发到另一端。在整个过程中，没有任何的数据存储，一切都是实时转发的。

​		如果消费者异常下线，重现上线后，只能接收到新的消息，在下线期间生产者发布的消息会丢失。如果使用这种方式：**消费者必须先订阅队列，生产者才能**

**发布消息，否则消息会丢失**。并且因为整个过程不进行任何的数据存储，所以在`Redis`宕机后，数据也会丢失。



​		`Spring`中的发布订阅模式，接受消息的类，是`MessageListener`接口的实现类。

​		`Redis`发布订阅监听类：

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

​		此时`Spring`中定义了监听类。但是还不能测试，还需要一个监听容器。`RedisMessageListenerContainer`，它可用于监听`Redis`的发布订阅消息。

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

​				2、追加文件：当Redis执行写命令后，在一定的条件下将执行过的写命令一次保存在Redis的文件中，将来就可以一次执行那些保存的命令恢复Redis的数据。

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

​		`Redis`也会因为内存不足而产生错误，也可能因为回收过久而导致系统长期的停顿。

​		当`Redis`的内存达到规定的最大值时，允许配置`6`中策略中的一种进行淘汰键值，并且将一些键值对进行回收。

![](image/QQ截图20191202214041.png)

​		`volatile-lru`：采用最近使用最少的淘汰策略，`Redis`将回收那么超时的`(`仅仅是超时的`)`键值对，也就是它只淘汰那些超时的键值对。

​		`allkey-lru`：采用淘汰最少使用的策略。`Redis`将对所有的`(`不仅仅是超时的`)`键值采用最近使用最少的淘汰策略。

​		`volatile-random`：采用随机淘汰策略删除超时的`(`仅仅是超时的`)`键值对。

​		`allkeys-random`：采用随机淘汰策略删除所有的`(`不仅仅是超时的`)`键值对，不常用。

​		`volatile-ttl`：采用删除存活时间最短的键值对策略。

​		`noeviction`：根本就不删除任何键值对，当内存已满，如果读操作，将正常工作，写操作，将返回错误。即只能读不能写。默认值

![](image/QQ截图20191202214737.png)

​		缺点：必须指明超时的键值对。对所有的键值对进行回收，有可能把正在使用的键值对删掉，增加了存储的不稳定性。



​		但`Redis`并不总是立即将空闲内存归还给操作系统，操作系统是以页为单位来回收内存的，如果某个页上还有一个`key`在使用，那么该页就不能被回收，但

`Redis`会重新使用那些尚未回收的空闲内存。

​		`Redis`将内存分配的细节交给了第三方内存分配库去实现，目前可以使用`jemalloc`和`tcmalloc`库，默认使用`jemalloc`。



## LRU

​		当`Redis`内存超出物理内存限制时，内存的数据会开始和磁盘产生频繁的交换，使`Redis`的性能急剧下降，为了限制最大使用内存，`Redis`提供了配置参数

`maxmemory`来限制内存超出期望大小。当实际内存超出`maxmemory`时，`Redis`提供了一些策略来决定`Redis`的行为：

​				`noeviction`：拒绝写请求`(del`请求可以继续服务`)`，读请求可以继续进行，此时可以保证不会丢失数据，但业务不能持续进行，默认的淘汰策略。

​				`volatile-lru`：尝试淘汰设置了过期时间的`key`，最少使用的`key`优先被淘汰。没有设置过期时间的`key`不会被淘汰，可以保证需要持久化的数据不会

​		突然丢失。

​				`volatile-ttl`：比较`key`的剩余寿命`ttl`的值，`ttl`越小越优先被淘汰。

​				`volatile-random`：随机淘汰设置了过期时间的`key`。

​				`allkeys-lru`：尝试淘汰所有的`key`集合，最少使用的`key`优先被淘汰。

​				`allkeys-random`：随机淘汰所有的`key`。

​		`Redis`使用的是一种近似`LRU`算法，`Redis`给每个`key`增加了一个额外的小字段，长度为`24bit`，即最后一次被访问的时间戳。当`Redis`执行写操作时，发

现内存超出`maxmemory`，就会执行一次`LRU`淘汰算法，随机采样出`5`个`key`，然后淘汰掉最旧的`key`，如果淘汰后内存还是超出`maxmemory`，则继续随机采样淘

汰，直到内存低于`maxmemory`为止。



## 10、复制



### 主从复制

​		`CAP`：一致性`(C)`、可用性`(A)`、分区容忍性`(P)`。

​		分布式系统的节点往往都是分布在不同的机器上进行网络隔离开的，此时必然会有网络断开的风险，这个场景叫做网络分区。当网络分区发生时，两个分布式

节点之间无法进行通信，对一个节点进行的修改操作将无法同步到另外一个节点，数据的一致性无法满足，即两个分布式节点的数据不在保持一致。

​		`CAP`原来：当网络分区发生时，一致性和可用性两难全。

​		`Redis`的主从数据时异步同步的，即不满足一致性要求。其保证的是最终一致性，从节点会尽力同步主节点的数据，最终从节点的状态和主节点的状态保持

一致。如果网络断开，主从节点的数据将会出现大量不一致，一旦网络恢复，从节点会采用多种策略同步数据，尽力保持和主节点一致。



#### 增量同步

​		`Redis`同步的是指令流，主节点会将那些对自己的状态产生修改性影响的指令记录在本地的内存`buffer`中，然后异步将`buffer`中的指令同步到从节点，从

节点一边执行同步的指令流来达到和主节点一致的状态，一边向主节点反馈同步的偏移量。

​		内存`buffer`是有限的，是一个定长的环形数组，如果数组内容满了，就会从头开始覆盖前面的内容。

​		如果出现网络分区，短时间内从节点无法和主节点进行同步，当网络恢复时，`Redis`的主节点中那些没有同步的指令在`buffer`中有可能已经被后续的指令覆

盖掉，从节点将无法直接通过指令流来进行同步。



#### 快照同步

​		快照同步是一个非常耗费资源的操作，首先需要在主节点上进行一次`bgsave`，将当前内存的数据全部快照到磁盘文件中，然后再将快照文件的内容全部传送

到从节点，从节点将快照文件接收完毕后，立即执行一次全量加载，加载之前要将当前内存的数据清空，加载完毕后通知主节点继续进行增量同步。

​		在整个快照同步的过程中，主节点的复制`buffer`还在不停的移动，如果同步的时间过长或者复制`buffer`太小，都会导致同步期间的增量指令在`buffer`中被

覆盖，会导致快照同步完成后无法进行增量同步，会再次发起快照同步，极有可能会陷入快照同步的死循环。当从节点刚刚加入到集群中，必须先进行一次快照同

步，同步完成后在继续进行增量同步。

​		从`Redis 2.8.18`开始支持无盘复制，即主节点直接通过套接字将快照内容发送到从节点，主节点一边遍历内存，一边将序列化的内容发送到从节点，从节点

还是先将接收到的内容存储到磁盘文件中，在进行一次性加载。







## 读写分离

​		主从架构，大概思路：

​				1、在多台数据服务器中，只有一台主服务器，而主服务器只负责写入数据，不负责让外部程序读取数据。

​				2、存在多台从服务器，从服务器不写入数据，只负责同步主服务器的数据，并让外部程序读取数据。

​				3、主服务器在写入数据后，即刻将写入数据的命令发送给从服务器，从而使得主从数据同步。

​				4、应用程序可以随机读取一台从服务器的数据，这样就分摊了读数据的压力。

​				5、当从服务器不能工作的时候，整个系统将不受影响；当主服务器不能工作的时候，可以方便地从从服务器中选举一台来当主服务器。

​		![](image/QQ截图20191204202343.png)

### 读写分离配置

​		1、首先明确主机，确定后，关键的两个配置是 dir 和 dbfilename 选项，必须保证这个两个文件是可写的。对于 Redis的默认配置而言，dir 的默认值为 "./"，而对于dbfilename的默认值为 dump.rdb。即，采用 Redis当前目录的 dump.rdb 文件进行同步 。

​		2、明确主机后，还要配置 slaveof 选项。

​					格式： slaveof  server  port

​							server：主机

​							port：端口

​				此时，当从机Redis服务重启时，就会同步对应主机的数据。如果不想让从机继续复制主机的数据了，执行 slaveof  no  one。

​		在实际的Linux环境中，配置文件 redis.conf 中有一个 bind 配置，默认为 127.0.0.1，即只允许本机访问，将其修改为 bind 0.0.0.0，其他的服务器就能访问了。



### 读写分离的过程

![](image/QQ截图20191204204151.png)

​		1、无论如何要先保证主服务器的开启，开启主服务器后，从服务器通过命令或者重启配置项可以同步到主服务器。

​		2、当从服务器启动时，读取同步的配置，根据配置决定是否使用当前数据响应客户端，但后发送 SYNC命令。当主服务器接收到同步命令的时候，就会执行 bgsave 命令备份数据，但是主服务器并不会拒绝客户端的读/写，而是将来自客户端的写命令写入缓冲期。从服务器未收到主服务器备份的快照文件的时候，会根据其配置决定使用现有数据响应客户端或者拒绝。

​		3、当 bgsave 命令被主服务器执行完后，开始向从服务器发送备份文件，这个时候从服务器就会丢弃所有现有的数据，开始载入发送的快照文件。

​		4、当主服务器发送完备份文件后，从 服务器就会执行这些写入命令。此时就会把 bgsave 执行之后的缓存区内的写命令也发送给从服务器，从服务器完成备份文件解析，就开始像往常一样，接收命令，等待命令写入。

​		5、缓冲区的命令发送完成后，当主服务器执行一条写命令后，就同时往从服务器发送同步写入命令，从服务器酒喝主服务器保持一致了。而此时当从服务器完成主服务器发送的缓冲区命令后，就开始等待主服务器的命令了。

​		在主服务器同步到从服务器的过程中，需要备份文件。所以在配置的时候一般需要预留一些内存空间给主服务器，用以腾出空间执行备份命令。

![](image/QQ截图20191204205326.png)



## 哨兵模式

​		`Redis`可以存在多台服务器，并且实现了读写分离的功能，哨兵模式是一种特殊的模式，首先`Redis`提供了哨兵的命令，哨兵是一个独立的进程，作为进

程，它会独立运行。原理是哨兵通过发送命令，等待`Redis`服务器响应，从而监控运行的多个`Redis`实例。

![](image/QQ截图20191204210111.png)

​		作用：

​				1、通过发送命令，让`Redis`服务器返回监测其运行状态，包括主服务器和从服务器。

​				2、当哨兵监测到`master`宕机，会自动将`slave`切换成`master`，然后通过发布订阅模式通知到其他的从服务器，修改配置文件，让它们切换主机。



​		客户端连接集群时，首先连接哨兵`(Sentinel)`，通过哨兵来查询主节点的地址，然后在连接主节点进行数据交互。当主节点发生故障时，客户端会重新想哨

兵要地址。哨兵会将最新的主节点地址告诉客户端，如此应用程序将无需重启即可自动完成节点切换。

​		哨兵无法保证消息完全不丢失，但提供了两个选项限制主从延迟过大：

```tex
min-slaves-to-write 1 表示正常复制的节点数，如果不满足，则停止对外写服务
min-slaves-max-lag 10 表示收到从节点反馈的时间，单位秒，如果此时间内没有收到反馈，则表示从节点不正常
```

​	

​		可以使用多个哨兵来进行多哨兵监控，多个哨兵不仅监控各个`Redis`服务器，而且哨兵之间相互监控，看看哨兵们是否还活着。

​		故障切换过程：当主服务器宕机，哨兵`1`先监测到这个结果，当时系统并不会马上进行`failover`操作，而仅仅是哨兵`1`主观地认为主机已经不可用，这个现

象被称为主观下线。当后面的哨兵监测也监测到了主服务器不可用，并且有了一定数据的哨兵认为主服务器不可用，那么哨兵之间就会形成一次投票，投票的结果

由一个哨兵发起，进行`failover`操作，在`failover`操作的过程中切换成功后，就会通过发布订阅方式，让各个哨兵把自己监控的服务器实现切换主机，这个过程

称为：客观下线。对于客户端而言，一切都是透明的。

![](image/QQ截图20191204211156.png)

### 1、搭建

![](image/QQ截图20191204212342.png)

​		在从服务器的`redis.conf`配置文件中，修改：

![](image/QQ截图20191204212425.png)

​		配置哨兵，三个哨兵分别监控三个服务器：

![](image/QQ截图20191204212523.png)

​		上述配置完后，进入`Redis`的`src`目录：

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

​		其中`sentinel  down-after-milliseconds`配置项只是一个哨兵在超过其指定额毫秒数依旧没有得到回答消息后，会自己认为主机不可用，对于其他哨兵而

言，并不会认为主机不可用。哨兵会记录这个消息，当拥有认为主观下线的哨兵到达`sentiel  monitor`所配置的数量的时候，就会发起一次新的投票，然后切换

主机，此时哨兵会重写`Redis`的哨兵配置文件，以适应新场景的需要。



## 集群

### Codis

​		`Codis`使用`Go`语言开发，是一个代理中间件，和`Redis`一样也使用`Redis`协议对外提供服务，当客户端向`Codis`发送指令时，`Codis`负责将指令转发到后面

的`Redis`实例来执行，并将返回结果再转回给客户端。

​		`Codis`上挂接的所有`Redis`实例构成一个`Redis`集群，当集群空间不足时，可以通过动态增加`Redis`实例来实现扩容需求。

​		客户端操纵`Codis`与操纵`Redis`几乎没有区别，因为它是无状态的，只是一个转发代理中间件，可以启动多个`Codis`实例，供客户端使用，每个`Codis`节点

都是对等的。单个`Codis`代理能支撑的`QPS`比较有限，通过启动多个`Codis`代理可以显著增加整体的`QPS`。

​		`Codis`默认将所有的`key`划分为`1024`个槽位，首先对客户端传的`key`进行`crc32`运算`hash`值，再将`hash`后的整数值对`1024`取模，得到`key`对应的槽位。

每个槽位都会唯一映射到后面的多个`Redis`实例之一，`Codis`会在内存中维护槽位和`Redis`实例的映射关系。

​		不同的`Codis`实例之间的槽位关系是通过分布式配置存储数据库来同步，最开始使用`zookeeper`，后来也支持`etcd`。使用`zookeeper`存储时，还提供了一个

`Dashboard`来观察和修改槽位关系。当槽位关系变化时，`Codis Proxy`会监听到变化并重新同步槽位关系，从而实现多个`Codis Proxy`之间共享相同的槽位关系配

置。

​		`Codis`对`Redis`进行了改造，增加了`SLOTSSACN`指令，可以遍历指定槽下所有的`key`，然后挨个迁移每个`key`到新的`Redis`节点，在迁移的过程中，`Codis`

还是会接收到新的请求打在当前正在迁移的槽位上，此时`Codis`无法判断`key`到底在哪个实例，所有其会强制对当前的`key`进行迁移，迁移完成后，再将请求转发

到新的`Redis`实例。



### Cluster

​		`Cluster`是官方的集群化方案，它是去中心化的，每个节点负责整个集群中的一部分数据，每个节点负责的数据多少可能不一样，它们之间通过一种特殊的二

进制协议交互集群信息。

​		`Cluster`将所有数据划分为`16384`个槽位，每个节点负责其中一部分槽位，槽位的信息存储于每个节点中，当客户端连接集群时，也会得到一份集群的槽位配

置信息，要查询某个`key`时，可以直接定位到目标节点。

​		`Cluster`默认对`key`值使用`crc16`进行运算得到`hash`值，然后再对`16384`取模得到`key`对应的槽。还允许强制将某个`key`挂在特定的槽位上，通过在`key`字

符串里面嵌入`tag`标记。

​		当客户端向一个错误的节点发出了指令后，该节点会发现指令的`key`所在的槽位并不归它管，此时会向客户端发送一个特殊的跳转指令`(MOVED)`携带目标操

作的节点地址，告诉客户端去连接这个节点以获取数据。客户端收到`MOVED`指令后，要立即纠正本地的槽位映射表，后续的所有`key`将使用新的槽位映射表。

​		`Cluster`需要手动调整槽位的分配情况。在迁移过程中，迁移的单位是槽，当一个槽正在迁移时，这个槽处理中间过渡状态。首先会源节点和目标节点设置好

中间过渡状态，然后一次性获取源节点槽位的所有`key`列表`(keysinslot`指令`)`，在挨个`key`进行迁移。迁移的过程是以源节点作为目标节点的客户端，源节点对

当前的`key`执行`dump`指令得到序列化内容，然后通过想目标节点发送`restore`指令携带系列化的内容作为参数，目标节点在进行反序列化就可以将内容恢复到目

标节点的内存中，然后返回`OK`，源节点在收到后在把`key`删除掉就完成了单个`key`的迁移过程。

​		迁移的过程是同步的，目标节点执行`restore`指令到源节点删除`key`之间，源节点的主线程会处理阻塞状态，直到`key`被删除成功。

​		在迁移的过程中对于客户端的访问，首先新旧两个节点对应的槽位都存在部分`key`数据，客户端首先尝试访问旧节点，如果数据还在旧节点，那么旧节点正

常处理。如果不在，那么旧节点会向客户端返回一个`-ASK targetNodeAddr`的重定向指令，客户端在收到这个重定向指令后，先去目标节点执行一个不带任何参数

的`ASKING`指令，然后在目标节点在重新执行原来的操作指令。

​		之所以需要先执行`ASKING`指令，是因为在迁移没有完成之前，槽位还不归新节点管理，如果直接向目标节点发送该槽位的指令，节点是不认的，会给客户端

返回`MOVED`指令`(`此指令会刷新客户端的槽位关系表`)`告诉客户端去源节点执行，如此就形成重定向循环，`ASKING`指令`(`临时纠正槽位，不会刷新客户端的槽位

关系表`)`的目的就是打开目标节点的选项，强制执行下一条指令。

​		`Cluster`可以为每个主节点设置多个从节点，主节点故障时，会自动将某个从节点提升为主节点，如果没有从节点，此时集群将完全处于不可用状态。

`cluster-require-full-coverage`参数允许部分节点发生故障，其余节点还可以继续提供对外访问。`cluster-node-timeout`参数表示某个节点失联到的时间，超过

这个时间才认定该节点出现故障，需要进行主从切换。`cluster-slave-validity-factor`参数作为倍乘系数放大超时时间来宽松容错的紧急程度。

​		`Cluster`是去中心化的，一个节点认为某个节点失联并不会触发主从切换，只有大多数节点都认定某个节点失联时，集群才认为该节点需要进行主从切换来容

错。集群节点采用`Gossip`协议来广播自己的状态以及改变对整个集群的认知。当某个节点发现另外一个失联时，会将这条信息向整个集群广播，其他节点会收到

失联信息，如果收到了某个节点失联的节点数量达到了集群的大多数，会标记该失联节点为确定下线状态，然后向整个集群广播，然后对该失联节点进行主从切

换。



​		





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



## 分布式锁

​		分布式锁本质上要实现的目标就是在`Redis`中占一个坑，当另一个线程来占坑时，发现坑里有值时，只能放弃或者稍后重试。

​		在`Redis 2.8`版本之后，通常使用`set`指令的扩展参数来实现分布式锁，即设值和过期时间是原子性的：

```shell
set id true ex 5 nx

del id
```

​		上面的方式存在一定的问题，即不能解决超时的问题。如果在加锁和释放锁之间的逻辑执行需要的时间很长，超出了锁的超时限制，锁就会被提前释放，此时

会出现并发问题。

​		为了解决这个问题，通常要求分布式锁不能用于较长时间的任务。



​		在集群环境中，当主节点挂掉，发生主从切换时，如果主节点中的分布式锁未能同步到从节点，而此时从节点又变成了主节点，当另一个客户端来加锁时，会

加锁成功，此时就会导致同一把锁被两个客户端同时拥有，产生不安全性。

​		`RedLock`算法解决了上面的问题，它需要提供多个`Redis`实例，实例之间相互独立，没有主从关系。加锁时，会向过半节点发送`set`指令，只要过半节点

`set`成功，就认为加锁成功，释放锁时，需要向所有节点发送`del`指令。相比于单个`Redis`实例，性能会有所下架。



## 队列

​		`Redis`的`list`数据结构可以作为异步消息队列使用，用`rpush`和`lpush`操作入队列，用`lpop`和`rpop`操作出队列。但这个消息队列并不是专业的消息队列，

没有`ack`保证，不适合对消息的可靠性有极高要求的场景。

​		它可以支持多个生产者和多个消费者并发进出消息，每个消费者拿到的消息都是不用的列表元素。

​		在消费消息时，客户端通常是使用循环读取，在队列为空时，循环依然会继续执行，这不仅会拉高客户端的`CPU`消耗，`Redis`的`QPS`也会被拉高。可以在循

环中使线程睡眠来解决，但是消息的延迟会增大。还可以`blpop`和`brpop`来替换上面的`lpop`和`rpop`来进行出队列操作。使用这两个指令时，如果队列中没有消息

时，则会进入阻塞状态，等有数据之后会被唤醒，相比起在循环中睡眠，这种方式消息的延迟几乎为`0`。此时，如果线程阻塞，`Redis`会主动断开连接，以减少闲

置资源占用，此时`blpop`和`brpop`会抛出异常，此时客户端需要有重连机制。



### 实现队列的方式

#### List

​		这种方式是最简单的方式，利用`list`的特性，可以使用阻塞指令来避免`CPU`空转的问题。属于**拉模型**。

​		存在的问题：

​				1、不支持重复消费：即一个消息被拉取后，就会从`list`中删除，无法被其他消费者再次消费，即不支持多个消费者消费同一批数据。

​				2、消息丢失：消费者在拉取消息后，因为某些原因异常宕机，则这条消息就丢失了。

​				3、消息积压：如果消费者消费的速度赶不上生产者生产的速度，则`list`中的消息会越来越多，造成内存的急剧增长。



#### 发布订阅

​		这种方式解决了上面的重复消费问题，支持多个生产者或消费者处理消息。属于**推模型**。

​		发布订阅模式在实现时非常简单，它没有基于任何数据类型，也没有做任何的数据存储，它只是单纯地为生产者、消费者建立数据转发通道，把符合规则的数

据，从一端转发到另一端。在整个过程中，没有任何的数据存储，一切都是实时转发的。

​				消息丢失：如果消费者异常下线，重现上线后，只能接收到新的消息，在下线期间生产者发布的消息会丢失。如果使用这种方式：**消费者必须先订阅队**

​		**列，生产者才能发布消息，否则消息会丢失**。并且因为整个过程不进行任何的数据存储，所以在`Redis`宕机后，数据 也会丢失。

​				消息积压：每个消费者订阅一个队列时，`Redis`都会在`Server`上给这个消费者在分配一个缓冲区，这个缓冲区其实就是一块内存。当生产者发布消息

​		时，`Redis`先把消息写到对应消费者的缓冲区中。之后，消费者不断地从缓冲区读取消息，处理消息。这个缓冲区其实是有上限的`(`可配置`)`，如果消费者

​		拉取消息很慢，就会造成生产者发布到缓冲区的消息开始积压，缓冲区内存持续增长。如果超过了缓冲区配置的上限，此时，`Redis`就会强制把这个消费者

​		踢下线这时消费者就会消费失败，也会丢失数据。



#### Stream

​		这种方式，可以很好的解决消息丢失的问题，在生产者发布消息时，会生成一个消息的唯一`ID`，除第一次拉取消息时，其余的拉取操作都需要传入上一次拉

取的消息的`ID`。同时支持发布`/`订阅模式，即多个消费者可以消费同一批数据。

​		除了拉取消息时用到了消息`ID`，为了保证重新消费，也要用到这个消息`ID`。当一组消费者处理完消息后，需要执行`XACK`命令告知`Redis`，这时`Redis`就会

把这条消息标记为处理完成。如果没有收到`XACK`命令，则`Redis`会保留这条消息。

​		`Stream`是新增加的数据类型，它与其它数据类型一样，每个写操作，也都会写入到`RDB`和`AOF`中，只需要配置好持久化策略，这样的话，就算`Redis`宕机重

启，`Stream`中的数据也可以从`RDB`或`AOF`中恢复回来。

​		当消息队列发生消息堆积时，一般只有`2`个解决方案：

​				生产者限流：避免消费者处理不及时，导致持续积压。

​				丢弃消息：中间件丢弃旧消息，只保留固定长度的新消息。

​		而`Redis`在实现`Stream`时，采用了第`2`个方案。在发布消息时，你可以指定队列的最大长度，防止队列积压导致内存爆炸。当队列长度超过上限后，旧消息

会被删除，只保留固定长度的新消息。此时也会存在消息丢失的情况。



​		`Redis`在以下场景下，都会导致数据丢失：

​				`AOF`持久化配置为每秒写盘，但这个写盘过程是异步的，`Redis`宕机时会存在数据丢失的可能。

​				主从复制也是异步的，主从切换时，也存在丢失数据的可能`(`从库还未同步完成主库发来的数据，就被提成主库`)`。

​		基于以上原因，**`Redis`本身的无法保证严格的数据完整性**。

​				

​				









### 延迟队列

​		延迟队列可以通过`Redis`的`zset`来实现，将消息序列化成一个字符串作为`zset`的`value`，消息的到期时间作为`score`，然后用多个线程轮询`zset`获取到期

的任务进行处理。由于有多个线程，所以需要考虑并发争抢任务，确保任务不会被多次执行。`Redis`的`zrem`方法是争抢任务的关键，它的返回值决定了有没有抢

到任务。

```java
public class RedisDelayingQueue<T> {
    static class TaskItem<T>{
        private String id;
        private T msg;

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public T getMsg() {
            return msg;
        }

        public void setMsg(T msg) {
            this.msg = msg;
        }
    }

    private Jedis jedis;
    private String queueKey;

    public RedisDelayingQueue(Jedis jedis,String queueKey){
        this.jedis = jedis;
        this.queueKey = queueKey;
    }

    public void delay(T msg) throws Exception{

        TaskItem<T> task = new TaskItem<>();
        task.setId(UUID.randomUUID().toString());
        task.setMsg(msg);

        String s = JsonUtil.ToJsonStr(task);
        jedis.zadd(queueKey, System.currentTimeMillis() + 5000,s);
    }

    public void loop() throws Exception{
        while (!Thread.interrupted()){
            List<String> values = jedis.zrangeByScore(queueKey, 0, System.currentTimeMillis(), 0, 1);

            if(values.isEmpty()){
                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                }
                catch (InterruptedException e){
                    break;
                }
                continue;
            }

            String s = values.iterator().next();
            if(jedis.zrem(queueKey,s) > 0){
                TaskItem<T> task = (TaskItem<T>)JsonUtil.ToEntity(s, TaskItem.class);
                this.handleMsg(task.getMsg());
            }
        }
    }

    public void handleMsg(T msg){
        System.out.println(msg);
    }


    public static void main(String[] args) throws Exception{
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.auth("");

        RedisDelayingQueue<String> queue = new RedisDelayingQueue<>(jedis,"q-xiaoshanshan");
        Thread producer = new Thread(){
            @Override
            public void run(){
                for(int i = 0 ; i < 10 ; i++){
                    try {
                        queue.delay("codehole" + i);
                    }
                    catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        };

        Thread consumer = new Thread(){
            @Override
            public void run(){
                try {
                    queue.loop();
                }
                catch (Exception e){
                    e.printStackTrace();
                }
            }
        };

        producer.start();
        consumer.start();

        try {
            producer.join();
            TimeUnit.MILLISECONDS.sleep(6000);
            consumer.interrupt();
            consumer.join();
        }
        catch (Exception e){
        }
    }
}
```

​		上面的实现方式中，同一个任务可能被多个线程获取后在使用`zerm`进行争抢，没有抢到的线程就浪费了一次机会，可以使用`lua scripting`来优化，将

`zrangebyscore`和`zerm`一同挪到服务端进行原子话操作。







​		

## 位图

​		位图的最小单位是`bit`，每个`bit`的取值只能是`0`或`1`。`Redis`中的位图并不是特殊的数据结构，它的内容其实是普通的字符串，即`byte`数组。可以使用

`get/set`来设置整个位图的内容，也可以使用`getbit/setbit`等将`byte`数组看成位数组来处理。

​		`Redis`的位数组时自动扩充的，如果设置了某个便宜位置超出了现有的内容范围，就会自动将位数组进行零扩容。

​		`Redis`提供了位图统计指令`bitcount`和位图查找指令`bitpos`。`bitcount`用来统计指定范围内的`1`的个数，`bitpos`用来查找指定范围内出现的第一个`0`或

`1`。但这个范围是以字节为单位，即必须是`8`的倍数。

​		`getbit/setbit`指定位的值都是单个位的，即一个只能指定或查询一个`bit`，如果一次想操作多个`bit`，可以使用`bitfield`指令，其有三个子指令：

​				`get`：从指定位开始连续读取多个`bit`，可以设置读取结果的格式`(`有符号数或无符号数`)`。

​				`set`：从指定位开始将连续多个`bit`设置为指定值。

​				`incrby`：对指定范围的位进行自增操作。此操作可能会出现溢出。其提供了溢出子指令`overflow`，可以指定溢出行为，但`overflow`只会影响接下来的

​		第一条指令，后序指令的溢出策略会变成默认值：

​						`wrap`：默认行为，即将溢出的符号位丢弃。

​						`fail`：报错，不执行。

​						`sat`：保留在最大或最小值。







## 限流

​		限流的目的是控制用户行为，避免垃圾请求。系统要限定用户的某个行为在规定的时间只能允许发生`N`此。



### 简单限流

​		使用一个利用`zset`实现一个滑动时间窗口，只保留窗口之内的数据，窗口之外的数据都可以删除。每一个行为都作为`zset`中的一个`key`保存下来，同一个

用户的同一种行为用一个`zset`记录。

![](image/QQ截图20211214162125.png)

```java
public class SimpleRateLimiter {
    private Jedis jedis;

    public SimpleRateLimiter(Jedis jedis){
        this.jedis = jedis;
    }

    public boolean isActionAllowed(String userId,String actionKey,int period,int maxCount){
        String key = String.format("hist:%s:%s", userId, actionKey); // 生成 zset 的 key 
        long nowTs = System.currentTimeMillis();

        Transaction transaction = jedis.multi();
        transaction.zadd(key,nowTs,"" + nowTs); // 第二个参数为 zset 的 score ，最后一个参数为 zset 的value
        transaction.zremrangeByScore(key,0,nowTs - period * 1000); // 移除 0 ~ nowTs - period * 1000 之间的元素，即只保留限制时间内的元素
        Response<Long> count = transaction.zcard(key); // 获取移除后，元素的个数
        transaction.expire(key,period + 1); // 设置键的过期时间，即如果限制时间内，没有收到相同的请求，则删除对应的 zset ，以
        transaction.exec();
        return count.get() <= maxCount;
    }
}
```



### 漏斗限流

​		漏斗中已使用的空间代表用户已经进行的行为，剩余空间代表当前行为可以持续进行的数量，漏斗的流水速度代表着系统允许该行为的最大频率。

​		单击版：

```java
public class FunnelRateLimiter {

    private Map<String,Funnel> funnels = new HashMap<>();

    public boolean isActionAllowed(String userId,String actionKey,int capacity,float leakingRate){
        String key = String.format("%s:%s", userId, actionKey);
        Funnel funnel = funnels.get(key);
        if(funnel == null){
            funnel = new Funnel(capacity,leakingRate);
            funnels.put(key,funnel);
        }
        return funnel.watering(1);
    }



    static class Funnel{
        private int capacity;
        private float leakingRate;
        private int leftQuota;
        private long leakingTs;

        public Funnel(int capacity,float leakingRate){
            this.capacity = capacity; // 漏斗的容量
            this.leakingRate = leakingRate; // 漏水速度
            this.leftQuota = capacity; // 剩余空间
            this.leakingTs = System.currentTimeMillis(); // 漏水时间
        }

        void makeSpace(){
            long nowTs = System.currentTimeMillis();
            long deltaTs = nowTs - leakingTs; // 距离上一次漏水的时间
            int deltaQuota = (int)(deltaTs * leakingRate); // 计算相邻行为发生之间腾出的空间
			
            // 间隔时间太长，整数数字过大溢出，即水漏完了
            if(deltaQuota < 0){
                this.leftQuota = capacity;
                this.leakingTs = nowTs;
                return ;
            }

            // 腾出空间太小，最下单位是 1，即距离上一次行为发生时间很短
            if(deltaQuota < 1){
                return ;
            }

            this.leftQuota += deltaQuota;
            this.leakingTs = nowTs;

            if(this.leftQuota > this.capacity){
                this.leftQuota = this.capacity;
            }
        }

        boolean watering(int quota){
            makeSpace();
            if(this.leftQuota >= quota){
                this.leftQuota -= quota;
                return  true;
            }
            return false;
        }

    }
}
```

​		分布式版：

​		`Redis 4.0`提供了一个限流`Redis`模块：`Rdis-Cell`，该模块使用了漏斗算法，并提供饿了原子的限流指令，该模块只有一条指令`cl.throttle`：

```shell
cl.throttle xiaoshanshan:reply 15 30 60 1 # 整个指令的意思是允许某行为的频率为：每 60s 最多 30 次，漏斗的初始容量为 15，最后一个参数代表使用的空间

#返回值有 5 个：
#第一个：是否被允许：0 表示允许，1 表示拒绝
#第二个：漏斗的容量。
#第三个：漏斗剩余空间。
#第四个：如果被拒绝，需要多长时间后重试。单位为秒
#第五个：多长时间后，漏斗完全空出来
```





## GeoHash

​		`Redis`的`Geo`模块支持地理位置，即可以使用该模块实现附近的人类似的功能。

​		地图元素的位置数据使用二维的经纬度表示。`Redis`的`Geo`模块使用`GeoHash`算来来进行地理位置距离排序。该算法将二维的经纬度数据映射到一维的整数

数组上，所有的元素都将挂载到一条线上，距离靠近的二维坐标映射到一维后的点之间的距离也会很接近。实现附近的人这种需求时，首先将目标位置映射到这条

线上，然后在这个一维的线上获取附近的点就行了。

​		`GeoHash`算法将整个地球看成一个二维平面，然后将其划分为一系列正方形的额方格，所有的地图元素被放置于唯一的方格中，方格越小，坐标越精确，然后

对这些方格进行整数编码。正方形划分的越小，二进制整数也越长，精度也越高。`GeoHash`算法继续对这个整数做一次`base32`编码，变成一个字符串。

​		在`Redis`中，经纬度使用`52`位的整数进行编码，然后放进`zset`中，`value`就是元素的`key`，`score`是`GeoHash`的`52`位的整数值。通过`zset`的`score`排序

就可以得坐标附近的其他元素，然后将`score`还原成坐标值就可以得到元素的原始坐标。





## scan

​		在进行`Redis`维护时，通常需要查找拥有指定前缀的`key`，`Redis`提供了一个简单粗暴的指令`keys`，该指令有两个明显的缺点：

​				1、无法限制结果条数，即一次性获取出所有满足条件的`key`。

​				2、`keys`使用的遍历算法，如果`Redis`中的`key`过多，那么该指令会造成服务器卡顿，因为`Redis`是单线程程序，顺序执行所有指令。

​		在`Redis 2.8`中加入了`scan`指令：

​				1、`scan`通过游标分布进行，不会阻塞线程。

​				2、提供`limit`参数，可以控制每次返回结果的条数。

​				3、同`keys`一样，提供模式匹配。

​				4、服务器不需要保存游标状态，游标的位置状态就是`scan`返回的游标整数。

​				5、返回的结果可能会有重复，需要客户端去重。

​				6、遍历的过程中如果有数据修改，改动后的数据能不能遍历到是不确定的。

​				7、单次返回的结果为空，并不意味着遍历结束，而要看返回的游标值是否为`0`。

​		`Redis`中的所有`key`都存储在一个很大的字典中，其是一个数组`+`链表的结构。`scan`指令返回的游标就是数组的位置索引。`limit`参数就表示需要遍历的一

维数组的元素个数。

​		`scan`的遍历采用了高位进位加法，避免在扩容和缩容是重复和遗漏遍历数组中的元素。

​		在对一维数组进行扩容时，`Redis`为了避免卡顿现象采用渐进式`rehash`，同时保留旧数组和新数组，然后在定时任务中以及后序对`hash`的指令操作中渐渐

地将旧数组中的元素迁移到新数组中。此时指令需要同时访问新旧两个数组，然后将结果融合，

​		`scan`除了可以遍历所有的`key`之外，还可以对指定的容器结合进行遍历。



## 线程I/O模型

### 非阻塞IO

​		当调用套接字的读写方法，默认是阻塞的。`read`方法在读取时，如果读不到数据就会阻塞；`write`一般不会阻塞，除非内核为套接字分配的写缓冲区满了，

此时才会阻塞。

![](image/QQ截图20211216155041.png)



​		非阻塞`IO`在套接字对象上提供了一个选项`Non_Blocking`，当这个选项打开时，读写方法不在阻塞，能读多少读多少，能写多少写多少。

​		事件轮询`API(Java`中的`NIO)`用来通知线程，可以再次进行读`/`写，拿到事件后，线程挨个处理相应的事件，处理完了继续轮询。即线程进入一个死循环，

把这个循环称为事件循环，一个循环为一个周期。

![](image/QQ截图20211216160409.png)

```python
read_events,write_events = select(read_fds,write_fds,timeout) # select 系统提供的最简单的事件轮询 API
for event in read_events:
    handle_read(event.fd)
for(event in write_events):
    handle_write(event.fd)
handle_others()
```

​		通过系统调用同时处理多个通道描述符的读写事件，将这类系统调用称为多路复用`API`，现代操作系统的多路复用`API`已经改为`epoll`和`kqueue`。



​		`Redis`会将每个客户端套接字都关联一个指令队列，客户端的指令通过队列来排队进行顺序处理，先到先服务。同样也会为每个客户端套接字关联一个响应

队列，通过响应队列来将指令的结果恢复给客户端。

​		`Redis`的定时任务会记录在一个最小堆的数据结构中，在这个堆中，最快要执行的任务排在堆的最上方，在每个循环周期里，`Redis`都会对最小堆里面已经

到时间点的任务进行处理，处理完毕后，将最快要执行的任务还需要的时间记录下来，这个时间就是多路复用`API`中的`timeout`参数。



## 通信协议

​		`RESP`是`Redis`序列化协议，是一种直观的文本协议，优势在于实现过程简单，解析性能极好。

​		`RESP`协议将传输的结构数据分为`5`中最小单元类型，单元结束时统一加上回车换行符号`\r\n`：

​				单行字符串：以`+`符号开头。

```tex
hello world
+hello world\r\n
```

​				多行字符串：以`$`符号开头，后跟字符串长度。

```tex
hello
world
$11\r\nhello\r\nworld\r\n
```

​				整数值：以`:`符号开头，后跟整数的字符串形式。

```tex
1024
:1024\r\n
```

​				错误消息：以`-`符号开头。

​				数组：以`*`符号开头，后跟数组的长度。

```tex
[1,2,3]
*3\r\n:1\r\n:2\r\n:3\r\n
```

​		`NULL`：

```tex
$-1\r\n
```

​		空串：

```tex
$0\r\n\r\n
```



​		客户端向服务器发送的指令只有一种格式，多行字符串数组。

​		服务器向客户端响应要支持多种数据结构，其都是下面`5`种基本类型的组合：

​				单行字符串响应：

```tex
OK
+OK
```

​				错误响应：

```tex
(error) ERR value is not an integer or out of range
-ERR value is not an integer or out of range
```

​				整数响应：

```tex
1
:1
```

​				多行字符串响应：

```tex
"codehole"

$8
codehole
```

​				数组响应：

```tex
1) "name"
2) "xxx"
3) "age"

*3
$4
name
$3
xxx
$3
age
```



## 持久化

​		`Redis`的持久化机制有两种，一种是快照，另一个中时`AOF`日志。

​		快照是一次全量备份，是内存数据的二进制序列化形式，在存储上非常紧凑。

​		`AOF`日志是连续的增量备份，记录的是内存数据修改的指令记录文本，在长期的运行过程中会变得无比庞大，重启时需要加载`AOF`日志进行指令重发，时间

会很长，所以需要定期对`AOF`进行重写。



### 快照

​		`Redis`是单线程程序，同时负责多个客户端套接字的并发读写操作和内存数据结构的逻辑读写。在服务线上请求的同时，`Redis`还需要进行内存快照，内存

快照要求`Redis`必须进行文件`IO`操作，文件`IO`操作时不能使用多路复用`API`。

​		`Redis`使用操作系统的多进程机制来实现快照持久化，在持久化时会调用`glibc`的函数`fork`产生一个子进程，快照持久化完全交给子进程来处理，父进程继

续处理客户端请求。子进程刚产生时，和父线程共享内存里面的代码段和数据段，数据段是由很多操作系统的页面组合而成，当父进程对其中一个页面的数据进行

修改时，会将被共享的页面复制一份分离出来，然后对这个复制的页面进行修改。子进程相应的页面时没有变化的，它可以继续遍历数据，进行序列化写磁盘。



### AOF

​		`Redis`在收到客户端修改指令后，进行参数校验、逻辑处理后将该指令存储到`AOF`日志中，即先执行指令后日志存盘。`Redis`提供了`bgrewriteaof`指令对

`AOF`日志进行瘦身，原理是开辟一个子进程对内存进行遍历，转换成一系列`Redis`的操作指令，序列化到一个新的`AOF`日志文件中，序列化完毕后再将操作期间发

生的增量`AOF`日志追加到这个新的`AOF`日志文件中，追加完毕后就立即替代旧的`AOF`日志文件，瘦身工作就完成了。

​		`AOF`日志是以文件的形式存在的，当程序对日志进行写操作时，实际上是将内容写到内核为文件描述符分配的一个内存缓存中，然后内核异步将数据刷回到

磁盘中。如果此时服务器宕机，日志将会丢失，`glibc`提供了`fsync(int fd)`函数可以将指令文件的内容强制从内核缓存中刷回到磁盘`(`磁盘`IO`操作很慢`)`。

`Redis`通常每个`1s`左右执行一次`fsync`操作。`Redis`还提供了另外两种策略：永不调用`fsync`，让操作系统决定合适同步磁盘；每个指令都调用`fsync`。这两种

策略基本不会使用。

​		

​		通常`Redis`的主节点不会进行持久化操作，持久化操作主要在从节点进行。只要有一个从节点数据同步正常，数据就不会轻易丢失。



### 混合持久化

​		如果只是用快照，可能会丢失大量数据，如果只是用`AOF`日志重放，又会很慢。`Redis 4.0`提供了一个新的持久化选项：混合持久化。

​		混合持久化将快照文件的内容和增量的`AOF`日志文件存放在一起，此时`AOF`日志不再是全量的日志，而是自持久化开始到持久化结束的这段时间发生的增量

`AOF`日志。

​		在`Redis`重启时，先加载快照的内容，然后在重放`AOF`日志。在保证数据不丢失的情况下，大幅提升效率。





## 小对象压缩

​		如果`Redis`内部管理的集合数据结构很小，它会使用紧凑存储形式压缩存储。

​		`Redis`的`ziplist`是一个紧凑的字节数组结构，每个元素之间都是紧挨着的。如果存储的是`hash`结构，那么`key`和`value`会作为两个`entry`被相邻存储，如

果存储的是`zset`结构，那么`value`和`score`会作为两个`entry`被相邻存储。

![](image/QQ截图20211220163817.png)

​		`Redis`的`intset`是一个紧凑的整数数组结构，用于存放元素都是整数且元素个数较少的`set`集合。如果整数用`uint16`表示，那么`intset`的元素就是`16`位

的数组；如果新加入的整数超过了`uint16`的表示范围，就用`uint32`表示；如果新加入的整数超过了`uint32`的表示范围，就用`uint64`表示。

![](image/QQ截图20211220164236.png)

​		如果`set`存储的是字符串，则会采用`hashtable`结构。

​		当集合对象的元素不断增加，或者某个`value`值过大，小对象存储会被升级为标准存储。

```tex
hash-max-zipmap-entries 512                    # hash 的元素个数超过 512 就必须用标准结构存储
hash-max-zipmap-value 64                       # hash 的任意元素的 key/value 的长度超过 64 就必须用标准结构存储
list-max-ziplist-entries 512                   # list 的元素个数超过 512 就必须用标准结构存储
list-max-ziplist-value 64                      # list 的任意元素的长度超过 64 就必须用标准结构存储
zset-max-ziplist-entries 128                   # zset 的元素个数超过 128 就必须用标准结构存储
zset-max-ziplist-value 64                      # zset 的任意元素的长度超过 64 就必须用标准结构存储
set-max-intset-entries 512                     # set 的整数元素个数超过 512 就必须用标准结构存储
```





## Stream

​		`Stream`是一个支持多播的可持久化消息队列，其借鉴了`kafka`的设计。

![](image/QQ截图20211230151426.png)

​		`Stream`内部有一个消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的`ID`和对应的内容，消息是持久化的，`Redis`重启后，内容还在。

​		每个`Stream`都有唯一的名称，即`Redis`的`key`，在首次使用`xadd`追加消息时会自动创建。其可以挂载多个消费者组，每个消费者组会有一个游标

`last_delivered_id`在`Stream`数组之上往前移动，表示当前消费者组已经消费到哪条消息了。每个消费者组都有一个`Stream`内唯一的名称，消费者组不会自动创

建，需要单独使用`xgroup create`进行创建，需要指定从`Stream`的某个消息`ID`进行消费，这个`ID`用来初始化`last_delivered_id`变量。各个消费者组之间是独

立的，即`Stream`内部的消息会被每个消费者组都消费到。

​		一个消费者组可以挂载多个消费者，这些消费者之间是竞争关系，任意一个消费者读取了消息都会使游标`last_delivered_id`往前移动，每个消费者有一个组

内唯一的名称。

​		消费者内部有一个数组`pending_ids`，记录了当前已经被客户度读取，但还没有`ack`的消息。这个数据结构确保了客户度至少消费了消息一次，而不会在网络

传输的中途丢失了而没被处理。

​		消息`ID`的形式是`timestampInMillis-sequence`，消息`ID`由服务器自动生成，也可由客户度指定，形式必须是：整数`-`整数。而且后面加入的消息的`ID`必须

要大于前面的消息`ID`。

```shell
xadd # 向 Stream 追加消息，可以提供了一个定长藏毒参数 maxlen ，用于指令消息链表的最大长度
xdel # 从 Stream 中删除消息，仅仅是设置标志位
xrange # 获取 Stream 中的消息列表，会自动过滤已经删除的消息
xlen # 获取 Stream 的消息长度
del # 删除整个 Stream 消息列表中的所有消息
```



​		可以在不定义消费者组的情况下进行`Stream`消息的独立消费，而且可以阻塞等待。`xread`指令可以将`Stream`当成普通的消息队列来使用，使用时需要将上

次返回的最后一个消息`ID`作为参数，才能继续消费后面的消息。

​		`xreadgroup`指令用于进行消费者组的组内消费，需要提供消费者组名称，消费者名称和起始消息`ID`，其也可以阻塞等待消息，读到新消息后，对应的消息

`ID`会进入消费者的`pending_ids`数组，客户端处理完毕后使用`xack`指令通知服务器，该消息`ID`会从`pending_ids`中删除。



​		当消费者客户端读取`Stream`后，突然宕机断开连接，因为消息`ID`此时保存在`pending_ids`数组中，当客户端重新连上后，可以再次收到`pending_ids`中的消

息`ID`列表，此时`xreadgroup`的起始消息`ID`必须是任意有效的消息`ID`，一般设置为`0-0`，表示读取所有`pending_ids`中的消息以及自`last_delivered_id`之后的

新消息。

​		`Stream`的高可用建立在主从复制上，在集群环境和哨兵机制下也支持高可用，但在发生主从切换时，由于主从复制的指令复制是异步进行的，此时可能会丢

失极小部分数据，这种情况同其他数据结构是一样的。

​		`Redis`服务器没有原生支持分区的能力，如果想要分区，则需要分配多个`Stream`，然后在客户端使用一定的额策略来生产消息到不同的`Stream`。



## info

​		`info`指令显示`Redis`内部的运行参数，分为`9`大块：

​				`server`：服务器运行的环境参数。

​				`clients`：客户端相关信息。

​				`memory`：服务器运行内存统计数据。

​				`persistence`：持久化信息。

​				`stats`：通用统计数据。

​				`replication`：主从复制相关信息。

### 常见参数

​		`stats`中，`instantaneous_ops_per_sec`，该参数表示每秒执行多少次指令。

​		`clients`中，`connected_clients`，该参数表示正在连接的客户端数量，如果发现异常连接，可以使用`client list`指令列出所有客户端连接地址来确定源

头；`rejected_connections`，该参数表示超出最大连接数限制而被聚聚的客户端连接次数，如果过大，需要调整`maxclients`数量。

​		`replication`中，`repl_backlog_size`，该参数表示复制积压缓冲区的大小，这个直接影响主从复制的效率，里面存放的是从节点断开连接而后又恢复连接过

程中，对主节点中的数据产生修改性的指令，该区域是一个环形数组，如果缓冲区太小，而从节点断开的时间过长，前面的指令将会被覆盖，导致从节点无法快速

恢复断开过程中的主从同步过程，而进入全量同步模式。此区域被多个从节点共享。通过查看`stats`中的`sync_partial_err`参数可以决定是否需要扩大积压缓冲

区，该参数表示主从半同步复制失败的次数。









## 过期策略

​		`Redis`所有的数据结构又可以设置过期时间，`Redis`会将每个设置了过期时间的`key`放入一个独立的字典中，然后定时遍历这个字典来删除到期的`key`，在

删除时使用惰性策略，即在客户端访问这个`key`时，才对这个`key`进行过期检查，如果过期了就立即删除。

​		`Redis`默认每`10`秒进行一次过期扫描，扫描不会遍历过期字典中的所有`key`：

​				1、从过期字典中随机选出`20`个`key`。

​				2、删除这`20`个`key`中已经过期的`key`。

​				3、如果过期的`key`的比列超过`1/4`，则重复这个过程。

​		并且在扫描的过程中，为了预防线程卡死的线程，其还增加了扫描时间的上限，默认`25ms`。

​		从节点不会进行过期扫描，主节点在对`key`到期时，会在`AOF`文件中增加一条`del`指令，同步到所有的从节点，从节点通过执行这个`del`指令来删除过期的

`key`。在此过程中，由于指令的同步是异步进行的，如果主节点的`del`指令没有及时同步到从节点，可能会出现主从数据的不一致。



​		**缓存穿透**：查询一个一定不存在的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的

数据每次请求都要到存储层去查询，失去了缓存的意义。在流量大时，可能数据库就挂掉了。最常见的解决方案是采用布隆过滤器，将所有可能存在的数据哈希到

一个足够大的`bitmap`中，一个一定不存在的数据会被这个`bitmap`拦截掉，从而避免了对底层存储系统的查询压力。另外也有一个更为简单粗暴的方法，如果一个

查询返回的数据为空，仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。



​		**缓存雪崩**：指在设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到数据库，数据库瞬时压力过重雪崩。解决方案：用加锁或

者队列的方式保证缓存的单线程`(`进程`)`读`/`写，从而避免失效时大量的并发请求落到底层存储系统上。还可以将缓存失效时间分隔开，即在失效时间上增加一

个随机值。



​		**缓存击穿**：缓存在某个时间点过期的时候，恰好在这个时间点对这个`key`有大量的并发请求过来，这些请求发现缓存过期一般都会从后端数据库加载数据并

回设到缓存，这个时候大并发的请求可能会瞬间把后端数据库压垮。这个和缓存雪崩的区别在于：这里针对某一`key`缓存，前者则是很多`key`。解决方案：

​				1、使用互斥锁：即在缓存失效的时候`(`判断拿出来的值为空`)`，不是立即去加载数据库，而是先使用缓存工具的某些带成功操作返回值的操作去`set`

​		一个`mutex key`，当操作返回成功时，再进行加载数据库的操作并回设缓存；否则，就重试整个`get`缓存的方法。

​				2、提前使用互斥锁：在`value`内部设置`1`个超时值，超时值比实际的缓存过期时间小。当从缓存读取到超时值发现它已经过期时候，马上延长超时值并

​		重新设置到缓存。然后再从数据库加载数据并设置到缓存中。

​				3、永远不过期：从`redis`上看，确实没有设置过期时间，这就保证了，不会出现热点`key`过期问题，也就是物理不过期。 从功能上看，把过期时间存

​		在`key`对应的`value`里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是逻辑过期，唯一不足的就是构建缓存时候，其余线程`(`非构

​		建缓存的线程`)`可能访问的是老数据，但是对于一般的互联网功能来说这个还是可以忍受。







​		

## 异步指令

​		通常认为`Redis`是单线程的，但`Redis`的内部并不是单线程，有一些异步线程专门处理一些耗时的操作。

### unlink

​		`del`指令会直接释放对象的内存，但如果删除的`key`是一个非常大的对象，删除操作就会导致`Redis`卡顿。`unlink`指令，能对删除操作进行懒处理，丢给后

台线程来异步回收内存。当执行`unlink`时，相应的对象就不会被主线程访问，所以不会出现线程安全问题。



### flush

​		`flushdb`和`flushall`指令用来清空数据库，其是一个极其缓慢的操作，通过在指令后面增加`async`参数，可以将清除工作交给后台线程执行。



​		当执行上面的删除操作时，会将这个`key`的内存回收操作包装成一个任务，塞进异步任务队列，后台线程会从这个异步队列中取任务。任务队列被主线程和

异步线程同时操作。对于`unlink`，如果对应的`key`所占用的内存很小，此时对应的内存会立即被回收。

​		`Redis`每`1`秒会同步`AOF`日志到磁盘，确保消息尽量不丢失，需要调用`sync`函数，此操作会导致主线程的效率下降，`Redis`将这个操作移到异步线程来执

行。执行`AOF Sync`操作的线程是一个独立的异步线程，拥有一个自己的任务队列，该队列李只用来存放`AOF Sync`任务。

​		`Redis`对于其他删除操作也提供了异步机制，需要配置额外的选项：

​				`slave-lazy-flush`：从节点接收完`rdb`文件后的`flush`操作。

​				`lazyfree-lazy-eviction`：内存达到`maxmemory`时进行淘汰。

​				`lazyfree-lazy-expire key`：过期删除。

​				`lazyfree-lazy-server-del rename`：指令删除`destKey`。







## 安全通信

​		当应用和`Redis`部署在不同的地区时，如果使用普通`TCP`直接访问，传输数据会暴露在公网上，不安全，存在被窃听的风险。`SSL`代理可以让通信数据得到

加密，官方推荐的的工具为：`spiped`。

​		`spiped`会在客户端和服务器各启动一个`spiped`进程。客户端的`spiped`进程负责接收来自客户端发送过来的请求数据，加密后传送到服务器的`spiped`进

程，服务器的`spiped`进程将接收到的数据解密后传送到服务器，然后服务器再走一个反向的流程将响应回复给客户端。

​		每个`spiped`进程都会有一个监听端口用来接收数据，同时还会作为一个客户端将数据转发到目标地址。`spiped`进程需要成对出现，相互之间需要使用相同的

共享密钥来加密消息。





## 源码

### 字符串

​		`Redis`中的数据对象`redisObject`是`Redis`对内部存储的数据定义的抽象类型：

```c++
// server.h 
typedef struct redisObject {
    unsigned type:4; // 数据类型
    unsigned encoding:4; // 编码格式，即存储数据使用的数据结构，同一个类型的数据，Redis 会根据数据量、占用内存等情况使用不同的编码，最大限度节省内存
    unsigned lru:LRU_BITS; // 24位，LRU 时间戳或 LFU 计数
    int refcount; // 引用计数，为了节省内存，Redis 会在多处引用同一个 redisObject
    void *ptr; // 指向知己的数据结构
} robj;
```

​		`redisObject`负责装载`Redis`中的所有键和值。`type,encoding,lru`使用了位段定义，三个属性使用同一个`unsigned int`的不同`bit`位，最大限度节省内

存。

​		数据类型和编码：

|  数据类型  |       说明        |          编码          | 使用的数据结构  |
| :--------: | :---------------: | :--------------------: | :-------------: |
| OBJ_STRING |      字符串       |    OBJ_ENCODING_INT    | long long、long |
|            |                   |  OBJ_ENCODING_EMBSTR   |     string      |
|            |                   |    OBJ_ENCODING_RAW    |     string      |
|  OBJ_LIST  |       列表        | OBJ_ENCODING_QUICKLIST |    quicklist    |
|  OBJ_SET   |       集合        |    OBJ_ENCODING_HT     |      dict       |
|            |                   |  OBJ_ENCODING_INTSET   |     intset      |
|  OBJ_ZSET  |     有序集合      |  OBJ_ENCODING_ZIPLIST  |     ziplist     |
|            |                   | OBJ_ENCODING_SKIPLIST  |    skiplist     |
|  OBJ_HASH  |       散列        |    OBJ_ENCODING_HT     |      dict       |
|            |                   |  OBJ_ENCODING_ZIPLIST  |     ziplist     |
| OBJ_STREAM |      消息流       |  OBJ_ENCODING_STREAM   |       rax       |
| OBJ_MODULE | Moudle 自定义类型 |    OBJ_ENCODING_RAW    |  Moudle 自定义  |



​		`C`语言中将空字符结尾的字符数组作为字符串，`Redis`做了扩展，定义了字符串类型`sds`：

```c++
typedef char *sds;


struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; // 已使用字节长度，即字符串长度，最大长度为 2^8 - 1
    uint8_t alloc; // 已申请字节长度，即 sds 总长度
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```





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

# JWT

## 1、认证流程

**传统的session认证**：

​		http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证。

**弊端**：

​		session：每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。

​		扩展性： 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

​		CSRF：因为是基于cookie来进行用户识别的， cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。



![](image/QQ截图20201010144117.png)

**JWT认证过程**：

​		JWT： (JSON Web Token)，一个开放标准(RFC 7519)，它定义了一种紧凑且自包含的方式，以JSON对象的形式在各方之间安全地传输信息。可以验证和信任

该信息，因为它是数字签名的。JWTs可以使用秘密(使用HMAC算法)或使用RSA或ECDSA的公钥/私钥对签名。

**流程：**

​		首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输(https协议)从而

避免敏感信息被嗅探。

​		后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload(负载)，将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是

一个形同xxx.xxx.xxx的字符串。

​		后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即

可。
		前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)

​		后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确，检查Token是否过期;检查Token的接收方是否是自己(可选)。

​		验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。



## 2、结构

组成：

​		1、标头（Header）：由两部分组成，令牌的类型和所使用的的签名算法。然后使用 Base64（编码方式，可以被还原，并不是加密过程） 编码组成 JWT 结构的第一部分。

```json
{
    "alg" : "HS256",
    "typ" : "JWT"
}
```

​		2、有效荷载（Payload）：其中包含声明。声明是有关实体和其他数据的声明。同样会使用 Base 64 编码组成 JWT 的第二部分。此部分不应该包含敏感数据。

```json
{
    "sub" : "123456789",
    "name" : "xiaoshanshan",
    "admin" : true
}
```

​		3、签名（Signature）：需要使用编码后的 header 和 payload 以及我们提供的一个密钥，然后使用 header 中指定的签名算法进行签名。作用是保证 JWT 没有被篡改过。 

​		**通常表示如下：xxx.yyy.zzz（Header.Payload.Signature）**



## 3、生成token

```xml
	<dependency>
      <groupId>com.auth0</groupId>
      <artifactId>java-jwt</artifactId>
      <version>3.10.3</version>
    </dependency>
```

```java
		String secret = "@xiaoshanshan";  //密钥

        String id = UUID.randomUUID().toString();

        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.SECOND,30);

        Map<String, Object> head = new HashMap<>();
        String token = JWT.create()
                .withHeader(head) // header
                .withClaim("id", id) // payload
                .withClaim("name","xiaoshanshan")
                .withExpiresAt(instance.getTime()) //过期时间
                .sign(Algorithm.HMAC256(secret)); //signature

        System.out.println(token);


eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoieGlhb3NoYW5zaGFuIiwiaWQiOiJhNTNlNmViYS0yYWZlLTQxYTQtYWRjZC1mZjBmYjQ4YzQ1MzciLCJleHAiOjE2MDI0MjM1MzR9.3rQUXsSNFttC6L50bWyxVg1_GFMb8siNkJja6nP9ERI
```

## 4、验证、解析

```java
		// 先验签，在解码
		JWTVerifier build = JWT.require(Algorithm.HMAC256(secret)).build();
        DecodedJWT verify = build.verify(token);
        System.out.println(verify.getClaim("id").asString()); // 获取payload信息，存的 xx 类型，这里就要调用 asxx 方法，否则会返回 null
        System.out.println(verify.getClaim("name").asString());
		System.out.println(verify.getExpiresAt()); // 获取过期时间
```

​		常见异常：

​				1、SignatureVerificationException：签名不一致异常

​				2、TokenExpiredException：令牌过期异常

​				3、AlgorithmMismatchException：算法不匹配异常

​				4、InvalidClaimException：失效的payload异常

```java
public class JWTUtil
{
    private static final String SECRET = "@xiaoshanshan";

    public static String getToken(Map<String,String> map)
    {
        JWTCreator.Builder builder = JWT.create();
        map.forEach((k,v)->
        {
            builder.withClaim(k,v);
        });

        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.DATE,7);

        builder.withExpiresAt(calendar.getTime());
        String token = builder.sign(Algorithm.HMAC256(SECRET));
        return token;
    }

    public static DecodedJWT verify(String token)
    {
        JWTVerifier build = JWT.require(Algorithm.HMAC256(SECRET)).build();
        return build.verify(token);
    }
}
```



  











