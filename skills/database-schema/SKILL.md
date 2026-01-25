# Database Schema Design

> Design efficient, scalable database schemas for applications.

## Trigger

- User needs to design a database schema
- User is starting a new project with data persistence
- User needs to model entities and relationships

## Instructions

### Step 1: Identify Entities

List all the "things" in your system:

**Questions to ask:**
- What are the main objects? (User, Product, Order)
- What needs to be stored?
- What will be queried?

**Example for e-commerce:**
```
Entities: User, Product, Order, OrderItem, Category, Review
```

### Step 2: Define Attributes

For each entity, list attributes:

```
User:
  - id (primary key)
  - email (unique)
  - name
  - password_hash
  - created_at
  - updated_at
```

**Attribute rules:**
- Every table needs a primary key (usually `id`)
- Add `created_at` and `updated_at` to most tables
- Use appropriate data types
- Mark nullable vs required

### Step 3: Define Relationships

**Relationship types:**

| Type | Example | Implementation |
|------|---------|----------------|
| One-to-One | User → Profile | Foreign key on either side |
| One-to-Many | User → Posts | Foreign key on "many" side |
| Many-to-Many | Post ↔ Tags | Junction table |

**Example relationships:**
```
User (1) → (many) Orders
Order (1) → (many) OrderItems
Product (1) → (many) OrderItems
Product (many) ↔ (many) Categories
```

### Step 4: Create Schema

**SQL Example:**

```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL,
  stock_quantity INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders table (one-to-many with users)
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  status VARCHAR(50) DEFAULT 'pending',
  total DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order items (junction for orders and products)
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price_at_time DECIMAL(10, 2) NOT NULL
);

-- Many-to-many: Products and Categories
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE product_categories (
  product_id INTEGER REFERENCES products(id) ON DELETE CASCADE,
  category_id INTEGER REFERENCES categories(id) ON DELETE CASCADE,
  PRIMARY KEY (product_id, category_id)
);
```

**Prisma Example:**

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  orders    Order[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Product {
  id          Int         @id @default(autoincrement())
  name        String
  description String?
  price       Decimal     @db.Decimal(10, 2)
  stock       Int         @default(0)
  categories  Category[]
  orderItems  OrderItem[]
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt
}

model Order {
  id        Int         @id @default(autoincrement())
  user      User        @relation(fields: [userId], references: [id])
  userId    Int
  status    String      @default("pending")
  total     Decimal     @db.Decimal(10, 2)
  items     OrderItem[]
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt
}

model OrderItem {
  id          Int     @id @default(autoincrement())
  order       Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  orderId     Int
  product     Product @relation(fields: [productId], references: [id])
  productId   Int
  quantity    Int
  priceAtTime Decimal @db.Decimal(10, 2)
}

model Category {
  id       Int       @id @default(autoincrement())
  name     String
  slug     String    @unique
  products Product[]
}
```

### Step 5: Add Indexes

**When to add indexes:**
- Columns used in WHERE clauses
- Columns used in JOIN conditions
- Columns used in ORDER BY
- Foreign keys (usually auto-indexed)

```sql
-- Index for common queries
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_products_name ON products(name);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

### Step 6: Consider Denormalization

**When to denormalize:**
- Read-heavy workloads
- Expensive joins
- Caching frequently computed values

```sql
-- Denormalized: store total on order instead of computing
ALTER TABLE orders ADD COLUMN item_count INTEGER DEFAULT 0;

-- Denormalized: store rating average on product
ALTER TABLE products ADD COLUMN avg_rating DECIMAL(3, 2);
```

## Data Types Reference

| Type | SQL | Prisma | Use Case |
|------|-----|--------|----------|
| Integer | INTEGER | Int | IDs, counts |
| String | VARCHAR(n) | String | Names, emails |
| Text | TEXT | String | Long content |
| Decimal | DECIMAL(10,2) | Decimal | Money |
| Boolean | BOOLEAN | Boolean | Flags |
| Timestamp | TIMESTAMP | DateTime | Dates |
| JSON | JSONB | Json | Flexible data |
| UUID | UUID | String @db.Uuid | Unique IDs |

## Success Criteria

- [ ] All entities identified
- [ ] Primary keys defined
- [ ] Relationships clear
- [ ] Appropriate data types
- [ ] Indexes on queried columns
- [ ] Timestamps on mutable tables

## Common Pitfalls

- Don't over-normalize (too many joins)
- Don't under-normalize (data duplication issues)
- Don't forget foreign key constraints
- Don't use strings for money (use DECIMAL)
- Don't skip indexes for frequently queried columns
