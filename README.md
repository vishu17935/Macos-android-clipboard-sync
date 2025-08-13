# Project: Macos-android clipboard sync
*A high-level overview of the architecture, tasks, and potential challenges for a secure, cross-platform clipboard utility.*

---

## 🧩 Core Architecture
The system will use a **client-server model**.  
A central server is required to reliably route messages between devices, which will be behind different firewalls and NATs.  
Peer-to-peer communication over the internet is **not practical** for this use case.

### Key Principles
- **Central WebSocket Server:** Acts as a real-time message broker. Clients (Mac/Android apps) maintain a persistent connection to the server, allowing for instant data push.
- **Stateless Pairing:** No traditional user accounts. Devices are paired using a temporary, unique, human-readable code. The server's only job is to map device IDs together.
- **End-to-End Encryption (E2EE):** Clipboard content is encrypted on the source device and can *only* be decrypted by a paired device. The server will never see the unencrypted data.

---

## 🖥 Backend Server (Node.js + WebSockets)

### Major Tasks
- Setup a Node.js project with a WebSocket library (e.g., `ws`).
- Implement logic to handle new WebSocket connections and assign a unique ID to each.
- Create an endpoint/message type for device pairing (linking two connection IDs).
- Develop the core message routing logic: when an encrypted message is received from one client, forward it to all its paired clients.
- Deploy the server to a hosting service (e.g., Vercel, Heroku, DigitalOcean).

### 💡 Details Often Ignored
- **Connection State Management:** Keep an in-memory map of connections (e.g., `{ "connectionId": { socket, paired_ids: [] } }`).
- **Graceful Disconnects:** Properly handle the `close` event on a WebSocket to remove the client from the map and notify paired devices if necessary.
- **Scalability:** For a personal project, a single server instance is fine. For a larger app, you'd need a message broker like Redis to handle state across multiple server instances.

### 🐞 Common Bugs to Watch For
- **Memory Leaks:** Not cleaning up disconnected client data from the connection map.
- **Zombie Connections:** Implement a heartbeat/ping-pong mechanism to detect and close dead connections.
- **Broadcast Loops:** Accidentally sending a message back to the original sender, creating an infinite loop.

---

## 🍏 macOS App (SwiftUI)

### Major Tasks
- Create a new SwiftUI project configured as a Menu Bar App (`MenuBarExtra`).
- Implement a background timer to periodically check `NSPasteboard.general.changeCount`.
- When `changeCount` changes, read the clipboard content (start with strings).
- Implement client-side E2EE: encrypt the content.
- Establish and maintain a WebSocket connection to the server. Send encrypted data on copy.
- Listen for incoming messages, decrypt them, and write the content to the pasteboard.

### 💡 Details Often Ignored
- **Efficient Polling:** 1-second timer is a good starting point.
- **Duplicate Prevention:** Avoid resending the same content by tracking `changeCount`.
- **Launch on Login:** Make the app start automatically.
- **Code Signing:** Recommended to bypass Gatekeeper warnings.

### 🐞 Common Bugs to Watch For
- **High CPU Usage:** Inefficient clipboard polling.
- **UI Freezing:** Do not perform networking or encryption on the main thread.
- **Notch Issues:** Keep the menu bar icon minimal.
- **Permissions:** App may be blocked by firewalls without user consent.

---

## 🤖 Android App (Kotlin)

### Major Tasks
- Create a new Android project using Kotlin.
- Implement a **Foreground Service** (mandatory for background clipboard access).
- Show a persistent notification while running.
- Register a `ClipboardManager.OnPrimaryClipChangedListener`.
- Implement client-side E2EE, WebSocket connection, and message handling.

### 💡 Details Often Ignored
- **Foreground Service:** Absolutely necessary; user will see a permanent notification.
- **Battery Optimization:** Request `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` to avoid Doze mode killing the service.
- **Network State Changes:** Reconnect WebSocket after Wi-Fi/mobile switch.
- **Duplicate Prevention:** Hash clipboard content to prevent re-uploads.

### 🐞 Common Bugs to Watch For
- **Service Killed by OS:** Some brands like Xiaomi/OnePlus are aggressive with background services.
- **Clipboard Listener Not Firing:** Re-register listener if service restarts.
- **UI Lag:** Use Coroutines for async work.

---
