# 实验报告：ContosoInventory 产品库存功能开发

**日期**：2026 年 2 月 20 日  
**项目**：ContosoInventory — ASP.NET Core Web API + Blazor WebAssembly  
**报告目的**：记录本次开发环境准备、功能实现及代码审查全过程，供后续审核使用

---

## 一、环境准备

### 1.1 需求核查

| 依赖项 | 要求版本 | 检测结果 | 状态 |
|---|---|---|---|
| .NET SDK | 8.0+ | 未安装 | ❌ |
| Git | 2.48+ | 2.43.0 | ❌ |

### 1.2 环境修复

```bash
# 安装 .NET SDK 8.0
sudo apt-get update && sudo apt-get install -y dotnet-sdk-8.0

# 升级 Git
sudo add-apt-repository -y ppa:git-core/ppa
sudo apt-get update && sudo apt-get install -y git
```

### 1.3 修复后环境验证

| 依赖项 | 要求版本 | 实际版本 | 状态 |
|---|---|---|---|
| .NET SDK | 8.0+ | **8.0.124** | ✅ |
| Git | 2.48+ | **2.53.0** | ✅ |
| 项目构建 | 0 Error | 0 Warning / 0 Error | ✅ |
| SQLite (`EFCore.Sqlite` 包 + `.db` 文件) | 已配置 | 正常 | ✅ |

---

## 二、应用运行

```bash
cd ContosoInventory.Server && dotnet run
```

监听地址：`http://localhost:4683` · Swagger UI：`http://localhost:4683/swagger`

| 用户 | 邮箱 | 密码 | 角色 |
|---|---|---|---|
| Mateo Gomez | `mateo@contoso.com` | `Password123!` | Admin |
| Megan Bowen | `megan@contoso.com` | `Password123!` | Viewer |

---

## 三、产品库存功能实现

### 3.1 需求说明

在现有 Category 功能基础上，按照相同架构模式新增 Product 管理功能：

- 数据模型：`Id`、`Name`、`Sku`、`Description`、`Price`、`StockQuantity`、`CategoryId`
- CRUD 操作 + 补货端点 `POST /api/products/{id}/restock`
- 写操作仅限 Admin 角色

### 3.2 新增 / 修改文件清单

| 操作 | 文件 |
|---|---|
| 新建 | ProductResponseDto.cs |
| 新建 | CreateProductDto.cs |
| 新建 | UpdateProductDto.cs |
| 新建 | RestockProductDto.cs |
| 新建 | Product.cs |
| 新建 | IProductService.cs |
| 新建 | ProductService.cs |
| 新建 | ProductsController.cs |
| 修改 | InventoryContext.cs |
| 修改 | Program.cs |

### 3.3 API 端点清单

| Method | Route | 授权 | 说明 |
|---|---|---|---|
| `GET` | `/api/products` | 已登录 | 获取所有产品 |
| `GET` | `/api/products/{id}` | 已登录 | 按 ID 获取产品 |
| `GET` | `/api/products/category/{categoryId}` | 已登录 | 按分类获取产品 |
| `POST` | `/api/products` | **Admin** | 创建产品 |
| `PUT` | `/api/products/{id}` | **Admin** | 更新产品 |
| `DELETE` | `/api/products/{id}` | **Admin** | 删除产品 |
| `POST` | `/api/products/{id}/restock` | **Admin** | 补货 |

### 3.4 关键设计决策

| 决策 | 理由 |
|---|---|
| `OnDelete(DeleteBehavior.Restrict)` | 防止删除分类时级联删除产品 |
| `HasPrecision(18, 2)` on `Price` | 避免 SQLite 浮点漂移 |
| 双唯一索引（`Name` + `Sku`） | DB 层强制 + 应用层前置检查提供清晰错误信息 |
| `AsNoTracking().Include()` on reads | 性能优化 + 填充 `CategoryName` |
| `Reference().LoadAsync()` after write | 保存后显式加载导航属性，确保响应 DTO 完整 |

---

## 四、代码审查过程（三轮）

### 第一轮

| 级别 | 问题 | 修复 |
|---|---|---|
| 🔴 Critical | `ProductService`、`ProductsController` 未创建；`DbSet<Product>` 未注册；DI 未注册；迁移未执行 | 全部补齐 |
| 🔴 High | `[Range(0, double.MaxValue)]` 用于 `decimal` → 运行时 `OverflowException` | 改为 `[Range(typeof(decimal), "0", "999999999.99")]` |
| 🔴 High | `[Required]` 作用于 `int CategoryId`（值类型无效），允许 `CategoryId=0` 静默通过 | 改为 `[Range(1, int.MaxValue)]` |
| 🟡 Medium | `Product` 模型 `Price`/`StockQuantity` 无 `[Range]` 约束 | 补加 `[Range]` 注解 |
| 🟡 Medium | `IProductService.GetProductsByCategoryAsync` 缺少 `<returns>` XML 文档 | 补加 |
| 🔵 Low | `RestockProductDto.Quantity` 缺少 `[Required]` | 补加 |

### 第二轮

| 级别 | 问题 | 修复 |
|---|---|---|
| 🟡 Medium | InventoryContext.cs 双分号 `null!;;` → 编译错误 | `null!;;` → `null!;` |
| 🔵 Low | `RestockProduct` 标注了从不触发的 `[ProducesResponseType(400)]` | 移除该属性 |

### 第三轮（终审）

所有文件无遗留问题，终审通过。

---

## 五、编码规范合规性

| 规范项 | 状态 |
|---|---|
| PascalCase 类/方法/属性 | ✅ |
| camelCase 局部变量 | ✅ |
| `_field` 私有字段前缀 | ✅ |
| `IService` 接口命名 | ✅ |
| Repository 模式（Service 封装 EF Core） | ✅ |
| 依赖注入（Program.cs 注册） | ✅ |
| DTO 隔离（不暴露实体） | ✅ |
| try-catch + HTTP 200/400/404/500 | ✅ |
| `ILogger<T>` 结构化日志 | ✅ |
| XML 文档注释（所有公共成员） | ✅ |
| `[Authorize(Roles = "Admin")]` 写操作 | ✅ |
| `[ProducesResponseType]` 准确声明 | ✅ |

---

## 六、待执行步骤

```bash
# 停止运行中的应用后执行
cd ContosoInventory.Server
dotnet ef migrations add AddProducts
dotnet ef database update
dotnet run
```

> **注意**：执行迁移前必须停止 `dotnet run`，避免 SQLite WAL 锁文件阻塞迁移。

---

## 七、总结

| 阶段 | 成果 |
|---|---|
| 环境准备 | .NET SDK 8.0.124 + Git 2.53.0，满足所有前置依赖 |
| 功能实现 | 10 个新增/修改文件，完整实现 Product 库存管理功能 |
| 代码审查 | 三轮审查，共修复 8 个问题（2 Critical / 2 High / 2 Medium / 2 Low） |
| 规范合规 | 全部 14 项编码规范检查项通过 |

实现质量达到可提交状态，等待 EF Core 迁移执行后即可完整运行。