# Real-Time Inventory Tracker — Spec-Driven Development Pipeline

This document walks the feature through every SDD stage in order: PRD → Functional/Technical Specs → User Stories → Acceptance Criteria → Technical Prompt. The working demo built from this pipeline is in `inventory-tracker.html`.

---

## 1. PRD (Product Requirements Document)

**Problem statement**
Store managers no longer trust the numbers in the inventory system. Stock counts drift out of sync with reality, so managers fall back to manual counts — which is slow, duplicates effort, and defeats the purpose of having a system at all.

**Goal**
Ship a demoable, real-time-feeling inventory tracker that a store manager can use to see current stock at a glance, add new items, and adjust stock levels, with every change reflected immediately and visibly logged — so trust in the numbers is rebuilt through transparency, not just accuracy.

**Target users**
- Store / warehouse managers (primary) — need a fast, trustworthy read on stock.
- Stockroom staff (secondary) — need to log stock movements without friction.

**Non-goals for this demo**
- No multi-user concurrency/sync, no auth, no backend/database. This is a clickable, stateful demo for client sign-off, not the production build.
- No barcode scanning, no supplier/purchase-order integration, no multi-location transfer logic.

**Success signals**
- A client stakeholder can, unassisted, add an item, adjust its stock, and see the change reflected with a timestamp and an entry in an activity log within one demo session.
- Stock status (in stock / low / out) is understandable at a glance without explanation.

**Constraints**
- Single HTML page, responsive (desktop + mobile).
- State persists across a page reload for demo continuity (no backend required).

---

## 2. Functional & Technical Specs

**Functional scope**
1. **Stock table** — name, SKU, category, quantity, reorder threshold, status badge (In Stock / Low Stock / Out of Stock), last-updated timestamp.
2. **Add item form** — name, SKU, category, starting quantity, reorder threshold. Validates required fields and duplicate SKUs.
3. **Update stock panel** — pick an existing item, apply a signed quantity delta (+/-) with a reason (Restock / Sale / Correction / Damage), or set an absolute new quantity.
4. **Activity log** — reverse-chronological feed of the last N stock events (item, change, reason, timestamp), so every number change is auditable.
5. **Search / filter** — filter table by name/SKU and by category; sort by column.
6. **Status computation** — derived, not stored: `quantity === 0` → Out of Stock; `quantity <= reorderThreshold` → Low Stock; else → In Stock.

**Technical scope**
- Plain HTML/CSS/JS, no build step, single file.
- State held in memory (JS) and mirrored to the artifact's persistent key-value storage (`window.storage`, personal/non-shared) so a reload doesn't wipe the demo data.
- No external network calls; no PII collected (names/SKUs of inventory items only — no customer or employee personal data).
- Responsive breakpoints: single-column stacked layout under ~720px; table becomes horizontally scrollable on narrow viewports rather than clipping.

**Out of scope (explicitly deferred)**
- Real backend, multi-device sync, authentication, role permissions, undo/redo beyond the activity log.

---

## 3. User Stories

- **US-1**: As a store manager, I want to see all current stock levels in one table, so I can trust what's on the shelf without doing a manual count.
- **US-2**: As a stockroom clerk, I want to add a new item to the system, so newly received products are trackable immediately.
- **US-3**: As a stockroom clerk, I want to adjust an existing item's quantity (restock, sale, correction, damage), so the system reflects reality after every stock movement.
- **US-4**: As a store manager, I want low-stock and out-of-stock items visually flagged, so I know what needs reordering without reading every row.
- **US-5**: As a store manager, I want a log of recent stock changes, so I can see *why* a number changed, not just that it did — which is what rebuilds trust.
- **US-6**: As a user on a phone in the stockroom, I want the tracker to work on a small screen, so I'm not tied to a desktop to log a change.

---

## 4. Acceptance Criteria

**US-1 — View stock**
- Given items exist, when the page loads, then every item's name, SKU, category, quantity, threshold, status, and last-updated time are visible in a table.
- Given no items exist, then an empty state explains how to add the first one (not a blank table).

**US-2 — Add item**
- Given the add-item form, when required fields (name, SKU, quantity, reorder threshold) are filled and submitted, then the item appears in the table immediately with status computed correctly.
- Given a SKU that already exists, when submitted, then the form rejects it with a clear inline message instead of creating a duplicate.
- Given quantity or threshold is negative or non-numeric, then submission is blocked with a clear inline message.

**US-3 — Update stock**
- Given an existing item is selected in the update panel, when a positive or negative delta is applied, then the table quantity updates immediately and cannot go below zero.
- Given a delta is applied, when submitted, then an entry is added to the activity log with item, delta, reason, and timestamp.

**US-4 — Status flags**
- Given quantity is 0, then status badge reads "Out of Stock" (danger color).
- Given 0 < quantity ≤ reorder threshold, then status badge reads "Low Stock" (warning color).
- Given quantity > reorder threshold, then status badge reads "In Stock" (success color).

**US-5 — Activity log**
- Given any add or update action, then a new log entry appears at the top of the activity feed without a page reload.
- Given more than the visible limit of entries, then older entries remain accessible (scrollable), not silently discarded.

**US-6 — Responsive**
- Given a viewport under ~720px wide, then the form panel stacks above/below the table (not side-by-side) and the table scrolls horizontally instead of overflowing the page.

---

## 5. Technical Prompt

*(This is the prompt handed to the code-generation step — the artifact this produces is `inventory-tracker.html`.)*

> **Role**: You are a front-end developer building a demoable, single-file real-time inventory tracker for a client-facing demo. Design for trust: every number must be traceable to a logged action.
>
> **Build**: One responsive HTML file (inline CSS + vanilla JS, no frameworks, no build step) implementing:
> 1. A stock table (name, SKU, category, qty, reorder threshold, computed status badge, last-updated) with client-side search and category filter.
> 2. An "Add item" form with validation (required fields, no duplicate SKU, non-negative numeric quantity/threshold).
> 3. An "Update stock" panel: select an item, apply a signed delta with a required reason (Restock/Sale/Correction/Damage), quantity floors at 0.
> 4. An activity log: reverse-chronological, scrollable, shows item + delta + reason + timestamp for every add/update.
> 5. Status derivation: 0 → Out of Stock (danger); ≤ threshold → Low Stock (warning); else → In Stock (success). Never store status directly — always derive it.
> 6. State persists across reloads using the environment's `window.storage` key-value API (personal scope), with try/catch on every call and a sensible fallback to in-memory state if storage fails.
> 7. Mobile-responsive: side panel stacks above the table under ~720px; table scrolls horizontally rather than clipping columns.
>
> **Constraints**: No external network calls. No customer/employee personal data fields — inventory item data only. Comment the code for handoff to another developer. Use tabular figures for the quantity column so digits align.

---

## 6. Code

See `inventory-tracker.html` — the working, demoable build from the technical prompt above.

---

## 7. Amendment (v1.1) — Multi-location, low-stock banner, CSV export

Client feedback after the v1 demo asked for three additions. Folding them into the pipeline rather than bolting them on:

**Updated user stories**
- **US-7**: As a store manager overseeing several locations, I want each item tagged to a location, so I can tell whether a shortage is store-wide or specific to one site.
- **US-8**: As a manager, I want a single glanceable summary of everything that needs reordering across all locations, so I don't have to scan the whole table to find what's low or out.
- **US-9**: As a manager, I want to export the activity log to CSV, so I can share it outside the tool (email, spreadsheet, audit trail).

**Updated acceptance criteria**
- Given an item is added, when the location field is left blank, then submission is blocked (location is now required alongside name/SKU/category).
- Given the location filter is set, then the table and the "update stock" item picker both respect it.
- Given one or more items are Low Stock or Out of Stock (in any location), then a banner appears above the table naming the affected items and their location; given none are, the banner is absent (no empty-state noise).
- Given the activity log has at least one entry, when "Export log as CSV" is clicked, then a `.csv` file downloads with timestamp, event, and detail columns; given the log is empty, then the user is told there's nothing to export instead of downloading an empty file.

**Technical notes**
- `location` is a required string field on each item (same free-text pattern as `category` — no fixed location list, since the client hasn't confirmed their store list yet).
- The low-stock banner is derived at render time from the full `items` array (not the filtered view), so it always reflects the true cross-location picture even while the table is filtered down to one location.
- CSV export builds the file client-side via `Blob` + an object URL — no server round-trip, consistent with the demo's no-backend constraint.
