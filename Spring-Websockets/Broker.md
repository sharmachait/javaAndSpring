after adding the dependency for websocket
we need to configure the app by implementing the websocket configuration

use endpoint wss for secure ssl endpoints
```java
@Configuration  
@EnableWebSocketMessageBroker  
public class WebsocketConfig implements WebSocketMessageBrokerConfigurer {  
  
    @Override  
    public void registerStompEndpoints(StompEndpointRegistry registry) {  
        registry.addEndpoint("/ws").withSockJS();  
    }  
  
    @Override  
    public void configureMessageBroker(MessageBrokerRegistry registry) {  
        registry.setApplicationDestinationPrefixes("/app");  
        registry.enableSimpleBroker("/topic");  
    }  
}
```

**`registry.setApplicationDestinationPrefixes("/app")`**:
- Defines a prefix for client-to-server messages (application destinations).
- For example, if a client sends a message to `/app/chat`, it is routed to a controller method (mapped with `@MessageMapping("/chat")`).
**`registry.enableSimpleBroker("/topic")`**:
- Activates a simple in-memory message broker that handles broadcasting messages to subscribers.
- Messages sent to destinations starting with `/topic` are routed through this broker.
- For example, if a client subscribes to `/topic/updates`, they receive messages published to that destination.
```javascript
import SockJS from 'sockjs-client'; // Import SockJS client 
import Stomp from '@stomp/stompjs'; // Import STOMP client

const socket = new SockJS('/ws'); // Establish connection with the server
const stompClient = Stomp.over(socket); // Use STOMP over WebSocket

stompClient.connect({}, function (frame) {
    console.log('Connected: ' + frame);
});

stompClient.subscribe('/topic/updates', function (message) {
    const content = JSON.parse(message.body); // Parse the incoming message
    console.log('Received update:', content); // Handle the message
});
```

```java
@Autowired
private SimpMessagingTemplate messagingTemplate;

@PostMapping("/sendUpdate")
public void sendUpdate(@RequestBody Update update) {
    messagingTemplate.convertAndSend("/topic/updates", update);
}
```

next thing we will have to configure event listeners to handle leaving the chat etc

```java
@Component  
@RequiredArgsConstructor  
@Slf4j  
public class WebSocketEventListener { 
	@EventListener
    public void handleWebSocketDisconnectListener(  
        SessionDisconnectEvent disconnectEvent  
    ){  
	    StompHeaderAccessor accessor = StompHeaderAccessor.wrap(event.getMessage());  
		String username = (String)accessor.getSessionAttributes().get("username");  
		if(username!=null && !username.isBlank()){  
		    log.info("User Disconnected: " + username);  
		    ChatMessage chatMessage = ChatMessage.builder()  
		            .messagetype(MessageType.LEAVE)  
		            .sender(username)  
		            .build();  
		    messageSender.convertAndSend("/topic/public", chatMessage);  
		}
    }  
}
```

next we have to create websocket controller where we use MessageMapping instead of Get or Post mapping
we also define here which topic or queue to redirect the message to
```java
public class ChatController {  
    @MessageMapping("/chat/message")  
    @SendTo("/topic/public")  
    public ChatMessage sendMessage(  
            @Payload ChatMessage message  
    ) {  
        return message;  
    }  
}
```
to play around with headers of the STOMP protocol message use **`SimpMessageHeaderAccessor`**
for something like add user to the session
```java
@MessageMapping("/chat/join")  
@SendTo("/topic/public")  
public ChatMessage addMessage(  
        @Payload ChatMessage message,  
        SimpMessageHeaderAccessor headerAccessor  
) {  
    headerAccessor.getSessionAttributes().put("username", message.getSender());  
    return message;  
}
```