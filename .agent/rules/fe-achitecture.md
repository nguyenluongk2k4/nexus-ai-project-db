---
trigger: always_on
---

# Frontend Architecture (React + TypeScript + Vite + DDD-lite)

Áp dụng cho:
- React 18 + TypeScript
- Vite
- WebSocket / API
- Chat / AI / Realtime UI
- Đồng bộ kiến trúc với Backend (FastAPI + DDD-lite)

Mục tiêu:
- UI thuần, không dính nghiệp vụ
- Business logic test được
- Đổi transport (REST / WS) không ảnh hưởng UI

---

## 1. Tư duy DDD-lite cho Frontend

### Frontend DDD KHÔNG phải DDD backend
- ❌ Không aggregate
- ❌ Không transaction
- ❌ Không repository phức tạp

### Frontend DDD NÊN tập trung
- Domain = **nghiệp vụ phía UI**
- UseCase = **user interaction**
- Infrastructure = **API / WebSocket**

> UI chỉ render  
> UseCase điều phối  
> Domain quyết định logic

---

## 2. Cấu trúc thư mục tổng thể

frontend/
│
├── src/
│ ├── app/ # App bootstrap
│ │ ├── main.tsx
│ │ ├── App.tsx
│ │ └── providers.tsx
│ │
│ ├── domain/ # CORE (UI business)
│ │ ├── entities/
│ │ │ ├── Message.ts
│ │ │ └── ChatSession.ts
│ │ │
│ │ ├── ports/ # Interface
│ │ │ └── ChatGateway.ts
│ │ │
│ │ └── services/
│ │ └── ChatService.ts
│ │
│ ├── usecases/ # Application layer
│ │ └── SendMessageUseCase.ts
│ │
│ ├── infrastructure/ # Adapter layer
│ │ ├── ws/
│ │ │ └── ChatWsGateway.ts
│ │ │
│ │ └── http/
│ │ └── ChatHttpGateway.ts
│ │
│ ├── ui/ # Presentation
│ │ ├── pages/
│ │ │ └── ChatPage.tsx
│ │ │
│ │ ├── components/
│ │ │ ├── ChatBox.tsx
│ │ │ └── MessageItem.tsx
│ │ │
│ │ └── hooks/
│ │ └── useChat.ts
│ │
│ ├── shared/
│ │ ├── types/
│ │ └── utils/
│ │
│ └── config/
│ └── env.ts
│
└── tests/

yaml
Copy code

---

## 3. App Layer (`app/`)

👉 Bootstrap React + DI

### `main.tsx`
```ts
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")!).render(<App />);
providers.tsx
ts
Copy code
export const chatGateway = new ChatWsGateway();

export const sendMessageUseCase =
  new SendMessageUseCase(
    new ChatService(chatGateway)
  );
4. Domain Layer (domain/) – CORE
4.1 Entities
Message.ts
ts
Copy code
export type Role = "user" | "assistant";

export interface Message {
  role: Role;
  content: string;
}
4.2 Ports (Interface)
ChatGateway.ts
ts
Copy code
export interface ChatGateway {
  send(message: string): Promise<string>;
}
4.3 Domain Service
ChatService.ts
ts
Copy code
export class ChatService {
  constructor(private gateway: ChatGateway) {}

  async chat(message: string): Promise<string> {
    if (!message.trim()) {
      throw new Error("Empty message");
    }
    return this.gateway.send(message);
  }
}
5. UseCase Layer (usecases/)
👉 Mapping trực tiếp hành vi người dùng

SendMessageUseCase.ts
ts
Copy code
export class SendMessageUseCase {
  constructor(private chatService: ChatService) {}

  async execute(message: string) {
    return await this.chatService.chat(message);
  }
}
6. Infrastructure Layer (infrastructure/)
6.1 WebSocket Adapter
ws/ChatWsGateway.ts
ts
Copy code
export class ChatWsGateway implements ChatGateway {
  async send(message: string): Promise<string> {
    // websocket send / receive
    return "AI response";
  }
}
6.2 HTTP Adapter (optional)
ts
Copy code
export class ChatHttpGateway implements ChatGateway {
  async send(message: string): Promise<string> {
    const res = await fetch("/api/chat", {
      method: "POST",
      body: JSON.stringify({ message })
    });
    return (await res.json()).reply;
  }
}
7. UI Layer (ui/) – Dumb UI
useChat.ts
ts
Copy code
export function useChat(useCase: SendMessageUseCase) {
  const [messages, setMessages] = useState<Message[]>([]);

  async function send(text: string) {
    setMessages(m => [...m, { role: "user", content: text }]);
    const reply = await useCase.execute(text);
    setMessages(m => [...m, { role: "assistant", content: reply }]);
  }

  return { messages, send };
}
ChatPage.tsx
tsx
Copy code
export function ChatPage() {
  const { messages, send } = useChat(sendMessageUseCase);

  return <ChatBox messages={messages} onSend={send} />;
}
8. Mapping FE ↔ BE (DDD đồng bộ)
Frontend	Backend
ChatService	SmartChatbot
ChatGateway	LLM / Vector Store
SendMessageUseCase	ChatUseCase
WS Adapter	FastAPI WebSocket

9. Quy tắc import (rất quan trọng)
❌ UI import Infrastructure → SAI
❌ Domain import UI → SAI

✅ UI → UseCase → Domain → Port
✅ Infrastructure → Port