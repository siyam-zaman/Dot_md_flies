# Agent Guidelines & Repository Workflow

This file defines the mandatory rules, architecture constraints, and development workflow for all AI coding agents working on the **UIU Food & Items Delivery System — Shop Owner Backend**.

---

## 📌 Core Operational Directives

### 1. Maintain & Update `PHASES.md`

* **Location:** [`PHASES.md`](PHASES.md) in the repository root.

* **Rule:** Before starting any development task, always inspect `PHASES.md` to determine:

  * The currently active Shop Owner phase.
  * Which backend deliverables are already completed.
  * Which deliverables remain pending.
  * Which phase the requested task belongs to.

* **Rule:** Whenever beginning or completing a deliverable:

  * Update the relevant checkbox using `[x]`.
  * Update the phase status:

    * `COMPLETED`
    * `IN_PROGRESS`
    * `PENDING`
  * Update the `Last Updated` date.

* **Scope Notice:**
  This repository's current phase tracking is specifically focused on the **Shop Owner Role Backend**.

* Do not modify Student, Runner, or Admin phase status unless explicitly requested.

---

## 2. Maintain & Update `UPDATE_LOG.md`

* **Location:** [`UPDATE_LOG.md`](UPDATE_LOG.md) in the repository root.

* **Rule:** After every user-requested backend task, bug fix, API modification, database change, validation improvement, or phase completion, append a new entry to `UPDATE_LOG.md`.

* Never overwrite previous log entries.

### Entry Structure

```markdown
### [YYYY-MM-DD] — <Feature / Task Title> (Phase X Status)

- **Domain:** <Shop backend area, e.g., Shop Menu / Shop Orders / Shop Profile>

- **Status:** <e.g., Phase 2 IN_PROGRESS / Phase 2 COMPLETED>

- **Changes Summary:**
  - **Backend:**
    - <Endpoints created or modified>
    - <Controllers created or modified>
    - <Models modified>
    - <Middleware or validation changes>

  - **Frontend Integration Impact:**
    - <Frontend page/API contract affected, if applicable>
    - <Write "None" if no frontend changes were required>

- **Files Modified/Created:**
  - <File path 1>
  - <File path 2>

- **Verification:**
  - <Commands executed>
  - <API response status codes>
  - <Database verification>
  - <Test/build results>
```

---

# 3. Backend-First Scope Rule

The current development scope is the **Shop Owner Backend**.

AI agents must prioritize:

```text
Controllers
Routes
MongoDB Models
Middleware
Services
Validation
Authentication
Authorization
Database Queries
Business Logic
API Responses
Error Handling
```

Frontend files may be inspected to understand required API behavior, but frontend implementation should not be changed unless explicitly requested.

For example:

```text
ShopAddMenuItem.jsx
        ↓
Used as API requirement reference
        ↓
POST /api/shops/menu
        ↓
Backend implementation
```

The frontend should act as the specification for what data the backend must provide or accept.

---

# 4. Role-Modular Architecture

All Shop Owner-specific backend logic must follow a role-modular architecture.

Preferred structure:

```text
server/
│
├── controllers/
│   └── shop/
│       ├── shopProfileController.js
│       ├── shopMenuController.js
│       ├── shopOrderController.js
│       ├── shopDashboardController.js
│       ├── shopReviewController.js
│       ├── shopReportController.js
│       └── shopChatController.js
│
├── routes/
│   └── shop/
│       └── shopRoutes.js
│
├── models/
│   ├── User.js
│   ├── Shop.js
│   ├── MenuItem.js
│   ├── Order.js
│   ├── Transaction.js
│   └── OrderChat.js
│
├── middlewares/
│   ├── auth.js
│   ├── role.js
│   ├── errorHandler.js
│   └── upload.js
│
└── services/
    ├── orderService.js
    ├── shopAnalyticsService.js
    └── uploadService.js
```

Do not place Shop Owner business logic inside Student, Runner, or Admin controller directories.

---

# 5. Strict Shop Ownership Validation

Every Shop Owner-specific operation must verify that the requested resource belongs to the currently authenticated Shop Owner.

Example:

```js
const shop = await Shop.findOne({
  owner: req.user._id
});
```

For Shop-related resources such as `MenuItem`:

```js
const menuItem = await MenuItem.findOne({
  _id: req.params.itemId,
  shop: shop._id
});
```

Never allow a Shop Owner to manipulate another Shop's:

```text
Menu Items
Orders
Reviews
Reports
Transactions
Chat Threads
Profile Information
```

Ownership validation must happen in the backend.

Never rely on frontend route protection alone.

---

# 6. Strict Role Security

All protected Shop Owner routes must verify authentication and authorization.

Use the existing authentication middleware from:

```text
server/middlewares/
```

Expected pattern:

```js
router.post(
  "/menu",
  protect,
  authorizeRoles("shop", "admin"),
  addMenuItem
);
```

At minimum, protected Shop endpoints must verify:

```text
Valid JWT
    ↓
Authenticated User
    ↓
Correct Role
    ↓
Associated Shop
    ↓
Requested Resource Ownership
```

Never trust role information sent from the frontend.

For example, this must never determine authorization:

```json
{
  "role": "shop"
}
```

Authorization must come from the authenticated backend user.

---

# 7. Never Trust Frontend Data

Frontend values must always be validated by the backend.

For example, when creating a Menu Item:

```json
{
  "name": "Chicken Burger",
  "price": 150,
  "stockQuantity": 20
}
```

The backend must independently validate:

```text
name exists
price is numeric
price > 0
stockQuantity >= 0
discount is between 0–100
taxRate >= 0
```

Never assume React validation is sufficient.

---

# 8. MongoDB Model Validation

Important business rules should be protected at both levels:

```text
Controller Validation
        +
Mongoose Schema Validation
```

Example:

```js
price: {
  type: Number,
  required: true,
  min: 1
}
```

Example:

```js
stockQuantity: {
  type: Number,
  default: 0,
  min: 0
}
```

Example:

```js
discount: {
  type: Number,
  default: 0,
  min: 0,
  max: 100
}
```

Controllers should return readable validation errors rather than exposing raw Mongoose errors whenever possible.

---

# 9. No Mock Backend Bypasses

Do not implement backend behavior using:

```text
Static JSON
Hardcoded Successful Responses
Fake Database Results
Temporary In-Memory Objects
Silent API Failure Fallbacks
```

Shop operations must persist through the actual backend and MongoDB.

Incorrect:

```js
return res.json({
  success: true,
  menuItem: fakeMenuItem
});
```

Correct:

```text
Validate Request
        ↓
MongoDB Operation
        ↓
Return Actual Database Result
```

Mock data may remain in frontend development files temporarily, but it must not be used to simulate completed backend functionality.

---

# 10. API Response Consistency

Shop APIs should use a consistent response structure.

Successful response:

```json
{
  "success": true,
  "message": "Menu item created successfully",
  "data": {}
}
```

Failed response:

```json
{
  "success": false,
  "message": "Menu item not found"
}
```

Use appropriate HTTP status codes.

Examples:

```text
200 OK
201 Created
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

Do not return `200` for failed operations.

---

# 11. Shop Menu Management Rules

Menu-related backend work belongs to the Shop Menu phase.

Expected operations include:

```http
GET    /api/shops/menu
POST   /api/shops/menu
PUT    /api/shops/menu/:itemId
PATCH  /api/shops/menu/:itemId/availability
DELETE /api/shops/menu/:itemId
```

Every Menu Item must be linked to a Shop:

```js
shop: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "Shop",
  required: true
}
```

Shop Owners must only be able to manage items belonging to their own Shop.

---

# 12. Menu Item Image Rules

Do not permanently store large Base64 image strings in MongoDB.

Avoid:

```text
data:image/jpeg;base64,/9j/4AAQSk...
```

Preferred architecture:

```text
Image File
   ↓
Multer
   ↓
Cloudinary
   ↓
Image URL
   ↓
MongoDB
```

MongoDB should store:

```js
image: "https://res.cloudinary.com/..."
```

When an image is replaced or deleted, remove the corresponding old Cloudinary asset where appropriate.

---

# 13. Shop Order Lifecycle Rules

Shop Owners may only perform valid Order state transitions.

Expected Shop-controlled lifecycle:

```text
PLACED
   ↓
CONFIRMED
   ↓
PREPARING
   ↓
READY_FOR_PICKUP
```

Alternative from `PLACED`:

```text
PLACED
   ↓
REJECTED
```

Do not allow invalid transitions such as:

```text
PLACED
   ↓
READY_FOR_PICKUP
```

or:

```text
DELIVERED
   ↓
PREPARING
```

Backend validation must enforce valid transitions regardless of frontend behavior.

---

# 14. Shop Order Ownership Rules

Shop Owners may only retrieve and modify Orders where:

```js
order.shop.toString() === shop._id.toString()
```

Preferred query pattern:

```js
const order = await Order.findOne({
  _id: req.params.orderId,
  shop: shop._id
});
```

This is safer than retrieving the Order first and checking ownership afterward.

---

# 15. Financial Data Rules

Never calculate Shop earnings from arbitrary frontend values.

Financial values must originate from trusted backend data.

An Order may include:

```text
subtotal
discount
deliveryFee
platformFee
grandTotal
shopAmount
runnerAmount
```

Shop reporting should use:

```text
shopAmount
```

for Shop revenue when the platform fee and runner share are separated.

Never assume:

```text
grandTotal = shop revenue
```

---

# 16. No Silent Shop Creation

Menu or Order controllers must not automatically create a missing Shop.

Avoid:

```js
if (!shop) {
  shop = await Shop.create(...);
}
```

Instead:

```js
if (!shop) {
  return res.status(404).json({
    success: false,
    message: "Shop not found"
  });
}
```

Shop registration and Shop creation must be handled by their designated authentication/onboarding flow.

---

# 17. Avoid Duplicate Sources of Truth

Do not maintain duplicate fields representing the same business data unless intentionally required.

For example, avoid maintaining Shop revenue independently in multiple locations unless synchronization is explicitly designed.

Prefer a clear source of truth:

```text
Orders
Transa
```
