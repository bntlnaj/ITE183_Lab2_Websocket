# Laboratory 2: WebSocket Server

**Course:** ITE183 – Web Systems and Technologies  
**Student Name:** Aaron John B. Bantilan  
**Date:** October 26, 2026  
**GitHub Repository:** [ITE183_Lab2_Websocket](https://github.com/bntlnaj/ITE183_Lab2_Websocket)

---

## Table of Contents

- [I. Executive Summary and Objectives](#i-executive-summary-and-objectives)
- [II. Environment Setup and Prerequisites](#ii-environment-setup-and-prerequisites)
- [III. Repository and Project Structure](#iii-repository-and-project-structure)
- [IV. Source Code Implementation](#iv-source-code-implementation)
- [V. Execution and Verification Deliverables](#v-execution-and-verification-deliverables)
- [VI. Technical Discussion and Protocol Analysis](#vi-technical-discussion-and-protocol-analysis)
- [VII. Conclusion](#vii-conclusion)

---

## I. Executive Summary and Objectives

The primary objective of this laboratory activity is to construct, configure, and evaluate an event-driven **WebSocket Server** using the **Bonezegei Scripting Language (BSL v1.3.3)** to facilitate real-time, full-duplex communication across multiple web clients.

### Key Objectives Achieved

1. **Event-Driven WebSocket Server Construction:** Built a full-duplex socket engine using the `http` package in BSL.
2. **Lifecycle Management:** Handled HTTP-to-WebSocket connection upgrades, tracked unique client IDs using `onConnect` and `onDisconnect`, and responded to inbound data streams.
3. **Multi-Client Message Broadcasting:** Implemented global payload broadcasting through `server.sendAll()` to enable real-time chat communication.
4. **Static HTTP Serving and Routing:** Served frontend assets through `public/index.html` while routing undefined HTTP paths to a custom 404 response through `public/404.html`.
5. **Verification and Testing:** Verified concurrent socket connections, message delivery, and server logs across multiple active browser sessions.

---

## II. Environment Setup and Prerequisites

The following tools and dependencies were used to configure and run the WebSocket server.

| Requirement / Tool | Configuration Details |
|---|---|
| Operating System | Windows OS |
| Scripting Language | Bonezegei Script Interpreter v1.3.3 |
| Package Manager | `bzg` |
| Dependencies | `http` (`^1.0.0`) |
| Server Port | `8000` |
| Frontend | HTML, CSS, and JavaScript |
| Repository | [GitHub Repository](https://github.com/bntlnaj/ITE183_Lab2_Websocket) |
| License | MIT |

### Installation and Execution

Open PowerShell in the project directory and execute the following commands.

**1. Verify the Bonezegei interpreter version**

```powershell
bonezegei -v
```

**2. Install project dependencies**

```powershell
bzg install
```

**3. Start the WebSocket server**

```powershell
bzg run
```

Once the server is running, open the following address in a web browser:

`http://localhost:8000`

To test the custom 404 page, open:

`http://localhost:8000/nonexistent`

---

## III. Repository and Project Structure

The project follows a standard file layout configured for the Bonezegei package manager and static HTTP asset resolution.

```text
ITE183_Lab2_Websocket/
├── .gitattributes
├── package.json
├── app.bzg
├── README.md
├── LICENSE
├── documentation/
│   ├── 01_Bonezegei_Version_1.3.3.png
│   ├── 02_Dependency_Installation.png
│   ├── 03_WebSocket_Server_Startup.png
│   ├── 04_Server_Connection_and_Message_Logs.png
│   ├── 05_Multi_Client_Message_Broadcasting.png
│   ├── 06_Custom_404_Page.png
│   └── 07_GitHub_Repository.png
└── public/
    ├── index.html
    └── 404.html
```

### File Descriptions

| File or Directory | Description |
|---|---|
| `.gitattributes` | Configures GitHub language classification for `.bzg` files. |
| `package.json` | Defines project metadata, scripts, and dependencies. |
| `app.bzg` | Contains the main HTTP server and WebSocket event handlers. |
| `public/index.html` | Provides the browser-based WebSocket chat interface. |
| `public/404.html` | Displays the custom page for undefined routes. |
| `documentation/` | Stores screenshots documenting execution and verification. |
| `README.md` | Documents the project, implementation, and test results. |
| `LICENSE` | Contains the repository's license terms. |

---

## IV. Source Code Implementation

### 1. `.gitattributes`

This file configures GitHub language highlighting so that `.bzg` files are recognized under JavaScript syntax highlighting.

```gitattributes
*.bzg linguist-language=JavaScript
```

### 2. `package.json`

This file specifies the project metadata, main script, start command, dependency declarations, and license.

```json
{
  "name": "ite183-lab2-websocket",
  "version": "1.0.0",
  "description": "Bonezegei WebSocket Server",
  "main": "app.bzg",
  "scripts": {
    "start": "bonezegei app.bzg"
  },
  "dependencies": {
    "http": "^1.0.0"
  },
  "license": "MIT"
}
```

### 3. `app.bzg`

This file contains the main server implementation. It handles HTTP routing, serves the frontend files, manages WebSocket connections, sends welcome messages, broadcasts incoming messages, and tracks client disconnections.

```javascript
include("lib/http.bzg");

var server = http(8000);

// ==========================================
// HTTP EVENT HANDLERS
// ==========================================

server.get = function (path, data) {
    print("get loc:" + pat

    if (path == "/") {
        server.sendFile("public/index.html");
    } else {
        server.sendFile("public/404.html");
    }
};

// ==========================================
// WEBSOCKET (WS) EVENT HANDLERS
// ==========================================

// Triggered when a client successfully connects via ws://
server.onConnect = function (clientId) {
    print("WebSocket connected! Client ID: " + clientId);

    // Send a welcome message to this specific client
    server.send(clientId, "Welcome to WebSocket server!");
};

// Triggered when a client sends a message
server.onData = function (message, clientId) {
    print("Received from Client " + clientId + ": " + message);

    // Broadcast message to all connected clients
    server.sendAll("User " + clientId + " says: " + message);
};

// Triggered when a client disconnects
server.onDisconnect = function (clientId) {
    print("WebSocket disconnected! Client ID: " + clientId);
};

// Start the server
server.run();
```

### 4. `public/index.html`

This file provides the browser-based chat interface. It establishes a WebSocket connection, updates the connection status, enables message input after connecting, and displays sent and received messages.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BSL WebSocket Test Client</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 30px auto;
            padding: 0 20px;
        }

        #status {
            font-weight: bold;
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 15px;
            background-color: #f0f0f0;
        }

        #controls {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
        }

        input[type="text"] {
            flex-grow: 1;
            padding: 8px;
        }

        button {
            padding: 8px 16px;
            cursor: pointer;
        }

        #log {
            border: 1px solid #ccc;
            height: 250px;
            overflow-y: auto;
            padding: 10px;
            background-color: #fafafa;
            font-family: monospace;
        }

        .sent {
            color: #0066cc;
        }

        .received {
            color: #008800;
        }

        .system {
            color: #888888;
            font-style: italic;
        }
    </style>
</head>

<body>
    <h2>BSL WebSocket Client</h2>

    <div id="status">Status: Connecting...</div>

    <div id="controls">
        <input
            type="text"
            id="msgInput"
            placeholder="Enter message..."
            disabled
        />

        <button id="sendBtn" disabled>Send</button>
    </div>

    <div id="log"></div>

    <script>
        const wsUrl = "ws://localhost:8000";
        const socket = new WebSocket(wsUrl);

        const statusDiv = document.getElementById("status");
        const logDiv = document.getElementById("log");
        const msgInput = document.getElementById("msgInput");
        const sendBtn = document.getElementById("sendBtn");

        function appendLog(text, className) {
            const entry = document.createElement("div");

            entry.className = className;
            entry.textContent = text;

            logDiv.appendChild(entry);
            logDiv.scrollTop = logDiv.scrollHeight;
        }

        socket.onopen = function () {
            statusDiv.textContent = "Status: Connected to " + wsUrl;
            statusDiv.style.backgroundColor = "#d4edda";
            statusDiv.style.color = "#155724";

            msgInput.disabled = false;
            sendBtn.disabled = false;

            appendLog(
                "[System] Handshake complete. WebSocket ready.",
                "system"
            );
        };

        socket.onmessage = function (event) {
            appendLog("[Server] " + event.data, "received");
        };

        socket.onclose = function (event) {
            statusDiv.textContent = "Status: Disconnected";
            statusDiv.style.backgroundColor = "#f8d7da";
            statusDiv.style.color = "#721c24";

            msgInput.disabled = true;
            sendBtn.disabled = true;

            appendLog(
                "[System] Connection closed by server.",
                "system"
            );
        };

        socket.onerror = function (error) {
            appendLog(
                "[Error] WebSocket error occurred.",
                "system"
            );
        };

        function sendMessage() {
            const text = msgInput.value.trim();

            if (
                text.length > 0 &&
                socket.readyState === WebSocket.OPEN
            ) {
                socket.send(text);

                appendLog("[Client] " + text, "sent");

                msgInput.value = "";
            }
        }

        sendBtn.addEventListener("click", sendMessage);

        msgInput.addEventListener("keypress", function (e) {
            if (e.key === "Enter") {
                sendMessage();
            }
        });
    </script>
</body>
</html>
```

### 5. `public/404.html`

This file defines the fallback page served when a client requests an undefined route.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>404 Page Not Found</title>
</head>

<body>
    <h1>404 Page Not Found</h1>
    <p>The requested resource could not be found on this server.</p>
</body>
</html>
```

---

## V. Execution and Verification Deliverables

### Deliverable 1: Bonezegei Interpreter Version Verification

Execution of `bonezegei -v` in PowerShell confirms the installed Bonezegei Script Interpreter version.

**Command Executed:**

```powershell
bonezegei -v
```

**Expected Verification:** Bonezegei Script Interpreter v1.3.3.

**Screenshot:**

![Bonezegei Interpreter Version Verification](documentation\1.png)

---

### Deliverable 2: Dependency Installation

Execution of `bzg install` reads the dependencies declared in `package.json` and installs the required `http` package.

**Command Executed:**

```powershell
bzg install
```

**Verification:** The dependency installation process completes successfully.

**Screenshot:**

![Dependency Installation](documentation\2.png)

---

### Deliverable 3: WebSocket Server Startup

Execution of `bzg run` initiates the application by executing `bonezegei app.bzg` and starts the server on port `8000`.

**Command Executed:**

```powershell
bzg run
```

**Verification:** The server starts and listens for HTTP requests and WebSocket connections on port `8000`.

**Screenshot:**

![WebSocket Server Startup](documentation\3.png)

---

### Deliverable 4: Multi-Client Chat and Custom 404 Routing Verification

To test system concurrency, message delivery, and route handling, three browser tabs were used.

#### 4.1. Custom 404 Routing Test (`http://localhost:8000/nonexistent`)

- Accessing an undefined path rendered the custom `404.html` page.
- The page displayed **"404 Page Not Found"**, confirming that the application returned the configured fallback page for an undefined route.

**Test Result:** The custom 404 page was displayed when the undefined path was accessed.

#### 4.2. Multi-Client Real-Time Broadcast Test (`http://localhost:8000`)

- **User 0** and **User 1** established WebSocket sessions simultaneously.
- Upon connection, both clients completed the WebSocket handshake and received the server welcome payload: `[Server] Welcome to WebSocket server!`.
- Messages sent from **User 0** (`"hey bruh what is upp"` and `"fine here, been doing great!"`) and **User 1** (`"heyyy"` and `"all good, how about you?"`) were broadcast to both connected clients in real time through `server.sendAll()`.
- Both clients displayed the transmitted messages in their respective chat logs, demonstrating successful multi-client communication.

**Test Result:** Successful concurrent WebSocket connections, welcome message delivery, and real-time message broadcasting.

**Server Connection and Message Logs:**

![Server Connection and Message Logs](documentation\4.png)

---

## VI. Technical Discussion and Protocol Analysis

### 1. HTTP-to-WebSocket Protocol Upgrade Handshake

When a client navigates to `http://localhost:8000`, the initial HTTP GET request serves `public/index.html`. The embedded JavaScript then creates a WebSocket connection using:

```javascript
const socket = new WebSocket("ws://localhost:8000");
```

This initiates the WebSocket opening handshake over HTTP, using upgrade headers such as `Upgrade: websocket` and `Connection: Upgrade`.

When the server accepts the connection, `server.onConnect(clientId)` is triggered and a client ID is assigned to the connection. The connection then supports full-duplex communication, allowing the client and server to exchange messages over the established WebSocket connection.

### 2. Unicast and Broadcast Communication

The application demonstrates two message delivery methods.

**Unicast:**

```javascript
server.send(clientId, "Welcome to WebSocket server!");
```

The `server.send()` function targets a specific client ID. In this implementation, it sends the welcome message to the newly connected client.

**Broadcast:**

```javascript
server.sendAll("User " + clientId + " says: " + message);
```

The `server.sendAll()` function broadcasts the received message to all connected WebSocket clients. It is called from `server.onData`, allowing messages from one client to be distributed to the other connected clients as well.

### 3. WebSocket Lifecycle Management

The application uses three WebSocket event handlers to manage the client lifecycle.

| Event Handler | Purpose |
|---|---|
| `server.onConnect` | Handles a new WebSocket connection and sends a welcome message. |
| `server.onData` | Processes incoming messages and broadcasts them to connected clients. |
| `server.onDisconnect` | Logs when a client disconnects from the server. |

These event handlers allow the server to respond to connection events and incoming data without requiring clients to repeatedly request updates through separate HTTP requests.

### 4. HTTP Routing and Static File Serving

The application uses `server.get` to handle incoming HTTP requests.

- Requests to `/` serve `public/index.html`.
- Requests to other paths serve `public/404.html`.

This implementation demonstrates basic request routing and static HTML file serving through the BSL `http` package.

---

## VII. Conclusion

This laboratory exercise demonstrated the configuration and implementation of an event-driven WebSocket server using the Bonezegei Scripting Language (v1.3.3).

The application handles WebSocket connections, sends welcome messages to newly connected clients, processes incoming messages, and broadcasts messages across multiple active sessions. It also serves the browser-based chat interface and provides a custom 404 page for undefined routes.

Testing with multiple browser clients demonstrated concurrent connections, welcome message delivery, and real-time message broadcasting through `server.sendAll()`. The activity provided practical experience with HTTP routing, WebSocket connection lifecycle management, and full-duplex client-server communication.

---

**End of Laboratory Report**