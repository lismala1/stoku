# Stoku – project proposal (draft)

SWEN-383 Software Design Principles and Patterns · RIT Kosovo · Fall 2026
Part 1 deliverable, draft. Owner: Lis Mala.

## Project name

**Stoku** – a warehouse inventory system.

## Problem

Small warehouses track their stock in spreadsheets or on paper. Three things go wrong again and
again:

- They lose track of what is packed where, because a box holds products and a pallet holds boxes,
  and none of that is written down anywhere useful.
- They notice too late that a product has run low, because nobody checks the minimum level until
  someone needs the product and it is already gone.
- They cannot reverse a mistake. A delivery typed in twice, or an order shipped with the wrong
  quantity, has to be corrected by hand and the record of what actually happened is lost.

## Description

Stoku is a small warehouse inventory app written in plain JavaScript (ES modules in the browser,
no framework and no build step). Warehouse staff can:

- add products, each with a quantity and a minimum level;
- record deliveries in and orders out;
- pack products into boxes and boxes onto pallets;
- see a warning when a product drops below its minimum level;
- undo a mistaken stock movement.

It serves one warehouse with one user role.

## Scope – in

- Products with quantity and minimum level
- Deliveries in (stock increases)
- Orders out (stock decreases)
- Packing products into boxes, and boxes into pallets
- Low-stock warnings when a product falls below its minimum level
- A history of stock movements, and undo of the last movement

## Scope – out

- Suppliers
- Invoicing
- Logins and user accounts
- Multiple warehouses

## Patterns we plan to apply

- **Composite** – a box holds products, a pallet holds boxes, and both have to be treated the same
  way when counting or displaying contents, so containers and products share one `StockItem` type.
- **Observer** – low-stock warnings should be raised by whoever is interested in stock changes, not
  re-checked inside every method that moves stock, so `Inventory` notifies observers and
  `LowStockAlert` reacts.
- **Command** – undo needs each stock movement to be an object that knows how to apply and reverse
  itself, so `ReceiveDeliveryCommand` and `ShipOrderCommand` implement `execute()`/`undo()` and
  `MovementHistory` keeps them on a stack.

## Approach

Version 1 is written the simple, naive way on purpose. The low-stock check is copied into every
method that changes stock and `Inventory` calls the view directly; shipping subtracts the quantity
in place and appends a line of text to a log; `Box` holds only products and cannot nest. This makes
two design risks really happen in the code instead of staying hypothetical.

The team then refactors with Observer, Command and Composite and presents real before/after
evidence from the repository: the duplicated checks that disappear, the undo that becomes possible,
and the nesting that the Composite allows. The UML handed in for Part 1 shows the *planned* design;
the v1 code is the *starting point* we refactor away from.

## Team and roles

| Member | Owns | Pattern |
| --- | --- | --- |
| Lis Mala (lead) | Repo, app shell, controller, UI, slide deck | – |
| Jetik Gjonbalaj | Products, boxes, pallets | Composite |
| Eris Visoka | Inventory and low-stock alerts | Observer |
| Mal Gerbeshi | Stock movements, history, undo | Command |
