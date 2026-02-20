# Contoso Inventory API - Coding Standards

## Naming Conventions
- PascalCase for classes, methods, properties
- camelCase for local variables
- Prefix private fields with underscore (_field)
- Suffix interfaces with "I" (IService)

## Architecture Patterns
- Repository pattern for data access
- Dependency injection in Program.cs
- Service classes for business logic
- DTOs for API payloads (never expose entities)

## Error Handling
- try-catch for external calls
- HTTP status codes: 200/400/404/500
- ILogger<T> for logging
- Meaningful error messages

## Documentation
- XML comments on public methods
- Inline comments for complex logic

## Testing
- xUnit + Moq
- Arrange-Act-Assert pattern
- Test names: MethodName_Scenario_ExpectedResult
