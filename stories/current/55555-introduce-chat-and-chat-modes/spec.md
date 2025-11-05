# Feature Specification: Introduce Chat and Chat Modes

**Story ID:** 55555  
**Feature:** introduce-chat-and-chat-modes  
**Status:** In Development

---

## User Story Overview

**As a** User  
**I want to** select a creative chat mode that transforms my message with personality and emojis  
**So that** I can send personalized, fun messages to other online users and receive responses in the same style

---

## Feature Description

This feature introduces a real-time chat system with 10 creative personality modes that transform messages before sending. Users can select any online user, choose a chat mode, type their message, and have it automatically transformed with personality-specific language and emojis before delivery. Each participant can choose their own chat mode for their responses, allowing for dynamic and fun conversations with different personality combinations.

### 10 Creative Chat Modes

1. **😎 Pirate Mode** - "Ahoy matey! ⚓🏴‍☠️"
2. **🎭 Shakespeare Mode** - "Verily, thy words... 📜✨"
3. **🤖 Robot Mode** - "BEEP.BOOP.PROCESSING... 🔧⚙️"
4. **😱 Horror Mode** - "Your words... whispered in shadows... 👻🕷️"
5. **🎉 Party Mode** - "YAAAS! Everything! Is! EXCITING! 🥳💃"
6. **🧙‍♂️ Fantasy Mode** - "By ancient magic... ✨🐉"
7. **👽 Alien Mode** - "Greetings earthling... 🛸👾"
8. **🕵️ Detective Mode** - "The evidence suggests... 🔍📝"
9. **💼 Corporate Mode** - "Circling back to synergize... 📊💻"
10. **🦖 Gen Z Mode** - "No cap, this is bussin fr fr! 💅✨"

### Example Flow
User selects online "UserA" → selects "Pirate Mode" → types "Hi" → UserA receives "Ahoy matey! ⚓🏴‍☠️" → UserA responds back with their own chosen mode (e.g., Robot Mode: "BEEP.BOOP.ACKNOWLEDGED... 🔧⚙️")

---

## What We're Building

1. User interface to select an online user to chat with
2. Chat mode selector with 10 personality modes (each user can select their own mode)
3. Message transformation service that converts messages based on sender's selected mode
4. Real-time chat thread that updates continuously for both users, showing transformed messages
5. Chat persistence that saves messages in DB and clears when both users go offline

---

## Technical Implementation

### Architecture Overview

**UI Layer (midwest-7-ui):**
- Online users list component
- Chat mode selector
- Chat message interface
- Real-time message display

**Service Layer (midwest-7-service):**
- Chat message transformation logic (10 modes)
- Chat thread management
- Real-time message delivery
- Chat persistence with conditional cleanup

### Files to Create/Modify

#### UI Repository (midwest-7-ui/src/)
```
src/
├── components/
│   ├── Chat/
│   │   ├── ChatWindow.js           (NEW) - Main chat interface
│   │   ├── ChatModeSelector.js     (NEW) - Mode selection dropdown
│   │   ├── MessageList.js          (NEW) - Chat thread display
│   │   ├── MessageInput.js         (NEW) - Message composition
│   │   └── OnlineUsersList.js      (NEW) - List of available users
│   └── Chat.css                     (NEW) - Chat styling
├── services/
│   └── chatService.js              (NEW) - API calls for chat
├── contexts/
│   └── ChatContext.js              (NEW) - Chat state management
└── utils/
    └── chatHelpers.js              (NEW) - Chat utilities
```

#### Service Repository (midwest-7-service/)
```
routes/
├── chat.js                         (NEW) - Chat endpoints
db/
├── chats.js                        (NEW) - Chat database operations
services/
└── messageTransformer.js           (NEW) - Mode transformation logic
```

### Implementation Steps

1. **Backend - Message Transformation Service**
   - Create `messageTransformer.js` with 10 mode transformation functions
   - Implement personality-specific text transformations
   - Add emoji injection logic for each mode

2. **Backend - Chat API & Database**
   - Create `db/chats.js` for chat persistence
   - Implement chat thread storage with user associations
   - Add cleanup logic for when both users go offline
   - Create `routes/chat.js` with endpoints for send/receive/fetch

3. **Backend - Real-time Updates**
   - Set up WebSocket or polling mechanism for live chat
   - Implement message broadcasting to chat participants
   - Add online/offline status tracking

4. **Frontend - Online Users List**
   - Create `OnlineUsersList.js` component
   - Fetch and display currently online users
   - Add user selection functionality

5. **Frontend - Chat Mode Selector**
   - Create `ChatModeSelector.js` with 10 mode options
   - Add mode icons and descriptions
   - Handle mode selection state (each user can change their mode independently)
   - Display current selected mode indicator

6. **Frontend - Chat Interface**
   - Create `ChatWindow.js` main component
   - Implement `MessageList.js` for thread display
   - Build `MessageInput.js` for message composition
   - Integrate mode selector with message sending

7. **Frontend - Real-time Chat Updates**
   - Implement real-time message fetching
   - Add auto-scroll for new messages
   - Handle typing indicators (optional)

8. **Integration & Testing**
   - Test all 10 chat modes with various messages
   - Verify chat persistence and cleanup logic
   - Test real-time updates between users
   - Validate edge cases (user goes offline mid-chat)

---

## Data Models

### Chat Message
```javascript
{
  id: "msg-uuid-123",
  chatThreadId: "thread-uuid-456",
  senderId: "user-123",
  receiverId: "user-456",
  originalMessage: "Hi",
  transformedMessage: "Ahoy matey! ⚓🏴‍☠️",
  chatMode: "pirate",
  timestamp: "2025-11-05T10:30:00Z",
  isRead: false
}
```

### Chat Thread
```javascript
{
  id: "thread-uuid-456",
  participants: ["user-123", "user-456"],
  messages: [/* array of message objects */],
  createdAt: "2025-11-05T10:00:00Z",
  lastMessageAt: "2025-11-05T10:30:00Z",
  activeParticipants: ["user-123", "user-456"] // for cleanup logic
}
```

### Chat Mode Definition
```javascript
{
  id: "pirate",
  name: "Pirate Mode",
  emoji: "😎",
  description: "Ahoy matey! ⚓🏴‍☠️",
  transformFunction: transformToPirate
}
```

---

## API Endpoints

### `POST /api/chat/send`
**Description:** Send a message with selected chat mode

**Request:**
```json
{
  "receiverId": "user-456",
  "message": "Hi",
  "chatMode": "pirate"
}
```

**Response (200):**
```json
{
  "success": true,
  "chatThreadId": "thread-uuid-456",
  "message": {
    "id": "msg-uuid-123",
    "originalMessage": "Hi",
    "transformedMessage": "Ahoy matey! ⚓🏴‍☠️",
    "chatMode": "pirate",
    "timestamp": "2025-11-05T10:30:00Z"
  }
}
```

**Errors:**
- 400: Invalid chat mode or missing fields
- 404: Receiver not found or offline
- 401: Unauthorized

### `GET /api/chat/threads/:userId`
**Description:** Get all active chat threads for a user

**Response (200):**
```json
{
  "threads": [
    {
      "id": "thread-uuid-456",
      "otherUser": {
        "id": "user-456",
        "username": "john_doe",
        "isOnline": true
      },
      "lastMessage": {
        "transformedMessage": "Ahoy matey! ⚓🏴‍☠️",
        "timestamp": "2025-11-05T10:30:00Z"
      },
      "unreadCount": 2
    }
  ]
}
```

### `GET /api/chat/messages/:threadId`
**Description:** Get all messages in a chat thread

**Response (200):**
```json
{
  "threadId": "thread-uuid-456",
  "messages": [
    {
      "id": "msg-uuid-123",
      "senderId": "user-123",
      "transformedMessage": "Ahoy matey! ⚓🏴‍☠️",
      "chatMode": "pirate",
      "timestamp": "2025-11-05T10:30:00Z"
    }
  ]
}
```

### `DELETE /api/chat/cleanup/:threadId`
**Description:** Cleanup chat thread (called when both users offline)

**Response (200):**
```json
{
  "success": true,
  "message": "Chat thread cleaned up"
}
```

---

## Key Logic & Algorithms

### Message Transformation Logic
Each chat mode will have a transformation function that:
1. Analyzes the input message
2. Applies mode-specific vocabulary changes
3. Adds personality-appropriate emojis
4. Maintains message intent while adding flair

**Example - Pirate Mode Transformer:**
```javascript
function transformToPirate(message) {
  const pirateVocab = {
    'hi': 'Ahoy',
    'hello': 'Ahoy matey',
    'yes': 'Aye',
    'no': 'Nay',
    'friend': 'matey',
    'you': 'ye',
    // ... more mappings
  };
  
  let transformed = message.toLowerCase();
  
  // Replace words with pirate vocabulary
  Object.keys(pirateVocab).forEach(word => {
    const regex = new RegExp(`\\b${word}\\b`, 'gi');
    transformed = transformed.replace(regex, pirateVocab[word]);
  });
  
  // Add pirate emojis
  const pirateEmojis = ['⚓', '🏴‍☠️', '☠️', '🦜'];
  const randomEmoji = pirateEmojis[Math.floor(Math.random() * pirateEmojis.length)];
  transformed += ` ${randomEmoji}`;
  
  return transformed;
}
```

### Chat Cleanup Logic
```javascript
function shouldCleanupThread(threadId) {
  const thread = getThread(threadId);
  const participants = thread.participants;
  
  // Check if both users are offline
  const onlineStatuses = participants.map(userId => getUserOnlineStatus(userId));
  const allOffline = onlineStatuses.every(status => status === false);
  
  if (allOffline) {
    deleteThread(threadId);
    return true;
  }
  
  return false; // Keep thread if at least one user is online
}
```

---

## Testing Plan

### Unit Tests

**Backend:**
- ✅ Test each of the 10 chat mode transformers with various inputs
- ✅ Test message validation and sanitization
- ✅ Test chat thread creation and retrieval
- ✅ Test cleanup logic (both users offline vs. one user offline)
- ✅ Test API error handling for invalid modes/users

**Frontend:**
- ✅ Test ChatModeSelector renders all 10 modes
- ✅ Test MessageInput validation and sending
- ✅ Test MessageList renders messages correctly
- ✅ Test OnlineUsersList filters and displays online users
- ✅ Test ChatWindow integration

### Integration Tests
- ✅ Test full chat flow: select user → select mode → send message → receive response (with different mode)
- ✅ Test real-time message updates between two users
- ✅ Test both users selecting different chat modes for their messages
- ✅ Test switching chat modes mid-conversation
- ✅ Test chat persistence across page refreshes (while users online)
- ✅ Test chat cleanup when both users go offline
- ✅ Test chat preservation when only one user goes offline
- ✅ Test multiple conversations with different mode combinations

### Manual Testing
- ✅ Send messages in all 10 chat modes and verify transformations
- ✅ Test chat with multiple online users simultaneously
- ✅ Verify real-time updates (send message, see it appear immediately)
- ✅ Test offline/online transitions and chat cleanup
- ✅ Test edge cases: special characters, emojis in original message, very long messages
- ✅ Test mobile responsiveness of chat interface
- ✅ Verify chat mode selector is accessible and user-friendly

---

## Dependencies

### Required Packages
- **Backend:** None (use existing Node.js/Express)
- **Frontend:** None (use existing React)
- **Optional:** Socket.io for real-time updates (or use polling)

### Story Dependencies
- ✅ Depends on: User authentication and online status tracking (existing)
- ✅ Blocks: None
- ✅ Related to: User profile features, notification system (future)

---

## Security & Performance Notes

### Security
- Validate all message input to prevent XSS attacks
- Sanitize messages before transformation and storage
- Ensure users can only access their own chat threads
- Rate limit message sending to prevent spam
- Validate chat mode selection against allowed modes list

### Performance
- Index chat threads by userId for fast retrieval
- Implement message pagination for large chat histories
- Cache frequently used transformation logic
- Consider WebSocket connection limits for real-time updates
- Optimize database queries for chat cleanup (batch operations)

### Data Privacy
- Do not store sensitive information in chat messages
- Clear chat data when both users offline (privacy by design)
- Consider adding user setting to auto-delete chats after X hours

---

## Acceptance Criteria

### Must Have
- ✅ User can view list of online users
- ✅ User can select an online user to chat with
- ✅ User can select from 10 different chat modes for their own messages
- ✅ Each user can independently choose their own chat mode when responding
- ✅ Messages are transformed correctly based on sender's selected mode
- ✅ Both users see the chat thread update in real-time
- ✅ Chat persists while at least one user is online
- ✅ Chat is automatically cleaned up when both users go offline
- ✅ All 10 chat modes produce unique, personality-appropriate transformations

### Nice to Have
- Typing indicators
- Message read receipts
- Chat mode preview before sending
- Ability to switch modes mid-conversation
- Message reactions

---

## Definition of Done

- ✅ All 10 chat modes implemented and tested
- ✅ Backend API endpoints created and documented
- ✅ Frontend chat interface built and integrated
- ✅ Real-time chat updates working
- ✅ Chat persistence and cleanup logic verified
- ✅ Unit tests written and passing (backend & frontend)
- ✅ Integration tests written and passing
- ✅ Manual testing completed for all modes and scenarios
- ✅ Code reviewed and approved
- ✅ Documentation updated (API docs, component docs)
- ✅ Security review completed
- ✅ Performance benchmarks met (message send/receive < 500ms)
- ✅ Deployed to staging environment
- ✅ Product owner approval received

---

**Next Step:** Review this specification and confirm before proceeding to implementation planning.
