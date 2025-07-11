# FULL Angular Feature Hierarchy (Comprehensive Cheat Sheet)



## 1. **Directives**

### Structural Directives

| Directive | Description |
|-----------|-------------|
| `*ngIf` | Conditionally render elements based on boolean expression |
| `*ngFor` | Loop through collections and render template for each item |
| `*ngSwitch` | Multi-conditional rendering with switch-case logic |
| `ng-template` | Define reusable template that can be instantiated conditionally |
| `ng-container` | Grouping element that doesn't render in DOM |
| `ngTemplateOutlet` | Dynamically render template with context |
| `*ngComponentOutlet` | Dynamically render component instances |

### Attribute Directives

| Directive | Description |
|-----------|-------------|
| `ngClass` | Add/remove CSS classes conditionally |
| `ngStyle` | Apply inline styles dynamically |
| `ngModel` | Two-way data binding for form controls |
| `[disabled]`, `[hidden]`, `[readonly]` | Bind to HTML element attributes |
| `[attr.aria-label]`, `[attr.data-*]` | Bind to custom HTML attributes |
| Custom Attribute Directives | User-defined directives that modify element behavior |

---

## 2. **Data Binding & Event System**

### Binding Types

| Type | Syntax | Description |
|------|--------|-------------|
| **Interpolation** | `{{ expression }}` | Display component data in template |
| **Property Binding** | `[property]="value"` | Bind component property to element property |
| **Attribute Binding** | `[attr.name]="value"` | Bind to HTML attributes |
| **Class Binding** | `[class.className]="condition"` | Toggle CSS classes conditionally |
| **Style Binding** | `[style.property]="value"` | Apply inline styles dynamically |
| **Event Binding** | `(event)="handler($event)"` | Handle DOM events |
| **Two-way Binding** | `[(ngModel)]="property"` | Bidirectional data binding |

### Event Handling

| Feature | Description |
|---------|-------------|
| `$event` | Access event object in event handlers |
| `preventDefault()` | Prevent default browser behavior |
| `stopPropagation()` | Stop event bubbling |
| Custom Events | Create and emit custom events using EventEmitter |
| Keyboard Events | Handle `keydown`, `keyup`, `keypress` events |
| Mouse Events | Handle `click`, `mouseenter`, `mouseleave` events |

---

## 3. **Component Architecture**

### Component Decorators

| Decorator | Description |
|-----------|-------------|
| `@Component` | Defines a UI component with metadata |
| `@Input()` | Accepts data from parent component |
| `@Output()` | Emits events to parent component |
| `@ViewChild()` | Access child element or component in view |
| `@ViewChildren()` | Access multiple child elements or components |
| `@ContentChild()` | Access projected content element |
| `@ContentChildren()` | Access multiple projected content elements |
| `@HostBinding()` | Bind to host element properties |
| `@HostListener()` | Listen to host element events |

### Component Communication

| Feature | Description |
|---------|-------------|
| `@Input()` with setters | Intercept and process input property changes |
| `@Output() EventEmitter` | Emit custom events to parent |
| Template Reference Variables | Access component/element using `#variable` |
| `ViewChild` with `static` | Access child before ngAfterViewInit |
| Parent-Child Communication | Pass data down and events up |
| Service Communication | Share data between unrelated components |

### Content Projection

| Feature | Description |
|---------|-------------|
| `<ng-content>` | Single-slot content projection |
| `<ng-content select="selector">` | Multi-slot content projection |
| `ngProjectAs` | Project content with different selector |
| Conditional Content Projection | Project content based on conditions |

---

## 4. **Lifecycle Hooks**

| Hook | When It Runs | Use Case |
|------|-------------|----------|
| `ngOnChanges()` | When input-bound properties change | React to input changes |
| `ngOnInit()` | After component initialization | Initialize component data |
| `ngDoCheck()` | During every change detection cycle | Custom change detection |
| `ngAfterContentInit()` | After content projection | Access projected content |
| `ngAfterContentChecked()` | After projected content is checked | Monitor projected content changes |
| `ngAfterViewInit()` | After view and child views initialization | Access ViewChild elements |
| `ngAfterViewChecked()` | After view and child views are checked | Monitor view changes |
| `ngOnDestroy()` | Before component destruction | Cleanup subscriptions and resources |

---

## 5. **Forms**

### Template-driven Forms

| Feature | Description |
|---------|-------------|
| `FormsModule` | Enables template-driven forms |
| `[(ngModel)]` | Two-way binding for form controls |
| `ngModelGroup` | Group form controls together |
| `ngForm` | Reference to form instance |
| `(ngSubmit)` | Handle form submission |
| `#formRef="ngForm"` | Template reference to form |
| `required`, `minlength`, `maxlength` | Built-in validators |
| `ngModel` tracking | Track form control states (touched, dirty, valid) |

### Reactive Forms

| Feature | Description |
|---------|-------------|
| `ReactiveFormsModule` | Enables reactive forms |
| `FormGroup` | Group of form controls |
| `FormControl` | Individual form control |
| `FormArray` | Array of form controls |
| `FormBuilder` | Service to create form controls |
| `formGroup`, `formControlName` | Bind reactive controls to template |
| `Validators` | Built-in validation functions |
| Custom Validators | Create custom validation logic |
| `AsyncValidator` | Asynchronous validation |
| `updateOn` | Control when validation runs |

### Form Validation

| Feature | Description |
|---------|-------------|
| `valid`, `invalid` | Form/control validation state |
| `touched`, `untouched` | User interaction state |
| `dirty`, `pristine` | Value modification state |
| `errors` | Object containing validation errors |
| `hasError()` | Check for specific validation errors |
| `setErrors()` | Set custom validation errors |
| Cross-field Validation | Validate across multiple form fields |

---

## 6. **Routing & Navigation**

### Basic Routing

| Feature | Description |
|---------|-------------|
| `RouterModule` | Enables routing functionality |
| `Routes[]` | Array of route configurations |
| `<router-outlet>` | Displays routed component |
| `[routerLink]` | Navigate to route declaratively |
| `routerLinkActive` | Add CSS class when route is active |
| `Router` | Service for imperative navigation |
| `ActivatedRoute` | Service to access route information |

### Advanced Routing

| Feature | Description |
|---------|-------------|
| Route Parameters | Pass data through URL segments |
| Query Parameters | Pass data through URL query string |
| Route Fragments | Navigate to specific page sections |
| Nested Routes | Child routes within parent routes |
| Route Guards | Control access to routes |
| Route Resolvers | Pre-load data before navigation |
| Lazy Loading | Load modules on demand |
| Auxiliary Routes | Multiple router outlets |

### Route Guards

| Guard | Description |
|-------|-------------|
| `CanActivate` | Control route activation |
| `CanDeactivate` | Control route deactivation |
| `CanLoad` | Control module loading |
| `CanActivateChild` | Control child route activation |
| `Resolve` | Pre-load data before navigation |

---

## 7. **Dependency Injection (DI)**

### Service Configuration

| Feature | Description |
|---------|-------------|
| `@Injectable()` | Mark class as injectable |
| `providedIn: 'root'` | Singleton service application-wide |
| `providers` | Configure service providers |
| `useClass` | Provide different implementation |
| `useValue` | Provide static value |
| `useFactory` | Provide using factory function |
| `useExisting` | Alias for existing provider |
| `multi: true` | Multiple providers for same token |

### Injection Tokens

| Feature | Description |
|---------|-------------|
| `InjectionToken` | Type-safe injection token |
| `@Inject()` | Inject using custom token |
| `@Optional()` | Mark dependency as optional |
| `@Self()` | Only look in current injector |
| `@SkipSelf()` | Skip current injector |
| `@Host()` | Only look in host component |
| Hierarchical Injection | Nested injector tree |

---

## 8. **Pipes**

### Built-in Pipes

| Pipe | Description |
|------|-------------|
| `date` | Format dates |
| `uppercase`, `lowercase` | Transform text case |
| `currency` | Format currency values |
| `percent` | Format percentage values |
| `decimal` | Format decimal numbers |
| `json` | Convert object to JSON string |
| `slice` | Extract portion of array/string |
| `async` | Subscribe to observables/promises |
| `keyvalue` | Transform object to key-value pairs |
| `titlecase` | Transform to title case |

### Custom Pipes

| Feature | Description |
|---------|-------------|
| `@Pipe()` | Create custom pipe |
| `transform()` | Pipe transformation method |
| `pure` vs `impure` | Pipe execution strategy |
| Parameterized Pipes | Pass arguments to pipes |
| Chaining Pipes | Apply multiple pipes in sequence |

---

## 9. **Modules & Application Structure**

### Module System

| Feature | Description |
|---------|-------------|
| `@NgModule()` | Declares an Angular module |
| `declarations` | Components, directives, pipes |
| `imports` | Other modules to import |
| `exports` | Make components available to other modules |
| `providers` | Services available to module |
| `bootstrap` | Root component to start application |
| `entryComponents` | Components for dynamic creation |

### Module Types

| Type | Description |
|------|-------------|
| Root Module | Main application module (AppModule) |
| Feature Modules | Modules for specific features |
| Shared Modules | Modules shared across features |
| Core Module | Singleton services and app-wide components |
| Lazy-loaded Modules | Modules loaded on demand |

### Standalone Components

| Feature | Description |
|---------|-------------|
| `standalone: true` | Component without NgModule |
| `imports` | Direct imports in component |
| `bootstrapApplication()` | Bootstrap standalone app |
| Standalone Directives | Directives without NgModule |
| Standalone Pipes | Pipes without NgModule |

---

## 10. **HTTP Client**

### HTTP Operations

| Feature | Description |
|---------|-------------|
| `HttpClientModule` | Enable HTTP functionality |
| `HttpClient` | Service for HTTP requests |
| `get()`, `post()`, `put()`, `delete()` | HTTP methods |
| `headers` | HTTP headers configuration |
| `params` | HTTP parameters |
| `observe` | Control response observation |
| `responseType` | Control response type |

### HTTP Interceptors

| Feature | Description |
|---------|-------------|
| `HttpInterceptor` | Intercept HTTP requests/responses |
| `intercept()` | Interceptor method |
| `HTTP_INTERCEPTORS` | Provider token for interceptors |
| Request Transformation | Modify outgoing requests |
| Response Transformation | Modify incoming responses |
| Error Handling | Global error handling |
| Authentication | Add auth tokens to requests |

---

## 11. **Reactive Programming (RxJS)**

### Observables

| Feature | Description |
|---------|-------------|
| `Observable` | Stream of data over time |
| `Subject` | Observable that can be multicasted |
| `BehaviorSubject` | Subject with initial value |
| `ReplaySubject` | Subject that replays values |
| `AsyncSubject` | Subject that emits only last value |
| `subscribe()` | Subscribe to observable |
| `unsubscribe()` | Clean up subscription |

### Operators

| Operator | Description |
|----------|-------------|
| `map()` | Transform emitted values |
| `filter()` | Filter emitted values |
| `mergeMap()`, `switchMap()` | Flatten nested observables |
| `combineLatest()` | Combine multiple observables |
| `tap()` | Perform side effects |
| `catchError()` | Handle errors |
| `takeUntil()` | Unsubscribe based on another observable |
| `debounceTime()` | Delay emissions |
| `distinctUntilChanged()` | Emit only distinct values |

---

## 12. **Signals (Angular 17+)**

### Signal Basics

| Feature | Description |
|---------|-------------|
| `signal()` | Create a signal |
| `computed()` | Create computed signal |
| `effect()` | Create side effect |
| `set()` | Update signal value |
| `update()` | Update signal based on current value |
| `mutate()` | Mutate signal value |
| Signal Inputs | Use signals as component inputs |
| Signal Queries | ViewChild and ContentChild as signals |

---

## 13. **Change Detection**

### Change Detection Strategies

| Strategy | Description |
|----------|-------------|
| `Default` | Check all bindings on every change |
| `OnPush` | Check only when inputs change or events occur |
| `ChangeDetectorRef` | Manually control change detection |
| `detectChanges()` | Trigger change detection manually |
| `markForCheck()` | Mark component for checking |
| `detach()` | Detach from change detection |
| `reattach()` | Reattach to change detection |

### Zone.js

| Feature | Description |
|---------|-------------|
| `NgZone` | Angular's zone for change detection |
| `runOutsideAngular()` | Execute code outside Angular zone |
| `run()` | Execute code inside Angular zone |
| Zone Patching | Automatically patch async operations |
| Manual Zone Control | Control when change detection runs |

---

## 14. **Testing**

### Unit Testing

| Feature | Description |
|---------|-------------|
| `TestBed` | Configure testing module |
| `ComponentFixture` | Wrapper for component testing |
| `DebugElement` | Access to DOM elements |
| `By.css()` | Query DOM elements |
| `spyOn()` | Create spies for testing |
| `fakeAsync()` | Control async testing |
| `tick()` | Simulate passage of time |
| `flush()` | Flush pending timers |

### Testing Utilities

| Feature | Description |
|---------|-------------|
| `HttpClientTestingModule` | Mock HTTP requests |
| `RouterTestingModule` | Mock routing |
| `MockBuilder` | Alternative to TestBed |
| `ngMocks` | Third-party mocking library |
| Shallow Testing | Test component in isolation |
| Integration Testing | Test component with dependencies |

---

## 15. **Performance & Optimization**

### Performance Techniques

| Technique | Description |
|-----------|-------------|
| `OnPush` Change Detection | Optimize change detection |
| `trackBy` Function | Optimize ngFor performance |
| Lazy Loading | Load modules on demand |
| Preloading | Preload modules in background |
| Tree Shaking | Remove unused code |
| Bundle Analysis | Analyze bundle size |
| Service Workers | Enable offline functionality |
| Virtual Scrolling | Handle large lists efficiently |

### Bundle Optimization

| Feature | Description |
|---------|-------------|
| `ng build --prod` | Production build |
| Code Splitting | Split code into chunks |
| Differential Loading | Modern vs legacy bundles |
| Source Maps | Debug production builds |
| Bundle Budgets | Set size limits for bundles |

---

## 16. **Angular CLI**

### CLI Commands

| Command | Description |
|---------|-------------|
| `ng new` | Create new project |
| `ng generate` | Generate components, services, etc. |
| `ng build` | Build application |
| `ng serve` | Serve application |
| `ng test` | Run unit tests |
| `ng e2e` | Run end-to-end tests |
| `ng lint` | Run linting |
| `ng update` | Update dependencies |
| `ng add` | Add packages to project |

### Schematics

| Feature | Description |
|---------|-------------|
| Angular Schematics | Code generation templates |
| Custom Schematics | Create custom generators |
| `ng generate` | Use schematics to generate code |
| Collection | Group of schematics |
| Workspace Configuration | Configure CLI behavior |

---

## 17. **Advanced Features**

### Dynamic Components

| Feature | Description |
|---------|-------------|
| `ComponentFactoryResolver` | Create component factories |
| `ViewContainerRef` | Container for dynamic components |
| `createComponent()` | Create component instance |
| Dynamic Module Loading | Load modules dynamically |
| Portal/Overlay | Advanced dynamic content |

### Internationalization (i18n)

| Feature | Description |
|---------|-------------|
| `@angular/localize` | Internationalization package |
| `i18n` attribute | Mark text for translation |
| `$localize` | Programmatic localization |
| ICU Expressions | Handle pluralization and selection |
| Build-time i18n | Generate localized builds |

### Accessibility

| Feature | Description |
|---------|-------------|
| `@angular/cdk/a11y` | Accessibility utilities |
| `FocusMonitor` | Track focus state |
| `LiveAnnouncer` | Announce messages to screen readers |
| `TrapFocus` | Trap focus within element |
| ARIA Attributes | Proper ARIA labeling |

---

## 18. **Security**

### Security Features

| Feature | Description |
|---------|-------------|
| Sanitization | Automatic HTML sanitization |
| `DomSanitizer` | Manual sanitization control |
| CSP | Content Security Policy support |
| XSS Protection | Cross-site scripting prevention |
| CSRF Protection | Cross-site request forgery prevention |
| `bypassSecurityTrust*` | Bypass sanitization (use carefully) |

---

## 19. **Development Tools**

### Debugging Tools

| Tool | Description |
|------|-------------|
| Angular DevTools | Browser extension for debugging |
| `ng.profiler` | Performance profiling |
| Augury | Legacy debugging tool |
| Source Maps | Debug TypeScript in browser |
| Component Inspector | Inspect component properties |
| Performance Profiler | Analyze change detection |

### Build Tools

| Tool | Description |
|------|-------------|
| Webpack | Module bundler |
| Angular CLI | Command-line interface |
| Nx | Monorepo tool |
| Bazel | Build system (optional) |
| esbuild | Fast JavaScript bundler |

---

## 20. **Environment & Configuration**

### Environment Configuration

| Feature | Description |
|---------|-------------|
| `environment.ts` | Environment-specific settings |
| `fileReplacements` | Replace files per environment |
| `APP_INITIALIZER` | Run code before app starts |
| Configuration Service | Centralized configuration |
| Build Configurations | Different build settings |

### Deployment

| Feature | Description |
|---------|-------------|
| `ng build --prod` | Production build |
| Base Href | Configure base URL |
| Service Worker | Enable PWA features |
| Docker | Containerize Angular apps |
| Static Hosting | Deploy to CDN |

---

## Summary

This comprehensive cheat sheet covers Angular's complete feature set including:

- **Core Concepts**: Components, Directives, Services, Modules
- **Template Features**: Data binding, Events, Pipes, Content projection
- **Forms**: Template-driven and Reactive forms with validation
- **Routing**: Navigation, Guards, Lazy loading
- **HTTP**: Client, Interceptors, Error handling
- **Reactive Programming**: RxJS, Observables, Operators
- **Modern Features**: Signals, Standalone components
- **Performance**: Change detection, Optimization techniques
- **Testing**: Unit testing, Integration testing
- **Development**: CLI, Debugging, Build tools
- **Advanced**: i18n, Accessibility, Security
- **Deployment**: Environment configuration, Production builds

Angular provides a complete framework for building scalable, maintainable web applications with a rich ecosystem of tools and features.