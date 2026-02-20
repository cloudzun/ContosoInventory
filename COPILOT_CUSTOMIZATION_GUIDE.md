## 🎯 核心操作流程（去除所有说明）

### 📦 1. 准备项目

```bash
# 在 GitHub 导入 starter 仓库
https://github.com/MicrosoftLearning/github-copilot-customization-starter-app
# 命名为 ContosoInventory

# 本地克隆
git clone https://github.com/YOUR-USERNAME/ContosoInventory.git
cd ContosoInventory
code .
```

------

### 📝 2. 创建自定义指令文件

#### 2.1 全局指令

**`.github/copilot-instructions.md`**

```markdown
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
```

#### 2.2 路径特定指令

**`.github/instructions/controllers.instructions.md`**

```markdown
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
```

**`.github/instructions/services.instructions.md`**

```markdown
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
```

------

### 🤖 3. 创建 Agents

#### 3.1 Planner Agent

**`.github/agents/planner.agent.md`**

```markdown
---
description: Analyzes feature requirements and generates implementation plans without writing code
tools: ['search', 'read', 'fetch']
handoffs:
  - label: Start Implementation
    agent: implementer
    prompt: "Implement the plan outlined above. Follow the project's custom instructions for coding standards. Create all necessary files including models, DTOs, services, interfaces, and controllers."
    send: false
---
# Planner

You are a senior software architect. Generate detailed implementation plans.

Your plan MUST include:
1. Summary
2. Files to create/modify
3. Implementation steps
4. Models and DTOs
5. Service interface/implementation
6. Controller endpoints
7. Dependency injection
8. Risks and considerations

RULES:
- Do NOT write code
- Follow project architecture patterns
- Ask clarifying questions if needed
```

#### 3.2 Implementer Agent

**`.github/agents/implementer.agent.md`**

```markdown
---
description: Implements code changes based on plans, following the project coding standards
tools: ['search', 'read', 'edit', 'terminal']
handoffs:
  - label: Review Code
    agent: reviewer
    prompt: "Review the code changes made in the implementation above. Check for bugs, security issues, naming convention violations, and adherence to the project's coding standards."
    send: false
---
# Implementer

You are an expert C# developer. Implement code changes.

WORKFLOW:
1. Read plan
2. Search existing patterns
3. Implement following standards
4. Create files in correct directories
5. Summarize changes

RULES:
- Repository pattern + DI
- DTOs for API payloads
- XML documentation
- Underscore prefix for private fields
- async/await for I/O
- Proper error handling
```

#### 3.3 Reviewer Agent

**`.github/agents/reviewer.agent.md`**

```markdown
---
description: Reviews code for bugs, security issues, and coding standards compliance
tools: ['search', 'read']
handoffs:
  - label: Fix Issues
    agent: implementer
    prompt: "Fix the issues identified in the code review above. Address each finding in order of severity, starting with Critical and High issues first."
    send: false
---
# Code Reviewer

Review code for:
1. Bugs and logical errors
2. Security vulnerabilities
3. Naming convention violations
4. Architecture compliance
5. Error handling
6. Missing documentation
7. Performance issues

Output Format:
- Group by severity: Critical/High/Medium/Low
- Include: file, location, issue, suggested fix
- Overall Assessment

IMPORTANT: Do NOT modify files.
```

------

### ⚙️ 4. 验证配置

```bash
# VS Code Settings
Ctrl + , 
搜索 "Use Instruction Files"
确保勾选 ✅ Code Generation: Use Instruction Files

# 在 GitHub Copilot Chat 测试
@planner what naming convention for private fields?
# 应该回答: underscore prefix (_field)
```

------

### 🚀 5. 运行完整工作流

**在 Copilot Chat 中：**

```
1️⃣ 选择 @planner
输入: 
"I need to add a Product Inventory management feature to this Web API, 
following the same patterns used by the existing Category feature. 
The feature should support:
- Product model: Id, Name, Sku, Description, Price, StockQuantity, CategoryId
- CRUD operations
- Restock endpoint
- Admin authorization for write operations"

2️⃣ 点击 "Start Implementation" 按钮
→ 自动切换到 @implementer

3️⃣ 点击 "Review Code" 按钮
→ 自动切换到 @reviewer

4️⃣ 如果有问题，点击 "Fix Issues" 按钮
→ 返回 @implementer 修复
```

------

## 🔑 关键点



| 步骤                | 必须做什么                                              | 自动发生什么           |
| ------------------- | ------------------------------------------------------- | ---------------------- |
| **创建指令文件**    | ✍️ 手工创建 `.github/` 下的所有 `.md` 文件               | -                      |
| **创建 Agent 文件** | ✍️ 手工创建 `.github/agents/` 下的 3 个 `.agent.md` 文件 | -                      |
| **验证配置**        | ✅ 检查 VS Code 设置                                     | Copilot 自动加载指令   |
| **运行工作流**      | 🎯 在 Chat 中选择 Agent + 输入需求                       | Agent 链自动切换       |
| **代码生成**        | 👀 审查 Planner 的计划                                   | Implementer 自动写代码 |
| **代码审查**        | 👀 审查 Reviewer 的发现                                  | Reviewer 自动检查      |

------

## 📋 文件清单（必须手工创建）

```
.github/
├── copilot-instructions.md          ← 全局编码规范
├── instructions/
│   ├── controllers.instructions.md  ← Controller 特定规范
│   └── services.instructions.md     ← Service 特定规范
└── agents/
    ├── planner.agent.md             ← 规划 Agent
    ├── implementer.agent.md         ← 实现 Agent
    └── reviewer.agent.md            ← 审查 Agent
```

**总计**: 6 个文件，全部手工创建（复制粘贴内容）
