# Quick Start Guide - Chat Feature

## What's Been Completed ✅

The **backend infrastructure** for real-time chat with AI-powered tone modes is fully implemented and tested.

### Service Layer (100% Complete)
- ✅ Socket.io server configured
- ✅ Azure OpenAI integration
- ✅ 10 chat modes with AI transformation
- ✅ Message caching (1-hour TTL)
- ✅ Database schema with indexes
- ✅ Automatic chat cleanup
- ✅ Fallback when AI unavailable
- ✅ All tests passing (11/11)

### UI Layer (Partial)
- ✅ Socket.io client service
- ✅ Chat modes configuration
- ⏳ React components (TODO)

---

## Running the Application

### 1. Start the Service
```bash
cd midwest-7-service
npm install  # If not already done
npm start    # Starts on port 8081
```

### 2. Start the UI
```bash
cd midwest-7-ui
npm install  # If not already done
npm start    # Starts on port 3000
```

### 3. Verify
- Service: http://localhost:8081
- UI: http://localhost:3000
- Swagger Docs: http://localhost:8081/api-docs

---

## Testing the Backend

### Run All Chat Tests
```bash
cd midwest-7-service
npm test -- --testPathPatterns="chatSchema|aiClient|messageTransformer" --no-coverage
```

### Run Individual Tests
```bash
npm test -- chatSchema.test.js --no-coverage
npm test -- aiClient.test.js --no-coverage
npm test -- messageTransformer.test.js --no-coverage
```

---

## How the Chat System Works

### 1. User Connection Flow
```javascript
// User logs in
socket.emit('user-join', { userId: 1 })

// Server updates online status in database
// Server broadcasts to others: 'user-online'
```

### 2. Sending a Message
```javascript
socket.emit('send-message', {
  senderId: 1,
  receiverId: 2,
  message: 'Hello',
  mode: 'pirate'  // One of 10 modes
})

// Server transforms message via Azure OpenAI
// Server saves to database
// Server sends to receiver: 'new-message'
// Server confirms to sender: 'message-sent'
```

### 3. Receiving a Message
```javascript
socket.on('new-message', (data) => {
  console.log(data.transformed)  // "Ahoy matey! ⚓🏴‍☠️"
})
```

### 4. Cleanup on Disconnect
```javascript
socket.disconnect()

// Server marks user offline
// If both users offline: deletes all chat messages between them
// Broadcasts: 'user-offline'
```

---

## The 10 Chat Modes

| Mode | ID | Emoji | Example Transformation |
|------|-------|-------|------------------------|
| Pirate | `pirate` | 🏴‍☠️ | "Ahoy matey! ⚓🏴‍☠️" |
| Shakespeare | `shakespeare` | 🎭 | "Verily, thy words... 📜✨" |
| Robot | `robot` | 🤖 | "BEEP.BOOP.PROCESSING... 🔧⚙️" |
| Horror | `horror` | 👻 | "Your words... whispered in shadows... 👻🕷️" |
| Party | `party` | 🎉 | "YAAAS! Everything! Is! EXCITING! 🥳💃" |
| Fantasy | `fantasy` | 🧙‍♂️ | "By ancient magic... ✨🐉" |
| Alien | `alien` | 👽 | "Greetings earthling... 🛸👾" |
| Detective | `detective` | 🕵️ | "The evidence suggests... 🔍📝" |
| Corporate | `corporate` | 💼 | "Circling back to synergize... 📊💻" |
| Gen Z | `genz` | 💅 | "No cap, this is bussin fr fr! 💅✨" |

---

## Socket.io Events Reference

### Client → Server

| Event | Data | Description |
|-------|------|-------------|
| `user-join` | `{ userId: number }` | User connects to chat |
| `send-message` | `{ senderId, receiverId, message, mode }` | Send a message |
| `disconnect` | - | User disconnects |

### Server → Client

| Event | Data | Description |
|-------|------|-------------|
| `user-online` | `{ userId: number }` | Another user came online |
| `user-offline` | `{ userId: number }` | Another user went offline |
| `new-message` | `{ id, senderId, receiverId, original, transformed, mode, createdAt }` | New message received |
| `message-sent` | `{ id, status, transformed }` | Confirmation message sent |
| `message-error` | `{ error: string }` | Error sending message |

---

## Database Schema

### chat_messages Table
```sql
CREATE TABLE chat_messages (
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

CREATE INDEX idx_chat_messages_users 
ON chat_messages(sender_id, receiver_id, created_at DESC);
```

---

## What Still Needs to Be Built

### React Components (UI Layer)

1. **ChatWindow.js**
   - Container for entire chat interface
   - Connects to Socket.io service
   - Manages chat state

2. **ChatModeSelector.js**
   - Dropdown to select tone mode
   - Shows mode emoji and description
   - Updates user's current mode

3. **MessageList.js**
   - Displays chat messages
   - Shows both original and transformed messages
   - Auto-scrolls to bottom
   - Shows timestamps

4. **MessageInput.js**
   - Text input field
   - Send button
   - Shows selected mode indicator
   - Handles Enter key to send

### Integration Tasks
- [ ] Import socket service in components
- [ ] Handle Socket.io events in React
- [ ] Display online users list
- [ ] Show typing indicators (optional)
- [ ] Add message delivery confirmation (optional)

---

## Environment Variables

### Service (.env)
```bash
AZURE_OPENAI_ENDPOINT=https://midwest7-openai-northcentral.openai.azure.com/
AZURE_OPENAI_API_KEY=your-api-key-here
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
AZURE_OPENAI_API_VERSION=2025-01-01-preview
ALLOWED_ORIGINS=http://localhost:3000
PORT=8081
```

### UI (.env.local)
```bash
REACT_APP_API_BASE_URL=http://localhost:8081
REACT_APP_SOCKET_URL=http://localhost:8081
```

---

## Troubleshooting

### Socket.io not connecting
- Check if service is running on port 8081
- Verify REACT_APP_SOCKET_URL is correct
- Check browser console for errors
- Verify ALLOWED_ORIGINS includes your UI URL

### AI transformation failing
- Check AZURE_OPENAI_API_KEY is valid
- Verify Azure OpenAI quota/limits
- Test fallback: messages should still send with emojis

### Tests failing
- Run `npm install` in midwest-7-service
- Check .env file has all required variables
- Clear jest cache: `npm test -- --clearCache`

### Database issues
- Check db/cache.db exists
- Verify chat_messages table: `echo "PRAGMA table_info(chat_messages);" | sqlite3 db/cache.db`
- Delete and restart: `rm db/cache.db && npm start`

---

## Next Steps

1. **Build UI Components** - Start with ChatWindow.js
2. **Connect to Socket.io** - Use src/services/socketService.js
3. **Test Locally** - Open two browser windows, chat between users
4. **Deploy to Azure** - Follow deployment checklist in implementation.md

---

## Useful Commands

```bash
# Run all tests
npm test

# Run specific test
npm test -- aiClient.test.js

# Start service
npm start

# Check database
echo "SELECT * FROM chat_messages;" | sqlite3 db/cache.db

# View API docs
open http://localhost:8081/api-docs
```

---

## Files You'll Need to Work With

**For UI Development:**
- `src/services/socketService.js` - Socket.io client (ready to use)
- `src/utils/chatModes.js` - Chat modes config (ready to use)
- `src/components/Chat/` - Create components here

**For Backend Changes:**
- `routes/chatSocket.js` - Socket event handlers
- `services/messageTransformer.js` - Message transformation logic
- `services/aiClient.js` - Azure OpenAI integration

---

**Questions?** Check the IMPLEMENTATION_SUMMARY.md for detailed architecture and decisions.
