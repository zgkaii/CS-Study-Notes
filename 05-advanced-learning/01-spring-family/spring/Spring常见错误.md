Spring 常见错误
[spring的错误汇总](https://my.oschina.net/u/2499959/blog/524194)


```shell
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSource' defined in class path resource [applicationContext.xml]: 
BeanPostProcessor before instantiation of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with 
name 'org.springframework.aop.support.DefaultBeanFactoryPointcutAdvisor#0': Cannot resolve reference to bean 'txPointCut' while setting bean property 'pointcut'; 
nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'txPointCut': Instantiation of bean failed; nested exception 
is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.aop.aspectj.AspectJExpressionPointcut]: No default constructor found; 
nested exception is java.lang.NoClassDefFoundError: org/aspectj/weaver/reflect/ReflectionWorld$ReflectionWorldException

```
缺少jar包
```xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
```