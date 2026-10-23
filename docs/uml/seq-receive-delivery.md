# Sequence diagram – receive a delivery

Owner: Lis Mala · SWEN-383 Part 1

**Scenario.** After the shipment in `seq-ship-order.md`, `TAPE-01` has 7 in stock (minimum 10), so
the low-stock warning is showing. A delivery of 10 rolls arrives. Staff record it, stock goes up to
17, and no warning is shown.

This flow shows **Command** (the delivery is a command run through the history, so it can be undone)
and **Observer** (the inventory notifies its observers, and only `LowStockAlert` decides whether to
warn) when stock goes **up**. Method names match `class-diagram.md`.

```mermaid
sequenceDiagram
  autonumber
  actor Staff
  participant View as :InventoryView
  participant Ctrl as :StokuController
  participant Cmd as cmd:ReceiveDeliveryCommand
  participant Hist as :MovementHistory
  participant Inv as :Inventory
  participant Alert as :LowStockAlert

  Staff->>View: enter SKU "TAPE-01", qty 10, click Receive
  View->>Ctrl: receiveDelivery("TAPE-01", 10)
  Ctrl->>Cmd: new ReceiveDeliveryCommand(inventory, "TAPE-01", 10)
  Ctrl->>Hist: run(cmd)
  Hist->>Cmd: execute()
  Cmd->>Inv: addStock("TAPE-01", 10)
  Inv->>Inv: product.quantity = 7 + 10 = 17
  Inv->>Inv: notify(product)
  loop for each subscribed StockObserver
    Inv->>Alert: onStockChanged(product)
    alt product.isLowStock()
      Alert->>View: showWarning(product)
    else stock is fine (17 >= 10)
      Alert-->>Inv: nothing to do
    end
  end
  Inv-->>Cmd: done
  Cmd-->>Hist: done
  Hist->>Hist: push(cmd) onto done stack
  Hist-->>Ctrl: done
  Ctrl->>View: render(products, containers, history)
  View-->>Staff: TAPE-01 shows 17, no warning, delivery in history
```

## What to notice

- **Same flow as shipping, opposite direction.** The controller builds a command and hands it to
  `MovementHistory.run()`. Only the command class and `addStock` instead of `removeStock` change.
  That is what makes the Command pattern easy to extend.
- **The warning goes away by itself.** `render()` redraws the table from current stock, and
  `LowStockAlert` only warns while `isLowStock()` is true, so no "clear warning" code is needed.
- **The delivery can be undone.** `ReceiveDeliveryCommand.undo()` calls `removeStock("TAPE-01", 10)`,
  and stock would go back to 7 with the warning showing again (see `seq-undo.md`).
