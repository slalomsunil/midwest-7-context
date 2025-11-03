# Technology Guide

## React Development Standards (UI Repository)

### Component Patterns
```javascript
// Functional components with hooks (preferred)
import React, { useState, useEffect } from 'react';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <LoadingSpinner />;
  if (!user) return <NotFound />;
  
  return (
    <div className="user-profile">
      <h1>{user.username}</h1>
      {/* Component content */}
    </div>
  );
};
```

### State Management Patterns
```javascript
// React Context for global state
const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = async (credentials) => {
    const response = await authAPI.login(credentials);
    setUser(response.user);
    setIsAuthenticated(true);
    localStorage.setItem('token', response.token);
  };
  
  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook for using auth context
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### API Integration Patterns
```javascript
// API client with consistent error handling
class APIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.token = localStorage.getItem('token');
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers,
      },
      ...options,
    };
    
    try {
      const response = await fetch(url, config);
      const data = await response.json();
      
      if (!response.ok) {
        throw new APIError(data.error);
      }
      
      return data;
    } catch (error) {
      if (error instanceof APIError) throw error;
      throw new APIError({ message: 'Network error occurred' });
    }
  }
  
  // Specific API methods
  async getPosts() {
    return this.request('/posts');
  }
  
  async createPost(postData) {
    return this.request('/posts', {
      method: 'POST',
      body: JSON.stringify(postData),
    });
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
// models/Database.js - In-memory database singleton
class InMemoryDatabase {
  constructor() {
    this.users = new Map();
    this.posts = new Map();
    this.comments = new Map();
    this.follows = new Map();
    
    // Auto-incrementing IDs
    this.nextUserId = 1;
    this.nextPostId = 1;
    this.nextCommentId = 1;
  }
  
  // User operations
  createUser(userData) {
    const user = {
      id: this.nextUserId++,
      ...userData,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    };
    
    this.users.set(user.id, user);
    return user;
  }
  
  findUserById(id) {
    return this.users.get(parseInt(id));
  }
  
  findUserByEmail(email) {
    return Array.from(this.users.values()).find(user => user.email === email);
  }
  
  // Post operations
  createPost(postData) {
    const post = {
      id: this.nextPostId++,
      ...postData,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    };
    
    this.posts.set(post.id, post);
    return post;
  }
  
  findPostsByUserId(userId) {
    return Array.from(this.posts.values()).filter(post => post.userId === userId);
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

### RESTful Endpoint Patterns
```
GET    /api/posts           # Get all posts
POST   /api/posts           # Create new post
GET    /api/posts/:id       # Get specific post
PUT    /api/posts/:id       # Update specific post
DELETE /api/posts/:id       # Delete specific post

GET    /api/users/:id       # Get user profile
PUT    /api/users/:id       # Update user profile
GET    /api/users/:id/posts # Get user's posts

POST   /api/auth/login      # User login
POST   /api/auth/register   # User registration
POST   /api/auth/logout     # User logout
GET    /api/auth/verify     # Verify token
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