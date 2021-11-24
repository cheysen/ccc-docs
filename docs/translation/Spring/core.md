# 核心技术

> 5.3.2

# 1.Ioc容器

## 1.1. Spring Ioc容器和Beans介绍

本章介绍了控制反转（IoC）原理的Spring框架实现。 IoC也称为依赖注入（DI）。 在此过程中，对象仅通过构造函数参数，工厂方法的参数或在构造后或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即与它们一起使用的其他对象） 。 然后，容器在创建bean时注入那些依赖项。 此过程从根本上讲是bean本身通过使用类的直接构造或诸如[服务定位器模式](#服务定位器模式)之类的机制来控制其依赖项的实例或位置的逆过程（因此称为控制的反转）。

`org.springframework.beans`和`org.springframework.context`包是Spring Framework的IoC容器的基础。`BeanFactory`接口提供了一种能够管理任何类型对象的高级配置机制。`ApplicationContext`是`BeanFactory`的子接口。 它增加了：

- 与Spring AOP功能轻松集成
- 消息资源处理（用于国际化）
- 事件发布
- 应用层特定的上下文，例如Web应用程序中使用的WebApplicationContext

简而言之，BeanFactory提供了配置框架和基本的功能，而ApplicationContext添加了更多企业特定的功能。 ApplicationContext是BeanFactory的完整超集，在本章中仅在Spring的IoC容器描述中使用。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中众多对象中的一个。bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 1.2. Container介绍

org.springframework.context.ApplicationContext接口代表Spring IoC容器，并负责实例化，配置和组装Bean。 容器通过读取配置元数据获取有关要实例化，配置和组装哪些对象的说明。 配置元数据以XML，Java注解或Java代码表示。 它使您能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

Spring提供了ApplicationContext接口的几种实现。 在独立应用程序中，通常创建`ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例。 尽管XML是定义配置元数据的传统格式，但是可以通过提供少量XML配置来声明性地启用对其他元数据格式的支持，从而指示容器将Java注解或代码用作元数据格式。

在大多数应用场景中，不需要显式用户代码即可实例化一个Spring IoC容器的一个或多个实例。 例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（约）样板Web描述符XML通常就足够了。 如果您使用Spring Tools for Eclipse（由Eclipse驱动的开发环境），则只需单击几下鼠标或击键即可轻松创建此样板配置。

下图显示了Spring的工作原理的高级视图。 您的应用程序类与配置元数据结合在一起，以便在创建和初始化ApplicationContext之后，您将拥有一个完全配置且可执行的系统或应用程序。

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

### 1.2.1. 元数据配置

如上图所示，Spring IoC容器使用一种形式的配置元数据。 此配置元数据表示您作为应用程序开发人员如何告诉Spring容器实例化，配置和组装应用程序中的对象。传统上，配置元数据以简单直观的XML格式提供，这是本章大部分用来传达Spring IoC容器的关键概念和功能的格式。

> 基于XML的元数据不是配置元数据的唯一允许形式。 Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。 如今，许多开发人员为他们的Spring应用程序选择基于Java代码的配置。

- 基于注解的配置：Spring 2.5引入了对基于注解的配置元数据的支持。
- 基于Java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。 因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。 要使用这些新功能，请参见`@ Configuration`，`@Bean`，`@Import`和`@DependsOn`注解。

Spring配置由容器必须管理的至少一个（通常是一个以上）bean定义组成。 基于XML的配置元数据将这些bean配置为顶级`<beans />`元素内的`<bean />`元素。 Java配置通常在`@Configuration`类中使用`@Bean`注释的方法。

这些bean定义对应于组成应用程序的实际对象。 通常，您定义服务层对象，数据访问对象（DAO），表示对象（例如Struts Action实例），基础结构对象（例如Hibernate SessionFactories，JMS队列）等等。 通常，不会在容器中配置细粒度的域对象，因为DAO和业务逻辑通常负责创建和加载域对象。 但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。 请参阅使用AspectJ通过Spring依赖注入域对象。

以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

- `id`属性是一个标识单个bean定义的字符串。它的值指代该对象的引用。

- `class`属性定义bean的类型并使用完全限定名。

### 1.2.2. 容器初始化

提供给ApplicationContext构造函数的一个或多个位置路径是资源字符串，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java CLASSPATH等）加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由PetStoreServiceImpl类和两个JpaAccountDao和JpaItemDao类型的数据访问对象组成（基于JPA对象关系映射标准）。 属性名称元素引用JavaBean属性的名称，而ref元素引用另一个bean定义的名称。 id和ref元素之间的这种联系表达了协作对象之间的依赖性。

#### 构造基于XML的元数据

使bean定义跨越多个XML文件可能很有用。 通常，每个单独的XML配置文件都代表体系结构中的逻辑层或模块。您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义。 如上一节中所示，此构造函数具有多个Resource位置。 或者，使用`<import />`元素的一个或多个实例从另一个文件加载bean定义。 以下示例显示了如何执行此操作：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义是从三个文件加载的：services.xml，messageSource.xml和themeSource.xml。 所有位置路径都相对于执行导入的定义文件，因此，services.xml必须与执行导入的文件位于同一目录或类路径位置，而messageSource.xml和themeSource.xml必须位于该位置下方的资源位置 。 如您所见，斜杠被忽略。 但是，*鉴于这些路径是相对的，最好不要使用任何斜线*。 根据Spring Schema，导入的文件的内容（包括顶级`<beans />`元素）必须是有效的XML bean定义。

> 可以但不建议使用相对路径“ ../”引用父目录中的文件。 这样做会创建对当前应用程序外部文件的依赖。 特别是，不建议对classpath：URL（例如，classpath：../ services.xml）使用此引用，在URL中，运行时解析过程会选择“最近”的classpath根目录，然后查看其父目录。 类路径配置的更改可能导致选择其他错误的目录。您始终可以使用绝对资源路径而不是相对路径：例如，file：C：/config/services.xml或classpath：/config/services.xml。 但是请注意，您正在将应用程序的配置耦合到特定的绝对路径。 通常，最好为这样的绝对路径保留一个间接寻址，例如，通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。

命名空间本身提供了import指令特性。除了普通bean定义之外，Spring提供的XML名称空间的选择中还有其他配置特性，例如`context`和`util`命名空间。

#### The Groovy Bean Definition DSL(领域特定语言)

作为外部化配置元数据的另一个示例，Bean定义也可以在Spring的Groovy Bean定义DSL中表达，如Grails框架所示。 通常，这种配置位于“ .groovy”文件中，其结构如以下示例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。 它还允许通过`importBeans`指令导入XML bean定义文件。

### 1.2.3. 使用容器

`ApplicationContext`是高级的工厂接口，该工厂能够维护不同bean及其依赖关系的注册表。 通过使用方法`T getBean(String name，Class <T> requiredType)`，可以检索bean的实例。

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

在Groovy配置中，使用看起来非常相似。它有一个不同的上下文实现类，该类支持groovy(但也理解XML bean定义)。下面的示例显示了Groovy配置

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是结合reader委托使用的GenericApplicationContext，例如，使用XML文件的XmlBeanDefinitionReader，如下面的示例所示

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

还可以为Groovy文件使用GroovyBeanDefinitionReader，如下面的示例所示

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以在相同的ApplicationContext上混合和匹配这样的读取器委派，从不同的配置源读取bean定义.

然后可以使用getBean检索bean的实例。 ApplicationContext接口还有其他几种检索bean的方法，但是理想情况下，您的应用程序代码永远不要使用它们。 实际上，您的应用程序代码应该根本不调用getBean（）方法，因此完全不依赖于Spring API。 例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖项注入，使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖项。

## 1.3. Bean介绍

在容器本身内，bean定义表示为BeanDefinition对象，其中包含（除其他信息外）以下元数据：

- 包限定的类名：通常，定义了Bean的实际实现类。

- Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。

- 引用该bean完成其工作所需的其他bean。 这些引用也称为协作者或依赖项。

- 要在新创建的对象中设置的其他配置设置，例如，池的大小限制或在管理连接池的bean中要使用的连接数。

该元数据转换为一组组成每个bean定义的属性。下表描述了这些属性

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，*ApplicationContext实现还允许注册在容器外部（由用户）创建的现有对象*。 这是通过通过getBeanFactory（）方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory DefaultListableBeanFactory实现。 DefaultListableBeanFactory通过registerSingleton（..）和registerBeanDefinition（..）方法支持此注册。 但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

> Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤期间正确地判断它们。虽然在某种程度上支持覆盖现有元数据和现有的单例实例，但在运行时注册新bean(与对工厂的实时访问并发)不受官方支持，可能会导致并发访问异常、bean容器中的不一致状态，或者两者都有。

### 1.3.1. Beans命名

每个bean具有一个或多个标识符。 这些标识符在承载Bean的容器内必须唯一。 一个bean通常只有一个标识符。 但是，如果需要多个，则可以将多余的别名视为别名。

在基于XML的配置元数据中，可以使用id属性和/或name属性来指定Bean标识符。 id属性可让您精确指定一个id。 按照惯例，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。 如果要为bean引入其他别名，还可以在name属性中指定它们，并用逗号（`，`），分号（`;`）或空格分隔。 作为历史记录，在Spring 3.1之前的版本中，id属性定义为`xsd：ID`类型，该类型限制了可能的字符。 从3.1开始，它被定义为`xsd：string`类型。 请注意，bean ID唯一性仍由容器强制执行，尽管不再由XML解析器执行。

您*不需要提供bean的名称或ID。 如果未明确提供名称或ID，则容器将为该bean生成一个唯一的名称。 但是，如果要按名称引用该bean，则通过使用ref元素或服务定位器样式查找，必须提供一个名称。 不提供名称的动机与使用内部bean和自动装配协作类有关*。

> 约定是在命名bean时使用标准Java约定来命名实例字段名。也就是说，bean名称以小写字母开头，并从那里采用驼峰格式。这些名称的例子包括accountManager、accountService、userDao、loginController等等。一致地命名bean可以使配置更易于阅读和理解。另外，如果您使用Spring AOP，在将通知(`advice`)应用到一组与名称相关的bean时，它会很有用。
>
> 通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成Bean名称：本质上，采用简单的类名称并将其初始字符转换为小写。 但是，在特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小写。 这些规则与`java.beans.Introspector.decapitalize`（Spring使用的）定义的规则相同。

#### Bean别名

在bean定义本身中，可以使用由`id`属性指定的最多一个名称和`name`属性中任意数量的其他名称的组合，为bean提供多个名称。 这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如，通过使用特定于该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。

但是，在实际定义bean的地方指定所有别名并不够。 有时需要为在别处定义的bean引入别名。 这在大型系统中通常是这种情况，在大型系统中，配置在每个子系统之间分配，每个子系统都有自己的对象定义集。 在基于XML的配置元数据中，可以使用`<alias />`元素来完成此操作。 以下示例显示了如何执行此操作：

```xml
<alias name="fromName" alias="toName"/>
```

例如，子系统A的配置元数据可以通过subsystemA-dataSource的名称引用数据源。 子系统B的配置元数据可以通过subsystemB-dataSource的名称引用数据源。 组成使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用数据源。 要使所有三个名称都引用相同的对象，可以将以下别名定义添加到配置元数据中：

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并且保证不会与任何其他定义(有效地创建一个名称空间)冲突，但它们引用的是同一个bean。

> 如果使用Javaconfiguration，则@Bean注解可用于提供别名。

### 1.3.2. 实例化Beans

bean定义本质上是创建一个或多个对象的方法。当被请求时，容器查看指定bean的方法，并使用该bean定义封装的配置元数据创建(或获取)实际对象。

如果使用基于XML的配置元数据，则在`<bean />`元素的class属性中指定要实例化的对象的类型（或类）。 这个类属性（在内部是BeanDefinition实例的Class属性）通常是必需的。 （可以通过以下两种方式之一使用Class属性：

- 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用new运算符的Java代码。

- 要指定包含用于创建对象的静态工厂方法的实际类，在不太常见的情况下，容器将在类上调用静态工厂方法以创建Bean。 从静态工厂方法的调用返回的对象类型可以是同一类，也可以是完全不同的另一类。

> **如果要为静态内部类配置Bean定义，则必须使用内部类的二进制名称**。例如，如果您在com.example包中有一个名为SomeThing的类，并且此SomeThing类具有一个名为OtherThing的静态内部类，则bean定义上的class属性的值为com.example.SomeThing $ OtherThing。请注意，名称中使用$字符将内部类的类名与外部类名分开。

#### 构造器实例化

当通过构造方法创建一个bean时，所有普通类都可以被Spring使用并兼容。 也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。 只需指定bean类就足够了。 但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您要管理的任何类。 它不仅限于管理真正的JavaBean。 大多数Spring用户更喜欢实际的JavaBean，它仅具有默认（无参数）构造函数，并具有根据容器中的属性构造的适当setter和getter。 您还可以在容器中具有更多非bean规范的类。 例如，如果您需要使用绝对不符合JavaBean规范的旧式连接池，则Spring也可以对其进行管理。

#### 静态工厂实例化

定义使用静态工厂方法创建的bean时，请使用class属性指定包含静态工厂方法的类，并使用名为factory-method的属性指定工厂方法本身的名称。 您应该能够调用此方法（使用可选参数，如稍后所述）并返回一个活动对象，该对象随后将被视为已通过构造函数创建。 这种bean定义的一种用法是在旧版代码中调用静态工厂。

以下bean定义指定通过调用工厂方法来创建bean。 该定义不指定返回对象的类型（类），而仅指定包含工厂方法的类。 在此示例中，createInstance（）方法必须是静态方法。 以下示例显示如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了可与前面的bean定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

#### 实例工厂实例化

类似于通过静态工厂方法进行实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。 要使用此机制，请将class属性保留为空，并在factory-bean属性中，在当前（或祖先）容器中指定包含要创建该对象的实例方法的bean的名称。 使用factory-method属性设置工厂方法本身的名称。 以下示例显示了如何配置此类Bean：

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含一个以上的工厂方法，如以下示例所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。 

> 在Spring文档中，“ factory bean”是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。 相比之下，`FactoryBean`（注意大小写）是指特定于Spring的FactoryBean实现类。

#### 确定Bean的运行时类型

确定特定bean的运行时类型并非易事。 Bean元数据定义中的指定类只是初始类引用，可能与声明的工厂方法结合使用，或者是FactoryBean类，这可能导致Bean的运行时类型不同，或者在实例级工厂方法（通过指定的factory-bean名称解析）的情况下完全不进行设置 。 此外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

*找出特定bean的实际运行时类型的推荐方法是对指定bean名称的BeanFactory.getType调用*。 这考虑了上述所有情况，并返回了针对相同bean名称的BeanFactory.getBean调用将返回的对象的类型。

## 1.4. 依赖Dependencies

典型的企业应用程序不包含单个对象(或Spring术语中的bean)。即使是最简单的应用程序也有几个对象一起工作，以表示最终用户认为一致的应用程序。下一节将解释如何从定义大量独立的bean定义过渡到一个完全实现的应用程序，在该应用程序中，对象相互协作以实现目标。

### 1.4.1. 依赖注入

依赖注入（DI）是一个过程，在这个过程中，对象仅通过构造函数参数、工厂方法的参数，或者在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖项(即与它们一起工作的其他对象)。 然后，容器在创建bean时注入那些依赖项。 从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖的实例或位置的bean创建的逆过程（因此称为Control Inversion-控制反转）

使用DI原则后，代码更加清晰，当提供对象的依赖关系时，解耦会更加有效。对象不查找它的依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这些依赖关系允许在单元测试中使用stub或mock 实现。

*依赖注入有两种主要的变体:基于构造函数的依赖注入和基于setter的依赖注入。*

#### 构造器注入

基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项。 调用带有特定参数的静态工厂方法来构造Bean几乎是等效的，并且本次讨论将构造函数和静态工厂方法的参数视为类似。 以下示例显示了只能通过构造函数注入进行依赖项注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
	/*这里的MovieFinder需由Ioc容器管理；若有多个构造器的话，需要注意是否正确使用，可参考
      https://blog.csdn.net/qq_41737716/article/details/85596817*/
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，该类没有什么特别的。 它是一个POJO，不依赖于特定于容器的接口，基类或注解。

**构造器参数解析**

构造函数参数解析匹配通过使用参数的类型进行。 如果Bean定义的构造函数参数中没有潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。 考虑以下类：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设ThingTwo和ThingThree类没有通过继承关联，则不存在潜在的歧义。 因此，以下配置可以正常运行，并且您无需在`<constructor-arg />`元素中显式指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。 当使用简单类型（例如<value> true </ value>）时，Spring无法确定值的类型，因此在没有指示的情况下无法按类型进行匹配。 考虑以下类别：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**构造器参数类型匹配**

在上述情况下，如果通过使用`type`属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。 如下例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**构造器参数索引**

您可以使用index属性来显式指定构造函数参数的索引，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义之外，指定索引还可以解决构造函数有两个相同类型的参数的歧义。

> 索引从0开始。

**构造器参数名**

还可以使用构造函数参数名来消除值的歧义，如下面的示例所示

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要使其能够开箱即用，您的代码必须在编译时启用debug标志，以便Spring可以从构造函数查找参数名。如果您不能或不希望用debug标志编译您的代码，您可以使用@ConstructorProperties JDK注解来显式地命名您的构造函数参数。示例类如下所示

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### setter注入

基于setter的DI是由容器在调用无参数构造函数或无参数静态工厂方法来实例化bean之后再调用bean上的setter方法来实现的。

下面的示例显示了一个只能通过使用纯setter注入来注入依赖项的类。这个类是传统的Java。它是一个不依赖于容器特定接口、基类或注解的POJO。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext`支持它管理的bean的基于构造函数和基于setter的DI。 在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。 您可以以`BeanDefinition`的形式配置依赖项，并与`PropertyEditor`实例结合使用，以将属性从一种格式转换为另一种格式。 但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML bean定义，带注解的组件（即以@ Component，@ Controller等进行注解的类）或@Bean方法来处理这些类。 基于Java的@Configuration类。 然后将它们在内部转换为BeanDefinition实例，并用于加载整个Spring IoC容器实例。

> 由于可以混合使用基于构造函数的DI和基于setter的DI，因此将构造函数用于强制性依赖项并将setter方法或配置方法用于可选的依赖项是一个很好的经验。 注意，在setter方法上使用@Required批注可以使该属性成为必需的依赖项。 但是，最好使用带有参数的程序验证的构造函数注入。
>
> Spring团队通常提倡构造函数注入，因为它使您可以将应用程序组件实现为不可变对象，并确保所需的依赖项不为null。 此外，注入构造函数的组件始终以完全初始化的状态返回到客户端（调用）代码。 附带说明一下，构造器包含大量的参数是不好的代码，这表明该类可能承担了太多的职责，应进行重构以更好地解决关注点分离问题。
>
> Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。 否则，必须在代码使用依赖项的任何地方执行非空检查。 setter注入的一个好处是，setter方法使该类的对象在以后可以重新配置或重新注入。 因此，通过JMX MBean进行管理是用于setter注入的典型用例。
>
> 使用对特定类最有意义的DI。 有时，在处理您没有源代码的第三方类时，将为您做出选择。 例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

#### 依赖解析过程

容器按如下方式执行bean依赖项解析：

- 使用描述所有bean的配置元数据创建和初始化ApplicationContext。 可以通过XML，Java代码或注解指定配置元数据。

- 对于每个bean，它的依赖关系都以属性、构造函数参数或静态工厂方法的参数的形式表示(如果您使用该方法而不是普通的构造函数)。当实际创建bean时，将向bean提供这些依赖项。

- 每个属性或构造函数参数都是要设置的实参，或者是对容器中另一个bean的引用。

- 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，比如int、long、string、boolean等等。

在创建容器时，Spring容器会验证每个bean的配置。 但是，在实际创建Bean之前，不会设置Bean属性本身。 创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。 作用域在Bean作用域中定义。 否则，仅在请求时才创建Bean。 创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。 请注意，这些依赖项之间的解析不匹配可能会在后期出现，即在第一次创建受影响的bean时。

> 如果主要使用构造函数注入，则可能会创建无法解析的循环依赖方案。
>
> 例如：A类通过构造函数注入需要B类的实例，而B类通过构造函数注入需要A类的实例。 如果您为将类A和B相互注入而配置了bean，则Spring IoC容器会在运行时检测到此循环引用，并抛出`BeanCurrentlyInCreationException`。
>
> 一种可能的解决方案是编辑某些类的源代码，这些类的源代码由setter而不是构造函数来配置。 或者，避免构造函数注入，而仅使用setter注入。 换句话说，尽管不建议这样做，但是您可以使用setter注入配置循环依赖项。
>
> 与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

通常，您可以信任Spring做正确的事。 它在容器加载时检测配置问题，例如对不存在的Bean的引用和循环依赖项。 Spring设置属性并尽可能晚地解决依赖关系--在实际创建bean时。 这意味着，如果创建对象或其依赖项之一有问题，则正确加载了Spring的容器以后可以在您请求对象时抛出异常-例如，bean会因不存在或无效属性而抛出异常。这样可能会延迟某些配置问题的可见性，所以这就是为什么默认情况下ApplicationContext实现会预先实例化单例bean的原因。 在实际需要这些bean之前需要花一些前期时间和内存来创建它们，您会在创建ApplicationContext时发现配置问题，而不是稍后。 您仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，则在将一个或多个协作Bean注入到从属Bean中时，每个协作Bean在注入到从属Bean中之前都已完全配置。 这意味着，如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB。换句话说，被实例化的bean（如果它不是预先实例化的单例） ），它的依赖项被设置，并调用了相关的生命周期方法（例如已配置的init方法或InitializingBean回调方法）。

#### 依赖注入实例

下面的示例为基于setter的DI使用基于xml的配置元数据。Spring XML配置文件的一小部分指定了一些bean定义，如下所示

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，将setter声明为与XML文件中指定的属性匹配。下面的示例使用基于构造函数的DI

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

bean定义中指定的构造函数参数用作ExampleBean构造函数的参数。

现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用静态工厂方法以返回对象的实例：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数由`<constructor-arg />`元素提供，与实际使用构造函数的情况完全相同。 factory方法返回的类的类型不必与包含静态工厂方法的类的类型相同（尽管在此示例中是）。 实例（非静态）工厂方法可以以基本上相同的方式使用（除了使用factory-bean属性代替class属性之外），因此在此不讨论这些细节。



### 1.4.2.  依赖配置详解

如上一节所述，您可以将bean属性和构造器参数定义为对容器中其他bean（协作者）的引用或内联定义的值。 Spring的基于XML的配置元数据为达到此目的在其`<property />`和`<constructor-arg />`元素中支持子元素类型。

#### 直接值（原语，Strings等）

`<property />`元素的`value`属性将属性或构造器参数指定为人类可读的字符串表示形式。 Spring的转换服务用于将这些值从字符串转换为属性或参数的实际类型。 以下示例显示了设置的各种值：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

下面的示例使用p-namespace进行更简洁的XML配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

前面的XML更简洁。 但是，除非在创建bean定义时使用支持自动属性补全的IDE（例如IntelliJ IDEA或Eclipse的Spring Tools），否则错误是在运行时而不是编码时发现的。 强烈建议您使用此类IDE帮助开发。

您还可以配置`java.util.Properties`实例，如下所示：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器通过使用JavaBeans `PropertyEditor`机制将`<value />`元素内的文本转换为`java.util.Properties`实例。 这很方便，并且是Spring团队偏爱使用嵌套的`<value />`元素而不是`value`属性的几个地方之一。

**`idref`元素**

`idref`元素只是将容器中另一个bean的id（字符串值-不是引用）传递给`<constructor-arg />`或`<property />`元素的一种防错方法。 以下示例显示了如何使用它：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义代码片段(在运行时)与下面的代码片段完全等价

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用idref标记可使容器在部署时验证所引用的名为bean的实际存在。 在第二个变体中，不对传递给客户端bean的targetName属性的值执行验证。 拼写错误仅在实际实例化客户端bean时才发现（最有可能导致致命的结果）。 如果客户端Bean是原型Bean，则可能在部署容器很久之后才发现此错误和所产生的异常。

> 这里传的都是字面量，而不是bean引用，idref与value的区别只是前者会做校验。
>
> 在4.0 Bean XSD中不再支持idref元素上的local属性，因为它不再提供常规Bean引用上的值（这里不好理解，之前local属性的作用是：只能指定与当前配置的对象在同一个配置文件的对象定义的名称，所以这里认为5.3.2把这个属性废弃就行了）。 升级到4.0模式时，将现有的idref local属性更改为idref bean。

`<idref />`元素带来价值的一个常见地方（至少在Spring 2.0之前的版本中）是在ProxyFactoryBean bean定义中配置AOP拦截器。 在指定拦截器名称时使用`<idref />`元素可防止您拼写错误的拦截器ID。

#### 引用其他Bean

ref元素是`<constructor-arg />`或`<property />`定义元素内的最后一个元素。 在这里，您将Bean的指定属性的值设置为对容器管理的另一个Bean（协作者）的引用。 引用的bean是要设置其属性的bean的依赖，并且在设置属性之前根据需要对其进行初始化。 （如果协作者是单例bean，则它可能已经由容器初始化了。）所有引用最终都是对另一个对象的引用。 作用于和验证取决于您是通过`bean`还是`parent`指定另一个对象的ID或名称。

通过`<ref />`标记的bean属性指定目标bean是最通用的形式，并且允许创建对同一容器或父容器中任何bean的引用，而不管它是否在同一XML文件中。 bean属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的值之一相同。 下面的示例演示如何使用ref元素：

```xml
<ref bean="someBean"/>
```

通过`parent`属性指定目标Bean将创建对当前容器的父容器中的Bean的引用。 `parent`属性的值可以与目标Bean的id属性或目标Bean的名称属性中的值之一相同。 目标Bean必须位于当前容器的父容器中。 在容器具有层次结构并且要包装一个与在父容器中已存在的bean具有相同名称的代理bean时，才应该使用`parent`属性。 以下清单显示了如何使用`parent`属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> ref元素的`local`属性在4.0 Bean XSD中不再受支持，因为它不再提供常规Bean引用上的值。 升级到4.0模式时，将现有的ref local更改为ref bean。

#### 内部Beans

`<property />`或`<constructor-arg />`元素内的`<bean />`元素定义了一个内部bean，如以下示例所示：

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义ID或名称。 如果指定，则容器不使用该值作为标识符。 容器在创建时也会忽略`scope`标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。 不能独立访问内部bean，也不能将它们注入到协作bean中（除了封装的bean）。

#### 集合

`<list />`，`<set />`，`<map />`和`<props />`元素分别设置Java集合类型的List，Set，Map和Properties的属性和参数。 以下示例显示了如何使用它们：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

其中，map的key或者value，或set的value，可以是以下任一元素：

```xml
bean | ref | idref | list | set | map | props | value | null
```

**集合合并**

Spring容器还支持合并集合。 应用程序开发人员可以定义父`<list />`，`<map />`，`<set />`或`<props />`元素，并具有从父集合继承并覆盖值的子元素<list />，<map />，<set />或<props />。 也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素会覆盖父集合中指定的值。

下面的示例演示了集合合并：

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意子bean定义的adminEmails属性的`<props />`元素上使用`merge = true`属性。 当子bean由容器解析并实例化后，生成的实例具有adminEmails Properties集合，其中包含将子bean的adminEmails集合与父bean的adminEmails集合合并的结果。 以下清单显示了结果：

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

子`Properties`集合的值集继承了父`<props />`的所有属性，子属性的`support`值覆盖了父集合中的值。

此合并行为类似地适用于`<list />`，`<map />`和`<set />`集合类型。 在`<list />`元素的特定情况下，将维护与List集合类型关联的语义（即，值的有序集合的概念）。 父级的值先于子级列表的所有值。 对于Map，Set和Properties集合类型，不存在排序。 因此，对于容器内部使用的关联Map，Set和Properties实现类型的集合，没有有效的排序语义。

**集合合并的限制**

您不能合并不同的集合类型(如Map和List)。如果您尝试这样做，则会抛出适当的异常。必须在下层的继承的子集合定义上指定merge属性。在父集合定义上指定merge属性是多余的，并且不会导致所需的合并。

**强类型的集合**

随着Java 5中泛型类型的引入，您可以使用强类型集合。 也就是说，可以声明一个Collection类型，使其只能包含（例如）String元素。 如果使用Spring将强类型的Collection依赖注入到Bean中，则可以利用Spring的类型转换支持，以便在将强类型的Collection实例的元素添加到Bean中之前，先将其转换为适当的类型。 以下Java类和bean定义显示了如何执行此操作：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当准备注入`something` bean的accounts属性时，可以通过反射获得有关强类型Map <String，Float>的元素类型的泛型信息。 因此，Spring的类型转换基础架构将各种值元素识别为Float类型，并将字符串值（9.99、2.75和3.99）转换为实际的Float类型。

#### Null和空String

Spring将属性之类的空参数视为空字符串。以下基于xml的配置元数据片段将email属性设置为空字符串值("")。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

等价于：

```java
exampleBean.setEmail("");
```

`<null />`元素处理空值。 以下清单显示了一个示例：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

等价于：

```java
exampleBean.setEmail(null);
```

#### p-namespace

通过p-namespace，您可以使用bean元素的属性（而不是嵌套的`<property />`元素）来描述协作bean的属性值，或同时使用这两者。

Spring支持带有命名空间的可扩展配置格式，这些名称空间基于XML Schema定义。 本章讨论的bean配置格式在XML Schema文档中定义。 但是，p-namespace未在XSD文件中定义，仅存在于Spring Core中。

下面的示例显示了两个解析到相同结果的XML片段(第一个使用标准XML格式，第二个使用p-namespace)：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

该示例显示了bean定义中的p-namespace中的一个名为email的属性。这告诉Spring包含一个属性声明。如前所述，p-namespace没有schema 定义，因此可以将属性的名称设置为属性名。

下一个示例包括另外两个bean定义，它们都引用了另一个bean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

此示例不仅包括使用p-namespace的属性值，而且还使用特殊格式来声明属性引用。 第一个bean定义使用`<property name="spouse" ref="jane" />`创建从bean john到bean jane的引用，而第二个bean定义使用`p:spouse-ref="jane"`属性做相同的事情。 在这种情况下，`spouse`是属性名称，而`-ref`部分表示这不是一个直接值，而是对另一个bean的引用。

> p-namespace不像标准XML格式那么灵活。例如，声明属性引用的格式与以Ref结尾的属性发生冲突(我验证仍能正确解析)，而标准XML格式没有。我们建议您谨慎地选择自己的方法，并将其告知团队成员，以避免生成同时使用这三种方法的XML文档。



#### c-namespace

与p-namespace的XML快捷方式类似，Spring 3.1中引入的c-namespace允许内联属性来配置构造函数参数，而不是嵌套的`constructor-arg`元素。

下面的示例使用`c:`namespace来做和基于构造器的依赖注入相同的事情

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

c:命名空间使用与p: 相同的约定(bean引用的末尾是-ref)，通过它们的名称设置构造器参数。类似地，它需要在XML文件中声明，即使它不是在XSD模式中定义的(它存在于Spring Core中)。

对于极少数情况下无法使用构造函数参数名称的情况（通常，如果字节码是在没有调试信息的情况下编译的），则可以使用参数索引，如下所示：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> 由于XML语法的原因，索引表示法需要有前导符号 "_"，因为XML属性名不能以数字开头(尽管有些ide允许)。`<constructor-arg>`元素也可以使用相应的索引表示法，但不常用，因为在那里声明的普通顺序通常就足够了。

#### 复合属性名

在设置bean属性时，可以使用复合或嵌套属性名，只要路径的所有组件(最终属性名除外)都不是null的。考虑下面的bean定义

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

`something` bean有一个`fred`属性，它有一个`bob`属性，bob还有一个sammy属性，最后的sammy属性被设置为值123。为了正常解析，在构造bean之后，`something`的fred属性和fred的bob属性不能为空。否则，抛出NullPointerException。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<bean id="student" class="pre.chl.spring.core.Student" p:ref-ref="courseRef">
    <!--如果把下面这行注释了就会抛异常NullValueInNestedPathException
	如果不注释，score最后的值也是99，可见p命名空间和原始标签混用可能会出现冲	突，应该只会选原始标签元素去解析-->
    <!--<property name="ref" ref="courseRef"/>-->
    <property name="ref.score" value="99"/>
</bean>
    <bean id="courseRef" class="pre.chl.spring.core.Course" p:score="100"/>
</beans>
```



### 1.4.3. depends-on

如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为另一个bean的属性。 通常，您可以使用基于XML的配置元数据中的`<ref />`元素来完成此操作。 但是，有时bean之间的依赖不太直接。 一个例子是当要触发类中的静态初始化器的时候，例如数据库驱动程序注册。 `depends-on`可以显式地强制初始化一个或多个使用该元素引用的bean。 下面的示例使用depends-on属性来表示对单例bean的依赖关系：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表达对多个bean的依赖关系，请提供一个bean名称的列表作为`depends-on`属性的值（逗号，空格和分号都是合法 的分隔符）：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

>  `depends-on`既可以指定初始化时依赖项，在单例bean的情况下，也可以指定相应的销毁时依赖项。在销毁给定bean本身之前，首先销毁 `depends-on`的bean。因此， `depends-on`还可以控制关闭顺序。

### 1.4.4. 懒加载Beans

默认情况下，作为初始化过程的一部分，`ApplicationContext`实现会尽早创建和配置所有单例bean。 通常，这种预初始化是可取的，因为与数小时甚至数天后相比，会立即发现配置或环境参数中的错误。 如果您不希望这样做，则可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。 延迟初始化的bean告诉IoC容器在首次请求时（而不是在启动时）才创建一个bean实例。

在XML中，此行为由`<bean />`元素上的`lazy-init`属性控制，如以下示例所示：

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

ApplicationContext使用上述配置时，当ApplicationContext启动，`lazy` bean不会立即预实例化，`not.lazy` bean被尽早地预实例化。

但是，当延迟初始化的bean是未延迟初始化的单例bean的依赖项时，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。 延迟初始化的bean被注入到其他未延迟初始化的单例bean中。

您还可以使用`<beans />`元素上的`default-lazy-init`属性在容器级别控制延迟初始化，如以下示例所示：

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 1.4.5. 自动装配协作者

Spring容器可以自动装配协作bean之间的关系。您可以通过检查ApplicationContext的内容，让Spring为您的bean自动解析协作者(其他bean)。自动装配具有以下优点：

- 自动装配可以显著减少指定属性或构造器参数的需要。(本章其他地方讨论的其他机制，比如bean模板，在这方面也很有价值。)

- 随着对象的更新，自动装配可以更新配置。 例如，如果需要将依赖项添加到类中，则无需修改配置即可自动满足该依赖项。 因此，自动装配在开发过程中特别有用，而不必否定在代码库变得更稳定时切换到显式装配的选择。

使用基于XML的配置元数据时（请参阅“依赖注入”），可以使用`<bean />`元素的`autowire`属性为bean定义指定自动装配模式。 自动装配功能具有四种模式。 您可以为每个bean指定自动装配，并选择要自动装配的模式。 下表描述了四种自动装配模式：

| Mode          | Explanation                                                  |
| :------------ | :----------------------------------------------------------- |
| `no`          | (默认)不自动装配。Bean引用必须由`ref`元素定义。对于较大的部署，不建议更改默认设置，因为显式地指定协作者可以提供更好的控制和透明度。在某种程度上，它记录了系统的结构。 |
| `byName`      | 按属性名称自动装配。 Spring查找与需要自动装配的属性同名的bean。 例如，如果一个bean定义被设置为按名称自动装配，并且包含一个master属性（即，它具有setMaster（..）方法），那么Spring将查找一个名为master的bean定义并使用它来设置该属性。 |
| `byType`      | 如果容器中恰好存在该属性类型的一个bean，则自动装配该属性。如果存在多个，将抛出一个致命异常，这表明您不能对该bean使用byType自动装配。如果没有匹配的bean，则什么也不会发生(属性没有设置)。 |
| `constructor` | 与byType类似，但适用于构造函数参数。 如果容器中不存在构造函数参数类型的一个bean，则将引发致命错误。 |

使用byType或构造器自动装配模式，您可以装配数组和集合类。 在这种情况下，将提供容器中与预期类型匹配的所有自动装配候选，以满足依赖。 如果期望的key类型为String，则可以自动装配强类型Map实例。 自动装配的Map实例的值包括与期望类型匹配的所有bean实例，并且Map实例的key包含相应的bean名称。

#### 自动装配的局限和缺点

当在项目中一致使用自动装配时，自动装配效果最佳。 如果不全部使用自动装配，则可能会使开发人员使用自动装配仅来装配一个或两个Bean定义而感到困惑。

考虑自动装配的局限性和缺点：

- `property`和`constructor-arg`设置中的显式依赖总是覆盖自动装配。您不能自动装配简单的属性，如原语、`Strings`和`Classes`(以及这些简单属性的数组)。这种限制是由设计造成的。

- 自动装配不如显示装配严格。尽管如此，正如前面的表中所指出的那样，在出现可能产生意外结果的歧义时，Spring会谨慎地避免猜测。spring管理的对象之间的关系不再显式地记录。

- 从Spring容器生成文档的工具可能无法使用装配信息。

- 容器内的多个bean定义可能与要自动装配的setter方法或构造器参数指定的类型匹配。 对于数组，集合或Map实例，这不一定是问题。 但是，对于需要单个值的依赖项，不会随便解决此歧义。 如果没有唯一的bean定义可用，则会引发异常。

在后一种情况下，您有几种选择：

- 使用显示装配而不是自动装配
- 通过将其bean的`autowire-candidate`属性设置为`false`，避免自动装配bean，如下一节所述。

- 通过将其`<bean />`元素的`primary`属性设置为`true`，将定义的单个bean指定为主要候选对象。

- 实现基于注解的配置提供的更细粒度的控制。

#### 从自动装配中排除Bean

在每个bean的基础上，您可以从自动装配中排除一个bean。 使用Spring的XML格式，将`<bean />`元素的`autowire-candidate`属性设置为`false`。 容器使特定的bean对自动装配基础架构不可用（包括注解式的配置，例如`@Autowired`）。

>  `autowire-candidate`属性被设计为只影响基于类型的自动装配。它不影响通过名称进行的显式引用，即使指定的bean没有被标记为自动装配候选对象，也会解析该引用。因此，如果名称匹配，按名称自动装配仍然会注入一个bean。

您还可以基于与Bean名称的模式匹配来限制自动装配候选项。 顶级`<beans />`元素在其`default-autowire-candidates`属性中接受一个或多个模式。 例如，要将自动装配候选状态限制为名称以`Repository`结尾的任何bean，请提供`* Repository`值。 要提供多种模式，请在以逗号分隔的列表中定义它们。 Bean定义的`autowire-candidate`属性的显式值`true`或`false`始终优先。 对于此类bean，模式匹配规则不适用。

这些技术对于那些您不希望通过自动装配被注入到其他bean中的bean非常有用。这并不意味着不能使用自动装配来配置被排除的bean本身。相反，bean本身并不是自动装配其他bean的候选对象。

### 1.4.6. 方法注入

在大多数应用场景中，容器中的大多数bean是单例的。 当单例Bean需要与另一个单例Bean协作或非单例Bean需要与另一个非单例Bean协作时，通常可以通过将一个Bean定义为另一个Bean的属性来处理依赖。 当bean的生命周期不同时会出现问题。 假设单例bean A可能需要使用非单例（原型）bean B，也许是在A的每个方法调用时使用的。容器仅创建一次单例bean A，因此只有一次机会来设置属性。 每次需要一个bean B时，容器都无法为bean A提供一个新的bean B实例。

一个解决方案是放弃一些控制反转。您可以通过实现`ApplicationContextAware`接口使bean A感知到容器，并在bean A每次需要bean B实例时向容器发出getBean(“B”)调用来获得(通常是新的)。下面的示例演示了这种方法：

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的情况是不可取的，因为业务代码能够感知Spring框架并与之耦合。方法注入是Spring IoC容器的一个高级特性，它让您很好地处理这个用例。

>  您可以在[这篇博客文章](https://spring.io/blog/2004/08/06/method-injection/)中阅读更多关于方法注入的探讨。

#### LookUp方法注入

Lookup 方法注入是容器重写容器管理的Bean上的方法并返回容器中另一个Bean的功能。 查找通常涉及原型bean，如上一节中所述。 Spring框架通过使用CGLIB库中的字节码生成功能来动态生成覆盖该方法的子类，从而实现此方法注入。

> - 为了使此动态子类起作用，Spring Bean容器子类的类型也不能是final，而要覆盖的方法也不能是final。
>
> - 对具有抽象方法的类进行单元测试需要您自己对该类进行子类化，并提供该抽象方法的stub实现。
>
> - 组件扫描也需要具体的方法，这需要具体的类。
>
> - 另一个关键限制是，查找方法不适用于工厂方法，尤其不适用于配置类中的@Bean方法，因为在这种情况下，容器不负责创建实例，因此无法创建运行时生成的 动态子类。

对于前面的代码片段中的CommandManager类，Spring容器动态地重写createCommand（）方法。 如重做的示例所示，CommandManager类没有任何Spring依赖项：

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法的客户端类（在本例中为CommandManager）中，要注入的方法需要以下形式的签名：

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，则动态生成的子类实现该方法。否则，动态生成的子类会覆盖在原始类中定义的具体方法。考虑以下示例:

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

id为commandManager的bean在需要myCommand bean的新实例时调用自己的createCommand()方法。如果确实需要部署myCommand bean，那么必须小心地将其作为原型部署。如果是单例，则每次返回myCommand bean的相同实例。

另外，在基于注解的组件模式中，您可以通过@Lookup注解声明一个查找方法，如下面的示例所示：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更常用的是，您可以依赖于根据查找方法声明的返回类型解析目标bean

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，通常应使用具体的stub实现声明此类带注解的查找方法，以使其与Spring的组件扫描规则（默认情况下抽象类会被忽略）兼容。 此限制不适用于显式注册或显式导入的Bean类。

> 访问作用域不同的目标bean的另一种方法是ObjectFactory / Provider注入点。 
>
> ServiceLocatorFactoryBean（在org.springframework.beans.factory.config包中）也很有用。

#### 任意方法替换

与查找方法注入相比，方法注入的一种不太有用的形式是能够用另一种方法实现替换托管bean中的任意方法。 您可以放心地跳过本节的其余部分，直到您真正需要此功能为止。

借助基于XML的配置元数据，您可以使用`replaced-method`元素将现有的方法实现替换为另一个。 考虑以下类，该类具有一个我们要覆盖的名为computeValue的方法：

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

一个实现`org.springframework.beans.factory.support.MethodReplacer`接口的类提供了新的方法定义，如以下示例所示：

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

用于部署原始类并指定方法重写的Bean定义类似于以下示例：

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以在`<replaced-method />`元素内使用一个或多个`<arg-type />`元素来指示被覆盖的方法的方法签名。 仅当方法重载且类中存在多个变体时，才需要对参数签名。 为了方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。 例如，以下所有都匹配java.lang.String：

```java
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过让您仅键入与参数类型匹配的最短字符串，此快捷方式可以节省很多输入。

## 1.5. Bean作用域

创建一个bean定义时，将创建一个配置，该配置用于创建该bean定义所定义的类的实例。 bean定义是配置的思想很重要，因为它意味您可以从一个配置中创建一个类的许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的作用域。 这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的作用域，而不必在Java类级别上bake（这里不知道如何翻译）对象的作用域。 可以将Bean定义为部署在多个作用域之一中。 Spring框架支持六个作用域，其中四个只有在使用Web感知的`ApplicationContext`时才可用。 您还可以创建自定义作用域。

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | （默认值）将每个Spring IoC容器的单个bean定义作用域限定为单个对象实例。 |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | 将单个bean定义的作用域限定为任意数量的对象实例。             |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | 将单个bean定义的作用域限定为单个HTTP请求的生命周期内。 也就是说，每个HTTP请求都有一个自己的bean实例，它是在单个bean定义的基础上创建的。 仅在web感知的Spring `ApplicationContext`上下文中有效。 |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | 将单个bean定义的作用域限定为HTTP `session`的生命周期内。  仅在web感知的Spring `ApplicationContext`上下文中有效。 |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | 将单个bean定义的作用域限定为`ServletContext`的生命周期内。  仅在web感知的Spring `ApplicationContext`上下文中有效。 |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | 将单个bean定义的作用域限定为`WebSocket`的生命周期内。  仅在web感知的Spring `ApplicationContext`上下文中有效。 |

> 从Spring 3.0开始，线程作用域可用，但默认情况下未注册。 有关更多信息，请参见[SimpleThreadScope](https://docs.spring.io/spring-framework/docs/5.3.3/javadoc-api/org/springframework/context/support/SimpleThreadScope.html)文档。 有关如何注册此或任何其他自定义作用域的说明，请参阅自定义作用域。

### 1.5.1. 单例作用域

一个单例bean只有一个共享实例被管理，所有对具有一个或多个与该bean定义匹配的ID的bean的请求都会导致Spring容器返回一个特定的bean实例。

换句话说，当您定义一个bean定义并将其定义为单例时，Spring IoC容器将为该bean定义所定义的对象创建一个实例。 该单个实例存储在此类单例bean的缓存中，并且对该命名bean的所有后续请求和引用都返回缓存的对象。 下图显示了单例作用域的工作方式：

![singleton](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/images/singleton.png)

Spring的单例bean概念与《Gang of Four (GoF)》一书中定义的单例模式不同。 GoF中的单例对对象的作用域进行硬编码，以使每个ClassLoader只能创建一个特定类的一个实例。 最好将Spring单例作用域描述为每个容器和每个bean。 这意味着，如果您在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且只有一个实例。 单例作用域是Spring中的默认作用域。 要在XML中定义单例bean，可以如以下示例所示定义bean：

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### 1.5.2. 原型作用域

每次请求特定bean时，非单例的原型作用域都会导致创建一个新的bean实例。 也就是说，该Bean被注入到另一个Bean中，或者您可以通过容器上的getBean（）方法调用来请求它。 通常，应将原型作用域用于所有有状态的Bean，将单例作用域用于无状态的Bean。

下面的图表说明了Spring原型作用域

![prototype](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/images/prototype.png)

(数据访问对象(DAO)通常不被配置为原型，因为典型的DAO不持有任何会话状态。重用单例图的核心对我们来说更容易。)

下面的示例在XML中将bean定义原型：

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相反，Spring不管理原型Bean的完整生命周期。 容器实例化，配置或组装原型对象，然后将其交给客户端，而无需保持对该原型实例的进一步记录。 因此，尽管无论在什么作用于下都会在所有对象上调用初始化生命周期回调方法，但对于原型，则不会调用已配置的销毁生命周期回调。 客户端代码必须清除原型作用域内的对象，并释放原型Bean拥有的昂贵资源。 为了使Spring容器释放原型作用下的bean所拥有的资源，请尝试使用自定义bean后置处理器，该处理器包含对需要清理的bean的引用。

在某些方面，Spring容器在原型作用域Bean方面的角色是Java new运算符的替代。 超过该点（也就是new之后）的所有生命周期管理必须由客户端处理。 （有关Spring容器中bean的生命周期的详细信息，请参阅生命周期回调。）

### 1.5.3. 包含原型bean引用的单例beans

当您使用对原型bean有依赖性的单例作用域Bean时，请注意，依赖关系在实例化时解析。 因此，如果将依赖的原型bean依赖项注入到单例bean中，则将实例化新的原型bean，然后将依赖项注入到单例bean中。 原型实例是曾经提供给单例bean的唯一实例。

但是，假设您希望单例作用域的bean在运行时重复获取原型作用域的bean的新实例。 您不能将原型作用域的bean依赖注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入仅发生一次。 如果在运行时不止一次需要原型bean的新实例，请参见[方法注入](###1.4.6. 方法注入)。

### 1.5.4. Request, Session, Application, and WebSocket 作用域

只有当您使用web感知的Spring ApplicationContext实现(如`XmlWebApplicationContext`)时，`request`,`session`, `application`, 和`websocket`作用域才可用。如果在常规的Spring IoC容器(如`ClassPathXmlApplicationContext`)中使用这些作用域，则会抛出一个未知bean作用域的`IllegalStateException`。

#### 初始Web配置

为了支持`request`,`session`,`application`,`websocket`这些基于web的bean作用域，需要在定义bean前做一些初始化配置（`singleton`,`prototype`标准作用域不是必须的）。

如和完成这些初始化步骤取决于你的Servlet环境。

如果您在 Spring Web MVC 中访问以上作用域 bean，实际上是在 Spring `DispatcherServlet` 处理的请求中，则不需要特殊设置。` DispatcherServlet` 已经暴露了所有相关的状态。

如果你用的是Servlet2.5的web容器，在Spring的`DispatcherServlet`外部处理请求（例如JSF或者Struts），你需要注册`org.springframework.web.context.request.RequestContextListener` （`ServletRequestListener`.Servlet 3.0+）,也可以用`WebApplicationInitializer`接口以编程方式实现，或者对于更老的容器来说，添加以下声明在`web.xml`文件中

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者可以用Spring的`RequestContextFilter`，这个过滤器映射取决于web应用的配置，所以必须合适得更改它，以下是一个web应用的部分过滤器配置：

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`、`RequestContextListener` 和 `RequestContextFilter` 都做完全相同的事情，即将 HTTP 请求对象绑定到为该请求提供服务的线程。 这使得request和session作用域的 bean 在调用链的更下游可用。 

#### Reqeust作用域

以下面的xml形式的bean定义举例：

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring 容器通过为每个 HTTP 请求使用 `loginAction` bean 定义来创建 `LoginAction` bean 的新实例。 也就是说， `loginAction` bean 的作用域是 HTTP 请求级别的。 您可以根据需要更改所创建实例的内部状态，因为从同一`loginAction` bean 定义创建的其他实例不会看到这些状态更改。 它们是针对单个请求的。 当请求完成处理时，该请求范围内的 bean 将被废弃。 

当使用注解驱动的组件或者Java配置时，`@ReqeustScope`注解可用于指定一个组件为`request`作用域，如下示例：

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

#### Session作用域

以下面的xml形式的bean定义举例：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring 容器通过在单个 HTTP 会话的生命周期中使用 ```userPreferences``` bean 定义创建 ```UserPreferences``` bean 的新实例。 换句话说，```userPreferences``` bean 作用域是 HTTP 会话级别。 与Reqeust作用域的 bean 一样，您可以根据需要更改所创建实例的内部状态，其他HTTP Session也在使用从相同 ```userPreferences``` bean 定义创建的实例不会看到这些状态更改 ，因为它们特定于单个 HTTP 会话。 当 HTTP 会话最终被丢弃时，作用域为该特定 HTTP 会话的 bean 也将被丢弃。

当使用注解驱动的组件或者Java配置时，`@SessionScope`注解可用于指定一个组件为`session`作用域，如下示例：

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```



#### Application作用域

以下面的xml形式的bean定义举例：

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring 容器通过为整个 Web 应用程序使用以上定义一次来创建 AppPreferences bean 的新实例。 也就是说，appPreferences bean 的范围在 ServletContext 级别，并存储为常规 ServletContext 属性。 这有点类似于 Spring 单例 bean，但在两个重要方面有所不同：它是每个 ServletContext 的单例，而不是每个 Spring 'ApplicationContext'（在任何给定的 Web 应用程序中可能有多个），并且它实际上是公开的，因此 作为 ServletContext 属性可见。

当使用注解驱动的组件或者Java配置时，`@ApplicationScope`注解可用于指定一个组件为`application`作用域，如下示例：

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

#### 依赖中的作用域bean

Spring IoC 容器不仅管理对象（bean）的实例化，还管理协作者（或依赖项）的连接。 如果您想将（例如）一个 HTTP 请求范围的 bean 注入到另一个生命周期更长的 bean 中，您可以选择注入一个 AOP 代理来代替该作用域的 bean。 也就是说，您需要注入一个代理对象，该对象公开与作用域对象相同的公共接口，但也可以从相关作用域（例如 HTTP 请求）中检索真实目标对象，并将方法调用委托给真实对象。

> 您还可以在作用域为单例的 bean 之间使用\<aop:scoped-proxy/>，然后引用通过可序列化的中间代理，因此能够在反序列化时重新获取目标单例 bean。
>
> 当针对原型的 bean 声明\<aop:scoped-proxy/> 时，共享代理上的每个方法调用都会导致创建一个新的目标实例，然后将调用转发到该实例。
>
> 此外，原型代理并不是以生命周期安全的方式从较短作用域访问 bean 的唯一方法。 您还可以将您的注入点（即构造函数或 setter 参数或自动装配字段）声明为 ObjectFactory<MyTargetBean>，从而允许每次需要时调用 getObject() 来按需检索当前实例 — 无需保留实例或单独存储。
>
> 作为扩展变体，您可以声明 ObjectProvider<MyTargetBean>，它提供了几个额外的访问变体，包括 getIfAvailable 和 getIfUnique。
>
> 此 JSR-330 变体称为 Provider，并与 Provider<MyTargetBean> 声明和每次检索尝试的相应 get() 调用一起使用。 有关 JSR-330 整体的更多详细信息，请参见[此处](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-standard-annotations )。



下例中的配置只有一行，但了解其背后的“why”以及“how”很重要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

要创建这样的代理，要将 \<aop:scoped-proxy/> 元素插入到作用域 bean 定义中（请参阅选择要创建的代理类型和基于XML配置模式）。 为什么在请求、会话和自定义范围级别范围内的 bean 定义需要\<aop:scoped-proxy/> 元素？ 考虑以下单例 bean 定义，并将其与您需要为上述范围定义的内容进行对比（请注意，以下 userPreferences bean 定义不完整）：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例 bean (userManager) 被注入了对 HTTP 会话范围 bean (userPreferences) 的引用。 这里的重点是 userManager bean 是一个单例：它在每个容器中被实例化一次，并且它的依赖项（在这种情况下只有一个，userPreferences bean）也只注入一次。 这意味着 userManager bean 只对完全相同的 userPreferences 对象（即最初注入它的那个对象）进行操作。

这不是您将生命周期较短的作用域 bean 注入到生命周期较长的作用域 bean（例如，将 HTTP 会话作用域的 bean 作为依赖项注入到单例 bean 中）时想要的行为。 相反，您需要一个 userManager 对象，并且，对于 HTTP 会话的生命周期，您需要一个特定于 HTTP 会话的 userPreferences 对象。 因此，容器创建一个对象，该对象公开与 UserPreferences 类完全相同的公共接口（理想情况下是一个 UserPreferences 实例的对象），它可以从作用域机制（HTTP 请求、会话等）中获取真正的 UserPreferences 对象 。容器将这个代理对象注入到 userManager bean 中，它不知道这个 UserPreferences 引用是一个代理。 在这个例子中，当一个 UserManager 实例在依赖注入的 UserPreferences 对象上调用一个方法时，它实际上是在调用代理上的一个方法。 然后代理从（在这种情况下）HTTP 会话中获取真实的 UserPreferences 对象，并将方法调用委托给检索到的真实 UserPreferences 对象。

因此，在将request和session作用域的 bean 注入对象时，您需要以下（正确且完整的）配置，如以下示例所示：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

#### 选择要创建的代理对象类型

默认情况下，当 Spring 容器为使用 \<aop:scoped-proxy/> 元素标记的 bean 创建代理时，将创建基于 CGLIB 的类代理。

> CGLIB 代理只拦截公共方法调用！ 不要在这样的代理上调用非公共方法。 它们不会委托给实际作用域的目标对象。

或者，您可以配置 Spring 容器，通过为 \<aop:scoped-proxy/> 元素的 `proxy-target-class` 属性的值指定 false 来为此类作用域 bean 创建标准的基于 JDK 接口的代理。 使用基于 JDK 接口的代理意味着您不需要应用程序类路径中的其他库来实现此类代理。 但是，这也意味着作用域 bean 的类必须至少实现一个接口，并且注入作用域 bean 的所有依赖都必须通过其接口之一引用 bean。 以下示例显示了基于接口的代理：

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，请参阅[代理机制](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#aop-proxying)。

### 1.5.5. 自定义作用域

bean 作用域机制是可扩展的。 您可以定义自己的作用域，甚至重新定义现有作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置的单例和原型作用域。

#### 创建自定义作用域

要将自定义作用域集成到 Spring 容器中，您需要实现 ```org.springframework.beans.factory.config.Scope```接口，这在本节中进行了描述。 有关如何实现您自己的作用域，请参阅 Spring Framework 本身提供的 Scope 实现和 [Scope](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/Scope.html) javadoc，它更详细地解释了您需要实现的方法。

Scope 接口有四种方法可以从作用域中获取对象，从作用域中移除它们，以及让它们被销毁。

例如，会话作用域实现返回会话作用域 bean（如果它不存在，则该方法在将它绑定到会话以供将来引用后返回该 bean 的一个新实例）。 以下方法从作用域返回对象：

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

例如，会话作用域实现从会话中删除会话作用域的 bean。 应该返回一个对象，但如果没有找到具有指定名称的对象，您可以返回 ```null```。 以下方法从作用域中删除对象：

```java
Object remove(String name)
```

以下方法注册了一个回调，当它被销毁或作用域中的指定对象被销毁时，该作用域应该调用该回调：

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关销毁回调的更多信息，请参阅 [javadoc](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback) 或 Spring scope实现。

以下方法获取作用域的会话标识符：

```java
String getConversationId()
```

这个标识符对于每个作用域都是不同的。 对于会话作用域的实现，此标识符可以是session标识符。

#### 使用自定义作用域

在您编写并测试一个或多个自定义 Scope 实现之后，您需要让 Spring 容器感知你的新作用域。 以下方法是向 Spring 容器注册新 Scope 的中心方法：

```java
void registerScope(String scopeName, Scope scope);
```

此方法在 ````ConfigurableBeanFactory``` 接口上声明，该接口可通过 Spring 附带的大多数具体 ```ApplicationContext``` 实现的 BeanFactory 属性获得。

```registerScope(..)``` 方法的第一个参数是与作用域关联的唯一名称。 Spring 容器本身中此类名称的示例是```singleton```和```prototype```。 ```registerScope(..)``` 方法的第二个参数是您希望注册和使用的自定义 Scope 实现的实际实例。

‎假设已编写了自定义实现，然后按照下一个示例进行注册。‎

> 下一个示例使用 SimpleThreadScope，它包含在 Spring 中，但默认情况下未注册。对于您自己的自定义 Scope 实现，使用方法是相同的。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以创建符合自定义 Scope 规则的 bean 定义，如下所示：

```xml
<bean id="..." class="..." scope="thread">
```

使用自定义 Scope 实现，您不仅限于以编程方式注册。您还可以使用 ```CustomScopeConfigurer``` 类以声明方式进行 Scope 注册，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

> 当您将 \<aop:scoped-proxy/> 放在 FactoryBean 实现中时，作用域的是工厂 bean 本身，而不是从 getObject() 返回的对象。

## 1.6. 自定义bean性质

Spring Framework提供了许多接口，可以用于自定义Bean的性质。本节将其分组如下：

- Lifecycle Callbacks
- ApplicationContextAware & BeanNameAware
- 其他Aware接口

### 1.6.1. Lifecycle 回调

要与容器管理的bean生命周期交互，你可以实现`InitializingBean`和`DisposableBean`接口，容器会在前者下调用`afterPropertiesSet()`，在后者调用`destroy()`方法，让bean在初始化和销毁的时候执行某些操作。

> :bulb: JSR-250 @PostConstruct和@predestroy注解通常被认为是在现代Spring应用中接收生命周期回调的最佳实践。使用这些注解意味着您的bean不耦合到Spring特定的接口。
>
> 如果您不想使用JSR-250注解，又需要解耦，请考虑使用`init-method`和`destroy-method`的 bean定义元数据。

在内部，Spring框架使用`BeanPostProcessor`实现来处理它可以找到的任何回调接口并调用适当的方法。如果您需要自定义功能或其他生命周期行为,Spring则不会默认提供，您可以自己实现`BeanPostProcessor`。有关更多信息，请参阅[容器扩展点](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-extension)。

除了初始化和销毁回调之外，Spring托管对象还可以实现`Lifecycle`接口，以便这些对象可以参与到容器启动和关闭过程，作为容器自己的生命周期的驱动。

#### Initialization初始化回调

`org.springframework.Beans.Factory.InitializationBean`接口允许bean在容器中设置完bean上的所有必需属性后执行初始化工作。`InitializingBean`接口定义了一个方法：

```java
void afterPropertiesSet() throws Exception;
```

**我们建议您不要使用`InitializingBean`接口，因为它不必要地将代码耦合到Spring。或者，我们建议使用`@PostConstruct`注解或指定POJO初始化方法。在基于XML的配置元数据的情况下，您可以使用init-method属性指定具有void No-Argument签名的方法的名称。使用Java配置，您可以使用@Bean的initmethod属性。**请参阅[接收生命周期回调](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-java-lifecycle-callbacks)。考虑以下示例：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

此示例与以下示例几乎完全相同:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，第一个示例代码没有与Spring耦合。

#### Destruction销毁回调

实现`org.springframework.beans.factory.disposablebean`接口允许bean当包含它的容器被销毁时会收到回调。`DisposableBean`接口定义了一个方法：

```java
void destroy() throws Exception;
```

**我们建议您不要使用`DisposableBean`接口，因为它不必要地将代码耦合到Spring。或者，我们建议使用`@PreDestroy`注解或指定bean定义支持的泛型方法。在基于XML的配置元数据的情况下，您可以在<bean/>中使用destroy-method属性。使用Java配置，您可以使用@Bean的destroyMethoad属性。**请参阅[接收生命周期回调](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-java-lifecycle-callbacks)。考虑以下示例：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

此示例与以下示例几乎完全相同:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，第一个示例代码没有与Spring耦合。

> :bulb: 您可以分配<bean>元素的destroy-method属性一个特殊（推断）值，该值指示Spring在特定bean类上自动检测pulic `close`或者`shutdown`。（任何实现`java.lang.autoclosable`或`java.io.closable`的类都会匹配。）您还可以在<bean>元素的default-destroy-method属性上设置此特殊（推断的）值以应用此行为到一组bean上（请参阅默认初始化和销毁方法）。请注意，这是Java配置的默认行为。

#### 默认初始化和销毁方法

当不通过实现Spring特定的`InitializingBean`和`DisposableBean`定义的接口来完成初始化和销毁方法回调时，通常会使用像`init()`，`initialize()`，`dispose()`等名称编写方法。理想情况下，这些生命周期回调方法的名称在项目中应该标准化，以便所有开发人员使用相同的方法名称并确保一致性。

您可以配置Spring容器在每个bean上查找指定命名的初始化和销毁回调方法。这意味着您作为应用程序开发人员，不用为每个bean配置`init-methoad="init"`的属性，就可以编写名为`init()`的初始化回调方法。Spring IoC容器在创建bean时调用该方法（并根据先前描述的标准生命周期回调约定）。此功能还确保了初始化和销毁方法回调的命名一致约定。

假设您的初始化回调方法名为`init()`，您的销毁回调方法名为`destroy()`。然后，您的类类似以下示例：

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后，您可以在类似于以下内容的bean定义中使用该类：

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

顶级<bean />元素属性上的`default-init-method`属性的存在会导致Spring IoC容器识别Bean类中称为`init`的方法作为初始化方法回调。创建并组装Bean时，如果Bean类具有这样的方法，则会在适当的时间调用它。

您可以类似地通过在顶级<bean />元素上使用`efault-destroy-method`属性来配置默认销毁回调。

当现有bean类已经有与约定回调方法有差异的方法时，可以通过使用<bean />的`init-method`和`destroy-method`属性指定方法名称来覆盖默认值。

Spring 容器保证在向bean注入所有依赖项之后立即调用配置的初始化回调方法。因此，在原始bean上才会调用初始化回调，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用具有其拦截链的AOP代理。如果目标bean和代理单独定义，则您的代码甚至可以绕过代理直接与原始目标bean交互。因此，将拦截器应用于`init`方法是矛盾的，因为这样做会将目标bean的生命周期耦合到其代理或拦截器，并在代码与原始目标bean直接交互时留下奇怪的语义。

#### 组合生命周期机制

截至Spring 2.5，您有三种方式控制bean的声明周期，你可以组合这些机制来控制bean的声明周期：

- `InitializingBean`和`DisposableBean`回调接口
- 自定义`init()`和`destroy`方法
- `@PostContruct`和`@PreDestroy`注解

> 如果为bean配置了多个生命周期机制，并且每个机制配置了不同名称的方法，则每个配置的方法按照列出的顺序运行。但是，如果配置了相同名称的方法 - 例如，初始化方法`init()` - 对于这些生命周期机制中的多个，该方法只会运行一次。

为同一bean配置的多个生命周期机制，r若具有不同的初始化方法，按照如下所示顺序执行：

1. `@PostConstruct`注释的方法
2. `InitializingBean`回调接口中的`afterPropertiesSet()` 方法
3.  自定义的 `init()` 方法

销毁方法以相同的顺序调用：

1. `@PreDestroy`注释的方法
2. `DisposableBean `回调接口中的`destroy()` 方法
3.  自定义的 `destroy()` 方法

#### 启动和停止回调

`Lifecycle`接口为对象定义了任何具有其自身生命周期要求的基本方法（例如启动和停止某些后台进程）：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现生命周期接口。然后，当`ApplicationContext`本身接收开始和停止信号（例如，在运行时进行停止/重启）时，它将这些调用级联到该上下文中定义的所有生命周期实现。它通过将其委派给`LifecycleProcessor`来实现这一点，如下所示所示：

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，`LifecyCleProcessor`本身是`Lifecycle`接口的扩展。它还添加了另外两个方法，将在应用上下文刷新和关闭时调用。

> :bulb:请注意，常规`org.springframework.context.Lifecycle`接口是一个简单的约定，用于显式启动和停止通知，并不意味着上下文刷新时自动启动。对特定bean的自动启动（包括启动阶段）进行细粒度控制，请考虑实现`org.springframework.context.SmartLifecycle`。
>
> 此外，请注意，销毁前不保证发起停止通知。一般在停止时，所有`Lifecycle`实现类首先在常规销毁回调传播之前接收停止通知。但是，在上下文热刷新或停止刷新尝试期间，只调用销毁方法。

启动和关闭调用的顺序很重要。如果在任何两个对象之间存在依赖关系，则需要依赖的将于其依赖项启动后启动，并且在其依赖项停止前停止。但是有时，直接依赖是未知的。您可能只知道某种类型的对象应该在另一个类型的对象之前启动。在这些情况下，`SmartLifecycle`接口定义了另一个选项，即在其父接口`Phased`定义的`getPhase()`方法。以下是该接口的定义：

```java
public interface Phased {

    int getPhase();
}
```

以下是`SmartLifecycle`接口的定义：

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，顺位最低的对象首先启动。停止时，顺序相反。因此，一个实现`SmartLifecycle`且其`getphase()`方法返回`Integer.MIN_VALUE`的对象将是第一个启动和最后停止。也就是说，返回`Integer.MAX_VALUE`的值的将最后启动并首先停止（可能是因为它取决于要运行的其他进程）。很重要的是要知道不实现`SmartLifecycle`的任何普通`Lifecycle`实例的默认顺位值是0。因此，任何负的顺位值表示对象应该在这些标准组件之前启动（并在其之后停止）。反之亦然。

`SmartLifecycle`定义的stop方法接受一个线程回调。在该实现的关闭进程完成后，任何实现必须调用回调线程的`run()`方法。这使得在必要时启用异步关闭，因为`LifecycleProcessor`接口的默认实现`DefaultLifeCycleProcessor`，在每个阶段内的对象组等待定义的超时时间后调用该回调。默认的超时时间为30秒。您可以通过在上下文中定义名为`lifecycleProcessor`的bean来覆盖默认生命周期处理器实例。如果您只想修改超时时间，以下定义就足够了：

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，`LifecyCleProcessor`接口也定义了刷新和关闭上下文的回调方法。后者驱动了停止过程，仿佛`stop()`被显示调用一样，但它发生在上下文关闭的过程中。另一方面，“refresh”回调可以启用`SmartLifecycle` bean的另一个功能。刷新上下文（在实例化并初始化所有对象后）时，调用该回调。此时，默认生命周期处理器检查每个`SmartLifecycle`对象的`isAutostArtup()`方法返回的布尔值。如果为true，则在刷新时启动该对象，而不是等待上下文或其自己的`start()`方法被显式调用（与上下文刷新不同，标准上下文实现不会自动启动）。`phase`值和依赖关系确定如前所述的启动顺序。

#### 在非web应用中优雅地关闭Spring Ioc容器

> 本节仅适用于非Web应用程序。Spring的基于Web的ApplicationContext实现已有相关代码，能在Web应用程序关闭时优雅地关闭Spring IoC容器。

如果在非Web应用程序环境中使用Spring的IoC容器（例如，在富客户端桌面环境中），请使用JVM注册关闭hook。这样做确保了一个优雅的停止，并在单例bean上调用相关的销毁方法，以便释放所有资源。您必须正确配置和实现这些销毁回调。

要注册关闭hook，请调用在`ConfigurableApplicationContext`接口上声明的`registerShutdownHook()`方法，如以下示例显示：

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```



### 1.6.2. `ApplicationContextAware`和`BeanNameAware`

当`ApplicationContext`创建实现`org.springframework.context.applicationContextaware`接口的对象实例时，将提供该`ApplicationContext`的引用。以下列表显示了`ApplicationContextaware`接口的定义：

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以通过编程方式操作创建它们的`ApplicationContext`，通过`ApplicationContext`接口或通过将引用转换为该接口的已知子类（例如公开其他功能的`ConfigurableApplicationContext`）。一个作用就是检索其他bean。有时这种能力很有用。但是，一般而言，您应该避免使用它，因为它将代码与Spring耦合，并且不遵循控制反转的风格(依赖作为bean的属性提供)。`ApplicationContext`的其他方法提供对文件资源的访问，发布应用程序事件和访问`MessagesSource`。这些附加功能在[ApplicationContext的附加功能](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#context-introduction)中描述。

自动注入是获得对`ApplicationContext`的引用的另一种选择。传统的构造函数注入和按类型注入模式（如1.4.5所述）可以分别为构造函数参数或Setter方法参数分别提供ApplicationContext的依赖。更灵活的方式，包括自动注入字段和多个参数方法的能力，请使用基于注解的自动注入功能。如果您在需要的地方使用了`@Autowired`注解，期望类型的`ApplicationContext`将自动注入到字段，构造函数或方法参数中。有关更多信息，请参阅[使用@Autowired](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-autowired-annotation)。

当`ApplicationContext`创建一个实现`org.springFramework.Beans.Factory.BeannameAware`接口的类时，该类会获得跟它自身定义相关的名称。以下列表显示了`Beannameaware`接口的定义：

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

该方法回调将会在普通bean属性设值后，初始化回调之前(如InitializingBean、afterPropertiesSet、自定义初始化方法)调用。

### 1.6.3. 其他`Aware`接口

除了`ApplicationContextaware`和`Beannameaware`之外，Spring还提供了更多的`Aware`回调接口，让bean向容器指明它们需要依赖某个明确的基础架构。作为一般规则，名称表示依赖类型。如下表格4(未翻译)总结了最重要的`Aware`接口：


| Name                             | Injected Dependency                                          | Explained in…                                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ApplicationContextAware`        | Declaring `ApplicationContext`.                              | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | Event publisher of the enclosing `ApplicationContext`.       | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#context-introduction) |
| `BeanClassLoaderAware`           | Class loader used to load the bean classes.                  | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-class) |
| `BeanFactoryAware`               | Declaring `BeanFactory`.                                     | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-aware) |
| `BeanNameAware`                  | Name of the declaring bean.                                  | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-aware) |
| `LoadTimeWeaverAware`            | Defined weaver for processing class definition at load time. | [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | Configured strategy for resolving messages (with support for parametrization and internationalization). | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX notification publisher.                           | [Notifications](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | Configured loader for low-level access to resources.         | [Resources](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#resources) |
| `ServletConfigAware`             | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/web.html#mvc) |
| `ServletContextAware`            | Current `ServletContext` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/web.html#mvc) |

请注意，使用这些接口你的代码将与Spring API耦合，并且不遵循控制反转的风格。因此，我们推荐它们在需要对容器进行编程访问的基础架构bean中使用。



## 1.7. Bean定义继承

Bean定义可以包含大量的配置信息，包括构造函数参数，属性值和特定于容器的信息，例如初始化方法，静态工厂方法名称等。子bean定义从父定义继承配置数据。子定义可以根据需要覆盖某些值或添加其他值。使用父、子bean定义可以减少配置。实际上，这是模板模式的一种形式。

如果您以编程方式使用`ApplicationContext`接口，则子Bean定义由`ChildBeanDefinition`类表示。大多数用户不会在这种级别使用它们。相反，他们在诸如`ClassPathxMlApplicationContext`等类中声明地配置bean定义。使用基于XML的配置元数据时，可以使用`parent`属性指定子bean定义，指定父bean作为此属性的值。以下示例显示了如何做：

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  1️⃣
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果没有指定，则子bean使用父bean的定义，但也可以覆盖它。在后一种情况下，子bean必须与父bean兼容（即它必须接受父属性值）。

子bean定义继承作用域，构造函数参数值，属性值和从父类覆盖的方法，并可选择添加新值。您指定的任何作用域，初始化方法，销毁方法或静态工厂方法设置将会覆盖父类中对应设置。

剩余的设置始终从子定义中引用：依赖，自动装配模式，依赖关系检查，单例和懒加载。

前面的示例通过使用`abstract`属性将父bean定义显式标记为抽象类。如果父定义未指定类，则父bean定义`abstract`属性是必须的，如以下示例显示：

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父bean无法自动化，因为它是不完整的，并且也明确标记为抽象。当定义是抽象的时候，它仅用为纯模板bean定义，它用作子定义的父定义。尝试自己使用这样一个抽象父bean，通过将其作为另一个bean的ref属性，否则使用父bean id显示执行getBean()调用将返回错误。同样，容器的内部预留`preInstantiateSingletons()`方法忽略了被定义为抽象的bean定义。

> :information_source:ApplicationContext默认预先实例化所有单例bean。因此，如果您有打算仅用作为模板的（父）bean定义，并且此定义定义了类，则必须确保将`abstract`属性设置为true，否则应用上下文将（尝试）预实例化抽象类。

## 1.8. 容器扩展点

通常，应用程序开发者不需要子类化ApplicationContext实现类。相反，Spring IoC容器可以通过特殊的集成接口实现扩展。接下来的几节将介绍这些集成接口。

###  1.8.1. 用`BeanPostProcessor`自定义Beans

`BeanPostProcessor`接口定义了可以实现的回调方法，以提供自己的（或覆盖容器的默认）实例化逻辑，依赖解析逻辑等。如果要在Spring容器完成实例化，配置和初始化Bean后实现某些自定义逻辑，可以插入一个或多个自定义`BeanPostProcessor`实现。

您可以配置多个`BeanPostProcessor`实例，您可以通过设置`order`属性来控制这些BeanPostProcessor实例运行的顺序。只有当`BeanPostProcessor`实现`Ordered`接口时，才能设置此属性。如果您编写自己的`BeanPostProcessor`，则应考虑实现`Ordered`接口。有关详细信息，请参阅[BeanPostProcessor](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)和[Ordered](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/core/Ordered.html)的Javadoc。另请参阅关于[BeanPostProcessor实例的程序化注册的说明](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-programmatically-registering-beanpostprocessors)。

> :information_source:BeanPostProcessor实例在Bean（或Object）实例上运行。也就是说，Spring IoC容器实例化了一个bean实例，然后Beanbostprocessor实例执行它们的回调方法。
>
> BeanPostProcessor实例作用域是容器。仅当您使用容器层次结构时才起作用。如果在一个容器中定义BeanPostProcessor，则它仅在该容器中处理bean。换句话说，即使两个容器都是相同层次结构的一部分，在一个容器中定义的bean也不会被另一个容器中定义的BeanPostProcessor处理。

该`org.springframework.beans.factory.config.BeanPostProcessor`接口包括正好两个回调方法。当此类被容器注册为后置处理器时，对于每个容器创建的bean实例，后处理器在容器初始化方法（例如`InitializationBean.afterPropertiesset()`或任何声明的`init`方法）被调用前，并在任何bean初始化回调之后调用。后处理器可以对bean实例执行任何操作，包括完全忽略回调。bean后置处理器通常检查回调接口，或者它可能会将bean包装为代理。一些Spring AOP基础架构类就是bean后处理器的实现，以提供代理包装逻辑。

ApplicationContext将自动检测在配置元数据中定义的实现BeanPostProcessor接口的任何Bean。ApplicationContext将这些bean注册为后处理器，以便稍后可以在bean创建前被调用。Bean后处理器能以与任何其他bean相同的方式部署在容器中。

注意，在配置类上使用`@Bean`工厂方法声明`BeanPostProcessor`时，工厂方法的返回类型应该是实现类本身或至少是org.springframework.beans.factory.config.beanpostProcessor接口，以清楚地表示该bean的后处理器性质。否则，ApplicationContext无法在完全创建它之前通过类型自动探测到它。由于需要提前实例化BeanPostProcessor以便在上下文中初始化其他Bean时应用处理器，因此该早期类型检测至关重要。

> :information_source:以编程方式注册BeanPostProcessor实例
>
> 虽然BeanPostProcessor注册的推荐方法是通过ApplicationContext自动检测（如前所述），但您可以使用ConfigurableBeanFactory的`addBeanPostProcessor`方法进行编程方式注册。当您需要评估条件逻辑或甚至用于在层次结构中的上下文中复制bean后处理器时，这可能很有用。但请注意，`Ordered`接口对编程添加的BeanPostProcessor实例无效。在这里，执行顺序就是注册顺序。还要注意，无论任何指定的顺序如何，编程方式注册的BeanPostProcessor实例都会在通过自动检测注册的那些处理器之前之前执行。



> :information_source:BeanPostProcessor实例和AOP自动代理
>
> 实现BeanPostProcessor接口的类是特殊的，容器会特殊处理。所有BeanPostProcessor实例和它们直接引用的Bean，在启动时都会作为ApplicationContext的特殊启动阶段的一部分。接下来，所有BeanPostProcessor实例都以指定的顺序注册，并应用于容器中的所有其他bean。由于AOP自动代理实现为BeanPostProcessor本身，因此BeanPostProcessor实例和它们直接引用的Bean都没有资格自动代理，因此，它们没有切面。
>
> 如果您使用autowiring或@Resource（可能autowiring失败）将beans装配到BeanPostProcessor中，则在搜索类型匹配的依赖候选时，Spring可能会得到错误的Bean，因此使其无法被自动代理或被其他后处理器处理。例如，如果您有被@Resource注解标注的依赖项，如字段或setter方法名与已声明名称的bean不直接对应且没有使用name属性的话，Spring搜索其他bean以按类型匹配它们。

以下示例显示如何在ApplicationContext中编写，注册和使用BeanPostProcessor实例

#### 例：Hell World, `BeanPostProcessor` -style

第一个例子说明了基本用法。该示例显示了自定义BeanPostProcessor实现，该实现调用每个bean的toString()方法，它由容器创建并将生成的字符串打印到系统控制台。

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

以下beans元素使用InstantialTracingBeanPostProcessor：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

请注意InstantialTracingBeanPostProcessor是如何被定义的。它甚至没有名字，而且因为它是一个bean，它可以像任何其他bean一样依赖注入。（前面的配置还定义了由Groovy脚本备份的bean。Spring动态语言支持在[此章节](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/languages.html#dynamic-language)中详述。）

以下Java应用程序运行前面的代码和配置：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

前面应用程序的输出类似于以下内容：

```java
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

#### 例：`RequiredAnnotationBeanPostProcessor`

使用回调接口或注解连接自定义BeanPostProcessor实现是扩展Spring IoC容器的常见方法。一个例子是Spring的`ResertandannotationBeanPostProcessor` - 一个Spring实现的BeanPostProcessor，它确保用（任意）注解标记的bean上的依赖注入属性实际上（被配置为)有值。

### 1.8.2. 使用`BeanFactoryPostProcessor`自定义配置元数据

我们查看的下一个扩展点是`org.springframework.beans.factory.config.BeanfactoryPostProcessor`。此接口的语义类似于BeanPostProcessor的语义，其中一个主要区别：BeanFactoryPostProcessor在操作的是Bean配置元数据。也就是说，Spring IoC容器允许BeanFactoryPostPostProcessor读取配置元数据，并且在容器实例化BeanFactoryPostProcessor实例之外的任何Bean之前可能会更改它。

您可以配置多个`BeanFactoryPostProcessor `实例，您可以通过设置`order`属性来控制这些BeanFactoryPostProcessor实例运行的顺序。只有当`BeanFactoryPostProcessor `实现`Ordered`接口时，才能设置此属性。如果您编写自己的`BeanFactoryPostProcessor `，则应考虑实现`Ordered`接口。有关详细信息，请参阅[BeanFactoryPostProcessor](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)和[Ordered](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/core/Ordered.html)的Javadoc。

> :information_source:如果要更改实际Bean实例（即，从配置元数据创建的对象），则需要使用BeanPostProcessor）。虽然技术上可以在BeanFactoryPostProcessor中使用bean实例（例如，通过使用BeanFactory.getBean()），但这样做会导致更早的bean实例化，违反标准容器生命周期。这可能导致负副作用，例如绕过Bean后处理。
>
> BeanFactoryPostProcessor实例作用域是容器。仅当您使用容器层次结构时才起作用。如果在一个容器中定义BeanFactoryPostProcessor，则它仅在该容器中处理bean元数据。换句话说，即使两个容器都是相同层次结构的一部分，在一个容器中bean的元数据也不会被另一个容器中定义的BeanFactoryPostProcessor处理。

将自动运行在ApplicationContext中声明的Bean工厂后处理器，以便将更改应用于定义容器的配置元数据。Spring包括许多预定义的Bean工厂后处理器，例如`PropertyOverrideConfigurer`和`PropertySourcesPlaceholderConfigurer`。您还可以使用自定义BeanFactoryPostProcessor - 例如，注册自定义属性编辑器。

ApplicationContext会自动检测部署在它中的任何实现了BeanFactoryPostPostPostPostor接口的Bean。它在适当的时间使用这些作为bean工厂后处理器的bean。您可以按照任何其他bean的方式部署这些后处理器bean。

> :information_source:与BeanPostProcessor一样，您通常不希望配置BeanFactoryPostProcessors为延迟初始化。如果没有其他bean引用Bean(Factory)PostProcessor，则根本不会立即实例化。因此，将其标记为延时初始化将被忽略，并且即使在<bean />元素声明上将`default-lazy-init`属性设置为true，Bean(Factory)PostProcessor也会立即实例化。

#### 例：类名称替换`PropertySourcesPlaceholderConfigurer`

您可以通过`PropertySourcesPlaceholderConfigurer `使用标准Java `Properties`格式在单独文件中从Bean定义中外部化属性值。这样做使得部署应用程序的人能够自定义特定于环境的属性，例如数据库URL和密码，而无需担心修改容器的主要XML定义文件带来的复杂性或风险。

考虑以下基于XML的配置元数据片段，其中定义了具有占位符值的数据源：

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

该示例显示了从外部属性文件配置的属性。在运行时，`PropertySourcePlaceHolterConfigurer`应用于替换数据源的某些属性的元数据。要替换的值被指定为$ {property-name}的占位符形式，它遵循Ant和log4j和JSP EL 风格。

实际值来自标准Java  `Properties`格式的另一个文件：

```properties
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

因此，$ {jdbc.username}字符串在运行时替换为'sa'，属性文件中其他占位符值也一样按键值匹配替换。`PropertySourcesPlaceholderConfigurer`检查在Bean定义的大多数属性中的占位符。**此外，您可以自定义占位符前缀和后缀。**

使用Spring 2.5中引入的上下文命名空间，您可以使用专用配置元素配置属性占位符。您可以在`location`属性中提供一个或多个位置作为逗号分隔的列表，如以下示例显示：

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

该`PropertySourcesPlaceholderConfigurer`不仅会在你指定的`Properties`文件中查找。默认情况下，如果不能在指定的属性文件中找到属性，它会针对Spring `Environment`属性配置和普通的Java  `System`属性配置进行查找。

> :bulb:您可以使用PropertySourcePlaceHolderConfigurer来替换类名，当您必须在运行时选择特定实现类时有用。以下示例显示了如何执行此操作：
>
> ```xml
> <bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
>     <property name="locations">
>         <value>classpath:com/something/strategy.properties</value>
>     </property>
>     <property name="properties">
>         <value>custom.strategy.class=com.something.DefaultStrategy</value>
>     </property>
> </bean>
> 
> <bean id="serviceStrategy" class="${custom.strategy.class}"/>
> ```
>
> 如果在运行时无法在运行时解析到有效类别，则在将要创建时（对于非懒加载bean，ApplicationContext的preInstantiateSingletons()阶段），Bean解析失败。

#### 例：`PropertyOverrideConfigurer`

PropertyOverrideConfigurer，另一个bean factory后处理器，类似于PropertySourcePlaceHolderConfigurer，但与后者不同，原始定义可以具有默认值或根本没有值作为bean的属性。如果覆盖的Properties文件没有某些bean属性的条目，则使用默认上下文定义。

请注意，bean定义无法感知被覆盖，因此覆盖的配置应用于XML定义文件不会显而易见。如果有多个PropertyOverrideConfigurer实例定义相同的bean属性的不同值，由于覆盖机制只会使用最后一个。

属性文件配置行采用以下格式：

```properties
beanName.property=value
```

以下列表显示了格式的示例：

```properties
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

此示例文件可以与包含一个名为dataSource的bean的容器定义一起使用，其中包含driver和url属性。

还支持复合属性名称，只要除了被覆盖的最终属性之外的路径上的的每个组件都是非空的（可能由构造函数初始化）。在以下示例中，tom bean的fred属性的bob属性的sammy属性设置为常量123：

```properties
tom.fred.bob.sammy=123
```

> :information_source:指定的覆盖值始终是字面量。它们不会翻译成bean引用。此约定还适用于XML Bean定义中的原始值指定为bean引用时。

在Spring 2.5中引入的上下文命名空间，可以使用专用配置元素配置属性覆盖，如以下示例显示：

```xml
<context:property-override location="classpath:override.properties"/>
```



### 1.8.3. 使用`FactoryBean`自定义实例化逻辑

本身是工厂的对象可以实现`org.springframework.beans.factory.FactoryBean `接口。

`FactoryBean`接口是Spring IoC容器的实例化逻辑的可插拔点。如果您有复杂的初始化代码，则在Java中可以比冗长的（可能）XML更好地表达，您可以创建自己的`FactoryBean`，在该类中写入复杂的初始化逻辑，然后将您的自定义`FactoryBean`插入容器中。

FactoryBean接口提供三个方法：

- `Object getObject()`：返回此工厂创建的对象的实例。实例可能会被共享，具体取决于此出厂是否返回单例或原型。

- `boolean isSingleton()`：如果FactoryBean返回单例bean此方法返回true否则返回false。

- `Class getObjectType()`：返回`getObject()`方法返回的对象类型，如果类型不能提前推断则返回`null`。

`FactoryBean`思想和接口用于Spring框架内的许多地方。spring内部就有`FactoryBean`接口的50多个实现。

当您需要询问实际FactoryBean实例而不是它生成的bean时，请在调用ApplicationContext的getBean()方法时使用and符号（＆）前缀。对于给定的FactoryBean，调用容器上的getBean("myBean")返回FactoryBean产生的bean，而调用getBean("&myBean")返回FactoryBean实例本身。



## 1.9. 基于注解的容器配置

> 基于注解的配置是否比基于XML的配置更好？
>
> 基于注解的配置引出了此问题。长话短说是“得看情况”。也就是说，每种方法都有其利弊。而且通常由开发人员决定哪种策略更适合他们。由于它们的定义方式，注解在他们的声明中提供了上下文，所以配置更短更简练。但是，XML配置的优势是不侵入源代码或不需要重新编译它们。一些开发人员喜欢注解的方式，而其他开发人员则认为被注解的类不再是POJO，而且这会使得配置变得分散，更难控制。
>
> 无论哪种选择，Spring都可以兼容这两种风格，甚至将它们混合在一起。值得注意的是，通过其[JavaConfig](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-java)选项，Spring让注解以非侵入性方式使用，而无需接触目标组件源代码，并且在工具方面，所有配置样式都支持[Eclipse的Spring Tools](https://spring.io/tools)支持。



基于注解的配置提供了XML配置的替代方法，它依赖于要装配的组件而不是用尖括号声明的字节码元数据。开发者通过在相关类，方法或字段声明上使用注解，将配置移动到组件类自身中而不是使用XML。如示例[RequiredAnnotationBeanPostProcessor](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-extension-bpp-examples-rabpp)，使用BeanPostProcessor与注解结合是扩展Spring IoC容器的常见方法。例如，Spring 2.0引入了@Required注解，使得强制依赖属性成为可能。Spring 2.5使得可以遵循相同的一般方法来驱动Spring的依赖注入。本质上，@Autowired注释提供了相同的能力，如[Autowiring](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-factory-autowire)所述，但具有更细粒度的控制和更广泛的适用性。Spring 2.5还增加了对JSR-250注解的支持，例如@PostConstruct和@PreDestroy。Spring 3.0添加了包含在`javax.inject`中的支持JSR-330（Java依赖注入）的注解，例如`@Inject`和`@Named`。有关这些注解的详细信息可以在[相关章节](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-standard-annotations)中找到。

> :information_source:基于注解的注入在XML注入之前进行。因此，基于XML的配置覆盖了通过两种方法属性的装配。

当然，您可以将它们注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包含以下标签来隐式注册（注意包含context命名空间）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

(隐式注册的后处理器包括[`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html), [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html), [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html),[`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html), [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html), [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html), [`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html).)

> :information_source:<context:annotation-config />仅查找在定义它的同一应用程序上下文中的注解。这意味着，如果将<context:annotation-config />放在DispatcherServlet的WebApplicationContext中，则只会扫描Controller中的@Autowired 注解的bean。有关更多信息，请参阅[DispatcherServlet](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/web.html#mvc-servlet)。



### 1.9.1. @Required

@Required注解适用于bean属性setter方法，如下例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注解表示必须通过Bean定义中的显式属性值或通过自动装配在配置bean时填充bean的属性。如果尚未填充受注解影响的bean属性，则该容器会抛出异常。这允许更早的失败，避免了后续的NullPointerException等。**我们仍然建议您将断言放入Bean类本身（例如，init方法）**。这样做即使在容器外部使用类时，也可以拿到所需的引用和值。

> :information_source:@Required在Spring Framework 5.1中正式过时，支持使用构造函数注入所需的设置（或InitializingBean.afterPropertiesSet()的自定义实现以及Bean属性的setter实现）。

### 1.9.2. 使用@Autowired

> :information_source:JSR 330的@Inject注解可以用于代替本节中包含的示例中的Spring @Autowired注释。有关详细信息，请参阅[此处](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans-standard-annotations)。

你可以应用`@Autowired`注解于构造器，就像下面这样：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> :information_source:截至Spring Framework 4.3，如果目标bean仅定义一个构造函数，则不再需要在此构造函数上用@Autowired注解。但是，如果有多个构造函数并且没有主/默认构造函数，则至少有一个构造函数必须用@Autowired注解，以指示容器使用哪一个构造器。

你也可以像下面这样应用`@Autowired`注解于传统setter方法：

```JAVA
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

你还可以像下面这样应用`@Autowired`注解于任意名称和多参数的方法：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

你也可以将@Autowired应用于字段，甚至将其与构造函数混合，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> :bulb:确保您的目标组件（例如，MovieCatalog或CustomerPreferenceDao）和用@ Autowired注解的注入点的类型声明一致。否则，运行时可能因类型不匹配而注入失败。
>
> 对于通过类路径扫描找到的XML定义的bean或组件类，容器通常预先知道确切的类型。但是，对于@Bean工厂方法，您需要确保声明的返回类型是足够明确的。对于实现多个接口或可能由其实现类型引用的组件，考虑在工厂方法上声明最具体的返回类型（至少跟注入点需要的类型一样）。

您还可以将@Autowired注解添加到期望得到该类型数组的字段或方法上，让Spring从ApplicationContext返回所有这种类型的bean，如下所示：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

类型集合也是一样：

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

> :bulb:如果您希望以特定顺序排序数组或列表中的元素，您的目标bean可以实现`org.springframework.core.Ordered`接口或使用`@Order`或标准`@Priority`注解。否则，它们的顺序跟容器中定义的目标bean的注册顺序一样。
>
> 您可以针对单个bean定义（如果有多个使用相同的bean类的定义）在目标类级别和@Bean方法中声明@Order注解。@Order值可能会影响注入点的优先级，但请注意它们不会影响单例的启动顺序，这是由依赖关系和@DependsOn声明决定的问题。
>
> 请注意，标准`javax.annotation.Priority`注解在@Bean级别不可用，因为它无法在方法中声明。它的语义跟通过@Order值与每种类型的单个bean上的@Primary组合类似。

只要预期的key类型为String，即使是Map实例也可以自动注入。映射值包含预期类型的所有bean，键包含相应的bean名称，如以下示例所示：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，当没有匹配的候选bean可用于给定注入点时，自动注入失败。在声明的数组，集合或映射的情况下，预期至少有一个匹配的元素。

默认行为是将带注解的方法和字段视为必须的依赖项。您可以通过以下示例中演示更改此行为，使框架能够让其变为非必须的来跳过一个不满足依赖的注入点（即，通过将@Autowired中的required属性设置为false)：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如果它的依赖项（或在多个参数的情况下）不可用，则不会调用此非必需的方法。在这种情况下，非必须的字段不会填充，会用其默认值代替。

被注入的构造器和工厂方法参数是一个特殊情况，因为@Autowired中required属性由于Spring的构造函数解析算法可能潜在地处理多个构造函数，因此具有稍微不同的含义。默认情况下构造器和工厂方法参数是必须的，但在单个构造函数场景中有一些特殊规则，例如多元素注入点（数组，集合，映射），如果没有可匹配的bean，则解析为空实例。这允许常见的实现模式，其中所有依赖关系都可以在唯一的多参数构造函数中声明 - 例如，在没有@Autowired注解的情况下被声明为单个公共构造函数。

> :information_source:给定bean只有一个构造函数可以被required=true的@Autowired声明，表明这个构造器是其作为Spring bean要注入的构造器。因此，如果其required属性为默认值true，则只有单个构造函数可以用@Autowired注解声明。如果多个构造函数声明了这个注解，则它们都必须将required设置为false，以便被视为用于自动装配的候选者（类似于XML中的autowire=constructor）。Spring容器将选择能最大满足依赖的构造器。如果候选者没有一个满足的，则将使用主/默认构造函数（如果存在）。同样，如果类声明多个构造函数，但都没有声明@Autowired注解，则将使用主/默认构造函数（如果存在）。如果类仅声明单个构造函数，即使没有注解，也将始终使用它。请注意，注解声明的构造函数修饰符不必为public。
>
> 在setter方法上建议使用@Autowired的required属性而不是已弃用的@Required注解。将required属性设置为false表示自动注入目标不是必须的，如果它无法自动注入则将忽略该属性。另一方面，@Required强制属性设置，无论用容器支持的什么手段，如果没有定义任何值，则会抛出相应的异常。

或者，您可以通过Java 8的`java.util.options`表示特定依赖的非必要性质，如以下示例所示：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

截至Spring Framework 5.0，您也可以使用@Nullable注释（任何包中的任何类型 - 例如，JSR-305中的`javax.annotation.Nullable`）或只是利用Kotlin内置的空安全支持：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以使用@Autowired用于众所周知的可解析依赖性的接口：BeanFactory，ApplicationContext，Environment，ResourceLoader，ApplicationEventPublisher和MessageSource。这些接口及其扩展接口（例如ConfigurableApplicationContext或ResourcePatternResolver）会自动解析，无需特殊设置。以下示例注入ApplicationContext对象：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

> :information_source:@Autowired，@Inject，@Value和@Resource注解由Spring BeanPostProcessor实现处理。这意味着您无法在您自己的BeanPostProcessor或BeanFactoryPostProcessor类型中应用这些注解（如果有）。这些类型必须通过使用XML或Spring @Bean方法显式注入。

### 1.9.3.  使用@Primary调整基于注解的自动注入

因为按类型的自动注入可能有多个候选者，所以通常需要更精细地控制选择过程。实现这一目标的一种方法是Spring的@Primary注解。@Primary表示当单值依赖有多个候选者时，应给出注入bean的首选项。如果候选者中存在一个主bean，则它将成为自动注入的值。

就像以下配置，将FirstMovieCatalog定义为MovieCatalog类型的主bean：

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

使用前面的配置，下面的MovieRecommender将注入firstMovieCatalog：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

相应的bean定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```



### 1.9.4.  使用@Qualifiers调整基于注解的自动注入


























# 附录

## 服务定位器模式

> 服务定位器模式 服务定位器模式（Service Locator Pattern）用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能

服务定位器模式（Service Locator Pattern）用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能。以下是这种设计模式的实体。

- **服务（Service）** - 实际处理请求的服务。对这种服务的引用可以在 JNDI 服务器中查找到。

- **Context / 初始的 Context** - JNDI Context 带有对要查找的服务的引用。
- **服务定位器（Service Locator）** - 服务定位器是通过 JNDI 查找和缓存服务来获取服务的单点接触。
- **缓存（Cache）** - 缓存存储服务的引用，以便复用它们。
- **客户端（Client）** - Client 是通过 ServiceLocator 调用服务的对象。

![servicelocator_pattern_uml_diagram.jpg](http://img.php.cn/upload/image/614/338/876/1486194513420223.jpg)