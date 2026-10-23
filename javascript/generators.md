# JavaScript Generators Cheat Sheet

A normal function runs start to finish, like a movie in a cinema. A **generator** can **pause, hand you a value, and resume later**, like Netflix.

## Quick reference

| What you want | Code |
|---|---|
| Create a generator | `function* name() { ... }` |
| Hand out a value and pause | `yield value` |
| Get the "remote control" | `const g = name();` |
| Get the next value | `g.next()` → `{ value, done }` |
| Send a value in | `g.next(value)` |
| Loop over all values | `for (const x of name())` |
| Turn into an array | `[...name()]` or `Array.from(name())` |
| Yield from another generator/array | `yield* other()` |
| End it early from inside | `return value` |
| Stop it from outside | `g.return(value)` |
| Throw an error inside it | `g.throw(error)` |
| Make an object loopable | `*[Symbol.iterator]() { ... }` |
| Async generator | `async function* name() { ... }` |
| Loop over an async generator | `for await (const x of name())` |

## The basics

```javascript
function* countToThree() {
  yield 1;
  yield 2;
  yield 3;
}

const counter = countToThree(); // nothing runs yet

counter.next(); // { value: 1, done: false }
counter.next(); // { value: 2, done: false }
counter.next(); // { value: 3, done: false }
counter.next(); // { value: undefined, done: true }

for (const num of countToThree()) console.log(num); // 1, 2, 3
[...countToThree()]; // [1, 2, 3]
```

Code between `yield`s only runs when you ask for the next value.

## Why use generators?

**1. Endless sequences**

```javascript
function* idMaker() {
  let id = 1;
  while (true) yield id++; // safe: pauses at every yield
}

const ids = idMaker();
ids.next().value; // 1
ids.next().value; // 2
```

**2. Saving memory (lazy evaluation)**

An array of a million numbers uses **O(n)** memory. A generator makes one at a time: **O(1)** memory.

```javascript
function* numbers() {
  for (let i = 0; i < 1_000_000; i++) yield i;
}
```

**3. Remembering where you are, even inside nested loops**

```javascript
function* gridCells(rows, cols) {
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      yield [r, c];
    }
  }
}

[...gridCells(2, 2)]; // [[0,0], [0,1], [1,0], [1,1]]
```

## Sending values in

```javascript
function* chat() {
  const name = yield "What's your name?";
  yield `Hello, ${name}!`;
}

const c = chat();
c.next().value;      // "What's your name?"  (first next() just starts it)
c.next("Ana").value; // "Hello, Ana!"
```

## `return` and `yield*`

```javascript
function* shortOne() {
  yield 1;
  return "finished"; // ends here
}
// next() → { value: 1, done: false }
// next() → { value: "finished", done: true }
// ⚠️ for...of and [...spread] ignore the returned value

function* letters() {
  yield* ["a", "b"]; // hand off to an array or another generator
  yield "c";
}
[...letters()]; // ["a", "b", "c"]
```

## Making your own objects loopable

```javascript
const playlist = {
  songs: ["Song A", "Song B", "Song C"],
  *[Symbol.iterator]() {
    yield* this.songs;
  },
};

for (const song of playlist) console.log(song);
[...playlist]; // ["Song A", "Song B", "Song C"]
```

## Async generators (generators + Promises)

```javascript
async function* loadPages() {
  let url = "https://api.example.com/items?page=1";
  while (url) {
    const response = await fetch(url);
    const data = await response.json();
    yield data.items;
    url = data.nextPageUrl; // null on the last page → loop ends
  }
}

for await (const items of loadPages()) {
  console.log(items);
  if (items.some(item => item.id === 42)) break; // remaining pages never fetched
}
```

## Generator vs closure

A closure can also remember values between calls:

```javascript
function idMaker() {
  let id = 1;
  return () => id++;
}
const nextId = idMaker();
nextId(); // 1
nextId(); // 2
```

| Situation | Use |
|---|---|
| Just "give me the next value" (counter, ID maker) | **Closure** |
| Need extra methods like `reset()` or `peek()` | **Closure** |
| Logic with steps, or pausing inside loops | **Generator** |
| Want `for...of`, spread, or destructuring to work | **Generator** |
| Sequence has a natural end | **Generator** |

**Closures remember a value. Generators remember where you are in a process.**

## Async generator vs alternatives (loading pages)

| Situation | Best choice |
|---|---|
| Few pages, need them all, simple code | **Plain async function + array** |
| Known page count, want maximum speed | **`Promise.all`** (all at once) |
| Just "do X with each page" | **Callback** (`loadPages(onPage)`) |
| Might stop early (search, "load more") | **Async generator** |
| Unknown number of pages (`nextPageUrl`) | **Async generator** |
| Huge data, can't hold it all in memory | **Async generator** |

## ⚠️ Common mistakes

**1. Forgetting the `*`**

```javascript
function count() { yield 1; }  // ❌ SyntaxError
function* count() { yield 1; } // ✅
```

**2. Expecting the call to run the code**

`countToThree()` runs nothing. Use `.next()`, `for...of`, or spread.

**3. Reusing a finished generator**

```javascript
const g = countToThree();
[...g]; // [1, 2, 3]
[...g]; // []  ← already used up; call countToThree() again
```

**4. Spreading an infinite generator**

```javascript
[...idMaker()]; // ❌ never ends, freezes your program
```

**5. Trying an arrow function**

There's no arrow generator. Use `function*`.
