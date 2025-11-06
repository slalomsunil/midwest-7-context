# C3 - Component Diagrams

## Overview

This document provides links to the component-level architecture diagrams for both the frontend and backend systems.

## Component Diagrams by Repository

### Backend Components (Node.js API)

For detailed backend component architecture, see:

**[midwest-7-service/docs/c3-component.md](https://github.com/slalomsunil/midwest-7-service/blob/main/docs/c3-component.md)**

Key backend components:
- Express application and middleware stack
- Route handlers (auth, users, chat)
- Business services layer
- AI Service (Azure OpenAI integration)
- Database layer (SQLite)
- Socket.IO handler for real-time messaging
- Configuration management

### Frontend Components (React UI)

For detailed frontend component architecture, see:

**[midwest-7-ui/docs/c3-component.md](https://github.com/slalomsunil/midwest-7-ui/blob/main/docs/c3-component.md)**

Key frontend components:
- App component (root)
- LoginPage and HomePage
- Chat components (ChatList, ChatWindow, MessageInput)
- LoggedInUsersPanel
- API Service (backend integration)
- Session Service (authentication state)
- Socket Service (real-time messaging)

## Cross-Repository Component Interactions

### Frontend → Backend Integration

**Authentication Flow**:
- LoginPage (UI) → auth routes (Backend) → User Service → Database

**Message Flow**:
- Chat Components (UI) → API Service → chat routes (Backend) → Chat Service → AI Service → Database

**Real-Time Updates**:
- Socket Service (UI) ↔ Socket.IO Handler (Backend) ↔ Chat Service

### Key Integration Points

1. **API Contracts**: REST endpoints defined in backend routes, consumed by frontend API service
2. **WebSocket Events**: Real-time messaging via Socket.IO between frontend and backend
3. **Session Management**: Backend creates sessions, frontend stores and manages them
4. **Tone Transformation**: Frontend selects tone, backend processes via Azure OpenAI

## Architecture Principles

### Separation of Concerns

**Backend**:
- Routes handle HTTP/WebSocket
- Services contain business logic
- Database layer manages data access

**Frontend**:
- Components handle UI
- Services handle API/WebSocket
- Session service manages auth state

### Component Communication

**Backend**: Layered architecture (Routes → Services → Database)

**Frontend**: Component hierarchy with service injection

**Cross-Layer**: REST API + WebSocket for synchronous and real-time communication

## Related Diagrams

- [C1 - System Context](./c1-system-context.md) - High-level system view
- [C2 - Container Diagram](./c2-container.md) - Container architecture
- [C4 - Code Diagram](./c4-code.md) - Code-level patterns
- [Main Architecture](./architecture.md) - Complete architecture overview
