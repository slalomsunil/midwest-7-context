# 77777 - Alert Users When a New Message Comes - Implementation Guide

## Implementation Overview

This guide provides step-by-step instructions for implementing real-time message notifications with visual indicators and session management. The implementation follows TDD principles and spans both the service and UI layers.

## Phase 1: Backend Implementation (Service Layer)

### Step 1: Session Management Service

**File: `midwest-7-service/services/sessionService.js`**

```javascript
// Create new session service for managing online users
class SessionService {
  constructor() {
    this.sessions = new Map();
  }

  addSession(userId, socketId) {
    this.sessions.set(userId, { socketId, lastSeen: Date.now() });
  }

  removeSession(userId) {
    this.sessions.delete(userId);
  }

  clearAllSessions() {
    this.sessions.clear();
  }

  getAllSessions() {
    return Array.from(this.sessions.keys());
  }

  isUserOnline(userId) {
    return this.sessions.has(userId);
  }
}

module.exports = new SessionService();
```

**Test First: `midwest-7-service/__tests__/services/sessionService.test.js`**

```javascript
const SessionService = require('../../services/sessionService');

describe('SessionService', () => {
  beforeEach(() => {
    SessionService.clearAllSessions();
  });

  test('should add and track user sessions', () => {
    SessionService.addSession('user1', 'socket1');
    expect(SessionService.isUserOnline('user1')).toBe(true);
  });

  test('should clear all sessions on restart', () => {
    SessionService.addSession('user1', 'socket1');
    SessionService.addSession('user2', 'socket2');
    SessionService.clearAllSessions();
    expect(SessionService.getAllSessions()).toHaveLength(0);
  });
});
```

### Step 2: Enhanced Socket Configuration

**File: `midwest-7-service/config/socketConfig.js` (MODIFY)**

```javascript
// Add session cleanup and notification events
const sessionService = require('../services/sessionService');

// In the socket connection handler, add:
socket.on('user_connected', (userId) => {
  sessionService.addSession(userId, socket.id);
  // Broadcast updated online users
  io.emit('online_users_updated', sessionService.getAllSessions());
});

socket.on('disconnect', () => {
  // Find and remove user session
  const userId = findUserBySocketId(socket.id);
  if (userId) {
    sessionService.removeSession(userId);
    io.emit('online_users_updated', sessionService.getAllSessions());
  }
});

// Add server restart handler
process.on('SIGTERM', () => {
  sessionService.clearAllSessions();
  io.emit('session_cleared', { reason: 'server_restart', timestamp: Date.now() });
});
```

### Step 3: Message Notification Events

**File: `midwest-7-service/routes/messages.js` (MODIFY)**

```javascript
// In the message creation endpoint, add notification emission
router.post('/messages', (req, res) => {
  const { fromUserId, toUserId, message } = req.body;
  
  // Save message (existing logic)
  // ...

  // Emit notification event
  io.emit('message_notification', {
    fromUserId,
    toUserId,
    messagePreview: message.substring(0, 50),
    timestamp: Date.now()
  });

  res.json({ success: true });
});
```

**Test: `midwest-7-service/__tests__/routes/messages.test.js` (MODIFY)**

```javascript
// Add test for notification emission
test('should emit notification when message is sent', async () => {
  const mockEmit = jest.fn();
  io.emit = mockEmit;

  await request(app)
    .post('/api/messages')
    .send({
      fromUserId: 'user1',
      toUserId: 'user2',
      message: 'Hello there!'
    })
    .expect(200);

  expect(mockEmit).toHaveBeenCalledWith('message_notification', {
    fromUserId: 'user1',
    toUserId: 'user2',
    messagePreview: 'Hello there!',
    timestamp: expect.any(Number)
  });
});
```

### Step 4: App.js Modifications

**File: `midwest-7-service/app.js` (MODIFY)**

```javascript
// Add session cleanup on startup
const sessionService = require('./services/sessionService');

// Add after app initialization
app.on('ready', () => {
  console.log('Server starting - clearing all sessions');
  sessionService.clearAllSessions();
});

// Add graceful shutdown
process.on('SIGTERM', () => {
  sessionService.clearAllSessions();
  process.exit(0);
});
```

## Phase 2: Frontend Implementation (UI Layer)

### Step 5: Notification Types and Interfaces

**File: `midwest-7-ui/src/types/notification.types.ts`**

```typescript
export interface NotificationState {
  userId: string;
  hasUnreadMessage: boolean;
  isBlinking: boolean;
  lastMessageTime: number;
  blinkStartTime?: number;
}

export interface MessageNotification {
  fromUserId: string;
  toUserId: string;
  messagePreview: string;
  timestamp: number;
}

export interface NotificationConfig {
  blinkDuration: number;
  blinkInterval: number;
  persistentIndicatorTimeout: number;
}
```

### Step 6: Notification Service

**File: `midwest-7-ui/src/services/notificationService.ts`**

```typescript
import { NotificationState, MessageNotification } from '../types/notification.types';

class NotificationService {
  private notifications: Map<string, NotificationState> = new Map();
  private config = {
    blinkDuration: 10000, // 10 seconds
    blinkInterval: 1000,  // 1 second
    persistentIndicatorTimeout: 30000 // 30 seconds
  };

  addNotification(userId: string): void {
    const existing = this.notifications.get(userId);
    if (existing?.isBlinking) return; // Don't duplicate

    this.notifications.set(userId, {
      userId,
      hasUnreadMessage: true,
      isBlinking: true,
      lastMessageTime: Date.now(),
      blinkStartTime: Date.now()
    });

    // Set timeout to stop blinking
    setTimeout(() => {
      this.stopBlinking(userId);
    }, this.config.blinkDuration);
  }

  stopBlinking(userId: string): void {
    const notification = this.notifications.get(userId);
    if (notification) {
      this.notifications.set(userId, {
        ...notification,
        isBlinking: false
      });
    }
  }

  clearNotification(userId: string): void {
    this.notifications.delete(userId);
  }

  getNotificationState(userId: string): NotificationState | null {
    return this.notifications.get(userId) || null;
  }

  clearAllNotifications(): void {
    this.notifications.clear();
  }
}

export default new NotificationService();
```

### Step 7: Message Notifications Hook

**File: `midwest-7-ui/src/hooks/useMessageNotifications.ts`**

```typescript
import { useState, useEffect } from 'react';
import { MessageNotification } from '../types/notification.types';
import notificationService from '../services/notificationService';
import websocketService from '../services/websocketService';

export const useMessageNotifications = (currentUserId: string) => {
  const [notifications, setNotifications] = useState<Map<string, any>>(new Map());

  useEffect(() => {
    const handleMessageNotification = (data: MessageNotification) => {
      if (data.toUserId === currentUserId) {
        notificationService.addNotification(data.fromUserId);
        updateNotifications();
      }
    };

    const handleSessionCleared = () => {
      notificationService.clearAllNotifications();
      updateNotifications();
    };

    const updateNotifications = () => {
      setNotifications(new Map(notificationService['notifications']));
    };

    websocketService.on('message_notification', handleMessageNotification);
    websocketService.on('session_cleared', handleSessionCleared);

    return () => {
      websocketService.off('message_notification', handleMessageNotification);
      websocketService.off('session_cleared', handleSessionCleared);
    };
  }, [currentUserId]);

  const clearNotification = (userId: string) => {
    notificationService.clearNotification(userId);
    setNotifications(new Map(notificationService['notifications']));
  };

  return {
    notifications,
    clearNotification
  };
};
```

### Step 8: Notification Indicator Component

**File: `midwest-7-ui/src/components/NotificationIndicator.tsx`**

```tsx
import React from 'react';
import { NotificationState } from '../types/notification.types';
import '../styles/notifications.css';

interface NotificationIndicatorProps {
  notificationState: NotificationState | null;
  username: string;
  onClick: () => void;
}

export const NotificationIndicator: React.FC<NotificationIndicatorProps> = ({
  notificationState,
  username,
  onClick
}) => {
  const getClassName = () => {
    if (!notificationState?.hasUnreadMessage) return 'username-normal';
    if (notificationState.isBlinking) return 'username-blinking';
    return 'username-has-notification';
  };

  return (
    <span 
      className={getClassName()}
      onClick={onClick}
      role="button"
      tabIndex={0}
      aria-label={
        notificationState?.hasUnreadMessage 
          ? `${username} has new messages` 
          : username
      }
    >
      {username}
    </span>
  );
};
```

### Step 9: CSS Animations

**File: `midwest-7-ui/src/styles/notifications.css`**

```css
/* Respect reduced motion preferences */
@media (prefers-reduced-motion: reduce) {
  .username-blinking {
    animation: none;
    color: #ff6b35;
    font-weight: bold;
  }
}

/* Blinking animation */
@keyframes blink {
  0%, 50% { opacity: 1; color: #ff6b35; font-weight: bold; }
  51%, 100% { opacity: 0.3; color: #ff6b35; font-weight: bold; }
}

.username-blinking {
  animation: blink 1s ease-in-out infinite;
}

/* Persistent notification indicator */
.username-has-notification {
  color: #ff6b35;
  font-weight: bold;
  position: relative;
}

.username-has-notification::after {
  content: '●';
  color: #ff6b35;
  margin-left: 5px;
  font-size: 0.8em;
}

.username-normal {
  color: inherit;
  font-weight: normal;
}

/* Accessibility improvements */
.username-blinking:focus,
.username-has-notification:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

### Step 10: Update OnlineUsers Component

**File: `midwest-7-ui/src/components/OnlineUsers.tsx` (MODIFY)**

```tsx
import React from 'react';
import { useMessageNotifications } from '../hooks/useMessageNotifications';
import { NotificationIndicator } from './NotificationIndicator';

// In the OnlineUsers component:
export const OnlineUsers: React.FC<OnlineUsersProps> = ({ currentUserId }) => {
  const { notifications, clearNotification } = useMessageNotifications(currentUserId);

  const handleUserClick = (userId: string) => {
    clearNotification(userId);
    // Navigate to chat with user (existing logic)
  };

  return (
    <div className="online-users">
      {onlineUsers.map(user => (
        <NotificationIndicator
          key={user.id}
          notificationState={notifications.get(user.id) || null}
          username={user.username}
          onClick={() => handleUserClick(user.id)}
        />
      ))}
    </div>
  );
};
```

## Phase 3: Testing Implementation

### Step 11: Frontend Tests

**File: `midwest-7-ui/src/__tests__/hooks/useMessageNotifications.test.tsx`**

```typescript
import { renderHook, act } from '@testing-library/react';
import { useMessageNotifications } from '../../hooks/useMessageNotifications';

describe('useMessageNotifications', () => {
  test('should add notification when message received', () => {
    const { result } = renderHook(() => useMessageNotifications('user1'));
    
    act(() => {
      // Simulate WebSocket message
      window.dispatchEvent(new CustomEvent('message_notification', {
        detail: {
          fromUserId: 'user2',
          toUserId: 'user1',
          messagePreview: 'Hello!',
          timestamp: Date.now()
        }
      }));
    });

    expect(result.current.notifications.has('user2')).toBe(true);
  });
});
```

### Step 12: Component Tests

**File: `midwest-7-ui/src/__tests__/components/NotificationIndicator.test.tsx`**

```tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { NotificationIndicator } from '../../components/NotificationIndicator';

describe('NotificationIndicator', () => {
  test('should show blinking class when notification is blinking', () => {
    const mockNotification = {
      userId: 'user1',
      hasUnreadMessage: true,
      isBlinking: true,
      lastMessageTime: Date.now()
    };

    render(
      <NotificationIndicator
        notificationState={mockNotification}
        username="John"
        onClick={() => {}}
      />
    );

    expect(screen.getByText('John')).toHaveClass('username-blinking');
  });
});
```

## Phase 4: Integration and Validation

### Step 13: WebSocket Service Updates

**File: `midwest-7-ui/src/services/websocketService.ts` (MODIFY)**

```typescript
// Add new event handlers
export class WebSocketService {
  // ... existing code ...

  setupEventListeners() {
    this.socket?.on('message_notification', this.handleMessageNotification);
    this.socket?.on('session_cleared', this.handleSessionCleared);
    this.socket?.on('online_users_updated', this.handleOnlineUsersUpdate);
  }

  private handleMessageNotification = (data: MessageNotification) => {
    this.emit('message_notification', data);
  };

  private handleSessionCleared = (data: any) => {
    this.emit('session_cleared', data);
  };
}
```

### Step 14: Run Tests and Validate

```bash
# Backend tests
cd midwest-7-service
npm test

# Frontend tests  
cd midwest-7-ui
npm test

# Integration testing
npm run test:e2e
```

## Deployment Checklist

- [ ] All unit tests pass
- [ ] Integration tests validate WebSocket events
- [ ] Accessibility testing with screen readers
- [ ] Cross-browser testing (Chrome, Firefox, Safari, Edge)
- [ ] Performance testing for animation smoothness
- [ ] Session cleanup works on Azure deployment
- [ ] WebSocket reconnection handling tested
- [ ] Mobile responsiveness verified

## Rollback Plan

If issues occur:
1. Disable notification features via feature flag
2. Revert to previous WebSocket configuration
3. Remove CSS animations if causing performance issues
4. Fallback to polling-based updates if WebSocket fails

## Monitoring and Metrics

- Track notification delivery success rate
- Monitor WebSocket connection stability
- Measure user engagement with notification features
- Track accessibility usage patterns
- Monitor performance impact of animations