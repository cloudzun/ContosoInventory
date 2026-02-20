---
name: 'Service Layer Standards'
description: 'Coding standards for business logic service classes'
applyTo: '**/Services/**/*.cs'
---

# Service Layer Standards

- Define interface for every service (IProductService)
- async/await for I/O operations
- Accept/return DTOs, not entities
- Input validation at method start
- Throw specific exceptions (ArgumentException, InvalidOperationException)
- ILogger<T> for structured logging
