# Anticipated design risks

## Risk 1: Low-stock checks spread everywhere (owner: Eris)
**What goes wrong.** Every method that changes stock has to check whether the product has dropped
below its minimum level and, if so, show a warning. In v1 `Inventory` does this with the same line
copied into `addProduct`, `addStock` and `removeStock`:

```js
if (product.quantity < product.minLevel) {
  this.view.showWarning(`Low stock: ${product.name} ...`);
}
```

and `Inventory` holds a direct reference to the view to do it.

**Why this is a risk.** It causes two problems we expect to hit as the app grows:

- **Shotgun Surgery / duplicated code.** Changing the rule (for example "warn at or below the
  minimum", or "warn only once") means finding and editing every copy. Any new method that moves
  stock (such as stock adjustments) must remember to copy the check, and if someone forgets, that
  movement silently produces no warning.
- **Tight coupling.** `Inventory` is model code but depends on the UI. It cannot be tested or reused
  without a view, and adding a second reaction (an email to the manager, a reorder list) means
  editing `Inventory` again, which breaks the Open/Closed Principle.

**Why v1 has it on purpose.** It is the quickest way to make warnings work, and it is what a team
without patterns would naturally write. Building it this way first gives us real code to point at
when we show the before/after.

**Planned fix: Observer.** `Inventory` becomes the subject. It keeps a list of `StockObserver`s,
offers `subscribe(observer)`, and calls `notify(product)` once after any stock change. The low-stock
rule moves into one class, `LowStockAlert implements StockObserver`, which decides whether to call
`view.showWarning(product)`. `Inventory` no longer knows about the view. A new reaction is a new
observer class with no change to `Inventory`.

**How we will show it.** Before: the duplicated `if` in each method of `src/inventory/Inventory.js`
on the `feat/v1-inventory` branch. After: the `refactor/observer` PR, where the copies disappear and
`src/alerts/LowStockAlert.js` holds the only check.


## Risk 2: Stock changes cannot be undone (owner: Mal)
_To be written by Mal: what goes wrong, why v1 has it on purpose, planned fix (Command)._
