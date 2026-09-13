# JavaScript `Map` — Notes

## 1. What is a `Map`?

A `Map` is a built-in JS data structure that stores **key-value pairs**, just like a plain object `{}` — but with some important differences and extra features.

```js
const map = new Map();
```

---

## 2. Map vs Plain Object

| Feature | `Object` | `Map` |
|---|---|---|
| Key types | Strings / Symbols only | **Any type** (object, function, number, etc.) |
| Key order | Not guaranteed (mostly insertion order for strings) | Guaranteed insertion order |
| Size | `Object.keys(obj).length` | `map.size` (direct property) |
| Iteration | Need `Object.keys()`/`entries()` | Directly iterable with `for...of` |
| Performance | Good for static shape data | Better for frequent add/remove |
| Default keys | Has prototype keys (`toString`, etc.) | No default keys — a clean slate |

**Rule of thumb:** use `Map` when keys aren't just strings, or when you need guaranteed order, frequent additions/removals, or an easy `.size`.

---

## 3. Creating a Map

```js
// Empty map
const map1 = new Map();

// From an array of [key, value] pairs
const map2 = new Map([
  ['name', 'Alex'],
  ['age', 25]
]);
```

---

## 4. Core Methods

```js
const map = new Map();

map.set('email', 'a@example.com'); // add/update a key
map.get('email');                  // 'a@example.com' - read a value
map.has('email');                  // true - check existence
map.delete('email');               // true - removes the key
map.clear();                       // removes all entries
map.size;                          // number of entries (property, not a function)
```

`set()` returns the map itself, so calls can be **chained**:

```js
map.set('a', 1).set('b', 2).set('c', 3);
```

---

## 5. Any Value Can Be a Key

This is the biggest difference from objects.

```js
const objKey = { id: 1 };
const funcKey = () => {};

const map = new Map();
map.set(objKey, 'this is an object key');
map.set(funcKey, 'this is a function key');
map.set(NaN, 'even NaN works');

map.get(objKey); // 'this is an object key'
```

⚠️ Note: keys are compared by reference for objects — `{}` !== `{}`, so two different object literals are two different keys even if they look the same.

---

## 6. Iterating a Map

Maps are directly iterable — no need for `Object.entries()`.

```js
const map = new Map([
  ['email', 'a@example.com'],
  ['password', 'hunter2']
]);

// entries (default iteration)
for (const [key, value] of map) {
  console.log(key, value);
}

// keys only
for (const key of map.keys()) {
  console.log(key);
}

// values only
for (const value of map.values()) {
  console.log(value);
}

// forEach style
map.forEach((value, key) => {
  console.log(key, value);
});
```

---

## 7. Converting Between Map and Other Structures

```js
// Map -> Array of entries
const arr = [...map]; 
// or
const arr2 = Array.from(map.entries());

// Map -> Object (only safe if keys are strings/symbols)
const obj = Object.fromEntries(map);

// Object -> Map
const map3 = new Map(Object.entries(obj));

// Array of pairs -> Map
const map4 = new Map([['x', 1], ['y', 2]]);
```

This is the same `Object.fromEntries()` pattern used to turn `FormData` into a plain object — `FormData` behaves like a `Map` (iterable `[key, value]` pairs), which is why the same conversion trick works there too.

---

## 8. Common Use Cases

- Caching / memoization (arbitrary keys, including objects/functions)
- Counting occurrences (word frequency, etc.)
- Storing metadata against DOM nodes or component instances
- Any time key order or non-string keys matter

```js
// Counting example
const words = ['a', 'b', 'a', 'c', 'b', 'a'];
const count = new Map();

for (const word of words) {
  count.set(word, (count.get(word) || 0) + 1);
}
// Map(3) { 'a' => 3, 'b' => 2, 'c' => 1 }
```

---

## 9. WeakMap (quick mention)

A `WeakMap` is a variant where:
- Keys **must be objects** (no primitives).
- Keys are held **weakly** — if there's no other reference to the key object, it can be garbage collected.
- Not iterable, no `.size`, no `.clear()`.

Useful for attaching private/hidden data to objects without causing memory leaks.

```js
const wm = new WeakMap();
const obj = {};
wm.set(obj, 'secret data');
```

---

## 10. Quick Reference Cheat Sheet

```js
new Map()                  // create
map.set(key, value)        // add/update
map.get(key)                // read
map.has(key)                // check
map.delete(key)             // remove one
map.clear()                 // remove all
map.size                    // count
[...map]                    // to array of entries
Object.fromEntries(map)     // to plain object (string keys only)
```
