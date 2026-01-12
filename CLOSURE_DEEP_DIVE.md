# Closure trong TypeScript: Deep Dive

> **Mục tiêu:** Hiểu rõ cơ chế hoạt động của Closure từ góc độ JavaScript Engine và ứng dụng thực tế trong TypeScript.
>
> **Đối tượng:** Developer đã nắm Higher Order Functions và Dependency Injection, cần hiểu bản chất "Under the hood".

---

## Mục lục

1. [Cơ chế hoạt động - The Internal](#1-cơ-chế-hoạt-động---the-internal)
   - [1.1 Lexical Scope & Execution Context](#11-lexical-scope--execution-context)
   - [1.2 Closure = Function + Lexical Environment](#12-closure--function--lexical-environment)
   - [1.3 Memory Model: Tại sao biến không bị Garbage Collect?](#13-memory-model-tại-sao-biến-không-bị-garbage-collect)
2. [TypeScript & Typing](#2-typescript--typing)
   - [2.1 Type Annotations cho Closure](#21-type-annotations-cho-closure)
   - [2.2 Type Inference và Generic Closures](#22-type-inference-và-generic-closures)
3. [Design Patterns với Closure](#3-design-patterns-với-closure)
   - [3.1 Data Encapsulation (Private Variables)](#31-data-encapsulation-private-variables)
   - [3.2 Function Factories & Currying](#32-function-factories--currying)
   - [3.3 Memoization](#33-memoization)
   - [3.4 Module Pattern](#34-module-pattern)
4. [Pitfalls & Performance](#4-pitfalls--performance)
   - [4.1 Memory Leaks](#41-memory-leaks)
   - [4.2 Stale Closures](#42-stale-closures)
   - [4.3 Performance Considerations](#43-performance-considerations)
5. [Closure vs Class](#5-closure-vs-class)

---

## 1. Cơ chế hoạt động - The Internal

### 1.1 Lexical Scope & Execution Context

#### Lexical Scope (Static Scope)

**Định nghĩa kỹ thuật:**
Lexical scope là cơ chế xác định phạm vi truy cập biến dựa trên **vị trí khai báo** trong source code, **không phải** thời điểm runtime.

**Quy tắc:**

- Hàm con có thể truy cập biến của hàm cha
- Scope được xác định lúc **parse time** (khi code được đọc)
- Scope chain được tạo theo cấu trúc lồng nhau của code

**Scope Chain:**

```typescript
const global = 'global scope'

function outer(): void {
  const outerVar = 'outer scope'

  function middle(): void {
    const middleVar = 'middle scope'

    function inner(): void {
      const innerVar = 'inner scope'

      // Scope chain: inner -> middle -> outer -> global
      console.log(innerVar) // ✅ Tìm trong inner scope
      console.log(middleVar) // ✅ Tìm trong middle scope (parent)
      console.log(outerVar) // ✅ Tìm trong outer scope (grandparent)
      console.log(global) // ✅ Tìm trong global scope
    }

    inner()
  }

  middle()
}
```

**Variable Resolution:**

1. Tìm trong scope hiện tại
2. Không thấy → tìm parent scope
3. Lặp lại cho đến global scope
4. Không thấy → ReferenceError

#### Execution Context

**Execution Context** là môi trường runtime chứa:

**1. Variable Environment:**

- Tất cả biến và function declarations trong scope
- `this` binding
- `arguments` object (trong function)

**2. Lexical Environment:**

- Tham chiếu đến outer lexical environment (parent scope)
- Tạo scope chain

**3. ThisBinding:**

- Giá trị của `this`

**Execution Context Stack (Call Stack):**

```typescript
function first(): void {
  const a = 1
  second()
}

function second(): void {
  const b = 2
  third()
}

function third(): void {
  const c = 3
}

first()

// Call Stack:
// ┌─────────────────┐
// │   third()       │  ← Top (đang chạy)
// ├─────────────────┤
// │   second()      │
// ├─────────────────┤
// │   first()       │
// ├─────────────────┤
// │   global        │  ← Bottom
// └─────────────────┘
```

**Khi function return:**

- Execution context bị **pop** khỏi call stack
- Variable environment **thường** bị garbage collect
- **NHƯNG** nếu có closure → giữ lại reference → **không** bị GC

---

### 1.2 Closure = Function + Lexical Environment

#### Định nghĩa chính xác

**Closure** không chỉ là một function. Closure là:

> **Function object** + **Reference** đến lexical environment nơi function được tạo ra.

**Ẩn dụ: Chiếc ba lô (Backpack)**

Khi một function được tạo ra bên trong function khác:

- Function đó được "đóng gói" vào một chiếc ba lô
- Ba lô chứa **snapshot** của tất cả biến mà function cần từ outer scope
- Khi function được return hoặc truyền đi, nó **mang theo ba lô**
- Mở ba lô bất cứ lúc nào để lấy biến ra dùng

**Code minh họa:**

```typescript
function createCounter(): () => number {
  let count = 0 // Biến này nằm trong "ba lô"

  return function increment(): number {
    count++ // Function mang theo "ba lô" chứa count
    return count
  }
}

const counter = createCounter()
// counter function giờ mang theo ba lô chứa count = 0

console.log(counter()) // 1 (lấy count từ ba lô, tăng lên)
console.log(counter()) // 2 (cùng ba lô, count = 1 → 2)
console.log(counter()) // 3

const counter2 = createCounter()
// counter2 có BA LÔ MỚI, count riêng = 0
console.log(counter2()) // 1
```

#### Internal Representation

**JavaScript Engine (V8) lưu closure như thế nào?**

```typescript
function outer(x: number): () => number {
  let y = 10

  return function inner(): number {
    return x + y
  }
}

const fn = outer(5)
```

**Trong memory:**

```
fn (Function Object)
├── [[Code]]: function body của inner()
├── [[Scope]]: Reference đến Closure Scope
│   └── Closure Scope Object:
│       ├── x: 5
│       └── y: 10
└── [[Prototype]]: Function.prototype
```

**Key points:**

- `inner` function **không copy** giá trị x, y
- `inner` giữ **reference** đến scope object
- Scope object chứa x, y tồn tại trên **Heap**
- Khi gọi `fn()`, engine lookup biến qua reference này

---

### 1.3 Memory Model: Tại sao biến không bị Garbage Collect?

#### Garbage Collection 101

**Mark-and-Sweep Algorithm:**

1. **Mark Phase:**
   - Từ GC roots (global, call stack), mark tất cả objects có thể reach được
   - Objects không được mark = unreachable = garbage

2. **Sweep Phase:**
   - Thu hồi memory của objects không được mark

**GC Roots:**

- Global variables
- Local variables trong active call stack
- Closures

#### Closure và GC

**Scenario 1: Không có closure - biến bị GC**

```typescript
function normalFunction(): void {
  let tempData = new Array(1000000) // Large array
  console.log(tempData.length)
}

normalFunction()
// ✅ normalFunction() return
// ✅ tempData không còn reference
// ✅ GC thu hồi memory
```

**Scenario 2: Có closure - biến KHÔNG bị GC**

```typescript
function createClosure(): () => void {
  let persistentData = new Array(1000000) // Large array

  return function (): void {
    console.log(persistentData.length)
  }
}

const fn = createClosure()
// ❌ createClosure() đã return
// ❌ NHƯNG persistentData vẫn tồn tại
// ❌ Vì fn closure giữ reference đến persistentData
// ❌ GC KHÔNG thu hồi được

// Sau này, khi:
fn = null // ✅ Giờ mới GC được persistentData
```

**Tại sao?**

```
GC Root (fn variable)
    ↓
fn (Function Object)
    ↓ [[Scope]]
Closure Scope Object
    ↓
persistentData (Array)

→ persistentData có thể reach từ GC root
→ Không bị GC
```

#### Optimization: Engine chỉ giữ biến được dùng

**Modern JS Engines thông minh:**

```typescript
function createClosure(): () => number {
  let used = 10
  let unused = new Array(1000000) // Không được dùng trong closure

  return function (): number {
    return used // Chỉ dùng 'used'
  }
}

const fn = createClosure()
// ✅ Engine detect 'unused' không được reference trong closure
// ✅ GC có thể thu hồi 'unused'
// ✅ Chỉ 'used' được giữ lại
```

**Scope object thực tế chỉ chứa:**

```
Closure Scope:
├── used: 10
└── (unused đã bị GC)
```

---

## 2. TypeScript & Typing

### 2.1 Type Annotations cho Closure

#### Function Type Syntax

**Basic closure type:**

```typescript
// Type cho function trả về closure
type CounterFactory = () => () => number

const createCounter: CounterFactory = () => {
  let count = 0
  return () => ++count
}
```

**Explicit parameter và return types:**

```typescript
type IncrementFn = () => number
type CounterFactory = () => IncrementFn

function createCounter(): IncrementFn {
  let count: number = 0

  const increment: IncrementFn = (): number => {
    return ++count
  }

  return increment
}
```

#### Complex Closure Types

**Closure với parameters:**

```typescript
type Adder = (b: number) => number
type AdderFactory = (a: number) => Adder

const createAdder: AdderFactory = (a: number): Adder => {
  return (b: number): number => a + b
}

const add5: Adder = createAdder(5)
console.log(add5(10)) // 15
```

**Closure trả về object:**

```typescript
interface Counter {
  increment: () => number
  decrement: () => number
  getValue: () => number
  reset: () => void
}

type CounterFactory = (initialValue?: number) => Counter

const createCounter: CounterFactory = (initialValue = 0): Counter => {
  let count = initialValue

  return {
    increment: (): number => ++count,
    decrement: (): number => --count,
    getValue: (): number => count,
    reset: (): void => {
      count = initialValue
    }
  }
}
```

#### Function Signatures với Generics

```typescript
type Predicate<T> = (value: T) => boolean
type FilterFactory<T> = (predicate: Predicate<T>) => (items: T[]) => T[]

function createFilter<T>(): FilterFactory<T> {
  return (predicate: Predicate<T>) => {
    return (items: T[]): T[] => {
      return items.filter(predicate)
    }
  }
}

const numberFilter = createFilter<number>()
const greaterThan5 = numberFilter((n) => n > 5)
console.log(greaterThan5([1, 3, 6, 8, 2])) // [6, 8]
```

---

### 2.2 Type Inference và Generic Closures

#### Type Inference tự động

TypeScript inference types từ closure:

```typescript
function multiplier(factor: number) {
  // Return type inferred: (n: number) => number
  return (n: number) => n * factor
}

const double = multiplier(2)
// Type of double: (n: number) => number
```

#### Generic Closures

**Memoization với Generic:**

```typescript
type MemoizedFn<T extends any[], R> = (...args: T) => R

function memoize<T extends any[], R>(fn: (...args: T) => R): MemoizedFn<T, R> {
  const cache = new Map<string, R>()

  return (...args: T): R => {
    const key = JSON.stringify(args)

    if (cache.has(key)) {
      return cache.get(key)!
    }

    const result = fn(...args)
    cache.set(key, result)
    return result
  }
}

// Usage với type inference
const add = (a: number, b: number): number => a + b
const memoizedAdd = memoize(add)
// Type: (a: number, b: number) => number

console.log(memoizedAdd(2, 3)) // 5 (computed)
console.log(memoizedAdd(2, 3)) // 5 (cached)
```

**Factory với constraints:**

```typescript
type Validator<T> = (value: unknown) => value is T

function createValidator<T>(check: (value: unknown) => boolean): Validator<T> {
  return (value: unknown): value is T => {
    return check(value)
  }
}

// Type guard closure
const isString = createValidator<string>((v) => typeof v === 'string')
const isNumber = createValidator<number>((v) => typeof v === 'number')

function process(value: unknown): void {
  if (isString(value)) {
    console.log(value.toUpperCase()) // value: string
  } else if (isNumber(value)) {
    console.log(value.toFixed(2)) // value: number
  }
}
```

---

## 3. Design Patterns với Closure

### 3.1 Data Encapsulation (Private Variables)

**Vấn đề:** Trước ES2022 `#private`, không có private thực sự trong JS.

**Solution:** Closure tạo truly private variables.

#### Counter với Private State

```typescript
interface Counter {
  increment: () => number
  decrement: () => number
  getValue: () => number
}

function createCounter(initialValue: number = 0): Counter {
  // Private variable - KHÔNG THỂ access từ bên ngoài
  let count = initialValue

  // Chỉ expose methods, không expose data
  return {
    increment(): number {
      return ++count
    },

    decrement(): number {
      return --count
    },

    getValue(): number {
      return count
    }
  }
}

const counter = createCounter(10)
console.log(counter.getValue()) // 10
counter.increment()
console.log(counter.getValue()) // 11

// ❌ Không thể access count trực tiếp
// console.log(counter.count); // undefined
// counter.count = 999; // Không có effect
```

#### Bank Account với Validation

```typescript
interface BankAccount {
  deposit: (amount: number) => void
  withdraw: (amount: number) => boolean
  getBalance: () => number
  getTransactionHistory: () => readonly string[]
}

function createBankAccount(initialBalance: number): BankAccount {
  // Private state
  let balance = initialBalance
  const transactions: string[] = []

  // Private helper
  const addTransaction = (type: string, amount: number): void => {
    const timestamp = new Date().toISOString()
    transactions.push(`[${timestamp}] ${type}: $${amount}`)
  }

  return {
    deposit(amount: number): void {
      if (amount <= 0) {
        throw new Error('Amount must be positive')
      }
      balance += amount
      addTransaction('DEPOSIT', amount)
    },

    withdraw(amount: number): boolean {
      if (amount <= 0 || amount > balance) {
        return false
      }
      balance -= amount
      addTransaction('WITHDRAW', amount)
      return true
    },

    getBalance(): number {
      return balance
    },

    getTransactionHistory(): readonly string[] {
      return [...transactions] // Return copy, không expose mutable array
    }
  }
}

const account = createBankAccount(1000)
account.deposit(500)
account.withdraw(200)
console.log(account.getBalance()) // 1300
// ❌ Không thể hack balance
// account.balance = 999999; // Không work
```

---

### 3.2 Function Factories & Currying

#### Logger Factory

```typescript
type LogLevel = 'DEBUG' | 'INFO' | 'WARN' | 'ERROR'

interface Logger {
  (message: string): void
}

function createLogger(prefix: string, level: LogLevel = 'INFO'): Logger {
  const timestamp = (): string => new Date().toISOString()

  return (message: string): void => {
    console.log(`[${timestamp()}] [${level}] [${prefix}] ${message}`)
  }
}

// Specialized loggers
const userLogger = createLogger('UserService', 'INFO')
const dbLogger = createLogger('Database', 'DEBUG')

userLogger('User created')
// [2026-01-12T10:30:00.000Z] [INFO] [UserService] User created

dbLogger('Query executed')
// [2026-01-12T10:30:00.000Z] [DEBUG] [Database] Query executed
```

#### Validator Factory

```typescript
type ValidationRule<T> = (value: T) => boolean
type Validator<T> = (value: T) => { valid: boolean; errors: string[] }

function createValidator<T>(rules: Record<string, ValidationRule<T>>): Validator<T> {
  return (value: T): { valid: boolean; errors: string[] } => {
    const errors: string[] = []

    for (const [ruleName, rule] of Object.entries(rules)) {
      if (!rule(value)) {
        errors.push(ruleName)
      }
    }

    return {
      valid: errors.length === 0,
      errors
    }
  }
}

// Email validator
const validateEmail = createValidator<string>({
  required: (email) => email.length > 0,
  format: (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
  maxLength: (email) => email.length <= 100
})

console.log(validateEmail('test@example.com'))
// { valid: true, errors: [] }

console.log(validateEmail('invalid'))
// { valid: false, errors: ['format'] }
```

#### Currying với Type Safety

```typescript
type CurriedFn2<A, B, R> = (a: A) => (b: B) => R
type CurriedFn3<A, B, C, R> = (a: A) => (b: B) => (c: C) => R

const curriedAdd: CurriedFn2<number, number, number> = (a: number) => (b: number) => a + b

const add5 = curriedAdd(5)
console.log(add5(10)) // 15

// Generic curry implementation
function curry2<A, B, R>(fn: (a: A, b: B) => R): CurriedFn2<A, B, R> {
  return (a: A) => (b: B) => fn(a, b)
}

const multiply = (a: number, b: number): number => a * b
const curriedMultiply = curry2(multiply)
const double = curriedMultiply(2)

console.log(double(5)) // 10
```

---

### 3.3 Memoization

#### Basic Memoization

```typescript
type MemoCache<T> = Map<string, T>

function memoize<Args extends any[], Result>(fn: (...args: Args) => Result): (...args: Args) => Result {
  const cache: MemoCache<Result> = new Map()

  return (...args: Args): Result => {
    const key = JSON.stringify(args)

    if (cache.has(key)) {
      console.log('Cache hit')
      return cache.get(key)!
    }

    console.log('Cache miss - computing...')
    const result = fn(...args)
    cache.set(key, result)
    return result
  }
}

// Fibonacci với memoization
const fibonacci = memoize((n: number): number => {
  if (n <= 1) return n
  return fibonacci(n - 1) + fibonacci(n - 2)
})

console.log(fibonacci(40)) // Fast! (với memoization)
console.log(fibonacci(40)) // Instant! (từ cache)
```

#### Advanced Memoization với TTL

```typescript
interface CacheEntry<T> {
  value: T
  timestamp: number
}

function memoizeWithTTL<Args extends any[], Result>(
  fn: (...args: Args) => Result,
  ttlMs: number = 60000 // 1 minute default
): (...args: Args) => Result {
  const cache = new Map<string, CacheEntry<Result>>()

  return (...args: Args): Result => {
    const key = JSON.stringify(args)
    const now = Date.now()

    const cached = cache.get(key)
    if (cached && now - cached.timestamp < ttlMs) {
      console.log('Cache hit')
      return cached.value
    }

    console.log('Cache miss or expired')
    const result = fn(...args)
    cache.set(key, { value: result, timestamp: now })

    return result
  }
}

// Expensive API call
const fetchUserData = memoizeWithTTL(
  async (userId: number): Promise<string> => {
    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return `User data for ${userId}`
  },
  5000 // 5 second TTL
)
```

#### Memoization với LRU Cache

```typescript
class LRUCache<K, V> {
  private cache = new Map<K, V>()

  constructor(private capacity: number) {}

  get(key: K): V | undefined {
    if (!this.cache.has(key)) return undefined

    // Move to end (most recently used)
    const value = this.cache.get(key)!
    this.cache.delete(key)
    this.cache.set(key, value)
    return value
  }

  set(key: K, value: V): void {
    // Delete if exists (to re-add at end)
    if (this.cache.has(key)) {
      this.cache.delete(key)
    }

    // Evict least recently used if at capacity
    if (this.cache.size >= this.capacity) {
      const firstKey = this.cache.keys().next().value
      this.cache.delete(firstKey)
    }

    this.cache.set(key, value)
  }
}

function memoizeLRU<Args extends any[], Result>(
  fn: (...args: Args) => Result,
  capacity: number = 100
): (...args: Args) => Result {
  const cache = new LRUCache<string, Result>(capacity)

  return (...args: Args): Result => {
    const key = JSON.stringify(args)
    const cached = cache.get(key)

    if (cached !== undefined) {
      return cached
    }

    const result = fn(...args)
    cache.set(key, result)
    return result
  }
}
```

---

### 3.4 Module Pattern

#### Revealing Module Pattern

```typescript
interface UserModule {
  createUser: (name: string, email: string) => void
  getUser: (id: number) => { id: number; name: string; email: string } | undefined
  getAllUsers: () => ReadonlyArray<{ id: number; name: string; email: string }>
  deleteUser: (id: number) => boolean
}

const UserModule: UserModule = (() => {
  // Private state
  const users: Map<number, { id: number; name: string; email: string }> = new Map()
  let nextId = 1

  // Private helpers
  const validateEmail = (email: string): boolean => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }

  const log = (action: string, userId?: number): void => {
    console.log(`[UserModule] ${action}`, userId ? `User ID: ${userId}` : '')
  }

  // Public API
  return {
    createUser(name: string, email: string): void {
      if (!validateEmail(email)) {
        throw new Error('Invalid email')
      }

      const user = { id: nextId++, name, email }
      users.set(user.id, user)
      log('Created', user.id)
    },

    getUser(id: number) {
      return users.get(id)
    },

    getAllUsers() {
      return Array.from(users.values())
    },

    deleteUser(id: number): boolean {
      const deleted = users.delete(id)
      if (deleted) log('Deleted', id)
      return deleted
    }
  }
})()

// Usage
UserModule.createUser('Alice', 'alice@example.com')
UserModule.createUser('Bob', 'bob@example.com')
console.log(UserModule.getAllUsers())
// ❌ Không thể access users, nextId, validateEmail từ bên ngoài
```

#### Singleton with Lazy Initialization

```typescript
interface Database {
  query: (sql: string) => Promise<any[]>
  close: () => void
}

const DatabaseSingleton = (() => {
  let instance: Database | null = null
  let connectionCount = 0

  const createConnection = (): Database => {
    console.log('Creating new database connection...')
    connectionCount++

    return {
      async query(sql: string): Promise<any[]> {
        console.log(`Executing: ${sql}`)
        return []
      },

      close(): void {
        console.log('Closing connection...')
        instance = null
      }
    }
  }

  return {
    getInstance(): Database {
      if (!instance) {
        instance = createConnection()
      }
      return instance
    },

    getConnectionCount(): number {
      return connectionCount
    }
  }
})()

// Always returns same instance
const db1 = DatabaseSingleton.getInstance()
const db2 = DatabaseSingleton.getInstance()
console.log(db1 === db2) // true
```

---

## 4. Pitfalls & Performance

### 4.1 Memory Leaks

#### Problem: Unintentional References

**Leak scenario:**

```typescript
// ❌ Memory leak
function createHeavyProcessor() {
  const hugeArray = new Array(1000000).fill('data')

  // Closure giữ reference đến TOÀN BỘ hugeArray
  // Ngay cả khi chỉ dùng length
  return () => {
    console.log(`Array has ${hugeArray.length} items`)
  }
}

const processor = createHeavyProcessor()
// hugeArray (vài MB) vẫn trong memory
// Chỉ vì closure reference nó
```

**Solution: Extract only needed data**

```typescript
// ✅ No leak
function createHeavyProcessor() {
  const hugeArray = new Array(1000000).fill('data')
  const length = hugeArray.length // Extract value

  // hugeArray có thể bị GC sau đây
  return () => {
    console.log(`Array had ${length} items`)
  }
}
```

#### Problem: Event Listeners

**Leak scenario:**

```typescript
// ❌ Memory leak
class Component {
  private data = new Array(1000000)

  attachListener(): void {
    document.addEventListener('click', () => {
      console.log(this.data.length)
      // Closure giữ reference đến 'this'
      // → Giữ toàn bộ Component instance
      // Ngay cả khi component bị destroy
    })
  }
}

const component = new Component()
component.attachListener()
// component = null; // Component vẫn không bị GC!
```

**Solution: Cleanup listeners**

```typescript
// ✅ Proper cleanup
class Component {
  private data = new Array(1000000)
  private listener: (() => void) | null = null

  attachListener(): void {
    this.listener = () => {
      console.log(this.data.length)
    }
    document.addEventListener('click', this.listener)
  }

  destroy(): void {
    if (this.listener) {
      document.removeEventListener('click', this.listener)
      this.listener = null
    }
  }
}
```

#### Problem: Timers

```typescript
// ❌ Memory leak
function startPolling() {
  const cache = new Map() // Large cache

  setInterval(() => {
    // Update cache
    cache.set(Date.now(), 'data')
  }, 1000)

  // Interval không bao giờ dừng
  // cache không bao giờ bị GC
}
```

**Solution: Clear timers**

```typescript
// ✅ Proper cleanup
function startPolling(): () => void {
  const cache = new Map()

  const intervalId = setInterval(() => {
    cache.set(Date.now(), 'data')
  }, 1000)

  // Return cleanup function
  return () => {
    clearInterval(intervalId)
    cache.clear()
  }
}

const stopPolling = startPolling()
// Khi cần dừng:
stopPolling()
```

---

### 4.2 Stale Closures

#### Problem: Loop với var

**Classic pitfall:**

```typescript
// ❌ Stale closure
function createHandlers() {
  const handlers: (() => void)[] = []

  for (var i = 0; i < 3; i++) {
    handlers.push(() => {
      console.log(i) // Closure capture 'i' reference
    })
  }

  return handlers
}

const handlers = createHandlers()
handlers[0]() // 3 (not 0!)
handlers[1]() // 3 (not 1!)
handlers[2]() // 3 (not 2!)

// Tại sao? var i là shared variable
// Tất cả closures reference cùng một 'i'
// Sau loop, i = 3
```

**Solution 1: Use let (block scope)**

```typescript
// ✅ Let creates new binding mỗi iteration
function createHandlers() {
  const handlers: (() => void)[] = []

  for (let i = 0; i < 3; i++) {
    handlers.push(() => {
      console.log(i)
    })
  }

  return handlers
}

const handlers = createHandlers()
handlers[0]() // 0 ✅
handlers[1]() // 1 ✅
handlers[2]() // 2 ✅
```

**Solution 2: IIFE**

```typescript
// ✅ IIFE creates new scope
function createHandlers() {
  const handlers: (() => void)[] = []

  for (var i = 0; i < 3; i++) {
    ;((index) => {
      handlers.push(() => {
        console.log(index)
      })
    })(i)
  }

  return handlers
}
```

#### Problem: React Hooks (Stale closure trong useEffect)

**Stale state:**

```typescript
// ❌ Stale closure (React example)
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // Always logs 0!
      // Closure captured initial count value
    }, 1000);

    return () => clearInterval(interval);
  }, []); // Empty deps → closure không update

  return <button onClick={() => setCount(count + 1)}>Increment</button>;
}
```

**Solution: Functional update hoặc đúng deps**

```typescript
// ✅ Solution 1: Functional update
useEffect(() => {
  const interval = setInterval(() => {
    setCount((c) => {
      console.log(c) // Always fresh value
      return c
    })
  }, 1000)

  return () => clearInterval(interval)
}, [])

// ✅ Solution 2: Include in deps (re-create interval)
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count)
  }, 1000)

  return () => clearInterval(interval)
}, [count]) // Re-run khi count changes
```

---

### 4.3 Performance Considerations

#### Memory Overhead

**Each closure allocates:**

- Function object: ~100 bytes
- Scope object: depends on captured variables
- Prototype chain overhead

**Benchmark:**

```typescript
// Heavy allocation
function heavyClosures() {
  const functions: (() => number)[] = []

  for (let i = 0; i < 100000; i++) {
    functions.push(() => i) // 100k closures
  }

  return functions
}

// Lighter alternative
function lighterAlternative() {
  const data = new Array(100000).fill(0).map((_, i) => i)

  return {
    get: (index: number) => data[index]
  }
}
```

#### Optimization: Hoist when possible

```typescript
// ❌ Creates new closure each call
function processItems(items: number[]) {
  return items.map((item) => {
    return item * 2 // New closure mỗi lần gọi processItems
  })
}

// ✅ Hoist closure ra ngoài
const doubleItem = (item: number): number => item * 2

function processItems(items: number[]) {
  return items.map(doubleItem) // Reuse same function
}
```

#### Closure vs Class Performance

**Closure:**

- ✅ Lighter weight cho simple cases
- ✅ Better encapsulation
- ❌ Mỗi instance tạo new functions

**Class:**

- ✅ Methods shared qua prototype
- ✅ Better performance khi nhiều instances
- ❌ Cần manual binding (this)

**Benchmark:**

```typescript
// Closure approach
function createCounter() {
  let count = 0
  return {
    increment: () => ++count, // New function mỗi instance
    getValue: () => count
  }
}

// Class approach
class Counter {
  private count = 0

  increment(): number {
    // Shared via prototype
    return ++this.count
  }

  getValue(): number {
    return this.count
  }
}

// Creating 10k instances:
// Closure: ~10k * 2 functions = 20k function objects
// Class: 2 methods on prototype, 10k instances
```

---

## 5. Closure vs Class

### Khi nào dùng Closure?

**✅ Dùng Closure khi:**

**1. Cần truly private state:**

```typescript
// Closure: count thực sự private
function createCounter() {
  let count = 0 // Không thể access từ ngoài
  return { increment: () => ++count }
}

// Class: private chỉ là compile-time
class Counter {
  private count = 0 // Runtime vẫn access được qua counter['count']
}
```

**2. One-off instances (không cần nhiều instances):**

```typescript
// Singleton/Module pattern
const ConfigManager = (() => {
  let config = {}
  return {
    set: (key: string, value: any) => (config[key] = value),
    get: (key: string) => config[key]
  }
})()
```

**3. Function factories:**

```typescript
const createMultiplier = (factor: number) => (n: number) => n * factor
const double = createMultiplier(2)
const triple = createMultiplier(3)
```

**4. Stateful utilities:**

```typescript
function createDebounce<T extends any[]>(fn: (...args: T) => void, delay: number) {
  let timeoutId: NodeJS.Timeout | null = null

  return (...args: T): void => {
    if (timeoutId) clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn(...args), delay)
  }
}
```

### Khi nào dùng Class?

**✅ Dùng Class khi:**

**1. Cần nhiều instances với shared behavior:**

```typescript
class User {
  constructor(
    public name: string,
    public email: string
  ) {}

  // Method shared qua prototype (memory efficient)
  greet(): string {
    return `Hello, ${this.name}`
  }
}

const users = Array.from({ length: 1000 }, (_, i) => new User(`User${i}`, `user${i}@example.com`))
// 1 greet method trên prototype, không phải 1000 copies
```

**2. Cần inheritance/polymorphism:**

```typescript
abstract class Animal {
  abstract makeSound(): string
}

class Dog extends Animal {
  makeSound(): string {
    return 'Woof'
  }
}

class Cat extends Animal {
  makeSound(): string {
    return 'Meow'
  }
}
```

**3. Cần instanceof checks:**

```typescript
class CustomError extends Error {
  constructor(message: string) {
    super(message)
  }
}

try {
  throw new CustomError('Something wrong')
} catch (e) {
  if (e instanceof CustomError) {
    // Type narrowing works
  }
}
```

**4. Complex state với lifecycle:**

```typescript
class DatabaseConnection {
  private connection: any = null

  async connect(): Promise<void> {
    /* ... */
  }
  async query(sql: string): Promise<any> {
    /* ... */
  }
  async disconnect(): Promise<void> {
    /* ... */
  }
}
```

### So sánh tổng quan

| Tiêu chí                     | Closure                       | Class                        |
| ---------------------------- | ----------------------------- | ---------------------------- |
| **Privacy**                  | Truly private                 | Private chỉ compile-time     |
| **Memory (nhiều instances)** | Mỗi instance = new functions  | Methods shared qua prototype |
| **Inheritance**              | Không có                      | Có (extends)                 |
| **instanceof**               | Không                         | Có                           |
| **this binding**             | Không cần (lexical scope)     | Cần (dynamic this)           |
| **Syntax**                   | Functional, concise           | OOP, verbose hơn             |
| **Use case**                 | Utilities, factories, modules | Entities, nhiều instances    |

### Hybrid Approach

**Combine cả hai:**

```typescript
class UserManager {
  private users: Map<number, User> = new Map()

  // Public method return closure
  createFinder(): (id: number) => User | undefined {
    // Closure capture 'users' reference
    return (id: number) => this.users.get(id)
  }

  // Factory method returns closure
  createValidator(): (email: string) => boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return (email: string) => regex.test(email)
  }
}

const manager = new UserManager()
const findUser = manager.createFinder() // Closure
const validateEmail = manager.createValidator() // Closure
```

---

## Tổng kết

### Core Takeaways

**1. Closure = Function + Lexical Environment**

- Function mang theo "ba lô" chứa variables từ outer scope
- Variables tồn tại trên Heap, không bị GC nếu closure còn reference

**2. Memory Model:**

- Closure giữ reference đến scope object
- Modern engines chỉ giữ variables thực sự được dùng
- Cẩn thận memory leaks với event listeners, timers

**3. Design Patterns:**

- **Encapsulation:** Truly private variables
- **Factories:** Specialized functions
- **Memoization:** Cache results
- **Modules:** Singleton pattern

**4. Pitfalls:**

- **Memory leaks:** Unintentional references
- **Stale closures:** Loop với var, React hooks
- **Performance:** Balance giữa closure convenience và class efficiency

**5. Closure vs Class:**

- Closure: Privacy, factories, one-off utilities
- Class: Multiple instances, inheritance, lifecycle

### Best Practices

**✅ DO:**

- Dùng `let`/`const` thay vì `var`
- Extract only needed data vào closure
- Cleanup event listeners và timers
- Consider memory implications khi tạo nhiều closures

**❌ DON'T:**

- Capture toàn bộ large objects unnecessarily
- Forget cleanup trong long-running applications
- Overuse closures khi class appropriate
- Create closures trong hot paths (performance-critical loops)

---

## Tài liệu tham khảo

- [MDN - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/scope-closures)
- [V8 Blog - Understanding Memory Management](https://v8.dev/blog/trash-talk)
- [JavaScript Execution Context](https://ui.dev/ultimate-guide-to-execution-contexts-hoisting-scopes-and-closures-in-javascript)

---

_Viết bởi Senior Software Engineer - Deep dive TypeScript/JavaScript - January 2026_
