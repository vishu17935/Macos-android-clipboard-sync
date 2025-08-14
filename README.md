# macOS ↔ Android Clipboard Sync

> **Goal:** Build a private, two-way clipboard synchronizer between a macOS menu-bar app and an Android app. Devices will exchange clipboard text over the internet (not LAN). Pairing uses per-device random codes entered manually; no account system. Primary transport: Firebase Cloud Messaging (FCM). Encryption for privacy.

---

## Quick summary

* Both devices automatically connect to the internet and sync when online.
* macOS app polls the clipboard (2×/sec), detects changes, and sends new text to the paired Android device via FCM.
* Android app runs a foreground service, receives FCM data messages, and writes to the clipboard.
* Two-way flow supported: Android detects local clipboard changes and sends them to macOS via FCM (mac app receives via small backend or directly using FCM REST API).
* Loop prevention, rate limiting, and optional payload encryption are required to make behavior sensible and secure.

---

## Table of Contents

1. Requirements
2. High-level design & architectures
3. Major tasks and subtasks
4. Detailed component design

   * macOS app
   * Android app
   * FCM & backend considerations
   * Pairing & key management
   * Loop prevention & conflict resolution
   * Security & encryption
5. API / message formats
6. Permissions and platform caveats
7. Rate limits, throttling and reliability
8. Testing checklist and debugging tips
9. Deploy / run checklist
10. Example folder structure & tech choices
11. Appendix: Sample curl to send FCM

---

## 1) Requirements

### Functional

* Two-way text-only clipboard sync between macOS and Android.
* Automatic sync when both devices are online; no manual action required beyond pairing.
* Pairing performed by manually exchanging short random device codes (no accounts).
* Messages delivered over the internet (not LAN-only).

### Non-functional

* Low latency (sub-second to a few-seconds typical).
* Conservative CPU and battery impact.
* Reasonable privacy: payloads encrypted in transit and at-rest on relay if possible.
* Robust to temporary offline conditions (queueing/retries).

### Constraints

* No app store distribution (personal use).
* No centralized user-account system.
* Host environment: macOS (desktop/laptop), Android (non-root).

---

## 2) High-level design & architectures

Three viable transport architectures — choose one:

### A. FCM (recommended for NAT simplicity)

* macOS sends clipboard changes to **FCM** via REST API (server key). Android receives push and writes clipboard.
* Android sends clipboard to FCM (server key or backend) targeting macOS.
* **Challenge:** macOS cannot be an FCM client; it *sends* via REST and *receives* either by exposing an HTTP endpoint (bad behind NAT) or by polling a tiny backend that forwards FCM pushes to macOS. Simpler: run a small **relay backend** you control that both sides talk to (mac posts -> backend calls FCM -> Android; Android posts -> backend sends to mac via secure WebSocket or stores for pull by mac).

### B. Mac as direct server + tunnel

* Mac runs server reachable from internet via **Cloudflare Tunnel / ngrok / Tailscale**. Android sends/receives to Mac directly.
* Works if you accept always-on Mac and tunnel. Not workable if hostel Wi‑Fi disallows outbound tunnels (rare) or you want simpler setup.

### C. P2P (WebRTC) — NOT recommended

* Complex NAT traversal, needs STUN/TURN. More work than benefit for clipboard text.

**Recommendation**: Use FCM for push delivery to Android (and optionally to macOS via a small backend). For macOS receiving, either:

* Option 1 (simpler): macOS polls your lightweight backend or opens a secure WebSocket to backend which forwards Android-origin messages; OR
* Option 2 (tunnel): use Tailscale so macOS can be addressed directly from Android; then both devices can use a simple HTTP/WS channel.

---

## 3) Major tasks and subtasks

### A. Project setup & infra

* Create Firebase project & get server credentials (server key, service account).
* Decide whether to run a small relay backend (recommended). Choose host: local dev machine, small VPS, or serverless function.
* If using tunnel approach, set up Tailscale/Cloudflare Tunnel.

### B. macOS app (menu bar)

* Initialize project (SwiftUI/AppKit hybrid or Electron if you prefer web).
* Clipboard poller (2 Hz) + debounce logic.
* Local storage for paired devices & tokens.
* FCM sender (REST wrapper) or backend client.
* WebSocket or polling client for receiving messages (if using relay backend).
* UI: status, paired devices list, generate device code, manual pairing input.
* Launch at login, run as menu-bar app.
* Logging and error handling.

### C. Android app

* Project setup (Kotlin recommended).
* Foreground service with persistent notification to keep the app alive and allowed to write clipboard.
* FCM integration: register for FCM tokens & receive messages.
* Clipboard listener: detect local clipboard changes (only while app is foreground or via appropriate permissions/foreground service if allowed).
* Local store for paired devices and their mac-side tokens/codes.
* UI: pairing screen, status, pause/resume sync.
* Request battery-optimization exemption (manual prompt link to settings).

### D. Pairing & key exchange

* Generate random device code on first run (UUID or short alphanumeric).
* Present code as text + QR (optional). Other device inputs it to pair.
* On pairing, derive symmetric key (e.g., HKDF from seed + device codes) or ask user to accept and generate AES key; store securely in macOS Keychain / Android Keystore.

### E. Messaging, encryption, and format

* Define JSON message envelope (includes source ID, device code, timestamp, nonce, ciphertext, optional HMAC).
* Implement AES-GCM for confidentiality + integrity.
* macOS encrypts payload before sending to FCM. Android decrypts and writes clipboard; vice versa.

### F. Loop prevention + conflict handling

* Attach `source_device_id` and `message_id` to each message.
* On receiving, check `if source_device_id == this_device_id` → ignore.
* Maintain `last_sent` and `last_received` hash or ID to ignore duplicate or echo messages.
* Timestamps: last-wins by default; consider prompting user if conflicts are frequent.

### G. Reliability & retry logic

* Retries with exponential backoff for failed sends.
* Local queue for outbound messages (persist across reboots until acked if needed).
* Optional: server-side short-term storage for messages until delivered.

### H. Testing & monitoring

* Unit tests for cryptography & message handling.
* Integration tests (mac→backend→FCM→Android and Android→backend→mac).
* Logging, telemetry (only local) for debugging.

---

## 4) Detailed component design

### macOS app — design notes

**Language / Framework:** Swift (AppKit or SwiftUI + AppKit glue for clipboard APIs). You can use a minimal Electron app if you prefer JavaScript — but native Swift is more lightweight and appropriate for menu-bar apps.

**Clipboard polling**

* Use `NSPasteboard.general.changeCount` to detect changes. Poll at 500ms (2×/sec).
* Debounce: after detecting change, wait 150–300ms to allow clipboard to stabilize (some apps set multiple clipboard entries).

**Loop prevention algorithm (mac)**

* Keep an in-memory + persistent store of:

  * `last_sent_hash` (hash of last text mac sent)
  * `last_received_hash` (hash of last text mac received and applied to clipboard)
* On clipboard change:

  1. Read text (trim small whitespace? configurable).
  2. Compute hash `h`.
  3. If `h == last_sent_hash` or `h == last_received_hash` → ignore.
  4. Otherwise, encrypt and send to paired devices; set `last_sent_hash = h`.

**Sending to Android via FCM**

* Option A (no backend): macOS calls FCM HTTP API directly, using Server Key. Include `to: <ANDROID_FCM_TOKEN>`.

  * Problem: server key stored locally in the mac binary. Acceptable for personal use.
* Option B (recommended): macOS posts to your small backend API `POST /send` with `{target_code, ciphertext, metadata}`. Backend calls FCM and returns status.

**Receiving from Android**

* If using backend: open a secure WebSocket to backend and receive push-forwarded messages; or poll `/inbox` endpoint.
* If using tunnel/Tailscale and direct connection from Android: run an HTTP/WS listener on the mac app and accept encrypted messages (beware of firewall and backgrounding).

**Key storage**

* Store symmetric keys in macOS Keychain. For pairing secrets, store only the necessary derived key.

**UI**

* Menu bar icon with small dropdown:

  * Toggle sync on/off
  * List of paired devices (show last-seen)
  * Pair new device (show code / enter code)
  * Settings: polling rate, encryption on/off, backend URL, logging

### Android app — design notes

**Language / Framework:** Kotlin

**Foreground service**

* Implement `Service` as `startForegroundService()` with persistent notification.
* Keep a WebSocket/FCM token registration active.

**FCM handling**

* Use Firebase Cloud Messaging library to obtain token and receive messages via `FirebaseMessagingService`.
* Ensure messages are **data messages** (not only notification messages) so the app can process payload even if backgrounded.
* For high reliability, mark messages high priority only when necessary.

**Writing to clipboard**

* When a message arrives and is validated/decrypted, write to `ClipboardManager.setPrimaryClip()`.
* No special permission required to write.
* To detect local clipboard changes: set a `ClipboardManager.OnPrimaryClipChangedListener` when app is running. On Android 10+, reading clipboard in background is restricted. So do local detection only while service is in foreground and visible or while user has opened the app.

**Loop prevention**

* Check `source_device_id` and message ID before applying.
* As an extra: temporarily set a `last_received_hash` and ignore clipboard listener triggers for short timeout after applying remote clipboard content.

**Battery & Doze**

* Request user to disable battery optimization for the app (direct them to Settings → Battery → Battery optimization → Exclude app).
* Use FCM for inbound messages because it’s Doze-friendly; data messages with `priority=high` will be delivered faster but should be used sparingly.

**Key storage**

* Use Android Keystore to store symmetric key and keep it non-exportable.

**UI**

* Simple main activity showing pairing, paired device list, last clipboard, logs, toggle foreground service.

### FCM & Backend considerations

**FCM usage model**

* Use FCM data messages for clipboard payloads. Example wrapper: backend receives encrypted payload from Mac and issues FCM send to Android target token(s).
* Each device must register its FCM token with your backend alongside its device code.

**Authentication & secrets**

* Use a backend to hold your FCM server key and send messages on behalf of clients — avoids embedding server key in mac app.
* Backend should authenticate clients (mac + android) using short-lived tokens or pre-shared secret generated on pairing. Because you control both apps and do not have user accounts, using symmetric shared secret per device is OK.

**Message lifecycle**

* Short-lived store: backend can optionally store last N messages for offline delivery or until device reconnects.
* If backend does not persist, rely on FCM store-for-delivery behavior.

**Alternative (no backend)**

* mac calls FCM directly (server key in binary). Android receives. Android sends to mac by calling FCM targeting a web-client? FCM to mac is not supported. So mac-to-Android direct via FCM is easy, but Android-to-mac needs backend or other mechanism unless you make mac pull content from a cloud store (e.g., Firestore) or use a tunnel.

### Pairing & key management

**Pairing flow (simple, manual)**

1. On first run each device generates `device_code` (short, human-copyable) + device\_id (UUID).
2. To pair: copy the other device's `device_code` into your device's UI + optional passphrase.
3. Pairing process: derive symmetric key (`K`) from both device codes and random nonces (e.g., HKDF of concatenated secrets + random salt) OR have the pairing device send an encrypted ephemeral key.
4. Store `K` in secure storage (Keychain / Keystore).

**Notes:** manual pairing is okay because this is personal use. You may optionally display a QR code for one-click pairing.

### Loop prevention & conflict resolution

**Message envelope fields**

* `message_id` (UUID)
* `source_device_id`
* `target_device_id`
* `timestamp` (UTC iso)
* `nonce` (for AES-GCM)
* `payload` (ciphertext)

**Rules**

* If `source_device_id == this_device_id` ⇒ ignore (message originated here).
* Keep last N message\_ids to detect duplicates.
* If local clipboard content equals decrypted payload ⇒ ignore.
* If both devices change clipboard within a short window, use `timestamp` to pick last-writer-wins.

### Security & encryption

**Transport**

* FCM is TLS encrypted between client ↔ Google and Google ↔ push service.
* But **payloads pass through Google servers**. For privacy, encrypt payloads client-side.

**Recommendation**

* Use **AES-256-GCM** for confidentiality & integrity. Use unique nonce for each message.
* Derive symmetric key per pair during pairing. Store keys in Keychain/Keystore.
* Optional: sign payload with HMAC-SHA256 if not using GCM.

**Key compromise risk**

* If backend holds secrets or keys, protect it. If mac app has server key for FCM embedded, keep it on a machine you trust and don't distribute the binary.

## 5) API / message formats

**Message JSON (unencrypted for clarity — actual payload should be ciphertext)**

```json
{
  "message_id": "uuid",
  "source_device_id": "mac-1234",
  "target_device_id": "phone-5678",
  "timestamp": "2025-08-14T12:00:00Z",
  "nonce": "base64-nonce",
  "ciphertext": "base64-ciphertext"
}
```

**FCM send (simplified)**

* Send to `https://fcm.googleapis.com/fcm/send` (legacy) or `https://fcm.googleapis.com/v1/projects/PROJECT_ID/messages:send` (HTTP v1 API). Use server key or service account.
* In the `data` field place your JSON envelope.

## 6) Permissions and platform caveats

### macOS

* Clipboard read/write: no special permission if not sandboxed.
* Firewall: user must allow incoming connections if you open server sockets.
* Sleep: server stops when Mac sleeps or lid closed. Use `caffeinate` or Energy Saver settings to prevent if needed.

### Android

* `INTERNET` permission required.
* Foreground service required for reliable background operation and clipboard writes.
* Clipboard **reading** is restricted when app is backgrounded on Android 10+; writing is permitted.
* Battery optimizations and Doze: ask user to exclude the app for reliable delivery.

## 7) Rate limits, throttling and reliability

* FCM has soft throttles. Practical advice:

  * Avoid sending identical rapid updates. Debounce clipboard changes and only send when content actually changes and is not the same as last\_sent/received.
  * Use exponential backoff on repeated failures.
  * Avoid using `priority: high` on every message — limit to real clipboard events.

Estimated safe usage for personal: thousands/day is fine; avoid >1 message/sec sustained bursts.

## 8) Testing checklist and debugging tips

* Test loop prevention by copying on device A and ensure device B receives but does not resend back.
* Test collisions: copy on both devices quickly — confirm last-writer-wins.
* Test network loss: bring Android offline, copy on mac, bring Android online — ensure delivery.
* Test FCM token rotation: uninstall/reinstall, ensure token is re-registered with backend.
* Test battery optimizations: see if Android receives pushes while screen off.
* Use verbose logging on both sides; rotate logs so they don’t consume disk.

## 9) Deploy / run checklist

* [ ] Create Firebase project and obtain credentials.
* [ ] Implement backend (or decide to embed server key in mac app).
* [ ] Build mac menu-bar app; test send flow to backend.
* [ ] Build Android foreground servi
