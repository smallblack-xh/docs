# Hystrix(豪猪哥)
## 服务降级
- 服务器忙,请稍后再试,不让客户端等待过久并立刻返回一个有好的-提示(fallback)
- 常见场景
  - 程序运行异常
  - 服务超时
  - 服务熔断触发服务降级
  - 线程池、信号量打满导致服务降级 
## 服务熔断
- 当服务访问达到最大限度的访问量后,直接拒绝访问,然后调用服务降级的方法返回友好提示(类似于保险丝负载过大后直接熔断拉闸)
- 服务降级--->服务熔断--->恢复链路调用
- 熔断机制描述：应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间过长时，进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当该节点微服务调用检测正常时，恢复链路的调用
- example
    - 使用
        - @HystrixCommand
        - HystrixCommand中的参数(在时间窗口期内请求次数中的失败次数达到规定比例时实现断流)
          - circuitBreaker.enabled 是否开启断路器
          - circuitBreaker.requestVolumeThreshold 请求次数
          - circuitBreaker.sleepWindowInMilliseconds 时间窗口期
          - circuitBreaker.errorThresholdPercentage 失败率达到多少比例时断流
  - code
    - 
    ``` 
    @HystrixCommand(
            commandProperties = {
                    @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
                    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
                    @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
                    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),

            },
            fallbackMethod = "circuitBreakerFallback"
    )
    public String circuitBreaker(Integer id){
        if(id<0){
            throw new RuntimeException("非正数不得入内");
        }
        return IdUtil.simpleUUID();
    }
    public String circuitBreakerFallback(Integer id){
        return "circuitBreakerFallback";
    }
    ```
- 熔断类型
  - 熔断打开(open) 
    - 请求不再调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间),当打开时长达到所设时钟则进入半熔断状态
  - 熔断关闭(closed)
    - 熔断关闭不会对服务进行熔断
  - 熔断半开(half open)
    - 部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务正在恢复户或者已经正常，关闭熔断
## 服务限流
- 后续使用``sentinel``替换
## 工作流程
- 