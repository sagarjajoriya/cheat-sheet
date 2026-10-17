# JavaScript `sort()` Cheat Sheet

| What you want | Code |
|---|---|
| Words A → Z | `arr.sort()` |
| Numbers small → big | `arr.sort((a, b) => a - b)` |
| Numbers big → small | `arr.sort((a, b) => b - a)` |
| Objects by a number field | `arr.sort((a, b) => a.age - b.age)` |
| Objects by a text field | `arr.sort((a, b) => a.name.localeCompare(b.name))` |
| Without changing the original | `arr.toSorted(...)` |

## ⚠️ Watch out

**1. Default sort treats numbers as text**

```javascript
[10, 1, 5, 100, 25].sort();                // [1, 10, 100, 25, 5]  ❌
[10, 1, 5, 100, 25].sort((a, b) => a - b); // [1, 5, 10, 25, 100]  ✅
```

**2. `sort()` changes the original array**

```javascript
const original = [3, 1, 2];
original.sort((a, b) => a - b);  // original is now [1, 2, 3]

const copy = original.toSorted((a, b) => a - b); // original stays the same
```

## How the compare function works

`(a, b) => ...` returns a number:

- **Negative** → `a` goes first
- **Positive** → `b` goes first
- **Zero** → keep their current order

## Speed

- Time complexity: **O(n log n)** (modern engines use TimSort)
- **Stable**: equal items keep their original order
