Here’s a **comprehensive set of notes** on `ngDoCheck` in Angular, focused on **custom change detection** — when and how to use it, how it compares to `ngOnChanges`, and best practices.

---

## 📘 What is `ngDoCheck`?

`ngDoCheck()` is a lifecycle hook in Angular that allows you to **implement custom change detection logic**.

* It's called **during every change detection run**, regardless of whether Angular detected any changes.
* It provides **more control** than `ngOnChanges()`, especially for **deep object comparisons** or when Angular’s default change detection doesn't notice a change.

---

## 🧠 When to Use `ngDoCheck`

Use `ngDoCheck` when:

| Use Case                                                                       | Why                                           |
| ------------------------------------------------------------------------------ | --------------------------------------------- |
| You want to detect **mutations to objects/arrays** that don’t change reference | Angular won't trigger `ngOnChanges` for these |
| You want **fine-grained performance control**                                  | E.g., avoid expensive checks until necessary  |
| You’re integrating with **legacy APIs** or **custom change trackers**          | External changes not tracked by Angular       |

---

## ⚠️ Why Not Always Use `ngDoCheck`?

* It runs **very frequently** — on every change detection cycle — so if it's heavy, it can impact performance.
* It’s **manual and error-prone** if misused.

---

## ✅ Example: Detecting Array Mutations

```ts
@Component({ ... })
export class MyComponent implements DoCheck {
  @Input() items!: any[];
  private previousItemsLength = 0;

  ngDoCheck() {
    if (this.items.length !== this.previousItemsLength) {
      console.log('Items changed!');
      this.previousItemsLength = this.items.length;
    }
  }
}
```

🟡 This will catch `.push()` or `.pop()`, which `ngOnChanges` **misses** (because the array reference doesn't change).

---

## 🆚 `ngDoCheck` vs `ngOnChanges`

| Feature                       | `ngOnChanges`              | `ngDoCheck`                    |
| ----------------------------- | -------------------------- | ------------------------------ |
| Triggered when input changes? | ✅ Yes (with new reference) | ✅ Yes (every change detection) |
| Detects deep changes?         | ❌ No                       | ✅ Yes (if you implement logic) |
| Manual comparison needed?     | ❌ No (Angular handles it)  | ✅ Yes (you write the logic)    |
| Frequency                     | When input changes         | Every CD cycle                 |

---

## 🔧 Best Practices

1. **Avoid heavy logic inside `ngDoCheck()`** — offload to services or debounce changes.
2. **Use `IterableDiffers` or `KeyValueDiffers`** (Angular utilities) for optimized comparison.
3. **Prefer immutable updates + `ngOnChanges`** if possible — cleaner and more performant.

---

## ✅ Example with `KeyValueDiffers`

```ts
constructor(private differs: KeyValueDiffers) {}

private userDiffer = this.differs.find({}).create();

ngDoCheck() {
  const changes = this.userDiffer.diff(this.user);
  if (changes) {
    changes.forEachChangedItem(item => {
      console.log('Changed:', item.key, item.currentValue);
    });
  }
}
```

---

## 🚦 Summary

* `ngDoCheck` gives **full control over detecting changes**, including deep ones.
* Use it **carefully and intentionally**, especially when Angular’s default behavior is insufficient.
* Combine it with `IterableDiffers` / `KeyValueDiffers` for scalable, performant logic.

---

Would you like an interactive assignment to practice `ngDoCheck` with an array or object mutation detection scenario?
