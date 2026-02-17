# Challenge 5: Advanced Combined Challenge - AI-Powered Content API

## 🎯 Objective
Build a complete AI-powered content management API that combines all previous challenges: OpenAI integration, Swagger documentation, JSON schema validation, and advanced filtering.

## 📖 Learning Goals
- Integrate multiple API patterns in a cohesive system
- Design complex, production-ready APIs
- Implement end-to-end feature development
- Apply best practices across the stack
- Handle real-world complexity and edge cases

## 🔧 Requirements

Build a **Content Generation & Management Platform** with the following features:

### 1. System Overview

Create an API that:
1. Generates content using OpenAI
2. Stores and manages generated content
3. Provides advanced search and filtering
4. Validates all requests with JSON schemas
5. Documents everything with Swagger/OpenAPI

### 2. Core Features

#### Feature 1: AI Content Generation

**POST /api/content/generate**

Generate content using OpenAI with customizable parameters:

```json
Request:
{
  "prompt": "Write a blog post about Kotlin coroutines",
  "contentType": "blog_post",
  "tone": "professional",
  "length": "medium",
  "keywords": ["kotlin", "coroutines", "async"],
  "targetAudience": "developers",
  "temperature": 0.7,
  "includeMetadata": true
}

Response:
{
  "id": "cnt-abc123",
  "generatedContent": "Kotlin coroutines are...",
  "metadata": {
    "title": "Understanding Kotlin Coroutines",
    "summary": "A comprehensive guide...",
    "suggestedTags": ["kotlin", "coroutines", "programming"],
    "readTimeMinutes": 8,
    "sentiment": "neutral"
  },
  "usage": {
    "promptTokens": 25,
    "completionTokens": 500,
    "totalTokens": 525,
    "estimatedCost": 0.0105
  },
  "generatedAt": "2024-01-15T10:30:00Z"
}
```

**JSON Schema for Content Generation:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["prompt", "contentType"],
  "properties": {
    "prompt": {
      "type": "string",
      "minLength": 10,
      "maxLength": 2000,
      "description": "The prompt for content generation"
    },
    "contentType": {
      "type": "string",
      "enum": ["blog_post", "social_media", "email", "description", "summary", "article"],
      "description": "Type of content to generate"
    },
    "tone": {
      "type": "string",
      "enum": ["professional", "casual", "friendly", "formal", "humorous", "educational"],
      "default": "professional"
    },
    "length": {
      "type": "string",
      "enum": ["short", "medium", "long"],
      "default": "medium",
      "description": "Desired content length"
    },
    "keywords": {
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
    "targetAudience": {
      "type": "string",
      "maxLength": 100
    },
    "temperature": {
      "type": "number",
      "minimum": 0,
      "maximum": 2,
      "default": 0.7
    },
    "includeMetadata": {
      "type": "boolean",
      "default": true
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
- tone: Filter by tone
- tags: Filter by tags (comma-separated)
- minLength: Minimum word count
- maxLength: Maximum word count
- generatedAfter: Date filter
- generatedBefore: Date filter
- search: Full-text search in content
- sortBy: title, generatedAt, wordCount, etc.
- sortOrder: asc, desc
- page: Page number
- size: Page size
```

**GET /api/content/{id}** - Get specific content

**PUT /api/content/{id}** - Update content
```json
{
  "title": "Updated Title",
  "content": "Updated content...",
  "tags": ["kotlin", "updated"],
  "published": true
}
```

**DELETE /api/content/{id}** - Delete content

**POST /api/content/{id}/regenerate** - Regenerate content with new parameters

#### Feature 3: Content Analysis

**POST /api/content/analyze**

Analyze existing content using AI:
```json
Request:
{
  "content": "Your text content here...",
  "analysisTypes": ["sentiment", "keywords", "summary", "readability"]
}

Response:
{
  "sentiment": {
    "score": 0.7,
    "label": "positive"
  },
  "keywords": ["kotlin", "programming", "async"],
  "summary": "This content discusses...",
  "readability": {
    "score": 75,
    "grade": "college",
    "readTimeMinutes": 5
  },
  "wordCount": 850,
  "characterCount": 4523
}
```

#### Feature 4: Batch Operations

**POST /api/content/batch/generate**

Generate multiple pieces of content in one request:
```json
{
  "requests": [
    {
      "prompt": "First prompt...",
      "contentType": "blog_post"
    },
    {
      "prompt": "Second prompt...",
      "contentType": "social_media"
    }
  ],
  "parallel": true
}
```

### 3. Data Models

```kotlin
data class Content(
    val id: String,
    val title: String,
    val content: String,
    val contentType: ContentType,
    val tone: Tone,
    val tags: List<String>,
    val metadata: ContentMetadata,
    val generationParams: GenerationParams,
    val usage: TokenUsage,
    val published: Boolean,
    val generatedAt: Instant,
    val updatedAt: Instant,
    val userId: String
)

enum class ContentType {
    BLOG_POST, SOCIAL_MEDIA, EMAIL, DESCRIPTION, SUMMARY, ARTICLE
}

enum class Tone {
    PROFESSIONAL, CASUAL, FRIENDLY, FORMAL, HUMOROUS, EDUCATIONAL
}

data class ContentMetadata(
    val wordCount: Int,
    val characterCount: Int,
    val readTimeMinutes: Int,
    val sentiment: Sentiment?,
    val suggestedTags: List<String>
)

data class Sentiment(
    val score: Double,
    val label: String
)

data class GenerationParams(
    val prompt: String,
    val temperature: Double,
    val keywords: List<String>,
    val targetAudience: String?
)

data class TokenUsage(
    val promptTokens: Int,
    val completionTokens: Int,
    val totalTokens: Int,
    val estimatedCost: Double
)
```

### 4. Implementation Requirements

#### OpenAI Integration
- Create a service layer for OpenAI API calls
- Implement proper error handling and retries
- Add rate limiting and quota management
- Cache similar prompts to reduce API costs
- Track token usage and costs

#### JSON Schema Validation
- Create schemas for all POST/PUT endpoints
- Validate content generation requests
- Validate content update requests
- Validate batch operation requests
- Return detailed validation errors

#### Swagger Documentation
- Document all endpoints with OpenAPI 3.0
- Include request/response examples
- Document all query parameters
- Add authentication requirements
- Include error response schemas
- Make Swagger UI available at `/swagger-ui`

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
  title: AI-Powered Content API
  version: 1.0.0
  description: |
    A comprehensive API for generating and managing AI-powered content.
    
    Features:
    - AI content generation using OpenAI
    - Advanced content filtering and search
    - JSON schema validation
    - Batch operations
    - Content analysis

paths:
  /api/content/generate:
    post:
      summary: Generate new content using AI
      tags:
        - Content Generation
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ContentGenerationRequest'
            examples:
              blogPost:
                summary: Generate a blog post
                value:
                  prompt: "Write about Kotlin coroutines"
                  contentType: "blog_post"
                  tone: "professional"
                  length: "medium"
      responses:
        '201':
          description: Content generated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GeneratedContent'
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

1. **WebSocket Support**: Real-time content generation progress
2. **Content Versioning**: Track content changes over time
3. **Content Templates**: Reusable generation templates
4. **Multi-language Support**: Generate content in multiple languages
5. **Content Scheduling**: Schedule content generation
6. **AI Model Selection**: Support multiple AI providers
7. **Content Collaboration**: Multiple users editing same content
8. **Export Formats**: Export content in various formats (PDF, Markdown, HTML)
9. **SEO Optimization**: Analyze and optimize for SEO
10. **Content Recommendations**: AI-powered content suggestions

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
    
    // HTTP Client for OpenAI
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
   - OpenAI service methods
   - Validation logic
   - Filter builders
   - Content analysis

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

## 📚 Resources

All resources from previous challenges, plus:
- [Building Production-Ready APIs](https://www.nginx.com/blog/building-microservices-inter-process-communication/)
- [API Security Best Practices](https://owasp.org/www-project-api-security/)
- [RESTful API Design Guide](https://restfulapi.net/)

## ✅ Acceptance Criteria

- [ ] All features are implemented and working
- [ ] OpenAI integration is functional with error handling
- [ ] All requests are validated with JSON schemas
- [ ] Complete Swagger documentation is available
- [ ] Advanced filtering works with all combinations
- [ ] Authentication and authorization are implemented
- [ ] Rate limiting is in place
- [ ] Comprehensive test coverage (>80%)
- [ ] Performance is acceptable under load
- [ ] API is documented with usage examples
- [ ] Error handling covers all scenarios
- [ ] Code follows Kotlin best practices
- [ ] Database migrations are managed properly

## 💭 Final Reflection

After completing this challenge, reflect on:
1. What design patterns worked best for this complex system?
2. How would you scale this to millions of users?
3. What security considerations are most important?
4. How would you monitor and maintain this in production?
5. What would you do differently if starting from scratch?
6. How would you handle API versioning as features evolve?
7. What's the best approach to cost optimization with AI APIs?

## 🎉 Congratulations!

If you've completed this challenge, you've built a sophisticated, production-ready API that combines:
- AI/ML integration
- Schema validation
- Comprehensive documentation
- Advanced query capabilities
- Industry best practices

This represents a significant achievement in API development with Kotlin!
