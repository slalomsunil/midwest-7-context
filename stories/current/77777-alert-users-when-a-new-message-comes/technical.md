# Technical Specification - Alert Users When a New Message Comes

## What We're Building
Implementation of real-time message notifications with visual blinking indicators for usernames and automatic session cleanup on backend restart.

## Technical Approach

### Files to Modify/Create

**Service Layer (midwest-7-service):**
```
├── app.js                           (MODIFY - add session cleanup on restart)
├── config/socketConfig.js           (MODIFY - enhance WebSocket handling)  
├── routes/messages.js               (MODIFY - add notification triggers)
├── services/sessionService.js       (NEW - session management)
└── __tests__/
    ├── services/sessionService.test.js  (NEW)
    └── routes/messages.test.js          (MODIFY)
```

**UI Layer (midwest-7-ui):**
```
src/
├── components/
│   ├── OnlineUsers.tsx              (MODIFY - add blinking logic)
│   ├── Chat.tsx                     (MODIFY - handle notifications)
│   └── NotificationIndicator.tsx    (NEW - reusable indicator component)
├── services/
│   ├── websocketService.ts          (MODIFY - handle notification events)
│   └── notificationService.ts       (NEW - notification management)
├── hooks/
│   └── useMessageNotifications.ts   (NEW - notification hook)
├── styles/
│   └── notifications.css            (NEW - blinking animations)
└── types/
    └── notification.types.ts        (NEW - notification interfaces)
```

### Implementation Steps

1. **Backend Session Management**
   - Implement session cleanup service that triggers on app restart
   - Add WebSocket event broadcasting for session clearing
   - Update socket configuration to handle reconnection scenarios

2. **Message Notification System**
   - Modify message routes to emit notification events
   - Implement WebSocket events for real-time notifications
   - Add message read status tracking

3. **Frontend Visual Indicators**
   - Create blinking animation CSS classes
   - Implement notification hook for managing blink states
   - Update OnlineUsers component with notification logic

4. **Progressive Notification States**
   - Implement initial blinking phase (5-10 seconds)
   - Transition to persistent color/icon indicator
   - Handle multiple messages and notification stacking

## Data Changes

### WebSocket Events
```javascript
// New WebSocket events
'session_cleared'     // Broadcasted on backend restart
'message_notification' // Sent when new message arrives
'notification_read'    // Sent when user reads message
```

### Session Storage
```javascript
// Local storage for notification state
interface NotificationState {
  userId: string;
  hasUnreadMessage: boolean;
  isBlinking: boolean;
  lastMessageTime: number;
}
```

## API Changes

### Enhanced WebSocket Events
```javascript
// Message notification event
socket.emit('message_notification', {
  fromUserId: 'user-123',
  toUserId: 'user-456',
  messagePreview: 'Hello there!',
  timestamp: Date.now()
});

// Session cleanup event  
socket.broadcast.emit('session_cleared', {
  reason: 'server_restart',
  timestamp: Date.now()
});
```

### CSS Animation Classes
```css
/* Blinking animation */
.username-blinking {
  animation: blink 1s ease-in-out infinite;
}

/* Persistent notification indicator */
.username-has-notification {
  color: #ff6b35;
  font-weight: bold;
}

.username-has-notification::after {
  content: '●';
  color: #ff6b35;
  margin-left: 5px;
}
```

## Testing Strategy

### Backend Tests
- Session cleanup on restart scenarios
- WebSocket event emission for notifications
- Message delivery with notification triggers

### Frontend Tests  
- Blinking animation lifecycle
- Notification state management
- WebSocket event handling
- Accessibility compliance for animations

## Performance Considerations
- Debounce multiple rapid messages from same user
- Cleanup notification timers on component unmount
- Optimize CSS animations for smooth performance
- Limit concurrent notification animations

## Accessibility Notes
- Respect `prefers-reduced-motion` media query
- Provide alternative indicators for users who disable animations
- Ensure color contrast meets WCAG guidelines
- Add ARIA labels for notification states
