# Stoku – class diagram (draft, planned design)

Owner: Jetik Gjonbalaj · SWEN-383 Part 1

This is the **planned** design we refactor towards during the semester. Version 1 of the code is
deliberately simpler (see `docs/risks.md`): it has no `StockObserver`, no commands and a `Box` that
cannot nest. The three patterns are marked in the notes below the diagram.

```mermaid
classDiagram
  direction TB

  class StokuController {
    -inventory: Inventory
    -history: MovementHistory
    -view: InventoryView
    +addProduct(sku, name, qty, minLevel, unitPrice)
    +receiveDelivery(sku, qty)
    +shipOrder(sku, qty)
    +packIntoContainer(label, item)
    +undoLast()
  }

  class InventoryView {
    +render(products, containers, history)
    +showWarning(product)
    +showMessage(text)
  }

  class Inventory {
    -products: Map~string, Product~
    -observers: List~StockObserver~
    +addProduct(p: Product)
    +getProduct(sku) Product
    +addStock(sku, qty)
    +removeStock(sku, qty)
    +subscribe(o: StockObserver)
    -notify(p: Product)
  }

  class StockObserver {
    <<interface>>
    +onStockChanged(p: Product)
  }

  class LowStockAlert {
    -view: InventoryView
    +onStockChanged(p: Product)
  }

  class StockCommand {
    <<interface>>
    +execute()
    +undo()
    +describe() string
  }

  class ReceiveDeliveryCommand {
    -inventory: Inventory
    -sku: string
    -qty: number
    +execute()
    +undo()
    +describe() string
  }

  class ShipOrderCommand {
    -inventory: Inventory
    -sku: string
    -qty: number
    +execute()
    +undo()
    +describe() string
  }

  class MovementHistory {
    -done: Stack~StockCommand~
    +run(c: StockCommand)
    +undoLast() boolean
    +getEntries() List~string~
  }

  class StockItem {
    <<interface>>
    +getQuantity() number
    +getValue() number
  }

  class Product {
    -sku: string
    -name: string
    -quantity: number
    -minLevel: number
    -unitPrice: number
    +getQuantity() number
    +getValue() number
    +isLowStock() boolean
  }

  class Container {
    -label: string
    -children: List~StockItem~
    +add(i: StockItem)
    +remove(i: StockItem)
    +getQuantity() number
    +getValue() number
  }

  StokuController --> Inventory
  StokuController --> MovementHistory
  StokuController --> InventoryView

  MovementHistory o-- "0..*" StockCommand
  StockCommand <|.. ReceiveDeliveryCommand
  StockCommand <|.. ShipOrderCommand
  ReceiveDeliveryCommand --> Inventory
  ShipOrderCommand --> Inventory

  Inventory --> "0..*" StockObserver : notifies
  StockObserver <|.. LowStockAlert
  LowStockAlert --> InventoryView

  Inventory *-- "0..*" Product
  StockItem <|.. Product
  StockItem <|.. Container
  Container o-- "0..*" StockItem
```

## Patterns in the diagram

| Pattern | Classes | Why |
| --- | --- | --- |
| **Composite** | `StockItem` (component), `Product` (leaf), `Container` (composite) | A box holds products, a pallet holds boxes. `getQuantity()` and `getValue()` work the same on one product or a whole pallet, so the rest of the app never asks "is this a box or a product?". |
| **Observer** | `Inventory` (subject), `StockObserver`, `LowStockAlert` | `Inventory` tells every subscribed observer when stock changes. The low-stock check lives in one place, and a new alert (e.g. email) is one new class with no change to `Inventory`. Fixes Risk 1. |
| **Command** | `StockCommand`, `ReceiveDeliveryCommand`, `ShipOrderCommand`, `MovementHistory` | Every stock movement is an object that can `execute()` and `undo()` itself. `MovementHistory` keeps them on a stack, which gives both the history list and undo. Fixes Risk 2. |
| **GRASP Controller** | `StokuController` | Receives every UI event and delegates, so the view has no business logic. Same idea as `TodoController` in the labs. |

## Relationship choices

- **`Inventory *-- Product` is composition.** A product record only exists inside the inventory; if
  the inventory is removed, its products go with it.
- **`Container o-- StockItem` is aggregation.** A product can be taken out of a box and still exist,
  and a box can be taken off a pallet. The parts live on without the container.
- **`MovementHistory o-- StockCommand` is aggregation.** The history holds commands but does not
  create them; the controller creates them and hands them over.
- **Interfaces in JavaScript.** JavaScript has no `interface` keyword, so `StockItem`,
  `StockObserver` and `StockCommand` are base classes whose methods throw `"not implemented"`. The
  UML still shows them as interfaces because that is their design role.
