# Angular Content Children and View Children

## Table of Contents

| # | Topic |
|---|-------|
| **Core Concepts** | |
| 1 | [Introduction to Component Composition](#introduction-to-component-composition) |
| 2 | [Understanding DOM Relationships in Angular](#understanding-dom-relationships-in-angular) |
| **View Children** | |
| 3 | [ViewChild Decorator](#viewchild-decorator) |
| 4 | [ViewChildren Decorator](#viewchildren-decorator) |
| 5 | [Static vs Dynamic Queries](#static-vs-dynamic-queries) |
| **Content Children** | |
| 6 | [ContentChild Decorator](#contentchild-decorator) |
| 7 | [ContentChildren Decorator](#contentchildren-decorator) |
| 8 | [Content Projection](#content-projection) |
| **Advanced Topics** | |
| 9 | [Using AfterViewInit and AfterContentInit](#using-afterviewinit-and-aftercontentinit) |
| 10 | [Query Options and Configuration](#query-options-and-configuration) |
| 11 | [ViewChild vs ContentChild Comparison](#viewchild-vs-contentchild-comparison) |
| 12 | [Best Practices](#best-practices) |

## Introduction to Component Composition

Component composition is a fundamental concept in Angular that allows developers to build complex UIs by combining smaller, more manageable components. This compositional approach supports reusability, maintainability, and clarity in your application architecture.

Key features:
- **Hierarchical Structure**: Angular applications are composed of component trees
- **Parent-Child Relationships**: Components can contain other components
- **Communication Patterns**: Angular provides several mechanisms for components to interact with each other

In Angular, there are two main ways components can relate to each other in the DOM:
1. **View Children**: Elements that are part of a component's template
2. **Content Children**: Elements that are projected into a component from outside

```typescript
// Example of basic component composition
@Component({
  selector: 'app-parent',
  template: `
    <div>
      <h2>Parent Component</h2>
      <app-child></app-child>  <!-- View Child -->
      <div>
        <ng-content></ng-content>  <!-- Content Child (projected) -->
      </div>
    </div>
  `
})
export class ParentComponent { }
```

[Back to Top](#table-of-contents)

## Understanding DOM Relationships in Angular

Angular's component architecture creates two distinct types of DOM relationships: view DOM and content DOM. Understanding these relationships is crucial for effective component design and communication.

Key concepts:
- **View DOM**: Elements defined directly in a component's template
- **Content DOM**: Elements projected into a component using `<ng-content>`
- **Shadow DOM**: Angular's encapsulation mechanism that isolates component styles and DOM

Angular provides specialized decorators to access elements from both the view DOM and content DOM:
- `@ViewChild` and `@ViewChildren` for accessing elements in the view DOM
- `@ContentChild` and `@ContentChildren` for accessing projected content

```typescript
@Component({
  selector: 'app-dom-example',
  template: `
    <div class="view-dom">
      <p>This is part of the view DOM</p>
      <app-child #viewChild></app-child>
    </div>
    <div class="content-dom">
      <ng-content></ng-content>
    </div>
  `
})
export class DomExampleComponent implements AfterViewInit, AfterContentInit {
  // Access view DOM element
  @ViewChild('viewChild') viewChild: ChildComponent;
  
  // Access content DOM element 
  @ContentChild('contentChild') contentChild: ElementRef;
  
  ngAfterViewInit() {
    // View children are available after view initialization
    console.log('View child:', this.viewChild);
  }
  
  ngAfterContentInit() {
    // Content children are available after content initialization
    console.log('Content child:', this.contentChild);
  }
}
```

[Back to Top](#table-of-contents)

## ViewChild Decorator

The `@ViewChild` decorator allows a component to query and access a single element from its view DOM. This element can be a directive, a component instance, or a DOM element.

Key features:
- Provides access to a **single** element in the component's view
- Can query by type, template reference variable, or directive class
- Returns the first matching element when multiple elements match the selector

```typescript
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-view-child-demo',
  template: `
    <h2>ViewChild Example</h2>
    
    <!-- DOM element reference -->
    <input #nameInput type="text" placeholder="Enter name">
    <button (click)="focusInput()">Focus Input</button>
    
    <!-- Component reference -->
    <app-counter #counter></app-counter>
    <button (click)="incrementCounter()">Increment Counter</button>
    
    <!-- Directive reference -->
    <div customDirective #directive="customDir">Directive Element</div>
  `
})
export class ViewChildDemoComponent implements AfterViewInit {
  // Query by template reference variable
  @ViewChild('nameInput') nameInputRef: ElementRef;
  
  // Query by component type
  @ViewChild('counter') counterComponent: CounterComponent;
  
  // Query by directive
  @ViewChild('directive') customDirective: CustomDirective;
  
  // Query by component type directly
  @ViewChild(CounterComponent) counterComponentByType: CounterComponent;
  
  ngAfterViewInit() {
    // Access is available in ngAfterViewInit
    console.log('Input element:', this.nameInputRef.nativeElement);
    console.log('Counter component:', this.counterComponent);
    console.log('Custom directive:', this.customDirective);
  }
  
  focusInput() {
    // Direct DOM manipulation using ElementRef
    this.nameInputRef.nativeElement.focus();
  }
  
  incrementCounter() {
    // Call method on component instance
    this.counterComponent.increment();
  }
}

// Example CounterComponent
@Component({
  selector: 'app-counter',
  template: `<div>Count: {{ count }}</div>`
})
export class CounterComponent {
  count = 0;
  
  increment() {
    this.count++;
    console.log('Counter incremented to', this.count);
  }
}

// Sample output in console:
// Input element: <input type="text" placeholder="Enter name">
// Counter component: CounterComponent {count: 0}
// Custom directive: CustomDirective {elementRef: ElementRef}
// (After clicking "Increment Counter")
// Counter incremented to 1
```

[Back to Top](#table-of-contents)

## ViewChildren Decorator

The `@ViewChildren` decorator allows a component to query and access multiple elements from its view DOM. It returns a `QueryList` collection of all elements that match the selector.

Key features:
- Provides access to **multiple** elements in the component's view
- Returns a `QueryList` object, which is a live collection that updates automatically
- `QueryList` provides an `changes` Observable to react to changes in the collection
- Can query for components, directives, or template reference variables

```typescript
import { Component, ViewChildren, QueryList, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-view-children-demo',
  template: `
    <h2>ViewChildren Example</h2>
    
    <!-- Multiple elements with the same directive -->
    <div *ngFor="let item of items">
      <app-item [name]="item" #itemComponent></app-item>
    </div>
    
    <button (click)="highlightAllItems()">Highlight All</button>
    <button (click)="addNewItem()">Add Item</button>
  `
})
export class ViewChildrenDemoComponent implements AfterViewInit {
  items = ['Item 1', 'Item 2', 'Item 3'];
  
  // Query all ItemComponent instances
  @ViewChildren('itemComponent') itemComponents: QueryList<ItemComponent>;
  
  // Alternative: query by component type
  @ViewChildren(ItemComponent) itemComponentsByType: QueryList<ItemComponent>;
  
  ngAfterViewInit() {
    // Access is available in ngAfterViewInit
    console.log('Number of items:', this.itemComponents.length);
    
    // Converting QueryList to array
    const itemArray = this.itemComponents.toArray();
    console.log('Items array:', itemArray);
    
    // Listening for changes in the QueryList
    this.itemComponents.changes.subscribe((items: QueryList<ItemComponent>) => {
      console.log('Items changed, new count:', items.length);
    });
  }
  
  highlightAllItems() {
    // Iterate through all item components
    this.itemComponents.forEach(item => {
      item.highlight();
    });
  }
  
  addNewItem() {
    // Adding a new item will trigger the QueryList changes observable
    this.items.push(`Item ${this.items.length + 1}`);
  }
}

// Example ItemComponent
@Component({
  selector: 'app-item',
  template: `
    <div [class.highlighted]="isHighlighted">{{ name }}</div>
  `,
  styles: [`.highlighted { background-color: yellow; }`]
})
export class ItemComponent {
  @Input() name: string;
  isHighlighted = false;
  
  highlight() {
    this.isHighlighted = true;
    setTimeout(() => {
      this.isHighlighted = false;
    }, 1500);
  }
}

// Sample output in console:
// Number of items: 3
// Items array: [ItemComponent, ItemComponent, ItemComponent]
// (After clicking "Add Item")
// Items changed, new count: 4
```

The `QueryList` provides several useful methods:
- `toArray()`: Converts the query list to a regular array
- `forEach()`: Iterates over each item in the collection
- `map()`, `filter()`, `find()`, etc.: Array-like operations
- `changes`: Observable that emits when the query results change

[Back to Top](#table-of-contents)

## Static vs Dynamic Queries

Angular provides two options for querying elements: static and dynamic. The timing of when these queries resolve depends on the `static` flag you provide.

Key concepts:
- **Static queries**: Resolved before change detection runs (available earlier)
- **Dynamic queries**: Resolved after change detection runs (available later, but reflect latest state)

```typescript
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-query-timing',
  template: `
    <h2>Query Timing Example</h2>
    
    <!-- Static element (always present) -->
    <div #staticElement>Static Element</div>
    
    <!-- Dynamic element (conditionally present) -->
    <div *ngIf="showDynamicElement" #dynamicElement>Dynamic Element</div>
    
    <button (click)="toggleDynamicElement()">Toggle Dynamic Element</button>
  `
})
export class QueryTimingComponent implements AfterViewInit, OnInit {
  showDynamicElement = false;
  
  // Static query (default in Angular 8+)
  @ViewChild('staticElement', { static: true }) staticElement: ElementRef;
  
  // Dynamic query
  @ViewChild('dynamicElement', { static: false }) dynamicElement: ElementRef;
  
  ngOnInit() {
    // Static elements are available in ngOnInit
    console.log('ngOnInit - Static element:', this.staticElement?.nativeElement.textContent);
    
    // Dynamic elements are NOT available yet
    console.log('ngOnInit - Dynamic element:', this.dynamicElement); // undefined
  }
  
  ngAfterViewInit() {
    // Both static and dynamic elements are available in ngAfterViewInit
    // (if the dynamic element is in the DOM)
    console.log('ngAfterViewInit - Static element:', this.staticElement?.nativeElement.textContent);
    console.log('ngAfterViewInit - Dynamic element:', this.dynamicElement?.nativeElement?.textContent);
  }
  
  toggleDynamicElement() {
    this.showDynamicElement = !this.showDynamicElement;
    
    // After toggle, we need to wait for next change detection cycle
    // to access the dynamic element
    setTimeout(() => {
      console.log('After toggle - Dynamic element:', 
                  this.dynamicElement?.nativeElement?.textContent || 'Not in DOM');
    }, 0);
  }
}

// Sample output in console:
// ngOnInit - Static element: Static Element
// ngOnInit - Dynamic element: undefined
// ngAfterViewInit - Static element: Static Element
// ngAfterViewInit - Dynamic element: undefined
// (After clicking "Toggle Dynamic Element")
// After toggle - Dynamic element: Dynamic Element
// (After clicking "Toggle Dynamic Element" again)
// After toggle - Dynamic element: Not in DOM
```

Best practices for static vs dynamic queries:
- Use `{ static: true }` for elements that are **always** present in the template
- Use `{ static: false }` for elements inside structural directives like `*ngIf`, `*ngFor`
- Default is `{ static: false }` in Angular 9+ if not specified

[Back to Top](#table-of-contents)

## ContentChild Decorator

The `@ContentChild` decorator allows a component to query and access a single element from its projected content (content DOM). This is used to interact with elements that are passed into the component through content projection.

Key features:
- Provides access to a **single** element from the projected content
- Similar to `@ViewChild` but works with content projection
- Can query by template reference variable, directive, or component type
- Returns the first matching element when multiple elements match the selector

```typescript
import { Component, ContentChild, AfterContentInit, ElementRef } from '@angular/core';

@Component({
  selector: 'app-panel',
  template: `
    <div class="panel">
      <div class="panel-header">
        <h3>{{ title }}</h3>
      </div>
      <div class="panel-body">
        <ng-content></ng-content>
      </div>
      <button (click)="focusContentInput()">Focus Content Input</button>
    </div>
  `,
  styles: [`
    .panel {
      border: 1px solid #ccc;
      border-radius: 4px;
      padding: 15px;
      margin-bottom: 20px;
    }
    .panel-header {
      border-bottom: 1px solid #eee;
      margin-bottom: 15px;
    }
  `]
})
export class PanelComponent implements AfterContentInit {
  @Input() title: string = 'Panel';
  
  // Query a specific input element from the projected content
  @ContentChild('contentInput') contentInput: ElementRef;
  
  // Query a component from the projected content
  @ContentChild(SpecialButtonComponent) specialButton: SpecialButtonComponent;
  
  ngAfterContentInit() {
    // Content children are available in ngAfterContentInit
    console.log('Content input element:', this.contentInput?.nativeElement);
    console.log('Special button component:', this.specialButton);
  }
  
  focusContentInput() {
    // Interact with the projected element
    if (this.contentInput) {
      this.contentInput.nativeElement.focus();
    } else {
      console.warn('No content input available to focus');
    }
  }
}

// Special button component that might be projected
@Component({
  selector: 'app-special-button',
  template: `<button class="special-btn">{{ label }}</button>`,
  styles: [`.special-btn { background-color: #4CAF50; color: white; }`]
})
export class SpecialButtonComponent {
  @Input() label: string = 'Special Button';
}

// Usage in a parent component
@Component({
  selector: 'app-content-child-demo',
  template: `
    <h2>ContentChild Example</h2>
    
    <app-panel title="User Form">
      <form>
        <div class="form-group">
          <label>Name:</label>
          <input #contentInput type="text" placeholder="Enter name">
        </div>
        <app-special-button label="Submit"></app-special-button>
      </form>
    </app-panel>
  `
})
export class ContentChildDemoComponent { }

// Sample output in console:
// Content input element: <input type="text" placeholder="Enter name">
// Special button component: SpecialButtonComponent {label: "Submit"}
```

[Back to Top](#table-of-contents)

## ContentChildren Decorator

The `@ContentChildren` decorator allows a component to query and access multiple elements from its projected content. Like `@ViewChildren`, it returns a `QueryList` collection that updates automatically.

Key features:
- Provides access to **multiple** elements from the projected content
- Returns a live `QueryList` that updates when the content changes
- Can query for components, directives, or elements with specific template variables
- Useful for creating wrapper components that need to interact with their children

```typescript
import { Component, ContentChildren, QueryList, AfterContentInit } from '@angular/core';

@Component({
  selector: 'app-tabs',
  template: `
    <div class="tabs-container">
      <div class="tabs-header">
        <ul>
          <li *ngFor="let tab of tabs; let i = index" 
              [class.active]="selectedIndex === i"
              (click)="selectTab(i)">
            {{ tab.title }}
          </li>
        </ul>
      </div>
      <div class="tabs-body">
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .tabs-container { border: 1px solid #ddd; border-radius: 4px; }
    .tabs-header ul { display: flex; list-style: none; padding: 0; margin: 0; }
    .tabs-header li { 
      padding: 10px 15px; 
      cursor: pointer; 
      border-bottom: 2px solid transparent; 
    }
    .tabs-header li.active { 
      border-bottom: 2px solid #007bff; 
      font-weight: bold;
    }
    .tabs-body { padding: 15px; }
  `]
})
export class TabsComponent implements AfterContentInit {
  // Query all TabComponent instances projected into this component
  @ContentChildren(TabComponent) tabs: QueryList<TabComponent>;
  
  selectedIndex = 0;
  
  ngAfterContentInit() {
    // Access is available in ngAfterContentInit
    console.log('Number of tabs:', this.tabs.length);
    
    // Initially hide all tabs except the selected one
    this.updateTabVisibility();
    
    // Listen for changes to tabs (additions or removals)
    this.tabs.changes.subscribe(() => {
      console.log('Tabs changed, new count:', this.tabs.length);
      this.updateTabVisibility();
    });
  }
  
  selectTab(index: number) {
    this.selectedIndex = index;
    this.updateTabVisibility();
  }
  
  private updateTabVisibility() {
    if (this.tabs) {
      this.tabs.forEach((tab, index) => {
        tab.visible = index === this.selectedIndex;
      });
    }
  }
}

// Tab component that will be projected
@Component({
  selector: 'app-tab',
  template: `
    <div [style.display]="visible ? 'block' : 'none'">
      <ng-content></ng-content>
    </div>
  `
})
export class TabComponent {
  @Input() title: string = 'Tab';
  visible: boolean = false;
}

// Usage in a parent component
@Component({
  selector: 'app-content-children-demo',
  template: `
    <h2>ContentChildren Example</h2>
    
    <app-tabs>
      <app-tab title="Profile">
        <h3>User Profile</h3>
        <p>This is the user profile content.</p>
      </app-tab>
      <app-tab title="Settings">
        <h3>User Settings</h3>
        <p>Configure your application settings here.</p>
      </app-tab>
      <app-tab title="Messages">
        <h3>User Messages</h3>
        <p>You have no new messages.</p>
      </app-tab>
    </app-tabs>
    
    <button (click)="addNewTab()">Add New Tab</button>
  `
})
export class ContentChildrenDemoComponent {
  @ViewChild(TabsComponent) tabsComponent: TabsComponent;
  tabCount = 3;
  
  addNewTab() {
    // This new tab will be detected by the ContentChildren query
    const newTab = document.createElement('app-tab');
    newTab.setAttribute('title', `Dynamic Tab ${++this.tabCount}`);
    newTab.innerHTML = `<p>This tab was added dynamically!</p>`;
    
    const tabsElement = document.querySelector('app-tabs');
    tabsElement.appendChild(newTab);
  }
}

// Sample output in console:
// Number of tabs: 3
// (After clicking "Add New Tab")
// Tabs changed, new count: 4
```

[Back to Top](#table-of-contents)

## Content Projection

Content projection is a powerful Angular feature that allows components to receive and display content from their parent components. It enables creating reusable component templates with customizable content.

Key features:
- Enables component reusability with customizable content
- Uses `<ng-content>` as a placeholder for projected content
- Supports targeted projection with select attribute
- Can be used with `@ContentChild` and `@ContentChildren` for interaction

Types of content projection:
1. **Single-slot projection**: Projects all content into a single location
2. **Multi-slot projection**: Projects different content into specific locations using the `select` attribute
3. **Conditional projection**: Selectively projects content based on conditions

```typescript
// Single-slot projection example
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-body">
        <!-- All content is projected here -->
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`.card { border: 1px solid #ddd; border-radius: 4px; padding: 15px; }`]
})
export class CardComponent { }

// Multi-slot projection example
@Component({
  selector: 'app-advanced-card',
  template: `
    <div class="card">
      <div class="card-header">
        <!-- Content with 'header' attribute goes here -->
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="card-body">
        <!-- Content with 'body' attribute goes here -->
        <ng-content select="[body]"></ng-content>
      </div>
      <div class="card-footer">
        <!-- Content with 'footer' attribute goes here -->
        <ng-content select="[footer]"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .card { border: 1px solid #ddd; border-radius: 4px; }
    .card-header { padding: 10px 15px; background-color: #f8f9fa; border-bottom: 1px solid #ddd; }
    .card-body { padding: 15px; }
    .card-footer { padding: 10px 15px; background-color: #f8f9fa; border-top: 1px solid #ddd; }
  `]
})
export class AdvancedCardComponent { }

// Usage of content projection components
@Component({
  selector: 'app-content-projection-demo',
  template: `
    <h2>Content Projection Examples</h2>
    
    <h3>Single-slot Projection</h3>
    <app-card>
      <h4>Simple Card</h4>
      <p>This is the content of the card.</p>
      <button>Click Me</button>
    </app-card>
    
    <h3>Multi-slot Projection</h3>
    <app-advanced-card>
      <div header>
        <h4>Advanced Card Title</h4>
      </div>
      <div body>
        <p>This content goes into the card body.</p>
        <p>Multiple elements can be projected here.</p>
      </div>
      <div footer>
        <button>Save</button>
        <button>Cancel</button>
      </div>
    </app-advanced-card>
    
    <h3>Selecting by CSS Classes</h3>
    <app-dynamic-content>
      <p class="intro">This is the introduction text.</p>
      <p>This is regular content.</p>
      <p class="conclusion">This is the conclusion.</p>
    </app-dynamic-content>
  `
})
export class ContentProjectionDemoComponent { }

// Selecting content by CSS classes
@Component({
  selector: 'app-dynamic-content',
  template: `
    <div class="dynamic-content">
      <!-- Select by CSS class -->
      <div class="intro-section">
        <ng-content select=".intro"></ng-content>
      </div>
      
      <!-- Default content (no select) -->
      <div class="main-section">
        <ng-content></ng-content>
      </div>
      
      <!-- Select by CSS class -->
      <div class="conclusion-section">
        <ng-content select=".conclusion"></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .intro-section { background-color: #e6f7ff; padding: 10px; margin-bottom: 10px; }
    .main-section { padding: 10px; }
    .conclusion-section { background-color: #f6ffed; padding: 10px; margin-top: 10px; }
  `]
})
export class DynamicContentComponent { }
```

Selection options for `<ng-content>`:
- By element: `<ng-content select="h1"></ng-content>`
- By CSS class: `<ng-content select=".my-class"></ng-content>`
- By attribute: `<ng-content select="[my-attr]"></ng-content>`
- By component: `<ng-content select="app-specific-component"></ng-content>`
- Default (no selector): `<ng-content></ng-content>` (captures anything not selected)

[Back to Top](#table-of-contents)

## Using AfterViewInit and AfterContentInit

Angular's lifecycle hooks provide specific moments to interact with view children and content children. The `AfterViewInit` and `AfterContentInit` hooks are essential for properly accessing these queried elements.

Key concepts:
- **AfterContentInit**: Called after content projection (content children) is initialized
- **AfterViewInit**: Called after the component's view (view children) is initialized
- **Lifecycle order**: Content is initialized before view

```typescript
import { Component, ViewChild, ContentChild, ElementRef, 
         AfterViewInit, AfterContentInit, OnInit } from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  template: `
    <div class="container">
      <h2>Lifecycle Hooks Example</h2>
      <div class="timeline">
        <div *ngFor="let event of lifecycleEvents" class="event">
          <span class="time">{{ event.time }}ms</span>
          <span class="name">{{ event.name }}</span>
        </div>
      </div>
      
      <!-- View child -->
      <div #viewElement>View Child Element</div>
      
      <!-- Content projection -->
      <ng-content></ng-content>
    </div>
  `,
  styles: [`
    .timeline { margin: 20px 0; }
    .event { margin-bottom: 5px; padding: 8px; background-color: #f5f5f5; }
    .time { font-weight: bold; margin-right: 10px; }
  `]
})
export class LifecycleDemoComponent implements OnInit, AfterContentInit, AfterViewInit {
  @ViewChild('viewElement') viewElement: ElementRef;
  @ContentChild('contentElement') contentElement: ElementRef;
  
  lifecycleEvents: {name: string, time: number}[] = [];
  private startTime: number;
  
  constructor() {
    this.startTime = performance.now();
    this.logEvent('constructor');
  }
  
  ngOnInit() {
    this.logEvent('ngOnInit');
    
    // Check element availability
    console.log('ngOnInit - View element:', this.viewElement); // undefined
    console.log('ngOnInit - Content element:', this.contentElement); // undefined
  }
  
  ngAfterContentInit() {
    this.logEvent('ngAfterContentInit');
    
    // Content children are available now
    console.log('ngAfterContentInit - View element:', this.viewElement); // undefined
    console.log('ngAfterContentInit - Content element:', this.contentElement?.nativeElement.textContent);
  }
  
  ngAfterViewInit() {
    this.logEvent('ngAfterViewInit');
    
    // Both view and content children are available now
    console.log('ngAfterViewInit - View element:', this.viewElement?.nativeElement.textContent);
    console.log('ngAfterViewInit - Content element:', this.contentElement?.nativeElement.textContent);
  }
  
  private logEvent(name: string) {
    const time = Math.round(performance.now() - this.startTime);
    this.lifecycleEvents.push({ name, time });
  }
}

// Usage in parent component
@Component({
  selector: 'app-parent',
  template: `
    <app-lifecycle-demo>
      <div #contentElement>Content Child Element</div>
    </app-lifecycle-demo>
  `
})
export class ParentComponent { }

// Sample output in console:
// ngOnInit - View element: undefined
// ngOnInit - Content element: undefined
// ngAfterContentInit - View element: undefined
// ngAfterContentInit - Content element: Content Child Element
// ngAfterViewInit - View element: View Child Element
// ngAfterViewInit - Content element: Content Child Element
```

Best practices for lifecycle hooks:
- Use `ngAfterContentInit` when you need to interact with content children
- Use `ngAfterViewInit` when you need to interact with view children
- Remember that content is initialized before view
- Don't try to access view children in `ngOnInit` or `ngAfterContentInit`
- Be cautious about updating the DOM in these hooks, as it may trigger additional change detection cycles

[Back to Top](#table-of-contents)

## Query Options and Configuration

Angular provides several options to customize the behavior of `@ViewChild`, `@ViewChildren`, `@ContentChild`, and `@ContentChildren` decorators.

Key configuration options:
- **static**: Determines when the query results will be available
- **read**: Specifies what to read from the matched elements
- **descendants**: For ContentChildren, determines whether to include all descendants or only direct children

```typescript
import { Component, ViewChild, ContentChild, ElementRef, TemplateRef,
         ViewContainerRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-query-options-demo',
  template: `
    <h2>Query Options Example</h2>
    
    <!-- Element with multiple tokens -->
    <div #element customDirective>Multiple Query Tokens Element</div>
    
    <!-- Template reference -->
    <ng-template #templateRef>
      <p>This is template content</p>
    </ng-template>
    
    <!-- Container for template rendering -->
    <div #container></div>
    
    <button (click)="renderTemplate()">Render Template</button>
    
    <!-- Content projection -->
    <ng-content></ng-content>
  `
})
export class QueryOptionsDemoComponent implements AfterViewInit {
  // Basic query with default options
  @ViewChild('element') basicElement: ElementRef;
  
  // Query with read option for ElementRef
  @ViewChild('element', { read: ElementRef }) elementRef: ElementRef;
  
  // Query with read option for different token (directive)
  @ViewChild('element', { read: CustomDirective }) customDirective: CustomDirective;
  
  // Query for a template
  @ViewChild('templateRef') templateRef: TemplateRef<any>;
  
  // Query for a container to render content
  @ViewChild('container', { read: ViewContainerRef }) container: ViewContainerRef;
  
  // Content query with descendants option
  @ContentChildren(ItemComponent, { descendants: true }) 
  allItemComponents: QueryList<ItemComponent>;
  
  // Content query with descendants false (direct children only)
  @ContentChildren(ItemComponent, { descendants: false }) 
  directItemComponents: QueryList<ItemComponent>;
  
  ngAfterViewInit() {
    console.log('Basic element query:', this.basicElement);
    console.log('Element ref query:', this.elementRef.nativeElement);
    console.log('Custom directive query:', this.customDirective);
    console.log('Template ref:', this.templateRef);
    console.log('All item components:', this.allItemComponents?.length);
    console.log('Direct item components:', this.directItemComponents?.length);
  }
  
  renderTemplate() {
    // Clear the container
    this.container.clear();
    
    // Create an embedded view from the template
    const view = this.templateRef.createEmbeddedView(null);
    
    // Insert the view into the container
    this.container.insert(view);
  }
}

// Custom directive for the read option example
@Directive({
  selector: '[customDirective]',
  exportAs: 'customDir'
})
export class CustomDirective {
  doSomething() {
    console.log('Custom directive method called');
  }
}

// Usage in parent component
@Component({
  selector: 'app-query-options-parent',
  template: `
    <app-query-options-demo>
      <!-- Direct child item -->
      <app-item name="Direct Child"></app-item>
      
      <!-- Nested item (descendant) -->
      <div>
        <app-item name="Nested Child"></app-item>
      </div>
    </app-query-options-demo>
  `
})
export class QueryOptionsParentComponent { }

// Sample output in console:
// Basic element query: {nativeElement: div}
// Element ref query: <div customDirective>Multiple Query Tokens Element</div>
// Custom directive query: CustomDirective {elementRef: ElementRef}
// Template ref: TemplateRef {...}
// All item components: 2
// Direct item components: 1
```

**The `read` option**

The `read` option allows you to read a different token from the queried elements. This is useful when an element has multiple tokens associated with it:

| Token Type | Description | Example |
|------------|-------------|---------|
| `ElementRef` | References the native DOM element | `@ViewChild('elementId', { read: ElementRef })` |
| `ViewContainerRef` | Container for dynamic views | `@ViewChild('container', { read: ViewContainerRef })` |
| `TemplateRef` | Reference to an `<ng-template>` | `@ViewChild('templateRef', { read: TemplateRef })` |
| Component/Directive class | Instance of the component/directive | `@ViewChild('element', { read: MyDirective })` |

**The `descendants` option**

For `@ContentChildren`, the `descendants` option determines whether to include only direct children or all descendants:

```typescript
// Include all nested descendants (default)
@ContentChildren(ItemComponent, { descendants: true }) 
allItems: QueryList<ItemComponent>;

// Include only direct children
@ContentChildren(ItemComponent, { descendants: false }) 
directItems: QueryList<ItemComponent>;
```

[Back to Top](#table-of-contents)

## ViewChild vs ContentChild Comparison

Understanding the differences between ViewChild and ContentChild is crucial for effective component design. Here's a comparison of these two query mechanisms:

| Feature | ViewChild/ViewChildren | ContentChild/ContentChildren |
|---------|------------------------|------------------------------|
| **Source DOM** | Elements from component's template (view DOM) | Elements projected into the component (content DOM) |
| **Timing** | Available in `ngAfterViewInit` | Available in `ngAfterContentInit` |
| **Use Case** | Access elements defined in a component's own template | Access elements projected into a component from outside |
| **Relationship** | Parent-to-child relationship | Container-to-content relationship |
| **Template Location** | Elements defined within the component's own template | Elements placed between the component's opening and closing tags |
| **Lifecycle Order** | Initialized after content children | Initialized before view children |

```typescript
// Component demonstrating the differences
@Component({
  selector: 'app-comparison-demo',
  template: `
    <div class="container">
      <h2>ViewChild vs ContentChild</h2>
      
      <!-- ViewChild - part of this component's view -->
      <div #viewExample class="view-element">
        This element is part of the view DOM
      </div>
      
      <!-- ContentChild - placeholders for projected content -->
      <div class="content-container">
        <h3>Projected Content:</h3>
        <ng-content></ng-content>
      </div>
      
      <div class="info-panel">
        <div *ngIf="viewInfoAvailable">
          <h4>View Child Info:</h4>
          <p>Text: {{ viewChildText }}</p>
        </div>
        <div *ngIf="contentInfoAvailable">
          <h4>Content Child Info:</h4>
          <p>Text: {{ contentChildText }}</p>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .container { border: 1px solid #ddd; padding: 15px; margin-bottom: 20px; }
    .view-element { background-color: #e6f7ff; padding: 10px; margin: 10px 0; }
    .content-container { background-color: #fff7e6; padding: 10px; margin: 10px 0; }
    .info-panel { background-color: #f9f9f9; padding: 15px; margin-top: 20px; }
  `]
})
export class ComparisonDemoComponent implements AfterContentInit, AfterViewInit {
  // Query an element from the view DOM
  @ViewChild('viewExample') viewExample: ElementRef;
  
  // Query an element from the content DOM
  @ContentChild('contentExample') contentExample: ElementRef;
  
  viewChildText = '';
  contentChildText = '';
  viewInfoAvailable = false;
  contentInfoAvailable = false;
  
  ngAfterContentInit() {
    console.log('ngAfterContentInit called');
    
    // ContentChild is available here
    if (this.contentExample) {
      this.contentChildText = this.contentExample.nativeElement.textContent.trim();
      this.contentInfoAvailable = true;
    }
    
    // ViewChild is NOT available yet
    console.log('View example in ngAfterContentInit:', this.viewExample); // undefined
  }
  
  ngAfterViewInit() {
    console.log('ngAfterViewInit called');
    
    // Both ContentChild and ViewChild are available here
    if (this.viewExample) {
      this.viewChildText = this.viewExample.nativeElement.textContent.trim();
      this.viewInfoAvailable = true;
    }
    
    console.log('Content example in ngAfterViewInit:', this.contentExample?.nativeElement.textContent);
  }
}

// Usage in parent component
@Component({
  selector: 'app-comparison-parent',
  template: `
    <app-comparison-demo>
      <!-- This is projected into the ContentChild -->
      <div #contentExample class="content-element">
        This element is part of the content DOM
      </div>
    </app-comparison-demo>
  `,
  styles: [`.content-element { background-color: #fffbe6; padding: 10px; }`]
})
export class ComparisonParentComponent { }

// Sample output in console:
// ngAfterContentInit called
// View example in ngAfterContentInit: undefined
// ngAfterViewInit called
// Content example in ngAfterViewInit: This element is part of the content DOM
```

Key differences illustrated:
1. **Source**: ViewChild elements are defined in the component's template, while ContentChild elements are projected from parent components
2. **Timing**: ContentChild elements are available earlier in the component lifecycle
3. **Relationship**: ViewChild represents a parent-to-child relationship, while ContentChild represents a container-to-content relationship

[Back to Top](#table-of-contents)

## Best Practices

Following these best practices will help you use ViewChild, ViewChildren, ContentChild, and ContentChildren effectively and avoid common pitfalls.

### Timing and Lifecycle Hooks

- **Use the correct lifecycle hook**:
  - Access `@ViewChild`/`@ViewChildren` queries in `ngAfterViewInit`
  - Access `@ContentChild`/`@ContentChildren` queries in `ngAfterContentInit`

- **Be careful with change detection**:
  - Modifying view or content in these lifecycle hooks can trigger additional change detection cycles
  - Use `setTimeout(() => { /* your code */ }, 0)` or `ngZone.runOutsideAngular` for complex manipulations

### Query Configuration

- **Use static queries appropriately**:
  - Use `{ static: true }` for elements that are always present (not inside `*ngIf` or `*ngFor`)
  - Use `{ static: false }` (default) for elements that might not be immediately available

- **Use the read option for clarity**:
  ```typescript
  // Be explicit about what you're reading
  @ViewChild('element', { read: ElementRef }) elementRef: ElementRef;
  ```

### Structural Best Practices

- **Prefer component composition over direct DOM manipulation**:
  - Use `@Input` and `@Output` for most parent-child communication
  - Use `@ViewChild`/`@ContentChild` only when necessary

- **Create clear component interfaces**:
  - Define public methods that external components can call through queries
  - Document what behaviors can be controlled through these interfaces

```typescript
// Good practice example
@Component({
  selector: 'app-collapsible-panel',
  template: `
    <div class="panel">
      <div class="panel-header" (click)="toggle()">
        {{ title }}
        <span class="toggle-icon">{{ isExpanded ? '▼' : '►' }}</span>
      </div>
      <div class="panel-body" [class.hidden]="!isExpanded">
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [`
    .panel { border: 1px solid #ddd; margin-bottom: 10px; }
    .panel-header { padding: 10px; background-color: #f5f5f5; cursor: pointer; }
    .panel-body { padding: 15px; }
    .panel-body.hidden { display: none; }
    .toggle-icon { float: right; }
  `]
})
export class CollapsiblePanelComponent {
  @Input() title: string = 'Panel';
  @Input() isExpanded: boolean = false;
  
  // Public methods that can be called via ViewChild
  toggle() {
    this.isExpanded = !this.isExpanded;
  }
  
  expand() {
    this.isExpanded = true;
  }
  
  collapse() {
    this.isExpanded = false;
  }
}

// Usage in parent component
@Component({
  selector: 'app-best-practices-demo',
  template: `
    <h2>Best Practices Example</h2>
    
    <app-collapsible-panel #firstPanel title="Panel One">
      <p>This is the content for panel one.</p>
    </app-collapsible-panel>
    
    <app-collapsible-panel #secondPanel title="Panel Two" [isExpanded]="true">
      <p>This is the content for panel two.</p>
    </app-collapsible-panel>
    
    <div class="controls">
      <button (click)="expandAll()">Expand All</button>
      <button (click)="collapseAll()">Collapse All</button>
    </div>
  `
})
export class BestPracticesDemoComponent implements AfterViewInit {
  // Query a single panel
  @ViewChild('firstPanel') firstPanel: CollapsiblePanelComponent;
  
  // Query all panels
  @ViewChildren(CollapsiblePanelComponent) 
  allPanels: QueryList<CollapsiblePanelComponent>;
  
  ngAfterViewInit() {
    console.log('First panel instance:', this.firstPanel);
    console.log('Number of panels:', this.allPanels.length);
  }
  
  expandAll() {
    this.allPanels.forEach(panel => panel.expand());
  }
  
  collapseAll() {
    this.allPanels.forEach(panel => panel.collapse());
  }
}
```

### Performance Considerations

- **Limit the number of queries**: Too many queries can impact performance
- **Be careful with deep content queries**: Setting `{ descendants: true }` on large component trees can be expensive
- **Consider lazy loading for complex component structures**

### Content Projection Best Practices

- **Use targeted projection with meaningful selectors**:
  ```html
  <ng-content select="[header]"></ng-content>
  <ng-content select="[body]"></ng-content>
  <ng-content select="[footer]"></ng-content>
  ```

- **Provide clear documentation for projection slots**:
  ```typescript
  /**
   * Advanced card component with multiple projection slots
   * 
   * Usage:
   * <app-advanced-card>
   *   <div header>Header content goes here</div>
   *   <div body>Body content goes here</div>
   *   <div footer>Footer content goes here</div>
   * </app-advanced-card>
   */
  @Component({ ... })
  export class AdvancedCardComponent { ... }
  ```

### Common Gotchas to Avoid

- **Don't access view children in ngOnInit**: They aren't available yet
- **Don't modify the view in AfterViewInit without precautions**: This can trigger the "changed after checked" error
- **Be cautious with OnPush change detection**: Child components with OnPush might not update when parent data changes
- **Remember that QueryLists are live**: They update automatically when the DOM changes

By following these best practices, you can create more maintainable, performant Angular applications with clear, well-defined component relationships.

[Back to Top](#table-of-contents)