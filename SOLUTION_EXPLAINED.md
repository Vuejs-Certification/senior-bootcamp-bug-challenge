# Employee Table Bug Challenge — Walkthrough

## What the candidate is looking at

When you open the app it's a single screen: a plain HTML table listing five employees, with a checkbox on every row and a "select all" checkbox up in the header. Tick some rows and they highlight blue, and a little "Selected Employees" list underneath updates to show who you picked. That's the whole app — no backend, no routing, no store, just Vue 3 with Vite and Tailwind. The five employees are hard-coded, so nothing is fetched or saved.

Everything looks like it works. And that's the point — the bug isn't something you can *see*, it's something you have to *notice*.

Here's the thing to notice: **every time you tick a single checkbox, all five rows re-render, not just the one you clicked.** The challenge is to make it so that selecting a row only re-renders that one row.

The task is spelled out in [README.md](README.md) and the acceptance criteria in [CHECKLIST.md](CHECKLIST.md), and it's deliberately scoped to just two files — [AppTable.vue](src/components/AppTable.vue) (the table) and [AppTableRow.vue](src/components/AppTableRow.vue) (a single row).

**How do you even see re-renders?** There's a little instrument built into the challenge. Look at [AppTableRow.vue:24](src/components/AppTableRow.vue#L24):

```js
{{ console.log("Rendered row with id:" + row.id) }}
```

That log sits right inside the row's template, so it fires every single time that row renders. [main.js](src/main.js) wires `console` up to the StackBlitz terminal, so you literally watch the messages scroll by. Before the fix, one click prints five lines. After the fix, one click prints one line. That's your scoreboard — and notice it lives in **both** branches, because it's the measuring tape, not a bug to delete.

There's no automated test here. "Done" is the three-line checklist in [CHECKLIST.md](CHECKLIST.md): selecting one row shouldn't re-render all rows, should only re-render the selected row, and the console should prove it.

---

## Why it's broken

The root cause is one line in the parent. In the buggy version, [AppTable.vue](src/components/AppTable.vue) hands the *entire selection array* down to every single row:

```html
<!-- the challenge version -->
:selectedRows="modelValue"
```

Then each row figures out for itself whether it's selected, by searching that whole array — `selectedRows.includes(row)` — both for its highlight and for its checkbox.

That `.includes()` call is the trap. When you run it on a reactive array, Vue quietly records "this row's render depends on the whole array." All five rows do it, so all five rows are now subscribed to the same array. And when you click a checkbox, the handler mutates that array *in place* with `push` or `splice` (see `toggleRowSelection` at [AppTable.vue:34-42](src/components/AppTable.vue#L34-L42)). Mutating the array pokes every subscriber — so all five rows re-render, even though only one of them actually changed.

It's correct on screen, but it's doing five times the work it needs to, and it gets worse the more rows you add.

---

## The fix, step by step

The whole solution is **four small edits across the two files**. `App.vue` and `main.js` don't change at all. The idea is simple: instead of giving each row the whole array and making it search, let the *parent* work out the one thing each row actually cares about — "am I selected, yes or no?" — and pass down just that boolean.

**1. The parent computes the boolean and passes it down.** In [AppTable.vue:87](src/components/AppTable.vue#L87):

```diff
- :selectedRows="modelValue"
+ :isSelected="modelValue.includes(row)"
```

**2. The row now accepts a boolean instead of the whole array.** In [AppTableRow.vue:7-10](src/components/AppTableRow.vue#L7-L10):

```diff
- selectedRows: {
-   type: Array,
-   required: true,
- },
+ isSelected: {
+   type: Boolean,
+   default: false,
+ },
```

**3. The row just uses that boolean.** In [AppTableRow.vue:16](src/components/AppTableRow.vue#L16) for the highlight and [AppTableRow.vue:20](src/components/AppTableRow.vue#L20) for the checkbox:

```diff
- <tr :class="{ 'bg-blue-50': selectedRows.includes(row) }">
+ <tr :class="{ 'bg-blue-50': isSelected }">
...
- :checked="selectedRows.includes(row)"
+ :checked="isSelected"
```

That's it. The row went from "here's the whole selection, go find yourself" to "here's a yes/no, use it."

---

## Why that actually fixes it

The key move is that the `.includes(row)` call moved *out of the five child rows and into the parent*. So now there's only **one** thing watching that array — `AppTable` itself — instead of five.

When you click a checkbox, the array changes, and `AppTable` re-renders once. As it re-renders it recomputes the boolean for each row and passes those down as plain props. Now Vue does its normal check on each child: "did this child's props change?" For the four rows you *didn't* touch, `row` is the same object and `isSelected` is the same `true`/`false` as before — nothing changed, so Vue skips them entirely. Only the row whose boolean actually flipped gets re-rendered. One click, one log line.

*(One senior-level detail worth mentioning if candidates ask: the `@toggle-select="toggleRowSelection(row)"` handler is technically a brand-new function on every parent render, which would normally force the child to update. It doesn't here, because the child declares `emits: ["toggle-select"]` — Vue recognizes it as an event listener and leaves it out of the prop comparison. That's the quiet reason the skipping works.)*

---

## What this is really testing

At its heart it's about **reactive-dependency granularity** — giving a list item only the minimum it needs to know, rather than the whole collection. A senior candidate should reach for "lift the derived value up to the parent and pass a primitive down," because that's what lets Vue's per-child prop comparison short-circuit the rows that didn't change. It also quietly tests API taste: a single row shouldn't demand the entire selection model just to know its own state, so the prop should be a `Boolean` with a sensible default, not a `required` `Array`.

---

## A few things to flag when you explain it

- **The `console.log` is intentional.** It's in both branches as the measuring instrument. Nobody should be "fixing" it.
- **Ignore the leftover "Contact List" bits.** This repo was scaffolded from an older exercise, so you'll see stray traces — the `<title>` in [index.html](index.html) still says "Contact List," [tests/app.test.js](tests/app.test.js) imports composables that don't exist (so it can't even run), and a few dependencies (`@faker-js/faker`, `fuse.js`, `h3`) aren't used anywhere. None of it is part of the challenge.
- **The solution deliberately leaves some things alone.** The rows are keyed by array index at [AppTable.vue:84-85](src/components/AppTable.vue#L84-L85), and the parent keeps a local copy of the prop and mutates it in place ([AppTable.vue:22](src/components/AppTable.vue#L22), [:34-42](src/components/AppTable.vue#L34-L42)). A sharp reviewer will spot both as things you *could* improve — but they're not what's being graded, and the official fix doesn't touch them. Good to acknowledge if a candidate brings them up.
