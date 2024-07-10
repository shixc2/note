> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [developer.aliyun.com](https://developer.aliyun.com/article/953777)

> 消息队列是大型分布式系统不可缺少的中间件，也是高并发系统的基石中间件，所以掌握好消息队列 MQ 就变得极其重要。

版权声明：

本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《 [阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和 《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写 [侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。

**简介：** 消息队列是大型分布式系统不可缺少的中间件，也是高并发系统的基石中间件，所以掌握好消息队列 MQ 就变得极其重要。接下来我就将从零开始介绍什么是消息队列？消息队列的应用场景？如何进行选型？如何在 Spring Boot 项目中整合集成消息队列。

前面介绍了分布式锁以及如何使用 Redis 实现分布式锁，接下来介绍分布式系统中另外一个非常重要的组件：消息队列。

消息队列是大型分布式系统不可缺少的中间件，也是高并发系统的基石中间件，所以掌握好消息队列 MQ 就变得极其重要。接下来我就将从零开始介绍什么是消息队列？消息队列的应用场景？如何进行选型？如何在 Spring Boot 项目中整合集成消息队列。

一、消息队列概述
--------

消息队列（Message Queue，简称 MQ）指保存消息的一个容器，其实本质就是一个保存数据的队列。

消息中间件是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信来进行分布式系统的构建。

消息中间件是分布式系统中重要的组件，主要解决应用解耦，异步消息，流量削峰等问题，实现高性能，高可用，可伸缩和最终一致性的系统架构。目前使用较多的消息队列有 ActiveMQ，RabbitMQ，ZeroMQ，Kafka，MetaMQ，RocketMQ 等。

二、消息队列应用场景
----------

消息中间件在互联网公司使用得越来越多，主要用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。以下介绍消息队列在实际应用中常用的使用场景。异步处理，应用解耦，流量削峰和消息通讯四个场景。

#### 2.1 异步处理

异步处理，就是将一些非核心的业务流程以异步并行的方式执行，从而减少请求响应时间，提高系统吞吐量。

![](https://ucc.alicdn.com/pic/developer-ecology/5c3ce3ea0d654fdabd9ca361663d9432.png?x-oss-process=image/resize,w_1400/format,webp)

以下单为例，用户下单后需要生成订单、赠送活动积分、赠送红包、发送下单成功通知等一系列业务处理。假设三个业务节点每个使用 100 毫秒钟，不考虑网络等其他开销，则串行方式的时间是 400 毫秒，并行的时间只需要 200 毫秒。这样就大大提高了系统的吞吐量。

#### 2.2 应用解耦

应用解耦，顾名思义就是解除应用系统之间的耦合依赖。通过消息队列，使得每个应用系统不必受其他系统影响，可以更独立自主。

以电商系统为例，用户下单后，订单系统需要通知积分系统。一般的做法是：订单系统直接调用积分系统的接口。这就使得应用系统间的耦合特别紧密。如果积分系统无法访问，则积分处理失败，从而导致订单失败。

![](https://ucc.alicdn.com/pic/developer-ecology/d49dbbdd14f84eca9f5dab2182b337ad.png?x-oss-process=image/resize,w_1400/format,webp)

加入消息队列之后，用户下单后，订单系统完成下单业务后，将消息写入消息队列，返回用户订单下单成功。积分系统通过订阅下单消息的方式获取下单通知消息，从而进行积分操作。实现订单系统与库存系统的应用解耦。如果，在下单时积分系统系统异常，也不影响用户正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作。

#### 2.3 流量削峰

流量削峰也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。

以秒杀活动为例，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列，秒杀业务处理系统根据消息队列中的请求信息，再做后续处理。

![](https://ucc.alicdn.com/pic/developer-ecology/050c3590a7b94a7799ad93e84bcb961c.png?x-oss-process=image/resize,w_1400/format,webp)

如上图所示，服务器接收到用户的请求后，首先写入消息队列，秒杀业务处理系统根据消息队列中的请求信息，做后续业务处理。假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面。

#### 2.4 消息通讯

消息通讯是指应用间的数据通信。消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等点对点通讯。

![](https://ucc.alicdn.com/pic/developer-ecology/f5d71084b1864b3aac4aed13779896e1.png?x-oss-process=image/resize,w_1400/format,webp)

以上实际是消息队列的两种消息模式，点对点或发布订阅模式。

三、如何选择合适的消息队列
-------------

目前使用较多的消息队列有 ActiveMQ，RabbitMQ，Kafka，RocketMQ 等。面对这么多的中消息队列中间件，如何选择适合我们自身业务的消息中间件呢？

#### 3.1 衡量标准

虽然这些消息队列在功能和特性方面各有优劣，但我们在选型时要有基本衡量标准：

1、首先，是开源。开源意味着，如果有一天你使用的消息队列遇到了一个影响你系统业务的 Bug，至少还有机会通过修改源代码来迅速修复或规避这个 Bug，解决你的系统的问题，而不是等待开发者发布的下一个版本来解决。

2、其次，是社区活跃度。这个产品必须是近年来比较流行并且有一定社区活跃度的产品。我们知道，开源产品越流行 Bug 越少，因为大部分遇到的 Bug，其他人早就遇到并且修复了。而且在使用过程中遇到的问题，也比较容易在网上搜索到类似的问题并快速找到解决方案。同时，流行开源产品一般与周边生态系统会有一个比较好的集成和兼容。

3、最后，作为一款及格的消息队列，必须具备的几个特性包括：

*   消息的可靠传递：确保不丢消息；
*   支持集群：确保不会因为某个节点宕机导致服务不可用，当然也不能丢消息；
*   性能：具备足够好的性能，能满足绝大多数场景的性能要求。

#### 3.2 选型对比

接下来我们一起看一下有哪些符合上面这些条件，可供选择的开源消息队列产品。以下是关于各个消息队列中间件的选型对比：

以上四种消息队列都有各自的优劣势，需要根据现有系统的情况，选择最适合的消息队列。

总结起来，电商、金融等对事务性要求很高的，可以考虑 RocketMQ；技术挑战不是特别高，用 RabbitMQ 是不错的选择；如果是大数据领域的实时计算、日志采集等场景可以考虑 Kafka。

四、Spring Boot 整合 RabbitMQ 实现消息队列
--------------------------------

Spring Boot 提供了 spring-bootstarter-amqp 组件对消息队列进行支持，使用非常简单，仅需要非常少的配置即可实现完整的消息队列服务。

接下来介绍 Spring Boot 对 RabbitMQ 的支持。如何在 SpringBoot 项目中使用 RabbitMQ？

### 4.1 Spring Boot 集成 RabbitMQ

Spring Boot 提供了 spring-boot-starter-amqp 组件，只需要简单的配置即可与 Spring Boot 无缝集成。下面通过示例演示集成 RabbitMQ 实现消息的接收和发送。

#### 第一步，配置 pom 包。

创建 Spring Boot 项目并在 pom.xml 文件中添加 spring-bootstarter-amqp 等相关组件依赖：

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

在上面的示例中，引入 Spring Boot 自带的 amqp 组件 spring-bootstarter-amqp。

#### 第二步，修改配置文件。

修改 application.properties 配置文件，配置 rabbitmq 的 host 地址、端口以及账户信息。

```
spring.rabbitmq.host=10.2.1.231
spring.rabbitmq.port=5672
spring.rabbitmq.username=zhangweizhong
spring.rabbitmq.password=weizhong1988
spring.rabbitmq.virtualHost=order
```

在上面的示例中，主要配置 RabbitMQ 服务的地址。RabbitMQ 配置由 spring.rabbitmq.* 配置属性控制。virtual-host 配置项指定 RabbitMQ 服务创建的虚拟主机，不过这个配置项不是必需的。

#### 第三步，创建消费者

消费者可以消费生产者发送的消息。接下来创建消费者类 Consumer，并使用 @RabbitListener 注解来指定消息的处理方法。示例代码如下：

```
@Component
public class Consumer {
  @RabbitHandler
  @RabbitListener(queuesToDeclare = @Queue("rabbitmq_queue"))
  public void process(String message) {
    System.out.println("消费者消费消息111=====" + message);
  }
}
```

在上面的示例中，Consumer 消费者通过 @RabbitListener 注解创建侦听器端点，绑定 rabbitmq_queue 队列。

（1）@RabbitListener 注解提供了 @QueueBinding、@Queue、@Exchange 等对象，通过这个组合注解配置交换机、绑定路由并且配置监听功能等。

（2）@RabbitHandler 注解为具体接收的方法。

#### 第四步，创建生产者

生产者用来产生消息并进行发送，需要用到 RabbitTemplate 类。与之前的 RedisTemplate 类似，RabbitTemplate 是实现发送消息的关键类。示例代码如下：

```
@Component
public class Producer {
  @Autowired
  private RabbitTemplate rabbitTemplate;
  
  public void produce() {
    String message = new Date() + "Beijing";
    System.out.println("生产者产生消息=====" + message);
    rabbitTemplate.convertAndSend("rabbitmq_queue", message);
  }
}
```

如上面的示例所示，RabbitTemplate 提供了 convertAndSend 方法发送消息。convertAndSend 方法有 routingKey 和 message 两个参数：

（1）routingKey 为要发送的路由地址。

（2）message 为具体的消息内容。发送者和接收者的 queuename 必须一致，不然无法接收。

#### 第五步，测试验证。

创建对应的测试类 ApplicationTests，验证消息发送和接收是否成功。

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {
  @Autowired
  Producer producer;
  @Test
  public void contextLoads() throws InterruptedException {
    producer.produce(); 
    Thread.sleep(1*1000);
  }
}
```

在上面的示例中，首先注入生产者对象，然后调用 produce() 方法来发送消息。

最后，单击 Run Test 或在方法上右击，选择 Run 'contextLoads()'，运行单元测试程序，查看后台输出情况，结果如下图所示。

![](https://ucc.alicdn.com/pic/developer-ecology/d57e9cda0d864e71a20f8d94fadaae06.png?x-oss-process=image/resize,w_1400/format,webp)

通过上面的程序输出日志可以看到，消费者已经收到了生产者发送的消息并进行了处理。这是常用的简单使用示例。

### 4.2 发送和接收实体对象

Spring Boot 支持对象的发送和接收，且不需要额外的配置。下面通过一个例子来演示 RabbitMQ 发送和接收实体对象。

#### 4.2.1 定义消息实体

首先，定义发送与接收的对象实体 User 类，代码如下：

```
public class User implements Serializable {
  public String name;
  public String password;
  // 省略get和set方法
}
```

在上面的示例中，定义了普通的 User 实体对象。需要注意的是，实体类对象必须继承 Serializable 序列化接口，否则会报数据无法序列化的错误。

#### 4.2.2 定义消费者

修改 Consumer 类，将参数换成 User 对象。示例代码如下：

```
@Component
public class Consumer {
  @RabbitHandler
  @RabbitListener(queuesToDeclare = @Queue("rabbitmq_queue_object"))
  public void process(User user) {
    System.out.println("消费者消费消息111user=====name：" + user.getName()+",password:"+user.getPassword());
  
  }
}
```

其实，消费者类和消息处理方法和之前的类似，只不过将参数换成了实体对象，监听 rabbitmq_queue_object 队列。

#### 4.2.3 定义生产者

修改 Producer 类，定义 User 实体对象，并通过 convertAndSend 方法发送对象消息。示例代码如下：

```
@Component
public class Producer {
  @Autowired
  private RabbitTemplate rabbitTemplate;
  
  public void produce() { 
    User user=new User();
    user.setName("weiz");
    user.setPassword("123456");
    System.out.println("生产者生产消息111=====" + user);
    
    rabbitTemplate.convertAndSend("rabbitmq_queue_object", user);
  }
}
```

在上面的示例中，还是调用 convertAndSend() 方法发送实体对象。convertAndSend() 方法支持 String、Integer、Object 等基础的数据类型。

#### 4.2.4 验证测试

创建单元测试类，注入生产者对象，然后调用 produceObj() 方法发送实体对象消息，从而验证消息能否被成功接收。

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {
  @Autowired
  Producer producer;
  @Test
  public void testProduceObj() throws InterruptedException {
    producer.produceObj();
    Thread.sleep(1*1000);
  }
}
```

最后，单击 Run Test 或在方法上右击，选择 Run 'contextLoads()'，运行单元测试程序，查看后台输出情况，运行结果如下图所示。

![](https://ucc.alicdn.com/pic/developer-ecology/b3213df604eb46a9a56f9ec64241df91.png?x-oss-process=image/resize,w_1400/format,webp)

通过上面的示例成功实现了 RabbitMQ 发送和接收实体对象，使得消息的数据结构更加清晰，也更加贴合面向对象的编程思想。

五、实现消息的 100% 可靠性发送
------------------

### 5.1 什么是实现消息的 100% 可靠性发送？

在使用消息队列时，因为生产者和消费者不直接交互，所以面临下面几个问题：

1）要把消息添加到队列中，怎么保证消息成功添加？

2）如何保证消息发送出去时一定会被消费者正常消费？

3）消费者正常消费了，生产者或者队列如何知道消费者已经成功消费了消息？

要解决前面这些问题，就要保证消息的可靠性发送，实现消息的 100% 可靠性发送。

### 5.2 技术实现方案

RabbitMQ 为我们提供了解决方案，下面以常见的创建订单业务为例进行介绍，假设订单创建成功后需要发送短信通知用户。实现消息的 100% 可靠性发送需要以下条件：

1）完成订单业务处理后，生产者发送一条消息到消息队列，同时记录这条操作日志（发送中）。

2）消费者收到消息后处理进行；

3）消费者处理成功后给消息队列发送 ack 应答；

4）消息队列收到 ack 应答后，给生成者的 Confirm Listener 发送确认；

5）生产者对消息日志表进行操作，修改之前的日志状态（发送成功）；

6）在消费端返回应答的过程中，可能发生网络异常，导致生产者未收到应答消息，因此需要一个定时任务去提取其状态为 “发送中” 并已经超时的消息集合；

7）使用定时任务判断为消息事先设置的最大重发次数，大于最大重发次数就判断消息发送失败，更新日志记录状态为发送失败。

具体流程如下图所示

![](https://ucc.alicdn.com/pic/developer-ecology/48247e3cabbd4c2085defccd1ea67f7b.png?x-oss-process=image/resize,w_1400/format,webp)  

### 5.3 实现消息的 100% 可靠性发送

前面介绍了实现消息的 100% 可靠性发送的解决方案，接下来从项目实战出发演示如何实现消息的可靠性发送。

#### 1. 创建生产者

首先把核心生产者的代码编写好，生产者由基本的消息发送和监听组成：

```
@Component
public class RabbitOrderSender {
    private static Logger logger = LoggerFactory.getLogger(RabbitOrderSender.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private MessageLogMapper messageLogMapper;

    /**
     * Broker应答后，会调用该方法区获取应答结果
     */
    final RabbitTemplate.ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            logger.info("correlationData："+correlationData);
            String messageId = correlationData.getId();
            logger.info("消息确认返回值："+ack);
            if (ack){
                //如果返回成功，则进行更新
                messageLogMapper.changeMessageLogStatus(messageId, Constans.ORDER_SEND_SUCCESS,new Date());
            }else {
                //失败进行操作：根据具体失败原因选择重试或补偿等手段
                logger.error("异常处理,返回结果："+cause);
            }
        }
    };

    /**
     * 发送消息方法调用: 构建自定义对象消息
     * @param order
     * @throws Exception
     */
    public synchronized void  sendOrder(OrderInfo order) throws Exception {
        // 通过实现 ConfirmCallback 接口，消息发送到 Broker 后触发回调，确认消息是否到达 Broker 服务器，也就是只确认是否正确到达 Exchange 中
        rabbitTemplate.setConfirmCallback(confirmCallback);
        //消息唯一ID
        CorrelationData correlationData = new CorrelationData(order.getMessageId());
        rabbitTemplate.convertAndSend("order.exchange", "order.message", order, correlationData);
    }
}
```

上面的消息发送示例代码和之前的没什么区别，只是增加了 confirmCallback 应答结果回调。通过实现 ConfirmCallback 接口，消息发送到 Broker 后触发回调，确认消息是否到达 Broker 服务器。因此，ConfirmCallback 只能确认消息是否正确到达交换机中。

#### 2. 消息重发定时任务

实现消息重发的定时任务，示例代码如下：

```
@Component
public class RetryMessageTasker {
    private static Logger logger = LoggerFactory.getLogger(RetryMessageTasker.class);
    @Autowired
    private RabbitOrderSender rabbitOrderSender;

    @Autowired
    private MessageLogMapper messageLogMapper;

    /**
     * 定时任务
     */
    @Scheduled(initialDelay = 5000, fixedDelay = 10000)
    public void reSend(){
        logger.info("-----------定时任务开始-----------");
        //抽取消息状态为0且已经超时的消息集合
        List<MessageLog> list = messageLogMapper.query4StatusAndTimeoutMessage();
        list.forEach(messageLog -> {
            //投递三次以上的消息
            if(messageLog.getTryCount() >= 3){
                //更新失败的消息
                messageLogMapper.changeMessageLogStatus(messageLog.getMessageId(), Constans.ORDER_SEND_FAILURE, new Date());
            } else {
                // 重试投递消息，将重试次数递增
                messageLogMapper.update4ReSend(messageLog.getMessageId(),  new Date());
                OrderInfo reSendOrder = JsonUtil.jsonToObject(messageLog.getMessage(), OrderInfo.class);
                try {
                    rabbitOrderSender.sendOrder(reSendOrder);
                } catch (Exception e) {
                    e.printStackTrace();
                    logger.error("-----------异常处理-----------");
                }
            }
        });
    }
}
```

在上面的定时任务程序中，每 10 秒钟提取状态为 0 且已经超时的消息，重发这些消息，如果发送次数已经在 3 次以上，则认定为发送失败。

#### 3. 创建消费者

创建消费者程序负责接收处理消息，处理成功后发送消息确认。示例代码如下：

```
@Component
public class OrderReceiver {
    /**
     * @RabbitListener 消息监听，可配置交换机、队列、路由key
     * 该注解会创建队列和交互机 并建立绑定关系
     * @RabbitHandler 标识此方法如果有消息过来，消费者要调用这个方法
     * @Payload 消息体
     * @Headers 消息头
     * @param order
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "order.queue",declare = "true"),
            exchange = @Exchange(name = "order.exchange",declare = "true",type = "topic"),
            key = "order.message"
    ))
    @RabbitHandler
    public void onOrderMessage(@Payload Order order, @Headers Map<String,Object> headers,
                               Channel channel) throws Exception{
        //消费者操作
        try {
            System.out.println("------收到消息，开始消费------");
            System.out.println("订单ID："+order.getId());

            Long deliveryTag = (Long)headers.get(AmqpHeaders.DELIVERY_TAG);
            //现在是手动确认消息 ACK
            channel.basicAck(deliveryTag,false);
        } finally {
           channel.close();
        }
    }
}
```

消息处理程序和一般的接收者类似，都是通过 @RabbitListener 注解监听消息队列。不同的是，发送程序处理成功后，通过 channel.basicAck(deliveryTag,false) 发送消息确认 ACK。

#### 4. 运行测试

创建单元测试程序。创建一个生成订单的测试方法，测试代码如下：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class MqApplicationTests {
    @Autowired
    private OrderService orderService;

    /**
     * 测试订单创建
     */
    @Test
    public void createOrder(){
        OrderInfo order = new OrderInfo();
        order.setId("201901236");
        order.setName("测试订单6");
        order.setMessageId(System.currentTimeMillis() + "$" + UUID.randomUUID().toString());
        try {
            orderService.createOrder(order);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

启动消费者程序，启动成功之后，运行 createOrder 创建订单测试方法。结果表明发送成功并且入库正确，业务表和消息记录表均有数据且 status 状态 =1，表示成功。

如果消费者程序处理失败或者超时，未返回 ack 确认；则生产者的定时程序会重新投递消息。直到三次投递均失败。

六、MQ 常见问题总结
-----------

### 6.1 怎么保证消息没有重复消费？使用消息队列如何保证幂等性?

**幂等性：就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用****问题出现原因**我们先来了解一下产生消息重复消费的原因，对于 MQ 的使用，有三个角色：生产者、[MQ](http://mp.weixin.qq.com/s?__biz=MzAwNjQwNzU2NQ==&mid=2650357391&idx=1&sn=98a68099613a0f3d22b1f75c90f27f27&chksm=83004a6db477c37b1cd81500be3e7a474475780d70c760804119dc841362d88bb375c47d4d70&scene=21#wechat_redirect)、消费者，那么消息的重复这三者会出现：

*   生产者：生产者可能会推送重复的数据到 MQ 中，有可能 controller 接口重复提交了两次，也可能是重试机制导致的
*   MQ：假设网络出现了波动，消费者消费完一条消息后，发送 ack 时，MQ 还没来得及接受，突然挂了，导致 MQ 以为消费者还未消费该条消息，MQ 回复后会再次推送了这条消息，导致出现重复消费。
*   消费者：消费者接收到消息后，正准备发送 ack 到 MQ，突然消费者挂了，还没得及发送 ack，这时 MQ 以为消费者还没消费该消息，消费者重启后，MQ 再次推送该条消息。

**解决方案**在正常情况下，生产者是客户，我们很难避免出现用户重复点击的情况，而 MQ 是允许存在多条一样的消息，但消费者是不允许出现消费两条一样的数据，所以幂等性一般是在消费端实现的：

*   状态判断：消费者把消费消息记录到 redis 中，再次消费时先到 redis 判断是否存在该数据，存在则表示消费过，直接丢弃
*   业务判断：消费完数据后，都是需要插入到数据库中，使用数据库的唯一约束防止重复消费。插入数据库前先查询是否存在该数据，存在则直接丢弃消息，这种方式是比较简单粗暴地解决问题

### 6.2 消息丢失的情况

消息丢失属于比较常见的问题。一般有生产端丢失、MQ 服务丢失、消费端丢失等三种情况。针对各种情况应对方式也不一样。

1. 生产端丢失的解决方案主要有

*   开启 confirm 模式，生产着收到 MQ 发回的 confirm 确认之后，再进行消息删除，否则消息重推。
*   生产者端消息保存的数据库，由后台定时程序异步推送，收到 confirm 确认则认为成功，否则消息重推，重推多次均未成功，则认为发送失败。

2.MQ 服务丢失则主要是开启消息持久化，让消息及时保存到磁盘。

3. 消费端消息丢失则关闭自动 ack 确认，消息消费成功后手动发送 ack 确认。消息消费失败，则重新消费。

### 6.3 消息的传输顺序性

**解决思路**在生产端发布消息时，每次法发布消息都把上一条消息的 ID 记录到消息体中，消费者接收到消息时，做如下操作

*   先根据上一条 Id 去检查是否存在上一条消息还没被消费，如果不存在 (消费后去掉 id)，则正常进行，如果正常操作
*   如果存在，则根据 id 到数据库检查是否被消费，如果被消费，则正常操作
*   如果还没被消费，则休眠一定时间 (比如 30ms)，再重新检查，如被消费，则正常操作
*   如果还没被消费，则抛出异常

### 6.4 怎么解决消息积压问题?

所谓的消息积压，就是生成者生成消息太快，而消费者处理消息太慢，从而导致消费端消息积压在 MQ 中无法处理的问题。遇到这种消息积压的情况，可以根据消息重要程度，分为两种情况处理：

*   如果消息可以被丢弃，那么直接丢弃就好了
*   一般情况下，消息是不可以被丢弃的，那么这样需要考虑策略了，我们可以把原来的消费端重新当做生产端，重新部署一天 MQ，再后面出现增加消费端，这样形成另一条生产 - 消息 - 消费的线路  
    ![](https://ucc.alicdn.com/pic/developer-ecology/a24b5f2b8d234e638d871b5afc4df4ac.png?x-oss-process=image/resize,w_1400/format,webp)

以上，我们就把消息队列介绍完了。消息中间件在互联网公司使用得越来越多，希望大家能够熟悉其使用。