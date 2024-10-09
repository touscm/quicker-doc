# Pulsar消息队列

Pulsar消息队列使用说明, 此说明简要阐述自动装配的使用

## 添加引用
```yaml
    <dependencies>
        <dependency>
            <groupId>com.touscm</groupId>
            <artifactId>spring-pulsar-starter</artifactId>
            <version>1.3.2-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

## 配置Pulsar服务信息
```yaml
pulsar:
  # pulsar服务地址
  url: pulsar://192.168.2.180:6650
  # pulsar认证令牌
  token: xxx

service:
  mq:
    topic:
      # 定义Topic
      track: persistent://public/the-namespace/track
    subscribe:
      # 定义消费者名
      track: subscribe
```

## 创建生产者

生产者支持发送即时消息, 延迟消息和定时消息<br>
`@PulsarProducerConfig`注解支持指定topic, 生产者名<br>
`注意`: Pulsar生产者所在Class需指定`@PulsarProcessor`注解, `@PulsarProducerConfig`的属性`topicKey`用于指定topic的配置Key

```java
@Service
@PulsarProcessor
public class TrackProducer {
    private static final Logger logger = LoggerFactory.getLogger(TrackProducer.class);

    @PulsarProducerConfig(topicKey = "service.mq.topic.track")
    private IProducer<TrackEntry> producer;

    /**
     * 发送消息
     *
     * @param entry 消息实体
     */
    public void send(@NotNull TrackEntry entry) {
        if (!producer.send(entry)) {
            logger.error("发送生产消息异常, entry:{}", EntryUtils.toString(entry));
        }
    }

    /**
     * 发送延迟消息
     *
     * @param entry 消息实体
     * @param delay 延迟
     * @param unit  时间单位
     */
    public void sendAfter(@NotNull TrackEntry entry, long delay, TimeUnit unit) {
        if (!producer.sendAfter(entry, delay, unit)) {
            logger.error("发送延迟生产消息异常, entry:{}", EntryUtils.toString(entry));
        }
    }

    /**
     * 发送定时消息
     *
     * @param entry 消息实体
     * @param time  未来的时间戳
     */
    public void sendAt(@NotNull TrackEntry entry, long time) {
        if (!producer.sendAt(entry, time)) {
            logger.error("指定时间发送生产消息异常, entry:{}", EntryUtils.toString(entry));
        }
    }
}
```

## 创建消费者

消费者的消费模式支持: 独享模式, 共享模式, 容错模式<br>
消费者的运行模式支持: 后台运行, 定时运行<br>
`@PulsarConsumerConfig`注解可以指定topic, 消费模式(默认为共享模式), 运行模式, 消费者名, 重试队列, 死信队列<br>
`注意`: Pulsar消费者所在Class需指定`@PulsarProcessor`注解, `@PulsarConsumerConfig`的属性`topicKey`用于指定topic的配置Key, `subscribeKey`用于指定消费者名的配置Key, `consumer.start()`代理方法参数返回`true`标识消费成功, 否则消费失败, 如果失败多次失败消息会进入死信队列(默认16次)

以下示例为单实例共享消费的消费者<br>
```java
@Service
@PulsarProcessor
public class TrackConsumer {
    private static final Logger logger = LoggerFactory.getLogger(TrackConsumer.class);

    // 消费线程数
    @Value("${service.mq.executor.track:1}")
    private int executorCount;

    @Resource
    private TrackService trackService;

    @PulsarConsumerConfig(topicKey = "service.mq.topic.track", subscribeKey = "service.mq.subscribe.track")
    private IConsumer<TrackEntry> consumer;

    @PostConstruct
    public void start() {
        if (executorCount < 1) {
            executorCount = 1;
        }
        consumer.start(entry -> {
            return trackService.fillTrack(entry.getCarNo(), entry.getTrackEnd(), DateUtils.dayEnding(entry.getTrackEnd()), entry.getDataProvider()).isSuccess();
        }, executorCount);
    }
}
```
