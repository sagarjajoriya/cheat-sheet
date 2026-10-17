# JavaScript `Map` Cheat Sheet

A `Map` stores **key → value** pairs, like a contacts list: look up a name, get the number.

| What you want | Code |
|---|---|
| Make an empty Map | `new Map()` |
| Make a Map with data | `new Map([["Ana", 22], ["Raj", 27]])` |
| Add or update | `map.set(key, value)` |
| Read | `map.get(key)` |
| Check if it exists | `map.has(key)` |
| Remove one | `map.delete(key)` |
| Count entries | `map.size` |
| Remove all | `map.clear()` |
| Loop | `for (const [k, v] of map)` |
| Just the keys / values | `map.keys()` / `map.values()` |

## Example

```javascript
const phoneBook = new Map();

phoneBook.set("Ana", "555-1234");
phoneBook.set("Raj", "555-9876");

phoneBook.get("Raj");   // "555-9876"
phoneBook.get("Zoe");   // undefined
phoneBook.has("Ana");   // true
phoneBook.size;         // 2

for (const [name, number] of phoneBook) {
  console.log(name, number);
}
```

## Counting things

```javascript
const words = ["apple", "banana", "apple"];
const counts = new Map();

for (const word of words) {
  counts.set(word, (counts.get(word) || 0) + 1);
}
// Map { "apple" → 2, "banana" → 1 }
```

## Object `{}` vs `Map`

| | Object `{}` | `Map` |
|---|---|---|
| Keys can be | Only text (or symbols) | Anything |
| Count items | `Object.keys(obj).length` | `map.size` |
| Order kept? | Mostly, with odd rules | Always insertion order |
| Hidden built-in keys? | Yes | No |
| Adding/removing often | OK | Faster |

**Rule of thumb:** fixed shape (a user with `name`, `age`) → object. Growing lookup list (contacts, counts, cache) → `Map`.

## ⚠️ Watch out

- `size` is a property, not a method: `map.size`, not `map.size()`.
- `Map` (capital M) is **not** the same as the array method `.map()`:

```javascript
[1, 2, 3].map(n => n * 2); // [2, 4, 6] → transforms an array
```

## Speed

- `get`, `set`, `has`, `delete` are about **O(1)**: just as fast with 10 items or 10 million.
- Searching an array with `.find()` is **O(n)**, so use a `Map` when you look things up by key often.
