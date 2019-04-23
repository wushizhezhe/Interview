# Spring

## 一、spring基础及组件的使用

1. 给容器中注册组件的方式
   - @Bean: [导入第三方的类或包的组件],比如Person为第三方的类, 需要在我们的IOC容器中使用

   * 包扫描+组件的标注注解(@ComponentScan:  @Controller、@Service、 @Reponsitory、@ Componet)，一般是针对 我们自己写的类，使用这个

    * @Import:[快速给容器导入一个组件] 注意:@Bean有点简单

      a)、@Import(要导入到容器中的组件):容器会自动注册这个组件，bean 的 id为全类名

      b)、ImportSelector:是一个接口,返回需要导入到容器的组件的全类名数组

      c)、ImportBeanDefinitionRegistrar:可以手动添加组件到IOC容器, 所有Bean的注册可以使用BeanDifinitionRegistry；写JamesImportBeanDefinitionRegistrar实现ImportBeanDefinitionRegistrar接口即可

    *  使用Spring提供的FactoryBean(工厂bean)进行注册

2. beanFactory和factoryBean的区别

   ​		BeanFactory是提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范，FactoryBean可以说为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，我们可以在getObject()方法中灵活配置。

