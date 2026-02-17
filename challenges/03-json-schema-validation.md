# Challenge 3: JSON Schema Validation

## 🎯 Objective
Implement comprehensive JSON schema validation for API request and response payloads in a Kotlin REST API.

## 📖 Learning Goals
- Understand JSON Schema specification
- Implement request payload validation
- Create reusable validation middleware
- Handle validation errors gracefully
- Write custom validators for complex rules

## 🔧 Requirements

### 1. Setup JSON Schema Validation

Add schema validation capabilities to your Kotlin application:
- Use a JSON Schema validation library (e.g., `networknt/json-schema-validator`, `everit-org/json-schema`)
- Create a validation middleware or interceptor
- Define schemas for all request payloads

### 2. Create User Management API with Validation

Build a user management API with strict validation:

**POST /api/users** - Create User
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecureP@ss123",
  "firstName": "John",
  "lastName": "Doe",
  "age": 30,
  "phoneNumber": "+1-555-0123",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "country": "USA"
  },
  "preferences": {
    "newsletter": true,
    "notifications": {
      "email": true,
      "sms": false,
      "push": true
    }
  },
  "roles": ["user", "editor"]
}
```

### 3. Define JSON Schemas

**User Creation Schema:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["username", "email", "password"],
  "properties": {
    "username": {
      "type": "string",
      "minLength": 3,
      "maxLength": 30,
      "pattern": "^[a-zA-Z0-9_]+$",
      "description": "Username must be 3-30 alphanumeric characters or underscores"
    },
    "email": {
      "type": "string",
      "format": "email",
      "maxLength": 100
    },
    "password": {
      "type": "string",
      "minLength": 8,
      "maxLength": 128,
      "pattern": "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
      "description": "Password must contain at least one uppercase, lowercase, digit, and special character"
    },
    "firstName": {
      "type": "string",
      "minLength": 1,
      "maxLength": 50,
      "pattern": "^[a-zA-Z\\s'-]+$"
    },
    "lastName": {
      "type": "string",
      "minLength": 1,
      "maxLength": 50,
      "pattern": "^[a-zA-Z\\s'-]+$"
    },
    "age": {
      "type": "integer",
      "minimum": 13,
      "maximum": 150,
      "description": "User must be at least 13 years old"
    },
    "phoneNumber": {
      "type": "string",
      "pattern": "^\\+?[1-9]\\d{1,14}$",
      "description": "Valid E.164 phone number format"
    },
    "address": {
      "type": "object",
      "required": ["street", "city", "country"],
      "properties": {
        "street": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200
        },
        "city": {
          "type": "string",
          "minLength": 1,
          "maxLength": 100
        },
        "state": {
          "type": "string",
          "maxLength": 50
        },
        "zipCode": {
          "type": "string",
          "pattern": "^\\d{5}(-\\d{4})?$|^[A-Z]\\d[A-Z] ?\\d[A-Z]\\d$"
        },
        "country": {
          "type": "string",
          "minLength": 2,
          "maxLength": 2,
          "pattern": "^[A-Z]{2}$",
          "description": "ISO 3166-1 alpha-2 country code"
        }
      },
      "additionalProperties": false
    },
    "preferences": {
      "type": "object",
      "properties": {
        "newsletter": {
          "type": "boolean",
          "default": false
        },
        "notifications": {
          "type": "object",
          "properties": {
            "email": {
              "type": "boolean",
              "default": true
            },
            "sms": {
              "type": "boolean",
              "default": false
            },
            "push": {
              "type": "boolean",
              "default": false
            }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },
    "roles": {
      "type": "array",
      "minItems": 1,
      "maxItems": 10,
      "uniqueItems": true,
      "items": {
        "type": "string",
        "enum": ["user", "admin", "editor", "viewer", "moderator"]
      }
    }
  },
  "additionalProperties": false
}
```

### 4. Implement Validation in Kotlin

Create a validation service:

```kotlin
import com.networknt.schema.JsonSchemaFactory
import com.networknt.schema.SpecVersion
import com.networknt.schema.ValidationMessage
import com.fasterxml.jackson.databind.JsonNode
import com.fasterxml.jackson.databind.ObjectMapper

class JsonSchemaValidator {
    private val objectMapper = ObjectMapper()
    private val schemaFactory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7)
    
    fun validateRequest(jsonPayload: String, schemaPath: String): ValidationResult {
        try {
            val schemaNode = loadSchema(schemaPath)
            val jsonNode = objectMapper.readTree(jsonPayload)
            val schema = schemaFactory.getSchema(schemaNode)
            
            val errors = schema.validate(jsonNode)
            
            return if (errors.isEmpty()) {
                ValidationResult.Success
            } else {
                ValidationResult.Failure(errors.map { it.toValidationError() })
            }
        } catch (e: Exception) {
            return ValidationResult.Failure(listOf(
                ValidationError("", "Invalid JSON format: ${e.message}")
            ))
        }
    }
    
    private fun loadSchema(schemaPath: String): JsonNode {
        val schemaContent = javaClass.getResourceAsStream(schemaPath)
            ?.bufferedReader()?.use { it.readText() }
            ?: throw IllegalArgumentException("Schema not found: $schemaPath")
        
        return objectMapper.readTree(schemaContent)
    }
}

sealed class ValidationResult {
    object Success : ValidationResult()
    data class Failure(val errors: List<ValidationError>) : ValidationResult()
}

data class ValidationError(
    val path: String,
    val message: String
)

fun ValidationMessage.toValidationError() = ValidationError(
    path = this.path ?: "",
    message = this.message
)
```

### 5. Create Validation Middleware

```kotlin
// For Ktor
fun Application.configureValidation() {
    install(StatusPages) {
        exception<ValidationException> { call, cause ->
            call.respond(
                HttpStatusCode.BadRequest,
                ErrorResponse(
                    status = 400,
                    message = "Validation failed",
                    errors = cause.errors
                )
            )
        }
    }
}

fun Route.validateRequest(schemaPath: String, handler: suspend PipelineContext<Unit, ApplicationCall>.() -> Unit) {
    intercept(ApplicationCallPipeline.Call) {
        val body = call.receiveText()
        val validator = JsonSchemaValidator()
        
        when (val result = validator.validateRequest(body, schemaPath)) {
            is ValidationResult.Success -> {
                // Continue to handler
                handler()
            }
            is ValidationResult.Failure -> {
                call.respond(
                    HttpStatusCode.BadRequest,
                    ErrorResponse(
                        status = 400,
                        message = "Validation failed",
                        errors = result.errors
                    )
                )
                finish()
            }
        }
    }
}
```

### 6. Use Validation in Routes

```kotlin
routing {
    post("/api/users") {
        validateRequest("/schemas/user-creation.json") {
            val userRequest = call.receive<UserCreationRequest>()
            val createdUser = userService.createUser(userRequest)
            call.respond(HttpStatusCode.Created, createdUser)
        }
    }
    
    put("/api/users/{id}") {
        validateRequest("/schemas/user-update.json") {
            val id = call.parameters["id"] ?: throw IllegalArgumentException("Invalid ID")
            val updateRequest = call.receive<UserUpdateRequest>()
            val updatedUser = userService.updateUser(id, updateRequest)
            call.respond(HttpStatusCode.OK, updatedUser)
        }
    }
}
```

## 🎁 Bonus Tasks

1. **Custom Validators**: Create custom validators for business logic (e.g., username uniqueness)
2. **Async Validation**: Implement async validators for database checks
3. **Response Validation**: Validate outgoing responses against schemas
4. **Schema Generation**: Generate JSON schemas from Kotlin data classes
5. **Conditional Schemas**: Implement `if/then/else` schema validation
6. **Schema Registry**: Create a centralized schema registry service
7. **Validation Reports**: Generate detailed validation reports with suggestions
8. **Performance**: Optimize validation by caching compiled schemas

## 📦 Suggested Dependencies

```kotlin
// build.gradle.kts
dependencies {
    // JSON Schema Validation
    implementation("com.networknt:json-schema-validator:1.0.86")
    
    // Alternative: Everit JSON Schema
    // implementation("com.github.erosb:everit-json-schema:1.14.2")
    
    // JSON Processing
    implementation("com.fasterxml.jackson.core:jackson-databind:2.15.0")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin:2.15.0")
    
    // Web Framework
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-content-negotiation:2.3.0")
    
    // Testing
    testImplementation("io.kotest:kotest-runner-junit5:5.6.0")
    testImplementation("io.kotest:kotest-assertions-json:5.6.0")
}
```

## 🧪 Testing Requirements

Write tests for:

1. **Valid Payloads**: Test with valid JSON that passes all constraints
2. **Missing Required Fields**: Test validation of required fields
3. **Type Mismatches**: Send wrong types (string instead of number)
4. **Pattern Violations**: Test regex patterns (email, phone, username)
5. **Range Violations**: Test min/max constraints
6. **Nested Object Validation**: Test nested address and preferences
7. **Array Validation**: Test roles array (uniqueness, allowed values)
8. **Additional Properties**: Test that extra fields are rejected
9. **Multiple Errors**: Test payload with multiple validation errors
10. **Edge Cases**: Empty strings, null values, boundary values

```kotlin
class UserValidationTest {
    private val validator = JsonSchemaValidator()
    
    @Test
    fun `should accept valid user payload`() {
        val validPayload = """
            {
              "username": "john_doe",
              "email": "john@example.com",
              "password": "SecureP@ss123",
              "roles": ["user"]
            }
        """.trimIndent()
        
        val result = validator.validateRequest(validPayload, "/schemas/user-creation.json")
        
        result shouldBe ValidationResult.Success
    }
    
    @Test
    fun `should reject invalid email format`() {
        val invalidPayload = """
            {
              "username": "john_doe",
              "email": "invalid-email",
              "password": "SecureP@ss123",
              "roles": ["user"]
            }
        """.trimIndent()
        
        val result = validator.validateRequest(invalidPayload, "/schemas/user-creation.json")
        
        result.shouldBeInstanceOf<ValidationResult.Failure>()
        result.errors.any { it.path.contains("email") } shouldBe true
    }
}
```

## 📚 Resources

- [JSON Schema Official Documentation](https://json-schema.org/)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/)
- [networknt JSON Schema Validator](https://github.com/networknt/json-schema-validator)
- [JSON Schema Validator](https://www.jsonschemavalidator.net/)
- [JSON Schema Lint](https://jsonschemalint.com/)

## ✅ Acceptance Criteria

- [ ] All endpoints validate request payloads against JSON schemas
- [ ] Validation errors return clear, actionable error messages
- [ ] Schemas cover all required fields and constraints
- [ ] Nested objects and arrays are validated correctly
- [ ] Invalid requests return 400 Bad Request with error details
- [ ] Valid requests are processed successfully
- [ ] Schemas are stored in organized resource files
- [ ] All validation scenarios have test coverage
- [ ] Error responses include field paths and violation details
- [ ] Performance is acceptable for typical payloads

## 💭 Reflection Questions

After completing this challenge, consider:
1. How would you handle schema versioning?
2. What's the trade-off between strict validation and flexibility?
3. How can you balance validation performance with thoroughness?
4. Should validation be synchronous or asynchronous?
5. How would you validate conditional fields (if X then Y is required)?
