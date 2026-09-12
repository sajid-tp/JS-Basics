### Object.fromEntries()

Object.fromEntries() is a built-in JavaScript method that converts a list of key-value pairs into a normal JavaScript object.

Basic example
```
const entries = [
  ["name", "Sajid"],
  ["age", 25],
  ["email", "sajid@gmail.com"]
];

const obj = Object.fromEntries(entries);

console.log(obj);
```
Result:
```
{
  name: "Sajid",
  age: 25,
  email: "sajid@gmail.com"
}
```
Think of it like this

It converts:
```
[
  ["name", "Sajid"],
  ["age", 25]
]
```
into:
```
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
