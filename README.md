# Angular Signals (Part II - The Signals Revenge)

> Angular Singals Workshops to explore the code benefits of the new reactive path

```js
 Soon in the best cinemas!
```

---

## Candidates for signal migration demonstration

> Isolated and self-contained

1- Eque2PageControlsComponent - Production ready reusable UI components

2- Eque2PageControlsComponent - Real-world scenario Component to demostrate how to use the real Eque2PageControlsComponent

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



