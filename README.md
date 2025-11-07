# Angular Signals (Part II - The Signals Revenge)

> Angular Singals Workshops to explore the code benefits of the new reactive path

```js
 Soon in the best cinemas!
```

---

## Candidates for signal migration demonstration

> Isolated and self-contained

1- Pagination Gallery Component (it's a gallery component that demonstrates page controls (pagination))
eq2-ui/src/app/components/ui-gallery/pagination/eq2-page-controls-gallery.component.ts
http://localhost:4200/#/pagination

2- Select Gallery Component
eq2-ui/src/app/components/ui-gallery/select/eq2-select-gallery.component.ts


## Benefits:
.subscription() -> toSignal() = for automatic cleanup
Use computed() for derived pagination values
remove manual ngOnDestroy subscription management
Automatic reactivity: currentPageNum or perPage change
It demonstrates computed signals for derived state
Simplify initialisation logic


## The migration would demonstrate:
Before: Manual Subscription management, manual calculations
After: Automatic cleanup, computed signals, cleaner code

---

## BEFORE / AFTER

1- Pagination Gallery Component
eq2-ui/src/app/components/ui-gallery/pagination/eq2-page-controls-gallery.component.ts

### Issues:
- Manual subscription management (Subscription, ngOnDestroy)
- anual calculation of derived values (totalPages, recordFrom, recordTo, etc.)
- No automatic reactivity when currentPageNum or perPage change
- Error handling mixed with success logic
- Manual state updates in multiple places

```js
// RUN: Library Development
yarn run lib-watch

// RUN: start app
yarn run start

http://localhost:4200/
https://cc-uipreview.azurewebsites.net/#/pagination
```

---

## Signals in Action: `Eque2PageControlsGallerySignalsComponent`

## Table of Contents

1. [Writable Signals](#1-writable-signals)
2. [Computed Signals](#2-computed-signals)
3. [Effects](#3-effects)
4. [Observable Interoperability](#4-observable-interoperability)
5. [Signal Update Methods](#5-signal-update-methods)
6. [Template Usage](#6-template-usage)
7. [Key Benefits](#7-key-benefits)

---

## 1. Writable Signals

**What are they?**
Writable signals are the foundation of reactive state in Angular. They hold a value and notify Angular when that value changes.

**Syntax:**
```typescript
// Create a writable signal
const count = signal(0);

// Read the value
const currentValue = count();  // Returns 0

// Update the value
count.set(5);                  // Direct assignment
count.update(value => value + 1);  // Update based on current value
```

**In our component:**
```typescript
currentPageNum = signal(1);
perPage = signal(20);
```

**Key Points:**
- ✅ Signals are functions - call them with `()` to read
- ✅ `.set(value)` for direct updates
- ✅ `.update(fn)` for updates based on current value
- ✅ Angular automatically tracks signal reads for change detection
- ✅ Type-safe - TypeScript infers types from initial value

---

## 2. Computed Signals

**What are they?**
Computed signals derive their value from other signals. They automatically recalculate when their dependencies change.

**Syntax:**
```typescript
const a = signal(5);
const b = signal(10);

// Computed signal that depends on a and b
const sum = computed(() => a() + b());  // Returns 15

// When a or b changes, sum automatically recalculates
a.set(7);  // sum() now returns 17 automatically!
```

**In our component:**
```typescript
// Simple computed
totalPages = computed(() => 
  Math.ceil(this.totalRecordsCount() / this.perPage())
);

// Complex computed with multiple dependencies
paginationInfo = computed(() => {
  const from = this.recordFrom();
  const to = this.recordTo();
  const total = this.totalRecordsCount();
  return `Showing ${from}-${to} of ${total} records`;
});
```

**Key Points:**
- ✅ Read-only - cannot use `.set()` or `.update()`
- ✅ Memoized - only recalculates when dependencies change
- ✅ Lazy - only computed when accessed
- ✅ Can depend on other computed signals
- ✅ Automatically tracks dependencies

**Performance Benefits:**
```typescript
// This computed only recalculates when currentPageNum or perPage change
recordFrom = computed(() => 
  (this.currentPageNum() - 1) * this.perPage() + 1
);

// If you call recordFrom() 100 times without changing dependencies,
// it only calculates once!
```

---

## 3. Effects

**What are they?**
Effects run side effects when signals change. They're perfect for logging, analytics, or triggering other actions.

**Syntax:**
```typescript
effect(() => {
  // This runs whenever any signal read inside changes
  console.log('Signal changed!');
});
```

**In our component:**
```typescript
constructor() {
  // Example: Log pagination changes
  effect(() => {
    const page = this.currentPageNum();
    const perPage = this.perPage();
    console.log(`[Signal Effect] Pagination changed: page=${page}, perPage=${perPage}`);
  });
  
  // Example: Log when data loads
  effect(() => {
    const result = this.dataResult();
    if (result) {
      console.log(`[Signal Effect] Data loaded: ${result.products?.length || 0} products`);
    }
  });
}
```

**Key Points:**
- ✅ Automatically tracks which signals are read
- ✅ Runs when any tracked signal changes
- ✅ Must run in injection context (constructor or injection function)
- ✅ Use `{ allowSignalWrites: true }` to write signals inside effects
- ✅ Automatically cleaned up when component is destroyed

**Common Use Cases:**
- Logging and debugging
- Analytics tracking
- Syncing state to external systems
- Triggering side effects based on state changes

**⚠️ Important:**
```typescript
// ❌ DON'T: Create infinite loops
effect(() => {
  this.currentPageNum.set(this.currentPageNum() + 1);  // Infinite loop!
});

// ✅ DO: Use allowSignalWrites flag if needed
effect(() => {
  if (someCondition) {
    this.currentPageNum.set(1);  // Only with allowSignalWrites: true
  }
}, { allowSignalWrites: true });
```

---

## 4. Observable Interoperability

**The Problem:**
Angular services often return Observables, but we want to use Signals. How do we bridge them?

**The Solution:**
Angular provides `toSignal()` and `toObservable()` to convert between the two.

### Converting Observables to Signals

**Pattern:**
```typescript
import { toSignal } from '@angular/core/rxjs-interop';

// Convert a single observable
const data$ = this.service.getData();
const data = toSignal(data$, { initialValue: null });
```

**In our component:**
```typescript
// Convert signals to observables, combine them, then convert back to signal
private dataResult = toSignal(
  combineLatest([
    toObservable(this.currentPageNum),
    toObservable(this.perPage)
  ]).pipe(
    switchMap(([page, perPage]) => 
      this.dataFetcherService.getData(page, perPage).pipe(
        catchError((error) => {
          console.error('Error fetching data:', error);
          return of(null);
        })
      )
    )
  ),
  { initialValue: null }
);
```

**Benefits:**
- ✅ Automatic cleanup - no `ngOnDestroy` needed!
- ✅ Automatic reactivity - updates when source signals change
- ✅ Type-safe with `initialValue` option
- ✅ Works seamlessly with RxJS operators

### Converting Signals to Observables

**Pattern:**
```typescript
import { toObservable } from '@angular/core/rxjs-interop';

const pageNum$ = toObservable(this.currentPageNum);
// Now you can use RxJS operators
pageNum$.subscribe(page => console.log(page));
```

**Use Case:**
When you need to use RxJS operators or integrate with existing Observable-based code.

---

## 5. Signal Update Methods

There are three ways to update writable signals:

### 1. `.set(value)` - Direct Assignment

```typescript
currentPageNum.set(5);  // Sets to 5
```

**Use when:** You have a new value ready.

### 2. `.update(fn)` - Update Based on Current Value

```typescript
currentPageNum.update(page => page + 1);  // Increment
currentPageNum.update(page => Math.max(1, page - 1));  // Decrement with min
```

**Use when:** You need the current value to calculate the new value.

### 3. `.mutate(fn)` - Mutate in Place

```typescript
const items = signal([1, 2, 3]);
items.mutate(arr => arr.push(4));  // Mutates the array
```

**⚠️ Use sparingly!** Prefer immutable updates:
```typescript
// ✅ Better: Immutable update
items.update(arr => [...arr, 4]);

// ⚠️ Only use mutate for performance-critical cases
items.mutate(arr => arr.push(4));
```

**In our component:**
```typescript
setPageSetup(event: { currentPage: number; perPage: number }): void {
  // Method 1: Direct set
  this.currentPageNum.set(event.currentPage);
  this.perPage.set(event.perPage);
  
  // Note: No need to call getData() - the toSignal() + combineLatest()
  // automatically triggers when currentPageNum or perPage change!
}

goToNextPage(): void {
  // Using update() to increment based on current value
  this.currentPageNum.update(page => page + 1);
}

goToPreviousPage(): void {
  // Using update() with conditional logic
  this.currentPageNum.update(page => Math.max(1, page - 1));
}

changePerPage(newPerPage: number): void {
  // Using set() for direct assignment
  this.perPage.set(newPerPage);
  // Reset to first page when changing items per page
  this.currentPageNum.set(1);
}
```

---

## 6. Template Usage

### Reading Signals

**Important:** Always call signals with `()` in templates!

```html
<!-- ✅ Correct -->
<div>{{ currentPageNum() }}</div>
<button [disabled]="isLoading()">Submit</button>

<!-- ❌ Wrong -->
<div>{{ currentPageNum }}</div>  <!-- Won't work! -->
```

### Control Flow

**New Angular Control Flow (Angular 17+):**
```html
<!-- ✅ New syntax with signals -->
@if (isLoading()) {
  <div>Loading...</div>
} @else {
  <div>Content</div>
}

@for (product of products(); track product.id) {
  <div>{{ product.name }}</div>
}
```

**Old syntax (still works, but not recommended):**
```html
<!-- ⚠️ Old syntax -->
<div *ngIf="isLoading()">Loading...</div>
<div *ngFor="let product of products()">{{ product.name }}</div>
```

### Property Bindings

```html
<!-- ✅ Signal bindings -->
<eq2-page-controls
  [currentPageNum]="currentPageNum()"
  [totalPages]="totalPages()"
></eq2-page-controls>
```

---

## 7. Key Benefits

### ✅ Automatic Change Detection
- Angular tracks signal reads automatically
- Only components reading changed signals re-render
- More granular than zone-based change detection

### ✅ No Manual Cleanup
- `toSignal()` handles subscription cleanup automatically
- No `ngOnDestroy` needed for subscriptions
- Less boilerplate code

### ✅ Type Safety
- Signals are strongly typed
- TypeScript infers types from signal values
- Better IDE autocomplete

### ✅ Performance
- Computed signals are memoized
- Only recalculates when dependencies change
- Granular change detection reduces unnecessary renders

### ✅ Simpler Code
- No `async` pipe needed in templates
- No manual subscription management
- Reactive by default
- Less boilerplate

### ✅ Better Developer Experience
- Clear data flow
- Easy to reason about
- Less code to maintain
- Better debugging with Angular DevTools

---

## Comparison: Before vs After

### Before (Traditional RxJS)
```typescript
export class Component implements OnInit, OnDestroy {
  private sub = new Subscription();
  products: any[] = [];
  currentPageNum = 1;
  totalPages: number;
  
  ngOnInit() {
    this.sub.add(
      this.service.getData(this.currentPageNum).subscribe(data => {
        this.products = data.products;
        this.totalPages = Math.ceil(data.total / this.perPage);
      })
    );
  }
  
  ngOnDestroy() {
    this.sub.unsubscribe();
  }
}
```

### After (Signals)
```typescript
export class Component {
  currentPageNum = signal(1);
  
  private dataResult = toSignal(
    toObservable(this.currentPageNum).pipe(
      switchMap(page => this.service.getData(page))
    )
  );
  
  products = computed(() => this.dataResult()?.products ?? []);
  totalPages = computed(() => 
    Math.ceil(this.dataResult()?.total / this.perPage())
  );
  
  // No ngOnInit, no ngOnDestroy needed!
}
```

**Lines of code:** 97 → 70 (28% reduction)
**Complexity:** High → Low
**Maintainability:** Medium → High

---

## Best Practices

1. **Use signals for component state**
   ```typescript
   // ✅ Good
   count = signal(0);
   
   // ❌ Avoid
   count = 0;  // Not reactive
   ```

2. **Use computed for derived state**
   ```typescript
   // ✅ Good
   total = computed(() => this.price() * this.quantity());
   
   // ❌ Avoid
   total = this.price * this.quantity;  // Won't update automatically
   ```

3. **Use toSignal() for observables**
   ```typescript
   // ✅ Good
   const data = toSignal(this.service.getData());
   
   // ❌ Avoid (unless necessary)
   this.service.getData().subscribe(data => {
     this.data = data;  // Manual subscription management
   });
   ```

4. **Always call signals with () in templates**
   ```html
   <!-- ✅ Good -->
   <div>{{ count() }}</div>
   
   <!-- ❌ Wrong -->
   <div>{{ count }}</div>
   ```

5. **Prefer immutable updates**
   ```typescript
   // ✅ Good
   items.update(arr => [...arr, newItem]);
   
   // ⚠️ Only if needed for performance
   items.mutate(arr => arr.push(newItem));
   ```

---

## Common Patterns

### Pattern 1: Loading State
```typescript
const data = toSignal(this.service.getData(), { initialValue: null });
const isLoading = computed(() => data() === null);
```

### Pattern 2: Error Handling
```typescript
const dataResult = toSignal(
  this.service.getData().pipe(
    catchError(error => {
      console.error(error);
      return of(null);
    })
  ),
  { initialValue: null }
);

const hasError = computed(() => dataResult() === null);
```

### Pattern 3: Filtered Lists
```typescript
const searchTerm = signal('');
const allItems = signal<Item[]>([]);

const filteredItems = computed(() => 
  allItems().filter(item => 
    item.name.toLowerCase().includes(searchTerm().toLowerCase())
  )
);
```

### Pattern 4: Form State
```typescript
const formData = signal({ name: '', email: '' });
const isFormValid = computed(() => 
  formData().name.length > 0 && formData().email.includes('@')
);
```

---

## Resources

- [Angular Signals Documentation](https://angular.dev/guide/signals)
- [Angular Signals RFC](https://github.com/angular/angular/discussions/49685)
- [RxJS Interop](https://angular.dev/guide/signals/rxjs-interop)

---

## Summary

Signals provide a modern, reactive way to manage state in Angular:

- **Writable signals** for state that changes
- **Computed signals** for derived state
- **Effects** for side effects
- **toSignal/toObservable** for interoperability
- **Automatic cleanup** and reactivity
- **Better performance** and developer experience


The pagination component demonstrates all these concepts in a Eque2 real-world scenario




