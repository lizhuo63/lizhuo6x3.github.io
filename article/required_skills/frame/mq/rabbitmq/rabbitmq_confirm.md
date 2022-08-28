# 发布确认（springboot）

## 配置

在配置文件当中需要添加 
spring.rabbitmq.publisher-confirm-type=correlated ，有三种模式：

+ NONE ：禁用发布确认模式，是默认值 

+ SIMPLE ：同步逐个确认

+ CORRELATED ：推荐

## 定义回调

**注意：** `ConfirmCallback `回调只能用于判定交换机能否接收到消息，对于通过此关卡而因路由错误导致的消息丢失，生产者将无法感知。所以需要另外实现 `ReturnsCallback` 对路由失败无法消费的消息进行回退处理。

```java
@Slf4j
@Component
public class MsgCallBack implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback {

	@Autowired
	private RabbitTemplate rabbitTemplate;

	@PostConstruct
	public void  init() {
		rabbitTemplate.setConfirmCallback(this);//注入
		rabbitTemplate.setReturnsCallback(this);//注入
	}

	/**
	 * @param correlationData 保存有回调消息的ID及其他
	 * @param b 【交换机】接收消息是否成功
	 * @param s 【交换机】接收失败原因（成功为null）
	 */
	@Override
	public void confirm(CorrelationData correlationData, boolean b, String s) {
		String id = correlationData == null ? null : correlationData.getId();
		if (b) {
			log.info("接收成功，消息ID为[{}]", id);
		}else {
			log.error("接受失败，消息ID为[{}]，失败原因是[{}]",id,s);
		}
	}
	
	//路由失败回调
	@Override
	public void returnedMessage(ReturnedMessage returned) {
		log.error("消息[{}]被交换机[{}]被退会，退回原因[{}],路由Key[{}]",
				new String(returned.getMessage().getBody()),
				returned.getExchange(),
				returned.getReplyText(),
				returned.getRoutingKey());
	}
}
```

## 定义备份

在回调之外，还可通过定义备份交换机的方式对回退消息进行转发，令其成功消费。但是，一旦该消息被消费成功，就无法触发路由失败致不可消费的回调了，需要单独配置一个报警队列用以提醒。

![image-20220723175125598](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220723175125598.png)

**在基于回调的基础上进行改进**

```java
	@Bean("backQueue")
	public Queue backQueue() {
		return QueueBuilder.durable(BACK_QUEUE).build();
	}

	@Bean("warnQueue")
	public Queue warnQueue() {
		return QueueBuilder.durable(WARN_QUEUE).build();
	}

	@Bean("confirmExchange")
	public DirectExchange confirmExchange() {
		return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE)
				//配置添加备份交换机
				.durable(true).withArgument("alternate-exchange",BACK_EXCHANGE).build();
	}

	@Bean("backExchange")
	public FanoutExchange backExchange() {
		return ExchangeBuilder.fanoutExchange(BACK_EXCHANGE).durable(true).build();
	}
	
	@Bean
	public Binding bindingBack() {
		return BindingBuilder.bind(backQueue()).to(backExchange());
	}

	@Bean
	public Binding bindingWarn() {
		return BindingBuilder.bind(warnQueue()).to(backExchange());
	}
```

```java
@Component
@Slf4j
@RabbitListener(queues = "warn_queue")
public class WarnConsumer {
	//监听指定的消息队列
	@RabbitHandler
	public void receive1(String content) {
		String s = new String(content.getBytes(StandardCharsets.UTF_8));
		log.error("发现不可路由的消息:[{}]", s);
	}
}
```

