Here’s a clear explanation and practical examples for each of the Angular best practices you mentioned, especially useful when you're handling events in your Angular components.

---

### ✅ 1. **Keep event handlers simple or call methods that contain complex logic**

**Why:**
Putting too much logic directly inside your template can make it hard to read and maintain.

**❌ Bad Practice:**

```html
<button (click)="count = count + 1; logToConsole(); alert('Clicked!')">Click Me</button>
```

**✅ Good Practice:**

```html
<button (click)="handleClick()">Click Me</button>
```

```ts
handleClick() {
  this.count++;
  this.logToConsole();
  alert('Clicked!');
}
```

---

### ✅ 2. **Use the `$event` object to access event data when needed**

**Why:**
The `$event` object gives you information about the triggered DOM event, such as mouse position, key pressed, etc.

**Example: Keyboard event**

```html
<input (keyup)="onKey($event)">
```

```ts
onKey(event: KeyboardEvent) {
  console.log('Key pressed:', event.key);
}
```

**Example: Mouse event**

```html
<button (click)="onClick($event)">Click</button>
```

```ts
onClick(event: MouseEvent) {
  console.log('Mouse clicked at:', event.clientX, event.clientY);
}
```

---

### ✅ 3. **Consider event filtering for keyboard, mouse, and other events**

**Why:**
To avoid unnecessary logic execution, especially for only specific keys or conditions.

**Example: Only respond to Enter key**

```html
<input (keyup.enter)="submitForm()">
```

**Example: Only respond to right-click**

```html
<div (contextmenu)="onRightClick($event)">Right click here</div>
```

```ts
onRightClick(event: MouseEvent) {
  event.preventDefault(); // prevent default context menu
  console.log('Right-clicked at', event.clientX, event.clientY);
}
```

---

### ✅ 4. **Clean up event listeners in `ngOnDestroy` if using direct DOM event listeners**

**Why:**
To prevent **memory leaks** when you manually attach event listeners using `addEventListener`.

**Example:**

```ts
export class MyComponent implements OnInit, OnDestroy {
  private clickHandler: any;

  ngOnInit() {
    this.clickHandler = () => console.log('Document clicked');
    document.addEventListener('click', this.clickHandler);
  }

  ngOnDestroy() {
    document.removeEventListener('click', this.clickHandler);
  }
}
```

**Note:** You only need this when adding listeners manually. Angular's built-in `(click)`, `(keyup)`, etc., automatically clean up.

---

Let me know if you want this in a tabular format with snippets too!
