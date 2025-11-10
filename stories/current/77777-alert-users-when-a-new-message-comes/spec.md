# 77777 - Alert Users When a New Message Comes - Specification

## Story Overview

As a chat application user
I want to receive real-time notifications when new messages arrive
So that I don't miss important conversations and can respond promptly

## Description

This story implements a comprehensive notification system that alerts users when new messages are received in the chat application. The system will provide both visual indicators and real-time updates to ensure users are aware of new messages. Based on the current requirements, this includes clearing online sessions when the backend restarts and implementing visual blinking notifications for usernames in the online users list when messages are received.

## Business Value

- Increases user engagement by ensuring timely responses to messages
- Improves user satisfaction by preventing missed communications
- Enhances the overall chat experience with immediate visual feedback
- Reduces message response time, leading to more active conversations

## Acceptance Criteria

### Session Management
- **Given** the backend application is running
- **When** the backend application restarts
- **Then** all online user sessions should be cleared automatically

### Visual Notifications
- **Given** I am logged into the chat application
- **When** another user sends me a message
- **Then** I should see the sender's username blink in the online users list

- **Given** I am viewing the online users list
- **When** a new message arrives from any user
- **Then** the corresponding username should have a visual blinking indicator

- **Given** a user's name is blinking due to a new message
- **When** I click on that user or read their message
- **Then** the blinking indicator should stop

### Additional Requirements
- Blinking should be subtle and not cause accessibility issues
- Session clearing should be automatic and not require user intervention
- Visual indicators should be consistent across different browsers
- Blinking animation should timeout after a reasonable period (30 seconds)
- Multiple messages from the same user should not create multiple blink instances

## Technical Implementation

### Architecture Changes

**Service Layer (midwest-7-service):**
- Modify `app.js` to add session cleanup on restart
- Enhance `config/socketConfig.js` for improved WebSocket handling
- Update `routes/messages.js` to add notification triggers
- Create new `services/sessionService.js` for session management

**UI Layer (midwest-7-ui):**
- Modify `OnlineUsers.tsx` to add blinking logic
- Update `Chat.tsx` to handle notifications
- Create `NotificationIndicator.tsx` as reusable component
- Enhance `websocketService.ts` for notification events
- Create `notificationService.ts` for notification management
- Add `useMessageNotifications.ts` hook
- Create notification CSS animations

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

### WebSocket Events

```javascript
// New WebSocket events
'session_cleared'     // Broadcasted on backend restart
'message_notification' // Sent when new message arrives
'notification_read'    // Sent when user reads message
```

### CSS Animation Implementation

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

## Testing Requirements

### Backend Testing
- Session cleanup on restart scenarios
- WebSocket event emission for notifications
- Message delivery with notification triggers

### Frontend Testing
- Blinking animation lifecycle
- Notification state management
- WebSocket event handling
- Accessibility compliance for animations

## Performance Considerations

- Debounce multiple rapid messages from same user
- Cleanup notification timers on component unmount
- Optimize CSS animations for smooth performance
- Limit concurrent notification animations

## Accessibility Requirements

- Respect `prefers-reduced-motion` media query
- Provide alternative indicators for users who disable animations
- Ensure color contrast meets WCAG guidelines
- Add ARIA labels for notification states

## Definition of Done

- [ ] Backend session cleanup works on application restart
- [ ] Username blinking works when new messages arrive
- [ ] Blinking transitions to persistent indicator after timeout
- [ ] WebSocket events properly handle notifications
- [ ] All tests pass (backend and frontend)
- [ ] Accessibility guidelines are met
- [ ] Cross-browser compatibility verified
- [ ] Performance benchmarks met
- [ ] Documentation updated