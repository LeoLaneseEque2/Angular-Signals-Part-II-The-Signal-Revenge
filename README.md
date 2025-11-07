# Angular Signals (Part II - The Signals Revenge) - [WIP]

> Angular Singals Workshops to explore the code benefits of the new reactive path

```js
 Soon in the best cinemas! WIP
```

---

## Candidates for signal migration demonstration

> Isolated and self-contained

WIP: (/eq2=page-controls/eq2-page-controls-signals.component.ts) <br>
1- Eque2PageControlsComponent - Production ready reusable UI components<br>
eq2-ui/projects/eq2/ui-core/src/lib/components/ — reusable UI components<br>

WIP: (/pagination/eq2-page-controls-gallery-signals.component.ts)<br>
2- Eque2PageControlsGalleryComponent - Real-world scenario Component to demostrate how to use the real Eque2PageControlsComponent<br>
eq2-ui/src/app/components/ui-gallery/ — these are demo/showcase examples<br>


## Benefits:
- .subscription() -> toSignal() = for automatic cleanup
- Use computed() for derived pagination values
- remove manual ngOnDestroy subscription management
- Automatic reactivity: currentPageNum or perPage change
- It demonstrates computed signals for derived state
- Simplify initialisation logic
- Pure signals — no Observable interop (except minimal Promise conversion)
- Automatic reactivity — effect runs when signals change
- Automatic cleanup — effects are cleaned up automatically
- Simpler code — no RxJS operators or Observable chains
- Type-safe — all signals are strongly typed


## The migration would demonstrate:
Before: Manual Subscription management, manual calculations
After: Automatic cleanup, computed signals, cleaner code

---

STASH:
eq2-ui - develop
stash@{0}: On develop: Signals migration: Pure signal-based pagination component

git stash apply 



