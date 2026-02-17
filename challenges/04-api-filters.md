# Challenge 4: API Filters & Query Parameters

## 🎯 Objective
Implement advanced filtering, searching, sorting, and pagination capabilities for REST APIs in Kotlin.

## 📖 Learning Goals
- Design flexible query parameter systems
- Implement complex filtering logic
- Build dynamic query builders
- Handle pagination efficiently
- Support sorting and ordering
- Create reusable filter specifications

## 🔧 Requirements

### 1. Basic Filtering System

Create an API for a blog/content platform with comprehensive filtering:

**GET /api/articles**

Support the following query parameters:
```
GET /api/articles?
  title=kotlin                    // Text search in title
  &author=john_doe                // Exact match
  &category=technology            // Filter by category
  &tags=kotlin,programming        // Match any of these tags
  &status=published               // Exact status match
  &minReadTime=5                  // Minimum reading time in minutes
  &maxReadTime=15                 // Maximum reading time
  &publishedAfter=2024-01-01      // Published after date
  &publishedBefore=2024-12-31     // Published before date
  &sortBy=publishedAt             // Sort field
  &sortOrder=desc                 // Sort direction (asc/desc)
  &page=0                         // Page number
  &size=20                        // Page size
```

### 2. Data Models

```kotlin
data class Article(
    val id: String,
    val title: String,
    val content: String,
    val author: String,
    val category: String,
    val tags: List<String>,
    val status: ArticleStatus,
    val readTimeMinutes: Int,
    val publishedAt: Instant?,
    val createdAt: Instant,
    val updatedAt: Instant,
    val viewCount: Int,
    val likeCount: Int
)

enum class ArticleStatus {
    DRAFT, PUBLISHED, ARCHIVED
}

data class ArticleFilter(
    val title: String? = null,
    val author: String? = null,
    val category: String? = null,
    val tags: List<String>? = null,
    val status: ArticleStatus? = null,
    val minReadTime: Int? = null,
    val maxReadTime: Int? = null,
    val publishedAfter: Instant? = null,
    val publishedBefore: Instant? = null,
    val sortBy: String = "createdAt",
    val sortOrder: SortOrder = SortOrder.DESC,
    val page: Int = 0,
    val size: Int = 20
)

enum class SortOrder {
    ASC, DESC
}

data class PageResult<T>(
    val content: List<T>,
    val page: Int,
    val size: Int,
    val totalElements: Long,
    val totalPages: Int,
    val hasNext: Boolean,
    val hasPrevious: Boolean
)
```

### 3. Filter Builder Implementation

```kotlin
class ArticleFilterBuilder {
    
    fun buildFilter(params: Parameters): ArticleFilter {
        return ArticleFilter(
            title = params["title"],
            author = params["author"],
            category = params["category"],
            tags = params["tags"]?.split(",")?.map { it.trim() },
            status = params["status"]?.let { ArticleStatus.valueOf(it.uppercase()) },
            minReadTime = params["minReadTime"]?.toIntOrNull(),
            maxReadTime = params["maxReadTime"]?.toIntOrNull(),
            publishedAfter = params["publishedAfter"]?.let { Instant.parse(it) },
            publishedBefore = params["publishedBefore"]?.let { Instant.parse(it) },
            sortBy = params["sortBy"] ?: "createdAt",
            sortOrder = params["sortOrder"]?.let { 
                SortOrder.valueOf(it.uppercase()) 
            } ?: SortOrder.DESC,
            page = params["page"]?.toIntOrNull() ?: 0,
            size = params["size"]?.toIntOrNull()?.coerceIn(1, 100) ?: 20
        )
    }
    
    fun validateFilter(filter: ArticleFilter): ValidationResult {
        val errors = mutableListOf<String>()
        
        // Validate sort field
        val validSortFields = setOf("title", "author", "publishedAt", "createdAt", "viewCount", "likeCount")
        if (filter.sortBy !in validSortFields) {
            errors.add("Invalid sortBy field: ${filter.sortBy}. Must be one of: $validSortFields")
        }
        
        // Validate page and size
        if (filter.page < 0) {
            errors.add("Page must be non-negative")
        }
        if (filter.size !in 1..100) {
            errors.add("Size must be between 1 and 100")
        }
        
        // Validate date range
        if (filter.publishedAfter != null && filter.publishedBefore != null) {
            if (filter.publishedAfter.isAfter(filter.publishedBefore)) {
                errors.add("publishedAfter must be before publishedBefore")
            }
        }
        
        // Validate read time range
        if (filter.minReadTime != null && filter.maxReadTime != null) {
            if (filter.minReadTime > filter.maxReadTime) {
                errors.add("minReadTime must be less than or equal to maxReadTime")
            }
        }
        
        return if (errors.isEmpty()) {
            ValidationResult.Success
        } else {
            ValidationResult.Failure(errors)
        }
    }
}
```

### 4. Repository with Dynamic Filtering

```kotlin
interface ArticleRepository {
    suspend fun findAll(filter: ArticleFilter): PageResult<Article>
}

// Example implementation with in-memory filtering
class InMemoryArticleRepository(
    private val articles: MutableList<Article>
) : ArticleRepository {
    
    override suspend fun findAll(filter: ArticleFilter): PageResult<Article> {
        var filtered = articles.asSequence()
        
        // Apply filters
        filter.title?.let { title ->
            filtered = filtered.filter { 
                it.title.contains(title, ignoreCase = true) 
            }
        }
        
        filter.author?.let { author ->
            filtered = filtered.filter { it.author == author }
        }
        
        filter.category?.let { category ->
            filtered = filtered.filter { it.category == category }
        }
        
        filter.tags?.let { tags ->
            filtered = filtered.filter { article ->
                tags.any { tag -> tag in article.tags }
            }
        }
        
        filter.status?.let { status ->
            filtered = filtered.filter { it.status == status }
        }
        
        filter.minReadTime?.let { min ->
            filtered = filtered.filter { it.readTimeMinutes >= min }
        }
        
        filter.maxReadTime?.let { max ->
            filtered = filtered.filter { it.readTimeMinutes <= max }
        }
        
        filter.publishedAfter?.let { after ->
            filtered = filtered.filter { 
                it.publishedAt != null && it.publishedAt.isAfter(after) 
            }
        }
        
        filter.publishedBefore?.let { before ->
            filtered = filtered.filter { 
                it.publishedAt != null && it.publishedAt.isBefore(before) 
            }
        }
        
        // Apply sorting
        val sorted = when (filter.sortBy) {
            "title" -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy { it.title } 
                else 
                    compareByDescending { it.title }
            )
            "author" -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy { it.author } 
                else 
                    compareByDescending { it.author }
            )
            "publishedAt" -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy(nullsLast()) { it.publishedAt } 
                else 
                    compareByDescending(nullsLast()) { it.publishedAt }
            )
            "viewCount" -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy { it.viewCount } 
                else 
                    compareByDescending { it.viewCount }
            )
            "likeCount" -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy { it.likeCount } 
                else 
                    compareByDescending { it.likeCount }
            )
            else -> filtered.sortedWith(
                if (filter.sortOrder == SortOrder.ASC) 
                    compareBy { it.createdAt } 
                else 
                    compareByDescending { it.createdAt }
            )
        }
        
        // Apply pagination
        val totalElements = sorted.count().toLong()
        val totalPages = ((totalElements + filter.size - 1) / filter.size).toInt()
        val content = sorted
            .drop(filter.page * filter.size)
            .take(filter.size)
            .toList()
        
        return PageResult(
            content = content,
            page = filter.page,
            size = filter.size,
            totalElements = totalElements,
            totalPages = totalPages,
            hasNext = filter.page < totalPages - 1,
            hasPrevious = filter.page > 0
        )
    }
}
```

### 5. REST Controller/Route

```kotlin
// Ktor example
fun Route.articleRoutes(repository: ArticleRepository) {
    val filterBuilder = ArticleFilterBuilder()
    
    get("/api/articles") {
        try {
            val filter = filterBuilder.buildFilter(call.parameters)
            
            when (val validation = filterBuilder.validateFilter(filter)) {
                is ValidationResult.Success -> {
                    val result = repository.findAll(filter)
                    call.respond(HttpStatusCode.OK, result)
                }
                is ValidationResult.Failure -> {
                    call.respond(
                        HttpStatusCode.BadRequest,
                        ErrorResponse(
                            status = 400,
                            message = "Invalid filter parameters",
                            errors = validation.errors
                        )
                    )
                }
            }
        } catch (e: Exception) {
            call.respond(
                HttpStatusCode.BadRequest,
                ErrorResponse(
                    status = 400,
                    message = "Invalid query parameters: ${e.message}"
                )
            )
        }
    }
}
```

## 🎁 Bonus Tasks

### 1. Advanced Search with Full-Text Search
Implement full-text search across multiple fields:
```
GET /api/articles?q=kotlin+coroutines&searchFields=title,content,tags
```

### 2. Complex Filter Expressions
Support advanced filter syntax:
```
GET /api/articles?filter=(category:technology OR category:science) AND status:published
```

### 3. Field Selection (Sparse Fieldsets)
Allow clients to request specific fields:
```
GET /api/articles?fields=id,title,author,publishedAt
```

### 4. Aggregations and Facets
Return aggregated data alongside results:
```json
{
  "content": [...],
  "facets": {
    "categories": {
      "technology": 45,
      "science": 23,
      "business": 12
    },
    "authors": {
      "john_doe": 8,
      "jane_smith": 5
    }
  }
}
```

### 5. Saved Filters
Allow users to save and reuse filter configurations:
```
POST /api/filters
{
  "name": "My Tech Articles",
  "filter": {
    "category": "technology",
    "status": "published"
  }
}

GET /api/articles?filterId=abc-123
```

### 6. Export Filtered Results
Support exporting filtered data:
```
GET /api/articles/export?format=csv&category=technology
```

### 7. Filter Performance Optimization
- Implement query caching
- Use database indexes effectively
- Add query execution time logging

## 📦 Suggested Dependencies

```kotlin
// build.gradle.kts
dependencies {
    // Web Framework
    implementation("io.ktor:ktor-server-core:2.3.0")
    implementation("io.ktor:ktor-server-content-negotiation:2.3.0")
    
    // Database (for real implementation)
    implementation("org.jetbrains.exposed:exposed-core:0.41.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.41.1")
    implementation("org.jetbrains.exposed:exposed-jdbc:0.41.1")
    
    // Or JPA/Hibernate
    implementation("org.springframework.data:spring-data-jpa:3.0.0")
    
    // Date/Time
    implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.4.0")
    
    // Testing
    testImplementation("io.kotest:kotest-runner-junit5:5.6.0")
    testImplementation("io.kotest:kotest-property:5.6.0")
}
```

## 🧪 Testing Requirements

Write comprehensive tests:

```kotlin
class ArticleFilterTest {
    
    @Test
    fun `should filter by title`() {
        // Given articles with different titles
        // When filtering by title "Kotlin"
        // Then only articles with "Kotlin" in title are returned
    }
    
    @Test
    fun `should filter by multiple tags`() {
        // Given articles with different tags
        // When filtering by tags ["kotlin", "coroutines"]
        // Then articles with any of those tags are returned
    }
    
    @Test
    fun `should apply pagination correctly`() {
        // Given 100 articles
        // When requesting page 2 with size 20
        // Then articles 40-59 are returned
    }
    
    @Test
    fun `should sort by different fields`() {
        // Test sorting by title, date, views, etc.
    }
    
    @Test
    fun `should handle date range filters`() {
        // Test publishedAfter and publishedBefore
    }
    
    @Test
    fun `should validate invalid filters`() {
        // Test invalid sort fields, negative pages, etc.
    }
    
    @Test
    fun `should combine multiple filters`() {
        // Test applying multiple filters simultaneously
    }
}
```

## 📚 Resources

- [REST API Query Parameter Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [JSON API Specification - Filtering](https://jsonapi.org/format/#fetching-filtering)
- [OData URL Conventions](https://www.odata.org/documentation/odata-version-3-0/url-conventions/)
- [GraphQL Filtering](https://graphql.org/learn/queries/#arguments)
- [Exposed Framework Documentation](https://github.com/JetBrains/Exposed/wiki)

## ✅ Acceptance Criteria

- [ ] All filter parameters work correctly
- [ ] Filters can be combined (AND logic)
- [ ] Pagination works with all filter combinations
- [ ] Sorting works in both directions for all fields
- [ ] Invalid parameters return clear error messages
- [ ] Performance is acceptable with large datasets
- [ ] Edge cases are handled (empty results, invalid dates, etc.)
- [ ] Query parameters are properly validated
- [ ] Response includes pagination metadata
- [ ] Tests cover all filter scenarios

## 💭 Reflection Questions

After completing this challenge, consider:
1. How would you optimize filter queries for a database?
2. What's the best way to handle OR conditions in filters?
3. How can you prevent SQL injection or NoSQL injection?
4. Should filters be case-sensitive or case-insensitive by default?
5. How would you implement saved search/filter functionality?
6. What's the trade-off between filter flexibility and complexity?
