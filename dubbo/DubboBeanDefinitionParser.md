- Dubbo Bean 定义解析器：实现了org.springframework.beans.factory.xml.BeanDefinitionParser 接口
- 解析方法
  ```
  @Override
    public BeanDefinition parse(Element element, ParserContext parserContext) {
        return parse(element, parserContext, beanClass, required);
    }
  ```
- 解析步骤
  - 创建RootBeanDefinition对象
    ```
    RootBeanDefinition beanDefinition = new RootBeanDefinition();
        beanDefinition.setBeanClass(beanClass);
        /*
        @KD 默认 lazyInit = false
        引用缺省是延迟初始化的，只有引用被注入到其它 Bean，或被 getBean() 获取，
        才会初始化。如果需要饥饿加载，即没有人引用也立即生成动态代理，
        可以配置：<dubbo:reference ... init="true" />
        */
        beanDefinition.setLazyInit(false);
    ```
  - 判断是否存在id属性配置，不存在则生成。当前beanDefinition在注册表中进行注册。同时将id为键以及对应的值放进当前beanDefinition#propertyValues#propertyValueList中存储起来
  - 处理各种dubbo标签并且解析其中的相关属性
    - dubbo:protocol
    - dubbo:service
    - <dubbo:provider><dubbo:service/></dubbo:provider>
    - <dubbo:consumer><dubbo:reference/></dubbo:consumer>
  - 通过反射将beanClass的相关属性