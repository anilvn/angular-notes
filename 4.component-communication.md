# Angular Component Communication Documentation

This comprehensive guide covers various methods of component communication in Angular applications.

## Table of Contents

| Section | Link |
|---------|------|
| **Core Communication Methods** |
| 1 | [Parent to Child Communication (@Input)](#parent-to-child-communication-input) |
| 2 | [Child to Parent Communication (@Output)](#child-to-parent-communication-output) |
| 3 | [Using Services for Communication](#using-services-for-communication) |
| 4 | [Signals in Angular](#signals-in-angular) |

## Parent to Child Communication (@Input)

The `@Input()` decorator in Angular allows a parent component to pass data to a child component. This is one of the most common forms of component communication in Angular applications.

### Key Features:
- **One-way data flow** from parent to child
- **Reactive updates** when parent data changes
- **Type safety** for input properties
- **Alias support** for property name flexibility

### Basic Implementation

#### Parent Component:
```typescript
// parent.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-parent',
  template: `
    <h2>Parent Component</h2>
    <app-child [message]="parentMessage" [user]="currentUser"></app-child>
    <button (click)="updateMessage()">Update Message</button>
  `
})
export class ParentComponent {
  parentMessage: string = 'Message from parent';
  currentUser = { name: 'John Doe', role: 'Admin' };
  
  updateMessage() {
    // When this method is called, the child component will receive the updated value
    this.parentMessage = 'Updated message from parent';
  }
}

// Component using the signal service
import { Component, OnInit } from '@angular/core';
import { UserSignalService, User } from './user-signal.service';

@Component({
  selector: 'app-user-dashboard',
  template: `
    <div class="dashboard">
      <h2>User Dashboard</h2>
      
      <div *ngIf="service.loading()" class="loading">
        Loading users...
      </div>
      
      <div *ngIf="!service.loading()">
        <div class="user-section">
          <h3>All Users</h3>
          <ul class="user-list">
            <li *ngFor="let user of service.users()"
                [class.selected]="user.id === service.selectedUserId()"
                (click)="service.selectUser(user.id)">
              {{ user.name }} ({{ user.role }})
            </li>
          </ul>
          <button (click)="service.fetchUsers()">Refresh Users</button>
        </div>
        
        <div class="admin-section">
          <h3>Admin Users</h3>
          <ul class="user-list">
            <li *ngFor="let admin of service.adminUsers()">
              {{ admin.name }} - {{ admin.email }}
            </li>
          </ul>
        </div>
        
        <div *ngIf="service.selectedUser()" class="selected-user">
          <h3>Selected User Details</h3>
          <div class="user-details">
            <p><strong>Name:</strong> {{ service.selectedUser()?.name }}</p>
            <p><strong>Email:</strong> {{ service.selectedUser()?.email }}</p>
            <p><strong>Role:</strong> {{ service.selectedUser()?.role }}</p>
          </div>
          <button (click)="service.selectUser(null)">Clear Selection</button>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .dashboard {
      padding: 15px;
    }
    .loading {
      padding: 20px;
      text-align: center;
      font-style: italic;
    }
    .user-list {
      list-style-type: none;
      padding: 0;
    }
    .user-list li {
      padding: 5px 10px;
      margin: 5px 0;
      cursor: pointer;
      border-radius: 3px;
      background-color: #f5f5f5;
    }
    .user-list li:hover {
      background-color: #e0e0e0;
    }
    .user-list li.selected {
      background-color: #d0e8ff;
    }
    .user-section, .admin-section, .selected-user {
      margin-bottom: 20px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
  `]
})
export class UserDashboardComponent implements OnInit {
  constructor(public service: UserSignalService) {}
  
  ngOnInit(): void {
    // Fetch users when component initializes
    this.service.fetchUsers();
  }
}
```

### Signal Input Pattern
Angular 16+ provides a cleaner way to handle inputs using signals:

```typescript
// signal-input-child.component.ts
import { Component, input } from '@angular/core';

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

@Component({
  selector: 'app-product-card',
  template: `
    <div class="product-card" [class.out-of-stock]="!product().inStock">
      <h3>{{ product().name }}</h3>
      <p class="price">${{ product().price.toFixed(2) }}</p>
      <p class="stock-status">
        {{ product().inStock ? 'In Stock' : 'Out of Stock' }}
      </p>
      <button 
        [disabled]="!product().inStock" 
        (click)="onAddToCart()">
        Add to Cart
      </button>
    </div>
  `,
  styles: [`
    .product-card {
      border: 1px solid #ddd;
      padding: 15px;
      border-radius: 4px;
      width: 200px;
    }
    .out-of-stock {
      opacity: 0.7;
    }
    .price {
      font-weight: bold;
      font-size: 1.2em;
    }
    .stock-status {
      font-style: italic;
    }
  `]
})
export class ProductCardComponent {
  // Using the new signal input syntax
  product = input.required<Product>();
  
  // Optional input with default value
  showDetails = input(false);
  
  onAddToCart(): void {
    if (this.product().inStock) {
      console.log(`Added ${this.product().name} to cart`);
      // Could emit an event here
    }
  }
}
```

### Signal-based Form Controls:

```typescript
// signal-form.component.ts
import { Component, signal, computed } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-signal-form',
  template: `
    <div class="form-container">
      <h2>Registration Form with Signals</h2>
      
      <form [formGroup]="form" (ngSubmit)="onSubmit()">
        <div class="form-field">
          <label for="username">Username:</label>
          <input 
            id="username" 
            formControlName="username" 
            (input)="updateUsername($any($event.target).value)"
          >
          <div *ngIf="usernameError()" class="error">
            {{ usernameError() }}
          </div>
        </div>
        
        <div class="form-field">
          <label for="email">Email:</label>
          <input 
            id="email" 
            type="email" 
            formControlName="email"
            (input)="updateEmail($any($event.target).value)"
          >
          <div *ngIf="emailError()" class="error">
            {{ emailError() }}
          </div>
        </div>
        
        <div class="form-field">
          <label for="password">Password:</label>
          <input 
            id="password" 
            type="password" 
            formControlName="password"
            (input)="updatePassword($any($event.target).value)"
          >
          <div *ngIf="passwordError()" class="error">
            {{ passwordError() }}
          </div>
        </div>
        
        <div class="password-strength" *ngIf="password().length > 0">
          <label>Password Strength:</label>
          <div 
            class="strength-meter" 
            [class]="passwordStrengthClass()"
            [style.width.%]="passwordStrengthPercent()">
          </div>
          <span>{{ passwordStrengthText() }}</span>
        </div>
        
        <button type="submit" [disabled]="!formIsValid()">Register</button>
      </form>
      
      <div *ngIf="submitted()" class="success-message">
        Registration successful! Welcome, {{ username() }}!
      </div>
    </div>
  `,
  styles: [`
    .form-container {
      max-width: 400px;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    .form-field {
      margin-bottom: 15px;
    }
    .form-field label {
      display: block;
      margin-bottom: 5px;
    }
    .form-field input {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    .error {
      color: red;
      font-size: 0.8em;
      margin-top: 5px;
    }
    .strength-meter {
      height: 5px;
      margin-top: 5px;
      border-radius: 2px;
    }
    .strength-weak {
      background-color: red;
    }
    .strength-medium {
      background-color: orange;
    }
    .strength-strong {
      background-color: green;
    }
    .success-message {
      margin-top: 20px;
      padding: 10px;
      background-color: #dff0d8;
      border: 1px solid #d6e9c6;
      color: #3c763d;
      border-radius: 4px;
    }
  `]
})
export class SignalFormComponent {
  // Form group for Angular Reactive Forms
  form: FormGroup;
  
  // Signal state
  username = signal('');
  email = signal('');
  password = signal('');
  submitted = signal(false);
  
  // Computed signals for validation
  usernameError = computed(() => {
    const value = this.username();
    if (!value) return 'Username is required';
    if (value.length < 3) return 'Username must be at least 3 characters';
    return '';
  });
  
  emailError = computed(() => {
    const value = this.email();
    if (!value) return 'Email is required';
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) return 'Please enter a valid email address';
    return '';
  });
  
  passwordError = computed(() => {
    const value = this.password();
    if (!value) return 'Password is required';
    if (value.length < 6) return 'Password must be at least 6 characters';
    return '';
  });
  
  // Computed signals for password strength
  passwordStrength = computed(() => {
    const value = this.password();
    if (value.length === 0) return 0;
    
    let strength = 0;
    
    // Length check
    if (value.length >= 8) strength += 1;
    
    // Contains number
    if (/\d/.test(value)) strength += 1;
    
    // Contains special character
    if (/[!@#$%^&*]/.test(value)) strength += 1;
    
    // Contains uppercase
    if (/[A-Z]/.test(value)) strength += 1;
    
    return strength;
  });
  
  passwordStrengthPercent = computed(() => {
    return (this.passwordStrength() / 4) * 100;
  });
  
  passwordStrengthText = computed(() => {
    const strength = this.passwordStrength();
    if (strength === 0) return 'Very Weak';
    if (strength === 1) return 'Weak';
    if (strength === 2) return 'Medium';
    if (strength === 3) return 'Strong';
    return 'Very Strong';
  });
  
  passwordStrengthClass = computed(() => {
    const strength = this.passwordStrength();
    if (strength <= 1) return 'strength-weak';
    if (strength <= 2) return 'strength-medium';
    return 'strength-strong';
  });
  
  // Form validity
  formIsValid = computed(() => {
    return !this.usernameError() && !this.emailError() && !this.passwordError();
  });
  
  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(6)]]
    });
  }
  
  // Update signal values from form inputs
  updateUsername(value: string): void {
    this.username.set(value);
  }
  
  updateEmail(value: string): void {
    this.email.set(value);
  }
  
  updatePassword(value: string): void {
    this.password.set(value);
  }
  
  onSubmit(): void {
    if (this.formIsValid()) {
      console.log('Form submitted:', {
        username: this.username(),
        email: this.email(),
        password: '********' // Don't log actual password
      });
      
      this.submitted.set(true);
      
      // Reset form after 3 seconds
      setTimeout(() => {
        this.form.reset();
        this.username.set('');
        this.email.set('');
        this.password.set('');
        this.submitted.set(false);
      }, 3000);
    } else {
      // Mark all fields as touched to display validation errors
      Object.keys(this.form.controls).forEach(key => {
        this.form.get(key)?.markAsTouched();
      });
    }
  }
}
```

### Best Practices for Signals:

1. **Use readable signal names** that clearly indicate their purpose
2. **Prefer computed signals** over manual calculations when a value depends on other signals
3. **Use the asReadonly() method** when exposing signals from services
4. **Keep side effects in effect() blocks** rather than in computed signals
5. **Use signal() for mutable state** and computed() for derived state
6. **Minimize the use of set() and update()** to reduce the number of state updates
7. **Consider combining signals and RxJS** for complex async operations
8. **Use signals with OnPush change detection** for optimized performance
9. **Document signal dependencies** for better maintainability

### Signals vs RxJS Comparison

| Feature | Signals | RxJS Observables |
|---------|---------|------------------|
| **Primary Use Case** | UI state management | Async data streams |
| **Syntax Complexity** | Simpler, more concise | More operators, higher learning curve |
| **Change Detection** | Built for Angular's change detection | Requires manual integration |
| **Lazy/Eager** | Eager evaluation | Lazy evaluation |
| **Side Effects** | Via effect() | Via operators like tap() |
| **Multicasting** | Built-in | Requires operators like share() |
| **Error Handling** | Manual | Built-in error channel |
| **Cancellation** | Automatic with component | Requires unsubscribe |
| **Time-based Operations** | Limited | Comprehensive (debounce, throttle, etc.) |
| **Transformation** | Via computed() | Rich set of operators |
| **Memory Usage** | Generally lower | May be higher with complex chains |

[Back to Top](#table-of-contents)
```

#### Child Component:
```typescript
// child.component.ts
import { Component, Input } from '@angular/core';

interface User {
  name: string;
  role: string;
}

@Component({
  selector: 'app-child',
  template: `
    <div class="child-component">
      <h3>Child Component</h3>
      <p>Message received: {{ message }}</p>
      <p>User: {{ user.name }} ({{ user.role }})</p>
    </div>
  `,
  styles: [`
    .child-component {
      border: 1px solid blue;
      padding: 10px;
      margin: 10px 0;
    }
  `]
})
export class ChildComponent {
  // Basic input
  @Input() message: string = '';
  
  // Input with an object
  @Input() user: User = { name: '', role: '' };
  
  // Input with an alias
  @Input('userData') alternativeUserData: any;
  
  // Optional input with default value
  @Input() showDetails: boolean = false;
}
```

### Using Input Setters for Reactive Actions

You can use setter methods with `@Input()` to execute code when an input property changes:

```typescript
// child.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <div class="child-component">
      <h3>Child Component</h3>
      <p>Current count: {{ count }}</p>
      <p *ngIf="count > 10">Count is getting high!</p>
    </div>
  `
})
export class ChildComponent {
  private _count: number = 0;
  
  // Using a setter with @Input()
  @Input()
  set count(value: number) {
    this._count = value;
    // Perform actions when count changes
    if (value > 10) {
      console.log('Count is over 10!');
      // You could trigger other methods here
      this.handleHighCount();
    }
  }
  
  get count(): number {
    return this._count;
  }
  
  private handleHighCount() {
    // Logic to handle when count is high
  }
}
```

### OnChanges Lifecycle Hook

For more complex input change detection, you can implement the `OnChanges` interface:

```typescript
// child.component.ts
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <div class="child-component">
      <h3>Child Component</h3>
      <p>Name: {{ name }}</p>
      <p>Age: {{ age }}</p>
      <div *ngIf="changeLog.length > 0">
        <h4>Change Log:</h4>
        <ul>
          <li *ngFor="let log of changeLog">{{ log }}</li>
        </ul>
      </div>
    </div>
  `
})
export class ChildComponent implements OnChanges {
  @Input() name: string = '';
  @Input() age: number = 0;
  
  changeLog: string[] = [];
  
  ngOnChanges(changes: SimpleChanges) {
    // This method is called when any input property changes
    for (const propName in changes) {
      const change = changes[propName];
      const current = JSON.stringify(change.currentValue);
      const previous = JSON.stringify(change.previousValue);
      
      // Add to change log
      this.changeLog.push(
        `${propName}: changed from ${previous} to ${current}`
      );
    }
  }
}
```

### Best Practices for @Input():

1. **Use proper typing** for all input properties
2. **Provide default values** to prevent undefined errors
3. **Consider immutability** when passing objects or arrays
4. **Avoid excessive @Input properties** on a single component
5. **Document each @Input** with JSDoc comments for better code readability
6. **Use OnPush change detection** for optimized performance

[Back to Top](#table-of-contents)

## Child to Parent Communication (@Output)

The `@Output()` decorator with EventEmitter enables child components to send data up to their parent components through event binding.

### Key Features:
- **Event-based communication** from child to parent
- **Custom event emission** with payload data
- **Strong typing** for event payloads
- **Multiple events** from a single component

### Basic Implementation

#### Child Component:
```typescript
// child.component.ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `
    <div class="child-component">
      <h3>Child Component</h3>
      <button (click)="sendMessage()">Send Message to Parent</button>
      <button (click)="notifyParent()">Notify Parent</button>
      <div>
        <input #itemInput placeholder="Add new item">
        <button (click)="addNewItem(itemInput.value); itemInput.value=''">Add</button>
      </div>
    </div>
  `,
  styles: [`
    .child-component {
      border: 1px solid green;
      padding: 10px;
      margin: 10px 0;
    }
  `]
})
export class ChildComponent {
  // Basic event emitter
  @Output() messageEvent = new EventEmitter<string>();
  
  // Event emitter with complex data
  @Output() itemAdded = new EventEmitter<{name: string, timestamp: Date}>();
  
  // Event emitter with no payload
  @Output() notify = new EventEmitter<void>();
  
  sendMessage() {
    this.messageEvent.emit('Hello from child component!');
  }
  
  addNewItem(name: string) {
    if (name.trim()) {
      // Emit an event with a complex object
      this.itemAdded.emit({
        name: name,
        timestamp: new Date()
      });
    }
  }
  
  notifyParent() {
    // Emit an event with no data
    this.notify.emit();
  }
}
```

#### Parent Component:
```typescript
// parent.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-parent',
  template: `
    <div class="parent-component">
      <h2>Parent Component</h2>
      
      <!-- Message from child will appear here -->
      <p *ngIf="message">Message from child: {{ message }}</p>
      
      <!-- List of items added by child -->
      <div *ngIf="items.length > 0">
        <h3>Items Added:</h3>
        <ul>
          <li *ngFor="let item of items">
            {{ item.name }} (added at {{ item.timestamp | date:'short' }})
          </li>
        </ul>
      </div>
      
      <!-- Notification indicator -->
      <p *ngIf="notificationReceived">
        Notification received at {{ notificationTime | date:'medium' }}
      </p>
      
      <!-- Child component with event bindings -->
      <app-child 
        (messageEvent)="receiveMessage($event)"
        (itemAdded)="onItemAdded($event)"
        (notify)="onNotify()">
      </app-child>
    </div>
  `,
  styles: [`
    .parent-component {
      border: 1px solid #ccc;
      padding: 15px;
    }
  `]
})
export class ParentComponent {
  message: string = '';
  items: Array<{name: string, timestamp: Date}> = [];
  notificationReceived = false;
  notificationTime?: Date;
  
  // Handler for messageEvent
  receiveMessage(msg: string) {
    this.message = msg;
    console.log('Received message from child:', msg);
  }
  
  // Handler for itemAdded event
  onItemAdded(item: {name: string, timestamp: Date}) {
    this.items.push(item);
    console.log('Item added:', item);
  }
  
  // Handler for notify event
  onNotify() {
    this.notificationReceived = true;
    this.notificationTime = new Date();
    console.log('Notification received from child');
    
    // Reset notification after 3 seconds
    setTimeout(() => {
      this.notificationReceived = false;
    }, 3000);
  }
}
```

### Sample Output:
When the user clicks "Send Message to Parent" button in the child component:
```
Parent Component
Message from child: Hello from child component!

Child Component
[Send Message to Parent] [Notify Parent]
[____________] [Add]
```

When the user adds an item:
```
Parent Component
Items Added:
• Book (added at 5/2/25, 2:30 PM)
• Laptop (added at 5/2/25, 2:31 PM)

Child Component
[Send Message to Parent] [Notify Parent]
[____________] [Add]
```

### Using EventEmitter with Forms

A common pattern is to emit form data from a child component:

```typescript
// child-form.component.ts
import { Component, Output, EventEmitter } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

interface UserForm {
  name: string;
  email: string;
  age: number;
}

@Component({
  selector: 'app-user-form',
  template: `
    <div class="user-form">
      <h3>User Form</h3>
      <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
        <div>
          <label for="name">Name:</label>
          <input id="name" formControlName="name">
          <div *ngIf="userForm.get('name')?.invalid && userForm.get('name')?.touched" class="error">
            Name is required
          </div>
        </div>
        
        <div>
          <label for="email">Email:</label>
          <input id="email" type="email" formControlName="email">
          <div *ngIf="userForm.get('email')?.invalid && userForm.get('email')?.touched" class="error">
            Valid email is required
          </div>
        </div>
        
        <div>
          <label for="age">Age:</label>
          <input id="age" type="number" formControlName="age">
        </div>
        
        <button type="submit" [disabled]="userForm.invalid">Submit</button>
      </form>
    </div>
  `,
  styles: [`
    .user-form {
      border: 1px solid #ddd;
      padding: 15px;
      margin: 10px 0;
    }
    .error {
      color: red;
      font-size: 0.8em;
      margin-top: 5px;
    }
  `]
})
export class UserFormComponent {
  @Output() formSubmitted = new EventEmitter<UserForm>();
  
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      age: [null, [Validators.min(18), Validators.max(100)]]
    });
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      // Emit the form values to the parent
      this.formSubmitted.emit(this.userForm.value);
      
      // Optionally reset the form
      this.userForm.reset();
    } else {
      // Mark all fields as touched to trigger validation messages
      Object.keys(this.userForm.controls).forEach(key => {
        const control = this.userForm.get(key);
        control?.markAsTouched();
      });
    }
  }
}
```

### Best Practices for @Output():

1. **Use descriptive event names** that clearly indicate the action or event
2. **Type your EventEmitter** correctly for better type safety
3. **Document your events** with JSDoc for better maintainability
4. **Use void type** for events that don't need to pass data
5. **Avoid emitting too frequently** to prevent performance issues
6. **Consider debouncing or throttling** for frequent events like input changes
7. **Keep component responsibilities clear** - don't emit events that should be handled internally

[Back to Top](#table-of-contents)

## Using Services for Communication

Services in Angular provide a way for components to communicate regardless of their relationship in the component hierarchy. This is especially useful for:

- Communication between sibling components
- Communication between unrelated components
- Centralizing shared state and functionality

### Key Features:
- **Decoupled communication** between any components
- **State management** across the application
- **Observable-based** reactive data flow
- **Dependency injection** for easy consumption
- **Singleton instances** for global state sharing

### Basic Implementation

#### Shared Service:
```typescript
// message.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root' // This makes the service a singleton at the root level
})
export class MessageService {
  // BehaviorSubject holds the current value and emits it to new subscribers
  private messageSource = new BehaviorSubject<string>('Initial message');
  
  // Expose an Observable for components to subscribe to
  currentMessage$: Observable<string> = this.messageSource.asObservable();
  
  // Method to update the message
  updateMessage(message: string): void {
    this.messageSource.next(message);
    console.log('Message updated in service:', message);
  }
}
```

#### Sender Component:
```typescript
// sender.component.ts
import { Component } from '@angular/core';
import { MessageService } from '../services/message.service';

@Component({
  selector: 'app-sender',
  template: `
    <div class="sender-component">
      <h2>Sender Component</h2>
      <input #messageInput placeholder="Type a message">
      <button (click)="sendMessage(messageInput.value)">Send Message</button>
    </div>
  `,
  styles: [`
    .sender-component {
      border: 1px solid blue;
      padding: 10px;
      margin: 10px 0;
    }
  `]
})
export class SenderComponent {
  constructor(private messageService: MessageService) {}
  
  sendMessage(message: string): void {
    if (message.trim()) {
      // Update the message in the service
      this.messageService.updateMessage(message);
    }
  }
}
```

#### Receiver Component:
```typescript
// receiver.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { MessageService } from '../services/message.service';
import { Subscription } from 'rxjs';

@Component({
  selector: 'app-receiver',
  template: `
    <div class="receiver-component">
      <h2>Receiver Component</h2>
      <p>Current message: {{ message }}</p>
      <p>Message history:</p>
      <ul>
        <li *ngFor="let msg of messageHistory">{{ msg }}</li>
      </ul>
    </div>
  `,
  styles: [`
    .receiver-component {
      border: 1px solid green;
      padding: 10px;
      margin: 10px 0;
    }
  `]
})
export class ReceiverComponent implements OnInit, OnDestroy {
  message: string = '';
  messageHistory: string[] = [];
  private subscription: Subscription = new Subscription();
  
  constructor(private messageService: MessageService) {}
  
  ngOnInit(): void {
    // Subscribe to the message service
    this.subscription = this.messageService.currentMessage$.subscribe(
      (message: string) => {
        this.message = message;
        
        // Add to history if it's a new message
        if (this.messageHistory.indexOf(message) === -1) {
          this.messageHistory.push(message);
        }
      }
    );
  }
  
  ngOnDestroy(): void {
    // Clean up the subscription when the component is destroyed
    this.subscription.unsubscribe();
  }
}
```

#### Parent App Component:
```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app-container">
      <h1>Component Communication with Services</h1>
      <div class="component-row">
        <app-sender></app-sender>
        <app-receiver></app-receiver>
      </div>
    </div>
  `,
  styles: [`
    .app-container {
      padding: 20px;
    }
    .component-row {
      display: flex;
      gap: 20px;
    }
  `]
})
export class AppComponent {}
```

### Sample Output:
When the user types "Hello World" in the sender component and clicks "Send Message":
```
Component Communication with Services

┌─────────────────────┐  ┌─────────────────────┐
│ Sender Component    │  │ Receiver Component  │
│                     │  │                     │
│ [Hello World      ] │  │ Current message:    │
│ [Send Message]      │  │ Hello World         │
│                     │  │                     │
│                     │  │ Message history:    │
│                     │  │ • Initial message   │
│                     │  │ • Hello World       │
└─────────────────────┘  └─────────────────────┘
```

### Advanced Service Communication with State Management

For more complex applications, you might want to manage state more effectively:

```typescript
// user-state.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface User {
  id: number;
  name: string;
  email: string;
  isAdmin: boolean;
}

export interface AppState {
  users: User[];
  selectedUserId: number | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: AppState = {
  users: [],
  selectedUserId: null,
  isLoading: false,
  error: null
};

@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  // Private BehaviorSubject for the state
  private state = new BehaviorSubject<AppState>(initialState);
  
  // Public observables (selectors)
  readonly state$: Observable<AppState> = this.state.asObservable();
  readonly users$: Observable<User[]> = this.state$.pipe(map(state => state.users));
  readonly selectedUser$: Observable<User | undefined> = this.state$.pipe(
    map(state => state.users.find(user => user.id === state.selectedUserId))
  );
  readonly isLoading$: Observable<boolean> = this.state$.pipe(map(state => state.isLoading));
  readonly error$: Observable<string | null> = this.state$.pipe(map(state => state.error));
  
  // Get the current state value
  private get currentState(): AppState {
    return this.state.getValue();
  }
  
  // State update methods
  setLoading(isLoading: boolean): void {
    this.updateState({ isLoading });
  }
  
  setError(error: string | null): void {
    this.updateState({ error });
  }
  
  setUsers(users: User[]): void {
    this.updateState({ users });
  }
  
  addUser(user: User): void {
    const users = [...this.currentState.users, user];
    this.updateState({ users });
  }
  
  updateUser(updatedUser: User): void {
    const users = this.currentState.users.map(user => 
      user.id === updatedUser.id ? updatedUser : user
    );
    this.updateState({ users });
  }
  
  deleteUser(userId: number): void {
    const users = this.currentState.users.filter(user => user.id !== userId);
    this.updateState({ users });
  }
  
  selectUser(userId: number | null): void {
    this.updateState({ selectedUserId: userId });
  }
  
  // Private helper to update state immutably
  private updateState(partialState: Partial<AppState>): void {
    this.state.next({
      ...this.currentState,
      ...partialState
    });
  }
  
  // Simulate fetching users from an API
  fetchUsers(): void {
    this.setLoading(true);
    this.setError(null);
    
    // Simulate API call
    setTimeout(() => {
      try {
        const mockUsers: User[] = [
          { id: 1, name: 'John Doe', email: 'john@example.com', isAdmin: false },
          { id: 2, name: 'Jane Smith', email: 'jane@example.com', isAdmin: true },
          { id: 3, name: 'Bob Johnson', email: 'bob@example.com', isAdmin: false }
        ];
        
        this.setUsers(mockUsers);
        this.setLoading(false);
      } catch (error) {
        this.setError('Failed to fetch users');
        this.setLoading(false);
      }
    }, 1000);
  }
}
```

### Best Practices for Service Communication:

1. **Use RxJS Observables** for reactive data flow
2. **Implement proper cleanup** with unsubscribe in ngOnDestroy
3. **Consider using the async pipe** to automatically handle subscriptions
4. **Use immutable patterns** for updating state
5. **Keep services focused** on a specific domain or feature
6. **Use hierarchical injection** when service state should be scoped to a component subtree
7. **Use the OnPush change detection strategy** for optimized performance
8. **Document your service methods and observables** for better maintainability

[Back to Top](#table-of-contents)

## Signals in Angular

Signals are a feature introduced in Angular 16+ that provide an alternative to RxJS Observables for managing reactive state. They offer a more streamlined API specifically designed for Angular's change detection and represent Angular's modern approach to reactive programming.

### Key Features:
- **Fine-grained reactivity** for optimal performance
- **Simplified syntax** compared to RxJS
- **Automatic dependency tracking**
- **Integration with Angular's change detection**
- **Computed values** that update automatically
- **Effects** for performing side effects

### Basic Implementation

#### Creating and Using Signals:

```typescript
// counter.component.ts
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div class="counter">
      <h2>Counter with Signals</h2>
      <p>Count: {{ count() }}</p>
      <p>Doubled: {{ doubled() }}</p>
      <p>Status: {{ status() }}</p>
      
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
      <button (click)="reset()">Reset</button>
    </div>
  `,
  styles: [`
    .counter {
      border: 1px solid #ddd;
      padding: 15px;
      border-radius: 4px;
    }
  `]
})
export class CounterComponent {
  // Create a signal with an initial value
  count = signal(0);
  
  // Computed signal that depends on count
  doubled = computed(() => this.count() * 2);
  
  // Another computed signal with more complex logic
  status = computed(() => {
    const currentCount = this.count();
    if (currentCount === 0) return 'Zero';
    if (currentCount < 0) return 'Negative';
    if (currentCount > 10) return 'High';
    return 'Positive';
  });
  
  constructor() {
    // Creating an effect (side effect when signal changes)
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
      // You could call other methods, log analytics, etc.
    });
  }
  
  increment() {
    // Update the signal value
    this.count.update(value => value + 1);
  }
  
  decrement() {
    this.count.update(value => value - 1);
  }
  
  reset() {
    // Set the signal to a specific value
    this.count.set(0);
  }
}
```

### Sample Output:
```
Counter with Signals

Count: 2
Doubled: 4
Status: Positive

[Increment] [Decrement] [Reset]
```

Console output after clicking increment:
```
Count changed to: 3
```

### Component Communication with Signals

Signals can also be used for component communication:

#### Parent Component:
```typescript
// parent.component.ts
import { Component, signal } from '@angular/core';

export interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-signal-parent',
  template: `
    <div class="parent">
      <h2>Parent Component with Signals</h2>
      
      <p>Selected User ID: {{ selectedUserId() || 'None' }}</p>
      
      <app-signal-child 
        [users]="users" 
        [selectedId]="selectedUserId"
        (selectUser)="selectUser($event)">
      </app-signal-child>
      
      <div class="actions">
        <button (click)="addUser()">Add Random User</button>
        <button (click)="clearSelection()">Clear Selection</button>
      </div>
    </div>
  `,
  styles: [`
    .parent {
      border: 1px solid #ccc;
      padding: 15px;
      margin: 10px 0;
    }
    .actions {
      margin-top: 15px;
    }
  `]
})
export class SignalParentComponent {
  // Signal for the list of users
  users = signal<User[]>([
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' },
    { id: 3, name: 'Charlie', email: 'charlie@example.com' }
  ]);
  
  // Signal for the selected user ID
  selectedUserId = signal<number | null>(null);
  
  selectUser(id: number) {
    this.selectedUserId.set(id);
  }
  
  clearSelection() {
    this.selectedUserId.set(null);
  }
  
  addUser() {
    const id = this.users().length + 1;
    this.users.update(users => [
      ...users,
      { 
        id, 
        name: `User ${id}`, 
        email: `user${id}@example.com` 
      }
    ]);
  }
}
```

#### Child Component:
```typescript
// child.component.ts
import { Component, Input, Output, EventEmitter, signal } from '@angular/core';
import { User } from './parent.component';

@Component({
  selector: 'app-signal-child',
  template: `
    <div class="child">
      <h3>Child Component</h3>
      
      <ul class="user-list">
        <li *ngFor="let user of users()" 
            [class.selected]="user.id === selectedId()"
            (click)="onSelectUser(user.id)">
          {{ user.name }} ({{ user.email }})
        </li>
      </ul>
    </div>
  `,
  styles: [`
    .child {
      border: 1px solid #ddd;
      padding: 10px;
      margin-top: 10px;
    }
    .user-list {
      list-style-type: none;
      padding: 0;
    }
    .user-list li {
      padding: 5px;
      margin: 5px 0;
      cursor: pointer;
      border-radius: 4px;
    }
    .user-list li:hover {
      background-color: #f0f0f0;
    }
    .user-list li.selected {
      background-color: #e0f0ff;
      font-weight: bold;
    }
  `]
})
export class SignalChildComponent {
  // Input signal from parent
  @Input({ required: true }) users = signal<User[]>([]);
  
  // Input signal for selected ID
  @Input({ required: true }) selectedId = signal<number | null>(null);
  
  // Output event to notify parent
  @Output() selectUser = new EventEmitter<number>();
  
  onSelectUser(id: number): void {
    this.selectUser.emit(id);
  }
}
```

### Using Signals in Services

Signals can also be used in services for cross-component communication:

```typescript
// user-signal.service.ts
import { Injectable, signal, computed } from '@angular/core';

export interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserSignalService {
  // Private signals for internal state management
  private usersSignal = signal<User[]>([]);
  private selectedUserIdSignal = signal<number | null>(null);
  private loadingSignal = signal<boolean>(false);
  
  // Public computed signals (read-only)
  readonly users = this.usersSignal.asReadonly();
  readonly selectedUserId = this.selectedUserIdSignal.asReadonly();
  readonly loading = this.loadingSignal.asReadonly();
  
  // Computed signal for selected user
  readonly selectedUser = computed(() => {
    const id = this.selectedUserIdSignal();
    if (id === null) return null;
    return this.usersSignal().find(user => user.id === id) || null;
  });
  
  // Computed signal for admin users
  readonly adminUsers = computed(() => 
    this.usersSignal().filter(user => user.role === 'admin')
  );
  
  constructor() {
    // Initialize with some data
    this.setUsers([
      { id: 1, name: 'John Doe', email: 'john@example.com', role: 'user' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'admin' },
      { id: 3, name: 'Bob Johnson', email: 'bob@example.com', role: 'user' }
    ]);
  }
  
  // Action methods
  setUsers(users: User[]): void {
    this.usersSignal.set(users);
  }
  
  addUser(user: User): void {
    this.usersSignal.update(users => [...users, user]);
  }
  
  updateUser(updatedUser: User): void {
    this.usersSignal.update(users => 
      users.map(user => user.id === updatedUser.id ? updatedUser : user)
    );
  }
  
  removeUser(id: number): void {
    this.usersSignal.update(users => users.filter(user => user.id !== id));
  }
  
  selectUser(id: number | null): void {
    this.selectedUserIdSignal.set(id);
  }
  
  // Simulated API call
  fetchUsers(): void {
    this.loadingSignal.set(true);
    
    // Simulate API delay
    setTimeout(() => {
      const mockUsers: User[] = [
        { id: 1, name: 'Alice Parker', email: 'alice@example.com', role: 'admin' },
        { id: 2, name: 'Bob Wright', email: 'bob@example.com', role: 'user' },
        { id: 3, name: 'Charlie Davis', email: 'charlie@example.com', role: 'user' },
        { id: 4, name: 'Diana Edwards', email: 'diana@example.com', role: 'admin' }
      ];
      
      this.usersSignal.set(mockUsers);
      this.loadingSignal.set(false);
    }, 1000);
  }
}