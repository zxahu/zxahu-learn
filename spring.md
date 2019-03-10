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

###Advice
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






























