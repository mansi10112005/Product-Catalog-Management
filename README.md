# Product-Catalog-Management
A full-stack product catalog system built with **Node.js**, **Express**, and **MongoDB/Mongoose**.   Demonstrates advanced database design: embedded documents, references, virtual fields, indexes, aggregation pipelines, and more.

# 🗂️ Product Catalog Management System
### Advanced Database Systems — MongoDB Project

A full-stack product catalog system built with **Node.js**, **Express**, and **MongoDB/Mongoose**.  
Demonstrates advanced database design: embedded documents, references, virtual fields, indexes, aggregation pipelines, and more.

---

## 📁 Project Structure

```
product-catalog/
├── backend/
│   ├── config/
│   │   └── database.js          # MongoDB connection setup
│   ├── models/
│   │   ├── Product.js           # Product schema (complex, with virtuals & statics)
│   │   └── Category.js          # Category schema with virtual sub-categories
│   ├── controllers/
│   │   ├── productController.js # All product CRUD + special operations
│   │   └── categoryController.js
│   ├── routes/
│   │   ├── productRoutes.js     # RESTful product API routes
│   │   └── categoryRoutes.js
│   ├── middleware/
│   │   └── errorHandler.js      # Centralized error handling
│   ├── seed/
│   │   └── seeder.js            # Database seed script (8 products, 5 categories)
│   ├── server.js                # Express app entry point
│   ├── package.json
│   └── .env.example
│
└── frontend/
    └── index.html               # Full UI dashboard (works in demo mode too)
```

---

## 🚀 Setup & Run

### Prerequisites
- [Node.js](https://nodejs.org/) (v18+)
- [MongoDB](https://www.mongodb.com/try/download/community) running locally on port 27017

### 1. Install Dependencies
```bash
cd backend
npm install
```

### 2. Configure Environment
```bash
cp .env.example .env
# Edit .env if needed (default: mongodb://localhost:27017/product_catalog_db)
```

### 3. Seed the Database
```bash
npm run seed
# To clear all data: npm run seed:clear
```

### 4. Start the Server
```bash
npm run dev     # development (with nodemon auto-reload)
npm start       # production
```

Server runs at: **http://localhost:5000**

### 5. Open the Frontend
Open `frontend/index.html` in your browser.  
*(Works in demo mode even without the backend running)*

---

## 🔌 API Endpoints

### Products
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/products` | List products (search, filter, paginate) |
| POST | `/api/products` | Create new product |
| GET | `/api/products/:id` | Get product by ID |
| PUT | `/api/products/:id` | Update product |
| DELETE | `/api/products/:id` | Delete product |
| GET | `/api/products/sku/:sku` | Get product by SKU |
| GET | `/api/products/featured` | Get featured products |
| PATCH | `/api/products/:id/status` | Update product status |
| PATCH | `/api/products/:id/stock` | Update stock quantity |
| POST | `/api/products/:id/reviews` | Add customer review |
| GET | `/api/products/stats/dashboard` | Dashboard analytics |
| GET | `/api/products/stats/by-category` | Stats grouped by category |
| POST | `/api/products/bulk-update` | Bulk update multiple products |

### Categories
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/categories` | List all categories |
| POST | `/api/categories` | Create category |
| GET | `/api/categories/:id` | Get category by ID |
| PUT | `/api/categories/:id` | Update category |
| DELETE | `/api/categories/:id` | Delete (blocked if products exist) |
| GET | `/api/categories/:id/products` | Products in a category |

### Query Parameters (GET /api/products)
```
?q=headphones         # Full-text search (name, description, tags)
?category=<id>        # Filter by category ObjectId
?brand=Sony           # Filter by brand (case-insensitive)
?status=active        # Filter by status
?minPrice=100         # Price range filter
?maxPrice=500
?isFeatured=true      # Featured products only
?sortBy=price.base    # Sort field
?sortOrder=asc        # asc | desc
?page=1&limit=10      # Pagination
```

---

## 🗄️ MongoDB Concepts Used

### 1. Schema Design
- **Embedded Documents**: Reviews, variants, SEO metadata, and supplier info are embedded inside the Product document — ideal for data that is always accessed together.
- **Document References**: Products reference Categories via `ObjectId` — allows efficient `populate()` and avoids duplicating category data.
- **Mixed Types & Maps**: `attributes` field uses `Map<String, Mixed>` for flexible, schema-less product attributes.

### 2. Advanced Mongoose Features
- **Virtual Fields**: `averageRating`, `discountPercentage`, `isLowStock` are computed at runtime without storing in DB.
- **Pre-save Hooks**: Auto-generate SEO slug from product name before saving.
- **Static Methods**: `searchProducts()`, `getDashboardStats()`, `getStatsByCategory()` — reusable query logic on the model class.
- **Instance Methods**: `incrementViews()`, `addReview()` — operations on individual documents.

### 3. Indexing Strategy
```javascript
// Text search index
{ name: 'text', description: 'text', tags: 'text', brand: 'text' }

// Compound indexes for common query patterns
{ 'price.base': 1, status: 1 }
{ category: 1, status: 1 }
{ isFeatured: 1, status: 1 }
{ 'analytics.views': -1 }
{ createdAt: -1 }
```

### 4. Aggregation Pipelines
```javascript
// Example: Stats grouped by category
Product.aggregate([
  { $match: { status: 'active' } },
  { $group: {
      _id: '$category',
      count: { $sum: 1 },
      avgPrice: { $avg: '$price.base' },
      totalStock: { $sum: '$stock.quantity' },
  }},
  { $lookup: { from: 'categories', localField: '_id', foreignField: '_id', as: 'category' }},
  { $unwind: '$category' },
  { $sort: { count: -1 } }
])
```

### 5. Query Operators Used
| Operator | Usage |
|----------|-------|
| `$text` / `$search` | Full-text product search |
| `$gte`, `$lte` | Price range filtering |
| `$in` | Tag-based filtering |
| `$expr` | Low-stock comparison: qty ≤ threshold |
| `$group`, `$sum`, `$avg` | Aggregation stats |
| `$lookup` | Join-like population |
| `$unwind` | Flatten arrays |

---

## 📊 Sample Data

The seed script inserts **5 categories** and **8 realistic products**:

| Product | Category | Price | Stock |
|---------|----------|-------|-------|
| Pro Wireless Headphones X200 | Electronics | $239.99 | 145 |
| Ultra 4K Smart TV 55" | Electronics | $649.00 | 30 |
| Men's Merino Wool Sweater | Clothing | $120.00 | 8 ⚠️ |
| Stand Mixer Pro 6500 | Home & Kitchen | $299.00 | 62 |
| Clean Code: Agile Handbook | Books | $35.00 | 250 |
| Adjustable Dumbbell Set | Sports & Fitness | $329.00 | 40 |
| Mechanical Gaming Keyboard | Electronics | $129.99 | 5 ⚠️ |
| Yoga Mat Premium Non-Slip | Sports & Fitness | $79.99 | 180 |

---

## 🧑‍💻 Technologies

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js v18+ |
| Web Framework | Express.js 4 |
| ODM | Mongoose 8 |
| Database | MongoDB 7 |
| Frontend | Vanilla HTML/CSS/JS |
| Dev Tools | nodemon, dotenv |

---

*Project for Advanced Database Systems subject — demonstrates MongoDB schema design, indexing, aggregation, and RESTful API design.*
