# Team Projects Management

A Flask web application with a Persian (RTL) UI for managing services, plans, customers, and orders, with automatic PDF invoice generation.

**Created and developed by Asal Najafizadeh.**

---

## Table of contents

- [Overview](#overview)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Project structure](#project-structure)
- [File reference](#file-reference)
- [Pages and routes](#pages-and-routes)
- [API](#api)
- [Data model (JSON)](#data-model-json)
- [Invoice types](#invoice-types)
- [Workflow](#workflow)
- [Development notes](#development-notes)
- [License](#license)

---

## Overview

This application provides:

1. **Admin dashboard (`/modir`)** — manage categories, services, plans, and customers
2. **Order page (`/`)** — customers select services/plans, enter details, and issue invoices

Data is stored in JSON files (no relational database). PDF invoices are written to the `reciept/` folder.

---

## Features

- Hierarchical **categories** (parent and child)
- **Services** with price, contract terms, and extra notes
- **Plans** combining multiple services and/or categories
- **Customer** registry (new or existing)
- **Standard orders** from the catalog
- **Custom orders** with four invoice layouts:
  - Standard (ReportLab)
  - Simple (line-item table)
  - Roadmap (phased)
  - Panel (tier cards from economic to exclusive)
- Full **Persian text and digits** in PDFs
- RTL UI with **Tailwind CSS** and the Dana font

---

## Tech stack

| Layer | Technology |
|-------|------------|
| Backend | Python 3, Flask |
| Storage | JSON files |
| PDF (standard) | ReportLab, arabic-reshaper, python-bidi |
| PDF (roadmap / simple / panel) | Playwright + Chromium, Jinja2 |
| Frontend | HTML, Tailwind CSS (CDN bundle), JavaScript |
| Font | Dana-Black |

---

## Prerequisites

- Python 3.10+
- pip
- **Playwright Chromium** for roadmap, simple, and panel invoices

---

## Installation

```bash
git clone <repository-url>
cd TeamProjectsManagement

python -m venv venv

# Windows
venv\Scripts\activate

pip install -r requirements.txt
playwright install chromium

python app.py
```

Server: `http://127.0.0.1:5000`

| URL | Description |
|-----|-------------|
| `http://127.0.0.1:5000/` | Order page |
| `http://127.0.0.1:5000/modir` | Admin dashboard |

> The `reciept/` folder and PDF files are created automatically when invoices are issued.

---

## Project structure

```
TeamProjectsManagement/
├── app.py                      # Flask app — routes and HTTP logic
├── json_store.py               # JSON persistence (CRUD)
├── receipt_pdf.py              # PDF generation (4 types)
├── requirements.txt
├── tailwind.js                 # Tailwind bundle for the UI
│
├── assets/
│   ├── fonts/Dana-Black.ttf
│   └── images/logo.png         # Logo for invoices and UI
│
├── Database/
│   ├── categories.json
│   ├── services.json
│   ├── plans.json
│   ├── customers.json
│   ├── orders.json
│   └── custom-order.json
│
├── templates/
│   ├── index.html              # Admin dashboard
│   ├── order.html              # Order page
│   ├── factor-roadmap.html
│   ├── factor-simple.html
│   └── factor-panel.html
│
└── reciept/                    # Generated PDFs
```

---

## File reference

### Core modules

- **`app.py`** — Flask routes, catalog helpers, CRUD endpoints, `POST /orders`, `POST /custom-orders`, `POST /issue-invoice`
- **`json_store.py`** — read/write JSON, entity CRUD, `parse_price_amount`, order helpers
- **`receipt_pdf.py`** — `create_receipt_pdf`, roadmap/simple/panel PDFs via Playwright, Persian shaping and digits

### Dependencies (`requirements.txt`)

```
flask>=3.0.0
reportlab>=4.0.0
arabic-reshaper>=3.0.0
python-bidi>=0.4.2
jinja2>=3.1.0
playwright>=1.40.0
```

---

## Pages and routes

### HTML

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Order page |
| GET | `/modir` | Admin dashboard |

### Admin forms (POST → redirect to `/modir`)

Category, service, plan, and customer add/edit/delete routes under `/add/*`, `/*/edit`, `/*/delete`.

### Static

| Method | Path | Description |
|--------|------|-------------|
| GET | `/tailwind.js` | Tailwind bundle |
| GET | `/assets/<filename>` | Fonts and images |
| GET | `/receipts/<filename>` | Download invoice PDF |

---

## API

### Standard order

```http
POST /orders
Content-Type: application/json
```

```json
{
  "service_ids": ["uuid"],
  "plan_ids": ["uuid"],
  "customer_mode": "new",
  "customer": { "name": "...", "phone": "...", "address": "..." },
  "invoice_type": "current"
}
```

### Custom order

```http
POST /custom-orders
Content-Type: application/json
```

Body fields depend on `invoice_type`: `steps`, `simple_lines`, or `panel_plans`.

### Issue invoice

```http
POST /issue-invoice
Content-Type: application/json
```

```json
{
  "source": "orders",
  "order_id": "uuid"
}
```

`source`: `"orders"` or `"custom-order"`

---

## Data model (JSON)

Entities use UUID `id` fields. Categories are hierarchical (`kind`: `parent` | `child`). Plans resolve services from `service_ids` and/or `category_ids`. Orders store customer snapshots and `invoice_type`.

---

## Invoice types

| `invoice_type` | Engine | Description |
|----------------|--------|-------------|
| `current` | ReportLab | Standard invoice with customer block and line items |
| `simple` | Playwright | Simple sales table |
| `roadmap` | Playwright | Phased collaboration timeline |
| `panel` | Playwright | Tier pricing cards |

Customize seller details and branding in `templates/factor-*.html` and related context in `receipt_pdf.py`.

---

## Workflow

1. Admin builds the catalog at `/modir`.
2. User places an order at `/`.
3. Order is saved to `orders.json` or `custom-order.json`.
4. `POST /issue-invoice` generates a PDF under `reciept/`.

---

## Development notes

- Deleting a category cleans dependent subcategories and references in services and plans.
- Deleting a service removes it from plan `service_ids`.
- Prices are free-form strings; `parse_price_amount` extracts numeric digits.
- `app.run(debug=True)` is for local development only.

---

## License

Open-source project. See repository settings for license terms if applicable.

---

<p align="center">
  Created and developed by <strong>Asal Najafizadeh</strong>
</p>
