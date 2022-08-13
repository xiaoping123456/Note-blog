# SpringBoot集成RabbitMQ---保证消息可靠性

在我们使用RabbitMQ消息队列时，使用生产者发送消息，可能出现发送失败，rabbitMQ宕机，一个消息重复发送等问题，一旦出现这种问题，如果不进行相应处理，就可能导致消息丢失，消费者重复消费一条信息等问题，消息就变得不可靠了，进而影响我们的业务逻辑。所以我们要对可能出现的问题进行相应的操作，来保证消息的可靠性。下面我们来介绍在生产端实现**消息可靠性投递**，**消费端幂等性操作**两种实现方式。

本篇文章是springboot集成RabbitMQ中保证消息可靠性的一种方案，还有其他的高可用方案（大家可以去网上study一下）



## 一、生产端消息可靠性投递

生产端消息可靠性传递是为了解决消息投递时丢失的问题，导致这种问题的原因不单一，比如rabbitMQ宕机，投递过程中交换机发生问题等。

### 生产端消息可靠性投递实现方案：

数据库中核心存储消息的唯一id，消息状态，重试次数，重试时间

如果生产者判定消息发送失败，则进行重试（重新发送），并设置重试时间（也就是超过这个时间才重新发送，这样可以尽量排除因系统延迟原因导致生产者判定消息发送失败），当然不能无限重试，要设定最大重试次数，如果超过最大重试次数，就判定为发送失败，不能重试

发送消息前，存储消息状态（投递中），然后生产者发送消息给MQ，若生产者监听到MQ的接收确认，则更改数据库中消息的状态为发送成功；若监听到MQ的接收拒绝，则更改数据库中的消息状态为发送失败（可以加上其他业务）。

开启一个定时任务，获取所有投递中，并且重试时间小于当前时间的消息集合。然后遍历这个消息集合，判断重试次数是否超过最大可重试次数，如果超过最大重试次数，就判定为发送失败，不能重试；如果不超过，消息的重试次数+1，修改时间为当前时间，重试时间为当前时间加上消息超时时间，重新发送消息；



![image-20220429165419752](https://picgo111.oss-cn-beijing.aliyuncs.com/image-20220429165419752.png)



step1:消息数据进行存储

step2:发送消息

step3:生产者监听到MQ的确认消息

step4:更改数据库中消息的状态

step5:定时任务（定时去获取数据库中消息状态为投递中状态，并且重试时间小于当前时间的消息）

step6:如果重试次数不超过3次，消息的重试次数+1，更新重试时间为当前时间+指定超时时间

step7:如果重试次数超过3次，更新状态为投递失败，不能重试

### 代码实现

为了不发生错乱 先写一下代码中会用到的参数

```java
public class MailConstants {

    //消息投递中
    public static final Integer DELIVERING = 0;
    //消息投递成功
    public static final Integer SUCCESS = 1;
    //消息投递失败
    public static final Integer FAILURE = 2;
    //最大重试次数
    public static final Integer MAX_TRY_COUNT = 3;
    //消息超时时间 单位min
    public static final Integer MSG_TIMEOUT = 1;
    //队列
    public static final String MAIL_QUEUE_NAME = "mail.queue";
    //交换机
    public static final String MAIL_EXCHANGE_NAME = "mail.exchange";
    //路由键
    public static final String MAIL_ROUTING_KEY_NAME = "mail.routing.key";
}
```



1. 消息落库，向rabbitMq发送消息前，把要发送的消息持久化，即保存到数据库中，核心参数：唯一的消息id，消息状态（消息投递中，投递成功，投递失败），重试次数，重试时间；

   ```java
   public class MailLog implements Serializable {
   
       private static final long serialVersionUID = 1L;
   
       @ApiModelProperty(value = "消息id")
       private String msgId;
   
       @ApiModelProperty(value = "接收员工id")
       private Integer eid;
   
       @ApiModelProperty(value = "状态（0:消息投递中 1:投递成功 2:投递失败）")
       private Integer status;
   
       @ApiModelProperty(value = "路由键")
       private String routeKey;
   
       @ApiModelProperty(value = "交换机")
       private String exchange;
   
       @ApiModelProperty(value = "重试次数")
       private Integer count;
   
       @ApiModelProperty(value = "重试时间")
       private LocalDateTime tryTime;
   
       @ApiModelProperty(value = "创建时间")
       private LocalDateTime createTime;
   
       @ApiModelProperty(value = "更新时间")
       private LocalDateTime updateTime;
   
   
   }
   ```

   

2. 发送消息时带上唯一的消息id

```java
//emp为实体类（传递的内容），msgId为唯一消息ID(
String msgId = UUID.randomUUID().toString();
rabbitTemplate.convertAndSend(MailConstants.MAIL_EXCHANGE_NAME,MailConstants.MAIL_ROUTING_KEY_NAME,emp,new CorrelationData(msgId));
```

3. 生产端配置RabbitTemplate,配置MQ的消息接收确认回调和消息接收拒绝回调

   ```java
   @Bean
       public RabbitTemplate rabbitTemplate(){
           RabbitTemplate rabbitTemplate = new RabbitTemplate(cachingConnectionFactory);
           /**
            * 消息接收确认回调，确认消息是都到达broker
            * data:消息唯一标识
            * ack:确认结果
            * cause:失败原因
            */
           rabbitTemplate.setConfirmCallback((data,ack,cause)->{
               String msgId = data.getId();
               if (ack){
                   LOGGER.info("{}=====>消息发送成功",msgId);
                   mailLogService.update(new UpdateWrapper<MailLog>().set("status",1).eq("msgId",msgId));
               }else{
                   LOGGER.error("{}=====>消息发送失败",msgId);
               }
           });
   
           /**
            * 消息接收拒绝回调，比如router不到queue时回调
            * msg:消息主体
            * repCode:响应码
            * repTest:相应描述
            * exchange:交换机
            * routingkey:路由键
            */
           rabbitTemplate.setReturnCallback((msg,repCode,repText,exchange,routingkey)->{
               LOGGER.error("{}=====>消息发送queue时失败",msg.getBody());
           });
           return rabbitTemplate;
       }
   ```

4. 开启定时任务，获取所有投递中，并且重试时间小于当前时间的消息集合。然后遍历这个消息集合，判断重试次数是否超过最大可重试次数

   ```java
   	/**
        * 10s执行1次
        */
       @Scheduled(cron = "0/10 * * * * ?")
       public void mailTask(){
           //获取所有投递中的消息 并且重试时间小于当前时间
           List<MailLog> list = mailLogService.list(new QueryWrapper<MailLog>()
                   .eq("status", 0).lt("tryTime", LocalDateTime.now()));
   
           list.forEach(mailLog -> {
               //如果重试次数超过3次，更新状态为投递失败，不能重试
               if (mailLog.getCount()>= MailConstants.MAX_TRY_COUNT){
                   mailLogService.update(new UpdateWrapper<MailLog>().set("status",2).eq("msgId",mailLog.getMsgId()));
               }
               //消息的重试次数+1，修改时间为当前时间，重试时间为当前时间加上消息超时时间
               mailLogService.update(new UpdateWrapper<MailLog>().set("count",mailLog.getCount()+1).set("updateTime",
                       LocalDateTime.now()).set("tryTime",LocalDateTime.now().plusMinutes(MailConstants.MSG_TIMEOUT))
                       .eq("msgId",mailLog.getMsgId()));
               //获取员工的id
               Employee emp = employeeService.getEmployee(mailLog.getEid()).get(0);
               //发送消息
               rabbitTemplate.convertAndSend(MailConstants.MAIL_EXCHANGE_NAME,MailConstants.MAIL_ROUTING_KEY_NAME,emp,
                       new CorrelationData(mailLog.getMsgId()));
           });
   
       }
   ```

   

## 二、消费端幂等性操作

**幂等性：通俗的说就是一个接口，多次发起同一个请求，必须保证操作只能执行一次**

有这么一种情况：当刚刚执行完发送消息给MQ,MQ接收到了，但是数据库中的消息状态还未来得及更改为已接收，此时定时任务刚好执行，那么就会判定这条消息发送失败，需要重新发送。这种情况下，这一条消息就给MQ发送了两次，很显然，会对系统的业务逻辑造成影响。我们需要在消费端进行幂等性操作，具体步骤如下：

### 消费端幂等性操作方案

当消费端监听到一条消息时，去redis中查找该消息的唯一id，若为查找到，则证明第一次接收这条消息，并在redis中保存这条消息的唯一id，回调接收确认；若查找到，则证明不是第一次接收这条消息，回调接收失败；

### 代码实现

1. rabbitmq必须开启手动确认

   ```yaml
   rabbitmq:
       listener:
         simple:
           # 开启手动确认
           acknowledge-mode: manual
   ```

   

2. 消费端监听时 若接收到一条消息，去redis中查找该消息的唯一id，若为查找到，则证明第一次接收这条消息，并在redis中保存这条消息的唯一id，回调接收确认；若查找到，则证明不是第一次接收这条消息，回调接收失败；

   ```java
   @RabbitListener(queues = MailConstants.MAIL_QUEUE_NAME)
       public void handler(Message message, Channel channel){
           Employee employee = (Employee) message.getPayload();
           MessageHeaders headers = message.getHeaders();
           //消息序号
           long tag = (long) headers.get(AmqpHeaders.DELIVERY_TAG);
           //headers.get("spring_returned_message_correlation")获取的是发送消息时 加上的new CorrelationData(msgId)里的msgId
           String msgId = (String) headers.get("spring_returned_message_correlation");
           HashOperations hashOperations = redisTemplate.opsForHash();
   
           try {
               if (hashOperations.entries("mail_log").containsKey(msgId)){
                   LOGGER.error("消息已经被消费=======>{}",msgId);
                   /**
                    * 手动确认消息
                    * tag:消息序号
                    * multiple:是否确认多条
                    */
                   channel.basicAck(tag,false);
                   return;
               }
   
   
               MimeMessage mimeMessage = javaMailSender.createMimeMessage();
               MimeMessageHelper helper = new MimeMessageHelper(mimeMessage);
               //发件人
               helper.setFrom(mailProperties.getUsername());
               //收件人
               helper.setTo(employee.getEmail());
               //主题
               helper.setSubject("入职欢迎邮件");
               //发送日期
               helper.setSentDate(new Date());
               //邮件内容
               Context context = new Context();
               context.setVariable("name",employee.getName());
               context.setVariable("posName",employee.getPosition().getName());
               context.setVariable("joblevelName",employee.getJoblevel().getName());
               context.setVariable("departmentName",employee.getDepartment().getName());
               String mail = templateEngine.process("mail",context);
               helper.setText(mail,true);
               //发送邮件
               javaMailSender.send(mimeMessage);
               LOGGER.info("邮件发送成功");
               //将消息id存入redis
               hashOperations.put("mail_log",msgId,"OK");
               //手动确认消息
               channel.basicAck(tag,false);
   
           } catch (Exception e) {
               /**
                * 手动确认消息 拒绝
                * tag:消息序号
                * multiple:是否确认多条
                * requeue:是否退回队列
                */
               try {
                   channel.basicNack(tag,false,true);
               } catch (IOException ioException) {
                   LOGGER.error("邮件发送失败=======》{}",e.getMessage());
               }
               LOGGER.error("邮件发送失败=======》{}",e.getMessage());
           }
       }
   ```

   