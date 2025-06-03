# 🧩 Procedural Cabinet Modeling System – Specification Prompt

## 🎯 Purpose

This document defines a **procedural cabinet modeling system** implemented using:
- **React**
- **Tailwind CSS**
- **Three.js**
- **React Three Fiber (R3F)**

The system enables users to create, edit, and visualize modular cabinet structures in 3D space. It supports dynamic layout behavior, nested item structure, configurable item properties, and intelligent merging logic.

---

## 📐 BaseRow – The Main Horizontal Layout Container

- `BaseRow` is the **main horizontal layout container** for all items.
- All boxes are aligned from **left to right**.
- Each box has its origin at **back-bottom-left**, ensuring consistent alignment along the floor plane.
- The height of the entire row (`BaseRow.height`) determines the height of all full-height items unless otherwise specified.

---

## 🧱 Box Types

There are two main types of boxes:

### ✅ TallItem
- A standalone full-height cabinet unit.
- Has no internal structure.
- Can be placed directly into `BaseRow`.
- Height syncs with `BaseRow.height` unless manually changed via UI.
- Supports variety (e.g., TallDrawer2, TallShelves5).
- Supports material assignment (e.g., WoodLight, MetalMatte).

### 🔹 VerticalBox
- A structured container that holds three sub-rows:
  - UpperRow: aligned at **back-top-left**
  - OffsetRow: vertical filler between upper and lower
  - LowerRow: aligned at **back-bottom-left**
- Automatically created when adding an `UpperItem` or `LowerItem`, if last box is not already a `VerticalBox`.
- Width equals the **maximum width among its rows**.
- Its height is determined by the sum of the heights of its sub-rows.
- Only changes to `TallItem` or `OffsetRow` height trigger global height updates.
- If two `VerticalBox` instances become adjacent due to deletion of a `TallItem`, they must **merge into one**.

---

## 🧩 Nested Item Types Inside VerticalBox

These items can only exist inside a `VerticalBox`.

| Type       | Name         | Description |
|------------|--------------|-------------|
| ✅         | UpperItem    | Lives inside `UpperRow`. Supports variety and material. |
| ✅         | LowerItem    | Lives inside `LowerRow`. Supports variety and material. |

> These items must live inside a `VerticalBox`.

---

## 🔄 Height Behavior – Final Clarified Version

Only specific actions affect the global height (`BaseRow.height`):

| Action                                | Triggers Global Height Sync? | Notes |
|-------------------------------------|------------------------------|-------|
| `TallItem.height` changed via slider | ✅ Yes                        | Updates BaseRow height |
| `OffsetRow.height` changed via slider| ✅ Yes                        | Updates BaseRow height |
| `UpperItem/LowerItem.height` changed via slider | ❌ No              | Only affects local row height |
| `UpperRow/LowerRow.height` auto-updated | ❌ No                      | Auto-adjusts `OffsetRow` only |
| `OffsetRow.height` auto-updated by rows | ❌ No                       | No global effect |

When a global sync occurs:
- New `BaseRow.height = UpperRow.height + OffsetRow.height + LowerRow.height`
- All other items (`TallItems`, `VerticalBoxes`) update their height accordingly.

---

## 🚨 Merge Logic – VerticalBox Merging on TallItem Deletion

### Rule:
> You should never have two `VerticalBox` instances next to each other without a `TallItem` in between.

### Trigger:
- Deleting a `TallItem` that sits **between two `VerticalBox` instances**.

### What Happens:
1. The two adjacent `VerticalBox` instances are merged into one.
2. The new merged `VerticalBox`:
   - Inherits the position of the first `VerticalBox`
   - Combines all items from both `UpperRow`s and `LowerRow`s
   - Recalculates width as max of combined sub-row widths
   - Keeps `OffsetRow` height consistent unless updated manually
3. All remaining items in the `BaseRow` re-calculate their `position.x` to maintain correct horizontal layout.

---

## 📏 Dynamic Positioning & Width Propagation

### When any item's **width** changes:
- Parent row width updates → parent `VerticalBox` width updates → downstream boxes shift left/right to fill gap or avoid overlap.

### When any item is deleted:
- Remaining boxes shift left to fill the space.
- May trigger `VerticalBox` merge logic.

---

## 🎨 Material System – Material Picker

Each item has material and it can be changed. 
