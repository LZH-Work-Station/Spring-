# Spring源码

## 1. IOC	

### IOC(控制反转)

#### 定义

原来我们使用java对象需要自己创建，但是现在是Spring容器帮我们创建，当我们需要的时候就从Spring的IOC容器内拿就可以了。

#### 数据结构

IOC容器本质上就是一个Map结构，Spring创建的所有对象都会按照<K, V>格式创建。我们可以通过Key来获得Spring创建的Bean实例对象。

### Spring创建对象的方式

创建对象的方式主要有3种：new一个新对象，工厂模式创建对象，使用反射创建对象。

##### 1. 读取配置文件  

在早期开发的时候，我们通过编辑xml配置文件，将对象的属性，是否单例，Bean的name的信息写在xml配置文件中作为要创建的Bean的定义信息。然后IOC容器有一个BeanDefinitionReader然后按照既定的规则去读取xml文件，并创建Bean的定义信息（在源码中定义出BeanDefinition对象）

- **BeanDefiition**: BeanDefinition是一个接口 描述一个bean实例，并拥有他的属性信息，构造方法的参数等信息用来创建Bean
- **BeanDefinitionReader**：BeanDefinitionReader也是一个接口，用来读取配置文件(例如：xml，json，priperties等)。每一种配置文件都拥有他对应的实现这个接口的对象用来读取对应的配置文件来创建BeanDefinition。如果我们使用xml配置文件 就指定一个xml文件的BeanDefinitionReader来读取配置文件

##### 2. 实例化

在获得了BeanDefinition之后 即我们获得了Bean的定义信息 我们就会根据这些BeanDefinition去实例化这些Bean对象。这个过程采用的是**反射**的方式创建，反射的过程是在**BeanFactory**中进行。

> 这里我们区分一下实例化和初始化：
>
> **实例化**：在堆中创建一个空间用于实例化对象，所有的属性都是默认值
>
> **初始化**：1. 给对象的属性赋值    2. 执行Bean的init方法

###### BeanFactory反射的过程

```java
// 获得BeanDefinition信息
BeanDefinition bd = beanFactory.getBeanDefinition("aa")

// 根据BeanDefinition的信息先获取Class对象
Class cls = Class.forName("com.mysql.jdbc.Driver"); // 报名+类名=完全限定名
Class cls = 对象.getClass();
Class cls = 类.class; // 这个要在上面import这个类

// 根据Class对象获得构造方法，并通过构造方法实例化对象
Construtor ctor = cls.getDeclareConstructor();
Object obj = ctor.newInstance();
```

##### 3. Bean的实例化信息的处理

![image-20211211200304457](images/image-20211211200304457.png)

- **BeanFacoryPostProcessor**: 这里的BeanFactoryPostProcessor可以对 BeanDefinition信息进行一个预处理。例如我们在xml配置文件中，如果有一些${jdbc.username}类似这种需要读取另一个yaml配置文件而获得的信息，就可以在这些BeanFactoryPostProcessor中进行一个BeanDefinition的预处理。

##### 4. 初始化

<img src="images/image-20211211203210186.png" alt="image-20211211203210186" style="zoom:80%;" />

之后这一步就是初始化Bean，去给这些对象的属性赋值，然后完成Bean对象的创建。



##### 5. 额外的内容

- **StandardEnvironment**: 为了方便我们的使用，Spring在创建IOC容器的时候会将系统的相关属性加载到StandardEnvironment对象中，方便后续使用。
- **观察者模式**：如果我们需要监控Bean创建的过程 需要提前建立监听器和监听事件，这个就是观察者模式