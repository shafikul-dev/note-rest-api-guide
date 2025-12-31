# Express.js RESTful API Guide

> 📚 This repository now has a Docsify-powered site — open `docs/` locally with Docsify or enable GitHub Pages (serve the `docs/` folder) to publish the site.

A comprehensive guide to designing and implementing RESTful APIs following industry best practices.

## Table of Contents

1. [What is REST?](#what-is-rest)
2. [REST Principles](#rest-principles)
3. [URL Design & Naming Conventions](#url-design--naming-conventions)
4. [HTTP Methods](#http-methods)
5. [HTTP Status Codes](#http-status-codes)
6. [Request & Response Formats](#request--response-formats)
7. [Error Handling](#error-handling)
8. [API Versioning](#api-versioning)
9. [Security Best Practices](#security-best-practices)
10. [Validation](#validation)
11. [Pagination & Filtering](#pagination--filtering)
12. [Documentation](#documentation)

---

## What is REST?

**REST (Representational State Transfer)** is an architectural style for designing networked applications. A RESTful API uses HTTP requests to access and use data, following these principles:

- **Stateless**: Each request contains all information needed to process it
- **Resource-based**: URLs represent resources, not actions
- **Standard HTTP methods**: GET, POST, PUT, PATCH, DELETE
- **Uniform interface**: Consistent way of interacting with resources

---

## REST Principles

### 1. **Client-Server Architecture**
- Separation of concerns between client and server
- Server handles data storage and business logic
- Client handles user interface and user experience

### 2. **Stateless**
- Each request must contain all information needed to process it
- Server doesn't store client context between requests
- Session state is kept entirely on the client

### 3. **Cacheable**
- Responses should be cacheable when possible
- Use appropriate cache headers (`Cache-Control`, `ETag`)

### 4. **Uniform Interface**
- Consistent resource identification (URLs)
- Standard HTTP methods
- Self-descriptive messages
- Hypermedia as the engine of application state (HATEOAS)

### 5. **Layered System**
- Architecture can be composed of hierarchical layers
- Each layer only knows about the immediate layer

### 6. **Code on Demand (Optional)**
- Server can send executable code to client (e.g., JavaScript)

---

## URL Design & Naming Conventions

### ✅ DO: Use Nouns, Not Verbs

```
✅ Good:
GET    /users
POST   /users
GET    /users/123
PUT    /users/123
DELETE /users/123

❌ Bad:
GET    /getUsers
POST   /createUser
GET    /getUserById/123
POST   /updateUser/123
DELETE /deleteUser/123
```

### ✅ DO: Use Plural Nouns for Collections

```
✅ Good:
/users
/products
/orders

❌ Bad:
/user
/product
/order
```

### ✅ DO: Use Hierarchical Structure for Related Resources

```
✅ Good:
GET    /users/123/posts          (Get all posts by user 123)
GET    /users/123/posts/456      (Get post 456 by user 123)
POST   /users/123/posts          (Create a post for user 123)

❌ Bad:
GET    /posts?userId=123
GET    /userPosts/123
```

### ✅ DO: Use Lowercase and Hyphens

```
✅ Good:
/user-profiles
/api-keys
/order-items

❌ Bad:
/UserProfiles
/api_keys
/orderItems
```

### ✅ DO: Keep URLs Simple and Intuitive

```
✅ Good:
GET /users/123
GET /users/123/payments
GET /users/123/payments?filter=recent

❌ Bad:
GET /u/123
GET /usr/123/pay
GET /api/v1/user/123/payment/list?type=recent
```

---

## HTTP Methods

### GET - Retrieve Resources

**Purpose**: Fetch data (should not modify server state)

```javascript
// Get all users
router.get('/users', (req, res) => {
    // Return list of users
});

// Get single user
router.get('/users/:id', (req, res) => {
    const userId = req.params.id;
    // Return user with id
});

// Get nested resource
router.get('/users/:id/payments', (req, res) => {
    const userId = req.params.id;
    // Return payments for user
});
```

**Characteristics**:
- Idempotent (same request = same result)
- Safe (no side effects)
- Can be cached
- Should not have request body

### POST - Create Resources

**Purpose**: Create new resources

```javascript
router.post('/users', (req, res) => {
    const { name, age, address, location } = req.body;
    // Validate input
    // Create user
    // Return 201 Created with new resource
    res.status(201).json({
        message: "User created successfully",
        user: newUser
    });
});
```

**Characteristics**:
- Not idempotent (multiple calls create multiple resources)
- Not safe (modifies server state)
- Returns `201 Created` on success
- Request body contains resource data

### PUT - Replace Resources

**Purpose**: Replace entire resource (full update)

```javascript
router.put('/users/:id', (req, res) => {
    const userId = req.params.id;
    const { name, age, address, location } = req.body;
    // Replace entire user resource
    res.status(200).json({ user: updatedUser });
});
```

**Characteristics**:
- Idempotent (same request = same result)
- Requires all fields (full resource replacement)
- Returns `200 OK` or `204 No Content`

### PATCH - Partial Update

**Purpose**: Update part of a resource

```javascript
router.patch('/users/:id', (req, res) => {
    const userId = req.params.id;
    const { age } = req.body; // Only update age
    // Partial update
    res.status(200).json({ user: updatedUser });
});
```

**Characteristics**:
- Idempotent
- Only updates provided fields
- Returns `200 OK` or `204 No Content`

### DELETE - Remove Resources

**Purpose**: Delete a resource

```javascript
router.delete('/users/:id', (req, res) => {
    const userId = req.params.id;
    // Delete user
    res.status(204).send(); // No Content
});
```

**Characteristics**:
- Idempotent
- Returns `204 No Content` or `200 OK` with deleted resource info

---

## HTTP Status Codes

### 2xx Success

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE, PUT, PATCH (no body) |

### 4xx Client Errors

| Code | Meaning | Usage |
|------|---------|-------|
| 400 | Bad Request | Invalid request syntax, validation errors |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource conflict (e.g., duplicate email) |
| 422 | Unprocessable Entity | Valid syntax but semantic errors |

### 5xx Server Errors

| Code | Meaning | Usage |
|------|---------|-------|
| 500 | Internal Server Error | Generic server error |
| 502 | Bad Gateway | Invalid response from upstream |
| 503 | Service Unavailable | Server temporarily unavailable |

### Example Usage

```javascript
// Success
res.status(200).json({ data: users });
res.status(201).json({ message: "Created", user: newUser });
res.status(204).send();

// Client Errors
res.status(400).json({ error: "Validation failed" });
res.status(401).json({ error: "Unauthorized" });
res.status(404).json({ error: "User not found" });
res.status(409).json({ error: "Email already exists" });

// Server Errors
res.status(500).json({ error: "Internal server error" });
```

---

## Request & Response Formats

### Request Headers

Always include appropriate headers:

```javascript
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

### Request Body (POST/PUT/PATCH)

```json
{
  "name": "John Doe",
  "age": 30,
  "address": "123 Main St",
  "location": "New York"
}
```

### Response Format

**Success Response**:
```json
{
  "message": "User created successfully",
  "user": {
    "id": 123,
    "name": "John Doe",
    "age": 30,
    "address": "123 Main St",
    "location": "New York"
  }
}
```

**Error Response**:
```json
{
  "error": "Validation failed",
  "details": [
    {
      "field": "name",
      "message": "Name must be at least 3 characters"
    }
  ]
}
```

### Best Practices

1. **Consistent Structure**: Always use the same response format
2. **Include Metadata**: Add timestamps, pagination info when needed
3. **Use Envelopes for Lists**: Wrap arrays in objects

```javascript
// ✅ Good
{
  "data": [user1, user2, user3],
  "count": 3,
  "page": 1,
  "total": 100
}

// ❌ Bad
[user1, user2, user3]
```

---

## Error Handling

### Standardized Error Response

```javascript
// Error middleware
const errorHandler = (err, req, res, next) => {
    const statusCode = err.statusCode || 500;
    res.status(statusCode).json({
        error: {
            message: err.message || 'Internal Server Error',
            code: err.code || 'INTERNAL_ERROR',
            details: err.details || null,
            timestamp: new Date().toISOString()
        }
    });
};
```

### Validation Errors

```javascript
router.post('/users', (req, res) => {
    const { error } = userSchema.validate(req.body);
    
    if (error) {
        return res.status(400).json({
            error: "Validation failed",
            details: error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message
            }))
        });
    }
    // Process request...
});
```

### Not Found Errors

```javascript
router.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        return res.status(404).json({
            error: "User not found",
            message: `User with ID ${req.params.id} does not exist`
        });
    }
    
    res.status(200).json({ user });
});
```

---

## API Versioning

### URL Versioning (Recommended)

```
/api/v1/users
/api/v2/users
```

```javascript
// app.js
app.use('/api/v1', userRoutes);
app.use('/api/v2', userRoutesV2);
```

### Header Versioning

```
Accept: application/vnd.api+json;version=2
```

### Best Practices

1. **Version from the start**: Even if v1, include version in URL
2. **Maintain backward compatibility**: Keep old versions working
3. **Document breaking changes**: Clearly communicate changes
4. **Deprecation strategy**: Give notice before removing versions

---

## Security Best Practices

### 1. Authentication & Authorization

```javascript
// JWT Authentication middleware
const authenticate = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: "Unauthorized" });
    }
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        res.status(401).json({ error: "Invalid token" });
    }
};

// Use in routes
router.get('/users/:id', authenticate, (req, res) => {
    // Protected route
});
```

### 2. Input Validation

Always validate and sanitize input:

```javascript
const userSchema = Joi.object({
    name: Joi.string().min(3).max(100).required(),
    age: Joi.number().positive().required(),
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required()
});
```

### 3. Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);
```

### 4. HTTPS Only

Always use HTTPS in production:

```javascript
// Force HTTPS
if (process.env.NODE_ENV === 'production') {
    app.use((req, res, next) => {
        if (req.header('x-forwarded-proto') !== 'https') {
            res.redirect(`https://${req.header('host')}${req.url}`);
        } else {
            next();
        }
    });
}
```

### 5. CORS Configuration

```javascript
const cors = require('cors');

app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
    credentials: true
}));
```

---

## Validation

### Using Joi (Recommended)

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
    name: Joi.string()
        .min(3)
        .max(100)
        .required()
        .messages({
            'string.min': 'Name must be at least 3 characters',
            'string.max': 'Name cannot exceed 100 characters',
            'any.required': 'Name is required'
        }),
    age: Joi.number()
        .positive()
        .integer()
        .required(),
    email: Joi.string()
        .email()
        .required(),
    address: Joi.string()
        .min(5)
        .required()
});

router.post('/users', (req, res) => {
    const { error, value } = userSchema.validate(req.body, {
        abortEarly: false // Return all errors, not just first
    });
    
    if (error) {
        return res.status(400).json({
            error: "Validation failed",
            details: error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message
            }))
        });
    }
    
    // Process validated data
    const user = new User(value);
    // ...
});
```

### Manual Validation (Not Recommended)

```javascript
// ❌ Avoid manual validation - use Joi instead
if (!name || !age) {
    return res.status(400).json({ error: "Missing fields" });
}
```

---

## Pagination & Filtering

### Query Parameters

```
GET /users?page=1&limit=10&sort=name&order=asc&filter=active
```

### Implementation

```javascript
router.get('/users', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const sort = req.query.sort || 'createdAt';
    const order = req.query.order || 'desc';
    const filter = req.query.filter;
    
    const skip = (page - 1) * limit;
    
    // Build query
    let query = {};
    if (filter) {
        query.status = filter;
    }
    
    // Execute query
    const users = await User.find(query)
        .sort({ [sort]: order })
        .skip(skip)
        .limit(limit);
    
    const total = await User.countDocuments(query);
    
    res.status(200).json({
        data: users,
        pagination: {
            page,
            limit,
            total,
            pages: Math.ceil(total / limit)
        }
    });
});
```

### Response Format

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "pages": 10
  }
}
```

---

## Documentation

### API Documentation Tools

1. **Swagger/OpenAPI**: Industry standard
2. **Postman Collections**: Shareable API collections
3. **README**: Simple documentation

### Example API Documentation

```markdown
## GET /users/:id

Get a user by ID.

**Parameters:**
- `id` (path, required): User ID

**Query Parameters:**
- `filter` (optional): Filter type (active, inactive)

**Response:**
- 200 OK: User object
- 404 Not Found: User doesn't exist

**Example:**
```http
GET /users/123?filter=active
```

**Response:**
```json
{
  "user": {
    "id": 123,
    "name": "John Doe",
    "age": 30
  }
}
```
```

---

## Complete Example

### Route File Structure

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const Joi = require('joi');

// Validation Schema
const userSchema = Joi.object({
    name: Joi.string().min(3).max(100).required(),
    age: Joi.number().positive().required(),
    address: Joi.string().min(5).required(),
    location: Joi.string().required()
});

// GET /users - Get all users
router.get('/users', async (req, res) => {
    try {
        const users = await User.find();
        res.status(200).json({ data: users });
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

// GET /users/:id - Get user by ID
router.get('/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                error: "User not found",
                message: `User with ID ${req.params.id} does not exist`
            });
        }
        
        res.status(200).json({ user });
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

// POST /users - Create user
router.post('/users', async (req, res) => {
    // Validate input
    const { error, value } = userSchema.validate(req.body, {
        abortEarly: false
    });
    
    if (error) {
        return res.status(400).json({
            error: "Validation failed",
            details: error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message
            }))
        });
    }
    
    try {
        const user = new User(value);
        await user.save();
        
        res.status(201).json({
            message: "User created successfully",
            user
        });
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

// PUT /users/:id - Update user (full)
router.put('/users/:id', async (req, res) => {
    const { error, value } = userSchema.validate(req.body);
    
    if (error) {
        return res.status(400).json({
            error: "Validation failed",
            details: error.details
        });
    }
    
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            value,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({ error: "User not found" });
        }
        
        res.status(200).json({ user });
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

// PATCH /users/:id - Update user (partial)
router.patch('/users/:id', async (req, res) => {
    const updateSchema = Joi.object({
        name: Joi.string().min(3).max(100),
        age: Joi.number().positive(),
        address: Joi.string().min(5),
        location: Joi.string()
    });
    
    const { error, value } = updateSchema.validate(req.body);
    
    if (error) {
        return res.status(400).json({
            error: "Validation failed",
            details: error.details
        });
    }
    
    try {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            value,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({ error: "User not found" });
        }
        
        res.status(200).json({ user });
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

// DELETE /users/:id - Delete user
router.delete('/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        
        if (!user) {
            return res.status(404).json({ error: "User not found" });
        }
        
        res.status(204).send();
    } catch (error) {
        res.status(500).json({ error: "Internal server error" });
    }
});

module.exports = router;
```

---

## Checklist for RESTful API Design

- [ ] Use nouns for URLs, not verbs
- [ ] Use plural nouns for collections
- [ ] Use appropriate HTTP methods (GET, POST, PUT, PATCH, DELETE)
- [ ] Return appropriate HTTP status codes
- [ ] Use consistent response format
- [ ] Implement proper error handling
- [ ] Validate all input data
- [ ] Implement authentication/authorization
- [ ] Add rate limiting
- [ ] Use HTTPS in production
- [ ] Implement API versioning
- [ ] Add pagination for list endpoints
- [ ] Document your API
- [ ] Use proper HTTP headers
- [ ] Handle CORS properly
- [ ] Implement logging and monitoring

---

## Resources

- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [Joi Validation](https://joi.dev/api/)
- [Express.js Documentation](https://expressjs.com/)
- [OpenAPI Specification](https://swagger.io/specification/)

---

## Deploying the docs (GitHub Pages) ✅

**Option A — Quick (no CI):**
1. Go to your repository Settings → **Pages**.
2. Under **Source**, choose **main** branch and folder **/docs**.
3. Save. The site will be available at `https://<your-username>.github.io/<repo>/` (may take a minute).

**Option B — Automatic (recommended):**
- I added a GitHub Action `.github/workflows/deploy-docs.yml` that publishes the `docs/` folder to the `gh-pages` branch on every push to `main` using `peaceiris/actions-gh-pages` (no static build required for Docsify).
- After the first Action run, update Pages settings to serve from **gh-pages** branch (root) if needed.

> Tip: If you prefer immediate manual control, use Option A; if you want automatic publishing on pushes to `main`, use Option B.

---

## License

ISC
