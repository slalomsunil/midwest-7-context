# C4 - Code Diagrams

## Overview

This document provides links to code-level architecture diagrams and patterns for both the frontend and backend systems.

## Code Diagrams by Repository

### Backend Code Patterns (Node.js API)

For detailed backend code patterns, see:

**[midwest-7-service/docs/c4-code.md](https://github.com/slalomsunil/midwest-7-service/blob/main/docs/c4-code.md)**

Key backend code patterns:
- Express middleware chain
- AI service integration (Azure OpenAI)
- Database access pattern (SQLite repository)
- Socket.IO event handling
- Route handler pattern (CommonJS)
- Configuration management
- Error handling patterns

### Frontend Code Patterns (React UI)

For detailed frontend code patterns, see:

**[midwest-7-ui/docs/c4-code.md](https://github.com/slalomsunil/midwest-7-ui/blob/main/docs/c4-code.md)**

Key frontend code patterns:
- React functional components with hooks
- Authentication flow and session management
- API service singleton pattern
- WebSocket service integration
- Component state management
- Controlled component pattern
- Testing with React Testing Library

## Cross-Repository Code Patterns

### API Communication Pattern

**Frontend (React)**:
```javascript
// API Service makes REST calls
const user = await apiService.login(username);
```

**Backend (Express)**:
```javascript
// Route handler processes request
router.post('/login', async (req, res) => {
  const user = await authService.login(req.body.username);
  res.json({ success: true, user });
});
```

### Real-Time Messaging Pattern

**Frontend (React)**:
```javascript
// Socket service listens for events
socketService.onMessage((message) => {
  setMessages(prev => [...prev, message]);
});

// Socket service emits events
socketService.sendMessage({ text, recipientId });
```

**Backend (Express)**:
```javascript
// Socket.IO handler processes events
io.on('connection', (socket) => {
  socket.on('message:send', (data) => {
    io.to(`user:${data.recipientId}`).emit('message:receive', message);
  });
});
```

### Error Handling Pattern

**Frontend**:
```javascript
try {
  const data = await apiService.getMessages(id);
  setMessages(data);
} catch (error) {
  setError(error.message);
  console.error('Failed to load messages:', error);
}
```

**Backend**:
```javascript
router.get('/messages', async (req, res, next) => {
  try {
    const messages = await chatService.getMessages(req.params.id);
    res.json({ success: true, messages });
  } catch (error) {
    next(error); // Global error handler
  }
});
```

## Shared Code Conventions

### Naming Conventions

**Backend (CommonJS)**:
- `var` for variables
- `require()` for imports
- `module.exports` for exports
- camelCase for functions and variables
- PascalCase for classes

**Frontend (ES6)**:
- `const`/`let` for variables
- `import`/`export` for modules
- camelCase for functions and variables
- PascalCase for React components

### File Organization

**Backend**:
```
routes/          # Route handlers
services/        # Business logic
db/             # Data access layer
config/         # Configuration
__tests__/      # Test files
```

**Frontend**:
```
components/     # React components
services/       # API and utility services
__tests__/      # Test files
```

### Async Patterns

**Both use async/await**:
```javascript
// Backend
async function getUser(id) {
  const user = await db.findUser(id);
  return user;
}

// Frontend
async function loadUser(id) {
  const user = await apiService.getUser(id);
  return user;
}
```

## Development Standards

### Code Quality

- **Linting**: ESLint for both repos
- **Formatting**: Consistent code style
- **Error Handling**: Try-catch blocks and error boundaries
- **Logging**: Structured logging for debugging

### Testing Approach

- **Backend**: Jest + Supertest for route/integration testing
- **Frontend**: Jest + React Testing Library for component testing
- **TDD**: Write tests before implementation (per Copilot instructions)

### Security Patterns

**Backend**:
- CORS configuration
- Security headers (CSP, X-Frame-Options)
- Input validation
- SQL injection prevention

**Frontend**:
- XSS prevention (React auto-escaping)
- Secure session storage
- HTTPS for API calls
- CORS credentials handling

## Related Diagrams

- [C1 - System Context](./c1-system-context.md) - High-level system view
- [C2 - Container Diagram](./c2-container.md) - Container architecture
- [C3 - Component Diagram](./c3-component.md) - Component-level view
- [Main Architecture](./architecture.md) - Complete architecture overview
