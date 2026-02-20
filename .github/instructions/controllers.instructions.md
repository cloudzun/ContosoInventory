---
name: 'API Controller Standards'
description: 'Coding standards for ASP.NET Core API controllers'
applyTo: '**/Controllers/**/*.cs'
---

# API Controller Standards

- Inherit from ControllerBase + [ApiController]
- [Route("api/[controller]")]
- Delegate business logic to services
- Use [HttpGet], [HttpPost], etc.
- Return IActionResult or ActionResult<T>
- [ProducesResponseType] attributes
- Inject services in constructor
