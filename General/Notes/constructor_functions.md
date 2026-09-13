# JavaScript Constructor Functions — Notes

## 1. What is a Constructor Function?

A constructor function is a regular JS function used as a **template/blueprint** to create multiple similar objects, using the `new` keyword.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const p1 = new Person('Alex', 25);
const p2 = new Person('Sam', 30);

console.log(p1.name); // 'Alex'
console.log(p2.name); // 'Sam'
```

Convention: constructor function names start with a **capital letter** (`Person`, not `person`) to signal "call me with `new`".

---

## 2. What Actually Happens When You Use `new`

`new Person('Alex', 25)` does four things behind the scenes:

1. Creates a brand-new empty object `{}`.
2. Sets that object's internal prototype to `Person.prototype`.
3. Runs the function body with `this` bound to the new object.
4. Returns that object automatically (unless the function explicitly returns another object).

Roughly equivalent to:

```js
function Person(name, age) {
  // const this = Object.create(Person.prototype);  // (1) + (2), conceptually
  this.name = name;
  this.age = age;
  // return this;                                    // (4)
}
```

---

## 3. Without `new` — the common bug

If you forget `new`, `this` doesn't refer to a new object — in non-strict mode it refers to the global object (or is `undefined` in strict mode/modules), silently creating bugs.

```js
const p3 = Person('Jamie', 22); // forgot `new`
console.log(p3); // undefined
console.log(window.name); // 'Jamie' (leaked onto global object, in non-strict mode)
```

This is one reason many codebases prefer `class` syntax (below), since `class` constructors **throw an error** if called without `new`.

---

## 4. Adding Methods — the right way

Avoid defining methods directly inside the constructor — it creates a new copy of the function for every instance, wasting memory.

```js
// ❌ Inefficient — new function created per instance
function Person(name) {
  this.name = name;
  this.greet = function () {
    console.log(`Hi, I'm ${this.name}`);
  };
}
```

```js
// ✅ Efficient — shared across all instances via the prototype
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function () {
  console.log(`Hi, I'm ${this.name}`);
};

const p1 = new Person('Alex');
const p2 = new Person('Sam');

p1.greet(); // "Hi, I'm Alex"
p2.greet(); // "Hi, I'm Sam"

console.log(p1.greet === p2.greet); // true — same function, shared
```

---

## 5. `prototype` and the Prototype Chain

Every function has a `.prototype` object. Any object created with `new Fn()` has its internal prototype pointed at `Fn.prototype`, so it can access properties/methods defined there.

```js
console.log(p1.__proto__ === Person.prototype); // true
console.log(p1 instanceof Person);               // true
```

This is how JS does inheritance under the hood, even before `class` syntax existed.

---

## 6. Constructor Functions vs `class` (ES6)

`class` is mostly **syntactic sugar** over constructor functions + prototypes.

```js
// Constructor function style
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.greet = function () {
  console.log(`Hi, I'm ${this.name}`);
};

// Equivalent class style
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hi, I'm ${this.name}`);
  }
}
```

Both produce objects that behave the same way (`instanceof`, prototype chain, etc.). Key practical differences:

| | Constructor Function | `class` |
|---|---|---|
| Calling without `new` | Silently buggy (or `undefined`) | Throws `TypeError` |
| Hoisting | Function declarations are hoisted | Class declarations are **not** usable before definition (temporal dead zone) |
| Syntax for methods | Manually attach to `.prototype` | Built into the class body |
| Strict mode | Not automatic | Class bodies run in strict mode automatically |

---

## 7. Inheritance with Constructor Functions

Before `class extends`, inheritance was done manually like this:

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.eat = function () {
  console.log(`${this.name} is eating`);
};

function Dog(name, breed) {
  Animal.call(this, name);   // call parent constructor
  this.breed = breed;
}

// link Dog's prototype to Animal's prototype
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  console.log(`${this.name} says woof!`);
};

const d = new Dog('Rex', 'Labrador');
d.eat();  // "Rex is eating"  (inherited)
d.bark(); // "Rex says woof!"
```

The modern equivalent:

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  eat() {
    console.log(`${this.name} is eating`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  bark() {
    console.log(`${this.name} says woof!`);
  }
}
```

---

## 8. Built-in Constructor Functions

Many built-in JS types are actually constructor functions under the hood:

```js
const arr = new Array(1, 2, 3);   // same as [1, 2, 3]
const obj = new Object();          // same as {}
const str = new String('hello');   // wrapper object, NOT a primitive string
const num = new Number(5);         // wrapper object, NOT a primitive number
```

⚠️ Avoid `new String()`, `new Number()`, `new Boolean()` — they create **wrapper objects**, not primitives, which behave unexpectedly in comparisons:

```js
const a = new String('hi');
console.log(a === 'hi');       // false! (object vs primitive)
console.log(typeof a);         // 'object'
```

---

## 9. Quick Reference Cheat Sheet

```js
function Person(name, age) {   // define constructor
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function () {   // shared method
  console.log(`Hi, I'm ${this.name}`);
};

const p = new Person('Alex', 25);  // create instance

p instanceof Person;               // true
p.constructor === Person;          // true
Object.getPrototypeOf(p) === Person.prototype; // true
```

**Rules of thumb:**
- Capitalize constructor function names.
- Always call with `new` (or use `class`, which enforces it).
- Put shared methods on `.prototype`, not inside the constructor body.
- Prefer `class` syntax in modern code — it's clearer and safer, but knowing the underlying constructor-function mechanics helps you understand *why* `class` works the way it does.
