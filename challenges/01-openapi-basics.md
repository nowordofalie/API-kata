# Challenge 1: OpenAPI Specification Basics

## 🎯 Objective
Build a Kotlin REST API and document it using OpenAPI Specification (OAS) 3.0+, following contract-first development principles and generating client code from the specification.

## 📖 Learning Goals
- Understand OpenAPI Specification (OAS) fundamentals
- Design APIs using contract-first development
- Create comprehensive OpenAPI documents
- Use OpenAPI tools (Swagger Editor, Swagger UI, code generators)
- Validate API implementations against OpenAPI specs
- Generate client/server code from OpenAPI specifications

## 🔧 Requirements

### 1. Create an OpenAPI Specification
Design a simple **Book Management API** with an OpenAPI 3.0 specification:

**Create `openapi.yaml` with the following structure:**

```yaml
openapi: 3.0.3
info:
  title: Book Management API
  description: A simple API for managing a collection of books
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: http://localhost:8080/api/v1
    description: Development server

paths:
  /books:
    get:
      summary: List all books
      operationId: listBooks
      tags:
        - Books
      parameters:
        - name: limit
          in: query
          description: Maximum number of books to return
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Book'
```

### 2. Define Complete API Operations
Your OpenAPI spec should include:

**Required Endpoints:**
- `GET /books` - List books with pagination
- `GET /books/{id}` - Get a specific book
- `POST /books` - Create a new book
- `PUT /books/{id}` - Update a book
- `DELETE /books/{id}` - Delete a book

**Book Schema:**
```yaml
components:
  schemas:
    Book:
      type: object
      required:
        - title
        - author
        - isbn
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
        title:
          type: string
          minLength: 1
          maxLength: 200
        author:
          type: string
          minLength: 1
          maxLength: 100
        isbn:
          type: string
          pattern: '^[0-9]{13}$'
        publishedYear:
          type: integer
          minimum: 1000
          maximum: 2100
        genre:
          type: string
          enum: [fiction, non-fiction, science, history, biography]
        price:
          type: number
          format: double
          minimum: 0
```

### 3. Implement the API in Kotlin
Create a Kotlin implementation that matches your OpenAPI specification:

```kotlin
// Data class matching OpenAPI schema
@Serializable
data class Book(
    val id: String? = null,
    val title: String,
    val author: String,
    val isbn: String,
    val publishedYear: Int? = null,
    val genre: String? = null,
    val price: Double? = null
)

// Ktor routing matching OpenAPI paths
fun Application.configureRouting() {
    routing {
        route("/api/v1/books") {
            get {
                // Implementation
            }
            get("/{id}") {
                // Implementation
            }
            post {
                // Implementation
            }
            put("/{id}") {
                // Implementation
            }
            delete("/{id}") {
                // Implementation
            }
        }
    }
}
```

### 4. Integrate Swagger UI
Set up Swagger UI to visualize and test your API:

```kotlin
// build.gradle.kts
dependencies {
    implementation("io.ktor:ktor-server-openapi:2.3.0")
    implementation("io.ktor:ktor-server-swagger:2.3.0")
}

// Application.kt
fun Application.module() {
    install(Plugins.SwaggerUI) {
        swaggerUrl = "/swagger-ui"
        forwardRoot = true
    }
    install(Plugins.OpenAPI) {
        path = "/openapi.json"
    }
}
```

### 5. Validate Implementation Against Spec
Use OpenAPI validation tools to ensure your implementation matches the specification:
- Request/response validation
- Schema validation
- Parameter validation
- Status code validation

## 🎁 Bonus Tasks

1. **Code Generation**: Use OpenAPI Generator to create Kotlin client code from your spec
2. **API Versioning**: Add support for multiple API versions (v1, v2)
3. **Security Schemas**: Add authentication (API Key, OAuth2, JWT) to your spec
4. **Response Examples**: Include comprehensive examples for all responses
5. **Error Schemas**: Define reusable error response schemas
6. **Mock Server**: Generate a mock server from your OpenAPI spec using Prism
7. **Documentation**: Generate beautiful documentation using ReDoc or Stoplight

## 📦 Suggested Dependencies

```kotlin
// build.gradle.kts
dependencies {
    // Web Framework
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-netty:2.3.0")
    
    // OpenAPI/Swagger Support
    implementation("io.ktor:ktor-server-openapi:2.3.0")
    implementation("io.ktor:ktor-server-swagger:2.3.0")
    
    // Content Negotiation
    implementation("io.ktor:ktor-server-content-negotiation:2.3.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.0")
    
    // Serialization
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.0")
    
    // OpenAPI Validation
    implementation("com.atlassian.oai:swagger-request-validator-core:2.35.0")
    
    // Testing
    testImplementation("io.ktor:ktor-server-test-host:2.3.0")
    testImplementation("io.kotest:kotest-runner-junit5:5.6.0")
}
```

## 🧪 Testing Requirements

Write tests for:
1. OpenAPI specification validation (valid YAML/JSON)
2. All endpoints return responses matching the spec
3. Request validation rejects invalid inputs
4. Schema validation for all data models
5. Error responses match defined error schemas
6. Contract testing using Pact or similar tools

## 📚 Resources

- [OpenAPI Specification 3.0](https://swagger.io/specification/)
- [Swagger Editor](https://editor.swagger.io/) - Design and validate OpenAPI specs
- [Swagger UI](https://swagger.io/tools/swagger-ui/) - Interactive API documentation
- [OpenAPI Generator](https://openapi-generator.tech/) - Generate code from specs
- [Stoplight Studio](https://stoplight.io/studio) - Modern OpenAPI editor
- [Prism](https://stoplight.io/open-source/prism) - Mock server from OpenAPI specs
- [OpenAPI Tools](https://openapi.tools/) - Comprehensive tooling directory

## ✅ Acceptance Criteria

- [ ] Complete OpenAPI 3.0+ specification is created
- [ ] All CRUD endpoints are documented in the spec
- [ ] Kotlin implementation matches the OpenAPI spec
- [ ] Swagger UI is accessible and functional
- [ ] Request/response validation is working
- [ ] Schema definitions include proper constraints
- [ ] Error responses are properly documented
- [ ] API can be tested through Swagger UI
- [ ] Documentation is clear and comprehensive
- [ ] Code follows OpenAPI best practices

## 💭 Reflection Questions

After completing this challenge, consider:
1. What are the benefits of contract-first API development?
2. How does OpenAPI improve API documentation and client generation?
3. When should you use OpenAPI 3.0 vs 3.1?
4. How can OpenAPI specs facilitate API testing and validation?
5. What role does OpenAPI play in microservices architectures?
6. How would you handle breaking changes in your API specification?
