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



### AOP

​		使用`XML`方式开启`AOP`，需要在`XML`中配置`<aop:aspectj-autoproxy />`，如果声明了自定义的注解，那么就一定会在程序中的某个地方注册了对应的解析

器：

```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.5+ XSDs
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
        /**
        	注册 AOP 解析器
        */
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace in 2.5+
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}

}
```

​		所有解析器，因为是对`BeanDefinitionParser`接口的统一实现，入口都是从`parse`函数开始的：

```java
// AspectJAutoProxyBeanDefinitionParser 类
public BeanDefinition parse(Element element, ParserContext parserContext) {
    	// 注册 AnnotationAwareAspectJAutoProxyCreator
		AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);
    	// 对于注解中子类的处理
		extendBeanDefinition(element, parserContext);
		return null;
}
```

```java
// AopNamespaceUtils 类
public static void registerAspectJAnnotationAutoProxyCreatorIfNecessary(
			ParserContext parserContext, Element sourceElement) {
		// 注册或升级 AutoProxyCreator 定义 beanNam e为org.Springframework.aop.config.linternalAutoProxyCreator 的 BeanDefinition
		BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
				parserContext.getRegistry(), parserContext.extractSource(sourceElement));
	    // 对于 proxy-target-class 以及 expose-proxy 属性的处理
		useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
    	// 注册组件并通知，便于监听器做进一步处理
		// 其中 beanDefinition 的 className 为 AnnotationAwareAspectJAutoProxyCreator
		registerComponentIfNecessary(beanDefinition, parserContext);
}
```



#### 注册或者升级AnnotationAwareAspectJAutoProxyCreator

​		对于`AOP`的实现，基本上都是靠`AnnotationAwareAspectJAutoProxyCreator`去完成，它可以根据`@Point`注解定义的切点来自动代理相匹配的`bean`。

```java
// AopConfigUtils 类
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(
			BeanDefinitionRegistry registry, @Nullable Object source) {

		return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}

private static BeanDefinition registerOrEscalateApcAsRequired(
			Class<?> cls, BeanDefinitionRegistry registry, @Nullable Object source) {

		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		// 如果已经存在了自动代理创建器且存在的自动代理创建器与现在的不一致，那么需要根据优先级来判断到底需要使用哪个
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
            // 如果已经存在自动代理创建器并且与将要创建的一致，那么无须再次创建
			return null;
		}

		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
}
```



#### 处理 proxy-target-class 以及 expose-proxy 属性

```java
// AopNamespaceUtils 类
private static void useClassProxyingIfNecessary(BeanDefinitionRegistry registry, @Nullable Element sourceElement) {
		if (sourceElement != null) {
            // 对于 proxy-target-class 属性的处理
			boolean proxyTargetClass = Boolean.parseBoolean(sourceElement.getAttribute(PROXY_TARGET_CLASS_ATTRIBUTE));
			if (proxyTargetClass) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
            // 对于 expose-proxy 属性的处理
			boolean exposeProxy = Boolean.parseBoolean(sourceElement.getAttribute(EXPOSE_PROXY_ATTRIBUTE));
			if (exposeProxy) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
}
```

```java
// AopConfigUtils 类
public static void forceAutoProxyCreatorToUseClassProxying(BeanDefinitionRegistry registry) {
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			definition.getPropertyValues().add("proxyTargetClass", Boolean.TRUE);
		}
}

public static void forceAutoProxyCreatorToExposeProxy(BeanDefinitionRegistry registry) {
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			definition.getPropertyValues().add("exposeProxy", Boolean.TRUE);
		}
}
```

​		`JDK`动态代理：其代理对象必须是某个接口的实现，它是通过在运行期间创建一个接口的实现类来完成对目标对象的代理。

​		`CGLIB`代理：实现原理类似于`JDK`动态代理，只是它在运行期间生成的代理对象是针对目标类扩展的子类。`CGLIB`是高效的代码生成包，底层是依靠`ASM`操

作字节码实现的，性能比`JDK`强。

​		有时候目标对象内部的自我调用将无法实施切面中的增强，实施，为了解决这个问题，可以配置`<aop:aspectj-autoproxy expose-proxy="true" />`然后将代

码修改为`“((T)AopContext.currentProxy().xxx();`即可。通过以上的修改便可以完成对多个方法的同时增强。



#### 创建 AOP

![](image/QQ截图20220312101214.png)

```java
// AbstractAutoProxyCreator 类
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
		if (bean != null) {
            // 根据给定的 bean 的 class 和 name 构建出一个 key，格式: beanClassName_beanName
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
                // 如果它适合被代理,则需要封装指定 bean
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    	// 如果已经处理过
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
    	// 无须增强
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
	    // 给定的 bean 类是否代表一个基础设施类，基础设施类不应代理,或者配置了指定 bean 不需要自动代理
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// Create proxy if we have advice.
	    // 如果存在增强方法则创建代理
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
	    // 如果获取到了增强则需要针对增强创建代理
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
            // 创建代理
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
}
```



##### 获取增强器

```java
// AbstractAdvisorAutoProxyCreator 类
protected Object[] getAdvicesAndAdvisorsForBean(
			Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {

		List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
		if (advisors.isEmpty()) {
			return DO_NOT_PROXY; // DO_NOT_PROXY = null
		}
		return advisors.toArray();
}

protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
		List<Advisor> candidateAdvisors = findCandidateAdvisors(); // 获取所有增强
		List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName); // 寻找适用于 bean 的增强
		extendAdvisors(eligibleAdvisors);
		if (!eligibleAdvisors.isEmpty()) {
			eligibleAdvisors = sortAdvisors(eligibleAdvisors);
		}
		return eligibleAdvisors;
}
```

​		

​		基于注解的`AOP`：

```java
// AnnotationAwareAspectJAutoProxyCreator 类
protected List<Advisor> findCandidateAdvisors() {
		// Add all the Spring advisors found according to superclass rules.
	    // 当使用注解方式配置 AOP 的时候并不是丢弃了对 XML 配置的支持，
    	// 在这里调用父类方法加载配置文件中的 AOP 声明
		List<Advisor> advisors = super.findCandidateAdvisors();
		// Build Advisors for all AspectJ aspects in the bean factory.
		if (this.aspectJAdvisorsBuilder != null) {
			advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
		}
		return advisors;
}
```

```java
// BeanFactoryAspectJAdvisorsBuilder 类
public List<Advisor> buildAspectJAdvisors() {
		List<String> aspectNames = this.aspectBeanNames;

		if (aspectNames == null) {
			synchronized (this) {
				aspectNames = this.aspectBeanNames;
				if (aspectNames == null) {
					List<Advisor> advisors = new ArrayList<>();
					aspectNames = new ArrayList<>();
                    // 获取所有的 beanName
					String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
							this.beanFactory, Object.class, true, false);
                    // 循环所有的 beanName 找出对应的增强方法
					for (String beanName : beanNames) {
                        // 不合法的 bean 则略过，由子类定义规则，默认返回 true
						if (!isEligibleBean(beanName)) {
							continue;
						}
						// We must be careful not to instantiate beans eagerly as in this case they
						// would be cached by the Spring container but would not have been weaved.
						Class<?> beanType = this.beanFactory.getType(beanName, false);
                        // 获取对应的 bean 的类型
						if (beanType == null) {
							continue;
						}
                        // 如果存在 Aspect 注解
						if (this.advisorFactory.isAspect(beanType)) {
							aspectNames.add(beanName);
							AspectMetadata amd = new AspectMetadata(beanType, beanName);
							if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
								MetadataAwareAspectInstanceFactory factory =
										new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                                // 解析标记 AspectJ 注解中的增强方法
								List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
								if (this.beanFactory.isSingleton(beanName)) {
									this.advisorsCache.put(beanName, classAdvisors);
								}
								else {
									this.aspectFactoryCache.put(beanName, factory);
								}
								advisors.addAll(classAdvisors);
							}
							else {
								// Per target or per this.
								if (this.beanFactory.isSingleton(beanName)) {
									throw new IllegalArgumentException("Bean with name '" + beanName +
											"' is a singleton, but aspect instantiation model is not singleton");
								}
								MetadataAwareAspectInstanceFactory factory =
										new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
								this.aspectFactoryCache.put(beanName, factory);
								advisors.addAll(this.advisorFactory.getAdvisors(factory));
							}
						}
					}
					this.aspectBeanNames = aspectNames;
					return advisors;
				}
			}
		}

		if (aspectNames.isEmpty()) {
			return Collections.emptyList();
		}
	    // 记录在缓存中
		List<Advisor> advisors = new ArrayList<>();
		for (String aspectName : aspectNames) {
			List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
			if (cachedAdvisors != null) {
				advisors.addAll(cachedAdvisors);
			}
			else {
				MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
				advisors.addAll(this.advisorFactory.getAdvisors(factory));
			}
		}
		return advisors;
}
```

```java
// ReflectiveAspectJAdvisorFactory 类
public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
    	// 获取标记为 AspectJ 的类
		Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
	    // 获取标记为 AspectJ 的 name
		String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
	    // 验证
		validate(aspectClass);

		// We need to wrap the MetadataAwareAspectInstanceFactory with a decorator
		// so that it will only instantiate once.
		MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
				new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

		List<Advisor> advisors = new ArrayList<>();
    	// 声明为 Pointcut 的方法不处理
		for (Method method : getAdvisorMethods(aspectClass)) {
            
			Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, 0, aspectName);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		// If it's a per target aspect, emit the dummy instantiating aspect.
	    // 如果寻找的增强器不为空而且又配置了增强延迟初始化，那么需要在首位加入同步实例化增强器
		if (!advisors.isEmpty() && lazySingletonAspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			Advisor instantiationAdvisor = new SyntheticInstantiationAdvisor(lazySingletonAspectInstanceFactory);
			advisors.add(0, instantiationAdvisor);
		}

		// Find introduction fields.
	    // 获取 DeclareParents 注解
		for (Field field : aspectClass.getDeclaredFields()) {
			Advisor advisor = getDeclareParentsAdvisor(field);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		return advisors;
}
```



###### 普通增强器的获取

```java
// ReflectiveAspectJAdvisorFactory 类
public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory,
			int declarationOrderInAspect, String aspectName) {

		validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());
		// 切点信息的获取
		AspectJExpressionPointcut expressionPointcut = getPointcut(
				candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
		if (expressionPointcut == null) {
			return null;
		}
		// 根据切点信息生成增强器
		return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
				this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}


private AspectJExpressionPointcut getPointcut(Method candidateAdviceMethod, Class<?> candidateAspectClass) {
	    // 获取方法上的注解
		AspectJAnnotation<?> aspectJAnnotation =
				AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
		if (aspectJAnnotation == null) {
			return null;
		}
    	// 使用 AspectJExpressionPointcut 实例封装获取的信息
		AspectJExpressionPointcut ajexp =
				new AspectJExpressionPointcut(candidateAspectClass, new String[0], new Class<?>[0]);
	    // 提取得到的注解中的表达式
		ajexp.setExpression(aspectJAnnotation.getPointcutExpression());
		if (this.beanFactory != null) {
			ajexp.setBeanFactory(this.beanFactory);
		}
		return ajexp;
}
```

```java
// AbstractAspectJAdvisorFactory 类
protected static AspectJAnnotation<?> findAspectJAnnotationOnMethod(Method method) {
    	// 设置敏感的注解类
		for (Class<?> clazz : ASPECTJ_ANNOTATION_CLASSES) {
			AspectJAnnotation<?> foundAnnotation = findAnnotation(method, (Class<Annotation>) clazz);
			if (foundAnnotation != null) {
				return foundAnnotation;
			}
		}
		return null;
}
// 获取指定方法上的注解并使用 AspectJAnnotation 封装
private static <A extends Annotation> AspectJAnnotation<A> findAnnotation(Method method, Class<A> toLookFor) {
		A result = AnnotationUtils.findAnnotation(method, toLookFor);
		if (result != null) {
			return new AspectJAnnotation<>(result);
		}
		else {
			return null;
		}
}
```



###### 根据切点信息生成增强

​		所有的增强都由`Advisor`的实现类`InstantiationModelAwarePointcutAdvisorImpl`统一封装。

```java
public InstantiationModelAwarePointcutAdvisorImpl(AspectJExpressionPointcut declaredPointcut,
			Method aspectJAdviceMethod, AspectJAdvisorFactory aspectJAdvisorFactory,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

		this.declaredPointcut = declaredPointcut;
		this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
		this.methodName = aspectJAdviceMethod.getName();
		this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
		this.aspectJAdviceMethod = aspectJAdviceMethod;
		this.aspectJAdvisorFactory = aspectJAdvisorFactory;
		this.aspectInstanceFactory = aspectInstanceFactory;
		this.declarationOrder = declarationOrder;
		this.aspectName = aspectName;

		if (aspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			// Static part of the pointcut is a lazy type.
			Pointcut preInstantiationPointcut = Pointcuts.union(
					aspectInstanceFactory.getAspectMetadata().getPerClausePointcut(), this.declaredPointcut);

			// Make it dynamic: must mutate from pre-instantiation to post-instantiation state.
			// If it's not a dynamic pointcut, it may be optimized out
			// by the Spring AOP infrastructure after the first evaluation.
			this.pointcut = new PerTargetInstantiationModelPointcut(
					this.declaredPointcut, preInstantiationPointcut, aspectInstanceFactory);
			this.lazy = true;
		}
		else {
			// A singleton aspect.
			this.pointcut = this.declaredPointcut;
			this.lazy = false;
			this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
		}
}
```

​		在封装过程中只是简单地将信息封装在类的实例中，所有的信息单纯地赋值，在实例初始化的过程中还完成了对于增强器的初始化。因为不同的增强所体现的

逻辑是不同的，所以就需要不同的增强器来完成不同的逻辑，而根据注解中的信息初始化对应的增强器就是在`instantiateAdvice`函数中实现的。

```java
// InstantiationModelAwarePointcutAdvisorImpl 类
private Advice instantiateAdvice(AspectJExpressionPointcut pointcut) {
		Advice advice = this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pointcut,
				this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
		return (advice != null ? advice : EMPTY_ADVICE);
}
```

```java
// ReflectiveAspectJAdvisorFactory 类
public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

		Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
		validate(candidateAspectClass);

		AspectJAnnotation<?> aspectJAnnotation =
				AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
		if (aspectJAnnotation == null) {
			return null;
		}

		// If we get here, we know we have an AspectJ method.
		// Check that it's an AspectJ-annotated class
		if (!isAspect(candidateAspectClass)) {
			throw new AopConfigException("Advice must be declared inside an aspect type: " +
					"Offending method '" + candidateAdviceMethod + "' in class [" +
					candidateAspectClass.getName() + "]");
		}

		if (logger.isDebugEnabled()) {
			logger.debug("Found AspectJ method: " + candidateAdviceMethod);
		}

		AbstractAspectJAdvice springAdvice;
		// 根据不同的注解类型封装不同的增强器
		switch (aspectJAnnotation.getAnnotationType()) {
			case AtPointcut:
				if (logger.isDebugEnabled()) {
					logger.debug("Processing pointcut '" + candidateAdviceMethod.getName() + "'");
				}
				return null;
			case AtAround:
				springAdvice = new AspectJAroundAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtBefore:
				springAdvice = new AspectJMethodBeforeAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtAfter:
				springAdvice = new AspectJAfterAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtAfterReturning:
				springAdvice = new AspectJAfterReturningAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
				if (StringUtils.hasText(afterReturningAnnotation.returning())) {
					springAdvice.setReturningName(afterReturningAnnotation.returning());
				}
				break;
			case AtAfterThrowing:
				springAdvice = new AspectJAfterThrowingAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
				if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
					springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
				}
				break;
			default:
				throw new UnsupportedOperationException(
						"Unsupported advice type on method: " + candidateAdviceMethod);
		}

		// Now to configure the advice...
		springAdvice.setAspectName(aspectName);
		springAdvice.setDeclarationOrder(declarationOrder);
		String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
		if (argNames != null) {
			springAdvice.setArgumentNamesFromStringArray(argNames);
		}
		springAdvice.calculateArgumentBindings();

		return springAdvice;
}
```



​		常用增强器实现：

`MethodBeforeAdviceInterceptor`：

```java
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {

   private final MethodBeforeAdvice advice; // AspectJMethodBeforeAdvice


   /**
    * Create a new MethodBeforeAdviceInterceptor for the given advice.
    * @param advice the MethodBeforeAdvice to wrap
    */
   public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
      Assert.notNull(advice, "Advice must not be null");
      this.advice = advice;
   }


   @Override
   @Nullable
   public Object invoke(MethodInvocation mi) throws Throwable {
      this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
      return mi.proceed();
   }

}
```

```java
// AspectJMethodBeforeAdvice 类
public void before(Method method, Object[] args, @Nullable Object target) throws Throwable {
		invokeAdviceMethod(getJoinPointMatch(), null, null);
}
```

```java
// AbstractAspectJAdvice 类
protected Object invokeAdviceMethod(
			@Nullable JoinPointMatch jpMatch, @Nullable Object returnValue, @Nullable Throwable ex)
			throws Throwable {

		return invokeAdviceMethodWithGivenArgs(argBinding(getJoinPoint(), jpMatch, returnValue, ex));
}
protected Object invokeAdviceMethodWithGivenArgs(Object[] args) throws Throwable {
		Object[] actualArgs = args;
		if (this.aspectJAdviceMethod.getParameterCount() == 0) {
			actualArgs = null;
		}
		try {
			ReflectionUtils.makeAccessible(this.aspectJAdviceMethod);
            // 激活增强方法
			return this.aspectJAdviceMethod.invoke(this.aspectInstanceFactory.getAspectInstance(), actualArgs);
		}
		catch (IllegalArgumentException ex) {
			throw new AopInvocationException("Mismatch on arguments to advice method [" +
					this.aspectJAdviceMethod + "]; pointcut expression [" +
					this.pointcut.getPointcutExpression() + "]", ex);
		}
		catch (InvocationTargetException ex) {
			throw ex.getTargetException();
		}
}
```



`AspectJAfterAdvice`：

```java
public class AspectJAfterAdvice extends AbstractAspectJAdvice
		implements MethodInterceptor, AfterAdvice, Serializable {

	public AspectJAfterAdvice(
			Method aspectJBeforeAdviceMethod, AspectJExpressionPointcut pointcut, AspectInstanceFactory aif) {

		super(aspectJBeforeAdviceMethod, pointcut, aif);
	}


	@Override
	@Nullable
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		finally {
			invokeAdviceMethod(getJoinPointMatch(), null, null);
		}
	}

	@Override
	public boolean isBeforeAdvice() {
		return false;
	}

	@Override
	public boolean isAfterAdvice() {
		return true;
	}

}
```



###### 增加同步实例化增强器

​		如果寻找的增强器不为空而且又配置了增强延迟初始化,那么就需要在首位加入同步实例化增强器：

```java
protected static class SyntheticInstantiationAdvisor extends DefaultPointcutAdvisor {

		public SyntheticInstantiationAdvisor(final MetadataAwareAspectInstanceFactory aif) {
            // 目标方法前调用，类似 @Before
			super(aif.getAspectMetadata().getPerClausePointcut(), (MethodBeforeAdvice)
                  	// 简单初始化 aspect
					(method, args, target) -> aif.getAspectInstance());
		}
}
```



###### 获取 DeclareParents

​		`DeclareParents`主要用于引介增强的注解形式的实现，而其实现方式与普通增强很类似，只不过使用`DeclareParentsAdvisor`对功能进行封装。

```java
// ReflectiveAspectJAdvisorFactory 类
private Advisor getDeclareParentsAdvisor(Field introductionField) {
		DeclareParents declareParents = introductionField.getAnnotation(DeclareParents.class);
		if (declareParents == null) {
			// Not an introduction field
			return null;
		}

		if (DeclareParents.class == declareParents.defaultImpl()) {
			throw new IllegalStateException("'defaultImpl' attribute must be set on DeclareParents");
		}

		return new DeclareParentsAdvisor(
				introductionField.getType(), declareParents.value(), declareParents.defaultImpl());
}
```



##### 寻找匹配的增强器

```java
// AbstractAdvisorAutoProxyCreator 类
protected List<Advisor> findAdvisorsThatCanApply(
			List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

		ProxyCreationContext.setCurrentProxiedBeanName(beanName);
		try {
            // 过滤已经得到的 advisors
			return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
		}
		finally {
			ProxyCreationContext.setCurrentProxiedBeanName(null);
		}
}
```

```java
// AopUtils 类
public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
		if (candidateAdvisors.isEmpty()) {
			return candidateAdvisors;
		}
		List<Advisor> eligibleAdvisors = new ArrayList<>();
    	// 首先处理引介增强
		for (Advisor candidate : candidateAdvisors) {
			if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
				eligibleAdvisors.add(candidate);
			}
		}
		boolean hasIntroductions = !eligibleAdvisors.isEmpty();
		for (Advisor candidate : candidateAdvisors) {
            // 引介增强已经处理
			if (candidate instanceof IntroductionAdvisor) {
				// already processed
				continue;
			}
            // 对于普通 bean 的处理
			if (canApply(candidate, clazz, hasIntroductions)) {
				eligibleAdvisors.add(candidate);
			}
		}
		return eligibleAdvisors;
}

public static boolean canApply(Advisor advisor, Class<?> targetClass) {
		return canApply(advisor, targetClass, false);
}

public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
		if (advisor instanceof IntroductionAdvisor) {
			return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
		}
		else if (advisor instanceof PointcutAdvisor) {
			PointcutAdvisor pca = (PointcutAdvisor) advisor;
			return canApply(pca.getPointcut(), targetClass, hasIntroductions);
		}
		else {
			// It doesn't have a pointcut so we assume it applies.
			return true;
		}
}

public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
		Assert.notNull(pc, "Pointcut must not be null");
		if (!pc.getClassFilter().matches(targetClass)) {
			return false;
		}

		MethodMatcher methodMatcher = pc.getMethodMatcher();
		if (methodMatcher == MethodMatcher.TRUE) {
			// No need to iterate the methods if we're matching any method anyway...
			return true;
		}

		IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
		if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
			introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
		}

		Set<Class<?>> classes = new LinkedHashSet<>();
		if (!Proxy.isProxyClass(targetClass)) {
			classes.add(ClassUtils.getUserClass(targetClass));
		}
		classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass));

		for (Class<?> clazz : classes) {
			Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
			for (Method method : methods) {
				if (introductionAwareMethodMatcher != null ?
						introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
						methodMatcher.matches(method, targetClass)) {
					return true;
				}
			}
		}

		return false;
}
```



##### 创建代理

```java
// AbstractAutoProxyCreator 类
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
		}

		ProxyFactory proxyFactory = new ProxyFactory();
	    // 获取当前类中相关属性
		proxyFactory.copyFrom(this);
		// 决定对于给定的 bean 是否应该使用 targetclass 而不是它的接口代理
    	// 检查 proxyTargeclass 设置以及 preserveTargetclass 属性

		if (proxyFactory.isProxyTargetClass()) {
			// Explicit handling of JDK proxy targets (for introduction advice scenarios)
			if (Proxy.isProxyClass(beanClass)) {
				// Must allow for introductions; can't just set interfaces to the proxy's interfaces only.
				for (Class<?> ifc : beanClass.getInterfaces()) {
                    // 添加代理接口
					proxyFactory.addInterface(ifc);
				}
			}
		}
		else {
			// No proxyTargetClass flag enforced, let's apply our default checks...
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				evaluateProxyInterfaces(beanClass, proxyFactory);
			}
		}

		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    	// 加人增强器
		proxyFactory.addAdvisors(advisors);
	    // 设置要代理的类
		proxyFactory.setTargetSource(targetSource);
	    // 定制代理
		customizeProxyFactory(proxyFactory);
		// 用来控制代理工厂被配置之后，是否还允许修改通知
		// 缺省值为 false（即在代理被配置之后，不允许修改代理的配置)
		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}

		// Use original ClassLoader if bean class not locally loaded in overriding class loader
		ClassLoader classLoader = getProxyClassLoader();
		if (classLoader instanceof SmartClassLoader && classLoader != beanClass.getClassLoader()) {
			classLoader = ((SmartClassLoader) classLoader).getOriginalClassLoader();
		}
		return proxyFactory.getProxy(classLoader);
}

// 将拦截器封装为增强器
protected Advisor[] buildAdvisors(@Nullable String beanName, @Nullable Object[] specificInterceptors) {
		// Handle prototypes correctly...
    	// 解析注册的所有 interceptorName
		Advisor[] commonInterceptors = resolveInterceptorNames();

		List<Object> allInterceptors = new ArrayList<>();
		if (specificInterceptors != null) {
			if (specificInterceptors.length > 0) {
				// specificInterceptors may equal PROXY_WITHOUT_ADDITIONAL_INTERCEPTORS
                // 加入拦截器
				allInterceptors.addAll(Arrays.asList(specificInterceptors));
			}
			if (commonInterceptors.length > 0) {
				if (this.applyCommonInterceptorsFirst) {
					allInterceptors.addAll(0, Arrays.asList(commonInterceptors));
				}
				else {
					allInterceptors.addAll(Arrays.asList(commonInterceptors));
				}
			}
		}
		if (logger.isTraceEnabled()) {
			int nrOfCommonInterceptors = commonInterceptors.length;
			int nrOfSpecificInterceptors = (specificInterceptors != null ? specificInterceptors.length : 0);
			logger.trace("Creating implicit proxy for bean '" + beanName + "' with " + nrOfCommonInterceptors +
					" common interceptors and " + nrOfSpecificInterceptors + " specific interceptors");
		}

		Advisor[] advisors = new Advisor[allInterceptors.size()];
		for (int i = 0; i < allInterceptors.size(); i++) {
            // 拦截器进行封装转化为 Advisor
			advisors[i] = this.advisorAdapterRegistry.wrap(allInterceptors.get(i));
		}
		return advisors;
}
```

```java
// DefaultAdvisorAdapterRegistry 类
public Advisor wrap(Object adviceObject) throws UnknownAdviceTypeException {
 	   // 如果要封装的对象本身就是 Advisor 类型的，那么无须再做过多处理
		if (adviceObject instanceof Advisor) {
			return (Advisor) adviceObject;
		}
	    // 因为此封装方法只对 Advisor 与 Advice 两种类型的数据有效，如果不是将不能封装
		if (!(adviceObject instanceof Advice)) {
			throw new UnknownAdviceTypeException(adviceObject);
		}
		Advice advice = (Advice) adviceObject;
		if (advice instanceof MethodInterceptor) {
			// So well-known it doesn't even need an adapter.
            // 如果是 MethodInterceptor 类型则使用 DefaultPointcutAdvisor 封装
			return new DefaultPointcutAdvisor(advice);
		}
	    // 如果存在 Advisor 的适配器那么也同样需要进行封装
		for (AdvisorAdapter adapter : this.adapters) {
			// Check that it is supported.
			if (adapter.supportsAdvice(advice)) {
				return new DefaultPointcutAdvisor(advice);
			}
		}
		throw new UnknownAdviceTypeException(advice);
}
```

​		完成了增强的封装，接下来就是代理的创建与获取：

```java
// ProxyFactory 类
public Object getProxy(@Nullable ClassLoader classLoader) {
		return createAopProxy().getProxy(classLoader);
}
```

​		创建代理：

```java
// ProxyCreatorSupport 类
protected final synchronized AopProxy createAopProxy() {
		if (!this.active) {
			activate();
		}
		return getAopProxyFactory().createAopProxy(this);
}
```

```java
// DefaultAopProxyFactory 类
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    	/**
    	optimize:用来控制通过CGLIB创建的代理是否使用激进的优化策略。除非完全了解AOP代理如何处理优化，否则不推荐用户使用这个设置。目前这个属性仅用于CGLIB代理，对于JDK动态代理（默认代理）无效
		proxyTargetClass:这个属性为true时，目标类本身被代理而不是目标类的接口。如果这个属性值被设为 true，CGLIB代理将被创建，设置方式为<aop:aspectj-autoproxy-proxy-target-class="true"/>
		hasNoUserSuppliedProxyInterfaces:是否存在代理接口
		
		如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP。
		如果目标对象实现了接口，可以强制使用CGLIB实现AOP。
		如果目标对象没有实现接口，必须采用CGLIB库，Spring 会自动在JDK动态代理和CGLIB之间转换。
		
		JDK动态代理只能对实现了接口的类生成代理，而不能针对类。
		CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以该类或方法最好不要声明成final。
    	*/
		if (!NativeDetector.inNativeImage() &&
				(config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config))) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass) || AopProxyUtils.isLambda(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
}
```



​		获取代理：

​		`JDK`动态代理：

```java
// JdkDynamicAopProxy 类
public Object getProxy(@Nullable ClassLoader classLoader) {
		if (logger.isTraceEnabled()) {
			logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
		}
		return Proxy.newProxyInstance(classLoader, this.proxiedInterfaces, this);
}

public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Object target = null;

		try {
            // equals 方法的处理
			if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
				// The target does not implement the equals(Object) method itself.
				return equals(args[0]);
			}
            // hash 方法的处理
			else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
				// The target does not implement the hashCode() method itself.
				return hashCode();
			}
			else if (method.getDeclaringClass() == DecoratingProxy.class) {
				// There is only getDecoratedClass() declared -> dispatch to proxy config.
				return AopProxyUtils.ultimateTargetClass(this.advised);
			}
            /*
				Class类的isAssignableFrom (class cls)方法:
				如果调用这个方法的class或接口与参数cls表示的类或接口相同，*或者是参数cls表示的类或接口的父类，则返回true。
				形象地:自身类.class.isAssignableFrom(自身类或子类.class) 返回true
				例:
					System.out.println(ArrayList.class.isAssignableFrom(Object.class) ) ;
						// false
					System.out.println (Object.class.isAssignableFrom(ArrayList.class));
						// true
			*/
			else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
					method.getDeclaringClass().isAssignableFrom(Advised.class)) {
				// Service invocations on ProxyConfig with the proxy config...
				return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
			}

			Object retVal;
			// 有时候目标对象内部的自我调用将无法实施切面中的增强则需要通过此属性暴露代理
			if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

			// Get as late as possible to minimize the time we "own" the target,
			// in case it comes from a pool.
			target = targetSource.getTarget();
			Class<?> targetClass = (target != null ? target.getClass() : null);

			// Get the interception chain for this method.
            // 获取当前方法的拦截器链
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			// Check whether we have any advice. If we don't, we can fallback on direct
			// reflective invocation of the target, and avoid creating a MethodInvocation.
			if (chain.isEmpty()) {
				// We can skip creating a MethodInvocation: just invoke the target directly
				// Note that the final invoker must be an InvokerInterceptor so we know it does
				// nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
                // 如果没有发现任何拦截器那么直接调用切点方法
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// We need to create a method invocation...
                // 将拦截器封装在 ReflectiveMethodInvocation
                // 以便于使用其 proceed 进行链接表用拦截器
				MethodInvocation invocation =
						new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				// Proceed to the joinpoint through the interceptor chain.
                // 执行拦截器链
				retVal = invocation.proceed();
			}

			// Massage return value if necessary.
			Class<?> returnType = method.getReturnType();
            // 返回结果
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
					!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
				// Special case: it returned "this" and the return type of the method
				// is type-compatible. Note that we can't help if the target sets
				// a reference to itself in another returned object.
				retVal = proxy;
			}
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				// Must have come from TargetSource.
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
}
```

```java
// ReflectiveMethodInvocation 类
/**
	ReflectiveMethodInvocation 封装类拦截器链，而 proceed 方法实现了拦截器的逐一调用
*/
public Object proceed() throws Throwable {
		// We start with an index of -1 and increment early.
		// 执行完所有增强后执行切点方法
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}
		// 获取下一个要执行的拦截器
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
            // 动态匹配
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
			if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
                // 不匹配则不执行拦截器
				return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
            /*
            	普通拦截器，直接调用拦截器，比如:
            		ExposeInvocationInterceptor 、
				   DelegatePerTargetobjectIntroductionInterceptor、
				   MethodBeforeAdviceInterceptor
				   AspectJAroundAdvice 、
				   AspectJAfterAdvice
			*/
			//将 this 作为参数传递以保证当前实例中调用链的执行
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
}
```



​		`CGLIB`动态代理：

​		`CGLIB`中对于方法的拦截是通过将自定义的拦截器`(`实现`MethodInterceptor`接口`)`加入`Calback`中并在调用代理时直接激活拦截器中的`intercept`方法来实

现的。

```java
// CglibAopProxy 类
public Object getProxy(@Nullable ClassLoader classLoader) {
		if (logger.isTraceEnabled()) {
			logger.trace("Creating CGLIB proxy: " + this.advised.getTargetSource());
		}

		try {
			Class<?> rootClass = this.advised.getTargetClass();
			Assert.state(rootClass != null, "Target class must be available for creating a CGLIB proxy");

			Class<?> proxySuperClass = rootClass;
			if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
				proxySuperClass = rootClass.getSuperclass();
				Class<?>[] additionalInterfaces = rootClass.getInterfaces();
				for (Class<?> additionalInterface : additionalInterfaces) {
					this.advised.addInterface(additionalInterface);
				}
			}

			// Validate the class, writing log messages as necessary.
            // 验证 class
			validateClassIfNecessary(proxySuperClass, classLoader);

			// Configure CGLIB Enhancer...
            // 创建及配置 Enhancer
			Enhancer enhancer = createEnhancer();           
			if (classLoader != null) {
				enhancer.setClassLoader(classLoader);
				if (classLoader instanceof SmartClassLoader &&
						((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
					enhancer.setUseCache(false);
				}
			}
			enhancer.setSuperclass(proxySuperClass);
			enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
			enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
			enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));
			// 设置拦截器
			Callback[] callbacks = getCallbacks(rootClass);
			Class<?>[] types = new Class<?>[callbacks.length];
			for (int x = 0; x < types.length; x++) {
				types[x] = callbacks[x].getClass();
			}
			// fixedInterceptorMap only populated at this point, after getCallbacks call above
			enhancer.setCallbackFilter(new ProxyCallbackFilter(
					this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
			enhancer.setCallbackTypes(types);

			// Generate the proxy class and create a proxy instance.
            // 生成代理类以及创建代理
			return createProxyClassAndInstance(enhancer, callbacks);
		}
		catch (CodeGenerationException | IllegalArgumentException ex) {
			throw new AopConfigException("Could not generate CGLIB subclass of " + this.advised.getTargetClass() +
					": Common causes of this problem include using a final class or a non-visible class",
					ex);
		}
		catch (Throwable ex) {
			// TargetSource.getTarget() failed
			throw new AopConfigException("Unexpected AOP exception", ex);
		}
}

private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
		// Parameters used for optimization choices...
	    // 对于 expose-proxy 属性的处理
		boolean exposeProxy = this.advised.isExposeProxy();
		boolean isFrozen = this.advised.isFrozen();
		boolean isStatic = this.advised.getTargetSource().isStatic();

		// Choose an "aop" interceptor (used for AOP calls).
	    // 将拦截器封装在 DynamicAdvisedInterceptor 中
		Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);

		// Choose a "straight to target" interceptor. (used for calls that are
		// unadvised but can return this). May be required to expose the proxy.
		Callback targetInterceptor;
		if (exposeProxy) {
			targetInterceptor = (isStatic ?
					new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
					new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource()));
		}
		else {
			targetInterceptor = (isStatic ?
					new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
					new DynamicUnadvisedInterceptor(this.advised.getTargetSource()));
		}

		// Choose a "direct to target" dispatcher (used for
		// unadvised calls to static targets that cannot return this).
		Callback targetDispatcher = (isStatic ?
				new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());

		Callback[] mainCallbacks = new Callback[] {
	            // 将拦截器链加人 Callback 中
				aopInterceptor,  // for normal advice
				targetInterceptor,  // invoke target without considering advice, if optimized
				new SerializableNoOp(),  // no override for methods mapped to this
				targetDispatcher, this.advisedDispatcher,
				new EqualsInterceptor(this.advised),
				new HashCodeInterceptor(this.advised)
		};

		Callback[] callbacks;

		// If the target is a static one and the advice chain is frozen,
		// then we can make some optimizations by sending the AOP calls
		// direct to the target using the fixed chain for that method.
		if (isStatic && isFrozen) {
			Method[] methods = rootClass.getMethods();
			Callback[] fixedCallbacks = new Callback[methods.length];
			this.fixedInterceptorMap = CollectionUtils.newHashMap(methods.length);

			// TODO: small memory optimization here (can skip creation for methods with no advice)
			for (int x = 0; x < methods.length; x++) {
				Method method = methods[x];
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, rootClass);
				fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
						chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
				this.fixedInterceptorMap.put(method, x);
			}

			// Now copy both the callbacks from mainCallbacks
			// and fixedCallbacks into the callbacks array.
			callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
			System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
			System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
			this.fixedInterceptorOffset = mainCallbacks.length;
		}
		else {
			callbacks = mainCallbacks;
		}
		return callbacks;
}


protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
		enhancer.setInterceptDuringConstruction(false);
		enhancer.setCallbacks(callbacks);
		return (this.constructorArgs != null && this.constructorArgTypes != null ?
				enhancer.create(this.constructorArgTypes, this.constructorArgs) :
				enhancer.create());
}
```

```java
// CglibAopProxy#DynamicAdvisedInterceptor 类
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
			Object oldProxy = null;
			boolean setProxyContext = false;
			Object target = null;
			TargetSource targetSource = this.advised.getTargetSource();
			try {
				if (this.advised.exposeProxy) {
					// Make invocation available if necessary.
					oldProxy = AopContext.setCurrentProxy(proxy);
					setProxyContext = true;
				}
				// Get as late as possible to minimize the time we "own" the target, in case it comes from a pool...
				target = targetSource.getTarget();
				Class<?> targetClass = (target != null ? target.getClass() : null);
                // 获取拦截器链
				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
				Object retVal;
				// Check whether we only have one InvokerInterceptor: that is,
				// no real advice, but just reflective invocation of the target.
				if (chain.isEmpty() && CglibMethodInvocation.isMethodProxyCompatible(method)) {
					// We can skip creating a MethodInvocation: just invoke the target directly.
					// Note that the final invoker must be an InvokerInterceptor, so we know
					// it does nothing but a reflective operation on the target, and no hot
					// swapping or fancy proxying.
					Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
					try {
                        // 如果拦截器链为空则直接激活原方法
						retVal = methodProxy.invoke(target, argsToUse);
					}
					catch (CodeGenerationException ex) {
						CglibMethodInvocation.logFastClassGenerationFailure(method);
						retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
					}
				}
				else {
					// We need to create a method invocation...
                    // 进入链
					retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
				}
				retVal = processReturnType(proxy, target, method, retVal);
				return retVal;
			}
			finally {
				if (target != null && !targetSource.isStatic()) {
					targetSource.releaseTarget(target);
				}
				if (setProxyContext) {
					// Restore old proxy.
					AopContext.setCurrentProxy(oldProxy);
				}
			}
}
```



#### 静态 AOP

​		加载时织入`(LTW)`指的是在虚拟机载入字节码文件时动态织入`AspectJ`切面。

​		`AOP`的静态代理主要是在虚拟机启动时通过改变目标对象字节码的方式来完成对目标对象的增强，它与动态代理相比具有更高的效率，因为在动态代理调用

的过程中，还需要一个动态创建代理类并代理目标对象的步骤，而静态代理则是在启动时便完成了字节码增强，当系统再次调用目标类时与调用正常的类并无差

别，所以在效率上会相对高些。

​		在`Spring`中的静态`AOP`直接使用了`AspectJ`提供的方法，而`AspectJ`又是在`Instrument`基础上进行的封装。至少在`AspectJ`中会有如下功能：

​				读取`META-INF/aop.xml`

​				将`aop.xml`中定义的增强器通过自定义的`ClassFileTransformer`织入对应的类中

​		在`Spring`中如果需要使用`AspectJ`的功能，首先要做的第一步就是在配置文件中加入配置`<context:load-time-weaver />`：

```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {
	@Override
	public void init() {
		registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
		registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
		registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
		registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
	}
}
```

```java
// LoadTimeWeaverBeanDefinitionParser 类
protected void doParse(Element element, ParserContext parserContext, BeanDefinitionBuilder builder) {
		builder.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);

		if (isAspectJWeavingEnabled(element.getAttribute(ASPECTJ_WEAVING_ATTRIBUTE), parserContext)) {
			if (!parserContext.getRegistry().containsBeanDefinition(ASPECTJ_WEAVING_ENABLER_BEAN_NAME)) {
            // 注册一个对于 ApectJ 处理的类：org.springframework.context.weaving.AspectJWeavingEnabler
                RootBeanDefinition def = new RootBeanDefinition(ASPECTJ_WEAVING_ENABLER_CLASS_NAME);
				parserContext.registerBeanComponent(
						new BeanComponentDefinition(def, ASPECTJ_WEAVING_ENABLER_BEAN_NAME));
			}

			if (isBeanConfigurerAspectEnabled(parserContext.getReaderContext().getBeanClassLoader())) {
				new SpringConfiguredBeanDefinitionParser().parse(element, parserContext);
			}
		}
}

/**
	在配置文件中加入了 <context:load-time-weaver /> 便相当于加入了 AspectJ 开关。但是，并不是配置了这个标签就意味着开启了 AspectJ 功能，这个标签中还有一个属性 aspectj-weaving，这个属性有 3 个备选值，on、off和 autodetect，默认为 autodetect，如果只是使用了 <context:load-time-weaver />，那么Spring会帮助我们检测是否可以使用 AspectJ 功能，而检测的依据便是文件 META-INF/aop.xml 是否存在
*/
protected boolean isAspectJWeavingEnabled(String value, ParserContext parserContext) {
		if ("on".equals(value)) {
			return true;
		}
		else if ("off".equals(value)) {
			return false;
		}
		else {
			// Determine default...
            // 自动检测
			ClassLoader cl = parserContext.getReaderContext().getBeanClassLoader();
			return (cl != null && cl.getResource(AspectJWeavingEnabler.ASPECTJ_AOP_XML_RESOURCE) != null);
		}
}
```

​		当`Spring`在读取到自定义标签`<context:load-time-weaver/>`后会产生一个`bean`，而这个`bean`的`id`为`loadTimeWeaver`，`class`为

`org.Springframework.context.weaving.DefaultContextLoadTimeWeaver`，也就是完成了`DefaultContextLoadTimeWeaver`类的注册。

​		完成了以上的注册功能后，还有一个很重要的步骤，就是`LoadTimeWeaverAwareProcessor`的注册：

```java
// AbstractApplicationContext 类 prepareBeanFactory 方法
if (!NativeDetector.inNativeImage() && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
}
```



##### 织入

![](image/QQ截图20220318125235.png)

```java
// LoadTimeWeaverAwareProcessor 类
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    	// 实现 LoadTimeWeaverAware 接口的类只有 AspectJWeavingEnabler
		if (bean instanceof LoadTimeWeaverAware) {
			LoadTimeWeaver ltw = this.loadTimeWeaver;
			if (ltw == null) {
				Assert.state(this.beanFactory != null,
						"BeanFactory required if no LoadTimeWeaver explicitly specified");
				ltw = this.beanFactory.getBean(
						ConfigurableApplicationContext.LOAD_TIME_WEAVER_BEAN_NAME, LoadTimeWeaver.class);
			}
			((LoadTimeWeaverAware) bean).setLoadTimeWeaver(ltw);
		}
		return bean;
}
```

​		当在`Spring`中调用`AspectJWeavingEnabler时`，`this.loadTimeWeaver`尚未被初始化，那么，会直接调用`beanFactory.getBean`方法获取对应的

`DefaultContextLoadTimeWeaver`类型的`bean`，并将其设置为`AspectJWeavingEnabler`类型`bean`的`loadTimeWeaver`属性中。当然`AspectJWeavingEnabler`同样实现

了`BeanClassLoaderAware`以及`Ordered`接口，实现`BeanClassLoaderAware`接口保证了在`bean`初始化的时候调用`AbstractAutowireCapableBeanFactory`的

`invokeAwareMethods`的时候将`beanClassLoader`赋值给当前类。而实现`Ordered`接口则保证在实例化`bean`时当前`bean`会被最先初始化。

​		`DefaultContextLoadTimeWeaver`类又同时实现了`LoadTimeWeaver`、`BeanClassLoaderAware`以及`DisposableBean`。其中`DisposableBean`接口保证在`bean`销毁

时会调用`destroy`方法进行`bean`的清理，而`BeanClassLoaderAware`接口则保证在`bean`的初始化调用`AbstractAutowireCapableBeanFactory`的

`invokeAwareMethods`时调用`setBeanClassLoader`方法。

```java
// DefaultContextLoadTimeWeaver 类
public void setBeanClassLoader(ClassLoader classLoader) {
		LoadTimeWeaver serverSpecificLoadTimeWeaver = createServerSpecificLoadTimeWeaver(classLoader);
		if (serverSpecificLoadTimeWeaver != null) {
			if (logger.isDebugEnabled()) {
				logger.debug("Determined server-specific load-time weaver: " +
						serverSpecificLoadTimeWeaver.getClass().getName());
			}
			this.loadTimeWeaver = serverSpecificLoadTimeWeaver;
		}
		else if (InstrumentationLoadTimeWeaver.isInstrumentationAvailable()) {
			logger.debug("Found Spring's JVM agent for instrumentation");
            // 检查当前虚拟机中的 Instrumentation 实例是否可用
			this.loadTimeWeaver = new InstrumentationLoadTimeWeaver(classLoader);
		}
		else {
			try {
				this.loadTimeWeaver = new ReflectiveLoadTimeWeaver(classLoader);
				if (logger.isDebugEnabled()) {
					logger.debug("Using reflective load-time weaver for class loader: " +
							this.loadTimeWeaver.getInstrumentableClassLoader().getClass().getName());
				}
			}
			catch (IllegalStateException ex) {
				throw new IllegalStateException(ex.getMessage() + " Specify a custom LoadTimeWeaver or start your " +
						"Java virtual machine with Spring's agent: -javaagent:spring-instrument-{version}.jar");
			}
		}
}
```

​		也就是经过以上程序的处理后，在`Spring`中的`bean`之间的关系如下：

​				`AspectJWeavingEnabler`类型的`bean`中的`loadTimeWeaver`属性被初始化为`DefaultContextLoadTimeWeaver`类型的`bean`。

​				`DefaultContextLoadTimeWeaver`类型的`bean`中的`loadTimeWeaver`属性被初始化为`InstrumentationLoadTimeWeaver`。

​		`AspectJWeavingEnabler`类同样实现了`BeanFactoryPostProcessor`，所以当所有`bean`解析结束后会调用其`postProcessBeanFactory`方法：

```java
// AspectJWeavingEnabler 类
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		enableAspectJWeaving(this.loadTimeWeaver, this.beanClassLoader);
}

public static void enableAspectJWeaving(
			@Nullable LoadTimeWeaver weaverToUse, @Nullable ClassLoader beanClassLoader) {

		if (weaverToUse == null) {
            // 此时已经被初始化为 DefaultContextLoadTimeweaver
			if (InstrumentationLoadTimeWeaver.isInstrumentationAvailable()) {
				weaverToUse = new InstrumentationLoadTimeWeaver(beanClassLoader);
			}
			else {
				throw new IllegalStateException("No LoadTimeWeaver available");
			}
		}
    // 使用 DefaultContextLoadTimeweaver 类型的 bean 中的 loadTimeWeaver 属性注册转换器
		weaverToUse.addTransformer(
				new AspectJClassBypassingClassFileTransformer(new ClassPreProcessorAgentAdapter()));
}

// AspectJWeavingEnabler#AspectJClassBypassingClassFileTransformer
// 作用仅仅是告诉 AspectJ 以 org.aspectj 开头的或者 org/aspectj 开头的类不进行处理
private static class AspectJClassBypassingClassFileTransformer implements ClassFileTransformer {

		private final ClassFileTransformer delegate;

		public AspectJClassBypassingClassFileTransformer(ClassFileTransformer delegate) {
			this.delegate = delegate;
		}

		@Override
		public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
				ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {

			if (className.startsWith("org.aspectj") || className.startsWith("org/aspectj")) {
				return classfileBuffer;
			}
            // 委托给 AspectJ 代理继续处理
			return this.delegate.transform(loader, className, classBeingRedefined, protectionDomain, classfileBuffer);
		}
}
```



### JDBC

​		`Spring`中对`JDBC`的支持中的所有数据库操作以`JdbcTemplate`为基础。其中，`execute`方法是最基础的操作，而其他操作则是传入不同的

`PreparedStatementCallback`参数来执行不同的逻辑。

```java
// JdbcTemplate 类
private <T> T execute(PreparedStatementCreator psc, PreparedStatementCallback<T> action, boolean closeResources)
			throws DataAccessException {

		Assert.notNull(psc, "PreparedStatementCreator must not be null");
		Assert.notNull(action, "Callback object must not be null");
		if (logger.isDebugEnabled()) {
			String sql = getSql(psc);
			logger.debug("Executing prepared SQL statement" + (sql != null ? " [" + sql + "]" : ""));
		}
		// 获取数据库连接
		Connection con = DataSourceUtils.getConnection(obtainDataSource());
		PreparedStatement ps = null;
		try {
			ps = psc.createPreparedStatement(con);
            // 应用设定的输入参数
			applyStatementSettings(ps);
            // 调用回调函数
			T result = action.doInPreparedStatement(ps);
			handleWarnings(ps);
			return result;
		}
		catch (SQLException ex) {
			// Release Connection early, to avoid potential connection pool deadlock
			// in the case when the exception translator hasn't been initialized yet.
            // 释放数据库连接避免当异常转换器没有被初始化的时候出现潜在的连接池死锁
			if (psc instanceof ParameterDisposer) {
				((ParameterDisposer) psc).cleanupParameters();
			}
			String sql = getSql(psc);
			psc = null;
			JdbcUtils.closeStatement(ps);
			ps = null;
			DataSourceUtils.releaseConnection(con, getDataSource());
			con = null;
			throw translateException("PreparedStatementCallback", sql, ex);
		}
		finally {
			if (closeResources) {
				if (psc instanceof ParameterDisposer) {
					((ParameterDisposer) psc).cleanupParameters();
				}
				JdbcUtils.closeStatement(ps);
				DataSourceUtils.releaseConnection(con, getDataSource());
			}
		}
}
```



#### 获取数据库连接

```java
// DataSourceUtils 类
public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
		try {
			return doGetConnection(dataSource);
		}
		catch (SQLException ex) {
			throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection", ex);
		}
		catch (IllegalStateException ex) {
			throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection: " + ex.getMessage());
		}
}

public static Connection doGetConnection(DataSource dataSource) throws SQLException {
		Assert.notNull(dataSource, "No DataSource specified");

		ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
		if (conHolder != null && (conHolder.hasConnection() || conHolder.isSynchronizedWithTransaction())) {
			conHolder.requested();
			if (!conHolder.hasConnection()) {
				logger.debug("Fetching resumed JDBC Connection from DataSource");
				conHolder.setConnection(fetchConnection(dataSource));
			}
			return conHolder.getConnection();
		}
		// Else we either got no holder or an empty thread-bound holder here.

		logger.debug("Fetching JDBC Connection from DataSource");
		Connection con = fetchConnection(dataSource);
		// 当前线程支持同步
		if (TransactionSynchronizationManager.isSynchronizationActive()) {
			try {
				// Use same Connection for further JDBC actions within the transaction.
				// Thread-bound object will get removed by synchronization at transaction completion.
                // 在事务中使用同一数据库连接
				ConnectionHolder holderToUse = conHolder;
				if (holderToUse == null) {
					holderToUse = new ConnectionHolder(con);
				}
				else {
					holderToUse.setConnection(con);
				}
                // 记录数据库连接
				holderToUse.requested();
				TransactionSynchronizationManager.registerSynchronization(
						new ConnectionSynchronization(holderToUse, dataSource));
				holderToUse.setSynchronizedWithTransaction(true);
				if (holderToUse != conHolder) {
					TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
				}
			}
			catch (RuntimeException ex) {
				// Unexpected exception from external delegation call -> close Connection and rethrow.
				releaseConnection(con, dataSource);
				throw ex;
			}
		}

		return con;
}
```



#### 应用输入参数

```java
// JdbcTemplate 类
protected void applyStatementSettings(Statement stmt) throws SQLException {
		int fetchSize = getFetchSize();
		if (fetchSize != -1) {
			stmt.setFetchSize(fetchSize);
		}
		int maxRows = getMaxRows();
		if (maxRows != -1) {
			stmt.setMaxRows(maxRows);
		}
		DataSourceUtils.applyTimeout(stmt, getDataSource(), getQueryTimeout());
}
```

​		`setFetchSize`最主要是为了减少网络交互次数设计的。访问`ResultSet`时，如果它每次只从服务器上读取一行数据，则会产生大量的开销。`setFetchSize`的

意思是当调用`rs.next`时，`ResultSet`会一次性从服务器上取得多少行数据回来，这样在下次`rs.next`时，它可以直接从内存中获取数据而不需要网络交互，提高

了效率。这个设置可能会被某些`JDBC`驱动忽略，而且设置过大也会造成内存的上升。

​		`setMaxRows`将此`Statement`对象生成的所有`ResultSet`对象可以包含的最大行数限制设置为给定数。



#### 资源释放

​		数据库的连接释放并不是直接调用了`Connection`的`API`中的`close`方法。考虑到存在事务的情况，如果当前线程存在事务，那么说明在当前线程中存在共用

数据库连接，这种情况下直接使用`ConnectionHolder`中的`released`方法进行连接数减一，而不是真正的释放连接。

```java
// DataSourceUtils 类
public static void releaseConnection(@Nullable Connection con, @Nullable DataSource dataSource) {
		try {
			doReleaseConnection(con, dataSource);
		}
		catch (SQLException ex) {
			logger.debug("Could not close JDBC Connection", ex);
		}
		catch (Throwable ex) {
			logger.debug("Unexpected exception on closing JDBC Connection", ex);
		}
}

public static void doReleaseConnection(@Nullable Connection con, @Nullable DataSource dataSource) throws SQLException {
		if (con == null) {
			return;
		}
		if (dataSource != null) {
            // 当前线程存在事务的情况下说明存在共用数据库连接直接使用 connectionHolder 中的 released 方法进行连接数减一而不是真正的释放连接
			ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
			if (conHolder != null && connectionEquals(conHolder, con)) {
				// It's the transactional Connection: Don't close it.
				conHolder.released();
				return;
			}
		}
		doCloseConnection(con, dataSource);
}

public static void doCloseConnection(Connection con, @Nullable DataSource dataSource) throws SQLException {
		if (!(dataSource instanceof SmartDataSource) || ((SmartDataSource) dataSource).shouldClose(con)) {
			con.close();
		}
}
```



#### 回调函数

​		`PreparedStatementCallback`作为一个接口，其中只有一个函数`doInPreparedStatement`，这个函数是用于调用通用方法`execute`的时候无法处理的一些个性

化处理方法：

```java
// ArgumentTypePreparedStatementSetter 类
public void setValues(PreparedStatement ps) throws SQLException {
		int parameterPosition = 1;
		if (this.args != null && this.argTypes != null) {
            // 遍历每个参数以作类型匹配及转换
			for (int i = 0; i < this.args.length; i++) {
				Object arg = this.args[i];
                // 如果是集合则需要进入集合类内部递归解析集合内部属性
				if (arg instanceof Collection && this.argTypes[i] != Types.ARRAY) {
					Collection<?> entries = (Collection<?>) arg;
					for (Object entry : entries) {
						if (entry instanceof Object[]) {
							Object[] valueArray = ((Object[]) entry);
							for (Object argValue : valueArray) {
								doSetValue(ps, parameterPosition, this.argTypes[i], argValue);
								parameterPosition++;
							}
						}
						else {
							doSetValue(ps, parameterPosition, this.argTypes[i], entry);
							parameterPosition++;
						}
					}
				}
				else {
                    // 解析当前属性
					doSetValue(ps, parameterPosition, this.argTypes[i], arg);
					parameterPosition++;
				}
			}
		}
}

protected void doSetValue(PreparedStatement ps, int parameterPosition, int argType, Object argValue)
			throws SQLException {

		StatementCreatorUtils.setParameterValue(ps, parameterPosition, argType, argValue);
}
```

```java
// StatementCreatorUtils 类
public static void setParameterValue(PreparedStatement ps, int paramIndex, int sqlType,
			@Nullable Object inValue) throws SQLException {

		setParameterValueInternal(ps, paramIndex, sqlType, null, null, inValue);
}
private static void setParameterValueInternal(PreparedStatement ps, int paramIndex, int sqlType,
			@Nullable String typeName, @Nullable Integer scale, @Nullable Object inValue) throws SQLException {

		String typeNameToUse = typeName;
		int sqlTypeToUse = sqlType;
		Object inValueToUse = inValue;

		// override type info?
		if (inValue instanceof SqlParameterValue) {
			SqlParameterValue parameterValue = (SqlParameterValue) inValue;
			if (logger.isDebugEnabled()) {
				logger.debug("Overriding type info with runtime info from SqlParameterValue: column index " + paramIndex +
						", SQL type " + parameterValue.getSqlType() + ", type name " + parameterValue.getTypeName());
			}
			if (parameterValue.getSqlType() != SqlTypeValue.TYPE_UNKNOWN) {
				sqlTypeToUse = parameterValue.getSqlType();
			}
			if (parameterValue.getTypeName() != null) {
				typeNameToUse = parameterValue.getTypeName();
			}
			inValueToUse = parameterValue.getValue();
		}

		if (logger.isTraceEnabled()) {
			logger.trace("Setting SQL statement parameter value: column index " + paramIndex +
					", parameter value [" + inValueToUse +
					"], value class [" + (inValueToUse != null ? inValueToUse.getClass().getName() : "null") +
					"], SQL type " + (sqlTypeToUse == SqlTypeValue.TYPE_UNKNOWN ? "unknown" : Integer.toString(sqlTypeToUse)));
		}

		if (inValueToUse == null) {
			setNull(ps, paramIndex, sqlTypeToUse, typeNameToUse);
		}
		else {
			setValue(ps, paramIndex, sqlTypeToUse, typeNameToUse, scale, inValueToUse);
		}
}
```



#### query 的实现

```java
// JdbcTemplate 类
public <T> T query(
			PreparedStatementCreator psc, @Nullable final PreparedStatementSetter pss, final ResultSetExtractor<T> rse)
			throws DataAccessException {

		Assert.notNull(rse, "ResultSetExtractor must not be null");
		logger.debug("Executing prepared SQL query");

		return execute(psc, new PreparedStatementCallback<T>() {
			@Override
			@Nullable
			public T doInPreparedStatement(PreparedStatement ps) throws SQLException {
				ResultSet rs = null;
				try {
					if (pss != null) {
						pss.setValues(ps);
					}
					rs = ps.executeQuery();
                    // 将查询结果封装并转换为 POJO
					return rse.extractData(rs);
				}
				finally {
					JdbcUtils.closeResultSet(rs);
					if (pss instanceof ParameterDisposer) {
						((ParameterDisposer) pss).cleanupParameters();
					}
				}
			}
		}, true);
}
```

```java
// RowMapperResultSetExtractor 类
public List<T> extractData(ResultSet rs) throws SQLException {
		List<T> results = (this.rowsExpected > 0 ? new ArrayList<>(this.rowsExpected) : new ArrayList<>());
		int rowNum = 0;
		while (rs.next()) {
			results.add(this.rowMapper.mapRow(rs, rowNum++));
		}
		return results;
}
```



### 事务

```java
// TxNamespaceHandler 类
public void init() {
		registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenBeanDefinitionParser());
		registerBeanDefinitionParser("jta-transaction-manager", new JtaTransactionManagerBeanDefinitionParser());
}
```

```java
// AnnotationDrivenBeanDefinitionParser 类
public BeanDefinition parse(Element element, ParserContext parserContext) {
		registerTransactionalEventListenerFactory(parserContext);
		String mode = element.getAttribute("mode");
		if ("aspectj".equals(mode)) {
			// mode="aspectj"
			registerTransactionAspect(element, parserContext);
			if (ClassUtils.isPresent("javax.transaction.Transactional", getClass().getClassLoader())) {
				registerJtaTransactionAspect(element, parserContext);
			}
		}
		else {
			// mode="proxy"
             // 默认配置
			AopAutoProxyConfigurer.configureAutoProxyCreator(element, parserContext);
		}
		return null;
}
```

```java
// AopAutoProxyConfigurer 类
public static void configureAutoProxyCreator(Element element, ParserContext parserContext) {
			AopNamespaceUtils.registerAutoProxyCreatorIfNecessary(parserContext, element);

			String txAdvisorBeanName = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME;
			if (!parserContext.getRegistry().containsBeanDefinition(txAdvisorBeanName)) {
				Object eleSource = parserContext.extractSource(element);

				// Create the TransactionAttributeSource definition.
                // 创建 TransactionAttributeSource 的 bean
				RootBeanDefinition sourceDef = new RootBeanDefinition(
						"org.springframework.transaction.annotation.AnnotationTransactionAttributeSource");
				sourceDef.setSource(eleSource);
				sourceDef.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
                // 注册 bean,并使用 Spring 中的定义规则生成 beanName
				String sourceName = parserContext.getReaderContext().registerWithGeneratedName(sourceDef);

				// Create the TransactionInterceptor definition.
                // 创建 TransactionInterceptor 的 bean
				RootBeanDefinition interceptorDef = new RootBeanDefinition(TransactionInterceptor.class);
				interceptorDef.setSource(eleSource);
				interceptorDef.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
				registerTransactionManager(element, interceptorDef);
				interceptorDef.getPropertyValues().add("transactionAttributeSource", new RuntimeBeanReference(sourceName));
                // 注册 bean,并使用 Spring 中的定义规则生成 beanName
				String interceptorName = parserContext.getReaderContext().registerWithGeneratedName(interceptorDef);

				// Create the TransactionAttributeSourceAdvisor definition.
                // 创建 TransactionAttributeSourceAdvisor 的 bean
				RootBeanDefinition advisorDef = new RootBeanDefinition(BeanFactoryTransactionAttributeSourceAdvisor.class);
				advisorDef.setSource(eleSource);
				advisorDef.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
                // 将 sourceName 的 bean 注入 advisorDef 的 transactionAttributesource 属性中
				advisorDef.getPropertyValues().add("transactionAttributeSource", new RuntimeBeanReference(sourceName));
                // 将 interceptorName 的 bean 注入 advisorDef 的 adviceBeanName 属性中
				advisorDef.getPropertyValues().add("adviceBeanName", interceptorName);
                // 如果配置了 order 属性,则加入到 bean 中
				if (element.hasAttribute("order")) {
					advisorDef.getPropertyValues().add("order", element.getAttribute("order"));
				}
				parserContext.getRegistry().registerBeanDefinition(txAdvisorBeanName, advisorDef);
				// 创建 CompositeComponentDefinition
				CompositeComponentDefinition compositeDef = new CompositeComponentDefinition(element.getTagName(), eleSource);
				compositeDef.addNestedComponent(new BeanComponentDefinition(sourceDef, sourceName));
				compositeDef.addNestedComponent(new BeanComponentDefinition(interceptorDef, interceptorName));
				compositeDef.addNestedComponent(new BeanComponentDefinition(advisorDef, txAdvisorBeanName));
				parserContext.registerComponent(compositeDef);
			}
}
```



#### 注册 InfrastructureAdvisorAutoProxyCreator

```java
// AopNamespaceUtils 类
public static void registerAutoProxyCreatorIfNecessary(
			ParserContext parserContext, Element sourceElement) {

		BeanDefinition beanDefinition = AopConfigUtils.registerAutoProxyCreatorIfNecessary(
				parserContext.getRegistry(), parserContext.extractSource(sourceElement));
		useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
		registerComponentIfNecessary(beanDefinition, parserContext);
}
```

```java
// AopConfigUtils 类
public static BeanDefinition registerAutoProxyCreatorIfNecessary(
			BeanDefinitionRegistry registry, @Nullable Object source) {

		return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);
}
```

![](image/QQ截图20220324160507.png)

```java
// AbstractAutoProxyCreator 类，因为 InfrastructureAdvisorAutoProxyCreator 间接实现了 BeanPostProcessor 接口
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
		if (bean != null) {
            // 根据给定的 bean 的 class 和 name 构建出个 key， beanClassName_beanName
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
            // 是否是由于避免循环依赖而创建的 bean 代理
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    	// 如果已经处理过
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
   		// 给定的 bea n类是否代表一个基础设施类，不应代理，或者配置了指定 bean 不需要自动代理
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// Create proxy if we have advice.
		// 找出指定 bean 对应的增强器
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
            // 根据找出的增强器创建代理
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
}
```



##### 获取对应的增强器

```java
// AbstractAdvisorAutoProxyCreator 类
protected Object[] getAdvicesAndAdvisorsForBean(
			Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {

		List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
		if (advisors.isEmpty()) {
			return DO_NOT_PROXY;
		}
		return advisors.toArray();
}

protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
		List<Advisor> candidateAdvisors = findCandidateAdvisors();
		List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
		extendAdvisors(eligibleAdvisors);
		if (!eligibleAdvisors.isEmpty()) {
			eligibleAdvisors = sortAdvisors(eligibleAdvisors);
		}
		return eligibleAdvisors;
}
```



###### 寻找增强器

```java
// AbstractAdvisorAutoProxyCreator 类
protected List<Advisor> findCandidateAdvisors() {
		Assert.state(this.advisorRetrievalHelper != null, "No BeanFactoryAdvisorRetrievalHelper available");
		return this.advisorRetrievalHelper.findAdvisorBeans();
}
```

```java
// BeanFactoryAdvisorRetrievalHelper 类
public List<Advisor> findAdvisorBeans() {
		// Determine list of advisor bean names, if not cached already.
		String[] advisorNames = this.cachedAdvisorBeanNames;
		if (advisorNames == null) {
			// Do not initialize FactoryBeans here: We need to leave all regular beans
			// uninitialized to let the auto-proxy creator apply to them!
			advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
					this.beanFactory, Advisor.class, true, false);
			this.cachedAdvisorBeanNames = advisorNames;
		}
		if (advisorNames.length == 0) {
			return new ArrayList<>();
		}

		List<Advisor> advisors = new ArrayList<>();
		for (String name : advisorNames) {
			if (isEligibleBean(name)) {
				if (this.beanFactory.isCurrentlyInCreation(name)) {
					if (logger.isTraceEnabled()) {
						logger.trace("Skipping currently created advisor '" + name + "'");
					}
				}
				else {
					try {
						advisors.add(this.beanFactory.getBean(name, Advisor.class));
					}
					catch (BeanCreationException ex) {
						Throwable rootCause = ex.getMostSpecificCause();
						if (rootCause instanceof BeanCurrentlyInCreationException) {
							BeanCreationException bce = (BeanCreationException) rootCause;
							String bceBeanName = bce.getBeanName();
							if (bceBeanName != null && this.beanFactory.isCurrentlyInCreation(bceBeanName)) {
								if (logger.isTraceEnabled()) {
									logger.trace("Skipping advisor '" + name +
											"' with dependency on currently created bean: " + ex.getMessage());
								}
								// Ignore: indicates a reference back to the bean we're trying to advise.
								// We want to find advisors other than the currently created bean itself.
								continue;
							}
						}
						throw ex;
					}
				}
			}
		}
		return advisors;
}
```

​		前面分析自定义标签时曾经注册了一个类型为`BeanFactoryTransactionAttributeSourceAdvisor`的`bean`，而在此`bean`中又注入了另外两个`Bean`，那么此时

这个`Bean`就会被开始使用了。因为`BeanFactoryTransactionAttributeSourceAdvisor`同样也实现了`Advisor`接口，那么在获取所有增强器时自然也会将此`bean`提

取出来，并随着其他增强器一起在后续的步骤中被织入代理。



###### 候选增强器中寻找到匹配项

```java
// AopUtils 类
public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
		if (candidateAdvisors.isEmpty()) {
			return candidateAdvisors;
		}
		List<Advisor> eligibleAdvisors = new ArrayList<>();
	    // 首先处理引介增强
		for (Advisor candidate : candidateAdvisors) {
			if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
				eligibleAdvisors.add(candidate);
			}
		}
		boolean hasIntroductions = !eligibleAdvisors.isEmpty();
		for (Advisor candidate : candidateAdvisors) {
            // 引介增强已经处理
			if (candidate instanceof IntroductionAdvisor) {
				// already processed
				continue;
			}
            // 对于普通 bean 的处理
			if (canApply(candidate, clazz, hasIntroductions)) {
				eligibleAdvisors.add(candidate);
			}
		}
		return eligibleAdvisors;
}

public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
		if (advisor instanceof IntroductionAdvisor) {
			return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
		}
		else if (advisor instanceof PointcutAdvisor) {
			PointcutAdvisor pca = (PointcutAdvisor) advisor;
			return canApply(pca.getPointcut(), targetClass, hasIntroductions);
		}
		else {
			// It doesn't have a pointcut so we assume it applies.
			return true;
		}
}

public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
		Assert.notNull(pc, "Pointcut must not be null");
		if (!pc.getClassFilter().matches(targetClass)) {
			return false;
		}
		// 此时的 pc 表示 TransactionAttributeSourcePointcut
    	// pc.getMethodMatcher() 返回的正是自身 ( this ) .
		MethodMatcher methodMatcher = pc.getMethodMatcher();
		if (methodMatcher == MethodMatcher.TRUE) {
			// No need to iterate the methods if we're matching any method anyway...
			return true;
		}

		IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
		if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
			introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
		}

		Set<Class<?>> classes = new LinkedHashSet<>();
    	// 获取对应类的所有接口及其本身

		if (!Proxy.isProxyClass(targetClass)) {
			classes.add(ClassUtils.getUserClass(targetClass));
		}
		classes.addAll(ClassUtils.getAllInterfacesForClassAsSet(targetClass));
		// 遍历，遍历过程中又对类中的方法再次遍历，一旦匹配成功便认为这个类适用于当前增强器
		for (Class<?> clazz : classes) {
			Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
			for (Method method : methods) {
				if (introductionAwareMethodMatcher != null ?
						introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions) :
						methodMatcher.matches(method, targetClass)) {
					return true;
				}
			}
		}

		return false;
}
```

```java
// TransactionAttributeSourcePointcut 类
public boolean matches(Method method, Class<?> targetClass) {
		TransactionAttributeSource tas = getTransactionAttributeSource();
    	// tas 是 AnnotationTransactionAttributeSource 的实例
		return (tas == null || tas.getTransactionAttribute(method, targetClass) != null);
}
```

```java
// AbstractFallbackTransactionAttributeSource 类
public TransactionAttribute getTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
		if (method.getDeclaringClass() == Object.class) {
			return null;
		}

		// First, see if we have a cached value.
    	// 首先尝试从缓存中加载
		Object cacheKey = getCacheKey(method, targetClass);
		TransactionAttribute cached = this.attributeCache.get(cacheKey);
		if (cached != null) {
			// Value will either be canonical value indicating there is no transaction attribute,
			// or an actual transaction attribute.
			if (cached == NULL_TRANSACTION_ATTRIBUTE) {
				return null;
			}
			else {
				return cached;
			}
		}
		else {
			// We need to work it out.
			TransactionAttribute txAttr = computeTransactionAttribute(method, targetClass);
			// Put it in the cache.
			if (txAttr == null) {
				this.attributeCache.put(cacheKey, NULL_TRANSACTION_ATTRIBUTE);
			}
			else {
				String methodIdentification = ClassUtils.getQualifiedMethodName(method, targetClass);
				if (txAttr instanceof DefaultTransactionAttribute) {
					DefaultTransactionAttribute dta = (DefaultTransactionAttribute) txAttr;
					dta.setDescriptor(methodIdentification);
					dta.resolveAttributeStrings(this.embeddedValueResolver);
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Adding transactional method '" + methodIdentification + "' with attribute: " + txAttr);
				}
				this.attributeCache.put(cacheKey, txAttr);
			}
			return txAttr;
		}
}
```



###### 提取事务标签

```java
// AbstractFallbackTransactionAttributeSource 类
protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
		// Don't allow non-public methods, as configured.
		if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
			return null;
		}

		// The method may be on an interface, but we need attributes from the target class.
		// If the target class is null, the method will be unchanged.
		// method 代表接口中的方法，specificMethod 代表实现类中的方法
		Method specificMethod = AopUtils.getMostSpecificMethod(method, targetClass);

		// First try is the method in the target class.
	    // 查看方法中是否存在事务声明
		TransactionAttribute txAttr = findTransactionAttribute(specificMethod);
		if (txAttr != null) {
			return txAttr;
		}

		// Second try is the transaction attribute on the target class.
	    // 查看方法所在类中是否存在事务声明
		txAttr = findTransactionAttribute(specificMethod.getDeclaringClass());
		if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
			return txAttr;
		}

	    // 如果存在接口，则到接口中去寻找
		if (specificMethod != method) {
			// Fallback is to look at the original method.
            // 查找接口方法
			txAttr = findTransactionAttribute(method);
			if (txAttr != null) {
				return txAttr;
			}
			// Last fallback is the class of the original method.
            // 到接口中的类中去寻找
			txAttr = findTransactionAttribute(method.getDeclaringClass());
			if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
				return txAttr;
			}
		}

		return null;
}
```

​		如果方法中存在事务属性，则使用方法上的属性，否则使用方法所在的类上的属性，如果方法所在类的属性上还是没有搜寻到对应的事务属性，那么再搜寻接

口中的方法，再没有的话，最后尝试搜寻接口的类上面的声明。

```java
// AnnotationTransactionAttributeSource 类
protected TransactionAttribute findTransactionAttribute(Method method) {
		return determineTransactionAttribute(method);
}

protected TransactionAttribute determineTransactionAttribute(AnnotatedElement element) {
		for (TransactionAnnotationParser parser : this.annotationParsers) {
			TransactionAttribute attr = parser.parseTransactionAnnotation(element);
			if (attr != null) {
				return attr;
			}
		}
		return null;
}
```

```java
// SpringTransactionAnnotationParser 类
public TransactionAttribute parseTransactionAnnotation(AnnotatedElement element) {
    	// 判断当前类是否含有 Transactional 注解
		AnnotationAttributes attributes = AnnotatedElementUtils.findMergedAnnotationAttributes(
				element, Transactional.class, false, false);
		if (attributes != null) {
			return parseTransactionAnnotation(attributes);
		}
		else {
			return null;
		}
}

protected TransactionAttribute parseTransactionAnnotation(AnnotationAttributes attributes) {
		RuleBasedTransactionAttribute rbta = new RuleBasedTransactionAttribute();

		Propagation propagation = attributes.getEnum("propagation");
	    // 解析 propagation
		rbta.setPropagationBehavior(propagation.value());
    	// 解析 propagation
		Isolation isolation = attributes.getEnum("isolation");
		rbta.setIsolationLevel(isolation.value());
    	// 解析 timeout
		rbta.setTimeout(attributes.getNumber("timeout").intValue());
		String timeoutString = attributes.getString("timeoutString");
		Assert.isTrue(!StringUtils.hasText(timeoutString) || rbta.getTimeout() < 0,
				"Specify 'timeout' or 'timeoutString', not both");
		rbta.setTimeoutString(timeoutString);
		// 解析 readOnly
		rbta.setReadOnly(attributes.getBoolean("readOnly"));
    	// 解析 value
		rbta.setQualifier(attributes.getString("value"));
		rbta.setLabels(Arrays.asList(attributes.getStringArray("label")));

    	// 解析 rollbackFor
		List<RollbackRuleAttribute> rollbackRules = new ArrayList<>();
		for (Class<?> rbRule : attributes.getClassArray("rollbackFor")) {
			rollbackRules.add(new RollbackRuleAttribute(rbRule));
		}
		// 解析 rollbackForClassName
		for (String rbRule : attributes.getStringArray("rollbackForClassName")) {
			rollbackRules.add(new RollbackRuleAttribute(rbRule));
		}
	    // 解析 noRollbackFor
		for (Class<?> rbRule : attributes.getClassArray("noRollbackFor")) {
			rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
		}
	    // 解析 noRollbackForClassName
		for (String rbRule : attributes.getStringArray("noRollbackForClassName")) {
			rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
		}
		rbta.setRollbackRules(rollbackRules);

		return rbta;
}
```

​		`BeanFactoryTransactionAttributeSourceAdvisor`作为Advisor的实现类，自然要遵从`Advisor`的处理方式，当代理被调用时会调用这个类的增强方法，也就

是此`bean`的`Advise`，又因为在解析事务定义标签时把`TransactionInterceptor`类型的`bean`注入到了`BeanFactoryTransactionAttributeSourceAdvisor`中，所

以，在调用事务增强器增强的代理类时会首先执行`TransactionInterceptor`进行增强，同时，也就是在`TransactionInterceptor`类中的`invoke`方法中完成了整个

事务的逻辑。



#### 事务增强器

​		`TransactionInterceptor`支撑着整个事务功能的架构，`TransactionInterceptor`类继承自`MethodInterceptor`：

![](image/QQ截图20220326111945.png)

```java
// TransactionInterceptor 类
public Object invoke(MethodInvocation invocation) throws Throwable {
    
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

		return invokeWithinTransaction(invocation.getMethod(), targetClass, new CoroutinesInvocationCallback() {
			@Override
			@Nullable
			public Object proceedWithInvocation() throws Throwable {
				return invocation.proceed();
			}
			@Override
			public Object getTarget() {
				return invocation.getThis();
			}
			@Override
			public Object[] getArguments() {
				return invocation.getArguments();
			}
		});
}
```

```java
// TransactionAspectSupport 类
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
			final InvocationCallback invocation) throws Throwable {

		// If the transaction attribute is null, the method is non-transactional.
	    // 获取对应事务属性
		TransactionAttributeSource tas = getTransactionAttributeSource();
		final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
	    // 获取 beanFactory 中的 transactionManager
		final TransactionManager tm = determineTransactionManager(txAttr);

		if (this.reactiveAdapterRegistry != null && tm instanceof ReactiveTransactionManager) {
			boolean isSuspendingFunction = KotlinDetector.isSuspendingFunction(method);
			boolean hasSuspendingFlowReturnType = isSuspendingFunction &&
					COROUTINES_FLOW_CLASS_NAME.equals(new MethodParameter(method, -1).getParameterType().getName());
			if (isSuspendingFunction && !(invocation instanceof CoroutinesInvocationCallback)) {
				throw new IllegalStateException("Coroutines invocation not supported: " + method);
			}
			CoroutinesInvocationCallback corInv = (isSuspendingFunction ? (CoroutinesInvocationCallback) invocation : null);

			ReactiveTransactionSupport txSupport = this.transactionSupportCache.computeIfAbsent(method, key -> {
				Class<?> reactiveType =
						(isSuspendingFunction ? (hasSuspendingFlowReturnType ? Flux.class : Mono.class) : method.getReturnType());
				ReactiveAdapter adapter = this.reactiveAdapterRegistry.getAdapter(reactiveType);
				if (adapter == null) {
					throw new IllegalStateException("Cannot apply reactive transaction to non-reactive return type: " +
							method.getReturnType());
				}
				return new ReactiveTransactionSupport(adapter);
			});

			InvocationCallback callback = invocation;
			if (corInv != null) {
				callback = () -> CoroutinesUtils.invokeSuspendingFunction(method, corInv.getTarget(), corInv.getArguments());
			}
			Object result = txSupport.invokeWithinTransaction(method, targetClass, callback, txAttr, (ReactiveTransactionManager) tm);
			if (corInv != null) {
				Publisher<?> pr = (Publisher<?>) result;
				return (hasSuspendingFlowReturnType ? KotlinDelegate.asFlow(pr) :
						KotlinDelegate.awaitSingleOrNull(pr, corInv.getContinuation()));
			}
			return result;
		}

		PlatformTransactionManager ptm = asPlatformTransactionManager(tm);
	    // 构造方法唯一标识（类.方法)
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);
		// 声明式事务处理
		if (txAttr == null || !(ptm instanceof CallbackPreferringPlatformTransactionManager)) {
			// Standard transaction demarcation with getTransaction and commit/rollback calls.
            // 创建 TransactionInfo
			TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

			Object retVal;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
                // 执行被增强方法
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
                // 清除信息
				cleanupTransactionInfo(txInfo);
			}

			if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
				// Set rollback-only in case of Vavr failure matching our rollback rules...
				TransactionStatus status = txInfo.getTransactionStatus();
				if (status != null && txAttr != null) {
					retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
				}
			}
			// 提交事务
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}

		else {
            // 编程式事务处理
			Object result;
			final ThrowableHolder throwableHolder = new ThrowableHolder();

			// It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.
			try {
				result = ((CallbackPreferringPlatformTransactionManager) ptm).execute(txAttr, status -> {
					TransactionInfo txInfo = prepareTransactionInfo(ptm, txAttr, joinpointIdentification, status);
					try {
						Object retVal = invocation.proceedWithInvocation();
						if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
							// Set rollback-only in case of Vavr failure matching our rollback rules...
							retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
						}
						return retVal;
					}
					catch (Throwable ex) {
						if (txAttr.rollbackOn(ex)) {
							// A RuntimeException: will lead to a rollback.
							if (ex instanceof RuntimeException) {
								throw (RuntimeException) ex;
							}
							else {
								throw new ThrowableHolderException(ex);
							}
						}
						else {
							// A normal return value: will lead to a commit.
							throwableHolder.throwable = ex;
							return null;
						}
					}
					finally {
						cleanupTransactionInfo(txInfo);
					}
				});
			}
			catch (ThrowableHolderException ex) {
				throw ex.getCause();
			}
			catch (TransactionSystemException ex2) {
				if (throwableHolder.throwable != null) {
					logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
					ex2.initApplicationException(throwableHolder.throwable);
				}
				throw ex2;
			}
			catch (Throwable ex2) {
				if (throwableHolder.throwable != null) {
					logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
				}
				throw ex2;
			}

			// Check result state: It might indicate a Throwable to rethrow.
			if (throwableHolder.throwable != null) {
				throw throwableHolder.throwable;
			}
			return result;
		}
}
```



##### 创建事务

```java
// TransactionAspectSupport 类
protected TransactionInfo createTransactionIfNecessary(@Nullable PlatformTransactionManager tm,
			@Nullable TransactionAttribute txAttr, final String joinpointIdentification) {

		// If no name specified, apply method identification as transaction name.
	    // 如果没有名称指定则使用方法唯一标识,并使用 DelegatingTransactionAttribute 封装 txAttr
    	// 传入的 txAttr 实际类型为 RuleBasedTransactionAttribute
		if (txAttr != null && txAttr.getName() == null) {
			txAttr = new DelegatingTransactionAttribute(txAttr) {
				@Override
				public String getName() {
					return joinpointIdentification;
				}
			};
		}

		TransactionStatus status = null;
		if (txAttr != null) {
			if (tm != null) {
                // 获取 Transactionstatus
				status = tm.getTransaction(txAttr);
			}
			else {
				if (logger.isDebugEnabled()) {
					logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
							"] because no transaction manager has been configured");
				}
			}
		}
	    // 根据指定的属性与 status 准备一个 TransactionInfo
		return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
}
```

```java
// AbstractPlatformTransactionManager 类
public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
			throws TransactionException {

		// Use defaults if no transaction definition given.
		TransactionDefinition def = (definition != null ? definition : TransactionDefinition.withDefaults());

		Object transaction = doGetTransaction();
		boolean debugEnabled = logger.isDebugEnabled();
		// 判断当前线程是否存在事务，判读依据为当前线程记录的连接不为空且连接中 (connectionHolder) 中的
    	// transactionActive属性不为空
		if (isExistingTransaction(transaction)) {
			// Existing transaction found -> check propagation behavior to find out how to behave.
            // 当前线程已经存在事务
			return handleExistingTransaction(def, transaction, debugEnabled);
		}

		// Check definition settings for new transaction.
	    // 事务超时设置验证
		if (def.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
			throw new InvalidTimeoutException("Invalid transaction timeout", def.getTimeout());
		}

		// No existing transaction found -> check propagation behavior to find out how to proceed.
		// 如果当前线程不存在事务，但是 propagationBehavior 却被声明为 PROPAGATION_MANDATORY 抛出异常
		if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
			throw new IllegalTransactionStateException(
					"No existing transaction found for transaction marked with propagation 'mandatory'");
		}
         // PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED 都需要新建事务
    	else if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
				def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
				def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {

            
			//空挂起
			SuspendedResourcesHolder suspendedResources = suspend(null);
			if (debugEnabled) {
				logger.debug("Creating new transaction with name [" + def.getName() + "]: " + def);
			}
			try {
				return startTransaction(def, transaction, debugEnabled, suspendedResources);
			}
			catch (RuntimeException | Error ex) {
				resume(null, suspendedResources);
				throw ex;
			}
		}
		else {
			// Create "empty" transaction: no actual transaction, but potentially synchronization.
			if (def.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT && logger.isWarnEnabled()) {
				logger.warn("Custom isolation level specified but no actual transaction initiated; " +
						"isolation level will effectively be ignored: " + def);
			}
			boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
			return prepareTransactionStatus(def, null, true, newSynchronization, debugEnabled, null);
		}
}

private TransactionStatus startTransaction(TransactionDefinition definition, Object transaction,
			boolean debugEnabled, @Nullable SuspendedResourcesHolder suspendedResources) {

		boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
		DefaultTransactionStatus status = newTransactionStatus(
				definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
   		 /**
   		 	构造 transaction ,包括设置 connectionHolder、隔离级别、timout 如果是新连接,绑定到当前线程
		*/
		doBegin(transaction, definition);
	    // 新同步事务的设置，针对于当前线程的设置
		prepareSynchronization(status, definition);
		return status;
}
```



​		创建对应的事务实例，使用的是`DataSourceTransactionManager`中的`doGetTransaction`方法，创建基于`JDBC`的事务实例。如果当前线程中存在关于

`dataSource`的连接，那么直接使用。这里有一个对保存点的设置，是否开启允许保存点取决于是否设置了允许嵌入式事务。

```java
// DataSourceTransactionManager 类
protected Object doGetTransaction() {
		DataSourceTransactionObject txObject = new DataSourceTransactionObject();
		txObject.setSavepointAllowed(isNestedTransactionAllowed());
	    // 如果当前线程已经记录数据库连接则使用原有连接
		ConnectionHolder conHolder =
				(ConnectionHolder) TransactionSynchronizationManager.getResource(obtainDataSource());
	    // false 表示非新创建连接
		txObject.setConnectionHolder(conHolder, false);
		return txObject;
}
```



​		对于一些隔离级别、`timeout`等功能的设置并不是由`Spring`来完成的，而是委托给底层的数据库连接去做的：

```java
// DataSourceTransactionManager 类
protected void doBegin(Object transaction, TransactionDefinition definition) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
		Connection con = null;

		try {
			if (!txObject.hasConnectionHolder() ||
					txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
				Connection newCon = obtainDataSource().getConnection();
				if (logger.isDebugEnabled()) {
					logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
				}
				txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
			}

			txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
			con = txObject.getConnectionHolder().getConnection();
			// 设置隔离级别
			Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
			txObject.setPreviousIsolationLevel(previousIsolationLevel);
			txObject.setReadOnly(definition.isReadOnly());

			// Switch to manual commit if necessary. This is very expensive in some JDBC drivers,
			// so we don't want to do it unnecessarily (for example if we've explicitly
			// configured the connection pool to set it already).
            // 更改自动提交设置，由 Spring 控制提交
			if (con.getAutoCommit()) {
				txObject.setMustRestoreAutoCommit(true);
				if (logger.isDebugEnabled()) {
					logger.debug("Switching JDBC Connection [" + con + "] to manual commit");
				}
				con.setAutoCommit(false);
			}

			prepareTransactionalConnection(con, definition);
            // 设置判断当前线程是否存在事务的依据
			txObject.getConnectionHolder().setTransactionActive(true);

			int timeout = determineTimeout(definition);
			if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {
				txObject.getConnectionHolder().setTimeoutInSeconds(timeout);
			}

			// Bind the connection holder to the thread.
			if (txObject.isNewConnectionHolder()) {
                // 将当前获取到的连接绑定到当前线程
				TransactionSynchronizationManager.bindResource(obtainDataSource(), txObject.getConnectionHolder());
			}
		}

		catch (Throwable ex) {
			if (txObject.isNewConnectionHolder()) {
				DataSourceUtils.releaseConnection(con, obtainDataSource());
				txObject.setConnectionHolder(null, false);
			}
			throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);
		}
}
```

```java
// DataSourceUtils 类
public static Integer prepareConnectionForTransaction(Connection con, @Nullable TransactionDefinition definition)
			throws SQLException {

		Assert.notNull(con, "No Connection specified");

		boolean debugEnabled = logger.isDebugEnabled();
		// Set read-only flag.
	    // 设置数据连接的只读标识
		if (definition != null && definition.isReadOnly()) {
			try {
				if (debugEnabled) {
					logger.debug("Setting JDBC Connection [" + con + "] read-only");
				}
				con.setReadOnly(true);
			}
			catch (SQLException | RuntimeException ex) {
				Throwable exToCheck = ex;
				while (exToCheck != null) {
					if (exToCheck.getClass().getSimpleName().contains("Timeout")) {
						// Assume it's a connection timeout that would otherwise get lost: e.g. from JDBC 4.0
						throw ex;
					}
					exToCheck = exToCheck.getCause();
				}
				// "read-only not supported" SQLException -> ignore, it's just a hint anyway
				logger.debug("Could not set JDBC Connection read-only", ex);
			}
		}

		// Apply specific isolation level, if any.
	    // 设置数据库连接的隔离级别
		Integer previousIsolationLevel = null;
		if (definition != null && definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT) {
			if (debugEnabled) {
				logger.debug("Changing isolation level of JDBC Connection [" + con + "] to " +
						definition.getIsolationLevel());
			}
			int currentIsolation = con.getTransactionIsolation();
			if (currentIsolation != definition.getIsolationLevel()) {
				previousIsolationLevel = currentIsolation;
				con.setTransactionIsolation(definition.getIsolationLevel());
			}
		}

		return previousIsolationLevel;
}
```

```java
// AbstractPlatformTransactionManager 类
// 在创建事务时被调用，用于将事务信息记录在当前线程中，在 AbstractPlatformTransactionManager 类中的 startTransaction 方法中被调用
protected void prepareSynchronization(DefaultTransactionStatus status, TransactionDefinition definition) {
		if (status.isNewSynchronization()) {
			TransactionSynchronizationManager.setActualTransactionActive(status.hasTransaction());
			TransactionSynchronizationManager.setCurrentTransactionIsolationLevel(
					definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT ?
							definition.getIsolationLevel() : null);
			TransactionSynchronizationManager.setCurrentTransactionReadOnly(definition.isReadOnly());
			TransactionSynchronizationManager.setCurrentTransactionName(definition.getName());
			TransactionSynchronizationManager.initSynchronization();
		}
}
```



​		处理已经存在的事务：

```java
// AbstractPlatformTransactionManager 类
private TransactionStatus handleExistingTransaction(
			TransactionDefinition definition, Object transaction, boolean debugEnabled)
			throws TransactionException {

		if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NEVER) {
			throw new IllegalTransactionStateException(
					"Existing transaction found for transaction marked with propagation 'never'");
		}

		if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NOT_SUPPORTED) {
			if (debugEnabled) {
				logger.debug("Suspending current transaction");
			}
			Object suspendedResources = suspend(transaction);
			boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
			return prepareTransactionStatus(
					definition, null, false, newSynchronization, debugEnabled, suspendedResources);
		}

		if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW) {
			if (debugEnabled) {
				logger.debug("Suspending current transaction, creating new transaction with name [" +
						definition.getName() + "]");
			}
            // 新事务的建立，suspend 将对原事务挂起，记录原有事务的状态，便于后续操作对事务的恢复
			SuspendedResourcesHolder suspendedResources = suspend(transaction);
			try {
				return startTransaction(definition, transaction, debugEnabled, suspendedResources);
			}
			catch (RuntimeException | Error beginEx) {
				resumeAfterBeginException(transaction, suspendedResources, beginEx);
				throw beginEx;
			}
		}
		// 嵌入式事务的处理
		if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
			if (!isNestedTransactionAllowed()) {
				throw new NestedTransactionNotSupportedException(
						"Transaction manager does not allow nested transactions by default - " +
						"specify 'nestedTransactionAllowed' property with value 'true'");
			}
			if (debugEnabled) {
				logger.debug("Creating nested transaction with name [" + definition.getName() + "]");
			}
			if (useSavepointForNestedTransaction()) {
				// Create savepoint within existing Spring-managed transaction,
				// through the SavepointManager API implemented by TransactionStatus.
				// Usually uses JDBC 3.0 savepoints. Never activates Spring synchronization.
                // 如果没有可以使用保存点的方式控制事务回滚，那么在嵌入式事务的建立初始建立保存点
				DefaultTransactionStatus status =
						prepareTransactionStatus(definition, transaction, false, false, debugEnabled, null);
				status.createAndHoldSavepoint();
				return status;
			}
			else {
				// Nested transaction through nested begin and commit/rollback calls.
				// Usually only for JTA: Spring synchronization might get activated here
				// in case of a pre-existing JTA transaction.
                // 有些情况是不能使用保存点操作，比如 JTA ，那么建立新事务
				return startTransaction(definition, transaction, debugEnabled, null);
			}
		}

		// Assumably PROPAGATION_SUPPORTS or PROPAGATION_REQUIRED.
		if (debugEnabled) {
			logger.debug("Participating in existing transaction");
		}
		if (isValidateExistingTransaction()) {
			if (definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT) {
				Integer currentIsolationLevel = TransactionSynchronizationManager.getCurrentTransactionIsolationLevel();
				if (currentIsolationLevel == null || currentIsolationLevel != definition.getIsolationLevel()) {
					Constants isoConstants = DefaultTransactionDefinition.constants;
					throw new IllegalTransactionStateException("Participating transaction with definition [" +
							definition + "] specifies isolation level which is incompatible with existing transaction: " +
							(currentIsolationLevel != null ?
									isoConstants.toCode(currentIsolationLevel, DefaultTransactionDefinition.PREFIX_ISOLATION) :
									"(unknown)"));
				}
			}
			if (!definition.isReadOnly()) {
				if (TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {
					throw new IllegalTransactionStateException("Participating transaction with definition [" +
							definition + "] is not marked as read-only but existing transaction is");
				}
			}
		}
		boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
		return prepareTransactionStatus(definition, transaction, false, newSynchronization, debugEnabled, null);
}
```



​		当已经建立事务连接并完成了事务信息的提取后，需要将所有的事务信息统一记录在`TransactionInfo`类型的实例中，这个实例包含了目标方法开始前的所有

状态信息，一旦事务执行失败，`Spring`会通过`TransactionInfo`类型的实例中的信息来进行回滚等后续工作。

```java
// TransactionAspectSupport 类
protected TransactionInfo prepareTransactionInfo(@Nullable PlatformTransactionManager tm,
			@Nullable TransactionAttribute txAttr, String joinpointIdentification,
			@Nullable TransactionStatus status) {

		TransactionInfo txInfo = new TransactionInfo(tm, txAttr, joinpointIdentification);
		if (txAttr != null) {
			// We need a transaction for this method...
			if (logger.isTraceEnabled()) {
				logger.trace("Getting transaction for [" + txInfo.getJoinpointIdentification() + "]");
			}
			// The transaction manager will flag an error if an incompatible tx already exists.
            // 记录事务状态
			txInfo.newTransactionStatus(status);
		}
		else {
			// The TransactionInfo.hasTransaction() method will return false. We created it only
			// to preserve the integrity of the ThreadLocal stack maintained in this class.
			if (logger.isTraceEnabled()) {
				logger.trace("No need to create transaction for [" + joinpointIdentification +
						"]: This method is not transactional.");
			}
		}

		// We always bind the TransactionInfo to the thread, even if we didn't create
		// a new transaction here. This guarantees that the TransactionInfo stack
		// will be managed correctly even if no transaction was created by this aspect.
		txInfo.bindToThread();
		return txInfo;
}
```





##### 回滚处理

```java
// TransactionAspectSupport 类
protected void completeTransactionAfterThrowing(@Nullable TransactionInfo txInfo, Throwable ex) {
	    // 当抛出异常时首先判断当前是否存在事务，这是基础依据
		if (txInfo != null && txInfo.getTransactionStatus() != null) {
			if (logger.isTraceEnabled()) {
				logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() +
						"] after exception: " + ex);
			}
            // 这里判断是否回滚默认的依据是抛出的异常是否是 RuntimeException 或者是 Error 的类型
			if (txInfo.transactionAttribute != null && txInfo.transactionAttribute.rollbackOn(ex)) {
				try {
                    // 根据 Transactionstatus 信息进行回滚处理
					txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
				}
				catch (TransactionSystemException ex2) {
					logger.error("Application exception overridden by rollback exception", ex);
					ex2.initApplicationException(ex);
					throw ex2;
				}
				catch (RuntimeException | Error ex2) {
					logger.error("Application exception overridden by rollback exception", ex);
					throw ex2;
				}
			}
			else {
				// We don't roll back on this exception.
				// Will still roll back if TransactionStatus.isRollbackOnly() is true.
                // 如果不满足回滚条件即使抛出异常也同样会提交
				try {
					txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
				}
				catch (TransactionSystemException ex2) {
					logger.error("Application exception overridden by commit exception", ex);
					ex2.initApplicationException(ex);
					throw ex2;
				}
				catch (RuntimeException | Error ex2) {
					logger.error("Application exception overridden by commit exception", ex);
					throw ex2;
				}
			}
		}
}
```

```java
// DefaultTransactionAttribute 类
public boolean rollbackOn(Throwable ex) {
		return (ex instanceof RuntimeException || ex instanceof Error);
}
```

```java
// AbstractPlatformTransactionManager 类
public final void rollback(TransactionStatus status) throws TransactionException {
	    // 如果事务已经完成,那么再次回滚会抛出异常
		if (status.isCompleted()) {
			throw new IllegalTransactionStateException(
					"Transaction is already completed - do not call commit or rollback more than once per transaction");
		}

		DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
		processRollback(defStatus, false);
}

private void processRollback(DefaultTransactionStatus status, boolean unexpected) {
		try {
			boolean unexpectedRollback = unexpected;

			try {
                // 激活所有 TransactionSynchronization 中对应的方法
				triggerBeforeCompletion(status);

				if (status.hasSavepoint()) {
					if (status.isDebug()) {
						logger.debug("Rolling back transaction to savepoint");
					}
                    // 如果有保存点．也就是当前事务为单独的线程则会退到保存点
                    // 当之前已经保存的事务信息中有保存点信息的时候，使用保存点信息进行回滚。常用于嵌入式事务，对于嵌入式的事务的处理，内嵌的事务异常并						不会引起外部事务的回滚。
					status.rollbackToHeldSavepoint();
				}
				else if (status.isNewTransaction()) {
					if (status.isDebug()) {
						logger.debug("Initiating transaction rollback");
					}
                    // 如果当前事务为独立的新事务，则直接回退
                    // 当之前已经保存的事务信息中的事务为新事务，那么直接回滚。常用于单独事务的处理。对于没有保存点的回滚，Spring 同样是使用底层数据库连						接提供的API来操作的。
					doRollback(status);
				}
				else {
					// Participating in larger transaction
					if (status.hasTransaction()) {
						if (status.isLocalRollbackOnly() || isGlobalRollbackOnParticipationFailure()) {
							if (status.isDebug()) {
								logger.debug("Participating transaction failed - marking existing transaction as rollback-only");
							}
                            // 如果当前事务不是独立的事务，那么只能标记状态，等到事务链执行完毕后统一回滚
                            // 当前事务信息中表明是存在事务的，又不属于以上两种情况，多数用于JTA，只做回滚标识，等到提交的时候统一不提交。
							doSetRollbackOnly(status);
						}
						else {
							if (status.isDebug()) {
								logger.debug("Participating transaction failed - letting transaction originator decide on rollback");
							}
						}
					}
					else {
						logger.debug("Should roll back transaction but cannot - no transaction available");
					}
					// Unexpected rollback only matters here if we're asked to fail early
					if (!isFailEarlyOnGlobalRollbackOnly()) {
						unexpectedRollback = false;
					}
				}
			}
			catch (RuntimeException | Error ex) {
				triggerAfterCompletion(status, TransactionSynchronization.STATUS_UNKNOWN);
				throw ex;
			}
			// 激活所有 TransactionSynchronization 中对应的方法
			triggerAfterCompletion(status, TransactionSynchronization.STATUS_ROLLED_BACK);

			// Raise UnexpectedRollbackException if we had a global rollback-only marker
			if (unexpectedRollback) {
				throw new UnexpectedRollbackException(
						"Transaction rolled back because it has been marked as rollback-only");
			}
		}
		finally {
            // 清空记录的资源并将挂起的资源恢复
			cleanupAfterCompletion(status);
		}
}
```

```java
// AbstractTransactionStatus 类
public void rollbackToHeldSavepoint() throws TransactionException {
		Object savepoint = getSavepoint();
		if (savepoint == null) {
			throw new TransactionUsageException(
					"Cannot roll back to savepoint - no savepoint associated with current transaction");
		}
		getSavepointManager().rollbackToSavepoint(savepoint);
		getSavepointManager().releaseSavepoint(savepoint);
		setSavepoint(null);
}
```

```java
// JdbcTransactionObjectSupport 类
public void rollbackToSavepoint(Object savepoint) throws TransactionException {
		ConnectionHolder conHolder = getConnectionHolderForSavepoint();
		try {
			conHolder.getConnection().rollback((Savepoint) savepoint);
			conHolder.resetRollbackOnly();
		}
		catch (Throwable ex) {
			throw new TransactionSystemException("Could not roll back to JDBC savepoint", ex);
		}
}
```

```java
// DataSourceTransactionManager 类
protected void doRollback(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.rollback();
		}
		catch (SQLException ex) {
			throw translateException("JDBC rollback", ex);
		}
}
```

```java
// AbstractPlatformTransactionManager 类
private void cleanupAfterCompletion(DefaultTransactionStatus status) {
	    // 设置完成状态
		status.setCompleted();
		if (status.isNewSynchronization()) {
			TransactionSynchronizationManager.clear();
		}
		if (status.isNewTransaction()) {
			doCleanupAfterCompletion(status.getTransaction());
		}
		if (status.getSuspendedResources() != null) {
			if (status.isDebug()) {
				logger.debug("Resuming suspended transaction after completion of inner transaction");
			}
			Object transaction = (status.hasTransaction() ? status.getTransaction() : null);
            // 结束之前事务的挂起状态
			resume(transaction, (SuspendedResourcesHolder) status.getSuspendedResources());
		}
}
```

```java
// DataSourceTransactionManager 类
protected void doCleanupAfterCompletion(Object transaction) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;

		// Remove the connection holder from the thread, if exposed.
		if (txObject.isNewConnectionHolder()) {
            // 将数据库连接从当前线程中解除绑定
			TransactionSynchronizationManager.unbindResource(obtainDataSource());
		}

		// Reset connection.
	    // 释放链接
		Connection con = txObject.getConnectionHolder().getConnection();
		try {
			if (txObject.isMustRestoreAutoCommit()) {
                // 恢复数据库连接的自动提交属性
				con.setAutoCommit(true);
			}
            // 重置数据库连接
			DataSourceUtils.resetConnectionAfterTransaction(
					con, txObject.getPreviousIsolationLevel(), txObject.isReadOnly());
		}
		catch (Throwable ex) {
			logger.debug("Could not reset JDBC Connection after transaction", ex);
		}

		if (txObject.isNewConnectionHolder()) {
			if (logger.isDebugEnabled()) {
				logger.debug("Releasing JDBC Connection [" + con + "] after transaction");
			}
            // 如果当前事务时独立的新创建的事务则在事务完成时释放数据库连接
			DataSourceUtils.releaseConnection(con, this.dataSource);
		}

		txObject.getConnectionHolder().clear();
}
```



##### 事务提交

```java
// TransactionAspectSupport 类
protected void commitTransactionAfterReturning(@Nullable TransactionInfo txInfo) {
    	/**
    		某个事务是另一个事务的嵌入事务，但是，这些事务又不在 Spring 的管理范围内，或者无法设置保存点，那么、Spring 会通过设置回滚标识的方式来禁止提交。首先当某个嵌入事务发生回滚的时候会设置回滚标识，而等到外部事务提交时，一旦判断出当前事务流被设置了回滚标识，则由外部事务来统一进行整体事务的回滚。
    	*/
		if (txInfo != null && txInfo.getTransactionStatus() != null) {
			if (logger.isTraceEnabled()) {
				logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() + "]");
			}
			txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
		}
}
```

```java
// AbstractPlatformTransactionManager 类
public final void commit(TransactionStatus status) throws TransactionException {
		if (status.isCompleted()) {
			throw new IllegalTransactionStateException(
					"Transaction is already completed - do not call commit or rollback more than once per transaction");
		}

		DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
	    // 如果在事务链中已经被标记回滚，那么不会尝试提交事务，直接回滚
		if (defStatus.isLocalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Transactional code has requested rollback");
			}
			processRollback(defStatus, false);
			return;
		}

		if (!shouldCommitOnGlobalRollbackOnly() && defStatus.isGlobalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Global transaction is marked as rollback-only but transactional code requested commit");
			}
			processRollback(defStatus, true);
			return;
		}
	    // 处理事务提交
		processCommit(defStatus);
}
```

```java
private void processCommit(DefaultTransactionStatus status) throws TransactionException {
		try {
			boolean beforeCompletionInvoked = false;

			try {
				boolean unexpectedRollback = false;
                // 预留
				prepareForCommit(status);
                //添加的 TransactionSynchronization 中的对应方法的调用
				triggerBeforeCommit(status);
                //添加的 Transactionsynchronization 中的对应方法的调用
				triggerBeforeCompletion(status);
				beforeCompletionInvoked = true;

                /**
                	此条件主要考虑内嵌事务的情况，对于内嵌事务，在Spring 中正常的处理方式是将内嵌事务开始之前设置保存点，一旦内嵌事务出现异常便根据保存					点信息进行回滚，但是如果没有出现异常，内嵌事务并不会单独提交，而是根据事务流由最外层事务负责提交，所以如果当前存在保存点信息便不是最					 外层事务，不做保存操作，对于是否是新事务的判断也是基于此考虑
                */
				if (status.hasSavepoint()) {
					if (status.isDebug()) {
						logger.debug("Releasing transaction savepoint");
					}
					unexpectedRollback = status.isGlobalRollbackOnly();
                    // 如果存在保存点则清除保存点信息
					status.releaseHeldSavepoint();
				}
				else if (status.isNewTransaction()) {
					if (status.isDebug()) {
						logger.debug("Initiating transaction commit");
					}
					unexpectedRollback = status.isGlobalRollbackOnly();
                    // 如果是独立的事务则直接提交
					doCommit(status);
				}
				else if (isFailEarlyOnGlobalRollbackOnly()) {
					unexpectedRollback = status.isGlobalRollbackOnly();
				}

				// Throw UnexpectedRollbackException if we have a global rollback-only
				// marker but still didn't get a corresponding exception from commit.
				if (unexpectedRollback) {
					throw new UnexpectedRollbackException(
							"Transaction silently rolled back because it has been marked as rollback-only");
				}
			}
			catch (UnexpectedRollbackException ex) {
				// can only be caused by doCommit
				triggerAfterCompletion(status, TransactionSynchronization.STATUS_ROLLED_BACK);
				throw ex;
			}
			catch (TransactionException ex) {
				// can only be caused by doCommit
				if (isRollbackOnCommitFailure()) {
					doRollbackOnCommitException(status, ex);
				}
				else {
					triggerAfterCompletion(status, TransactionSynchronization.STATUS_UNKNOWN);
				}
				throw ex;
			}
			catch (RuntimeException | Error ex) {
				if (!beforeCompletionInvoked) {
                    // 添加的 Transactionsynchronization 中的对应方法的调用
					triggerBeforeCompletion(status);
				}
                // 提交过程中出现异常则回滚
				doRollbackOnCommitException(status, ex);
				throw ex;
			}

			// Trigger afterCommit callbacks, with an exception thrown there
			// propagated to callers but the transaction still considered as committed.
			try {
                // 添加的 TransactionSynchronization 中的对应方法的调用
				triggerAfterCommit(status);
			}
			finally {
				triggerAfterCompletion(status, TransactionSynchronization.STATUS_COMMITTED);
			}

		}
		finally {
			cleanupAfterCompletion(status);
		}
}
```

```java
// 
protected void doCommit(DefaultTransactionStatus status) {
		DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Committing JDBC transaction on Connection [" + con + "]");
		}
		try {
			con.commit();
		}
		catch (SQLException ex) {
			throw translateException("JDBC commit", ex);
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



## 源码

### ContextLoaderlistener

​		当使用编程方式的时候可以直接将`Spring`配置信息作为参数传入`Spring`容器中，但是在`Web`下，更多的是与`Web`环境相互结合，通常的办法是将路径以

`context-param`的方式注册并使用`ContextLoaderListener`进行监听读取。

​		`ContextLoaderListener`的作用就是启动`Web`容器时，自动装配`ApplicationContext`的配置信息。因为它实现了`ServletContextListener`接口，开发者能够

在为客户端请求提就会默认执行它实现的方法，使用`ServletContextListener`接口，开发者能够在为客户端请求提供服务之前向`ServletContext`中添加任意的对

象。这个对象在`ServletContext`启动的时候被初始化，然后在`ServletContext`整个运行期间都是可见的。

​		每一个`Web`应用都有一个`ServletContext`与之相关联。`ServletContext`对象在应用启动时被创建，在应用关闭的时候被销毁。`ServletContext`在全局范围

内有效，类似于应用中的一个全局变量。在`ServletContextListener`中的核心逻辑便是初始化`WebApplicationContext`实例并存放至`ServletContext`中。

```java
// ContextLoaderListener 类
public void contextInitialized(ServletContextEvent event) {
		initWebApplicationContext(event.getServletContext());
}
```

```java
// ContextLoader 类
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    	// 存在多次 ContextLoader 定义
		if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
			throw new IllegalStateException(
					"Cannot initialize context because there is already a root application context present - " +
					"check whether you have multiple ContextLoader* definitions in your web.xml!");
		}

		servletContext.log("Initializing Spring root WebApplicationContext");
		Log logger = LogFactory.getLog(ContextLoader.class);
		if (logger.isInfoEnabled()) {
			logger.info("Root WebApplicationContext: initialization started");
		}
		long startTime = System.currentTimeMillis();

		try {
			// Store context in local instance variable, to guarantee that
			// it is available on ServletContext shutdown.
			if (this.context == null) {
                // 初始化 context
				this.context = createWebApplicationContext(servletContext);
			}
			if (this.context instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent ->
						// determine parent for root web application context, if any.
						ApplicationContext parent = loadParentContext(servletContext);
						cwac.setParent(parent);
					}
					configureAndRefreshWebApplicationContext(cwac, servletContext);
				}
			}
            // 记录在 servletContext 中
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

			ClassLoader ccl = Thread.currentThread().getContextClassLoader();
			if (ccl == ContextLoader.class.getClassLoader()) {
				currentContext = this.context;
			}
			else if (ccl != null) {
                // 映射当前的类加载器与创建的实例到全局变量 currentContextPerThread 中
				currentContextPerThread.put(ccl, this.context);
			}

			if (logger.isInfoEnabled()) {
				long elapsedTime = System.currentTimeMillis() - startTime;
				logger.info("Root WebApplicationContext initialized in " + elapsedTime + " ms");
			}

			return this.context;
		}
		catch (RuntimeException | Error ex) {
			logger.error("Context initialization failed", ex);
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
			throw ex;
		}
}
```

```java
// ContextLoader 类
static {
		// Load default strategy implementations from properties file.
		// This is currently strictly internal and not meant to be customized
		// by application developers.
	    // 读取 ContextLoader 类的同目录下的属性文件 ContextLoader.properties
		try {
			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, ContextLoader.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Could not load 'ContextLoader.properties': " + ex.getMessage());
		}
}

protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
		Class<?> contextClass = determineContextClass(sc);
		if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
			throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
					"] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
		}
	    // 根据这个实现类通过反射的方式进行实例的创建
		return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}

protected Class<?> determineContextClass(ServletContext servletContext) {
		String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
		if (contextClassName != null) {
			try {
				return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
			}
			catch (ClassNotFoundException ex) {
				throw new ApplicationContextException(
						"Failed to load custom context class [" + contextClassName + "]", ex);
			}
		}
		else {
            // 提取将要实现 WebApplicationContext 接口的实现类
			contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
			try {
				return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
			}
			catch (ClassNotFoundException ex) {
				throw new ApplicationContextException(
						"Failed to load default context class [" + contextClassName + "]", ex);
			}
		}
}
```



### DispatcherServlet

​		在`Spring`中，`ContextLoaderListener`只是辅助功能，用于创建`WebApplicationContext`类型实例，而真正的逻辑实现其实是在`DispatcherServlet`中进行

的，`DispatcherServlet`是实现`servlet`接口的实现类。

​		`servlet`的生命周期是由`servlet`的容器来控制的，它可以分为`3`个阶段：初始化、运行和销毁。

​		初始化阶段：

​				`servlet`容器加载`servlet`类，把`servlet`类的`class`文件中的数据读到内存中。

​				`servlet`容器创建一个`ServletConfig`对象。`ServletConfig`对象包含了`servlet`的初始化配置信息。

​				`servlet`容器创建一个`servlet`对象。

​				`servlet`容器调用`servlet`对象的`init`方法进行初始化。

​		运行阶段：

​				当`servlet`容器接收到一个请求时，`servlet`容器会针对这个请求创建`servletRequest`和`servletResponse`对象，然后调用`service`方法。并把这两个

​		参数传递给`service`方法。`service`方法通过`servletRequest`对象获得请求的信息。并处理该请求。再通过`servletResponse`对象生成这个请求的响应结

​		果。然后销毁`servletRequest`和`servletResponse`对象。我们不管这个请求是`post`提交的还是`get`提交的，最终这个请求都会由`service`方法来处理。

​		销毁阶段：

​				当`Web`应用被终止时，`servlet`容器会先调用`servlet`对象的`destrory`方法，然后再销毁`servlet`对象，同时也会销毁与`servlet`对象相关联的

​		`servletConfig`对象。



#### DispatcherServlet 的初始化

```java
// DispatcherServlet 的父类 HttpServletBean 类
public final void init() throws ServletException {

		// Set bean properties from init parameters.
	    // 解析 init-param 并封装至pvs中，FrameworkServlet 类中包含对应的同名属性
		PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
		if (!pvs.isEmpty()) {
			try {
                // 将当前的这个 servlet 类转化为一个 BeanWrapper,从而能够以 Spring 的方式来对 init-param 的值进行注入
				BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
				ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
                // 注册自定义属性编辑器，一旦遇到 Resource 类型的属性将会使用 ResourceEditor 进行解析
				bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
                // 空实现,留给子类覆盖
				initBeanWrapper(bw);
                // 属性注入
				bw.setPropertyValues(pvs, true);
			}
			catch (BeansException ex) {
				if (logger.isErrorEnabled()) {
					logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
				}
				throw ex;
			}
		}

		// Let subclasses do whatever initialization they like.
	    // 留给子类扩展，对 WebApplicationContext 进行补充初始化
		initServletBean();
}
```



##### 封装及验证初始化参数

```java
// HttpServletBean#ServletConfigPropertyValues 类
public ServletConfigPropertyValues(ServletConfig config, Set<String> requiredProperties)
				throws ServletException {

			Set<String> missingProps = (!CollectionUtils.isEmpty(requiredProperties) ?
					new HashSet<>(requiredProperties) : null);

			Enumeration<String> paramNames = config.getInitParameterNames();
			while (paramNames.hasMoreElements()) {
				String property = paramNames.nextElement();
				Object value = config.getInitParameter(property);
				addPropertyValue(new PropertyValue(property, value));
				if (missingProps != null) {
					missingProps.remove(property);
				}
			}

			// Fail if we are still missing properties.
			if (!CollectionUtils.isEmpty(missingProps)) {
				throw new ServletException(
						"Initialization from ServletConfig for servlet '" + config.getServletName() +
						"' failed; the following required properties were missing: " +
						StringUtils.collectionToDelimitedString(missingProps, ", "));
			}
}
```



##### ServletBean 的初始化

```java
// FrameworkServlet 类
protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
		if (logger.isInfoEnabled()) {
			logger.info("Initializing Servlet '" + getServletName() + "'");
		}
		long startTime = System.currentTimeMillis();

		try {
			this.webApplicationContext = initWebApplicationContext();
            // 设计为子类覆盖
			initFrameworkServlet();
		}
		catch (ServletException | RuntimeException ex) {
			logger.error("Context initialization failed", ex);
			throw ex;
		}

		if (logger.isDebugEnabled()) {
			String value = this.enableLoggingRequestDetails ?
					"shown which may lead to unsafe logging of potentially sensitive data" :
					"masked to prevent unsafe logging of potentially sensitive data";
			logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
					"': request parameters and headers will be " + value);
		}

		if (logger.isInfoEnabled()) {
			logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
		}
}
```



#### WebApplicationContext 的初始化

```java
// FrameworkServlet 类
protected WebApplicationContext initWebApplicationContext() {
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		WebApplicationContext wac = null;

		if (this.webApplicationContext != null) {
			// A context instance was injected at construction time -> use it
            // context 实例在构造函数中被注入
			wac = this.webApplicationContext;
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent -> set
						// the root application context (if any; may be null) as the parent
						cwac.setParent(rootContext);
					}
                    // 刷新上下文环境
                    // 使用公共父类 AbstractApplicationContext 提供的 refresh() 进行配置文件加载
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		if (wac == null) {
			// No context instance was injected at construction time -> see if one
			// has been registered in the servlet context. If one exists, it is assumed
			// that the parent context (if any) has already been set and that the
			// user has performed any initialization such as setting the context id
            // 根据 contextAttribute 属性加载 webApplicationContext
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			// No context instance is defined for this servlet -> create a local one
			wac = createWebApplicationContext(rootContext);
		}

		if (!this.refreshEventReceived) {
			// Either the context is not a ConfigurableApplicationContext with refresh
			// support or the context injected at construction time had already been
			// refreshed -> trigger initial onRefresh manually here.
			synchronized (this.onRefreshMonitor) {
				onRefresh(wac);
			}
		}

		if (this.publishContext) {
			// Publish the context as a servlet context attribute.
			String attrName = getServletContextAttributeName();
			getServletContext().setAttribute(attrName, wac);
		}

		return wac;
}
```

```java
// DispatcherServlet 类
protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    	// 初始化 MultipartResolver，除妖用于处理文件上传
		initMultipartResolver(context);
    	// 初始化 LocaleResolver，国际化配置
		initLocaleResolver(context);
        // 初始化 ThemeResolver，主题配置
		initThemeResolver(context);
	    // 初始化 HandlerMappings，当客户端发出请求时，DispatcherServlet 会将请求提交给 HandlerMapping，HandlerMapping 根据配置来回传给 				DispatcherServlet 相应的 Controller
		initHandlerMappings(context);
	    // 初始化 HandlerAdapters，HandlerAdapters 通过适配器模式寻找合适的控制器进行处理
		initHandlerAdapters(context);
    	// 初始化 HandlerExceptionResolvers，处理异常，实现 HandlerExceptionResolver 接口的 bean ，会返回一个 ModelAndView 。Spring 会搜索容器		中所有实现了 HandlerExceptionResolver 接口的 bean 逐个执行，知道某一个返回一个 ModelAndView 对象
		initHandlerExceptionResolvers(context);
    	// 初始化 RequestToViewNameTranslator，当 Controller 处理器方法没有返回一个 View 对象或逻辑视图名称,并且在该方法中没有直接往 response 的输			出流里面写数据的时候，Spring 就会采用约定好的方式提供一个逻辑视图名称。这个逻辑视图名称是通过 Spring 定义的 					  				org.Springframework.web.servlet.RequestToViewNameTranslator接口的getViewName方法来实现的
		initRequestToViewNameTranslator(context);
	    // 初始化 ViewResolvers，当 Controller 将请求处理结果放入到 ModelAndView 中以后，DispatcherServlet 会根据 ModelAndView 选择合适的视图进			行渲染。ViewResolver 接口定义了resolverViewName方法，根据viewName创建合适类型的 View实现
		initViewResolvers(context);
	    // 初始化 FlashMapManager，SpringMVC Flash attributes提供了一个请求存储属性，可供其他请求使用。在使用重定向时候非常必要，Flash attributes			  在重定向之前暂存（就像存在session中）以便重定向之后还能使用，并立即删除。FlashMap 用于保持 flash attributes ,而 FlashMapManager 用于存			  储、检索、管理FlashMap实例
		initFlashMapManager(context);
}

private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;

		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
           	// 加载系统中所有实现了 HandlerMapping 接口的 bean
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
		}
		else {
            // 配置了 detectAllHandlerMappings 参数为 false，即只加载 name 为 handlerMapping 的 bean
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}

		// Ensure we have at least one HandlerMapping, by registering
		// a default HandlerMapping if no other mappings are found.
    	// 没有找到时，将在类 DispatcherServlet 所在的目录下的 DispatcherServlet.properties 中定义的 org.springframework.web.servlet.HandlerMapping 的内容来加载默认的 handlerMapping
		if (this.handlerMappings == null) {
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}

		for (HandlerMapping mapping : this.handlerMappings) {
			if (mapping.usesPathPatterns()) {
				this.parseRequestPath = true;
				break;
			}
		}
}
```



#### DispatcherServlet 的逻辑处理

```java
// FrameworkServlet 类
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
}

protected final void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
}

protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
    	// 记录当前时间，用于计算 web 请求的处理时间
		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;

		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		// 为了保证当前线程的 LocaleContext 以及 RequestAttributes 可以在当前请求后还能恢复，提取当前线程的两个属性
    	// 根据当前 request 创建对应的 LocaleContext 和 RequestAttributes
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
		// 绑定到当前线程
		initContextHolders(request, localeContext, requestAttributes);

		try {
			// 委托给 doService 方法进一步处理
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}
		finally {
			// 请求处理结束后恢复线程到原始状态
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
           	// 请求处理结束后无论成功与否发布事件通知
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
}
```

```java
// DispatcherServlet 类
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
    	// 设置功能辅助工具变量
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		RequestPath previousRequestPath = null;
		if (this.parseRequestPath) {
			previousRequestPath = (RequestPath) request.getAttribute(ServletRequestPathUtils.PATH_ATTRIBUTE);
			ServletRequestPathUtils.parseAndCache(request);
		}

		try {
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
			if (this.parseRequestPath) {
				ServletRequestPathUtils.setParsedRequestPath(previousRequestPath, request);
			}
		}
}

protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				// 如果是 MultipartContent 类型的 request 则转换 request 为 MultipartHttpServletRequest 类型的 request
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
                 // 根据 request 信息寻找对应的 Handler
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
                    // 如果没有找到对应的 handler 则通过 response 反馈错误信息
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
                // 根据当前的 handler 寻找对应的 HandlerAdapter
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = HttpMethod.GET.matches(method);
                // 如果当前 handler 支持 last-modified 头处理
                // Last-Modified 缓存机制。在客户端第一次输入 URL 时，服务器端会返回内容和状态码 200，表示请求成功，同时会添加一个 Last-Modified 的					响应头，表示此文件在服务器上的最后更新时间，客户端第二次请求此 URL 时，客户端会向服务器发送请求头 If-Modified-Since，询问服务器该				  时间之后当前请求内容是否有被修改过，如果服务器端的内容没有变化，则自动返回 HTTP 304 状态码（只要响应头，内容为空，这样就节省了网络带					宽)。
				if (isGet || HttpMethod.HEAD.matches(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
				// 拦截器的 preHandler 方法的调用
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
                // 真正的激活 handler 并返回视图
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}
				// 视图名称转换应用于需要添加前缀后缀的情况
				applyDefaultViewName(processedRequest, mv);
				// 应用所有拦截器的 postHandle 方法
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            // 如果在 Handler 实例的处理中返回了 view，那么需要做页面的处理
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;
		// 异常视图的处理
		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
            // 处理页面跳转
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		if (mappedHandler != null) {
			// Exception (if any) is already handled..
            // 完成处理激活触发器
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
}
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
    void *ptr; // 指向实际的数据结构
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
// sds.h
typedef char *sds;

/** __attribute__ ((__packed__)) 取消结构体内的字节对齐以节省内存
	buf 数组并没有指定数组长度，使用的是柔性数组，即结构体中最后一个属性可以被定义为一个大小可变的数组，使用 sizeof 函数计算结构体大小时，返回结构不包含数组占用的内存
*/
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; // 低 3 位代表 sdshdr 的类型，高 5 位代表字符串长度，这个类型定义的是常量字符串，不支持扩容，所以没有 alloc 属性
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; // 已使用字节长度，即字符串长度，最大长度为 2^8 - 1，Redis规定字符串长度最大不能超过 512M，由于该属性记录了字符串长度，所以字符串中可以存放空字符 '\0'
    uint8_t alloc; // 已申请字节长度，即 sds 总长度，alloc - len 为可用（空闲）空间
    unsigned char flags; // 低 3 位代表 sdshdr 的类型
    char buf[]; // 字符串内容，其遵循 C 语言的规范，保存一个空字符作为 buf 的结尾并且不计入 len、alloc 属性，所以直接可以使用 strcmp、strcpy 等函数
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



#### 构建字符串

```c++
// sds.c
sds sdsnewlen(const void *init, size_t initlen) {
    return _sdsnewlen(init, initlen, 0);
}
// init：字符串内容；initlen：字符串长度
sds _sdsnewlen(const void *init, size_t initlen, int trymalloc) {
    void *sh;
    sds s;
    // 根据字符串长度，判断对应的 sdshdr 类型
    char type = sdsReqType(initlen);

    // 长度为 0 的字符串通常需要扩容，使用 sdshdr 8
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
    
    // sdsHdrSize 查询 sdshdr 结构体的长度
    int hdrlen = sdsHdrSize(type);
    unsigned char *fp; /* flags pointer. */
    size_t usable;

    assert(initlen + hdrlen + 1 > initlen); /* Catch size_t overflow */
    // 申请内存空间，hdrlen：sdshdr 结构体长度（不包含 buf 数组）；initlen：字符串内容长度；1：存放空字符 '\0'
    sh = trymalloc ? s_trymalloc_usable(hdrlen + initlen + 1, &usable) : s_malloc_usable(hdrlen + initlen + 1, &usable);
    if (sh == NULL) return NULL;
    if (init==SDS_NOINIT)
        init = NULL;
    else if (!init)
        memset(sh, 0, hdrlen+initlen+1);
    
    // 给 sdshdr 属性赋值
    s = (char*)sh+hdrlen;
    fp = ((unsigned char*)s)-1;
    usable = usable-hdrlen-1;
    if (usable > sdsTypeMaxSize(type))
        usable = sdsTypeMaxSize(type);
    switch(type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            // SDS_HDR_VAR 将 sh 指针转化为对应的 sdshdr 结构体指针
            SDS_HDR_VAR(8,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64,s);
            sh->len = initlen;
            sh->alloc = usable;
            *fp = type;
            break;
        }
    }
    if (initlen && init)
        memcpy(s, init, initlen);
    s[initlen] = '\0';
    
    // sds 实际上 char* 的别名，s 指针指向 sdshdr.buf 属性，即字符串内容，Redis 通过该指针可以直接读/写字符串数据
    return s;
}
```



#### 扩容

```c++
// sds.c
// addlen：要求扩容后可用长度 (alloc - len)大于该参数
sds sdsMakeRoomFor(sds s, size_t addlen) {
    void *sh, *newsh;
    // 获取当前可用空间长度，如果当前可用空间长度满足要求，直接返回
    size_t avail = sdsavail(s);
    size_t len, newlen, reqlen;
    char type, oldtype = s[-1] & SDS_TYPE_MASK;
    int hdrlen;
    size_t usable;

    /* Return ASAP if there is enough space left. */
    if (avail >= addlen) return s;
	
    // sdslen：获取字符串长度，len 为原 sds 字符串长度；newlen 为新 sds 长度；sh 指向原 sds 的 sdshdr 结构体
    len = sdslen(s);
    sh = (char*)s-sdsHdrSize(oldtype);
    reqlen = newlen = (len+addlen);
    assert(newlen > len);   /* Catch size_t overflow */
    
    // 预分配比参数要求多的内存空间，避免每次扩容都要进行内存拷贝操作
    // 新的 sds 长度如果小于 SDS_MAX_PREALLOC（1024 x 1024 字节），则扩大两倍，否则增加 SDS_MAX_PREALLOC
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;

    // sdsReqType：计算新的 sdshdr 类型
    type = sdsReqType(newlen);

    /* Don't use type 5: the user is appending to the string and type 5 is
     * not able to remember empty space, so sdsMakeRoomFor() must be called
     * at every appending operation. */
    // 扩容后不使用 sdshdr5
    if (type == SDS_TYPE_5) type = SDS_TYPE_8;
	
    // 如果扩容后 sds 还是同一类型，则直接申请内存，否则，由于 sds 结构已经变动，必须移动整个 sds，直接分配新的内容空间，并将原来的字符串内容复制到新的内存空间
    hdrlen = sdsHdrSize(type);
    assert(hdrlen + newlen + 1 > reqlen);  /* Catch size_t overflow */
    if (oldtype==type) {
        newsh = s_realloc_usable(sh, hdrlen+newlen+1, &usable);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+hdrlen;
    } else {
        /* Since the header size changes, need to move the string forward,
         * and can't use realloc */
        newsh = s_malloc_usable(hdrlen+newlen+1, &usable);
        if (newsh == NULL) return NULL;
        memcpy((char*)newsh+hdrlen, s, len+1);
        s_free(sh);
        s = (char*)newsh+hdrlen;
        s[-1] = type;
        sdssetlen(s, len);
    }
    usable = usable-hdrlen-1;
    if (usable > sdsTypeMaxSize(type))
        usable = sdsTypeMaxSize(type);
    
    // 更新 sdshdr.alloc 属性
    sdssetalloc(s, usable);
    return s;
}
```

​		`Redis`字符串支持二进制安全，可用将输入存储为没有特定格式意义的原始数据流，因此`Redis`字符串可以存储任何数据。



#### 常用函数

|                 函数                  |                            作用                            |
| :-----------------------------------: | :--------------------------------------------------------: |
|           sdsnew，sdsempty            |                          创建 sds                          |
| sdsfree，sdsclear，sdsRemoveFreeSpace | 释放 sds，清空 sds 中的字符串内容，移除 sds 剩余的可用空间 |
|                sdslen                 |                    获取 sds 字符串长度                     |
|                 sdsup                 |          将给定字符串复制到 sds 中，覆盖原字符串           |
|                sdscat                 |            将给定字符串拼接到 sds 字符串内容后             |
|                sdscmp                 |                对比两个 sds 字符串是否相同                 |
|               sdstrange               |        获取子字符串，不在指定范围内的字符串将被清除        |



#### 编码

​		字符串有 3 中编码：

​				`OBJ_ENCODING_EMBSTR`：长度小于或等于`OBJ_ENCODING_EMBSTR_SIZE_LIMIT(44`字节`)`的字符串。该编码是`Redis`针对短字符串的优化。其内存申请和释

​		放都只需要调用一次内存操作函数。并且`redisObject`和`sdshdr`结构保存在一块连续的内存中，减少了内存碎片。

![](image/QQ截图20220406140809.png)

​				`OBJ_ENCODING_RAW`：长度大于`OBJ_ENCODING_EMBSTR_SIZE_LIMIT`的字符串，`redisObject`和`sdshdr`结构存放在两个不连续的内存块中。

​				`OBJ_ENCODING_INT`：将数值型字符串转换为整型，可以大幅江都数据占用的内存空间。



​		向`Redis`发送一个请求后，`Redis`会解析请求报文，并将命令、参数转化为`redisObject`：

```c++
// object.c
robj *createStringObject(const char *ptr, size_t len) {
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr,len);
    else
        return createRawStringObject(ptr,len);
}
```

​		根据字符串长度，将`encoding`转化为`OBJ_ENCODING_EMBSTR`或`OBJ_ENCODING_RAW`的`redisObject`，然后`Redis`再将`redisObject`存入数据库。

​		转为`OBJ_ENCODING_INT`：

```c++
// object.c
robj *tryObjectEncoding(robj *o) {
    long value;
    sds s = o->ptr;
    size_t len;

    serverAssertWithInfo(NULL,o,o->type == OBJ_STRING);

    if (!sdsEncodedObject(o)) return o;
	// 如果被多处引用，不能进行编码操作，否则会影响其他地方的正常运行
    if (o->refcount > 1) return o;

    len = sdslen(s);
    // 如果字符串长度小于或等于 20 ，则调用 string2l 函数尝试将其转换为 long long 类型，如果成功返回 1，long long 最大保存长度为 20 的数值型字符串（19 位数值 + 1 位符号位）
    if (len <= 20 && string2l(s,len,&value)) {
        // 首先尝试使用 shared.integers 中的共享数据，避免重复创建相同数据对象而浪费内存。shared.integers 是一个整数数组，存放了 0 ~ 9999
        /*
        	如果配置了 server.maxmemory ，并使用了不支持共享数据的淘汰算法（LRU，LFU），将不能使用共享数据，此时每个数据中心都必须存在一个 					redisObject.lru 属性，这些算法才能正常工作
        */
        if ((server.maxmemory == 0 ||
            !(server.maxmemory_policy & MAXMEMORY_FLAG_NO_SHARED_INTEGERS)) &&
            value >= 0 &&
            value < OBJ_SHARED_INTEGERS)
        {
            decrRefCount(o);
            incrRefCount(shared.integers[value]);
            return shared.integers[value];
        } else {
            // 如果不能使用共享数据并且原编码格式为 OBJ_ENCODING_RAW，则将 redisObject.ptr 原来的 sds 类型替换为字符串转换后的数值
            if (o->encoding == OBJ_ENCODING_RAW) {
                sdsfree(o->ptr);
                o->encoding = OBJ_ENCODING_INT;
                o->ptr = (void*) value;
                return o;
            } else if (o->encoding == OBJ_ENCODING_EMBSTR) {
                // 如果不能使用共享数据并且原编码格式为 OBJ_ENCODING_EMBSTR，由于 redisObject、sds 存在在同一个内存块中，无法直接替换 							redisObject.ptr ，所以调用 createStringObjectFromLongLongForValue 函数创建一个新的 redisObject，编码为 									OBJ_ENCODING_INT，redisObject.ptr 指向 long long 类型或 long 类型
                decrRefCount(o);
                return createStringObjectFromLongLongForValue(value);
            }
        }
    }
	// 此时说明字符串不能转换为 OBJ_ENCODING_INT 编码，尝试将其转换为 OBJ_ENCODING_EMBSTR 编码
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT) {
        robj *emb;

        if (o->encoding == OBJ_ENCODING_EMBSTR) return o;
        emb = createEmbeddedStringObject(s,sdslen(s));
        decrRefCount(o);
        return emb;
    }
	// 此时说明字符串只能使用 OBJ_ENCODING_RAW编码，尝试释放 sds 中剩余的可用空间
    trimStringObjectIfNeeded(o);

    return o;
}
```

​		`server.c/redisCommandTable`定义了每个`Redis`命令与对应的处理函数。

​		通过`TYPE`命令查看数据对象类型，`OBJECT ENCODING`命令查看编码。





### 列表

​		列表类型可以存储一组按插入顺序排序的字符串，支持两端插入、弹出数据、可以充当栈和队列的角色。

​		`Redis`中链表结构实现列表：

```c++
// adlist.h
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

​		`Redis`使用上面的结构保存运行数据，但不保存用户列表数据：链表中每个节点都占用独立的一块内存，导致内存碎片过多；链表节点中前后节点指针占用

过多的额外内存。

​		对于用户数据`Redis`使用类似数组的紧凑型链表结构，申请一整块内存，在这个内存上存放该链表所有数据。

```tex
<zlbytes> <zltail> <zllen> <entry> <entry> ... <entry> <zlend>
```

​		`zlbytes`：`uint32_t`，记录整个`ziplist`占用的字节数，包括`zlbytes`占用的`4`字节。

​		`zltail`：`uint32_t`，记录从`ziplist`起始位置到最后一个节点的偏移量，用于支持链表从尾部弹出或反向`(`从尾到头`)`遍历链表。

​		`zllen`：`uint16_t`，记录节点数量，如果存在`2 ^ 16 - 2`个节点，则该值设置为`2 ^ 16 - 1`，这是需要遍历整个`ziplist`获取真正的节点数量。

​		`zlend`：一个特殊的标志节点，等于`255`，标志`ziplist`结尾，其他节点数据不会以`255`开头。

​		`entry`：`ziplist`中保存数据的节点：

```tex
<prevlen> <encoding> <entry-data>
```

​				`entry-data`：节点存储的数据。

​				`prevlen`：记录前驱节点长度，单位为字节，该属性长度为`1`字节或`5`字节。如果前驱节点长度小于`254`，则使用`1`字节存储前驱节点长度。否则，使

​		用`5`字节，并且第一个字节固定为`254`，剩余`4`个字节存储前驱节点长度。

​				`encoding`：代表当前节点元素的编码格式，包含编码类型和节点长度。`ziplist`中不同节点元素的编码格式可以不同：

​						`00xxxxxx`：`(xxxxxx)`代表`encoding`低`6`位，字符串编码，长度小于或等于`63(2 ^ 6 - 1)`，长度存放在`encoding`的低`6`位。

​						`01xxxxxx`：字符串编码，长度小于或等于`16383(2 ^ 14 - 1)`，长度存放在`encoding`的后`6`位和`encoding`后一个字节中。

​						`10000000`：字符串编码，长度大于`16383(2 ^ 14 - 1)`，长度存放在`encoding`后`4`字节中。

​						`11000000`：数值编码。类型为`int16_t`，占用`2`字节。

​						`11010000`：数值编码，类型为`int32_t`，占用`4`字节。

​						`11100000`：数值编码，类型为`int64_t`，占用`8`字节。

​						`11110000`：数值编码，使用`3`字节保存一个整数。

​						`11111110`：数值编码，使用`1`字节保存一个整数。

​						`1111xxxx`：使用`encoding`低`4`位存储一个整数，范围为`0 ~ 12`，该编码下`encoding`低`4`位的可用范围为`0001 ~ 1101`，`encoding`低`4`位减`1`

​				为实际存储的值。这种编码是针对小数字的优化，节点元素直接存放在`encoding`上，`entry-data`为空。

​						`11111111`：`255`，`ziplist`结束节点。



#### 查找

```c++
// ziplist.c
/**
	zl：待查找 ziplist
	p：指定从 ziplist 哪个节点开始查找
	vstr、vlen：待查找元素的内容和长度
	skip：间隔多少个节点才执行一次元素对比操作
*/
unsigned char *ziplistFind(unsigned char *zl, unsigned char *p, unsigned char *vstr, unsigned int vlen, unsigned int skip) {
    int skipcnt = 0;
    unsigned char vencoding = 0;
    long long vll = 0;
    size_t zlbytes = ziplistBlobLen(zl);

    while (p[0] != ZIP_END) {
        struct zlentry e;
        unsigned char *q;
		/*
			zipEntrySafe：计算 p 指向的节点的 prevrawlen 属性长度是 1 字节还是5 字节，将结果存在 e->prevrawlensize 属性中。然后解析节点的相关属				性，e->encoding 节点编码格式， e->lensize 额外存放节点元素长度的字节数，即 01xxxxxx 和 10000000 两种编码格式需要额外的空间存放节点元素			长度， e->len 节点元素的长度
		*/
        assert(zipEntrySafe(zl, zlbytes, p, &e, 1));
        q = p + e.prevrawlensize + e.lensize;

        if (skipcnt == 0) {
            /* Compare current entry with specified entry */
            // 如果节点时字符串编码，则对比 String 的内容，相等则返回
            if (ZIP_IS_STR(e.encoding)) {
                if (e.len == vlen && memcmp(q, vstr, vlen) == 0) {
                    return p;
                }
            } else {
                /* Find out if the searched field can be encoded. Note that
                 * we do it only the first time, once done vencoding is set
                 * to non-zero and vll is set to the integer value. */
                // 如果节点是数值编码，并且没有对，待查找的内容 vstr 进行编码，则对 vstr 进行编码，编码后的数值存储在 vll 中
                if (vencoding == 0) {
                    if (!zipTryEncoding(vstr, vlen, &vll, &vencoding)) {
                        /* If the entry can't be encoded we set it to
                         * UCHAR_MAX so that we don't retry again the next
                         * time. */
                        vencoding = UCHAR_MAX;
                    }
                    /* Must be non-zero by now */
                    assert(vencoding);
                }

                /* Compare current entry with specified entry, do it only
                 * if vencoding != UCHAR_MAX because if there is no encoding
                 * possible for the field it can't be a valid integer. */
                // 若编码成功，则对比编码后的结果
                if (vencoding != UCHAR_MAX) {
                    // zipLoadInteger 从节点中提取存储的数值
                    long long ll = zipLoadInteger(q, e.encoding);
                    if (ll == vll) {
                        return p;
                    }
                }
            }

            /* Reset skip count */
            // skipcnt 不为 0，则直接跳过节点并将 skipcnt - 1，知道 skipcnt 为 0 才对比数据
            skipcnt = skip;
        } else {
            /* Skip entry */
            skipcnt--;
        }

        /* Move to next entry */
        // 得到下一个节点的起始位置
        p = q + e.len;
    }

    return NULL;
}
```



#### 插入

```java
// ziplist.c
/**
	zl：待插入 ziplist
	p：指向插入位置的后驱节点
	s,slen：待插入元素的内容和长度
*/
unsigned char *__ziplistInsert(unsigned char *zl, unsigned char *p, unsigned char *s, unsigned int slen) {
    size_t curlen = intrev32ifbe(ZIPLIST_BYTES(zl)), reqlen, newlen;
    unsigned int prevlensize, prevlen = 0;
    size_t offset;
    int nextdiff = 0;
    unsigned char encoding = 0;
    long long value = 123456789; /* initialized to avoid warning. Using a value
                                    that is easy to see if for some reason
                                    we use it uninitialized. */
    zlentry tail;

    /* Find out prevlen for the entry that is inserted. */
    // 计算前驱节点长度并存放到 prevlen
    if (p[0] != ZIP_END) {
        // 如果 p 没有指定 ZIP_END 则直接读取 p 节点的 prevlen 属性，否则需要通过 zpulist.zltail 找到前驱节点，在获取前驱节点的长度
        ZIP_DECODE_PREVLEN(p, prevlensize, prevlen);
    } else {
        unsigned char *ptail = ZIPLIST_ENTRY_TAIL(zl);
        if (ptail[0] != ZIP_END) {
            prevlen = zipRawEntryLengthSafe(zl, curlen, ptail);
        }
    }

    /* See if the entry can be encoded */
    // 对插入元素的内容编码，并将内容的长度存放在 reqlen
    // zipTryEncoding 尝试将内容编码为数值，如果成功返回 1 ，此时 value 指向编码后的值，encoding 存储对应编码格式，否则返回 0
    if (zipTryEncoding(s,slen,&value,&encoding)) {
        /* 'encoding' is set to the appropriate integer encoding */
        reqlen = zipIntSize(encoding);
    } else {
        /* 'encoding' is untouched, however zipStoreEntryEncoding will use the
         * string length to figure out how to encode it. */
        reqlen = slen;
    }
    /* We need space for both the length of the previous entry and
     * the length of the payload. */
    // zipStorePrevEntryLength：计算 prevlen 属性的长度（1 字节或 5 字节）
    // zipStoreEntryEncoding：计算额外存放元素长度所需字节数
    // reqlen 最后的结果为插入节点长度
    reqlen += zipStorePrevEntryLength(NULL,prevlen);
    reqlen += zipStoreEntryEncoding(NULL,encoding,slen);

    /* When the insert position is not equal to the tail, we need to
     * make sure that the next entry can hold this entry's length in
     * its prevlen field. */
    // zipPrevLenByteDiff：计算后驱节点 prevlen 属性长度需要调整多少个字节，结果存入 nextdiff
    int forcelarge = 0;
    nextdiff = (p[0] != ZIP_END) ? zipPrevLenByteDiff(p,reqlen) : 0;
    if (nextdiff == -4 && reqlen < 4) {
        nextdiff = 0;
        forcelarge = 1; // 这里是插入小节点时避免级联更新，所以强制保持后驱节点的 prevlen 属性长度不变
    }

    /* Store offset because a realloc may change the address of zl. */
    // 重新为 ziplist 分配内存，主要是为插入节点申请空间
    offset = p-zl;
    // 新的ziplist 的内存大小，curlen 为插入前 ziplist 的长度
    newlen = curlen+reqlen+nextdiff;
    // ziplistResize 可能会为 ziplist 申请新的内存地址，需要给 p 重新赋值，offset 为插入节点的偏移量
    zl = ziplistResize(zl,newlen);
    p = zl+offset;

    /* Apply memory move when necessary and update tail offset. */
    if (p[0] != ZIP_END) {
        /* Subtract one because of the ZIP_END bytes */
        // 将插入位置后面的节点后移
        memmove(p+reqlen,p-nextdiff,curlen-offset-1+nextdiff);

        /* Encode this entry's raw length in the next entry. */
        // 修改后驱节点的 prevlen 属性
        if (forcelarge)
            zipStorePrevEntryLengthLarge(p+reqlen,reqlen);
        else
            zipStorePrevEntryLength(p+reqlen,reqlen);

        /* Update offset for tail */
        // 更新 ziplist.zltail
        ZIPLIST_TAIL_OFFSET(zl) =
            intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+reqlen);

        /* When the tail contains more than one entry, we need to take
         * "nextdiff" in account as well. Otherwise, a change in the
         * size of prevlen doesn't have an effect on the *tail* offset. */
        // 如果存在多个后驱节点，则 ziplist.zltail 还需要加上 nextdiff
        assert(zipEntrySafe(zl, newlen, p+reqlen, &tail, 1));
        if (p[reqlen+tail.headersize+tail.len] != ZIP_END) {
            ZIPLIST_TAIL_OFFSET(zl) =
                intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+nextdiff);
        }
    } else {
        /* This element will be the new tail. */
        // 不存在后驱节点，只更新 ziplist.zltail
        ZIPLIST_TAIL_OFFSET(zl) = intrev32ifbe(p-zl);
    }

    /* When nextdiff != 0, the raw length of the next entry has changed, so
     * we need to cascade the update throughout the ziplist */
    // 级联更新
    if (nextdiff != 0) {
        offset = p-zl;
        zl = __ziplistCascadeUpdate(zl,p+reqlen);
        p = zl+offset;
    }

    /* Write the entry */
    // 写入插入数据
    p += zipStorePrevEntryLength(p,prevlen);
    p += zipStoreEntryEncoding(p,encoding,slen);
    if (ZIP_IS_STR(encoding)) {
        memcpy(p,s,slen);
    } else {
        zipSaveInteger(p,value,encoding);
    }
    // 更新 ziplist 节点数量 ziplist.zllen
    ZIPLIST_INCR_LENGTH(zl,1);
    return zl;
}
```

​		在新插入一个节点后，可能需要；两次内存拷贝：

​				为整个链表分配新内存空间，主要是为新节点创建空间。

​				将插入节点所有后驱节点后移，为插入节点腾出空间。



#### 级联更新

​		如果插入的节点长度大于等于`254`，则插入后，其后驱节点的`prevlen`属性可能由`1`字节变为`5`字节，此时后驱节点的长度也有可能大于等于`254`，则它的

后驱节点的`prevlen`也有可能由`1`字节变为`5`字节，即级联更新。最极端的情况：插入位置后面的所有节点都需要更新`prevlen`属性。

```c++
// ziplist.c
/**
	p：指向插入节点的后驱节点
*/
unsigned char *__ziplistCascadeUpdate(unsigned char *zl, unsigned char *p) {
    zlentry cur;
    size_t prevlen, prevlensize, prevoffset; /* Informat of the last changed entry. */
    size_t firstentrylen; /* Used to handle insert at head. */
    size_t rawlen, curlen = intrev32ifbe(ZIPLIST_BYTES(zl));
    size_t extra = 0, cnt = 0, offset;
    size_t delta = 4; /* Extra bytes needed to update a entry's prevlen (5-1). */
    unsigned char *tail = zl + intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl));

    /* Empty ziplist */
    if (p[0] == ZIP_END) return zl;

    zipEntry(p, &cur); /* no need for "safe" variant since the input pointer was validated by the function that returned it. */
    firstentrylen = prevlen = cur.headersize + cur.len;
    prevlensize = zipStorePrevEntryLength(NULL, prevlen);
    prevoffset = p - zl;
    p += prevlen;

    /* Iterate ziplist to find out how many extra bytes do we need to update it. */
    // 遇到 ZIP_END ，终止循环
    while (p[0] != ZIP_END) {
        assert(zipEntrySafe(zl, curlen, p, &cur, 0));

        /* Abort when "prevlen" has not changed. */
        // 下一个节点是 ZIP_END，退出循环
        if (cur.prevrawlen == prevlen) break;

        /* Abort when entry's "prevlensize" is big enough. */
        if (cur.prevrawlensize >= prevlensize) {
            if (cur.prevrawlensize == prevlensize) {
                zipStorePrevEntryLength(p, prevlen);
            } else {
                /* This would result in shrinking, which we want to avoid.
                 * So, set "prevlen" in the available bytes. */
                zipStorePrevEntryLengthLarge(p, prevlen);
            }
            break;
        }

        /* cur.prevrawlen means cur is the former head entry. */
        assert(cur.prevrawlen == 0 || cur.prevrawlen + delta == prevlen);

        /* Update prev entry's info and advance the cursor. */
        rawlen = cur.headersize + cur.len;
        prevlen = rawlen + delta; 
        prevlensize = zipStorePrevEntryLength(NULL, prevlen);
        prevoffset = p - zl;
        p += rawlen;
        extra += delta;
        cnt++;
    }

    /* Extra bytes is zero all update has been done(or no need to update). */
    if (extra == 0) return zl;

    /* Update tail offset after loop. */
    if (tail == zl + prevoffset) {
        /* When the the last entry we need to update is also the tail, update tail offset
         * unless this is the only entry that was updated (so the tail offset didn't change). */
        if (extra - delta != 0) {
            ZIPLIST_TAIL_OFFSET(zl) =
                intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+extra-delta);
        }
    } else {
        /* Update the tail offset in cases where the last entry we updated is not the tail. */
        ZIPLIST_TAIL_OFFSET(zl) =
            intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+extra);
    }

    /* Now "p" points at the first unchanged byte in original ziplist,
     * move data after that to new ziplist. */
    offset = p - zl;
    zl = ziplistResize(zl, curlen + extra);
    p = zl + offset;
    memmove(p + extra, p, curlen - offset - 1);
    p += extra;

    /* Iterate all entries that need to be updated tail to head. */
    while (cnt) {
        zipEntry(zl + prevoffset, &cur); /* no need for "safe" variant since we already iterated on all these entries above. */
        rawlen = cur.headersize + cur.len;
        /* Move entry to tail and reset prevlen. */
        memmove(p - (rawlen - cur.prevrawlensize), 
                zl + prevoffset + cur.prevrawlensize, 
                rawlen - cur.prevrawlensize);
        p -= (rawlen + delta);
        if (cur.prevrawlen == 0) {
            /* "cur" is the previous head entry, update its prevlen with firstentrylen. */
            zipStorePrevEntryLength(p, firstentrylen);
        } else {
            /* An entry's prevlen can only increment 4 bytes. */
            zipStorePrevEntryLength(p, cur.prevrawlen+delta);
        }
        /* Foward to previous entry. */
        prevoffset -= cur.prevrawlen;
        cnt--;
    }
    return zl;
}
```



#### quicklist

​		`ziplist`在插入和删除节点时有可能需要进行大量的内存拷贝，性能影响较大。`quicklist`将一个长`ziplist`拆分为多个短`ziplist`，避免插入或删除元素时

导致大量的内存拷贝。

​		`quicklist`基于链表结构，由`quicklistNode`节点链接而成，在`quicklistNode`中使用`ziplist`存储数据。

```c++
// quicklist.h
typedef struct quicklistNode {
    struct quicklistNode *prev; // 指向前驱节点
    struct quicklistNode *next; // 指向后继节点
    unsigned char *zl; // ziplist，负责存储数据
    unsigned int sz; // ziplist 占用的字节数
    unsigned int count : 16; // ziplist 的元素数量
    unsigned int encoding : 2; // 2 代表节点已压缩，1 代表没有压缩
    unsigned int container : 2; // 目前固定为 2，代表使用 ziplist 存储数据
    unsigned int recompress : 1; // 1 代表暂时解压（用于读取数据），后续需要时再将其压缩
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; // 预留属性，暂未使用
} quicklistNode;
```

​		当链表很长时，中间节点数据访问频率较低，`Redis`会将中间节点数据进行压缩`(LZF`算法`)`，节省内存空间，压缩后的定义：

```c++
// quicklist.h
typedef struct quicklistLZF {
    unsigned int sz; // 压缩后 ziplist 的大小
    char compressed[]; // 存放压缩后的 ziplist 字节数组
} quicklistLZF;
```

​		`quiclist`：

```c++
// quicklist.h
typedef struct quicklist {
    quicklistNode *head; // 指向头节点
    quicklistNode *tail; // 指向尾节点
    unsigned long count; // 所有节点的 ziplist 元素数量总和
    unsigned long len; // 节点数量
    int fill : QL_FILL_BITS; // 16bit，用于判断节点 ziplist 是否已满
    unsigned int compress : QL_COMP_BITS; // 16bit，存放节点压缩配置
    unsigned int bookmark_count: QL_BM_BITS;
    quicklistBookmark bookmarks[];
} quicklist;
```

![](image/quicklist.png)

##### 头插法

```c++
// quicklist.c
/**
	value：插入元素的内容
	sz：插入元素的大小
*/
int quicklistPushHead(quicklist *quicklist, void *value, size_t sz) {
    quicklistNode *orig_head = quicklist->head;
    assert(sz < UINT32_MAX); /* TODO: add support for quicklist nodes that are sds encoded (not zipped) */
    // 判断 head 节点 ziplist 是否已满
    if (likely(
            _quicklistNodeAllowInsert(quicklist->head, quicklist->fill, sz))) {
        //  head 节点未满，直接调用 ziplistPush 函数，插入到 ziplist 中
        quicklist->head->zl =
            ziplistPush(quicklist->head->zl, value, sz, ZIPLIST_HEAD);
        // 更新 quicklistNode.sz 属性
        quicklistNodeUpdateSz(quicklist->head);
    } else {
        // head 节点已满，创建一个新节点，将元素插入新节点的 ziplist 中，在将该节点头插入到 quicklist 中
        quicklistNode *node = quicklistCreateNode();
        node->zl = ziplistPush(ziplistNew(), value, sz, ZIPLIST_HEAD);

        quicklistNodeUpdateSz(node);
        _quicklistInsertNodeBefore(quicklist, quicklist->head, node);
    }
    quicklist->count++;
    quicklist->head->count++;
    return (orig_head != quicklist->head);
}
```



##### 指定位置插入

```c++
// quicklist.c
/**
	entry：quicklistEntry 结构，quicklistEntry.node 指定元素插入的 quicklistNode 节点，quicklistEntry.offset 指定插入 ziplist 的索引位置
	after：是否在 quicklistEntry.offset 之后插入
*/
REDIS_STATIC void _quicklistInsert(quicklist *quicklist, quicklistEntry *entry,
                                   void *value, const size_t sz, int after) {
    int full = 0, at_tail = 0, at_head = 0, full_next = 0, full_prev = 0;
    int fill = quicklist->fill;
    quicklistNode *node = entry->node;
    quicklistNode *new_node = NULL;
    assert(sz < UINT32_MAX); /* TODO: add support for quicklist nodes that are sds encoded (not zipped) */

    if (!node) {
        /* we have no reference node, so let's create only node in the list */
        D("No node given!");
        new_node = quicklistCreateNode();
        new_node->zl = ziplistPush(ziplistNew(), value, sz, ZIPLIST_HEAD);
        __quicklistInsertNode(quicklist, NULL, new_node, after);
        new_node->count++;
        quicklist->count++;
        return;
    }

    /* Populate accounting flags for easier boolean checks later */
    /** 根据参数设置标志：
    	full：待插入节点 ziplsit 是否已满
    	at_tail：是否 ziplist 尾插
    	at_head：是否 ziplist 头插
    	full_next：后驱节点是否已满
    	full_prev：前驱节点是否已满
    */
    if (!_quicklistNodeAllowInsert(node, fill, sz)) {
        D("Current node is full with count %d with requested fill %lu",
          node->count, fill);
        full = 1;
    }

    if (after && (entry->offset == node->count)) {
        D("At Tail of current ziplist");
        at_tail = 1;
        if (!_quicklistNodeAllowInsert(node->next, fill, sz)) {
            D("Next node is full too.");
            full_next = 1;
        }
    }

    if (!after && (entry->offset == 0)) {
        D("At Head");
        at_head = 1;
        if (!_quicklistNodeAllowInsert(node->prev, fill, sz)) {
            D("Prev node is full too.");
            full_prev = 1;
        }
    }
    
    // 待插入节点未满，ziplist 尾插：再次检查 ziplist 插入位置是否存在后驱元素，如果不存在则调用 ziplistPush 函数插入元素，否则调用 ziplistInsert 插入元素
    if (!full && after) {
        D("Not full, inserting after current position.");
        quicklistDecompressNodeForUse(node);
        unsigned char *next = ziplistNext(node->zl, entry->zi);
        if (next == NULL) {
            node->zl = ziplistPush(node->zl, value, sz, ZIPLIST_TAIL);
        } else {
            node->zl = ziplistInsert(node->zl, next, value, sz);
        }
        node->count++;
        quicklistNodeUpdateSz(node);
        quicklistRecompressOnly(quicklist, node);
    } else if (!full && !after) {
        // 待插入节点未满，非 ziplist 尾插：调用 ziplistInsert 插入元素
        D("Not full, inserting before current position.");
        quicklistDecompressNodeForUse(node);
        node->zl = ziplistInsert(node->zl, entry->zi, value, sz);
        node->count++;
        quicklistNodeUpdateSz(node);
        quicklistRecompressOnly(quicklist, node);
    } else if (full && at_tail && node->next && !full_next && after) {
        // 待插入节点已满，尾插，后驱节点未满：将元素插入后驱节点 ziplist 中
        /* If we are: at tail, next has free space, and inserting after:
         *   - insert entry at head of next node. */
        D("Full and tail, but next isn't full; inserting next node head");
        new_node = node->next;
        quicklistDecompressNodeForUse(new_node);
        new_node->zl = ziplistPush(new_node->zl, value, sz, ZIPLIST_HEAD);
        new_node->count++;
        quicklistNodeUpdateSz(new_node);
        quicklistRecompressOnly(quicklist, new_node);
    } else if (full && at_head && node->prev && !full_prev && !after) {
        // 待插入节点已满， ziplist 头插，前驱节点未满：将元素插入前驱节点 ziplist 中
        /* If we are: at head, previous has free space, and inserting before:
         *   - insert entry at tail of previous node. */
        D("Full and head, but prev isn't full, inserting prev node tail");
        new_node = node->prev;
        quicklistDecompressNodeForUse(new_node);
        new_node->zl = ziplistPush(new_node->zl, value, sz, ZIPLIST_TAIL);
        new_node->count++;
        quicklistNodeUpdateSz(new_node);
        quicklistRecompressOnly(quicklist, new_node);
    } else if (full && ((at_tail && node->next && full_next && after) ||
                        (at_head && node->prev && full_prev && !after))) {
        // 待插入节点已满，尾插且后驱节点已满，或者头插且前驱节点已满：构建一个新节点，将元素插入新节点，并根据 after 参数将新节点插入 quicklist 中
        /* If we are: full, and our prev/next is full, then:
         *   - create new node and attach to quicklist */
        D("\tprovisioning new node...");
        new_node = quicklistCreateNode();
        new_node->zl = ziplistPush(ziplistNew(), value, sz, ZIPLIST_HEAD);
        new_node->count++;
        quicklistNodeUpdateSz(new_node);
        __quicklistInsertNode(quicklist, node, new_node, after);
    } else if (full) {
        // 待插入节点已满，并且在节点 ziplist 中间插入：将插入节点的数据拆分到两个节点中，在插入拆分后的新节点中
        /* else, node is full we need to split it. */
        /* covers both after and !after cases */
        D("\tsplitting node...");
        // 如果节点已压缩，则解压节点
        quicklistDecompressNodeForUse(node);
        // 从插入节点中拆分出一个新节点，并将元素插入新节点中
        new_node = _quicklistSplitNode(node, entry->offset, after);
        new_node->zl = ziplistPush(new_node->zl, value, sz,
                                   after ? ZIPLIST_HEAD : ZIPLIST_TAIL);
        new_node->count++;
        quicklistNodeUpdateSz(new_node);
        // 将新节点插入 quicklist 中
        __quicklistInsertNode(quicklist, node, new_node, after);
        // 尝试合并节点，合并条件：如果合并后节点大小扔满足 quicklist.fill 参数要求，则合并节点
        /**
        	将 node-prev-prev 合并到 node-prev
        	将 node-next 合并到 node-next-next
        	将 node-prev 合并到 node
        	将 node 合并到 node-next
        */
        _quicklistMergeNodes(quicklist, node);
    }

    quicklist->count++;
}
```



##### 参数

​		`list-max-ziplist-size`：配置`server.list-max-ziplist-size`属性，会赋值给`quicklist.fill`。取正值，表示`quicklist`节点的`ziplist`最多可以存放多

少个元素。取负值，表示`quicklist`节点的`ziplist`最多占用字节数，此时只能取`-1 ~ -5`，默认为`-2`。

​				`-5`：每个`quicklist`节点上的`ziplist`大小不能超过`64KB`。

​				`-4`：每个`quicklist`节点上的`ziplist`大小不能超过`32KB`。

​				`-3`：每个`quicklist`节点上的`ziplist`大小不能超过`16KB`。

​				`-2`：每个`quicklist`节点上的`ziplist`大小不能超过`8KB`。

​				`-1`：每个`quicklist`节点上的`ziplist`大小不能超过`4KB`。

​		`list-compress-depth`：配置`server.list-compress-depth`属性，会赋值`quicklist.compress`：

​				`0`：表示节点都不压缩，`Redis`的默认配置。

​				`1`：表示`quicklist`两端各有`1`个节点不压缩，中间的节点压缩。

​				`2`：表示`quicklist`两端各有`2`个节点不压缩，中间的节点压缩。

​				`3`：表示`quicklist`两端各有`3`个节点不压缩，中间的节点压缩。



​		列表类型只有一种编码格式`OBJ_ENCODING_QUICKLIST`，即使用`quicklist`存储数据`(redisObject.ptr`指向`quicklist`结构`)`。



### 字典

​		`Redis`通常使用字典结构存储用户散列数据。`Redis`数据库也使用了字典结构，其使用`Hash`表实现字典结构。

​		键值对：

```c++
// dict.h
typedef struct dictEntry {
    void *key; // 键
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v; // 值
    struct dictEntry *next; // 下一个键值对指针，即用链表法解决 Hash 冲突
} dictEntry;
```

​		`Hash`表：

```c++
// dict.h
typedef struct dictht {
    dictEntry **table; // Hash 表数组，负责存储数据
    unsigned long size; // 记录存储键值对的数量
    unsigned long sizemask;
    unsigned long used; // Hash 表数组长度
} dictht;
```

![](image/QQ截图20220409110009.png)

​		字典：

```c++
// dict.h
typedef struct dict {
    dictType *type; // 指定操作数据的函数指针
    void *privdata; 
    dictht ht[2]; // 定义两个 Hash 表用于实现字典扩容机制，通常只使用 ht[0]，在扩容时会创建 ht[1]，并在操作数据时逐步将 ht[0] 的数据移到 ht[1] 中
    long rehashidx; // 下一次执行扩容单步操作要迁移的 ht[0] Hash 表数组索引，-1 代表当前没有进行扩容操作
    int16_t pauserehash; /* If >0 rehashing is paused (<0 indicates coding error) */
} dict;
```

​		`dictType`定义了字典中用于操作数据的函数指针，这些函数负责实现数据复制，比较等：

```c++
// dict.h
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key); // Hash 算法，使用 SipHash 算法
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
    int (*expandAllowed)(size_t moreMem, double usedRatio);
} dictType;
```

​		通过`dictType`指定操作数据的函数指针，字典可以存放不同类型的数据，键、值可以是不同的类型，但键必须类型相同，值也必须类型相同。



#### 插入或查找

```c++
// dict.c
int dictAdd(dict *d, void *key, void *val)
{
    dictEntry *entry = dictAddRaw(d,key,NULL);
    if (!entry) return DICT_ERR;
    dictSetVal(d, entry, val);
    return DICT_OK;
}
/**
	existing：如果字典中存在参数 key，且将对应的 dictEntry 指针赋值给 *existing，并返回 null，否则返回创建的 dictEntry
	这个方法只会插入键，并不会插入对应的值。可以使用返回的 dictEntry 插入值
*/
dictEntry *dictAddRaw(dict *d, void *key, dictEntry **existing)
{
    long index;
    dictEntry *entry;
    dictht *ht;
	// 如果字典正在扩容，则执行一次扩容单步操作
    if (dictIsRehashing(d)) _dictRehashStep(d);

    // 计算 key 的 Hash 表数组索引，返回 -1 表示键已存在，数组索引计算方式 hash&ht.sizemask 等同于 hash % ht.size，链表拆分方式与 HashMap 
    if ((index = _dictKeyIndex(d, key, dictHashKey(d,key), existing)) == -1)
        return NULL;

    // 如果字典正在扩容，则将新的 dictEntry 添加到 ht[1] 中，否则添加到 ht[0] 中
    ht = dictIsRehashing(d) ? &d->ht[1] : &d->ht[0];
    
    // 创建 dictEntry，头插法插入链表中
    entry = zmalloc(sizeof(*entry));
    entry->next = ht->table[index];
    ht->table[index] = entry;
    ht->used++;

    // 将键设置到 dictEntry 中
    dictSetKey(d, entry, key);
    return entry;
}

// 计算键的 Hash 表数组索引
static long _dictKeyIndex(dict *d, const void *key, uint64_t hash, dictEntry **existing)
{
    unsigned long idx, table;
    dictEntry *he;
    if (existing) *existing = NULL;

    /* Expand the hash table if needed */
    // 根据需要进行扩容或初始化 Hash 表操作
    if (_dictExpandIfNeeded(d) == DICT_ERR)
        return -1;
    // 遍历 ht[0]，ht[1] ，计算 Hash 表数组索引，并判断 Hash 表中是否已存在参数 key
    for (table = 0; table <= 1; table++) {
        idx = hash & d->ht[table].sizemask;
        /* Search if this slot does not already contain the given key */
        he = d->ht[table].table[idx];
        while(he) {
            if (key==he->key || dictCompareKeys(d, key, he->key)) {
                // 如果存在 key
                if (existing) *existing = he;
                return -1;
            }
            he = he->next;
        }
        // 如果没有进行扩容操作，则计算 ht[0] 索引后边退出，不需要计算 ht[1]
        if (!dictIsRehashing(d)) break;
    }
    return idx;
}
```



#### 扩容

​		`Redis`使用了一种渐进式的扩容方式，`Redis`是单线程的。如果在一个操作内将`ht[0]`所有数据迁移到`ht[1]`，可能会引起线程长期阻塞。所以，`Redis`字

典扩容是在每次操作数据时都执行一次扩容单步操作，即将`ht[0].table[rehashidx]`的数据迁移到`ht[1]`，等到`ht[0]`的所有数据都迁移到`ht[1]`，便将`ht[0]`

指向`ht[1]`，完成扩容。

```c++
// dict.c
// 判断 Hash 表是否需要扩容
static int _dictExpandIfNeeded(dict *d)
{
    /* Incremental rehashing already in progress. Return. */
    if (dictIsRehashing(d)) return DICT_OK;

    /* If the hash table is empty expand it to the initial size. */
    if (d->ht[0].size == 0) return dictExpand(d, DICT_HT_INITIAL_SIZE);

    /* If we reached the 1:1 ratio, and we are allowed to resize the hash
     * table (global setting) or we should avoid it but the ratio between
     * elements/buckets is over the "safe" threshold, we resize doubling
     * the number of buckets. */
   	/**
   		扩容条件：
   			Hash 表存储的键值对数量大于或等于 Hash 表数组的长度
   			开启了 dict_can_resize 或者 负载因子（键值对数量 / Hash 表数组的长度）大于 dict_force_resize_ratio
   		dict_can_resize 默认开启，负载因子等于 1 可能出现比较高的 Hash 冲突率，但可以提高 Hash 表的内存使用率，dict_can_resize 关闭时，必须等到负载		因子等于 5 时才强制扩容
   	*/
    if (d->ht[0].used >= d->ht[0].size &&
        (dict_can_resize ||
         d->ht[0].used/d->ht[0].size > dict_force_resize_ratio) &&
        dictTypeExpandAllowed(d))
    {
        return dictExpand(d, d->ht[0].used + 1);
    }
    return DICT_OK;
}

// size：新 Hash 表长度
int dictExpand(dict *d, unsigned long size) {
    return _dictExpand(d, size, NULL);
}

int _dictExpand(dict *d, unsigned long size, int* malloc_failed)
{
    if (malloc_failed) *malloc_failed = 0;

    /* the size is invalid if it is smaller than the number of
     * elements already inside the hash table */
    if (dictIsRehashing(d) || d->ht[0].used > size)
        return DICT_ERR;
	// _dictNextPower 会将 size 调整为 2 的 n 次幂，为了使 ht[1] Hash 表数组长度是 ht[0] Hash 表数组长度的整数倍，有利于数据迁移更均匀
    dictht n; /* the new hash table */
    unsigned long realsize = _dictNextPower(size);

    /* Detect overflows */
    if (realsize < size || realsize * sizeof(dictEntry*) < realsize)
        return DICT_ERR;

    /* Rehashing to the same table size is not useful. */
    if (realsize == d->ht[0].size) return DICT_ERR;

    /* Allocate the new hash table and initialize all pointers to NULL */
    // 构建一个新的 Hash 表 dictht
    n.size = realsize;
    n.sizemask = realsize-1;
    if (malloc_failed) {
        n.table = ztrycalloc(realsize*sizeof(dictEntry*));
        *malloc_failed = n.table == NULL;
        if (*malloc_failed)
            return DICT_ERR;
    } else
        n.table = zcalloc(realsize*sizeof(dictEntry*));

    n.used = 0;

    /* Is this the first initialization? If so it's not really a rehashing
     * we just set the first hash table so that it can accept keys. */
    // 代表字典的 Hash 表数组没有初始化，即字典第一次使用前的初始化操作
    if (d->ht[0].table == NULL) {
        d->ht[0] = n;
        return DICT_OK;
    }

    /* Prepare a second hash table for incremental rehashing */
    // 将新的 dictht 赋值给 ht[1]，并将 rehashidx 赋值为 0，rehashidx 代表下一次扩容单步操作要迁移的 ht[0] Hash 表数组索引
    d->ht[1] = n;
    d->rehashidx = 0;
    return DICT_OK;
}
```

​		单步扩容：

```c++
// dict.c
static void _dictRehashStep(dict *d) {
    if (d->pauserehash == 0) dictRehash(d,1);
}
// 本次操作迁移的 Hash 数组索引的数量
int dictRehash(dict *d, int n) {
    int empty_visits = n*10; /* Max number of empty buckets to visit. */
    // 如果当前没有进行扩容，则直接退出
    if (!dictIsRehashing(d)) return 0;

    while(n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long)d->rehashidx);
        // 从 rehashidx 开始，找到第一个非空索引位
        while(d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            // 如果查找的的空索引位的数量超过 n * 10，则直接返回
            if (--empty_visits == 0) return 1;
        }
        // 遍历该索引位链表上所有的元素
        de = d->ht[0].table[d->rehashidx];
        /* Move all the keys in this bucket from the old to the new hash HT */
        while(de) {
            uint64_t h;

            nextde = de->next;
            /* Get the index in the new hash table */
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            de = nextde;
        }
        d->ht[0].table[d->rehashidx] = NULL;
        d->rehashidx++;
    }

    /* Check if we already rehashed the whole table... */
    // ht[0].used == 0 表示 ht[0] 的数据已经全部移到 ht[1] 中
    if (d->ht[0].used == 0) {
        // 释放 ht[0].table
        zfree(d->ht[0].table);
        // 将 ht[0] 指向 ht[1]
        d->ht[0] = d->ht[1];
        // 重置 rehashidx d.ht[1]
        _dictReset(&d->ht[1]);
        d->rehashidx = -1;
        return 0;
    }

    /* More to rehash... */
    return 1;
}
```



#### 缩容

​		执行删除操作后，`Redis`会检查字典是否需要缩容，当`Hash`表长度大于`4`且负载因子小于`0.1`时，会执行缩容操作，以节省内存。缩容操作也是通过

`dictExpand`函数完成，只是`size`参数是缩容后的大小。



#### 编码

​		散列类型有`OBJ_ENCODING_HT`和`OBJ_ENCODING_ZIPLIST`，分别使用`dict`、`ziplist`结构存储数据`(redisObject.ptr`指向`dict`、`ziplist`结构`)`。`ziplist`

会优先使用`ziplist`存储散列元素，使用一个`ziplist`节点存储键，后驱节点存放值，查找时需要遍历`ziplist`。使用`dict`存储散列元素，字典的键和值都是

`sds`类型。

​		使用`OBJ_ENCODING_ZIPLIST`编码，需要满足的条件：

​				散列中所有键或值的长度小于或等于`server.hash_max_ziplist_value`，可通过`hash-max-ziplist-value`配置项调整。

​				散列中键值对的数量小于`server.hash_max_ziplist_entries`，可通过`hash-max-ziplist-entries`配置项调整。



#### 数据库

​		`Redis`是内存数据库，内部定义了数据库对象`redisDb`负责存储数据，`redisDb`也使用了字典结构管理数据：

```c++
// server.h
typedef struct redisDb {
    dict *dict; // 数据库字典，该 redisDb 所有的数据都存储在这里
    dict *expires; // 过期字典，存储了 Redis 中所有设置了过期时间的键及其对应的过期时间，过期时间是 long long 类型的 UNIX 时间戳
    dict *blocking_keys; // 处于阻塞状态的键和相应的客户端
    dict *ready_keys; // 准备好数据后可以解除阻塞状态的键和相应的客户端
    dict *watched_keys; // 被 watch 命令监控的键和相应的客户端
    int id; // 数据库 ID 标识
    long long avg_ttl;          /* Average TTL, just for stats */
    unsigned long expires_cursor; /* Cursor of the active expire cycle. */
    list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */
} redisDb;
```

​		`Redis`本身也是一个字典服务，`redisDb.dict`字典中的键都是`sds`，值都是`redisObject`。`db.c`中定义了一系列函数可以通过键找到`redisDb.dict`中对应

的`redisObject`。

![](image/redisdb.png)



### 集合

#### 无序集合

​		`Redis`通常使用字典结构保存用户集合数据，字典键存储集合元素，字典值为空。如果一个集合全是整数，`Redis`设计了`intset`来保存：

```c++
// intset.h
typedef struct intset {
    uint32_t encoding; // 编码格式，intset 中的所有元素必须是同一种编码格式
    uint32_t length; // 元素数量
    int8_t contents[]; // 存储元素数据，元素必须排序，并且无重复
} intset;
```

​		编码格式：

|       定义       | 存储类型 |
| :--------------: | :------: |
| INTSET_ENC_INT16 | int16_t  |
| INTSET_ENC_INT32 | int32_t  |
| INTSET_ENC_INT64 | int64_t  |



##### 插入

```c++
// intset.c
intset *intsetAdd(intset *is, int64_t value, uint8_t *success) {
    // 获取插入元素编码
    uint8_t valenc = _intsetValueEncoding(value);
    uint32_t pos;
    if (success) *success = 1;

	// 如果插入元素编码级别高于 intset 编码，则升级 intset 编码格式，Redis 不会对 intset 进行编码格式降级操作
    if (valenc > intrev32ifbe(is->encoding)) {
        return intsetUpgradeAndAdd(is,value);
    } else {
		// 使用二分查找法查找待插入元素，如果元素存在，则插入失败，否则将 pos 指向插入位置
        if (intsetSearch(is,value,&pos)) {
            if (success) *success = 0;
            return is;
        }
		// 为 intset 重新分配内存空间，主要是分配插入元素所需空间
        is = intsetResize(is,intrev32ifbe(is->length)+1);
        if (pos < intrev32ifbe(is->length)) intsetMoveTail(is,pos,pos+1);
    }
	// 插入元素，更新 intset.length 属性
    _intsetSet(is,pos,value);
    is->length = intrev32ifbe(intrev32ifbe(is->length)+1);
    return is;
}

/**
	如果需要对编码进行升级，只有两种情况：
		待插入元素是正数，并且比 intset 中所有元素都大
		待插入元素是负数，并且比 intset 中所有元素都小
	这两种情况，新元素只能插入到 intset 头部或尾部
*/
static intset *intsetUpgradeAndAdd(intset *is, int64_t value) {
    uint8_t curenc = intrev32ifbe(is->encoding);
    uint8_t newenc = _intsetValueEncoding(value);
    int length = intrev32ifbe(is->length);
    int prepend = value < 0 ? 1 : 0;

    // 设置 intset 新的编码格式，并重新分配新的内存空间
    is->encoding = intrev32ifbe(newenc);
    is = intsetResize(is,intrev32ifbe(is->length)+1);

	// 将 intset 的元素移动到新的位置
    while(length--)
        _intsetSet(is,length+prepend,_intsetGetEncoded(is,length,curenc));
	
    // 插入新的元素，prepend 为 1 代表插入头部
    if (prepend)
        _intsetSet(is,0,value);
    else
        _intsetSet(is,intrev32ifbe(is->length),value);
    is->length = intrev32ifbe(intrev32ifbe(is->length)+1);
    return is;
}
```



##### 编码

​		无序集合有`OBJ_ENCODING_HT`和`OBJ_ENCODING_INTSET`，采用`dict`与`intset`存储数据，使用`OBJ_ENCODING_HT`时，键存储集合元素，值为空。

​		使用`OBJ_ENCODING_INTSET`需要满足的元素：

​				集合中只存在整数型元素。

​				集合元素数量小于或等于`server.set_max_intset_entries`，该值可通过`set-max-intset-entries`配置项配置。



#### 有序集合

​		有序集合数据都是有序的，采用跳表`skiplist`结构实现，`skiplist`兼具了数组和链表的优点。

​		`skiplist`是一个多层级的链表结构：

​				上层链表是相邻下层链表的子集。

​				头结点层数不小于其他节点的层数。

​				每个节点`(`除了头结点`)`都有一个随机的层数。

![](image/skiplist.png)

​		`skiplist`可以看成一颗平衡树，其使用的是一种概率平衡而不是精准平衡。在查找数据时，需要从最高层开始查找，如果某一层后驱节点元素已经大于目标

元素`(`或者不存在后驱节点`)`，则下降一层，从下一层当前位置继续查找。如果在第一层找不到目标元素，则查找失败。在高层查找时，没向后移动一个几点，实

际上会跨越底层多个节点，最终达到二分查找的效率。

```c++
// server.h
typedef struct zskiplistNode {
    sds ele; // 节点值
    double score; // 分数，用于排序节点
    struct zskiplistNode *backward; // 指向前驱节点，一个节点只有第一层有前驱节点指针，skiplist 第一层是一个双向链表
    struct zskiplistLevel {
        struct zskiplistNode *forward; // 指向本层后驱节点
        unsigned long span; // 本层后驱节点跨越了多少个第一层节点，用于计算节点索引
    } level[];
} zskiplistNode;
```

```c++
// server.h
typedef struct zskiplist {
    struct zskiplistNode *header, *tail; // 指向头、尾节点指针
    unsigned long length; // 节点数量
    int level; // skiplist 最大的层数，最多为 ZSKIPLIST_MAXLEVEL（固定为 32）层
} zskiplist;
```

![](image/20171113194041499.png)



##### 查找指定索引的节点

```c++
// t_zset.c
zskiplistNode* zslGetElementByRank(zskiplist *zsl, unsigned long rank) {
    zskiplistNode *x;
    unsigned long traversed = 0;
    int i;

    x = zsl->header;
    // 从头结点的最高层开始查找
    for (i = zsl->level-1; i >= 0; i--) {
        // 如果存在后驱节点，并且后驱节点的索引小于目标值，则沿 forward 继续查找，每次使用 forward 指针跳转到后驱节点，traversed 都需要加上 span，得到		下一个节点的索引
        while (x->level[i].forward && (traversed + x->level[i].span) <= rank)
        {
            traversed += x->level[i].span;
            x = x->level[i].forward;
        }
        // 如果节点索引等于目标值，则返回该节点，否则下降一层继续查找，直到在第一层查找失败
        if (traversed == rank) {
            return x;
        }
    }
    return NULL;
}
```



##### 插入

```c++
// t_zset.c
zskiplistNode *zslInsert(zskiplist *zsl, double score, sds ele) {
    zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
    unsigned int rank[ZSKIPLIST_MAXLEVEL];
    int i, level;

    serverAssert(!isnan(score));
    // 查找每层插入节点的前驱节点，update 数组记录了每层插入位置的前驱节点， rank 数组记录了每层插入位置的前驱节点索引
    x = zsl->header;
    for (i = zsl->level-1; i >= 0; i--) {
        /* store rank that is crossed to reach the insert position */
        rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
        while (x->level[i].forward &&
                (x->level[i].forward->score < score ||
                    (x->level[i].forward->score == score &&
                    sdscmp(x->level[i].forward->ele,ele) < 0)))
        {
            rank[i] += x->level[i].span;
            x = x->level[i].forward;
        }
        update[i] = x;
    }
	// 随机生成新节点的层数
    level = zslRandomLevel();
    // 新节点的层数比其他节点都大，此时 skiplist 需要添加新的层
    if (level > zsl->level) {
        for (i = zsl->level; i < level; i++) {
            rank[i] = 0;
            update[i] = zsl->header;
            update[i]->level[i].span = zsl->length; 
        }
        zsl->level = level;
    }
    // 创建一个节点
    x = zslCreateNode(level,score,ele);
    for (i = 0; i < level; i++) {
        x->level[i].forward = update[i]->level[i].forward;
        update[i]->level[i].forward = x;

        /* update span covered by update[i] as x is inserted here */
        x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
        update[i]->level[i].span = (rank[0] - rank[i]) + 1;
    }

    /* increment span for untouched levels */
    // 如果某个节点存在比新节点层数大的层，则其前驱节点 span 都需要加 1，因为第一层插入了一个新节点
    for (i = level; i < zsl->level; i++) {
        update[i]->level[i].span++;
    }
	// 设置新节点的 backward 属性，更新 skiplist.length
    x->backward = (update[0] == zsl->header) ? NULL : update[0];
    if (x->level[0].forward)
        x->level[0].forward->backward = x;
    else
        zsl->tail = x;
    zsl->length++;
    return x;
}
```

​		为什么不使用红黑树：

​				`skiplist`没有过度占用内存，内存使用率在合理范围内。

​				游戏集合常常执行`ZRANGE`或`ZREVRANGE`等遍历操作，使用`skiplist`可以更高效地实现这些操作`(skiplist`第一层的双向链表可以遍历数据`)`。

​				`skiplist`实现简单。



##### 编码

​		有序集合编码类型有`OBJ_ENCODING_ZIPLIST`和`OBJ_ENCODING_SKIPLIST`,使用`ziplist`或`skiplist`存储数据。

​		使用`OBJ_ENCODING_ZIPLIST`的条件：

​				有序集合元素刷领小于或等于`server.zste_max_ziplist_entries`，可通过`zset-max-ziplist-entries`配置项配置。

​				有序集合所有元素长度都小于等于`serer.zset_max_ziplist_value`，可通过`zset-max-ziplist-value`配置项配置。



### Redis 启动过程

​		`Redis`中定义了`redisServer`结构体，存储`Redis`服务器信息，包括服务器配置项和运行时数据，可以使用`info`指令查看服务器信息：

```c++
struct redisServer {
    /* General */
    pid_t pid;                  /* Main process pid. */
    pthread_t main_thread_id;         /* Main thread id */
    char *configfile;           /* Absolute config file path, or NULL */
    char *executable;           /* Absolute executable file path. */
    char **exec_argv;           /* Executable argv vector (copy). */
	...
};

extern struct redisServer server;
```

#### 启动 Redis 服务

```c++
// server.h
int main(int argc, char **argv) {
    
	...
    // 检查 Redis 服务器是否以 sentinel 模式启动
    server.sentinel_mode = checkForSentinelMode(argc,argv);
    // initServerConfig：将 redisServer 中记录配置项的属性初始化为默认值
    initServerConfig();
    // 初始化 ACL 机制
    ACLInit(); /* The ACL subsystem must be initialized ASAP because the
                  basic networking code and client creation depends on it. */
    // 初始化 Module 机制
    moduleInitModulesSystem();
    tlsInit();

    /* Store the executable path and arguments in a safe place in order
     * to be able to restart the server later. */
    // 记录 Redis 程序可执行路径及启动参数，一边后续重启服务器
    server.executable = getAbsolutePath(argv[0]);
    server.exec_argv = zmalloc(sizeof(char*)*(argc+1));
    server.exec_argv[argc] = NULL;
    for (j = 0; j < argc; j++) server.exec_argv[j] = zstrdup(argv[j]);

    /* We need to init sentinel right now as parsing the configuration file
     * in sentinel mode will have the effect of populating the sentinel
     * data structures with master nodes to monitor. */
    // 如果以 Sentinel 模式启动，则初始化 Sentinel 机制
    if (server.sentinel_mode) {
        initSentinelConfig();
        initSentinel();
    }

    /* Check if we need to start in redis-check-rdb/aof mode. We just execute
     * the program main. However the program is part of the Redis executable
     * so that we can easily execute an RDB check on loading errors. */
    // 如果启动程序是 redis-check-rdb 或 redis-check-aof ，则执行对应的函数，尝试检验并修复 RDB、AOF 文件后便退出程序
    if (strstr(argv[0],"redis-check-rdb") != NULL)
        redis_check_rdb_main(argc,argv,NULL);
    else if (strstr(argv[0],"redis-check-aof") != NULL)
        redis_check_aof_main(argc,argv);
	
    if (argc >= 2) {
        j = 1; /* First option to parse in argv[] */
        sds options = sdsempty();

        /* Handle special options --help and --version */
        // 对 -v、--version、--help、-h、--test-memory 等命令进行优先处理
        if (strcmp(argv[1], "-v") == 0 ||
            strcmp(argv[1], "--version") == 0) version();
        if (strcmp(argv[1], "--help") == 0 ||
            strcmp(argv[1], "-h") == 0) usage();
        if (strcmp(argv[1], "--test-memory") == 0) {
            if (argc == 3) {
                memtest(atoi(argv[2]),50);
                exit(0);
            } else {
                fprintf(stderr,"Please specify the amount of memory to test in megabytes.\n");
                fprintf(stderr,"Example: ./redis-server --test-memory 4096\n\n");
                exit(1);
            }
        }
        /* Parse command line options
         * Precedence wise, File, stdin, explicit options -- last config is the one that matters.
         *
         * First argument is the config file name? */
        // 如果启动命令的第二个参数不是以 -- 开始的，则为配置文件参数，将配置文件路径转化为绝对路径，存入 server.configfile
        if (argv[1][0] != '-') {
            /* Replace the config file in server.exec_argv with its absolute path. */
            server.configfile = getAbsolutePath(argv[1]);
            zfree(server.exec_argv[1]);
            server.exec_argv[1] = zstrdup(server.configfile);
            j = 2; // Skip this arg when parsing options
        }
        // 读取启动命令中的启动配置项，并将它们拼接到一个字符串中
        while(j < argc) {
            /* Either first or last argument - Should we read config from stdin? */
            if (argv[j][0] == '-' && argv[j][1] == '\0' && (j == 1 || j == argc-1)) {
                config_from_stdin = 1;
            }
            /* All the other options are parsed and conceptually appended to the
             * configuration file. For instance --port 6380 will generate the
             * string "port 6380\n" to be parsed after the actual config file
             * and stdin input are parsed (if they exist). */
            else if (argv[j][0] == '-' && argv[j][1] == '-') {
                /* Option name */
                if (sdslen(options)) options = sdscat(options,"\n");
                options = sdscat(options,argv[j]+2);
                options = sdscat(options," ");
            } else {
                /* Option argument */
                options = sdscatrepr(options,argv[j],strlen(argv[j]));
                options = sdscat(options," ");
            }
            j++;
        }
		// 从配置文件中加载所有配置项，并使用启动命令配置项覆盖配置文件中的配置项
        loadServerConfig(server.configfile, config_from_stdin, options);
        if (server.sentinel_mode) loadSentinelConfigFromQueue();
        sdsfree(options);
    }
    if (server.sentinel_mode) sentinelCheckConfigFile();
    // server.supervised 属性指定是否以 upstart 服务或 systemd 服务启动 Redis，如果配置了 server.daemonize 且没有妹纸 server.supervised，则以守		护进程的方式启动 Redis
    server.supervised = redisIsSupervised(server.supervised_mode);
    int background = server.daemonize && !server.supervised;
    if (background) daemonize();
	// 打印启动日志
    serverLog(LL_WARNING, "oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo");
    serverLog(LL_WARNING,
        "Redis version=%s, bits=%d, commit=%s, modified=%d, pid=%d, just started",
            REDIS_VERSION,
            (sizeof(long) == 8) ? 64 : 32,
            redisGitSHA1(),
            strtol(redisGitDirty(),NULL,10) > 0,
            (int)getpid());

    if (argc == 1) {
        serverLog(LL_WARNING, "Warning: no config file specified, using the default config. In order to specify a config file use %s /path/to/redis.conf", argv[0]);
    } else {
        serverLog(LL_WARNING, "Configuration loaded");
    }

    readOOMScoreAdj();
    // 初始化 Redis 运行时数据
    initServer();
    // 创建 pid 文件
    if (background || server.pidfile) createPidFile();
    if (server.set_proc_title) redisSetProcTitle(NULL);
    redisAsciiArt();
    checkTcpBacklogSettings();
	// 非 Sentinel 模式启动
    if (!server.sentinel_mode) {
        /* Things not needed when running in Sentinel mode. */
        serverLog(LL_WARNING,"Server initialized");
    #ifdef __linux__
        linuxMemoryWarnings();
    #if defined (__arm64__)
        int ret;
        if ((ret = linuxMadvFreeForkBugCheck())) {
            if (ret == 1)
                serverLog(LL_WARNING,"WARNING Your kernel has a bug that could lead to data corruption during background save. "
                                     "Please upgrade to the latest stable kernel.");
            else
                serverLog(LL_WARNING, "Failed to test the kernel for a bug that could lead to data corruption during background save. "
                                      "Your system could be affected, please report this error.");
            if (!checkIgnoreWarning("ARM64-COW-BUG")) {
                serverLog(LL_WARNING,"Redis will now exit to prevent data corruption. "
                                     "Note that it is possible to suppress this warning by setting the following config: ignore-warnings ARM64-COW-BUG");
                exit(1);
            }
        }
    #endif /* __arm64__ */
    #endif /* __linux__ */
        moduleInitModulesSystemLast();
        // 加载配置文件指定的 Module 模块
        moduleLoadFromQueue();
        // 加载 ACL 用户控制列表
        ACLLoadUsersAtStartup();
        // 负责创建后台线程、I/O线程、该步骤需在 Module 模块加载后再执行
        InitServerLast();
        // 从磁盘中加载 AOF 或 RDB 文件
        loadDataFromDisk();
        // 如果以 Cluster 模式启动，还需要验证加载的数据是否正确
        if (server.cluster_enabled) {
            if (verifyClusterConfigWithData() == C_ERR) {
                serverLog(LL_WARNING,
                    "You can't have keys in a DB different than DB 0 when in "
                    "Cluster mode. Exiting.");
                exit(1);
            }
        }
        if (server.ipfd.count > 0 || server.tlsfd.count > 0)
            serverLog(LL_NOTICE,"Ready to accept connections");
        if (server.sofd > 0)
            serverLog(LL_NOTICE,"The server is now ready to accept connections at %s", server.unixsocket);
        if (server.supervised_mode == SUPERVISED_SYSTEMD) {
            if (!server.masterhost) {
                redisCommunicateSystemd("STATUS=Ready to accept connections\n");
            } else {
                redisCommunicateSystemd("STATUS=Ready to accept connections in read-only mode. Waiting for MASTER <-> REPLICA sync\n");
            }
            redisCommunicateSystemd("READY=1\n");
        }
    } else {
        ACLLoadUsersAtStartup();
        // 以 Sentinel 模式启动，则调用 sentinelIsRunning 函数启动 Sentinel机制
        InitServerLast();
        sentinelIsRunning();
        if (server.supervised_mode == SUPERVISED_SYSTEMD) {
            redisCommunicateSystemd("STATUS=Ready to accept connections\n");
            redisCommunicateSystemd("READY=1\n");
        }
    }

    /* Warning the user about suspicious maxmemory setting. */
    if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
        serverLog(LL_WARNING,"WARNING: You specified a maxmemory value that is less than 1MB (current value is %llu bytes). Are you sure this is what you really want?", server.maxmemory);
    }
	// 尽可能将 Redis 主线程绑定到 server.server_cpulist 配置的 CPU 列表上，Redis 4开始支持多线程，该操作可以减少不必要的线程切换，提高性能
    redisSetCpuAffinity(server.server_cpulist);
    setOOMScoreAdj(-1);
	// 启动事件循环器，事件循环器是 Redis 中的重要组件，在 Redis 运行期间，由事件循环器提供服务
    aeMain(server.el);
    // 到这里说明 Redis 服务已停止，清除事件循环器中的事件，然后退出程序
    aeDeleteEventLoop(server.el);
    return 0;
}
```



#### Redis 初始化过程

```c++
// server.c
void initServer(void) {
    int j;
	// 设置 UNIX 信号处理函数，使 Redis 服务器收到 SIGINT 信号后退出程序
    signal(SIGHUP, SIG_IGN);
    signal(SIGPIPE, SIG_IGN);
    setupSignalHandlers();
    // 设置线程随时相应 CANCEL 信号，终止线程，以便停止程序
    makeThreadKillable();
	// 如果开启了系统日志，调用 openlog 与系统日志简历输出连接，以便输出系统日志
    if (server.syslog_enabled) {
        openlog(server.syslog_ident, LOG_PID | LOG_NDELAY | LOG_NOWAIT,
            server.syslog_facility);
    }

    /* Initialization after setting defaults from the config system. */
    // 初始化 server 中负责存储运行时数据的相关属性
    server.aof_state = server.aof_enabled ? AOF_ON : AOF_OFF;
    server.hz = server.config_hz;
    server.pid = getpid();
    server.in_fork_child = CHILD_TYPE_NONE;
    server.main_thread_id = pthread_self();
    server.current_client = NULL;
    server.errors = raxNew();
    server.fixed_time_expire = 0;
    server.clients = listCreate();
    server.clients_index = raxNew();
    server.clients_to_close = listCreate();
    server.slaves = listCreate();
    server.monitors = listCreate();
    server.clients_pending_write = listCreate();
    server.clients_pending_read = listCreate();
    server.clients_timeout_table = raxNew();
    server.replication_allowed = 1;
    server.slaveseldb = -1; /* Force to emit the first SELECT command. */
    server.unblocked_clients = listCreate();
    server.ready_keys = listCreate();
    server.clients_waiting_acks = listCreate();
    server.get_ack_from_slaves = 0;
    server.client_pause_type = 0;
    server.paused_clients = listCreate();
    server.events_processed_while_blocked = 0;
    server.system_memory_size = zmalloc_get_memory_size();
    server.blocked_last_cron = 0;
    server.blocking_op_nesting = 0;

    if ((server.tls_port || server.tls_replication || server.tls_cluster)
                && tlsConfigure(&server.tls_ctx_config) == C_ERR) {
        serverLog(LL_WARNING, "Failed to configure TLS. Check logs for more info.");
        exit(1);
    }

    // 创建共享数据集，这些数据可以各场景下共享使用
    createSharedObjects();
    // 尝试修改环境变量，提高系统允许打开的文件描述符上限，避免由于大量客户端连接导致错误
    adjustOpenFilesLimit();
    const char *clk_msg = monotonicInit();
    serverLog(LL_NOTICE, "monotonic clock: %s", clk_msg);
    // 创建事件循环器
    server.el = aeCreateEventLoop(server.maxclients+CONFIG_FDSET_INCR);
    if (server.el == NULL) {
        serverLog(LL_WARNING,
            "Failed creating the event loop. Error message: '%s'",
            strerror(errno));
        exit(1);
    }
    server.db = zmalloc(sizeof(redisDb)*server.dbnum);

    /* Open the TCP listening socket for the user commands. */
    // 如果配置了 server.port ，则开始 TCP Socket 服务，接收用户请求
    if (server.port != 0 &&
        listenToPort(server.port,&server.ipfd) == C_ERR) {
        serverLog(LL_WARNING, "Failed listening on port %u (TCP), aborting.", server.port);
        exit(1);
    }
    // 如果配置了 server.tls_port ，则开始 TLS Socket 服务，接收用户请求，Redis 6.0 开始支持 TLS 连接
    if (server.tls_port != 0 &&
        listenToPort(server.tls_port,&server.tlsfd) == C_ERR) {
        serverLog(LL_WARNING, "Failed listening on port %u (TLS), aborting.", server.tls_port);
        exit(1);
    }

    /* Open the listening Unix domain socket. */
   	// 如果配置了 server.unixsocket ，则开始 UNIX Socket 服务，接收用户请求，Redis 6.0 开始支持 TLS 连接
    if (server.unixsocket != NULL) {
        unlink(server.unixsocket); /* don't care if this fails */
        server.sofd = anetUnixServer(server.neterr,server.unixsocket,
            server.unixsocketperm, server.tcp_backlog);
        if (server.sofd == ANET_ERR) {
            serverLog(LL_WARNING, "Opening Unix socket: %s", server.neterr);
            exit(1);
        }
        anetNonBlock(NULL,server.sofd);
        anetCloexec(server.sofd);
    }

    /* Abort if there are no listening sockets at all. */
    // 上面三个都没有配置，则报错退出
    if (server.ipfd.count == 0 && server.tlsfd.count == 0 && server.sofd < 0) {
        serverLog(LL_WARNING, "Configured to not listen anywhere, exiting.");
        exit(1);
    }

    /* Create the Redis databases, and initialize other internal state. */
    // 初始化数据库 server.db，用于存储数据
    for (j = 0; j < server.dbnum; j++) {
        server.db[j].dict = dictCreate(&dbDictType,NULL);
        server.db[j].expires = dictCreate(&dbExpiresDictType,NULL);
        server.db[j].expires_cursor = 0;
        server.db[j].blocking_keys = dictCreate(&keylistDictType,NULL);
        server.db[j].ready_keys = dictCreate(&objectKeyPointerValueDictType,NULL);
        server.db[j].watched_keys = dictCreate(&keylistDictType,NULL);
        server.db[j].id = j;
        server.db[j].avg_ttl = 0;
        server.db[j].defrag_later = listCreate();
        listSetFreeMethod(server.db[j].defrag_later,(void (*)(void*))sdsfree);
    }
    // 初始化 LRU/LFU 样本池，用于实现 LRU/LFU 近似算法
    evictionPoolAlloc(); /* Initialize the LRU keys pool. */
    server.pubsub_channels = dictCreate(&keylistDictType,NULL);
    server.pubsub_patterns = dictCreate(&keylistDictType,NULL);
    server.cronloops = 0;
    server.in_eval = 0;
    server.in_exec = 0;
    server.propagate_in_transaction = 0;
    server.client_pause_in_transaction = 0;
    server.child_pid = -1;
    server.child_type = CHILD_TYPE_NONE;
    server.rdb_child_type = RDB_CHILD_TYPE_NONE;
    server.rdb_pipe_conns = NULL;
    server.rdb_pipe_numconns = 0;
    server.rdb_pipe_numconns_writing = 0;
    server.rdb_pipe_buff = NULL;
    server.rdb_pipe_bufflen = 0;
    server.rdb_bgsave_scheduled = 0;
    server.child_info_pipe[0] = -1;
    server.child_info_pipe[1] = -1;
    server.child_info_nread = 0;
    aofRewriteBufferReset();
    server.aof_buf = sdsempty();
    server.lastsave = time(NULL); /* At startup we consider the DB saved. */
    server.lastbgsave_try = 0;    /* At startup we never tried to BGSAVE. */
    server.rdb_save_time_last = -1;
    server.rdb_save_time_start = -1;
    server.dirty = 0;
    resetServerStats();
    /* A few stats we don't want to reset: server startup time, and peak mem. */
    server.stat_starttime = time(NULL);
    server.stat_peak_memory = 0;
    server.stat_current_cow_bytes = 0;
    server.stat_current_cow_updated = 0;
    server.stat_current_save_keys_processed = 0;
    server.stat_current_save_keys_total = 0;
    server.stat_rdb_cow_bytes = 0;
    server.stat_aof_cow_bytes = 0;
    server.stat_module_cow_bytes = 0;
    server.stat_module_progress = 0;
    for (int j = 0; j < CLIENT_TYPE_COUNT; j++)
        server.stat_clients_type_memory[j] = 0;
    server.cron_malloc_stats.zmalloc_used = 0;
    server.cron_malloc_stats.process_rss = 0;
    server.cron_malloc_stats.allocator_allocated = 0;
    server.cron_malloc_stats.allocator_active = 0;
    server.cron_malloc_stats.allocator_resident = 0;
    server.lastbgsave_status = C_OK;
    server.aof_last_write_status = C_OK;
    server.aof_last_write_errno = 0;
    server.repl_good_slaves_count = 0;

    /* Create the timer callback, this is our way to process many background
     * operations incrementally, like clients timeout, eviction of unaccessed
     * expired keys and so forth. */
    // 创建一个事件事件，负责处理 Redis 中的定时任务，如清理过期数据、生成 RDB 文件等
    if (aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL) == AE_ERR) {
        serverPanic("Can't create event loop timers.");
        exit(1);
    }

    /* Create an event handler for accepting new connections in TCP and Unix
     * domain sockets. */
    // 分别为 TCP Scoket、TSL Socket、Unix Socket 注册 AE_READABLE 文件事件的处理函数，分别为 acceptTcpHandler、acceptTLSHandler、acceptUnixHandler，这些函数负责接收 Socket 中的新连接
    if (createSocketAcceptHandler(&server.ipfd, acceptTcpHandler) != C_OK) {
        serverPanic("Unrecoverable error creating TCP socket accept handler.");
    }
    if (createSocketAcceptHandler(&server.tlsfd, acceptTLSHandler) != C_OK) {
        serverPanic("Unrecoverable error creating TLS socket accept handler.");
    }
    if (server.sofd > 0 && aeCreateFileEvent(server.el,server.sofd,AE_READABLE,
        acceptUnixHandler,NULL) == AE_ERR) serverPanic("Unrecoverable error creating server.sofd file event.");


    /* Register a readable event for the pipe used to awake the event loop
     * when a blocked client in a module needs attention. */
    if (aeCreateFileEvent(server.el, server.module_blocked_pipe[0], AE_READABLE,
        moduleBlockedClientPipeReadable,NULL) == AE_ERR) {
            serverPanic(
                "Error registering the readable event for the module "
                "blocked clients subsystem.");
    }

    /* Register before and after sleep handlers (note this needs to be done
     * before loading persistence since it is used by processEventsWhileBlocked. */
    // 注册事件循环器的钩子函数，事件循环器在每次阻塞前后都会调用钩子函数
    aeSetBeforeSleepProc(server.el,beforeSleep);
    aeSetAfterSleepProc(server.el,afterSleep);

    /* Open the AOF file if needed. */
    // 如果开始前 AOF，则预先打开 AOF 文件
    if (server.aof_state == AOF_ON) {
        server.aof_fd = open(server.aof_filename,
                               O_WRONLY|O_APPEND|O_CREAT,0644);
        if (server.aof_fd == -1) {
            serverLog(LL_WARNING, "Can't open the append-only file: %s",
                strerror(errno));
            exit(1);
        }
    }

    /* 32 bit instances are limited to 4GB of address space, so if there is
     * no explicit limit in the user provided configuration we set a limit
     * at 3 GB using maxmemory with 'noeviction' policy'. This avoids
     * useless crashes of the Redis instance for out of memory. */
    // 如果 Redis 运行在 32 位的机器上， 32 位操作系统内存空闲限制为 4 GB，所以 Redis 将内存限制在 3GB，避免 Redis 服务器因内存不足而崩溃
    if (server.arch_bits == 32 && server.maxmemory == 0) {
        serverLog(LL_WARNING,"Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.");
        server.maxmemory = 3072LL*(1024*1024); /* 3 GB */
        server.maxmemory_policy = MAXMEMORY_NO_EVICTION;
    }
	// 如果以 Cluster 模式启动，则初始化 Cluster 机制
    if (server.cluster_enabled) clusterInit();
    // 初始化 server.repl_scriptcache_dict 属性
    replicationScriptCacheInit();
    // 初始化 LUA 机制
    scriptingInit(1);
    // 初始化慢日志机制
    slowlogInit();
    // 初始化延迟监控机制
    latencyMonitorInit();
    
    /* Initialize ACL default password if it exists */
    ACLUpdateDefaultUserPassword(server.requirepass);
}
```





### 事件机制

​		`Redis`服务器是一个事件驱动程序，主要负责处理两种事件：

​				文本事件：利用`I/O`复用机制，监听`Scoket`等文件描述符上发生的事件，这类事件主要由客户端`(`或其他`Redis`服务器`)`发送网络请求触发。

​				时间事件：定时触发的事件，负责完成`Redis`内部定时任务。

​		`Redis`利用`I/O`复用机制实现网络通信，`I/O`复用是一种高性能`I/O`模型，利用单进程监听多个客户端连接，当某个连接状态发生变化`(`可读、可写`)`时，

操作系统会发送事件`(`称为已就绪事件`)`通知进程处理该连接的数据，`Redis`实现了自己的事件机制，支持不同系统的`I/O`复用`API`。

![](image/QQ截图20220414105540.png)

​		事件循环器：

```c++
// ae.h
typedef struct aeEventLoop {
    int maxfd; // 当前已注册的最大文件描述符
    int setsize; // 该事件循环器允许监听的最大的文件描述符
    long long timeEventNextId; // 下一个时间事件 ID
    aeFileEvent *events; // 已注册的文件事件表
    aeFiredEvent *fired; // 已就绪的事件表
    aeTimeEvent *timeEventHead; // 时间事件表的头结点指针
    int stop; // 事件循环器是否停止
    void *apidata; // 存放用于 I/O 复用层的附加数据
    aeBeforeSleepProc *beforesleep; // 进程阻塞前调用的钩子函数
    aeBeforeSleepProc *aftersleep; // 进程阻塞后调用的钩子函数
    int flags;
} aeEventLoop;

/**
	aeFileEvent 并没有记录文件描述符 fd 的属性，fd 的约束：
		值为 0、1、2 的文件描述符分别表示标准输入、标准输出、错误输出
		每次新打开的文件描述符，必须使用当前进程中最小可用的文件描述符
*/
typedef struct aeFileEvent {
    int mask; // 已注册的文件事件类型，AE_NONE，AE_READABLE，AE_WRITABLE
    aeFileProc *rfileProc; // AE_READABLE 事件处理函数
    aeFileProc *wfileProc; // AE_WRITABLE 事件处理函数
    void *clientData; // 附加数据
} aeFileEvent;

/**
	I/O 复用层会将已就绪的事件转化为 aeFiredEvent
*/
typedef struct aeFiredEvent {
    int fd; // 产生事件的文件描述符
    int mask; // 产生的事件类型
} aeFiredEvent;

// 存储时间事件的信息
typedef struct aeTimeEvent {
    long long id; // 时间事件 ID
    monotime when; // 时间事件下一次执行的秒数和剩余毫秒数
    aeTimeProc *timeProc; // 时间事件处理函数
    aeEventFinalizerProc *finalizerProc; // 时间事件终结函数
    void *clientData; // 客户端传入的附加数据
    struct aeTimeEvent *prev; // 后一个时间事件
    struct aeTimeEvent *next; // 前一个时间事件
    int refcount; /* refcount to prevent timer events from being
  		   * freed in recursive time event calls. */
} aeTimeEvent;
```



#### 创建事件

```c++
// ae.c
// Redis 启动时，在 server.c 的 initServer 函数中会调用 aeCreateEventLoop 创建循环器
aeEventLoop *aeCreateEventLoop(int setsize) {
    aeEventLoop *eventLoop;
    int i;

    monotonicInit();    /* just in case the calling app didn't initialize */
	// 初始化 aeEventLoop 属性
    if ((eventLoop = zmalloc(sizeof(*eventLoop))) == NULL) goto err;
    eventLoop->events = zmalloc(sizeof(aeFileEvent)*setsize);
    eventLoop->fired = zmalloc(sizeof(aeFiredEvent)*setsize);
    if (eventLoop->events == NULL || eventLoop->fired == NULL) goto err;
    eventLoop->setsize = setsize;
    eventLoop->timeEventHead = NULL;
    eventLoop->timeEventNextId = 0;
    eventLoop->stop = 0;
    eventLoop->maxfd = -1;
    eventLoop->beforesleep = NULL;
    eventLoop->aftersleep = NULL;
    eventLoop->flags = 0;
    // aeApiCreate 由 I/O 复用层实现，此时已经根据运行系统选择了具体的 I/O 复用层适配代码
    if (aeApiCreate(eventLoop) == -1) goto err;
    /* Events with mask == AE_NONE are not set. So let's initialize the
     * vector with it. */
    for (i = 0; i < setsize; i++)
        eventLoop->events[i].mask = AE_NONE;
    return eventLoop;

err:
    if (eventLoop) {
        zfree(eventLoop->events);
        zfree(eventLoop->fired);
        zfree(eventLoop);
    }
    return NULL;
}

// Redis 启动时，在 server.c 的 initServer 函数中会调用 aeCreateFileEvent 为 TCP Socket 等文件描述符注册了 AE_WRITABLE 文本事件的处理函数。所以事件循环器会监听 TCP Socket ，并使用指定函数处理 AE_WRITABLE 事件
/**
	fd：需监听的文件描述符
	mask：监听事件类型
	proc：事件处理函数
	clientData：附加数据
*/
int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask,
        aeFileProc *proc, void *clientData)
{
    // 如果超出了 eventLoop.setsize 限制，则返回错误
    if (fd >= eventLoop->setsize) {
        errno = ERANGE;
        return AE_ERR;
    }
    aeFileEvent *fe = &eventLoop->events[fd];
	// aeApiAddEvent 由 I/O 复用层实现，调用 I/O 复用函数添加事件监听对象
    if (aeApiAddEvent(eventLoop, fd, mask) == -1)
        return AE_ERR;
    // 初始化 aeFileEvent 属性
    fe->mask |= mask;
    if (mask & AE_READABLE) fe->rfileProc = proc;
    if (mask & AE_WRITABLE) fe->wfileProc = proc;
    fe->clientData = clientData;
    if (fd > eventLoop->maxfd)
        eventLoop->maxfd = fd;
    return AE_OK;
}

// Redis 启动时，在 server.c 的 initServer 函数中会调用 aeCreateTimeEvent 创建一个处理函数为 serverCron 的时间事件，负责处理 Redis 中的定时任务
long long aeCreateTimeEvent(aeEventLoop *eventLoop, long long milliseconds,
        aeTimeProc *proc, void *clientData,
        aeEventFinalizerProc *finalizerProc)
{
    // 初始化 aeTimeEvent 属性
    long long id = eventLoop->timeEventNextId++;
    aeTimeEvent *te;

    te = zmalloc(sizeof(*te));
    if (te == NULL) return AE_ERR;
    te->id = id;
    // 计算时间事件下次执行时间
    te->when = getMonotonicUs() + milliseconds * 1000;
    te->timeProc = proc;
    te->finalizerProc = finalizerProc;
    te->clientData = clientData;
    // 头插入 eventLoop->timeEventHead 链表
    te->prev = NULL;
    te->next = eventLoop->timeEventHead;
    te->refcount = 0;
    if (te->next)
        te->next->prev = te;
    eventLoop->timeEventHead = te;
    return id;
}
```

​		在`Redis`启动的最后，会调用`aeMain`函数，启动事件循环器：

```c++
// ae.c
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|
                                   AE_CALL_BEFORE_SLEEP|
                                   AE_CALL_AFTER_SLEEP);
    }
}
```



#### 事件循环器的运行

```c++
// ae.c
/**
	flags：指定 aeProcessEvents 函数处理的事件类型和事件处理策略
	AE_ALL_EVENTS：处理所有事件
	AE_FILE_ENENTS：处理文本事件
	AE_TIME_EVENTS：处理时间事件
	AE_DONT_WAIT：是否阻塞进程
	AE_CALL_AFTER_SLEEP：阻塞后是否调用 eventLoop.aftersleep 函数
	AE_CALL_BEFORE_SLEEP：阻塞前是否调用 eventLoop.beforesleep 函数
	
*/
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int processed = 0, numevents;

    /* Nothing to do? return ASAP */
    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

    /* Note that we want to call select() even if there are no
     * file events to process as long as we want to process time
     * events, in order to sleep until the next time event is ready
     * to fire. */
    // 阻塞进程，等待文本事件就绪或时间事件到达执行时间
    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        int j;
        // 计算最大阻塞时间
        /**
        	查找最先执行的时间事件，能找到，则将该事件执行时间减去当前时间作为进程的最大阻塞时间
        	找不到时间事件，检查 flags 参数是否为 AE_DONT_WAIT 标志，不是，进程将一直阻塞，直到有文本事件就绪，否则，进程不阻塞，将不断询问系统是否有已		就绪的文本事件。如果 eventLoop.flags 中为 AE_DONT_WAIT ，进程也不会阻塞
        */
        struct timeval tv, *tvp;
        int64_t usUntilTimer = -1;

        if (flags & AE_TIME_EVENTS && !(flags & AE_DONT_WAIT))
            usUntilTimer = usUntilEarliestTimer(eventLoop);

        if (usUntilTimer >= 0) {
            tv.tv_sec = usUntilTimer / 1000000;
            tv.tv_usec = usUntilTimer % 1000000;
            tvp = &tv;
        } else {
            /* If we have to check for events but need to return
             * ASAP because of AE_DONT_WAIT we need to set the timeout
             * to zero */
            if (flags & AE_DONT_WAIT) {
                tv.tv_sec = tv.tv_usec = 0;
                tvp = &tv;
            } else {
                /* Otherwise we can block */
                tvp = NULL; /* wait forever */
            }
        }

        if (eventLoop->flags & AE_DONT_WAIT) {
            tv.tv_sec = tv.tv_usec = 0;
            tvp = &tv;
        }
		// 进程阻塞前，执行钩子函数 beforeSleep
        if (eventLoop->beforesleep != NULL && flags & AE_CALL_BEFORE_SLEEP)
            eventLoop->beforesleep(eventLoop);

        /* Call the multiplexing API, will return only on timeout or when
         * some event fires. */
        // 由 I/O 复用层实现，负责阻塞当前进程，直到有文本事件就绪或者给定时间到期，该函数返回已就绪文件事件的数量，并将事件存储在 aeEventLoop.fired 中
        numevents = aeApiPoll(eventLoop, tvp);

        /* After sleep callback. */
        // 进程阻塞后，执行钩子函数 afterSleep
        if (eventLoop->aftersleep != NULL && flags & AE_CALL_AFTER_SLEEP)
            eventLoop->aftersleep(eventLoop);
		// 处理所有已就绪的文本事件
        for (j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            int mask = eventLoop->fired[j].mask;
            int fd = eventLoop->fired[j].fd;
            int fired = 0; /* Number of events fired for current fd. */

            // 如果是 AE_READABLE 事件，则调用 rfileProc 函数处理，通常 Redis 会先处理 AE_READABLE 事件，再处理 AE_WRITABLE 事件，有助于服务器尽快			处理请求并回复结果给客户端
            int invert = fe->mask & AE_BARRIER;

            /* Note the "fe->mask & mask & ..." code: maybe an already
             * processed event removed an element that fired and we still
             * didn't processed, so we check if the event is still valid.
             *
             * Fire the readable event if the call sequence is not
             * inverted. */
            if (!invert && fe->mask & mask & AE_READABLE) {
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                fired++;
                fe = &eventLoop->events[fd]; /* Refresh in case of resize. */
            }

            /* Fire the writable event. */
            // 如果就绪的是 AE_WRITABLE 事件，则调用 wfileProc 函数处理
            if (fe->mask & mask & AE_WRITABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            /* If we have to invert the call, fire the readable event now
             * after the writable one. */
            // 如果 aeFileEvent.mask 中设置了 AE_BARRIER 标志，在这里处理 AE_REABLE 事件
            if (invert) {
                fe = &eventLoop->events[fd]; /* Refresh in case of resize. */
                if ((fe->mask & mask & AE_READABLE) &&
                    (!fired || fe->wfileProc != fe->rfileProc))
                {
                    fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            processed++;
        }
    }
    /* Check time events */
    // 处理时间事件
    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);

    return processed; /* return the number of processed file/time events */
}


static int processTimeEvents(aeEventLoop *eventLoop) {
    int processed = 0;
    aeTimeEvent *te;
    long long maxId;
	// 遍历时间事件
    te = eventLoop->timeEventHead;
    maxId = eventLoop->timeEventNextId-1;
    monotime now = getMonotonicUs();
    while(te) {
        long long id;

        /* Remove events scheduled for deletion. */
        // 该时间事件已删除，从链表中删除
        if (te->id == AE_DELETED_EVENT_ID) {
            aeTimeEvent *next = te->next;
            /* If a reference exists for this timer event,
             * don't free it. This is currently incremented
             * for recursive timerProc calls */
            if (te->refcount) {
                te = next;
                continue;
            }
            if (te->prev)
                te->prev->next = te->next;
            else
                eventLoop->timeEventHead = te->next;
            if (te->next)
                te->next->prev = te->prev;
            if (te->finalizerProc) {
                te->finalizerProc(eventLoop, te->clientData);
                now = getMonotonicUs();
            }
            zfree(te);
            te = next;
            continue;
        }

        if (te->id > maxId) {
            te = te->next;
            continue;
        }
		// 时间事件到达执行时间，执行 aeTimeEvent.timeProc 函数，该函数执行时间事件的逻辑并返回事件下次执行的间隔时间
        if (te->when <= now) {
            int retval;

            id = te->id;
            te->refcount++;
            retval = te->timeProc(eventLoop, id, te->clientData);
            te->refcount--;
            processed++;
            now = getMonotonicUs();
            if (retval != AE_NOMORE) {
                te->when = now + retval * 1000;
            } else {
                // 该事件需删除
                te->id = AE_DELETED_EVENT_ID;
            }
        }
        // 处理下一个时间事件
        te = te->next;
    }
    return processed;
}
```

![](image/QQ截图20220414144346.png)



### 多路复用与网络通信

​		传统的阻塞`I/O`模型下存在严重的性能问题，在`accept`操作后，只有读缓冲区有数据或写缓冲区有空闲空间时才能进行对应的操作，否则`CPU`不能进行任何

操作只能阻塞等待，严重浪费`CPU`性能。并且如果想在阻塞`I/O`模型下支持高并发，将需要使用大量的进程或线程，会给`CPU`造成压力，导致性能低下。

![](image/QQ截图20220415123647.png)

​		`I/O`多路复用模型，使用一个进程监听大量连接，当某个连接缓冲区状态变化`(`可读、可写`)`时，系统发送事件通知进程处理该连接的数据。当缓冲区数据

未准备好时，进程不会阻塞等待，而是去处理其他任务，当数据准备好后，进程再返回处理当前连接数据，减少进程等待时间，提供性能。

![](image/QQ截图20220415124253.png)

​		`Redis`启动时，在`initServer`函数中调用`listenToPort`函数打开套接字，并绑定端口：

```c++
// server.c
int listenToPort(int port, socketFds *sfd) {
    int j;
    char **bindaddr = server.bindaddr;
    int bindaddr_count = server.bindaddr_count;
    char *default_bindaddr[2] = {"*", "-::*"};

    /* Force binding of 0.0.0.0 if no bind address is specified. */
    // 如果配置项中没有指定 IP 地址，则将 server.bindaddr[j] 赋值为 NULL
    if (server.bindaddr_count == 0) {
        bindaddr_count = 2;
        bindaddr = default_bindaddr;
    }

    for (j = 0; j < bindaddr_count; j++) {
        char* addr = bindaddr[j];
        int optional = *addr == '-';
        if (optional) addr++;
        if (strchr(addr,':')) {
            // 绑定 IPv6 地址
            /* Bind IPv6 address. */
            sfd->fd[sfd->count] = anetTcp6Server(server.neterr,port,addr,server.tcp_backlog);
        } else {
            /* Bind IPv4 address. */
            // 绑定 IPv4 地址
            sfd->fd[sfd->count] = anetTcpServer(server.neterr,port,addr,server.tcp_backlog);
        }
        if (sfd->fd[sfd->count] == ANET_ERR) {
            int net_errno = errno;
            serverLog(LL_WARNING,
                "Warning: Could not create server TCP listening socket %s:%d: %s",
                addr, port, server.neterr);
            if (net_errno == EADDRNOTAVAIL && optional)
                continue;
            if (net_errno == ENOPROTOOPT     || net_errno == EPROTONOSUPPORT ||
                net_errno == ESOCKTNOSUPPORT || net_errno == EPFNOSUPPORT ||
                net_errno == EAFNOSUPPORT)
                continue;

            /* Rollback successful listens before exiting */
            closeSocketListeners(sfd);
            return C_ERR;
        }
        // 设置套接字为非阻塞模式
        anetNonBlock(NULL,sfd->fd[sfd->count]);
        anetCloexec(sfd->fd[sfd->count]);
        sfd->count++;
    }
    return C_OK;
}
```

```c++
// anet.c
// anetTcp6Server 与 anetTcpServer 都会调用此函数
/**
	err：记录错误信息
	port：端口
	bindaddr：地址
	af：指定 IP 类型为 IPv4 或 IPv6
*/
static int _anetTcpServer(char *err, int port, char *bindaddr, int af, int backlog)
{
    int s = -1, rv;
    char _port[6];  /* strlen("65535") */
    struct addrinfo hints, *servinfo, *p;
	// getaddrinfo 将 IP 地址解析为数值格式的 IP 地址
    snprintf(_port,6,"%d",port);
    memset(&hints,0,sizeof(hints));
    hints.ai_family = af;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;    /* No effect if bindaddr != NULL */
    if (bindaddr && !strcmp("*", bindaddr))
        bindaddr = NULL;
    if (af == AF_INET6 && bindaddr && !strcmp("::*", bindaddr))
        bindaddr = NULL;

    if ((rv = getaddrinfo(bindaddr,_port,&hints,&servinfo)) != 0) {
        anetSetError(err, "%s", gai_strerror(rv));
        return ANET_ERR;
    }
    // 处理 getaddrinfo 返回的所有 IP 地址
    for (p = servinfo; p != NULL; p = p->ai_next) {
        // 打开套接字
        if ((s = socket(p->ai_family,p->ai_socktype,p->ai_protocol)) == -1)
            continue;
		// anetV6Only 开启 IPv6 连接的 IPV6_V6ONLY 标志，限制该连接仅能发送和接收 IPv6 数据包
        if (af == AF_INET6 && anetV6Only(err,s) == ANET_ERR) goto error;
        // anetSetReuseAddr 开启 TCP 的 SO_REUSEADDR 选项，保证端口释放后可以立即被再次使用
        if (anetSetReuseAddr(err,s) == ANET_ERR) goto error;
        // 监听端口
        if (anetListen(err,s,p->ai_addr,p->ai_addrlen,backlog) == ANET_ERR) s = ANET_ERR;
        goto end;
    }
    if (p == NULL) {
        anetSetError(err, "unable to bind socket, errno: %d", errno);
        goto error;
    }

error:
    if (s != -1) close(s);
    s = ANET_ERR;
end:
    freeaddrinfo(servinfo);
    return s;
}

static int anetListen(char *err, int s, struct sockaddr *sa, socklen_t len, int backlog) {
    // 绑定端口
    if (bind(s,sa,len) == -1) {
        anetSetError(err, "bind: %s", strerror(errno));
        close(s);
        return ANET_ERR;
    }
	// 服务器套接字转换到可接收连接状态
    if (listen(s, backlog) == -1) {
        anetSetError(err, "listen: %s", strerror(errno));
        close(s);
        return ANET_ERR;
    }
    return ANET_OK;
}
```



#### 多路复用

​		`aeApiState`结构体负责存放`epoll`数据：

```c++
// ae_epoll.c
typedef struct aeApiState {
    int epfd; // epoll 专用文件描述符
    struct epoll_event *events; // epoll 的事件结构体，用于接收已就绪事件
} aeApiState;
```

​		在创建事件循环器时，会调用`aeApiCreate`初始化`I/O`多路复用机制的上下文环境：

```c++
// ae_epoll.c
static int aeApiCreate(aeEventLoop *eventLoop) {
    // 创建 aeApiState 结构体
    aeApiState *state = zmalloc(sizeof(aeApiState));

    if (!state) return -1;
    state->events = zmalloc(sizeof(struct epoll_event)*eventLoop->setsize);
    if (!state->events) {
        zfree(state);
        return -1;
    }
    // 创建 epoll 实例
    state->epfd = epoll_create(1024); /* 1024 is just a hint for the kernel */
    if (state->epfd == -1) {
        zfree(state->events);
        zfree(state);
        return -1;
    }
    anetCloexec(state->epfd);
    // 将 aeApiState 赋值给 eventLoop->apidata
    eventLoop->apidata = state;
    return 0;
}
```

​		在为事件循环器注册文本事件时会调用`aeApiAddEvent`添加对应的监听对象：

```c++
// ae_epoll.c
static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask) {
    aeApiState *state = eventLoop->apidata;
    struct epoll_event ee = {0}; /* avoid valgrind warning */
    /* If the fd was already monitored for some event, we need a MOD
     * operation. Otherwise we need an ADD operation. */
    // 如果文件描述费已存在监听对象，则使用 EPOLL_CTL_MOD 修改监听对象 ，否则使用 EPOLL_CTL_ADD 添加监听对象
    int op = eventLoop->events[fd].mask == AE_NONE ?
            EPOLL_CTL_ADD : EPOLL_CTL_MOD;
	// 将 AE 抽象层事件转化为 epoll 事件，AE_READABLE 对应 EPOLLIN 事件，AE_WRITABLE 对应 EPOLLOUT 事件
    ee.events = 0;
    mask |= eventLoop->events[fd].mask; /* Merge old events */
    if (mask & AE_READABLE) ee.events |= EPOLLIN;
    if (mask & AE_WRITABLE) ee.events |= EPOLLOUT;
    ee.data.fd = fd;
    // 向 epoll 实例添加或修改监听对象
    if (epoll_ctl(state->epfd,op,fd,&ee) == -1) return -1;
    return 0;
}
```

​		在循环调用事件循环器时会调用`aeApiPoll`阻塞进程等待时间发生或给定时间到期：

```c++
// ae_epoll.c
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;
	// 阻塞等待事件发生或给定时间到期
    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + (tvp->tv_usec + 999)/1000) : -1);
    if (retval > 0) {
        int j;

        numevents = retval;
        // 如果有就绪事件，则装载到 eventLoop->fired 中
        for (j = 0; j < numevents; j++) {
            int mask = 0;
            struct epoll_event *e = state->events+j;
			
            /**
            	EPOLLIN、EPOLLERR、EPOLLHUP 对应 AE_READABLE
            	EPOLLOUT、EPOLLERR、EPOLLHUP 对应 AE_WRITABLE
            */
            if (e->events & EPOLLIN) mask |= AE_READABLE;
            if (e->events & EPOLLOUT) mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) mask |= AE_WRITABLE|AE_READABLE;
            if (e->events & EPOLLHUP) mask |= AE_WRITABLE|AE_READABLE;
            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }
    return numevents;
}
```

​		`epoll`默认使用条件触发模式：

​				`EPOLLIN`事件：缓冲区当前可读取。

​				`EPOLLOUT`事件：缓冲区当前可写入。

​		边缘触发模式：

​				`EPOLLIN`事件：只有在收到客户端数据时才会触发一次。

​				`EPOLLOUT`事件：只有在缓冲区从不可写状态切换到可写状态，才会触发一次。



### 客户端

​		客户端是指`Redis`服务器将为每个客户连接封装为客户端，`connection`结构体负责存储每个连接的相关信息：

```c++
// connection.h
struct connection {
    ConnectionType *type; // 包含操作连接通道的函数
    ConnectionState state; // 定义连接状态
    short int flags;
    short int refs;
    int last_errno; // 最新的 errno
    void *private_data; // 存放附加数据
    ConnectionCallbackFunc conn_handler; // 执行连接操作的回调函数
    ConnectionCallbackFunc write_handler; // 执行写入操作的回调函数
    ConnectionCallbackFunc read_handler; // 执行读取操作的回调函数
    int fd; // 数据套接字描述符
};
```

​		`Redis`在收到新的连接请求时，会调用`connCreateAcceptedSocket`为网络连接创建一个`connection`，这些`connection`的`type`都指向`CT_Socket`，

`CT_Socket`指定了所有操作`Socket`连接的函数，`Redis`通过这些函数操作连接，而不是直接操作连接。

```c++
// connection.c
ConnectionType CT_Socket = {
    .ae_handler = connSocketEventHandler, // 分发函数，根据事件类型，调用真正的回调函数（connection.write_handler、connection.read_handler）
    .close = connSocketClose,
    .write = connSocketWrite,
    .read = connSocketRead,
    .accept = connSocketAccept,
    .connect = connSocketConnect,
    .set_write_handler = connSocketSetWriteHandler, // 为连接注册 write 事件回调函数
    .set_read_handler = connSocketSetReadHandler, // 为连接注册 read 事件回调函数，这两个函数会将 ae_handler 注册为回调函数
    ...
};
```

​		`client`结构体存储每个客户端的相关信息，连接建立成功后，`Redis`将创建一个`client`结构体维护客户端信息：

```c++
// server.h
typedef struct client {
    uint64_t id;            /* Client incremental unique ID. */
    connection *conn;
    int resp;               /* RESP protocol version. Can be 2 or 3. */
    redisDb *db;            /* Pointer to currently SELECTed DB. */
    robj *name;             /* As set by CLIENT SETNAME. */
    sds querybuf; // 查询缓冲区，存放客户端请求数据
    size_t qb_pos; // 查询缓冲区最新读取位置
    sds pending_querybuf;   /* If this client is flagged as master, this buffer
                               represents the yet not applied portion of the
                               replication stream that we are receiving from
                               the master. */
    size_t querybuf_peak; // 客户端单次读取请求数据量的峰值
    int argc;               /* Num of arguments of current command. */
    robj **argv;            /* Arguments of current command. */
    int original_argc;      /* Num of arguments of original command if arguments were rewritten. */
    robj **original_argv;   /* Arguments of original command if arguments were rewritten. */
    size_t argv_len_sum;    /* Sum of lengths of objects in argv list. */
    struct redisCommand *cmd, *lastcmd;  /* Last command executed. */
    user *user;             /* User associated with this connection. If the
                               user is set to NULL the connection can do
                               anything (admin). */
    int reqtype; // 请求数据协议类型
    int multibulklen; // 当前解析的命令请求中尚未处理的命令参数数量
    long bulklen; // 当前读取命令参数长度
    list *reply; // 链表回复缓冲区
    unsigned long long reply_bytes; // 链表回复缓冲区字节数
    size_t sentlen;         /* Amount of bytes already sent in the current
                               buffer or object being sent. */
    time_t ctime;           /* Client creation time. */
    long duration;          /* Current command duration. Used for measuring latency of blocking/non-blocking cmds */
    time_t lastinteraction; /* Time of the last interaction, used for timeout */
    time_t obuf_soft_limit_reached_time;
    uint64_t flags; // 客户端标志
    int authenticated;      /* Needed when the default user requires auth. */
    int replstate;          /* Replication state if this is a slave. */
    int repl_put_online_on_ack; /* Install slave write handler on first ACK. */
    int repldbfd;           /* Replication DB file descriptor. */
    off_t repldboff;        /* Replication DB file offset. */
    off_t repldbsize;       /* Replication DB file size. */
    sds replpreamble;       /* Replication DB preamble. */
    long long read_reploff; /* Read replication offset if this is a master. */
    long long reploff;      /* Applied replication offset if this is a master. */
    long long repl_ack_off; /* Replication ack offset, if this is a slave. */
    long long repl_ack_time;/* Replication ack time, if this is a slave. */
    long long repl_last_partial_write; /* The last time the server did a partial write from the RDB child pipe to this replica  */
    long long psync_initial_offset; /* FULLRESYNC reply offset other slaves
                                       copying this slave output buffer
                                       should use. */
    char replid[CONFIG_RUN_ID_SIZE+1]; /* Master replication ID (if master). */
    int slave_listening_port; /* As configured with: REPLCONF listening-port */
    char *slave_addr;       /* Optionally given by REPLCONF ip-address */
    int slave_capa;         /* Slave capabilities: SLAVE_CAPA_* bitwise OR. */
    multiState mstate;      /* MULTI/EXEC state */
    int btype;              /* Type of blocking op if CLIENT_BLOCKED. */
    blockingState bpop;     /* blocking state */
    long long woff;         /* Last write global replication offset. */
    list *watched_keys;     /* Keys WATCHED for MULTI/EXEC CAS */
    dict *pubsub_channels;  /* channels a client is interested in (SUBSCRIBE) */
    list *pubsub_patterns;  /* patterns a client is interested in (SUBSCRIBE) */
    sds peerid;             /* Cached peer ID. */
    sds sockname;           /* Cached connection target address. */
    listNode *client_list_node; /* list node in client list */
    listNode *paused_list_node; /* list node within the pause list */
    RedisModuleUserChangedFunc auth_callback; /* Module callback to execute
                                               * when the authenticated user
                                               * changes. */
    void *auth_callback_privdata; /* Private data that is passed when the auth
                                   * changed callback is executed. Opaque for
                                   * Redis Core. */
    void *auth_module;      /* The module that owns the callback, which is used
                             * to disconnect the client if the module is
                             * unloaded for cleanup. Opaque for Redis Core.*/

    /* If this client is in tracking mode and this field is non zero,
     * invalidation messages for keys fetched by this client will be send to
     * the specified client ID. */
    uint64_t client_tracking_redirection;
    rax *client_tracking_prefixes; /* A dictionary of prefixes we are already
                                      subscribed to in BCAST mode, in the
                                      context of client side caching. */
    /* In clientsCronTrackClientsMemUsage() we track the memory usage of
     * each client and add it to the sum of all the clients of a given type,
     * however we need to remember what was the old contribution of each
     * client, and in which categoty the client was, in order to remove it
     * before adding it the new value. */
    uint64_t client_cron_last_memory_usage;
    int      client_cron_last_memory_type;
    /* Response buffer */
    int bufpos; // 固定回复缓冲区最新操作位置
    char buf[PROTO_REPLY_CHUNK_BYTES]; // 固定回复缓冲区
} client;
```

​		客户端标志：

|             值              |                             描述                             |
| :-------------------------: | :----------------------------------------------------------: |
|        CLIENT_MASTER        |         客户端是主节点客户端（当前服务节点是从节点）         |
|        CLIENT_SLAVE         |         客户端是从节点客户端（当前服务节点是主节点）         |
|       CLIENT_BLOCKED        |             客户端正在被 BRPOP、BLPOP 等命令阻塞             |
|     CLIENT_PENDING_READ     |              客户端请求数据已交给 I/O 线程处理               |
|   CLIENT_PENDING_COMMAND    |     I/O 线程已处理完成客户端请求数据，主进程可以执行命令     |
|  CLIENT_CLOSE_AFTER_REPLY   | 不在执行新命令，发送完回复缓冲区的内容后立即关闭客户端。出现该标志通常由于用户对该客户端执行了CLIENT_KILL 命令，或者客户端请求数据包含了错误的协议内容 |
| CLIENT_CLOSE_AFATER_COMMAND | 执行完当前命令并返回内容后即关闭客户端，通常在 ACL 中删除某一个用户后，Redis 会对该用户客户端打开这个标志 |
|      CLIENT_CLOSE_ASAP      |                       该客户端正在关闭                       |
|        CLIENT_MULTI         |                     该客户端正在执行事务                     |
|         CLIENT_LUA          | 该客户端是在 Lua 脚本中执行 Redis 命令的伪客户端。在 Lua 脚本中通过 redis.call 等函数调用 Redis 命令，Redis 将创建一个伪客户端执行命令 |
|      CLIENT_FORCE_AOF       |    强制将执行的命令写入 AOF 文件，即使命令并没有变更数据     |
|      CLIENT_FORCE_REPL      | 强制将当前执行的命令复制给所有从节点，即使命令并没有变更数据 |
|   CLIENT_PREVENT_AOF_PROP   |    禁止将当前执行的命令写入 AOF 文件，即使命令已变更数据     |
|  CLIENT_PREVENT_REPL_PROP   |     禁止将当前执行的命令复制给从节点，即使命令已变更数据     |
|     CLIENT_PREVENT_PROP     | 禁止将当前执行的命令写入 AOF 文件及复制给从节点，即使命令已变更数据 |
|      CMD_CALL_SLOWLOG       |                          记录慢查询                          |
|       CMD_CALL_STATS        |                       统计命令执行信息                       |
|   CLIENT_TRACKING_CACHING   |                  客户端开启了客户端缓存功能                  |



#### 创建客户端

​		`Redis`启动时，会为`Socket`连接`(`监听套接字`)`注册`AE_READABLE`文件事件的回调函数`acceptTcpHandler`函数，负责接收客户端连接，创建数据交换套接

字，并为数据交换套接字注册文本事件回调函数：

```c++
// networking.c
void acceptTcpHandler(aeEventLoop *el, int fd, void *privdata, int mask) {
    int cport, cfd, max = MAX_ACCEPTS_PER_CALL;
    char cip[NET_IP_STR_LEN];
    UNUSED(el);
    UNUSED(mask);
    UNUSED(privdata);
	// 每次事件循环中最多接收 MAX_ACCEPTS_PER_CALL 个客户请求，防止短时间内处理过多客户请求导致进程阻塞
    while(max--) {
        // 接收新的客户端连接，并返回数据套接字文件描述符。
        cfd = anetTcpAccept(server.neterr, fd, cip, sizeof(cip), &cport);
        // 如果没有连接请求，返回 ANET_ERR，此时退出函数，当有新的连接请求，Redis 事件循环器会重新调用 acceptTcpHandler
        if (cfd == ANET_ERR) {
            if (errno != EWOULDBLOCK)
                serverLog(LL_WARNING,
                    "Accepting client connection: %s", server.neterr);
            return;
        }
        anetCloexec(cfd);
        serverLog(LL_VERBOSE,"Accepted %s:%d", cip, cport);
        // 创建并返回 connection 结构体，为数据套接字注册文本事件回调函数
        acceptCommonHandler(connCreateAcceptedSocket(cfd),0,cip);
    }
}


static void acceptCommonHandler(connection *conn, int flags, char *ip) {
    client *c;
    char conninfo[100];
    UNUSED(ip);

    if (connGetState(conn) != CONN_STATE_ACCEPTING) {
        serverLog(LL_VERBOSE,
            "Accepted client connection in error state: %s (conn: %s)",
            connGetLastError(conn),
            connGetInfo(conn, conninfo, sizeof(conninfo)));
        connClose(conn);
        return;
    }
    
    // 如果 client 数量加上 Cluster 连接数量已经超过 server.maxclients 配置项，则返回错误信息并关闭网络连接
    if (listLength(server.clients) + getClusterConnectionsCount()
        >= server.maxclients)
    {
        char *err;
        if (server.cluster_enabled)
            err = "-ERR max number of clients + cluster "
                  "connections reached\r\n";
        else
            err = "-ERR max number of clients reached\r\n";

        if (connWrite(conn,err,strlen(err)) == -1) {
            /* Nothing to do, Just to avoid the warning... */
        }
        server.stat_rejected_conn++;
        connClose(conn);
        return;
    }

    /* Create connection and client */
    // 创建 client 结构体，存储客户端信息
    if ((c = createClient(conn)) == NULL) {
        serverLog(LL_WARNING,
            "Error registering fd event for the new client: %s (conn: %s)",
            connGetLastError(conn),
            connGetInfo(conn, conninfo, sizeof(conninfo)));
        connClose(conn); /* May be already closed, just ignore errors */
        return;
    }

    /* Last chance to keep flags */
    c->flags |= flags;

  
    // 设置 clientAcceptHandler 为 Accept 回调函数，该函数会被立即调用，主要检查服务器是否开启 protected 模式，开启了 protected 模式而且客户端连接没有满足要求，则返回错误信息并关闭客户端
    if (connAccept(conn, clientAcceptHandler) == C_ERR) {
        char conninfo[100];
        if (connGetState(conn) == CONN_STATE_ERROR)
            serverLog(LL_WARNING,
                    "Error accepting a client connection: %s (conn: %s)",
                    connGetLastError(conn), connGetInfo(conn, conninfo, sizeof(conninfo)));
        freeClient(connGetPrivateData(conn));
        return;
    }
}


client *createClient(connection *conn) {
    client *c = zmalloc(sizeof(client));
    
    // 如果 conn 为空，则创建伪客户端，伪客户端用于支持命令不在 client 上下文执行的环境
    if (conn) {
        // 将文件描述符设置为非阻塞模式
        connNonBlock(conn);
        // 关闭 TCP 的 Delay 选项
        connEnableTcpNoDelay(conn);
        if (server.tcpkeepalive)
            // 开启 TCP 的 keepAlive 选项，服务器定时向客户端发送 ACK 进行探测
            connKeepAlive(conn,server.tcpkeepalive);
        // 为数据套接字注册 AE_READABLE 文本事件的处理函数 readQueryFromClient，会调用 ConnectionType.set_read_handler 给连接注册回调函数，当连接的 readable 事件就绪后，将触发 readQueryFromClient 函数，它将负责读取客户端发送的请求数据
        connSetReadHandler(conn, readQueryFromClient);
        // 将 client 复制给 conn.private_data
        connSetPrivateData(conn, c);
    }
	// 选择 0 号数据库并初始化 client 属性
    selectDb(c,0);
    uint64_t client_id;
    atomicGetIncr(server.next_client_id, client_id, 1);
    c->id = client_id;
    c->resp = 2;
    c->conn = conn;
    ...
    // 将 client 添加到 server.client、server.clients_index 中
    if (conn) linkClient(c);
    // 初始化 client 事务上下文
    initClientMultiState(c);
    return c;
}
```



#### 关闭客户端

​		客户端发送`quit`命令，或者`Socket`连接断开，服务器会调用`freeClientAsync`函数将客户端添加到`server.clients_to_close`中，以便后续关闭客户端。

`freeClientAsyncFreeQueue`函数`(beforeSleep`函数触发`)`会遍历`server.clients_to_close`，调用`freeClient`关闭客户端。

​		`freeClient`逻辑：

​				释放内存空间。

​				如果客户端是主节点客户端，则缓存该客户端信息，并将主从状态转换为待连接状态，以便后续与主节点重新建立连接。

​				取消所有的`Pub/Sub`订阅。

​				如果客户端是从节点客户端，则将其从`server.monitors`或`server.slaves`中剔除，并减少`server.repl_good_slaves_count`计数。

​				调用`unlinkClient`函数，关闭`Socket`连接。

​		`serverCron`时间事件会调用`clientCron`关闭超时未发送命令的客户端：

```c++
// server.c
#define CLIENTS_CRON_MIN_ITERATIONS 5
void clientsCron(void) {

    // 每次处理 numclients(客户端数量) / server.hz 个客户端，由于该函数每秒调用 server.hz 次，所以每秒都会量所有客户端处理一遍
    int numclients = listLength(server.clients);
    int iterations = numclients/server.hz;
    mstime_t now = mstime();

    if (iterations < CLIENTS_CRON_MIN_ITERATIONS)
        iterations = (numclients < CLIENTS_CRON_MIN_ITERATIONS) ?
                     numclients : CLIENTS_CRON_MIN_ITERATIONS;


    int curr_peak_mem_usage_slot = server.unixtime % CLIENTS_PEAK_MEM_USAGE_SLOTS;
    
    int zeroidx = (curr_peak_mem_usage_slot+1) % CLIENTS_PEAK_MEM_USAGE_SLOTS;
    ClientsPeakMemInput[zeroidx] = 0;
    ClientsPeakMemOutput[zeroidx] = 0;

    while(listLength(server.clients) && iterations--) {
        client *c;
        listNode *head;

        
        listRotateTailToHead(server.clients);
        head = listFirst(server.clients);
        c = listNodeValue(head);
        // clientsCronHandleTimeout 关闭超过 server.maxidletime 指定时间内没有发送请求的客户端
        if (clientsCronHandleTimeout(c,now)) continue;
        // 收缩客户端查询缓冲区以节省内存
        if (clientsCronResizeQueryBuffer(c)) continue;
        // 跟踪最近几秒内使用内存量最大的客户端，以便在 INFO 命令中提供此类信息
        if (clientsCronTrackExpansiveClients(c, curr_peak_mem_usage_slot)) continue;
        // 统计增加的内存使用量，以便在 INFO 命令中提供此类信息
        if (clientsCronTrackClientsMemUsage(c)) continue;
        if (closeClientOnOutputBufferLimitReached(c, 0)) continue;
    }
}
```

```c++
// server.c
int clientsCronResizeQueryBuffer(client *c) {
    size_t querybuf_size = sdsAllocSize(c->querybuf);
    time_t idletime = server.unixtime - c->lastinteraction;

    if (querybuf_size > PROTO_MBULK_BIG_ARG &&
         ((querybuf_size/(c->querybuf_peak+1)) > 2 ||
          idletime > 2))
    {
       
        if (sdsavail(c->querybuf) > 1024*4) {
            c->querybuf = sdsRemoveFreeSpace(c->querybuf);
        }
    }
   
    c->querybuf_peak = 0;

    if (c->flags & CLIENT_MASTER) {
       
        size_t pending_querybuf_size = sdsAllocSize(c->pending_querybuf);
        if(pending_querybuf_size > LIMIT_PENDING_QUERYBUF &&
           sdslen(c->pending_querybuf) < (pending_querybuf_size/2))
        {
            c->pending_querybuf = sdsRemoveFreeSpace(c->pending_querybuf);
        }
    }
    return 0;
}
```

​		收缩客户端查询缓冲区的条件：

​				查询缓冲区总空间大于`PROTO_MBULK_BIG_ARG`。

​				查询缓冲区总空间大于单次读取数据量峰值`(`大于`2`倍`)`，或者客户端当前处于非活跃状态。

​				查询缓冲区空闲空间大于`4KB`。



### Redis 命令执行过程

​		`RESP`协议定义了客户端与服务器通信的数据序列化协议。可以序列化整数、错误信息、单行字符串、多行字符串、数组。数据的类型通过第一个字节进行判

断，客户端请求的数据格式都是多行字符串数组。服务器需要根据不同的命令回复不同类型的响应数据。

​		`RESP`使用`\r\n`作为结束标志或者直接记录字符串长度`(`多行字符串`)`，如果没有读取到结束标志或指定长度，则认为参数没有读取完全，需要继续读取数

据，即**支持`TCP`拆包场景**；`Redis`支持客户端`pipeline`机制，即客户端在一个请求中发送多条命令，即**支持`TCP`粘包**。



#### 解析请求

```c++
// networking.c
void readQueryFromClient(connection *conn) {
    // 从 connection 中获取 client
    client *c = connGetPrivateData(conn);
    int nread, readlen;
    size_t qblen;

    // 如果开启了 I/O 线程，则交给 I/O 线程读取并解析请求数据，postponeClientRead 会将当前客户端添加到 server.clients_pending_read 中，等待 I/O 线程处理
    if (postponeClientRead(c)) return;

    /* Update total number of reads on server */
    atomicIncr(server.stat_total_reads_processed, 1);
	// readlen 为读取请求最大字节数，PROTO_IOBUF_LEN：16 KB
    readlen = PROTO_IOBUF_LEN;
    
    // 如果当前读取的是超大参数，则需要保证查询缓冲区中只有当前参数数据，PROTO_MBULK_BIG_ARG：32 KB
    // c->multibulklen 不为 0 ，代表发生了 TCP 拆包，此函数上一次并没有读取一个完整命令请求
    if (c->reqtype == PROTO_REQ_MULTIBULK && c->multibulklen && c->bulklen != -1
        && c->bulklen >= PROTO_MBULK_BIG_ARG)
    {
        ssize_t remaining = (size_t)(c->bulklen+2)-sdslen(c->querybuf);

        /* Note that the 'remaining' variable may be zero in some edge case,
         * for example once we resume a blocked client after CLIENT PAUSE. */
        if (remaining > 0 && remaining < readlen) readlen = remaining;
    }
	// 更新单次去读数据流峰值 client.querybuf_peak
    qblen = sdslen(c->querybuf);
    if (c->querybuf_peak < qblen) c->querybuf_peak = qblen;
    // sdsMakeRoomFor：扩容查询缓冲区，保证其可用内存不小于读取字节数 readlen
    c->querybuf = sdsMakeRoomFor(c->querybuf, readlen);
    // 从 socket 中读取树，该函数返回实际读取字节数
    nread = connRead(c->conn, c->querybuf+qblen, readlen);
    if (nread == -1) {
        if (connGetState(conn) == CONN_STATE_CONNECTED) {
            return;
        } else {
            serverLog(LL_VERBOSE, "Reading from client: %s",connGetLastError(c->conn));
            freeClientAsync(c);
            return;
        }
    } else if (nread == 0) {
        serverLog(LL_VERBOSE, "Client closed connection");
        freeClientAsync(c);
        return;
    } else if (c->flags & CLIENT_MASTER) {
        /* Append the query buffer to the pending (not applied) buffer
         * of the master. We'll use this buffer later in order to have a
         * copy of the string applied by the last command executed. */
        c->pending_querybuf = sdscatlen(c->pending_querybuf,
                                        c->querybuf+qblen,nread);
    }

    sdsIncrLen(c->querybuf,nread);
    c->lastinteraction = server.unixtime;
    // 如果客户端是主几点，还需要更新 client.read_reploff ，用于主从同步机制
    if (c->flags & CLIENT_MASTER) c->read_reploff += nread;
    atomicIncr(server.stat_net_input_bytes, nread);
    if (sdslen(c->querybuf) > server.client_max_querybuf_len) {
        sds ci = catClientInfoString(sdsempty(),c), bytes = sdsempty();

        bytes = sdscatrepr(bytes,c->querybuf,64);
        serverLog(LL_WARNING,"Closing client that reached max query buffer length: %s (qbuf initial bytes: %s)", ci, bytes);
        sdsfree(ci);
        sdsfree(bytes);
        freeClientAsync(c);
        return;
    }

    /* There is more data in the client input buffer, continue parsing it
     * in case to check if there is a full command to execute. */
    // 处理读取的数据
     processInputBuffer(c);
}


void processInputBuffer(client *c) {
    /* Keep processing while there is something in the input buffer */
    // client.qb_pos 为查询缓冲区最新读取位置，该位置小于查询缓冲区内容长度时， while 继续执行
    while(c->qb_pos < sdslen(c->querybuf)) {
        /**
        	直接退出的情况：
        		客户端是从服务器，而且当前服务处于 Paused 状态
        		当前服务器处于阻塞状态
        		解析数据任务已交给 I/O 线程处理
        		客户端是主节点客户端，并且当前服务器处于 Lua 脚本超时状态
        		客户端标志中存在 CLIENT_CLOSE_AFTER_REPLY、CLIENT_CLOSE_ASAP 标志，不再执行命令，尽快关闭客户端
        */
        /* Immediately abort if the client is in the middle of something. */
        if (c->flags & CLIENT_BLOCKED) break;

        /* Don't process more buffers from clients that have already pending
         * commands to execute in c->argv. */
        if (c->flags & CLIENT_PENDING_COMMAND) break;

        /* Don't process input from the master while there is a busy script
         * condition on the slave. We want just to accumulate the replication
         * stream (instead of replying -BUSY like we do with other clients) and
         * later resume the processing. */
        if (server.lua_timedout && c->flags & CLIENT_MASTER) break;

        /* CLIENT_CLOSE_AFTER_REPLY closes the connection once the reply is
         * written to the client. Make sure to not let the reply grow after
         * this flag has been set (i.e. don't process more commands).
         *
         * The same applies for clients we want to terminate ASAP. */
        if (c->flags & (CLIENT_CLOSE_AFTER_REPLY|CLIENT_CLOSE_ASAP)) break;

        /* Determine request type when unknown. */
        // 请求数据类型未确认，代表当前解析的是一个新命令请求，判断请求的数据类型
        if (!c->reqtype) {
            if (c->querybuf[c->qb_pos] == '*') {
                c->reqtype = PROTO_REQ_MULTIBULK;
            } else {
                c->reqtype = PROTO_REQ_INLINE; // 该类型用于支持 telnet 客户端发送的请求
            }
        }

        if (c->reqtype == PROTO_REQ_INLINE) {
            if (processInlineBuffer(c) != C_OK) break;
            /* If the Gopher mode and we got zero or one argument, process
             * the request in Gopher mode. To avoid data race, Redis won't
             * support Gopher if enable io threads to read queries. */
            if (server.gopher_enabled && !server.io_threads_do_reads &&
                ((c->argc == 1 && ((char*)(c->argv[0]->ptr))[0] == '/') ||
                  c->argc == 0))
            {
                processGopherRequest(c);
                resetClient(c);
                c->flags |= CLIENT_CLOSE_AFTER_REPLY;
                break;
            }
        } else if (c->reqtype == PROTO_REQ_MULTIBULK) {
            // 从请求报文中解析命令参数，如果返回 C_OK ，代表命令参数读取完毕，可以执行命令，否则就是 TCP 拆包场景
            if (processMultibulkBuffer(c) != C_OK) break;
        } else {
            serverPanic("Unknown request type");
        }

        /* Multibulk processing could see a <= 0 length. */
        // 重置客户端 ，客户端的 multibulklen、reqtype 属性都置为 0，代表当前命令请求已处理完成
        if (c->argc == 0) {
            resetClient(c);
        } else {
            /* If we are in the context of an I/O thread, we can't really
             * execute the command here. All we can do is to flag the client
             * as one that needs to process the command. */
            if (c->flags & CLIENT_PENDING_READ) {
                c->flags |= CLIENT_PENDING_COMMAND;
                break;
            }

            /* We are finally ready to execute the command. */
            // 重置客户端
            if (processCommandAndResetClient(c) == C_ERR) {
                /* If the client is no longer valid, we avoid exiting this
                 * loop and trimming the client buffer later. So we return
                 * ASAP in that case. */
                return;
            }
        }
    }

    /* Trim to pos */
    // 说明命令执行成功，抛弃查询缓冲区已处理的命令请求报文
    if (c->qb_pos) {
        sdsrange(c->querybuf,c->qb_pos,-1);
        c->qb_pos = 0;
    }
}


int processMultibulkBuffer(client *c) {
    char *newline = NULL;
    int ok;
    long long ll;
	// 上一个命令请求数据已解析完成，开始一个新的命令请求。通过 \r\n 分隔符从当前请求数据中解析当前命令参数数量，赋值给 client.multibulklen
    if (c->multibulklen == 0) {
        /* The client should have been reset */
        serverAssertWithInfo(c,NULL,c->argc == 0);

        /* Multi bulk length cannot be read without a \r\n */
        newline = strchr(c->querybuf+c->qb_pos,'\r');
        if (newline == NULL) {
            if (sdslen(c->querybuf)-c->qb_pos > PROTO_INLINE_MAX_SIZE) {
                addReplyError(c,"Protocol error: too big mbulk count string");
                setProtocolError("too big mbulk count string",c);
            }
            return C_ERR;
        }

        /* Buffer should also contain \n */
        if (newline-(c->querybuf+c->qb_pos) > (ssize_t)(sdslen(c->querybuf)-c->qb_pos-2))
            return C_ERR;

        /* We know for sure there is a whole line since newline != NULL,
         * so go ahead and find out the multi bulk length. */
        serverAssertWithInfo(c,NULL,c->querybuf[c->qb_pos] == '*');
        ok = string2ll(c->querybuf+1+c->qb_pos,newline-(c->querybuf+1+c->qb_pos),&ll);
        if (!ok || ll > 1024*1024) {
            addReplyError(c,"Protocol error: invalid multibulk length");
            setProtocolError("invalid mbulk count",c);
            return C_ERR;
        } else if (ll > 10 && authRequired(c)) {
            addReplyError(c, "Protocol error: unauthenticated multibulk length");
            setProtocolError("unauth mbulk count", c);
            return C_ERR;
        }

        c->qb_pos = (newline-c->querybuf)+2;

        if (ll <= 0) return C_OK;

        c->multibulklen = ll;

        /* Setup argv array on client structure */
        if (c->argv) zfree(c->argv);
        c->argv = zmalloc(sizeof(robj*)*c->multibulklen);
        c->argv_len_sum = 0;
    }

    serverAssertWithInfo(c,NULL,c->multibulklen > 0);
    // 读取当前命令的所有参数
    while(c->multibulklen) {
        /* Read bulk length if unknown */
        if (c->bulklen == -1) {
            // 通过 \r\n 分隔符读取当前参数长度，赋值给 client.bulklen
            newline = strchr(c->querybuf+c->qb_pos,'\r');
            if (newline == NULL) {
                if (sdslen(c->querybuf)-c->qb_pos > PROTO_INLINE_MAX_SIZE) {
                    addReplyError(c,
                        "Protocol error: too big bulk count string");
                    setProtocolError("too big bulk count string",c);
                    return C_ERR;
                }
                break;
            }

            /* Buffer should also contain \n */
            if (newline-(c->querybuf+c->qb_pos) > (ssize_t)(sdslen(c->querybuf)-c->qb_pos-2))
                break;

            if (c->querybuf[c->qb_pos] != '$') {
                addReplyErrorFormat(c,
                    "Protocol error: expected '$', got '%c'",
                    c->querybuf[c->qb_pos]);
                setProtocolError("expected $ but got something else",c);
                return C_ERR;
            }

            ok = string2ll(c->querybuf+c->qb_pos+1,newline-(c->querybuf+c->qb_pos+1),&ll);
            if (!ok || ll < 0 ||
                (!(c->flags & CLIENT_MASTER) && ll > server.proto_max_bulk_len)) {
                addReplyError(c,"Protocol error: invalid bulk length");
                setProtocolError("invalid bulk length",c);
                return C_ERR;
            } else if (ll > 16384 && authRequired(c)) {
                addReplyError(c, "Protocol error: unauthenticated bulk length");
                setProtocolError("unauth bulk length", c);
                return C_ERR;
            }

            c->qb_pos = newline-c->querybuf+2;
            // 如果当前参数是一个超大参数，则清除查询缓冲区中其他参数的数据，确保查询缓冲区只有当前参数数据，并对查询缓冲区扩容，确保可以容纳当前参数
            if (ll >= PROTO_MBULK_BIG_ARG) {

                if (sdslen(c->querybuf)-c->qb_pos <= (size_t)ll+2) {
                    sdsrange(c->querybuf,c->qb_pos,-1);
                    c->qb_pos = 0;
                    /* Hint the sds library about the amount of bytes this string is
                     * going to contain. */
                    c->querybuf = sdsMakeRoomFor(c->querybuf,ll+2-sdslen(c->querybuf));
                }
            }
            c->bulklen = ll;
        }

        /* Read bulk argument */
        if (sdslen(c->querybuf)-c->qb_pos < (size_t)(c->bulklen+2)) {
            /* Not enough data (+2 == trailing \r\n) */
            // 当前查询缓冲区字符串长度小于当前参数长度，说明当前参数并没有读取完整，退出，等待下一个被调用继续读取剩余数据
            break;
        } else {
            /* Optimization: if the buffer contains JUST our bulk element
             * instead of creating a new object by *copying* the sds we
             * just use the current sds string. */
            if (c->qb_pos == 0 &&
                c->bulklen >= PROTO_MBULK_BIG_ARG &&
                sdslen(c->querybuf) == (size_t)(c->bulklen+2))
            {
                // 读取超大参数，直接使用查询缓冲区创建一个 redisObject 作为参数（redisObject.ptr指向查询缓冲区），并申请新的内存空间作为查询缓冲区
                c->argv[c->argc++] = createObject(OBJ_STRING,c->querybuf);
                c->argv_len_sum += c->bulklen;
                sdsIncrLen(c->querybuf,-2); /* remove CRLF */
                /* Assume that if we saw a fat argument we'll see another one
                 * likely... */
                c->querybuf = sdsnewlen(SDS_NOINIT,c->bulklen+2);
                sdsclear(c->querybuf);
            } else {
                // 读取非超大参数，则赋值查询缓冲区中的数据并创建一个 redisObject作为参数
                c->argv[c->argc++] =
                    createStringObject(c->querybuf+c->qb_pos,c->bulklen);
                c->argv_len_sum += c->bulklen;
                c->qb_pos += c->bulklen+2;
            }
            c->bulklen = -1;
            c->multibulklen--;
        }
    }

    /* We're done when c->multibulk == 0 */
    // 代表当前命令数据已读取完全，返回 C_OK，返回 processInputBuffer 函数后会执行命令
    if (c->multibulklen == 0) return C_OK;

    /* Still not ready to process the command */
    // 需要 readQueryFromClient 函数下次执行时继续读取数据
    return C_ERR;
}
```



#### 返回响应

​		`client`定义了两个回复缓冲区，用于缓存返回给客户端的响应数据。

​				`client.buf`：字符数组，大小为`16KB`，`bufpos`记录最新写入位置。

​				`client.reply`：`clientReplyBlock`结构体链表。

```c++
typedef struct clientReplyBlock {
    size_t size, used; // buf 数组总长度和已使用长度
    char buf[]; // 存储数据
} clientReplyBlock;
```

​		响应数据小于`16KB`，使用`client.buf`。`client.buf`和`client`结构体存在同一块内存中，减少内存分配次数及内存碎片。大于`16KB`时，才使用

`client.reply`。

​		响应数据写入回复缓冲区：

​				1、按`RESP`协议处理数据。

​				2、先尝试写入`client.buf`，如果`client.buf`写不下，则写入`client.reply`。

​				3、每次写入前，调用`prepareClientToWrite`函数，将当前`client`添加到`server.clients_pending_write`中。

​		返回响应数据的最后一步是将回复缓冲区数据写入`TCP`发送缓冲区：

```c++
int handleClientsWithPendingWrites(void) {
    // 遍历 server.clients_pending_write
    listIter li;
    listNode *ln;
    int processed = listLength(server.clients_pending_write);

    listRewind(server.clients_pending_write,&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_WRITE;
        listDelNode(server.clients_pending_write,ln);

        /* If a client is protected, don't do anything,
         * that may trigger write error or recreate handler. */
        if (c->flags & CLIENT_PROTECTED) continue;

        /* Don't write to clients that are going to be closed anyway. */
        if (c->flags & CLIENT_CLOSE_ASAP) continue;

        /* Try to write buffers to the client socket. */
        // 将 client 回复缓冲区内容写入 TCP 发送缓冲区
        if (writeToClient(c,0) == C_ERR) continue;

        // 如果回复缓冲区还有数据，则为当前连接注册 WRITABLE 文件事件回调函数，等到 TCP 发送缓冲区可写后，继续写入数据
        if (clientHasPendingReplies(c)) {
            int ae_barrier = 0;

            if (server.aof_state == AOF_ON &&
                server.aof_fsync == AOF_FSYNC_ALWAYS)
            {
                ae_barrier = 1;
            }
            if (connSetWriteHandlerWithBarrier(c->conn, sendReplyToClient, ae_barrier) == C_ERR) {
                freeClientAsync(c);
            }
        }
    }
    return processed;
}
```



#### 执行命令

​		经过解析后命令参数存储在`client.argv`中，`client.argv[0]`是命令名，后面是执行命令的参数。

​		`redisCommand`存储了`Redis`命令的信息：

```c++
// server.h
struct redisCommand {
    char *name; // 命令名称
    redisCommandProc *proc; // 命令处理函数
    int arity; // 命令参数数量
    char *sflags;   
    uint64_t flags; 
    redisGetKeysProc *getkeys_proc;
    int firstkey; /* The first argument that's a key (0 = no keys) */
    int lastkey;  /* The last argument that's a key */
    int keystep;  /* The step between first and last key */
    long long microseconds, calls, rejected_calls, failed_calls;
    int id;     
};
```

​		`redisCommandTable`数组，存放了服务器所有支持的命令：

```c++
// server.c
struct redisCommand redisCommandTable[] = {
    {"module",moduleCommand,-2,
     "admin no-script",
     0,NULL,0,0,0,0,0,0},

    {"get",getCommand,2,
     "read-only fast @string",
     0,NULL,1,1,1,0,0,0},

    {"getex",getexCommand,-2,
     "write fast @string",
     0,NULL,1,1,1,0,0,0},
	
    ...
}
```

​		`Redis`启动时，调用`populateCommandTable`加载`redisCommandTable` 数据，将命令名和命令记录到`server.commands`命令字典中。

`processCommandAndResetClient`函数调用`processCommand`函数执行命令，并在命令执行后调用`commandProcessed`执行后续逻辑。

```c++
// server.c
int processCommand(client *c) {
    if (!server.lua_timedout) {

        serverAssert(!server.propagate_in_transaction);
        serverAssert(!server.in_exec);
        serverAssert(!server.in_eval);
    }
	// 触发 Modle Filter
    moduleCallCommandFilters(c);

    // 针对 quit 命令处理，给 client 添加 CLIENT_CLOSE_AFTER_REPLY 标志，退出
    if (!strcasecmp(c->argv[0]->ptr,"quit")) {
        addReply(c,shared.ok);
        c->flags |= CLIENT_CLOSE_AFTER_REPLY;
        return C_ERR;
    }

    // 使用命令名，从 server.commands 命令字典中查找对应的 redisCommand，并检查参数数量是否满足命令要求
    c->cmd = c->lastcmd = lookupCommand(c->argv[0]->ptr);
    if (!c->cmd) {
        sds args = sdsempty();
        int i;
        for (i=1; i < c->argc && sdslen(args) < 128; i++)
            args = sdscatprintf(args, "`%.*s`, ", 128-(int)sdslen(args), (char*)c->argv[i]->ptr);
        rejectCommandFormat(c,"unknown command `%s`, with args beginning with: %s",
            (char*)c->argv[0]->ptr, args);
        sdsfree(args);
        return C_OK;
    } else if ((c->cmd->arity > 0 && c->cmd->arity != c->argc) ||
               (c->argc < -c->cmd->arity)) {
        rejectCommandFormat(c,"wrong number of arguments for '%s' command",
            c->cmd->name);
        return C_OK;
    }

    int is_read_command = (c->cmd->flags & CMD_READONLY) ||
                           (c->cmd->proc == execCommand && (c->mstate.cmd_flags & CMD_READONLY));
    int is_write_command = (c->cmd->flags & CMD_WRITE) ||
                           (c->cmd->proc == execCommand && (c->mstate.cmd_flags & CMD_WRITE));
    int is_denyoom_command = (c->cmd->flags & CMD_DENYOOM) ||
                             (c->cmd->proc == execCommand && (c->mstate.cmd_flags & CMD_DENYOOM));
    int is_denystale_command = !(c->cmd->flags & CMD_STALE) ||
                               (c->cmd->proc == execCommand && (c->mstate.cmd_inv_flags & CMD_STALE));
    int is_denyloading_command = !(c->cmd->flags & CMD_LOADING) ||
                                 (c->cmd->proc == execCommand && (c->mstate.cmd_inv_flags & CMD_LOADING));
    int is_may_replicate_command = (c->cmd->flags & (CMD_WRITE | CMD_MAY_REPLICATE)) ||
                                   (c->cmd->proc == execCommand && (c->mstate.cmd_flags & (CMD_WRITE | CMD_MAY_REPLICATE)));
	// 如果服务器要求客户端身份验证，则检查客户端是否通过身份验证，未通过验证的客户端只能执行 auth 命令，将拒绝其他命令
    if (authRequired(c)) {
        /* AUTH and HELLO and no auth commands are valid even in
         * non-authenticated state. */
        if (!(c->cmd->flags & CMD_NO_AUTH)) {
            rejectCommand(c,shared.noautherr);
            return C_OK;
        }
    }

	// 根据 ACL 权限控制列表，检查该客户端使用有权限执行该命令
    int acl_errpos;
    int acl_retval = ACLCheckAllPerm(c,&acl_errpos);
    if (acl_retval != ACL_OK) {
        addACLLogEntry(c,acl_retval,acl_errpos,NULL);
        switch (acl_retval) {
        case ACL_DENIED_CMD:
            rejectCommandFormat(c,
                "-NOPERM this user has no permissions to run "
                "the '%s' command or its subcommand", c->cmd->name);
            break;
        case ACL_DENIED_KEY:
            rejectCommandFormat(c,
                "-NOPERM this user has no permissions to access "
                "one of the keys used as arguments");
            break;
        case ACL_DENIED_CHANNEL:
            rejectCommandFormat(c,
                "-NOPERM this user has no permissions to access "
                "one of the channels used as arguments");
            break;
        default:
            rejectCommandFormat(c, "no permission");
            break;
        }
        return C_OK;
    }

	// 如果该服务器运行在 Cluster 模式下，并且该节点不是该命令的键的存储节点，则使用 ASK 或 MOVED 通知客户端请求真正的存储节点
    if (server.cluster_enabled &&
        !(c->flags & CLIENT_MASTER) &&
        !(c->flags & CLIENT_LUA &&
          server.lua_caller->flags & CLIENT_MASTER) &&
        !(!cmdHasMovableKeys(c->cmd) && c->cmd->firstkey == 0 &&
          c->cmd->proc != execCommand))
    {
        int hashslot;
        int error_code;
        clusterNode *n = getNodeByQuery(c,c->cmd,c->argv,c->argc,
                                        &hashslot,&error_code);
        if (n == NULL || n != server.cluster->myself) {
            if (c->cmd->proc == execCommand) {
                discardTransaction(c);
            } else {
                flagTransaction(c);
            }
            clusterRedirectClient(c,n,hashslot,error_code);
            c->cmd->rejected_calls++;
            return C_OK;
        }
    }

    // 如果配置了内存最大限制，则检查内存占用情况，并在有需要时进行数据淘汰，如果数据淘汰失败，则拒绝命令
    if (server.maxmemory && !server.lua_timedout) {
        int out_of_memory = (performEvictions() == EVICT_FAIL);
        /* performEvictions may flush slave output buffers. This may result
         * in a slave, that may be the active client, to be freed. */
        if (server.current_client == NULL) return C_ERR;

        int reject_cmd_on_oom = is_denyoom_command;
        /* If client is in MULTI/EXEC context, queuing may consume an unlimited
         * amount of memory, so we want to stop that.
         * However, we never want to reject DISCARD, or even EXEC (unless it
         * contains denied commands, in which case is_denyoom_command is already
         * set. */
        if (c->flags & CLIENT_MULTI &&
            c->cmd->proc != execCommand &&
            c->cmd->proc != discardCommand &&
            c->cmd->proc != resetCommand) {
            reject_cmd_on_oom = 1;
        }

        if (out_of_memory && reject_cmd_on_oom) {
            rejectCommand(c, shared.oomerr);
            return C_OK;
        }

        /* Save out_of_memory result at script start, otherwise if we check OOM
         * until first write within script, memory used by lua stack and
         * arguments might interfere. */
        if (c->cmd->proc == evalCommand || c->cmd->proc == evalShaCommand) {
            server.lua_oom = out_of_memory;
        }
    }

    /* Make sure to use a reasonable amount of memory for client side
     * caching metadata. */
    // Tracking 机制要求服务记录客户端查询过的键，如果服务器记录的键的数量大于 server.tracking_table_max_keys 配置，那么随机删除其中一些键，并向对应的客户端发送失败消息
    if (server.tracking_clients) trackingLimitUsedSlots();

    /* Don't accept write commands if there are problems persisting on disk
     * and if this is a master instance. */
    // 该服务器是主节点并且当前存在持久化错误，则拒绝命令
    int deny_write_type = writeCommandsDeniedByDiskError();
    if (deny_write_type != DISK_ERROR_TYPE_NONE &&
        server.masterhost == NULL &&
        (is_write_command ||c->cmd->proc == pingCommand))
    {
        if (deny_write_type == DISK_ERROR_TYPE_RDB)
            rejectCommand(c, shared.bgsaveerr);
        else
            rejectCommandFormat(c,
                "-MISCONF Errors writing to the AOF file: %s",
                strerror(server.aof_last_write_errno));
        return C_OK;
    }

    /* Don't accept write commands if there are not enough good slaves and
     * user configured the min-slaves-to-write option. */
    // 如果该服务器是主节点并且正常从服务器数量小于 server.min-replicas-to-write 配置，则拒绝命令
    if (server.masterhost == NULL &&
        server.repl_min_slaves_to_write &&
        server.repl_min_slaves_max_lag &&
        is_write_command &&
        server.repl_good_slaves_count < server.repl_min_slaves_to_write)
    {
        rejectCommand(c, shared.noreplicaserr);
        return C_OK;
    }

    /* Don't accept write commands if this is a read only slave. But
     * accept write commands if this is our master. */
    // 该服务器是从节点并且客户端非主节点客户端，则拒绝命令
    if (server.masterhost && server.repl_slave_ro &&
        !(c->flags & CLIENT_MASTER) &&
        is_write_command)
    {
        rejectCommand(c, shared.roslaveerr);
        return C_OK;
    }
	
    // 处于 Pub/Sub 模式下，并且使用 RESP2 协议，只支持 PING、SUBSCRIBE、UNSUBSCRIBE、PSUBSCRIBE、PUNSUBSCRIBE命令，拒绝其他命令
    if ((c->flags & CLIENT_PUBSUB && c->resp == 2) &&
        c->cmd->proc != pingCommand &&
        c->cmd->proc != subscribeCommand &&
        c->cmd->proc != unsubscribeCommand &&
        c->cmd->proc != psubscribeCommand &&
        c->cmd->proc != punsubscribeCommand &&
        c->cmd->proc != resetCommand) {
        rejectCommandFormat(c,
            "Can't execute '%s': only (P)SUBSCRIBE / "
            "(P)UNSUBSCRIBE / PING / QUIT / RESET are allowed in this context",
            c->cmd->name);
        return C_OK;
    }

    /* Only allow commands with flag "t", such as INFO, SLAVEOF and so on,
     * when slave-serve-stale-data is no and we are a slave with a broken
     * link with master. */
    // 该服务器是从节点并且与主节点处于断连状态，拒绝查询数据的命令。可以通过关闭服务器 server.repl+serve_stable_data 配置跳过该检查，允许从服务器返回过期数据
    if (server.masterhost && server.repl_state != REPL_STATE_CONNECTED &&
        server.repl_serve_stale_data == 0 &&
        is_denystale_command)
    {
        rejectCommand(c, shared.masterdownerr);
        return C_OK;
    }

    /* Loading DB? Return an error if the command has not the
     * CMD_LOADING flag. */
    // 服务正在加载树，只有特定命令能执行
    if (server.loading && is_denyloading_command) {
        rejectCommand(c, shared.loadingerr);
        return C_OK;
    }

	// 服务器处于 Lua 脚本超时状态，只有特定命令能执行
    if (server.lua_timedout &&
          c->cmd->proc != authCommand &&
          c->cmd->proc != helloCommand &&
          c->cmd->proc != replconfCommand &&
          c->cmd->proc != multiCommand &&
          c->cmd->proc != discardCommand &&
          c->cmd->proc != watchCommand &&
          c->cmd->proc != unwatchCommand &&
          c->cmd->proc != resetCommand &&
        !(c->cmd->proc == shutdownCommand &&
          c->argc == 2 &&
          tolower(((char*)c->argv[1]->ptr)[0]) == 'n') &&
        !(c->cmd->proc == scriptCommand &&
          c->argc == 2 &&
          tolower(((char*)c->argv[1]->ptr)[0]) == 'k'))
    {
        rejectCommand(c, shared.slowscripterr);
        return C_OK;
    }

    /* Prevent a replica from sending commands that access the keyspace.
     * The main objective here is to prevent abuse of client pause check
     * from which replicas are exempt. */
    if ((c->flags & CLIENT_SLAVE) && (is_may_replicate_command || is_write_command || is_read_command)) {
        rejectCommandFormat(c, "Replica can't interract with the keyspace");
        return C_OK;
    }

    /* If the server is paused, block the client until
     * the pause has ended. Replicas are never paused. */
    if (!(c->flags & CLIENT_SLAVE) && 
        ((server.client_pause_type == CLIENT_PAUSE_ALL) ||
        (server.client_pause_type == CLIENT_PAUSE_WRITE && is_may_replicate_command)))
    {
        c->bpop.timeout = 0;
        blockClient(c,BLOCKED_PAUSE);
        return C_OK;       
    }

    /* Exec the command */
    // 如果当前 client 处于事务上下文中，则除 exec、discard、multi、watch 外的命令都会被入队到事务队列中，否则执行命令
    if (c->flags & CLIENT_MULTI &&
        c->cmd->proc != execCommand && c->cmd->proc != discardCommand &&
        c->cmd->proc != multiCommand && c->cmd->proc != watchCommand &&
        c->cmd->proc != resetCommand)
    {
        queueMultiCommand(c);
        addReply(c,shared.queued);
    } else {
        call(c,CMD_CALL_FULL);
        c->woff = server.master_repl_offset;
        if (listLength(server.ready_keys))
            handleClientsBlockedOnKeys();
    }

    return C_OK;
}


void call(client *c, int flags) {
    long long dirty;
    monotime call_timer;
    int client_old_flags = c->flags;
    struct redisCommand *real_cmd = c->cmd;
    static long long prev_err_count;

	// 命令执行前，重置传播控制标志
    c->flags &= ~(CLIENT_FORCE_AOF|CLIENT_FORCE_REPL|CLIENT_PREVENT_PROP);
    redisOpArray prev_also_propagate = server.also_propagate;
    redisOpArrayInit(&server.also_propagate);

    /* Call the command. */
    // 调用命令处理函数 redisCommand.proc ，执行命令处理逻辑
    dirty = server.dirty;
    prev_err_count = server.stat_total_error_replies;

    /* Update cache time, in case we have nested calls we want to
     * update only on the first call*/
    if (server.fixed_time_expire++ == 0) {
        updateCachedTime(0);
    }

    elapsedStart(&call_timer);
    c->cmd->proc(c);
    const long duration = elapsedUs(call_timer);
    c->duration = duration;
    dirty = server.dirty-dirty;
    if (dirty < 0) dirty = 0;

    /* Update failed command calls if required.
     * We leverage a static variable (prev_err_count) to retain
     * the counter across nested function calls and avoid logging
     * the same error twice. */
    if ((server.stat_total_error_replies - prev_err_count) > 0) {
        real_cmd->failed_calls++;
    }

    /* After executing command, we will close the client after writing entire
     * reply if it is set 'CLIENT_CLOSE_AFTER_COMMAND' flag. */
    // 将 CLIENT_CLOSE_AFTER_COMMAND 替换为 CLIENT_CLOSE_AFTER_REPLY
    if (c->flags & CLIENT_CLOSE_AFTER_COMMAND) {
        c->flags &= ~CLIENT_CLOSE_AFTER_COMMAND;
        c->flags |= CLIENT_CLOSE_AFTER_REPLY;
    }

    /* When EVAL is called loading the AOF we don't want commands called
     * from Lua to go into the slowlog or to populate statistics. */
    // 如果当前正加载数据并且当前命令执行的是 Lua 脚本，则清除慢日志，命令统计这两个客户端标志，即该命令既不输出慢日志，也不添加到命令统计中
    if (server.loading && c->flags & CLIENT_LUA)
        flags &= ~(CMD_CALL_SLOWLOG | CMD_CALL_STATS);

    /* If the caller is Lua, we want to force the EVAL caller to propagate
     * the script if the command flag or client flag are forcing the
     * propagation. */
    // 当前客户端是 Lua 脚本伪客户端，将该客户端的 CLIENT_FORCE_REPL、CLIENT_FORCE_AOF 标志转移到只是客户端中
    if (c->flags & CLIENT_LUA && server.lua_caller) {
        if (c->flags & CLIENT_FORCE_REPL)
            server.lua_caller->flags |= CLIENT_FORCE_REPL;
        if (c->flags & CLIENT_FORCE_AOF)
            server.lua_caller->flags |= CLIENT_FORCE_AOF;
    }

    /* Note: the code below uses the real command that was executed
     * c->cmd and c->lastcmd may be different, in case of MULTI-EXEC or
     * re-written commands such as EXPIRE, GEOADD, etc. */

    /* Record the latency this command induced on the main thread.
     * unless instructed by the caller not to log. (happens when processing
     * a MULTI-EXEC from inside an AOF). */
    if (flags & CMD_CALL_SLOWLOG) {
        char *latency_event = (real_cmd->flags & CMD_FAST) ?
                               "fast-command" : "command";
        latencyAddSampleIfNeeded(latency_event,duration/1000);
    }

    /* Log the command into the Slow log if needed.
     * If the client is blocked we will handle slowlog when it is unblocked. */
    // 记录慢日志并统计命令信息
    if ((flags & CMD_CALL_SLOWLOG) && !(c->flags & CLIENT_BLOCKED))
        slowlogPushCurrentCommand(c, real_cmd, duration);

    // 发送命令信息给监控模式下的客户端
    if (!(c->cmd->flags & (CMD_SKIP_MONITOR|CMD_ADMIN))) {
        robj **argv = c->original_argv ? c->original_argv : c->argv;
        int argc = c->original_argv ? c->original_argc : c->argc;
        replicationFeedMonitors(c,server.monitors,c->db->id,argv,argc);
    }

    /* Clear the original argv.
     * If the client is blocked we will handle slowlog when it is unblocked. */
    if (!(c->flags & CLIENT_BLOCKED))
        freeClientOriginalArgv(c);

    /* populate the per-command statistics that we show in INFO commandstats. */
    if (flags & CMD_CALL_STATS) {
        real_cmd->microseconds += duration;
        real_cmd->calls++;
    }

    /* Propagate the command into the AOF and replication link */
    if (flags & CMD_CALL_PROPAGATE &&
        (c->flags & CLIENT_PREVENT_PROP) != CLIENT_PREVENT_PROP)
    {
        int propagate_flags = PROPAGATE_NONE;

        // 如果命令修改了数据，则 propagate_flags 添加 PROPAGATE_AOF、PROPAGATE_REPL 标志
        if (dirty) propagate_flags |= (PROPAGATE_AOF|PROPAGATE_REPL);

        // client 打开了 CLIENT_FORCE_REPL、CLIENT_FORCE_AOF 标志，则 propagate_flags 添加 PROPAGATE_AOF、PROPAGATE_REPL 标志
        if (c->flags & CLIENT_FORCE_REPL) propagate_flags |= PROPAGATE_REPL;
        if (c->flags & CLIENT_FORCE_AOF) propagate_flags |= PROPAGATE_AOF;

        /* However prevent AOF / replication propagation if the command
         * implementation called preventCommandPropagation() or similar,
         * or if we don't have the call() flags to do so. */
        // client 被打开了 CLIENT_PREVENT_REPL_PROP、CLIENT_PREVENT_AOF_PROP 标志，则 propagate_flags 清除 PROPAGATE_AOF、PROPAGATE_REPL 标志
        if (c->flags & CLIENT_PREVENT_REPL_PROP ||
            !(flags & CMD_CALL_PROPAGATE_REPL))
                propagate_flags &= ~PROPAGATE_REPL;
        if (c->flags & CLIENT_PREVENT_AOF_PROP ||
            !(flags & CMD_CALL_PROPAGATE_AOF))
                propagate_flags &= ~PROPAGATE_AOF;

        if (propagate_flags != PROPAGATE_NONE && !(c->cmd->flags & CMD_MODULE))
            /**
            	根据 propagate_flags，将命令记录到 AOF 文件或复制到从服务器中。
            */
            propagate(c->cmd,c->db->id,c->argv,c->argc,propagate_flags);
    }
	// 命令执行前清除 client 中的传播控制标志。如果 client.flags 中本来存在这些标志，则将它们重新赋值给 client.flags
    c->flags &= ~(CLIENT_FORCE_AOF|CLIENT_FORCE_REPL|CLIENT_PREVENT_PROP);
    c->flags |= client_old_flags &
        (CLIENT_FORCE_AOF|CLIENT_FORCE_REPL|CLIENT_PREVENT_PROP);

    /* Handle the alsoPropagate() API to handle commands that want to propagate
     * multiple separated commands. Note that alsoPropagate() is not affected
     * by CLIENT_PREVENT_PROP flag. */
    // server.also_propagate.numops 存放了一系列需额外传播的命令，将它记录到 AOF 或复制到从服务器中
    if (server.also_propagate.numops) {
        int j;
        redisOp *rop;

        if (flags & CMD_CALL_PROPAGATE) {
            int multi_emitted = 0;
            /* Wrap the commands in server.also_propagate array,
             * but don't wrap it if we are already in MULTI context,
             * in case the nested MULTI/EXEC.
             *
             * And if the array contains only one command, no need to
             * wrap it, since the single command is atomic. */
            if (server.also_propagate.numops > 1 &&
                !(c->cmd->flags & CMD_MODULE) &&
                !(c->flags & CLIENT_MULTI) &&
                !(flags & CMD_CALL_NOWRAP))
            {
                execCommandPropagateMulti(c->db->id);
                multi_emitted = 1;
            }

            for (j = 0; j < server.also_propagate.numops; j++) {
                rop = &server.also_propagate.ops[j];
                int target = rop->target;
                /* Whatever the command wish is, we honor the call() flags. */
                if (!(flags&CMD_CALL_PROPAGATE_AOF)) target &= ~PROPAGATE_AOF;
                if (!(flags&CMD_CALL_PROPAGATE_REPL)) target &= ~PROPAGATE_REPL;
                if (target)
                    propagate(rop->cmd,rop->dbid,rop->argv,rop->argc,target);
            }

            if (multi_emitted) {
                execCommandPropagateExec(c->db->id);
            }
        }
        redisOpArrayFree(&server.also_propagate);
    }
    server.also_propagate = prev_also_propagate;

    /* Client pause takes effect after a transaction has finished. This needs
     * to be located after everything is propagated. */
    if (!server.in_exec && server.client_pause_in_transaction) {
        server.client_pause_in_transaction = 0;
    }

    /* If the client has keys tracking enabled for client side caching,
     * make sure to remember the keys it fetched via this command. */
    // 如果是查询命令，Redis 会记住该命令，后续当查询的键发生变化时，需要通知客户端。Redis 6 新增的 Tracking 机制
    if (c->cmd->flags & CMD_READONLY) {
        client *caller = (c->flags & CLIENT_LUA && server.lua_caller) ?
                            server.lua_caller : c;
        if (caller->flags & CLIENT_TRACKING &&
            !(caller->flags & CLIENT_TRACKING_BCAST))
        {
            trackingRememberKeys(caller);
        }
    }

    server.fixed_time_expire--;
    server.stat_numcommands++;
    prev_err_count = server.stat_total_error_replies;

    /* Record peak memory after each command and before the eviction that runs
     * before the next command. */
    size_t zmalloc_used = zmalloc_used_memory();
    if (zmalloc_used > server.stat_peak_memory)
        server.stat_peak_memory = zmalloc_used;
}
```

​		命令执行完成后，`commandProcessed`执行后续逻辑：

```c++
void commandProcessed(client *c) {

    if (c->flags & CLIENT_BLOCKED) return;
	// 重置客户端
    resetClient(c);

    long long prev_offset = c->reploff;
    // 客户端是主节点客户端，并且客户端不处于事务上下文中，则更新 client.reploff ，该属性记录当前服务器已同步命令偏移量，用于主从同步机制
    if (c->flags & CLIENT_MASTER && !(c->flags & CLIENT_MULTI)) {
        /* Update the applied replication offset of our master. */
        c->reploff = c->read_reploff - sdslen(c->querybuf) + c->qb_pos;
    }

    if (c->flags & CLIENT_MASTER) {
        long long applied = c->reploff - prev_offset;
        if (applied) {
            // 将接收的命令继续复制到当前服务器的从节点
            replicationFeedSlavesFromMasterStream(server.slaves,
                    c->pending_querybuf, applied);
            sdsrange(c->pending_querybuf,applied,-1);
        }
    }
}
```



### 网络 I/O 线程

​		在`Redis 6`之前，网络`I/O`数据的读`/`写是单线程串行处理的，在数据吞吐量特别大时会成为`Redis`主要性能瓶颈之一。`Redis 6`创建了`I/O`线程，并将不

同客户端的`I/O`数据的读`/`写操作分配到不同的`I/O`线程中进行处理，从而提高网络`I/O`性能。可以通过`io-threads`进行配置，默认为`1`，即使用单线程。

#### 初始化 I/O 线程

```c++
// networking.c
pthread_t io_threads[IO_THREADS_MAX_NUM]; // 存储所有线程的线程描述符 pthread_t：作为线程的唯一标志
pthread_mutex_t io_threads_mutex[IO_THREADS_MAX_NUM]; // 用于启停 I/O 线程的互斥量
redisAtomic unsigned long io_threads_pending[IO_THREADS_MAX_NUM]; // 每个客户端待处理客户端数量，Atomic 定义原子类型变量，当一个原子类型变量执行原子操作时，其他线程不能访问该变量，从而保证线程安全
int io_threads_op; // 标志当前 I/O 线程执行的是 read 或 write 操作
list *io_threads_list[IO_THREADS_MAX_NUM]; // 每个线程的客户端队列，该变量通过 io_threads_pending 变量控制主线程与 I/O 线程交错访问来规避共享数据竞争问题。io_threads_pending 为 0，I/O 线程只访问 io_threads_list 变量，主线程访问并修改 io_threads_list 变量，io_threads_pending 不为 0，主线程只访问 io_threads_list 变量，I/O 线程访问并修改 io_threads_list 变量
```

​		`Redis`启动时，会调用`initThreadedIO`函数创建`I/O`线程`(`由`server.c/InitServerLast`函数触发`)`：

```c++
// networking.c
void initThreadedIO(void) {
    // I/O 线程默认处于停用状态，server.io_threads_active 标志 I/O 线程状态，0：停用，I/O 的读/写操作都在主线程中执行；1：启用
    server.io_threads_active = 0; /* We start with threads not active. */

    /* Don't spawn any thread if the user selected a single thread:
     * we'll handle I/O directly from the main thread. */
    if (server.io_threads_num == 1) return;

    if (server.io_threads_num > IO_THREADS_MAX_NUM) {
        serverLog(LL_WARNING,"Fatal: too many I/O threads configured. "
                             "The maximum number is %d.", IO_THREADS_MAX_NUM);
        exit(1);
    }

    /* Spawn and initialize the I/O threads. */
    for (int i = 0; i < server.io_threads_num; i++) {
        /* Things we do for all the threads including the main thread. */
        io_threads_list[i] = listCreate();
        //  i == 0，代表当前是主线程，直接返回
        if (i == 0) continue; /* Thread 0 is the main thread. */

        /* Things we do only for the additional threads. */
        pthread_t tid;
        // 初始化该线程的互斥量 io_threads_mutex
        pthread_mutex_init(&io_threads_mutex[i],NULL);
        setIOPendingCount(i, 0);
        // 锁住 io_threads_mutex
        pthread_mutex_lock(&io_threads_mutex[i]); /* Thread will be stopped. */
        // 创建线程，线程执行函数为 IOThreadMain，i 为线程 ID
        if (pthread_create(&tid,NULL,IOThreadMain,(void*)(long)i) != 0) {
            serverLog(LL_WARNING,"Fatal: Can't initialize IO thread.");
            exit(1);
        }
        io_threads[i] = tid;
    }
}
```



#### 解析请求

​		`postponeClientRead`函数会将待读取数据的客户端放入`server.clients_pending_read`中，交给`I/O`线程处理。如果想使用多线程执行`I/O`读操作，需要配

置`io-threads-do-reads`。

​		`handleClientsWithPendingReadsUsingThreads`函数`(`由`beforeSleep`触发`)`会将`server.clients_pending_read`的客户端分配给各个`I/O`线程，等待`I/O`线

程读取并解析请求数据。

```c++
int handleClientsWithPendingReadsUsingThreads(void) {
    // 如果 I/O 线程处于停用状态或者没有开启多线程读 I/O 配置，则直接退出
    if (!server.io_threads_active || !server.io_threads_do_reads) return 0;
    int processed = listLength(server.clients_pending_read);
    if (processed == 0) return 0;

    /* Distribute the clients across N different lists. */
    // 将待处理客户端划分到各个线程的客户端队列中
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_read,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }

    /* Give the start condition to the waiting threads, by setting the
     * start condition atomic var. */
    // 设置 io_threads_op 及每个线程的 io_threads_pending。io_threads_pengding 记录 I/O 线程待处理客户端数量， I/O 线程会在检测到 io_threads_pending 不为 0 后开始工作，并在处理完所有客户端后将 io_threads_pending 设置为 0
    io_threads_op = IO_THREADS_OP_READ;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        setIOPendingCount(j, count);
    }

    /* Also use the main thread to process a slice of clients. */
    // 主线程也需要处理分配给它的客户端
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        readQueryFromClient(c->conn);
    }
    listEmpty(io_threads_list[0]);

    /* Wait for all the other threads to end their work. */
    // 不断检测每个线程的 io_threads_pending，直到所有 io_threads_pending 都为 0。
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }

    /* Run the list of clients again to process the new buffers. */
    // 主线程遍历所有客户端，执行命令
    while(listLength(server.clients_pending_read)) {
        ln = listFirst(server.clients_pending_read);
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_READ;
        listDelNode(server.clients_pending_read,ln);

        serverAssert(!(c->flags & CLIENT_BLOCKED));
        if (processPendingCommandsAndResetClient(c) == C_ERR) {
            /* If the client is no longer valid, we avoid
             * processing the client later. So we just go
             * to the next. */
            continue;
        }

        processInputBuffer(c);

        /* We may have pending replies if a thread readQueryFromClient() produced
         * replies and did not install a write handler (it can't).
         */
        if (!(c->flags & CLIENT_PENDING_WRITE) && clientHasPendingReplies(c))
            clientInstallWriteHandler(c);
    }

    /* Update processed count on server */
    server.stat_io_reads_processed += processed;

    return processed;
}
```

​		只有`I/O`操作交给`I/O`线程处理，`Redis`命令还是由主线程单线程执行：

​				`Redis`作为数据库，瓶颈通常不在于数据处理操作，而在于内存和网络`I/O`。

​				单线程降低了数据操作的复杂度。

​				多线程可能存在线程切换甚至加`/`解锁、死锁造成的性能损耗。



#### I/O 线程主逻辑

​		`IOThreadMain`是创建`I/O`线程时指定的线程执行函数，该函数负责处理分配给当前线程的客户端：

```c++
// networking.c
/**
	myid：线程 ID，通过这个 ID 每个线程可以获取io_threads_list、io_threads_pending、io_threads_mutex 中属于自己的数据
*/
void *IOThreadMain(void *myid) {
    /* The ID is the thread number (from 0 to server.iothreads_num-1), and is
     * used by the thread to just manipulate a single sub-array of clients. */
    long id = (unsigned long)myid;
    char thdname[16];

    snprintf(thdname, sizeof(thdname), "io_thd_%ld", id);
    redis_set_thread_title(thdname);
    // 尽可能将 I/O 线程绑定到用户配置的 CPU 列表上，减少不必要的线程切换，提高性能
    redisSetCpuAffinity(server.server_cpulist);
    makeThreadKillable();

    while(1) {
        /* Wait for start */
        // 不断检查 io_threads_pending[id]，如果 io_threads_pending[id] 不为 0，则开始处理任务。使用忙等方式，I/O 线程等待主线程将待处理客户端分配完成后在开始工作
        for (int j = 0; j < 1000000; j++) {
            if (getIOPendingCount(id) != 0) break;
        }

        /* Give the main thread a chance to stop this thread. */
        // 对 io_threads_mutex[id] 互斥量执行一次锁定及释放操作，如果主线程已经锁住了 io_threads_mutex ，则线程会阻塞等待，I/O 线程进入停用状态
        if (getIOPendingCount(id) == 0) {
            pthread_mutex_lock(&io_threads_mutex[id]);
            pthread_mutex_unlock(&io_threads_mutex[id]);
            continue;
        }

        serverAssert(getIOPendingCount(id) != 0);

        /* Process: note that the main thread will never touch our list
         * before we drop the pending count to 0. */
        // 遍历 io_threads_list[id] 的客户端，genju io_threads_op 标志，执行 I/O 读/写操作
        listIter li;
        listNode *ln;
        listRewind(io_threads_list[id],&li);
        while((ln = listNext(&li))) {
            client *c = listNodeValue(ln);
            if (io_threads_op == IO_THREADS_OP_WRITE) {
                writeToClient(c,0);
            } else if (io_threads_op == IO_THREADS_OP_READ) {
                readQueryFromClient(c->conn);
            } else {
                serverPanic("io_threads_op value is unknown");
            }
        }
        // 清空 io_threads_list[id]，并将io_threads_pending[id] 设置为 0，表示当前线程已完成工作
        listEmpty(io_threads_list[id]);
        setIOPendingCount(id, 0);
    }
}
```



#### 返回响应

​		`handleClientsWithPendingWrites`将客户端恢复缓冲区写入`TCP`发送缓冲区，该函数由`handleClientsWithPendingWritesUsingThreads`触发

`(handleClientsWithPendingWritesUsingThreads`由`beforeSleep`触发`)`，`handleClientsWithPendingWritesUsingThreads`会尝试将

`server.clients_pending_wiret`的客户端分配给`I/O`线程处理。

```c++
// networking.c
int handleClientsWithPendingWritesUsingThreads(void) {
    int processed = listLength(server.clients_pending_write);
    if (processed == 0) return 0; /* Return ASAP if there are no clients. */

    /* If I/O threads are disabled or we have few clients to serve, don't
     * use I/O threads, but the boring synchronous code. */
    // stopThreadedIOIfNeeded：根据客户端清空，判断是否需要停用 I/O 线程，如果停用返回 1，此时调用 handleClientsWithPendingWrites 使用单线程处理 serer.clients_pending_write 的客户端
    if (server.io_threads_num == 1 || stopThreadedIOIfNeeded()) {
        return handleClientsWithPendingWrites();
    }

    /* Start threads if needed. */
    // 启动 I/O 线程
    if (!server.io_threads_active) startThreadedIO();

    /* Distribute the clients across N different lists. */
    // 将 server.clients_pending_write 可读分配到各个 I/O 线程中，并等待 I/O 线程处理完成
    listIter li;
    listNode *ln;
    listRewind(server.clients_pending_write,&li);
    int item_id = 0;
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_WRITE;

        /* Remove clients from the list of pending writes since
         * they are going to be closed ASAP. */
        if (c->flags & CLIENT_CLOSE_ASAP) {
            listDelNode(server.clients_pending_write, ln);
            continue;
        }

        int target_id = item_id % server.io_threads_num;
        listAddNodeTail(io_threads_list[target_id],c);
        item_id++;
    }

    /* Give the start condition to the waiting threads, by setting the
     * start condition atomic var. */
    io_threads_op = IO_THREADS_OP_WRITE;
    for (int j = 1; j < server.io_threads_num; j++) {
        int count = listLength(io_threads_list[j]);
        setIOPendingCount(j, count);
    }

    /* Also use the main thread to process a slice of clients. */
    listRewind(io_threads_list[0],&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        writeToClient(c,0);
    }
    listEmpty(io_threads_list[0]);

    /* Wait for all the other threads to end their work. */
    while(1) {
        unsigned long pending = 0;
        for (int j = 1; j < server.io_threads_num; j++)
            pending += getIOPendingCount(j);
        if (pending == 0) break;
    }

    /* Run the list of clients again to install the write handler where
     * needed. */
    listRewind(server.clients_pending_write,&li);
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);

        /* Install the write handler if there are pending writes in some
         * of the clients. */
        if (clientHasPendingReplies(c) &&
                connSetWriteHandler(c->conn, sendReplyToClient) == AE_ERR)
        {
            freeClientAsync(c);
        }
    }
    listEmpty(server.clients_pending_write);

    /* Update processed count on server */
    server.stat_io_writes_processed += processed;

    return processed;
}
```



#### I/O 线程状态切换

​		`stopTHreadedIOIfNeeded`负责根据待处理客户端数量决定是否停用`I/O`线程：

```c++
// networking.c
int stopThreadedIOIfNeeded(void) {
    int pending = listLength(server.clients_pending_write);

    /* Return ASAP if IO threads are disabled (single threaded mode). */
    // io_threads_num 为 1，只使用主线程
    if (server.io_threads_num == 1) return 1;
	// 待处理客户端数量小于 serer.io_threads_num x 2 ，则停用 I/O 线程
    if (pending < (server.io_threads_num*2)) {
        if (server.io_threads_active) stopThreadedIO();
        return 1;
    } else {
        return 0;
    }
}

void stopThreadedIO(void) {
	// 在停用之前，处理 server.clients_pending_read 中等待 I/O 处理的客户端
    handleClientsWithPendingReadsUsingThreads();
    serverAssert(server.io_threads_active == 1);
    
    // 锁住所有的 io_threads_mutex，并将 server.threads_active 设置为 0
    for (int j = 1; j < server.io_threads_num; j++)
        pthread_mutex_lock(&io_threads_mutex[j]);
    server.io_threads_active = 0;
}

void startThreadedIO(void) {
    serverAssert(server.io_threads_active == 0);
    // 使用所有 io_threads_mutex
    for (int j = 1; j < server.io_threads_num; j++)
        pthread_mutex_unlock(&io_threads_mutex[j]);
    server.io_threads_active = 1;
}
```



### RDB

​		`serverCron`函数负责定时生成`RDB`文件：

```c++
// server.c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
	
    ...
    // 如果当前程序中存在子进程，则调用 checkChildrenDone 函数检查是否有子进程已执行完成，如果有，则执行对应的父进程收尾工作
    if (hasActiveChildProcess() || ldbPendingChildren())
    {
        run_with_period(1000) receiveChildInfo();
        checkChildrenDone();
    } else {
		
        // server.dirty 记录上一次生成 RDB 后，Redis 服务器变更了多少个键
        for (j = 0; j < server.saveparamslen; j++) {
            struct saveparam *sp = server.saveparams+j;
			
            /**
            	生成 RDB 文件的条件：
            		满足 server.saveparams 配置的条件
            		上一次 RDB 操作成功或者现在离上一次 RDB 操作已过去时间大于 CONFIG_BGSAVE_RETRY_DELAY（5 秒）
            */
            if (server.dirty >= sp->changes &&
                server.unixtime-server.lastsave > sp->seconds &&
                (server.unixtime-server.lastbgsave_try >
                 CONFIG_BGSAVE_RETRY_DELAY ||
                 server.lastbgsave_status == C_OK))
            {
                serverLog(LL_NOTICE,"%d changes in %d seconds. Saving...",
                    sp->changes, (int)sp->seconds);
                rdbSaveInfo rsi, *rsiptr;
                rsiptr = rdbPopulateSaveInfo(&rsi);
                rdbSaveBackground(server.rdb_filename,rsiptr);
                break;
            }
        }

        ...
            
    }
    
    ...    
}
```

​		`rdbSaveBackground`函数负责在后台生成`RDB`文件`(bgsave`命令也由该函数处理`)`，`bgsaveCommand`函数会创建一个子进程，由子进程负责将`Redis`数据快照

保存到磁盘中，而父进程继续处理客户端请求。

```c++
// rdb.c
int rdbSaveBackground(char *filename, rdbSaveInfo *rsi) {
    pid_t childpid;

    if (hasActiveChildProcess()) return C_ERR;

    server.dirty_before_bgsave = server.dirty;
    server.lastbgsave_try = time(NULL);
	// 创建 RDB 子进程
    /**
    	redisFork 会调用 fork 函数创建子进程，在子进程返回 0 ，父进程返回子进程的 PID
    */
    if ((childpid = redisFork(CHILD_TYPE_RDB)) == 0) {
        int retval;

        /* Child */
        // 尽可能将 RDB 子进程绑定到用户配置的 CPU 列表上 bgsave_cpulist 上，减少不必要的进程切换，最大限度地提高性能
        redisSetProcTitle("redis-rdb-bgsave");
        redisSetCpuAffinity(server.bgsave_cpulist);
        // RDB 子进程，生成 RDB 文件
        retval = rdbSave(filename,rsi);
        if (retval == C_OK) {
            sendChildCowInfo(CHILD_INFO_TYPE_RDB_COW_SIZE, "RDB");
        }
        // 退出 RDB 进程
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        /* Parent */
        // 父进程更新 server 的运行时数据， server.rdb_child_pid 记录 RDB 子进程 ID，不为 -1 代表当前 Redis 中存在 RDB 子进程
        if (childpid == -1) {
            server.lastbgsave_status = C_ERR;
            serverLog(LL_WARNING,"Can't save in background: fork: %s",
                strerror(errno));
            return C_ERR;
        }
        serverLog(LL_NOTICE,"Background saving started by pid %ld",(long) childpid);
        server.rdb_save_time_start = time(NULL);
        server.rdb_child_type = RDB_CHILD_TYPE_DISK;
        return C_OK;
    }
    return C_OK; /* unreached */
}
```



#### 生成 RDB 文件

​		生成`RDB`文件需要执行`I/O`写操作。`Redis`设计了`I/O`的读`/`写层`rio`，将不同存储介质的`I/O`操作统一封装在其中。

```c++
// rio.h
struct _rio {

    size_t (*read)(struct _rio *, void *buf, size_t len); // 对底层介质执行读操作
    size_t (*write)(struct _rio *, const void *buf, size_t len); // 对底层介质执行写操作
    off_t (*tell)(struct _rio *);
    int (*flush)(struct _rio *); // 对底层介质执行刷新操作

    void (*update_cksum)(struct _rio *, const void *buf, size_t len);

    uint64_t cksum, flags;

    size_t processed_bytes;

    size_t max_processing_chunk;

    union {
        struct {
            sds ptr;
            off_t pos;
        } buffer; // 支持内存底层介质
        struct {
            FILE *fp;
            off_t buffered; /* Bytes written since last fsync. */
            off_t autosync; /* fsync after 'autosync' bytes written. */
        } file; // 支持文件底层介质
        struct {
            connection *conn;   /* Connection */
            off_t pos;    /* pos in buf that was returned */
            sds buf;      /* buffered data */
            size_t read_limit;  /* don't allow to buffer/read more than that */
            size_t read_so_far; /* amount of data read from the rio (not buffered) */
        } conn; // 支持连接底层介质
        struct {
            int fd;       /* File descriptor. */
            off_t pos;
            sds buf;
        } fd; // 支持文件描述符底层介质
    } io;
};
```

```c++
// rdb.c
int rdbSave(char *filename, rdbSaveInfo *rsi) {
    char tmpfile[256];
    char cwd[MAXPATHLEN]; /* Current working dir path for error messages. */
    FILE *fp = NULL;
    rio rdb;
    int error = 0;
    
    snprintf(tmpfile,256,"temp-%d.rdb", (int) getpid());
    // 打开一个临时文件用于保存数据，文件名为 temp-<pid>.rdb
    fp = fopen(tmpfile,"w");
    if (!fp) {
        char *cwdp = getcwd(cwd,MAXPATHLEN);
        serverLog(LL_WARNING,
            "Failed opening the RDB file %s (in server root dir %s) "
            "for saving: %s",
            filename,
            cwdp ? cwdp : "unknown",
            strerror(errno));
        return C_ERR;
    }
	// 初始化 rio 变量，返回 rioFileIO，rioFileIO 负责读/写文件
    rioInitWithFile(&rdb,fp);
    startSaving(RDBFLAGS_NONE);
	// 如果配置了 server.rdb_save_incremental_fsync，则将该配置赋值给 rio.io.file.autosync 属性，当系统缓存的数据量大于该属性值时，Redis 执行一次 fsync
    if (server.rdb_save_incremental_fsync)
        rioSetAutoSync(&rdb,REDIS_AUTOSYNC_BYTES);
	// 将 Redis 数据库的内容写到临时文件中
    if (rdbSaveRio(&rdb,&error,RDBFLAGS_NONE,rsi) == C_ERR) {
        errno = error;
        goto werr;
    }

    /* Make sure data will not remain on the OS's output buffers */
    // 将系统缓存刷新到文件中
    if (fflush(fp)) goto werr;
    if (fsync(fileno(fp))) goto werr;
    if (fclose(fp)) { fp = NULL; goto werr; }
    fp = NULL;
    
    /* Use RENAME to make sure the DB file is changed atomically only
     * if the generate DB file is ok. */
    // 重命名临时文件，替换就的 RDB 文件，RDB 文件由server.rdb_filename 指定
    if (rename(tmpfile,filename) == -1) {
        char *cwdp = getcwd(cwd,MAXPATHLEN);
        serverLog(LL_WARNING,
            "Error moving temp DB file %s on the final "
            "destination %s (in server root dir %s): %s",
            tmpfile,
            filename,
            cwdp ? cwdp : "unknown",
            strerror(errno));
        unlink(tmpfile);
        stopSaving(0);
        return C_ERR;
    }

    serverLog(LL_NOTICE,"DB saved on disk");
    // 更新 server 中的 RDB 的相关属性
    server.dirty = 0;
    server.lastsave = time(NULL);
    server.lastbgsave_status = C_OK;
    stopSaving(1);
    return C_OK;

werr:
    serverLog(LL_WARNING,"Write error saving DB on disk: %s", strerror(errno));
    if (fp) fclose(fp);
    unlink(tmpfile);
    stopSaving(0);
    return C_ERR;
}
```

​		`save`命令会在`Redis`主进程中调用`rdbSave`生成`RDB`文件，可能导致主进程长期阻塞。



#### 写入 RDB 数据

​		`Redis`会在`RDB`文件的每一部分内容之前添加一个类型字节，标志其内容类型：

|           标志           |             内容             |
| :----------------------: | :--------------------------: |
|     RDB_OPCODE_IDLE      |  键空闲时间，用于 LRU 算法   |
|     RDB_OPCODE_FREQ      |  键 LFU 计数，用于 LFU 算法  |
|      RDB_OPCODE_AUX      |         RDB 辅助字段         |
|  RDB_OPCODE_MODULE_AUX   | Module 自定义类型的辅助字段  |
|   RDB_OPCODE_RESIZEDB    | 数据库字典大小和过期字典大小 |
| RDB_OPCODE_EXPIRETIME_MS |   键过期时间戳，单位为毫秒   |
|  RDB_OPCODE_EXPIRETIME   |    键过期时间戳，单位为秒    |
|   RDB_OPCODE_SELECTDB    |        数据库索引标志        |
|      RDB_OPCODE_EOF      |           结束标志           |

```c++
// rdb.c
int rdbSaveRio(rio *rdb, int *error, int rdbflags, rdbSaveInfo *rsi) {
    dictIterator *di = NULL;
    dictEntry *de;
    char magic[10];
    uint64_t cksum;
    size_t processed = 0;
    int j;
    long key_count = 0;
    long long info_updated_time = 0;
    char *pname = (rdbflags & RDBFLAGS_AOF_PREAMBLE) ? "AOF rewrite" :  "RDB";

    if (server.rdb_checksum)
        rdb->update_cksum = rioGenericUpdateChecksum;
    // 写入一个 RDB 标志，内容为 REDIS<RDB_VERSION>，标志该文件时 RDB 文件
    snprintf(magic,sizeof(magic),"REDIS%04d",RDB_VERSION);
    if (rdbWriteRaw(rdb,magic,9) == -1) goto werr;
    /**
    	依次写入如下辅助字段：
    		redis-ver：Redis 版本号
    		redis-bits：64/32 位 Redis
    		ctime：RDB 创建时间
    		used-mem：内容使用量
    	如果 rsi 不为空，则还会写入：
    		repl-stream-db：存储server.slaveseldb 属性
    		repl-id：存储 server.replid 属性
    		repl-offset：存储 server.master_repl_offset 属性
    		这三个字段用于实现主从复制机制
    */
    if (rdbSaveInfoAuxFields(rdb,rdbflags,rsi) == -1) goto werr;
    // 触发 Module 自定义类中指定的 aux_save 回调函数，将数据库字典挚爱的数据保存到 RDB 文件中
    if (rdbSaveModulesAux(rdb, REDISMODULE_AUX_BEFORE_RDB) == -1) goto werr;
	// 遍历所有的数据库 redisDb
    for (j = 0; j < server.dbnum; j++) {
        redisDb *db = server.db+j;
        dict *d = db->dict;
        if (dictSize(d) == 0) continue;
        di = dictGetSafeIterator(d);

        /* Write the SELECT DB opcode */
        // 写入 RDB_OPCODE_SELECTDB 标志和数据库 ID
        if (rdbSaveType(rdb,RDB_OPCODE_SELECTDB) == -1) goto werr;
        if (rdbSaveLen(rdb,j) == -1) goto werr;

        /* Write the RESIZE DB opcode. */
        // 写入 RDB_OPCODE_RESIZEDB 标志和数据库字典大小，过期字典大小
        uint64_t db_size, expires_size;
        db_size = dictSize(db->dict);
        expires_size = dictSize(db->expires);
        if (rdbSaveType(rdb,RDB_OPCODE_RESIZEDB) == -1) goto werr;
        if (rdbSaveLen(rdb,db_size) == -1) goto werr;
        if (rdbSaveLen(rdb,expires_size) == -1) goto werr;

        /* Iterate this DB writing every entry */
        // 遍历数据库中的键值对，rdbSaveKeyValuePair 将每一个键值对的键、值、过期时间写入 RDB 文件
        while((de = dictNext(di)) != NULL) {
            sds keystr = dictGetKey(de);
            robj key, *o = dictGetVal(de);
            long long expire;

            initStaticStringObject(key,keystr);
            expire = getExpire(db,&key);
            if (rdbSaveKeyValuePair(rdb,&key,o,expire) == -1) goto werr;

            /* When this RDB is produced as part of an AOF rewrite, move
             * accumulated diff from parent to child while rewriting in
             * order to have a smaller final write. */
            if (rdbflags & RDBFLAGS_AOF_PREAMBLE &&
                rdb->processed_bytes > processed+AOF_READ_DIFF_INTERVAL_BYTES)
            {
                processed = rdb->processed_bytes;
                aofReadDiffFromParent();
            }

            /* Update child info every 1 second (approximately).
             * in order to avoid calling mstime() on each iteration, we will
             * check the diff every 1024 keys */
            if ((key_count++ & 1023) == 0) {
                long long now = mstime();
                if (now - info_updated_time >= 1000) {
                    sendChildInfo(CHILD_INFO_TYPE_CURRENT_INFO, key_count, pname);
                    info_updated_time = now;
                }
            }
        }
        dictReleaseIterator(di);
        di = NULL; /* So that we don't release it again on error. */
    }
    // 数据库的数据都已经写入 RDB 文件，将 Redis 中的 Lua 脚本内容写入 RDB 文件
    if (rsi && dictSize(server.lua_scripts)) {
        di = dictGetIterator(server.lua_scripts);
        while((de = dictNext(di)) != NULL) {
            robj *body = dictGetVal(de);
            if (rdbSaveAuxField(rdb,"lua",3,body->ptr,sdslen(body->ptr)) == -1)
                goto werr;
        }
        dictReleaseIterator(di);
        di = NULL; /* So that we don't release it again on error. */
    }

    if (rdbSaveModulesAux(rdb, REDISMODULE_AUX_AFTER_RDB) == -1) goto werr;

    /* EOF opcode */
    // 写入结束标志 RDB_OPCODE_EOF
    if (rdbSaveType(rdb,RDB_OPCODE_EOF) == -1) goto werr;

    /* CRC64 checksum. It will be zero if checksum computation is disabled, the
     * loading code skips the check in this case. */
    // 写入 CRC64 校验码
    cksum = rdb->cksum;
    memrev64ifbe(&cksum);
    if (rioWrite(rdb,&cksum,8) == 0) goto werr;
    return C_OK;

werr:
    if (error) *error = errno;
    if (di) dictReleaseIterator(di);
    return C_ERR;
}

int rdbSaveKeyValuePair(rio *rdb, robj *key, robj *val, long long expiretime) {
    int savelru = server.maxmemory_policy & MAXMEMORY_FLAG_LRU;
    int savelfu = server.maxmemory_policy & MAXMEMORY_FLAG_LFU;

    /* Save the expire time */
    // 如果设置了过期时间，写入 RDB_OPCODE_EXPIRETIME_MS 标志和过期时间戳
    if (expiretime != -1) {
        if (rdbSaveType(rdb,RDB_OPCODE_EXPIRETIME_MS) == -1) return -1;
        if (rdbSaveMillisecondTime(rdb,expiretime) == -1) return -1;
    }

    /* Save the LRU info. */
    // 如果淘汰策略使用的是 LRU 算法或者 LFU 算法，则记录键的空闲时间或 LFU 计数
    if (savelru) {
        uint64_t idletime = estimateObjectIdleTime(val);
        idletime /= 1000; /* Using seconds is enough and requires less space.*/
        if (rdbSaveType(rdb,RDB_OPCODE_IDLE) == -1) return -1;
        if (rdbSaveLen(rdb,idletime) == -1) return -1;
    }

    /* Save the LFU info. */
    if (savelfu) {
        uint8_t buf[1];
        buf[0] = LFUDecrAndReturn(val);
        /* We can encode this in exactly two bytes: the opcode and an 8
         * bit counter, since the frequency is logarithmic with a 0-255 range.
         * Note that we do not store the halving time because to reset it
         * a single time when loading does not affect the frequency much. */
        if (rdbSaveType(rdb,RDB_OPCODE_FREQ) == -1) return -1;
        if (rdbWriteRaw(rdb,buf,1) == -1) return -1;
    }

    /* Save type, key, value */
    // 写入 RDB 键值对标志，在写入内容（以字符数组格式保存），最后写入值内容
    if (rdbSaveObjectType(rdb,val) == -1) return -1;
    if (rdbSaveStringObject(rdb,key) == -1) return -1;
    if (rdbSaveObject(rdb,val,key) == -1) return -1;

    /* Delay return if required (for testing) */
    if (server.rdb_key_save_delay)
        debugDelay(server.rdb_key_save_delay);

    return 1;
}
```

​		`server.rdb_child_pid`记录`RDB`进程`ID`，但`RDB`子进程和父主进程都有独立的进程空间`(`即父主进程和`RDB`子进程都存在`server.rdb_chile_pid`变量`)`，

所以并不能在`RDB`子进程中修改`server.rdb_chile_pid`变量，必须在`RDB`子进程结束后由父主进程修改父主进程的`server.rdb_chile_pid`变量。

```c++
// server.c
// 该函数由 serverCron 触发，会检查子进程是否执行完成，子进程执行完成后，主进程完成收尾工作
void checkChildrenDone(void) {
    int statloc = 0;
    pid_t pid;
	// 检测是否存在已结束的子进程
    if ((pid = waitpid(-1, &statloc, WNOHANG)) != 0) {
        int exitcode = WIFEXITED(statloc) ? WEXITSTATUS(statloc) : -1;
        int bysignal = 0;
		// 获取子进程的结束代码和中断信号
        if (WIFSIGNALED(statloc)) bysignal = WTERMSIG(statloc);

        if (exitcode == SERVER_CHILD_NOERROR_RETVAL) {
            bysignal = SIGUSR1;
            exitcode = 1;
        }

        if (pid == -1) {
            serverLog(LL_WARNING,"waitpid() returned an error: %s. "
                "child_type: %s, child_pid = %d",
                strerror(errno),
                strChildType(server.child_type),
                (int) server.child_pid);
        } else if (pid == server.child_pid) {
            if (server.child_type == CHILD_TYPE_RDB) {
                // 如果是 RDB 子进程，调用 backgroundSaveDoneHandler 函数，如果 RDB 子进程处理成功，backgroundSaveDoneHandler 会更新父进程的 server.dirty、server.lastbgsave_status、server.lastsave 等属性。如果 RDB 数据保存到磁盘中，则需要量 RDB 文件发送给正在进行全量同步的从服务器
                backgroundSaveDoneHandler(exitcode, bysignal);
            } else if (server.child_type == CHILD_TYPE_AOF) {
                backgroundRewriteDoneHandler(exitcode, bysignal);
            } else if (server.child_type == CHILD_TYPE_MODULE) {
                ModuleForkDoneHandler(exitcode, bysignal);
            } else {
                serverPanic("Unknown child type %d for child pid %d", server.child_type, server.child_pid);
                exit(1);
            }
            if (!bysignal && exitcode == 0) receiveChildInfo();
            resetChildState();
        } else {
            if (!ldbRemoveChild(pid)) {
                serverLog(LL_WARNING,
                          "Warning, detected child with unmatched pid: %ld",
                          (long) pid);
            }
        }

        /* start any pending forks immediately. */
        replicationStartPendingFork();
    }
}
```



#### RDB 文件加载过程

​		在启动时`Redis`会调用`loadDataFromDisk`尝试加载`RDB`数据。`loadDataFromDisk`依次调用：`loadDataFromDisk->rdbLoad->rdbLoadRio`：

```c++
// rdb.c
int rdbLoadRio(rio *rdb, int rdbflags, rdbSaveInfo *rsi) {
    uint64_t dbid;
    int type, rdbver;
    redisDb *db = server.db+0;
    char buf[1024];
    int error;
    long long empty_keys_skipped = 0, expired_keys_skipped = 0, keys_loaded = 0;

    rdb->update_cksum = rdbLoadProgressCallback;
    rdb->max_processing_chunk = server.loading_process_events_interval_bytes;
    // 读取 RDB 文件的 RDB 标志
    if (rioRead(rdb,buf,9) == 0) goto eoferr;
    buf[9] = '\0';
    if (memcmp(buf,"REDIS",5) != 0) {
        serverLog(LL_WARNING,"Wrong signature trying to load DB from file");
        errno = EINVAL;
        return C_ERR;
    }
    rdbver = atoi(buf+5);
    if (rdbver < 1 || rdbver > RDB_VERSION) {
        serverLog(LL_WARNING,"Can't handle RDB format version %d",rdbver);
        errno = EINVAL;
        return C_ERR;
    }

    /* Key-specific attributes, set by opcodes before the key type. */
    long long lru_idle = -1, lfu_freq = -1, expiretime = -1, now = mstime();
    long long lru_clock = LRU_CLOCK();

    while(1) {
        sds key;
        robj *val;

        /* Read type. */
        // 分析 RDB 文件，首先读取标志字节，在根据标志字节进行对应的处理
        if ((type = rdbLoadType(rdb)) == -1) goto eoferr;

        ...

        /* Read key */
        // 此时读取的是键值对类型，rdbGenericLoadStringObject 读取键内容，rdbLoadObject 读取值内容，并转化为 redisObject
        if ((key = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL)
            goto eoferr;
        /* Read value */
        val = rdbLoadObject(type,rdb,key,&error);


        if (val == NULL) {

            if (error == RDB_LOAD_ERR_EMPTY_KEY) {
                if(empty_keys_skipped++ < 10)
                    serverLog(LL_WARNING, "rdbLoadObject skipping empty key: %s", key);
                sdsfree(key);
            } else {
                sdsfree(key);
                goto eoferr;
            }
        } 
        // 如果键已过期，且当前服务器是主节点，则删除该键
        else if (iAmMaster() &&
            !(rdbflags&RDBFLAGS_AOF_PREAMBLE) &&
            expiretime != -1 && expiretime < now)
        {
            sdsfree(key);
            decrRefCount(val);
            expired_keys_skipped++;
        } else {
            robj keyobj;
            initStaticStringObject(keyobj,key);

            /* Add the new object in the hash table */
            // 将读取的键值对添加到数据库字典中
            int added = dbAddRDBLoad(db,key,val);
            keys_loaded++;
            if (!added) {
                if (rdbflags & RDBFLAGS_ALLOW_DUP) {
                    /* This flag is useful for DEBUG RELOAD special modes.
                     * When it's set we allow new keys to replace the current
                     * keys with the same name. */
                    dbSyncDelete(db,&keyobj);
                    dbAddRDBLoad(db,key,val);
                } else {
                    serverLog(LL_WARNING,
                        "RDB has duplicated key '%s' in DB %d",key,db->id);
                    serverPanic("Duplicated key found in RDB file");
                }
            }

            /* Set the expire time if needed */
            if (expiretime != -1) {
                setExpire(NULL,db,&keyobj,expiretime);
            }

            /* Set usage information (for eviction). */
            objectSetLRUOrLFU(val,lfu_freq,lru_idle,lru_clock,1000);

            /* call key space notification on key loaded for modules only */
            moduleNotifyKeyspaceEvent(NOTIFY_LOADED, "loaded", &keyobj, db->id);
        }

        /* Loading the database more slowly is useful in order to test
         * certain edge cases. */
        if (server.key_load_delay)
            debugDelay(server.key_load_delay);

        /* Reset the state that is key-specified and is populated by
         * opcodes before the key, so that we start from scratch again. */
        expiretime = -1;
        lfu_freq = -1;
        lru_idle = -1;
    }
    /* Verify the checksum if RDB version is >= 5 */
    // 生成 RDB 文件的 Redis 版本大于等于 5 时需要检查 CRC64 校验码
    if (rdbver >= 5) {
        uint64_t cksum, expected = rdb->cksum;

        if (rioRead(rdb,&cksum,8) == 0) goto eoferr;
        if (server.rdb_checksum && !server.skip_checksum_validation) {
            memrev64ifbe(&cksum);
            if (cksum == 0) {
                serverLog(LL_WARNING,"RDB file was saved with checksum disabled: no check performed.");
            } else if (cksum != expected) {
                serverLog(LL_WARNING,"Wrong RDB checksum expected: (%llx) but "
                    "got (%llx). Aborting now.",
                        (unsigned long long)expected,
                        (unsigned long long)cksum);
                rdbReportCorruptRDB("RDB CRC error");
                return C_ERR;
            }
        }
    }

    if (empty_keys_skipped) {
        serverLog(LL_WARNING,
            "Done loading RDB, keys loaded: %lld, keys expired: %lld, empty keys skipped: %lld.",
                keys_loaded, expired_keys_skipped, empty_keys_skipped);
    } else {
        serverLog(LL_WARNING,
            "Done loading RDB, keys loaded: %lld, keys expired: %lld.",
                keys_loaded, expired_keys_skipped);
    }
    return C_OK;
eoferr:
    serverLog(LL_WARNING,
        "Short read or OOM loading DB. Unrecoverable error, aborting now.");
    rdbReportReadError("Unexpected EOF reading RDB file");
    return C_ERR;
}
```



### AOF

​		`Redis`运行期间，不断将`Redis`执行的写命令`(`修改数据的命令`)`写到文件中，`Redis`重启后，只要将这些写命令重复执行一遍就可以恢复数据。`AOF`功能

包括两部分：

​				`AOF`持久化：将执行的写命令写入`AOF`缓冲区，并定时刷新缓冲区到文件。

​				`AOF`重写：将当前数据库内容保存一份到文件中，并替换原来的`AOF`文件。

​		`AOF`文件是文本格式，以`RESP`协议格式存储每一个命令的命令名，以及执行命令的所有参数。

​		`serverCron`时间事件负责定时触发刷新`AOF`缓冲区和重写`AOF`文件的操作：

```c++
// server.c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
    
    ...
    // server.aof_rewrite_scheduled 不为 0 ，代表存在延迟的 AOF 重写操作，如果当场当前没有子进程，则执行 AOF 重写操作
    if (!hasActiveChildProcess() &&
        server.aof_rewrite_scheduled)
    {
        rewriteAppendOnlyFileBackground();
    }

    /* Check if a background saving or AOF rewrite in progress terminated. */
    if (hasActiveChildProcess() || ldbPendingChildren())
    {
        run_with_period(1000) receiveChildInfo();
        checkChildrenDone();
    } else {
        
        ...
        /** 服务器开启了 AOF 功能，执行 AOF 重写操作，执行的条件：
        		当前 AOF 文件大小大于 server.aof_rewrite_min_size 配置
        		对于上次 AOF 重写后的 AOF 文件大小，当前 AOF 文件增加的空间大小所占比例已经超过 server.aof_rewrite_perc 配置
        */
        if (server.aof_state == AOF_ON &&
            !hasActiveChildProcess() &&
            server.aof_rewrite_perc &&
            server.aof_current_size > server.aof_rewrite_min_size)
        {
            long long base = server.aof_rewrite_base_size ?
                server.aof_rewrite_base_size : 1;
            long long growth = (server.aof_current_size*100/base) - 100;
            if (growth >= server.aof_rewrite_perc) {
                serverLog(LL_NOTICE,"Starting automatic rewriting of AOF on %lld%% growth",growth);
                rewriteAppendOnlyFileBackground();
            }
        }
    }
    /* Just for the sake of defensive programming, to avoid forgeting to
     * call this function when need. */
    updateDictResizePolicy();


    /* AOF postponed flush: Try at every cron cycle if the slow fsync
     * completed. */
    // server.aof_flush_postponed_start 不为 0 ，代表存在延迟的 AOF 缓冲区刷新操作，此时 刷新 AOF 缓冲区内容到文件
    if (server.aof_state == AOF_ON && server.aof_flush_postponed_start)
        // 正常情况下，次函数在 beforeSleep 函数中触发
        flushAppendOnlyFile(0);

    /* AOF write errors: in this case we have a buffer to flush as well and
     * clear the AOF error in case of success to make the DB writable again,
     * however to try every second is enough in case of 'hz' is set to
     * a higher frequency. */
    // 每经过 1000 毫秒执行：如果上次 AOF 缓冲区刷新操作中写入磁盘出错，则再次刷新 AOF 缓冲区
    run_with_period(1000) {
        if (server.aof_state == AOF_ON && server.aof_last_write_status == C_ERR)
            flushAppendOnlyFile(0);
    }

    ...
}
```

​		`AOF`持久化过程：

​				1、命令传播。

​				2、刷新`AOF`缓冲区。

​				3、同步磁盘。

​		`server.aof_fsync`由`appendfsync`配置项配置，用于指定`AOF`缓冲区同步磁盘的策略：

​				`no`：使用操作系统的同步机制，`Redis`不执行`fsync`，由操作系统保证数据同步到磁盘。速度最快，安全性最低。

​				`always`：每次写入都执行`fsync`，以保证数据同步到磁盘，效率最低。

​				`AOF_FSYNC_EVERYSET`：每秒执行一次`fsync`，性能和安全的折中处理，极端情况下可能丢失最近`1`秒的数据。



#### 命令传播

​		开启`AOF`功能后，`Redis`将执行的每个写命令都传播到`AOF`缓冲区`server.aof_buf`。`feedAppendOnlyFile`函数负责将命令传播到`AOF`缓存区`(`由`propagate`

函数触发`)`：

```c++
// aof.c
void feedAppendOnlyFile(struct redisCommand *cmd, int dictid, robj **argv, int argc) {
    sds buf = sdsempty();
    /* The DB this command was targeting is not the same as the last command
     * we appended. To issue a SELECT command is needed. */
    if (dictid != server.aof_selected_db) {
        char seldb[64];

        snprintf(seldb,sizeof(seldb),"%d",dictid);
        buf = sdscatprintf(buf,"*2\r\n$6\r\nSELECT\r\n$%lu\r\n%s\r\n",
            (unsigned long)strlen(seldb),seldb);
        server.aof_selected_db = dictid;
    }
	// 对 expire、expireat、pexpire、setex、psetex 或者带 ex、px 选项的 set 命令的特殊处理，这些命令中对键设置了过期时间，需要将这些命令转换为 pexpireat 命令，将过期时间的时间戳写入 buf 暂存区
    if (cmd->proc == expireCommand || cmd->proc == pexpireCommand ||
        cmd->proc == expireatCommand) {
        /* Translate EXPIRE/PEXPIRE/EXPIREAT into PEXPIREAT */
        buf = catAppendOnlyExpireAtCommand(buf,cmd,argv[1],argv[2]);
    } else if (cmd->proc == setCommand && argc > 3) {
        robj *pxarg = NULL;
        /* When SET is used with EX/PX argument setGenericCommand propagates them with PX millisecond argument.
         * So since the command arguments are re-written there, we can rely here on the index of PX being 3. */
        if (!strcasecmp(argv[3]->ptr, "px")) {
            pxarg = argv[4];
        }
        /* For AOF we convert SET key value relative time in milliseconds to SET key value absolute time in
         * millisecond. Whenever the condition is true it implies that original SET has been transformed
         * to SET PX with millisecond time argument so we do not need to worry about unit here.*/
        if (pxarg) {
            robj *millisecond = getDecodedObject(pxarg);
            long long when = strtoll(millisecond->ptr,NULL,10);
            when += mstime();

            decrRefCount(millisecond);

            robj *newargs[5];
            newargs[0] = argv[0];
            newargs[1] = argv[1];
            newargs[2] = argv[2];
            newargs[3] = shared.pxat;
            newargs[4] = createStringObjectFromLongLong(when);
            buf = catAppendOnlyGenericCommand(buf,5,newargs);
            decrRefCount(newargs[4]);
        } else {
            buf = catAppendOnlyGenericCommand(buf,argc,argv);
        }
    } else {
        /* All the other commands don't need translation or need the
         * same translation already operated in the command vector
         * for the replication itself. */
        // 将命令写入 buf 暂存区
        buf = catAppendOnlyGenericCommand(buf,argc,argv);
    }

    /* Append to the AOF buffer. This will be flushed on disk just before
     * of re-entering the event loop, so before the client will get a
     * positive reply about the operation performed. */
    // 服务器开启 AOF 功能，则将 buf 暂存区内容写入 AOF 缓冲区
    if (server.aof_state == AOF_ON)
        server.aof_buf = sdscatlen(server.aof_buf,buf,sdslen(buf));

    /* If a background append only file rewriting is in progress we want to
     * accumulate the differences between the child DB and the current one
     * in a buffer, so that when the child process will do its work we
     * can append the differences to the new append only file. */
    // 存在 AOF 子进程执行 AOF 重写操作，则将 buf 暂存区内容写入 AOF 重写缓存区 server.aof_rewrite_buf_blocks
    if (server.child_type == CHILD_TYPE_AOF)
        aofRewriteBufferAppend((unsigned char*)buf,sdslen(buf));

    sdsfree(buf);
}
```



#### 刷新 AOF 缓冲区及同步磁盘

```c++
void flushAppendOnlyFile(int force) {
    ssize_t nwritten;
    int sync_in_progress = 0;
    mstime_t latency;
	// AOF 缓冲区为空
    if (sdslen(server.aof_buf) == 0) {
		// 同步策略为每秒同步，并且当前存在待同步磁盘的数据，距上次磁盘同步也过去了 1 秒，则执行磁盘同步
        if (server.aof_fsync == AOF_FSYNC_EVERYSEC &&
            server.aof_fsync_offset != server.aof_current_size &&
            server.unixtime > server.aof_last_fsync &&
            !(sync_in_progress = aofFsyncInProgress())) {
            goto try_fsync;
        } else {
            return;
        }
    }
    
    if (server.aof_fsync == AOF_FSYNC_EVERYSEC)
    	// 当前是否存在后台线程正在同步磁盘    
        sync_in_progress = aofFsyncInProgress();

    if (server.aof_fsync == AOF_FSYNC_EVERYSEC && !force) {
		// 同步策略为每秒同步，则延迟该 AOF 缓冲区刷新操作，然后退出。如果已延迟多次并且延迟时间超过 2 秒，则强制刷新 AOF 缓冲区
        if (sync_in_progress) {
            if (server.aof_flush_postponed_start == 0) {
                /* No previous write postponing, remember that we are
                 * postponing the flush and return. */
                server.aof_flush_postponed_start = server.unixtime;
                return;
            } else if (server.unixtime - server.aof_flush_postponed_start < 2) {
                /* We were already waiting for fsync to finish, but for less
                 * than two seconds this is still ok. Postpone again. */
                return;
            }
            /* Otherwise fall trough, and go write since we can't wait
             * over two seconds. */
            server.aof_delayed_fsync++;
            serverLog(LL_NOTICE,"Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.");
        }
    }
    /* We want to perform a single write. This should be guaranteed atomic
     * at least if the filesystem we are writing is a real physical one.
     * While this will save us against the server being killed I don't think
     * there is much to do about the whole server stopping for power problems
     * or alike */

    if (server.aof_flush_sleep && sdslen(server.aof_buf)) {
        usleep(server.aof_flush_sleep);
    }

    latencyStartMonitor(latency);
	// 将 AOF 缓冲区内容写入文件
    nwritten = aofWrite(server.aof_fd,server.aof_buf,sdslen(server.aof_buf));
    latencyEndMonitor(latency);
    /* We want to capture different events for delayed writes:
     * when the delay happens with a pending fsync, or with a saving child
     * active, and when the above two conditions are missing.
     * We also use an additional event name to save all samples which is
     * useful for graphing / monitoring purposes. */
    if (sync_in_progress) {
        latencyAddSampleIfNeeded("aof-write-pending-fsync",latency);
    } else if (hasActiveChildProcess()) {
        latencyAddSampleIfNeeded("aof-write-active-child",latency);
    } else {
        latencyAddSampleIfNeeded("aof-write-alone",latency);
    }
    latencyAddSampleIfNeeded("aof-write",latency);

    /* We performed the write so reset the postponed flush sentinel to zero. */
    server.aof_flush_postponed_start = 0;

    if (nwritten != (ssize_t)sdslen(server.aof_buf)) {
        static time_t last_write_error_log = 0;
        int can_log = 0;

        /* Limit logging rate to 1 line per AOF_WRITE_LOG_ERROR_RATE seconds. */
        if ((server.unixtime - last_write_error_log) > AOF_WRITE_LOG_ERROR_RATE) {
            can_log = 1;
            last_write_error_log = server.unixtime;
        }

        /* Log the AOF write error and record the error code. */
        if (nwritten == -1) {
            if (can_log) {
                serverLog(LL_WARNING,"Error writing to the AOF file: %s",
                    strerror(errno));
                server.aof_last_write_errno = errno;
            }
        } else {
            if (can_log) {
                serverLog(LL_WARNING,"Short write while writing to "
                                       "the AOF file: (nwritten=%lld, "
                                       "expected=%lld)",
                                       (long long)nwritten,
                                       (long long)sdslen(server.aof_buf));
            }

            if (ftruncate(server.aof_fd, server.aof_current_size) == -1) {
                if (can_log) {
                    serverLog(LL_WARNING, "Could not remove short write "
                             "from the append-only file.  Redis may refuse "
                             "to load the AOF the next time it starts.  "
                             "ftruncate: %s", strerror(errno));
                }
            } else {
                /* If the ftruncate() succeeded we can set nwritten to
                 * -1 since there is no longer partial data into the AOF. */
                nwritten = -1;
            }
            server.aof_last_write_errno = ENOSPC;
        }

        /* Handle the AOF write error. */
        if (server.aof_fsync == AOF_FSYNC_ALWAYS) {
            /* We can't recover when the fsync policy is ALWAYS since the reply
             * for the client is already in the output buffers (both writes and
             * reads), and the changes to the db can't be rolled back. Since we
             * have a contract with the user that on acknowledged or observed
             * writes are is synced on disk, we must exit. */
            serverLog(LL_WARNING,"Can't recover from AOF write error when the AOF fsync policy is 'always'. Exiting...");
            exit(1);
        } else {
            /* Recover from failed write leaving data into the buffer. However
             * set an error to stop accepting writes as long as the error
             * condition is not cleared. */
            server.aof_last_write_status = C_ERR;

            /* Trim the sds buffer if there was a partial write, and there
             * was no way to undo it with ftruncate(2). */
            if (nwritten > 0) {
                server.aof_current_size += nwritten;
                sdsrange(server.aof_buf,nwritten,-1);
            }
            return; /* We'll try again on the next call... */
        }
    } else {
        /* Successful write(2). If AOF was in error state, restore the
         * OK state and log the event. */
        if (server.aof_last_write_status == C_ERR) {
            serverLog(LL_WARNING,
                "AOF write error looks solved, Redis can write again.");
            server.aof_last_write_status = C_OK;
        }
    }
    server.aof_current_size += nwritten;

    /* Re-use AOF buffer when it is small enough. The maximum comes from the
     * arena size of 4k minus some overhead (but is otherwise arbitrary). */
    // 此时缓冲区内容已刷新成功，如果 AOF 缓冲区总空间小于 4KB ，则情况内容并重用 AOF 缓冲区，否则创建一个新的 AOF 缓冲区
    if ((sdslen(server.aof_buf)+sdsavail(server.aof_buf)) < 4000) {
        sdsclear(server.aof_buf);
    } else {
        sdsfree(server.aof_buf);
        server.aof_buf = sdsempty();
    }

// 同步磁盘 
try_fsync:
    /* Don't fsync if no-appendfsync-on-rewrite is set to yes and there are
     * children doing I/O in the background. */
    // 如果程序存在子进程，并且开始了 server.aof_no_fsync_on_rewrite 配置，则不同步磁盘
    if (server.aof_no_fsync_on_rewrite && hasActiveChildProcess())
        return;

    /* Perform the fsync if needed. */
    if (server.aof_fsync == AOF_FSYNC_ALWAYS) {
        /* redis_fsync is defined as fdatasync() for Linux in order to avoid
         * flushing metadata. */
        latencyStartMonitor(latency);
        /* Let's try to get this data on the disk. To guarantee data safe when
         * the AOF fsync policy is 'always', we should exit if failed to fsync
         * AOF (see comment next to the exit(1) after write error above). */
        // 同步策略为每次同步，则同步磁盘
        if (redis_fsync(server.aof_fd) == -1) {
            serverLog(LL_WARNING,"Can't persist AOF for fsync error when the "
              "AOF fsync policy is 'always': %s. Exiting...", strerror(errno));
            exit(1);
        }
        latencyEndMonitor(latency);
        latencyAddSampleIfNeeded("aof-fsync-always",latency);
        server.aof_fsync_offset = server.aof_current_size;
        server.aof_last_fsync = server.unixtime;
    } else if ((server.aof_fsync == AOF_FSYNC_EVERYSEC &&
                server.unixtime > server.aof_last_fsync)) {
        // 同步策略为每秒同步，并且距离上次磁盘同步已经过去 1 秒，则添加一个后台任务负责同步磁盘
        if (!sync_in_progress) {
            aof_background_fsync(server.aof_fd);
            server.aof_fsync_offset = server.aof_current_size;
        }
        server.aof_last_fsync = server.unixtime;
    }
}
```



#### AOF 重写

​		`Redis 4`提供了`AOF`混合持久化，开启`AOF`混合持久化后，`AOF`重写时，会将`Redis`数据以`RDB`格式保存到新文件中，再将重写缓存区的增量命令以`AOF`格

式写入文件。关闭`AOF`持久化后，`AOF`重写时，则将`Redis`所有数据转换为写入命令写入新文件。

​		`AOF`重写过程：

​				1、创建一个字进程，称为`AOF`子进程，`AOF`子进程负责将当前内存数据保存到一个新文件中。

​				2、`AOF`子进程将步骤`1`执行期间父主进程执行的增量命令写到新文件中，最后结束`AOF`子进程。

​				3、父主进程进行收尾工作，将步骤`2`执行期间主进程执行的增量命令也写到新文件中，替换`AOF`文件，`AOF`重写完成。

​		`rewriteAppendOnlyFileBackground`负责在后台完成`AOF`重写操作，`bgrewriteaof`命令也是调用此函数实现。该函数会创建`AOF`进程，并在`AOF`进程中调用

`rewriteAppendOnlyFile`重写`AOF`文件。该函数也调用了`aofCreatePips`函数打开一条数据管道和两条控制管道，数据管道用于父主进程向`AOF`子进程传输增量命

令，控制管道用于父子进程交互，控制何时停止数据传输。

|            管道文件描述符             |                       作用                       |
| :-----------------------------------: | :----------------------------------------------: |
|  server.aof_pipe_write_data_to_child  |               父进程向子进程写数据               |
| server.aof_pipe_read_data_from_parent |               子进程从父进程读数据               |
|  server.aof_pipe_write_ack_to_parent  | 子进程向父进程发送停止标志，告诉父进程停止写数据 |
|  server.aof_pipe_read_ack_from_child  |            父进程从子进程读取停止标志            |
|  server.aof_pipe_write_ack_to_child   |    父进程收到停止标志后，向子进程回复确认标志    |
| server.aof_pipe_read_ack_from_parent  |          子进程从父进程读取回复确认标志          |

```c++
// aof.c
int rewriteAppendOnlyFile(char *filename) {
    rio aof;
    FILE *fp = NULL;
    char tmpfile[256];
    char byte;

    /* Note that we have to use a different temp name here compared to the
     * one used by rewriteAppendOnlyFileBackground() function. */
    // 打开一个临时文件并初始化 rio 变量
    snprintf(tmpfile,256,"temp-rewriteaof-%d.aof", (int) getpid());
    fp = fopen(tmpfile,"w");
    if (!fp) {
        serverLog(LL_WARNING, "Opening the temp file for AOF rewrite in rewriteAppendOnlyFile(): %s", strerror(errno));
        return C_ERR;
    }

    server.aof_child_diff = sdsempty();
    rioInitWithFile(&aof,fp);

    if (server.aof_rewrite_incremental_fsync)
        rioSetAutoSync(&aof,REDIS_AUTOSYNC_BYTES);

    startSaving(RDBFLAGS_AOF_PREAMBLE);
	// 如果开启了 AOF 混合持久化，则生成 RDB 数据到临时文件中，否则，将 Redis 所有数据转化为写入命令写入临时文件
    if (server.aof_use_rdb_preamble) {
        int error;
        if (rdbSaveRio(&aof,&error,RDBFLAGS_AOF_PREAMBLE,NULL) == C_ERR) {
            errno = error;
            goto werr;
        }
    } else {
        if (rewriteAppendOnlyFileRio(&aof) == C_ERR) goto werr;
    }

    /* Do an initial slow fsync here while the parent is still sending
     * data, in order to make the next final fsync faster. */
    if (fflush(fp) == EOF) goto werr;
    if (fsync(fileno(fp)) == -1) goto werr;

    // 重复从 server.aof_pipe_read_data_from_parent 中读取增量命令，并放入 server.aof_child_diff 暂存区
    int nodata = 0;
    mstime_t start = mstime();
    // 读取增量命令的总时间超过 1 秒 或者 连续 20 毫秒没有读取到增量命令，则停止读取数据
    while(mstime()-start < 1000 && nodata < 20) {
        // aeWait 没有读取到增量命令时，会阻塞 1 毫秒等待增量命令
        if (aeWait(server.aof_pipe_read_data_from_parent, AE_READABLE, 1) <= 0)
        {
            nodata++;
            continue;
        }
        nodata = 0; /* Start counting from zero, we stop on N *contiguous*
                       timeouts. */
        aofReadDiffFromParent();
    }

    /* Ask the master to stop sending diffs. */
    // 向 server.aof_pipe_write_ack_to_parent 发送停止标志，通知父主进程停止发送增量命令
    if (write(server.aof_pipe_write_ack_to_parent,"!",1) != 1) goto werr;
    // 从 server.aof_pipe_read_ack_from_parent 读取父主进程的确认标志。此时，父子进程达成共识，父主进程不在发送增量命令到 server.aof_pipe_read_data_from_parent
    if (anetNonBlock(NULL,server.aof_pipe_read_ack_from_parent) != ANET_OK)
        goto werr;
    /* We read the ACK from the server using a 5 seconds timeout. Normally
     * it should reply ASAP, but just in case we lose its reply, we are sure
     * the child will eventually get terminated. */
    if (syncRead(server.aof_pipe_read_ack_from_parent,&byte,1,5000) != 1 ||
        byte != '!') goto werr;
    serverLog(LL_NOTICE,"Parent agreed to stop sending diffs. Finalizing AOF...");

    /* Read the final diff if any. */
    // 再一次从 server.aof_pipe_read_data_from_parent 中读取增量命令到 server.aof_child_diff 暂存区，以保证管道中所有的命令数据都被读取
    aofReadDiffFromParent();

    /* Write the received diff to the file. */
    serverLog(LL_NOTICE,
        "Concatenating %.2f MB of AOF diff received from parent.",
        (double) sdslen(server.aof_child_diff) / (1024*1024));

    size_t bytes_to_write = sdslen(server.aof_child_diff);
    const char *buf = server.aof_child_diff;
    long long cow_updated_time = mstime();
    long long key_count = dbTotalServerKeyCount();
    while (bytes_to_write) {
        /* We write the AOF buffer in chunk of 8MB so that we can check the time in between them */
        size_t chunk_size = bytes_to_write < (8<<20) ? bytes_to_write : (8<<20);
		// 将 server.aof_child_diff 暂存区内容写入文件并同步磁盘
        if (rioWrite(&aof,buf,chunk_size) == 0)
            goto werr;

        bytes_to_write -= chunk_size;
        buf += chunk_size;

        /* Update COW info */
        long long now = mstime();
        if (now - cow_updated_time >= 1000) {
            sendChildInfo(CHILD_INFO_TYPE_CURRENT_INFO, key_count, "AOF rewrite");
            cow_updated_time = now;
        }
    }

    /* Make sure data will not remain on the OS's output buffers */
    if (fflush(fp)) goto werr;
    if (fsync(fileno(fp))) goto werr;
    if (fclose(fp)) { fp = NULL; goto werr; }
    fp = NULL;

    /* Use RENAME to make sure the DB file is changed atomically only
     * if the generate DB file is ok. */
    // 将文件重命名，此时完成 AOF 重写的第 2 步
    if (rename(tmpfile,filename) == -1) {
        serverLog(LL_WARNING,"Error moving temp append only file on the final destination: %s", strerror(errno));
        unlink(tmpfile);
        stopSaving(0);
        return C_ERR;
    }
    serverLog(LL_NOTICE,"SYNC append only file rewrite performed");
    stopSaving(1);
    return C_OK;

werr:
    serverLog(LL_WARNING,"Write error writing append only file on disk: %s", strerror(errno));
    if (fp) fclose(fp);
    unlink(tmpfile);
    stopSaving(0);
    return C_ERR;
}
```

​		`aofRewriteBufferAppend`除了将命令写入`AOF`重写缓冲区，还会为`server.aof_pipe_write_data_to_child`管道注册一个`WRITE`事件回调函数

`aofChildWriteDiffData`，该函数会将重写缓冲区的内容写到`server.aof_pipe_read_data_from_parent`中。

​		`aofCreatePipes`函数创建匿名管道后，还会为`aof_pipe_read_ack_from_child`注册`READ`事件的回调函数`aofChildPipeReadable`，该函数在读取到`AOF`子进

程发送的停止标志后，会向`server.aof_pipe_write_ack_to_child`写入确认标志，并且删除`server.aof_pipe_write_data_to_child`上`WRITE`事件的回调函数，此

时父主进程不再向管道发送命令。



#### 父进程收尾

​		在`checkChildrenDone`中如果发现子进程是`AOF`进程，则调用`backgroundRewriteDoneHandler`：

```c++
// aof.c
void backgroundRewriteDoneHandler(int exitcode, int bysignal) {
    if (!bysignal && exitcode == 0) {
        int newfd, oldfd;
        char tmpfile[256];
        long long now = ustime();
        mstime_t latency;

        serverLog(LL_NOTICE,
            "Background AOF rewrite terminated with success");

        /* Flush the differences accumulated by the parent to the
         * rewritten AOF. */
        latencyStartMonitor(latency);
        // 打开 AOF 进程创建的临时文件
        snprintf(tmpfile,256,"temp-rewriteaof-bg-%d.aof",
            (int)server.child_pid);
        newfd = open(tmpfile,O_WRONLY|O_APPEND);
        if (newfd == -1) {
            serverLog(LL_WARNING,
                "Unable to open the temporary AOF produced by the child: %s", strerror(errno));
            goto cleanup;
        }
		// 再次将重新缓冲区内容写入临时文件，当父主进程不在发送命令到管道后，主进程执行的增量命令会暂存在重写缓冲区中
        if (aofRewriteBufferWrite(newfd) == -1) {
            serverLog(LL_WARNING,
                "Error trying to flush the parent diff to the rewritten AOF: %s", strerror(errno));
            close(newfd);
            goto cleanup;
        }
        latencyEndMonitor(latency);
        latencyAddSampleIfNeeded("aof-rewrite-diff-write",latency);
  
        if (server.aof_fsync == AOF_FSYNC_EVERYSEC) {
            aof_background_fsync(newfd);
        } else if (server.aof_fsync == AOF_FSYNC_ALWAYS) {
            latencyStartMonitor(latency);
            if (redis_fsync(newfd) == -1) {
                serverLog(LL_WARNING,
                    "Error trying to fsync the parent diff to the rewritten AOF: %s", strerror(errno));
                close(newfd);
                goto cleanup;
            }
            latencyEndMonitor(latency);
            latencyAddSampleIfNeeded("aof-rewrite-done-fsync",latency);
        }

        serverLog(LL_NOTICE,
            "Residual parent diff successfully flushed to the rewritten AOF (%.2f MB)", (double) aofRewriteBufferSize() / (1024*1024));

        if (server.aof_fd == -1) {
            /* AOF disabled */

            /* Don't care if this fails: oldfd will be -1 and we handle that.
             * One notable case of -1 return is if the old file does
             * not exist. */
            oldfd = open(server.aof_filename,O_RDONLY|O_NONBLOCK);
        } else {
            /* AOF enabled */
            oldfd = -1; /* We'll set this to the current AOF filedes later. */
        }

        /* Rename the temporary file. This will not unlink the target file if
         * it exists, because we reference it with "oldfd". */
        latencyStartMonitor(latency);
        // 将临时文件重命名为 server.aof_filename 指定的文件名，该 AOF 文件会替换旧的 AOF 文件
        if (rename(tmpfile,server.aof_filename) == -1) {
            serverLog(LL_WARNING,
                "Error trying to rename the temporary AOF file %s into %s: %s",
                tmpfile,
                server.aof_filename,
                strerror(errno));
            close(newfd);
            if (oldfd != -1) close(oldfd);
            goto cleanup;
        }
        // 磁盘同步并且清空 server.aof_buf 内容
        latencyEndMonitor(latency);
        latencyAddSampleIfNeeded("aof-rename",latency);

        if (server.aof_fd == -1) {
            /* AOF disabled, we don't need to set the AOF file descriptor
             * to this new file, so we can close it. */
            close(newfd);
        } else {
            /* AOF enabled, replace the old fd with the new one. */
            oldfd = server.aof_fd;
            server.aof_fd = newfd;
            server.aof_selected_db = -1; /* Make sure SELECT is re-issued */
            aofUpdateCurrentSize();
            server.aof_rewrite_base_size = server.aof_current_size;
            server.aof_fsync_offset = server.aof_current_size;
            server.aof_last_fsync = server.unixtime;

            /* Clear regular AOF buffer since its contents was just written to
             * the new AOF from the background rewrite buffer. */
            sdsfree(server.aof_buf);
            server.aof_buf = sdsempty();
        }

        server.aof_lastbgrewrite_status = C_OK;

        serverLog(LL_NOTICE, "Background AOF rewrite finished successfully");
        /* Change state from WAIT_REWRITE to ON if needed */
        if (server.aof_state == AOF_WAIT_REWRITE)
            server.aof_state = AOF_ON;

        /* Asynchronously close the overwritten AOF. */
        if (oldfd != -1) bioCreateCloseJob(oldfd);

        serverLog(LL_VERBOSE,
            "Background AOF rewrite signal handler took %lldus", ustime()-now);
    } else if (!bysignal && exitcode != 0) {
        server.aof_lastbgrewrite_status = C_ERR;

        serverLog(LL_WARNING,
            "Background AOF rewrite terminated with error");
    } else {
        /* SIGUSR1 is whitelisted, so we have a way to kill a child without
         * triggering an error condition. */
        if (bysignal != SIGUSR1)
            server.aof_lastbgrewrite_status = C_ERR;

        serverLog(LL_WARNING,
            "Background AOF rewrite terminated by signal %d", bysignal);
    }

cleanup:
    aofClosePipes();
    aofRewriteBufferReset();
    aofRemoveTempFile(server.child_pid);
    server.aof_rewrite_time_last = time(NULL)-server.aof_rewrite_time_start;
    server.aof_rewrite_time_start = -1;
    /* Schedule a new rewrite if we are waiting for it to switch the AOF ON. */
    if (server.aof_state == AOF_WAIT_REWRITE)
        server.aof_rewrite_scheduled = 1;
}
```

![](image/QQ截图20220423145402.png)



#### AOF 文件加载过程

​		`Redis`启动时会调用`loadDataFromDisk`加载持久化数据，如果服务器开启了`AOF`机制，则`loadDataFromDisk`会调用`loadAppendOnlyFile`从`AOF`文件中加载

数据：

```c++
int loadAppendOnlyFile(char *filename) {
    struct client *fakeClient;
    // 打开 AOF 文件
    FILE *fp = fopen(filename,"r");
    struct redis_stat sb;
    int old_aof_state = server.aof_state;
    long loops = 0;
    off_t valid_up_to = 0; /* Offset of latest well-formed command loaded. */
    off_t valid_before_multi = 0; /* Offset before MULTI command loaded. */

    if (fp == NULL) {
        serverLog(LL_WARNING,"Fatal error: can't open the append log file for reading: %s",strerror(errno));
        exit(1);
    }

    if (fp && redis_fstat(fileno(fp),&sb) != -1 && sb.st_size == 0) {
        server.aof_current_size = 0;
        server.aof_fsync_offset = server.aof_current_size;
        fclose(fp);
        return C_ERR;
    }

    server.aof_state = AOF_OFF;
	// 创建一个伪客户端，用于执行 AOF 文件中的命令
    fakeClient = createAOFClient();
    startLoadingFile(fp, filename, RDBFLAGS_AOF_PREAMBLE);

    // AOF 文件以 Redis 标志开头，则该 AOF 文件使用混合持久化方式生成，调用 rdbLoadRio 函数加载 RDB 内容
    char sig[5]; /* "REDIS" */
    if (fread(sig,1,5,fp) != 5 || memcmp(sig,"REDIS",5) != 0) {
        /* No RDB preamble, seek back at 0 offset. */
        if (fseek(fp,0,SEEK_SET) == -1) goto readerr;
    } else {
        /* RDB preamble. Pass loading the RDB functions. */
        rio rdb;

        serverLog(LL_NOTICE,"Reading RDB preamble from AOF file...");
        if (fseek(fp,0,SEEK_SET) == -1) goto readerr;
        rioInitWithFile(&rdb,fp);
        if (rdbLoadRio(&rdb,RDBFLAGS_AOF_PREAMBLE,NULL) != C_OK) {
            serverLog(LL_WARNING,"Error reading the RDB preamble of the AOF file, AOF loading aborted");
            goto readerr;
        } else {
            serverLog(LL_NOTICE,"Reading the remaining AOF tail...");
        }
    }

    // 开始处理 AOF 文件中的命令
    while(1) {
        int argc, j;
        unsigned long len;
        robj **argv;
        char buf[128];
        sds argsds;
        struct redisCommand *cmd;
        
        // 按照 RESP 协议格式，读取命令参数数量
        if (!(loops++ % 1000)) {
            loadingProgress(ftello(fp));
            processEventsWhileBlocked();
            processModuleLoadingProgressEvent(1);
        }

        if (fgets(buf,sizeof(buf),fp) == NULL) {
            if (feof(fp))
                break;
            else
                goto readerr;
        }
        if (buf[0] != '*') goto fmterr;
        if (buf[1] == '\0') goto readerr;
        argc = atoi(buf+1);
        if (argc < 1) goto fmterr;

        /* Load the next command in the AOF as our fake client
         * argv. */
        argv = zmalloc(sizeof(robj*)*argc);
        fakeClient->argc = argc;
        fakeClient->argv = argv;
		// 读取每一个参数
        for (j = 0; j < argc; j++) {
            /* Parse the argument len. */
            char *readres = fgets(buf,sizeof(buf),fp);
            if (readres == NULL || buf[0] != '$') {
                fakeClient->argc = j; /* Free up to j-1. */
                freeFakeClientArgv(fakeClient);
                if (readres == NULL)
                    goto readerr;
                else
                    goto fmterr;
            }
            len = strtol(buf+1,NULL,10);

            /* Read it into a string object. */
            argsds = sdsnewlen(SDS_NOINIT,len);
            if (len && fread(argsds,len,1,fp) == 0) {
                sdsfree(argsds);
                fakeClient->argc = j; /* Free up to j-1. */
                freeFakeClientArgv(fakeClient);
                goto readerr;
            }
            argv[j] = createObject(OBJ_STRING,argsds);

            /* Discard CRLF. */
            if (fread(buf,2,1,fp) == 0) {
                fakeClient->argc = j+1; /* Free up to j. */
                freeFakeClientArgv(fakeClient);
                goto readerr;
            }
        }

        /* Command lookup */
        // 查找命令 redisCommand
        cmd = lookupCommand(argv[0]->ptr);
        if (!cmd) {
            serverLog(LL_WARNING,
                "Unknown command '%s' reading the append only file",
                (char*)argv[0]->ptr);
            exit(1);
        }

        if (cmd == server.multiCommand) valid_before_multi = valid_up_to;

        /* Run the command in the context of a fake client */
        // 调用 redisCommand.proc 执行命令
        fakeClient->cmd = fakeClient->lastcmd = cmd;
        if (fakeClient->flags & CLIENT_MULTI &&
            fakeClient->cmd->proc != execCommand)
        {
            queueMultiCommand(fakeClient);
        } else {
            cmd->proc(fakeClient);
        }

        ...
    }
    ...
}
```



### 主从复制

​		`Redis`主从复制机制中有两个角色：主节点与从节点。主节点处理用户请求，并将数据复制给从节点。主要作用：

​				1、数据冗余，将数据热备份到从节点，即使主节点由于磁盘损坏丢失数据，从节点依然保留数据副本。

​				2、读`/`写分离，可以由主节点提供写服务，从节点提供读服务，提高`Redis`服务整体吞吐量。

​				3、故障恢复，主节点故障下线后，可以手动将从节点切换为主节点，继续提供服务。

​				4、高可用基础，主从复制机制是`Sentinel`和`Cluster`机制的基础。

​		主从复制流程：

​				1、握手阶段：主从连接成功后，主节点需要将自身信息发送给主节点，以便主节点能认识自己。

​				2、同步阶段：从节点连接主节点后，需要先同步数据，数据达到一致`(`或者只有最新的变更不一致`)`后才能进入复制阶段。

​				3、复制节点：主节点在运行期间，将执行的写命令传播给从节点，从节点接收并执行这些命令，从而达到复制数据的效果。`Redis`使用的是异步复制，

​		主节点传播命令后，并不会等待从节点返回`ACK`确认。异步复制的优点是低延迟和高性能，缺点是可能在短期内主从节点数据不一致。



​		同步机制：

​				全量同步：从节点发送命令`PSYNC ? -1`，要求进行全量同步，主节点返回响应`+FULLRESYNC`，表明同意全量同步。随后，主节点生成`RDB`数据并发送给

​		从节点。常用于新的从节点首次同步数据。

​				部分同步：从节点发送命令`PSYNC replid offset`，要求进行部分同步，主节点响应`+CONTINUE`，表明同意部分同步。主节点只需要吧复制积压区中

​		`offset`偏移量之后的命令发送给从节点即可`(`主节点会将执行的写命令都写入复制积压区`)`。常用于主从连接断开重连时同步数据。如果`offset`不在复制

​		积压区，那么主节点也会返回`+FULLRESYNC`，要求进行全量同步。



#### 主从握手

​		主从复制的机制是由从节点发起流程，通过发送`replicaof`命令到某个服务器，或者在配置文件中配置`replicaof`配置项，这样`Redis`服务器启动后将成为指

定服务器的从节点。

​		从节点使用`replicaofCommand`函数处理`replicaof`命令：

​				如果处理的命令时`replicaof no one`，则将当前服务器转换为主节点，取消原来的主从复制关系，退出函数。

​				调用`replicationSetMaster`函数，与给定服务器建立主从复制关系。

```c++
// replication.c
void replicationSetMaster(char *ip, int port) {
    int was_master = server.masterhost == NULL;

    sdsfree(server.masterhost);
    server.masterhost = NULL;
    // 如果当前已连接了主节点，则断开原来的主从连接
    if (server.master) {
        freeClient(server.master);
    }
    disconnectAllBlockedClients(); /* Clients blocked in master, now slave. */

    server.masterhost = sdsnew(ip);
    server.masterport = port;

    /* Update oom_score_adj */
    setOOMScoreAdj(-1);
	// 断开当前服务器所有从节点的连接，是这些从节点重新发起同步流程
    disconnectSlaves();
    cancelReplicationHandshake(0);

    if (was_master) {
        replicationDiscardCachedMaster();
        replicationCacheMasterUsingMyself();
    }

    /* Fire the role change modules event. */
    moduleFireServerEvent(REDISMODULE_EVENT_REPLICATION_ROLE_CHANGED,
                          REDISMODULE_EVENT_REPLROLECHANGED_NOW_REPLICA,
                          NULL);

    /* Fire the master link modules event. */
    if (server.repl_state == REPL_STATE_CONNECTED)
        moduleFireServerEvent(REDISMODULE_EVENT_MASTER_LINK_CHANGE,
                              REDISMODULE_SUBEVENT_MASTER_LINK_DOWN,
                              NULL);
	// 将 server.repl_state 设置为 REPL_STATE_CONNECT 状态。server.repl_state 用于欧从节点，标志从节点当前复制状态
    server.repl_state = REPL_STATE_CONNECT;
    serverLog(LL_NOTICE,"Connecting to MASTER %s:%d",
        server.masterhost, server.masterport);
    connectWithMaster();
}
```

​		如果在配置文件中配置`replicaof`配置项，`Redis`加载该配置，也会将`server.repl_state`设置为`REPL_STATE_CONNECT`。

​		`serverCron`时间事件负责对`REPL_STATE_CONNECT`状态进行处理，然后调用`connectWithMaster`进行处理，该函数负责建立主从网络连接：

```c++
// replication.c
int connectWithMaster(void) {
    // 创建一个 Socket 套接字，两个函数都返回套接字文件描述符。该连接是主从节点网络通信的连接
    server.repl_transfer_s = server.tls_replication ? connCreateTLS() : connCreateSocket();
    // 连接到主节点，并且在连接成功后调用 syncWithMaster 函数
    if (connConnect(server.repl_transfer_s, server.masterhost, server.masterport,
                NET_FIRST_BIND_ADDR, syncWithMaster) == C_ERR) {
        serverLog(LL_WARNING,"Unable to connect to MASTER: %s",
                connGetLastError(server.repl_transfer_s));
        connClose(server.repl_transfer_s);
        server.repl_transfer_s = NULL;
        return C_ERR;
    }

	// 从节点 server.repl_state 进入 REPL_STATE_CONNECTING
    server.repl_transfer_lastio = server.unixtime;
    server.repl_state = REPL_STATE_CONNECTING;
    serverLog(LL_NOTICE,"MASTER <-> REPLICA sync started");
    return C_OK;
}
```

​		网络连接成功后，从节点调用`syncWithMaster`，进入握手阶段：

```c++
// replication.c
void syncWithMaster(connection *conn) {
    char tmpfile[256], *err = NULL;
    int dfd = -1, maxtries = 5;
    int psync_result;

    if (server.repl_state == REPL_STATE_NONE) {
        connClose(conn);
        return;
    }

    if (connGetState(conn) != CONN_STATE_CONNECTED) {
        serverLog(LL_WARNING,"Error condition on socket for SYNC: %s",
                connGetLastError(conn));
        goto error;
    }

    if (server.repl_state == REPL_STATE_CONNECTING) {
        serverLog(LL_NOTICE,"Non blocking connect for SYNC fired the event.");
        connSetReadHandler(conn, syncWithMaster);
        connSetWriteHandler(conn, NULL);
        server.repl_state = REPL_STATE_RECEIVE_PING_REPLY;
        err = sendCommand(conn,"PING",NULL);
        if (err) goto write_error;
        return;
    }
	
    if (server.repl_state == REPL_STATE_RECEIVE_PING_REPLY) {
        err = receiveSynchronousResponse(conn);
        if (err[0] != '+' &&
            strncmp(err,"-NOAUTH",7) != 0 &&
            strncmp(err,"-NOPERM",7) != 0 &&
            strncmp(err,"-ERR operation not permitted",28) != 0)
        {
            serverLog(LL_WARNING,"Error reply to PING from master: '%s'",err);
            sdsfree(err);
            goto error;
        } else {
            serverLog(LL_NOTICE,
                "Master replied to PING, replication can continue...");
        }
        sdsfree(err);
        err = NULL;
        server.repl_state = REPL_STATE_SEND_HANDSHAKE;
    }
	// 
    if (server.repl_state == REPL_STATE_SEND_HANDSHAKE) {
        /* AUTH with the master if required. */
        if (server.masterauth) {
            char *args[3] = {"AUTH",NULL,NULL};
            size_t lens[3] = {4,0,0};
            int argc = 1;
            if (server.masteruser) {
                args[argc] = server.masteruser;
                lens[argc] = strlen(server.masteruser);
                argc++;
            }
            args[argc] = server.masterauth;
            lens[argc] = sdslen(server.masterauth);
            argc++;
            err = sendCommandArgv(conn, argc, args, lens);
            if (err) goto write_error;
        }

        /* Set the slave port, so that Master's INFO command can list the
         * slave listening port correctly. */
        {
            int port;
            if (server.slave_announce_port)
                port = server.slave_announce_port;
            else if (server.tls_replication && server.tls_port)
                port = server.tls_port;
            else
                port = server.port;
            sds portstr = sdsfromlonglong(port);
            err = sendCommand(conn,"REPLCONF",
                    "listening-port",portstr, NULL);
            sdsfree(portstr);
            if (err) goto write_error;
        }

        /* Set the slave ip, so that Master's INFO command can list the
         * slave IP address port correctly in case of port forwarding or NAT.
         * Skip REPLCONF ip-address if there is no slave-announce-ip option set. */
        if (server.slave_announce_ip) {
            err = sendCommand(conn,"REPLCONF",
                    "ip-address",server.slave_announce_ip, NULL);
            if (err) goto write_error;
        }

        /* Inform the master of our (slave) capabilities.
         *
         * EOF: supports EOF-style RDB transfer for diskless replication.
         * PSYNC2: supports PSYNC v2, so understands +CONTINUE <new repl ID>.
         *
         * The master will ignore capabilities it does not understand. */
        err = sendCommand(conn,"REPLCONF",
                "capa","eof","capa","psync2",NULL);
        if (err) goto write_error;

        server.repl_state = REPL_STATE_RECEIVE_AUTH_REPLY;
        return;
    }

    if (server.repl_state == REPL_STATE_RECEIVE_AUTH_REPLY && !server.masterauth)
        server.repl_state = REPL_STATE_RECEIVE_PORT_REPLY;

    /* Receive AUTH reply. */
    if (server.repl_state == REPL_STATE_RECEIVE_AUTH_REPLY) {
        err = receiveSynchronousResponse(conn);
        if (err[0] == '-') {
            serverLog(LL_WARNING,"Unable to AUTH to MASTER: %s",err);
            sdsfree(err);
            goto error;
        }
        sdsfree(err);
        err = NULL;
        server.repl_state = REPL_STATE_RECEIVE_PORT_REPLY;
        return;
    }

    /* Receive REPLCONF listening-port reply. */
    if (server.repl_state == REPL_STATE_RECEIVE_PORT_REPLY) {
        err = receiveSynchronousResponse(conn);
        /* Ignore the error if any, not all the Redis versions support
         * REPLCONF listening-port. */
        if (err[0] == '-') {
            serverLog(LL_NOTICE,"(Non critical) Master does not understand "
                                "REPLCONF listening-port: %s", err);
        }
        sdsfree(err);
        server.repl_state = REPL_STATE_RECEIVE_IP_REPLY;
        return;
    }

    if (server.repl_state == REPL_STATE_RECEIVE_IP_REPLY && !server.slave_announce_ip)
        server.repl_state = REPL_STATE_RECEIVE_CAPA_REPLY;

    /* Receive REPLCONF ip-address reply. */
    if (server.repl_state == REPL_STATE_RECEIVE_IP_REPLY) {
        err = receiveSynchronousResponse(conn);
        /* Ignore the error if any, not all the Redis versions support
         * REPLCONF listening-port. */
        if (err[0] == '-') {
            serverLog(LL_NOTICE,"(Non critical) Master does not understand "
                                "REPLCONF ip-address: %s", err);
        }
        sdsfree(err);
        server.repl_state = REPL_STATE_RECEIVE_CAPA_REPLY;
        return;
    }

    /* Receive CAPA reply. */
    if (server.repl_state == REPL_STATE_RECEIVE_CAPA_REPLY) {
        err = receiveSynchronousResponse(conn);
        /* Ignore the error if any, not all the Redis versions support
         * REPLCONF capa. */
        if (err[0] == '-') {
            serverLog(LL_NOTICE,"(Non critical) Master does not understand "
                                  "REPLCONF capa: %s", err);
        }
        sdsfree(err);
        err = NULL;
        server.repl_state = REPL_STATE_SEND_PSYNC;
    }

    if (server.repl_state == REPL_STATE_SEND_PSYNC) {
        if (slaveTryPartialResynchronization(conn,0) == PSYNC_WRITE_ERROR) {
            err = sdsnew("Write error sending the PSYNC command.");
            abortFailover("Write error to failover target");
            goto write_error;
        }
        server.repl_state = REPL_STATE_RECEIVE_PSYNC_REPLY;
        return;
    }

    /* If reached this point, we should be in REPL_STATE_RECEIVE_PSYNC. */
    // 主从握手已经完成，server.repl_state 必须处于 REPL_STATE_RECEIVE_PSYNC_REPLY 否则报错
    if (server.repl_state != REPL_STATE_RECEIVE_PSYNC_REPLY) {
        serverLog(LL_WARNING,"syncWithMaster(): state machine error, "
                             "state should be RECEIVE_PSYNC but is %d",
                             server.repl_state);
        goto error;
    }

    psync_result = slaveTryPartialResynchronization(conn,1);
    if (psync_result == PSYNC_WAIT_REPLY) return; /* Try again later... */

    if (server.failover_state == FAILOVER_IN_PROGRESS) {
        if (psync_result == PSYNC_CONTINUE || psync_result == PSYNC_FULLRESYNC) {
            clearFailoverState();
        } else {
            abortFailover("Failover target rejected psync request");
            return;
        }
    }

    if (psync_result == PSYNC_TRY_LATER) goto error;

    if (psync_result == PSYNC_CONTINUE) {
        serverLog(LL_NOTICE, "MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.");
        if (server.supervised_mode == SUPERVISED_SYSTEMD) {
            redisCommunicateSystemd("STATUS=MASTER <-> REPLICA sync: Partial Resynchronization accepted. Ready to accept connections in read-write mode.\n");
        }
        return;
    }

    disconnectSlaves(); /* Force our slaves to resync with us as well. */
    freeReplicationBacklog(); /* Don't allow our chained slaves to PSYNC. */

    if (psync_result == PSYNC_NOT_SUPPORTED) {
        serverLog(LL_NOTICE,"Retrying with SYNC...");
        if (connSyncWrite(conn,"SYNC\r\n",6,server.repl_syncio_timeout*1000) == -1) {
            serverLog(LL_WARNING,"I/O error writing to MASTER: %s",
                strerror(errno));
            goto error;
        }
    }

    /* Prepare a suitable temp file for bulk transfer */
    if (!useDisklessLoad()) {
        while(maxtries--) {
            snprintf(tmpfile,256,
                "temp-%d.%ld.rdb",(int)server.unixtime,(long int)getpid());
            dfd = open(tmpfile,O_CREAT|O_WRONLY|O_EXCL,0644);
            if (dfd != -1) break;
            sleep(1);
        }
        if (dfd == -1) {
            serverLog(LL_WARNING,"Opening the temp file needed for MASTER <-> REPLICA synchronization: %s",strerror(errno));
            goto error;
        }
        server.repl_transfer_tmpfile = zstrdup(tmpfile);
        server.repl_transfer_fd = dfd;
    }

    /* Setup the non blocking download of the bulk file. */
    if (connSetReadHandler(conn, readSyncBulkPayload)
            == C_ERR)
    {
        char conninfo[CONN_INFO_LEN];
        serverLog(LL_WARNING,
            "Can't create readable event for SYNC: %s (%s)",
            strerror(errno), connGetInfo(conn, conninfo, sizeof(conninfo)));
        goto error;
    }

    server.repl_state = REPL_STATE_TRANSFER;
    server.repl_transfer_size = -1;
    server.repl_transfer_read = 0;
    server.repl_transfer_last_fsync_off = 0;
    server.repl_transfer_lastio = server.unixtime;
    return;

error:
    if (dfd != -1) close(dfd);
    connClose(conn);
    server.repl_transfer_s = NULL;
    if (server.repl_transfer_fd != -1)
        close(server.repl_transfer_fd);
    if (server.repl_transfer_tmpfile)
        zfree(server.repl_transfer_tmpfile);
    server.repl_transfer_tmpfile = NULL;
    server.repl_transfer_fd = -1;
    server.repl_state = REPL_STATE_CONNECT;
    return;

write_error: /* Handle sendCommand() errors. */
    serverLog(LL_WARNING,"Sending command to master in replication handshake: %s", err);
    sdsfree(err);
    goto error;
}
```



#### 从节点同步

##### 部分同步

​		`slaveTryPartialResynchronization`函数，负责发送`psync`命令及处理主节点对`psync`命令的响应，当`read_reply`为`0`，负责发送`psync`命令；当

`read_reply`为`1`，处理主节点对`psync`命令的响应。

```c++
// replication.c
int slaveTryPartialResynchronization(connection *conn, int read_reply) {
    char *psync_replid;
    char psync_offset[32];
    sds reply;
	// read_reply 为 0 ，发送 psync 命令
    if (!read_reply) {
        server.master_initial_offset = -1;

        if (server.cached_master) {
            //  server.cached_master 不为空，使用 server.cached_master 信息发送 psync 命令，尝试发起部分同步流程
            psync_replid = server.cached_master->replid;
            snprintf(psync_offset,sizeof(psync_offset),"%lld", server.cached_master->reploff+1);
            serverLog(LL_NOTICE,"Trying a partial resynchronization (request %s:%s).", psync_replid, psync_offset);
        } else {
            serverLog(LL_NOTICE,"Partial resynchronization not possible (no cached master)");
            // server.cached_master 不存在，只能使用全量同步机制，发送 psync ? -1 命令
            psync_replid = "?";
            memcpy(psync_offset,"-1",3);
        }
		// 发送 psync 命令
        if (server.failover_state == FAILOVER_IN_PROGRESS) {
            reply = sendCommand(conn,"PSYNC",psync_replid,psync_offset,"FAILOVER",NULL);
        } else {
            reply = sendCommand(conn,"PSYNC",psync_replid,psync_offset,NULL);
        }

        if (reply != NULL) {
            serverLog(LL_WARNING,"Unable to send PSYNC to master: %s",reply);
            sdsfree(reply);
            connSetReadHandler(conn, NULL);
            return PSYNC_WRITE_ERROR;
        }
        return PSYNC_WAIT_REPLY;
    }
	// read_reply 不为 0，读取主节点对 psync 命令的响应数据
    reply = receiveSynchronousResponse(conn);
    // 主节点并没有返回有效数据
    if (sdslen(reply) == 0) {
        sdsfree(reply);
        return PSYNC_WAIT_REPLY;
    }

    connSetReadHandler(conn, NULL);
	// 主节点要求进行全量同步
    if (!strncmp(reply,"+FULLRESYNC",11)) {
        char *replid = NULL, *offset = NULL;

        replid = strchr(reply,' ');
        if (replid) {
            replid++;
            offset = strchr(replid,' ');
            if (offset) offset++;
        }
        if (!replid || !offset || (offset-replid-1) != CONFIG_RUN_ID_SIZE) {
            serverLog(LL_WARNING,
                "Master replied with wrong +FULLRESYNC syntax.");
            memset(server.master_replid,0,CONFIG_RUN_ID_SIZE+1);
        } else {
            memcpy(server.master_replid, replid, offset-replid-1);
            server.master_replid[CONFIG_RUN_ID_SIZE] = '\0';
            server.master_initial_offset = strtoll(offset,NULL,10);
            serverLog(LL_NOTICE,"Full resync from master: %s:%lld",
                server.master_replid,
                server.master_initial_offset);
        }
        replicationDiscardCachedMaster();
        sdsfree(reply);
        return PSYNC_FULLRESYNC;
    }
	// 可以进行部分同步，
    if (!strncmp(reply,"+CONTINUE",9)) {
        serverLog(LL_NOTICE,
            "Successful partial resynchronization with master.");

        char *start = reply+10;
        char *end = reply+9;
        while(end[0] != '\r' && end[0] != '\n' && end[0] != '\0') end++;
        if (end-start == CONFIG_RUN_ID_SIZE) {
            char new[CONFIG_RUN_ID_SIZE+1];
            memcpy(new,start,CONFIG_RUN_ID_SIZE);
            new[CONFIG_RUN_ID_SIZE] = '\0';

            if (strcmp(new,server.cached_master->replid)) {
                serverLog(LL_WARNING,"Master replication ID changed to %s",new);

                memcpy(server.replid2,server.cached_master->replid,
                    sizeof(server.replid2));
                server.second_replid_offset = server.master_repl_offset+1;

                memcpy(server.replid,new,sizeof(server.replid));
                memcpy(server.cached_master->replid,new,sizeof(server.replid));

                disconnectSlaves();
            }
        }

        sdsfree(reply);
        // 部分同步的逻辑处理
        /**
        	1、使用当前主从连接，将 server.cached_master 转化为 server.master
        	2、为主从连接注册 read 事件回调函数 readQueryFromClient，负责接收并处理主节点传播的命令
        	3、server.repl_state 进入 REPL_STATE_CONNECTED 状态
        	最后，初始化从节点复制积压区，从节点部分同步完成，进入复制阶段
        */
        replicationResurrectCachedMaster(conn);

        if (server.repl_backlog == NULL) createReplicationBacklog();
        return PSYNC_CONTINUE;
    }
    

    if (!strncmp(reply,"-NOMASTERLINK",13) ||
        !strncmp(reply,"-LOADING",8))
    {
        serverLog(LL_NOTICE,
            "Master is currently unable to PSYNC "
            "but should be in the future: %s", reply);
        sdsfree(reply);
        return PSYNC_TRY_LATER;
    }

    if (strncmp(reply,"-ERR",4)) {
        /* If it's not an error, log the unexpected event. */
        serverLog(LL_WARNING,
            "Unexpected reply to PSYNC from master: %s", reply);
    } else {
        serverLog(LL_NOTICE,
            "Master does not support PSYNC or is in "
            "error state (reply: %s)", reply);
    }
	// 主节点不支持 psync 命令
    sdsfree(reply);
    replicationDiscardCachedMaster();
    return PSYNC_NOT_SUPPORTED;
}
```



##### 全量同步

​		在`syncWithMaster`中，在`server.repl_state`进入`REPL_STATE_RECEIVE_PSYNC`状态后处理全量同步：

```c++
// replication.c
void syncWithMaster(connetion *conn) {
	...
    // 读取主节点对 psync 命令的响应数据
    psync_result = slaveTryPartialResynchronization(conn,1);
    // 主节点未返回有效数据
    if (psync_result == PSYNC_WAIT_REPLY) return; /* Try again later... */

    if (server.failover_state == FAILOVER_IN_PROGRESS) {
        if (psync_result == PSYNC_CONTINUE || psync_result == PSYNC_FULLRESYNC) {
            clearFailoverState();
        } else {
            abortFailover("Failover target rejected psync request");
            return;
        }
    }

    // 进入异常处理逻辑，将 server.repl_state 设置为 REPL_STATE_CONNECT，重新发起同步流程
    if (psync_result == PSYNC_TRY_LATER) goto error;
	// 主节点同意部分同步
    if (psync_result == PSYNC_CONTINUE) {
        serverLog(LL_NOTICE, "MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.");
        if (server.supervised_mode == SUPERVISED_SYSTEMD) {
            redisCommunicateSystemd("STATUS=MASTER <-> REPLICA sync: Partial Resynchronization accepted. Ready to accept connections in read-write mode.\n");
        }
        return;
    }

    disconnectSlaves(); /* Force our slaves to resync with us as well. */
    freeReplicationBacklog(); /* Don't allow our chained slaves to PSYNC. */
	// 主节点不支持 psync 命令，重新发送 sync 命令
    if (psync_result == PSYNC_NOT_SUPPORTED) {
        serverLog(LL_NOTICE,"Retrying with SYNC...");
        if (connSyncWrite(conn,"SYNC\r\n",6,server.repl_syncio_timeout*1000) == -1) {
            serverLog(LL_WARNING,"I/O error writing to MASTER: %s",
                strerror(errno));
            goto error;
        }
    }

    if (!useDisklessLoad()) {
        while(maxtries--) {
            snprintf(tmpfile,256,
                "temp-%d.%ld.rdb",(int)server.unixtime,(long int)getpid());
            dfd = open(tmpfile,O_CREAT|O_WRONLY|O_EXCL,0644);
            if (dfd != -1) break;
            sleep(1);
        }
        if (dfd == -1) {
            serverLog(LL_WARNING,"Opening the temp file needed for MASTER <-> REPLICA synchronization: %s",strerror(errno));
            goto error;
        }
        server.repl_transfer_tmpfile = zstrdup(tmpfile);
        server.repl_transfer_fd = dfd;
    }
	// 此时需要进行全量同步，为主从连接设置 read 事件处理函数 readSyncBulkPayload，该函数负责接收主节点发送的 RDB 数据，server.repl_state 进入 REPL_STATE_TRANSFER 状态
    if (connSetReadHandler(conn, readSyncBulkPayload)
            == C_ERR)
    {
        char conninfo[CONN_INFO_LEN];
        serverLog(LL_WARNING,
            "Can't create readable event for SYNC: %s (%s)",
            strerror(errno), connGetInfo(conn, conninfo, sizeof(conninfo)));
        goto error;
    }

    server.repl_state = REPL_STATE_TRANSFER;
    server.repl_transfer_size = -1;
    server.repl_transfer_read = 0;
    server.repl_transfer_last_fsync_off = 0;
    server.repl_transfer_lastio = server.unixtime;
    return;

error:
    if (dfd != -1) close(dfd);
    connClose(conn);
    server.repl_transfer_s = NULL;
    if (server.repl_transfer_fd != -1)
        close(server.repl_transfer_fd);
    if (server.repl_transfer_tmpfile)
        zfree(server.repl_transfer_tmpfile);
    server.repl_transfer_tmpfile = NULL;
    server.repl_transfer_fd = -1;
    server.repl_state = REPL_STATE_CONNECT;
    return;

write_error: /* Handle sendCommand() errors. */
    serverLog(LL_WARNING,"Sending command to master in replication handshake: %s", err);
    sdsfree(err);
    goto error;
}

void readSyncBulkPayload(connection *conn) {
    char buf[PROTO_IOBUF_LEN];
    ssize_t nread, readlen, nwritten;
    int use_diskless_load = useDisklessLoad();
    dbBackup *diskless_load_backup = NULL;
    int empty_db_flags = server.repl_slave_lazy_flush ? EMPTYDB_ASYNC :
                                                        EMPTYDB_NO_FLAGS;
    off_t left;

    static char eofmark[CONFIG_RUN_ID_SIZE];
    static char lastbytes[CONFIG_RUN_ID_SIZE];
    static int usemark = 0;

    if (server.repl_transfer_size == -1) {
        if (connSyncReadLine(conn,buf,1024,server.repl_syncio_timeout*1000) == -1) {
            serverLog(LL_WARNING,
                "I/O error reading bulk count from MASTER: %s",
                strerror(errno));
            goto error;
        }

        if (buf[0] == '-') {
            serverLog(LL_WARNING,
                "MASTER aborted replication with an error: %s",
                buf+1);
            goto error;
        } else if (buf[0] == '\0') {
            /* At this stage just a newline works as a PING in order to take
             * the connection live. So we refresh our last interaction
             * timestamp. */
            server.repl_transfer_lastio = server.unixtime;
            return;
        } else if (buf[0] != '$') {
            serverLog(LL_WARNING,"Bad protocol from MASTER, the first byte is not '$' (we received '%s'), are you sure the host and port are right?", buf);
            goto error;
        }
        // 判断主节点发送 RDB 数据的格式，以$EOF开头为不定长格式
        if (strncmp(buf+1,"EOF:",4) == 0 && strlen(buf+5) >= CONFIG_RUN_ID_SIZE) {
            usemark = 1;
            memcpy(eofmark,buf+5,CONFIG_RUN_ID_SIZE);
            memset(lastbytes,0,CONFIG_RUN_ID_SIZE);
            /* Set any repl_transfer_size to avoid entering this code path
             * at the next call. */
            server.repl_transfer_size = 0;
            serverLog(LL_NOTICE,
                "MASTER <-> REPLICA sync: receiving streamed RDB from master with EOF %s",
                use_diskless_load? "to parser":"to disk");
        } else {
            // 定长格式
            usemark = 0;
            server.repl_transfer_size = strtol(buf+1,NULL,10);
            serverLog(LL_NOTICE,
                "MASTER <-> REPLICA sync: receiving %lld bytes from master %s",
                (long long) server.repl_transfer_size,
                use_diskless_load? "to parser":"to disk");
        }
        return;
    }
	// 从 Socket 读取 RDB 数据
    if (!use_diskless_load) {
        /* Read the data from the socket, store it to a file and search
         * for the EOF. */
        if (usemark) {
            readlen = sizeof(buf);
        } else {
            left = server.repl_transfer_size - server.repl_transfer_read;
            readlen = (left < (signed)sizeof(buf)) ? left : (signed)sizeof(buf);
        }

        nread = connRead(conn,buf,readlen);
        if (nread <= 0) {
            if (connGetState(conn) == CONN_STATE_CONNECTED) {
                /* equivalent to EAGAIN */
                return;
            }
            serverLog(LL_WARNING,"I/O error trying to sync with MASTER: %s",
                (nread == -1) ? strerror(errno) : "connection lost");
            cancelReplicationHandshake(1);
            return;
        }
        atomicIncr(server.stat_net_input_bytes, nread);

        /* When a mark is used, we want to detect EOF asap in order to avoid
         * writing the EOF mark into the file... */
        int eof_reached = 0;

        if (usemark) {
            /* Update the last bytes array, and check if it matches our
             * delimiter. */
            if (nread >= CONFIG_RUN_ID_SIZE) {
                memcpy(lastbytes,buf+nread-CONFIG_RUN_ID_SIZE,
                       CONFIG_RUN_ID_SIZE);
            } else {
                int rem = CONFIG_RUN_ID_SIZE-nread;
                memmove(lastbytes,lastbytes+nread,rem);
                memcpy(lastbytes+rem,buf,nread);
            }
            if (memcmp(lastbytes,eofmark,CONFIG_RUN_ID_SIZE) == 0)
                eof_reached = 1;
        }

        /* Update the last I/O time for the replication transfer (used in
         * order to detect timeouts during replication), and write what we
         * got from the socket to the dump file on disk. */
        server.repl_transfer_lastio = server.unixtime;
        if ((nwritten = write(server.repl_transfer_fd,buf,nread)) != nread) {
            serverLog(LL_WARNING,
                "Write error or short write writing to the DB dump file "
                "needed for MASTER <-> REPLICA synchronization: %s",
                (nwritten == -1) ? strerror(errno) : "short write");
            goto error;
        }
        server.repl_transfer_read += nread;

        /* Delete the last 40 bytes from the file if we reached EOF. */
        if (usemark && eof_reached) {
            if (ftruncate(server.repl_transfer_fd,
                server.repl_transfer_read - CONFIG_RUN_ID_SIZE) == -1)
            {
                serverLog(LL_WARNING,
                    "Error truncating the RDB file received from the master "
                    "for SYNC: %s", strerror(errno));
                goto error;
            }
        }

        /* Sync data on disk from time to time, otherwise at the end of the
         * transfer we may suffer a big delay as the memory buffers are copied
         * into the actual disk. */
        if (server.repl_transfer_read >=
            server.repl_transfer_last_fsync_off + REPL_MAX_WRITTEN_BEFORE_FSYNC)
        {
            off_t sync_size = server.repl_transfer_read -
                              server.repl_transfer_last_fsync_off;
            rdb_fsync_range(server.repl_transfer_fd,
                server.repl_transfer_last_fsync_off, sync_size);
            server.repl_transfer_last_fsync_off += sync_size;
        }

        /* Check if the transfer is now complete */
        if (!usemark) {
            if (server.repl_transfer_read == server.repl_transfer_size)
                eof_reached = 1;
        }

        /* If the transfer is yet not complete, we need to read more, so
         * return ASAP and wait for the handler to be called again. */
        if (!eof_reached) return;
    }

    serverLog(LL_NOTICE, "MASTER <-> REPLICA sync: Flushing old data");


    if (server.aof_state != AOF_OFF) stopAppendOnly();

    if (use_diskless_load &&
        server.repl_diskless_load == REPL_DISKLESS_LOAD_SWAPDB)
    {
        /* Create a backup of server.db[] and initialize to empty
         * dictionaries. */
        diskless_load_backup = disklessLoadMakeBackup();
    }
    /* We call to emptyDb even in case of REPL_DISKLESS_LOAD_SWAPDB
     * (Where disklessLoadMakeBackup left server.db empty) because we
     * want to execute all the auxiliary logic of emptyDb (Namely,
     * fire module events) */
    emptyDb(-1,empty_db_flags,replicationEmptyDbCallback);

    /* Before loading the DB into memory we need to delete the readable
     * handler, otherwise it will get called recursively since
     * rdbLoad() will call the event loop to process events from time to
     * time for non blocking loading. */
    connSetReadHandler(conn, NULL);
    serverLog(LL_NOTICE, "MASTER <-> REPLICA sync: Loading DB in memory");
    rdbSaveInfo rsi = RDB_SAVE_INFO_INIT;
    // 使用磁盘加载
    if (use_diskless_load) {
        rio rdb;
        rioInitWithConn(&rdb,conn,server.repl_transfer_size);

        /* Put the socket in blocking mode to simplify RDB transfer.
         * We'll restore it when the RDB is received. */
        connBlock(conn);
        connRecvTimeout(conn, server.repl_timeout*1000);
        startLoading(server.repl_transfer_size, RDBFLAGS_REPLICATION);

        if (rdbLoadRio(&rdb,RDBFLAGS_REPLICATION,&rsi) != C_OK) {
            /* RDB loading failed. */
            stopLoading(0);
            serverLog(LL_WARNING,
                "Failed trying to load the MASTER synchronization DB "
                "from socket");
            cancelReplicationHandshake(1);
            rioFreeConn(&rdb, NULL);

            /* Remove the half-loaded data in case we started with
             * an empty replica. */
            emptyDb(-1,empty_db_flags,replicationEmptyDbCallback);

            if (server.repl_diskless_load == REPL_DISKLESS_LOAD_SWAPDB) {
                /* Restore the backed up databases. */
                disklessLoadRestoreBackup(diskless_load_backup);
            }

            /* Note that there's no point in restarting the AOF on SYNC
             * failure, it'll be restarted when sync succeeds or the replica
             * gets promoted. */
            return;
        }

        /* RDB loading succeeded if we reach this point. */
        if (server.repl_diskless_load == REPL_DISKLESS_LOAD_SWAPDB) {
            /* Delete the backup databases we created before starting to load
             * the new RDB. Now the RDB was loaded with success so the old
             * data is useless. */
            disklessLoadDiscardBackup(diskless_load_backup, empty_db_flags);
        }

        /* Verify the end mark is correct. */
        if (usemark) {
            if (!rioRead(&rdb,buf,CONFIG_RUN_ID_SIZE) ||
                memcmp(buf,eofmark,CONFIG_RUN_ID_SIZE) != 0)
            {
                stopLoading(0);
                serverLog(LL_WARNING,"Replication stream EOF marker is broken");
                cancelReplicationHandshake(1);
                rioFreeConn(&rdb, NULL);
                return;
            }
        }

        stopLoading(1);

        /* Cleanup and restore the socket to the original state to continue
         * with the normal replication. */
        rioFreeConn(&rdb, NULL);
        connNonBlock(conn);
        connRecvTimeout(conn,0);
    } else {
        /* Ensure background save doesn't overwrite synced data */
        if (server.child_type == CHILD_TYPE_RDB) {
            serverLog(LL_NOTICE,
                "Replica is about to load the RDB file received from the "
                "master, but there is a pending RDB child running. "
                "Killing process %ld and removing its temp file to avoid "
                "any race",
                (long) server.child_pid);
            killRDBChild();
        }

        /* Make sure the new file (also used for persistence) is fully synced
         * (not covered by earlier calls to rdb_fsync_range). */
        if (fsync(server.repl_transfer_fd) == -1) {
            serverLog(LL_WARNING,
                "Failed trying to sync the temp DB to disk in "
                "MASTER <-> REPLICA synchronization: %s",
                strerror(errno));
            cancelReplicationHandshake(1);
            return;
        }

        /* Rename rdb like renaming rewrite aof asynchronously. */
        int old_rdb_fd = open(server.rdb_filename,O_RDONLY|O_NONBLOCK);
        if (rename(server.repl_transfer_tmpfile,server.rdb_filename) == -1) {
            serverLog(LL_WARNING,
                "Failed trying to rename the temp DB into %s in "
                "MASTER <-> REPLICA synchronization: %s",
                server.rdb_filename, strerror(errno));
            cancelReplicationHandshake(1);
            if (old_rdb_fd != -1) close(old_rdb_fd);
            return;
        }
        /* Close old rdb asynchronously. */
        if (old_rdb_fd != -1) bioCreateCloseJob(old_rdb_fd);

        if (rdbLoad(server.rdb_filename,&rsi,RDBFLAGS_REPLICATION) != C_OK) {
            serverLog(LL_WARNING,
                "Failed trying to load the MASTER synchronization "
                "DB from disk");
            cancelReplicationHandshake(1);
            if (server.rdb_del_sync_files && allPersistenceDisabled()) {
                serverLog(LL_NOTICE,"Removing the RDB file obtained from "
                                    "the master. This replica has persistence "
                                    "disabled");
                bg_unlink(server.rdb_filename);
            }
            /* Note that there's no point in restarting the AOF on sync failure,
               it'll be restarted when sync succeeds or replica promoted. */
            return;
        }

        /* Cleanup. */
        if (server.rdb_del_sync_files && allPersistenceDisabled()) {
            serverLog(LL_NOTICE,"Removing the RDB file obtained from "
                                "the master. This replica has persistence "
                                "disabled");
            bg_unlink(server.rdb_filename);
        }

        zfree(server.repl_transfer_tmpfile);
        close(server.repl_transfer_fd);
        server.repl_transfer_fd = -1;
        server.repl_transfer_tmpfile = NULL;
    }

	// 全量同步完成，在进入复制阶段之前，完成的操作
    // 创建 server.master 客户端，并为主从连接注册 read 事件回调函数 readQueryFromClient ，负责接收并执行主节点传播的命令
    replicationCreateMasterClient(server.repl_transfer_s,rsi.repl_stream_db);
    server.repl_state = REPL_STATE_CONNECTED;
    server.repl_down_since = 0;

    /* Fire the master link modules event. */
    moduleFireServerEvent(REDISMODULE_EVENT_MASTER_LINK_CHANGE,
                          REDISMODULE_SUBEVENT_MASTER_LINK_UP,
                          NULL);

    /* After a full resynchronization we use the replication ID and
     * offset of the master. The secondary ID / offset are cleared since
     * we are starting a new history. */
    memcpy(server.replid,server.master->replid,sizeof(server.replid));
    server.master_repl_offset = server.master->reploff;
    clearReplicationId2();

    /* Let's create the replication backlog if needed. Slaves need to
     * accumulate the backlog regardless of the fact they have sub-slaves
     * or not, in order to behave correctly if they are promoted to
     * masters after a failover. */
    if (server.repl_backlog == NULL) createReplicationBacklog();
    serverLog(LL_NOTICE, "MASTER <-> REPLICA sync: Finished with success");

    if (server.supervised_mode == SUPERVISED_SYSTEMD) {
        redisCommunicateSystemd("STATUS=MASTER <-> REPLICA sync: Finished with success. Ready to accept connections in read-write mode.\n");
    }

    /* Send the initial ACK immediately to put this replica in online state. */
    if (usemark) replicationSendAck();

    /* Restart the AOF subsystem now that we finished the sync. This
     * will trigger an AOF rewrite, and when done will start appending
     * to the new file. */
    if (server.aof_enabled) restartAOFAfterSYNC();
    return;

error:
    cancelReplicationHandshake(1);
    return;
}
```



#### 主节点同步

​		`syncCommand`负责处理`psync`和`sync`命令：

```c++
// replication.c
void syncCommand(client *c) {
	...
    // 处理 psync 
    if (!strcasecmp(c->argv[0]->ptr,"psync")) {
        // 进行部分同步操作
        if (masterTryPartialResynchronization(c) == C_OK) {
            server.stat_sync_partial_ok++;
            return; /* No full resync needed, return. */
        } else {
            char *master_replid = c->argv[1]->ptr;

            /* Increment stats for failed PSYNCs, but only if the
             * replid is not "?", as this is used by slaves to force a full
             * resync on purpose when they are not albe to partially
             * resync. */
            if (master_replid[0] != '?') server.stat_sync_partial_err++;
        }
    } else {
        /* If a slave uses SYNC, we are dealing with an old implementation
         * of the replication protocol (like redis-cli --slave). Flag the client
         * so that we don't expect to receive REPLCONF ACK feedbacks. */
        c->flags |= CLIENT_PRE_PSYNC;
    }

    /* Full resynchronization. */
    // 无法使用部分同步机制，需要进行全量同步
    server.stat_sync_full++;

    /* Setup the slave as one waiting for BGSAVE to start. The following code
     * paths will change the state if we handle the slave differently. */
    c->replstate = SLAVE_STATE_WAIT_BGSAVE_START;
    if (server.repl_disable_tcp_nodelay)
        connDisableTcpNoDelay(c->conn); /* Non critical if it fails. */
    // 将该客户端添加到 server.slaves
    c->repldbfd = -1;
    c->flags |= CLIENT_SLAVE;
    listAddNodeTail(server.slaves,c);

    /* Create the replication backlog if needed. */
    // 主节点复制积压区未创建，则创建复制积压区
    if (listLength(server.slaves) == 1 && server.repl_backlog == NULL) {
        /* When we create the backlog from scratch, we always use a new
         * replication ID and clear the ID2, since there is no valid
         * past history. */
        changeReplicationId();
        clearReplicationId2();
        createReplicationBacklog();
        serverLog(LL_NOTICE,"Replication backlog created, my new "
                            "replication IDs are '%s' and '%s'",
                            server.replid, server.replid2);
    }

    /* CASE 1: BGSAVE is in progress, with disk target. */
    // 此时主节点需要生成一个 RDB 文件，并发送到从节点
    
    /*
    	主节点正在生成 RDB 数据，并保存在磁盘中。如果当前从节点列表中有其他处于 SLAVE_STATE_WAIT_BGSAVE_END 状态的从节点客户端，则说明当前生成的 RDB 文件也可以被当前从节点客户端使用，此时将当前从节点客户端 client.replstate 设置为 SLAVE_STATE_WAIT_BGSAVE_END 状态，直接使用当前生成的 RDB 文件
    */
    if (server.child_type == CHILD_TYPE_RDB &&
        server.rdb_child_type == RDB_CHILD_TYPE_DISK)
    {
        /* Ok a background save is in progress. Let's check if it is a good
         * one for replication, i.e. if there is another slave that is
         * registering differences since the server forked to save. */
        client *slave;
        listNode *ln;
        listIter li;

        listRewind(server.slaves,&li);
        while((ln = listNext(&li))) {
            slave = ln->value;
            /* If the client needs a buffer of commands, we can't use
             * a replica without replication buffer. */
            if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_END &&
                (!(slave->flags & CLIENT_REPL_RDBONLY) ||
                 (c->flags & CLIENT_REPL_RDBONLY)))
                break;
        }
        /* To attach this slave, we check that it has at least all the
         * capabilities of the slave that triggered the current BGSAVE. */
        if (ln && ((c->slave_capa & slave->slave_capa) == slave->slave_capa)) {
            /* Perfect, the server is already registering differences for
             * another slave. Set the right state, and copy the buffer.
             * We don't copy buffer if clients don't want. */
            if (!(c->flags & CLIENT_REPL_RDBONLY)) copyClientOutputBuffer(c,slave);
            replicationSetupSlaveForFullResync(c,slave->psync_initial_offset);
            serverLog(LL_NOTICE,"Waiting for end of BGSAVE for SYNC");
        } else {
            /* No way, we need to wait for the next BGSAVE in order to
             * register differences. */
            serverLog(LL_NOTICE,"Can't attach the replica to the current BGSAVE. Waiting for next BGSAVE for SYNC");
        }

    /* CASE 2: BGSAVE is in progress, with socket target. */
    // 主节点正在生成 RDB 文件，并直接发送到 Socket ，此时需要的等待该 RDB 操作完成后，在生成一个新的 RDB 文件
    } else if (server.child_type == CHILD_TYPE_RDB &&
               server.rdb_child_type == RDB_CHILD_TYPE_SOCKET)
    {
        /* There is an RDB child process but it is writing directly to
         * children sockets. We need to wait for the next BGSAVE
         * in order to synchronize. */
        serverLog(LL_NOTICE,"Current BGSAVE has socket target. Waiting for next BGSAVE for SYNC");

    /* CASE 3: There is no BGSAVE is progress. */
    // 当前程序中不存在子进程
    } else {
        if (server.repl_diskless_sync && (c->slave_capa & SLAVE_CAPA_EOF) &&
            server.repl_diskless_sync_delay)
        {
            /* Diskless replication RDB child is created inside
             * replicationCron() since we want to delay its start a
             * few seconds to wait for more slaves to arrive. */
            serverLog(LL_NOTICE,"Delay next BGSAVE for diskless SYNC");
        } else {
            /* We don't have a BGSAVE in progress, let's start one. Diskless
             * or disk-based mode is determined by replica's capacity. */
            if (!hasActiveChildProcess()) {
                // 生成 RDB 文件
                startBgsaveForReplication(c->slave_capa);
            } else {
                serverLog(LL_NOTICE,
                    "No BGSAVE in progress, but another BG operation is active. "
                    "BGSAVE for replication delayed");
            }
        }
    }
    return;
}
```



##### 全量同步

​		`startBgsaveForReplication`负责生成`RDB`文件并发送给从节点：

```c++
// replication.c
int startBgsaveForReplication(int mincapa) {
    int retval;
    int socket_target = server.repl_diskless_sync && (mincapa & SLAVE_CAPA_EOF);
    listIter li;
    listNode *ln;

    serverLog(LL_NOTICE,"Starting BGSAVE for SYNC with target: %s",
        socket_target ? "replicas sockets" : "disk");

    rdbSaveInfo rsi, *rsiptr;
    rsiptr = rdbPopulateSaveInfo(&rsi);
    
    // 开启了无磁盘同步功能
    if (rsiptr) {
        if (socket_target)
            // 生成 RDB 数据并直接发送到 Socket 中
            retval = rdbSaveToSlavesSockets(rsiptr);
        else
            // 生成 RDB 文件
            retval = rdbSaveBackground(server.rdb_filename,rsiptr);
    } else {
        serverLog(LL_WARNING,"BGSAVE for replication: replication information not available, can't generate the RDB file right now. Try later.");
        retval = C_ERR;
    }
    
    if (retval == C_OK && !socket_target && server.rdb_del_sync_files)
        RDBGeneratedByReplication = 1;

    
    if (retval == C_ERR) {
        serverLog(LL_WARNING,"BGSAVE for replication failed");
        listRewind(server.slaves,&li);
        while((ln = listNext(&li))) {
            client *slave = ln->value;

            if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_START) {
                slave->replstate = REPL_STATE_NONE;
                slave->flags &= ~CLIENT_SLAVE;
                listDelNode(server.slaves,ln);
                addReplyError(slave,
                    "BGSAVE failed, replication can't continue");
                slave->flags |= CLIENT_CLOSE_AFTER_REPLY;
            }
        }
        return retval;
    }

    // 未开启无磁盘同步功能
    if (!socket_target) {
        listRewind(server.slaves,&li);
        // 遍历所有从节点
        while((ln = listNext(&li))) {
            client *slave = ln->value;

            if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_START) {
                // 处理所有 SLAVE_STATE_WAIT_BGSAVE_START 状态的从节点客户端，这些客户端都可以使用本次生成的 RDB 文件进行全量同步
                    replicationSetupSlaveForFullResync(slave,
                            getPsyncInitialOffset());
            }
        }
    }

    if (retval == C_OK) replicationScriptCacheFlush();
    return retval;
}
```

​		在生成`RDB`文件之后，再由`updateSlavesWaitingBgsave`发送`RDB`数据：

```c++
// replication.c
void updateSlavesWaitingBgsave(int bgsaveerr, int type) {
    listNode *ln;
    listIter li;

    listRewind(server.slaves,&li);
    while((ln = listNext(&li))) {
        client *slave = ln->value;

        if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_END) {
            struct redis_stat buf;

            if (bgsaveerr != C_OK) {
                freeClient(slave);
                serverLog(LL_WARNING,"SYNC failed. BGSAVE child returned an error");
                continue;
            }
			// 使用的是无磁盘同步功能
            if (type == RDB_CHILD_TYPE_SOCKET) {
                serverLog(LL_NOTICE,
                    "Streamed RDB transfer with replica %s succeeded (socket). Waiting for REPLCONF ACK from slave to enable streaming",
                        replicationGetSlaveName(slave));
                
                slave->replstate = SLAVE_STATE_ONLINE;
                slave->repl_put_online_on_ack = 1;
                slave->repl_ack_time = server.unixtime; /* Timeout otherwise. */
            } else {
                // 磁盘同步功能
                if ((slave->repldbfd = open(server.rdb_filename,O_RDONLY)) == -1 ||
                    redis_fstat(slave->repldbfd,&buf) == -1) {
                    freeClient(slave);
                    serverLog(LL_WARNING,"SYNC failed. Can't open/stat DB after BGSAVE: %s", strerror(errno));
                    continue;
                }
                slave->repldboff = 0;
                slave->repldbsize = buf.st_size;
                // 从节点客户端 client.replstate 进入 SLAVE_STATE_SEND_BULK 状态
                slave->replstate = SLAVE_STATE_SEND_BULK;
                slave->replpreamble = sdscatprintf(sdsempty(),"$%lld\r\n",
                    (unsigned long long) slave->repldbsize);

                connSetWriteHandler(slave->conn,NULL);
                // 为主从连接注册 write 事件的回调函数 sendBulkToSlave
                if (connSetWriteHandler(slave->conn,sendBulkToSlave) == C_ERR) {
                    freeClient(slave);
                    continue;
                }
            }
        }
    }
}

void sendBulkToSlave(connection *conn) {
    client *slave = connGetPrivateData(conn);
    char buf[PROTO_IOBUF_LEN];
    ssize_t nwritten, buflen;
	// 使用磁盘同步功能，发送 RDB 数据使用的是定长格式，先发送数据长度标志
    if (slave->replpreamble) {
        nwritten = connWrite(conn,slave->replpreamble,sdslen(slave->replpreamble));
        if (nwritten == -1) {
            serverLog(LL_VERBOSE,
                "Write error sending RDB preamble to replica: %s",
                connGetLastError(conn));
            freeClient(slave);
            return;
        }
        atomicIncr(server.stat_net_output_bytes, nwritten);
        sdsrange(slave->replpreamble,nwritten,-1);
        if (sdslen(slave->replpreamble) == 0) {
            sdsfree(slave->replpreamble);
            slave->replpreamble = NULL;
        } else {
            return;
        }
    }
	// 发送 RDB 内容
    lseek(slave->repldbfd,slave->repldboff,SEEK_SET);
    buflen = read(slave->repldbfd,buf,PROTO_IOBUF_LEN);
    if (buflen <= 0) {
        serverLog(LL_WARNING,"Read error sending DB to replica: %s",
            (buflen == 0) ? "premature EOF" : strerror(errno));
        freeClient(slave);
        return;
    }
    if ((nwritten = connWrite(conn,buf,buflen)) == -1) {
        if (connGetState(conn) != CONN_STATE_CONNECTED) {
            serverLog(LL_WARNING,"Write error sending DB to replica: %s",
                connGetLastError(conn));
            freeClient(slave);
        }
        return;
    }
    slave->repldboff += nwritten;
    atomicIncr(server.stat_net_output_bytes, nwritten);
    // 发送完成后，删除主从连接的 wirte 事件的回调函数
    if (slave->repldboff == slave->repldbsize) {
        close(slave->repldbfd);
        slave->repldbfd = -1;
        connSetWriteHandler(slave->conn,NULL);
        /**
        	设置从节点客户端 client.replstate 进入 SLAVE_STATE_ONLINE 状态
        	为主从连接注册 write 事件的回调函数 sendReplyToClient ，将该从节点客户端恢复缓冲区的内容发送给从节点
        */
        putSlaveOnline(slave);
    }
}
```



##### 部分同步

​		为了支持部分同步，主节点定义了复制积压区。`Redis`每执行一条写命令，都会调用`feedReplicationBacklog`函数将命令写入复制积压区。部分同步只需要

将复制积压区中待同步的命令发送给从节点即可。

​		主节点收到`psync`命令后，调用`masterTryPartialResynchronization`尝试进行部分同步：

```c++
// replication.c
int masterTryPartialResynchronization(client *c) {
    long long psync_offset, psync_len;
    char *master_replid = c->argv[1]->ptr;
    char buf[128];
    int buflen;
	// 读取 psync replid offset 命令中的 offset 参数
    if (getLongLongFromObjectOrReply(c,c->argv[2],&psync_offset,NULL) !=
       C_OK) goto need_full_resync;
	// replid 参数需要等于 server.replid 或 server.replid2，否则只能使用全量同步机制，主节点存放了 server.replid 和 server.replid2 用于实现 psync2 协议
    if (strcasecmp(master_replid, server.replid) &&
        (strcasecmp(master_replid, server.replid2) ||
         psync_offset > server.second_replid_offset))
    {
        if (master_replid[0] != '?') {
            if (strcasecmp(master_replid, server.replid) &&
                strcasecmp(master_replid, server.replid2))
            {
                serverLog(LL_NOTICE,"Partial resynchronization not accepted: "
                    "Replication ID mismatch (Replica asked for '%s', my "
                    "replication IDs are '%s' and '%s')",
                    master_replid, server.replid, server.replid2);
            } else {
                serverLog(LL_NOTICE,"Partial resynchronization not accepted: "
                    "Requested offset for second ID was %lld, but I can reply "
                    "up to %lld", psync_offset, server.second_replid_offset);
            }
        } else {
            serverLog(LL_NOTICE,"Full resync requested by replica %s",
                replicationGetSlaveName(c));
        }
        goto need_full_resync;
    }

    // 检查 offset 偏移量是否在复制积压区范围内。如果 offset 偏移量不在复制积压区范围内，则说明从节点复制进度已经落后太多，此时只能使用全量同步机制
    if (!server.repl_backlog ||
        psync_offset < server.repl_backlog_off ||
        psync_offset > (server.repl_backlog_off + server.repl_backlog_histlen))
    {
        serverLog(LL_NOTICE,
            "Unable to partial resync with replica %s for lack of backlog (Replica request was: %lld).", replicationGetSlaveName(c), psync_offset);
        if (psync_offset > server.master_repl_offset) {
            serverLog(LL_WARNING,
                "Warning: replica %s tried to PSYNC with an offset that is greater than the master replication offset.", replicationGetSlaveName(c));
        }
        goto need_full_resync;
    }
	// 客户端添加 CLIENT_SLAVE 标志，并且设置 client.replstate 为 SLAVE_STATE_ONLINE 状态，代表该客户端已进入在线状态，最后将该客户端添加到 server.slaves 中
    c->flags |= CLIENT_SLAVE;
    c->replstate = SLAVE_STATE_ONLINE;
    c->repl_ack_time = server.unixtime;
    c->repl_put_online_on_ack = 0;
    listAddNodeTail(server.slaves,c);
	// 发送 +CONTINUE 响应，告诉从节点可以进行部分同步。需要发送最新的 replid 给从节点，同样是为了实现 psync2 协议
    if (c->slave_capa & SLAVE_CAPA_PSYNC2) {
        buflen = snprintf(buf,sizeof(buf),"+CONTINUE %s\r\n", server.replid);
    } else {
        buflen = snprintf(buf,sizeof(buf),"+CONTINUE\r\n");
    }
    if (connWrite(c->conn,buf,buflen) != buflen) {
        freeClientAsync(c);
        return C_OK;
    }
    // 将复制积压区的内容复制给客户端回复缓冲区
    psync_len = addReplyReplicationBacklog(c,psync_offset);
    serverLog(LL_NOTICE,
        "Partial resynchronization request from %s accepted. Sending %lld bytes of backlog starting from offset %lld.",
            replicationGetSlaveName(c),
            psync_len, psync_offset);

    refreshGoodSlavesCount();

    moduleFireServerEvent(REDISMODULE_EVENT_REPLICA_CHANGE,
                          REDISMODULE_SUBEVENT_REPLICA_CHANGE_ONLINE,
                          NULL);

    return C_OK; /* The caller can return, no full resync needed. */

need_full_resync:
    
    return C_ERR;
}
```



### Raft 算法

​		`Redis Sentinel`和`Cluster`机制中都用到了一致性算法`Raft`。一致性算法即在分布式系统中，保证数据一致性的算法。

​		分布式系统通常由多个节点组成，集群中的每个节点都需要将自己接收的修改请求广播给其他节点，其他节点接收后在执行相同的操作，以保持集群内所有节

点的数据一致。保证数据一致的问题：

​				时钟不同步：由于网络延迟等原因，节点接收的多个修改请求的前后顺序不一致。在一个节点上，可以通过时间`(`或序列号`)`区分操作的先后顺序，而

​		在多个节点上，由于不同机器上的物理时钟难以同步，因此在分布式系统中无法直接使用物理时钟区分事件时序。

​						中心化：选举一个`leader`节点，并由`leader`节点负责所有的写操作。

​						去中心化：集群中所有节点都可以处理写操作，但每个写操作执行前都需要集群内对该操作达成共识。

​				网络不可靠：由于网络不可靠，某些节点可能由于网络阻塞等原因，没有收到其他节点广播的修改请求，导致部分修改丢失，最终导致数据不一致。

​				节点崩溃：集群中的每个节点随时可能崩溃重启或者运行缓慢，导致它在某一段时间内没有收到其他节点广播的修改请求，最终导致数据不一致。



#### CAP理论

​		`P`：分区容错性：正常情况下一个集群内的所有节点网络应该是互通的。但由于网络不可靠，可能导致一些节点之间网络不通，整个网络就分成了几块区

域。如果一个数据只在一个节点中保存，出现网络分区后，其他不连通的分区就访问不到这个数据。通常将数据复制到集群的所有节点，保证即使出现网络分区

后，不同网络分区都可以访问到数据。

​		`C`：一致性：集群中所有节点的数据保持一致。

​		`A`：可用性：集群一致处于可用状态，即正常处理用户请求并返回响应。



​		由于网络不可靠，并且分布式存储系统都使用多个节点存储数据，所以分布式系统必须实现分区容错性。如果追求可用性，当数据不一致时分布式系统扔提供

服务，但可能返回不一致的数据`(`可能某一段时间内集群数据不一致，但集群内的数据最终会达成一致，即最终一致性`)`。而追求一致性，当集群数据无法达成一

致时，集群就应该停止服务。

​		`Raft`算法实现的是强一致性，即优先追求数据一致，当集群无法达到数据一致时将停止服务，但`Raft`使用了`Quorum`机制，只要集群中超过半数节点正常运

行，集群就可以正常提供服务。并且当集群中超过半数节点接收某个提议后，集群内就达成共识：该操作可以被集群执行。

​		`Raft`算法会选举一个`leader`节点`(`其他节点称为`follower`节点`)`，由`leader`节点处理所有写请求，然后将自己收到的请求广播给集群内其他`follower`节

点，从而达成数据一致。通过日志存储所有的修改操作，通过保证已提交的日志安全，即保证了对应的修改操作不能被抛弃或篡改。

​		在`Raft`算法中，每个节点都维护了以下属性：

|    属性     |                             说明                             |
| :---------: | :----------------------------------------------------------: |
| currentTerm |                        服务器当前任期                        |
|  votedFor   |     当前任期获得该节点投票的 candidate （候选）节点的 ID     |
|    log[]    | 本地日志集，每个条目包含日志索引 ，操作内容，以及 leader 收到该日志时的任期 |
| commitIndex |                  已提交的最大的日志条目索引                  |
| lastApplied |              已执行修改操作的最大的日志条目索引              |

​		`Raft`算法中定义了任期的概念，将时间切分为一个个任期，可以认为是逻辑上的时间。每个任期开始时都需要选举`leader`，该`leader`将在该任期内一致完

成`leader`工作，如果`leader`崩溃下线，则需要开启一个新的任期，并选举新的`leader`。

​		每个节点都可以有`3`种状态：`leadeer`、`follower`、`candidate`。`leader`节点会定时向所有`follower`节点发送心跳请求，以维护`leader`地位。如果某个

`leader`节点超过指定时间没有收到`leader`心跳`(`该超时时间称为选举超时时间`)`，那么它就认为`leader`节点已下线，将转化为`candidate(`候选`)`节点，并发

起选举流程。

​		集群刚启动后，每个节点都是`follower`节点，直到某个节点转化为`candidate`节点，将发起选举流程：

​				增加任期，更新`currentTerm`为`currentTerm + 1`。

​				给自己投票。

​				向集群其他节点发送投票报文，要求它们给自己投票。报文内容：

|     属性     |               说明               |
| :----------: | :------------------------------: |
|     term     |        候选人的当前任期号        |
| candidateId  |           发送节点 ID            |
| lastLogIndex | 发送节点最后一条日志条目的索引值 |
| lastLogTerm  | 发送节点最后一条日志条目的任期号 |

​		其他节点收到该报文后：

​				1、如果`request.term < receiver.currentTerm`，则拒绝投票。

​				2、如果`request.lastLogTerm < receiver.lastLogterm`或`request.lastLogIndex < receiver.lastLogindex`，则拒绝投票。

​				3、如果`request.term == receiver.currentTerm`，并且接收节点`votedFor`不为空，则说明该任期内接收节点已经给其他节点投过票，拒绝投票。

​				4、到这里，说明可以给请求节点投票，更新`receiver.currentTerm`为`request.term`，重置选举超时时间，并返回投票响应。



​		`candidate`节点转换为其他状态：

​				1、超过半数节点给自己投票，当前节点当选为`leader`节点。

​				2、收到其他节点的心跳报文，并且任期不小于自己当前任期，说明其他节点已成为`leader`节点，当前节点转化为`follower`节点。

​				3、等待一段时间，直到超过选举超时时间仍没有当选或没有收到其他`leader`节点的心跳报文，则开始新一轮选举。

​		如果一个任期内同时有多个节点发起选举，则该任期的选票可能被多个节点瓜分，导致该任期最终没有任何一个节点能成为`leader`节点，投票失败，为了避

免这种情况，`Raft`算法要求每个节点都在一个固定的区间内选择一个随机时间作为选举超时时间，当`leader`下线后，最先超时的节点会先发起选举，通常可以获

得多数选票，然后该节点赢得选择并在其他节点选举超时之前发送心跳包，从而成为`leader`节点。



​		心跳报文也称为附加日志报文，由`leader`节点发送，负责维持`leader`节点地位或发送日志条目：

|     属性     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|     term     |                      leader 节点的任期                       |
|   leaderId   |   leader 节点的 ID，用于 follower 节点通知客户端进行重定向   |
| prevLogIndex |                新日志条目前一条日志条目的索引                |
| prevlogTerm  |                新日志条目前一条日志条目的任期                |
|  entries[]   | 需要被 follower 节点保存的新日志条目（为了提高效率，可能一次性发送多条，心跳报文则为空） |
| leaderCommit |          leader 节点中已提交的最大的日志条目的索引           |

​		`follower`节点收到心跳报文后：

​				1、`request.term < local.currentTerm`，无效报文，拒绝处理。

​				2、执行日志一致性检查操作，如果不通过，则拒绝接收请求中的新日志。

​				3、添加请求中的新日志条目到本地日志集并返回结果，如果本地已经存在的日志条目和请求中的新日志条目冲突`(`索引值相同但是任期号不同`)`，则删

​		除本地日志集中该索引及后续的所有日志条目，将心跳报文中的新日志条目添加到本地日志集中。

​		`Raft`中通过`commited`索引维护状态机已提交的日志索引。当`leader`节点收到多数节点接收某个日志的成功响应后，将修改`commited`索引。此时`leader`节

点才真正执行修改数据的操作并修改`lastApplied`索引。随后，`leader`节点通过心跳报文将最新的`commited`索引发送给其他`follower`节点。

​		`follower`节点根据最新的`commited`索引执行操作：

​				1、如果`request.leaderCommited > receiver.commitIndex`，则令`receiver.commitIndex`等于`request.leaderCommit`和新日志条目索引中较小的一个

​		值。

​				2、如果接收节点中`commitIndex > lastApplied`，则使`lastApplied`加`1`，并执行`lastApplied`位置的日志操作，直到`lastApplied == commitIndex`。



​		在投票时，只有当请求投票节点的日志条目至少和接收节点一样新时，接收节点才给请求节点投票，否则拒绝为该节点投票。由于一个节点当选`leader`节点

要求得到多数节点的投票，所以最终选出来的节点的日志条目必然比多数节点更新，也必然包含已提交的日志。

​		新上任的`leader`节点不能直接抛弃前一任期中未提交的日志，而是要继续处理这些日志。但对于前一任期的日志要做一个特殊的处理：当前一任期的日志被

多数节点接收后，并不能提交该日志，而是要等到当前任期的日志被多数节点接收后才能提交日志。



​		`Redis`利用了`Raft`算法的领导选举机制，在`Redis Sentinel`和`Cluster`机制中，如果某个主节点崩溃下线，那么`Redis`将利用`Raft`算法选举一个`leader`节

点，并由`leader`节点完成故障转移。但`Redis`使用的是异步复制机制，从而保证低延迟和高性能。所以`Redis`实现的是数据最终一致性，即在复制过程中，从节

点数据可能会落后于主节点。



### Sentinel

​		每个`Sentinel`节点都维护一份自己视角下的当前`Sentinel`集群的状态，该状态信息存储在`sentinelState`结构体中：

```c++
// sentinel.c
struct sentinelState {
    char myid[CONFIG_RUN_ID_SIZE+1]; // 标志 ID，用于区分不同的 Sentinel 节点
    uint64_t current_epoch; // Sentinel 集群当前的任期号，用于故障转移时使用 Raft 算法选举 leader 节点
    dict *masters; // 监控的主节点字典，记录当前 sentinel 节点监控的所有主节点，键是主从集群名称，值指向 sentinelRedisInstance 变量
    int tilt; // 是否处于 TILT 模式
    int running_scripts;    /* Number of scripts in execution right now. */
    mstime_t tilt_start_time;       /* When TITL started. */
    mstime_t previous_time;         /* Last time we ran the time handler. */
    list *scripts_queue;            /* Queue of user scripts to execute. */
    char *announce_ip;  /* IP addr that is gossiped to other sentinels if
                           not NULL. */
    int announce_port;  /* Port that is gossiped to other sentinels if
                           non zero. */
    unsigned long simfailure_flags; /* Failures simulation. */
    int deny_scripts_reconfig; /* Allow SENTINEL SET ... to change script
                                  paths at runtime? */
    char *sentinel_auth_pass;    /* Password to use for AUTH against other sentinel */
    char *sentinel_auth_user;    /* Username for ACLs AUTH against other sentinel. */
    int resolve_hostnames;       /* Support use of hostnames, assuming DNS is well configured. */
    int announce_hostnames;      /* Announce hostnames instead of IPs when we have them. */
} sentinel;

// 负责存储 Sentinel 集群中主从节点，以及其他 Sentinel 节点的实例数据
typedef struct sentinelRedisInstance {
    /**
    	节点标志，存储该节点的状态，属性等信息：
    		SRI_MASTER：该节点是主节点
    		SRI_SLAVE：该节点是从节点
    		SRI_SENTINEL：该节点是 Sentinel 节点
    		SRI_S_DOWN：该节点已主观下线
    		SRI_O_DOWN：该节点已客观下线
    		SRI_FAILOVER_IN_PROGRESS：该主节点正进行故障迁移
    		SRI_PROMOTED：该节点被选为故障迁移中的晋升节点
    */
    int flags;
    char *name; // 节点名称，也是 sentinels 字典或 slaves 字典的键，在主节点中它是主从集群名称，在从节点它是 IP:PORT，在 Sentinel 节点中是 myid属性
    char *runid; // 作为节点唯一标识的随机字符串，Sentinel 节点使用 myid 属性，主从节点从 INFO 响应中获取 runid 属性作为该值，每个节点都会在启动时初始化 redisSerer.runid 属性作为节点标识，并在 INFO 响应中返回该属性
    uint64_t config_epoch;  /* Configuration epoch. */
    sentinelAddr *addr; // 节点网络地址信息，包括 IP 地址与端口信息
    instanceLink *link; // instanceLink 结构体，存储该节点与当前 Sentinel 节点的连接信息
    mstime_t last_pub_time;   /* Last time we sent hello via Pub/Sub. */
    mstime_t last_hello_time; /* Only used if SRI_SENTINEL is set. Last time
                                 we received a hello from this Sentinel
                                 via Pub/Sub. */
    mstime_t last_master_down_reply_time; /* Time of last reply to
                                             SENTINEL is-master-down command. */
    mstime_t s_down_since_time; /* Subjectively down since time. */
    mstime_t o_down_since_time; /* Objectively down since time. */
    mstime_t down_after_period; // 判断节点是否已主观下线
    mstime_t info_refresh;  /* Time at which we received INFO output from it. */
    dict *renamed_commands;     /* Commands renamed in this instance:
                                   Sentinel will use the alternative commands
                                   mapped on this table to send things like
                                   SLAVEOF, CONFING, INFO, ... */

    /* Role and the first time we observed it.
     * This is useful in order to delay replacing what the instance reports
     * with our own configuration. We need to always wait some time in order
     * to give a chance to the leader to report the new configuration before
     * we do silly things. */
    int role_reported;
    mstime_t role_reported_time;
    mstime_t slave_conf_change_time; /* Last time slave master addr changed. */

    /* Master specific. */
    dict *sentinels; // 主节点专用字段，sentinel 字典，存放集群中其他 Sentinel 节点实例，但不包含当前 Sentinel 节点实例
    dict *slaves; //  主节点专用字段，slaves 字段，存放该主节点下所有的从节点
    unsigned int quorum; // 法定节点数，Sentinel 集群中必须存在不小于该数量的 Sentinel 节点接收某个提议，Sentinel 集群才能达成共识
    int parallel_syncs; // 用于指定故障迁移过程中最多有多少个从节点同时与晋升节点建立主从关系
    char *auth_pass;    /* Password to use for AUTH against master & replica. */
    char *auth_user;    /* Username for ACLs AUTH against master & replica. */

    /* Slave specific. */
    mstime_t master_link_down_time; /* Slave replication link down time. */
    int slave_priority; // 从节点专用字段，从节点优先级，用于故障迁移时选择从节点
    int replica_announced; /* Replica announcing according to its INFO output. */
    mstime_t slave_reconf_sent_time; /* Time at which we sent SLAVE OF <new> */
    struct sentinelRedisInstance *master; /* Master instance if it's slave. */
    char *slave_master_host;    /* Master host as reported by INFO */
    int slave_master_port;      /* Master port as reported by INFO */
    int slave_master_link_status; // 主从连接状态，记录 INFO 响应的 master_link_status 属性
    unsigned long long slave_repl_offset; // 主从复制偏移量，记录 INFO 响应的 slave_repl_offset 属性
    /* Failover */
    char *leader; // 故障迁移专用字段，获取该节点最新投票的节点的 myid 属性
    uint64_t leader_epoch; // 故障迁移专用字段，获取该节点最新投票的节点的任期号
    uint64_t failover_epoch; // 故障迁移专用字段，该主节点最新执行故障迁移的任期
    int failover_state; // 故障迁移状态
    mstime_t failover_state_change_time;
    mstime_t failover_start_time;   /* Last failover attempt start time. */
    mstime_t failover_timeout; // 故障迁移专用字段，指定故障迁移最长时间
    mstime_t failover_delay_logged; /* For what failover_start_time value we
                                       logged the failover delay. */
    struct sentinelRedisInstance *promoted_slave; /* Promoted slave instance. */
    /* Scripts executed to notify admin or reconfigure clients: when they
     * are set to NULL no script is executed. */
    char *notification_script;
    char *client_reconfig_script;
    sds info; /* cached INFO output */
} sentinelRedisInstance;

// 存储该节点与当前 Sentinel 节点的连接信息
typedef struct instanceLink {
    int refcount;          /* Number of sentinelRedisInstance owners. */
    int disconnected;      /* Non-zero if we need to reconnect cc or pc. */
    int pending_commands;  /* Number of commands sent waiting for a reply. */
    redisAsyncContext *cc; // 命令连接，Sentinel 节点需要与监控的主从节点，集群其他 Sentinel 节点建立命令连接，以便给这些节点发送命令
    redisAsyncContext *pc; // 订阅连接，Sentinel 节点只与主从节点建立订阅连接，并从特定频道中接收其他 Sentinel 节点发送的数据
    mstime_t cc_conn_time; /* cc connection time. */
    mstime_t pc_conn_time; /* pc connection time. */
    mstime_t pc_last_activity; // 上次接收到该节点频道数据的时间
    mstime_t last_avail_time; // 上次收到该节点 PING 命令响应的时间
    mstime_t act_ping_time; // 上次给该节点发送 PING 命令响应的时间，当收到响应时会重置为 0
    mstime_t last_ping_time; // 上次发送该节点 PING 命令的时间
    mstime_t last_pong_time;  /* Last time the instance replied to ping,
                                 whatever the reply was. That's used to check
                                 if the link is idle and must be reconnected. */
    mstime_t last_reconn_time;  /* Last reconnection attempt performed when
                                   the link was down. */
} instanceLink;
```



#### Sentinel 节点启动

​		`Redis`以`Sentinel`模式启动，则`main`函数会调用`initSentinelConfig`、`initSentinel`、`sentinelIsRunning`初始化并启动`Sentinel`机制：

```c++
// sentinel.c
void initSentinelConfig(void) {
    // 修改 TCP 端口，默认为 26379
    server.port = REDIS_SENTINEL_PORT;
    server.protected_mode = 0; // 关闭保护模式
}

void initSentinel(void) {
    unsigned int j;

    /* Remove usual Redis commands from the command table, then just add
     * the SENTINEL command. */
    dictEmpty(server.commands,NULL);
    dictEmpty(server.orig_commands,NULL);
    // 清空命令字典 server.commands
    ACLClearCommandID();
    // 加载 Sentinel 节点支持的特定命令
    for (j = 0; j < sizeof(sentinelcmds)/sizeof(sentinelcmds[0]); j++) {
        int retval;
        struct redisCommand *cmd = sentinelcmds+j;
        cmd->id = ACLGetCommandID(cmd->name); /* Assign the ID used for ACL. */
        retval = dictAdd(server.commands, sdsnew(cmd->name), cmd);
        serverAssert(retval == DICT_OK);
        retval = dictAdd(server.orig_commands, sdsnew(cmd->name), cmd);
        serverAssert(retval == DICT_OK);

        /* Translate the command string flags description into an actual
         * set of flags. */
        if (populateCommandTableParseFlags(cmd,cmd->sflags) == C_ERR)
            serverPanic("Unsupported command flag");
    }

    /* Initialize various data structures. */
    // 初始化全局变量 sentinel
    sentinel.current_epoch = 0;
    sentinel.masters = dictCreate(&instancesDictType,NULL);
    sentinel.tilt = 0;
    sentinel.tilt_start_time = 0;
    sentinel.previous_time = mstime();
    sentinel.running_scripts = 0;
    sentinel.scripts_queue = listCreate();
    sentinel.announce_ip = NULL;
    sentinel.announce_port = 0;
    sentinel.simfailure_flags = SENTINEL_SIMFAILURE_NONE;
    sentinel.deny_scripts_reconfig = SENTINEL_DEFAULT_DENY_SCRIPTS_RECONFIG;
    sentinel.sentinel_auth_pass = NULL;
    sentinel.sentinel_auth_user = NULL;
    sentinel.resolve_hostnames = SENTINEL_DEFAULT_RESOLVE_HOSTNAMES;
    sentinel.announce_hostnames = SENTINEL_DEFAULT_ANNOUNCE_HOSTNAMES;
    memset(sentinel.myid,0,sizeof(sentinel.myid));
    server.sentinel_config = NULL;
}

void sentinelIsRunning(void) {
    int j;

	// 检查应用是否有配置文件的写权限，Sentinel 节点运行过程中，需要将部分运行数据写入配置文件，使得 Sentinel 节点重启后不丢失数据
    for (j = 0; j < CONFIG_RUN_ID_SIZE; j++)
        if (sentinel.myid[j] != 0) break;
	// 如果配置文件没有指定 sentinel.myid ，则初始化 sentinel.myid
    if (j == CONFIG_RUN_ID_SIZE) {
        /* Pick ID and persist the config. */
        getRandomHexChars(sentinel.myid,CONFIG_RUN_ID_SIZE);
        sentinelFlushConfig();
    }

    serverLog(LL_WARNING,"Sentinel ID is %s", sentinel.myid);

    sentinelGenerateInitialMonitorEvents();
}
```



​		`serverCron`时间事件会检查服务器是否运行在`Sentinel`模式下，如果是，则调用`Sentinel`机制的定时逻辑函数`sentinelTimer`：

```c++
// sentinel.c
void sentinelTimer(void) {
    // 检查是否需要进入 TILT 模式
    sentinelCheckTiltCondition();
    // 调用 Sentinel 机制的主逻辑触发函数
    sentinelHandleDictOfRedisInstances(sentinel.masters);
    // 定时执行 Sentinel 脚本
    sentinelRunPendingScripts();
    sentinelCollectTerminatedScripts();
    sentinelKillTimedoutScripts();
	// 随机化 sentinelTimer 下次执行时间，为了避免故障迁移中使用 Raft 算法选举 leader 节点时有多个节点同时发送投票请求
    server.hz = CONFIG_DEFAULT_HZ + rand() % CONFIG_DEFAULT_HZ;
}
```

​		`Sentinel`机制非常依赖系统时间，其定义了`TILT`模式：每次执行`sentinelTimer`都会检查上次执行`sentinelTimer`函数的时间与当前系统时间之差，如果出

现负数或时间差特别大，则`Sentinel`节点进入`TILT`模式，`TILT`模式是一种保护模式，该模式下的`Sentinel`节点会继续监视所有目标，但有区别：

​				它不在执行任何操作。

​				当其他`Sentinel`节点询问它对于某个主节点主观下线的判定结果时，它将返回节点未下线的结果，因为它执行的下线判断已经不在准确。

```c++
// sentinel.c
void sentinelHandleDictOfRedisInstances(dict *instances) {
    dictIterator *di;
    dictEntry *de;
    sentinelRedisInstance *switch_to_promoted = NULL;

    di = dictGetIterator(instances);
    while((de = dictNext(di)) != NULL) {
        sentinelRedisInstance *ri = dictGetVal(de);
		// 调用主逻辑函数 sentinelHandleRedisInstance
        sentinelHandleRedisInstance(ri);
        // 如果当前处理的是主节点，则递归调用 sentinelHandleDictOfRedisInstances 处理该主节点下的 slaves 字典和 sentinel 字典
        if (ri->flags & SRI_MASTER) {
            sentinelHandleDictOfRedisInstances(ri->slaves);
            sentinelHandleDictOfRedisInstances(ri->sentinels);
            if (ri->failover_state == SENTINEL_FAILOVER_STATE_UPDATE_CONFIG) {
                switch_to_promoted = ri;
            }
        }
    }
    // 完成故障迁移最后一步
    if (switch_to_promoted)
        sentinelFailoverSwitchToPromotedSlave(switch_to_promoted);
    dictReleaseIterator(di);
}

void sentinelHandleRedisInstance(sentinelRedisInstance *ri) {
    /* ========== MONITORING HALF ============ */
    /* Every kind of instance */
    // 建立网络连接
    sentinelReconnectInstance(ri);
    // 发送定时消息
    sentinelSendPeriodicCommands(ri);

    /* ============== ACTING HALF ============= */
    /* We don't proceed with the acting half if we are in TILT mode.
     * TILT happens when we find something odd with the time, like a
     * sudden change in the clock. */
    if (sentinel.tilt) {
        if (mstime()-sentinel.tilt_start_time < SENTINEL_TILT_PERIOD) return;
        sentinel.tilt = 0;
        sentinelEvent(LL_WARNING,"-tilt",NULL,"#tilt mode exited");
    }

    /* Every kind of instance */
    // 检查是否存在主观下线的节点
    sentinelCheckSubjectivelyDown(ri);

    /* Masters and slaves */
    if (ri->flags & (SRI_MASTER|SRI_SLAVE)) {
        /* Nothing so far. */
    }

    // 只对主节点执行
    if (ri->flags & SRI_MASTER) {
        // 检查是否存在客观下线的节点
        sentinelCheckObjectivelyDown(ri);
        // 判断是否可以进行故障迁移
        if (sentinelStartFailoverIfNeeded(ri))
            // 发起投票请求
            sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_ASK_FORCED);
        // 实现故障迁移
        sentinelFailoverStateMachine(ri);
        // 发起询问请求，询问其他 Sentinel 节点对该主节点主观下线的判定结果
        sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_NO_FLAGS);
    }
}
```



##### 建立网络连接

​		`sentinelReconnectInstance`负责建立当前`Sentinel`节点与其他节点的网络连接，包括首次建立连接，以及连接断开后重建连接。每个`Sentinel`节点都与所

有监控的主从节点建立命令连接、订阅连接，并与集群其他`Sentinel`节点建立命令连接。

```c++
// sentinel.c
void sentinelReconnectInstance(sentinelRedisInstance *ri) {
    if (ri->link->disconnected == 0) return;
    if (ri->addr->port == 0) return; /* port == 0 means invalid address. */
    instanceLink *link = ri->link;
    mstime_t now = mstime();

    if (now - ri->link->last_reconn_time < SENTINEL_PING_PERIOD) return;
    ri->link->last_reconn_time = now;

    /* Commands connection. */
    // 创建命令连接
    if (link->cc == NULL) {
        link->cc = redisAsyncConnectBind(ri->addr->ip,ri->addr->port,NET_FIRST_BIND_ADDR);
        if (link->cc && !link->cc->err) anetCloexec(link->cc->c.fd);
        if (!link->cc) {
            sentinelEvent(LL_DEBUG,"-cmd-link-reconnection",ri,"%@ #Failed to establish connection");
        } else if (!link->cc->err && server.tls_replication &&
                (instanceLinkNegotiateTLS(link->cc) == C_ERR)) {
            sentinelEvent(LL_DEBUG,"-cmd-link-reconnection",ri,"%@ #Failed to initialize TLS");
            instanceLinkCloseConnection(link,link->cc);
        } else if (link->cc->err) {
            sentinelEvent(LL_DEBUG,"-cmd-link-reconnection",ri,"%@ #%s",
                link->cc->errstr);
            instanceLinkCloseConnection(link,link->cc);
        } else {
            link->pending_commands = 0;
            link->cc_conn_time = mstime();
            link->cc->data = link;
            redisAeAttach(server.el,link->cc);
            redisAsyncSetConnectCallback(link->cc,
                    sentinelLinkEstablishedCallback);
            redisAsyncSetDisconnectCallback(link->cc,
                    sentinelDisconnectCallback);
            sentinelSendAuthIfNeeded(ri,link->cc);
            sentinelSetClientName(ri,link->cc,"cmd");

            /* Send a PING ASAP when reconnecting. */
            sentinelSendPing(ri);
        }
    }
    /* Pub / Sub */
    // 如果待连接的节点是主从节点，则创建订阅连接
    if ((ri->flags & (SRI_MASTER|SRI_SLAVE)) && link->pc == NULL) {
        link->pc = redisAsyncConnectBind(ri->addr->ip,ri->addr->port,NET_FIRST_BIND_ADDR);
        if (link->pc && !link->pc->err) anetCloexec(link->pc->c.fd);
        if (!link->pc) {
            sentinelEvent(LL_DEBUG,"-pubsub-link-reconnection",ri,"%@ #Failed to establish connection");
        } else if (!link->pc->err && server.tls_replication &&
                (instanceLinkNegotiateTLS(link->pc) == C_ERR)) {
            sentinelEvent(LL_DEBUG,"-pubsub-link-reconnection",ri,"%@ #Failed to initialize TLS");
        } else if (link->pc->err) {
            sentinelEvent(LL_DEBUG,"-pubsub-link-reconnection",ri,"%@ #%s",
                link->pc->errstr);
            instanceLinkCloseConnection(link,link->pc);
        } else {
            int retval;
            link->pc_conn_time = mstime();
            link->pc->data = link;
            redisAeAttach(server.el,link->pc);
            redisAsyncSetConnectCallback(link->pc,
                    sentinelLinkEstablishedCallback);
            redisAsyncSetDisconnectCallback(link->pc,
                    sentinelDisconnectCallback);
            sentinelSendAuthIfNeeded(ri,link->pc);
            sentinelSetClientName(ri,link->pc,"pubsub");
            /* Now we subscribe to the Sentinels "Hello" channel. */
            // 为订阅连接注册回调函数 sentinelReceiveHelloMessages，负责处理 Sentinel 频道收到的数据
            retval = redisAsyncCommand(link->pc,
                sentinelReceiveHelloMessages, ri, "%s %s",
                sentinelInstanceMapCommand(ri,"SUBSCRIBE"),
                SENTINEL_HELLO_CHANNEL);
            if (retval != C_OK) {
                /* If we can't subscribe, the Pub/Sub connection is useless
                 * and we can simply disconnect it and try again. */
                instanceLinkCloseConnection(link,link->pc);
                return;
            }
        }
    }
    /* Clear the disconnected status only if we have both the connections
     * (or just the commands connection if this is a sentinel instance). */
    if (link->cc && (ri->flags & SRI_SENTINEL || link->pc))
        link->disconnected = 0;
}
```



##### 定时消息

```c++
// sentinel.c
void sentinelSendPeriodicCommands(sentinelRedisInstance *ri) {
    mstime_t now = mstime();
    mstime_t info_period, ping_period;
    int retval;

    if (ri->link->disconnected) return;

    if (ri->link->pending_commands >=
        SENTINEL_MAX_PENDING_COMMANDS * ri->link->refcount) return;

    if ((ri->flags & SRI_SLAVE) &&
        ((ri->master->flags & (SRI_O_DOWN|SRI_FAILOVER_IN_PROGRESS)) ||
         (ri->master_link_down_time != 0)))
    {
        info_period = 1000;
    } else {
        info_period = SENTINEL_INFO_PERIOD;
    }

    ping_period = ri->down_after_period;
    if (ping_period > SENTINEL_PING_PERIOD) ping_period = SENTINEL_PING_PERIOD;
	// 给所有主从节点发送 INFO 命令，并注册 sentinelInfoReplyCallback 为回调函数，INFO 命令默认发送间隔为 1 秒，如果接收命令的是从节点，而且已经下线或其主节点正在执行故障迁移，则将时间间隔调整为 10 秒
    if ((ri->flags & SRI_SENTINEL) == 0 &&
        (ri->info_refresh == 0 ||
        (now - ri->info_refresh) > info_period))
    {
        retval = redisAsyncCommand(ri->link->cc,
            sentinelInfoReplyCallback, ri, "%s",
            sentinelInstanceMapCommand(ri,"INFO"));
        if (retval == C_OK) ri->link->pending_commands++;
    }
	// 给所有节点发送 PING 命令，Sentinel 命令通过 PING 命令检测其他节点下线状态，发送间隔为 1 秒
    if ((now - ri->link->last_pong_time) > ping_period &&
               (now - ri->link->last_ping_time) > ping_period/2) {
        sentinelSendPing(ri);
    }
	// 给所有节点发送 Sentinel 频道命令，发送间隔为 2 秒
    if ((now - ri->last_pub_time) > SENTINEL_PUBLISH_PERIOD) {
        sentinelSendHello(ri);
    }
}
```

​		通过`sentinelInfoReplyCallback`以及`sentinelReceiveHelloMessages`两个回调函数，`Sentinel`可以获取所有的从节点和其他`Sentinel`节点信息。最后每个

`Sentinel`节点最终都拥有其监控的主从集群信息及集群其他`Sentinel`节点信息。



#### 故障迁移

​		在执行故障迁移前，需要先判断节点是否下线。如果是的那个`Sentinel`节点，则可通过检查某个节点是否及时响应请求来判断该节点是否下线。但在

`Sentinel`集群中，还需要使用`Quorum`机制，使`Sentinel`集群内达成共识：该节点已经下线。所以在`Sentinel`集群中判定节点是否下线需要两步：

​				主观下线：如果某个节点超时未响应，则`Sentinel`节点可以判定该节点主观下线，代表该`Sentinel`节点认为该节点已下线。

​				客观下线：如果集群判定该节点主观下线的`Sentinel`节点数量不少于法定节点数，则可以判定该节点客观下线，代表集群达成了该节点已下线的共识。



##### 主观下线

​		主逻辑函数定时调用`sentinelCheckSubjectivelyDown`，检测某个节点是否主观下线：

```c++
// sentinel.c
void sentinelCheckSubjectivelyDown(sentinelRedisInstance *ri) {
    mstime_t elapsed = 0;
	// 计算目标节点上次响应后过去的时间
    // 上次给目标节点发送 PING 命令未收到响应
    if (ri->link->act_ping_time)
        elapsed = mstime() - ri->link->act_ping_time;
    else if (ri->link->disconnected)
        // 收到响应，但当前连接已断开
        elapsed = mstime() - ri->link->last_avail_time;

    if (ri->link->cc &&
        (mstime() - ri->link->cc_conn_time) >
        SENTINEL_MIN_LINK_RECONNECT_PERIOD &&
        ri->link->act_ping_time != 0 && /* There is a pending ping... */
        /* The pending ping is delayed, and we did not receive
         * error replies as well. */
        (mstime() - ri->link->act_ping_time) > (ri->down_after_period/2) &&
        (mstime() - ri->link->last_pong_time) > (ri->down_after_period/2))
    {
        instanceLinkCloseConnection(ri->link,ri->link->cc);
    }

    if (ri->link->pc &&
        (mstime() - ri->link->pc_conn_time) >
         SENTINEL_MIN_LINK_RECONNECT_PERIOD &&
        (mstime() - ri->link->pc_last_activity) > (SENTINEL_PUBLISH_PERIOD*3))
    {
        instanceLinkCloseConnection(ri->link,ri->link->pc);
    }

    // 若节点上次响应后过去的时间已超过目标实例属性 down_after_period 指定时间 或 当前 Sentinel 节点认为目标节点是主节点，但 INFO 响应中是从节点，而且目标节点上次响应 INFO 命令已过去时间超过了目标实例属性 down_after_period 指定时间加上两个 INFO 命令发送的间隔时间，则判定目标节点主观下线，并给该节点实例添加 SRI_S_DOWN 标志
    if (elapsed > ri->down_after_period ||
        (ri->flags & SRI_MASTER &&
         ri->role_reported == SRI_SLAVE &&
         mstime() - ri->role_reported_time >
          (ri->down_after_period+SENTINEL_INFO_PERIOD*2)))
    {
        /* Is subjectively down */
        if ((ri->flags & SRI_S_DOWN) == 0) {
            sentinelEvent(LL_WARNING,"+sdown",ri,"%@");
            ri->s_down_since_time = mstime();
            ri->flags |= SRI_S_DOWN;
        }
    } else {
        // 不满足上面的条件，则清除 SRI_S_DOWN 标志
        /* Is subjectively up */
        if (ri->flags & SRI_S_DOWN) {
            sentinelEvent(LL_WARNING,"-sdown",ri,"%@");
            ri->flags &= ~(SRI_S_DOWN|SRI_SCRIPT_KILL_SENT);
        }
    }
}
```



##### 客观下线

​		主逻辑函数会定时针对主节点调用`sentinelAskMasterStateToOtherSentinels`函数。当某个主节点已客观下线后，该函数会循环其他`Sentinel`节点对该主节

点主观下线的判定结果。其他`Sentinel`节点会回复一个标志`isdown`，该标志为`1`代表它也判定主节点主观下线。而当前`Sentinel`节点收到该标志后，会给返回

该标志的`Sentinel`节点的实例添加`ARI_MASTER_DOWN`标志，代表`Sentinel`节点也判定主节点下线。

​		主逻辑函数会调用`sentinelCheckObjectivelyDown`，检查每个主节点是否已客观下线：

```c++
// sentinel.c
void sentinelCheckObjectivelyDown(sentinelRedisInstance *master) {
    dictIterator *di;
    dictEntry *de;
    unsigned int quorum = 0, odown = 0;

    if (master->flags & SRI_S_DOWN) {
        /* Is down for enough sentinels? */
        // 如果目标节点已主观下线，则遍历其 sentinels 字典，统计集群中其他判定目标节点主观下线的 Sentinel 节点的数量
        quorum = 1; /* the current sentinel. */
        /* Count all the other sentinels. */
        di = dictGetIterator(master->sentinels);
        while((de = dictNext(di)) != NULL) {
            sentinelRedisInstance *ri = dictGetVal(de);

            if (ri->flags & SRI_MASTER_DOWN) quorum++;
        }
        dictReleaseIterator(di);
        // 统计数量不少于目标实例属性 quorum (法定节点数)，则判定目标节点已客观下线
        if (quorum >= master->quorum) odown = 1;
    }

    /* Set the flag accordingly to the outcome. */
    // 根据判定结果，给目标节点实例添加或清除 SRI_O_DOWN 标志
    if (odown) {
        if ((master->flags & SRI_O_DOWN) == 0) {
            sentinelEvent(LL_WARNING,"+odown",master,"%@ #quorum %d/%d",
                quorum, master->quorum);
            master->flags |= SRI_O_DOWN;
            master->o_down_since_time = mstime();
        }
    } else {
        if (master->flags & SRI_O_DOWN) {
            sentinelEvent(LL_WARNING,"-odown",master,"%@");
            master->flags &= ~SRI_O_DOWN;
        }
    }
}
```



​		主逻辑函数`sentinelHandleRedisInstance`调用`sentinelStartFailoverIfNeeded`函数检查当前节点是否可以对目标节点执行故障迁移，如果可以，则当前

`Sentinel`节点会发送投票请求，如果当前`Sentinel`节点能被选为`leader`节点，则由当前节点完成故障迁移：

```c++
// sentinel.c
int sentinelStartFailoverIfNeeded(sentinelRedisInstance *master) {
    /* We can't failover if the master is not in O_DOWN state. */
    // 如果目标节点非客观下线状态，则不允许故障迁移
    if (!(master->flags & SRI_O_DOWN)) return 0;

    /* Failover already in progress? */
    // 如果当前 Sentinel 节点正在执行故障迁移，则不允许故障迁移
    if (master->flags & SRI_FAILOVER_IN_PROGRESS) return 0;

    /* Last failover attempt started too little time ago? */
    // failover_start_time 记录了当前 Sentinel 节点上次故障迁移开始时间或者上次给其他 Sentinel 节点投票的时间，可以认为该属性是 Sentinel 集群上次故障迁移的开始时间
    if (mstime() - master->failover_start_time <
        master->failover_timeout*2)
    {
        if (master->failover_delay_logged != master->failover_start_time) {
            time_t clock = (master->failover_start_time +
                            master->failover_timeout*2) / 1000;
            char ctimebuf[26];

            ctime_r(&clock,ctimebuf);
            ctimebuf[24] = '\0'; /* Remove newline. */
            master->failover_delay_logged = master->failover_start_time;
            serverLog(LL_WARNING,
                "Next failover delay: I will not start a failover before %s",
                ctimebuf);
        }
        return 0;
    }
	// 启动故障迁移处理流程
    sentinelStartFailover(master);
    return 1;
}
```

​		目标实例属性`failover_start_time`可以理解为一个锁，当集群中某个`Sentinel`节点抢先开始故障迁移，将占有这个锁。而其他`Sentinel`节点暂时不能执行

故障迁移，知道上次故障迁移开始后过去的时间超过`failover_timeout`属性的`2`倍，才可以开始新一次故障迁移。

​		目标实例属性`failover_timeout`指定了对主节点执行故障转移最多的花费时间，默认为`180000`秒，即当`Sentinel`节点开始故障迁移后，需要在`3`分钟内完

成故障迁移操作。如果`Sentinel`节点故障迁移失败了，则需要等到`6`分钟后，其他`Sentinel`节点才可以开始新一次故障迁移。

```c++
// sentinel.c
void sentinelStartFailover(sentinelRedisInstance *master) {
    serverAssert(master->flags & SRI_MASTER);

    master->failover_state = SENTINEL_FAILOVER_STATE_WAIT_START; // 代表已经开始对该主节点执行故障迁移
    master->flags |= SRI_FAILOVER_IN_PROGRESS;
    master->failover_epoch = ++sentinel.current_epoch; // 进入新的任期，赋值操作代表任期正在执行故障迁移
    sentinelEvent(LL_WARNING,"+new-epoch",master,"%llu",
        (unsigned long long) sentinel.current_epoch);
    sentinelEvent(LL_WARNING,"+try-failover",master,"%@");
    master->failover_start_time = mstime()+rand()%SENTINEL_MAX_DESYNC;
    master->failover_state_change_time = mstime();
}
```



​		`sentinelAskMasterStateToOtherSentinels`可以发送询问请求和投票请求：

```c++
// sentinel.c
void sentinelAskMasterStateToOtherSentinels(sentinelRedisInstance *master, int flags) {
    dictIterator *di;
    dictEntry *de;
	// 遍历主节点实例的 sentinels 字典
    di = dictGetIterator(master->sentinels);
    while((de = dictNext(di)) != NULL) {
        sentinelRedisInstance *ri = dictGetVal(de);
        mstime_t elapsed = mstime() - ri->last_master_down_reply_time;
        char port[32];
        int retval;

        if (elapsed > SENTINEL_ASK_PERIOD*5) {
            ri->flags &= ~SRI_MASTER_DOWN;
            sdsfree(ri->leader);
            ri->leader = NULL;
        }
		// 仅当目标节点处于主观下线状态时才发送请求
        if ((master->flags & SRI_S_DOWN) == 0) continue;
        if (ri->link->disconnected) continue;
        if (!(flags & SENTINEL_ASK_FORCED) &&
            mstime() - ri->last_master_down_reply_time < SENTINEL_ASK_PERIOD)
            continue;

        /* Ask */
        // 发送询问请求或投票请求，如果是询问请求，最后一个参数为 *；如果是投票请求，最后一个参数为当前 Sentinel 节点的 myid 属性
        ll2string(port,sizeof(port),master->addr->port);
        retval = redisAsyncCommand(ri->link->cc,
                    sentinelReceiveIsMasterDownReply, ri,
                    "%s is-master-down-by-addr %s %s %llu %s",
                    sentinelInstanceMapCommand(ri,"SENTINEL"),
                    announceSentinelAddr(master->addr), port,
                    sentinel.current_epoch,
                    (master->failover_state > SENTINEL_FAILOVER_STATE_NONE) ?
                    sentinel.myid : "*");
        if (retval == C_OK) ri->link->pending_commands++;
    }
    dictReleaseIterator(di);
}
```

​		请求命令由`sentinelCommand`函数处理：

```c++
// sentinel.c
void sentinelCommand(client *c) {
    
    	...
            
    } else if (!strcasecmp(c->argv[1]->ptr,"is-master-down-by-addr")) {
        
        sentinelRedisInstance *ri;
        long long req_epoch;
        uint64_t leader_epoch = 0;
        char *leader = NULL;
        long port;
        int isdown = 0;

        if (c->argc != 6) goto numargserr;
        if (getLongFromObjectOrReply(c,c->argv[3],&port,NULL) != C_OK ||
            getLongLongFromObjectOrReply(c,c->argv[4],&req_epoch,NULL)
                                                              != C_OK)
            return;
    	// 使用 master.ip、master.port 参数煀目标实例
        ri = getSentinelRedisInstanceByAddrAndRunID(sentinel.masters,
            c->argv[2]->ptr,port,NULL);

		// 判断目标节点是否为主观下线状态
        if (!sentinel.tilt && ri && (ri->flags & SRI_S_DOWN) &&
                                    (ri->flags & SRI_MASTER))
            isdown = 1;

		// 如果发送的是选举请求
        if (ri && ri->flags & SRI_MASTER && strcasecmp(c->argv[5]->ptr,"*")) {
            leader = sentinelVoteLeader(ri,(uint64_t)req_epoch,
                                            c->argv[5]->ptr,
                                            &leader_epoch);
        }

		// 返回结果给发送节点
	    /**
	    	isdown：判断结果标志，如果目标节点在接收节点中已被判定为主观下线，则 isdown 为 1，否则为 0
	    	leader：获得接收节点最新投票的节点的 myid 属性，* 代表空
	    	leader_epoch：获得接收节点最新投票的节点的任期
	    */
        addReplyArrayLen(c,3);
        addReply(c, isdown ? shared.cone : shared.czero);
        addReplyBulkCString(c, leader ? leader : "*");
        addReplyLongLong(c, (long long)leader_epoch);
        if (leader) sdsfree(leader);
    } 
	...
}

// 负责判断是否可以给发送节点投票
char *sentinelVoteLeader(sentinelRedisInstance *master, uint64_t req_epoch, char *req_runid, uint64_t *leader_epoch) {

    // 如果发送节点 current_epoch 比接收节点 sentinel.current_epoch 大，则更新接收节点的 sentinel.current_epoch 属性并写入配置文件
    if (req_epoch > sentinel.current_epoch) {
        sentinel.current_epoch = req_epoch;
        sentinelFlushConfig();
        sentinelEvent(LL_WARNING,"+new-epoch",master,"%llu",
            (unsigned long long) sentinel.current_epoch);
    }
	// 如果发送节点 current_epoch 比目标实例属性 leader_epoch 大，则可以给发送节点投票
    if (master->leader_epoch < req_epoch && sentinel.current_epoch <= req_epoch)
    {
        sdsfree(master->leader);
        master->leader = sdsnew(req_runid);
        master->leader_epoch = sentinel.current_epoch;
        sentinelFlushConfig();
        sentinelEvent(LL_WARNING,"+vote-for-leader",master,"%s %llu",
            master->leader, (unsigned long long) master->leader_epoch);
        /* If we did not voted for ourselves, set the master failover start
         * time to now, in order to force a delay before we can start a
         * failover for the same master. */
        if (strcasecmp(master->leader,sentinel.myid))
            master->failover_start_time = mstime()+rand()%SENTINEL_MAX_DESYNC;
    }

    *leader_epoch = master->leader_epoch;
    return master->leader ? sdsnew(master->leader) : NULL;
}
```

​		`sentinelReceiveIsMasterDownReply`负责处理其他节点对请求命令返回的响应数据：

```c++
// sentinel.c
/**
	privdata：请求命令接收节点实例
*/
void sentinelReceiveIsMasterDownReply(redisAsyncContext *c, void *reply, void *privdata) {
    sentinelRedisInstance *ri = privdata;
    instanceLink *link = c->data;
    redisReply *r;

    if (!reply || !link) return;
    link->pending_commands--;
    r = reply;

    /* Ignore every error or unexpected reply.
     * Note that if the command returns an error for any reason we'll
     * end clearing the SRI_MASTER_DOWN flag for timeout anyway. */
    if (r->type == REDIS_REPLY_ARRAY && r->elements == 3 &&
        r->element[0]->type == REDIS_REPLY_INTEGER &&
        r->element[1]->type == REDIS_REPLY_STRING &&
        r->element[2]->type == REDIS_REPLY_INTEGER)
    {
        ri->last_master_down_reply_time = mstime();
        // 如果响应结果的第一个参数为 1，则代表接收节点认为目标节点已下线，给接收节点实例添加 SRI_MASTER_DOWN 标志，用于客观下线的统计，否则清除 SRI_MASTER_DOWN 标志
        if (r->element[0]->integer == 1) {
            ri->flags |= SRI_MASTER_DOWN;
        } else {
            ri->flags &= ~SRI_MASTER_DOWN;
        }
        // 如果第二个参数不为 * ，代表接收节点同意给发送节点 leader 选举投票，然后将第2/3个参数记录到接收节点实例的 leader、leader_epoch属性中，用于统计该任期投票结果
        if (strcmp(r->element[1]->str,"*")) {
            /* If the runid in the reply is not "*" the Sentinel actually
             * replied with a vote. */
            sdsfree(ri->leader);
            if ((long long)ri->leader_epoch != r->element[2]->integer)
                serverLog(LL_WARNING,
                    "%s voted for %s %llu", ri->name,
                    r->element[1]->str,
                    (unsigned long long) r->element[2]->integer);
            ri->leader = sdsnew(r->element[1]->str);
            ri->leader_epoch = r->element[2]->integer;
        }
    }
}
```

​		`sentinelFailoverStateMachine`负责完成故障迁移：

```c++
// sentinel.c
void sentinelFailoverStateMachine(sentinelRedisInstance *ri) {
    serverAssert(ri->flags & SRI_MASTER);

    if (!(ri->flags & SRI_FAILOVER_IN_PROGRESS)) return;

    switch(ri->failover_state) {
        case SENTINEL_FAILOVER_STATE_WAIT_START:
            sentinelFailoverWaitStart(ri); // 统计选举投票结果
            break;
        case SENTINEL_FAILOVER_STATE_SELECT_SLAVE:
            sentinelFailoverSelectSlave(ri); // 选择晋升的从节点
            break;
        case SENTINEL_FAILOVER_STATE_SEND_SLAVEOF_NOONE:
            sentinelFailoverSendSlaveOfNoOne(ri); // 将选择的从节点晋升为主节点
            break;
        case SENTINEL_FAILOVER_STATE_WAIT_PROMOTION:
            sentinelFailoverWaitPromotion(ri); // 等待从节点晋升为主节点
            break;
        case SENTINEL_FAILOVER_STATE_RECONF_SLAVES:
            sentinelFailoverReconfNextSlave(ri); // 使其他从节点与晋升从节点建立主从关系
            break;
    }
}
```



​		统计选举投票结果：

```c++
// sentinel.c
void sentinelFailoverWaitStart(sentinelRedisInstance *ri) {
    char *leader;
    int isleader;

    /* Check if we are the leader for the failover epoch. */
    // 统计当前任期的投票结果，返回当选 leader 节点的 myid 属性
    leader = sentinelGetLeader(ri, ri->failover_epoch);
    isleader = leader && strcasecmp(leader,sentinel.myid) == 0;
    sdsfree(leader);

    /* If I'm not the leader, and it is not a forced failover via
     * SENTINEL FAILOVER, then I can't continue with the failover. */
    // 如果当前 Sentinel 节点非 leader 节点
    if (!isleader && !(ri->flags & SRI_FORCE_FAILOVER)) {
        int election_timeout = SENTINEL_ELECTION_TIMEOUT;

        /* The election timeout is the MIN between SENTINEL_ELECTION_TIMEOUT
         * and the configured failover timeout. */
        if (election_timeout > ri->failover_timeout)
            election_timeout = ri->failover_timeout;
        /* Abort the failover if I'm not the leader after some time. */
		// 如果故障迁移开始后过去的时间超过选举超时时间
        if (mstime() - ri->failover_start_time > election_timeout) {
            sentinelEvent(LL_WARNING,"-failover-abort-not-elected",ri,"%@");
            sentinelAbortFailover(ri);
        }
        return;
    }
    // 如果当前 Sentinel 节点是 leader 节点，则进入 SENTINEL_FAILOVER_STATE_SELECT_SLAVE 状态
    sentinelEvent(LL_WARNING,"+elected-leader",ri,"%@");
    if (sentinel.simfailure_flags & SENTINEL_SIMFAILURE_CRASH_AFTER_ELECTION)
        sentinelSimFailureCrash();
    ri->failover_state = SENTINEL_FAILOVER_STATE_SELECT_SLAVE;
    ri->failover_state_change_time = mstime();
    sentinelEvent(LL_WARNING,"+failover-state-select-slave",ri,"%@");
}

char *sentinelGetLeader(sentinelRedisInstance *master, uint64_t epoch) {
    dict *counters;
    dictIterator *di;
    dictEntry *de;
    unsigned int voters = 0, voters_quorum;
    char *myvote;
    char *winner = NULL;
    uint64_t leader_epoch;
    uint64_t max_votes = 0;

    serverAssert(master->flags & (SRI_O_DOWN|SRI_FAILOVER_IN_PROGRESS));
    counters = dictCreate(&leaderVotesDictType,NULL);

    voters = dictSize(master->sentinels)+1; /* All the other sentinels and me.*/

    /* Count other sentinels votes */
    // 每个 Sentinel 的 leader、leader_epoch 属性都存放了该 Sentinel 节点最新投票的节点的任期及 myid 属性，使用 counters 字典统计每个 Sentinel 节点当前任期获得票数，并将 counters 字典获得最多票数的节点存放在 winner 变量中
    di = dictGetIterator(master->sentinels);
    while((de = dictNext(di)) != NULL) {
        sentinelRedisInstance *ri = dictGetVal(de);
        if (ri->leader != NULL && ri->leader_epoch == sentinel.current_epoch)
            sentinelLeaderIncr(counters,ri->leader);
    }
    dictReleaseIterator(di);

    /* Check what's the winner. For the winner to win, it needs two conditions:
     * 1) Absolute majority between voters (50% + 1).
     * 2) And anyway at least master->quorum votes. */
    di = dictGetIterator(counters);
    while((de = dictNext(di)) != NULL) {
        uint64_t votes = dictGetUnsignedIntegerVal(de);

        if (votes > max_votes) {
            max_votes = votes;
            winner = dictGetKey(de);
        }
    }
    dictReleaseIterator(di);

    /* Count this Sentinel vote:
     * if this Sentinel did not voted yet, either vote for the most
     * common voted sentinel, or for itself if no vote exists at all. */
    // 如果当前节点现在未投票，则投给获票最多的几点，如果没有选出最多的节点，则投给自己
    if (winner)
        myvote = sentinelVoteLeader(master,epoch,winner,&leader_epoch);
    else
        myvote = sentinelVoteLeader(master,epoch,sentinel.myid,&leader_epoch);

    if (myvote && leader_epoch == epoch) {
        uint64_t votes = sentinelLeaderIncr(counters,myvote);

        if (votes > max_votes) {
            max_votes = votes;
            winner = myvote;
        }
    }
	// 获得最多的节点其获票数如果不少于集群 Sentinel 节点数量的一半，并且不少于目标实例属性 quorum，那么该节点将赢得选举
    voters_quorum = voters/2+1;
    if (winner && (max_votes < voters_quorum || max_votes < master->quorum))
        winner = NULL;

    winner = winner ? sdsnew(winner) : NULL;
    sdsfree(myvote);
    dictRelease(counters);
    return winner;
}
```



​		选择晋升的从节点：

```c++
// sentinel.c
void sentinelFailoverSelectSlave(sentinelRedisInstance *ri) {
    sentinelRedisInstance *slave = sentinelSelectSlave(ri);

    /* We don't handle the timeout in this state as the function aborts
     * the failover or go forward in the next state. */
    if (slave == NULL) {
        sentinelEvent(LL_WARNING,"-failover-abort-no-good-slave",ri,"%@");
        sentinelAbortFailover(ri);
    } else {
        sentinelEvent(LL_WARNING,"+selected-slave",slave,"%@");
        slave->flags |= SRI_PROMOTED;
        ri->promoted_slave = slave;
        ri->failover_state = SENTINEL_FAILOVER_STATE_SEND_SLAVEOF_NOONE;
        ri->failover_state_change_time = mstime();
        sentinelEvent(LL_NOTICE,"+failover-state-send-slaveof-noone",
            slave, "%@");
    }
}

sentinelRedisInstance *sentinelSelectSlave(sentinelRedisInstance *master) {
    sentinelRedisInstance **instance =
        zmalloc(sizeof(instance[0])*dictSize(master->slaves));
    sentinelRedisInstance *selected = NULL;
    int instances = 0;
    dictIterator *di;
    dictEntry *de;
    mstime_t max_master_down_time = 0;

    if (master->flags & SRI_S_DOWN)
        max_master_down_time += mstime() - master->s_down_since_time;
    max_master_down_time += master->down_after_period * 10;

    di = dictGetIterator(master->slaves);
    while((de = dictNext(di)) != NULL) {
        sentinelRedisInstance *slave = dictGetVal(de);
        mstime_t info_validity_time;
		// 过滤主观下线和客观下线的节点
        if (slave->flags & (SRI_S_DOWN|SRI_O_DOWN)) continue;
        // 过滤命令连接或订阅连接已断开的节点
        if (slave->link->disconnected) continue;
        // 过滤超过 5 秒没有响应 PING 命令的节点
        if (mstime() - slave->link->last_avail_time > SENTINEL_PING_PERIOD*5) continue;
        // 过滤优先级 slave_priority 为 0 的节点
        if (slave->slave_priority == 0) continue;
		
        // 过滤主从连接已断开太久的节点
        if (master->flags & SRI_S_DOWN)
            info_validity_time = SENTINEL_PING_PERIOD*5;
        else
            info_validity_time = SENTINEL_INFO_PERIOD*3;
        if (mstime() - slave->info_refresh > info_validity_time) continue;
        if (slave->master_link_down_time > max_master_down_time) continue;
        instance[instances++] = slave;
    }
    dictReleaseIterator(di);
    if (instances) {
        // 排序
        /**
        	排序规则：
        		按 slave_priority 排序
        		如果 slave_priority 相同，则按 slave_repl_offset 排序
        		如果 slave_repl_offset 相同，按 runid 排序
        */
        qsort(instance,instances,sizeof(sentinelRedisInstance*),
            compareSlavesForPromotion);
        // 取优先级最高的节点为晋升节点
        selected = instance[0];
    }
    zfree(instance);
    return selected;
}
```



​		将选择的从节点晋升为主节点：

```c++
// sentinel.c
void sentinelFailoverSendSlaveOfNoOne(sentinelRedisInstance *ri) {
    int retval;

    if (ri->promoted_slave->link->disconnected) {
        if (mstime() - ri->failover_state_change_time > ri->failover_timeout) {
            sentinelEvent(LL_WARNING,"-failover-abort-slave-timeout",ri,"%@");
            sentinelAbortFailover(ri);
        }
        return;
    }

	// 给晋升的从节点发送 SLAVEOF NO ONE 命令，取消该节点之前的主从关系，将该节点晋升为主节点
    retval = sentinelSendSlaveOf(ri->promoted_slave,NULL);
    if (retval != C_OK) return;
    sentinelEvent(LL_NOTICE, "+failover-state-wait-promotion",
        ri->promoted_slave,"%@");
    ri->failover_state = SENTINEL_FAILOVER_STATE_WAIT_PROMOTION;
    ri->failover_state_change_time = mstime();
}
```



​		等待晋升完成：

```c++
// sentinel.c
void sentinelFailoverWaitPromotion(sentinelRedisInstance *ri) {
    // 检查本次故障迁移时间是否超过目标实例属性 failover_timeout 指定的时间，超过则中断故障迁移
    if (mstime() - ri->failover_state_change_time > ri->failover_timeout) {
        sentinelEvent(LL_WARNING,"-failover-abort-slave-timeout",ri,"%@");
        sentinelAbortFailover(ri);
    }
}
```

​		`sentinelRefreshInstanceInfo`会在`INFO`响应中发现晋升节点已切换至主节点角色，然后切换故障迁移至下一个状态

```c++
// sentinel.c
void sentinelRefreshInstanceInfo(sentinelRedisInstance *ri, const char *info) {
	
    ...
    // 如果当前 Sentinel 节点认为某个节点是从节点，但该节点报告自己为主节点，则说明该节点由从节点切换为主节点
    if ((ri->flags & SRI_SLAVE) && role == SRI_MASTER) {
        /* If this is a promoted slave we can change state to the
         * failover state machine. */
        // 如果该从节点是晋升节点，并且其主节点正在执行故障迁移，则说明该从节点已晋升完成
        if ((ri->flags & SRI_PROMOTED) &&
            (ri->master->flags & SRI_FAILOVER_IN_PROGRESS) &&
            (ri->master->failover_state ==
                SENTINEL_FAILOVER_STATE_WAIT_PROMOTION))
        {

            ri->master->config_epoch = ri->master->failover_epoch;
            ri->master->failover_state = SENTINEL_FAILOVER_STATE_RECONF_SLAVES;
            ri->master->failover_state_change_time = mstime();
            sentinelFlushConfig();
            sentinelEvent(LL_WARNING,"+promoted-slave",ri,"%@");
            if (sentinel.simfailure_flags &
                SENTINEL_SIMFAILURE_CRASH_AFTER_PROMOTION)
                sentinelSimFailureCrash();
            sentinelEvent(LL_WARNING,"+failover-state-reconf-slaves",
                ri->master,"%@");
            sentinelCallClientReconfScript(ri->master,SENTINEL_LEADER,
                "start",ri->master->addr,ri->addr);
            sentinelForceHelloUpdateForMaster(ri->master);
        } else {

            mstime_t wait_time = SENTINEL_PUBLISH_PERIOD*4;

            if (!(ri->flags & SRI_PROMOTED) &&
                 sentinelMasterLooksSane(ri->master) &&
                 sentinelRedisInstanceNoDownFor(ri,wait_time) &&
                 mstime() - ri->role_reported_time > wait_time)
            {
                int retval = sentinelSendSlaveOf(ri,ri->master->addr);
                if (retval == C_OK)
                    sentinelEvent(LL_NOTICE,"+convert-to-slave",ri,"%@");
            }
        }
    }

    ...
}
```



​		建立主从关系：

​		下线节点原来的从节点需要与晋升节点建立主从关系，主从状态标志：

​				`SRI_RECONF_SENT`：已发送`SALVEOF`命令给从节点

​				`SRI_RECONFC_INPROG`：正在建立主从关系

​				`SRI_RECONF_DONE`：主从关系建立完成

​		`sentinelFailoverReconfNextSlave`给下线节点原来的从节点发送`SALVEOF`命令，使它们称为晋升节点的从节点：

```c++
// sentinel.c
void sentinelFailoverReconfNextSlave(sentinelRedisInstance *master) {
    dictIterator *di;
    dictEntry *de;
    int in_progress = 0;

    di = dictGetIterator(master->slaves);
    // 统计当前正在建立主从关系的节点数量，如果当前正在建立主从关系的节点数量不少于目标实例属性 parallel_syncs，则当前不允许再与新的从节点建立主从关系
    while((de = dictNext(di)) != NULL) {
        sentinelRedisInstance *slave = dictGetVal(de);
        if (slave->flags & (SRI_RECONF_SENT|SRI_RECONF_INPROG))
            in_progress++;
    }
    dictReleaseIterator(di);
	// 遍历下线节点所有的从节点
    di = dictGetIterator(master->slaves);
    while(in_progress < master->parallel_syncs &&
          (de = dictNext(di)) != NULL)
    {
        sentinelRedisInstance *slave = dictGetVal(de);
        int retval;

        /* Skip the promoted slave, and already configured slaves. */
        // 过滤晋升节点以及已经开始建立主从关系的节点
        if (slave->flags & (SRI_PROMOTED|SRI_RECONF_DONE)) continue;

        if ((slave->flags & SRI_RECONF_SENT) &&
            (mstime() - slave->slave_reconf_sent_time) >
            SENTINEL_SLAVE_RECONF_TIMEOUT)
        {
            sentinelEvent(LL_NOTICE,"-slave-reconf-sent-timeout",slave,"%@");
            slave->flags &= ~SRI_RECONF_SENT;
            slave->flags |= SRI_RECONF_DONE;
        }

        /* Nothing to do for instances that are disconnected or already
         * in RECONF_SENT state. */
        if (slave->flags & (SRI_RECONF_SENT|SRI_RECONF_INPROG)) continue;
        if (slave->link->disconnected) continue;

        /* Send SLAVEOF <new master>. */
        // 发送 SALVEOF 命令，并给从节点实例添加 SRI_RECONF_SENT 标志
        retval = sentinelSendSlaveOf(slave,master->promoted_slave->addr);
        if (retval == C_OK) {
            slave->flags |= SRI_RECONF_SENT;
            slave->slave_reconf_sent_time = mstime();
            sentinelEvent(LL_NOTICE,"+slave-reconf-sent",slave,"%@");
            in_progress++;
        }
    }
    dictReleaseIterator(di);

    /* Check if all the slaves are reconfigured and handle timeout. */
    // 统计所有从节点的主从状态，如果所有从节点都存在 SRI_RECONF_DONE 标志，则故障迁移状态切换到 SENTINEL_FAILOVER_STATE_UPDATE_CONFIG 状态
    sentinelFailoverDetectEnd(master);
}
```

​		`sentinelRefreshInstanceInfo`在`INFO`响应中发现主从关系已建立完成时，将修改从节点实例的主从状态标志：

```c++
// sentinel.c
void sentinelRefreshInstanceInfo(sentinelRedisInstance *ri, const char *info) {

    ...
        
    if ((ri->flags & SRI_SLAVE) && role == SRI_SLAVE &&
        (ri->flags & (SRI_RECONF_SENT|SRI_RECONF_INPROG)))
    {
		// 如果返回 INFO 响应的节点实例存在 SRI_RECONF_SENT 标志，并且该节点的主节点已经是晋升节点，则将该节点实例的主从状态标志变更为 SRI_RECONF_INPROG，代表主从节点正在建立主从关系
        if ((ri->flags & SRI_RECONF_SENT) &&
            ri->slave_master_host &&
            sentinelAddrEqualsHostname(ri->master->promoted_slave->addr,
                ri->slave_master_host) &&
            ri->slave_master_port == ri->master->promoted_slave->addr->port)
        {
            ri->flags &= ~SRI_RECONF_SENT;
            ri->flags |= SRI_RECONF_INPROG;
            sentinelEvent(LL_NOTICE,"+slave-reconf-inprog",ri,"%@");
        }

		// 如果返回 INFO 响应的节点的实例存在 SRI_RECONF_SENT 标志，并且该节点的主从连接已处于在线状态，则将该节点实例的主从状态标志变更为 SRI_RECONF_DONE，代表主从关系建立完成
        if ((ri->flags & SRI_RECONF_INPROG) &&
            ri->slave_master_link_status == SENTINEL_MASTER_LINK_STATUS_UP)
        {
            ri->flags &= ~SRI_RECONF_INPROG;
            ri->flags |= SRI_RECONF_DONE;
            sentinelEvent(LL_NOTICE,"+slave-reconf-done",ri,"%@");
        }
    }
}
```



​		更新视图数据与配置文件：

​		主逻辑函数`sentinelHandleDictOfRedisInstances`在故障迁移状态进入`SENTINEL_FAILOVER_STATE_UPDATE_CONFIG`状态时，会调用

`sentinelFailoverSwitchToPromotedSlave`函数，更新当前`Sentinel`节点的`sentinel.masters`字典和配置文件。

​		最后还需要通知集群的其他节点更新数据与配置文件，当前`Sentinel`节点会通过`Sentinel`频道发送最新的主节点信息给其他`Sentinel`节点，当其他

`Sentinel`节点在`sentinelProcessHelloMessage`函数中发现主节点已变更时，则更新自己的`sentinel.masters`字典和配置文件：

```c++
// sentinel.c
void sentinelProcessHelloMessage(char *hello, int hello_len) {
    /* Format is composed of 8 tokens:
     * 0=ip,1=port,2=runid,3=current_epoch,4=master_name,
     * 5=master_ip,6=master_port,7=master_config_epoch. */
    int numtokens, port, removed, master_port;
    uint64_t current_epoch, master_config_epoch;
    char **token = sdssplitlen(hello, hello_len, ",", 1, &numtokens);
    sentinelRedisInstance *si, *master;

    if (numtokens == 8) {
        /* Obtain a reference to the master this hello message is about */
        master = sentinelGetMasterByName(token[4]);
        if (!master) goto cleanup; /* Unknown master, skip the message. */

        /* First, try to see if we already have this sentinel. */
        port = atoi(token[1]);
        master_port = atoi(token[6]);
        si = getSentinelRedisInstanceByAddrAndRunID(
                        master->sentinels,token[0],port,token[2]);
        current_epoch = strtoull(token[3],NULL,10);
        master_config_epoch = strtoull(token[7],NULL,10);

        if (!si) {
            /* If not, remove all the sentinels that have the same runid
             * because there was an address change, and add the same Sentinel
             * with the new address back. */
            removed = removeMatchingSentinelFromMaster(master,token[2]);
            if (removed) {
                sentinelEvent(LL_NOTICE,"+sentinel-address-switch",master,
                    "%@ ip %s port %d for %s", token[0],port,token[2]);
            } else {
                /* Check if there is another Sentinel with the same address this
                 * new one is reporting. What we do if this happens is to set its
                 * port to 0, to signal the address is invalid. We'll update it
                 * later if we get an HELLO message. */
                sentinelRedisInstance *other =
                    getSentinelRedisInstanceByAddrAndRunID(
                        master->sentinels, token[0],port,NULL);
                if (other) {
                    sentinelEvent(LL_NOTICE,"+sentinel-invalid-addr",other,"%@");
                    other->addr->port = 0; /* It means: invalid address. */
                    sentinelUpdateSentinelAddressInAllMasters(other);
                }
            }

            /* Add the new sentinel. */
            si = createSentinelRedisInstance(token[2],SRI_SENTINEL,
                            token[0],port,master->quorum,master);

            if (si) {
                if (!removed) sentinelEvent(LL_NOTICE,"+sentinel",si,"%@");
                /* The runid is NULL after a new instance creation and
                 * for Sentinels we don't have a later chance to fill it,
                 * so do it now. */
                si->runid = sdsnew(token[2]);
                sentinelTryConnectionSharing(si);
                if (removed) sentinelUpdateSentinelAddressInAllMasters(si);
                sentinelFlushConfig();
            }
        }

        /* Update local current_epoch if received current_epoch is greater.*/
        if (current_epoch > sentinel.current_epoch) {
            sentinel.current_epoch = current_epoch;
            sentinelFlushConfig();
            sentinelEvent(LL_WARNING,"+new-epoch",master,"%llu",
                (unsigned long long) sentinel.current_epoch);
        }

        /* Update master info if received configuration is newer. */
        // 如果频道消息中的主节点 config_epoch 大于当前节点视图中的主节点 config_epoch，说明发送节点完成了新的故障转移，更新当前节点的 sentinel.masters 字典与配置文件
        if (si && master->config_epoch < master_config_epoch) {
            master->config_epoch = master_config_epoch;
            if (master_port != master->addr->port ||
                !sentinelAddrEqualsHostname(master->addr, token[5]))
            {
                sentinelAddr *old_addr;

                sentinelEvent(LL_WARNING,"+config-update-from",si,"%@");
                sentinelEvent(LL_WARNING,"+switch-master",
                    master,"%s %s %d %s %d",
                    master->name,
                    announceSentinelAddr(master->addr), master->addr->port,
                    token[5], master_port);

                old_addr = dupSentinelAddr(master->addr);
                sentinelResetMasterAndChangeAddress(master, token[5], master_port);
                sentinelCallClientReconfScript(master,
                    SENTINEL_OBSERVER,"start",
                    old_addr,master->addr);
                releaseSentinelAddr(old_addr);
            }
        }

        /* Update the state of the Sentinel. */
        if (si) si->last_hello_time = mstime();
    }

cleanup:
    sdsfreesplitres(token,numtokens);
}
```



​		由于故障迁移可能导致主节点变更，客户端不能直接访问主节点，而要从`Sentinel`集群中获取最新主节点的地址信息：

​				1、依次尝试连接`Sentinel`集群中的节点，直到找到一个连接成功的`Sentinel`节点。

​				2、向`Sentinel`节点发送命令询问主节点的地址信息。

​				3、客户端使用获取的地址信息连接主节点，如果客户端与主节点连接断开，则从第`1`步开始重新执行。

















​				

​		

​		

















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



  













