# TypeScript Problem Solving Challenges

Concept → Challenge format for testing TypeScript skills.

---

## 1. TYPES & TYPE INFERENCE

### Concept Questions

**Q1:** What's the difference between `type` and `interface`?
> `interface` is extendable and mergeable. `type` can represent unions, primitives, tuples.

**Q2:** When does TypeScript infer types?
> When variables are initialized, function return values, and array literals.

**Q3:** What does `readonly` do?
> Makes property immutable after initialization.

### Challenges

**Challenge 1:** Fix the type errors:
```typescript
// Problem:
const user = {
    name: "John",
    age: 30
};
user.email = "john@test.com"; // Error!

// Solution:
interface User {
    name: string;
    age: number;
    email?: string;
}
const user: User = { name: "John", age: 30 };
user.email = "john@test.com"; // OK
```

**Challenge 2:** Create a type for API response:
```typescript
// Solution:
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
    timestamp: Date;
}

interface User {
    id: number;
    name: string;
}

const response: ApiResponse<User> = {
    data: { id: 1, name: "John" },
    status: 200,
    message: "Success",
    timestamp: new Date()
};
```

---

## 2. UNION & INTERSECTION TYPES

### Concept Questions

**Q1:** What is a union type?
> A type that can be one of several types: `string | number`.

**Q2:** What is an intersection type?
> A type that combines multiple types: `TypeA & TypeB`.

**Q3:** How do you narrow a union type?
> Using type guards: `typeof`, `instanceof`, `in`, or custom type predicates.

### Challenges

**Challenge 1:** Handle different response types:
```typescript
// Solution:
type SuccessResponse = { status: 'success'; data: any };
type ErrorResponse = { status: 'error'; message: string };
type Response = SuccessResponse | ErrorResponse;

function handleResponse(response: Response) {
    if (response.status === 'success') {
        console.log(response.data); // TypeScript knows it's SuccessResponse
    } else {
        console.log(response.message); // TypeScript knows it's ErrorResponse
    }
}
```

**Challenge 2:** Combine types with intersection:
```typescript
// Solution:
type Timestamped = { createdAt: Date; updatedAt: Date };
type Identifiable = { id: string };

type Entity<T> = T & Timestamped & Identifiable;

interface UserData {
    name: string;
    email: string;
}

type User = Entity<UserData>;
// User has: name, email, id, createdAt, updatedAt
```

---

## 3. GENERICS

### Concept Questions

**Q1:** What are generics?
> Type variables that allow creating reusable components that work with any type.

**Q2:** What is a generic constraint?
> Limiting generic types using `extends`: `<T extends SomeType>`.

**Q3:** What does `keyof` do?
> Creates a union type of all keys of an object type.

### Challenges

**Challenge 1:** Create a generic function to get property:
```typescript
// Solution:
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const user = { name: "John", age: 30 };
const name = getProperty(user, "name"); // type: string
const age = getProperty(user, "age");   // type: number
// getProperty(user, "email"); // Error: "email" not in keyof user
```

**Challenge 2:** Create a generic Stack class:
```typescript
// Solution:
class Stack<T> {
    private items: T[] = [];
    
    push(item: T): void {
        this.items.push(item);
    }
    
    pop(): T | undefined {
        return this.items.pop();
    }
    
    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }
    
    isEmpty(): boolean {
        return this.items.length === 0;
    }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2
```

**Challenge 3:** Generic function with multiple type parameters:
```typescript
// Solution:
function merge<T, U>(obj1: T, obj2: U): T & U {
    return { ...obj1, ...obj2 };
}

const result = merge({ name: "John" }, { age: 30 });
// result type: { name: string } & { age: number }
```

---

## 4. UTILITY TYPES

### Concept Questions

**Q1:** What does `Partial<T>` do?
> Makes all properties optional.

**Q2:** What does `Pick<T, K>` do?
> Creates type with only selected keys.

**Q3:** What does `Omit<T, K>` do?
> Creates type excluding specified keys.

**Q4:** What does `Record<K, V>` do?
> Creates object type with keys K and values V.

### Challenges

**Challenge 1:** Use Partial for update function:
```typescript
// Solution:
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

function updateUser(id: number, updates: Partial<User>): User {
    const user = getUserById(id);
    return { ...user, ...updates };
}

updateUser(1, { name: "Jane" }); // Only update name
updateUser(1, { age: 31, email: "jane@test.com" }); // Update multiple
```

**Challenge 2:** Create type excluding sensitive fields:
```typescript
// Solution:
interface UserRecord {
    id: number;
    name: string;
    email: string;
    password: string;
    ssn: string;
}

type PublicUser = Omit<UserRecord, 'password' | 'ssn'>;
// PublicUser: { id: number; name: string; email: string }
```

**Challenge 3:** Create dictionary type:
```typescript
// Solution:
type UserRole = 'admin' | 'user' | 'guest';

type RolePermissions = Record<UserRole, string[]>;

const permissions: RolePermissions = {
    admin: ['read', 'write', 'delete'],
    user: ['read', 'write'],
    guest: ['read']
};
```

---

## 5. TYPE GUARDS

### Concept Questions

**Q1:** What is a type guard?
> A runtime check that narrows the type within a conditional block.

**Q2:** What is a type predicate?
> Return type `value is Type` that tells TypeScript the narrowed type.

**Q3:** When would you use `instanceof` vs `typeof`?
> `typeof` for primitives. `instanceof` for class instances.

### Challenges

**Challenge 1:** Custom type guard:
```typescript
// Solution:
interface Cat {
    meow(): void;
}

interface Dog {
    bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
    return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog) {
    if (isCat(animal)) {
        animal.meow(); // TypeScript knows it's Cat
    } else {
        animal.bark(); // TypeScript knows it's Dog
    }
}
```

**Challenge 2:** Discriminated union:
```typescript
// Solution:
interface Square {
    kind: 'square';
    size: number;
}

interface Circle {
    kind: 'circle';
    radius: number;
}

interface Rectangle {
    kind: 'rectangle';
    width: number;
    height: number;
}

type Shape = Square | Circle | Rectangle;

function getArea(shape: Shape): number {
    switch (shape.kind) {
        case 'square':
            return shape.size ** 2;
        case 'circle':
            return Math.PI * shape.radius ** 2;
        case 'rectangle':
            return shape.width * shape.height;
    }
}
```

---

## 6. MAPPED TYPES

### Concept Questions

**Q1:** What is a mapped type?
> A type that transforms properties of another type.

**Q2:** What does `[K in keyof T]` mean?
> Iterates over all keys of type T.

**Q3:** What does `-?` do in mapped types?
> Removes optional modifier (makes required).

### Challenges

**Challenge 1:** Create Readonly type from scratch:
```typescript
// Solution:
type MyReadonly<T> = {
    readonly [K in keyof T]: T[K];
};

interface User {
    name: string;
    age: number;
}

type ReadonlyUser = MyReadonly<User>;
// All properties are readonly
```

**Challenge 2:** Create type that makes all properties nullable:
```typescript
// Solution:
type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};

interface Config {
    host: string;
    port: number;
}

type NullableConfig = Nullable<Config>;
// { host: string | null; port: number | null }
```

**Challenge 3:** Create getters type:
```typescript
// Solution:
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

---

## 7. CONDITIONAL TYPES

### Concept Questions

**Q1:** What is a conditional type?
> A type that selects one of two types based on a condition: `T extends U ? X : Y`.

**Q2:** What does `infer` do?
> Extracts a type from within a conditional type.

**Q3:** What are distributive conditional types?
> When conditional type is applied to each member of a union separately.

### Challenges

**Challenge 1:** Extract return type:
```typescript
// Solution:
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(): string { return "Hello"; }
type GreetReturn = MyReturnType<typeof greet>; // string
```

**Challenge 2:** Extract array element type:
```typescript
// Solution:
type ElementType<T> = T extends (infer U)[] ? U : T;

type NumArray = ElementType<number[]>; // number
type StrArray = ElementType<string[]>; // string
type NotArray = ElementType<boolean>;  // boolean
```

**Challenge 3:** Create NonNullable from scratch:
```typescript
// Solution:
type MyNonNullable<T> = T extends null | undefined ? never : T;

type MaybeString = string | null | undefined;
type DefiniteString = MyNonNullable<MaybeString>; // string
```

---

## 8. ASYNC TYPES

### Concept Questions

**Q1:** How do you type a Promise?
> `Promise<T>` where T is the resolved value type.

**Q2:** What does `Awaited<T>` do?
> Extracts the type that a Promise resolves to.

**Q3:** How do you type async function return?
> Return type is `Promise<T>` where T is the actual return value.

### Challenges

**Challenge 1:** Type async function:
```typescript
// Solution:
interface User {
    id: number;
    name: string;
}

async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

async function fetchUsers(): Promise<User[]> {
    const response = await fetch('/api/users');
    return response.json();
}
```

**Challenge 2:** Promise.all typing:
```typescript
// Solution:
async function fetchData(): Promise<[User, Post[]]> {
    const [user, posts] = await Promise.all([
        fetchUser(1),
        fetchPosts()
    ]);
    return [user, posts];
}
```

---

## 9. FUNCTION OVERLOADS

### Concept Questions

**Q1:** What are function overloads?
> Multiple function signatures for the same function with different parameter/return types.

**Q2:** How do overloads work in TypeScript?
> Declare multiple signatures, then one implementation that handles all cases.

### Challenges

**Challenge 1:** Overload for different input types:
```typescript
// Solution:
function format(value: string): string;
function format(value: number): string;
function format(value: Date): string;
function format(value: string | number | Date): string {
    if (typeof value === 'string') return value.toUpperCase();
    if (typeof value === 'number') return value.toFixed(2);
    return value.toISOString();
}

format("hello");     // returns string
format(42.123);      // returns string
format(new Date());  // returns string
```

**Challenge 2:** Overload with different return types:
```typescript
// Solution:
function parse(input: string, asNumber: true): number;
function parse(input: string, asNumber: false): string;
function parse(input: string, asNumber: boolean): string | number {
    return asNumber ? parseFloat(input) : input;
}

const num = parse("42", true);   // type: number
const str = parse("42", false);  // type: string
```

---

## 10. PROBLEM SOLVING

### Challenge 1: Implement groupBy
```typescript
// Problem: Group array items by a key
function groupBy<T, K extends keyof T>(arr: T[], key: K): Record<string, T[]> {
    // Your solution
}

// Test:
const people = [
    { name: 'John', dept: 'IT' },
    { name: 'Jane', dept: 'HR' },
    { name: 'Bob', dept: 'IT' }
];
console.log(groupBy(people, 'dept'));
// { IT: [{name:'John',...}, {name:'Bob',...}], HR: [{name:'Jane',...}] }
```
<details><summary>Solution</summary>

```typescript
function groupBy<T, K extends keyof T>(arr: T[], key: K): Record<string, T[]> {
    return arr.reduce((acc, item) => {
        const groupKey = String(item[key]);
        if (!acc[groupKey]) acc[groupKey] = [];
        acc[groupKey].push(item);
        return acc;
    }, {} as Record<string, T[]>);
}
```
</details>

### Challenge 2: Implement debounce with types
```typescript
// Problem: Create typed debounce function
function debounce<T extends (...args: any[]) => any>(
    fn: T,
    ms: number
): (...args: Parameters<T>) => void {
    // Your solution
}

// Test:
const log = debounce((msg: string) => console.log(msg), 300);
log("a"); log("b"); log("c"); // Only "c" after 300ms
```
<details><summary>Solution</summary>

```typescript
function debounce<T extends (...args: any[]) => any>(
    fn: T,
    ms: number
): (...args: Parameters<T>) => void {
    let timeoutId: ReturnType<typeof setTimeout>;
    return (...args: Parameters<T>) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn(...args), ms);
    };
}
```
</details>

### Challenge 3: Implement deep clone with types
```typescript
// Problem: Deep clone an object preserving types
function deepClone<T>(obj: T): T {
    // Your solution
}

// Test:
const original = { a: 1, b: { c: 2 } };
const cloned = deepClone(original);
cloned.b.c = 99;
console.log(original.b.c); // 2
```
<details><summary>Solution</summary>

```typescript
function deepClone<T>(obj: T): T {
    if (obj === null || typeof obj !== 'object') return obj;
    if (Array.isArray(obj)) return obj.map(deepClone) as T;
    
    const cloned = {} as T;
    for (const key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}
```
</details>

### Challenge 4: Implement memoize with types
```typescript
// Problem: Memoize function with proper typing
function memoize<T extends (...args: any[]) => any>(fn: T): T {
    // Your solution
}

// Test:
let calls = 0;
const expensive = (n: number) => { calls++; return n * 2; };
const memoized = memoize(expensive);
memoized(5); // calls: 1
memoized(5); // calls: 1 (cached)
```
<details><summary>Solution</summary>

```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
    const cache = new Map<string, ReturnType<T>>();
    return ((...args: Parameters<T>): ReturnType<T> => {
        const key = JSON.stringify(args);
        if (cache.has(key)) return cache.get(key)!;
        const result = fn(...args);
        cache.set(key, result);
        return result;
    }) as T;
}
```
</details>

### Challenge 5: Implement pick function
```typescript
// Problem: Pick specific keys from object
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    // Your solution
}

// Test:
const user = { id: 1, name: 'John', email: 'john@test.com', age: 30 };
const picked = pick(user, ['name', 'email']);
// { name: 'John', email: 'john@test.com' }
```
<details><summary>Solution</summary>

```typescript
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    const result = {} as Pick<T, K>;
    for (const key of keys) {
        result[key] = obj[key];
    }
    return result;
}
```
</details>

### Challenge 6: Implement omit function
```typescript
// Problem: Omit specific keys from object
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
    // Your solution
}

// Test:
const user = { id: 1, name: 'John', password: 'secret' };
const safe = omit(user, ['password']);
// { id: 1, name: 'John' }
```
<details><summary>Solution</summary>

```typescript
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
    const result = { ...obj };
    for (const key of keys) {
        delete result[key];
    }
    return result as Omit<T, K>;
}
```
</details>

### Challenge 7: Implement flatten object
```typescript
// Problem: Flatten nested object
function flatten(obj: object, prefix = ''): Record<string, any> {
    // Your solution
}

// Test:
const nested = { a: 1, b: { c: 2, d: { e: 3 } } };
console.log(flatten(nested));
// { 'a': 1, 'b.c': 2, 'b.d.e': 3 }
```
<details><summary>Solution</summary>

```typescript
function flatten(obj: object, prefix = ''): Record<string, any> {
    const result: Record<string, any> = {};
    for (const [key, value] of Object.entries(obj)) {
        const newKey = prefix ? `${prefix}.${key}` : key;
        if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
            Object.assign(result, flatten(value, newKey));
        } else {
            result[newKey] = value;
        }
    }
    return result;
}
```
</details>

### Challenge 8: Implement retry function
```typescript
// Problem: Retry async function with backoff
async function retry<T>(
    fn: () => Promise<T>,
    retries: number,
    delay: number
): Promise<T> {
    // Your solution
}

// Test:
let attempts = 0;
const unstable = async () => {
    attempts++;
    if (attempts < 3) throw new Error('Failed');
    return 'Success';
};
const result = await retry(unstable, 5, 100);
```
<details><summary>Solution</summary>

```typescript
async function retry<T>(
    fn: () => Promise<T>,
    retries: number,
    delay: number
): Promise<T> {
    try {
        return await fn();
    } catch (error) {
        if (retries <= 0) throw error;
        await new Promise(r => setTimeout(r, delay));
        return retry(fn, retries - 1, delay * 2);
    }
}
```
</details>

### Challenge 9: Implement event emitter
```typescript
// Problem: Type-safe event emitter
class EventEmitter<Events extends Record<string, any>> {
    // Your implementation
    on<K extends keyof Events>(event: K, handler: (data: Events[K]) => void): void;
    emit<K extends keyof Events>(event: K, data: Events[K]): void;
}

// Test:
interface AppEvents {
    login: { userId: string };
    logout: void;
}

const emitter = new EventEmitter<AppEvents>();
emitter.on('login', ({ userId }) => console.log(userId));
emitter.emit('login', { userId: '123' });
```
<details><summary>Solution</summary>

```typescript
class EventEmitter<Events extends Record<string, any>> {
    private listeners: { [K in keyof Events]?: Array<(data: Events[K]) => void> } = {};

    on<K extends keyof Events>(event: K, handler: (data: Events[K]) => void): void {
        if (!this.listeners[event]) this.listeners[event] = [];
        this.listeners[event]!.push(handler);
    }

    emit<K extends keyof Events>(event: K, data: Events[K]): void {
        this.listeners[event]?.forEach(handler => handler(data));
    }
}
```
</details>

### Challenge 10: Implement pipe function
```typescript
// Problem: Compose functions left to right
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
    // Your solution
}

// Test:
const addOne = (x: number) => x + 1;
const double = (x: number) => x * 2;
const square = (x: number) => x * x;

const compute = pipe(addOne, double, square);
console.log(compute(2)); // ((2+1)*2)^2 = 36
```
<details><summary>Solution</summary>

```typescript
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
    return (arg: T) => fns.reduce((result, fn) => fn(result), arg);
}
```
</details>

---

## Quick Type Reference

| Utility | Description |
|---------|-------------|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties readonly |
| `Pick<T, K>` | Only selected keys |
| `Omit<T, K>` | Exclude selected keys |
| `Record<K, V>` | Object with K keys, V values |
| `Exclude<T, U>` | Remove U from T union |
| `Extract<T, U>` | Keep only U from T union |
| `NonNullable<T>` | Remove null/undefined |
| `ReturnType<T>` | Function return type |
| `Parameters<T>` | Function parameters tuple |
| `Awaited<T>` | Unwrap Promise type |

---

## Scoring Guide

| Level | Expected |
|-------|----------|
| Junior | Concept questions + Basic challenges |
| Mid | All concepts + Half problem solving |
| Senior | All sections + Clean implementation |

---

*Focus on type safety and proper TypeScript usage, not just making it work.*
