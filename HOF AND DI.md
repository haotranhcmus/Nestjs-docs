# Dependency Injection và Higher Order Functions

## Mục lục
- [Dependency Injection](#dependency-injection)
  - [Khái niệm](#khái-niệm-dependency-injection)
  - [Lợi ích](#lợi-ích-của-dependency-injection)
  - [Các loại Dependency Injection](#các-loại-dependency-injection)
  - [Ví dụ trong TypeScript](#ví-dụ-dependency-injection-trong-typescript)
- [Higher Order Functions](#higher-order-functions)
  - [Khái niệm](#khái-niệm-higher-order-functions)
  - [Đặc điểm](#đặc-điểm)
  - [Ví dụ phổ biến](#ví-dụ-higher-order-functions)
  - [Ứng dụng thực tế](#ứng-dụng-thực-tế)

---

## Dependency Injection

### Khái niệm Dependency Injection

**Dependency Injection (DI)** là một design pattern trong lập trình hướng đối tượng, cho phép các dependencies (phụ thuộc) được "tiêm" vào một class từ bên ngoài thay vì class tự tạo ra chúng.

**Nguyên tắc cơ bản:**
- Một class không nên tự tạo ra các dependencies của nó
- Dependencies được cung cấp từ bên ngoài (injected)
- Giúp giảm sự phụ thuộc chặt chẽ (tight coupling) giữa các module

### Lợi ích của Dependency Injection

1. **Tăng tính module hóa**: Các component độc lập, dễ tái sử dụng
2. **Dễ test**: Có thể mock/stub dependencies khi unit testing
3. **Loose coupling**: Giảm sự phụ thuộc trực tiếp giữa các class
4. **Dễ bảo trì**: Thay đổi implementation không ảnh hưởng đến code sử dụng
5. **Tuân thủ SOLID principles**: Đặc biệt là Dependency Inversion Principle

### Các loại Dependency Injection

#### 1. Constructor Injection
Dependencies được truyền qua constructor của class.

```typescript
interface ILogger {
  log(message: string): void;
}

class ConsoleLogger implements ILogger {
  log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }
}

class UserService {
  private logger: ILogger;

  // Inject dependency qua constructor
  constructor(logger: ILogger) {
    this.logger = logger;
  }

  createUser(name: string): void {
    // Logic tạo user
    this.logger.log(`User ${name} created`);
  }
}

// Sử dụng
const logger = new ConsoleLogger();
const userService = new UserService(logger);
userService.createUser('John');
```

#### 2. Property Injection
Dependencies được gán trực tiếp vào property của class.

```typescript
class UserService {
  public logger!: ILogger;

  createUser(name: string): void {
    this.logger.log(`User ${name} created`);
  }
}

// Sử dụng
const userService = new UserService();
userService.logger = new ConsoleLogger();
userService.createUser('Jane');
```

#### 3. Method Injection
Dependencies được truyền qua method parameter.

```typescript
class UserService {
  createUser(name: string, logger: ILogger): void {
    // Logic tạo user
    logger.log(`User ${name} created`);
  }
}

// Sử dụng
const userService = new UserService();
userService.createUser('Bob', new ConsoleLogger());
```

### Ví dụ Dependency Injection trong TypeScript

#### Ví dụ hoàn chỉnh với Interface

```typescript
// Định nghĩa interfaces
interface IDatabase {
  save(data: any): Promise<void>;
  find(id: string): Promise<any>;
}

interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// Implementations
class MongoDatabase implements IDatabase {
  async save(data: any): Promise<void> {
    console.log('Saving to MongoDB:', data);
  }

  async find(id: string): Promise<any> {
    console.log('Finding in MongoDB:', id);
    return { id, name: 'Sample Data' };
  }
}

class SMTPEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    console.log(`Sending email to ${to}: ${subject}`);
  }
}

// Service sử dụng DI
class UserRegistrationService {
  constructor(
    private database: IDatabase,
    private emailService: IEmailService
  ) {}

  async registerUser(email: string, name: string): Promise<void> {
    // Lưu user vào database
    await this.database.save({ email, name });

    // Gửi email chào mừng
    await this.emailService.sendEmail(
      email,
      'Welcome!',
      `Hello ${name}, welcome to our platform!`
    );
  }
}

// DI Container (đơn giản)
class DIContainer {
  private services: Map<string, any> = new Map();

  register<T>(name: string, instance: T): void {
    this.services.set(name, instance);
  }

  resolve<T>(name: string): T {
    return this.services.get(name);
  }
}

// Sử dụng
const container = new DIContainer();
container.register('database', new MongoDatabase());
container.register('emailService', new SMTPEmailService());

const registrationService = new UserRegistrationService(
  container.resolve('database'),
  container.resolve('emailService')
);

registrationService.registerUser('user@example.com', 'John Doe');
```

#### Ví dụ với Decorator (TypeScript)

```typescript
// Decorator để đánh dấu injectable class
function Injectable() {
  return function (target: any) {
    target.injectable = true;
  };
}

@Injectable()
class ApiService {
  fetchData(): string {
    return 'Data from API';
  }
}

@Injectable()
class DataProcessor {
  constructor(private apiService: ApiService) {}

  process(): void {
    const data = this.apiService.fetchData();
    console.log('Processing:', data);
  }
}
```

---

## Higher Order Functions

### Khái niệm Higher Order Functions

**Higher Order Function (HOF)** là một hàm thỏa mãn ít nhất một trong hai điều kiện:
1. Nhận một hoặc nhiều hàm làm tham số (arguments)
2. Trả về một hàm làm kết quả

### Đặc điểm

- **First-class functions**: Trong JavaScript/TypeScript, hàm là first-class citizen, có thể:
  - Gán vào biến
  - Truyền làm argument
  - Trả về từ hàm khác
  - Lưu trong data structures

- **Tính linh hoạt cao**: Cho phép tạo ra các hàm động, tái sử dụng logic
- **Functional Programming**: HOF là nền tảng của lập trình hàm

### Ví dụ Higher Order Functions

#### 1. Built-in Higher Order Functions

```typescript
// Array.map() - nhận function làm argument
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((num) => num * 2);
// [2, 4, 6, 8, 10]

// Array.filter() - nhận function làm argument
const evenNumbers = numbers.filter((num) => num % 2 === 0);
// [2, 4]

// Array.reduce() - nhận function làm argument
const sum = numbers.reduce((acc, num) => acc + num, 0);
// 15

// Array.forEach() - nhận function làm argument
numbers.forEach((num) => console.log(num));
```

#### 2. Custom Higher Order Functions

##### Hàm nhận hàm làm tham số

```typescript
// HOF nhận callback function
function repeat(n: number, action: (index: number) => void): void {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

// Sử dụng
repeat(3, (i) => console.log(`Iteration ${i}`));
// Output:
// Iteration 0
// Iteration 1
// Iteration 2

// HOF để đo thời gian thực thi
function measureTime<T>(fn: () => T): T {
  const start = Date.now();
  const result = fn();
  const end = Date.now();
  console.log(`Execution time: ${end - start}ms`);
  return result;
}

// Sử dụng
const result = measureTime(() => {
  let sum = 0;
  for (let i = 0; i < 1000000; i++) {
    sum += i;
  }
  return sum;
});
```

##### Hàm trả về hàm

```typescript
// HOF trả về function
function multiply(factor: number): (num: number) => number {
  return (num: number) => num * factor;
}

// Sử dụng
const double = multiply(2);
const triple = multiply(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// HOF tạo greeting function
function createGreeting(greeting: string): (name: string) => string {
  return (name: string) => `${greeting}, ${name}!`;
}

const sayHello = createGreeting('Hello');
const sayHi = createGreeting('Hi');

console.log(sayHello('Alice'));  // "Hello, Alice!"
console.log(sayHi('Bob'));       // "Hi, Bob!"
```

#### 3. Function Composition

```typescript
// Compose functions - kết hợp nhiều hàm
function compose<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduceRight((result, fn) => fn(result), arg);
}

// Các hàm đơn giản
const addOne = (x: number) => x + 1;
const double = (x: number) => x * 2;
const square = (x: number) => x * x;

// Kết hợp các hàm
const combined = compose(square, double, addOne);
console.log(combined(3));  // square(double(addOne(3))) = square(8) = 64

// Pipe - ngược lại với compose
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduce((result, fn) => fn(result), arg);
}

const piped = pipe(addOne, double, square);
console.log(piped(3));  // square(double(addOne(3))) = square(8) = 64
```

#### 4. Currying

```typescript
// Currying - chuyển đổi hàm nhiều tham số thành chuỗi hàm
function curry<A, B, C>(fn: (a: A, b: B) => C): (a: A) => (b: B) => C {
  return (a: A) => (b: B) => fn(a, b);
}

// Hàm gốc
const add = (a: number, b: number) => a + b;

// Curried version
const curriedAdd = curry(add);
const add5 = curriedAdd(5);

console.log(add5(3));   // 8
console.log(add5(10));  // 15

// Currying với nhiều tham số
function curriedMultiply(a: number): (b: number) => (c: number) => number {
  return (b: number) => (c: number) => a * b * c;
}

const result1 = curriedMultiply(2)(3)(4);  // 24
```

### Ứng dụng thực tế

#### 1. Middleware Pattern

```typescript
type Middleware<T> = (data: T, next: (data: T) => T) => T;

function createMiddlewareChain<T>(...middlewares: Middleware<T>[]) {
  return (initialData: T): T => {
    let index = 0;

    function next(data: T): T {
      if (index >= middlewares.length) {
        return data;
      }
      const middleware = middlewares[index++];
      return middleware(data, next);
    }

    return next(initialData);
  };
}

// Các middleware
const logger: Middleware<any> = (data, next) => {
  console.log('Logging:', data);
  return next(data);
};

const validator: Middleware<any> = (data, next) => {
  if (!data) {
    throw new Error('Invalid data');
  }
  return next(data);
};

const transformer: Middleware<any> = (data, next) => {
  return next({ ...data, transformed: true });
};

// Sử dụng
const chain = createMiddlewareChain(logger, validator, transformer);
const result2 = chain({ value: 42 });
```

#### 2. Memoization

```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, ReturnType<T>>();

  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('Returning cached result');
      return cache.get(key)!;
    }

    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

// Sử dụng
const fibonacci = memoize((n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(40));  // Nhanh hơn nhiều nhờ memoization
```

#### 3. Debounce và Throttle

```typescript
// Debounce - chỉ gọi function sau khi user ngừng action
function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: NodeJS.Timeout;

  return (...args: Parameters<T>) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

// Throttle - giới hạn số lần gọi function trong khoảng thời gian
function throttle<T extends (...args: any[]) => any>(
  fn: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;

  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Sử dụng
const searchAPI = debounce((query: string) => {
  console.log('Searching for:', query);
}, 300);

const handleScroll = throttle(() => {
  console.log('Scroll event handled');
}, 100);
```

#### 4. Event Handling

```typescript
// HOF để tạo event handlers
function createEventHandler<T>(
  handler: (event: T) => void,
  options?: {
    preventDefault?: boolean;
    stopPropagation?: boolean;
  }
): (event: T) => void {
  return (event: any) => {
    if (options?.preventDefault) {
      event.preventDefault();
    }
    if (options?.stopPropagation) {
      event.stopPropagation();
    }
    handler(event);
  };
}

// Sử dụng
const handleClick = createEventHandler(
  (event) => console.log('Clicked!'),
  { preventDefault: true }
);
```

---

## Kết hợp Dependency Injection và Higher Order Functions

```typescript
// Kết hợp DI và HOF
interface ICache {
  get(key: string): any;
  set(key: string, value: any): void;
}

class MemoryCache implements ICache {
  private cache = new Map();

  get(key: string): any {
    return this.cache.get(key);
  }

  set(key: string, value: any): void {
    this.cache.set(key, value);
  }
}

// HOF sử dụng DI
function createCachedFunction<T extends (...args: any[]) => any>(
  fn: T,
  cache: ICache
): T {
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    const cached = cache.get(key);

    if (cached !== undefined) {
      return cached;
    }

    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

// Sử dụng
const cache = new MemoryCache();
const expensiveOperation = (n: number) => {
  console.log('Computing...');
  return n * n;
};

const cached = createCachedFunction(expensiveOperation, cache);
console.log(cached(5));  // Computing... 25
console.log(cached(5));  // 25 (from cache)
```

---

## Tổng kết

### Dependency Injection
- **Ưu điểm**: Loose coupling, dễ test, dễ bảo trì
- **Khi nào dùng**: Khi có nhiều dependencies phức tạp, cần test, hoặc cần thay đổi implementation
- **Best practices**: Dùng interfaces, prefer constructor injection, sử dụng DI containers cho ứng dụng lớn

### Higher Order Functions
- **Ưu điểm**: Code ngắn gọn, tái sử dụng cao, functional programming
- **Khi nào dùng**: Xử lý collections, tạo utilities, middleware, event handling
- **Best practices**: Giữ functions pure khi có thể, tránh side effects, sử dụng TypeScript generics

Cả hai pattern đều là công cụ mạnh mẽ trong TypeScript/JavaScript hiện đại và thường được sử dụng cùng nhau để tạo ra code sạch, maintainable và scalable.
