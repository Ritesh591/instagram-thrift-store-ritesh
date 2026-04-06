# Instagram Thrift Store — ER Diagram

---

## What This Repository Contains

This repository contains the database design (Entity–Relationship Diagram) for a small Instagram-based creator store that sells **thrifted fashion items** and **handmade products**.

| File | Description |
|---|---|
| `index.html` | Full interactive HTML page with the ER diagram, legend, entity walkthrough, and business Q&A |
| `instagram_thrift_erd.png` | Exported image of the ER diagram (attach your screenshot here) |
| `README.md` | This file |

---

## The Business

A small creator runs a store through Instagram DMs and WhatsApp. They sell two types of products:

- **Thrifted items** — second-hand fashion pieces. Each is unique (one-of-a-kind). Once sold, it's gone.
- **Handmade items** — products the creator makes themselves, potentially in batches (multiple units available).

The store needs to manage products, track inventory, handle customer orders, record payments, and monitor shipping.

---

## ER Diagram Overview

The diagram models **8 entities**:

| Entity | Type | Purpose |
|---|---|---|
| `CUSTOMER` | Core | Stores buyer details including Instagram handle |
| `PRODUCT` | Core | Central catalog — common fields for all items |
| `THRIFT_ITEM` | Sub-type | Thrift-specific details: condition, brand, is_sold |
| `HANDMADE_ITEM` | Sub-type | Handmade-specific details: material, quantity_available |
| `ORDER` | Transaction | Represents a purchase event by a customer |
| `ORDER_ITEM` | Junction | Links orders to products (many-to-many bridge) |
| `PAYMENT` | Financial | Tracks payment status, method, and reference |
| `SHIPPING` | Logistics | Tracks courier, tracking number, and delivery status |

---

## Key Design Decisions

### 1. Product split into base + sub-type tables
Rather than one giant `PRODUCT` table with many nullable columns, the design uses:
- `PRODUCT` for fields common to all items (name, price, category)
- `THRIFT_ITEM` for thrift-only fields (`condition`, `is_sold`, `brand`)
- `HANDMADE_ITEM` for handmade-only fields (`quantity_available`, `material`)

This avoids `NULL` pollution and keeps the schema clean and normalised.

### 2. Inventory is tracked differently per type
- **Thrift items**: `is_sold` (boolean) — because they are unique pieces
- **Handmade items**: `quantity_available` (integer) — because they can be restocked

### 3. ORDER_ITEM as a junction table
A single order can contain multiple products, and a product can appear across many orders — this is a many-to-many relationship. `ORDER_ITEM` resolves it and also stores `unit_price` at the time of purchase (important if prices change later).

### 4. Payment and Shipping are separate from Order
Separating these concerns means:
- You can update shipping status without touching payment data
- You can track partial payments or payment failures independently
- Each table has a clear, single responsibility

---

## Relationships and Cardinality

| Relationship | Notation | Meaning |
|---|---|---|
| Customer → Order | `\|\|--o{` | One customer, zero or many orders |
| Order → Order_Item | `\|\|--\|{` | One order, one or many items |
| Product → Order_Item | `\|\|--o{` | One product in zero or many order items |
| Product → Thrift_Item | `\|\|--o\|` | One product, zero or one thrift detail |
| Product → Handmade_Item | `\|\|--o\|` | One product, zero or one handmade detail |
| Order → Payment | `\|\|--\|\|` | One order, exactly one payment record |
| Order → Shipping | `\|\|--\|\|` | One order, exactly one shipping record |

---

## Notation Used

This diagram uses **Crow's Foot notation**, the standard for ER diagrams:

| Symbol | Meaning |
|---|---|
| `\|\|` | Exactly one |
| `o\|` | Zero or one |
| `o{` | Zero or many |
| `\|{` | One or many |

**PK** = Primary Key — unique identifier for each row  
**FK** = Foreign Key — reference to the Primary Key of another table

