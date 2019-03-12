## FactoryBean 
### 定义
是一个java bean， 是一个能生产出当前对象的工厂bean，用户可以通过实现该接口来定制实例化bean的逻辑。它隐藏了实例化一些复杂bean的细节，给上层应用带来便利。
根据beanId从BeanFactory获取的，是beanFactory.getObject()的返回，而不是factoryBean本身。如果要获取factoryBean本身，则需要使用 &beanId 来获取。
### 实现
在AbstractBeanFactory的doGetBean方法实现中，如果这个methodBeanDefinition是继承了FactoryBean的，就会调用getObjectFromFactoryBean，这里会返回factory.getObject()

## BeanFactory
### 定义
beanFactory 是一个接口，是IOC容器的最基本形式。Spring中用到的applicationContext等 
是在beanFactory的基础上实现了其他的容器相关的功能。
### 功能
实例化、定位、配置对象、建立对象之间的依赖

## IOC的原理
### 几个定义
BeanDefinition : 抽象定义了bean， spring 是通过beanDefinition来管理bean的以及它们之间的依赖关系
BeanFactory : IOC容器的接口定义，定义了IOC容器的基本功能。
ApplicationContext : 在beanFactory的基础上，增加了其他容器的功能，包括：messageSource, ResourceLoader, ApplicationEventPublisher等

### IOC启动的过程

1. 构建beanFactory， 加载不同来源的beanDefinition，将beanDefinition放到concurrentHashMap中维护
2. 注册事件
3. 实例化bean，根据beanDefinition来实例化。
	A. 使用递归的方式来实例化，即，如果beanA 依赖beanB，则先实例化beanB
	B. 实例化对象后，拿到属性名称，从容器中拿到实例，通过反射(method.invoke()) 将实例set到对象中
4. 触发监听事件

## AOP的原理
### AOP用到的几个元素
#### PointCut 
连接点，定义匹配哪些方法。包含了classFIlter(类过滤器) 和 methodMatcher(方法匹配器)
methodMatcher有2中实现：由方法匹配器的inRuntime()来决定
1. 静态方法匹配器 ： 对方法名和入参类型 && 顺序 进行匹配
2. 动态方法匹配器 ：在运行期检查方法入参的值

### Advice
通知，定义在连接点做什么。它有很多个子类，例如要定义在方法执行完做什么，有afterReturningAdive

### Advisor
通知器，将advisor和pointcut结合起来，是在实现AOP时可以用到的对象

### AOP的实现过程
回到IOC启动的过程，在构建bean的时候，会调用getBean方法，而在getBean的方式有好几种，如果是继承了FactoryBean的，则会调用getObjectFromFactoryBean。
1. 在执行getObjectFromFactoryBean时，会调用ProxyFactoryBean.getObject
2. 在执行getObject时会创建proxy，根据配置，选择cglib/jdk的方式来生成proxyBean

## Bean的生命周期
### Bean初始化过程
1. 反射调用bean的构造方法，实例化bean
2. 反射注入bean的属性值
3. aware注入，例如beanNameAware, beanFactoryAware等
4. 调用每个调用beannPostProcessor接口的postProcessBeforeInitialization方法
5. 调用初始化方法
6. 调用每个调用beanPostProcessor接口的postBeanAfterInitialization方法
7. 注册需要执行销毁方法的bean，初始化结束

### Bean的销毁
如果bean使用了destory-method属性或实现了

## 动态代理
### 动态代理的定义
利用java反射计数，在运行时创建一个实现某些给定接口的类及其实例

### 代理的种类
代理类的主要功能，其实就是在调用某个方法前后，做一些额外的业务。实现这个功能有几种办法，一个是静态代理，一个是动态代理。

1. 静态代理：代理类Proxy中的方法，都写死了调用指定某个RealSubject
2. 动态代理：将自己的方法功能的实现交给InvocationHandler，外界对Proxy角色中的每个方法的调用，Proxy都会交给InvocationHandler来处理，而InvocationHandler则调用具体的角色

### 代理模式的角色
1. Subject : 负责定义RealSubject和Proxy应该实现的接口
2. RealSubject : 真正完成业务服务的功能
3. Proxy : 接收请求，并调研RealSubject的方法来实现特定功能

### 动态代理的实现
动态生成的Proxy需要实现RealSubject的所有功能，这样Proxy才能代理RealSubject。在Java中，如果要Proxy要覆盖RealSubject的功能，有2种方式：
1. 定义一个功能接口，让Proxy和RealSubject都来实现这个接口 -- JDK的动态代理
2. Proxy继承RealSubject -- cglib

### 基于接口的JDK动态代理
1. 获取RealSubject的所有接口列表
2. 确认生成代理类的类名，默认为: com.sun.proxy.$ProxyXXX
3. 根据需要实现的接口信息，在代码中动态创建该Proxy类的字节码
4. 将对应的字节码转为对应的class对象
5. 创建InvocationHandler实例handler，用来处理Proxy所有方法调用
6. Proxy的class对象，以创建的handler对象为参数，实例化一个proxy对象

为了实现以上6点，需要下面的方法来支持：
```
Proxy类：
Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
返回代理类的实例，该接口可以将方法调用指派到指定的调用处理程序
```
```
InvocationHandler类：
Object invoke(Object proxy,Method method,Object[] args)
在代理实例上处理方法调用并返回结果
```

缺点：
1. RealSubject必须要继承Subject接口，如果不继承接口，则无法实现代理
2. Proxy只能代理在Subject中声明的方法，RealSubject中声明的非继承方法，都无法实现代理

###  cglib动态代理
1. 查找RealSubject上所有非final的public类型的方法定义
2. 将这些方法转为字节码
3. 将组成的字节码转换成相应的代理class对象
4. 实现MethodInterceptor接口，用来处理对代理类上的所有方法的请求(类似于InvocationHandler)

生成的代理类，会继承RealSubject，如果callback不为空，那么在调用RealSubject方法的时候，会变为调用MethodInterceptor的inceptor()方法





























