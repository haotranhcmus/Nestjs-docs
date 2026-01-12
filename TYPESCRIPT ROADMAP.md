# 🚀 TypeScript Advanced Learning Roadmap

> **Mục tiêu:** Sau khi hoàn thành checklist này, bạn sẽ tự tin xây dựng backend phức tạp với độ chặt chẽ cao về Type Safety.
>
> **Đối tượng:** Người đã biết JavaScript cơ bản và setup môi trường TypeScript.

---

## 📋 Mục lục

1. [Phần 1: TypeScript Fundamentals](#phần-1-typescript-fundamentals)
2. [Phần 2: Object-Oriented Programming (OOP) - TRỌNG TÂM](#phần-2-object-oriented-programming-oop---trọng-tâm)
3. [Phần 3: Asynchronous Programming - TRỌNG TÂM](#phần-3-asynchronous-programming---trọng-tâm)
4. [Phần 4: Generics & Advanced Types](#phần-4-generics--advanced-types)
5. [Phần 5: Utility Types](#phần-5-utility-types)
6. [Phần 6: Decorators](#phần-6-decorators)
7. [Phần 7: Module System & Project Structure](#phần-7-module-system--project-structure)
8. [Phần 8: Real-world Application](#phần-8-real-world-application)

---

## Phần 1: TypeScript Fundamentals

> ⏱️ Thời gian ước tính: 1-2 ngày

### Basic Types

- [ ] Hiểu primitive types: `string`, `number`, `boolean`, `null`, `undefined`
- [ ] Sử dụng `any`, `unknown`, `never`, `void` đúng ngữ cảnh
- [ ] Phân biệt `any` vs `unknown` và khi nào nên dùng
- [ ] Type Inference - để TypeScript tự suy luận type
- [ ] Type Assertion (`as` keyword và angle-bracket syntax)

### Arrays & Tuples

- [ ] Khai báo Array types (`T[]` và `Array<T>`)
- [ ] Tuple types với fixed length và types
- [ ] Readonly arrays và tuples

### Objects & Type Aliases

- [ ] Object type annotations
- [ ] Type Aliases với `type` keyword
- [ ] Optional properties (`?`)
- [ ] Index Signatures cho dynamic keys

### Union & Intersection Types

- [ ] Union types (`|`)
- [ ] Intersection types (`&`)
- [ ] Type Narrowing với `typeof`, `instanceof`, `in`
- [ ] Discriminated Unions (Tagged Unions)

### Literal Types & Enums

- [ ] String/Number literal types
- [ ] `const` assertions
- [ ] Numeric Enums vs String Enums
- [ ] `const` enums và khi nào nên dùng

---

## Phần 2: Object-Oriented Programming (OOP) - TRỌNG TÂM

> ⏱️ Thời gian ước tính: 5-7 ngày | 🎯 **ĐÂY LÀ PHẦN TRỌNG TÂM**

### 2.1 Classes Fundamentals

#### Khai báo và Khởi tạo

- [ ] Class declaration và class expression
- [ ] Constructor và parameter properties (shorthand syntax)
- [ ] Property initialization (trong constructor vs trực tiếp)
- [ ] Strict property initialization (`strictPropertyInitialization`)

#### Access Modifiers

- [ ] `public` - mặc định, truy cập từ mọi nơi
- [ ] `private` - chỉ truy cập trong class, **không** kế thừa
- [ ] `protected` - truy cập trong class và subclass
- [ ] `readonly` - chỉ gán giá trị 1 lần (trong constructor hoặc khai báo)
- [ ] Kết hợp modifiers: `private readonly`, `protected readonly`
- [ ] So sánh `private` của TS vs `#` private fields của ES2022

#### Static Members

- [ ] Static properties
- [ ] Static methods
- [ ] Static blocks (ES2022)
- [ ] Khi nào dùng static vs instance members
- [ ] Singleton pattern với static

#### Getters & Setters

- [ ] Định nghĩa getter (`get`)
- [ ] Định nghĩa setter (`set`)
- [ ] Validation trong setter
- [ ] Computed properties với getter
- [ ] Lazy initialization pattern

### 2.2 Interfaces

#### Interface Basics

- [ ] Khai báo interface cho object shape
- [ ] Optional properties (`?`) và readonly properties
- [ ] Function type trong interface
- [ ] Index signatures trong interface

#### Interface vs Type Alias

- [ ] Khi nào dùng `interface` vs `type`
- [ ] Declaration merging (chỉ interface có)
- [ ] Extending interfaces vs intersecting types

#### Interface cho Class

- [ ] `implements` keyword
- [ ] Implement multiple interfaces
- [ ] Interface cho constructor signature
- [ ] Interface segregation (chia nhỏ interface)

### 2.3 Inheritance & Polymorphism

#### Class Inheritance

- [ ] `extends` keyword
- [ ] `super()` gọi constructor cha
- [ ] `super.method()` gọi method cha
- [ ] Method overriding
- [ ] Property overriding

#### Abstract Classes

- [ ] Khai báo abstract class
- [ ] Abstract methods (không có implementation)
- [ ] Abstract properties
- [ ] Khi nào dùng Abstract Class vs Interface
- [ ] Template Method pattern với abstract class

#### Polymorphism

- [ ] Hiểu polymorphism qua method overriding
- [ ] Upcasting và Downcasting
- [ ] Type guards với `instanceof`
- [ ] Duck typing trong TypeScript

### 2.4 SOLID Principles trong TypeScript

#### S - Single Responsibility Principle (SRP)

- [ ] Hiểu nguyên lý: Một class chỉ nên có một lý do để thay đổi
- [ ] Nhận diện class vi phạm SRP
- [ ] Tách biệt responsibilities vào các class khác nhau
- [ ] Áp dụng SRP cho services, repositories, controllers

#### O - Open/Closed Principle (OCP)

- [ ] Hiểu nguyên lý: Open for extension, closed for modification
- [ ] Sử dụng inheritance để extend behavior
- [ ] Strategy pattern để thay đổi algorithm
- [ ] Plugin architecture

#### L - Liskov Substitution Principle (LSP)

- [ ] Hiểu nguyên lý: Subtype phải thay thế được base type
- [ ] Nhận diện vi phạm LSP (thay đổi behavior không mong muốn)
- [ ] Covariance và Contravariance
- [ ] Design by Contract

#### I - Interface Segregation Principle (ISP)

- [ ] Hiểu nguyên lý: Client không nên phụ thuộc vào interface không dùng
- [ ] Chia nhỏ "fat" interfaces
- [ ] Role interfaces
- [ ] Compose interfaces từ các interface nhỏ

#### D - Dependency Inversion Principle (DIP)

- [ ] Hiểu nguyên lý: Phụ thuộc vào abstraction, không phải implementation
- [ ] Dependency Injection patterns
- [ ] Constructor injection
- [ ] Property injection
- [ ] Method injection
- [ ] IoC (Inversion of Control) containers

### 2.5 Design Patterns với TypeScript

#### Creational Patterns

##### Singleton Pattern

- [ ] Hiểu vấn đề Singleton giải quyết
- [ ] Implement với private constructor
- [ ] Lazy initialization
- [ ] Thread-safe Singleton (trong context async)
- [ ] Singleton vs Static class
- [ ] Anti-patterns và khi nào KHÔNG nên dùng Singleton

##### Factory Pattern

- [ ] Simple Factory
- [ ] Factory Method Pattern
- [ ] Abstract Factory Pattern
- [ ] Khi nào dùng Factory thay vì `new`
- [ ] Factory với Generics

##### Builder Pattern

- [ ] Fluent interface / Method chaining
- [ ] Director class
- [ ] Khi nào dùng Builder (object phức tạp, nhiều optional params)

#### Structural Patterns

##### Adapter Pattern

- [ ] Class adapter vs Object adapter
- [ ] Wrapping third-party libraries
- [ ] Interface adaptation

##### Decorator Pattern

- [ ] Hiểu khác biệt với TS Decorators
- [ ] Wrapping objects để extend behavior
- [ ] Composition over inheritance

##### Facade Pattern

- [ ] Simplify complex subsystems
- [ ] Unified interface

#### Behavioral Patterns

##### Observer Pattern

- [ ] Subject và Observer interfaces
- [ ] Subscribe/Unsubscribe mechanism
- [ ] Event-driven architecture
- [ ] Pub/Sub variation
- [ ] Memory leak prevention (unsubscribe)

##### Strategy Pattern

- [ ] Define family of algorithms
- [ ] Encapsulate và make interchangeable
- [ ] Context class
- [ ] Runtime strategy switching

##### Command Pattern

- [ ] Encapsulate request as object
- [ ] Undo/Redo functionality
- [ ] Command queue

##### Repository Pattern

- [ ] Abstract data access layer
- [ ] Generic repository
- [ ] Unit of Work pattern

---

## Phần 3: Asynchronous Programming - TRỌNG TÂM

> ⏱️ Thời gian ước tính: 5-7 ngày | 🎯 **ĐÂY LÀ PHẦN TRỌNG TÂM**

### 3.1 JavaScript Runtime & Event Loop

#### Tư duy về JavaScript Engine

- [ ] Call Stack - LIFO (Last In First Out)
- [ ] Heap - Memory allocation
- [ ] Single-threaded nature của JavaScript

#### Event Loop Deep Dive

- [ ] Hiểu Event Loop cycle
- [ ] Macro Tasks (setTimeout, setInterval, I/O)
- [ ] Micro Tasks (Promise callbacks, queueMicrotask)
- [ ] Thứ tự ưu tiên: Microtask > Macrotask
- [ ] `process.nextTick()` trong Node.js (ưu tiên cao nhất)
- [ ] Visualize Event Loop để debug async issues

#### Task Queues

- [ ] Callback Queue (Task Queue)
- [ ] Microtask Queue
- [ ] Render Queue (trong browser)
- [ ] Tại sao Promise callback chạy trước setTimeout

### 3.2 Callbacks (Legacy nhưng cần hiểu)

- [ ] Callback pattern cơ bản
- [ ] Error-first callbacks (Node.js convention)
- [ ] Callback Hell / Pyramid of Doom
- [ ] Tại sao cần Promise

### 3.3 Promises

#### Promise Fundamentals

- [ ] Promise states: pending, fulfilled, rejected
- [ ] Tạo Promise với `new Promise((resolve, reject) => {})`
- [ ] `resolve(value)` và `reject(reason)`
- [ ] Promise là **immutable** sau khi settled

#### Promise Consumption

- [ ] `.then(onFulfilled, onRejected)`
- [ ] `.catch(onRejected)`
- [ ] `.finally(onFinally)`
- [ ] Chaining promises
- [ ] Return value trong `.then()` tạo Promise mới

#### Promise Static Methods

- [ ] `Promise.resolve(value)` - tạo fulfilled Promise
- [ ] `Promise.reject(reason)` - tạo rejected Promise

#### Promise Combinators (QUAN TRỌNG)

- [ ] `Promise.all([])` - tất cả phải fulfilled, fail-fast
- [ ] `Promise.allSettled([])` - chờ tất cả, không fail-fast
- [ ] `Promise.race([])` - return Promise đầu tiên settled
- [ ] `Promise.any([])` - return Promise đầu tiên fulfilled (ES2021)
- [ ] Khi nào dùng `all` vs `allSettled`
- [ ] Khi nào dùng `race` vs `any`
- [ ] Implement timeout với `Promise.race`

#### Typing Promises trong TypeScript

- [ ] `Promise<T>` generic type
- [ ] Type inference với async functions
- [ ] Union types trong Promise: `Promise<T | null>`
- [ ] `PromiseLike<T>` interface

### 3.4 Async/Await

#### Async Functions

- [ ] `async` keyword - function luôn return Promise
- [ ] Implicit wrapping trong `Promise.resolve()`
- [ ] Return type annotation: `async function(): Promise<T>`

#### Await Expression

- [ ] `await` chỉ dùng trong `async` function (hoặc top-level await)
- [ ] `await` unwrap Promise value
- [ ] `await` với non-Promise value
- [ ] Sequential vs Parallel execution

#### Parallel Execution Patterns

- [ ] Sequential await (chậm, khi cần order)
- [ ] Parallel với `Promise.all` + await
- [ ] Parallel với `Promise.allSettled` + await
- [ ] Start parallel, await separately

#### Top-Level Await

- [ ] ES Modules với top-level await
- [ ] Cấu hình `tsconfig.json` cho top-level await
- [ ] Use cases và limitations

### 3.5 Error Handling trong Async Code

#### Try/Catch với Async/Await

- [ ] Wrap `await` trong `try/catch`
- [ ] Catch specific error types
- [ ] Re-throwing errors
- [ ] Error transformation

#### Promise Error Handling

- [ ] `.catch()` placement trong chain
- [ ] Error propagation trong Promise chain
- [ ] Unhandled Promise rejections
- [ ] `process.on('unhandledRejection')` trong Node.js

#### Error Handling Patterns

- [ ] Result type pattern: `Promise<{ data: T } | { error: Error }>`
- [ ] Tuple pattern: `Promise<[Error | null, T | null]>`
- [ ] Custom Error classes extending `Error`
- [ ] Error boundaries và centralized error handling
- [ ] Graceful degradation

#### Retry Patterns

- [ ] Simple retry với loop
- [ ] Exponential backoff
- [ ] Circuit breaker pattern
- [ ] Timeout handling

### 3.6 Advanced Async Patterns

#### Async Iterators & Generators

- [ ] `async function*` - Async Generator
- [ ] `for await...of` loop
- [ ] Streaming data patterns
- [ ] Pagination với async iterators

#### Concurrency Control

- [ ] Limiting concurrent promises
- [ ] Semaphore pattern
- [ ] Queue-based processing
- [ ] Batch processing

#### Cancellation Patterns

- [ ] AbortController và AbortSignal
- [ ] Cancellable promises
- [ ] Timeout với AbortController
- [ ] Cleanup resources on cancellation

#### Real-world Async Scenarios

- [ ] Database transactions với async
- [ ] API calls với retry và timeout
- [ ] File I/O operations
- [ ] WebSocket connections
- [ ] Worker threads communication

---

## Phần 4: Generics & Advanced Types

> ⏱️ Thời gian ước tính: 2-3 ngày

### Generics Basics

- [ ] Generic functions `<T>`
- [ ] Generic interfaces
- [ ] Generic classes
- [ ] Multiple type parameters `<T, U>`
- [ ] Default type parameters `<T = string>`

### Generic Constraints

- [ ] `extends` keyword để constrain types
- [ ] `keyof` constraint
- [ ] Multiple constraints với intersection

### Conditional Types

- [ ] `T extends U ? X : Y`
- [ ] `infer` keyword
- [ ] Distributive conditional types

### Mapped Types

- [ ] `{ [K in keyof T]: ... }`
- [ ] Modifier changes (`+`, `-`, `readonly`, `?`)

### Template Literal Types

- [ ] String manipulation types
- [ ] Pattern matching với template literals

---

## Phần 5: Utility Types

> ⏱️ Thời gian ước tính: 1-2 ngày

### Transformation Utilities

- [ ] `Partial<T>` - tất cả properties optional
- [ ] `Required<T>` - tất cả properties required
- [ ] `Readonly<T>` - tất cả properties readonly
- [ ] `Record<K, V>` - object type với keys K và values V

### Picking & Omitting

- [ ] `Pick<T, K>` - chọn một số properties
- [ ] `Omit<T, K>` - loại bỏ một số properties

### Union Utilities

- [ ] `Exclude<T, U>` - loại bỏ types từ union
- [ ] `Extract<T, U>` - extract types từ union
- [ ] `NonNullable<T>` - loại bỏ null và undefined

### Function Utilities

- [ ] `Parameters<T>` - tuple type của function params
- [ ] `ReturnType<T>` - return type của function
- [ ] `ConstructorParameters<T>`
- [ ] `InstanceType<T>`

### String Utilities

- [ ] `Uppercase<S>`, `Lowercase<S>`
- [ ] `Capitalize<S>`, `Uncapitalize<S>`

### Awaited Type

- [ ] `Awaited<T>` - unwrap Promise type

---

## Phần 6: Decorators

> ⏱️ Thời gian ước tính: 1-2 ngày

### Decorator Basics

- [ ] Enable decorators trong `tsconfig.json`
- [ ] Decorator syntax (`@decorator`)
- [ ] Decorator factories

### Types of Decorators

- [ ] Class decorators
- [ ] Method decorators
- [ ] Property decorators
- [ ] Parameter decorators
- [ ] Accessor decorators

### Decorator Composition

- [ ] Multiple decorators
- [ ] Evaluation order vs Execution order

### Practical Use Cases

- [ ] Logging decorator
- [ ] Validation decorator
- [ ] Caching decorator
- [ ] Dependency injection decorator

---

## Phần 7: Module System & Project Structure

> ⏱️ Thời gian ước tính: 1 ngày

### ES Modules

- [ ] `import` / `export` syntax
- [ ] Default vs Named exports
- [ ] Re-exporting (`export * from`)
- [ ] Dynamic imports `import()`

### Module Resolution

- [ ] `moduleResolution` trong tsconfig
- [ ] Path aliases
- [ ] Barrel files (`index.ts`)

### Declaration Files

- [ ] `.d.ts` files
- [ ] `@types` packages
- [ ] Ambient declarations
- [ ] Module augmentation

### Project Organization

- [ ] Feature-based structure
- [ ] Layer-based structure (controllers, services, repositories)
- [ ] Monorepo considerations

---

## Phần 8: Real-world Application

> ⏱️ Thời gian ước tính: Ongoing

### Type-Safe Backend Patterns

- [ ] Type-safe API routes
- [ ] Type-safe database queries
- [ ] Type-safe configuration
- [ ] Type-safe environment variables

### Testing với TypeScript

- [ ] Unit testing setup
- [ ] Mocking với types
- [ ] Type-safe test utilities

### Performance & Best Practices

- [ ] Strict mode và tất cả strict flags
- [ ] `noImplicitAny`, `strictNullChecks`
- [ ] Avoiding `any` - strategies
- [ ] Type narrowing best practices

### Tooling

- [ ] ESLint với TypeScript rules
- [ ] Prettier configuration
- [ ] Pre-commit hooks
- [ ] CI/CD type checking

---

## 📊 Tracking Progress

| Phần                 | Trạng thái     | Ghi chú |
| -------------------- | -------------- | ------- |
| 1. Fundamentals      | ⬜ Not Started |         |
| 2. OOP (Trọng tâm)   | ⬜ Not Started |         |
| 3. Async (Trọng tâm) | ⬜ Not Started |         |
| 4. Generics          | ⬜ Not Started |         |
| 5. Utility Types     | ⬜ Not Started |         |
| 6. Decorators        | ⬜ Not Started |         |
| 7. Module System     | ⬜ Not Started |         |
| 8. Real-world        | ⬜ Not Started |         |

**Trạng thái:** ⬜ Not Started | 🟡 In Progress | ✅ Completed

---

## 📚 Tài liệu tham khảo

- [TypeScript Official Handbook](https://www.typescriptlang.org/docs/handbook/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [JavaScript Event Loop Visualized](https://www.jsv9000.app/)

---

> 💡 **Tip:** Không cần học theo thứ tự tuyến tính. Sau khi nắm Fundamentals, bạn có thể học song song OOP và Async Programming.

> ⚠️ **Lưu ý:** Checklist này tập trung vào **tư duy** và **concepts**. Hãy tự thực hành code cho mỗi checkbox để thực sự hiểu sâu.

---

_Được tạo bởi Senior TypeScript Engineer - Cập nhật: January 2026_
