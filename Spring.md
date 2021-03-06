## Spring

### 1. 事务

- 1.1 编程式事务管理

    >> 通过 TransactionTemplate或者TransactionManager手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助

- 1.2 声明式事务管理

    >> 推荐使用（代码侵入性最小），实际是通过 AOP 实现（基于@Transactional 的全注解方式使用最多）。


### 2. 事务的传播

|传播行为                   | 含义 |
|-------------------------|-----|
|PROPAGATION_REQUIRED     |表示当前方法必须在一个具有事务的上下文中运行，如有客户端有事务在进行，那么被调用端将在该事务中运行，否则的话重新开启一个事务。（如果被调用端发生异常，那么调用端和被调用端事务都将回滚）|
|PROPAGATION_SUPPORTS     |表示当前方法不必需要具有一个事务上下文，但是如果有一个事务的话，它也可以在这个事务中运行|
|PROPAGATION_MANDATORY    |表示当前方法必须在一个事务中运行，如果没有事务，将抛出异常|
|PROPAGATION_NESTED       |表示如果当前方法正有一个事务在运行中，则该方法应该运行在一个嵌套事务中，被嵌套的事务 ( 核心思想就是子事务不会独立提交，而是取决于父事务，当父事务提交，那么子事务才会随之提交；如果父事务回滚，那么子事务也回滚.但是子事务又有自己的特性，那 就是可以独立进行回滚，不会引发父事务整体的回滚(当然需要try catch子事务，避免异常传递至父层事务，如果没有，则也会引发父事务整体回滚 )) 可以独立于被封装的事务中进行提交或者回滚。如果封装事务存在，并且外层事务抛出异常回滚，那么内层事务必须回滚，反之，内层事务并不影响外层事务。如果封装事务不存在，则同PROPAGATION_REQUIRED的一样|
|PROPAGATION_NEVER        |表示当方法务不应该在一个事务中运行，如果存在一个事务，则抛出异常||
|PROPAGATION_REQUIRES_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将启动，而且如果有一个现有的事务在运行的话，则这个方法将在运行期被挂起，直到新的事务提交或者回滚才恢复执行。|
|PROPAGATION_NOT_SUPPORTED|表示该方法不应该在一个事务中运行。如果有一个事务正在运行，他将在运行期被挂起，直到这个事务提交或者回滚才恢复执行|

### 3. 动态代理

- 3.1 JDK动态代理

    >> JDK动态代理面向接口，通过反射生成目标代理接口的匿名实现类；

- 3.2 CGLIB动态代理

    >> CGLIB动态代理则通过继承，使用字节码增强技术（或者objenesis类库）为目标代理类生成代理子类。

>> Spring默认对接口实现使用JDK动态代理，对具体类使用CGLIB，同时也支持配置全局使用CGLIB来生成代理对象。

### 4. AOP

- 术语

    |术语	|含义|
    |-------|---|
    |横切关注点	|从每个方法中抽取出来的同一类非核心业务|
    |通知(Advice)	|切面必须要完成的各个具体工作,安全，事物，日志等|
    |连接点(Joinpoint)	|横切关注点在程序代码中的具体体现，对应程序执行的某个特定位置。方法前后，异常时|
    |切入点(pointcut)	|执行或找到连接点的一些方式，通过表达式|
    |切面(Aspect)	|封装横切关注点信息的类，每个关注点体现为一个通知方法。通知和切入点的集合|
    |引入(introduction)	|允许我们向现有的类添加新方法属性。这不就是把切面（也就是新方法属性：通知定义的）用到目标类中吗|
    |目标(Target)	|被通知的对象|
    |代理(proxy)	|向目标对象应用通知之后创建的代理对象|

- 原理

    Spring用代理类包裹切面，把他们织入到Spring管理的bean中。也就是说代理类伪装成目标类，它会截取对目标类中方法的调用，让调用者对目标类的调用都先变成调用伪装类，伪装类中就先执行了切面，再把调用转发给真正的目标bean。

    - 实现和目标类相同的接口，我也实现和你一样的接口，反正上层都是接口级别的调用，这样我就伪装成了和目标类一样的类（实现了同一接口，咱是兄弟了），也就逃过了类型检查，到java运行期的时候，利用多态的后期绑定（所以spring采用运行时），伪装类（代理类）就变成了接口的真正实现，而他里面包裹了真实的那个目标类，最后实现具体功能的还是目标类，只不过伪装类在之前干了点事情（写日志，安全检查，事物等）。

    - 生成子类调用，这次用子类来做为伪装类，当然这样也能逃过JVM的强类型检查，我继承的吗，当然查不出来了，子类重写了目标类的所有方法，当然在这些重写的方法中，不仅实现了目标类的功能，还在这些功能之前，实现了一些其他的（写日志，安全检查，事物等）。

### 5. 依赖注入

- 5.1 Set方式注入

    ```java
    @Service
    public class UserServiceImpl implements UserService {
    
        private UserMapper userMapper;
    
        @Autowired
        public void setUserMapper(UserMapper userMapper) {
            this.userMapper = userMapper;
        }
    
    }
    ```

- 5.2 构造器注入

    ```java
    @Service
    public class UserServiceImpl implements UserService {
    
        private final UserMapper userMapper;
    
        @Autowired
        public UserServiceImpl(UserMapper userMapper) {
            this.userMapper = userMapper;
        }
    
    }
    ```

- 5.3 注解方式

    ```java
    @Service
    public class UserServiceImpl implements UserService {
        @Autowired
        private UserMapper userMapper;
        
        //...
    
    }
    ```

    - @Autowired 注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。(通过类型匹配找到多个candidate,在没有@Qualifier、@Primary注解的情况下，会使用对象名作为最后的fallback匹配)

    - @Resource 默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

            ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。

            ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。

            ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。

            ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

            @Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入

### 6. 循环依赖

#### 6.1 是什么

>> 从字面上来理解就是A依赖B的同时B也依赖了A

![依赖](./img/Spring/%E5%BE%AA%E7%8E%AF.png)

#### 6.2 什么情况的可以被处理

|依赖情况	|依赖注入方式	|循环依赖是否被解决|
|----------|-------------|--------------|
|AB相互依赖（循环依赖）|	均采用@Autowried+@Lazy方法注入|	是|
|AB相互依赖（循环依赖）|	均采用setter方法注入|	是|
|AB相互依赖（循环依赖）|	均采用构造器注入	|否|
|AB相互依赖（循环依赖）|	A中注入B的方式为setter方法，B中注入A的方式为构造器|	是|
|AB相互依赖（循环依赖）|	B中注入A的方式为setter方法，A中注入B的方式为构造器|	否|


#### 6.3 如何解决色

- 简单的循环依赖（没有AOP）

![简单](./img/Spring//%E7%AE%80%E5%8D%95%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)

- aop循环依赖

![aop](./img//Spring//aop%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96.png)


-  一级缓存 singleObjects

>> 存放完整bean对象(已经初始化完毕，可使用的bean。)

-  二级缓存 earlySingleObjects

>> 存放不完整bean对象(已经实例化，但是还未进行属性注入及初始化的对象，提前曝光早产bean)

-  三级缓存 singleFactories

>> 存放bean工厂(提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象)

**Spring通过三级缓存解决了循环依赖，其中一级缓存为单例池（singletonObjects）,二级缓存为早期曝光对象earlySingletonObjects，三级缓存为早期曝光对象工厂（singletonFactories）。当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取，第一步，先获取到三级缓存中的工厂；第二步，调用对象工工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。至此，循环依赖结束！**

- 为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？

>> 如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。从软件设计角度考虑，三个缓存代表三种不同的职责，根据单一职责原理，从设计角度就需分离三种职责的缓存，所以形成三级缓存的状态

### 7. Bean 生命周期

- 实例化bean对象(通过构造方法或者工厂方法)

- 设置对象属性(setter等)（依赖注入）

- 如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。（和下面的一条均属于检查Aware接口）

- 如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身

- 将Bean实例传递给Bean的前置处理器的postProcessBeforeInitialization(Object bean, String beanname)方法

- 调用Bean的初始化方法

- 将Bean实例传递给Bean的后置处理器的postProcessAfterInitialization(Object bean, String beanname)方法

- 使用Bean

- 容器关闭之前，调用Bean的销毁方法