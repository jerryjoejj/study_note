![20190121232919-SpringBean实例化过程.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/20190121232919-SpringBean%E5%AE%9E%E4%BE%8B%E5%8C%96%E8%BF%87%E7%A8%8B.png)

**具体调用过程如下**<br>
- (1)当调用者通过getBean(beanName)向容器请求某一个bean的时候，如果容器注册了；org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor接口，则在实例化bean之前调用接口的postProcessBeforeInstantiation()方法；
- (2)根据配置情况调用bean的构造函数或者工厂方法实例化bean；
- (3)如果容器注册了InstantiationAwareBeanPostProcessor接口，那么在实例化bean之后调用该接口的postProcessAfterInstantiation()方法对bean进行修饰；
- (4)如果bean配置了配置信息，那么容器将进一步把属性值设置到对应的属性上，在设置前先调用InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法；
- (5)调用bean的属性设置方法设置属性值；
- (6)如果bean实现了org.springframework.beans.factory.BeanNameAware接口，则将调用setBeanName()接口方法，将配置文件中该bean对应的名称设置到bean中；
- (7)如果bean实现了org.springframework.beans.factory.BeanFactoryAware接口，则调用setBeanFactory()接口方法，将beanFactory容器实例设置到bean中；
- (8)如果beanFactory装配了org.springframework.beans.factory.config.BeanPostProcessor后处理器，则将调用BeanPostProcessor的Object postProcessBeforeInitialization(Object bean, String beanName)接口方法对应的bean进行加工操作，其中入参bean是当前正在处理的bean，beanName是当前bean的配置名，返回对象是加工处理后的bean。`AOP`和`动态代理`都是通过BeanPostProcessor实现的；
- (9)如果bean实现了org.springframework.beans.factory.InitializingBean接口，则将调用接口的afterPropertiesSet()方法；
- (10)如果`<bean>`中通过init-method属性定义了初始化方法则调用这个方法；
- (11)BeanPostProcessor后处理器定义了两个方法：postProcessBeforeInitialization()在第8步调用， Object postProcessAfterInitialization(Object bean, String beanName)在此调用，容器在此获得对bean加工处理机会；
- (12)如果`<bean>`中指定的bean的作用范围scope="prototype"，则将bean返回给调用者，调用者负责后续bean的生命周期管理。如果将作用范围设置为scope="singleton"，则将bean放入SpringIoC容器缓存池中，并将bean返回给调用者，Spring继续对这些bean进行后续的生命周期管理；
- (13)scope="singleton"是默认情况，当容器关闭时，将触发Spring对bean后续生命周期的管理。如果bean实现了org.springframework.beans.factory.DisposableBean接口，则将调用接口的destory()方法，可以在这里释放资源，记录日志等;
- (14)scope="singleton"的bean，如果通过`<bean>`的destory-method方法指定了bean的销毁方法，那么Spring将执行bean的这个方法，完成bean的资源释放。