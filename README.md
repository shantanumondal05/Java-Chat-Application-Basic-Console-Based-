# Java-Chat-Application-Basic-Console-Based


## Overview

This is a **multi-client, server-based console chat application** built in pure Java using socket programming and multi-threading. The application allows multiple users to connect to a central server and communicate with each other in real-time. All communication occurs through a command-line interface.

### Key Information
- **Server Port:** 5000
- **Communication Protocol:** TCP/IP
- **Server Name:** Team ApexDevs Chat Server
- **Architecture Type:** Client-Server (Many-to-Many)

---

## Architecture

The application follows a **client-server architecture** with the following structure:

```
┌─────────────────────────────────────────────┐
│          ChatServer (Port 5000)             │
│  ┌─────────────────────────────────────┐   │
│  │  ServerSocket - Accepts Connections │   │
│  └─────────────────────────────────────┘   │
│            │                                │
│  ┌─────────────────────────────────────┐   │
│  │  CopyOnWriteArrayList<ClientHandler>│   │
│  │   (Stores all active client threads)│   │
│  └─────────────────────────────────────┘   │
│            │                                │
│  ┌─────────────────────────────────────┐   │
│  │  broadcast() - Message Distribution │   │
│  └─────────────────────────────────────┘   │
└─────────┬───────────────────┬───────────────┘
          │                   │
    ┌─────▼────┐        ┌──────▼───┐
    │ Client 1  │        │ Client 2  │
    │(ChatClient)│        │(ChatClient)│
    └───────────┘        └───────────┘
          │                   │
    ┌─────▼────┐        ┌──────▼───┐
    │Handler 1  │        │Handler 2  │
    │(Separate  │        │(Separate  │
    │ Thread)   │        │ Thread)   │
    └───────────┘        └───────────┘
```

---

## System Components

### 1. **ChatServer** (Server-side Core)
- Listens for incoming client connections on port 5000
- Manages all active client connections
- Broadcasts messages from one client to all others
- Removes disconnected clients from the active list

### 2. **ChatClient** (Client-side)
- Connects to the ChatServer on localhost:5000
- Sends user input to the server
- Receives and displays messages from other clients
- Uses multi-threading to handle simultaneous send/receive operations

### 3. **ClientHandler** (Server-side Worker)
- Handles individual client connections
- Manages client identification (name/username)
- Reads incoming messages from the client
- Forwards messages to the server for broadcasting

---

## How It Works

### Step-by-Step Communication Flow

```
1. Server Startup
   └─ ChatServer starts → Listens on port 5000 → Ready to accept connections

2. Client Connection
   └─ ChatClient connects to localhost:5000
      └─ New Socket created
      └─ New ClientHandler thread spawned on server
      └─ ClientHandler added to clients list

3. Client Identification
   └─ Server asks: "Enter your name: "
   └─ Client enters name (e.g., "Alice")
   └─ ClientHandler reads the name

4. Join Notification
   └─ Server broadcasts: "Alice joined the chat!"
   └─ Message sent to all OTHER clients

5. Message Exchange
   └─ Client sends message: "Hello everyone!"
   └─ ServerSocket prints to console: "Alice: Hello everyone!"
   └─ Message broadcast to all other clients

6. Client Disconnection
   └─ Client closes connection (Ctrl+C or EOF)
   └─ ClientHandler catches IOException
   └─ Client removed from active list
   └─ Disconnect notification broadcast to remaining clients

7. Server Shutdown
   └─ Press Ctrl+C on server
   └─ All client connections persist until they disconnect individually
```

---

## Setup Instructions

### Prerequisites
- **Java Development Kit (JDK)** version 8 or higher
- **Command Prompt** or **PowerShell** (Windows) / **Terminal** (Mac/Linux)
- Files: `ChatServer.java`, `ChatClient.java`, `ClientHandler.java`

### Installation Steps

#### 1. **Compile the Java Files**

```bash
# Navigate to the project directory
cd e:\javachat

# Compile all Java files
javac ChatServer.java ChatClient.java ClientHandler.java
```

**Expected Output:**
- Three `.class` files generated: `ChatServer.class`, `ChatClient.class`, `ClientHandler.class`, `ListenThread.class`

#### 2. **Start the Server**

Open a new terminal/command prompt and run:

```bash
java ChatServer
```

**Expected Output:**
```
Team ApexDevs Chat Server started on port 5000
```

#### 3. **Connect Multiple Clients**

In separate terminal windows, run:

```bash
java ChatClient
```

**Expected Output (Client side):**
```
Enter your name: 
```

---

## Usage Guide

### Running a Chat Session

#### **Terminal 1 - Server**
```bash
java ChatServer
```
Output:
```
Team ApexDevs Chat Server started on port 5000
New connection: 127.0.0.1/127.0.0.1:12345
New connection: 127.0.0.1/127.0.0.1:12346
Alice: Hello everyone!
Bob: Hi Alice!
Alice disconnected.
```

#### **Terminal 2 - Client 1 (Alice)**
```bash
java ChatClient
```
```
Enter your name: 
Alice
Bob joined the chat!
Bob: Hi Alice!
```

#### **Terminal 3 - Client 2 (Bob)**
```bash
java ChatClient
```
```
Enter your name: 
Bob
Alice joined the chat!
Alice: Hello everyone!
```

### Sending Messages

- Simply type your message and press **Enter**
- Messages appear on all other clients' screens
- Type `Ctrl+C` to disconnect

---

## Key Features

### 1. **Multi-Client Support**
- Unlimited number of clients can connect simultaneously
- Each client runs on a separate thread to prevent blocking

### 2. **Real-Time Broadcasting**
- Messages are instantly distributed to all connected clients
- Sender does not see their own message (by design)

### 3. **Thread-Safe Operations**
- Uses `CopyOnWriteArrayList` for thread-safe client management
- No synchronization issues when multiple clients connect/disconnect

### 4. **User Identification**
- Each client must enter a name upon connection
- Client name appears in all broadcast messages

### 5. **Connection Notifications**
- Server notifies all clients when someone joins
- Server notifies all clients when someone leaves

### 6. **Graceful Disconnection**
- Clients can disconnect cleanly
- Server properly removes disconnected clients
- Resources are properly closed (finally blocks)

---

## Technical Details

### Networking Protocol
- **Transport Layer:** TCP (Transmission Control Protocol)
- **Port:** 5000
- **Connection Type:** Persistent socket connection
- **Communication:** Bidirectional (send and receive simultaneously)

### Threading Model

#### **Server-Side Threading**
```
Main Thread (ChatServer)
├─ Accepts incoming connections in infinite loop
│
└─ For Each Connection:
   └─ Creates new Thread for ClientHandler
      ├─ Reads from client input stream
      ├─ Broadcasts to other clients
      └─ Cleans up on disconnection
```

#### **Client-Side Threading**
```
Main Thread (ChatClient)
├─ Sends messages from console

Listener Thread (ListenThread)
├─ Continuously reads from server
├─ Displays received messages
└─ Detects connection closure
```

### Stream Usage

| Class | Input Stream | Output Stream | Purpose |
|-------|---|---|---|
| **ChatServer** | N/A | N/A | Manages connections |
| **ChatClient** | `socket.getInputStream()` | `socket.getOutputStream()` | Send/Receive |
| **ClientHandler** | `socket.getInputStream()` | `socket.getOutputStream()` | Handle individual client |
| **ListenThread** | `socket.getInputStream()` | N/A | Listen for messages |

### Data Flow

```
Client Input (Console)
        ↓
PrintWriter (out)
        ↓
Socket Output Stream
        ↓
Network (TCP/IP)
        ↓
Server Receives via ClientHandler
        ↓
ChatServer.broadcast()
        ↓
Other Clients' Input Streams
        ↓
ListenThread reads
        ↓
System.out.println()
        ↓
Client Display (Console)
```
## Author

Shantanu Mondal    
GitHub: https://github.com/shantanumondal05    
LinkedIn: https://linkedin.com/in/shantanu-mondal-42785b32a  

