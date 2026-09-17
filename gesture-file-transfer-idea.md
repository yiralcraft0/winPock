# Gesture-Based File Transfer for Windows (AirDrop-style)

## Concept
A Windows application that lets users transfer files to nearby devices using hand gestures (e.g., a pinch-and-swipe or "pass" motion), captured via webcam and combined with wireless peer-to-peer transfer — similar in spirit to Apple's AirDrop, but for Windows.

## Core Decisions
- **Trigger**: Hand gesture detected via webcam (e.g., pinch-and-swipe, air-tap, "pass" motion)
- **Transfer mode**: Wireless, no cables — two main architecture options (see below)
- **MVP scope**: Two Windows PCs, webcam-based gesture detection, nearby wireless transfer

## Wireless Transfer: Two Architecture Options

### Option A — Wireless but Local (true AirDrop-style) — recommended for MVP
- No internet required, devices just need to be near each other
- **Discovery**: Bluetooth Low Energy (BLE) or WiFi Direct
- **Transfer**: WiFi Direct (device-to-device WiFi, no router) for actual file transfer — faster than Bluetooth for large files
- **Windows APIs**: `Windows.Devices.Bluetooth`, `Windows.Devices.WiFiDirect` (Windows Runtime)
- No server to run/maintain

### Option B — True Online Transfer (internet-based)
- Devices don't need to be near each other
- Requires a relay/signaling server (WebSocket or WebRTC data channel)
- **NAT traversal**: WebRTC + STUN/TURN servers for direct P2P when possible
- **Fallback**: Relay through your own server if direct P2P fails (costs bandwidth/hosting)
- This is a different product direction — more like a gesture-triggered Dropbox/WeTransfer

## Roadmap

### Phase 0 — Define Scope
- Finalize trigger gesture(s)
- Decide transfer scope: local-only (Option A) vs. internet-capable (Option B)
- Lock MVP: two Windows PCs, webcam gesture detection, LAN/local wireless transfer

### Phase 1 — Tech Stack Decisions
- Gesture recognition: MediaPipe Hands (Python)
- App shell: C# (.NET) for Windows integration
- Networking: BLE/WiFi Direct (Option A) or WebRTC + signaling server (Option B)
- Security: TLS for transfer, device pairing/trust model

### Phase 2 — Core Gesture Engine
- Webcam feed → hand landmarks via MediaPipe
- Define gesture as a landmark trajectory pattern / state machine
- Heuristic rules before considering custom ML
- Debouncing/confirmation to avoid false triggers

### Phase 3 — Peer Discovery + Transfer
- Local peer discovery (BLE/WiFi Direct, or Zeroconf/mDNS-style)
- Chunked file transfer with progress reporting; resume support as a stretch goal
- First-time pairing/trust confirmation (like AirDrop's "Accept?" prompt)

### Phase 4 — Windows Integration
- System tray app (ambient, not a full window)
- Native "incoming file" toast notification with accept/reject
- Drag-and-drop or file-picker tied to gesture trigger
- Auto-start on login, background service handling

### Phase 5 — UX Polish
- Visual feedback during gesture tracking (on-screen hand/status indicator)
- Graceful fallback for poor lighting/camera failure (manual send button)
- Settings: sensitivity, camera selection, trusted devices list

### Phase 6 — Testing & Hardening
- Test across webcams, lighting conditions, hand sizes/skin tones
- Network edge cases: firewalls, VPNs, multiple NICs
- Security review of pairing/transfer protocol

### Phase 7 — Packaging & Distribution
- Installer via Inno Setup or MSIX, CV runtime bundled
- Decide: open source, Microsoft Store, or standalone distribution

## Languages & Skills Needed

### Core Languages
- **Python** — gesture recognition engine (MediaPipe has first-class Python support)
- **C# (.NET)** — Windows app shell (tray icon, notifications, native dialogs, background service)
- **C++** (optional) — only needed later for performance-critical CV work

Common architecture: Python microservice for gesture detection ↔ C# Windows app via local socket/IPC.

### Skills to Build
1. **Computer vision basics** — hand landmark detection (MediaPipe Hands), basic OpenCV (frame capture/preprocessing). No custom ML training needed for v1.
2. **Networking** — socket programming (TCP/UDP), peer/service discovery (BLE, WiFi Direct, or Zeroconf/mDNS), TLS basics.
3. **Windows platform specifics** — Win32/WPF/WinUI for tray + notifications, Windows services/startup behavior, MSIX/Inno Setup packaging.
4. **Systems/architecture thinking** — state machines for gesture logic (ties directly into discrete math/logic work), IPC between Python and C# processes.
5. **Security fundamentals** — device pairing/trust models, encryption-in-transit basics.

### Suggested Learning Order
1. Python + MediaPipe — get a hand landmark demo running (weekend project)
2. Basic socket programming — send a file between two Python scripts on LAN
3. C# + WinUI/WPF fundamentals — build a "hello world" tray app
4. Wire the two together via IPC
5. Circle back to security/pairing once the happy path works

## Open Questions / Next Decisions
- Final call: Option A (local wireless) vs. Option B (internet-based) — Option A recommended for staying true to the original AirDrop-style vision and avoiding server maintenance
- Specific gesture(s) to support for v1
