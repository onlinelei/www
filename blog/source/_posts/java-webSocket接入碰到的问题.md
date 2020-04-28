---
title: webSocket接入碰到的问题
date: 2020-4-28 15:40:46
tags: java
---

## webSocket session 无法序列化存储
项目中希望引入webSocket实现前后端实时通讯，为了解决分布式session的问题希望吧webSocket session存入reids，多端通过账户id去获取session。想法很美好，实现的时候发现session无法序列化，原因是session中有webSocket的长连接，序列化，会导致连接断开，而且即使序列化完成，别的服务从redis中取出session也是不能用的，应为长连接已经断开。

解决方案：session 还是放在每台服务里（做线程安全），每台服务轮询去redis里取当前账户的消息，然后在发送消息。（也可使用mq、redis发布订阅模式）

## webSocket无法注入Bean
事情时这样的，websocket服务端往往需要和服务层打交道，因此需要将服务层的一些bean注入到websocket实现类中使用，但是呢，websocket实现类虽然顶部加上了@Component注解，依然无法通过@Resource和@Autowire注入spring容器管理下的bean。后来就想用ApplicationContext获取spring容器管理下的bean。但是无法获取ApplicationContext的实例，因为该实例也是在spring下管理的，所以就又碰到前面的问题，当时都快崩溃了，这不是个死路吗，又回到原先的问题了。。后来在网上找到了该问题的解决办法，那就是在初始化ApplicationContext实例的时候将该引用保存到websocket类里。如下

``` java
	@Autowired
	private SocketMessageHandler socketMessageHandler;
```

需要了解一个事实：websocket 是多对象的，每个用户的聊天客户端对应 java 后台的一个 websocket 对象，前后台一对一（多对多）实时连接，所以 websocket 不可能像 servlet 一样做成单例的，让所有聊天用户连接到一个 websocket对象，这样无法保存所有用户的实时连接信息。可能 spring 开发者考虑到这个问题，没有让 spring 创建管理 websocket ，而是由 java 原来的机制管理websocket ，所以用户聊天时创建的 websocket 连接对象不是 spring 创建的，spring 也不会为不是他创建的对象进行依赖注入，所以如果不用static关键字，每个 websocket 对象的 service 都是 null。

解决办法：

``` java
@Slf4j
@Component
@ServerEndpoint(value = "/webSocket/{account}")
public class SocketController {

	//  这里使用静态，让 service 属于类
    private static ChatService chatService;

    // 注入的时候，给类的 service 注入
    @Autowired
    public void setChatService(ChatService chatService) {
        ChatSocket.chatService = chatService;
    }
    
	@OnOpen
	public void openSession(Session session, @PathParam("account") 
		// ...
	}

	@OnMessage
	public void onMessage(String message, Session session, @PathParam("account") String account) {
		// ...
	}

	@OnClose
	public void onClose(Session session, @PathParam("account") String account) {
		// ...
	}

	@OnError
	public void onError(Session session, @PathParam("account") String account, Throwable throwable) {
		// ...
	}
}
```
