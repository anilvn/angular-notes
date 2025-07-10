# Angular Routing and Navigation

## Table of Contents

| Topic | Link |
|-------|------|
| **Basics** |  |
| Routing Overview | [What is Angular Routing?](#what-is-angular-routing) |
| Router Module Setup | [Router Module Setup](#router-module-setup) |
| **Route Configuration** |  |
| Basic Routing | [Basic Routing (Component-Based)](#basic-routing-component-based) |
| Nested Routes | [Nested/Child Routes](#nestedchild-routes) |
| Route Parameters | [Route Parameters and Query Parameters](#route-parameters-and-query-parameters) |
| **Advanced Routing** |  |
| Route Guards | [Route Guards](#route-guards) |
| CanActivate | [CanActivate Guards](#canactivate-guards) |
| CanDeactivate | [CanDeactivate Guards](#candeactivate-guards) |
| CanLoad | [CanLoad Guards](#canload-guards) |
| Resolve | [Resolve Guards (Pre-fetch Data)](#resolve-guards-pre-fetch-data) |
| Lazy Loading | [Lazy Loading Routes](#lazy-loading-routes) |

## What is Angular Routing?

Angular routing allows you to navigate between different components/views in your application without having to reload the entire page. It enables you to create a Single Page Application (SPA) with multiple views and navigation paths.

**Key Features:**
- URL-based navigation without page reloads
- Support for parameterized routes
- Component-based routing
- Lazy loading capabilities for better performance
- Advanced navigation control with guards
- Preservation of navigation history

The Angular Router is a powerful service that manages the navigation flow between states of your application, mapping URLs to components.

```typescript
// Basic routing concept
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'products', component: ProductsComponent },
  { path: '', redirectTo: '/home', pathMatch: 'full' }
];
```

[Back to Top](#table-of-contents)

## Router Module Setup

To use the Angular Router, you need to import the `RouterModule` from `@angular/router` and configure it with your application's routes.

### Step 1: Import RouterModule in your app module

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { HomeComponent } from './home/home.component';
import { ProductsComponent } from './products/products.component';
import { NotFoundComponent } from './not-found/not-found.component';

// Define routes
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'products', component: ProductsComponent },
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: '**', component: NotFoundComponent } // Wildcard route for 404
];

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    ProductsComponent,
    NotFoundComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes) // Register routes
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Step 2: Add router outlet in your main component template

```html
<!-- app.component.html -->
<header>
  <nav>
    <ul>
      <li><a routerLink="/home" routerLinkActive="active">Home</a></li>
      <li><a routerLink="/products" routerLinkActive="active">Products</a></li>
    </ul>
  </nav>
</header>

<main>
  <!-- Router outlet is where the routed components will be displayed -->
  <router-outlet></router-outlet>
</main>

<footer>
  <p>My Angular Application &copy; 2025</p>
</footer>
```

### Alternative: Using a Separate Routing Module

For larger applications, it's recommended to create a separate routing module:

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { HomeComponent } from './home/home.component';
import { ProductsComponent } from './products/products.component';
import { NotFoundComponent } from './not-found/not-found.component';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'products', component: ProductsComponent },
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Then import this module in your app.module.ts:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
// Import other components...

@NgModule({
  declarations: [
    AppComponent,
    // Other components...
  ],
  imports: [
    BrowserModule,
    AppRoutingModule // Import the routing module
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**Best Practices:**
- Use a separate routing module for complex applications
- Place the wildcard route (`**`) at the end of your routes configuration
- Use `RouterModule.forRoot()` only in the app's main module
- Use `RouterModule.forChild()` in feature modules

[Back to Top](#table-of-contents)

## Basic Routing (Component-Based)

Component-based routing is the foundation of Angular routing, where each route is mapped to a specific component.

### Setting Up Basic Routes

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { ContactComponent } from './contact/contact.component';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent },
  { path: '', redirectTo: '/home', pathMatch: 'full' } // Default route
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Navigation in Templates

```html
<!-- app.component.html -->
<nav>
  <ul>
    <li><a routerLink="/home" routerLinkActive="active">Home</a></li>
    <li><a routerLink="/about" routerLinkActive="active">About</a></li>
    <li><a routerLink="/contact" routerLinkActive="active">Contact</a></li>
  </ul>
</nav>

<router-outlet></router-outlet>
```

### Programmatic Navigation

You can also navigate programmatically from your component classes:

```typescript
// some.component.ts
import { Router } from '@angular/router';

@Component({
  selector: 'app-some',
  templateUrl: './some.component.html'
})
export class SomeComponent {
  constructor(private router: Router) { }
  
  navigateToAbout(): void {
    // Navigate to the about page
    this.router.navigate(['/about']);
  }
  
  navigateToContact(): void {
    // Navigate with additional options
    this.router.navigateByUrl('/contact', { skipLocationChange: true });
  }
}
```

```html
<!-- some.component.html -->
<button (click)="navigateToAbout()">Go to About</button>
<button (click)="navigateToContact()">Go to Contact</button>
```

### Route Configuration Options

```typescript
const routes: Routes = [
  { 
    path: 'home', 
    component: HomeComponent,
    pathMatch: 'full', // Match the entire URL
    title: 'Home Page', // Page title
    data: { showSidebar: true } // Custom data to be passed to the route
  }
];
```

**Best Practices:**
- Use `routerLink` directive for template-based navigation
- Use `routerLinkActive` to style active links
- For programmatic navigation, inject the Router service
- Use path matching strategies appropriately ('prefix' or 'full')
- Always include a default route that redirects to a valid page

[Back to Top](#table-of-contents)

## Nested/Child Routes

Nested routes (or child routes) allow you to create a hierarchical route structure where a parent component can have its own set of child routes.

### Basic Structure

```typescript
// app-routing.module.ts
const routes: Routes = [
  { 
    path: 'products', 
    component: ProductsComponent,
    children: [
      { path: 'list', component: ProductListComponent },
      { path: 'details/:id', component: ProductDetailsComponent },
      { path: '', redirectTo: 'list', pathMatch: 'full' }
    ]
  }
];
```

### Parent Component Template

```html
<!-- products.component.html -->
<h2>Products Section</h2>

<nav>
  <ul>
    <li><a routerLink="list">Product List</a></li>
    <li><a routerLink="details/1">Product Details (ID: 1)</a></li>
  </ul>
</nav>

<!-- Child components will be rendered here -->
<router-outlet></router-outlet>

<footer>
  <p>Products Footer</p>
</footer>
```

### Complete Example

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { HomeComponent } from './home/home.component';
import { ProfileComponent } from './profile/profile.component';
import { SettingsComponent } from './settings/settings.component';
import { DashboardComponent } from './dashboard/dashboard.component';
import { StatsComponent } from './stats/stats.component';
import { ActivityComponent } from './activity/activity.component';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { 
    path: 'dashboard', 
    component: DashboardComponent,
    children: [
      { path: 'stats', component: StatsComponent },
      { path: 'activity', component: ActivityComponent },
      { path: '', redirectTo: 'stats', pathMatch: 'full' }
    ]
  },
  { 
    path: 'profile', 
    component: ProfileComponent,
    children: [
      { path: 'settings', component: SettingsComponent }
    ]
  },
  { path: '', redirectTo: '/home', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Multiple Router Outlets (Named Outlets)

You can also have multiple router outlets using named outlets:

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    children: [
      { path: '', component: AdminDashboardComponent },
      { path: 'users', component: UsersComponent, outlet: 'adminPanel' },
      { path: 'settings', component: AdminSettingsComponent, outlet: 'adminPanel' }
    ]
  }
];
```

```html
<!-- admin.component.html -->
<h2>Admin Panel</h2>

<nav>
  <ul>
    <li><a [routerLink]="['/admin', { outlets: { adminPanel: ['users'] } }]">Manage Users</a></li>
    <li><a [routerLink]="['/admin', { outlets: { adminPanel: ['settings'] } }]">Admin Settings</a></li>
  </ul>
</nav>

<!-- Primary router outlet -->
<router-outlet></router-outlet>

<!-- Named router outlet -->
<div class="admin-panel">
  <router-outlet name="adminPanel"></router-outlet>
</div>
```

**Best Practices:**
- Use child routes for logical grouping of related features
- Keep parent components lightweight, focusing on layout
- Use route parameters for dynamic child routes
- Consider using named outlets for complex layouts with multiple dynamic areas
- Always provide default child routes for better user experience

[Back to Top](#table-of-contents)

## Route Parameters and Query Parameters

Route parameters and query parameters allow you to pass data between routes in your Angular application.

### Route Parameters

Route parameters are part of the URL path and are typically used for identifying specific resources:

```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: 'products/:id', component: ProductDetailComponent },
  { path: 'users/:userId/posts/:postId', component: UserPostComponent }
];
```

#### Accessing Route Parameters

```typescript
// product-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-product-detail',
  templateUrl: './product-detail.component.html'
})
export class ProductDetailComponent implements OnInit {
  productId: string | null = null;
  
  constructor(private route: ActivatedRoute) { }
  
  ngOnInit(): void {
    // Method 1: Using snapshot (one-time)
    this.productId = this.route.snapshot.paramMap.get('id');
    
    // Method 2: Using observable (reactive)
    this.route.paramMap.subscribe(params => {
      this.productId = params.get('id');
      // Fetch product details based on this ID
      this.loadProductDetails();
    });
  }
  
  loadProductDetails(): void {
    // Service call to load product details
    console.log(`Loading product with ID: ${this.productId}`);
  }
}
```

#### Navigating with Route Parameters

```typescript
// In a component
import { Router } from '@angular/router';

@Component({/* ... */})
export class SomeComponent {
  constructor(private router: Router) { }
  
  viewProduct(productId: string): void {
    this.router.navigate(['/products', productId]);
  }
}
```

```html
<!-- In a template -->
<a [routerLink]="['/products', product.id]">View Product</a>
```

### Query Parameters

Query parameters are appended to the URL after a `?` and are typically used for filtering, sorting, or pagination:

#### Setting Query Parameters

```typescript
// In a component
import { Router } from '@angular/router';

@Component({/* ... */})
export class ProductListComponent {
  constructor(private router: Router) { }
  
  applyFilters(category: string, sortBy: string): void {
    this.router.navigate(['/products'], {
      queryParams: { category, sortBy, page: 1 }
    });
  }
  
  // Preserving existing query params
  addFilter(newParam: string): void {
    this.router.navigate([], {
      relativeTo: this.route,
      queryParams: { filter: newParam },
      queryParamsHandling: 'merge' // 'preserve' or 'merge'
    });
  }
}
```

```html
<!-- In a template -->
<a [routerLink]="['/products']" [queryParams]="{category: 'electronics', sortBy: 'price'}">
  Electronics (Sort by price)
</a>
```

#### Accessing Query Parameters

```typescript
// product-list.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html'
})
export class ProductListComponent implements OnInit {
  category: string | null = null;
  sortBy: string | null = null;
  
  constructor(private route: ActivatedRoute) { }
  
  ngOnInit(): void {
    // Method 1: Using snapshot (one-time)
    this.category = this.route.snapshot.queryParamMap.get('category');
    this.sortBy = this.route.snapshot.queryParamMap.get('sortBy');
    
    // Method 2: Using observable (reactive)
    this.route.queryParamMap.subscribe(params => {
      this.category = params.get('category');
      this.sortBy = params.get('sortBy');
      // Apply filters based on these parameters
      this.applyFilters();
    });
  }
  
  applyFilters(): void {
    console.log(`Filtering by category: ${this.category}, Sorting by: ${this.sortBy}`);
    // Filter products
  }
}
```

### Matrix Parameters (URL Matrix Notation)

Another way to pass parameters is using matrix notation:

```typescript
// In a component
navigateWithMatrix(): void {
  this.router.navigate(['/products', { category: 'electronics', brand: 'samsung' }]);
}
```

This produces a URL like: `/products;category=electronics;brand=samsung`

```typescript
// Access matrix parameters
this.route.paramMap.subscribe(params => {
  const category = params.get('category');
  const brand = params.get('brand');
});
```

### Comparison: Route Parameters vs Query Parameters

| Feature | Route Parameters | Query Parameters |
|---------|------------------|------------------|
| URL Format | `/products/123` | `/products?id=123` |
| Typical Use | Resource identification | Filtering, sorting, pagination |
| Required | Generally required | Optional |
| Multiple Values | Needs separate parameters | Can use arrays `?tags=a&tags=b` |
| Preservation | Part of route definition | Can be preserved across navigations |
| Bookmarkability | Good for core resource identity | Good for view state |

**Best Practices:**
- Use route parameters for essential resource identification
- Use query parameters for optional filters or view states
- Always subscribe to parameter observables when you need to react to parameter changes
- Use `queryParamsHandling` to preserve or merge query parameters during navigation
- Consider unsubscribing from parameter observables in `ngOnDestroy` to prevent memory leaks

[Back to Top](#table-of-contents)

## Route Guards

Route guards in Angular allow you to control navigation to and from routes. They are used to prevent unauthorized access, unsaved changes from being lost, or ensure data is loaded before a component is displayed.

### Types of Route Guards

| Guard Type | Interface | Purpose |
|------------|-----------|---------|
| CanActivate | `CanActivate` | Controls if a route can be activated |
| CanActivateChild | `CanActivateChild` | Controls if children routes can be activated |
| CanDeactivate | `CanDeactivate` | Controls if a user can leave a route |
| CanLoad | `CanLoad` | Controls if a lazy-loaded module can be loaded |
| Resolve | `Resolve` | Pre-fetches data before a route is activated |

### Implementation Options

Starting with Angular 14+, there are two ways to implement guards:

1. **Functional Guards** (recommended approach in modern Angular)
2. **Class-based Guards** (traditional approach)

Here's a comparison:

| Feature | Functional Guards | Class-based Guards |
|---------|-------------------|-------------------|
| Implementation | Pure functions | Classes implementing interfaces |
| Dependency Injection | Uses inject() function | Uses constructor injection |
| Syntax | More concise | More verbose |
| Introduced | Angular 14+ | Initial implementation |

[Back to Top](#table-of-contents)

## CanActivate Guards

CanActivate guards protect routes by preventing unauthorized access. They are commonly used for authentication and authorization.

### Functional Implementation (Angular 14+)

```typescript
// auth.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router, UrlTree } from '@angular/router';
import { AuthService } from './auth.service';
import { map } from 'rxjs/operators';
import { Observable } from 'rxjs';

export const authGuard: CanActivateFn = (route, state): 
  boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree> => {
  
  const authService = inject(AuthService);
  const router = inject(Router);
  
  // Simple synchronous check
  if (authService.isLoggedIn()) {
    return true; // Allow navigation
  }
  
  // Return URL tree for redirection
  return router.createUrlTree(['/login'], { 
    queryParams: { returnUrl: state.url } 
  });
  
  // Alternative: Return Observable
  // return authService.isAuthenticated().pipe(
  //   map(isAuth => isAuth ? true : router.createUrlTree(['/login']))
  // );
};
```

### Class-based Implementation

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { 
  CanActivate, 
  ActivatedRouteSnapshot, 
  RouterStateSnapshot, 
  Router, 
  UrlTree 
} from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  
  constructor(private authService: AuthService, private router: Router) {}
  
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    
    // Simple synchronous check
    if (this.authService.isLoggedIn()) {
      return true;
    }
    
    // Redirect to login page with return URL
    return this.router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url }
    });
    
    // Alternative: Using Observable
    // return this.authService.isAuthenticated().pipe(
    //   map(isAuth => isAuth ? true : this.router.createUrlTree(['/login']))
    // );
  }
}
```

### Using Route Data with Guards

You can pass data to guards using the route's data property:

```typescript
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [authGuard],
    data: { requiredRole: 'ADMIN' }
  }
];
```

```typescript
// Role-based auth guard
export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  // Get required role from route data
  const requiredRole = route.data['requiredRole'];
  
  if (authService.hasRole(requiredRole)) {
    return true;
  }
  
  return router.createUrlTree(['/unauthorized']);
};
```

### Applying Guards in Route Configuration

```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { 
    path: 'profile', 
    component: ProfileComponent,
    canActivate: [authGuard]
  },
  { 
    path: 'admin', 
    component: AdminComponent,
    canActivate: [authGuard, roleGuard],
    data: { requiredRole: 'ADMIN' }
  },
  { 
    path: 'settings', 
    component: SettingsComponent,
    canActivate: [authGuard],
    canActivateChild: [roleGuard],
    children: [
      { path: 'security', component: SecuritySettingsComponent },
      { path: 'billing', component: BillingSettingsComponent }
    ]
  }
];
```

**Best Practices:**
- Use functional guards in newer Angular applications
- Return UrlTree instead of boolean false for better user experience (automatic redirection)
- Keep guards focused on a single responsibility
- Use route data to make guards reusable and configurable
- Consider combining guards for complex authorization scenarios

[Back to Top](#table-of-contents)

## CanDeactivate Guards

CanDeactivate guards prevent users from navigating away from a route prematurely, often used to prevent loss of unsaved changes.

### Functional Implementation (Angular 14+)

```typescript
// unsaved-changes.guard.ts
import { CanDeactivateFn } from '@angular/router';
import { Observable } from 'rxjs';

// Interface that components should implement
export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

export const unsavedChangesGuard: CanDeactivateFn<CanComponentDeactivate> = 
  (component, currentRoute, currentState, nextState) => {
  
  // Call the component's canDeactivate method
  return component.canDeactivate ? 
    component.canDeactivate() : 
    true;
};
```

### Component Implementation

```typescript
// edit-product.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';
import { CanComponentDeactivate } from './unsaved-changes.guard';

@Component({
  selector: 'app-edit-product',
  templateUrl: './edit-product.component.html'
})
export class EditProductComponent implements OnInit, CanComponentDeactivate {
  productForm!: FormGroup;
  originalData: any = {};
  savedChanges = true;
  
  constructor(private fb: FormBuilder) { }
  
  ngOnInit(): void {
    this.productForm = this.fb.group({
      name: [''],
      price: [''],
      description: ['']
    });
    
    // Load initial data
    this.loadProduct();
    
    // Monitor form changes
    this.productForm.valueChanges.subscribe(() => {
      this.savedChanges = false;
    });
  }
  
  loadProduct(): void {
    // Simulate loading data
    const productData = {
      name: 'Sample Product',
      price: 29.99,
      description: 'This is a sample product'
    };
    
    this.originalData = { ...productData };
    this.productForm.setValue(productData);
    this.savedChanges = true;
  }
  
  saveProduct(): void {
    // Save logic...
    console.log('Product saved:', this.productForm.value);
    this.savedChanges = true;
  }
  
  // Implement canDeactivate method
  canDeactivate(): boolean | Promise<boolean> {
    if (this.savedChanges) {
      return true;
    }
    
    return new Promise<boolean>(resolve => {
      const result = window.confirm('You have unsaved changes. Do you really want to leave?');
      resolve(result);
    });
  }
}
```

### Class-based Implementation

```typescript
// unsaved-changes.guard.ts
import { Injectable } from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { Observable } from 'rxjs';

export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({
  providedIn: 'root'
})
export class UnsavedChangesGuard implements CanDeactivate<CanComponentDeactivate> {
  
  canDeactivate(
    component: CanComponentDeactivate
  ): Observable<boolean> | Promise<boolean> | boolean {
    
    return component.canDeactivate ? 
      component.canDeactivate() : 
      true;
  }
}
```

### Route Configuration

```typescript
// app-routing.module.ts
import { UnsavedChangesGuard } from './unsaved-changes.guard';
import { EditProductComponent } from './edit-product/edit-product.component';
// or import { unsavedChangesGuard } from './unsaved-changes.guard'; // for functional guard

const routes: Routes = [
  { 
    path: 'products/:id/edit', 
    component: EditProductComponent,
    canDeactivate: [unsavedChangesGuard] // or [UnsavedChangesGuard] for class-based
  }
];
```

### Advanced Example: Generic Deactivation with Form Detection

```typescript
// enhanced-deactivate.guard.ts
import { CanDeactivateFn } from '@angular/router';
import { Observable } from 'rxjs';
import { FormGroup } from '@angular/forms';

// Interface for components with reactive forms
export interface FormComponent {
  form?: FormGroup;
  isDirty?: () => boolean;
  canDeactivate?: () => Observable<boolean> | Promise<boolean> | boolean;
}

export const formDeactivateGuard: CanDeactivateFn<FormComponent> = 
  (component, currentRoute, currentState, nextState) => {
  
  // Check if component has explicit canDeactivate method
  if (component.canDeactivate) {
    return component.canDeactivate();
  }
  
  // Check if component has isDirty method
  if (component.isDirty) {
    const isDirty = component.isDirty();
    if (isDirty) {
      return confirm('You have unsaved changes. Do you want to leave this page?');
    }
    return true;
  }
  
  // Fall back to form dirty check
  if (component.form && component.form.dirty) {
    return confirm('You have unsaved changes. Do you want to leave this page?');
  }
  
  // No changes detected
  return true;
};
```

**Best Practices:**
- Implement a consistent interface for components that need deactivation guards
- Make guards generic and reusable
- Provide clear messages to users about potential data loss
- Consider using custom dialog components instead of browser confirm
- Allow components to have their own canDeactivate logic
- Check form dirty state as a fallback mechanism

[Back to Top](#table-of-contents)

## CanLoad Guards

CanLoad guards prevent lazy-loaded modules from being loaded until certain conditions are met. This is more restrictive than CanActivate because it blocks the entire module from loading, not just the navigation. This is particularly useful for security and performance optimization.

### Functional Implementation (Angular 14+)

```typescript
// auth-canload.guard.ts
import { inject } from '@angular/core';
import { CanLoadFn, Router, UrlTree } from '@angular/router';
import { AuthService } from './auth.service';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export const authCanLoadGuard: CanLoadFn = 
  (route): boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree> => {
  
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isLoggedIn()) {
    return true;
  }
  
  // Redirect to login
  return router.createUrlTree(['/login']);
  
  // Alternative with Observable
  // return authService.isAuthenticated().pipe(
  //   map(isAuth => isAuth ? true : router.createUrlTree(['/login']))
  // );
};
```

### Class-based Implementation

```typescript
// auth-canload.guard.ts
import { Injectable } from '@angular/core';
import { CanLoad, Route, UrlSegment, Router, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthCanLoadGuard implements CanLoad {
  
  constructor(private authService: AuthService, private router: Router) {}
  
  canLoad(
    route: Route,
    segments: UrlSegment[]
  ): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    
    if (this.authService.isLoggedIn()) {
      return true;
    }
    
    // Redirect to login
    return this.router.createUrlTree(['/login']);
    
    // Alternative with Observable
    // return this.authService.isAuthenticated().pipe(
    //   map(isAuth => isAuth ? true : this.router.createUrlTree(['/login']))
    // );
  }
}

### Route Configuration with CanLoad

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { authCanLoadGuard } from './auth-canload.guard';
// or import { AuthCanLoadGuard } from './auth-canload.guard'; // for class-based

const routes: Routes = [
  { 
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canLoad: [authCanLoadGuard] // or [AuthCanLoadGuard] for class-based
  },
  { 
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule),
    canLoad: [authCanLoadGuard]
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### CanLoad vs CanActivate

| Guard Type | When to Use | Prevents | Bundle Loading |
|------------|-------------|----------|----------------|
| CanLoad | When you want to prevent unauthorized users from downloading module code | Module loading and navigation | Yes, prevents bundle download |
| CanActivate | When you want to prevent access but may still allow code to be downloaded | Only navigation to route | No, bundle still downloads |

**Best Practices:**
- Use CanLoad for sensitive features that shouldn't be downloaded by unauthorized users
- Apply CanLoad guards to lazy-loaded modules, not eagerly loaded ones
- Remember that CanLoad only runs once when the module is first loaded
- Use CanLoad for features that have significant bundle size to improve load performance
- Consider combining with CanActivate guards for complete protection

[Back to Top](#table-of-contents)

## Resolve Guards (Pre-fetch Data)

Resolve guards are used to pre-fetch data before a route is activated. They ensure that necessary data is available before a component is displayed, improving user experience by avoiding partially loaded views.

### Functional Implementation (Angular 14+)

```typescript
// product-resolver.ts
import { inject } from '@angular/core';
import { ResolveFn, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { Product } from './product.model';
import { ProductService } from './product.service';

export const productResolver: ResolveFn<Product | null> = 
  (route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Product | null> => {
  
  const productService = inject(ProductService);
  const productId = route.paramMap.get('id');
  
  if (!productId) {
    return of(null);
  }
  
  return productService.getProduct(productId).pipe(
    catchError(error => {
      console.error('Error fetching product data', error);
      return of(null);
    })
  );
};
```

### Class-based Implementation

```typescript
// product-resolver.service.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { Product } from './product.model';
import { ProductService } from './product.service';

@Injectable({
  providedIn: 'root'
})
export class ProductResolver implements Resolve<Product | null> {
  
  constructor(private productService: ProductService) {}
  
  resolve(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<Product | null> {
    
    const productId = route.paramMap.get('id');
    
    if (!productId) {
      return of(null);
    }
    
    return this.productService.getProduct(productId).pipe(
      catchError(error => {
        console.error('Error fetching product data', error);
        return of(null);
      })
    );
  }
}
```

### Route Configuration

```typescript
// app-routing.module.ts
import { productResolver } from './product-resolver';
// or import { ProductResolver } from './product-resolver.service'; // for class-based

const routes: Routes = [
  { 
    path: 'products/:id',
    component: ProductDetailComponent,
    resolve: {
      product: productResolver // or ProductResolver for class-based
    }
  }
];
```

### Accessing Resolved Data in Component

```typescript
// product-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Product } from './product.model';

@Component({
  selector: 'app-product-detail',
  templateUrl: './product-detail.component.html'
})
export class ProductDetailComponent implements OnInit {
  product: Product | null = null;
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit(): void {
    // Resolved data is available in the data object 
    // with the key specified in the route configuration
    this.product = this.route.snapshot.data['product'];
    
    // Alternative: reactive approach for handling route changes
    this.route.data.subscribe(data => {
      this.product = data['product'];
    });
  }
}
```

### Multiple Resolvers

You can use multiple resolvers for a single route:

```typescript
const routes: Routes = [
  { 
    path: 'dashboard',
    component: DashboardComponent,
    resolve: {
      user: userResolver,
      stats: statsResolver,
      notifications: notificationsResolver
    }
  }
];
```

```typescript
// In component
ngOnInit(): void {
  this.route.data.subscribe(data => {
    this.user = data['user'];
    this.stats = data['stats'];
    this.notifications = data['notifications'];
  });
}
```

### Error Handling in Resolvers

```typescript
export const productsResolver: ResolveFn<Product[] | null> = 
  (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
  
  const productService = inject(ProductService);
  const router = inject(Router);
  
  return productService.getProducts().pipe(
    // Option 1: Return empty data on error
    catchError(error => {
      console.error('Error fetching products', error);
      return of([]);
    })
    
    // Option 2: Redirect on error
    // catchError(error => {
    //   console.error('Error fetching products', error);
    //   router.navigate(['/error']);
    //   return EMPTY; // Stop the navigation
    // })
  );
};
```

**Best Practices:**
- Use resolvers for critical data that must be available before showing a view
- Always handle errors in resolvers to prevent navigation from getting stuck
- Keep resolvers focused on data fetching only, not business logic
- For multiple related data items, consider a single resolver that returns an object
- Use functional resolvers in newer Angular applications
- Consider loading indicators during resolver execution for better UX
- For optional data, consider loading it in the component instead

[Back to Top](#table-of-contents)

## Lazy Loading Routes

Lazy loading is a design pattern that delays the loading of non-critical resources until they are needed. In Angular routing, lazy loading means loading feature modules only when the user navigates to their routes, which improves initial load time and application performance.

### Basic Lazy Loading Setup

1. First, organize your application into feature modules

```typescript
// admin/admin.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AdminRoutingModule } from './admin-routing.module';
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
import { UserManagementComponent } from './user-management/user-management.component';
import { SettingsComponent } from './settings/settings.component';

@NgModule({
  declarations: [
    AdminDashboardComponent,
    UserManagementComponent,
    SettingsComponent
  ],
  imports: [
    CommonModule,
    AdminRoutingModule
  ]
})
export class AdminModule { }
```

2. Create a routing module for the feature module

```typescript
// admin/admin-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
import { UserManagementComponent } from './user-management/user-management.component';
import { SettingsComponent } from './settings/settings.component';

const routes: Routes = [
  {
    path: '', // Empty path since the parent route already has 'admin'
    component: AdminDashboardComponent,
    children: [
      { path: 'users', component: UserManagementComponent },
      { path: 'settings', component: SettingsComponent },
      { path: '', redirectTo: 'users', pathMatch: 'full' }
    ]
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AdminRoutingModule { }
```

3. Configure lazy loading in the main routing module

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { authGuard } from './auth.guard';

const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { 
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canActivate: [authGuard]
  },
  { 
    path: 'products',
    loadChildren: () => import('./products/products.module').then(m => m.ProductsModule)
  },
  { 
    path: 'customer',
    loadChildren: () => import('./customer/customer.module').then(m => m.CustomerModule)
  },
  { path: '', redirectTo: 'home', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Preloading Strategies

Angular provides several strategies to preload lazy-loaded modules:

1. **NoPreloading** (default): Modules are loaded only when requested
2. **PreloadAllModules**: All lazy-loaded modules are preloaded after the app loads
3. **Custom Preloading**: Define your own logic for which modules to preload

#### Configuring Preloading

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes, PreloadAllModules } from '@angular/router';

const routes: Routes = [ /* ... */ ];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: PreloadAllModules
    })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

#### Custom Preloading Strategy

```typescript
// selective-preloading-strategy.ts
import { Injectable } from '@angular/core';
import { PreloadingStrategy, Route } from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Only preload routes that have data.preload set to true
    if (route.data && route.data['preload']) {
      console.log(`Preloading: ${route.path}`);
      return load();
    } else {
      return of(null);
    }
  }
}
```

```typescript
// app-routing.module.ts
import { SelectivePreloadingStrategy } from './selective-preloading-strategy';

const routes: Routes = [
  // ...
  { 
    path: 'frequently-used',
    loadChildren: () => import('./frequently-used/frequently-used.module').then(m => m.FrequentlyUsedModule),
    data: { preload: true } // Mark this for preloading
  },
  { 
    path: 'rarely-used',
    loadChildren: () => import('./rarely-used/rarely-used.module').then(m => m.RarelyUsedModule)
    // Not marked for preloading
  }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: SelectivePreloadingStrategy
    })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### On-Demand Lazy Loading

You can also trigger lazy loading programmatically:

```typescript
// app.component.ts
import { Component, Injector } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  constructor(private router: Router, private injector: Injector) { }
  
  preloadAdminModule(): void {
    // Preload the admin module
    import('./admin/admin.module').then(m => {
      console.log('Admin module preloaded');
      // No need to do anything else - it's now in cache
    });
  }
  
  onUserLogin(role: string): void {
    if (role === 'admin') {
      // Preload admin module when admin logs in
      this.preloadAdminModule();
    }
  }
}
```

### Comparison: Eager Loading vs Lazy Loading

| Feature | Eager Loading | Lazy Loading |
|---------|---------------|--------------|
| Bundle Size | Larger initial bundle | Smaller initial bundle |
| Load Time | Longer initial load | Faster initial load |
| Subsequent Navigation | Immediate | Small delay on first access |
| Setup Complexity | Simpler | More complex |
| Best For | Small applications | Large applications |
| Module Import | Direct import | Dynamic import |
| Route Configuration | Standard routes | loadChildren property |

**Best Practices:**
- Organize your application into feature modules for effective lazy loading
- Apply lazy loading to large feature modules not needed at startup
- Consider using preloading strategies for better user experience
- Combine lazy loading with route guards for secure and efficient loading
- Use meaningful names for feature modules to make bundle names clear
- Monitor bundle sizes to ensure effective code splitting
- Keep critical path components in the main bundle for fast initial load
- Consider on-demand preloading for predictable user flows

[Back to Top](#table-of-contents)