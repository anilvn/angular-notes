# Angular Asynchronous Validators

## Table of Contents

| # | Topic |
|---|-------|
| **Asynchronous Validators Fundamentals** |
| 1 | [Introduction to Asynchronous Validators](#introduction-to-asynchronous-validators) |
| 2 | [Creating Custom Asynchronous Validators](#creating-custom-asynchronous-validators) |
| 3 | [Combining Synchronous and Asynchronous Validators](#combining-synchronous-and-asynchronous-validators) |
| **Form Controls and Events** |
| 4 | [Form Control States](#form-control-states) |
| 5 | [setValue() vs patchValue()](#setvalue-vs-patchvalue) |
| 6 | [valueChanges and statusChanges](#valuechanges-and-statuschanges) |
| **Best Practices and Advanced Techniques** |
| 7 | [Performance Optimization](#performance-optimization) |
| 8 | [Error Handling](#error-handling) |
| 9 | [Testing Asynchronous Validators](#testing-asynchronous-validators) |

## Introduction to Asynchronous Validators

Asynchronous validators are specialized form validators in Angular that perform validation logic requiring asynchronous operations such as API calls, database queries, or other time-consuming validations.

### Key Features:

- **Return Promise/Observable**: Asynchronous validators return either a Promise or an Observable that resolves to validation errors or null
- **Server-side validation**: Used primarily for validations that require server interaction
- **Pending state**: Forms show a pending state during asynchronous validation
- **Delayed validation**: Executes after all synchronous validators have passed

### Basic Structure:

```typescript
function asyncValidator(control: AbstractControl): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
  // Validation logic that returns Promise or Observable
  return someAsyncOperation().pipe(
    map(result => result ? { customError: true } : null)
  );
}
```

### Common Use Cases:

- Username availability checking
- Email existence verification
- ZIP code validation
- Credit card verification
- Domain name availability

### Example Implementation:

```typescript
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, delay } from 'rxjs/operators';

export class UserService {
  // Simulated database
  private existingUsers = ['admin', 'john', 'test'];
  
  checkUsernameExists(username: string): Observable<boolean> {
    // Simulate API call
    return of(this.existingUsers.includes(username))
      .pipe(delay(1000)); // Simulate network delay
  }
}

export class CustomAsyncValidators {
  static usernameAvailable(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value) {
        return of(null); // Skip validation if empty
      }
      
      return userService.checkUsernameExists(control.value).pipe(
        map(exists => exists ? { usernameExists: true } : null)
      );
    };
  }
}
```

[Back to Top](#table-of-contents)

## Creating Custom Asynchronous Validators

Custom asynchronous validators allow you to implement business-specific validation logic that requires server interaction or other asynchronous operations.

### Implementation Steps:

1. **Create a service** for API interactions
2. **Define validator function** that returns AsyncValidatorFn
3. **Return an Observable/Promise** with validation results
4. **Apply the validator** to form controls

### Example - Email Domain Validator:

```typescript
import { Injectable } from '@angular/core';
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { map, catchError, debounceTime, switchMap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class EmailValidationService {
  constructor(private http: HttpClient) {}
  
  // Check if email domain is valid
  validateEmailDomain(email: string): Observable<boolean> {
    // Extract domain from email
    const domain = email.substring(email.lastIndexOf('@') + 1);
    
    // Call API to validate domain (example endpoint)
    return this.http.get<any>(`https://api.example.com/validate-domain/${domain}`).pipe(
      map(response => response.valid),
      catchError(() => of(false))
    );
  }
}

export class EmailValidators {
  // Custom async validator for validating email domain
  static validDomain(service: EmailValidationService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      const email = control.value;
      
      // Skip validation if empty or not an email format
      if (!email || !email.includes('@')) {
        return of(null);
      }
      
      return of(email).pipe(
        debounceTime(500), // Avoid API spam on every keystroke
        switchMap(emailValue => 
          service.validateEmailDomain(emailValue).pipe(
            map(isValid => isValid ? null : { invalidDomain: true })
          )
        )
      );
    };
  }
}

// Usage in component:
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { EmailValidationService, EmailValidators } from './email-validators';

@Component({
  selector: 'app-email-form',
  template: `
    <form [formGroup]="emailForm">
      <div>
        <label for="email">Email:</label>
        <input id="email" type="email" formControlName="email">
        
        <!-- Show pending state -->
        <div *ngIf="email.pending">
          Validating email domain...
        </div>
        
        <!-- Show error messages -->
        <div *ngIf="email.invalid && (email.dirty || email.touched)">
          <div *ngIf="email.errors?.required">Email is required</div>
          <div *ngIf="email.errors?.email">Please enter a valid email format</div>
          <div *ngIf="email.errors?.invalidDomain">The email domain is invalid or unreachable</div>
        </div>
      </div>
      <button type="submit" [disabled]="emailForm.invalid || emailForm.pending">Submit</button>
    </form>
  `
})
export class EmailFormComponent implements OnInit {
  emailForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private emailValidationService: EmailValidationService
  ) {}
  
  ngOnInit() {
    this.emailForm = this.fb.group({
      email: ['', 
        [Validators.required, Validators.email], // Synchronous validators
        [EmailValidators.validDomain(this.emailValidationService)] // Async validator
      ]
    });
  }
  
  get email() { return this.emailForm.get('email')!; }
}
```

### Key Considerations:

- **Debounce input** to reduce unnecessary API calls
- **Handle edge cases** like empty values or invalid formats
- **Provide visual feedback** during validation with pending state
- **Handle errors** from API calls gracefully

[Back to Top](#table-of-contents)

## Combining Synchronous and Asynchronous Validators

Angular allows combining both synchronous and asynchronous validators for robust form validation. Synchronous validators run first, and if they pass, asynchronous validators execute.

### Implementation Strategy:

```typescript
// In component class
this.form = this.fb.group({
  username: ['', 
    // First parameter: initial value
    
    // Second parameter: synchronous validators (runs first)
    [
      Validators.required,
      Validators.minLength(4),
      Validators.pattern(/^[a-zA-Z0-9_-]*$/)
    ],
    
    // Third parameter: asynchronous validators (runs after sync validators pass)
    [
      CustomAsyncValidators.usernameAvailable(this.userService)
    ]
  ]
});
```

### Complete Registration Form Example:

```typescript
import { Component, OnInit } from '@angular/core';
import { AbstractControl, AsyncValidatorFn, FormBuilder, FormGroup, ValidationErrors, Validators } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { debounceTime, distinctUntilChanged, map, switchMap } from 'rxjs/operators';

// User service for backend validation
export class UserService {
  private existingUsernames = ['admin', 'john', 'jane', 'user1', 'test'];
  private existingEmails = ['admin@example.com', 'john@example.com', 'test@test.com'];
  
  checkUsernameExists(username: string): Observable<boolean> {
    // Simulate API delay
    return of(this.existingUsernames.includes(username.toLowerCase()))
      .pipe(debounceTime(800));
  }
  
  checkEmailExists(email: string): Observable<boolean> {
    return of(this.existingEmails.includes(email.toLowerCase()))
      .pipe(debounceTime(800));
  }
}

// Custom validators
export class FormValidators {
  // Synchronous password strength validator
  static passwordStrength(control: AbstractControl): ValidationErrors | null {
    const value = control.value;
    
    if (!value) {
      return null;
    }
    
    const hasUpperCase = /[A-Z]+/.test(value);
    const hasLowerCase = /[a-z]+/.test(value);
    const hasNumeric = /[0-9]+/.test(value);
    const hasSpecial = /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]+/.test(value);
    
    const passwordValid = hasUpperCase && hasLowerCase && hasNumeric && hasSpecial;
    
    return !passwordValid ? {
      passwordStrength: {
        hasUpperCase,
        hasLowerCase,
        hasNumeric,
        hasSpecial
      }
    } : null;
  }
  
  // Asynchronous username validation
  static usernameAvailable(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value || control.value.length < 4) {
        return of(null);
      }
      
      return control.valueChanges.pipe(
        debounceTime(500),
        distinctUntilChanged(),
        switchMap(value => userService.checkUsernameExists(value)),
        map(exists => exists ? { usernameTaken: true } : null),
      );
    };
  }
  
  // Asynchronous email validation
  static emailAvailable(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value || !control.value.includes('@')) {
        return of(null);
      }
      
      return control.valueChanges.pipe(
        debounceTime(500),
        distinctUntilChanged(),
        switchMap(value => userService.checkEmailExists(value)),
        map(exists => exists ? { emailTaken: true } : null),
      );
    };
  }
}

@Component({
  selector: 'app-registration-form',
  template: `
    <form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
      <!-- Username field -->
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <div *ngIf="username.pending" class="pending">
          <span class="spinner"></span> Checking availability...
        </div>
        
        <div *ngIf="username.invalid && (username.dirty || username.touched)" class="error">
          <div *ngIf="username.errors?.required">Username is required</div>
          <div *ngIf="username.errors?.minlength">Username must be at least 4 characters</div>
          <div *ngIf="username.errors?.pattern">Username can only contain letters, numbers, hyphens and underscores</div>
          <div *ngIf="username.errors?.usernameTaken">This username is already taken</div>
        </div>
      </div>
      
      <!-- Email field -->
      <div class="form-group">
        <label for="email">Email</label>
        <input type="email" id="email" formControlName="email">
        
        <div *ngIf="email.pending" class="pending">
          <span class="spinner"></span> Checking email...
        </div>
        
        <div *ngIf="email.invalid && (email.dirty || email.touched)" class="error">
          <div *ngIf="email.errors?.required">Email is required</div>
          <div *ngIf="email.errors?.email">Please enter a valid email address</div>
          <div *ngIf="email.errors?.emailTaken">This email is already registered</div>
        </div>
      </div>
      
      <!-- Password field with strength validation -->
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" id="password" formControlName="password">
        
        <div *ngIf="password.invalid && (password.dirty || password.touched)" class="error">
          <div *ngIf="password.errors?.required">Password is required</div>
          <div *ngIf="password.errors?.minlength">Password must be at least 8 characters</div>
          <div *ngIf="password.errors?.passwordStrength">
            Password must contain:
            <ul>
              <li [class.valid]="!password.errors?.passwordStrength?.hasUpperCase">Uppercase letter</li>
              <li [class.valid]="!password.errors?.passwordStrength?.hasLowerCase">Lowercase letter</li>
              <li [class.valid]="!password.errors?.passwordStrength?.hasNumeric">Number</li>
              <li [class.valid]="!password.errors?.passwordStrength?.hasSpecial">Special character</li>
            </ul>
          </div>
        </div>
      </div>
      
      <button type="submit" [disabled]="registrationForm.invalid || registrationForm.pending">Register</button>
    </form>
  `,
  styles: [`
    .pending { color: blue; }
    .error { color: red; }
    .valid { color: green; }
    .spinner { display: inline-block; width: 12px; height: 12px; border-radius: 50%; 
               border: 2px solid #ccc; border-top-color: #333; animation: spin 1s infinite linear; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  `],
  providers: [UserService]
})
export class RegistrationFormComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      // Username with both sync and async validators
      username: ['', 
        [
          Validators.required,
          Validators.minLength(4),
          Validators.pattern(/^[a-zA-Z0-9_-]*$/)
        ],
        [
          FormValidators.usernameAvailable(this.userService)
        ]
      ],
      
      // Email with both sync and async validators
      email: ['',
        [
          Validators.required,
          Validators.email
        ],
        [
          FormValidators.emailAvailable(this.userService)
        ]
      ],
      
      // Password with sync validators only
      password: ['', 
        [
          Validators.required,
          Validators.minLength(8),
          FormValidators.passwordStrength
        ]
      ]
    });
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted:', this.registrationForm.value);
      // Submit to backend API
    } else {
      this.markFormGroupTouched(this.registrationForm);
    }
  }
  
  // Helper to mark all controls as touched (to show validation errors)
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Getters for template access
  get username() { return this.registrationForm.get('username')!; }
  get email() { return this.registrationForm.get('email')!; }
  get password() { return this.registrationForm.get('password')!; }
}
```

### Execution Order:

1. **Synchronous validators** run first
2. **If synchronous validators pass**, asynchronous validators execute
3. **Form control status** changes to "PENDING" during async validation
4. **Final validation result** is determined after async validation completes

### Best Practices:

- **Use synchronous validators** for immediate feedback on basic validation
- **Reserve asynchronous validators** for complex server-dependent validations
- **Always handle both pending state** and error messages in the UI
- **Disable submit buttons** when the form is in pending state

[Back to Top](#table-of-contents)

## Form Control States

Angular form controls have multiple states that help track their validation status and user interaction. Understanding these states is crucial for effectively working with asynchronous validators.

### Key Form Control States:

| State | Description | Property |
|-------|-------------|----------|
| **Pristine/Dirty** | Tracks if value has changed | `pristine`, `dirty` |
| **Untouched/Touched** | Tracks if control has been focused/blurred | `untouched`, `touched` |
| **Valid/Invalid** | Tracks if all validators have passed | `valid`, `invalid` |
| **Pending** | Tracks if async validators are running | `pending` |
| **Disabled/Enabled** | Tracks if control is disabled | `disabled`, `enabled` |

### Working with Pending State:

The `pending` state is particularly important for asynchronous validators, as it indicates validation is in progress.

```typescript
// In component template
<div *ngIf="username.pending" class="alert alert-info">
  <span class="spinner-border spinner-border-sm"></span>
  Checking username availability...
</div>

// Conditionally disable submit button
<button type="submit" [disabled]="form.invalid || form.pending">Submit</button>
```

### Status Changes Example:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { UserService } from './user.service';
import { AsyncValidators } from './async-validators';

@Component({
  selector: 'app-form-states',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div class="form-field">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <!-- Status indicators -->
        <div class="status-indicators">
          <span class="badge" [ngClass]="getStatusClass('pristine')">
            Pristine: {{ username.pristine }}
          </span>
          <span class="badge" [ngClass]="getStatusClass('touched')">
            Touched: {{ username.touched }}
          </span>
          <span class="badge" [ngClass]="getStatusClass('valid')">
            Valid: {{ username.valid }}
          </span>
          <span class="badge" [ngClass]="getStatusClass('pending')">
            Pending: {{ username.pending }}
          </span>
        </div>
        
        <!-- Pending state -->
        <div *ngIf="username.pending" class="pending-message">
          <span class="spinner"></span> Validating...
        </div>
        
        <!-- Error messages -->
        <div *ngIf="username.invalid && (username.dirty || username.touched)" class="error-message">
          <div *ngIf="username.errors?.required">Username is required</div>
          <div *ngIf="username.errors?.minlength">Username must be at least 4 characters</div>
          <div *ngIf="username.errors?.usernameTaken">This username is already taken</div>
        </div>
      </div>
      
      <button type="submit" [disabled]="userForm.invalid || userForm.pending">Submit</button>
      <button type="button" (click)="resetForm()">Reset</button>
    </form>
    
    <!-- Current form state debugging -->
    <div class="debug-panel">
      <h3>Form Status</h3>
      <pre>Status: {{ userForm.status }}</pre>
      <pre>Value: {{ userForm.value | json }}</pre>
      <pre>Errors: {{ usernameErrors | json }}</pre>
    </div>
  `,
  styles: [`
    .status-indicators { margin-top: 8px; }
    .badge { padding: 4px 8px; margin-right: 4px; border-radius: 4px; }
    .badge-primary { background-color: #007bff; color: white; }
    .badge-secondary { background-color: #6c757d; color: white; }
    .pending-message { color: blue; margin-top: 8px; }
    .error-message { color: red; margin-top: 8px; }
    .debug-panel { margin-top: 20px; padding: 10px; background-color: #f8f9fa; border-radius: 4px; }
    .spinner { display: inline-block; width: 12px; height: 12px; border-radius: 50%; 
               border: 2px solid #ccc; border-top-color: #333; animation: spin 1s infinite linear; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  `]
})
export class FormStatesComponent implements OnInit {
  userForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
    this.monitorFormStateChanges();
  }
  
  createForm() {
    this.userForm = this.fb.group({
      username: ['', 
        [Validators.required, Validators.minLength(4)],
        [AsyncValidators.usernameTaken(this.userService)]
      ]
    });
  }
  
  monitorFormStateChanges() {
    // Monitor status changes to username field
    this.username.statusChanges.subscribe(status => {
      console.log(`Username status changed to: ${status}`);
    });
  }
  
  resetForm() {
    this.userForm.reset();
    console.log('Form has been reset');
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log('Form submitted:', this.userForm.value);
    }
  }
  
  // Helper method for template
  getStatusClass(statusType: string): string {
    switch (statusType) {
      case 'pristine':
        return this.username.pristine ? 'badge-primary' : 'badge-secondary';
      case 'touched':
        return this.username.touched ? 'badge-primary' : 'badge-secondary';
      case 'valid':
        return this.username.valid ? 'badge-primary' : 'badge-secondary';
      case 'pending':
        return this.username.pending ? 'badge-primary' : 'badge-secondary';
      default:
        return 'badge-secondary';
    }
  }
  
  // Getters
  get username() { return this.userForm.get('username')!; }
  get usernameErrors() { return this.username.errors; }
}
```

### Control Status Lifecycle:

1. **Initial state**: VALID (if no validators) or INVALID (if required)
2. **During async validation**: PENDING
3. **After validation**: VALID or INVALID
4. **When disabled**: DISABLED

### Best Practices:

- **Always handle the pending state** in your UI to provide visual feedback
- **Use a combination of states** (like `dirty && invalid`) to show errors at appropriate times
- **Avoid showing errors on pristine fields** to prevent overwhelming users
- **Monitor status changes** with `statusChanges` Observable for debugging

[Back to Top](#table-of-contents)

## setValue() vs patchValue()

When working with reactive forms and asynchronous validators, understanding the difference between `setValue()` and `patchValue()` methods is crucial for effectively manipulating form data.

### Key Differences:

| Feature | setValue() | patchValue() |
|---------|------------|--------------|
| **Completeness** | Requires values for all form controls | Allows partial updates |
| **Error Behavior** | Throws error if missing fields | Silent - updates only provided fields |
| **Validation Trigger** | Triggers all validators | Triggers all validators for updated fields |
| **Use Case** | Complete form reset or initialization | Partial form updates |

### setValue() Example:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { UserService } from './user.service';
import { AsyncValidators } from './async-validators';

@Component({
  selector: 'app-profile-form',
  template: `
    <form [formGroup]="profileForm">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
      </div>
      
      <div class="form-group">
        <label for="email">Email</label>
        <input type="email" id="email" formControlName="email">
      </div>
      
      <div class="form-group">
        <label for="phone">Phone</label>
        <input type="tel" id="phone" formControlName="phone">
      </div>
      
      <button type="button" (click)="loadUserProfile()">Load Profile</button>
      <button type="button" (click)="resetForm()">Reset</button>
    </form>
  `
})
export class ProfileFormComponent implements OnInit {
  profileForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.profileForm = this.fb.group({
      username: ['', 
        [Validators.required], 
        [AsyncValidators.usernameTaken(this.userService)]
      ],
      email: ['', 
        [Validators.required, Validators.email],
        [AsyncValidators.emailTaken(this.userService)]
      ],
      phone: ['', Validators.pattern(/^\d{10}$/)]
    });
  }
  
  // Using setValue() - must provide all form control values
  loadUserProfile() {
    // Example user data from API
    const userData = {
      username: 'johnsmith',
      email: 'john@example.com',
      phone: '1234567890'
    };
    
    try {
      // setValue requires all form control values to be provided
      this.profileForm.setValue(userData);
      console.log('Profile loaded successfully');
    } catch (error) {
      console.error('Error loading profile:', error);
    }
  }
  
  // Reset form with empty values
  resetForm() {
    // Reset all controls to their initial state
    this.profileForm.setValue({
      username: '',
      email: '',
      phone: ''
    });
  }
  
  // This would throw an error because not all fields are provided
  loadIncompleteProfile() {
    const partialData = {
      username: 'johnsmith',
      email: 'john@example.com'
      // phone is missing!
    };
    
    // This will throw an error
    // this.profileForm.setValue(partialData);
  }
}
```

### patchValue() Example:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { UserService } from './user.service';
import { AsyncValidators } from './async-validators';

@Component({
  selector: 'app-user-form',
  template: `
    <form [formGroup]="userForm">
      <div class="form-group">
        <label for="firstName">First Name</label>
        <input type="text" id="firstName" formControlName="firstName">
      </div>
      
      <div class="form-group">
        <label for="lastName">Last Name</label>
        <input type="text" id="lastName" formControlName="lastName">
      </div>
      
      <div class="form-group">
        <label for="email">Email</label>
        <input type="email" id="email" formControlName="email">
        
        <!-- Show pending state for async validation -->
        <div *ngIf="email.pending" class="text-info">
          Checking email availability...
        </div>
      </div>
      
      <div class="form-group">
        <label for="address">Address</label>
        <input type="text" id="address" formControlName="address">
      </div>
      
      <button type="button" (click)="updateUserEmail()">Update Email Only</button>
      <button type="button" (click)="updateUserName()">Update Name Only</button>
    </form>
  `
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.userForm = this.fb.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      email: ['', 
        [Validators.required, Validators.email],
        [AsyncValidators.emailTaken(this.userService)]
      ],
      address: ['']
    });
  }
  
  // Using patchValue() to update only specific fields
  updateUserEmail() {
    // Only update the email field
    this.userForm.patchValue({
      email: 'newemail@example.com'
    });
    
    // This will trigger async validation only for the email field
    console.log('Email updated and validation triggered');
  }
  
  // Update multiple fields but not all
  updateUserName() {
    this.userForm.patchValue({
      firstName: 'Jane',
      lastName: 'Doe'
    });
    
    console.log('Name updated');
  }
  
  // This would work fine with patchValue
  loadPartialUserData() {
    const userData = {
      firstName: 'John',
      lastName: 'Smith'
      // email and address are missing, but that's OK with patchValue
    };
    
    this.userForm.patchValue(userData);
  }
  
  get email() { return this.userForm.get('email')!; }
}
```

### Validation Behavior:

Both methods trigger validation, but with important differences:

- **setValue()**: Triggers validators for all form controls
- **patchValue()**: Triggers validators only for the updated controls

### When to Use Each:

- **Use setValue() when**:
  - Loading complete form data from a server
  - Resetting a form to a known state
  - Ensuring all fields are accounted for

- **Use patchValue() when**:
  - Updating specific field(s) without affecting others
  - Receiving partial data from an API
  - Performing incremental updates

### Best Practices:

- **Error handling**: Always wrap `setValue()` calls in try/catch blocks
- **Async validation**: Be mindful that both methods will trigger async validators on updated fields
- **Performance**: Use `patchValue()` for partial updates to avoid unnecessary validation
- **Form initialization**: Use `setValue()` for complete form initialization to ensure all fields are set

[Back to Top](#table-of-contents)

## valueChanges and statusChanges

Angular reactive forms provide two powerful observables for monitoring form changes: `valueChanges` and `statusChanges`. These are especially useful when working with asynchronous validators.

### Key Features:

- **valueChanges**: Observable that emits the current value whenever the control's value changes
- **statusChanges**: Observable that emits the current validation status whenever it changes ('VALID', 'INVALID', 'PENDING', 'DISABLED')

### Basic Usage:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Subscription } from 'rxjs';
import { UserService } from './user.service';
import { AsyncValidators } from './async-validators';

@Component({
  selector: 'app-form-changes',
  template: `
    <form [formGroup]="userForm">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <div *ngIf="username.pending">
          <span class="spinner"></span> Checking...
        </div>
        
        <div *ngIf="username.invalid && username.dirty">
          <div *ngIf="username.errors?.required">Username is required</div>
          <div *ngIf="username.errors?.minlength">Username must be at least 4 characters</div>
          <div *ngIf="username.errors?.usernameTaken">This username is already taken</div>
        </div>
      </div>
      
      <div class="log-panel">
        <h3>Form Status Log</h3>
        <div *ngFor="let log of formStatusLogs" class="log-entry">
          {{ log }}
        </div>
      </div>
    </form>
  `,
  styles: [`
    .log-panel { margin-top: 20px; padding: 10px; background-color: #f8f9fa; max-height: 200px; overflow-y: auto; }
    .log-entry { border-bottom: 1px solid #eee; padding: 4px 0; }
    .spinner { display: inline-block; width: 12px; height: 12px; border-radius: 50%; 
             border: 2px solid #ccc; border-top-color: #333; animation: spin 1s infinite linear; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  `]
})
export class FormChangesComponent implements OnInit, OnDestroy {
  userForm!: FormGroup;
  formStatusLogs: string[] = [];
  private valueChangesSubscription!: Subscription;
  private statusChangesSubscription!: Subscription;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
    this.subscribeToFormChanges();
  }
  
  ngOnDestroy() {
    // Clean up subscriptions to prevent memory leaks
    this.valueChangesSubscription.unsubscribe();
    this.statusChangesSubscription.unsubscribe();
  }
  
  createForm() {
    this.userForm = this.fb.group({
      username: ['', 
        [Validators.required, Validators.minLength(4)],
        [AsyncValidators.usernameTaken(this.userService)]
      ]
    });
  }
  
  subscribeToFormChanges() {
    // Monitor value changes
    this.valueChangesSubscription = this.username.valueChanges.subscribe(value => {
      const timestamp = new Date().toLocaleTimeString();
      this.formStatusLogs.unshift(`[${timestamp}] Value changed to: "${value}"`);
    });
    
    // Monitor status changes
    this.statusChangesSubscription = this.username.statusChanges.subscribe(status => {
      const timestamp = new Date().toLocaleTimeString();
      this.formStatusLogs.unshift(`[${timestamp}] Status changed to: "${status}"`);
    });
  }
  
  get username() { return this.userForm.get('username')!; }
}
```

### Advanced Usage with Debounce:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Subscription } from 'rxjs';
import { debounceTime, distinctUntilChanged, filter } from 'rxjs/operators';
import { UserService } from './user.service';
import { AsyncValidators } from './async-validators';

@Component({
  selector: 'app-advanced-form-changes',
  template: `
    <form [formGroup]="searchForm">
      <div class="form-group">
        <label for="searchTerm">Search</label>
        <input type="text" id="searchTerm" formControlName="searchTerm" 
               placeholder="Type to search...">
        
        <div *ngIf="isLoading" class="spinner-container">
          <span class="spinner"></span> Searching...
        </div>
      </div>
      
      <div class="results" *ngIf="searchResults.length > 0">
        <h3>Results</h3>
        <ul>
          <li *ngFor="let result of searchResults">{{ result }}</li>
        </ul>
      </div>
      
      <div *ngIf="searchTerm.value && !isLoading && searchResults.length === 0">
        No results found.
      </div>
    </form>
  `
})
export class AdvancedFormChangesComponent implements OnInit, OnDestroy {
  searchForm!: FormGroup;
  searchResults: string[] = [];
  isLoading = false;
  private searchSubscription!: Subscription;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService // Example service that performs searches
  ) {}
  
  ngOnInit() {
    this.createForm();
    this.setupSearchListener();
  }
  
  ngOnDestroy() {
    if (this.searchSubscription) {
      this.searchSubscription.unsubscribe();
    }
  }
  
  createForm() {
    this.searchForm = this.fb.group({
      searchTerm: ['', Validators.minLength(2)]
    });
  }
  
  setupSearchListener() {
    // Efficient search with debounce
    this.searchSubscription = this.searchTerm.valueChanges.pipe(
      // Wait 300ms after each keystroke before considering the term
      debounceTime(300),
      
      // Ignore if same as previous search term
      distinctUntilChanged(),
      
      // Ignore empty search terms
      filter(term => term.trim().length > 2)
    ).subscribe(searchTerm => {
      this.performSearch(searchTerm);
    });
  }
  
  performSearch(term: string) {
    this.isLoading = true;
    this.searchResults = [];
    
    // Simulate API call
    setTimeout(() => {
      // Generate dummy results based on search term
      if (term) {
        this.searchResults = [
          `Result 1 for "${term}"`,
          `Result 2 for "${term}"`,
          `Result 3 for "${term}"`
        ];
      }
      this.isLoading = false;
    }, 1000);
  }
  
  get searchTerm() { return this.searchForm.get('searchTerm')!; }
}
```

### Monitoring Form-wide Changes:

```typescript
// Monitor the entire form
this.form.valueChanges.subscribe(formValues => {
  console.log('Form values changed:', formValues);
});

this.form.statusChanges.subscribe(status => {
  console.log('Form status changed to:', status);
  
  // Update UI based on form status
  if (status === 'PENDING') {
    this.showLoadingIndicator();
  } else {
    this.hideLoadingIndicator();
  }
});
```

### Cross-field Validation with valueChanges:

```typescript
// Password confirmation example
this.passwordForm = this.fb.group({
  password: ['', [Validators.required, Validators.minLength(8)]],
  confirmPassword: ['', Validators.required]
});

// Use valueChanges to validate password confirmation
const password = this.passwordForm.get('password');
const confirmPassword = this.passwordForm.get('confirmPassword');

// Update validation when either password field changes
merge(
  password!.valueChanges,
  confirmPassword!.valueChanges
).pipe(
  debounceTime(300)
).subscribe(() => {
  if (password!.value !== confirmPassword!.value) {
    confirmPassword!.setErrors({ passwordMismatch: true });
  } else {
    // Only clear the passwordMismatch error, not other errors
    const errors = confirmPassword!.errors;
    if (errors) {
      delete errors['passwordMismatch'];
      confirmPassword!.setErrors(Object.keys(errors).length ? errors : null);
    }
  }
});
```

### Best Practices:

- **Always unsubscribe** in `ngOnDestroy` to prevent memory leaks
- **Use debounceTime** to limit the frequency of operations
- **Use distinctUntilChanged** to avoid duplicate operations
- **Combine with async validators** to create responsive and dynamic forms
- **Consider using the async pipe** in templates for simpler subscription management

[Back to Top](#table-of-contents)

## Performance Optimization

Optimizing performance when working with asynchronous validators is crucial, especially for applications with complex forms or those making frequent API calls.

### Key Optimization Strategies:

1. **Debounce User Input**
2. **Implement Caching**
3. **Cancel Previous Requests**
4. **Reduce Server Calls**
5. **Optimize Form Structure**

### 1. Debouncing User Input:

```typescript
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { debounceTime, distinctUntilChanged, switchMap, first, map } from 'rxjs/operators';
import { UserService } from './user.service';

export class OptimizedValidators {
  static usernameTaken(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value || control.value.length < 3) {
        return of(null); // Skip validation for short inputs
      }
      
      // Debounce to wait until user stops typing
      return control.valueChanges.pipe(
        debounceTime(500), // Wait 500ms after last keystroke
        distinctUntilChanged(), // Skip if value hasn't changed
        switchMap(value => userService.checkUsernameExists(value)),
        map(exists => exists ? { usernameTaken: true } : null),
        first() // Complete the observable after first emission
      );
    };
  }
}
```

### 2. Implementing Caching:

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { map, shareReplay, tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class CachedUserService {
  private readonly apiUrl = 'https://api.example.com/users';
  private readonly cache = new Map<string, boolean>();
  
  constructor(private http: HttpClient) {}
  
  checkUsernameExists(username: string): Observable<boolean> {
    // Return from cache if available
    if (this.cache.has(username)) {
      console.log(`Cache hit for username: ${username}`);
      return of(this.cache.get(username)!);
    }
    
    // Otherwise, call API and cache result
    console.log(`Cache miss for username: ${username}, calling API`);
    return this.http.get<any>(`${this.apiUrl}/check-username/${username}`).pipe(
      map(response => response.exists),
      tap(exists => {
        // Store in cache
        this.cache.set(username, exists);
        console.log(`Cached result for username: ${username}`);
      }),
      // Share the same response with multiple subscribers
      shareReplay(1)
    );
  }
  
  // Optional: Clear cache method
  clearCache(): void {
    this.cache.clear();
    console.log('Cache cleared');
  }
}
```

### 3. Canceling Previous Requests:

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of, Subject } from 'rxjs';
import { map, takeUntil, catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class OptimizedApiService {
  private readonly apiUrl = 'https://api.example.com/users';
  private cancelPrevious = new Subject<void>();
  
  constructor(private http: HttpClient) {}
  
  checkUsernameExists(username: string): Observable<boolean> {
    // Cancel any previous incomplete requests
    this.cancelPrevious.next();
    
    // Skip API call for empty or short usernames
    if (!username || username.length < 3) {
      return of(false);
    }
    
    return this.http.get<any>(`${this.apiUrl}/check-username/${username}`).pipe(
      // Cancel this request if a new one comes in
      takeUntil(this.cancelPrevious),
      map(response => response.exists),
      catchError(error => {
        console.error('Error checking username:', error);
        return of(false); // Default to false on error
      })
    );
  }
  
  // Clean up when component is destroyed
  cleanup(): void {
    this.cancelPrevious.next();
    this.cancelPrevious.complete();
  }
}
```

### 4. Optimized Form Component:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { OptimizedApiService } from './optimized-api.service';
import { OptimizedValidators } from './optimized-validators';

@Component({
  selector: 'app-optimized-form',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <!-- Only show pending for longer usernames -->
        <div *ngIf="username.pending && username.value.length >= 3" class="text-info">
          <span class="spinner-border spinner-border-sm"></span>
          Checking availability...
        </div>
        
        <div *ngIf="username.invalid && username.dirty" class="text-danger">
          <div *ngIf="username.errors?.required">Username is required</div>
          <div *ngIf="username.errors?.minlength">Username must be at least 3 characters</div>
          <div *ngIf="username.errors?.usernameTaken">This username is already taken</div>
        </div>
      </div>
      
      <button type="submit" [disabled]="userForm.invalid || userForm.pending">Submit</button>
    </form>
    
    <!-- Performance stats -->
    <div class="performance-stats">
      <p>API Calls: {{ apiCallCount }}</p>
      <p>Cache Hits: {{ cacheHitCount }}</p>
    </div>
  `
})
export class OptimizedFormComponent implements OnInit, OnDestroy {
  userForm!: FormGroup;
  apiCallCount = 0;
  cacheHitCount = 0;
  
  constructor(
    private fb: FormBuilder,
    private apiService: OptimizedApiService
  ) {}
  
  ngOnInit() {
    // Create form with optimized async validator
    this.userForm = this.fb.group({
      username: ['', 
        [
          Validators.required,
          Validators.minLength(3)
        ],
        [
          // Only run async validator if sync validators pass
          OptimizedValidators.usernameTaken(this.apiService)
        ]
      ]
    });
    
    // Track API calls for demonstration
    this.apiService.onApiCall.subscribe(() => this.apiCallCount++);
    this.apiService.onCacheHit.subscribe(() => this.cacheHitCount++);
  }
  
  ngOnDestroy() {
    // Clean up service resources
    this.apiService.cleanup();
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log('Form submitted:', this.userForm.value);
    }
  }
  
  get username() { return this.userForm.get('username')!; }
}
```

### 5. Lazy Loading Validators:

```typescript
import { Component, OnInit } from '@angular/core';
import { AbstractControl, AsyncValidatorFn, FormBuilder, FormGroup, ValidationErrors, Validators } from '@angular/forms';
import { Observable, of, timer } from 'rxjs';
import { switchMap, map } from 'rxjs/operators';
import { UserService } from './user.service';

@Component({
  selector: 'app-lazy-validators',
  template: `
    <form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
      <!-- Basic info fields -->
      <div formGroupName="basicInfo">
        <div class="form-group">
          <label for="name">Name</label>
          <input type="text" id="name" formControlName="name">
        </div>
        
        <div class="form-group">
          <label for="email">Email</label>
          <input type="email" id="email" formControlName="email">
        </div>
      </div>
      
      <!-- Show account fields only after basic info is complete -->
      <div *ngIf="showAccountSection" formGroupName="accountInfo">
        <h3>Account Information</h3>
        
        <div class="form-group">
          <label for="username">Username</label>
          <input type="text" id="username" formControlName="username">
          
          <div *ngIf="username?.pending" class="text-info">
            Checking availability...
          </div>
          
          <div *ngIf="username?.invalid && username?.dirty" class="text-danger">
            <div *ngIf="username?.errors?.usernameTaken">This username is already taken</div>
          </div>
        </div>
      </div>
      
      <button type="button" *ngIf="!showAccountSection" (click)="proceedToAccountInfo()">
        Next
      </button>
      
      <button type="submit" *ngIf="showAccountSection" [disabled]="registrationForm.invalid || registrationForm.pending">
        Register
      </button>
    </form>
  `
})
export class LazyValidatorsComponent implements OnInit {
  registrationForm!: FormGroup;
  showAccountSection = false;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {}
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      // First step with synchronous validators only
      basicInfo: this.fb.group({
        name: ['', [Validators.required]],
        email: ['', [Validators.required, Validators.email]]
      }),
      
      // Second step with async validators - these won't run until this group is interacted with
      accountInfo: this.fb.group({
        username: [''] // Async validator will be added later
      })
    });
  }
  
  // Proceed to second step and add async validators only when needed
  proceedToAccountInfo() {
    const basicInfo = this.registrationForm.get('basicInfo');
    
    if (basicInfo?.valid) {
      // Show the account section
      this.showAccountSection = true;
      
      // Add async validator to username field now that we need it
      const usernameControl = this.registrationForm.get('accountInfo.username');
      usernameControl?.setValidators([Validators.required, Validators.minLength(4)]);
      usernameControl?.setAsyncValidators(this.createUsernameValidator());
      usernameControl?.updateValueAndValidity();
    } else {
      // Mark all basic info fields as touched to show validation errors
      Object.keys(basicInfo?.controls || {}).forEach(key => {
        basicInfo?.get(key)?.markAsTouched();
      });
    }
  }
  
  createUsernameValidator(): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value || control.value.length < 4) {
        return of(null);
      }
      
      // Optimized API call with cache
      return timer(500).pipe(
        switchMap(() => this.userService.checkUsernameExists(control.value)),
        map(exists => exists ? { usernameTaken: true } : null)
      );
    };
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted:', this.registrationForm.value);
    }
  }
  
  get username() { return this.registrationForm.get('accountInfo.username'); }
}
```

### Performance Best Practices:

1. **Debounce input** to reduce API calls during typing
2. **Skip validation** for empty or invalid inputs
3. **Implement caching** for frequently checked values
4. **Cancel previous requests** when new ones are made
5. **Use optimistic UI updates** to improve perceived performance
6. **Lazy load validators** only when needed
7. **Monitor performance metrics** in production

[Back to Top](#table-of-contents)

## Error Handling

Proper error handling is crucial when working with asynchronous validators as they involve network requests that can fail for various reasons.

### Common Error Scenarios:

1. **Network failures**
2. **Server errors** (500 responses)
3. **Timeout issues**
4. **Backend validation inconsistencies**
5. **Authentication/authorization failures**

### Implementing Robust Error Handling:

```typescript
import { Injectable } from '@angular/core';
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { catchError, debounceTime, map, retry, switchMap, timeout } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class UserValidationService {
  private readonly apiUrl = 'https://api.example.com/users';
  
  constructor(private http: HttpClient) {}
  
  checkUsername(username: string): Observable<boolean> {
    return this.http.get<any>(`${this.apiUrl}/check-username/${username}`).pipe(
      // Timeout after 5 seconds
      timeout(5000),
      
      // Retry failed requests up to 2 times
      retry(2),
      
      // Extract the result
      map(response => response.exists),
      
      // Handle HTTP errors
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse): Observable<boolean> {
    // Log the error for debugging
    console.error('API Error:', error);
    
    if (error.status === 0) {
      // Client-side error (network issue)
      console.error('Network error. Please check your connection.');
    } else if (error.status >= 500) {
      // Server error
      console.error('Server error. Please try again later.');
    } else if (error.status === 401 || error.status === 403) {
      // Authentication/authorization error
      console.error('Authorization error. Please log in again.');
    } else if (error.status === 404) {
      // Not found
      console.error('Resource not found.');
    }
    
    // Return a safe default for validation - assume username is NOT taken on error
    // (better UX to allow the user to try to submit rather than blocking on error)
    return of(false);
  }
}

export class RobustAsyncValidators {
  static usernameValidator(service: UserValidationService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      const username = control.value;
      
      if (!username || username.length < 3) {
        return of(null);
      }
      
      return control.valueChanges.pipe(
        debounceTime(500),
        switchMap(() => service.checkUsername(username).pipe(
          map(exists => exists ? { usernameTaken: true } : null),
          catchError(() => {
            // Fall back to allowing the value on API error
            return of({ apiError: true });
          })
        ))
      );
    };
  }
}
```

### Displaying Validation Errors to Users:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { UserValidationService } from './user-validation.service';
import { RobustAsyncValidators } from './robust-async-validators';

@Component({
  selector: 'app-robust-form',
  template: `
    <form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <!-- Loading state -->
        <div *ngIf="username.pending && !hasConnectionError" class="text-info">
          <span class="spinner"></span> Checking username...
        </div>
        
        <!-- Connection error banner - global form level -->
        <div *ngIf="hasConnectionError" class="alert alert-warning">
          <strong>Connection issues detected!</strong> We couldn't verify if this username is available.
          You can continue, but you may need to choose another username later.
        </div>
        
        <!-- Validation errors -->
        <div *ngIf="username.invalid && username.dirty" class="text-danger">
          <div *ngIf="username.errors?.required">Username is required</div>
          <div *ngIf="username.errors?.minlength">Username must be at least 3 characters</div>
          <div *ngIf="username.errors?.usernameTaken">This username is already taken</div>
          <div *ngIf="username.errors?.apiError">
            Could not verify username availability. You may continue, but this username might be unavailable.
          </div>
        </div>
      </div>
      
      <button type="submit" [disabled]="isFormUnsubmittable()">Register</button>
      
      <!-- Try again button for network issues -->
      <button type="button" *ngIf="hasConnectionError" (click)="retryValidation()">
        Try again
      </button>
    </form>
  `
})
export class RobustFormComponent implements OnInit {
  registrationForm!: FormGroup;
  hasConnectionError = false;
  
  constructor(
    private fb: FormBuilder,
    private validationService: UserValidationService
  ) {}
  
  ngOnInit() {
    this.createForm();
    this.monitorConnectionErrors();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      username: ['', 
        [Validators.required, Validators.minLength(3)],
        [RobustAsyncValidators.usernameValidator(this.validationService)]
      ]
    });
  }
  
  monitorConnectionErrors() {
    this.username.statusChanges.subscribe(status => {
      // Check for apiError in validation errors
      this.hasConnectionError = this.username.errors?.apiError === true;
    });
  }
  
  retryValidation() {
    // Clear the error state
    this.hasConnectionError = false;
    
    // Re-trigger validation
    this.username.updateValueAndValidity();
  }
  
  isFormUnsubmittable(): boolean {
    // Disable submit button on normal validation failures
    if (this.registrationForm.invalid && !this.hasConnectionError) {
      return true;
    }
    
    // Disable button during pending validation
    if (this.registrationForm.pending) {
      return true;
    }
    
    return false;
  }
  
  onSubmit() {
    if (this.registrationForm.valid || (this.hasConnectionError && this.username.value)) {
      console.log('Form submitted with potential connection issues:', this.registrationForm.value);
      // In real app, handle this case specially on the server to re-validate
    }
  }
  
  get username() { return this.registrationForm.get('username')!; }
}
```

### Fallback Strategies:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import { catchError, switchMap, tap } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({
  selector: 'app-fallback-validation',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <!-- Form fields -->
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" formControlName="username">
        
        <!-- Loading indicator -->
        <div *ngIf="isValidating" class="text-info">