# System Architecture

## Repository Structure

This is a **centralized AI instruction hub** that coordinates feature development across two separate repositories:

- **UI Repository**: `slalomsunil/midwest-7-ui` (React-based frontend)
- **Service Repository**: `slalomsunil/midwest-7-service` (Node.js-based backend)
- **Context Repository**: `slalomsunil/midwest-7-context` (This repository - AI orchestration hub)

## Cross-Repository Relationships

### UI Repository (React)
- **Purpose**: Frontend user interface for social media POC
- **Framework**: React with modern JavaScript/TypeScript patterns
- **Responsibilities**: User interface, client-side logic, API consumption
- **Dependencies on Service**: REST API endpoints for data operations

### Service Repository (Node.js)
- **Purpose**: Backend API service for social media POC
- **Runtime**: Node.js with Express or similar framework
- **Responsibilities**: Business logic, API endpoints, data management
- **Data Store**: In-memory database for POC purposes
- **Dependencies**: Provides REST APIs consumed by UI

### Context Repository (This Repository)
- **Purpose**: Centralized AI instructions for coordinating cross-layer development
- **Contents**: AI context files, development guidelines, cross-repository coordination patterns
- **Usage**: Referenced by AI agents working in UI and Service repositories

## Cross-Layer Dependencies

### Feature Development Flow
1. **Feature Specification**: Define requirements that span UI and Service layers
2. **API Design**: Determine what Service endpoints are needed
3. **Service Implementation**: Build required backend functionality
4. **UI Implementation**: Build frontend to consume Service APIs
5. **Integration Testing**: Validate full-stack feature functionality

### AI Agent Coordination Points
- **Endpoint Discovery**: AI agents should check existing Service APIs before creating new ones
- **Data Contract Alignment**: Ensure UI and Service agree on data structures
- **Error Handling Patterns**: Consistent error handling across layers
- **Authentication Flow**: Coordinated auth implementation across UI and Service

## In-Memory Database Architecture

### Database Purpose
- **POC Simplicity**: In-memory storage for development and demonstration
- **Data Persistence**: Data exists only during service runtime
- **Reset Behavior**: Data resets on service restart (expected POC behavior)

### Data Organization Patterns
- **Entity Models**: User profiles, posts, comments, likes, follows
- **Relationships**: User-to-posts, post-to-comments, user-to-user follows
- **Indexing Strategy**: Simple in-memory lookups by ID and basic queries

### API Design Implications
- **Stateless Operations**: Each API call includes necessary context
- **Data Validation**: Service validates all incoming data
- **Response Patterns**: Consistent JSON response structures across endpoints

## Development Coordination Guidelines

### For AI Agents Working Across Repositories

1. **Always check existing Service APIs** before implementing new endpoints
2. **Coordinate data structures** between UI components and Service responses  
3. **Follow consistent naming conventions** across UI and Service layers
4. **Implement matching error handling patterns** in both repositories
5. **Validate full-stack functionality** when implementing features that span layers

### Repository-Specific Considerations

**When working in UI repository**:
- Check Service repository for available endpoints
- Ensure API calls match Service endpoint signatures
- Handle loading states and error conditions consistently

**When working in Service repository**:
- Consider UI requirements when designing APIs
- Provide comprehensive data in responses to minimize additional API calls
- Include proper error messages that UI can display to users

**When coordinating across repositories**:
- Update this Context repository with new patterns discovered during development
- Document API contracts that emerge during feature development
- Share reusable patterns and components between UI and Service teams