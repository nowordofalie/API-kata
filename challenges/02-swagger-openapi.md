# Challenge 2: Swagger/OpenAPI Documentation

## 🎯 Objective
Create a Kotlin REST API with comprehensive OpenAPI 3.0 documentation using Swagger, implementing contract-first development practices.

## 📖 Learning Goals
- Understand OpenAPI specification structure
- Generate interactive API documentation
- Implement contract-first vs code-first approaches
- Use Swagger annotations for automatic documentation
- Validate API implementations against OpenAPI specs

## 🔧 Requirements

### 1. Setup Swagger/OpenAPI
Configure Swagger in your Kotlin application:
- Add Swagger UI for interactive documentation
- Set up OpenAPI 3.0 specification
- Configure Swagger UI to be accessible at `/swagger-ui`
- Make OpenAPI spec available at `/api-docs` or `/openapi.json`

### 2. Create a Product API with Full Documentation

Implement a REST API for product management with complete OpenAPI documentation:

**API Endpoints:**

**GET /api/products**
- Query parameters: `category`, `minPrice`, `maxPrice`, `page`, `size`
- Returns paginated list of products
- Document all possible response codes

**GET /api/products/{id}**
- Path parameter: product ID
- Returns single product or 404

**POST /api/products**
- Request body: Product creation data
- Returns created product with 201 status
- Validate required fields

**PUT /api/products/{id}**
- Path parameter: product ID
- Request body: Product update data
- Returns updated product or 404

**DELETE /api/products/{id}**
- Path parameter: product ID
- Returns 204 on success or 404

### 3. OpenAPI Specification Example

Create a detailed OpenAPI spec (you can use annotations or YAML):

```yaml
openapi: 3.0.0
info:
  title: Product Management API
  description: API for managing product catalog
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: http://localhost:8080
    description: Development server
  - url: https://api.example.com
    description: Production server

paths:
  /api/products:
    get:
      summary: List all products
      description: Retrieve a paginated list of products with optional filters
      operationId: getProducts
      tags:
        - Products
      parameters:
        - name: category
          in: query
          description: Filter by product category
          required: false
          schema:
            type: string
            enum: [electronics, clothing, food, books]
        - name: minPrice
          in: query
          description: Minimum price filter
          required: false
          schema:
            type: number
            format: double
            minimum: 0
        - name: maxPrice
          in: query
          description: Maximum price filter
          required: false
          schema:
            type: number
            format: double
        - name: page
          in: query
          description: Page number (0-indexed)
          required: false
          schema:
            type: integer
            default: 0
            minimum: 0
        - name: size
          in: query
          description: Page size
          required: false
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
                $ref: '#/components/schemas/ProductPage'
        '400':
          description: Invalid request parameters
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    post:
      summary: Create a new product
      description: Add a new product to the catalog
      operationId: createProduct
      tags:
        - Products
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProductInput'
      responses:
        '201':
          description: Product created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '400':
          description: Invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    Product:
      type: object
      required:
        - id
        - name
        - price
        - category
      properties:
        id:
          type: string
          format: uuid
          description: Unique product identifier
          example: "550e8400-e29b-41d4-a716-446655440000"
        name:
          type: string
          description: Product name
          minLength: 1
          maxLength: 200
          example: "Laptop Computer"
        description:
          type: string
          description: Product description
          maxLength: 1000
          example: "High-performance laptop with 16GB RAM"
        price:
          type: number
          format: double
          description: Product price in USD
          minimum: 0
          example: 999.99
        category:
          type: string
          enum: [electronics, clothing, food, books]
          description: Product category
          example: "electronics"
        inStock:
          type: boolean
          description: Whether product is in stock
          default: true
        createdAt:
          type: string
          format: date-time
          description: Creation timestamp
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp
    
    ProductInput:
      type: object
      required:
        - name
        - price
        - category
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 200
        description:
          type: string
          maxLength: 1000
        price:
          type: number
          format: double
          minimum: 0
        category:
          type: string
          enum: [electronics, clothing, food, books]
        inStock:
          type: boolean
          default: true
    
    ProductPage:
      type: object
      properties:
        content:
          type: array
          items:
            $ref: '#/components/schemas/Product'
        page:
          type: integer
        size:
          type: integer
        totalElements:
          type: integer
        totalPages:
          type: integer
    
    Error:
      type: object
      required:
        - message
        - status
      properties:
        message:
          type: string
          description: Error message
        status:
          type: integer
          description: HTTP status code
        timestamp:
          type: string
          format: date-time
        path:
          type: string
          description: Request path that caused the error
```

### 4. Code-First Approach with Annotations

If using annotations (e.g., Springdoc or Ktor OpenAPI), document your endpoints:

```kotlin
@Tag(name = "Products", description = "Product management operations")
@RestController
@RequestMapping("/api/products")
class ProductController(private val productService: ProductService) {
    
    @Operation(
        summary = "Get all products",
        description = "Retrieve a paginated list of products with optional filters"
    )
    @ApiResponses(value = [
        ApiResponse(responseCode = "200", description = "Successful operation"),
        ApiResponse(responseCode = "400", description = "Invalid parameters")
    ])
    @GetMapping
    fun getProducts(
        @Parameter(description = "Filter by category")
        @RequestParam(required = false) category: String?,
        
        @Parameter(description = "Minimum price")
        @RequestParam(required = false) minPrice: Double?,
        
        @Parameter(description = "Maximum price")
        @RequestParam(required = false) maxPrice: Double?,
        
        @Parameter(description = "Page number")
        @RequestParam(defaultValue = "0") page: Int,
        
        @Parameter(description = "Page size")
        @RequestParam(defaultValue = "20") size: Int
    ): ProductPage {
        // Implementation
    }
}
```

## 🎁 Bonus Tasks

1. **API Versioning**: Document multiple API versions (v1, v2) in the same spec
2. **Authentication**: Add security schemes (API Key, JWT, OAuth2)
3. **Examples**: Include request/response examples for all endpoints
4. **Code Generation**: Use OpenAPI Generator to generate client SDKs
5. **Validation**: Implement automatic request/response validation against spec
6. **Custom UI**: Customize Swagger UI branding and theme
7. **Export**: Generate PDF documentation from OpenAPI spec

## 📦 Suggested Dependencies

```kotlin
// build.gradle.kts

// For Spring Boot
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.1.0")
}

// For Ktor
dependencies {
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-openapi:2.3.0")
    implementation("io.ktor:ktor-server-swagger:2.3.0")
}
```

## 🧪 Testing Requirements

1. Verify Swagger UI is accessible at `/swagger-ui`
2. Verify OpenAPI spec is valid JSON/YAML
3. Test that all endpoints match the specification
4. Validate response schemas match documented schemas
5. Test all documented error scenarios

## 📚 Resources

- [OpenAPI Specification 3.0](https://swagger.io/specification/)
- [Swagger Editor](https://editor.swagger.io/)
- [Springdoc OpenAPI](https://springdoc.org/)
- [Ktor OpenAPI Plugin](https://ktor.io/docs/openapi.html)
- [OpenAPI Generator](https://openapi-generator.tech/)

## ✅ Acceptance Criteria

- [ ] Swagger UI is accessible and functional
- [ ] All CRUD endpoints are documented
- [ ] Request/response schemas are defined
- [ ] Query parameters are documented with types and constraints
- [ ] Error responses are documented
- [ ] API can be tested directly from Swagger UI
- [ ] OpenAPI spec validates with no errors
- [ ] Examples are provided for complex schemas
- [ ] Security requirements are documented (if applicable)

## 💭 Reflection Questions

After completing this challenge, consider:
1. What are the benefits of contract-first vs code-first development?
2. How would you keep documentation in sync with code changes?
3. What's the best way to version APIs?
4. How can OpenAPI specs facilitate API testing?
