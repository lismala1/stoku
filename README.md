# Stoku – Warehouse Inventory System

Stoku is a small warehouse inventory app written in plain JavaScript (ES modules in the browser,
no framework and no build step). Staff can add products, record deliveries in and orders out, pack
products into boxes and pallets, get a warning when a product drops below its minimum level, and
undo a mistaken stock movement. It covers one warehouse with one user role. Suppliers, invoicing,
logins and multiple warehouses are out of scope.

Course project for **SWEN-383 Software Design Principles and Patterns**, RIT Kosovo, Fall 2026.

## Team

| Member | Owns | Pattern |
| --- | --- | --- |
| Lis Mala (lead) | Repo, app shell, controller, UI, slide deck | – |
| Jetik Gjonbalaj | Products, boxes, pallets | Composite |
| Eris Visoka | Inventory and low-stock alerts | Observer |
| Mal Gerbeshi | Stock movements, history, undo | Command |

## How to run

Stoku uses plain ES modules, so it has to be **served over HTTP** — opening `index.html` as a file
(`file://`) will not work, the browser blocks module loading.

Either:

- **VS Code Live Server extension** — right-click `index.html` → *Open with Live Server*, or
- **npx** — run `npx serve .` in this folder and open the URL it prints.

## Final folder structure

This is the target layout after the refactors, with the owner of each file:

```
index.html, styles.css, README.md, src/main.js,
src/StokuController.js, src/ui/InventoryView.js        Lis
src/model/StockItem.js, Product.js, Container.js       Jetik
src/inventory/Inventory.js,
src/alerts/StockObserver.js, LowStockAlert.js          Eris
src/commands/StockCommand.js, ReceiveDeliveryCommand.js,
ShipOrderCommand.js, MovementHistory.js                Mal
docs/proposal.md                                       Lis
docs/uml/class-diagram.md                              Jetik
docs/uml/seq-ship-order.md                             Eris
docs/uml/seq-undo.md                                   Mal
docs/risks.md                                          Eris + Mal
```

Version 1 uses simpler files first: `src/model/Box.js` (Jetik, no nesting) and
`src/history/MovementLog.js` (Mal, text-only log, no undo). These are replaced during the
refactors by the files listed above.

## Team rules

1. Never push straight to `main`. Make your own branch from `main`, push it, and open a Pull Request.
2. Only touch your own files. If you need something from someone else's file, ask them or open an Issue.
3. Every task is a GitHub Issue. Commit messages say what changed and end with `Fixes #n` when they close one.
4. Lis reviews and merges the PRs. Someone else reviews Lis's PRs, so nobody merges their own work.
5. JavaScript has no interfaces, so `StockItem`, `StockObserver` and `StockCommand` are base classes whose methods throw `"not implemented"`. The UML still shows them as interfaces.
6. UML diagrams are written in Mermaid inside `.md` files, so GitHub draws them and they can be edited like code.

## Branches

| Member | Part 1 branch | v1 code branch | Refactor branch |
| --- | --- | --- | --- |
| Lis | `docs/proposal` | `feat/v1-app-shell` | `refactor/controller` |
| Jetik | `docs/class-diagram` | `feat/v1-items` | `refactor/composite` |
| Eris | `docs/seq-ship-order` | `feat/v1-inventory` | `refactor/observer` |
| Mal | `docs/seq-undo` | `feat/v1-history` | `refactor/command` |

## Quick Git workflow

```bash
git checkout main
git pull
git checkout -b docs/class-diagram   # your own branch
# ...edit your files...
git add .
git commit -m "Add draft class diagram. Fixes #3"
git push -u origin docs/class-diagram
# then open a Pull Request on GitHub
```
