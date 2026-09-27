# JavaScript Promises & Async/Await — Complete Notes

## 1. Why Promises exist — the problem they solve

Before Promises, async code (network calls, timers, file reads) used **callbacks** — a function passed in to be called later, when the work finished:

```javascript
getUser(1, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments);
    }, (err) => console.log(err));
  }, (err) => console.log(err));
}, (err) => console.log(err));
```

This is **"callback hell"** — deeply nested, hard to read, hard to handle errors consistently (every level needs its own error callback). A Promise is an object that represents **a value that doesn't exist yet, but will (or will fail to) in the future** — and it gives you a standard, chainable way to react to that, instead of nesting callbacks.

## 2. What a Promise actually is

A Promise is an object with an internal **state** and an internal **value**. That's it — conceptually:

```javascript
{
  state: "pending" | "fulfilled" | "rejected",
  value: undefined | <resolved value> | <rejection reason>
}
```

It always starts as `pending`, and can move to exactly **one** of two final states — never both, and never back again:

```
pending ──resolve(value)──▶ fulfilled (settled)
pending ──reject(reason)──▶ rejected  (settled)
```

Once it's `fulfilled` or `rejected`, it's called **settled**, and it stays that way forever — a Promise cannot change state twice.

## 3. Creating a Promise from scratch

```javascript
const myPromise = new Promise((resolve, reject) => {
  const success = true;

  setTimeout(() => {
    if (success) {
      resolve("Data loaded!");     // moves state to fulfilled, value = "Data loaded!"
    } else {
      reject("Something broke");   // moves state to rejected, value = "Something broke"
    }
  }, 1000);
});
```

- `new Promise(executor)` — the `executor` function runs **immediately, synchronously**, the instant the Promise is created.
- `resolve` and `reject` are functions **given to you** by the Promise constructor — you just call whichever one applies when the async work finishes.
- Everything inside `setTimeout` is async — the `resolve`/`reject` call happens later, whenever that timer fires.

## 4. Consuming a Promise — `.then()`, `.catch()`, `.finally()`

```javascript
myPromise
  .then((value) => {
    console.log("Success:", value);
  })
  .catch((error) => {
    console.log("Failed:", error);
  })
  .finally(() => {
    console.log("Runs no matter what — success or failure");
  });
```

- `.then(onFulfilled, onRejected)` — actually takes **two** optional callbacks (success handler, failure handler), though people almost always write the failure one as a separate `.catch()` instead.
- `.catch(fn)` is just shorthand for `.then(undefined, fn)`.
- `.finally(fn)` runs regardless of outcome, and doesn't receive the value/error at all — used for cleanup (hiding a loading spinner, etc.).

```javascript
// These two are functionally identical:
promise.then(onSuccess, onError);
promise.then(onSuccess).catch(onError);
```

The difference matters once chaining is involved (see section 6) — `.then(a, b)`'s `b` only catches errors from the *original* promise, while a separate `.catch()` after `.then(a)` catches errors from **both** the original promise AND from inside `a` itself. The chained version is almost always what you want.

## 5. The golden rule: `.then()` always returns a NEW Promise

This is the single most important mechanical fact about Promises. Every `.then()` call doesn't just "run a callback" — it **returns a brand new Promise**, whose value depends on what your callback returned:

| What your `.then()` callback does | What the new Promise resolves to |
|---|---|
| `return someValue;` | Resolves immediately with `someValue` |
| `return anotherPromise;` | Waits for `anotherPromise`, then adopts its state/value |
| `throw new Error(...)` | The new Promise **rejects** with that error |
| (returns nothing) | Resolves with `undefined` |

This is exactly what makes chaining work:

```javascript
fetchUser(1)
  .then((user) => fetchPosts(user.id))     // returns a Promise → next .then() waits for IT
  .then((posts) => fetchComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch((err) => console.log("Any failure anywhere above lands here:", err));
```

Compare this to the callback-hell version in section 1 — same logic, flat instead of nested, and **one single `.catch()`** handles errors from *any* step in the whole chain, because a rejection at any point skips straight past all remaining `.then()`s until it finds a `.catch()`.

## 6. Chaining in detail — tracing the flow

```javascript
Promise.resolve(1)
  .then((val) => {
    console.log(val);       // 1
    return val + 1;
  })
  .then((val) => {
    console.log(val);       // 2
    throw new Error("oops");
  })
  .then((val) => {
    console.log("skipped"); // never runs — a throw skips straight to the next .catch()
  })
  .catch((err) => {
    console.log(err.message); // "oops"
    return "recovered";       // a .catch() can also return a value, resuming the chain as FULFILLED
  })
  .then((val) => {
    console.log(val); // "recovered" — chain continues normally after being caught
  });
```

**Key insight:** once an error is caught by a `.catch()`, the chain is "healed" — subsequent `.then()`s run normally again, unless the `.catch()` itself also throws.

## 7. `Promise.resolve()` and `Promise.reject()`

Shortcuts to create an already-settled Promise, without the `new Promise(...)` ceremony:

```javascript
Promise.resolve(5);              // an already-fulfilled Promise with value 5
Promise.reject("failed");        // an already-rejected Promise with reason "failed"

// Useful for normalizing — if you're not sure if something is a Promise:
Promise.resolve(maybeAPromise).then((val) => { ... });
```

## 8. Running multiple Promises together

This is where Promises genuinely outperform plain callbacks — four different built-in combinators for four different needs:

### `Promise.all()` — wait for ALL, fail fast on first rejection

```javascript
Promise.all([fetchUser(1), fetchPosts(1), fetchComments(1)])
  .then(([user, posts, comments]) => {
    console.log(user, posts, comments); // all three, in the SAME order you passed them in
  })
  .catch((err) => {
    console.log("At least one failed:", err); // if ANY one rejects, the WHOLE thing rejects immediately
  });
```
Use when: you need **everything** to succeed, and one failure means the whole operation is pointless (e.g., loading a dashboard that needs 3 API calls to render at all).

### `Promise.allSettled()` — wait for ALL, never fails, tells you each outcome

```javascript
Promise.allSettled([fetchUser(1), fetchPosts(1)])
  .then((results) => {
    results.forEach((result) => {
      if (result.status === "fulfilled") {
        console.log("Got:", result.value);
      } else {
        console.log("Failed:", result.reason);
      }
    });
  });
// results = [
//   { status: "fulfilled", value: {...} },
//   { status: "rejected", reason: "network error" }
// ]
```
Use when: you want to attempt several independent things and know the outcome of **each one**, without one failure cancelling your ability to see the others (e.g., uploading 5 files — some might fail, but you still want to know which ones succeeded).

### `Promise.race()` — settles as soon as the FIRST one settles (success or failure)

```javascript
Promise.race([fetchData(), timeout(5000)])
  .then((result) => console.log("Whichever finished first:", result))
  .catch((err) => console.log("Whichever failed first:", err));
```
Use when: implementing a **timeout** — race your real request against a Promise that rejects after N seconds; whichever finishes first wins.

```javascript
function timeout(ms) {
  return new Promise((_, reject) => setTimeout(() => reject(new Error("Timed out")), ms));
}
```

### `Promise.any()` — settles as soon as the FIRST one succeeds, ignores failures unless ALL fail

```javascript
Promise.any([fetchFromServerA(), fetchFromServerB(), fetchFromServerC()])
  .then((firstSuccess) => console.log("First one that worked:", firstSuccess))
  .catch((err) => {
    // only reached if ALL of them failed
    console.log(err); // an AggregateError containing all individual errors
  });
```
Use when: you have several equivalent sources (mirrors, fallback servers) and just want **whichever succeeds first**, ignoring individual failures unless every single one fails.

### Quick comparison table

| Method | Resolves when | Rejects when | Use case |
|---|---|---|---|
| `Promise.all` | all fulfill | any one rejects | need everything, fail together |
| `Promise.allSettled` | all settle (never rejects itself) | never | need every outcome, regardless of failures |
| `Promise.race` | first one settles (either way) | first one settles as a rejection | timeouts, "whichever is fastest" |
| `Promise.any` | first one fulfills | only if ALL reject | fallback sources, "first success wins" |

## 9. `async` / `await` — Promises with nicer syntax

`async`/`await` is **not a different mechanism** — it's syntax sugar sitting directly on top of Promises. Every `async` function **always returns a Promise**, and `await` is just a way to "pause" until a Promise settles, without writing `.then()`.

```javascript
// Promise-chain version
function getUser() {
  return fetchUser(1)
    .then((user) => fetchPosts(user.id))
    .then((posts) => posts[0]);
}

// async/await version — same behavior, reads top-to-bottom like sync code
async function getUser() {
  const user = await fetchUser(1);
  const posts = await fetchPosts(user.id);
  return posts[0];
}
```

**What `await` actually does:** it pauses execution of the `async` function (without blocking the rest of the program) until the Promise on its right settles. If it fulfills, `await` "unwraps" the value. If it rejects, `await` **throws** that rejection as a regular JS exception — which is why error handling switches to `try/catch`:

```javascript
async function getUser() {
  try {
    const user = await fetchUser(1);   // if this rejects, jumps straight to catch
    const posts = await fetchPosts(user.id);
    return posts[0];
  } catch (err) {
    console.log("Something failed:", err);
    throw err; // or handle it here and don't rethrow
  }
}
```

## 10. `async` functions ALWAYS return a Promise — even without `await`

```javascript
async function greet() {
  return "hello";
}

greet(); // NOT "hello" — it's a Promise: Promise { "hello" }
greet().then((val) => console.log(val)); // "hello"
```

Even a `return` of a plain value gets automatically wrapped in a resolved Promise, because `async` functions are Promises by contract — you can `.then()` or `await` the result of calling one, always, no exceptions.

```javascript
async function mightFail() {
  throw new Error("nope");
}

mightFail().catch((err) => console.log(err.message)); // "nope" — a throw inside async = a REJECTED promise, not a thrown exception at the call site
```

## 11. Sequential vs parallel `await` — a critical performance distinction

```javascript
// SEQUENTIAL — each await waits for the previous one to finish first. Total time = sum of all three.
async function sequential() {
  const a = await taskA(); // waits ~1s
  const b = await taskB(); // THEN starts, waits another ~1s
  const c = await taskC(); // THEN starts, waits another ~1s
  // total: ~3 seconds
}

// PARALLEL — all three start immediately, THEN we wait for all of them. Total time = the SLOWEST one.
async function parallel() {
  const [a, b, c] = await Promise.all([taskA(), taskB(), taskC()]);
  // total: ~1 second (assuming each task takes ~1s and they run concurrently)
}
```

**Rule of thumb:** if the tasks **don't depend on each other's results**, always use `Promise.all` + one `await`, not multiple sequential `await`s — sequential `await`ing of independent work is one of the most common real-world performance mistakes.

## 12. `await` inside loops — another common pitfall

```javascript
// SLOW — sequential, one request finishes before the next STARTS
async function loadAllSlow(ids) {
  const results = [];
  for (const id of ids) {
    const data = await fetchById(id); // blocks the loop until THIS ONE finishes
    results.push(data);
  }
  return results;
}

// FAST — all requests fire at once, then we wait for all together
async function loadAllFast(ids) {
  const promises = ids.map((id) => fetchById(id)); // fires ALL requests immediately (no await here yet)
  return Promise.all(promises); // NOW wait for all of them together
}
```

Use the sequential loop version only when you **genuinely need each step to finish before starting the next** (e.g., each request depends on the previous result, or you're intentionally rate-limiting).

## 13. Top-level `await`

Modern JS (in ES modules) allows `await` directly at the top of a file, outside any `async function`:

```javascript
// inside a .mjs file or a <script type="module">
const data = await fetch("/api/data").then((r) => r.json());
console.log(data);
```

Not allowed in regular (non-module) `.js` scripts or inside `CommonJS` (`require`-based) files without wrapping it in an `async` IIFE:

```javascript
(async () => {
  const data = await fetch("/api/data");
})();
```

## 14. Converting an old callback-based function into a Promise

A very common real task — "promisifying" a legacy callback API:

```javascript
function oldStyleRead(path, callback) {
  fs.readFile(path, (err, data) => {
    if (err) callback(err, null);
    else callback(null, data);
  });
}

// Wrapped into a Promise
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    oldStyleRead(path, (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// Now usable with async/await
async function main() {
  const data = await readFilePromise("./file.txt");
  console.log(data);
}
```

Node's built-in `util.promisify()` does exactly this automatically for any standard Node-style `(err, data) => {}` callback function.

## 15. The Microtask Queue — why Promises run "almost immediately," but not truly synchronously

This is the deepest, most-asked-in-interviews part of Promises. JavaScript has two separate queues for scheduling work after the current synchronous code finishes:

- **Macrotask queue** — `setTimeout`, `setInterval`, I/O, UI rendering.
- **Microtask queue** — Promise `.then/.catch/.finally` callbacks, `queueMicrotask()`.

**The rule: after every single synchronous block of code finishes, JS fully empties the ENTIRE microtask queue before touching even one macrotask.**

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0); // macrotask — queued for LATER

Promise.resolve().then(() => console.log("3")); // microtask — queued, but BEFORE any macrotask

console.log("4");

// Output: 1, 4, 3, 2
```

Walkthrough:
1. `"1"` logs immediately (synchronous).
2. `setTimeout` schedules `"2"` into the **macrotask** queue — even with `0` ms delay, it does NOT run next.
3. `.then()` schedules `"3"` into the **microtask** queue.
4. `"4"` logs immediately (synchronous) — the rest of the main script finishes.
5. Main script is done → JS checks the **microtask queue first** → runs `"3"`.
6. Microtask queue is now empty → JS finally moves to the **macrotask queue** → runs `"2"`.

This is *why* a `0ms` `setTimeout` never "wins" against a Promise — microtasks always drain completely before the next macrotask, no matter how small the macrotask's delay is.

## 16. Error handling comparison — Promise chain vs async/await

```javascript
// Promise chain style
function load() {
  return fetchUser()
    .then((user) => fetchPosts(user.id))
    .catch((err) => {
      console.log("Failed:", err);
      throw err; // re-throw if caller needs to know too
    });
}

// async/await style — equivalent behavior
async function load() {
  try {
    const user = await fetchUser();
    return await fetchPosts(user.id);
  } catch (err) {
    console.log("Failed:", err);
    throw err;
  }
}
```

One subtlety: `return fetchPosts(user.id)` vs `return await fetchPosts(user.id)` inside a `try` — **without `await`, an error thrown by `fetchPosts` will NOT be caught by this function's `catch` block**, because the function already returned the (still-pending, unawaited) Promise before it had a chance to reject inside this stack frame. Always `await` a Promise you're returning from inside a `try` block if you want its rejection to be caught locally.

## 17. Unhandled Promise rejections

If a Promise rejects and **nothing** ever calls `.catch()` on it (or an `await` inside a `try/catch`), it becomes an **unhandled rejection** — in browsers this logs a console warning; in Node it can even crash the process depending on version/config.

```javascript
async function risky() {
  throw new Error("boom");
}
risky(); // no .catch(), no try/catch around it → UnhandledPromiseRejection warning
```

Always either `.catch()` a Promise, or `await` it inside a `try/catch` — never leave one dangling.

## 18. Summary — the mental model to keep

- A Promise is an object tracking one of three states: `pending → fulfilled` or `pending → rejected`, and once settled, it never changes again.
- `.then()` always returns a **new** Promise — that's the entire mechanism that makes chaining and `async/await` possible.
- `async` functions **always** return a Promise; `await` unwraps a Promise's value or throws its rejection as a catchable exception.
- Use `Promise.all` for "all must succeed," `allSettled` for "tell me every outcome," `race` for timeouts, `any` for "first success wins."
- Independent `await`s in sequence waste time — use `Promise.all` when tasks don't depend on each other.
- Microtasks (Promises) always fully drain before the next macrotask (`setTimeout` etc.), which is why Promise callbacks "feel" faster than even a `0ms` timeout.
