config
```java
@Configuration  
@EnableWebSocketMessageBroker  
public class WebsocketConfig implements WebSocketMessageBrokerConfigurer {   
    @Override  
    public void configureMessageBroker(MessageBrokerRegistry registry) {  
        registry.setApplicationDestinationPrefixes("/app");  
        registry.enableSimpleBroker("/user");  
        registry.setUserDestinationPrefix("/user");  
    }  
}
```

`registry.setUserDestinationPrefix("/user");  `
is used to specify user specific messages
routed at `/user/{username}/queue/some-destination`

Spring internally maps these destinations to WebSocket sessions.

**How It Works in a WebSocket Config**:

- You typically configure it in the WebSocket message broker configuration, like so:
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.setUserDestinationPrefix("/user"); // User-specific prefix
        registry.enableSimpleBroker("/topic", "/queue");
    }
}
```

**Usage Example**:

- If a client subscribes to `"/user/queue/notifications"`, they will receive private messages sent to them at that destination.
- The server can send a message to a specific user like this
```java
messagingTemplate.convertAndSendToUser("username", "/queue/notifications", payload);
```

## example of a full configuration

```java
@Configuration  
@EnableWebSocketMessageBroker  
public class WebsocketConfig implements WebSocketMessageBrokerConfigurer {  
  
    @Override  
    public void configureMessageBroker(MessageBrokerRegistry registry) {  
        registry.setApplicationDestinationPrefixes("/app");  
        registry.enableSimpleBroker("/user");  
        registry.setUserDestinationPrefix("/user");  
    }  
  
    @Override  
    public void registerStompEndpoints(StompEndpointRegistry registry) {  
        registry.addEndpoint("/ws").withSockJS();  
    }  
  
    @Override  
    public boolean configureMessageConverters(List<MessageConverter> messageConverters) {  
        DefaultContentTypeResolver resolver = new DefaultContentTypeResolver();  
        resolver.setDefaultMimeType(MimeTypeUtils.APPLICATION_JSON);  
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();  
        converter.setObjectMapper(new ObjectMapper());  
        converter.setContentTypeResolver(resolver);  
  
        messageConverters.add(converter);  
  
        return false;  
    }  
}
```

The method `configureMessageConverters` returns `false`, which means the default message converters will still be used in addition to the one you've added. If you want to use only your custom converter, you should return `true`.

```java
messagingTemplate.convertAndSendToUser(  
        user.getUsername(),  
        "/queue/messages",  
        notification);
```

this actually send a message to /user/{username}/queue/messages

and now the person who knows his own username can subscribe to it