# Challenge 1: OpenAI API Integration

## 🎯 Objective
Build a Kotlin REST API that integrates with OpenAI's API to provide AI-powered text generation capabilities.

## 📖 Learning Goals
- Integrate external APIs in Kotlin applications
- Handle API authentication and secrets management
- Implement async/await patterns for external API calls
- Parse and transform JSON responses
- Handle rate limiting and error scenarios

## 🔧 Requirements

### 1. Setup OpenAI Client
Create a Kotlin service that can communicate with OpenAI's API:
- Use OkHttp or Ktor client for HTTP requests
- Implement proper authentication using API keys
- Store API keys securely (environment variables or configuration)

```kotlin
// Example structure
class OpenAIService(private val apiKey: String) {
    private val client = OkHttpClient()
    
    suspend fun generateCompletion(prompt: String): CompletionResponse {
        // TODO: Implement
    }
}
```

### 2. Create REST Endpoints
Implement the following endpoints:

**POST /api/completions**
```json
Request:
{
  "prompt": "Explain Kotlin coroutines in simple terms",
  "maxTokens": 100,
  "temperature": 0.7
}

Response:
{
  "text": "Generated completion text...",
  "usage": {
    "promptTokens": 10,
    "completionTokens": 90,
    "totalTokens": 100
  }
}
```

**POST /api/chat**
```json
Request:
{
  "messages": [
    {"role": "user", "content": "What is Kotlin?"}
  ],
  "model": "gpt-3.5-turbo"
}

Response:
{
  "message": "Kotlin is a modern programming language...",
  "model": "gpt-3.5-turbo",
  "finishReason": "stop"
}
```

### 3. Error Handling
Implement robust error handling for:
- Invalid API keys (401 Unauthorized)
- Rate limiting (429 Too Many Requests)
- Network failures
- Invalid request parameters
- OpenAI API errors

### 4. Request Validation
Validate incoming requests:
- `prompt` or `messages` must not be empty
- `maxTokens` should be between 1 and 4000
- `temperature` should be between 0.0 and 2.0
- Model names should be from allowed list

## 🎁 Bonus Tasks

1. **Streaming Responses**: Implement Server-Sent Events (SSE) for streaming completions
2. **Caching**: Add Redis/in-memory caching for identical prompts
3. **Rate Limiting**: Implement client-side rate limiting to prevent API quota exhaustion
4. **Retry Logic**: Add exponential backoff for failed requests
5. **Metrics**: Track API usage, response times, and error rates
6. **Multiple Providers**: Abstract the AI provider to support multiple LLM APIs (OpenAI, Anthropic, etc.)

## 📦 Suggested Dependencies

```kotlin
// build.gradle.kts
dependencies {
    // Web Framework
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-netty:2.3.0")
    
    // HTTP Client
    implementation("io.ktor:ktor-client-core:2.3.0")
    implementation("io.ktor:ktor-client-cio:2.3.0")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.0")
    
    // JSON Serialization
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.0")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.0")
    
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.0")
    
    // Testing
    testImplementation("io.ktor:ktor-server-test-host:2.3.0")
    testImplementation("io.mockk:mockk:1.13.5")
}
```

## 🧪 Testing Requirements

Write tests for:
1. Successful API calls to OpenAI
2. Request validation (invalid parameters)
3. Error handling for various failure scenarios
4. Mock OpenAI responses to avoid actual API calls in tests
5. Timeout scenarios

## 📚 Resources

- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [Ktor Client Documentation](https://ktor.io/docs/client.html)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)

## ✅ Acceptance Criteria

- [ ] OpenAI client successfully makes API calls
- [ ] All required endpoints are implemented
- [ ] Request validation returns appropriate error messages
- [ ] Error handling covers all specified scenarios
- [ ] API keys are not hardcoded
- [ ] Unit tests achieve >80% code coverage
- [ ] API can be tested with Postman or curl
- [ ] README included with setup and usage instructions

## 💭 Reflection Questions

After completing this challenge, consider:
1. How would you handle different API versions?
2. What strategies would you use to reduce API costs?
3. How would you implement prompt templates?
4. What security concerns exist when exposing LLM APIs?
