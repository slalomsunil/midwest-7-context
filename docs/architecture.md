# System Architecture

## Repository Structure

This is a **centralized AI instruction hub** that coordinates feature development across two separate repositories:

- **UI Repository**: `slalomsunil/midwest-7-ui` (React-based frontend)
- **Service Repository**: `slalomsunil/midwest-7-service` (Node.js-based backend)
- **Context Repository**: `slalomsunil/midwest-7-context` (This repository - AI orchestration hub)

## Cross-Repository Relationships

### UI Repository (React)
- **Purpose**: Frontend user interface for chat tool with tone modification
- **Framework**: React with modern JavaScript/TypeScript patterns
- **Responsibilities**: Chat interface, tone selection, user profiles, message display
- **Look & Feel**: WhatsApp-inspired design and user experience
- **Dependencies on Service**: REST API endpoints for messaging, user management, and tone transformation

### Service Repository (Node.js)
- **Purpose**: Backend API service for chat tool with AI-powered tone modification
- **Runtime**: Node.js with Express or similar framework
- **Responsibilities**: Message processing, tone transformation, user management, chat persistence
- **Data Store**: In-memory database for POC purposes
- **Dependencies**: Provides REST APIs consumed by UI, integrates with tone transformation services

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

**When working in Service repository**:
- Consider chat UI requirements when designing messaging APIs
- Provide comprehensive message and user data to minimize additional API calls
- Include proper error messages for messaging failures and tone processing issues
- Design endpoints for real-time chat functionality

**When coordinating across repositories**:
- Update this Context repository with new chat patterns discovered during development
- Document messaging API contracts and tone transformation workflows
- Share reusable chat components and messaging patterns between UI and Service teams
- Maintain consistency in tone options and transformation logic