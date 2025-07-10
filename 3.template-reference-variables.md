# Template Reference Variables

## Table of Contents

| **Angular Templates in Depth** | |
|---|-------|
| 1 | [Template Reference Variables](#template-reference-variables) |
| 2 | [Content Projection with ng-content](#content-projection-with-ng-content) |
| 3 | [ng-container and ng-template](#ng-container-and-ng-template) |
| 4 | [ViewChild and ViewChildren](#viewchild-and-viewchildren) |
| 5 | [Content Children and View Children](#content-children-and-view-children) |


## Template Reference Variables

Template reference variables provide a way to directly access elements, components, or directives in your templates.

### Key Features

* Declared with a hash symbol (`#`) followed by a name
* Access DOM elements, Angular components, or directives
* Can be referenced anywhere in the template after declaration
* Useful for form validation, focusing elements, and passing references

### Examples

**Basic Template Reference Variables**

```html
<!-- component.html -->
<!-- Reference to a DOM element -->
<input #nameInput type="text" placeholder="Enter your name">
<button (click)="greet(nameInput.value)">Greet</button>

<!-- Reference to a button and toggle its disabled state -->
<button #submitButton [disabled]="isDisabled">Submit</button>
<button (click)="submitButton.disabled = !submitButton.disabled">
  Toggle Submit Button
</button>

<!-- Accessing the value property of an input element -->
<input #quantity type="number" value="1" min="1">
<button (click)="updateQuantity(quantity.value)">Update Quantity</button>
<p>Current quantity: {{ currentQuantity }}</p>
```

```typescript
// component.ts
@Component({
  selector: 'app-template-var-example',
  templateUrl: './template-var-example.component.html'
})
export class TemplateVarExampleComponent {
  isDisabled = false;
  currentQuantity = 1;
  
  greet(name: string) {
    if (name) {
      alert(`Hello, ${name}!`);
    } else {
      alert('Please enter a name first.');
    }
  }
  
  updateQuantity(quantity: string) {
    this.currentQuantity = parseInt(quantity);
  }
}
```

**Template Variables with NgForm and NgModel**

```html
<!-- component.html -->
<form #loginForm="ngForm" (ngSubmit)="onSubmit(loginForm)">
  <div>
    <label for="username">Username</label>
    <input 
      id="username" 
      name="username" 
      #username="ngModel" 
      [(ngModel)]="user.username" 
      required 
      minlength="4">
      
    <!-- Validation messages -->
    <div *ngIf="username.invalid && (username.dirty || username.touched)">
      <div *ngIf="username.errors?.['required']">Username is required</div>
      <div *ngIf="username.errors?.['minlength']">
        Username must be at least 4 characters long
      </div>
    </div>
  </div>
  
  <div>
    <label for="password">Password</label>
    <input 
      id="password" 
      name="password" 
      type="password" 
      #password="ngModel" 
      [(ngModel)]="user.password" 
      required>
      
    <!-- Validation messages -->
    <div *ngIf="password.invalid && (password.dirty || password.touched)">
      <div *ngIf="password.errors?.['required']">Password is required</div>
    </div>
  </div>
  
  <button type="submit" [disabled]="loginForm.invalid">Login</button>
  
  <!-- Form debugging information -->
  <div class="debug-info">
    <p>Form Valid: {{ loginForm.valid }}</p>
    <p>Form Values: {{ loginForm.value | json }}</p>
    <p>Username Errors: {{ username.errors | json }}</p>
  </div>
</form>
```

```typescript
// component.ts
@Component({
  selector: 'app-form-example',
  templateUrl: './form-example.component.html'
})
export class FormExampleComponent {
  user = {
    username: '',
    password: ''
  };
  
  onSubmit(form: NgForm) {
    if (form.valid) {
      console.log('Form submitted', form.value);
      // Process login
      form.resetForm();
    }
  }
}
```

**Accessing Directive References**

```typescript
// countdown.component.ts
@Component({
  selector: 'app-countdown',
  template: `
    <div>{{ remainingTime }} seconds remaining</div>
    <button (click)="start()">Start</button>
    <button (click)="pause()">Pause</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CountdownComponent {
  @Input() startTime = 60;
  remainingTime = 0;
  private timerId: any;
  private isRunning = false;
  
  ngOnInit() {
    this.remainingTime = this.startTime;
  }
  
  start() {
    if (!this.isRunning) {
      this.isRunning = true;
      this.timerId = setInterval(() => {
        if (this.remainingTime > 0) {
          this.remainingTime--;
        } else {
          this.pause();
        }
      }, 1000);
    }
  }
  
  pause() {
    clearInterval(this.timerId);
    this.isRunning = false;
  }
  
  reset() {
    this.pause();
    this.remainingTime = this.startTime;
  }
}
```

```html
<!-- parent.component.html -->
<h2>Countdown Example</h2>
<app-countdown #countdown [startTime]="90"></app-countdown>

<div>
  <button (click)="countdown.start()">Start</button>
  <button (click)="countdown.pause()">Pause</button>
  <button (click)="countdown.reset()">Reset</button>
  <button (click)="setupFor30Seconds(countdown)">Setup for 30 Seconds</button>
</div>
```

```typescript
// parent.component.ts
@Component({
  selector: 'app-parent',
  templateUrl: './parent.component.html'
})
export class ParentComponent {
  setupFor30Seconds(countdown: CountdownComponent) {
    countdown.startTime = 30;
    countdown.reset();
  }
}
```

### Template Variables and ViewChild

```html
<!-- component.html -->
<input #nameInput type="text" placeholder="Enter your name">
<button (click)="focusNameInput()">Focus Input</button>
```

```typescript
// component.ts
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-viewchild-example',
  templateUrl: './viewchild-example.component.html'
})
export class ViewChildExampleComponent implements AfterViewInit {
  @ViewChild('nameInput') nameInputRef!: ElementRef;
  
  ngAfterViewInit() {
    // Element is available after view initialization
    console.log('Input element:', this.nameInputRef.nativeElement);
  }
  
  focusNameInput() {
    this.nameInputRef.nativeElement.focus();
  }
}
```

### Best Practices

* Choose descriptive variable names for clarity
* Use template variables only within their declaration context
* Be careful with scope - variables declared in structural directives are only available within that instance
* Prefer ViewChild for accessing elements in component code (see ViewChild section)
* Remember that the variable refers to the element, directive, or component instance - not just its value
* Avoid excessive use of template variables for complex logic - keep templates clean and move logic to the component

[Back to Top](#table-of-contents)

<!-- ## Content Projection with ng-content -->
<!-- **Basic Content Projection**
**Multiple Content Projections with Select**
**Advanced Selectors for Content Projection** -->


## Content Projection with ng-content

Content projection is a powerful Angular feature that allows you to insert content from a parent component into a child component's template. This is Angular's implementation of the "transclusion" concept from AngularJS.

### Key Features:

- **Component Composition**: Create reusable components that can host dynamic content
- **Multiple Projection Slots**: Project content into specific locations within a component
- **Conditional Projection**: Show projected content based on specific conditions

### Basic Content Projection

The simplest form of content projection uses a single `<ng-content>` element to project all content from the parent component.

```typescript
// card.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-content">
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .card {
      border: 1px solid #ccc;
      border-radius: 4px;
      padding: 16px;
      margin-bottom: 16px;
    }
  `]
})
export class CardComponent {
}
```

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <app-card>
      <h2>Card Title</h2>
      <p>This content will be projected into the card component.</p>
    </app-card>
  `
})
export class AppComponent {
}
```

**Sample Output:**
```
┌─────────────────────────────────────┐
│                                     │
│  Card Title                         │
│                                     │
│  This content will be projected     │
│  into the card component.           │
│                                     │
└─────────────────────────────────────┘
```

### Multi-Slot Content Projection

You can use the `select` attribute on `<ng-content>` to create multiple projection slots and target specific content to each slot.

```typescript
// panel.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-panel',
  template: `
    <div class="panel">
      <div class="panel-header">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="panel-body">
        <ng-content select="[body]"></ng-content>
      </div>
      <div class="panel-footer">
        <ng-content select="[footer]"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .panel {
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    .panel-header {
      background-color: #f5f5f5;
      padding: 10px;
      border-bottom: 1px solid #ddd;
    }
    .panel-body {
      padding: 15px;
    }
    .panel-footer {
      background-color: #f5f5f5;
      padding: 10px;
      border-top: 1px solid #ddd;
    }
  `]
})
export class PanelComponent {
}
```

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <app-panel>
      <div header>
        <h3>Panel Title</h3>
      </div>
      <div body>
        <p>This is the main content of the panel.</p>
        <p>Content projection makes component composition easy.</p>
      </div>
      <div footer>
        <button>Save</button>
        <button>Cancel</button>
      </div>
    </app-panel>
  `
})
export class AppComponent {
}
```

**Sample Output:**
```
┌─────────────────────────────────────┐
│ Panel Title                         │
├─────────────────────────────────────┤
│                                     │
│ This is the main content of the     │
│ panel.                              │
│                                     │
│ Content projection makes component  │
│ composition easy.                   │
│                                     │
├─────────────────────────────────────┤
│ [Save]  [Cancel]                    │
└─────────────────────────────────────┘
```

### Selection Methods for ng-content

There are several ways to select content for projection:

```typescript
// advanced-card.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-advanced-card',
  template: `
    <div class="card">
      <!-- Select by CSS class -->
      <div class="title">
        <ng-content select=".card-title"></ng-content>
      </div>
      
      <!-- Select by element type -->
      <div class="image">
        <ng-content select="img"></ng-content>
      </div>
      
      <!-- Select by attribute -->
      <div class="content">
        <ng-content select="[card-content]"></ng-content>
      </div>
      
      <!-- Default content (without a select) -->
      <div class="footer">
        <ng-content></ng-content>
      </div>
    </div>
  `
})
export class AdvancedCardComponent {
}
```

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <app-advanced-card>
      <h2 class="card-title">My Advanced Card</h2>
      <img src="path/to/image.jpg" alt="Card Image">
      <p card-content>This content goes in the content section.</p>
      <div>This is unprojected content that goes to the default ng-content.</div>
    </app-advanced-card>
  `
})
export class AppComponent {
}
```

### Conditional Content Projection

You can conditionally display projected content using `*ngIf`:

```typescript
// conditional-card.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-conditional-card',
  template: `
    <div class="card">
      <div class="card-header" *ngIf="showHeader">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>
      </div>
    </div>
  `
})
export class ConditionalCardComponent {
  @Input() showHeader = true;
}
```

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <app-conditional-card [showHeader]="true">
      <h3 header>This header will be shown</h3>
      <p>This is the card body content.</p>
    </app-conditional-card>
    
    <app-conditional-card [showHeader]="false">
      <h3 header>This header will not be shown</h3>
      <p>Only the body content appears here.</p>
    </app-conditional-card>
  `
})
export class AppComponent {
}
```

### Best Practices:

- **Use Specific Selectors**: When using multi-slot projection, use specific and meaningful selectors
- **Default Content**: Provide default content using the `ng-content` tag with no selector for fallback content
- **Keep Components Focused**: Don't overuse content projection - keep components focused on a single responsibility
- **Document Projection Slots**: Clearly document the available projection slots for other developers

[Back to Top](#table-of-contents)

## ng-container and ng-template

`ng-container` and `ng-template` are powerful structural elements in Angular that help organize templates without introducing extra DOM elements or creating conditional UI fragments.

### Key Features:

- **No DOM Rendering**: Unlike regular HTML elements, these don't add extra nodes to the DOM
- **Structural Directive Group**: Group multiple elements under a single structural directive
- **Template References**: Create reusable template fragments
- **Contextual Template Rendering**: Render templates with specific contexts

### ng-container

`ng-container` is a logical container that doesn't create any DOM element. It's useful for grouping elements or applying structural directives without affecting the DOM structure.

```typescript
// list.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-list',
  template: `
    <div class="list-container">
      <h2>User List</h2>
      <ul>
        <!-- Without ng-container, we'd need a wrapper div -->
        <ng-container *ngFor="let user of users">
          <li *ngIf="user.active">
            {{ user.name }} ({{ user.role }})
          </li>
        </ng-container>
      </ul>
    </div>
  `
})
export class ListComponent {
  users = [
    { name: 'Alice', role: 'Admin', active: true },
    { name: 'Bob', role: 'User', active: false },
    { name: 'Charlie', role: 'User', active: true },
    { name: 'Diana', role: 'Manager', active: true }
  ];
}
```

**Sample Output:**
```
User List
• Alice (Admin)
• Charlie (User)
• Diana (Manager)
```

### Using Multiple Structural Directives

A common use case for `ng-container` is when you need to apply multiple structural directives, which is not possible on a single element:

```typescript
// product-list.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-product-list',
  template: `
    <div class="products">
      <ng-container *ngFor="let product of products">
        <ng-container *ngIf="product.inStock">
          <div class="product-card">
            <h3>{{ product.name }}</h3>
            <p>Price: ${{ product.price }}</p>
            <button>Add to Cart</button>
          </div>
        </ng-container>
      </ng-container>
    </div>
  `
})
export class ProductListComponent {
  products = [
    { name: 'Laptop', price: 1200, inStock: true },
    { name: 'Phone', price: 800, inStock: false },
    { name: 'Tablet', price: 500, inStock: true },
    { name: 'Headphones', price: 100, inStock: true }
  ];
}
```

### ng-template

`ng-template` defines a template that is not rendered by default but can be used for conditional rendering or with structural directives.

```typescript
// tabs.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-tabs',
  template: `
    <div class="tabs">
      <div class="tab-buttons">
        <button 
          *ngFor="let tab of tabs" 
          (click)="activeTab = tab.id"
          [class.active]="activeTab === tab.id"
        >
          {{ tab.title }}
        </button>
      </div>
      
      <div class="tab-content">
        <!-- Each template is defined but not rendered initially -->
        <ng-template #tab1>
          <h3>First Tab Content</h3>
          <p>This is the content for the first tab.</p>
        </ng-template>
        
        <ng-template #tab2>
          <h3>Second Tab Content</h3>
          <p>This is the content for the second tab.</p>
        </ng-template>
        
        <ng-template #tab3>
          <h3>Third Tab Content</h3>
          <p>This is the content for the third tab.</p>
        </ng-template>
        
        <!-- Conditionally render the active template -->
        <div [ngSwitch]="activeTab">
          <ng-container *ngSwitchCase="'tab1'" [ngTemplateOutlet]="tab1"></ng-container>
          <ng-container *ngSwitchCase="'tab2'" [ngTemplateOutlet]="tab2"></ng-container>
          <ng-container *ngSwitchCase="'tab3'" [ngTemplateOutlet]="tab3"></ng-container>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .tab-buttons button {
      padding: 8px 16px;
      border: 1px solid #ccc;
      background: #f5f5f5;
      cursor: pointer;
    }
    .tab-buttons button.active {
      background: #fff;
      border-bottom: none;
    }
    .tab-content {
      padding: 16px;
      border: 1px solid #ccc;
      border-top: none;
    }
  `]
})
export class TabsComponent {
  tabs = [
    { id: 'tab1', title: 'Tab 1' },
    { id: 'tab2', title: 'Tab 2' },
    { id: 'tab3', title: 'Tab 3' }
  ];
  activeTab = 'tab1';
}
```

### ngTemplateOutlet

`ngTemplateOutlet` is a directive that renders an `ng-template` at a specified location. It can also pass context to the template:

```typescript
// template-outlet.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-template-outlet',
  template: `
    <div class="container">
      <h2>Template Outlet Example</h2>
      
      <!-- Define a reusable template -->
      <ng-template #userTemplate let-user="user" let-isAdmin="isAdmin">
        <div class="user-card" [class.admin]="isAdmin">
          <h3>{{ user.name }}</h3>
          <p>{{ user.email }}</p>
          <p>Role: {{ isAdmin ? 'Administrator' : 'Regular User' }}</p>
        </div>
      </ng-template>
      
      <!-- Use template with different contexts -->
      <div class="user-list">
        <h3>Regular Users</h3>
        <ng-container *ngFor="let user of regularUsers">
          <ng-container 
            [ngTemplateOutlet]="userTemplate" 
            [ngTemplateOutletContext]="{ user: user, isAdmin: false }"
          ></ng-container>
        </ng-container>
        
        <h3>Administrators</h3>
        <ng-container *ngFor="let user of adminUsers">
          <ng-container 
            [ngTemplateOutlet]="userTemplate" 
            [ngTemplateOutletContext]="{ user: user, isAdmin: true }"
          ></ng-container>
        </ng-container>
      </div>
    </div>
  `,
  styles: [`
    .user-card {
      border: 1px solid #ddd;
      padding: 10px;
      margin-bottom: 10px;
      border-radius: 4px;
    }
    .user-card.admin {
      border-color: #f44336;
      background-color: #ffebee;
    }
  `]
})
export class TemplateOutletComponent {
  regularUsers = [
    { name: 'John Smith', email: 'john@example.com' },
    { name: 'Sarah Johnson', email: 'sarah@example.com' }
  ];
  
  adminUsers = [
    { name: 'Mike Admin', email: 'mike@example.com' },
    { name: 'Emma Admin', email: 'emma@example.com' }
  ];
}
```

### Creating Custom Structural Directives

You can use `ng-template` to create custom structural directives:

```typescript
// unless.directive.ts
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}

  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      // If condition is false and view isn't created yet, create it
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      // If condition is true and view exists, destroy it
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
```

```typescript
// component using custom directive
import { Component } from '@angular/core';

@Component({
  selector: 'app-unless-example',
  template: `
    <div>
      <button (click)="show = !show">Toggle Content</button>
      <p *appUnless="show">
        This content is shown when "show" is false (opposite of *ngIf)
      </p>
    </div>
  `
})
export class UnlessExampleComponent {
  show = false;
}
```

### Comparison: ng-container vs ng-template

| Feature | ng-container | ng-template |
|---------|-------------|-------------|
| DOM Rendering | Doesn't render a DOM element | Doesn't render unless explicitly used |
| Primary Use | Grouping elements without affecting DOM | Defining template fragments for later use |
| Rendering | Renders its children immediately | Renders only when referenced |
| Use with Structural Directives | Can host structural directives | Is used by structural directives |
| Context | Doesn't create a new context | Can define a context with input variables |

### Best Practices:

- **Use ng-container** to avoid extra DOM elements when grouping elements
- **Use ng-template** to define reusable template fragments
- **Keep Templates Small**: Break large templates into smaller, maintainable pieces
- **Meaningful Template References**: Use clear naming for template reference variables
- **Document Template Contexts**: When using complex context objects, document the expected structure

[Back to Top](#table-of-contents)

## ViewChild and ViewChildren

`ViewChild` and `ViewChildren` are decorators in Angular that provide access to DOM elements, directives, or components within the view of a component.

### Key Features:

- **Direct Element Access**: Query DOM elements in the component's view
- **Component Communication**: Access child component instances and their properties/methods
- **Directive Access**: Query instances of directives used in the view
- **Template References**: Access elements marked with template reference variables

### ViewChild

`ViewChild` is used to query a single element or directive from the view.

```typescript
// view-child.component.ts
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-view-child',
  template: `
    <div class="container">
      <h2>ViewChild Example</h2>
      
      <!-- Reference to a DOM element -->
      <input #nameInput type="text" placeholder="Enter your name">
      
      <button (click)="focusInput()">Focus Input</button>
      
      <!-- Reference to a child component -->
      <app-counter #counter></app-counter>
      
      <button (click)="incrementCounter()">Increment Counter</button>
    </div>
  `
})
export class ViewChildComponent implements AfterViewInit {
  // Access the input element
  @ViewChild('nameInput') nameInputRef: ElementRef;
  
  // Access the counter component
  @ViewChild('counter') counterComponent: any; // Replace 'any' with actual component type
  
  // Alternative way to query by component type instead of template reference
  // @ViewChild(CounterComponent) counterComp: CounterComponent;
  
  ngAfterViewInit() {
    // ViewChild references are only available in ngAfterViewInit
    console.log('Input element:', this.nameInputRef.nativeElement);
    console.log('Counter component:', this.counterComponent);
  }
  
  focusInput() {
    // Directly interact with the DOM element
    this.nameInputRef.nativeElement.focus();
  }
  
  incrementCounter() {
    // Call a method on the child component
    this.counterComponent.increment();
  }
}
```

```typescript
// counter.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div class="counter">
      <h3>Counter: {{ count }}</h3>
    </div>
  `
})
export class CounterComponent {
  count = 0;
  
  increment() {
    this.count++;
  }
  
  decrement() {
    this.count--;
  }
  
  reset() {
    this.count = 0;
  }
}
```

### Static Query Option

The `@ViewChild` decorator has a static option that determines when the query is resolved:

```typescript
// Static query resolved before change detection
@ViewChild('someElement', { static: true }) someElement: ElementRef;

// Non-static query (default) resolved after first change detection
@ViewChild('dynamicElement') dynamicElement: ElementRef;
```

Static queries can be accessed in `ngOnInit`, while non-static queries are only guaranteed to be available in `ngAfterViewInit`.

### ViewChildren

`ViewChildren` queries multiple elements or directives that match the selector:

```typescript
// view-children.component.ts
import { Component, ViewChildren, QueryList, AfterViewInit } from '@angular/core';
import { TabComponent } from './tab.component';

@Component({
  selector: 'app-view-children',
  template: `
    <div class="tab-container">
      <h2>ViewChildren Example</h2>
      
      <!-- Multiple child components -->
      <app-tab title="Tab 1" [active]="activeTab === 0">
        Content for Tab 1
      </app-tab>
      
      <app-tab title="Tab 2" [active]="activeTab === 1">
        Content for Tab 2
      </app-tab>
      
      <app-tab title="Tab 3" [active]="activeTab === 2">
        Content for Tab 3
      </app-tab>
      
      <div class="tab-buttons">
        <button (click)="activateTab(i)" *ngFor="let tab of tabs; let i = index">
          {{ tab.title }}
        </button>
        <button (click)="logTabInfo()">Log Tab Info</button>
      </div>
    </div>
  `
})
export class ViewChildrenComponent implements AfterViewInit {
  @ViewChildren(TabComponent) tabComponents: QueryList<TabComponent>;
  
  tabs = [
    { title: 'Tab 1' },
    { title: 'Tab 2' },
    { title: 'Tab 3' }
  ];
  
  activeTab = 0;
  
  ngAfterViewInit() {
    // QueryList is accessible in ngAfterViewInit
    console.log('Tab components:', this.tabComponents);
    
    // Subscribe to changes in the QueryList
    this.tabComponents.changes.subscribe(tabList => {
      console.log('Tabs changed:', tabList);
    });
  }
  
  activateTab(index: number) {
    this.activeTab = index;
    
    // Access individual items in the QueryList
    this.tabComponents.forEach((tab, i) => {
      tab.active = i === index;
    });
  }
  
  logTabInfo() {
    // Convert QueryList to array and use array methods
    const tabsArray = this.tabComponents.toArray();
    
    console.log('Number of tabs:', this.tabComponents.length);
    console.log('First tab:', this.tabComponents.first);
    console.log('Last tab:', this.tabComponents.last);
    console.log('All tabs:', tabsArray);
  }
}
```

```typescript
// tab.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-tab',
  template: `
    <div class="tab" [class.active]="active">
      <h3>{{ title }}</h3>
      <div class="content">
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .tab {
      display: none;
      border: 1px solid #ccc;
      padding: 10px;
      margin-bottom: 10px;
    }
    .tab.active {
      display: block;
    }
  `]
})
export class TabComponent {
  @Input() title: string;
  @Input() active = false;
}
```

### Reading Multiple Query Types

You can specify what to read from the query using the `read` option:

```typescript
// read-example.component.ts
import { Component, ViewChild, ViewChildren, ElementRef, QueryList, AfterViewInit } from '@angular/core';
import { NgModel } from '@angular/forms';

@Component({
  selector: 'app-read-example',
  template: `
    <div class="form-container">
      <h2>Read Option Example</h2>
      
      <!-- Same element, different query targets -->
      <input #nameInput [(ngModel)]="name" name="name">
      
      <div class="info">
        <p>Current value: {{ name }}</p>
        <button (click)="logInfo()">Log Information</button>
      </div>
    </div>
  `
})
export class ReadExampleComponent implements AfterViewInit {
  name = '';
  
  // Get the ElementRef of the input
  @ViewChild('nameInput') inputElement: ElementRef;
  
  // Get the NgModel directive from the same element
  @ViewChild('nameInput', { read: NgModel }) inputModel: NgModel;
  
  ngAfterViewInit() {
    console.log('Input element:', this.inputElement);
    console.log('NgModel directive:', this.inputModel);
  }
  
  logInfo() {
    // Access the DOM element
    console.log('Input value (from DOM):', this.inputElement.nativeElement.value);
    
    // Access the NgModel information
    console.log('Model value:', this.inputModel.value);
    console.log('Is model valid?', this.inputModel.valid);
    console.log('Is model dirty?', this.inputModel.dirty);
  }
}
```

### Dynamic Components with ViewChild

`ViewChild` can be used with dynamic components created using `ComponentFactoryResolver`:

```typescript
// dynamic-component.component.ts
import { 
  Component, 
  ViewChild, 
  ComponentFactoryResolver, 
  ViewContainerRef, 
  AfterViewInit 
} from '@angular/core';
import { AlertComponent } from './alert.component';

@Component({
  selector: 'app-dynamic-component',
  template: `
    <div class="container">
      <h2>Dynamic Component with ViewChild</h2>
      
      <button (click)="showAlert('info', 'This is an information message')">
        Show Info Alert
      </button>
      <button (click)="showAlert('warning', 'Warning! This is important')">
        Show Warning Alert
      </button>
      <button (click)="showAlert('error', 'Error occurred! Please try again')">
        Show Error Alert
      </button>
      
      <!-- This container will host our dynamic components -->
      <div class="alerts-container">
        <ng-container #alertContainer></ng-container>
      </div>
    </div>
  `
})
export class DynamicComponentComponent implements AfterViewInit {
  @ViewChild('alertContainer', { read: ViewContainerRef }) alertContainer: ViewContainerRef;
  
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {}
  
  ngAfterViewInit() {
    console.log('Alert container:', this.alertContainer);
  }
  
  showAlert(type: 'info' | 'warning' | 'error', message: string) {
    // Clear previous alerts
    this.alertContainer.clear();
    
    // Create component factory
    const factory = this.componentFactoryResolver.resolveComponentFactory(AlertComponent);
    
    // Create component instance
    const componentRef = this.alertContainer.createComponent(factory);
    
    // Set component inputs
    componentRef.instance.type = type;
    componentRef.instance.message = message;
    
    // Auto-remove after 3 seconds
    setTimeout(() => {
      componentRef.destroy();
    }, 3000);
  }
}
```

```typescript
// alert.component.ts
import { Component, Input, OnInit } from '@angular/core';

@Component({
  selector: 'app-alert',
  template: `
    <div class="alert" [ngClass]="'alert-' + type">
      <span class="close" (click)="onClose()">&times;</span>
      <strong>{{ typeTitle }}:</strong> {{ message }}
    </div>
  `,
  styles: [`
    .alert {
      padding: 15px;
      margin-bottom: 15px;
      border-radius: 4px;
    }
    .alert-info {
      background-color: #d9edf7;
      border: 1px solid #bce8f1;
      color: #31708f;
    }
    .alert-warning {
      background-color: #fcf8e3;
      border: 1px solid #faebcc;
      color: #8a6d3b;
    }
    .alert-error {
      background-color: #f2dede;
      border: 1px solid #ebccd1;
      color: #a94442;
    }
    .close {
      cursor: pointer;
      float: right;
      font-weight: bold;
    }
  `]
})
export class AlertComponent implements OnInit {
  @Input() type: 'info' | 'warning' | 'error' = 'info';
  @Input() message: string;
  
  typeTitle: string;
  
  ngOnInit() {
    // Set title based on type
    this.typeTitle = {
      'info': 'Information',
      'warning': 'Warning',
      'error': 'Error'
    }[this.type];
  }
  
  onClose() {
    // In a real app, we would emit an event to the parent
    console.log('Alert closed');
  }
}
```

### Best Practices:

- **Use Appropriate Lifecycle Hooks**: 
  - For static queries (static: true), use `ngOnInit`
  - For non-static queries, use `ngAfterViewInit`
- **Avoid DOM Manipulation**: When possible, use data binding rather than direct DOM manipulation
- **Proper Error Handling**: Always check if the view child exists before accessing it
- **Specify Return Type**: Always specify the correct type for @ViewChild and @ViewChildren to get proper type checking
- **Prefer Component Interaction**: Use @Input/@Output for parent-child communication when possible, and @ViewChild only when necessary
- **Use read Option Carefully**: Only use the read option when you need a specific token from an element
- **Handle QueryList Changes**: Subscribe to changes on QueryList when dealing with dynamic content

[Back to Top](#table-of-contents)