# 构造 starter 脚手架

## Spring 常用配置注释

+ **@Conditional   和 @ConditionalOnXXXX**

  利用 Condition ，在一个bean快被注册前， 我们可以根据任何的自由标准，立即触发条件的检查 ，使用 matches方法去 决定 是否注册； 可实现 Condition 接口自定义筛选条件

  ```
  public interface Condition {
      // 返回 true 符合条件
      boolean matches(ConditionContext var1, AnnotatedTypeMetadata var2);
  }
  ```

+ @Configuration

  该注解标志这是一个配置类

  **自动配置类可以不加该注解， 通过 `spring.factories`中加载 **

+ @EnableConfigurationProperties

  读取yml 配置文件中的配置项

+ @AutoConfigureAfter

  自定义自动配置类的执行顺序：当前配置类在指定配置类之后执行

+ @AutoConfigureBefore

  自定义自动配置类的执行顺序：当前配置类在指定配置类之前执行

+ `@AutoConfigureOrder`

  指定优先级，数值越小，优先级越高。

+ **@ConditionalOnProperty** 

  当指定的属性有指定的值时进行实例化。

  ```
  # 判断配置文件中是否有匹配值
  @ConditionalOnProperty(name="app.enable",havingValue = true)
  
  # yaml 配置文件中
  app:
    enable: true
  ```

  

+ 1

