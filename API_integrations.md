

# 🎯 **CASP – How Users Interact with Your App**

CASP supports **two interaction modes**:

1. **Chat Interactions (Text) — OpenAI-style SDKs & REST APIs**
2. **Voice Interactions (Real-time Audio) — WebSocket Streaming (Twilio-compatible)**
2. **App Integrations (OAuth2) — Gmail, Outlook, Slack, Teams**

This section shows **how an app developer can integrate CASP** exactly like they integrate OpenAI or Twilio.

---

# 🧩 **1. Chat Interactions (Text API)**

CASP provides a **Chat Completion API** identical in structure to OpenAI, Qwen, and DeepSeek.

Developers can simply change the **baseURL** to your CASP endpoint.

---

## ✅ **JavaScript (OpenAI-like)**

```js
import CASP from "casp-sdk";

const client = new CASP({
  apiKey: process.env.CASP_API_KEY,
  baseURL: "https://api.casp.ai/v1",
});

const response = await client.chat.completions.create({
  model: "casp-realtime",
  messages: [
    { role: "user", content: "Schedule a meeting tomorrow at 10 AM." }
  ],
});

console.log(response.choices[0].message.content);
```

---

## ✅ **Python (OpenAI-like)**

```python
from casp import CASP

client = CASP(
    api_key="YOUR_KEY",
    base_url="https://api.casp.ai/v1"
)

resp = client.chat.completions.create(
    model="casp-realtime",
    messages=[
        {"role": "user", "content": "Check my unread Gmail messages."}
    ]
)

print(resp.choices[0].message["content"])
```

---

## 🏷️ **API Users Can:**

* Chat with CASP like ChatGPT
* Trigger actions (Emails, Calendar, Jira, Drive, CRM)
* Control their apps by natural language
* Use it inside web/mobile apps using REST or WebSockets
* Receive structured responses (JSON mode)

---

# 🎧 **2. Voice Interactions (Real-Time WebSocket)**

CASP provides a **Twilio-compatible audio WebSocket**.
Developers can directly connect using:

```
wss://api.casp.ai/v1/realtime?model=casp-realtime&api_key=XXX
```

---

# 🆕 **Twilio Start Message Support**

If your user is on a phone call, Twilio sends a **“start” event** before media begins.

CASP consumes this exactly like Twilio Media Streams.

---

## 📥 Example Twilio → CASP Start Message

```json
{
  "event": "start",
  "start": {
    "accountSid": "AC12345",
    "callSid": "CA9876",
    "streamSid": "MZabc123",
    "resource": "user@company.com",
    "conf_name": "GoogleGmail",
    "bearer": "Base64ConfigString"
  }
}
```

---

# 🧠 **What CASP Does With "start" Event**

* Parses accountSid, callSid, streamSid
* Loads correct assistant / resource config
* Builds **greeting**
* Builds **one-shot system prompt**
* Creates a short-lived conversation context
* Responds with greeting through TTS or text

This is powered by your function:

### 🔧 `load_twilio_start_message(fp, data, casp_casual_prompt)`

---

# 📤 **User → CASP (Audio Upload)**

```json
{
  "event": "media",
  "media": {
    "track": "audio",
    "payload": "<base64>"
  },
  "streamSid": "MZabc123"
}
```

---

# 📥 **CASP → User (Audio Response)**

```json
{
  "event": "media",
  "media": {
    "track": "audio",
    "payload": "<base64>"
  }
}
```

---

# 💬 **Unified Text Messages (Transcripts)**

### User → CASP (text)

```json
{
  "event": "transcript",
  "media": { "role": "user", "text": "Hello" },
  "streamSid": "MZabc123"
}
```

### CASP → User (assistant text)

```json
{
  "event": "transcript",
  "media": { "role": "assistant", "text": "How can I help you?" },
  "streamSid": "MZabc123"
}
```

---

# 🔄 **Full Voice Workflow Diagram**

```
Client / Twilio Call
        │
        ▼
Twilio “start” → CASP Backend
        │
load_twilio_start_message()
        │
        ▼
CASP WebSocket Session
        │
   ┌────┴────┐
   ▼         ▼
Audio → CASP    Text → CASP
   │             │
   ▼             ▼
CASP AI Reasoning Engine
   │             │
   ▼             ▼
AI Audio ←────── AI Text
```

---

# 🏠 **Your Final Home Page Sections**

### ✔️ **Chat Interactions (OpenAI-style)**

* JS, Python examples
* SDK integration
* Chat format
* Action calling

### ✔️ **Voice Interactions (Twilio-style WebSocket)**

* Twilio “start” event
* Audio streaming
* Text streaming
* Media pipeline

### ✔️ **Unified API Playground + Examples**

* Send text
* Send audio
* Read events
* Start session

### ✔️ **Developer Guides**

* WebSocket reference
* Twilio integration
* Start message builder
* Session management

---


# 🔐 **3. App Integrations (OAuth2) — Gmail, Outlook, Slack, Teams**

CASP allows developers to connect user accounts (Gmail, Outlook, Teams, Slack, Calendar, Drive, CRM) using **standard OAuth2 flows** so the AI agent can act on behalf of the user.

This section explains:

* How apps authorize CASP
* How access tokens are stored
* How CASP uses OAuth tokens to perform actions
* Developer endpoints for managing OAuth
* Email / Calendar / File actions that the assistant can perform

---

# ⚙️ **OAuth2 Integration Overview**

CASP supports OAuth2 in two modes:

### **1️⃣ CASP-Managed OAuth (recommended)**

You redirect your users to:

```
https://api.casp.ai/v1/oauth/connect/{provider}
```

CASP handles:

* Login
* Consent screen
* Token exchange
* Refresh tokens
* Safe token storage
* Scopes & permissions
* Auto-renewal

### **2️⃣ Custom OAuth (your app manages tokens)**

You handle the OAuth authorization yourself and send tokens to CASP:

```json
POST /v1/oauth/register
{
  "provider": "gmail",
  "access_token": "",
  "refresh_token": "",
  "expires_in": 3600,
  "user_email": "user@gmail.com"
}
```

CASP stores and rotates the tokens securely.

---

# 📬 **OAuth2 Integration: Gmail**

### 🔑 Required Scopes

| Feature     | Scopes                                           |
| ----------- | ------------------------------------------------ |
| Read Email  | `https://www.googleapis.com/auth/gmail.readonly` |
| Send Email  | `https://www.googleapis.com/auth/gmail.send`     |
| Full Access | `https://mail.google.com/`                       |

### 📌 Authorization URL

```
https://accounts.google.com/o/oauth2/v2/auth
?client_id=...
&redirect_uri=...
&response_type=code
&scope=...
&access_type=offline
&prompt=consent
```

### 🔄 Token Exchange

CASP handles:

* Code → Access Token
* Access Token → Refresh Token
* Automatic refresh on expiration

### 📧 Actions CASP Can Perform

| Action            | Example Voice Command                   | Example Chat API Prompt           |
| ----------------- | --------------------------------------- | --------------------------------- |
| Send Mail         | "Send a mail to John about the meeting" | `{action: "gmail.send"}`          |
| Read Unread Mails | "Read my unread messages"               | `"List my last 10 unread emails"` |
| Search Threads    | "Find conversations with Alice"         | `{action: "gmail.search"}`        |

---

# 📧 **OAuth2 Integration: Outlook (Microsoft 365)**

### 🔑 Required Scopes (Microsoft Graph)

| Feature    | Scopes                        |
| ---------- | ----------------------------- |
| Emails     | `Mail.Read`, `Mail.Send`      |
| Calendars  | `Calendars.ReadWrite`         |
| Contacts   | `Contacts.Read`               |
| Teams Chat | `Chat.Read`, `Chat.ReadWrite` |

### 📌 Authorization URL

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
?client_id=...
&redirect_uri=...
&response_type=code
&scope=Mail.Read Mail.Send Calendars.ReadWrite
```

### 🔄 Microsoft Graph Token Exchange

CASP handles:

* Code → Access Token
* Access Token → Refresh Token
* Auto-refresh & rotate

### 📧 Outlook Email Actions

| Action       | Example Voice Command                                 |
| ------------ | ----------------------------------------------------- |
| Send Mail    | “Send an Outlook email to CEO about revenue meetings” |
| Read Inbox   | “Read my Outlook unread emails”                       |
| Find Threads | “Find Outlook mails about invoice payment”            |

---

# 📅 **OAuth2 Integration: Google Calendar / Outlook Calendar**

### 📌 Supported Calendar Actions

| Action           | Example Voice Command               |
| ---------------- | ----------------------------------- |
| Create event     | "Create a meeting tomorrow at 6 PM" |
| Reschedule event | "Move my 3 PM meeting to 4 PM"      |
| Cancel event     | "Cancel my call with Anil"          |
| List agenda      | "What's on my calendar today?"      |

### 🔑 Scopes

* Google: `calendar.readonly`, `calendar.events`
* Microsoft: `Calendars.ReadWrite`

---

# 💬 **OAuth2 Integration: Slack & Teams**

### 🔗 Slack Scopes

| Feature       | Scopes             |
| ------------- | ------------------ |
| Read Channels | `channels:history` |
| Send Messages | `chat:write`       |
| Search        | `search:read`      |

### 🔗 Teams Scopes (Graph API)

| Feature  | Scopes                         |
| -------- | ------------------------------ |
| Chat     | `Chat.Read`, `Chat.ReadWrite`  |
| Channels | `ChannelMessage.ReadWrite.All` |

### 🤖 Example Voice Commands

| App   | Command                                                 |
| ----- | ------------------------------------------------------- |
| Slack | “Send a Slack message to Priya saying I will join late” |
| Teams | “Send a Teams chat reply to Ritika”                     |
| Slack | “Search Slack for updates on Project Phoenix”           |

---

# 📁 **OAuth2 Integration: Google Drive / OneDrive**

### 🔑 Google Drive Scopes

`drive.readonly`, `drive.file`, `drive.metadata.readonly`

### 🔑 Microsoft OneDrive Scopes

`Files.ReadWrite`, `Files.Read.All`

### 📁 CASP Supported File Actions

* "Find the document titled ‘Budget 2024’"
* "Summarize the file I last opened"
* "Search drive for PDF invoices"
* "Upload a file to my drive"

---

# 💠 **4. How CASP Uses OAuth Tokens During Chat & Voice**

Regardless of **chat** or **voice**, every user command enters a unified pipeline:

```
User → CASP AI → Action Executor → Provider API
```

### ⭐ Example Flow (User Saying in Voice Call):

> “Send a mail to Ramesh saying I approved the quotation.”

CASP pipeline:

1. Speech → CASP Realtime (via WebSocket)
2. CASP → Decodes Intent: “gmail.send”
3. CASP → Uses stored OAuth2 Gmail Token
4. CASP → Calls Gmail API action
5. CASP → Responds:
   **“Email sent to Ramesh.”**

   * audio version over Twilio stream

---

# 🧱 **Developer API — OAuth URLs**

### Start OAuth

```
GET /v1/oauth/connect/{provider}
```

### Check OAuth Status

```
GET /v1/oauth/status?provider=gmail
```

### Disconnect OAuth

```
DELETE /v1/oauth/disconnect/{provider}
```

### Webhook (optional)

CASP can notify your backend when token refresh happens.

```
POST /webhooks/oauth/refresh
```

---

# 🧩 **5. OAuth2 Integration Samples for JS / Python**

## JavaScript OAuth Helper

```js
import { CASP } from "casp-sdk";

const authUrl = CASP.oauth.authorize("gmail", {
  redirectUri: "https://yourapp.com/callback"
});

// redirect user to authUrl
```

## Python OAuth Helper

```python
from casp import CASP

auth_url = CASP.oauth.authorize(
    provider="outlook",
    redirect_uri="https://yourapp.com/callback"
)
print(auth_url)
```

---

# 🎉 Your Documentation Now Covers:

### ✔️ Chat API (OpenAI-style)

### ✔️ Voice API (Twilio-style Realtime)

### ✔️ Twilio Start Message Handling

### ✔️ Unified WebSocket Streaming

### ✔️ OAuth2 for Gmail / Outlook / Slack / Teams

### ✔️ Email, Calendar, File, CRM actions

### ✔️ Developer API & Code Samples

---
