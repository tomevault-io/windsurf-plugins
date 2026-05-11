---
trigger: always_on
description: Guidelines for implementing real-time communication in Armature.
---


# Real-time Features: WebSocket & SSE

Guidelines for implementing real-time communication in Armature.

## Choosing Between WebSocket and SSE

| Feature | WebSocket | SSE |
|---------|-----------|-----|
| Direction | Bidirectional | Server → Client only |
| Protocol | Custom | HTTP |
| Reconnection | Manual | Automatic |
| Browser Support | Good | Excellent |
| Proxy Friendly | Sometimes | Yes |
| Use Case | Chat, Games | Notifications, Feeds |

## WebSocket Implementation

### Basic WebSocket Handler

```rust
use armature::websocket::{WebSocket, Message, WebSocketUpgrade};

#[controller("/ws")]
pub struct WebSocketController;

impl WebSocketController {
    #[get("/")]
    async fn connect(&self, ws: WebSocketUpgrade) -> WebSocketResponse {
        ws.on_upgrade(|socket| handle_socket(socket))
    }
}

async fn handle_socket(mut socket: WebSocket) {
    // Send welcome message
    socket.send(Message::Text("Connected!".to_string())).await.ok();

    // Message loop
    while let Some(msg) = socket.recv().await {
        match msg {
            Ok(Message::Text(text)) => {
                // Echo back
                socket.send(Message::Text(format!("Echo: {}", text))).await.ok();
            }
            Ok(Message::Binary(data)) => {
                // Handle binary data
                socket.send(Message::Binary(data)).await.ok();
            }
            Ok(Message::Ping(data)) => {
                socket.send(Message::Pong(data)).await.ok();
            }
            Ok(Message::Close(_)) => break,
            Err(e) => {
                eprintln!("WebSocket error: {}", e);
                break;
            }
            _ => {}
        }
    }
}
```

### Room-Based Chat

```rust
use std::sync::Arc;
use tokio::sync::{broadcast, RwLock};
use std::collections::HashMap;

#[derive(Clone)]
pub struct ChatRooms {
    rooms: Arc<RwLock<HashMap<String, broadcast::Sender<ChatMessage>>>>,
}

#[derive(Clone, Debug, Serialize, Deserialize)]
pub struct ChatMessage {
    pub room: String,
    pub user: String,
    pub content: String,
    pub timestamp: u64,
}

impl ChatRooms {
    pub fn new() -> Self {
        Self {
            rooms: Arc::new(RwLock::new(HashMap::new())),
        }
    }

    pub async fn join(&self, room: &str) -> broadcast::Receiver<ChatMessage> {
        let mut rooms = self.rooms.write().await;

        if let Some(tx) = rooms.get(room) {
            tx.subscribe()
        } else {
            let (tx, rx) = broadcast::channel(100);
            rooms.insert(room.to_string(), tx);
            rx
        }
    }

    pub async fn send(&self, message: ChatMessage) -> Result<(), Error> {
        let rooms = self.rooms.read().await;

        if let Some(tx) = rooms.get(&message.room) {
            tx.send(message).map_err(|_| Error::ChannelClosed)?;
        }

        Ok(())
    }

    pub async fn leave(&self, room: &str) {
        let mut rooms = self.rooms.write().await;

        if let Some(tx) = rooms.get(room) {
            if tx.receiver_count() == 0 {
                rooms.remove(room);
            }
        }
    }
}

#[controller("/ws/chat")]
pub struct ChatController {
    rooms: ChatRooms,
}

impl ChatController {
    #[get("/:room")]
    async fn join_room(
        &self,
        room: Path<String>,
        user: Query<UserQuery>,
        ws: WebSocketUpgrade,
    ) -> WebSocketResponse {
        let rooms = self.rooms.clone();
        let room_name = room.to_string();
        let username = user.name.clone();

        ws.on_upgrade(move |socket| async move {
            handle_chat_socket(socket, rooms, room_name, username).await
        })
    }
}

async fn handle_chat_socket(
    socket: WebSocket,
    rooms: ChatRooms,
    room: String,
    user: String,
) {
    let (mut sender, mut receiver) = socket.split();
    let mut rx = rooms.join(&room).await;

    // Announce join
    let join_msg = ChatMessage {
        room: room.clone(),
        user: "system".to_string(),
        content: format!("{} joined the room", user),
        timestamp: timestamp_now(),
    };
    rooms.send(join_msg).await.ok();

    // Spawn task to forward broadcast messages to client
    let room_clone = room.clone();
    let send_task = tokio::spawn(async move {
        while let Ok(msg) = rx.recv().await {
            let json = serde_json::to_string(&msg).unwrap();
            if sender.send(Message::Text(json)).await.is_err() {
                break;
            }
        }
    });

    // Receive messages from client
    let rooms_clone = rooms.clone();
    let user_clone = user.clone();
    let room_clone2 = room.clone();
    let recv_task = tokio::spawn(async move {
        while let Some(Ok(Message::Text(text))) = receiver.next().await {
            let msg = ChatMessage {
                room: room_clone2.clone(),
                user: user_clone.clone(),
                content: text,
                timestamp: timestamp_now(),
            };
            rooms_clone.send(msg).await.ok();
        }
    });

    // Wait for either task to complete
    tokio::select! {
        _ = send_task => {}
        _ = recv_task => {}
    }

    // Announce leave
    let leave_msg = ChatMessage {
        room: room.clone(),
        user: "system".to_string(),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
