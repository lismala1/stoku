# Anticipated design risks

## Risk 1: Low-stock checks spread everywhere (owner: Eris)
_To be written by Eris: what goes wrong, why v1 has it on purpose, planned fix (Observer)._

## Risk 2: Stock changes cannot be undone (owner: Mal)
**What goes wrong.** In v1 a stock movement is applied directly and then only described in text.
Shipping an order does

```js
product.quantity -= qty;
log.record(`Shipped ${qty} × ${sku}`);
```

and `MovementLog` keeps nothing but a list of strings.

**Why this is a risk.** Warehouse staff make mistakes: a delivery typed in twice, an order shipped
with the wrong quantity. With v1 there is no way to reverse a movement:

- **The log cannot undo anything.** A string like "Shipped 5 × TAPE-01" says what happened but holds
  no behaviour. To undo it we would have to parse the text back into a SKU and quantity and know
  which operation is the opposite of "Shipped". That breaks as soon as the wording changes.
- **The knowledge of how to reverse a movement is scattered.** If we added undo in v1, the
  controller would need a growing `if/else` on the movement type ("if it was a shipment, add the
  stock back; if it was a delivery, remove it..."). Every new movement type means editing that
  chain, which breaks the Open/Closed Principle.
- **No reliable history.** The log and the real stock can drift apart, because changing the
  quantity and writing the log line are two separate steps.

**Why v1 has it on purpose.** Changing a number and writing a line of text is the simplest thing that
works for the first version. It makes the missing undo obvious and gives us real code to refactor.

**Planned fix: Command.** Each movement becomes an object implementing `StockCommand` with
`execute()`, `undo()` and `describe()`: `ReceiveDeliveryCommand` and `ShipOrderCommand` each store
their `sku` and `qty`. The controller no longer changes stock; it creates a command and passes it to
`MovementHistory.run(cmd)`, which executes it and pushes it on a stack. `undoLast()` pops the top
command and calls its `undo()`. The history list is built from `describe()`, so the record and the
stock change always come from the same object. New movement types are new command classes, with no
change to `MovementHistory`.

**How we will show it.** Before: `src/history/MovementLog.js` on the `feat/v1-history` branch, a
list of strings with no undo. After: the `refactor/command` PR with `src/commands/` and a working
Undo button, demonstrated live.
