# System Architecture

## Architecture Documentation

This document provides an overview of the system architecture. For detailed views at different levels, see:

- **[C1 - System Context](./c1-system-context.md)** - High-level view of the system and external interactions
- **[C2 - Container Diagram](./c2-container.md)** - Major containers and their relationships
- **[C3 - Component Diagram](./c3-component.md)** - Internal components within containers
- **[C4 - Code Diagram](./c4-code.md)** - Detailed code-level patterns

### C4 Model Overview

The C4 model provides a hierarchical way to visualize software architecture:

1. **Level 1 - System Context**: Shows how the system fits into the world (users, external systems)
2. **Level 2 - Containers**: Shows the high-level technical building blocks (apps, databases)
3. **Level 3 - Components**: Shows how containers are made up of components
4. **Level 4 - Code**: Shows how components are implemented (code patterns and structures)

**Navigation**: Start with C1 for high-level understanding, then drill down to C2, C3, and C4 as needed for implementation details.

---

## Repository Structure

This is a **centralized AI instruction hub** that coordinates feature development across two separate repositories:

- **UI Repository**: `slalomsunil/midwest-7-ui` (React-based frontend)
- **Service Repository**: `slalomsunil/midwest-7-service` (Node.js-based backend)
- **Context Repository**: `slalomsunil/midwest-7-context` (This repository - AI orchestration hub)

## Cross-Repository Relationships

### UI Repository (React)
- **Purpose**: Frontend user interface for chat tool with tone modification
- **Framework**: React with modern JavaScript/TypeScript patterns
- **Responsibilities**: Chat interface, tone selection, user profiles, message display, real-time notifications, visual notification indicators
- **Real-time Features**: WebSocket client for live messaging, notification rendering, username blinking indicators, session state management
- **Look & Feel**: WhatsApp-inspired design and user experience
- **Dependencies on Service**: REST API endpoints for messaging, user management, tone transformation, and WebSocket connections for real-time updates

### Service Repository (Node.js)
- **Purpose**: Backend API service for chat tool with AI-powered tone modification
- **Runtime**: Node.js with Express or similar framework
- **Responsibilities**: Message processing, tone transformation, user management, chat persistence, real-time notifications, session management
- **Real-time Features**: WebSocket server for live messaging, notification broadcasting, online user tracking, session cleanup on restart
- **Data Store**: In-memory database for POC purposes
- **Dependencies**: Provides REST APIs consumed by UI, integrates with tone transformation services, manages WebSocket connections

### Context Repository (This Repository)
- **Purpose**: Centralized AI instructions for coordinating cross-layer development
- **Contents**: AI context files, development guidelines, cross-repository coordination patterns
- **Usage**: Referenced by AI agents working in UI and Service repositories

## Cross-Layer Dependencies

### Chat Tool Feature Development Flow
1. **User Story Input**: Define chat tool requirements and user interactions
2. **Spec Generation**: Create detailed specifications for messaging and tone features
3. **API Design**: Determine Service endpoints for messaging, users, and tone transformation
4. **Service Implementation**: Build backend functionality with tone processing
5. **UI Implementation**: Build WhatsApp-style chat interface with tone selection
6. **Integration Testing**: Validate complete messaging workflows with tone modification

### AI Agent Coordination Points
- **Endpoint Discovery**: AI agents should check existing Service APIs before creating new messaging endpoints
- **Data Contract Alignment**: Ensure UI and Service agree on message, user, and tone data structures
- **Error Handling Patterns**: Consistent error handling for messaging failures and tone processing
- **User Management Flow**: Coordinated username-only authentication and profile creation
- **Message Flow**: Seamless message sending, tone transformation, and delivery coordination
- **Real-time Coordination**: WebSocket event synchronization between UI and Service for notifications
- **Session Management**: Coordinated session cleanup patterns for development and deployment scenarios
- **Notification State**: Consistent notification handling across WebSocket connections and UI state

## In-Memory Database Architecture

### Database Purpose
- **POC Simplicity**: In-memory storage for development and demonstration
- **Data Persistence**: Data exists only during service runtime
- **Reset Behavior**: Data resets on service restart (expected POC behavior)

### Data Organization Patterns
- **Entity Models**: User profiles, chat conversations, messages, tone preferences
- **Relationships**: User-to-conversations, conversation-to-messages, user-to-user messaging
- **Message Processing**: Original messages, tone-modified messages, delivery status
- **Indexing Strategy**: Simple in-memory lookups by user ID, conversation ID, and message chronology

### API Design Implications
- **Stateless Operations**: Each API call includes necessary context for messaging and tone processing
- **Data Validation**: Service validates all incoming messages, usernames, and tone selections
- **Response Patterns**: Consistent JSON response structures for messages, users, and conversations
- **Real-time Considerations**: WebSocket support for live chat functionality
- **Tone Processing**: Asynchronous tone transformation with status tracking
- **WebSocket Events**: Standardized event patterns for notifications, session management, and message delivery
- **Session Lifecycle**: Automatic session cleanup on service restart with proper client notification
- **Notification Patterns**: Consistent notification event structure across different message types

## Development Coordination Guidelines

### For AI Agents Working Across Repositories

1. **Always check existing Service APIs** before implementing new endpoints
2. **Coordinate data structures** between UI components and Service responses  
3. **Follow consistent naming conventions** across UI and Service layers
4. **Implement matching error handling patterns** in both repositories
5. **Validate full-stack functionality** when implementing features that span layers

### Repository-Specific Considerations

**When working in UI repository**:
- Check Service repository for available messaging and user management endpoints
- Ensure API calls match Service endpoint signatures for chat functionality
- Handle loading states for message sending and tone processing
- Implement WhatsApp-style UI patterns consistently
- Manage WebSocket connections and implement notification rendering
- Handle session cleanup and reconnection scenarios gracefully
- Implement accessible notification patterns with proper ARIA labels

**When working in Service repository**:
- Consider chat UI requirements when designing messaging APIs
- Provide comprehensive message and user data to minimize additional API calls
- Include proper error messages for messaging failures and tone processing issues
- Design endpoints for real-time chat functionality
- Implement WebSocket event broadcasting for notifications and session management
- Handle session cleanup on restart and deployment scenarios
- Provide consistent notification event structures

**When coordinating across repositories**:
- Update this Context repository with new chat patterns discovered during development
- Document messaging API contracts and tone transformation workflows
- Share reusable chat components and messaging patterns between UI and Service teams
- Maintain consistency in tone options and transformation logic
- Coordinate WebSocket event naming and data structures between UI and Service
- Document notification patterns and session management approaches
- Ensure real-time features work consistently across development and deployment environments