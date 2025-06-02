# Angular Forms Documentation

This documentation provides a comprehensive guide to working with forms in Angular, covering both Template-Driven and Reactive approaches with detailed examples and best practices.

## Table of Contents

| # | Topic |
|---|-------|
| **Forms in Angular** |
| 1 | [Introduction to Angular Forms](#introduction-to-angular-forms) |
| 2 | [Template-Driven Forms](#template-driven-forms) |
| 3 | [Reactive Forms](#reactive-forms) |
| 4 | [FormGroup and FormControl](#formgroup-and-formcontrol) |
| 5 | [FormBuilder](#formbuilder) |
| 6 | [FormArray for Dynamic Forms](#formarray-for-dynamic-forms) |
| **Form Validation** |
| 7 | [Built-in Validators](#built-in-validators) |
| 8 | [Custom Validators](#custom-validators) |
| 9 | [Asynchronous Validators](#asynchronous-validators) |
| **Form Controls and Events** |
| 10 | [Form Control Events](#form-control-events) |
| 11 | [setValue() vs patchValue()](#setvalue-vs-patchvalue) |
| 12 | [valueChanges and statusChanges](#valuechanges-and-statuschanges) |

## Introduction to Angular Forms

Angular provides two different approaches to handling user input through forms:

- **Template-driven forms**: Angular infers the form object model from the DOM
- **Reactive forms**: Form models are explicitly defined in the component class

Both approaches capture user input events, validate user input, create a form model, and track form control changes.

Key differences:

| Feature | Template-driven | Reactive |
|---------|----------------|----------|
| Form model setup | Implicit, created by directives | Explicit, created in component class |
| Data model | Mutable | Immutable |
| Form validation | Directives | Functions |
| Testing | End-to-end testing | Unit testing |
| Scalability | Simple forms | Complex forms |
| Predictability | Less predictable | More predictable |

**To use either approach, you need to import the appropriate module:**

```typescript
// app.module.ts
import { FormsModule } from '@angular/forms';    // For Template-driven forms
import { ReactiveFormsModule } from '@angular/forms'; // For Reactive forms

@NgModule({
  imports: [
    // ... other imports
    FormsModule,        // Import for template-driven forms
    ReactiveFormsModule // Import for reactive forms
  ],
  // ... declarations, providers, etc.
})
export class AppModule { }
```

[Back to Top](#table-of-contents)

## Template-Driven Forms

Template-driven forms rely on directives in the template to create and manipulate the underlying form model. They're useful for adding simple forms to an app with minimal code.

Key features:
- **Easy to add** to an app
- Form controls created **implicitly** with directives like `ngModel`
- **Less code** in the component class
- **Two-way data binding** with `[(ngModel)]`
- Automatic validation based on HTML5 attributes

Basic example of a template-driven form:

```typescript
// user-profile.component.ts
import { Component } from '@angular/core';

export class User {
  constructor(
    public name: string = '',
    public email: string = '',
    public phone: string = ''
  ) {}
}

@Component({
  selector: 'app-user-profile',
  templateUrl: './user-profile.component.html'
})
export class UserProfileComponent {
  user = new User();
  
  onSubmit() {
    // Form handling logic
    console.log('Form submitted with:', this.user);
  }
}
```

```html
<!-- user-profile.component.html -->
<form #userForm="ngForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="name">Name</label>
    <input type="text" 
           class="form-control" 
           id="name" 
           name="name"
           [(ngModel)]="user.name" 
           required>
    <!-- Display validation message -->
    <div *ngIf="userForm.controls['name']?.invalid && 
               (userForm.controls['name']?.dirty || userForm.controls['name']?.touched)" 
         class="alert alert-danger">
      Name is required
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           name="email"
           [(ngModel)]="user.email" 
           required 
           email>
    <!-- Display validation messages -->
    <div *ngIf="userForm.controls['email']?.errors?.['required'] && 
               (userForm.controls['email']?.dirty || userForm.controls['email']?.touched)" 
         class="alert alert-danger">
      Email is required
    </div>
    <div *ngIf="userForm.controls['email']?.errors?.['email'] && 
               (userForm.controls['email']?.dirty || userForm.controls['email']?.touched)" 
         class="alert alert-danger">
      Please enter a valid email
    </div>
  </div>
  
  <div class="form-group">
    <label for="phone">Phone</label>
    <input type="tel" 
           class="form-control" 
           id="phone" 
           name="phone"
           [(ngModel)]="user.phone">
  </div>
  
  <button type="submit" 
          class="btn btn-primary"
          [disabled]="userForm.invalid">Submit</button>
</form>

<!-- Display form values for debugging -->
<div class="form-debug" *ngIf="userForm">
  <h3>Form Value</h3>
  <pre>{{ userForm.value | json }}</pre>
  <h3>Form Status</h3>
  <pre>Valid? {{ userForm.valid }}</pre>
  <pre>Touched? {{ userForm.touched }}</pre>
  <pre>Submitted? {{ userForm.submitted }}</pre>
</div>
```

**How it works:**
- `#userForm="ngForm"` creates a reference to the form
- `[(ngModel)]="user.name"` sets up two-way data binding
- `required` and `email` attributes provide validation
- `ngSubmit` directive handles form submission
- Form state is tracked with properties like `valid`, `touched`, etc.

**Best Practices for Template-Driven Forms:**
- Use for simple forms with straightforward validation
- Add proper validation messages for each possible error
- Leverage NgForm to track the state of the entire form
- Utilize NgModel for two-way data binding
- Structure your HTML with proper labeling and accessibility

[Back to Top](#table-of-contents)

## Reactive Forms

Reactive forms provide a model-driven approach to handling form inputs where form controls are explicitly created in the component class. Reactive forms are more robust, scalable, reusable, and testable compared to template-driven forms.

Key features:
- **Explicit form control creation** in the component class
- **More predictable** because the form structure is defined in code
- **Easier unit testing** compared to template-driven forms
- **Better for complex forms** with dynamic controls
- **Synchronous access** to the form model and form control values

Basic example of a reactive form:

```typescript
// reactive-profile.component.ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-reactive-profile',
  templateUrl: './reactive-profile.component.html'
})
export class ReactiveProfileComponent implements OnInit {
  profileForm!: FormGroup;
  
  ngOnInit() {
    this.initForm();
  }
  
  initForm() {
    this.profileForm = new FormGroup({
      name: new FormControl('', [Validators.required]),
      email: new FormControl('', [Validators.required, Validators.email]),
      phone: new FormControl('')
    });
  }
  
  onSubmit() {
    if (this.profileForm.valid) {
      console.log('Form submitted with:', this.profileForm.value);
      // Handle form submission logic
    } else {
      // Mark all fields as touched to trigger validation messages
      this.markFormGroupTouched(this.profileForm);
    }
  }
  
  // Helper method to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      // If control is another FormGroup, recursively mark its controls
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Convenience getters for form controls
  get name() { return this.profileForm.get('name'); }
  get email() { return this.profileForm.get('email'); }
  get phone() { return this.profileForm.get('phone'); }
}
```

```html
<!-- reactive-profile.component.html -->
<form [formGroup]="profileForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="name">Name</label>
    <input type="text" 
           class="form-control" 
           id="name" 
           formControlName="name">
    <!-- Display validation message -->
    <div *ngIf="name?.invalid && (name?.dirty || name?.touched)" 
         class="alert alert-danger">
      <div *ngIf="name?.errors?.['required']">Name is required</div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           formControlName="email">
    <!-- Display validation messages -->
    <div *ngIf="email?.invalid && (email?.dirty || email?.touched)" 
         class="alert alert-danger">
      <div *ngIf="email?.errors?.['required']">Email is required</div>
      <div *ngIf="email?.errors?.['email']">Please enter a valid email</div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="phone">Phone</label>
    <input type="tel" 
           class="form-control" 
           id="phone" 
           formControlName="phone">
  </div>
  
  <button type="submit" 
          class="btn btn-primary"
          [disabled]="profileForm.invalid">Submit</button>
</form>

<!-- Display form values for debugging -->
<div class="form-debug">
  <h3>Form Value</h3>
  <pre>{{ profileForm.value | json }}</pre>
  <h3>Form Status</h3>
  <pre>Valid? {{ profileForm.valid }}</pre>
  <pre>Touched? {{ profileForm.touched }}</pre>
  <pre>Dirty? {{ profileForm.dirty }}</pre>
</div>
```

**How it works:**
- Form structure is defined in the component using `FormGroup` and `FormControl`
- Template binds to form model using `[formGroup]` and `formControlName`
- Validation is set up in the component using `Validators`
- Form state can be checked programmatically in the component

[Back to Top](#table-of-contents)

## FormGroup and FormControl

In reactive forms, `FormGroup` and `FormControl` are the basic building blocks used to define the form model.

**FormControl**:
- Represents a single input field
- Tracks value and validation state of an individual form control
- Contains properties like `value`, `errors`, `valid`, `touched`

**FormGroup**:
- Container for a collection of FormControl instances
- Tracks the value and validity state of a group of FormControl instances
- Can contain nested FormGroups for complex forms

Basic usage example:

```typescript
// user-form.component.ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  templateUrl: './user-form.component.html'
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;
  
  ngOnInit() {
    // Initialize the form with FormGroup and FormControl
    this.userForm = new FormGroup({
      firstName: new FormControl('', Validators.required),
      lastName: new FormControl('', Validators.required),
      // Nested FormGroup for address
      address: new FormGroup({
        street: new FormControl(''),
        city: new FormControl(''),
        state: new FormControl(''),
        zip: new FormControl('')
      }),
      // Form controls with multiple validators
      email: new FormControl('', [
        Validators.required,
        Validators.email
      ]),
      // Form control with initial value
      newsletterSubscription: new FormControl(true)
    });
  }
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log('Form submitted with:', this.userForm.value);
    }
  }
  
  // Reset the form with specific values
  resetForm() {
    this.userForm.reset({
      firstName: '',
      lastName: '',
      address: {
        street: '',
        city: '',
        state: '',
        zip: ''
      },
      email: '',
      newsletterSubscription: false
    });
  }
  
  // Convenience getters for form controls
  get firstName() { return this.userForm.get('firstName'); }
  get lastName() { return this.userForm.get('lastName'); }
  get email() { return this.userForm.get('email'); }
  // Access nested form group
  get address() { return this.userForm.get('address') as FormGroup; }
  // Access control inside nested form group
  get city() { return this.address?.get('city'); }
}
```

```html
<!-- user-form.component.html -->
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
  <!-- Basic form controls -->
  <div class="form-row">
    <div class="form-group col-md-6">
      <label for="firstName">First Name</label>
      <input type="text" 
             class="form-control" 
             id="firstName" 
             formControlName="firstName">
      <div *ngIf="firstName?.invalid && (firstName?.dirty || firstName?.touched)" 
           class="alert alert-danger">
        First Name is required
      </div>
    </div>
    
    <div class="form-group col-md-6">
      <label for="lastName">Last Name</label>
      <input type="text" 
             class="form-control" 
             id="lastName" 
             formControlName="lastName">
      <div *ngIf="lastName?.invalid && (lastName?.dirty || lastName?.touched)" 
           class="alert alert-danger">
        Last Name is required
      </div>
    </div>
  </div>
  
  <!-- Nested form group -->
  <div formGroupName="address">
    <h3>Address</h3>
    
    <div class="form-group">
      <label for="street">Street</label>
      <input type="text" 
             class="form-control" 
             id="street" 
             formControlName="street">
    </div>
    
    <div class="form-row">
      <div class="form-group col-md-6">
        <label for="city">City</label>
        <input type="text" 
               class="form-control" 
               id="city" 
               formControlName="city">
      </div>
      
      <div class="form-group col-md-4">
        <label for="state">State</label>
        <input type="text" 
               class="form-control" 
               id="state" 
               formControlName="state">
      </div>
      
      <div class="form-group col-md-2">
        <label for="zip">Zip</label>
        <input type="text" 
               class="form-control" 
               id="zip" 
               formControlName="zip">
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           formControlName="email">
    <div *ngIf="email?.invalid && (email?.dirty || email?.touched)" 
         class="alert alert-danger">
      <div *ngIf="email?.errors?.['required']">Email is required</div>
      <div *ngIf="email?.errors?.['email']">Please enter a valid email</div>
    </div>
  </div>
  
  <div class="form-group">
    <div class="form-check">
      <input class="form-check-input" 
             type="checkbox" 
             id="newsletterSubscription" 
             formControlName="newsletterSubscription">
      <label class="form-check-label" for="newsletterSubscription">
        Subscribe to newsletter
      </label>
    </div>
  </div>
  
  <button type="submit" 
          class="btn btn-primary"
          [disabled]="userForm.invalid">Submit</button>
  <button type="button" 
          class="btn btn-secondary ml-2"
          (click)="resetForm()">Reset</button>
</form>

<!-- Form debugging section -->
<div class="form-debug mt-4">
  <h3>Form Value</h3>
  <pre>{{ userForm.value | json }}</pre>
  <h3>Form Status</h3>
  <pre>Valid: {{ userForm.valid }}</pre>
</div>
```

**Key operations with FormGroup and FormControl:**

1. **Creating a FormControl**:
```typescript
// Basic FormControl
const nameControl = new FormControl(''); // Empty initial value

// With initial value
const ageControl = new FormControl(25);

// With validators
const emailControl = new FormControl('user@example.com', [
  Validators.required,
  Validators.email
]);

// With disabled state
const disabledControl = new FormControl({ 
  value: 'Disabled value', 
  disabled: true 
});
```

2. **Creating a FormGroup**:
```typescript
const userForm = new FormGroup({
  firstName: new FormControl(''),
  lastName: new FormControl(''),
  email: new FormControl('')
});

// Nested FormGroups
const registrationForm = new FormGroup({
  userData: new FormGroup({
    username: new FormControl(''),
    password: new FormControl('')
  }),
  contactInfo: new FormGroup({
    email: new FormControl(''),
    phone: new FormControl('')
  })
});
```

3. **Accessing form controls**:
```typescript
// Get a control directly from FormGroup
const emailControl = userForm.get('email');

// Get control from nested FormGroup
const usernameControl = registrationForm.get('userData.username');
// Or
const userDataGroup = registrationForm.get('userData') as FormGroup;
const usernameControl = userDataGroup.get('username');

// Using convenience getters (recommended)
get email() { return userForm.get('email'); }
get username() { return registrationForm.get('userData.username'); }
```

**Best Practices for FormGroup and FormControl:**
- Create convenience getter methods for commonly accessed form controls
- Use strongly typed forms with interfaces for better type safety
- Structure complex forms with nested FormGroups for logical organization
- Initialize forms in ngOnInit rather than in the constructor
- Add validation error messages for all required fields

[Back to Top](#table-of-contents)

## FormBuilder

The `FormBuilder` service provides syntactic sugar that shortens the process of creating complex forms. It's a helper service that simplifies form creation compared to directly using `FormGroup` and `FormControl` constructors.

Key features:
- **Simplified syntax** for creating form controls, groups and arrays
- **Reduces boilerplate code** for complex forms
- Part of the `ReactiveFormsModule`

Example using FormBuilder:

```typescript
// form-builder-example.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-form-builder-example',
  templateUrl: './form-builder-example.component.html'
})
export class FormBuilderExampleComponent implements OnInit {
  registrationForm!: FormGroup;
  
  // Inject FormBuilder service
  constructor(private fb: FormBuilder) { }
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    // Using FormBuilder to create the form
    this.registrationForm = this.fb.group({
      // Control with initial value and validators
      username: ['', [Validators.required, Validators.minLength(4)]],
      email: ['', [Validators.required, Validators.email]],
      // Nested form group
      contactInfo: this.fb.group({
        phone: [''],
        address: ['']
      }),
      // Control with initial value
      newsletter: [true],
      // FormArray for skills
      skills: this.fb.array([
        this.fb.control('')
      ])
    });
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted:', this.registrationForm.value);
    } else {
      this.markFormGroupTouched(this.registrationForm);
    }
  }
  
  // Helper method to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Convenience getters
  get username() { return this.registrationForm.get('username'); }
  get email() { return this.registrationForm.get('email'); }
  get contactInfo() { return this.registrationForm.get('contactInfo') as FormGroup; }
  get phone() { return this.contactInfo.get('phone'); }
}
```

```html
<!-- form-builder-example.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="username">Username</label>
    <input type="text" 
           class="form-control" 
           id="username" 
           formControlName="username">
    <div *ngIf="username?.invalid && (username?.dirty || username?.touched)" 
         class="alert alert-danger">
      <div *ngIf="username?.errors?.['required']">Username is required</div>
      <div *ngIf="username?.errors?.['minlength']">
        Username must be at least 4 characters long
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           formControlName="email">
    <div *ngIf="email?.invalid && (email?.dirty || email?.touched)" 
         class="alert alert-danger">
      <div *ngIf="email?.errors?.['required']">Email is required</div>
      <div *ngIf="email?.errors?.['email']">Please enter a valid email</div>
    </div>
  </div>
  
  <!-- Nested form group -->
  <div formGroupName="contactInfo">
    <h3>Contact Information</h3>
    
    <div class="form-group">
      <label for="phone">Phone</label>
      <input type="tel" 
             class="form-control" 
             id="phone" 
             formControlName="phone">
    </div>
    
    <div class="form-group">
      <label for="address">Address</label>
      <textarea class="form-control" 
                id="address" 
                formControlName="address" 
                rows="3"></textarea>
    </div>
  </div>
  
  <div class="form-group form-check">
    <input type="checkbox" 
           class="form-check-input" 
           id="newsletter" 
           formControlName="newsletter">
    <label class="form-check-label" for="newsletter">
      Subscribe to newsletter
    </label>
  </div>
  
  <button type="submit" 
          class="btn btn-primary"
          [disabled]="registrationForm.invalid">Register</button>
</form>

<!-- Form debugging -->
<div class="form-debug mt-4">
  <h3>Form Value</h3>
  <pre>{{ registrationForm.value | json }}</pre>
  <h3>Form Valid</h3>
  <pre>{{ registrationForm.valid }}</pre>
</div>
```

**FormBuilder Methods:**

1. **control()**: Creates a new FormControl
```typescript
// FormBuilder syntax
const nameControl = fb.control('John', Validators.required);

// Equivalent to 
const nameControl = new FormControl('John', Validators.required);
```

2. **group()**: Creates a new FormGroup
```typescript
// FormBuilder syntax
const addressForm = fb.group({
  street: [''],
  city: [''],
  state: [''],
  zip: ['']
});

// Equivalent to
const addressForm = new FormGroup({
  street: new FormControl(''),
  city: new FormControl(''),
  state: new FormControl(''),
  zip: new FormControl('')
});
```

3. **array()**: Creates a new FormArray
```typescript
// FormBuilder syntax
const skillsArray = fb.array([
  fb.control('HTML'),
  fb.control('CSS'),
  fb.control('JavaScript')
]);

// Equivalent to
const skillsArray = new FormArray([
  new FormControl('HTML'),
  new FormControl('CSS'),
  new FormControl('JavaScript')
]);
```

**Comparison with direct instantiation:**

| FormBuilder | Direct Instantiation |
|-------------|----------------------|
| `this.fb.control('John')` | `new FormControl('John')` |
| `this.fb.group({ name: ['John'] })` | `new FormGroup({ name: new FormControl('John') })` |
| `this.fb.array([this.fb.control('')])` | `new FormArray([new FormControl('')])` |

**Best Practices for FormBuilder:**
- Use FormBuilder for complex forms to reduce boilerplate code
- Break down large forms into smaller, reusable form groups
- Separate form creation logic into dedicated methods
- Use FormBuilder.group() with objects for clear initial values
- Maintain a clear structure for nested form groups

[Back to Top](#table-of-contents)

## FormArray for Dynamic Forms

`FormArray` is a way to manage an arbitrary number of form controls. It's particularly useful for dynamic forms where the number of controls can change at runtime, such as adding or removing items from a list.

Key features:
- **Dynamically add or remove form controls** at runtime
- Manages a collection of controls as an array
- Each control can be a FormControl, FormGroup, or even another FormArray
- Useful for handling lists, multiple entries, and repeated sections

Example of a dynamic form with FormArray:

```typescript
// dynamic-form.component.ts
import { Component, OnInit } from '@angular/core';
import { FormArray, FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-dynamic-form',
  templateUrl: './dynamic-form.component.html'
})
export class DynamicFormComponent implements OnInit {
  skillsForm!: FormGroup;
  
  constructor(private fb: FormBuilder) { }
  
  ngOnInit() {
    this.skillsForm = this.fb.group({
      name: ['', Validators.required],
      skills: this.fb.array([
        this.createSkillFormGroup()
      ])
    });
  }
  
  // Helper method to create a skill FormGroup
  createSkillFormGroup(): FormGroup {
    return this.fb.group({
      skillName: ['', Validators.required],
      proficiency: ['beginner', Validators.required],
      yearsExperience: [0, [Validators.required, Validators.min(0)]]
    });
  }
  
  // Getter for the skills FormArray
  get skills(): FormArray {
    return this.skillsForm.get('skills') as FormArray;
  }
  
  // Method to add a new skill to the FormArray
  addSkill() {
    this.skills.push(this.createSkillFormGroup());
  }
  
  // Method to remove a skill from the FormArray
  removeSkill(index: number) {
    this.skills.removeAt(index);
  }
  
  // Method to clear all skills
  clearSkills() {
    this.skills.clear();
  }
  
  // Form submission handler
  onSubmit() {
    if (this.skillsForm.valid) {
      console.log('Form submitted with:', this.skillsForm.value);
    } else {
      // Mark all fields as touched to show validation errors
      this.markFormGroupTouched(this.skillsForm);
    }
  }
  
  // Helper to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup | FormArray) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup || control instanceof FormArray) {
        this.markFormGroupTouched(control);
      }
    });
  }
}
```

```html
<!-- dynamic-form.component.html -->
<form [formGroup]="skillsForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="name">Name</label>
    <input type="text" 
           class="form-control" 
           id="name" 
           formControlName="name">
    <div *ngIf="skillsForm.get('name')?.invalid && 
               (skillsForm.get('name')?.dirty || skillsForm.get('name')?.touched)" 
         class="alert alert-danger">
      Name is required
    </div>
  </div>
  
  <h3>Skills</h3>
  
  <div formArrayName="skills">
    <!-- Loop through each skill FormGroup in the FormArray -->
    <div *ngFor="let skillGroup of skills.controls; let i = index">
      <!-- Set the formGroupName to the current index -->
      <div [formGroupName]="i" class="skill-group p-3 mb-3 border rounded">
        <h4>Skill #{{ i + 1 }}</h4>
        
        <div class="form-group">
          <label [for]="'skillName' + i">Skill Name</label>
          <input type="text" 
                 class="form-control" 
                 [id]="'skillName' + i" 
                 formControlName="skillName">
          <div *ngIf="skillGroup.get('skillName')?.invalid && 
                     (skillGroup.get('skillName')?.dirty || skillGroup.get('skillName')?.touched)" 
               class="alert alert-danger">
            Skill name is required
          </div>
        </div>
        
                  <div class="form-group">
          <label [for]="'proficiency' + i">Proficiency</label>
          <select class="form-control" 
                  [id]="'proficiency' + i" 
                  formControlName="proficiency">
            <option value="beginner">Beginner</option>
            <option value="intermediate">Intermediate</option>
            <option value="advanced">Advanced</option>
            <option value="expert">Expert</option>
          </select>
        </div>
        
        <div class="form-group">
          <label [for]="'yearsExperience' + i">Years of Experience</label>
          <input type="number" 
                 class="form-control" 
                 [id]="'yearsExperience' + i" 
                 formControlName="yearsExperience" 
                 min="0">
          <div *ngIf="skillGroup.get('yearsExperience')?.invalid && 
                     (skillGroup.get('yearsExperience')?.dirty || skillGroup.get('yearsExperience')?.touched)" 
               class="alert alert-danger">
            <div *ngIf="skillGroup.get('yearsExperience')?.errors?.['required']">Years of experience is required</div>
            <div *ngIf="skillGroup.get('yearsExperience')?.errors?.['min']">Years must be 0 or more</div>
          </div>
        </div>
        
        <button type="button" 
                class="btn btn-danger" 
                (click)="removeSkill(i)">Remove Skill</button>
      </div>
    </div>
  </div>
  
  <div class="button-group mt-3 mb-3">
    <button type="button" 
            class="btn btn-secondary mr-2" 
            (click)="addSkill()">Add Skill</button>
    <button type="button" 
            class="btn btn-warning" 
            (click)="clearSkills()"
            [disabled]="skills.length === 0">Clear All Skills</button>
  </div>
  
  <button type="submit" 
          class="btn btn-primary" 
          [disabled]="skillsForm.invalid">Submit</button>
</form>

<!-- Form debug output -->
<div class="form-debug mt-4">
  <h3>Form Value</h3>
  <pre>{{ skillsForm.value | json }}</pre>
  <h3>Form Valid</h3>
  <pre>{{ skillsForm.valid }}</pre>
  <h3>Number of Skills</h3>
  <pre>{{ skills.length }}</pre>
</div>
```

**Sample output of the form:**
```json
{
  "name": "John Smith",
  "skills": [
    {
      "skillName": "Angular",
      "proficiency": "advanced",
      "yearsExperience": 3
    },
    {
      "skillName": "TypeScript",
      "proficiency": "intermediate",
      "yearsExperience": 2
    },
    {
      "skillName": "HTML/CSS",
      "proficiency": "expert",
      "yearsExperience": 5
    }
  ]
}
```

**Key FormArray Operations:**

1. **Creating a FormArray**:
```typescript
// Create FormArray with empty array
const skillsArray = new FormArray([]);

// Create FormArray with initial controls
const skillsArray = new FormArray([
  new FormControl('JavaScript'),
  new FormControl('TypeScript'),
  new FormControl('Angular')
]);

// Using FormBuilder
const skillsArray = this.fb.array([
  this.fb.control('JavaScript'),
  this.fb.control('TypeScript'),
  this.fb.control('Angular')
]);
```

2. **Adding controls to a FormArray**:
```typescript
// Add a simple control
skillsArray.push(new FormControl('React'));

// Add a FormGroup
skillsArray.push(new FormGroup({
  name: new FormControl('React'),
  level: new FormControl('Beginner')
}));

// Insert at specific position
skillsArray.insert(1, new FormControl('Vue.js'));
```

3. **Removing controls from a FormArray**:
```typescript
// Remove control at specific index
skillsArray.removeAt(2);

// Remove all controls
skillsArray.clear();
```

4. **Accessing FormArray controls**:
```typescript
// Get control at index
const firstSkill = skillsArray.at(0);

// Get all controls
const allControls = skillsArray.controls;

// Get FormArray length
const numberOfSkills = skillsArray.length;
```

**Working with FormArray in Templates:**

```html
<div formArrayName="skills">
  <div *ngFor="let skill of skillsArray.controls; let i = index">
    <!-- For FormControl -->
    <input [formControlName]="i">
    
    <!-- For FormGroup -->
    <div [formGroupName]="i">
      <input formControlName="name">
      <input formControlName="level">
    </div>
  </div>
</div>
```

**Best Practices for FormArray:**
- Use a helper method to create form groups for consistency
- Implement proper validation for dynamically added controls
- Provide user-friendly ways to add, remove, and reorder items
- Handle empty arrays appropriately (e.g., disable "Clear All" button when empty)
- Use descriptive names for array items (e.g., "Skill #1" rather than "Item #1")
- Implement proper UI feedback for array operations

[Back to Top](#table-of-contents)

## Built-in Validators

Angular provides a set of built-in validators that can be used with both template-driven and reactive forms. These validators help ensure that form data meets specific criteria before submission.

Key built-in validators:
- **required**: Field must have a value
- **minLength/maxLength**: Field must meet length requirements
- **min/max**: Numeric field must be within range
- **email**: Field must contain a valid email format
- **pattern**: Field must match a regular expression pattern

Example of using built-in validators in reactive forms:

```typescript
// validators-example.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-validators-example',
  templateUrl: './validators-example.component.html'
})
export class ValidatorsExampleComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(private fb: FormBuilder) { }
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      // Required validator
      username: ['', [
        Validators.required,
        Validators.minLength(4),
        Validators.maxLength(20)
      ]],
      
      // Email validator
      email: ['', [
        Validators.required,
        Validators.email
      ]],
      
      // Pattern validator (alphanumeric with special characters)
      password: ['', [
        Validators.required,
        Validators.minLength(8),
        // Password must contain at least one number, one uppercase, one lowercase and one special character
        Validators.pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/)
      ]],
      
      // Number validators
      age: [null, [
        Validators.required,
        Validators.min(18),
        Validators.max(120)
      ]],
      
      // Pattern for phone number (simple example)
      phone: ['', [
        Validators.pattern(/^\d{10}$/) // 10-digit format
      ]]
    });
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted with:', this.registrationForm.value);
    } else {
      this.markFormGroupTouched(this.registrationForm);
    }
  }
  
  // Helper method to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Convenience getters for form controls
  get username() { return this.registrationForm.get('username'); }
  get email() { return this.registrationForm.get('email'); }
  get password() { return this.registrationForm.get('password'); }
  get age() { return this.registrationForm.get('age'); }
  get phone() { return this.registrationForm.get('phone'); }
}
```

```html
<!-- validators-example.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="username">Username</label>
    <input type="text" 
           class="form-control" 
           id="username" 
           formControlName="username">
    <div *ngIf="username?.invalid && (username?.dirty || username?.touched)" 
         class="alert alert-danger">
      <div *ngIf="username?.errors?.['required']">Username is required</div>
      <div *ngIf="username?.errors?.['minlength']">
        Username must be at least {{ username?.errors?.['minlength'].requiredLength }} characters
      </div>
      <div *ngIf="username?.errors?.['maxlength']">
        Username cannot exceed {{ username?.errors?.['maxlength'].requiredLength }} characters
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           formControlName="email">
    <div *ngIf="email?.invalid && (email?.dirty || email?.touched)" 
         class="alert alert-danger">
      <div *ngIf="email?.errors?.['required']">Email is required</div>
      <div *ngIf="email?.errors?.['email']">Please enter a valid email</div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" 
           class="form-control" 
           id="password" 
           formControlName="password">
    <div *ngIf="password?.invalid && (password?.dirty || password?.touched)" 
         class="alert alert-danger">
      <div *ngIf="password?.errors?.['required']">Password is required</div>
      <div *ngIf="password?.errors?.['minlength']">
        Password must be at least 8 characters
      </div>
      <div *ngIf="password?.errors?.['pattern']">
        Password must contain at least one uppercase letter, one lowercase letter, 
        one number, and one special character
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="age">Age</label>
    <input type="number" 
           class="form-control" 
           id="age" 
           formControlName="age">
    <div *ngIf="age?.invalid && (age?.dirty || age?.touched)" 
         class="alert alert-danger">
      <div *ngIf="age?.errors?.['required']">Age is required</div>
      <div *ngIf="age?.errors?.['min']">
        Age must be at least {{ age?.errors?.['min'].min }} years
      </div>
      <div *ngIf="age?.errors?.['max']">
        Age cannot exceed {{ age?.errors?.['max'].max }} years
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="phone">Phone (10 digits)</label>
    <input type="tel" 
           class="form-control" 
           id="phone" 
           formControlName="phone">
    <div *ngIf="phone?.invalid && (phone?.dirty || phone?.touched)" 
         class="alert alert-danger">
      <div *ngIf="phone?.errors?.['pattern']">
        Please enter a valid 10-digit phone number
      </div>
    </div>
  </div>
  
  <button type="submit" 
          class="btn btn-primary" 
          [disabled]="registrationForm.invalid">Register</button>
</form>

<!-- Form debugging section -->
<div class="form-debug mt-4">
  <h3>Form Value</h3>
  <pre>{{ registrationForm.value | json }}</pre>
  <h3>Form Status</h3>
  <pre>Valid: {{ registrationForm.valid }}</pre>
</div>
```

**Built-in Validators Reference:**

| Validator | Description | Example |
|-----------|-------------|---------|
| `required` | Field value cannot be null, undefined, or empty string | `Validators.required` |
| `email` | Field must contain a valid email format | `Validators.email` |
| `minLength` | Field must have a minimum number of characters | `Validators.minLength(8)` |
| `maxLength` | Field must not exceed maximum number of characters | `Validators.maxLength(20)` |
| `min` | Numeric field must be greater than or equal to specified value | `Validators.min(18)` |
| `max` | Numeric field must be less than or equal to specified value | `Validators.max(120)` |
| `pattern` | Field value must match the specified regular expression | `Validators.pattern(/^\d{5}$/)` |
| `nullValidator` | Does nothing but can be used as a placeholder | `Validators.nullValidator` |
| `requiredTrue` | Field value must be true (useful for checkboxes) | `Validators.requiredTrue` |

**Best Practices for Built-in Validators:**
- Provide clear, specific error messages for each validation error
- Combine multiple validators when appropriate (e.g., required + minLength)
- Only show validation errors after user interaction (touched or dirty)
- Use appropriate HTML5 input types to enhance built-in validation
- Avoid excessive validation that might frustrate users
- Test validation with edge cases to ensure robustness

[Back to Top](#table-of-contents)

## Custom Validators

When built-in validators don't meet your specific requirements, you can create custom validators to implement custom validation logic for your forms.

Custom validators are functions that check a control's value against specific criteria and return a validation error object if the check fails, or null if validation passes.

Key concepts:
- **Custom validators** are functions that receive a `FormControl` instance
- They return `null` if validation passes, or an error object if validation fails
- Can be used with both reactive and template-driven forms
- Multiple custom validators can be combined with built-in validators

Example of custom validators in a reactive form:

```typescript
// custom-validators.component.ts
import { Component, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup, ValidationErrors, Validators } from '@angular/forms';

// Custom validator functions
export class CustomValidators {
  // Validator to check if field contains spaces
  static noWhitespace(control: AbstractControl): ValidationErrors | null {
    // Check if control has a value and if it contains any whitespace
    if (control.value && control.value.indexOf(' ') !== -1) {
      return { noWhitespace: true };
    }
    return null;
  }
  
  // Validator to check if password and confirmation match
  static passwordsMatch(control: AbstractControl): ValidationErrors | null {
    const password = control.get('password')?.value;
    const confirmPassword = control.get('confirmPassword')?.value;
    
    // Check if both fields have values and if they don't match
    if (password && confirmPassword && password !== confirmPassword) {
      // Set error on the confirmPassword control
      control.get('confirmPassword')?.setErrors({ passwordsMatch: true });
      return { passwordsMatch: true };
    }
    return null;
  }
  
  // Validator to check if username is already taken (simulated)
  static usernameTaken(control: AbstractControl): ValidationErrors | null {
    const takenUsernames = ['admin', 'root', 'superuser', 'user'];
    
    if (control.value && takenUsernames.includes(control.value.toLowerCase())) {
      return { usernameTaken: true };
    }
    return null;
  }
  
  // Validator to ensure password contains username
  static passwordContainsUsername(control: AbstractControl): ValidationErrors | null {
    const username = control.get('username')?.value;
    const password = control.get('password')?.value;
    
    if (username && password && password.toLowerCase().includes(username.toLowerCase())) {
      // Set error on the password control
      control.get('password')?.setErrors({ passwordContainsUsername: true });
      return { passwordContainsUsername: true };
    }
    return null;
  }
}

@Component({
  selector: 'app-custom-validators',
  templateUrl: './custom-validators.component.html'
})
export class CustomValidatorsComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(private fb: FormBuilder) { }
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      // Apply custom validator to username
      username: ['', [
        Validators.required,
        Validators.minLength(4),
        CustomValidators.noWhitespace,
        CustomValidators.usernameTaken
      ]],
      
      // Group for password fields
      passwords: this.fb.group({
        password: ['', [
          Validators.required,
          Validators.minLength(8)
        ]],
        confirmPassword: ['', Validators.required]
      }, { validators: CustomValidators.passwordsMatch }) // Group-level validation
    }, { validators: CustomValidators.passwordContainsUsername }); // Form-level validation
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted with:', this.registrationForm.value);
    } else {
      this.markFormGroupTouched(this.registrationForm);
    }
  }
  
  // Helper method to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Convenience getters
  get username() { return this.registrationForm.get('username'); }
  get passwords() { return this.registrationForm.get('passwords') as FormGroup; }
  get password() { return this.passwords.get('password'); }
  get confirmPassword() { return this.passwords.get('confirmPassword'); }
}
```

```html
<!-- custom-validators.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="username">Username</label>
    <input type="text" 
           class="form-control" 
           id="username" 
           formControlName="username">
    <div *ngIf="username?.invalid && (username?.dirty || username?.touched)" 
         class="alert alert-danger">
      <div *ngIf="username?.errors?.['required']">Username is required</div>
      <div *ngIf="username?.errors?.['minlength']">
        Username must be at least {{ username?.errors?.['minlength'].requiredLength }} characters
      </div>
      <div *ngIf="username?.errors?.['noWhitespace']">
        Username cannot contain spaces
      </div>
      <div *ngIf="username?.errors?.['usernameTaken']">
        This username is already taken
      </div>
    </div>
  </div>
  
  <div formGroupName="passwords">
    <div class="form-group">
      <label for="password">Password</label>
      <input type="password" 
             class="form-control" 
             id="password" 
             formControlName="password">
      <div *ngIf="password?.invalid && (password?.dirty || password?.touched)" 
           class="alert alert-danger">
        <div *ngIf="password?.errors?.['required']">Password is required</div>
        <div *ngIf="password?.errors?.['minlength']">
          Password must be at least 8 characters
        </div>
        <div *ngIf="password?.errors?.['passwordContainsUsername']">
          Password cannot contain your username
        </div>
      </div>
    </div>
    
    <div class="form-group">
      <label for="confirmPassword">Confirm Password</label>
      <input type="password" 
             class="form-control" 
             id="confirmPassword" 
             formControlName="confirmPassword">
      <div *ngIf="confirmPassword?.invalid && (confirmPassword?.dirty || confirmPassword?.touched)" 
           class="alert alert-danger">
        <div *ngIf="confirmPassword?.errors?.['required']">
          Please confirm your password
        </div>
        <div *ngIf="confirmPassword?.errors?.['passwordsMatch']">
          Passwords don't match
        </div>
      </div>
    </div>
  </div>
  
  <button type="submit" 
          class="btn btn-primary" 
          [disabled]="registrationForm.invalid">Register</button>
</form>

<!-- Form debugging -->
<div class="form-debug mt-4">
  <h3>Form Value</h3>
  <pre>{{ registrationForm.value | json }}</pre>
  <h3>Form Status</h3>
  <pre>Valid: {{ registrationForm.valid }}</pre>
  <h3>Username Errors</h3>
  <pre>{{ username?.errors | json }}</pre>
  <h3>Password Form Errors</h3>
  <pre>{{ passwords?.errors | json }}</pre>
</div>
```

**Creating Custom Validators**:

1. **Control-level validator**:
```typescript
// Simple validator function
function forbiddenNameValidator(forbiddenName: string) {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = control.value.toLowerCase() === forbiddenName.toLowerCase();
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Usage:
this.form = new FormGroup({
  name: new FormControl('', [
    Validators.required,
    forbiddenNameValidator('admin')
  ])
});
```

2. **Group-level validator**:
```typescript
// Validator that operates on multiple controls
function ageRangeValidator(form: FormGroup) {
  const minAge = form.get('minAge')?.value;
  const maxAge = form.get('maxAge')?.value;
  
  if (minAge && maxAge && minAge > maxAge) {
    return { ageRange: true };
  }
  
  return null;
}

// Usage:
this.form = new FormGroup({
  minAge: new FormControl(''),
  maxAge: new FormControl('')
}, { validators: ageRangeValidator });
```

3. **Directive for template-driven forms**:
```typescript
// Custom validator directive
@Directive({
  selector: '[appForbiddenName]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: ForbiddenNameDirective,
    multi: true
  }]
})
export class ForbiddenNameDirective implements Validator {
  @Input('appForbiddenName') forbiddenName: string = '';
  
  validate(control: AbstractControl): ValidationErrors | null {
    const forbidden = control.value.toLowerCase() === this.forbiddenName.toLowerCase();
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  }
}

// Usage in template:
<input name="username" [(ngModel)]="username" appForbiddenName="admin">
```

**Best Practices for Custom Validators:**
- Keep validators simple, focused on a single validation concern
- Create reusable validators for common validation scenarios
- Provide descriptive error keys for easy identification
- Return null when validation passes, not an empty object
- Include detailed error information for complex validation
- Create validator factories for validators that need configuration
- Test validators thoroughly with a variety of inputs

[Back to Top](#table-of-contents)

## Asynchronous Validators

Asynchronous validators are used when validation logic needs to perform asynchronous operations, such as API calls to check if a username is already taken or if an email exists in the database.

Key features:
- **Returns a Promise or Observable** that resolves to validation errors or null
- Used for **server-side validation** or operations that take time
- Can be combined with synchronous validators
- Shows pending state during validation

Example of asynchronous validators:

```typescript
// async-validators.component.ts
import { Component, OnInit } from '@angular/core';
import { AbstractControl, AsyncValidatorFn, FormBuilder, FormGroup, ValidationErrors, Validators } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { delay, map } from 'rxjs/operators';

// Service to simulate API calls
export class UserService {
  // Simulated database of existing usernames
  private existingUsernames = ['admin', 'john', 'jane', 'user1', 'test'];
  
  // Simulated API call to check if username exists
  checkUsernameExists(username: string): Observable<boolean> {
    // Simulate API delay
    return of(this.existingUsernames.includes(username.toLowerCase()))
      .pipe(delay(1000)); // Simulate 1 second delay
  }
  
  // Simulated API call to check if email exists
  checkEmailExists(email: string): Observable<boolean> {
    // Simulated existing emails
    const existingEmails = ['admin@example.com', 'test@test.com', 'user@domain.com'];
    
    // Simulate API delay
    return of(existingEmails.includes(email.toLowerCase()))
      .pipe(delay(1500)); // Simulate 1.5 second delay
  }
}

// Custom async validators
export class AsyncValidators {
  // Async validator to check if username is already taken
  static usernameTaken(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value) {
        return of(null); // Skip validation if no value
      }
      
      return userService.checkUsernameExists(control.value).pipe(
        map(exists => exists ? { usernameTaken: true } : null)
      );
    };
  }
  
  // Async validator to check if email is already registered
  static emailTaken(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value || !control.value.includes('@')) {
        return of(null); // Skip validation if no value or invalid email format
      }
      
      return userService.checkEmailExists(control.value).pipe(
        map(exists => exists ? { emailTaken: true } : null)
      );
    };
  }
}

@Component({
  selector: 'app-async-validators',
  templateUrl: './async-validators.component.html',
  providers: [UserService] // Provide the service
})
export class AsyncValidatorsComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) { }
  
  ngOnInit() {
    this.createForm();
  }
  
  createForm() {
    this.registrationForm = this.fb.group({
      // Synchronous + asynchronous validators
      username: ['', 
        // Synchronous validators
        [
          Validators.required,
          Validators.minLength(4)
        ],
        // Asynchronous validator
        [
          AsyncValidators.usernameTaken(this.userService)
        ]
      ],
      
      email: ['',
        // Synchronous validators
        [
          Validators.required,
          Validators.email
        ],
        // Asynchronous validator
        [
          AsyncValidators.emailTaken(this.userService)
        ]
      ],
      
      password: ['', [
        Validators.required,
        Validators.minLength(8)
      ]]
    });
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log('Form submitted with:', this.registrationForm.value);
    } else {
      this.markFormGroupTouched(this.registrationForm);
    }
  }
  
  // Helper method to mark all controls as touched
  markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();
      
      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }
  
  // Convenience getters
  get username() { return this.registrationForm.get('username'); }
  get email() { return this.registrationForm.get('email'); }
  get password() { return this.registrationForm.get('password'); }
}
```

```html
<!-- async-validators.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="username">Username</label>
    <input type="text" 
           class="form-control" 
           id="username" 
           formControlName="username">
    
    <!-- Loading indicator for pending validation -->
    <div *ngIf="username?.pending" class="text-info">
      <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
      Checking username availability...
    </div>
    
    <!-- Validation errors -->
    <div *ngIf="username?.invalid && (username?.dirty || username?.touched)" 
         class="alert alert-danger">
      <div *ngIf="username?.errors?.['required']">Username is required</div>
      <div *ngIf="username?.errors?.['minlength']">
        Username must be at least {{ username?.errors?.['minlength'].requiredLength }} characters
      </div>
      <div *ngIf="username?.errors?.['usernameTaken']">
        This username is already taken
      </div>
    </div>
  </div>
  
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" 
           class="form-control" 
           id="email" 
           formControlName="email">
    
    <!-- Loading indicator for pending validation -->
    <div *ngIf="email?.pending" class="text-info">
      <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
      Checking email availability...
    </div>
    
    <!-- Validation errors -->
    <div *ngIf="email?.invalid && (email?.dirty