# Challenge 5: Advanced Combined Challenge - Content Management API

## 🎯 Objective
Build a complete content management API that combines all previous challenges: comprehensive OpenAPI documentation, JSON schema validation, and advanced filtering. Focus on best practices for API design, documentation, and implementation.

## 📖 Learning Goals
- Integrate multiple API patterns in a cohesive system
- Design complex, production-ready APIs
- Implement end-to-end feature development
- Apply best practices across the stack
- Handle real-world complexity and edge cases

## 🔧 Requirements

Build a **Content Management & Publishing Platform** with the following features:

### 1. System Overview

Create an API that:
1. Manages articles, blog posts, and other content
2. Stores and retrieves content with rich metadata
3. Provides advanced search and filtering
4. Validates all requests with JSON schemas
5. Documents everything comprehensively with OpenAPI specification

### 2. Core Features

#### Feature 1: Content Creation and Management

**POST /api/content/create**

Create new content with rich metadata and validation:

```json
Request:
{
  "title": "Understanding Kotlin Coroutines",
  "body": "Kotlin coroutines are a powerful feature...",
  "contentType": "blog_post",
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com"
  },
  "tags": ["kotlin", "coroutines", "async"],
  "category": "programming",
  "metadata": {
    "seoTitle": "Kotlin Coroutines Guide",
    "seoDescription": "A comprehensive guide to Kotlin coroutines",
    "keywords": ["kotlin", "coroutines", "programming"]
  },
  "publishDate": "2024-01-15T10:30:00Z",
  "status": "draft"
}

Response:
{
  "id": "cnt-abc123",
  "title": "Understanding Kotlin Coroutines",
  "body": "Kotlin coroutines are a powerful feature...",
  "contentType": "blog_post",
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com"
  },
  "tags": ["kotlin", "coroutines", "async"],
  "category": "programming",
  "metadata": {
    "seoTitle": "Kotlin Coroutines Guide",
    "seoDescription": "A comprehensive guide to Kotlin coroutines",
    "keywords": ["kotlin", "coroutines", "programming"],
    "wordCount": 1250,
    "readTimeMinutes": 6
  },
  "publishDate": "2024-01-15T10:30:00Z",
  "status": "draft",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

**JSON Schema for Content Creation:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["title", "body", "contentType", "author"],
  "properties": {
    "title": {
      "type": "string",
      "minLength": 5,
      "maxLength": 200,
      "description": "The title of the content"
    },
    "body": {
      "type": "string",
      "minLength": 50,
      "maxLength": 50000,
      "description": "The main content body"
    },
    "contentType": {
      "type": "string",
      "enum": ["blog_post", "article", "tutorial", "documentation", "news"],
      "description": "Type of content"
    },
    "author": {
      "type": "object",
      "required": ["name", "email"],
      "properties": {
        "name": {
          "type": "string",
          "minLength": 1,
          "maxLength": 100
        },
        "email": {
          "type": "string",
          "format": "email"
        }
      }
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string",
        "minLength": 2,
        "maxLength": 50
      },
      "minItems": 0,
      "maxItems": 10,
      "uniqueItems": true
    },
    "category": {
      "type": "string",
      "maxLength": 50
    },
    "status": {
      "type": "string",
      "enum": ["draft", "published", "archived"],
      "default": "draft"
    }
  }
}
```

#### Feature 2: Content Management

**CRUD Operations for Generated Content:**

**GET /api/content** - List all content with advanced filtering
```
Query Parameters:
- contentType: Filter by type
- category: Filter by category
- tags: Filter by tags (comma-separated)
- author: Filter by author email
- status: Filter by status (draft, published, archived)
- minWordCount: Minimum word count
- maxWordCount: Maximum word count
- createdAfter: Date filter
- createdBefore: Date filter
- search: Full-text search in title and body
- sortBy: title, createdAt, updatedAt, wordCount, etc.
- sortOrder: asc, desc
- page: Page number
- size: Page size
```

**GET /api/content/{id}** - Get specific content

**PUT /api/content/{id}** - Update content
```json
{
  "title": "Updated Title",
  "body": "Updated content...",
  "tags": ["kotlin", "updated"],
  "status": "published"
}
```

**DELETE /api/content/{id}** - Delete content

**POST /api/content/{id}/duplicate** - Duplicate content with new ID

#### Feature 3: Content Statistics and Analytics

**GET /api/content/{id}/stats**

Get statistics for a specific content item:
```json
Response:
{
  "views": 1523,
  "uniqueVisitors": 892,
  "averageReadTime": 4.2,
  "shares": {
    "twitter": 45,
    "linkedin": 23,
    "facebook": 12
  },
  "engagement": {
    "likes": 156,
    "comments": 23,
    "bookmarks": 67
  }
}
```

#### Feature 4: Batch Operations

**POST /api/content/batch/create**

Create multiple pieces of content in one request:
```json
{
  "contents": [
    {
      "title": "First Article",
      "body": "Content for first article...",
      "contentType": "article"
    },
    {
      "title": "Second Article",
      "body": "Content for second article...",
      "contentType": "blog_post"
    }
  ]
}
```

### 3. Data Models

```kotlin
data class Content(
    val id: String,
    val title: String,
    val body: String,
    val contentType: ContentType,
    val author: Author,
    val category: String?,
    val tags: List<String>,
    val metadata: ContentMetadata,
    val status: ContentStatus,
    val publishDate: Instant?,
    val createdAt: Instant,
    val updatedAt: Instant
)

enum class ContentType {
    BLOG_POST, ARTICLE, TUTORIAL, DOCUMENTATION, NEWS
}

enum class ContentStatus {
    DRAFT, PUBLISHED, ARCHIVED
}

data class Author(
    val name: String,
    val email: String,
    val bio: String?
)

data class ContentMetadata(
    val wordCount: Int,
    val characterCount: Int,
    val readTimeMinutes: Int,
    val seoTitle: String?,
    val seoDescription: String?,
    val keywords: List<String>
)
```

### 4. Implementation Requirements

#### OpenAPI Documentation
- Create comprehensive OpenAPI 3.0+ specification
- Document all endpoints with detailed descriptions
- Include request/response examples for all operations
- Define reusable schemas and components
- Document authentication requirements
- Specify all error responses
- Make OpenAPI spec available at `/openapi.json` or `/openapi.yaml`
- Serve interactive documentation via Swagger UI at `/swagger-ui`

#### JSON Schema Validation
- Create schemas for all POST/PUT endpoints
- Validate content creation requests
- Validate content update requests
- Validate batch operation requests
- Return detailed validation errors with field-level feedback

#### Swagger/OpenAPI Best Practices
- Use $ref to avoid duplication
- Define reusable components (schemas, responses, parameters)
- Include comprehensive examples
- Use proper HTTP status codes
- Document rate limiting headers
- Include OpenAPI extensions where appropriate

#### Advanced Filtering
- Implement full-text search
- Support multiple filter combinations
- Add pagination and sorting
- Optimize query performance
- Return pagination metadata

#### Additional Features
- User authentication (JWT or API keys)
- Rate limiting per user
- Audit logging for all operations
- Metrics and monitoring
- Async job processing for long operations

### 5. OpenAPI Documentation Example

```yaml
openapi: 3.0.0
info:
  title: Content Management API
  version: 1.0.0
  description: |
    A comprehensive API for managing content with advanced features.
    
    Features:
    - Content creation and management
    - Advanced content filtering and search
    - JSON schema validation
    - Batch operations
    - Content analytics and statistics

paths:
  /api/content/create:
    post:
      summary: Create new content
      tags:
        - Content Management
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ContentCreationRequest'
            examples:
              blogPost:
                summary: Create a blog post
                value:
                  title: "Understanding Kotlin Coroutines"
                  body: "Kotlin coroutines are..."
                  contentType: "blog_post"
                  author:
                    name: "Jane Developer"
                    email: "jane@example.com"
      responses:
        '201':
          description: Content created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Content'
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationError'
        '401':
          description: Unauthorized
        '429':
          description: Rate limit exceeded

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## 🎁 Bonus Tasks

1. **WebSocket Support**: Real-time content updates and notifications
2. **Content Versioning**: Track content changes over time with full revision history
3. **Content Templates**: Reusable content templates with variable substitution
4. **Multi-language Support**: Content localization and internationalization
5. **Content Scheduling**: Schedule content publication and archival
6. **Media Management**: Handle images, videos, and other media assets
7. **Content Collaboration**: Multiple users editing same content with conflict resolution
8. **Export Formats**: Export content in various formats (PDF, Markdown, HTML, ePub)
9. **SEO Optimization**: Analyze and optimize content for search engines
10. **Content Recommendations**: Related content suggestions based on tags and categories

## 📦 Required Dependencies

```kotlin
// build.gradle.kts
dependencies {
    // Web Framework
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-netty:2.3.0")
    implementation("io.ktor:ktor-server-auth:2.3.0")
    implementation("io.ktor:ktor-server-auth-jwt:2.3.0")
    
    // OpenAPI/Swagger
    implementation("io.ktor:ktor-server-openapi:2.3.0")
    implementation("io.ktor:ktor-server-swagger:2.3.0")
    
    // HTTP Client for external APIs (if needed)
    implementation("io.ktor:ktor-client-core:2.3.0")
    implementation("io.ktor:ktor-client-cio:2.3.0")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.0")
    
    // JSON Schema Validation
    implementation("com.networknt:json-schema-validator:1.0.86")
    
    // Database
    implementation("org.jetbrains.exposed:exposed-core:0.41.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.41.1")
    implementation("org.jetbrains.exposed:exposed-jdbc:0.41.1")
    implementation("com.h2database:h2:2.1.214")
    
    // Caching
    implementation("io.github.reactivecircus.cache4k:cache4k:0.12.0")
    
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.0")
    
    // Serialization
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.0")
    
    // Testing
    testImplementation("io.ktor:ktor-server-test-host:2.3.0")
    testImplementation("io.kotest:kotest-runner-junit5:5.6.0")
    testImplementation("io.mockk:mockk:1.13.5")
}
```

## 🧪 Testing Requirements

Comprehensive test coverage:

1. **Unit Tests**:
   - Content service methods
   - Validation logic
   - Filter builders
   - Content metadata calculation

2. **Integration Tests**:
   - End-to-end API flows
   - Database operations
   - Authentication/authorization

3. **API Tests**:
   - All endpoints with valid inputs
   - All validation scenarios
   - Error handling
   - Pagination and filtering

4. **Performance Tests**:
   - Load testing for batch operations
   - Response time benchmarks
   - Database query optimization

5. **OpenAPI Contract Tests**:
   - Validate implementation matches OpenAPI spec
   - Ensure all documented endpoints exist
   - Verify response schemas match specification

## 📚 Resources

All resources from previous challenges, plus:
- [Building Production-Ready APIs](https://www.nginx.com/blog/building-microservices-inter-process-communication/)
- [API Security Best Practices](https://owasp.org/www-project-api-security/)
- [RESTful API Design Guide](https://restfulapi.net/)

## ✅ Acceptance Criteria

- [ ] All features are implemented and working
- [ ] Content management operations are functional with proper error handling
- [ ] All requests are validated with JSON schemas
- [ ] Complete OpenAPI documentation is available and accurate
- [ ] Swagger UI is accessible and functional
- [ ] Advanced filtering works with all combinations
- [ ] Authentication and authorization are implemented
- [ ] Rate limiting is in place
- [ ] Comprehensive test coverage (>80%)
- [ ] Performance is acceptable under load
- [ ] API is documented with usage examples
- [ ] Error handling covers all scenarios
- [ ] Code follows Kotlin best practices
- [ ] Database migrations are managed properly
- [ ] OpenAPI specification validates successfully

## 💭 Final Reflection

After completing this challenge, reflect on:
1. What design patterns worked best for this complex system?
2. How would you scale this to millions of users?
3. What security considerations are most important?
4. How would you monitor and maintain this in production?
5. What would you do differently if starting from scratch?
6. How would you handle API versioning as features evolve?
7. What's the best approach to maintaining OpenAPI documentation as the API evolves?
8. How do you ensure the implementation stays in sync with the OpenAPI specification?

## 🎉 Congratulations!

If you've completed this challenge, you've built a sophisticated, production-ready API that combines:
- Comprehensive OpenAPI documentation
- Schema validation
- Advanced query capabilities
- Industry best practices

This represents a significant achievement in API development with Kotlin!
