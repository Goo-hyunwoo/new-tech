# Server(NodeJS)

> NodeJS의 소켓 구현체는 FE와 밀접한 라이브러리를 사용해서 가장 짧은 코드로 구성 가능
> Express와 socket.io만 사용하여 구현 가능

```typescript
export interface User {
  id: string;
}

export interface Chat {
  sender: User;
  message: string;
  date: Date;
}

export interface ServerChat extends Chat {
  color: string;
}

export interface ServerToClientEvents {
  fromServerToClientMessage: (data: ServerChat) => void;
}

export interface ClientToServerEvents {
  fromClientToServerMessage: (data: Chat) => void;
}
```

```typescript
import Express from "express";
import http from "http";
import { Server } from "socket.io";
import { Chat, ClientToServerEvents, ServerToClientEvents } from "./types";

const app = Express();
const server = http.createServer(app);
const io = new Server<ClientToServerEvents, ServerToClientEvents>(server);

io.on("connection", (socket) => {
  console.log("a user connected");

  socket.on("fromClientToServerMessage", (data: Chat) => {
    console.log({ ...data, ip: socket.request.connection.remoteAddress });

    io.emit("fromServerToClientMessage", {
      ...data,
      color: "blue" || "",
    });
  });

  socket.on("disconnect", () => {
    console.log("user disconnected");
  });
});

server.listen(9000, () => {
  console.log("listening on *:9000");
});
```


# Server(java)

> Spring에서 제공하는 기본 Socket-starter로는 정상적인 구현을 하지 못했다.
> FE에서 Socket.io-client 라이브러리를 받았기 때문에, RequestResolver를 손봐줘야 할 것 같지만, 조금 더 자세히 공부하게 될 때 구현하기로...

> 증상: /socket.io/?EID=4&transport=websocket -> No handler Mapping

### netty-socketio
> implementation 'com.corundumstudio.socketio:netty-socketio:2.0.8'
> netty로 구현된 socketio 구현체를 이용하여 socket server 구현

```java
public class JacksonJsonSupportImpl extends JacksonJsonSupport{
	public JacksonJsonSupportImpl() {
		super.objectMapper.registerModule(new Jdk8Module()); 
		super.objectMapper.registerModule(new JavaTimeModule());
		super.objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS); 
	}
}
```
> 상위 클래스인 JacksonJsonSupport를 상속하여 변수에 필요한 설정 추가

```java
@org.springframework.context.annotation.Configuration
public class SocketIOConfig {

	@Bean
	public SocketIOServer socketIOServer(JsonSupport jsonSupport) {
		Configuration config = new Configuration();
		config.setHostname("192.168.0.150");
		config.setPort(9001);
		config.setJsonSupport(jsonSupport);

		final SocketIOServer server = new SocketIOServer(config);
		return server;
	}
	
	@Bean
	public JsonSupport jsonSupport() {
		return new JacksonJsonSupportImpl();
	}
}
```
> Config에서 빈 생성

```java
@RequiredArgsConstructor
@Component
public class WebSocketServer {
	
	private final SocketIOServer server;
	private static List<SocketIOClient> clients = Collections.synchronizedList(new ArrayList<SocketIOClient>());
	
    @PostConstruct
    public void startServer() {
        server.addConnectListener(new ConnectListener() {
            @Override
            public void onConnect(SocketIOClient client) {
            	if(clients.contains(client)) return;
            	clients.add(client);
            }
        });

        server.addDisconnectListener(new DisconnectListener() {
            @Override
            public void onDisconnect(SocketIOClient client) {
            	clients.remove(client);
            }
        });
        
        server.addEventListener("fromClientToServerMessage", Chat.class, new DataListener<Chat>() {
            @Override
            public void onData(SocketIOClient client, Chat data, AckRequest ackSender) {
            	data.setColor("blue");
            	clients.forEach(session -> {
            		session.sendEvent("fromServerToClientMessage", data);
            	});
            }
        });
    }
    
    public SocketIOServer getServer() {
    	return this.server;
    }
	
}
```
> 채널 클라이언트 관리
> SERVER에 접속된 사용자 정보 관리
> QQQ 채팅 방이 여러 개인 경우에 대한 로직 필요 // 프론트 역시 마찬가지
> REDIS?
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class NettyServerLifeManager {

	private final WebSocketServer webSocketServer;

    @PostConstruct
    private void startServer() {
    	this.webSocketServer.getServer().start();
    }

    @EventListener
    public void gracefulShutdown(ContextClosedEvent event) {
    	log.debug("Socket 서버의 연결이 종료됩니다."); 
    	this.webSocketServer.getServer().stop();
    }
}
```
> Netty 서버와 Wrapping한 SpringBoot Web과의 Lifecycle 관리를 위한 컴포넌트
> Netty 서버 단독 구성 고려



# REACT(socket.io-client)

> "socket.io-client": "^4.6.1",

> type 선언은 node server와 동일하므로 제외

#### socket.ts
```typescript
import { io, Socket } from "socket.io-client";
import {
  Chat,
  ClientToServerEvents,
  ServerChat,
  ServerToClientEvents,
} from "./type";

export const socket: Socket<ServerToClientEvents, ClientToServerEvents> = io(
  "http://192.168.0.150:9001",
  { transports: ["websocket"] }
);

export const initSocketConnection = () => {
  if (socket.connected) return;
  socket.connect();
};

// 이벤트 명을 지정하고 데이터를 보냄
export const sendSocketMessage = (data: Chat) => {
  if (socket === null || socket.connected === false) {
    initSocketConnection();
  }
  socket.emit("fromClientToServerMessage", data);
};

// 해당 이벤트를 받고 콜백 함수를 실행함
export const socketInfoReceived = (
  prev: ServerChat[],
  cb: (messages: ServerChat[]) => void
) => {
  if (socket.hasListeners("fromServerToClientMessage")) {
    socket.off("fromServerToClientMessage");
  }

  socket.on("fromServerToClientMessage", (data: ServerChat) => {
    cb([...prev, data]);
  });
};

// 소켓 연결을 끊음
export const disconnectSocket = () => {
  if (socket == null || socket.connected === false) {
    return;
  }
  socket.disconnect();
};
```

#### App.tsx
```typescript
  ...
  
  const [messages, setMessages] = useState<ServerChat[]>([]);

  const onClickHandler = (content: string = "") => {
    const chat: Chat = {
      date: new Date(),
      sender: { id: idState },
      message: content,
    };
    sendSocketMessage(chat);
  };
  
  useEffect(() => {
    initSocketConnection();
    return () => {
      disconnectSocket();
    };
  }, []);

  useEffect(() => {
    socketInfoReceived(messages, setMessages);
  }, [messages, setMessages]);
  
  ...
```
