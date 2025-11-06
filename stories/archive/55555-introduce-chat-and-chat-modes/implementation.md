# Implementation Plan: Chat with AI-Powered Tone Modes

**Story ID:** 55555  
**Feature:** introduce-chat-and-chat-modes  
**Status:** In Development  
**Approach:** Test-Driven Development (TDD)

---

## Overview

Implement real-time chat system with 10 AI-powered personality modes using Socket.io + single-instance Azure Web App + SQLite + Azure OpenAI.

**Key Principles:**
- ✅ Write tests FIRST, implementation SECOND
- ✅ Follow Red-Green-Refactor cycle
- ✅ Keep tests simple and focused
- ✅ Run tests after each step to verify

---

## AI Model Configuration

**Provider:** Azure OpenAI  
**Endpoint:** `https://midwest7-openai-northcentral.openai.azure.com/`  
**Deployment:** `gpt-4o-mini`  
**API Version:** `2025-01-01-preview`  
**Authentication:** API key via environment variable

---

## Prerequisites

### Environment Setup (COMPLETED ✅)

**Local Development:**
- [x] `.env` file created in `midwest-7-service/`
- [x] `AZURE_OPENAI_API_KEY` configured
- [x] All environment variables set

**Azure Deployment:**
- [ ] Add environment variables to Azure App Service Configuration
- [ ] Enable Web Sockets in Azure Web App
- [ ] Enable Always On
- [ ] Enable ARR Affinity

### Dependencies to Install

**Service Layer (midwest-7-service):**
```bash
cd midwest-7-service
npm install socket.io dotenv axios node-cache
npm install --save-dev jest supertest socket.io-client
```

**UI Layer (midwest-7-ui):**
```bash
cd midwest-7-ui
npm install socket.io-client
```

---

## TDD Workflow Reminder

For each feature:

1. **RED 🔴**: Write failing test first
   ```bash
   npm test -- --watch aiClient.test.js
   # Confirm test fails
   ```

2. **GREEN 🟢**: Write minimal code to pass
   ```bash
   # Implement feature
   npm test
   # Confirm test passes
   ```

3. **REFACTOR 🔵**: Improve code while keeping tests green
   ```bash
   # Clean up code
   npm test
   # Confirm tests still pass
   ```

---

## Phase 1: Service Layer - Configuration & Database

### Step 1.1: Create AI Configuration Module

**Location:** `midwest-7-service/config/aiConfig.js`

**Test First (RED 🔴):**
```bash
# No test needed for configuration - it's a simple module
# We'll verify it works in the AI client tests
```

**Implement (GREEN 🟢):**

Create `config/aiConfig.js`:
```javascript
require('dotenv').config();

var aiConfig = {
  endpoint: process.env.AZURE_OPENAI_ENDPOINT,
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  deployment: process.env.AZURE_OPENAI_DEPLOYMENT || 'gpt-4o-mini',
  apiVersion: process.env.AZURE_OPENAI_API_VERSION || '2025-01-01-preview',
  maxTokens: 150,
  temperature: 0.7,
  topP: 1.0
};

// Validate required configuration
if (!aiConfig.endpoint || !aiConfig.apiKey) {
  throw new Error('AZURE_OPENAI_ENDPOINT and AZURE_OPENAI_API_KEY must be set in environment variables');
}

module.exports = aiConfig;
```

**Verify:**
```bash
node -e "require('./config/aiConfig.js'); console.log('✅ Config loaded successfully');"
```

---

### Step 1.2: Update Database Schema

**Location:** `midwest-7-service/db/init.js`

**Test First (RED 🔴):**

Create `__tests__/db/chatSchema.test.js`:
```javascript
var Database = require('better-sqlite3');
var path = require('path');
var fs = require('fs');

describe('Chat Messages Schema', function() {
  var db;
  var testDbPath = path.join(__dirname, 'test-chat.db');

  beforeEach(function() {
    // Create fresh test database
    if (fs.existsSync(testDbPath)) {
      fs.unlinkSync(testDbPath);
    }
    db = new Database(testDbPath);
  });

  afterEach(function() {
    db.close();
    if (fs.existsSync(testDbPath)) {
      fs.unlinkSync(testDbPath);
    }
  });

  it('should create chat_messages table', function() {
    // Arrange & Act
    db.exec(`
      CREATE TABLE IF NOT EXISTS chat_messages (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        sender_id INTEGER NOT NULL,
        receiver_id INTEGER NOT NULL,
        original_message TEXT NOT NULL,
        transformed_message TEXT NOT NULL,
        chat_mode TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      );
    `);

    // Assert
    var tableInfo = db.prepare("PRAGMA table_info(chat_messages)").all();
    var columnNames = tableInfo.map(function(col) { return col.name; });
    
    expect(columnNames).toContain('id');
    expect(columnNames).toContain('sender_id');
    expect(columnNames).toContain('receiver_id');
    expect(columnNames).toContain('original_message');
    expect(columnNames).toContain('transformed_message');
    expect(columnNames).toContain('chat_mode');
    expect(columnNames).toContain('created_at');
  });

  it('should have indexes for efficient querying', function() {
    // Arrange & Act
    db.exec(`
      CREATE TABLE chat_messages (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        sender_id INTEGER NOT NULL,
        receiver_id INTEGER NOT NULL,
        original_message TEXT NOT NULL,
        transformed_message TEXT NOT NULL,
        chat_mode TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      );
      CREATE INDEX idx_chat_messages_users ON chat_messages(sender_id, receiver_id, created_at DESC);
    `);

    // Assert
    var indexes = db.prepare("PRAGMA index_list(chat_messages)").all();
    var indexNames = indexes.map(function(idx) { return idx.name; });
    
    expect(indexNames).toContain('idx_chat_messages_users');
  });
});
```

**Run Test:** `npm test -- chatSchema.test.js` → Should PASS ✅ (schema validation test)

**Implement (GREEN 🟢):**

Update `db/init.js` - add at the end of the `db.exec()` call:
```javascript
CREATE TABLE IF NOT EXISTS chat_messages (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  sender_id INTEGER NOT NULL,
  receiver_id INTEGER NOT NULL,
  original_message TEXT NOT NULL,
  transformed_message TEXT NOT NULL,
  chat_mode TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (sender_id) REFERENCES users(id),
  FOREIGN KEY (receiver_id) REFERENCES users(id)
);

CREATE INDEX IF NOT EXISTS idx_chat_messages_users 
ON chat_messages(sender_id, receiver_id, created_at DESC);
```

**Verify:**
```bash
npm start
# Check console for "Initializing database" message
# Stop server (Ctrl+C)
```

---

## Phase 2: Service Layer - AI Integration

### Step 2.1: AI Client Service

**Location:** `midwest-7-service/services/aiClient.js`

**Test First (RED 🔴):**

Create `__tests__/services/aiClient.test.js`:
```javascript
var aiClient = require('../../services/aiClient');
var axios = require('axios');

jest.mock('axios');

describe('AI Client Service', function() {
  afterEach(function() {
    jest.clearAllMocks();
  });

  it('should transform message with pirate mode', async function() {
    // Arrange
    var mockResponse = {
      data: {
        choices: [{
          message: {
            content: 'Ahoy matey! How be ye? ⚓🏴‍☠️'
          }
        }]
      }
    };
    axios.post.mockResolvedValue(mockResponse);

    // Act
    var result = await aiClient.transformMessage('How are you?', 'pirate');

    // Assert
    expect(result).toBe('Ahoy matey! How be ye? ⚓🏴‍☠️');
    expect(axios.post).toHaveBeenCalledWith(
      expect.stringContaining('/chat/completions'),
      expect.objectContaining({
        messages: expect.arrayContaining([
          expect.objectContaining({ role: 'system' }),
          expect.objectContaining({ role: 'user', content: 'How are you?' })
        ]),
        model: 'gpt-4o-mini'
      }),
      expect.objectContaining({
        headers: expect.objectContaining({
          'api-key': expect.any(String)
        })
      })
    );
  });

  it('should use fallback when AI service fails', async function() {
    // Arrange
    axios.post.mockRejectedValue(new Error('Network error'));

    // Act
    var result = await aiClient.transformMessage('Hello', 'robot');

    // Assert
    expect(result).toBe('Hello 🤖⚙️');
  });

  it('should handle timeout errors', async function() {
    // Arrange
    axios.post.mockRejectedValue({ code: 'ECONNABORTED' });

    // Act
    var result = await aiClient.transformMessage('Test', 'shakespeare');

    // Assert
    expect(result).toContain('Test');
    expect(result).toContain('📜✨');
  });

  it('should handle all 10 chat modes', async function() {
    // Arrange
    var modes = ['pirate', 'shakespeare', 'robot', 'horror', 'party', 
                 'fantasy', 'alien', 'detective', 'corporate', 'genz'];
    axios.post.mockResolvedValue({
      data: { choices: [{ message: { content: 'Transformed ✨' } }] }
    });

    // Act & Assert
    for (var mode of modes) {
      var result = await aiClient.transformMessage('test', mode);
      expect(result).toBe('Transformed ✨');
    }
    
    expect(axios.post).toHaveBeenCalledTimes(10);
  });
});
```

**Run Test:** `npm test -- aiClient.test.js` → Should FAIL (RED 🔴)

**Implement (GREEN 🟢):**

Create `services/aiClient.js`:
```javascript
var axios = require('axios');
var aiConfig = require('../config/aiConfig');

var chatModePrompts = {
  pirate: 'Transform this message into pirate-speak with nautical personality. Add emojis: ⚓🏴‍☠️☠️🦜. Keep it under 2 sentences.',
  shakespeare: 'Transform into Shakespearean English with classical flair. Add emojis: 📜✨🎭. Keep it under 2 sentences.',
  robot: 'Transform into robotic/mechanical speech. Add emojis: 🤖🔧⚙️. Keep it under 2 sentences.',
  horror: 'Transform into eerie, mysterious tone. Add emojis: 👻🕷️🌑. Keep it under 2 sentences.',
  party: 'Transform into super excited, energetic party mode. Add emojis: 🥳💃🎉. Keep it under 2 sentences.',
  fantasy: 'Transform into mystical fantasy language. Add emojis: ✨🐉🧙‍♂️. Keep it under 2 sentences.',
  alien: 'Transform into alien/extraterrestrial speech. Add emojis: 🛸👾👽. Keep it under 2 sentences.',
  detective: 'Transform into detective/investigative tone. Add emojis: 🔍📝🕵️. Keep it under 2 sentences.',
  corporate: 'Transform into corporate jargon with buzzwords. Add emojis: 📊💼💻. Keep it under 2 sentences.',
  genz: 'Transform into Gen Z slang and expressions. Add emojis: 💅✨🔥. Keep it under 2 sentences.'
};

async function transformMessage(message, chatMode) {
  var systemPrompt = chatModePrompts[chatMode] || chatModePrompts.pirate;
  
  try {
    var url = aiConfig.endpoint + '/openai/deployments/' + aiConfig.deployment + 
              '/chat/completions?api-version=' + aiConfig.apiVersion;
    
    var response = await axios.post(
      url,
      {
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: message }
        ],
        max_tokens: aiConfig.maxTokens,
        temperature: aiConfig.temperature,
        top_p: aiConfig.topP,
        model: aiConfig.deployment
      },
      {
        headers: {
          'api-key': aiConfig.apiKey,
          'Content-Type': 'application/json'
        },
        timeout: 5000
      }
    );

    return response.data.choices[0].message.content.trim();
  } catch (error) {
    console.error('AI transformation error:', error.message);
    return getFallbackMessage(message, chatMode);
  }
}

function getFallbackMessage(message, chatMode) {
  var modeEmojis = {
    pirate: ' ⚓🏴‍☠️',
    shakespeare: ' 📜✨',
    robot: ' 🤖⚙️',
    horror: ' 👻🕷️',
    party: ' 🥳🎉',
    fantasy: ' ✨🐉',
    alien: ' 🛸👾',
    detective: ' 🔍📝',
    corporate: ' 📊💼',
    genz: ' 💅✨'
  };
  
  return message + (modeEmojis[chatMode] || ' ✨');
}

module.exports = {
  transformMessage: transformMessage
};
```

**Run Test:** `npm test -- aiClient.test.js` → Should PASS (GREEN 🟢)

**Refactor (BLUE 🔵):**
- Extract prompts to separate config file if needed
- Add JSDoc comments
- Optimize error handling

---

### Step 2.2: Message Transformer with Caching

**Location:** `midwest-7-service/services/messageTransformer.js`

**Test First (RED 🔴):**

Create `__tests__/services/messageTransformer.test.js`:
```javascript
var messageTransformer = require('../../services/messageTransformer');
var aiClient = require('../../services/aiClient');

jest.mock('../../services/aiClient');

describe('Message Transformer Service', function() {
  beforeEach(function() {
    jest.clearAllMocks();
    aiClient.transformMessage.mockResolvedValue('Ahoy matey! ⚓');
  });

  it('should transform message using AI client', async function() {
    // Act
    var result = await messageTransformer.transform('Hello', 'pirate');

    // Assert
    expect(result).toBe('Ahoy matey! ⚓');
    expect(aiClient.transformMessage).toHaveBeenCalledWith('Hello', 'pirate');
  });

  it('should cache transformed messages', async function() {
    // Act
    var result1 = await messageTransformer.transform('Hello', 'pirate');
    var result2 = await messageTransformer.transform('Hello', 'pirate');

    // Assert
    expect(result1).toBe('Ahoy matey! ⚓');
    expect(result2).toBe('Ahoy matey! ⚓');
    expect(aiClient.transformMessage).toHaveBeenCalledTimes(1); // Called once, cached second time
  });

  it('should handle different modes separately in cache', async function() {
    // Arrange
    aiClient.transformMessage
      .mockResolvedValueOnce('Ahoy! ⚓')
      .mockResolvedValueOnce('BEEP.BOOP 🤖');

    // Act
    var pirate = await messageTransformer.transform('Hi', 'pirate');
    var robot = await messageTransformer.transform('Hi', 'robot');

    // Assert
    expect(pirate).toBe('Ahoy! ⚓');
    expect(robot).toBe('BEEP.BOOP 🤖');
    expect(aiClient.transformMessage).toHaveBeenCalledTimes(2);
  });

  it('should validate message input', async function() {
    // Act & Assert
    await expect(messageTransformer.transform('', 'pirate')).rejects.toThrow('Message cannot be empty');
    await expect(messageTransformer.transform(null, 'pirate')).rejects.toThrow('Message cannot be empty');
  });

  it('should validate chat mode', async function() {
    // Act & Assert
    await expect(messageTransformer.transform('Hello', 'invalid')).rejects.toThrow('Invalid chat mode');
  });
});
```

**Run Test:** `npm test -- messageTransformer.test.js` → Should FAIL (RED 🔴)

**Implement (GREEN 🟢):**

Create `services/messageTransformer.js`:
```javascript
var NodeCache = require('node-cache');
var aiClient = require('./aiClient');

var cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

var validModes = ['pirate', 'shakespeare', 'robot', 'horror', 'party', 
                  'fantasy', 'alien', 'detective', 'corporate', 'genz'];

async function transform(message, chatMode) {
  // Validate input
  if (!message || message.trim() === '') {
    throw new Error('Message cannot be empty');
  }
  
  if (!validModes.includes(chatMode)) {
    throw new Error('Invalid chat mode: ' + chatMode);
  }
  
  // Check cache
  var cacheKey = chatMode + ':' + message.trim();
  var cached = cache.get(cacheKey);
  if (cached) {
    return cached;
  }
  
  // Transform via AI
  var transformed = await aiClient.transformMessage(message, chatMode);
  
  // Cache result
  cache.set(cacheKey, transformed);
  
  return transformed;
}

function clearCache() {
  cache.flushAll();
}

module.exports = { 
  transform: transform,
  clearCache: clearCache
};
```

**Run Test:** `npm test -- messageTransformer.test.js` → Should PASS (GREEN 🟢)

---

## Phase 3: Service Layer - Socket.io Integration

### Step 3.1: Socket.io Server Configuration

**Location:** `midwest-7-service/config/socketConfig.js`

**Implement:**

Create `config/socketConfig.js`:
```javascript
var isProduction = process.env.NODE_ENV === 'production';

var socketConfig = {
  cors: {
    origin: process.env.ALLOWED_ORIGINS 
      ? process.env.ALLOWED_ORIGINS.split(',')
      : ['http://localhost:3000'],
    methods: ['GET', 'POST'],
    credentials: true
  },
  pingTimeout: 60000,
  pingInterval: 25000,
  transports: ['websocket', 'polling']
};

module.exports = socketConfig;
```

---

### Step 3.2: Update Server with Socket.io

**Location:** `midwest-7-service/bin/www` or `server.js`

**Test First (RED 🔴):**

Create `__tests__/server/socketServer.test.js`:
```javascript
var io = require('socket.io-client');
var http = require('http');

describe('Socket.io Server', function() {
  var server;
  var ioServer;
  var clientSocket;
  var serverPort = 3002;

  beforeAll(function(done) {
    var express = require('express');
    var app = express();
    server = http.createServer(app);
    
    ioServer = require('socket.io')(server, {
      cors: { origin: '*' }
    });
    
    server.listen(serverPort, done);
  });

  afterAll(function() {
    if (ioServer) ioServer.close();
    if (server) server.close();
  });

  beforeEach(function(done) {
    clientSocket = io('http://localhost:' + serverPort);
    clientSocket.on('connect', done);
  });

  afterEach(function() {
    if (clientSocket) clientSocket.close();
  });

  it('should connect to socket.io server', function() {
    expect(clientSocket.connected).toBe(true);
  });

  it('should disconnect properly', function(done) {
    clientSocket.on('disconnect', function() {
      expect(clientSocket.connected).toBe(false);
      done();
    });
    clientSocket.close();
  });
});
```

**Run Test:** `npm test -- socketServer.test.js` → Should PASS (basic connection test)

**Implement (GREEN 🟢):**

Update `bin/www` (or create new `server.js`):
```javascript
var app = require('../app');
var http = require('http');
var socketIo = require('socket.io');
var socketConfig = require('../config/socketConfig');

var port = process.env.PORT || 3001;
app.set('port', port);

var server = http.createServer(app);
var io = socketIo(server, socketConfig);

// Socket.io chat handling
require('../routes/chatSocket')(io);

server.listen(port, function() {
  console.log('Server with Socket.io running on port ' + port);
});

server.on('error', function(error) {
  if (error.syscall !== 'listen') {
    throw error;
  }
  // Handle specific listen errors
  switch (error.code) {
    case 'EACCES':
      console.error('Port ' + port + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error('Port ' + port + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
});
```

---

### Step 3.3: Chat Socket Event Handlers

**Location:** `midwest-7-service/routes/chatSocket.js`

**Test First (RED 🔴):**

Create `__tests__/routes/chatSocket.test.js`:
```javascript
var io = require('socket.io-client');
var http = require('http');
var messageTransformer = require('../../services/messageTransformer');

jest.mock('../../services/messageTransformer');

describe('Chat Socket Events', function() {
  var server;
  var ioServer;
  var clientSocket1;
  var clientSocket2;
  var serverPort = 3003;

  beforeAll(function(done) {
    var express = require('express');
    var app = express();
    server = http.createServer(app);
    ioServer = require('socket.io')(server, { cors: { origin: '*' } });
    
    // Mock chat socket handlers
    require('../../routes/chatSocket')(ioServer);
    
    server.listen(serverPort, done);
  });

  afterAll(function() {
    if (ioServer) ioServer.close();
    if (server) server.close();
  });

  beforeEach(function(done) {
    messageTransformer.transform.mockResolvedValue('Ahoy matey! ⚓');
    clientSocket1 = io('http://localhost:' + serverPort);
    clientSocket1.on('connect', done);
  });

  afterEach(function() {
    if (clientSocket1) clientSocket1.close();
    if (clientSocket2) clientSocket2.close();
    jest.clearAllMocks();
  });

  it('should emit message-sent confirmation', function(done) {
    // Arrange
    clientSocket1.on('message-sent', function(data) {
      // Assert
      expect(data.status).toBe('sent');
      done();
    });

    // Act
    clientSocket1.emit('send-message', {
      senderId: 1,
      receiverId: 2,
      message: 'Hello',
      mode: 'pirate'
    });
  });

  it('should transform message before sending', function(done) {
    // Arrange
    clientSocket1.emit('send-message', {
      senderId: 1,
      receiverId: 2,
      message: 'Hello',
      mode: 'pirate'
    });

    // Wait for processing
    setTimeout(function() {
      // Assert
      expect(messageTransformer.transform).toHaveBeenCalledWith('Hello', 'pirate');
      done();
    }, 100);
  });
});
```

**Run Test:** `npm test -- chatSocket.test.js` → Should FAIL (RED 🔴)

**Implement (GREEN 🟢):**

Create `routes/chatSocket.js`:
```javascript
var db = require('../db/init');
var messageTransformer = require('../services/messageTransformer');

var connectedUsers = new Map(); // userId -> socket.id

module.exports = function(io) {
  io.on('connection', function(socket) {
    console.log('User connected:', socket.id);

    // User joins
    socket.on('user-join', function(data) {
      var userId = data.userId;
      connectedUsers.set(userId, socket.id);
      socket.userId = userId;
      socket.join('user-' + userId);
      
      // Update online status
      db.prepare('UPDATE users SET is_online = 1, last_active = CURRENT_TIMESTAMP WHERE id = ?')
        .run(userId);
      
      // Broadcast user online
      socket.broadcast.emit('user-online', { userId: userId });
      
      console.log('User joined:', userId);
    });

    // Send message
    socket.on('send-message', async function(data) {
      try {
        var senderId = data.senderId;
        var receiverId = data.receiverId;
        var originalMessage = data.message;
        var chatMode = data.mode;

        // Transform message
        var transformedMessage = await messageTransformer.transform(originalMessage, chatMode);

        // Save to database
        var stmt = db.prepare(`
          INSERT INTO chat_messages (sender_id, receiver_id, original_message, transformed_message, chat_mode)
          VALUES (?, ?, ?, ?, ?)
        `);
        var result = stmt.run(senderId, receiverId, originalMessage, transformedMessage, chatMode);

        var messageData = {
          id: result.lastInsertRowid,
          senderId: senderId,
          receiverId: receiverId,
          original: originalMessage,
          transformed: transformedMessage,
          mode: chatMode,
          createdAt: new Date().toISOString()
        };

        // Send to receiver
        io.to('user-' + receiverId).emit('new-message', messageData);

        // Confirm to sender
        socket.emit('message-sent', { 
          id: messageData.id,
          status: 'sent',
          transformed: transformedMessage
        });

        console.log('Message sent from', senderId, 'to', receiverId, 'in', chatMode, 'mode');
      } catch (error) {
        console.error('Error sending message:', error);
        socket.emit('message-error', { error: error.message });
      }
    });

    // User disconnects
    socket.on('disconnect', function() {
      var userId = socket.userId;
      if (userId) {
        connectedUsers.delete(userId);
        
        // Update online status
        db.prepare('UPDATE users SET is_online = 0, last_active = CURRENT_TIMESTAMP WHERE id = ?')
          .run(userId);
        
        // Check and cleanup chats
        cleanupChats(userId);
        
        // Broadcast user offline
        socket.broadcast.emit('user-offline', { userId: userId });
        
        console.log('User disconnected:', userId);
      }
    });
  });
};

function cleanupChats(userId) {
  // Get all users this user chatted with
  var chatPartners = db.prepare(`
    SELECT DISTINCT 
      CASE 
        WHEN sender_id = ? THEN receiver_id 
        ELSE sender_id 
      END as partner_id
    FROM chat_messages
    WHERE sender_id = ? OR receiver_id = ?
  `).all(userId, userId, userId);

  // For each partner, check if both are offline
  chatPartners.forEach(function(partner) {
    var partnerId = partner.partner_id;
    var partnerOnline = db.prepare('SELECT is_online FROM users WHERE id = ?')
      .get(partnerId);
    
    if (partnerOnline && partnerOnline.is_online === 0) {
      // Both offline - delete chat messages
      db.prepare(`
        DELETE FROM chat_messages 
        WHERE (sender_id = ? AND receiver_id = ?) 
           OR (sender_id = ? AND receiver_id = ?)
      `).run(userId, partnerId, partnerId, userId);
      
      console.log('Cleaned up chat between', userId, 'and', partnerId);
    }
  });
}
```

**Run Test:** `npm test -- chatSocket.test.js` → Should PASS (GREEN 🟢)

---

## Phase 4: UI Layer - Socket.io Client

### Step 4.1: Socket Service

**Location:** `midwest-7-ui/src/services/socketService.js`

**Implement:**

```javascript
import { io } from 'socket.io-client';

const SOCKET_URL = process.env.NODE_ENV === 'production'
  ? process.env.REACT_APP_SOCKET_URL || 'https://your-service.azurewebsites.net'
  : 'http://localhost:3001';

const socket = io(SOCKET_URL, {
  autoConnect: false,
  reconnection: true,
  reconnectionDelay: 1000,
  reconnectionAttempts: 5
});

export default socket;
```

---

### Step 4.2: Chat Modes Configuration

**Location:** `midwest-7-ui/src/utils/chatModes.js`

**Implement:**

```javascript
export const CHAT_MODES = [
  { id: 'pirate', name: 'Pirate Mode', emoji: '😎', description: 'Ahoy matey! ⚓🏴‍☠️' },
  { id: 'shakespeare', name: 'Shakespeare Mode', emoji: '🎭', description: 'Verily, thy words... 📜✨' },
  { id: 'robot', name: 'Robot Mode', emoji: '🤖', description: 'BEEP.BOOP.PROCESSING... 🔧⚙️' },
  { id: 'horror', name: 'Horror Mode', emoji: '😱', description: 'Your words... whispered in shadows... 👻🕷️' },
  { id: 'party', name: 'Party Mode', emoji: '🎉', description: 'YAAAS! Everything! Is! EXCITING! 🥳💃' },
  { id: 'fantasy', name: 'Fantasy Mode', emoji: '🧙‍♂️', description: 'By ancient magic... ✨🐉' },
  { id: 'alien', name: 'Alien Mode', emoji: '👽', description: 'Greetings earthling... 🛸👾' },
  { id: 'detective', name: 'Detective Mode', emoji: '🕵️', description: 'The evidence suggests... 🔍📝' },
  { id: 'corporate', name: 'Corporate Mode', emoji: '💼', description: 'Circling back to synergize... 📊💻' },
  { id: 'genz', name: 'Gen Z Mode', emoji: '🦖', description: 'No cap, this is bussin fr fr! 💅✨' }
];

export const getChatModeById = (id) => {
  return CHAT_MODES.find(mode => mode.id === id) || CHAT_MODES[0];
};
```

---

### Step 4.3: Chat Components (Basic Structure)

**Location:** `midwest-7-ui/src/components/Chat/`

Component files to create:
- `ChatWindow.js` - Main chat container
- `ChatModeSelector.js` - Mode dropdown
- `MessageList.js` - Display messages
- `MessageInput.js` - Input field

**Implementation details provided in Phase 5 after testing**

---

## Phase 5: Integration Testing

### Step 5.1: Manual Testing Checklist

**Local Development:**
- [ ] Start service: `cd midwest-7-service && npm start`
- [ ] Start UI: `cd midwest-7-ui && npm start`
- [ ] Open two browser windows (User A and User B)
- [ ] Login as different users in each window
- [ ] Verify Socket.io connection in browser console
- [ ] Select User B from online users list (User A's window)
- [ ] Select a chat mode (e.g., Pirate)
- [ ] Type "Hello" and send
- [ ] Verify message transforms to pirate speak
- [ ] Verify User B receives message in real-time
- [ ] User B responds with different mode (e.g., Robot)
- [ ] Verify both messages display correctly
- [ ] Refresh page - verify chat history loads
- [ ] Disconnect both users - verify chat cleanup

**AI Integration:**
- [ ] Test all 10 chat modes
- [ ] Verify AI transformations are appropriate
- [ ] Test fallback when AI service unavailable
- [ ] Monitor transformation latency (< 3 seconds)

---

### Step 5.2: Azure Deployment

**Pre-Deployment Checklist:**
- [ ] Azure Web App Configuration complete
- [ ] Web Sockets enabled
- [ ] Always On enabled
- [ ] ARR Affinity enabled
- [ ] Environment variables set:
  - `AZURE_OPENAI_ENDPOINT`
  - `AZURE_OPENAI_API_KEY`
  - `AZURE_OPENAI_DEPLOYMENT`
  - `AZURE_OPENAI_API_VERSION`
  - `ALLOWED_ORIGINS`
  - `NODE_ENV=production`

**Deployment Steps:**
```bash
# Service deployment
cd midwest-7-service
git add .
git commit -m "Add chat with AI modes feature"
git push azure main

# UI deployment
cd midwest-7-ui
# Update .env.production with Azure URLs
npm run build
# Deploy via Azure Static Web Apps
```

**Post-Deployment Testing:**
- [ ] Verify Socket.io connection from Static Web App
- [ ] Test message transformation in production
- [ ] Verify SQLite persistence in `/home/data`
- [ ] Monitor Azure OpenAI API usage
- [ ] Test cleanup logic in production

---

## Phase 6: Documentation Updates

### Step 6.1: Update Project Documentation

- [x] Architecture diagrams in `docs/architecture.md`
- [ ] Update `README.md` with chat feature instructions
- [ ] Document environment variables in `.env.example`
- [ ] Add troubleshooting guide for Socket.io connection issues

---

## Environment Variables Reference

### Service Layer (.env)
```bash
# Azure OpenAI Configuration
AZURE_OPENAI_ENDPOINT=https://midwest7-openai-northcentral.openai.azure.com/
AZURE_OPENAI_API_KEY=your-api-key-here
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
AZURE_OPENAI_API_VERSION=2025-01-01-preview

# CORS Configuration
ALLOWED_ORIGINS=http://localhost:3000

# Server Configuration
NODE_ENV=development
PORT=3001
```

### UI Layer (.env)
```bash
REACT_APP_API_URL=http://localhost:3001
REACT_APP_SOCKET_URL=http://localhost:3001
```

### UI Layer (.env.production)
```bash
REACT_APP_API_URL=https://your-service.azurewebsites.net
REACT_APP_SOCKET_URL=https://your-service.azurewebsites.net
```

---

## Acceptance Criteria Checklist

**Must Have:**
- [ ] 10 chat modes available and working
- [ ] Real-time message delivery via Socket.io
- [ ] AI transforms messages correctly
- [ ] Messages persist in SQLite database
- [ ] Messages cleanup when both users offline
- [ ] Online users list updates in real-time
- [ ] Each user can select their own mode independently

**Nice to Have:**
- [ ] Typing indicators
- [ ] Message delivery confirmation
- [ ] Read receipts
- [ ] Message search

---

## Definition of Done

**Code Quality:**
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] No linting errors
- [ ] Code follows existing patterns
- [ ] API keys not committed to Git

**Documentation:**
- [x] Architecture diagrams updated
- [ ] README updated
- [ ] Environment setup documented
- [ ] Troubleshooting guide created

**Testing:**
- [ ] Manual testing complete (local)
- [ ] Manual testing complete (Azure)
- [ ] All 10 modes tested
- [ ] Performance acceptable (< 3s transformation)

**Deployment:**
- [ ] Service deployed to Azure Web App
- [ ] UI deployed to Static Web App
- [ ] Environment variables configured
- [ ] Socket.io working in production
- [ ] AI integration working in production

---

## Troubleshooting Guide

### Common Issues

**Socket.io not connecting:**
- Check Web Sockets enabled in Azure
- Verify CORS configuration
- Check browser console for errors
- Verify Socket URL in UI .env file

**AI transformation failing:**
- Verify API key is correct
- Check Azure OpenAI quota/limits
- Review error logs in console
- Test fallback mechanism

**Messages not persisting:**
- Check SQLite database path (`/home/data`)
- Verify database schema created
- Check file permissions in Azure

**Chat not cleaning up:**
- Verify both users marked as offline
- Check cleanup logic in chatSocket.js
- Review database foreign key constraints

---

## Next Steps After Completion

1. Move story folder to `stories/archive/55555-introduce-chat-and-chat-modes`
2. Update main project README with chat feature
3. Document lessons learned
4. Plan next iteration:
   - File sharing in chat
   - Voice messages
   - Chat notifications
   - Group chat support

---

**You're ready to implement! Start with Phase 1, Step 1.1** 🚀

Follow TDD: Red → Green → Refactor for each step.
