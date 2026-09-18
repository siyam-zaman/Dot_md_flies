# UIU Food & Items Delivery System — Update Log

This document records the chronological history of updates, backend changes, verification results, bug fixes, and phase completions across the project.

> **🎯 Current Collaboration Scope:**
> The active development scope is now focused on the **Shop Owner Role Backend**.
> Historical Student, Runner, Admin, and shared-system entries are preserved for project traceability, but new development entries should follow the Shop Owner backend phase structure defined in `PHASES.md`.

---

## 📝 Update Log

> Existing historical entries remain unchanged above this point.

---

### [2026-09-18] — Shop Owner Backend Scope & Architecture Restructure

* **Domain:** Shop Owner Backend Architecture, Collaboration Scope & Phase Tracking

* **Status:** Shop Owner Architecture Setup ✅ COMPLETED / Phase 2 ⏳ IN_PROGRESS

* **Changes Summary:**

  * **Backend:**

    * Defined the Shop Owner backend as the active implementation scope.
    * Established role-modular Shop Owner controller organization under:

      * `server/controllers/shop/`
    * Established Shop Owner route organization under:

      * `server/routes/shop/`
    * Defined Shop backend domains:

      * Shop Profile
      * Menu Management
      * Incoming Orders
      * Order Preparation
      * Dashboard Analytics
      * Reviews
      * Sales & Reports
      * Order Chat
    * Defined strict Shop ownership validation requirements for:

      * Menu items
      * Orders
      * Reviews
      * Transactions
      * Reports
      * Chat threads
    * Defined backend-first development policy:

      * Frontend files may be inspected as API-contract references.
      * Frontend implementation must not be modified unless explicitly requested.
    * Defined Shop-controlled Order lifecycle:

      * `PLACED`
      * `CONFIRMED`
      * `PREPARING`
      * `READY_FOR_PICKUP`
      * `REJECTED`
    * Defined backend validation and authorization rules for all Shop Owner endpoints.
    * Confirmed that backend operations must use MongoDB persistence and must not rely on mock success responses.

  * **Frontend Integration Impact:**

    * No frontend files modified.
    * Existing Shop Owner frontend pages are treated as backend API contract references only.

* **Files Modified/Created:**

  * `PHASES.md`
  * `AGENTS.md`
  * `FEATURES_MAP.md`
  * `UPDATE_LOG.md`

* **Verification:**

  * Documentation architecture reviewed against existing Shop Owner frontend routes and backend structure.
  * Existing Shop Menu APIs confirmed as the starting backend implementation area.
  * No runtime code changed in this task.

---

### [2026-09-18] — Shop Owner Menu Management Backend Audit

* **Domain:** Shop Menu Management

* **Status:** Phase 2 ⏳ IN_PROGRESS

* **Changes Summary:**

  * **Backend:**

    * Audited existing Shop Owner Menu Management implementation.
    * Confirmed existing menu-related backend functionality:

      * `GET /api/shops/my-shop`
      * `POST /api/shops/menu`
      * `PUT /api/shops/menu/:id`
      * `PATCH /api/shops/menu/:id/availability`
      * `DELETE /api/shops/menu/:id`
      * `PUT /api/shops/profile`
    * Confirmed existing Shop menu functionality currently resides in:

      * `server/controllers/shopController.js`
      * `server/routes/shopRoutes.js`
    * Identified future modularization target:

      * `server/controllers/shop/shopMenuController.js`
      * `server/controllers/shop/shopProfileController.js`
      * `server/routes/shop/shopRoutes.js`
    * Confirmed `MenuItem` currently supports:

      * `stockQuantity`
      * `taxRate`
      * `discount`
      * `todaySpecial`
      * `featured`
      * `recommended`
    * Identified validation improvements required for `POST /api/shops/menu`:

      * Validate required item name.
      * Validate numeric price.
      * Reject price values `<= 0`.
      * Reject negative stock.
      * Restrict discount to `0–100`.
      * Reject negative tax values.
      * Reject negative low-stock threshold.
    * Identified unsafe numeric fallback pattern such as:

      * `Number(stockQuantity) || 50`
    * Defined replacement behavior so valid `0` values are preserved.
    * Confirmed Shop lookup must fail explicitly when the authenticated Shop Owner has no Shop record instead of silently creating a Shop.
    * Identified menu image storage improvement:

      * Current Base64 image handling should eventually be replaced with Multer + Cloudinary.
      * MongoDB should store the Cloudinary URL rather than large Base64 strings.

  * **Frontend Integration Impact:**

    * `ShopAddMenuItem.jsx` currently serves as the request-contract reference for `POST /api/shops/menu`.
    * `ShopMenuManagement.jsx` serves as the reference for menu retrieval, update, availability toggle, and deletion.
    * No frontend files modified during the backend audit.

* **Files Modified/Created:**

  * `PHASES.md`
  * `FEATURES_MAP.md`
  * `AGENTS.md`
  * `UPDATE_LOG.md`

* **Verification:**

  * Existing historical verification confirms:

    * `POST /api/shops/menu` → `201 Created`
    * `PATCH /api/shops/menu/:id/availability` → `200 OK`
    * `PUT /api/shops/menu/:id` → `200 OK`
    * `DELETE /api/shops/menu/:id` → `200 OK`
    * `PUT /api/shops/profile` → `200 OK`
  * These results were previously recorded in the project update history and were not re-executed during this documentation audit.

---

### [2026-09-18] — Shop Owner Phase 2A Planning: Add New Menu Item Backend

* **Domain:** Shop Menu / Add New Item

* **Status:** Phase 2A ⏳ IN_PROGRESS

* **Changes Summary:**

  * **Backend:**

    * Selected `POST /api/shops/menu` as the first Shop Owner backend endpoint to finalize.

    * Defined expected backend flow:

      ```text
      Request
         ↓
      JWT Authentication
         ↓
      Shop Role Authorization
         ↓
      Find Shop owned by authenticated user
         ↓
      Validate request body
         ↓
      Create MenuItem
         ↓
      Persist to MongoDB
         ↓
      Return 201 Created
      ```

    * Defined required validation rules:

      * Item name must not be empty.
      * Price must be numeric.
      * Price must be greater than `0`.
      * Stock must not be negative.
      * Discount must remain between `0` and `100`.
      * Tax rate must not be negative.
      * Low-stock threshold must not be negative.

    * Defined ownership rule:

      * Created Menu Item must always be associated with the Shop owned by `req.user._id`.

    * Defined API security requirements:

      * Missing/invalid JWT → unauthorized.
      * Non-Shop role → forbidden.
      * Missing Shop → not found.

    * Defined future Phase 2B image-upload architecture:

      * Multer
      * Cloudinary
      * Store returned URL in `MenuItem.image`

  * **Frontend Integration Impact:**

    * No frontend implementation changes required for Phase 2A planning.
    * Existing `ShopAddMenuItem.jsx` request body remains the reference contract.

* **Files Modified/Created:**

  * `PHASES.md`
  * `FEATURES_MAP.md`
  * `AGENTS.md`
  * `UPDATE_LOG.md`

* **Verification:**

  * No runtime code changed in this planning task.
  * Implementation verification will be recorded after `POST /api/shops/menu` is finalized and tested.

---

## 📌 Shop Owner Update Log Rules Going Forward

Every new Shop Owner backend task must append a new entry using this structure:

```markdown
### [YYYY-MM-DD] — <Feature / Task Title> (Phase X Status)

- **Domain:** <Shop backend domain>

- **Status:** <Phase X IN_PROGRESS / COMPLETED>

- **Changes Summary:**

  - **Backend:**
    - <Controller changes>
    - <Route changes>
    - <Model changes>
    - <Middleware changes>
    - <Validation/business logic changes>

  - **Frontend Integration Impact:**
    - <Frontend API-contract impact>
    - Or `None`

- **Files Modified/Created:**
  - `<file path>`

- **Verification:**
  - <API tests>
  - <HTTP response codes>
  - <Database verification>
  - <Runtime verification>
```

---

## 🎯 Current Active Development State

```text
Phase 1 — Shop Authentication & Profile
Status: Existing foundation / partially completed

Phase 2 — Menu Management Backend
Status: IN_PROGRESS
```

Current task sequence:

```text
2A — Add New Menu Item
        ↓
2B — Menu Image Upload
        ↓
2C — Edit Menu Item
        ↓
2D — Availability Toggle
        ↓
2E — Delete Menu Item
        ↓
2F — Stock & Low-Stock Management
        ↓
Phase 2 Complete
```

Immediate implementation target:

```http
POST /api/shops/menu
```
