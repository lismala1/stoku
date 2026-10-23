# Sequence diagram – pack products into a box and the box onto a pallet

Owner: Lis Mala · SWEN-383 Part 1

**Scenario.** Staff pack `TAPE-01` (12 in stock) and `LABEL-02` (20 in stock) into a box called
`Box-01`, then put the box on a pallet called `Pallet-A`. The pallet shows 32 units in total.

This flow shows the **Composite** pattern: `Product` and `Container` are both `StockItem`s, so the
same `add()` call packs a product into a box and a box onto a pallet, and `getQuantity()` adds up the
whole tree by itself. Method names match `class-diagram.md`.

```mermaid
sequenceDiagram
  autonumber
  actor Staff
  participant View as :InventoryView
  participant Ctrl as :StokuController
  participant Bx as box:Container
  participant Pallet as pallet:Container
  participant P1 as tape:Product
  participant P2 as label:Product

  Staff->>View: pack TAPE-01 into "Box-01"
  View->>Ctrl: packIntoContainer("Box-01", tape)
  Ctrl->>Bx: new Container("Box-01")
  Ctrl->>Bx: add(tape)

  Staff->>View: pack LABEL-02 into "Box-01"
  View->>Ctrl: packIntoContainer("Box-01", label)
  Ctrl->>Bx: add(label)

  Staff->>View: put Box-01 on "Pallet-A"
  View->>Ctrl: packIntoContainer("Pallet-A", box)
  Ctrl->>Pallet: new Container("Pallet-A")
  Ctrl->>Pallet: add(box)

  Ctrl->>View: render(products, containers, history)
  View->>Pallet: getQuantity()
  Pallet->>Bx: getQuantity()
  Bx->>P1: getQuantity()
  P1-->>Bx: 12
  Bx->>P2: getQuantity()
  P2-->>Bx: 20
  Bx-->>Pallet: 32
  Pallet-->>View: 32
  View-->>Staff: Pallet-A shows 32 units
```

## What to notice

- **One method for every kind of packing.** `packIntoContainer(label, item)` takes any `StockItem`,
  so packing a product into a box and a box onto a pallet is the same call. A new `Container` is only
  created the first time a label is used.
- **The view never asks what is inside.** It calls `getQuantity()` on the pallet exactly as it would
  on one product. The pallet asks its children, the box asks its children, and the total comes back
  up. That is the point of Composite.
- **Stock and packing stay in sync.** The box holds the same `Product` objects as the inventory, so
  shipping 5 × TAPE-01 later would make the pallet show 27 without any extra code.
- **In v1 none of this works.** `Box` can only hold products and cannot go on a pallet.
