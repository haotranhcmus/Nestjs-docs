# Deep Dive: Decorators trong TypeScript và NestJS

> **Mục tiêu:** Hiểu rõ cơ chế hoạt động của Decorators từ core concept đến ứng dụng thực tế trong NestJS framework.
>
> **Đối tượng:** Backend Developer đã có kinh nghiệm với Node.js/Express, đang học NestJS.

---

## Mục lục

1. [Phần 1: Decorator trong TypeScript - Core Concept](#phần-1-decorator-trong-typescript---core-concept)
   - [1.1 Khái niệm cốt lõi](#11-khái-niệm-cốt-lõi)
   - [1.2 Cơ chế hoạt động](#12-cơ-chế-hoạt-động)
   - [1.3 Bốn loại Decorator chính](#13-bốn-loại-decorator-chính)
   - [1.4 Metadata và reflect-metadata](#14-metadata-và-reflect-metadata)
2. [Phần 2: Decorator trong NestJS - Practical](#phần-2-decorator-trong-nestjs---practical)
   - [2.1 NestJS Architecture với Decorator](#21-nestjs-architecture-với-decorator)
   - [2.2 Phân tích các Decorator phổ biến](#22-phân-tích-các-decorator-phổ-biến)
   - [2.3 So sánh Express vs NestJS](#23-so-sánh-express-vs-nestjs)
   - [2.4 Dependency Injection và Decorator](#24-dependency-injection-và-decorator)

---

## Phần 1: Decorator trong TypeScript - Core Concept

### 1.1 Khái niệm cốt lõi

#### Decorator là gì?

**Decorator** là một **Higher-Order Function** đặc biệt trong TypeScript, cho phép **meta-programming** - tức là viết code để **modify/annotate** code khác tại thời điểm **class definition**.

**Bản chất:**

- Decorator là một function nhận vào một **target** (class, method, property, hoặc parameter)
- Có thể **modify** hoặc **replace** target đó
- Hoặc chỉ **attach metadata** lên target để sử dụng sau này

**Meta-programming:**

- Thay vì chỉ viết logic nghiệp vụ, bạn viết code để **mô tả** code (metadata)
- Framework/Library có thể đọc metadata này để tự động cấu hình hoặc thay đổi behavior
- Giảm boilerplate code, tăng tính declarative (khai báo rõ ràng ý đồ)

**Higher-Order Function:**

- Decorator nhận function/class làm input
- Trả về function/class mới (hoặc không trả về gì - chỉ modify)
- Tương tự như decorator pattern trong design patterns, nhưng với syntax đặc biệt của TS

#### Tại sao cần Decorator?

**Vấn đề trước khi có Decorator:**

```typescript
// Trước: Phải manually wrap mỗi method
class UserService {
  getUser() {
    console.log('Calling getUser')
    // logic...
  }

  updateUser() {
    console.log('Calling updateUser')
    // logic...
  }
}
```

**Với Decorator:**

```typescript
// Sau: Declarative, DRY (Don't Repeat Yourself)
class UserService {
  @Log
  getUser() {
    /* logic */
  }

  @Log
  updateUser() {
    /* logic */
  }
}
```

**Lợi ích:**

- **Separation of Concerns:** Logic nghiệp vụ tách biệt khỏi cross-cutting concerns (logging, validation, caching)
- **Reusability:** Một decorator có thể dùng cho nhiều classes/methods
- **Declarative Programming:** Code dễ đọc, ý đồ rõ ràng
- **Framework Magic:** Framework có thể scan decorators để auto-configure

---

### 1.2 Cơ chế hoạt động

#### Thời điểm thực thi: Class Definition Time

**Quan trọng:** Decorator **KHÔNG** chạy khi bạn gọi method/instantiate class. Nó chạy **ngay khi class được define** (parse).

**Timeline:**

1. **Parse time (khi file được load):**
   - TypeScript compiler đọc code
   - Gặp decorator `@Something`
   - **Ngay lập tức execute** decorator function
   - Decorator có thể modify class/method ngay tại đây

2. **Runtime (khi code chạy):**
   - Class đã được modified rồi
   - Khi instantiate class hoặc gọi method, code đã bị "decorate" sẽ chạy

**Minh họa:**

```typescript
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log('Decorator executed!') // Chạy NGAY khi file load
  const originalMethod = descriptor.value

  descriptor.value = function (...args: any[]) {
    console.log('Before method') // Chạy KHI method được gọi
    const result = originalMethod.apply(this, args)
    console.log('After method')
    return result
  }
}

class Test {
  @Log
  method() {
    console.log('Method body')
  }
}

// Output khi file load:
// "Decorator executed!"

const instance = new Test()
instance.method()
// Output:
// "Before method"
// "Method body"
// "After method"
```

**Kết luận:**

- Decorator chạy **1 lần** khi class definition
- Code được modify sẽ chạy **mỗi lần** method/property được access

#### Runtime vs Compile time

**TypeScript Decorator là Runtime feature:**

- Mặc dù syntax được check lúc compile time
- Nhưng decorator function thực tế execute lúc **runtime** (khi JS code chạy)
- TypeScript chỉ transpile decorator syntax thành function calls

**Transpilation:**

TypeScript code:

```typescript
@Controller('users')
class UserController {}
```

Compiled JavaScript:

```javascript
let UserController = class UserController {}
UserController = __decorate([Controller('users')], UserController)
```

**`__decorate` helper:** TypeScript tự động inject function này để apply decorators lúc runtime.

---

### 1.3 Bốn loại Decorator chính

#### 1. Class Decorator

**Signature:**

```typescript
function ClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) {
  // constructor: class constructor function
  // Có thể return new constructor để replace class
}
```

**Mục đích:**

- Modify/extend class constructor
- Add properties/methods vào class
- Replace entire class với subclass
- **Attach metadata** lên class

**Use cases:**

- Đánh dấu class là Injectable, Controller, Entity
- Auto-register class vào container
- Seal/freeze class

**Ví dụ ngắn gọn:**

```typescript
function Sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}

@Sealed
class User {
  name: string
}
// Class User giờ không thể add properties mới
```

#### 2. Method Decorator

**Signature:**

```typescript
function MethodDecorator(
  target: any, // prototype của class (instance methods) hoặc constructor (static methods)
  propertyKey: string | symbol, // tên method
  descriptor: PropertyDescriptor // descriptor object (value, writable, enumerable, configurable)
) {
  // Có thể modify descriptor.value (replace method)
  // Return descriptor mới hoặc undefined
}
```

**Mục đích:**

- Wrap method với additional logic (logging, timing, caching)
- Validate parameters/return value
- Change method behavior
- **Store metadata** về method

**Use cases:**

- `@Get()`, `@Post()` trong NestJS
- `@Cache()`, `@Retry()`, `@Timeout()`
- `@Transactional()` trong database operations

**Ví dụ ngắn gọn:**

```typescript
function Measure(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value

  descriptor.value = async function (...args: any[]) {
    const start = Date.now()
    const result = await original.apply(this, args)
    console.log(`${propertyKey} took ${Date.now() - start}ms`)
    return result
  }
}

class API {
  @Measure
  async fetchData() {
    /* ... */
  }
}
```

#### 3. Property Decorator

**Signature:**

```typescript
function PropertyDecorator(
  target: any, // prototype (instance property) hoặc constructor (static property)
  propertyKey: string | symbol // tên property
) {
  // KHÔNG có PropertyDescriptor
  // KHÔNG thể modify property value trực tiếp
  // Chủ yếu dùng để attach metadata
}
```

**Giới hạn:**

- Property decorator **không** nhận `descriptor`
- **Không** thể intercept get/set operations
- Chủ yếu dùng để **attach metadata**

**Mục đích:**

- Mark property cho serialization/validation
- Define database column metadata
- Dependency injection hints

**Use cases:**

- `@Column()` trong TypeORM
- `@Inject()` trong DI containers
- `@Required()`, `@Min()`, `@Max()` validation decorators

**Ví dụ ngắn gọn:**

```typescript
function Column(options: any) {
  return function (target: any, propertyKey: string) {
    Reflect.defineMetadata('column:options', options, target, propertyKey)
  }
}

class User {
  @Column({ type: 'varchar', length: 255 })
  name: string
}
// Metadata được lưu, ORM sẽ đọc để tạo database schema
```

#### 4. Parameter Decorator

**Signature:**

```typescript
function ParameterDecorator(
  target: any, // prototype hoặc constructor
  propertyKey: string, // tên method chứa parameter này
  parameterIndex: number // vị trí parameter (0-based)
) {
  // Chỉ biết vị trí parameter
  // Dùng để attach metadata
}
```

**Giới hạn:**

- **Không** có quyền access giá trị parameter
- **Không** có descriptor
- **Chỉ** attach metadata

**Mục đích:**

- Mark parameter để framework extract value
- Validation rules
- Dependency injection hints

**Use cases:**

- `@Body()`, `@Param()`, `@Query()` trong NestJS
- `@Inject()` cho constructor parameters

**Ví dụ ngắn gọn:**

```typescript
function Validate(target: any, propertyKey: string, parameterIndex: number) {
  const existingParams = Reflect.getMetadata('validate:params', target, propertyKey) || []
  existingParams.push(parameterIndex)
  Reflect.defineMetadata('validate:params', existingParams, target, propertyKey)
}

class UserService {
  createUser(@Validate email: string) {
    // Framework có thể đọc metadata để biết cần validate param 0
  }
}
```

---

### 1.4 Metadata và reflect-metadata

> **ĐÂY LÀ PHẦN QUAN TRỌNG NHẤT** để hiểu cách framework như NestJS hoạt động.

#### Vấn đề: Decorator cần lưu thông tin để dùng sau

**Scenario:**

- Bạn dùng `@Get('/users')` để mark một method là route handler
- **Câu hỏi:** Làm sao NestJS biết được:
  - Method nào có decorator `@Get`?
  - Path là gì (`'/users'`)?
  - HTTP method là gì (`GET`)?

**Câu trả lời:** **Metadata**

#### Metadata là gì?

**Metadata** = "Data about data"

- Thông tin **mô tả** code (class, method, property)
- **Không** phải data nghiệp vụ
- Được **attach** vào code để framework/library đọc lại

**Ví dụ:**

- `@Get('/users')` → Metadata: `{ httpMethod: 'GET', path: '/users' }`
- `@Column({ type: 'int' })` → Metadata: `{ type: 'int' }`

#### Thư viện reflect-metadata

**reflect-metadata** là polyfill của [Metadata Reflection API](https://rbuckton.github.io/reflect-metadata/) (TC39 proposal - stage 2).

**Cài đặt:**

```bash
npm install reflect-metadata
```

**Import trong entry file:**

```typescript
import 'reflect-metadata' // Phải import trước tất cả
```

**Cấu hình tsconfig.json:**

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true // Quan trọng!
  }
}
```

#### API chính của Reflect Metadata

**1. `Reflect.defineMetadata(key, value, target, propertyKey?)`**

- **Lưu** metadata vào target
- `key`: string - metadata key (như một dictionary key)
- `value`: any - giá trị metadata
- `target`: object/class - nơi lưu metadata
- `propertyKey`: (optional) property/method name

**2. `Reflect.getMetadata(key, target, propertyKey?)`**

- **Đọc** metadata từ target
- Return value đã lưu, hoặc `undefined`

**3. `Reflect.getOwnMetadata(key, target, propertyKey?)`**

- Giống `getMetadata`, nhưng **không** search prototype chain

**4. `Reflect.hasMetadata(key, target, propertyKey?)`**

- Check xem có metadata key hay không (boolean)

**5. `Reflect.getMetadataKeys(target, propertyKey?)`**

- Lấy **tất cả** metadata keys của target

#### Design metadata keys

**Metadata keys** là string, có thể thiết kế theo pattern:

**Metadata key naming conventions:**

```typescript
'route:method' // HTTP method
'route:path' // Route path
'injectable:scope' // DI scope
'validation:rules' // Validation rules
'design:type' // TypeScript type info (auto-generated)
'design:paramtypes' // Parameter types (auto-generated)
'design:returntype' // Return type (auto-generated)
```

**Namespacing:** Dùng prefix để tránh conflict (vd: `mylib:feature:key`)

#### emitDecoratorMetadata - TypeScript Magic

**Khi bật `emitDecoratorMetadata: true`:**

TypeScript **tự động** emit 3 loại metadata cho decorated members:

**1. `design:type`** - Type của property/return type của method
**2. `design:paramtypes`** - Array các types của parameters
**3. `design:returntype`** - Return type của method

**Ví dụ:**

```typescript
class UserService {
  constructor(
    private logger: Logger,
    private db: Database
  ) {}
  //          ^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^
  //          TypeScript tự động lưu types này vào metadata!
}
```

**Metadata được emit:**

```typescript
Reflect.getMetadata('design:paramtypes', UserService)
// => [Logger, Database]
```

**Đây chính là cách NestJS biết inject gì vào constructor!**

#### Cơ chế hoạt động Under the Hood

**Bước 1: Decorator lưu metadata**

```typescript
function Get(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Lưu HTTP method
    Reflect.defineMetadata('http:method', 'GET', target, propertyKey)

    // Lưu path
    Reflect.defineMetadata('http:path', path, target, propertyKey)
  }
}

class UserController {
  @Get('/users')
  getUsers() {
    /* ... */
  }
}
```

**Bước 2: Framework scan và đọc metadata**

```typescript
// NestJS internal code (simplified)
function scanController(controller: any) {
  const prototype = controller.prototype
  const methodNames = Object.getOwnPropertyNames(prototype)

  methodNames.forEach((methodName) => {
    const httpMethod = Reflect.getMetadata('http:method', prototype, methodName)
    const path = Reflect.getMetadata('http:path', prototype, methodName)

    if (httpMethod && path) {
      // Register route
      app[httpMethod.toLowerCase()](path, prototype[methodName])
      console.log(`Registered ${httpMethod} ${path}`)
    }
  })
}

scanController(UserController)
// Output: "Registered GET /users"
```

**Bước 3: Runtime - route được gọi**

```typescript
// HTTP request: GET /users
// NestJS router match path, gọi method đã register
```

#### Metadata cho Dependency Injection

**Decorator `@Injectable()` hoạt động như thế nào?**

```typescript
function Injectable() {
  return function <T extends { new (...args: any[]): {} }>(constructor: T) {
    // Mark class as injectable
    Reflect.defineMetadata('injectable', true, constructor)

    // TypeScript đã emit design:paramtypes rồi
    // Không cần làm gì thêm
    return constructor
  }
}

@Injectable()
class UserService {
  constructor(
    private logger: Logger,
    private db: Database
  ) {}
}
```

**DI Container resolve dependencies:**

```typescript
class DIContainer {
  resolve<T>(target: any): T {
    // 1. Lấy parameter types từ metadata
    const paramTypes = Reflect.getMetadata('design:paramtypes', target) || []
    // => [Logger, Database]

    // 2. Resolve mỗi dependency recursively
    const dependencies = paramTypes.map((type) => this.resolve(type))
    // => [loggerInstance, databaseInstance]

    // 3. Instantiate với dependencies
    return new target(...dependencies)
  }
}

const container = new DIContainer()
const userService = container.resolve(UserService)
// UserService được tạo với Logger và Database đã inject!
```

#### Metadata Inheritance

**Metadata có kế thừa qua prototype chain:**

```typescript
class BaseController {
  @Authenticated()
  baseMethod() {}
}

class UserController extends BaseController {
  @Get('/users')
  getUsers() {}
}

// Có thể đọc metadata từ parent class
Reflect.getMetadata('authenticated', UserController.prototype, 'baseMethod')
// => true
```

**Lưu ý:**

- `getMetadata()` search prototype chain
- `getOwnMetadata()` chỉ check chính object đó

#### Tại sao cần Metadata thay vì thuộc tính thông thường?

**So sánh:**

**Cách 1: Dùng property thông thường**

```typescript
class UserController {
  getUsers() {}
}
UserController.prototype.getUsers.__httpMethod = 'GET'
UserController.prototype.getUsers.__path = '/users'
// ❌ Pollute function object
// ❌ Không có standardized API
```

**Cách 2: Dùng Metadata**

```typescript
Reflect.defineMetadata('http:method', 'GET', UserController.prototype, 'getUsers')
Reflect.defineMetadata('http:path', '/users', UserController.prototype, 'getUsers')
// ✅ Standardized API
// ✅ Không pollute code
// ✅ Framework có thể query dễ dàng
```

**Lợi ích Metadata:**

- **Non-invasive:** Không làm ô nhiễm code structure
- **Standardized:** API thống nhất (Reflect API)
- **Discoverable:** Framework có thể scan tất cả metadata keys
- **Type-safe:** TypeScript hỗ trợ metadata types

---

## Phần 2: Decorator trong NestJS - Practical

### 2.1 NestJS Architecture với Decorator

#### Triết lý thiết kế của NestJS

NestJS được lấy cảm hứng từ **Angular** (frontend framework), áp dụng:

- **Decorator-based configuration:** Khai báo ý đồ thay vì viết code setup
- **Dependency Injection:** IoC container tự động wire dependencies
- **Modular architecture:** Chia ứng dụng thành modules độc lập

**Tại sao lại dùng Decorator thay vì code thông thường?**

**Vấn đề với Express:**

```typescript
// Express: Imperative, scattered configuration
const app = express()

app.get('/users', async (req, res) => {
  // Manually parse body, validate, handle errors
  const users = await userService.getUsers()
  res.json(users)
})

app.post('/users', async (req, res) => {
  const userData = req.body
  // Manually validate
  if (!userData.email) {
    return res.status(400).json({ error: 'Email required' })
  }
  const user = await userService.createUser(userData)
  res.json(user)
})

// Middleware applied globally or per-route manually
app.use(authMiddleware)
```

**NestJS với Decorator:**

```typescript
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  getUsers() {
    return this.userService.getUsers()
  }

  @Post()
  @UsePipes(ValidationPipe)
  @UseGuards(AuthGuard)
  createUser(@Body() userData: CreateUserDto) {
    return this.userService.createUser(userData)
  }
}
```

**So sánh:**

- **Express:** Code imperative, manually wire mọi thứ
- **NestJS:** Declarative, framework tự động wire dựa trên metadata

#### NestJS Boot Process (Under the Hood)

**Khi NestJS app khởi động:**

**1. Scan modules**

- Đọc `@Module()` decorator
- Lấy metadata: `imports`, `controllers`, `providers`

**2. Build Dependency Graph**

- Scan `@Injectable()` classes
- Đọc `design:paramtypes` metadata
- Tạo dependency tree

**3. Instantiate Providers**

- Resolve dependencies theo thứ tự
- Inject vào constructor
- Cache instances (singleton by default)

**4. Register Routes**

- Scan `@Controller()` classes
- Scan methods có `@Get()`, `@Post()`, etc.
- Đọc metadata: HTTP method, path, guards, pipes
- Register vào underlying platform (Express/Fastify)

**5. Start HTTP Server**

- Listen port
- Route requests đến handlers đã register

**Minh họa simplified:**

```typescript
// NestJS internal (pseudo-code)
class NestFactory {
  async create(AppModule) {
    // 1. Scan module
    const moduleMetadata = Reflect.getMetadata('module', AppModule)
    const { controllers, providers } = moduleMetadata

    // 2. Create DI container
    const container = new DIContainer()

    // 3. Register providers
    providers.forEach((provider) => {
      container.register(provider)
    })

    // 4. Resolve controllers (auto-inject dependencies)
    const controllerInstances = controllers.map((ctrl) => container.resolve(ctrl))

    // 5. Register routes
    controllerInstances.forEach((instance) => {
      this.registerRoutes(instance)
    })

    // 6. Start server
    app.listen(3000)
  }

  registerRoutes(controller) {
    const prototype = Object.getPrototypeOf(controller)
    const basePath = Reflect.getMetadata('path', prototype.constructor)

    Object.getOwnPropertyNames(prototype).forEach((methodName) => {
      const httpMethod = Reflect.getMetadata('method', prototype, methodName)
      const path = Reflect.getMetadata('path', prototype, methodName)

      if (httpMethod && path) {
        const fullPath = basePath + path
        app[httpMethod](fullPath, (req, res) => {
          const result = controller[methodName](/* extract params */)
          res.json(result)
        })
      }
    })
  }
}
```

---

### 2.2 Phân tích các Decorator phổ biến

#### 2.2.1 @Controller(path?)

**Mục đích:**

- Đánh dấu class là controller
- Define base path cho routes

**Metadata được lưu:**

```typescript
Reflect.defineMetadata('path', path || '/', target)
Reflect.defineMetadata('scope', options.scope || 'singleton', target)
```

**Implementation simplified:**

```typescript
export function Controller(path: string = ''): ClassDecorator {
  return (target: Function) => {
    Reflect.defineMetadata('path', path, target)
  }
}
```

**Cách NestJS sử dụng:**

- Module scanner tìm tất cả classes có metadata `'path'`
- Kết hợp base path với route paths từ `@Get()`, `@Post()`
- Register vào Express/Fastify router

**Ví dụ:**

```typescript
@Controller('users') // Base path: /users
class UserController {
  @Get('profile') // Full path: /users/profile
  getProfile() {}

  @Get(':id') // Full path: /users/:id
  getUserById() {}
}
```

---

#### 2.2.2 @Get(), @Post(), @Put(), @Delete(), @Patch()

**Mục đích:**

- Đánh dấu method là route handler
- Define HTTP method và sub-path

**Metadata được lưu:**

```typescript
Reflect.defineMetadata('method', 'GET', target, propertyKey)
Reflect.defineMetadata('path', path || '/', target, propertyKey)
```

**Implementation simplified:**

```typescript
export function Get(path: string = ''): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    Reflect.defineMetadata('method', 'GET', target, propertyKey)
    Reflect.defineMetadata('path', path, target, propertyKey)
  }
}

// Post, Put, Delete, Patch tương tự, chỉ khác HTTP method
```

**Cách NestJS sử dụng:**

- Scan tất cả methods của controller
- Đọc metadata `'method'` và `'path'`
- Register route: `app[method](fullPath, handler)`

**Route resolution:**

```typescript
// @Controller('api/users') + @Get('active')
// => GET /api/users/active

// @Controller('api/users') + @Get()
// => GET /api/users

// @Controller() + @Get('health')
// => GET /health
```

---

#### 2.2.3 @Injectable()

**Mục đích:**

- Đánh dấu class có thể được inject vào dependencies
- Cho phép DI container quản lý lifecycle

**Metadata được lưu:**

```typescript
Reflect.defineMetadata('injectable', true, target)
Reflect.defineMetadata('scope', options.scope || 'DEFAULT', target)
// design:paramtypes tự động emit bởi TypeScript
```

**Implementation simplified:**

```typescript
export function Injectable(options?: { scope?: Scope }): ClassDecorator {
  return (target: Function) => {
    Reflect.defineMetadata('injectable', true, target)
    Reflect.defineMetadata('scope', options?.scope || 'DEFAULT', target)
  }
}
```

**Dependency Resolution:**

Khi NestJS gặp:

```typescript
@Injectable()
class UserService {
  constructor(private db: DatabaseService) {}
}
```

**Steps:**

1. Đọc `design:paramtypes` → `[DatabaseService]`
2. Check `DatabaseService` có `@Injectable()` không
3. Resolve `DatabaseService` trước (recursive)
4. Instantiate `UserService` với resolved dependencies
5. Cache instance (nếu singleton scope)

**Scope types:**

- `DEFAULT` (Singleton): Một instance cho cả app
- `REQUEST`: Một instance mỗi HTTP request
- `TRANSIENT`: Instance mới mỗi lần inject

---

#### 2.2.4 @Body(), @Param(), @Query(), @Headers()

**Mục đích:**

- Extract data từ HTTP request
- Map vào method parameters

**Vấn đề cần giải quyết:**

Trong Express:

```typescript
app.post('/users/:id', (req, res) => {
  const id = req.params.id // Path param
  const body = req.body // Request body
  const query = req.query // Query string
  const headers = req.headers // Headers

  // Manually extract mọi thứ
})
```

Trong NestJS:

```typescript
@Post(':id')
createUser(
  @Param('id') id: string,
  @Body() body: CreateUserDto,
  @Query('filter') filter: string,
  @Headers('authorization') auth: string
) {
  // NestJS tự động extract và inject
}
```

**Cách hoạt động:**

**1. Decorator lưu metadata:**

```typescript
export function Body(property?: string): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    const existingParams = Reflect.getMetadata('params', target, propertyKey) || []
    existingParams.push({
      index: parameterIndex,
      type: 'body',
      property: property
    })
    Reflect.defineMetadata('params', existingParams, target, propertyKey)
  }
}
```

**2. NestJS request handler:**

```typescript
// Pseudo-code
function createHandler(controller, methodName) {
  return async (req, res) => {
    const paramsMetadata = Reflect.getMetadata('params', controller, methodName)

    // Sort by parameter index
    const args = paramsMetadata
      .sort((a, b) => a.index - b.index)
      .map((param) => {
        switch (param.type) {
          case 'body':
            return param.property ? req.body[param.property] : req.body
          case 'param':
            return req.params[param.property]
          case 'query':
            return req.query[param.property]
          case 'headers':
            return req.headers[param.property]
        }
      })

    // Call method với extracted args
    const result = await controller[methodName](...args)
    res.json(result)
  }
}
```

**Type safety:**

- NestJS + Validation pipes: Auto-validate và transform types
- `@Body() createUserDto: CreateUserDto` → DTO class với validation decorators
- NestJS validate incoming data, throw error nếu invalid

---

#### 2.2.5 @Module()

**Mục đích:**

- Tổ chức app thành modules
- Define dependencies giữa modules

**Metadata được lưu:**

```typescript
Reflect.defineMetadata('imports', options.imports, target)
Reflect.defineMetadata('controllers', options.controllers, target)
Reflect.defineMetadata('providers', options.providers, target)
Reflect.defineMetadata('exports', options.exports, target)
```

**Implementation simplified:**

```typescript
export function Module(options: ModuleMetadata): ClassDecorator {
  return (target: Function) => {
    Object.keys(options).forEach((key) => {
      Reflect.defineMetadata(key, options[key], target)
    })
  }
}
```

**Module Resolution:**

```typescript
@Module({
  imports: [DatabaseModule], // Import modules khác
  controllers: [UserController], // Controllers trong module này
  providers: [UserService], // Services/providers
  exports: [UserService] // Export để modules khác dùng
})
class UserModule {}
```

**NestJS xử lý:**

1. Scan `imports` → load DatabaseModule trước
2. Register `providers` vào DI container (scope: module)
3. Resolve `controllers` với providers
4. `exports` cho phép modules khác import và dùng

---

### 2.3 So sánh Express vs NestJS

#### Middleware: app.use() vs Decorators

**Express Middleware:**

```typescript
// Global middleware
app.use(logger)
app.use(cors())
app.use(express.json())

// Route-specific middleware
app.get('/users', authMiddleware, rateLimitMiddleware, (req, res) => {
  // Handler
})

// Error handling middleware
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message })
})
```

**NestJS Equivalents:**

**1. Middleware (traditional style vẫn có):**

```typescript
// Implement NestMiddleware
@Injectable()
class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...')
    next()
  }
}

// Apply trong module
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*')
  }
}
```

**2. Guards (thay thế auth middleware):**

```typescript
@Injectable()
class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest()
    return validateToken(request.headers.authorization)
  }
}

// Apply với decorator
@Controller('users')
@UseGuards(AuthGuard) // Apply cho toàn controller
class UserController {
  @Get()
  @UseGuards(RoleGuard) // Apply cho specific route
  getUsers() {}
}
```

**3. Interceptors (thay thế logging/transformation middleware):**

```typescript
@Injectable()
class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...')
    const now = Date.now()

    return next.handle().pipe(tap(() => console.log(`After... ${Date.now() - now}ms`)))
  }
}

// Apply
@UseInterceptors(LoggingInterceptor)
@Controller('users')
class UserController {}
```

**4. Pipes (validation/transformation):**

```typescript
@Post()
createUser(@Body(ValidationPipe) createUserDto: CreateUserDto) {
  // DTO đã được validate tự động
}
```

**5. Exception Filters (error handling):**

```typescript
@Catch(HttpException)
class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp()
    const response = ctx.getResponse()
    const status = exception.getStatus()

    response.status(status).json({
      statusCode: status,
      message: exception.message
    })
  }
}

// Apply
@UseFilters(HttpExceptionFilter)
@Controller('users')
class UserController {}
```

#### Request/Response Flow Comparison

**Express Flow:**

```
HTTP Request
  → Middleware 1 (next())
  → Middleware 2 (next())
  → Route Handler
  → Error Middleware (nếu có lỗi)
  → Response
```

**NestJS Flow:**

```
HTTP Request
  → Middleware (NestMiddleware)
  → Guards (CanActivate) - Authentication/Authorization
    ↓ (nếu pass)
  → Interceptors (BEFORE) - Logging, Transform request
    ↓
  → Pipes - Validation, Transformation
    ↓
  → Route Handler (Controller method)
    ↓
  → Interceptors (AFTER) - Transform response, Logging
    ↓
  → Exception Filters (nếu có lỗi)
    ↓
  → Response
```

**Key Differences:**

| Aspect                     | Express                        | NestJS                                  |
| -------------------------- | ------------------------------ | --------------------------------------- |
| **Configuration**          | Imperative (`app.use()`)       | Declarative (`@UseGuards()`)            |
| **Scope**                  | Global hoặc per-route manually | Global, Controller, Method, Param level |
| **Type Safety**            | Không có                       | Strong typing với TypeScript            |
| **Dependency Injection**   | Manual                         | Tự động                                 |
| **Separation of Concerns** | Code lẫn lộn logic             | Guards, Pipes, Interceptors rõ ràng     |

---

### 2.4 Dependency Injection và Decorator

#### Tại sao NestJS phụ thuộc nặng vào Decorator cho DI?

**Vấn đề với manual DI:**

```typescript
// Không có DI - tightly coupled
class UserController {
  private userService = new UserService() // ❌ Hard-coded dependency

  getUsers() {
    return this.userService.getUsers()
  }
}

// Vấn đề:
// - Không thể mock UserService khi test
// - Phải manually manage dependencies
// - Khó thay đổi implementation
```

**Constructor Injection thủ công:**

```typescript
class UserController {
  constructor(private userService: UserService) {}
}

// Phải manually wire:
const userService = new UserService(new Database(), new Logger())
const controller = new UserController(userService)
// ❌ Phức tạp khi nhiều dependencies
```

**Với Decorator + DI Container:**

```typescript
@Injectable()
class UserService {
  constructor(
    private db: Database,
    private logger: Logger
  ) {}
}

@Controller('users')
class UserController {
  constructor(private userService: UserService) {}
}

// ✅ NestJS tự động wire mọi thứ!
```

#### IoC Container hoạt động như thế nào?

**Inversion of Control (IoC):**

- Thay vì code **chủ động** tạo dependencies
- Framework **điều khiển** việc tạo và inject dependencies

**NestJS IoC Container:**

**1. Registration Phase:**

```typescript
// Module metadata
@Module({
  providers: [UserService, Database, Logger]
})
class AppModule {}

// NestJS scan và register:
container.register(UserService)
container.register(Database)
container.register(Logger)
```

**2. Resolution Phase:**

```typescript
class Container {
  private instances = new Map()
  private providers = new Map()

  register(provider: Type<any>) {
    this.providers.set(provider, provider)
  }

  resolve<T>(target: Type<T>): T {
    // Check cache (singleton)
    if (this.instances.has(target)) {
      return this.instances.get(target)
    }

    // Get parameter types từ metadata
    const paramTypes = Reflect.getMetadata('design:paramtypes', target) || []

    // Resolve dependencies recursively
    const dependencies = paramTypes.map((type) => this.resolve(type))

    // Create instance
    const instance = new target(...dependencies)

    // Cache
    this.instances.set(target, instance)

    return instance
  }
}
```

**3. Circular Dependency Detection:**

NestJS detect circular dependencies:

```typescript
// UserService depends on OrderService
@Injectable()
class UserService {
  constructor(private orderService: OrderService) {}
}

// OrderService depends on UserService
@Injectable()
class OrderService {
  constructor(private userService: UserService) {}
}

// ❌ NestJS sẽ throw error: Circular dependency detected
```

**Giải pháp:** `forwardRef()`

```typescript
@Injectable()
class UserService {
  constructor(
    @Inject(forwardRef(() => OrderService))
    private orderService: OrderService
  ) {}
}
```

#### Custom Providers

**Value Provider:**

```typescript
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: { apiKey: 'secret' }
    }
  ]
})

// Inject
constructor(@Inject('CONFIG') private config: any) {}
```

**Factory Provider:**

```typescript
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async () => {
        const connection = await createConnection();
        return connection;
      }
    }
  ]
})
```

**Class Provider:**

```typescript
@Module({
  providers: [
    {
      provide: UserService,
      useClass: MockUserService  // Swap implementation
    }
  ]
})
```

#### Design:paramtypes - The Magic Behind DI

**Khi bật `emitDecoratorMetadata`:**

```typescript
@Injectable()
class UserService {
  constructor(
    private db: Database,
    private logger: Logger,
    private config: ConfigService
  ) {}
}
```

**TypeScript emit:**

```javascript
UserService = __decorate(
  [Injectable(), __metadata('design:paramtypes', [Database, Logger, ConfigService])],
  UserService
)
```

**NestJS đọc:**

```typescript
const deps = Reflect.getMetadata('design:paramtypes', UserService)
// => [Database, Logger, ConfigService]

// Resolve mỗi dependency
const db = container.resolve(Database)
const logger = container.resolve(Logger)
const config = container.resolve(ConfigService)

// Inject
const userService = new UserService(db, logger, config)
```

**Đây chính là lý do phải có `@Injectable()` decorator:**

- Không chỉ để mark class
- Để trigger TypeScript emit `design:paramtypes` metadata
- Không có decorator → không có metadata → DI không hoạt động

#### Interface Injection Problem

**Vấn đề:** TypeScript interfaces biến mất sau compile

```typescript
interface ILogger {
  log(message: string): void
}

@Injectable()
class UserService {
  constructor(private logger: ILogger) {} // ❌ ILogger không tồn tại runtime
}
```

**Giải pháp 1: Injection Token**

```typescript
const LOGGER_TOKEN = 'ILogger'

@Module({
  providers: [
    {
      provide: LOGGER_TOKEN,
      useClass: ConsoleLogger
    }
  ]
})
@Injectable()
class UserService {
  constructor(@Inject(LOGGER_TOKEN) private logger: ILogger) {}
}
```

**Giải pháp 2: Abstract Class**

```typescript
abstract class ILogger {
  abstract log(message: string): void
}

@Injectable()
class ConsoleLogger extends ILogger {
  log(message: string) {
    console.log(message)
  }
}

@Module({
  providers: [{ provide: ILogger, useClass: ConsoleLogger }]
})
@Injectable()
class UserService {
  constructor(private logger: ILogger) {} // ✅ Works!
}
```

---

## Tổng kết

### Core Concepts Recap

**1. Decorator = Higher-Order Function + Meta-programming**

- Decorator modify/annotate code tại class definition time
- Execute ngay khi class được parse, không phải khi instantiate

**2. Metadata = "Data về Data"**

- `reflect-metadata` lưu thông tin vào code
- Framework đọc metadata để auto-configure
- `emitDecoratorMetadata` cho phép TypeScript tự emit type info

**3. Bốn loại Decorator:**

- **Class:** Modify/annotate class
- **Method:** Wrap/modify method behavior
- **Property:** Attach metadata (không thể modify value trực tiếp)
- **Parameter:** Mark parameter (không access value)

### NestJS Decorator Architecture

**1. Configuration through Decoration**

- `@Controller()`, `@Get()` → Routing
- `@Injectable()` → DI
- `@UseGuards()`, `@UsePipes()` → Request pipeline

**2. Request Lifecycle:**

```
Middleware → Guards → Interceptors (before) → Pipes → Handler → Interceptors (after) → Filters
```

**3. Dependency Injection:**

- `design:paramtypes` metadata → auto-wire dependencies
- IoC Container quản lý lifecycle
- Decorators trigger metadata emission

### Express vs NestJS Philosophy

| Express             | NestJS               |
| ------------------- | -------------------- |
| Imperative          | Declarative          |
| Manual wiring       | Auto DI              |
| Scattered config    | Centralized metadata |
| Runtime flexibility | Compile-time safety  |

### Khi nào nên dùng Decorator?

**✅ Nên dùng khi:**

- Framework-level configuration (routing, DI)
- Cross-cutting concerns (logging, caching, auth)
- Declarative validation/transformation
- Cần type safety và auto-completion

**❌ Không nên dùng khi:**

- Business logic đơn giản
- Dynamic behavior cần runtime flexibility cao
- Performance critical (decorator có overhead nhỏ)

---

## Tài liệu tham khảo

- [TypeScript Decorators Handbook](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [TC39 Decorator Proposal](https://github.com/tc39/proposal-decorators)
- [reflect-metadata Documentation](https://rbuckton.github.io/reflect-metadata/)
- [NestJS Official Docs - Custom Decorators](https://docs.nestjs.com/custom-decorators)
- [NestJS Source Code](https://github.com/nestjs/nest) - Đọc source để hiểu sâu hơn

---

_Viết bởi Senior Backend Engineer - Chuyên sâu NestJS/TypeScript - January 2026_
