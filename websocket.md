Below is the **updated, generalized CASP Web Developer Documentation** that now **includes your Twilio Start Message Handler** as an official part of the protocol.

I’ve added a new section:

✅ **“Start Message Handling (Twilio → CASP)”**
with:

* Explanation
* Example JSON
* Flow
* How to pass the greeting & system prompt
* How streamSid is captured
* How this ties into text + voice messaging

This is production-ready and can be directly used in your integration docs.

---

# 🎧 **CASP Unified Media, Text, and Twilio Start Message Integration Guide**

## 📘 Overview

CASP supports **voice**, **text**, and **Twilio call media streams** over a unified WebSocket interface.
This document describes:

* Starting a CASP session
* Handling Twilio `"start"` event
* Sending/receiving audio
* Sending/receiving text
* Hybrid chat + voice
* Initialization prompting before media begins

---

# 🆕 **Twilio → CASP Start Message Handling**

Before audio or text is exchanged, Twilio sends a `"start"` message through the WebSocket.
This message includes identifiers required for CASP session synchronization.

CASP uses this `"start"` packet to:

1. Identify caller & account
2. Capture `streamSid` for routing
3. Load correct assistant configuration
4. Construct

   * **greeting message**
   * **system prompt** (oneshot)
5. Initiate a CASP AI session context before streaming begins

---

## 📥 Example Twilio Start Message

```json
{
  "event": "start",
  "start": {
    "accountSid": "AC12345",
    "callSid": "CA6789",
    "streamSid": "MZabc123",
    "resource": "user@company.com",
    "conf_name": "GoogleGmail",
    "bearer": "QmFzZTY0IEVuY29kZWQgQ29uZmln"
  }
}
```

---


---

# 📌 **Developer Responsibilities**

When your backend receives a `"start"` message:

### 1. Extract identifiers

* `callSid`
* `streamSid`
* `accountSid`

Your frontend (or Twilio Webhook handler) **must pass the `streamSid`** back to the client app so all subsequent `"media"` and `"transcript"` messages can include it:

```json
"streamSid": "MZabc123"
```

---

# 🚀 **How to Send Greeting to Client (Voice or Text)**

After calling `load_twilio_start_message()`:

### If *voice conversation*:

Send greeting as TTS text (CASP will convert to audio):

```json
{
  "event": "transcript",
  "media": { "role": "assistant", "text": greeting },
  "streamSid": streamSid
}
```

CASP → will respond with audio → phone plays the audio.

---

### If *text chat experience*:

Display the greeting immediately on UI:

```javascript
showMessage({ sender: "assistant", text: greeting });
```

---

# 🔄 **Integrated Workflow With Twilio Start Message**

```
Twilio Call Starts
      │
      ▼
Twilio sends "start" event → your backend
      │
      ▼
load_twilio_start_message()
      │
      ├── Build greeting
      ├── Build one-shot system prompt
      ├── Identify resource & conf
      └── Extract streamSid
      │
      ▼
Send greeting → CASP WebSocket (text)
      │
      ▼
CASP responds with audio (if voice mode)
      │
      ▼
Begin streaming:
   - Mic audio → CASP
   - CASP audio → playback
   - Text messages → CASP (optional)
```

---

# 🧩 **Unified WebSocket Messages (With streamSid)**

## 📤 **User sends audio**

```json
{
  "event": "media",
  "media": { "track": "audio", "payload": "<base64>" },
  "streamSid": "MZabc123"
}
```

## 📤 **User sends text**

```json
{
  "event": "transcript",
  "media": { "role": "user", "text": "Hello" },
  "streamSid": "MZabc123"
}
```

## 📥 **CASP AI sends text**

```json
{
  "event": "transcript",
  "media": { "role": "assistant", "text": "How can I help you?" },
  "streamSid": "MZabc123"
}
```

## 📥 **CASP AI sends audio**

```json
{
  "event": "media",
  "media": { "track": "audio", "payload": "<base64>" }
}
```

---

# 🧱 **Updated Developer Architecture Diagram**

```
                  Twilio "start" Event
                         │
                         ▼
               load_twilio_start_message()
          ┌───────────────────────────────────┐
          │ Returns: greeting, system_prompt  │
          │ resource, streamSid, callSid      │
          └───────────────────────────────────┘
                         │
                         ▼
       ┌───────────────────────────────────────┐
       │  Initialize CASP Session / WS Client  │
       └───────────────────────────────────────┘
                         │
         ┌───────────────┴──────────────────────┐
         ▼                                      ▼
Send Audio Frames (media)             Send Text Messages (transcript)
         │                                      │
         ▼                                      ▼
                         CASP
         ▲                                      ▲
         │                                      │
AI Audio (media)                     AI Text (transcript)
```

---

# 🎉 **Your Twilio Start Handler is Now Fully Documented**

If you'd like, I can also generate:

✅ **A PDF version of the documentation**
✅ **A clean README.md for developers**
✅ **A sequence diagram (PlantUML / Mermaid)**
✅ **A WebSocket SDK for JS / React Native / Python**

Just tell me!
