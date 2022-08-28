# 什么是死信

RabbitMQ可以保证消息的安全，持久化，但是当出现某一个特定业务条件时，需要设置该消息无法消费，或者是后台原因导致消息无法消费。这种没有后续处理且没能即时消费的消息就称为死信。**说白了，就是在当前还无法被消费的消息。此时就需要有一个特立独行的队列对其进行维护，死信队列应运而生。**

程序层次的产生原因：

+ 消息过期，无法被消费。
+ 队列被挤满了，无法新添加数据
+ 自动应答时，采用了拒绝或者否定策略，并且requeue=false。




# 架构实现

![image-20220723091825590](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220723091825590.png)

如图所示，该架构的核心在Consumer1，它关联了两个交换机和两个队列，另外两个，只是与对应的交换机或者队列建立联系，相对简单。

**组件配置：**

```java
@Configuration
public class TtlConfig {

	private static String NORMAL_QUEUE_1 = "normal_queue_1";
	private static String NORMAL_QUEUE_2 = "normal_queue_2";
	private static String NORMAL_EXCHANGE = "normal_exchange";

	private static String DEAD_QUEUE = "dead_queue";
	private static String DEAD_EXCHANGE = "dead_exchange";

	private static String NORMAL_KEY = "active.#";
	private static String DEAD_KEY = "dead";

	@Bean("normalQueue1")
	public Queue normalQueue1() {
		HashMap<String, Object> map = new HashMap<>();
		//设置过期时间
		map.put("x-message-ttl", 5000);
		//设置死信交换机
		map.put("x-dead-letter-exchange", DEAD_EXCHANGE);
		//设置死信路由
		map.put("x-dead-letter-routing-key", DEAD_KEY);
		return QueueBuilder.durable(NORMAL_QUEUE_1).withArguments(map).build();
	}

	@Bean("normalQueue2")
	public Queue normalQueue2() {
		HashMap<String, Object> map = new HashMap<>();
		//设置过期时间
		map.put("x-message-ttl", 10000);
		//设置死信交换机
		map.put("x-dead-letter-exchange", DEAD_EXCHANGE);
		//设置死信路由
		map.put("x-dead-letter-routing-key", DEAD_KEY);
		return QueueBuilder.durable(NORMAL_QUEUE_2).withArguments(map).build();
	}

	@Bean("normalExchange")
	public TopicExchange normalExchange() {
		return ExchangeBuilder.topicExchange(NORMAL_EXCHANGE).build();
	}

	@Bean("deadQueue")
	public Queue deadQueue() {
		return QueueBuilder.durable(DEAD_QUEUE).build();
	}

	@Bean("deadExchange")
	public DirectExchange deadExchange() {
		return ExchangeBuilder.directExchange(DEAD_EXCHANGE).build();
	}


	@Bean
	public Binding bindingNormal1(){
		return BindingBuilder.bind(normalQueue1()).to(normalExchange()).with("active.5");
	}

	@Bean
	public Binding bindingNormal2(){
		return BindingBuilder.bind(normalQueue2()).to(normalExchange()).with("active.10");
	}


	@Bean
	public Binding bindingDead(){
		return BindingBuilder.bind(deadQueue()).to(deadExchange()).with(DEAD_KEY);
	}

}
```

**Productor**

```java
@RestController
@RequestMapping("ttl")
@Slf4j
public class ProducerController {

	@Autowired
	private RabbitTemplate rabbitTemplate;

	@Resource
	private TopicExchange normalExchange;

	@GetMapping("send/{message}")
	public void send(@PathVariable String message) {
		log.info("当前时间[{}],sendMessage[{}]", new Date().toString(), message);
		rabbitTemplate.convertAndSend(normalExchange.getName(),"active.5",message);
		rabbitTemplate.convertAndSend(normalExchange.getName(),"active.10",message);
	}
}
```

**Consumer2**

```java
@Component
@Slf4j
@RabbitListener(queues = "dead_queue")
public class TtlConsumer {

	//监听指定的消息队列
	@RabbitHandler
	public void receive1(String content){
		String s = new String(content.getBytes(StandardCharsets.UTF_8));
		log.info("当前时间[{}],dead_queue receive[{}]", new Date().toString(), s);
	}
}
```

# 延时队列的优化

以上基于死信结构可以实现延时队列的效果，但是通用性太差，每给定一个延时需求都要构建一个新的队列，十分不友好。为此我们可以设置一个不参延时时间的队列，由于接收灵活的过期时间。

```java
@Bean("normalQueue3")
	public Queue normalQueue3() {
		HashMap<String, Object> map = new HashMap<>();
		//设置死信交换机
		map.put("x-dead-letter-exchange", DEAD_EXCHANGE);
		//设置死信路由
		map.put("x-dead-letter-routing-key", DEAD_KEY);
		return QueueBuilder.durable(NORMAL_QUEUE_2).withArguments(map).build();
	}

```

## 致命缺陷

以上方式确实提供了极大的方便，但是当多个消息进来，并且均设置了过期时间时，就极有可能会出现延时不符预期的问题。因为如果在消息的属性上设置TTL值，那么RabbitMQ在检测时，只能排队地先来后到地挨个检查，如果第一个消息的延时时长很长，而第二个消息的延时时长很短，那么第二个消息也必将在第一个消息之后得到执行。 