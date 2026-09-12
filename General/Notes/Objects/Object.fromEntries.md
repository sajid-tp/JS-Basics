### Object.fromEntries()

Object.fromEntries() is a built-in JavaScript method that converts a list of key-value pairs into a normal JavaScript object.

Basic example
```javascript
const entries = [
  ["name", "Sajid"],
  ["age", 25],
  ["email", "sajid@gmail.com"]
];

const obj = Object.fromEntries(entries);

console.log(obj);
```
Result:
```javascript
{
  name: "Sajid",
  age: 25,
  email: "sajid@gmail.com"
}
```
Think of it like this

It converts:
```javascript
[
  ["name", "Sajid"],
  ["age", 25]
]
```
into:
```javascript
{
  name: "Sajid",
  age: 25
}
```
So the basic idea is:
```
key-value pairs
      ↓
Object.fromEntries()
      ↓
JavaScript object
```
