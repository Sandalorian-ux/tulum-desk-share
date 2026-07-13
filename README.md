# tulum-desk-share

A single self-contained page (`index.html`, no build step) you open in Chrome on the **Blade** to share its
screen into the **Tulum venue** DESK panel over **peer-to-peer WebRTC** on your home LAN.

- **Role:** `sharer` (the venue is `viewer`).
- **Signaling:** the fixed-room mailbox Worker `https://tulum-desk-signal.hartwell.workers.dev`, room `tulum-desk`
  — it only relays the offer/answer/ICE handshake. The **video is P2P and never touches the Worker.**
- **STUN:** `stun:stun.l.google.com:19302` (same as the venue) for LAN candidate gathering.

## Use

1. Open `index.html` in Chrome on the Blade — over **`http://localhost`** (or https). `getDisplayMedia` needs a
   secure context; a bare `file://` may block it in some Chrome builds, so prefer serving it:
   ```bash
   cd ~/Projects/tulum-desk-share && python3 -m http.server 8000
   # then open http://localhost:8000/
   ```
2. Click **Share Screen** and pick a screen/window. The page resets the room and waits for the venue.
3. On the headset, enter the venue and press **A** to cycle to **DESK** mode (mode 5). The panel announces
   `ready`; this page offers; they connect P2P and your screen appears on the panel.
4. **Stop** (or the browser's own "Stop sharing" bar) tears down and clears the room.

## How the handshake flows

```
sharer (this page)                 viewer (venue DESK)
  getDisplayMedia
  POST /reset
  poll inbox  <----------- "ready"  (posted on DESK-on, re-sent until paired)
  createOffer, POST "offer" ------->  setRemoteDescription, createAnswer
  setRemoteDescription  <--- "answer"  POST "answer"
  POST/apply "ice" <--------------> POST/apply "ice"   (trickle both ways)
  RTCPeerConnection connected → video track flows P2P → rendered on the panel
```

The viewer re-announces `ready` until connected, so it works regardless of which side starts first, and
re-entering DESK mode reconnects cleanly (each `ready` triggers a fresh offer, rate-limited by a 2s cooldown).
