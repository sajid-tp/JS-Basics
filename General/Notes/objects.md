# JavaScript Objects — Complete Notes

## 1. What is an object, really?

An object is a **collection of key-value pairs**, where keys are strings (or Symbols) and values can be anything — numbers, strings, functions, arrays, or other objects. Under the hood, JS engines implement objects as hash maps with some extra bookkeeping (property descriptors, a hidden prototype link).

```javascript
const user = {
  name: "Aditya",
  age: 25,
  isActive: true
};
```

Every non-primitive value in JS — arrays, functions, dates, regexes, even `null`'s cousin-in-spirit `Map`/`Set` — is technically an object under the hood. Arrays are objects with numeric-like keys and a `length` property; functions are objects that happen to be callable.

## 2. Primitives vs Objects — the core distinction

This is the single most important mental model in JS.

| | Primitives (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) | Objects |
|---|---|---|
| Stored as | Value, directly | Reference (pointer to memory) |
| Copied by | Value (a true copy) | Reference (same underlying object) |
| Compared by | Value | Reference identity |
| Mutable? | No — a "changed" primitive is a new value | Yes — you can mutate in place |

```javascript
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 — unaffected, because b got a COPY

let obj1 = { val: 5 };
let obj2 = obj1;
obj2.val = 10;
console.log(obj1.val); // 10 — SAME object, obj2 was just another pointer to it
```

```javascript
console.log({} === {}); // false — different objects in memory, even though identical shape
const x = {};
console.log(x === x); // true — same reference
```

This is why deep-cloning matters, why Redux/React care about immutability, and why `error.config` mutation worked in our earlier axios discussion — `failedRequest` was a *reference* to the same object Axios held internally.

## 3. Creating objects — all the ways

```javascript
// 1. Object literal (most common)
const obj1 = { key: "value" };

// 2. new Object()
const obj2 = new Object();
obj2.key = "value";

// 3. Object.create() — explicit prototype control
const obj3 = Object.create(null); // no prototype at all — a truly "bare" object
const obj4 = Object.create(someOtherObject); // someOtherObject becomes the prototype

// 4. Constructor function
function Person(name) { this.name = name; }
const obj5 = new Person("Aditya");

// 5. Class syntax (syntactic sugar over constructor functions)
class Animal { constructor(name) { this.name = name; } }
const obj6 = new Animal("Dog");

// 6. Factory function
function createUser(name) { return { name }; }
const obj7 = createUser("Aditya");
```

## 4. Accessing and setting properties

```javascript
const user = { name: "Aditya", "favorite color": "blue" };

// Dot notation — only works for valid identifier keys
user.name;

// Bracket notation — works for ANY key, including dynamic/computed ones
user["favorite color"];
const key = "name";
user[key]; // dynamic access — dot notation can't do this
```

**Adding/updating** is identical syntax to reading — just assign:
```javascript
user.age = 25;        // adds a new key
user.name = "Rahul";  // overwrites existing key
```

**Deleting**:
```javascript
delete user.age; // removes the key entirely (returns true/false)
```

## 5. Computed property names

Lets you use an expression as a key, right inside the literal:
```javascript
const field = "email";
const user = {
  [field]: "test@example.com",      // key becomes "email"
  [`${field}_verified`]: true       // key becomes "email_verified"
};
```

## 6. Shorthand syntax (ES6)

```javascript
const name = "Aditya";
const age = 25;

// Old way
const user = { name: name, age: age };

// Shorthand — key and variable name are identical
const user2 = { name, age };

// Method shorthand
const obj = {
  greet: function() { console.log("hi"); },  // old
  greet2() { console.log("hi"); }             // shorthand — identical behavior
};
```

## 7. `this` inside objects

`this` refers to **whatever object the method was called on** — not where it was defined.

```javascript
const user = {
  name: "Aditya",
  greet() { console.log(this.name); }
};
user.greet(); // "Aditya" — this = user, because user.greet() was the call

const greetFn = user.greet;
greetFn(); // undefined (or throws in strict mode) — this is now lost/global,
           // because it's just a bare function call, no object before the dot
```

This "losing `this`" problem is exactly why React class components used to need `this.method = this.method.bind(this)` in constructors, and why arrow functions (which don't have their own `this` — they inherit it from where they're *defined*) became the preferred fix:

```javascript
const user = {
  name: "Aditya",
  greetLater() {
    setTimeout(() => {
      console.log(this.name); // arrow function — `this` still = user, inherited from greetLater's scope
    }, 1000);
  }
};
```

## 8. Object equality and comparison

```javascript
{} === {}          // false, different references
const a = {};
a === a             // true, same reference

Object.is(NaN, NaN)   // true (unlike === which says NaN === NaN is false)
```

To compare objects **by content** (deep equality), JS has no built-in — you either write your own recursive check, use `JSON.stringify(a) === JSON.stringify(b)` (unreliable — key order matters, functions/undefined get dropped), or use a library like Lodash's `_.isEqual`.

## 9. Nested objects and deep vs shallow copy

```javascript
const user = {
  name: "Aditya",
  address: { city: "Delhi", pin: "110001" }
};

// SHALLOW copy — top-level keys are copied, but nested objects are still SHARED references
const copy1 = { ...user };
copy1.address.city = "Mumbai";
console.log(user.address.city); // "Mumbai" too! Because address wasn't truly copied.

// TRUE deep copy
const copy2 = structuredClone(user); // modern, built-in (Node 17+/all current browsers)
copy2.address.city = "Pune";
console.log(user.address.city); // still "Mumbai" — unaffected
```

`JSON.parse(JSON.stringify(obj))` is the old-school deep clone trick, but it silently drops `undefined`, functions, `Date` objects (converts to string), and breaks on circular references. `structuredClone()` is the modern correct answer.

## 10. Spread and rest with objects

```javascript
const base = { a: 1, b: 2 };

// Spread — merges/copies keys. LATER keys override earlier ones.
const merged = { ...base, b: 99, c: 3 }; // { a: 1, b: 99, c: 3 }

// Rest — "the remaining keys" during destructuring
const { a, ...rest } = merged;
console.log(a);    // 1
console.log(rest); // { b: 99, c: 3 }
```

This spread pattern is the standard **immutable update** pattern in React/Redux — you never mutate `state` directly, you spread it into a new object with the changed key overridden, which is exactly what a slice's reducer conceptually does under the hood (Redux Toolkit uses Immer to let you *write* mutating-looking code that produces this behind the scenes).

## 11. Destructuring

```javascript
const user = { name: "Aditya", age: 25, address: { city: "Delhi" } };

const { name, age } = user;                  // basic
const { name: userName } = user;             // renaming while destructuring
const { country = "India" } = user;          // default value if key doesn't exist
const { address: { city } } = user;          // nested destructuring
```

Function parameters are a hugely common destructuring site:
```javascript
function greet({ name, age = 18 }) {
  console.log(`${name} is ${age}`);
}
greet({ name: "Aditya" }); // "Aditya is 18"
```

## 12. Optional chaining and nullish coalescing

This is exactly the `?.` used throughout our earlier `error.response?.data?.code` discussion:

```javascript
const user = { profile: null };

user.profile.bio;      // TypeError: Cannot read properties of null
user.profile?.bio;     // undefined — safely short-circuits instead of throwing

// Nullish coalescing — provide a fallback ONLY for null/undefined (not 0, "", false)
const bio = user.profile?.bio ?? "No bio yet";
```

`??` differs from `||`: `0 || "default"` gives `"default"` (0 is falsy), but `0 ?? "default"` gives `0` (0 is not null/undefined). This matters a lot when a legitimate value could be `0` or `""`.

## 13. Object iteration

```javascript
const user = { name: "Aditya", age: 25 };

for (const key in user) console.log(key); // "name", "age" — iterates keys (including inherited ones)

Object.keys(user);    // ["name", "age"]
Object.values(user);  // ["Aditya", 25]
Object.entries(user); // [["name","Aditya"], ["age",25]]

Object.entries(user).forEach(([key, value]) => console.log(key, value));
```

`for...in` also walks up the **prototype chain** and includes inherited enumerable properties — a common footgun. `Object.keys/values/entries` only look at the object's **own** properties, which is almost always what you actually want.

## 14. Checking existence

```javascript
const user = { name: "Aditya", age: undefined };

"name" in user;              // true
"age" in user;                // true — key exists even though value is undefined
"email" in user;              // false

user.hasOwnProperty("name");  // true — own property, not inherited
Object.hasOwn(user, "name");  // true — modern replacement, safer (works even if hasOwnProperty was overridden)

user.name !== undefined;      // true — but this is WRONG for checking existence,
                               // since it can't distinguish "key missing" from "key present with value undefined"
```

## 15. Object methods — the essential toolkit

```javascript
Object.assign(target, ...sources);
// Copies own enumerable properties from sources INTO target, mutating target, returns target
const merged = Object.assign({}, obj1, obj2); // common pattern to avoid mutating obj1

Object.freeze(obj);
// Makes obj IMMUTABLE — no add/remove/change of properties (shallow only! nested objects still mutable)
const frozen = Object.freeze({ a: 1 });
frozen.a = 99; // silently fails (or throws in strict mode)

Object.isFrozen(obj); // check

Object.seal(obj);
// Prevents ADDING/REMOVING keys, but EXISTING keys can still be reassigned (unlike freeze)

Object.keys(obj).length; // common way to check "how many properties"

Object.fromEntries([["a", 1], ["b", 2]]); // { a: 1, b: 2 } — inverse of Object.entries
```

## 16. Property descriptors — the layer beneath normal syntax

Every property isn't just a value — it has hidden metadata (a "descriptor") controlling its behavior:

```javascript
Object.getOwnPropertyDescriptor(user, "name");
// { value: "Aditya", writable: true, enumerable: true, configurable: true }
```

- `writable` — can the value be reassigned?
- `enumerable` — does it show up in `for...in` / `Object.keys` / spread?
- `configurable` — can the property be deleted or its descriptor changed?

```javascript
Object.defineProperty(user, "id", {
  value: 101,
  writable: false,     // read-only
  enumerable: false,   // hidden from Object.keys, JSON.stringify, for...in
  configurable: false
});
```

This is how libraries build "hidden" internal properties, and it's the actual mechanism `Object.freeze` uses under the hood (it sets `writable: false, configurable: false` on every property).

## 17. Getters and setters

Properties that run a function when read or written, but look like normal property access from the outside:

```javascript
const user = {
  firstName: "Aditya",
  lastName: "Kumar",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(" ");
  }
};

console.log(user.fullName);  // "Aditya Kumar" — called like a property, runs the getter function
user.fullName = "Rahul Sharma"; // runs the setter — splits and reassigns firstName/lastName
```

## 18. The prototype chain — how inheritance actually works

Every object has an internal, hidden link to another object called its **prototype**. When you access a property that doesn't exist on the object itself, JS walks up this chain looking for it.

```javascript
const animal = { eats: true };
const dog = Object.create(animal); // dog's prototype is now `animal`
dog.barks = true;

console.log(dog.eats);   // true — not found on dog itself, found on its prototype `animal`
console.log(dog.barks);  // true — found directly on dog

Object.getPrototypeOf(dog) === animal; // true
```

This chain keeps going until it hits `Object.prototype` (which is where `.hasOwnProperty`, `.toString()` etc. actually live), and finally `null`, which ends the chain.

```javascript
const obj = {};
Object.getPrototypeOf(obj) === Object.prototype; // true — every plain object literal's prototype
Object.getPrototypeOf(Object.prototype);          // null — end of the chain
```

`__proto__` is the old, informal way to access this (`dog.__proto__ === animal`) — still works everywhere but is considered legacy; `Object.getPrototypeOf`/`Object.setPrototypeOf` are the modern, correct APIs.

## 19. Constructor functions and `new`

Before ES6 classes existed, this was how "instances" were made:

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.greet = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const p1 = new Person("Aditya", 25);
p1.greet(); // "Hi, I'm Aditya"
```

What `new Person(...)` actually does, step by step:
1. Creates a brand-new empty object `{}`.
2. Sets that object's prototype to `Person.prototype` (so `p1.greet` resolves via the chain).
3. Calls `Person` with `this` bound to that new object.
4. Returns the object (unless the function explicitly returns some other object itself).

This is exactly why every instance shares `.greet` via the prototype (memory-efficient — one function, not one copy per instance) instead of each instance getting its own copy of the method.

## 20. ES6 classes — same thing, nicer syntax

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  greet() {              // goes on Person.prototype automatically, same as before
    console.log(`Hi, I'm ${this.name}`);
  }
  static create(name) {  // lives on the class itself, not on instances
    return new Person(name, 0);
  }
}

class Student extends Person {
  constructor(name, age, school) {
    super(name, age); // must call before using `this` — runs Person's constructor
    this.school = school;
  }
  greet() {
    super.greet();     // call the parent's version too
    console.log(`I study at ${this.school}`);
  }
}
```

Under the hood, `class` is **syntactic sugar** — `Student extends Person` sets up the exact same prototype chain linkage as `Object.setPrototypeOf(Student.prototype, Person.prototype)` would manually. Nothing new exists at the engine level; it's a cleaner syntax over the same prototype mechanism from section 18–19.

## 21. Object vs Map — when to use which

```javascript
// Object — keys coerced to strings, has prototype baggage, no guaranteed size
const obj = { a: 1 };

// Map — keys can be ANY type (objects, functions, etc.), ordered, has .size, no prototype pollution risk
const map = new Map();
map.set("a", 1);
map.set({ id: 1 }, "objectAsKey"); // impossible with plain objects
map.size; // 1 (well, 2 here) — objects have no built-in equivalent, you'd need Object.keys(obj).length
```

Rule of thumb: use plain objects for fixed, known-shape records (like a user profile). Use `Map` when keys are dynamic/unknown ahead of time, need non-string keys, or you need reliable insertion-order iteration and a `.size`.

## 22. JSON and objects

```javascript
JSON.stringify({ a: 1, b: undefined, c: function(){}, d: new Date() });
// '{"a":1,"d":"2026-09-27T..."}'  — undefined and functions are SILENTLY dropped, Date becomes a string

JSON.parse('{"a":1}'); // { a: 1 } — parses back into a real object
```

This silent-dropping behavior is exactly why `structuredClone` (section 9) is safer than the `JSON.parse(JSON.stringify())` trick for cloning.

## 23. Summary — the mental model to keep

- An object is a reference-type bag of key-value pairs; variables holding objects hold **pointers**, not the data itself.
- Every object has a hidden prototype link; property lookups walk up that chain until found or `null`.
- Classes are prototype-chain setup with nicer syntax — no new mechanism underneath.
- Copying an object with `{...obj}` or `Object.assign` is **shallow** — nested objects still share references unless you deep-clone.
- `Object.freeze`/`seal`/`defineProperty` operate on the hidden **descriptor** layer beneath normal property syntax, not the value itself.
