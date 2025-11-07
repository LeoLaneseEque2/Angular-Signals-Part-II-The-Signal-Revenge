# Angular Signals (Part II - The Signals Revenge)

> Angular Singals Workshops to explore the code benefits of the new reactive path

```js
 Soon in the best cinemas! WIP
```

---

## Candidates for signal migration demonstration

> Isolated and self-contained

1- Eque2PageControlsComponent - Production ready reusable UI components
eq2-ui/projects/eq2/ui-core/src/lib/components/ — reusable UI components

2- Eque2PageControlsGalleryComponent - Real-world scenario Component to demostrate how to use the real Eque2PageControlsComponent
eq2-ui/src/app/components/ui-gallery/ — these are demo/showcase examples


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



