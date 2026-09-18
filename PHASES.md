# UIU Food & Items Delivery System — Shop Owner Role Engineering & Integration Phases

> **🤝 4-Member Collaboration Scope Note:**
> This repository is collaboratively engineered by 4 team members across four dedicated roles (**Admin**, **Shop Owner**, **Delivery Runner**, and **Ordering Student**).
> This phase tracking document is exclusively dedicated to the **Shop Owner Role**, tracking all Shop Owner-specific backend controllers, API endpoints, MongoDB models, business logic, and frontend integration deliverables.

---

## 📊 Shop Owner Role Phase Status Overview

|    Phase    | Description                                                 | Scope / Domain                                                       |           Status           | Last Updated |
| :---------: | :---------------------------------------------------------- | :------------------------------------------------------------------- | :------------------------: | :----------: |
| **Phase 1** | **Shop Authentication, Authorization & Profile Management** | Shop Login, JWT Protection, Shop Ownership, Profile Management       | 🟡 **PARTIALLY COMPLETED** |  2026-09-18  |
| **Phase 2** | **Menu Management & Item Creation**                         | Add Item, Edit Item, Delete Item, Availability, Stock                |      ⏳ **IN_PROGRESS**     |  2026-09-18  |
| **Phase 3** | **Incoming Order Management**                               | Receive Orders, Order Details, Accept / Reject Orders                |        ⏳ **PENDING**       |       —      |
| **Phase 4** | **Food Preparation & Ready-for-Pickup Workflow**            | Preparing Queue, Order Status Updates, Pickup Readiness              |        ⏳ **PENDING**       |       —      |
| **Phase 5** | **Shop Dashboard & Operational Analytics**                  | Daily Orders, Revenue, Pending Orders, Popular Items                 |        ⏳ **PENDING**       |       —      |
| **Phase 6** | **Customer Reviews & Ratings Management**                   | Ratings, Reviews, Food Feedback, Reputation                          |        ⏳ **PENDING**       |       —      |
| **Phase 7** | **Sales Reports, Earnings & Transaction History**           | Revenue Reports, Sales Analytics, Earnings, Transaction Ledger       |        ⏳ **PENDING**       |       —      |
| **Phase 8** | **Shop Order Chat & Customer Communication**                | Order-Specific Messaging, Student Communication, Runner Coordination |        ⏳ **PENDING**       |       —      |

---

# 🚀 Detailed Shop Owner Phase Breakdown

---

## Phase 1: Shop Authentication, Authorization & Profile Management

* **Status:** 🟡 **PARTIALLY COMPLETED**

### Target Frontend Pages

* Shop Login / Shared Login Page
* Shop Dashboard Layout
* Shop Profile Page
* Shop Settings / Profile Editing Interface

### Backend Responsibilities

The Shop Owner must first be authenticated before accessing any shop-specific functionality.

The backend must identify:

```text
Logged-in User
      ↓
User Role = shop
      ↓
Associated Shop
      ↓
Shop-specific data
```

### Backend Endpoints

```http
POST /api/auth/login
GET  /api/auth/me

GET  /api/shops/my-shop
PUT  /api/shops/profile
```

### Authentication Flow

```text
Shop Owner Login
       ↓
Email + Password
       ↓
Backend validates credentials
       ↓
JWT generated
       ↓
Frontend stores JWT
       ↓
JWT sent with protected API requests
```

Example:

```http
Authorization: Bearer <JWT_TOKEN>
```

### Backend Middleware

Protected Shop Owner endpoints should use:

```js
protect
```

and:

```js
authorizeRoles("shop", "admin")
```

Example:

```js
router.get(
  "/my-shop",
  protect,
  authorizeRoles("shop", "admin"),
  getMyShop
);
```

### MongoDB Models Involved

```text
User
Shop
```

Relationship:

```text
User
 │
 │ owner
 ↓
Shop
```

Example:

```js
Shop.findOne({
  owner: req.user._id
});
```

### Deliverables

* [x] JWT-based authentication.
* [x] Password hashing using bcrypt.
* [x] Protected Shop routes.
* [x] Shop role authorization.
* [x] Shop Owner → Shop ownership relationship.
* [x] Fetch current Shop profile.
* [x] Basic Shop profile editing.
* [ ] Finalize profile validation.
* [ ] Finalize Shop profile image upload.
* [ ] Remove any temporary or mock Shop information.
* [ ] Properly handle cases where a Shop account has no Shop document.

---

# Phase 2: Menu Management & Item Creation

* **Status:** ⏳ **IN_PROGRESS**

This will be the **first Shop Owner feature we fully complete**.

---

## Target Frontend Pages

* `ShopMenuManagement.jsx`
* `ShopAddMenuItem.jsx`
* Menu Item Edit interface
* Menu Item Availability controls

---

## Core Features

Shop owners must be able to:

```text
View Menu
Add Item
Edit Item
Delete Item
Enable / Disable Item
Manage Stock
Manage Price
Manage Discount
Upload Food Image
```

---

## Backend Endpoints

```http
GET    /api/shops/menu

POST   /api/shops/menu

PUT    /api/shops/menu/:itemId

DELETE /api/shops/menu/:itemId

PATCH  /api/shops/menu/:itemId/availability
```

If the existing frontend continues using:

```http
GET /api/shops/my-shop
```

to retrieve both:

```text
Shop
+
Menu Items
```

that endpoint can remain temporarily.

---

## MongoDB Models Involved

```text
Shop
MenuItem
```

Relationship:

```text
Shop
 │
 │ shop
 ↓
MenuItem
```

Every `MenuItem` must belong to exactly one Shop.

Example:

```js
{
  shop: ObjectId,
  name: String,
  description: String,
  category: String,
  price: Number,
  image: String,
  preparationTime: String,
  stockQuantity: Number,
  lowStockWarning: Number,
  discount: Number,
  taxRate: Number,
  isAvailable: Boolean,
  todaySpecial: Boolean,
  featured: Boolean,
  recommended: Boolean,
  dietary: [String]
}
```

---

## Phase 2A — Add New Menu Item

### Endpoint

```http
POST /api/shops/menu
```

### Flow

```text
ShopAddMenuItem.jsx
        ↓
Form Validation
        ↓
POST /api/shops/menu
        ↓
JWT Authentication
        ↓
Shop Authorization
        ↓
Find Shop by Owner ID
        ↓
Validate Menu Item
        ↓
MenuItem.create()
        ↓
MongoDB
        ↓
Return Created Item
        ↓
Menu Management Page
```

### Fields

Required:

```text
Item Name
Price
Category
```

Optional:

```text
Description
Image
Preparation Time
Stock Quantity
Low Stock Warning
Discount
Tax Rate
Dietary Tags
Today's Special
Featured
Recommended
Availability
```

### Validation Rules

* [ ] Item name cannot be empty.
* [ ] Price must be greater than `0`.
* [ ] Stock cannot be negative.
* [ ] Discount must be between `0–100`.
* [ ] Tax rate cannot be negative.
* [ ] Low-stock threshold cannot be negative.
* [ ] Shop Owner can only add items to their own Shop.

---

## Phase 2B — Food Image Upload

Temporary version:

```text
Default Image URL
```

Final version:

```text
Select Image
     ↓
Multer
     ↓
Cloudinary
     ↓
Cloudinary URL
     ↓
MongoDB
```

MongoDB should store:

```text
https://res.cloudinary.com/.../burger.jpg
```

instead of storing Base64 image strings.

### Deliverables

* [ ] Configure Multer.
* [ ] Connect Cloudinary.
* [ ] Validate file type.
* [ ] Validate file size.
* [ ] Save Cloudinary URL.
* [ ] Delete old Cloudinary image when replacing item image.

---

## Phase 2C — Edit Menu Item

### Endpoint

```http
PUT /api/shops/menu/:itemId
```

The backend must verify:

```text
MenuItem exists
       ↓
MenuItem belongs to logged-in Shop
       ↓
Update allowed fields
```

### Deliverables

* [ ] Edit item name.
* [ ] Edit description.
* [ ] Edit category.
* [ ] Edit price.
* [ ] Edit preparation time.
* [ ] Edit stock.
* [ ] Edit discount.
* [ ] Replace item image.
* [ ] Update dietary tags.

---

## Phase 2D — Availability Management

### Endpoint

```http
PATCH /api/shops/menu/:itemId/availability
```

Example:

```text
Chicken Burger

Available ✅
     ↓

Unavailable ❌
```

Student frontend must not allow unavailable items to be ordered.

### Deliverables

* [ ] Availability toggle.
* [ ] Save availability to MongoDB.
* [ ] Reflect status immediately on Menu Management page.
* [ ] Hide or disable unavailable items from Student ordering flow.

---

## Phase 2E — Delete Menu Item

### Endpoint

```http
DELETE /api/shops/menu/:itemId
```

### Security Requirement

The Shop Owner must not be able to delete another Shop's menu item.

Required ownership check:

```text
MenuItem.shop === loggedInShop._id
```

### Deliverables

* [ ] Delete item.
* [ ] Ownership validation.
* [ ] Confirmation modal.
* [ ] Remove Cloudinary image when applicable.
* [ ] Refresh Menu Management UI.

---

# Phase 3: Incoming Order Management

* **Status:** ⏳ **PENDING**

Once Menu Management is complete, this should become the next major phase.

---

## Target Frontend Pages

* `ShopIncomingOrders.jsx`
* `ShopOrderDetails.jsx`

---

## Order Flow

The Shop becomes involved when a Student submits an order.

```text
Student
   ↓
Places Order
   ↓
PLACED
   ↓
Shop Incoming Orders
```

---

## Backend Endpoints

```http
GET /api/shop/orders

GET /api/shop/orders/:orderId

PATCH /api/shop/orders/:orderId/accept

PATCH /api/shop/orders/:orderId/reject
```

---

## GET Shop Orders

```http
GET /api/shop/orders
```

Backend should only retrieve:

```text
Orders belonging to logged-in Shop
```

Example:

```js
Order.find({
  shop: shop._id
});
```

Optional filtering:

```http
GET /api/shop/orders?status=PLACED
```

---

## Incoming Order Information

Each incoming order should provide:

```text
Order Number
Student Name
Student ID
Ordered Items
Item Quantity
Subtotal
Delivery Fee
Grand Total
Delivery Room
Special Instructions
Order Time
Order Status
```

---

## Accept Order

```http
PATCH /api/shop/orders/:orderId/accept
```

Status transition:

```text
PLACED
   ↓
CONFIRMED
```

---

## Reject Order

```http
PATCH /api/shop/orders/:orderId/reject
```

Status:

```text
PLACED
   ↓
REJECTED
```

If payment has already been deducted, rejection must trigger the appropriate Student refund logic.

### Deliverables

* [ ] Fetch live Incoming Orders.
* [ ] Remove JSON mock data.
* [ ] Fetch single Order details.
* [ ] Accept Order.
* [ ] Reject Order.
* [ ] Validate Shop ownership.
* [ ] Prevent duplicate accept/reject actions.
* [ ] Record status change in Order timeline.

---

# Phase 4: Food Preparation & Ready-for-Pickup Workflow

* **Status:** ⏳ **PENDING**

---

## Target Frontend Pages

* `ShopPreparingOrder.jsx`
* `ShopReadyForPickup.jsx`
* `ShopOrderDetails.jsx`

---

## Order Lifecycle

```text
PLACED
   ↓
CONFIRMED
   ↓
PREPARING
   ↓
READY_FOR_PICKUP
```

---

## Backend Endpoints

```http
PATCH /api/shop/orders/:orderId/preparing

PATCH /api/shop/orders/:orderId/ready
```

---

## Start Preparing

```http
PATCH /api/shop/orders/:orderId/preparing
```

Transition:

```text
CONFIRMED
   ↓
PREPARING
```

---

## Mark Ready for Pickup

```http
PATCH /api/shop/orders/:orderId/ready
```

Transition:

```text
PREPARING
   ↓
READY_FOR_PICKUP
```

At this point the Shop has completed the food-preparation portion of the Order.

The Runner system can then handle:

```text
Pickup
Delivery
Completion
```

---

## Order Timeline

Each status change should update:

```js
timeline: [
  {
    status: "PLACED",
    time: Date
  },
  {
    status: "CONFIRMED",
    time: Date
  },
  {
    status: "PREPARING",
    time: Date
  },
  {
    status: "READY_FOR_PICKUP",
    time: Date
  }
]
```

### Deliverables

* [ ] Preparing Order queue.
* [ ] "Start Preparing" action.
* [ ] Preparation status persistence.
* [ ] Ready-for-Pickup queue.
* [ ] "Mark Ready" action.
* [ ] Order timeline updates.
* [ ] Prevent invalid status transitions.
* [ ] Connect live MongoDB Orders instead of mock JSON.

---

# Phase 5: Shop Dashboard & Operational Analytics

* **Status:** ⏳ **PENDING**

---

## Target Frontend Page

* `ShopDashboard.jsx`

---

## Backend Endpoint

```http
GET /api/shop/dashboard
```

---

## Dashboard Metrics

The dashboard should show:

```text
Today's Orders
Pending Orders
Preparing Orders
Ready Orders
Completed Orders
Today's Revenue
Total Revenue
Popular Items
Low Stock Items
Recent Orders
Average Rating
```

Example response:

```json
{
  "todayOrders": 18,
  "pendingOrders": 3,
  "preparingOrders": 4,
  "readyOrders": 2,
  "completedOrders": 9,
  "todayRevenue": 5420,
  "averageRating": 4.6
}
```

---

## MongoDB Models Involved

```text
Shop
Order
MenuItem
```

Potentially:

```text
Transaction
```

### Deliverables

* [ ] Replace Dashboard JSON.
* [ ] Today's Order statistics.
* [ ] Revenue calculations.
* [ ] Low-stock warnings.
* [ ] Recent Order list.
* [ ] Best-selling Menu Item.
* [ ] Average Shop rating.

---

# Phase 6: Customer Reviews & Ratings Management

* **Status:** ⏳ **PENDING**

---

## Target Frontend Page

* `ShopCustomerReviews.jsx`

---

## Backend Endpoint

```http
GET /api/shop/reviews
```

Potential filtering:

```http
GET /api/shop/reviews?rating=5
```

---

## Review Information

```text
Student Name
Order Number
Rating
Food Rating
Comment
Date
Ordered Items
```

---

## MongoDB Data Source

Ratings may initially live inside:

```text
Order
```

For example:

```js
ratings: {
  shopRating: Number,
  foodRating: Number,
  shopComment: String
}
```

If review functionality becomes larger, it can later be separated into:

```text
Review
```

model.

### Deliverables

* [ ] Fetch live Shop reviews.
* [ ] Remove review mock JSON.
* [ ] Calculate average rating.
* [ ] Show rating distribution.
* [ ] Filter reviews by rating.
* [ ] Connect reviews with original Orders.
* [ ] Update Shop rating aggregation.

---

# Phase 7: Sales Reports, Earnings & Transaction History

* **Status:** ⏳ **PENDING**

---

## Target Frontend Page

* `ShopSalesReports.jsx`

---

## Backend Endpoints

Potential API structure:

```http
GET /api/shop/reports

GET /api/shop/reports/sales

GET /api/shop/reports/items

GET /api/shop/transactions
```

The exact endpoint structure can be simplified later if required.

---

## Report Metrics

Reports should provide:

```text
Daily Sales
Weekly Sales
Monthly Sales
Total Orders
Completed Orders
Cancelled Orders
Average Order Value
Best-Selling Items
Least-Selling Items
Revenue by Category
```

---

## Date Filtering

Example:

```http
GET /api/shop/reports?from=2026-09-01&to=2026-09-30
```

---

## Shop Earnings

Revenue should not simply equal:

```text
Order Grand Total
```

because the full total can include:

```text
Food Price
Delivery Fee
Platform Fee
Discount
```

Therefore the Order should eventually contain values such as:

```text
subtotal
deliveryFee
platformFee
discount
grandTotal
shopAmount
```

The Shop revenue calculation should use:

```text
shopAmount
```

rather than blindly using:

```text
grandTotal
```

### MongoDB Models Involved

```text
Order
Transaction
Shop
MenuItem
```

### Deliverables

* [ ] Daily revenue.
* [ ] Weekly revenue.
* [ ] Monthly revenue.
* [ ] Custom date filtering.
* [ ] Total completed Orders.
* [ ] Best-selling products.
* [ ] Category-based analytics.
* [ ] Transaction history.
* [ ] Shop earnings calculation.
* [ ] Replace Reports mock JSON.

---

# Phase 8: Shop Order Chat & Customer Communication

* **Status:** ⏳ **PENDING**

This should be treated as the final Shop Owner integration phase.

---

## Potential Target Components

```text
Order Chat Drawer
Order Details Chat Panel
Shop Notification Badge
```

---

## Backend Endpoints

Depending on whether shared Chat routes are used:

```http
GET /api/shop/chat/:orderNumber

POST /api/shop/chat/:orderNumber
```

or a unified architecture:

```http
GET /api/orders/:orderNumber/chat

POST /api/orders/:orderNumber/chat
```

---

## Chat Participants

```text
Student
   ↕
Shop
   ↕
Runner
```

But the Shop Owner should only access conversations connected to Orders belonging to their Shop.

---

## Potential Messages

Examples:

```text
"Your order is being prepared."

"Item X is unavailable. Would you like Item Y?"

"Your food will be ready in approximately 10 minutes."

"Please contact the runner regarding the delivery location."
```

### MongoDB Model

```text
OrderChat
```

Potential structure:

```js
{
  order: ObjectId,

  messages: [
    {
      sender: ObjectId,
      role: String,
      message: String,
      createdAt: Date
    }
  ]
}
```

### Deliverables

* [ ] Shop Order Chat retrieval.
* [ ] Shop message submission.
* [ ] Validate Order ownership.
* [ ] Message persistence.
* [ ] Quick-reply buttons.
* [ ] Unread-message badge.
* [ ] Order-linked chat history.

---

# 🧱 Final Shop Owner Backend Architecture

The Shop Owner backend should eventually contain:

```text
server/
│
├── controllers/
│   └── shop/
│       │
│       ├── shopProfileController.js
│       ├── shopMenuController.js
│       ├── shopOrderController.js
│       ├── shopDashboardController.js
│       ├── shopReviewController.js
│       ├── shopReportController.js
│       └── shopChatController.js
│
├── routes/
│   └── shopRoutes.js
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
│   └── upload.js
│
└── services/
    ├── orderService.js
    ├── shopAnalyticsService.js
    └── uploadService.js
```

---

# 🔗 Shop Owner System Relationship

```text
User
 │
 │ owns
 ↓
Shop
 │
 ├───────────────┐
 │               │
 ↓               ↓
MenuItem        Order
                  │
                  ├──────────→ Student
                  │
                  ├──────────→ Runner
                  │
                  ├──────────→ Transaction
                  │
                  └──────────→ OrderChat
```

---

# 🔄 Complete Shop Owner Workflow

```text
SHOP OWNER LOGIN
       ↓
SHOP PROFILE
       ↓
MENU MANAGEMENT
       ↓
ADD / EDIT / DELETE ITEMS
       ↓
STUDENT PLACES ORDER
       ↓
INCOMING ORDER
       ↓
ACCEPT / REJECT
       ↓
CONFIRMED
       ↓
START PREPARING
       ↓
PREPARING
       ↓
MARK READY
       ↓
READY_FOR_PICKUP
       ↓
RUNNER TAKES OVER
       ↓
DELIVERED
       ↓
SHOP RECEIVES REVENUE DATA
       ↓
CUSTOMER REVIEW
       ↓
DASHBOARD / SALES REPORT
```

---

# ✅ Recommended Implementation Order

Do not work on all phases simultaneously.

Use this sequence:

```text
Phase 1
Shop Authentication & Profile
        ↓
Phase 2
Menu Management
        ↓
Phase 3
Incoming Orders
        ↓
Phase 4
Preparing & Ready for Pickup
        ↓
Phase 5
Dashboard
        ↓
Phase 6
Reviews
        ↓
Phase 7
Sales & Reports
        ↓
Phase 8
Chat
```

---

# 🎯 Current Development Focus

The active development phase should now be:

```text
PHASE 2
MENU MANAGEMENT & ITEM CREATION
```

And inside Phase 2, development should proceed as:

```text
2A — Add New Menu Item
        ↓
2B — Image Upload
        ↓
2C — Edit Menu Item
        ↓
2D — Availability Toggle
        ↓
2E — Delete Menu Item
        ↓
2F — Stock / Low-Stock Management
        ↓
Phase 2 Complete
```

The **immediate implementation task** is therefore:

```http
POST /api/shops/menu
```

with this complete flow:

```text
ShopAddMenuItem.jsx
        ↓
Frontend Validation
        ↓
POST /api/shops/menu
        ↓
JWT Authentication
        ↓
Shop Authorization
        ↓
Shop Ownership Lookup
        ↓
Backend Validation
        ↓
MenuItem.create()
        ↓
MongoDB
        ↓
201 Created
        ↓
ShopMenuManagement.jsx
        ↓
New Item Displayed
```
