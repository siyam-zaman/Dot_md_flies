# UIU Food & Items Delivery Portal — Feature Audit & Page Map

This document provides a comprehensive audit of all stakeholders, features, routes, components, and backend controller mappings implemented across the **UIU Food & Items Delivery System**.

> **🎯 Collaboration Scope:**
> For our 4-member team collaboration, this document specifically highlights the **Shop Owner Role** and its complete backend system mappings, while referencing the relevant frontend pages that consume those APIs.

---

## 📋 Shop Owner Role Feature & Controller Mapping

| Shop Owner Feature                | Route / Page URL                                      | Frontend Component                                 | Backend Endpoint                                                        | Backend Controller                                   |
| :-------------------------------- | :---------------------------------------------------- | :------------------------------------------------- | :---------------------------------------------------------------------- | :--------------------------------------------------- |
| **Shop Authentication & Session** | `/login`                                              | `LoginPage.jsx`                                    | `POST /api/auth/login`, `GET /api/auth/me`                              | `authController.js`                                  |
| **Shop Dashboard**                | `/dashboard/shop`                                     | `ShopDashboard.jsx`                                | `GET /api/shops/dashboard`                                              | `server/controllers/shop/shopDashboardController.js` |
| **Shop Profile Management**       | `/dashboard/shop/profile`                             | `ShopProfile.jsx`                                  | `GET /api/shops/my-shop`, `PUT /api/shops/profile`                      | `server/controllers/shop/shopProfileController.js`   |
| **Menu Management**               | `/dashboard/shop/menu`                                | `ShopMenuManagement.jsx`                           | `GET /api/shops/menu` or existing `GET /api/shops/my-shop`              | `server/controllers/shop/shopMenuController.js`      |
| **Add New Menu Item**             | `/dashboard/shop/menu/add`                            | `ShopAddMenuItem.jsx`                              | `POST /api/shops/menu`                                                  | `server/controllers/shop/shopMenuController.js`      |
| **Edit Menu Item**                | `/dashboard/shop/menu/:itemId/edit`                   | Menu edit interface                                | `PUT /api/shops/menu/:itemId`                                           | `server/controllers/shop/shopMenuController.js`      |
| **Toggle Menu Item Availability** | `/dashboard/shop/menu`                                | `ShopMenuManagement.jsx`                           | `PATCH /api/shops/menu/:itemId/availability`                            | `server/controllers/shop/shopMenuController.js`      |
| **Delete Menu Item**              | `/dashboard/shop/menu`                                | `ShopMenuManagement.jsx`                           | `DELETE /api/shops/menu/:itemId`                                        | `server/controllers/shop/shopMenuController.js`      |
| **Incoming Orders**               | `/dashboard/shop/orders`                              | `ShopIncomingOrders.jsx`                           | `GET /api/shops/orders?status=PLACED`                                   | `server/controllers/shop/shopOrderController.js`     |
| **Order Details**                 | `/dashboard/shop/orders/:orderId`                     | `ShopOrderDetails.jsx`                             | `GET /api/shops/orders/:orderId`                                        | `server/controllers/shop/shopOrderController.js`     |
| **Accept Order**                  | `/dashboard/shop/orders/:orderId`                     | `ShopOrderDetails.jsx`, `ShopIncomingOrders.jsx`   | `PATCH /api/shops/orders/:orderId/accept`                               | `server/controllers/shop/shopOrderController.js`     |
| **Reject Order**                  | `/dashboard/shop/orders/:orderId`                     | `ShopOrderDetails.jsx`, `ShopIncomingOrders.jsx`   | `PATCH /api/shops/orders/:orderId/reject`                               | `server/controllers/shop/shopOrderController.js`     |
| **Start Preparing Order**         | `/dashboard/shop/orders/:id/preparing`                | `ShopPreparingOrder.jsx`                           | `PATCH /api/shops/orders/:orderId/preparing`                            | `server/controllers/shop/shopOrderController.js`     |
| **Mark Order Ready for Pickup**   | `/dashboard/shop/orders/:id/preparing` or ready queue | `ShopPreparingOrder.jsx`, `ShopReadyForPickup.jsx` | `PATCH /api/shops/orders/:orderId/ready`                                | `server/controllers/shop/shopOrderController.js`     |
| **Ready-for-Pickup Queue**        | `/dashboard/shop/orders/ready`                        | `ShopReadyForPickup.jsx`                           | `GET /api/shops/orders?status=READY_FOR_PICKUP`                         | `server/controllers/shop/shopOrderController.js`     |
| **Customer Reviews**              | `/dashboard/shop/reviews`                             | `ShopCustomerReviews.jsx`                          | `GET /api/shops/reviews`                                                | `server/controllers/shop/shopReviewController.js`    |
| **Sales Reports & Analytics**     | `/dashboard/shop/reports`                             | `ShopSalesReports.jsx`                             | `GET /api/shops/reports`                                                | `server/controllers/shop/shopReportController.js`    |
| **Shop Transaction History**      | `/dashboard/shop/reports`                             | `ShopSalesReports.jsx`                             | `GET /api/shops/transactions`                                           | `server/controllers/shop/shopReportController.js`    |
| **Shop Order Chat**               | Order-specific chat interface                         | Order Chat component / drawer                      | `GET /api/shops/chat/:orderNumber`, `POST /api/shops/chat/:orderNumber` | `server/controllers/shop/shopChatController.js`      |

---

## 🧩 Shop Owner Backend Domain Map

The Shop Owner backend is divided into the following major domains:

| Backend Domain           | Responsibility                                                 | Primary Models                     |
| :----------------------- | :------------------------------------------------------------- | :--------------------------------- |
| **Authentication**       | Validate Shop Owner JWT and role                               | `User`                             |
| **Shop Profile**         | Retrieve and update the Shop owned by the authenticated user   | `User`, `Shop`                     |
| **Menu Management**      | Create, update, delete, and control availability of menu items | `Shop`, `MenuItem`                 |
| **Order Management**     | Retrieve Shop orders and handle accept/reject actions          | `Shop`, `Order`                    |
| **Preparation Workflow** | Move orders through confirmed, preparing, and ready states     | `Order`                            |
| **Dashboard**            | Aggregate current Shop operational metrics                     | `Shop`, `Order`, `MenuItem`        |
| **Reviews**              | Retrieve Shop ratings and customer feedback                    | `Order`, `Shop`                    |
| **Sales & Reports**      | Calculate Shop-specific revenue and product analytics          | `Order`, `Transaction`, `MenuItem` |
| **Chat**                 | Persist Shop messages related to Orders                        | `Order`, `OrderChat`               |

---

## 🔐 Shop Owner Authentication & Ownership Mapping

Every Shop-specific request must follow this relationship:

```text
Authenticated User
       ↓
role = shop
       ↓
Shop.owner = req.user._id
       ↓
Associated Shop
       ↓
Shop-owned resource
```

Example Shop lookup:

```js
const shop = await Shop.findOne({
  owner: req.user._id
});
```

A Shop Owner must never be able to retrieve or modify another Shop's:

```text
Menu Items
Orders
Reviews
Transactions
Reports
Chat Threads
```

---

## 🍔 Menu Management Feature Map

The Menu Management module is the current primary backend development area.

| Feature             | HTTP Method | Endpoint                               | Controller Function            | Model      |
| :------------------ | :---------: | :------------------------------------- | :----------------------------- | :--------- |
| Get Shop Menu       |    `GET`    | `/api/shops/menu`                      | `getMenuItems()`               | `MenuItem` |
| Add Menu Item       |    `POST`   | `/api/shops/menu`                      | `addMenuItem()`                | `MenuItem` |
| Update Menu Item    |    `PUT`    | `/api/shops/menu/:itemId`              | `updateMenuItem()`             | `MenuItem` |
| Toggle Availability |   `PATCH`   | `/api/shops/menu/:itemId/availability` | `toggleMenuItemAvailability()` | `MenuItem` |
| Delete Menu Item    |   `DELETE`  | `/api/shops/menu/:itemId`              | `deleteMenuItem()`             | `MenuItem` |

### Menu Item Data

A Shop Menu Item may contain:

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

  dietary: [String],

  isAvailable: Boolean,
  todaySpecial: Boolean,
  featured: Boolean,
  recommended: Boolean
}
```

---

## ➕ Add New Menu Item Mapping

### Frontend Reference

```text
ShopAddMenuItem.jsx
```

### Endpoint

```http
POST /api/shops/menu
```

### Backend Flow

```text
POST /api/shops/menu
        ↓
protect
        ↓
authorizeRoles("shop", "admin")
        ↓
Find Shop owned by authenticated user
        ↓
Validate request body
        ↓
Create MenuItem
        ↓
Save to MongoDB
        ↓
Return 201 Created
```

### Required Backend Validation

```text
Item Name
Price
Associated Shop
```

Additional validation:

```text
Price > 0
Stock >= 0
Discount between 0 and 100
Tax Rate >= 0
Low Stock Warning >= 0
```

---

## 🖼️ Menu Image Upload Mapping

Preferred final architecture:

```text
Shop selects image
        ↓
Multipart/Form-Data request
        ↓
Multer middleware
        ↓
Cloudinary
        ↓
Cloudinary image URL
        ↓
MenuItem.image
        ↓
MongoDB
```

The backend should not permanently store large Base64 image strings.

---

## 📦 Shop Order Management Mapping

| Feature          | HTTP Method | Endpoint                               | Controller Function     |
| :--------------- | :---------: | :------------------------------------- | :---------------------- |
| Get Shop Orders  |    `GET`    | `/api/shops/orders`                    | `getShopOrders()`       |
| Get Single Order |    `GET`    | `/api/shops/orders/:orderId`           | `getShopOrderById()`    |
| Accept Order     |   `PATCH`   | `/api/shops/orders/:orderId/accept`    | `acceptOrder()`         |
| Reject Order     |   `PATCH`   | `/api/shops/orders/:orderId/reject`    | `rejectOrder()`         |
| Start Preparing  |   `PATCH`   | `/api/shops/orders/:orderId/preparing` | `startPreparingOrder()` |
| Mark Ready       |   `PATCH`   | `/api/shops/orders/:orderId/ready`     | `markOrderReady()`      |

---

## 🔄 Shop-Controlled Order Lifecycle

The Shop Owner controls this portion of the order lifecycle:

```text
PLACED
   ↓
CONFIRMED
   ↓
PREPARING
   ↓
READY_FOR_PICKUP
```

Alternative flow:

```text
PLACED
   ↓
REJECTED
```

Once an order reaches:

```text
READY_FOR_PICKUP
```

the delivery workflow becomes primarily the responsibility of the Runner role.

---

## 📊 Shop Dashboard Mapping

### Frontend Reference

```text
ShopDashboard.jsx
```

### Endpoint

```http
GET /api/shops/dashboard
```

### Expected Dashboard Data

```text
Today's Orders
Incoming Orders
Preparing Orders
Ready-for-Pickup Orders
Completed Orders
Today's Sales
Total Revenue
Average Rating
Popular Menu Items
Low-Stock Items
Recent Orders
```

### Backend Controller

```text
server/controllers/shop/shopDashboardController.js
```

---

## ⭐ Customer Review Mapping

### Frontend Reference

```text
ShopCustomerReviews.jsx
```

### Endpoint

```http
GET /api/shops/reviews
```

### Data Returned

```text
Student
Order Number
Rating
Food Rating
Comment
Date
Purchased Items
```

### Backend Controller

```text
server/controllers/shop/shopReviewController.js
```

---

## 📈 Sales Reports & Analytics Mapping

### Frontend Reference

```text
ShopSalesReports.jsx
```

### Backend Endpoints

```http
GET /api/shops/reports

GET /api/shops/transactions
```

Optional filtering:

```http
GET /api/shops/reports?from=2026-09-01&to=2026-09-30
```

### Report Metrics

```text
Daily Revenue
Weekly Revenue
Monthly Revenue
Total Orders
Completed Orders
Rejected Orders
Average Order Value
Best-Selling Items
Least-Selling Items
Revenue by Category
Transaction History
```

### Backend Controller

```text
server/controllers/shop/shopReportController.js
```

---

## 💬 Shop Order Chat Mapping

### Backend Endpoints

```http
GET /api/shops/chat/:orderNumber

POST /api/shops/chat/:orderNumber
```

### Backend Controller

```text
server/controllers/shop/shopChatController.js
```

The Shop must only be able to access a conversation when the associated Order belongs to that Shop.

---

# 📋 Teammate Stakeholder Feature Matrix

The following roles are maintained by the other team members and are shown only for collaboration awareness.

| Stakeholder / Role   | Key Feature Requirements         | Route / Page URL                    | Component File                       |
| :------------------- | :------------------------------- | :---------------------------------- | :----------------------------------- |
| **Admin**            | Approve Shop Owner accounts      | `/dashboard/admin/shop-owners`      | `AdminShopOwnerApproval.jsx`         |
| **Admin**            | Approve / manage Runner accounts | `/dashboard/admin/runners`          | `AdminRunnerApproval.jsx`            |
| **Admin**            | Create and manage campus Shops   | `/dashboard/admin/shops`            | `AdminManageShops.jsx`               |
| **Admin**            | Overall system reports           | `/dashboard/admin/reports`          | `AdminReportsAnalytics.jsx`          |
| **Admin**            | Handle complaints                | `/dashboard/admin/complaints`       | `AdminComplaintManagement.jsx`       |
| **Ordering Student** | Browse Shops and Menu Items      | `/dashboard/student/shops`          | `BrowseShops.jsx`, `ShopDetails.jsx` |
| **Ordering Student** | Place Orders                     | `/checkout`                         | `CheckoutPage.jsx`                   |
| **Ordering Student** | Track Orders                     | `/dashboard/student/orders`         | `MyOrdersPage.jsx`                   |
| **Ordering Student** | Rate Shop                        | `/dashboard/student/orders`         | `MyOrdersPage.jsx`                   |
| **Delivery Runner**  | View available deliveries        | `/dashboard/runner/deliveries`      | `RunnerAvailableDeliveries.jsx`      |
| **Delivery Runner**  | Accept delivery requests         | `/dashboard/runner/deliveries`      | `RunnerOrderAccepted.jsx`            |
| **Delivery Runner**  | Update delivery status           | `/dashboard/runner/active/tracking` | `RunnerOrderTracking.jsx`            |
| **Delivery Runner**  | View earnings                    | `/dashboard/runner/earnings`        | `RunnerEarnings.jsx`                 |

---

# 🔗 Cross-Role Integration Points

Although the active development scope belongs to the Shop Owner backend, several Shop features depend on data created by other roles.

### Student → Shop

```text
Student places Order
       ↓
Order.shop
       ↓
Shop Incoming Orders
```

---

### Shop → Runner

```text
Shop marks Order
READY_FOR_PICKUP
       ↓
Runner discovers delivery
```

---

### Student → Shop Review

```text
Order delivered
       ↓
Student submits rating
       ↓
Shop Reviews
```

---

### Admin → Shop

```text
Shop Owner registration / Shop creation
       ↓
Admin approval
       ↓
Shop becomes operational
```

These integration points should be respected, but Shop Owner backend code should remain inside the Shop-specific architecture wherever possible.

---

# 🗂️ Shop Owner Backend Controller Map

Recommended Shop backend controller structure:

```text
server/
└── controllers/
    └── shop/
        ├── shopProfileController.js
        ├── shopMenuController.js
        ├── shopOrderController.js
        ├── shopDashboardController.js
        ├── shopReviewController.js
        ├── shopReportController.js
        └── shopChatController.js
```

Recommended Shop route structure:

```text
server/
└── routes/
    └── shop/
        └── shopRoutes.js
```

Shared models:

```text
server/models/
├── User.js
├── Shop.js
├── MenuItem.js
├── Order.js
├── Transaction.js
└── OrderChat.js
```

---

# 🎯 Current Shop Owner Implementation Priority

Current focus:

```text
Phase 2 — Menu Management Backend
```

Implementation sequence:

```text
Add New Menu Item
       ↓
Image Upload
       ↓
Edit Menu Item
       ↓
Availability Toggle
       ↓
Delete Menu Item
       ↓
Stock Management
       ↓
Menu Phase Complete
```

Immediate backend target:

```http
POST /api/shops/menu
```

Required flow:

```text
Authenticated Shop Owner
        ↓
Find owned Shop
        ↓
Validate Menu Item
        ↓
Create MenuItem
        ↓
Persist to MongoDB
        ↓
Return 201 Created
```

---

## 🎯 Verification Summary

* **Shop Owner Backend Scope:** Shop-specific controllers should be modularized under `server/controllers/shop/`.
* **Shop Owner Routes:** Shop-specific endpoints should be modularized under `server/routes/shop/`.
* **Authentication:** Protected Shop routes require JWT authentication and Shop role authorization.
* **Ownership Security:** Shop Owners may only access resources belonging to their own Shop.
* **MongoDB:** All Shop operations must use persisted MongoDB data rather than mock backend responses.
* **Frontend Role:** Frontend pages serve as API contract references unless frontend modification is explicitly requested.
* **Backend Server:** `http://localhost:5001`
* **Frontend Development Server:** `http://localhost:5173`
* **Vite API Proxy:** `/api -> http://localhost:5001`
