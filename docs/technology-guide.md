# Technology Guide

## React Development Standards (UI Repository)

### Chat Component Patterns
```javascript
// WhatsApp-style chat interface components
import React, { useState, useEffect, useRef } from 'react';

const ChatInterface = ({ conversationId, currentUser }) => {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const [selectedTone, setSelectedTone] = useState('normal');
  const [loading, setLoading] = useState(true);
  const messagesEndRef = useRef(null);
  
  useEffect(() => {
    fetchMessages(conversationId).then(setMessages).finally(() => setLoading(false));
    // WebSocket connection for real-time messages
    const ws = connectToChat(conversationId, (message) => {
      setMessages(prev => [...prev, message]);
    });
    
    return () => ws.disconnect();
  }, [conversationId]);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(scrollToBottom, [messages]);

  if (loading) return <ChatSkeleton />;
  
  return (
    <div className="chat-interface whatsapp-style">
      <div className="messages-container">
        {messages.map(message => (
          <MessageBubble 
            key={message.id} 
            message={message} 
            isOwn={message.senderId === currentUser.id}
            showToneIndicator={message.transformedTone}
          />
        ))}
        <div ref={messagesEndRef} />
      </div>
      <MessageInput 
        value={newMessage}
        onChange={setNewMessage}
        selectedTone={selectedTone}
        onToneChange={setSelectedTone}
        onSend={handleSendMessage}
      />
    </div>
  );
};
```

### Chat State Management Patterns
```javascript
// Chat Context for global chat state
const ChatContext = createContext();

export const ChatProvider = ({ children }) => {
  const [currentUser, setCurrentUser] = useState(null);
  const [conversations, setConversations] = useState([]);
  const [activeConversation, setActiveConversation] = useState(null);
  const [isConnected, setIsConnected] = useState(false);
  
  const loginWithUsername = async (username) => {
    const response = await chatAPI.loginOrCreate(username);
    setCurrentUser(response.user);
    localStorage.setItem('username', username);
    return response.user;
  };
  
  const sendMessage = async (conversationId, content, tone = 'normal') => {
    const message = await chatAPI.sendMessage({
      conversationId,
      content,
      tone,
      senderId: currentUser.id
    });
    
    // Optimistic update
    setConversations(prev => prev.map(conv => 
      conv.id === conversationId 
        ? { ...conv, messages: [...conv.messages, message] }
        : conv
    ));
    
    return message;
  };
  
  return (
    <ChatContext.Provider value={{ 
      currentUser, 
      conversations, 
      activeConversation,
      isConnected,
      loginWithUsername,
      sendMessage,
      setActiveConversation
    }}>
      {children}
    </ChatContext.Provider>
  );
};

// Custom hook for using chat context
export const useChat = () => {
  const context = useContext(ChatContext);
  if (!context) {
    throw new Error('useChat must be used within ChatProvider');
  }
  return context;
};
```

### Chat API Integration Patterns
```javascript
// Chat API client with WebSocket support
class ChatAPIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.username = localStorage.getItem('username');
    this.ws = null;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.username && { 'X-Username': this.username }),
        ...options.headers,
      },
      ...options,
    };
    
    try {
      const response = await fetch(url, config);
      const data = await response.json();
      
      if (!response.ok) {
        throw new ChatAPIError(data.error);
      }
      
      return data;
    } catch (error) {
      if (error instanceof ChatAPIError) throw error;
      throw new ChatAPIError({ message: 'Chat service unavailable' });
    }
  }
  
  // Chat-specific API methods
  async loginOrCreate(username) {
    return this.request('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ username }),
    });
  }
  
  async getConversations() {
    return this.request('/conversations');
  }
  
  async sendMessage(messageData) {
    return this.request('/messages', {
      method: 'POST',
      body: JSON.stringify(messageData),
    });
  }
  
  async transformMessageTone(messageId, tone) {
    return this.request(`/messages/${messageId}/transform`, {
      method: 'PUT',
      body: JSON.stringify({ tone }),
    });
  }
  
  // WebSocket connection for real-time chat
  connectToChat(conversationId, onMessage) {
    const wsUrl = `ws://localhost:3001/chat/${conversationId}`;
    this.ws = new WebSocket(wsUrl);
    
    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      onMessage(message);
    };
    
    return {
      disconnect: () => this.ws?.close(),
      send: (message) => this.ws?.send(JSON.stringify(message))
    };
  }
}
```

### Form Handling Best Practices
```javascript
// Custom hook for form management
const useForm = (initialValues, validationRules) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: null }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    Object.keys(validationRules).forEach(key => {
      const rule = validationRules[key];
      if (rule.required && !values[key]) {
        newErrors[key] = `${key} is required`;
      }
    });
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async (onSubmit) => {
    if (!validate()) return;
    
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } catch (error) {
      // Handle submission errors
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return { values, errors, isSubmitting, handleChange, handleSubmit };
};
```

## Node.js Development Standards (Service Repository)

### Express Server Setup
```javascript
// app.js - Main application setup
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.UI_URL || 'http://localhost:3000',
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: {
      message: 'Internal server error',
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

module.exports = app;
```

### Controller Pattern
```javascript
// controllers/postsController.js
const postsService = require('../services/postsService');

const postsController = {
  async getAllPosts(req, res) {
    try {
      const { page = 1, limit = 10 } = req.query;
      const posts = await postsService.getPosts({ page, limit });
      
      res.json({
        success: true,
        data: posts,
        pagination: {
          page: parseInt(page),
          limit: parseInt(limit),
          total: posts.length
        }
      });
    } catch (error) {
      res.status(500).json({
        error: {
          message: 'Failed to retrieve posts',
          details: error.message
        }
      });
    }
  },
  
  async createPost(req, res) {
    try {
      const postData = req.body;
      const userId = req.user.id; // From auth middleware
      
      const post = await postsService.createPost({ ...postData, userId });
      
      res.status(201).json({
        success: true,
        data: post
      });
    } catch (error) {
      if (error.name === 'ValidationError') {
        res.status(400).json({
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid post data',
            details: error.details
          }
        });
      } else {
        res.status(500).json({
          error: {
            message: 'Failed to create post'
          }
        });
      }
    }
  }
};

module.exports = postsController;
```

### Authentication Middleware
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const { findUserById } = require('../services/usersService');

const authMiddleware = async (req, res, next) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({
        error: {
          code: 'NO_TOKEN',
          message: 'Access denied. No token provided.'
        }
      });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await findUserById(decoded.userId);
    
    if (!user) {
      return res.status(401).json({
        error: {
          code: 'INVALID_TOKEN',
          message: 'Invalid token.'
        }
      });
    }
    
    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({
      error: {
        code: 'INVALID_TOKEN',
        message: 'Invalid token.'
      }
    });
  }
};

module.exports = authMiddleware;
```

## In-Memory Database Patterns

### Data Storage Implementation
```javascript
// models/ChatDatabase.js - In-memory chat database singleton
class InMemoryChatDatabase {
  constructor() {
    this.users = new Map();
    this.conversations = new Map();
    this.messages = new Map();
    this.toneTransformations = new Map();
    
    // Auto-incrementing IDs
    this.nextUserId = 1;
    this.nextConversationId = 1;
    this.nextMessageId = 1;
    
    // Available tone options
    this.availableTones = ['normal', 'funny', 'playful', 'serious', 'formal', 'casual'];
  }
  
  // User operations (username-only authentication)
  createOrFindUser(username) {
    const existingUser = this.findUserByUsername(username);
    if (existingUser) return existingUser;
    
    const user = {
      id: this.nextUserId++,
      username,
      displayName: username,
      profileImage: null,
      defaultTone: 'normal',
      createdAt: new Date().toISOString(),
      lastSeen: new Date().toISOString()
    };
    
    this.users.set(user.id, user);
    return user;
  }
  
  findUserById(id) {
    return this.users.get(parseInt(id));
  }
  
  findUserByUsername(username) {
    return Array.from(this.users.values()).find(user => user.username === username);
  }
  
  getAllUsers() {
    return Array.from(this.users.values());
  }
  
  // Conversation operations
  createConversation(participantIds) {
    const conversation = {
      id: this.nextConversationId++,
      participants: participantIds,
      createdAt: new Date().toISOString(),
      lastMessageAt: new Date().toISOString(),
      lastMessage: null
    };
    
    this.conversations.set(conversation.id, conversation);
    return conversation;
  }
  
  findConversationsByUserId(userId) {
    return Array.from(this.conversations.values()).filter(conv => 
      conv.participants.includes(userId)
    );
  }
  
  // Message operations
  createMessage(messageData) {
    const message = {
      id: this.nextMessageId++,
      conversationId: messageData.conversationId,
      senderId: messageData.senderId,
      originalContent: messageData.content,
      transformedContent: null,
      selectedTone: messageData.tone || 'normal',
      deliveredAt: null,
      readAt: null,
      createdAt: new Date().toISOString()
    };
    
    this.messages.set(message.id, message);
    
    // Update conversation last message
    const conversation = this.conversations.get(messageData.conversationId);
    if (conversation) {
      conversation.lastMessage = message;
      conversation.lastMessageAt = message.createdAt;
    }
    
    return message;
  }
  
  findMessagesByConversationId(conversationId) {
    return Array.from(this.messages.values())
      .filter(msg => msg.conversationId === conversationId)
      .sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
  }
  
  // Tone transformation operations
  transformMessageTone(messageId, tone) {
    const message = this.messages.get(parseInt(messageId));
    if (!message) return null;
    
    // Simulate tone transformation (in real app, this would use AI/ML service)
    const transformedContent = this.applyToneTransformation(message.originalContent, tone);
    
    message.transformedContent = transformedContent;
    message.selectedTone = tone;
    
    return message;
  }
  
  applyToneTransformation(content, tone) {
    // Mock tone transformation - in real implementation, use AI service
    const toneMap = {
      'funny': `😄 ${content} (but funnier!)`,
      'playful': `😉 ${content} ~playful vibes~`,
      'serious': `${content}.`, // More formal punctuation
      'formal': `I would like to convey that ${content.toLowerCase()}.`,
      'casual': `hey! ${content.toLowerCase()} 😊`
    };
    
    return toneMap[tone] || content;
  }
  
  getAllPosts() {
    return Array.from(this.posts.values()).sort((a, b) => 
      new Date(b.createdAt) - new Date(a.createdAt)
    );
  }
  
  deletePost(id) {
    return this.posts.delete(parseInt(id));
  }
}

// Export singleton instance
module.exports = new InMemoryDatabase();
```

### Service Layer Pattern
```javascript
// services/postsService.js
const database = require('../models/Database');

const postsService = {
  async createPost(postData) {
    // Validation
    if (!postData.content || postData.content.trim().length === 0) {
      throw new ValidationError('Post content is required');
    }
    
    if (postData.content.length > 1000) {
      throw new ValidationError('Post content must be less than 1000 characters');
    }
    
    // Create post
    const post = database.createPost({
      userId: postData.userId,
      content: postData.content.trim(),
      imageUrl: postData.imageUrl || null
    });
    
    // Get user info for response
    const user = database.findUserById(postData.userId);
    
    return {
      ...post,
      author: {
        id: user.id,
        username: user.username,
        profileImage: user.profileImage
      }
    };
  },
  
  async getPosts({ page = 1, limit = 10 }) {
    const allPosts = database.getAllPosts();
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    
    return allPosts
      .slice(startIndex, endIndex)
      .map(post => {
        const author = database.findUserById(post.userId);
        return {
          ...post,
          author: {
            id: author.id,
            username: author.username,
            profileImage: author.profileImage
          }
        };
      });
  },
  
  async deletePost(postId, userId) {
    const post = database.posts.get(parseInt(postId));
    
    if (!post) {
      throw new NotFoundError('Post not found');
    }
    
    if (post.userId !== userId) {
      throw new ForbiddenError('You can only delete your own posts');
    }
    
    return database.deletePost(postId);
  }
};

module.exports = postsService;
```

## API Design Guidelines

### Chat API Endpoint Patterns
```
# User Management (Username-only auth)
POST   /api/auth/login         # Login with username (creates profile if new)
GET    /api/users              # Get all users (for user discovery)
GET    /api/users/:username    # Get user profile by username
PUT    /api/users/:username    # Update user profile

# Conversations
GET    /api/conversations      # Get user's conversations
POST   /api/conversations      # Create new conversation
GET    /api/conversations/:id  # Get specific conversation details

# Messages
GET    /api/conversations/:id/messages  # Get messages for conversation
POST   /api/messages                    # Send new message
PUT    /api/messages/:id/tone          # Transform message tone
DELETE /api/messages/:id               # Delete message (if allowed)

# Tone Management
GET    /api/tones                      # Get available tone options
GET    /api/users/:username/preferences # Get user's tone preferences
PUT    /api/users/:username/preferences # Update user's tone preferences

# WebSocket Endpoints
WS     /chat/:conversationId           # Real-time chat connection
```

### Response Format Standards
```javascript
// Success Response
{
  "success": true,
  "data": { /* response data */ },
  "message": "Optional success message"
}

// Error Response
{
  "error": {
    "code": "ERROR_CODE",
    "message": "User-friendly error message",
    "details": ["Specific error details"]
  }
}

// Paginated Response
{
  "success": true,
  "data": [/* array of items */],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10
  }
}
```

### Input Validation Patterns
```javascript
// Validation middleware using Joi
const Joi = require('joi');

const validatePost = (req, res, next) => {
  const schema = Joi.object({
    content: Joi.string().min(1).max(1000).required(),
    imageUrl: Joi.string().uri().optional().allow(null)
  });
  
  const { error } = schema.validate(req.body);
  
  if (error) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid input data',
        details: error.details.map(detail => detail.message)
      }
    });
  }
  
  next();
};
```