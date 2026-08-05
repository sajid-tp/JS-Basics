
The browser creates the window and document objects and provides many APIs through them for JavaScript to interact with the browser and the web page.

### window object

The window object is the global object in the browser. It represents the browser window or tab.

It provides APIs such as:
```javascript
window.alert("Hello");
window.setTimeout(...);
window.setInterval(...);
window.addEventListener(...);
window.localStorage;
window.location;
window.history;
window.fetch(...);
```
Many of these can be used without writing window. because it's the global object:
```javascript
alert("Hello");       // Same as window.alert("Hello")
setTimeout(...);      // Same as window.setTimeout(...)
fetch(...);           // Same as window.fetch(...)
document object
```
The document object represents the web page (DOM) currently loaded in the browser.

It provides APIs for working with HTML elements.

Examples:
```javascript
document.getElementById("title");
document.querySelector(".box");
document.createElement("div");
document.addEventListener("click", ...);
```
These APIs let JavaScript:

- Find elements
- Create elements
- Remove elements
- Change text
- Change styles
- Listen for events on the document
- Relationship between them
```text
Browser
   │
   ▼
window
   │
   ├── document
   ├── localStorage
   ├── location
   ├── history
   ├── fetch()
   ├── setTimeout()
   └── addEventListener()
```
Notice that document is actually a property of window:

window.document === document // true
