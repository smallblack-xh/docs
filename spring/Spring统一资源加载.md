# Spring资源统一加载笔记
- Spring资源加载策略
  - 职能划分清楚：资源的定义和资源的加载应该要有一个清晰的界限
  - 统一的抽象：统一的资源定义和资源加载策略。资源加载后要返回统一的抽象给客户端，客户端要对资源进行怎样的处理，应该由抽象资源接口来界定。
- 统一资源：***Resource*([Resource.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/Resource.java))**<a name='Resource'></a>   
   org.springframework.core.io.*Resource 为 Spring 框架所有资源的抽象和访问接口，它继承 org.springframework.core.io.InputStreamSource接口。作为所有资源的统一抽象，Resource 定义了一些通用的方法，由子类 AbstractResource 提供统一的默认实现。
  - 子类结构

    ![ResourceLoader 类图](imgs/2020-10-16-15-29-37.png)
    - FileSystemResource ：对 java.io.File 类型资源的封装，只要是跟 File 打交道的，基本上与 FileSystemResource 也可以打交道。支持文件和 URL 的形式，实现 WritableResource 接口，且从 Spring Framework 5.0 开始，FileSystemResource 使用 NIO2 API进行读/写交互
    - ByteArrayResource ：对字节数组提供的数据的封装。如果通过 InputStream 形式访问该类型的资源，该实现会根据字节数组的数据构造一个相应的 ByteArrayInputStream
    - UrlResource ：对 java.net.URL类型资源的封装。内部委派 URL 进行具体的资源操作
    - ClassPathResource ：class path 类型资源的实现。使用给定的 ClassLoader 或者给定的 Class 来加载资源
    - InputStreamResource ：将给定的 InputStream 作为一种资源的 Resource 的实现类
  - ***AbstractResource***([AbstractResource.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/AbstractResource.java))
    - org.springframework.core.io.AbstractResource ，为 Resource 接口的默认抽象实现,实现了 Resource 接口的大部分的公共实现。
    - 当需要实现自定义的Resource时，应该继承<u>*AbstractResource*</u>接口<a name='AbstractResource'></a>
- 统一资源定位：***ResourceLoader*([ResourceLoader.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/ResourceLoader.java))**<a name='ResourceLoader'></a>  
  - 子类结构  
    ![ResourceLoader 类图](imgs/2020-10-16-15-32-43.png)
  - Spring 将资源的定义和资源的加载区分开了，Resource 定义了统一的资源，而资源的加载由 ***ResourceLoader***统一定义
  - org.springframework.core.io.ResourceLoader 为 Spring 资源加载的统一抽象，具体的资源加载则由相应的实现类来完成，所以我们可以将 ResourceLoader 称作为**统一资源定位器**
  - ResourceLoader根据给定的资源文件地址，返回对应的 Resource
  - 方法解读
    - Resource getResource(String location)  
        根据所提供资源的路径 location 返回 Resource 实例，但是它不确保该 Resource 一定存在，需要调用 Resource#exist() 方法来判断
      - 该方法支持以下模式的资源加载
        - URL位置资源，如 "file:C:/test.dat" 。
        - ClassPath位置资源，如 "classpath:test.dat 。
        - 相对路径资源，如 "WEB-INF/test.dat" ，此时返回的Resource 实例，根据实现不同而不同
      - 该方法的主要实现是在其子类[DefaultResourceLoader.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/DefaultResourceLoader.java)中实现
    - ClassLoader getClassLoader()  
        返回 ClassLoader 实例，对于想要获取 ResourceLoader 使用的 ClassLoader 用户来说，可以直接调用该方法来获取。在分析 Resource 时，提到了一个类 ClassPathResource ，这个类是可以根据指定的 ClassLoader 来加载资源的
  - 默认实现 ***DefaultResourceLoader*([DefaultResourceLoader.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/DefaultResourceLoader.java))**
    - [构造方法](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/DefaultResourceLoader.java#L63-L76):无参时，使用ClassUtils.getDefaultClassLoader()获取ClassLoader。存在参数时,使用传入的 ClassLoader 参数
    - [DefaultResourceLoader#getResource](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/DefaultResourceLoader.java#L150-L194)
    - 上面说到<a href='#AbstractResource'>AbstractResource</a>时提到，如果要实现Resource的解析，需要继承AbstractResource接口。但是在*DefaultResourceLoader#getResource*中,我们能发现这样一块代码
      ```
      for (ProtocolResolver protocolResolver : this.protocolResolvers) {
        Resource resource = protocolResolver.resolve(location, this);
        if (resource != null) {
          return resource;
        }
      }
      ```
      - ***ProtocolResolver***:是用户自定义协议资源解决策略,是DefaultResourceLoader的SPI。通过它我们可以不再需要继承AbstractResource、DefaultResourceLoader，直接实现ProtocolResolver接口即可实现自定义的ResourceLoader
      - 通过实现*ProtocolResolver*接口自定义资源加载实现类以后，需要调用[DefaultResourceLoader#addProtocolResolver(ProtocolResolver)](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/DefaultResourceLoader.java#L99-L112)方法，将实现类添加至spring中即可使用。
      - 使用*DefaultResourceLoader#getResource*作为ResourceLoader，不同的资源会出现不同的返回，如下：
        - D://a.text  ClassPathResource 
        - /a.text ClassPathResource
        - file:/a.text UrlResource
        - `http://www.baidu.com` UrlResource
        - **PS：** 从上面可以知道，当使用D://方式进行资源定位时，返回的结果为*ClassPathResource*,但实际上这应该看成是一个*FileSystemResource* 类型,所以为了解析这种格式的资源，我们可以使用*DefaultResourceLoader*的子类*FileSystemResourceLoader*来处理这种格式的文件
  - ***FileSystemResourceLoader***([FileSystemResourceLoader](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/FileSystemResourceLoader.java))
    - *FileSystemResourceLoader*继承自*DefaultResourceLoader*,覆盖了其[getResourceByPath()](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/FileSystemResourceLoader.java#L49-L56)方法，如下
      ```
      protected Resource getResourceByPath(String path) {
        if (path.startsWith("/")) {
          path = path.substring(1);
        }
        return new FileSystemContextResource(path);
      }
      ```  
      - ***FileSystemContextResource***([FileSystemContextResource.java](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/core/io/FileSystemResourceLoader.java#L58-L72))
        - *FileSystemResourceLoader*内部类，继承自*FileSystemResource*，实现了*ContextResource*接口
        - 在构造类直接调用父类构造方法，父类构造方法中调用了[StringUtils.cleanPath(path)](https://github.com/smallblack-xh/spring-framework/blob/5.0.x/spring-core/src/main/java/org/springframework/util/StringUtils.java#L647-L708)方法处理传入的路径，通过该方法，将注入**D://a.text**格式的文件转化为FileSystemResource。同时加入传入的path为 **/a.text且当前目录为E://**,则最终通过StringUtils.cleanPath(path)构造出的path为**E://a.text**
          ```
          public FileSystemContextResource(String path) {
            super(path);
            /**
            public FileSystemResource(String path) {
              Assert.notNull(path, "Path must not be null");
              this.file = new File(path);
              this.path = StringUtils.cleanPath(path);
            }
            **/
          }
          ```
  - ***ClassRelativeResourceLoader***:给定的class 所在包或者所在包的子包下加载资源
  - ***ResourcePatternResolver***:interface 继承自 *ResourceLoader*,增加了``Resource[] getResources(String locationPattern)``方法，用于解析``classpath*:``前缀配置，支持一次加载多个资源。实现类 *PathMatchingResourcePatternResolver*
# 总结
- Spring 提供了 Resource 和 ResourceLoader 来统一抽象整个资源及其定位。使得资源与资源的定位有了一个更加清晰的界限，并且提供了合适的 Default 类，使得自定义实现更加方便和清晰。
- AbstractResource 为 Resource 的默认抽象实现，它对 Resource 接口做了一个统一的实现，子类继承该类后只需要覆盖相应的方法即可，同时对于自定义的 Resource 我们也是继承该类。
- DefaultResourceLoader 同样也是 ResourceLoader 的默认实现，在自定 ResourceLoader 的时候我们除了可以继承该类外还可以实现 ProtocolResolver 接口来实现自定资源加载协议。
- DefaultResourceLoader 每次只能返回单一的资源，所以 Spring 针对这个提供了另外一个接口 ResourcePatternResolver ，该接口提供了根据指定的 locationPattern 返回多个资源的策略。其子类 PathMatchingResourcePatternResolver 是一个集大成者的 ResourceLoader ，因为它即实现了 Resource getResource(String location) 方法，也实现了 Resource[] getResources(String locationPattern) 方法。