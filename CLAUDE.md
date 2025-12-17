# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Spring REST Docs API specification extension** that generates OpenAPI 2.0/3.0 and Postman collections from Spring REST Docs tests. It takes a test-driven approach to API documentation generation.

## Essential Commands

### Build and Test
```bash
./gradlew build         # Build all modules
./gradlew test          # Run all tests
./gradlew check         # Run tests and code quality checks
./gradlew clean build   # Clean build
```

### Code Quality
```bash
./gradlew ktlintCheck   # Check Kotlin code style
./gradlew ktlintFormat  # Format Kotlin code
```

### API Specification Generation
```bash
./gradlew openapi       # Generate OpenAPI 2.0 specification  
./gradlew openapi3      # Generate OpenAPI 3.0.1 specification
./gradlew postman       # Generate Postman collection
```

### Coverage Reports
```bash
./gradlew jacocoTestReport      # Generate coverage reports for individual modules
./gradlew jacocoRootReport      # Generate aggregate coverage report
```

### Working with Samples
```bash
cd samples/restdocs-api-spec-sample
./gradlew build openapi3        # Build sample and generate OpenAPI spec
```

## Architecture Overview

### Multi-Module Structure
This is a Gradle multi-module project with the following key modules:

- **restdocs-api-spec**: Core library with `ResourceDocumentation` entry point and `ResourceSnippet`
- **restdocs-api-spec-model**: Shared model classes for API specifications  
- **restdocs-api-spec-jsonschema**: JSON Schema generation from field descriptors
- **restdocs-api-spec-mockmvc**: MockMvc integration wrapper
- **restdocs-api-spec-restassured**: RestAssured integration wrapper
- **restdocs-api-spec-webtestclient**: WebTestClient integration wrapper
- **restdocs-api-spec-openapi-generator**: OpenAPI 2.0 specification generator
- **restdocs-api-spec-openapi3-generator**: OpenAPI 3.0.1 specification generator
- **restdocs-api-spec-postman-generator**: Postman collection generator
- **restdocs-api-spec-gradle-plugin**: Gradle plugin that aggregates resource.json files

### Test-to-Specification Flow

1. **Test Phase**: Tests use `ResourceDocumentation.resource()` with Spring REST Docs
2. **Snippet Generation**: Each test produces a `resource.json` file in snippets directory
3. **Aggregation**: Gradle plugin collects all `resource.json` files
4. **Specification Generation**: Generates OpenAPI/Postman specifications from aggregated data

### Core Classes
- `ResourceDocumentation`: Main entry point for creating resource snippets
- `ResourceSnippet`: Generates JSON metadata about documented resources
- `ResourceSnippetParameters`: Builder for configuring resource documentation
- `*RestDocumentationWrapper`: Convenience wrappers for existing Spring REST Docs tests

## Key Development Patterns

### Resource Documentation Usage
```java
// Basic usage
mockMvc.perform(post("/carts"))
  .andDo(document("carts-create", resource("Create a cart")));

// Advanced usage with parameters  
mockMvc.perform(get("/carts/{id}", cartId))
  .andDo(document("cart-get",
    resource(ResourceSnippetParameters.builder()
      .description("Get cart by ID")
      .pathParameters(parameterWithName("id").description("Cart ID"))
      .responseFields(
        fieldWithPath("total").description("Total amount"),
        fieldWithPath("products").description("Cart items"))
      .build())));
```

### Gradle Plugin Configuration
Projects need openapi/openapi3/postman configuration blocks in build.gradle:

```groovy
openapi3 {
    server = 'https://localhost:8080'
    title = 'My API'
    description = 'API description'
    version = '0.1.0'
    format = 'yaml'
}
```

## Technology Stack

- **Languages**: Kotlin (main library), Java (samples and model classes)
- **Build**: Gradle with Kotlin DSL
- **Testing**: JUnit 5, Spring Boot Test, Spring REST Docs
- **Code Quality**: Kotlinter (ktlint), JaCoCo, SonarQube
- **Target**: Spring Boot 3.x, Spring REST Docs 3.x, Java 17

## Testing Strategy

### Test Location Patterns
- Core library tests: `restdocs-api-spec/src/test/kotlin/`
- Integration tests: `**/src/test/**/ResourceSnippetIntegrationTest.kt`
- Sample application tests: `samples/*/src/test/java/`

### Running Specific Tests
```bash
# Test specific module
./gradlew :restdocs-api-spec:test

# Test specific pattern
./gradlew test --tests "*ResourceSnippet*"

# Sample application tests
./gradlew :restdocs-api-spec-sample:test
```

## Version Compatibility

- Spring Boot 3.x requires restdocs-api-spec 0.17.1+
- Spring Boot 2.x requires restdocs-api-spec 0.16.4
- Uses semantic versioning based on Git tags