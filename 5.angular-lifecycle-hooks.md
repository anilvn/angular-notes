# Angular Lifecycle Hooks

## Table of Contents

| # | Topic |
|---|-------|
| **Introduction** |
| 1 | [Understanding Angular Lifecycle Hooks](#understanding-angular-lifecycle-hooks) |
| **Main Lifecycle Hooks** |
| 2 | [ngOnInit - Component Initialization](#ngonInit-component-initialization) |
| 3 | [ngOnChanges - Input Property Changes](#ngonchanges-input-property-changes) |
| 4 | [ngAfterViewInit - View Initialization](#ngafterviewinit-view-initialization) |
| 5 | [ngOnDestroy - Component Cleanup](#ngondestroy-component-cleanup) |
| **Additional Lifecycle Hooks** |
| 6 | [ngDoCheck - Custom Change Detection](#ngdocheck-custom-change-detection) |
| 7 | [ngAfterContentInit & ngAfterContentChecked](#ngaftercontentinit--ngaftercontentchecked) |
| 8 | [ngAfterViewChecked - After View Check](#ngafterviewchecked-after-view-check) |
| **Best Practices** |
| 9 | [Lifecycle Hooks Best Practices](#lifecycle-hooks-best-practices) |
| 10 | [Lifecycle Execution Sequence](#lifecycle-execution-sequence) |

## Understanding Angular Lifecycle Hooks

Angular components and directives go through a lifecycle as they are created, updated, and eventually destroyed. Angular provides **lifecycle hook methods** that give you the opportunity to tap into these key moments and execute custom code in response to changes.

The lifecycle hooks allow you to:
* Initialize components with data from external sources
* Respond to property changes
* Interact with the DOM after components are fully initialized
* Clean up resources before a component is destroyed


Each hook serves a specific purpose and is called at a particular time in the component lifecycle.

### Key Concepts:

* **Lifecycle Phases**: Angular components go through several phases during their lifetime
* **Hook Methods**: Special methods prefixed with "ng" that execute at specific points in time
* **Interface Implementation**: For each hook, there's a corresponding interface that can be implemented

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-example',
  template: '<div>Example Component</div>'
})
export class ExampleComponent implements OnInit {
  // Implementing the OnInit interface
  ngOnInit() {
    // This code runs when the component is initialized
    console.log('Component initialized!');
  }
}
```

[Back to Top](#angular-lifecycle-hooks)

## ngOnInit - Component Initialization

The `ngOnInit()` lifecycle hook is one of the most commonly used hooks in Angular. It's called after Angular has initialized all data-bound properties of a component.

### Key Features:

* Runs **once** after the first `ngOnChanges()`
* Perfect for initialization tasks that require bound properties to be set
* More appropriate than constructor for most initialization logic
* Guaranteed to run only once in a component's lifecycle

### When to use ngOnInit:

* Fetching initial data from a service
* Setting up subscriptions
* Performing complex initialization logic

```typescript
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';
import { User } from './user.model';

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user">
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
    </div>
  `
})
export class UserProfileComponent implements OnInit {
  user: User | null = null;
  
  // Inject the service in the constructor
  constructor(private userService: UserService) {
    // Only basic initialization here
    console.log('Constructor called');
  }

  ngOnInit(): void {
    // Fetch data after component is initialized
    console.log('ngOnInit called');
    this.userService.getCurrentUser().subscribe(
      (userData) => {
        this.user = userData;
        console.log('User data loaded:', this.user);
      }
    );
  }
}

// Sample output in console:
// Constructor called
// ngOnInit called
// User data loaded: {name: "John Doe", email: "john@example.com"}
```

### Constructor vs ngOnInit:

| Constructor | ngOnInit |
|-------------|----------|
| Used for basic initialization and DI | Used for complex initialization logic |
| Runs before input properties are set | Runs after input properties are set |
| Called only once when class is instantiated | Called once after first ngOnChanges |
| Not specific to Angular | Angular-specific lifecycle hook |
| Cannot access @Input properties | Can safely access @Input properties |

[Back to Top](#angular-lifecycle-hooks)

## ngOnChanges - Input Property Changes

The `ngOnChanges()` lifecycle hook is called whenever one or more data-bound input properties of a component change. This hook receives a `SimpleChanges` object that contains the current and previous values of the changed properties.

### Key Features:

* Called **before** `ngOnInit()` and whenever input properties change
* Receives a `SimpleChanges` object with current and previous values
* Only triggered for properties decorated with `@Input()`
* Not called for internal property changes

```typescript
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-change-detector',
  template: `
    <div>
      <h3>Current Value: {{ value }}</h3>
      <p *ngIf="previousValue !== undefined">Previous Value: {{ previousValue }}</p>
      <p *ngIf="changeCount > 0">Change Count: {{ changeCount }}</p>
    </div>
  `
})
export class ChangeDetectorComponent implements OnChanges {
  @Input() value: string = '';
  previousValue: string | undefined;
  changeCount: number = 0;

  ngOnChanges(changes: SimpleChanges): void {
    // Check if our specific input property has changed
    if (changes['value']) {
      // Increment our change counter
      this.changeCount++;
      
      // Store the previous value for display
      this.previousValue = changes['value'].previousValue;
      
      console.log('Value changed from:', changes['value'].previousValue, 
                  'to:', changes['value'].currentValue);
      console.log('First change?', changes['value'].firstChange);
    }
  }
}
```

### Parent Component Usage:

```typescript
@Component({
  selector: 'app-parent',
  template: `
    <app-change-detector [value]="currentValue"></app-change-detector>
    <button (click)="updateValue()">Update Value</button>
  `
})
export class ParentComponent {
  currentValue: string = 'initial';
  
  updateValue() {
    this.currentValue = 'updated-' + new Date().toISOString();
  }
}

// Sample output in console when button is clicked:
// Value changed from: initial to: updated-2025-02-05T15:30:45.123Z
// First change? false
```

### SimpleChanges Object Structure:

```typescript
// The structure of the SimpleChanges object
interface SimpleChanges {
  [propName: string]: SimpleChange;
}

// Each property change is represented by a SimpleChange object
interface SimpleChange {
  currentValue: any;  // The current value
  previousValue: any; // The previous value
  firstChange: boolean; // Whether this is the first change
}
```

[Back to Top](#angular-lifecycle-hooks)

## ngAfterViewInit - View Initialization

The `ngAfterViewInit()` lifecycle hook is called after Angular has fully initialized a component's view and all of its child views. This is the perfect hook for any logic that needs to access or manipulate DOM elements.

### Key Features:

* Runs after the component's view and child views are fully initialized
* Called **once** during the component lifecycle
* Allows safe access to view children (with `@ViewChild` or `@ViewChildren`)
* Perfect for integrating with third-party DOM libraries
* Perform DOM manipulation (like adding focus, dimensions, etc.)
* Start animations or third-party DOM-based libraries (like charts, sliders)


```typescript
import { Component, AfterViewInit, ViewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-chart',
  template: `
    <h3>Chart Example</h3>
    <div #chartContainer class="chart-container" style="width: 400px; height: 300px;"></div>
  `
})
export class ChartComponent implements AfterViewInit {
  // Reference to the chart container element in the DOM
  @ViewChild('chartContainer') chartContainer!: ElementRef;
  
  constructor() {
    // The ViewChild reference is NOT available here yet!
    console.log('Constructor - chartContainer available?', this.chartContainer ? true : false);
    // Output: Constructor - chartContainer available? false
  }
  
  ngAfterViewInit(): void {
    // Now the view is initialized and we can safely access the chart container
    console.log('ngAfterViewInit - chartContainer available?', this.chartContainer ? true : false);
    // Output: ngAfterViewInit - chartContainer available? true
    
    // This is where you would typically initialize third-party libraries
    this.initializeChart();
  }
  
  private initializeChart(): void {
    // Get the native DOM element
    const container = this.chartContainer.nativeElement;
    
    // In a real scenario, you might initialize a chart library like this:
    console.log('Initializing chart in element:', container);
    
    // Example of using a third-party chart library (commented as pseudo-code)
    /*
    const chart = new ChartLibrary({
      container: container,
      data: [10, 20, 30, 40, 50],
      options: {
        type: 'bar',
        title: 'Sample Chart'
      }
    });
    */
  }
}
```

### Using ngAfterViewInit with Dynamic Components:

```typescript
import { Component, AfterViewInit, ViewChild, ComponentFactoryResolver, ViewContainerRef } from '@angular/core';
import { DynamicComponent } from './dynamic.component';

@Component({
  selector: 'app-dynamic-loader',
  template: `
    <h3>Dynamic Component Loading Example</h3>
    <ng-container #dynamicComponentContainer></ng-container>
  `
})
export class DynamicLoaderComponent implements AfterViewInit {
  @ViewChild('dynamicComponentContainer', { read: ViewContainerRef }) 
  container!: ViewContainerRef;
  
  constructor(private componentFactoryResolver: ComponentFactoryResolver) {}
  
  ngAfterViewInit(): void {
    // Create a factory for our dynamic component
    const componentFactory = this.componentFactoryResolver.resolveComponentFactory(DynamicComponent);
    
    // Clear the container
    this.container.clear();
    
    // Create the component and add it to the container
    const componentRef = this.container.createComponent(componentFactory);
    
    // We can set properties on the dynamic component instance
    componentRef.instance.title = 'Dynamically Created Component';
    
    console.log('Dynamic component created and mounted to DOM');
  }
}
```

> **Important Note:** When modifying the view in `ngAfterViewInit()`, you might encounter the "Expression has changed after it was checked" error. This happens because Angular has already run a change detection cycle. To avoid this, you can use `setTimeout()` or `ngZone.runOutsideAngular()` for view modifications.

[Back to Top](#angular-lifecycle-hooks)

## ngOnDestroy - Component Cleanup

The `ngOnDestroy()` lifecycle hook is called just before Angular destroys a component. This is the ideal place to perform cleanup operations to prevent memory leaks.

### Key Features:

* Called just before Angular destroys the component
* Perfect for cleaning up resources and subscriptions
* Essential for preventing memory leaks
* Only called once in the component lifecycle

### When to use ngOnDestroy:

* Unsubscribing from Observable subscriptions
* Detaching event handlers
* Clearing timers and intervals
* Releasing references to DOM elements
* Removing event listeners

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { DataService } from './data.service';
import { Subscription, interval } from 'rxjs';

@Component({
  selector: 'app-data-monitor',
  template: `
    <div>
      <h3>Data Monitor</h3>
      <p>Count: {{ counter }}</p>
      <p>Latest Data: {{ latestData || 'No data yet' }}</p>
    </div>
  `
})
export class DataMonitorComponent implements OnInit, OnDestroy {
  counter: number = 0;
  latestData: string | null = null;
  
  // We'll keep track of our subscriptions
  private timerSubscription!: Subscription;
  private dataSubscription!: Subscription;
  
  constructor(private dataService: DataService) {
    console.log('Component created');
  }
  
  ngOnInit(): void {
    // Set up a timer that updates every second
    this.timerSubscription = interval(1000).subscribe(() => {
      this.counter++;
      console.log('Timer tick:', this.counter);
    });
    
    // Subscribe to a data service
    this.dataSubscription = this.dataService.getData().subscribe(
      (data) => {
        this.latestData = data;
        console.log('New data received:', data);
      }
    );
    
    // Add a window event listener
    window.addEventListener('resize', this.handleResize);
  }
  
  // We need to store the handler as a property to be able to remove it later
  private handleResize = () => {
    console.log('Window resized');
  };
  
  ngOnDestroy(): void {
    // Clean up subscriptions to prevent memory leaks
    if (this.timerSubscription) {
      this.timerSubscription.unsubscribe();
      console.log('Timer subscription unsubscribed');
    }
    
    if (this.dataSubscription) {
      this.dataSubscription.unsubscribe();
      console.log('Data subscription unsubscribed');
    }
    
    // Remove event listeners
    window.removeEventListener('resize', this.handleResize);
    console.log('Resize event listener removed');
    
    console.log('Component destroyed - all resources cleaned up');
  }
}

// Sample output when component is destroyed:
// Timer subscription unsubscribed
// Data subscription unsubscribed
// Resize event listener removed
// Component destroyed - all resources cleaned up
```

### Common Memory Leak Sources:

| Source | Potential Issue | Cleanup Solution |
|--------|-----------------|------------------|
| RxJS Subscriptions | Subscriptions continue to receive events | Call `subscription.unsubscribe()` |
| Event Listeners | Listeners remain active and reference component | Remove with `removeEventListener()` |
| Timers | setInterval continues running | Clear with `clearInterval(timerId)` |
| WebSocket Connections | Connection stays open | Close connection with `socket.close()` |
| Third-party Libraries | May hold references to component | Call library's cleanup methods |

[Back to Top](#angular-lifecycle-hooks)

## ngDoCheck - Custom Change Detection

The `ngDoCheck()` lifecycle hook gives you a chance to implement your own custom change detection. It's called during every change detection run, even when Angular's default change detection wouldn't detect any changes.

> **Warning:** Use `ngDoCheck()` carefully as it runs very frequently and can impact performance. Only implement the minimum logic needed and avoid complex operations.


### Key Features:

* Called during **every** change detection cycle
* Allows implementing custom change detection logic
* Runs very frequently, so keep it lightweight
* Can be used for detecting changes that Angular doesn't catch automatically
* It's called **during every change detection run**, regardless of whether Angular detected any changes.
* It provides **more control** than `ngOnChanges()`, especially for **deep object comparisons** or when Angular’s default change detection doesn't notice a change.

### When to use ngDoCheck:

* When you need to detect changes in objects or arrays that Angular doesn't track by reference
* When integrating with non-Angular code that modifies data
* For implementing custom change detection logic

Use `ngDoCheck` when:

| Use Case                                                                       | Why                                           |
| ------------------------------------------------------------------------------ | --------------------------------------------- |
| You want to detect **mutations to objects/arrays** that don’t change reference | Angular won't trigger `ngOnChanges` for these |
| You want **fine-grained performance control**                                  | E.g., avoid expensive checks until necessary  |
| You’re integrating with **legacy APIs** or **custom change trackers**          | External changes not tracked by Angular       |

---
### ⚠️ Why Not Always Use `ngDoCheck`?

* It runs **very frequently** — on every change detection cycle — so if it's heavy, it can impact performance.
* It’s **manual and error-prone** if misused.

### 🔧 Best Practices

1. **Avoid heavy logic inside `ngDoCheck()`** — offload to services or debounce changes.
2. **Use `IterableDiffers` or `KeyValueDiffers`** (Angular utilities) for optimized comparison.
3. **Prefer immutable updates + `ngOnChanges`** if possible — cleaner and more performant.
    * Use it **carefully and intentionally**, especially when Angular’s default behavior is insufficient.
    * Combine it with `IterableDiffers` / `KeyValueDiffers` for scalable, performant logic.

### 🆚 `ngDoCheck` vs `ngOnChanges`

| Feature                       | `ngOnChanges`              | `ngDoCheck`                    |
| ----------------------------- | -------------------------- | ------------------------------ |
| Triggered when input changes? | ✅ Yes (with new reference) | ✅ Yes (every change detection) |
| Detects deep changes?         | ❌ No                       | ✅ Yes (if you implement logic) |
| Manual comparison needed?     | ❌ No (Angular handles it)  | ✅ Yes (you write the logic)    |
| Frequency                     | When input changes         | Every CD cycle                 |

---

### ✅ Example: Detecting Array Mutations

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

### ✅ Example with `KeyValueDiffers`

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
### Ex

```typescript
import { Component, DoCheck, Input, KeyValueDiffer, KeyValueDiffers } from '@angular/core';

@Component({
  selector: 'app-object-tracker',
  template: `
    <div>
      <h3>Object Change Tracker</h3>
      <div *ngIf="changes.length">
        <h4>Recent Changes:</h4>
        <ul>
          <li *ngFor="let change of changes">{{ change }}</li>
        </ul>
      </div>
    </div>
  `
})
export class ObjectTrackerComponent implements DoCheck {
  @Input() object: any = {};
  changes: string[] = [];
  
  // Create a differ to detect changes in the object
  private differ: KeyValueDiffer<string, any>;
  
  constructor(private differs: KeyValueDiffers) {
    // Initialize the differ in the constructor
    this.differ = this.differs.find({}).create();
  }
  
  ngDoCheck(): void {
    // Use the differ to detect changes in the object
    if (this.object) {
      const changes = this.differ.diff(this.object);
      
      if (changes) {
        console.log('Changes detected in input object');
        
        changes.forEachAddedItem(item => {
          this.changes.push(`Added: ${item.key} = ${item.currentValue}`);
          console.log(`Added: ${item.key} = ${item.currentValue}`);
        });
        
        changes.forEachRemovedItem(item => {
          this.changes.push(`Removed: ${item.key}`);
          console.log(`Removed: ${item.key}`);
        });
        
        changes.forEachChangedItem(item => {
          this.changes.push(`Changed: ${item.key} from ${item.previousValue} to ${item.currentValue}`);
          console.log(`Changed: ${item.key} from ${item.previousValue} to ${item.currentValue}`);
        });
        
        // Limit the number of changes we display
        if (this.changes.length > 5) {
          this.changes = this.changes.slice(-5);
        }
      }
    }
  }
}
```

### Parent Component Using the Object Tracker:

```typescript
@Component({
  selector: 'app-parent',
  template: `
    <app-object-tracker [object]="dataObject"></app-object-tracker>
    <button (click)="addProperty()">Add Property</button>
    <button (click)="changeProperty()">Change Property</button>
    <button (click)="removeProperty()">Remove Property</button>
  `
})
export class ParentComponent {
  dataObject: any = { name: 'Initial Object', value: 42 };
  
  addProperty() {
    this.dataObject.newProp = 'New Property';
    console.log('Added property to object', this.dataObject);
  }
  
  changeProperty() {
    this.dataObject.value = Math.floor(Math.random() * 100);
    console.log('Changed property value', this.dataObject);
  }
  
  removeProperty() {
    if (this.dataObject.newProp) {
      delete this.dataObject.newProp;
      console.log('Removed property from object', this.dataObject);
    }
  }
}
```


[Back to Top](#angular-lifecycle-hooks)

## ngAfterContentInit & ngAfterContentChecked

These lifecycle hooks are related to projected content in a component using `<ng-content>`. They allow you to work with content that is projected into your component from a parent component.

### ngAfterContentInit

* Called once after Angular projects external content into the component's view
* Runs after the first `ngDoCheck()`
* Perfect for initializing anything based on content projected via `<ng-content>`

### ngAfterContentChecked

* Called after Angular checks the content projected into the component
* Runs after every `ngDoCheck()` and after `ngAfterContentInit()`
* Can be used to react to changes in the projected content

```typescript
import { 
  Component, 
  AfterContentInit, 
  AfterContentChecked, 
  ContentChild,
  ContentChildren,
  QueryList
} from '@angular/core';
import { TabComponent } from './tab.component';

@Component({
  selector: 'app-tab-container',
  template: `
    <div class="tab-container">
      <div class="tab-header">
        <div 
          *ngFor="let tab of tabs; let i = index" 
          class="tab-title"
          [class.active]="selectedTab === i"
          (click)="selectTab(i)">
          {{ tab.title }}
        </div>
      </div>
      <div class="tab-content">
        <!-- Here we render the selected tab's content -->
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .tab-container { border: 1px solid #ccc; border-radius: 4px; }
    .tab-header { display: flex; border-bottom: 1px solid #ccc; }
    .tab-title { padding: 10px; cursor: pointer; }
    .tab-title.active { font-weight: bold; border-bottom: 2px solid blue; }
    .tab-content { padding: 16px; }
  `]
})
export class TabContainerComponent implements AfterContentInit, AfterContentChecked {
  // Query for all tab components projected into this container
  @ContentChildren(TabComponent) tabs!: QueryList<TabComponent>;
  
  // Track the selected tab index
  selectedTab: number = 0;
  
  ngAfterContentInit() {
    console.log('ngAfterContentInit - ContentChildren are available');
    console.log(`Found ${this.tabs.length} tabs`);
    
    // Initialize: Hide all tabs except the selected one
    this.updateTabVisibility();
    
    // Subscribe to changes in the tabs collection
    this.tabs.changes.subscribe(() => {
      console.log('Tabs collection changed');
      this.updateTabVisibility();
    });
  }
  
  ngAfterContentChecked() {
    console.log('ngAfterContentChecked - Content has been checked');
  }
  
  selectTab(index: number) {
    this.selectedTab = index;
    this.updateTabVisibility();
  }
  
  private updateTabVisibility() {
    if (this.tabs) {
      // Hide all tabs
      this.tabs.forEach((tab, index) => {
        tab.visible = (index === this.selectedTab);
      });
    }
  }
}
```

### Tab Component:

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-tab',
  template: `
    <div *ngIf="visible">
      <ng-content></ng-content>
    </div>
  `
})
export class TabComponent {
  @Input() title: string = 'Tab';
  visible: boolean = false;
}
```

### Using the Tab System in a Parent Component:

```typescript
@Component({
  selector: 'app-tabs-demo',
  template: `
    <h2>Tabs Example</h2>
    <app-tab-container>
      <app-tab title="First Tab">
        <h3>First Tab Content</h3>
        <p>This is the content of the first tab.</p>
      </app-tab>
      <app-tab title="Second Tab">
        <h3>Second Tab Content</h3>
        <p>This is the content of the second tab.</p>
        <button (click)="doSomething()">Click Me</button>
      </app-tab>
      <app-tab title="Third Tab">
        <h3>Third Tab Content</h3>
        <p>This is the content of the third tab.</p>
      </app-tab>
    </app-tab-container>
  `
})
export class TabsDemoComponent {
  doSomething() {
    console.log('Button clicked in second tab');
  }
}

// Sample console output during initialization:
// ngAfterContentInit - ContentChildren are available
// Found 3 tabs
```

The key difference between `ngAfterContentInit` and `ngAfterViewInit` is that content children are set before view children. Content refers to the projected content from the parent component, while view refers to the component's own template.

[Back to Top](#angular-lifecycle-hooks)

## ngAfterViewChecked - After View Check

The `ngAfterViewChecked()` lifecycle hook is called after Angular checks a component's views and child views. It's called after `ngAfterViewInit()` and then after every subsequent `ngDoCheck()`.

### Key Features:

* Runs after Angular checks the component's views and child views
* Called after every change detection cycle
* Useful for responding to view changes
* Called very frequently, so keep operations lightweight

```typescript
import { Component, AfterViewInit, AfterViewChecked, ViewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-resizable-panel',
  template: `
    <div class="panel" #panel>
      <div class="panel-header">
        <h3>{{ title }}</h3>
      </div>
      <div class="panel-content">
        <ng-content></ng-content>
      </div>
      <div class="panel-info">
        Panel size: {{ panelWidth }}px × {{ panelHeight }}px
      </div>
    </div>
  `,
  styles: [`
    .panel {
      border: 1px solid #ccc;
      border-radius: 4px;
      margin: 16px;
      padding: 16px;
      transition: all 0.3s ease;
    }
    .panel-header {
      border-bottom: 1px solid #eee;
      margin-bottom: 16px;
    }
    .panel-info {
      color: #888;
      font-size: 12px;
      margin-top: 16px;
      text-align: right;
    }
  `]
})
export class ResizablePanelComponent implements AfterViewInit, AfterViewChecked {
  @Input() title: string = 'Panel';
  @ViewChild('panel') panelElement!: ElementRef;
  
  panelWidth: number = 0;
  panelHeight: number = 0;
  private previousWidth: number = 0;
  private previousHeight: number = 0;
  
  ngAfterViewInit() {
    console.log('ngAfterViewInit - View initialized');
    this.updateDimensions();
  }
  
  ngAfterViewChecked() {
    // Check if dimensions have changed
    this.updateDimensions();
  }
  
  private updateDimensions() {
    if (this.panelElement) {
      const element = this.panelElement.nativeElement;
      const width = element.offsetWidth;
      const height = element.offsetHeight;
      
      // Only update and log if dimensions have changed
      if (width !== this.previousWidth || height !== this.previousHeight) {
        this.panelWidth = width;
        this.panelHeight = height;
        this.previousWidth = width;
        this.previousHeight = height;
        
        console.log(`Panel dimensions updated: ${width}px × ${height}px`);
      }
    }
  }
}
```

### When to use ngAfterViewChecked:

* Monitoring the size or position of elements
* Synchronizing your component with an external UI library
* Updating calculations that depend on the DOM layout

> **Performance Warning:** Since `ngAfterViewChecked()` is called frequently during each change detection cycle, keep the operations inside it as lightweight as possible. Avoid expensive computations or DOM manipulations that could impact performance.

[Back to Top](#angular-lifecycle-hooks)

## Lifecycle Hooks Best Practices

Following these best practices will help you use lifecycle hooks effectively in your Angular applications.

### General Best Practices:

* **Implement Interfaces**: Always implement the corresponding interface for each lifecycle hook
  ```typescript
  // Good practice
  export class MyComponent implements OnInit, OnDestroy {
    ngOnInit() { /* ... */ }
    ngOnDestroy() { /* ... */ }
  }
  ```

* **Keep Hooks Lightweight**: Minimize complex operations in frequently called hooks like `ngDoCheck` and `ngAfterViewChecked`

* **Avoid Side Effects**: Don't trigger additional change detection cycles within hooks

* **Use Appropriate Hooks**: Choose the right hook for each task:
  - Use `ngOnInit` for initialization
  - Use `ngOnChanges` for responding to input changes
  - Use `ngOnDestroy` for cleanup

* **Always Clean Up**: Unsubscribe from observables and remove event listeners in `ngOnDestroy`

### Hook-Specific Best Practices:

#### ngOnInit
* Use for initialization logic instead of the constructor
* Ideal for service calls and setting up subscriptions
* Keep the logic focused on initialization

```typescript
// Good
ngOnInit() {
  this.loadInitialData();
  this.setupSubscriptions();
}

// Bad - Don't do heavy processing in constructor
constructor(private service: DataService) {
  this.service.getData().subscribe(/* ... */); // Should be in ngOnInit
}
```

#### ngOnChanges
* Use simple comparison logic
* Avoid deep object comparisons which can be expensive
* Consider using `OnPush` change detection for better performance

```typescript
// Good
ngOnChanges(changes: SimpleChanges) {
  if (changes['userId'] && !changes['userId'].firstChange) {
    this.loadUserData();
  }
}

// Bad - Expensive deep comparison
ngOnChanges() {
  if (JSON.stringify(this.previousData) !== JSON.stringify(this.newData)) {
    // Do something
  }
}
```

#### ngAfterViewInit
* Avoid changing properties that affect the view
* Use `setTimeout` if you need to update bound properties

```typescript
// Avoiding ExpressionChangedAfterItHasBeenCheckedError
ngAfterViewInit() {
  // This might cause ExpressionChangedAfterItHasBeenCheckedError
  // this.showPanel = true;
  
  // Instead, use setTimeout
  setTimeout(() => {
    this.showPanel = true;
  });
}
```

#### ngOnDestroy
* Always unsubscribe from Observables
* Clear timers and intervals
* Remove manually added event listeners

```typescript
// Good cleanup pattern
private subscription = new Subscription();

ngOnInit() {
  this.subscription.add(
    this.dataService.getData().subscribe(data => {
      this.data = data;
    })
  );
  
  this.intervalId = setInterval(() => this.checkStatus(), 5000);
}

ngOnDestroy() {
  this.subscription.unsubscribe();
  clearInterval(this.intervalId);
}
```

### Performance Considerations:

* **Be careful with `ngDoCheck`**: It runs on every change detection cycle and can hurt performance
* **Avoid DOM manipulation** in frequently called hooks
* **Use `OnPush` change detection** strategy for components that don't need to detect all changes
* **Debounce operations** that might be triggered frequently

```typescript
// Using OnPush change detection
@Component({
  selector: 'app-performance',
  template: `...`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class PerformanceComponent implements OnInit {
  // Component implementation
}
```

[Back to Top](#angular-lifecycle-hooks)

## Lifecycle Execution Sequence

Understanding the order in which lifecycle hooks are called is crucial for properly organizing your component logic.

### Complete Lifecycle Sequence:

1. **constructor** - Called when component is created (not technically a hook)
2. **ngOnChanges** - Called before ngOnInit and when input properties change
3. **ngOnInit** - Called once after first ngOnChanges
4. **ngDoCheck** - Called during every change detection run
5. **ngAfterContentInit** - Called after content is projected
6. **ngAfterContentChecked** - Called after content is checked
7. **ngAfterViewInit** - Called after component & child views are initialized
8. **ngAfterViewChecked** - Called after component & child views are checked
9. **ngOnDestroy** - Called just before the component is destroyed

### Visual Execution Flow:

```
Component Creation
     |
Constructor
     |
ngOnChanges (first time)
     |
ngOnInit (once)
     |
ngDoCheck
     |     \
     |      \
     |    ngAfterContentInit (once)
     |           |
     |    ngAfterContentChecked
     |           |
     |    ngAfterViewInit (once)
     |           |
     |    ngAfterViewChecked
     |      /
     |     /
     |
Input Properties Change
     |
ngOnChanges
     |
     ↓
Change Detection Cycle
     |
ngDoCheck
     |
ngAfterContentChecked
     |
ngAfterViewChecked
     |
     ↓
Component Destruction
     |
ngOnDestroy
```

### Lifecycle Hooks for Nested Components:

When working with parent and child components, the lifecycle hooks execute in a specific order:

1. Parent constructor
2. Parent ngOnChanges
3. Parent ngOnInit
4. Parent ngDoCheck
5. Parent ngAfterContentInit
6. Parent ngAfterContentChecked
7. **Child constructor**
8. **Child ngOnChanges**
9. **Child ngOnInit**
10. **Child ngDoCheck**
11. **Child ngAfterContentInit**
12. **Child ngAfterContentChecked**
13. **Child ngAfterViewInit**
14. **Child ngAfterViewChecked**
15. Parent ngAfterViewInit
16. Parent ngAfterViewChecked

This sequence is important to understand as it explains why parent components can't directly access their child views in `ngAfterContentInit` but must wait until `ngAfterViewInit`.

### Example: Lifecycle Sequence Demonstration

```typescript
import {
  Component,
  OnInit,
  OnChanges,
  DoCheck,
  AfterContentInit,
  AfterContentChecked,
  AfterViewInit,
  AfterViewChecked,
  OnDestroy,
  Input,
  SimpleChanges
} from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  template: `
    <div class="lifecycle-demo">
      <h3>Lifecycle Demo Component</h3>
      <p>Value: {{ value }}</p>
      <div class="lifecycle-log">
        <h4>Lifecycle Events (latest first):</h4>
        <ul>
          <li *ngFor="let event of lifecycleLog">{{ event }}</li>
        </ul>
      </div>
    </div>
  `,
  styles: [`
    .lifecycle-demo {
      border: 1px solid #ccc;
      padding: 15px;
      margin: 15px;
    }
    .lifecycle-log {
      max-height: 200px;
      overflow-y: auto;
      border: 1px solid #eee;
      padding: 10px;
    }
  `]
})
export class LifecycleDemoComponent implements 
    OnInit, 
    OnChanges, 
    DoCheck, 
    AfterContentInit, 
    AfterContentChecked, 
    AfterViewInit, 
    AfterViewChecked, 
    OnDestroy {
  
  @Input() value: string = '';
  
  lifecycleLog: string[] = [];
  
  constructor() {
    this.logLifecycleEvent('constructor');
  }
  
  ngOnChanges(changes: SimpleChanges): void {
    this.logLifecycleEvent(`ngOnChanges - value: ${this.value}`);
    
    if (changes['value']) {
      this.logLifecycleEvent(`  - value changed from "${changes['value'].previousValue}" to "${changes['value'].currentValue}"`);
    }
  }
  
  ngOnInit(): void {
    this.logLifecycleEvent('ngOnInit');
  }
  
  ngDoCheck(): void {
    this.logLifecycleEvent('ngDoCheck');
  }
  
  ngAfterContentInit(): void {
    this.logLifecycleEvent('ngAfterContentInit');
  }
  
  ngAfterContentChecked(): void {
    this.logLifecycleEvent('ngAfterContentChecked');
  }
  
  ngAfterViewInit(): void {
    this.logLifecycleEvent('ngAfterViewInit');
  }
  
  ngAfterViewChecked(): void {
    this.logLifecycleEvent('ngAfterViewChecked');
  }
  
  ngOnDestroy(): void {
    this.logLifecycleEvent('ngOnDestroy');
    console.log('Component destroyed with final lifecycle log:', this.lifecycleLog);
  }
  
  private logLifecycleEvent(event: string): void {
    console.log(event);
    this.lifecycleLog.unshift(`${new Date().toLocaleTimeString()}: ${event}`);
    
    // Keep the log at a reasonable size
    if (this.lifecycleLog.length > 20) {
      this.lifecycleLog.pop();
    }
  }
}
```

### Parent Component Using the Lifecycle Demo:

```typescript
@Component({
  selector: 'app-lifecycle-parent',
  template: `
    <h2>Lifecycle Hooks Demonstration</h2>
    <button (click)="updateValue()">Update Value</button>
    <button (click)="toggleChild()">Toggle Child Component</button>
    
    <app-lifecycle-demo *ngIf="showChild" [value]="currentValue"></app-lifecycle-demo>
  `
})
export class LifecycleParentComponent {
  currentValue: string = 'initial';
  showChild: boolean = true;
  
  updateValue() {
    this.currentValue = `updated-${new Date().toLocaleTimeString()}`;
  }
  
  toggleChild() {
    this.showChild = !this.showChild;
  }
}

// Sample console output:
// constructor
// ngOnChanges - value: initial
//   - value changed from "undefined" to "initial"
// ngOnInit
// ngDoCheck
// ngAfterContentInit
// ngAfterContentChecked
// ngAfterViewInit
// ngAfterViewChecked
```

[Back to Top](#angular-lifecycle-hooks)

### Summary of Lifecycle Hooks and Their Use Cases:

| Lifecycle Hook | Timing | Use Cases |
|----------------|--------|-----------|
| **ngOnChanges** | Before ngOnInit and when input properties change | Respond to input property changes, initial setup based on inputs |
| **ngOnInit** | Once, after first ngOnChanges | Initialization logic, service calls, subscriptions setup |
| **ngDoCheck** | Every change detection cycle | Custom change detection, detect changes Angular doesn't track |
| **ngAfterContentInit** | Once after content projection | Initialize based on projected content |
| **ngAfterContentChecked** | After every content check | React to changes in projected content |
| **ngAfterViewInit** | Once after views are initialized | DOM manipulation, third-party library integration |
| **ngAfterViewChecked** | After every view check | Monitor view changes, sync with external UI libraries |
| **ngOnDestroy** | Just before component destruction | Cleanup resources, unsubscribe from observables |

Understanding the Angular component lifecycle and using the appropriate hooks at the right time will help you build more efficient and maintainable applications.

[Back to Top](#angular-lifecycle-hooks)