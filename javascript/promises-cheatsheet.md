# JavaScript Promises Cheat Sheet

A **Promise** is like a restaurant buzzer: a result that isn't ready yet. Later it either **works** (food is ready 🟢) or **fails** (they ran out 🔴).

## The 3 states

| State | Meaning |
|---|---|
| **pending** | Not finished yet |
| **fulfilled** | It worked, and here's the result |
| **rejected** | It failed, and here's the error |

Once fulfilled or rejected, a Promise is **settled** and never changes again.

## Quick reference

| What you want | Code |
|---|---|
| Do something when it works | `promise.then(result => ...)` |
| Handle an error | `promise.catch(error => ...)` |
| Run code either way | `promise.finally(() => ...)` |
| Wait for the result (modern way) | `const result = await promise;` |
| Allow `await` in a function | `async function name() { ... }` |
| Handle errors with `await` | `try { ... } catch (error) { ... }` |
| Make your own | `new Promise((resolve, reject) => ...)` |
| Instantly successful Promise | `Promise.resolve(value)` |
| Instantly failed Promise | `Promise.reject(error)` |
| Run many at once, all must work | `await Promise.all([p1, p2])` |
| Run many, get every result | `await Promise.allSettled([p1, p2])` |
| First to finish (good or bad) | `await Promise.race([p1, p2])` |
| First to succeed | `await Promise.any([p1, p2])` |

## `.then()` / `.catch()` / `.finally()`

```javascript
fetch("https://api.example.com/user")
  .then(response => response.json())
  .then(user => console.log(user))
  .catch(error => console.log("Something went wrong:", error))
  .finally(() => console.log("Done, success or not"));
```

Whatever you `return` inside a `.then()` is passed to the next `.then()`.

## `async` / `await` (same thing, easier to read)

```javascript
async function loadUser() {
  try {
    const response = await fetch("https://api.example.com/user");
    const user = await response.json();
    console.log(user);
  } catch (error) {
    console.log("Something went wrong:", error);
  }
}
```

- `await` waits for the Promise and gives you the result.
- `await` only works inside an `async` function (or at the top level of a module).
- An `async` function **always returns a Promise**, even if you `return 5`.

## Making your own Promise

```javascript
const orderFood = new Promise((resolve, reject) => {
  const foodAvailable = true;

  setTimeout(() => {
    if (foodAvailable) {
      resolve("🍕 Your pizza!");    // success → fulfilled
    } else {
      reject("Sorry, we ran out"); // failure → rejected
    }
  }, 2000);
});

orderFood
  .then(food => console.log(food))
  .catch(error => console.log(error));
```

A handy one-liner, a "sleep" helper:

```javascript
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
await sleep(1000); // waits 1 second
```

## Running several Promises

```javascript
// Slow: one after another, about 3 seconds
const user  = await getUser();
const posts = await getPosts();
const likes = await getLikes();

// Fast: all at the same time, about 1 second
const [user, posts, likes] = await Promise.all([
  getUser(),
  getPosts(),
  getLikes(),
]);
```

| Method | Succeeds when | Fails when |
|---|---|---|
| `Promise.all` | **All** succeed | **Any one** fails |
| `Promise.allSettled` | Always (gives every result) | Never |
| `Promise.race` | The first to finish succeeded | The first to finish failed |
| `Promise.any` | **Any one** succeeds | **All** fail (`AggregateError`) |

**Remember:**
- **all**: "I need everything"
- **allSettled**: "tell me how everything went"
- **race**: "whoever finishes first, good or bad"
- **any**: "I just need one that works"

### `Promise.allSettled` result shape

```javascript
const results = await Promise.allSettled([p1, p2]);
// [
//   { status: "fulfilled", value: ... },
//   { status: "rejected",  reason: ... }
// ]
```

### `Promise.any` when everything fails

```javascript
try {
  const data = await Promise.any([p1, p2, p3]);
} catch (error) {
  console.log(error.errors); // [error1, error2, error3]
}
```

## ⚠️ Common mistakes

**1. Forgetting `await`**

```javascript
const user = fetch("/user");       // ❌ a Promise, not the data
const user = await fetch("/user"); // ✅ the actual response
```

Seeing `Promise { <pending> }` in the console usually means a missing `await`.

**2. Not handling errors**

An unhandled failure shows "Uncaught (in promise)". Always add `.catch()` or `try` / `catch`.

**3. Expecting code to wait without `await`**

```javascript
console.log("1");
fetch("/user").then(() => console.log("2"));
console.log("3");
// Prints: 1, 3, 2
```

**4. Using `await` inside `forEach`**

`forEach` doesn't wait for `await`. Use `for...of` instead:

```javascript
for (const id of ids) {
  await saveItem(id);   // ✅ waits for each one
}

ids.forEach(async id => {
  await saveItem(id);   // ❌ doesn't wait; everything fires at once
});
```
