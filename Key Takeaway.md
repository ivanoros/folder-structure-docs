Key Takeaway

Build Angular apps by feature, not by file type — and keep UI simple by separating logic from presentation.


---

## 🧠 Key Principles

- Keep logic **out of components**
- Keep features **independent**
- Keep UI **reusable**
- Keep structure **predictable**



---

# 📄 2. `docs/architecture-overview.md`

```md
# Architecture Overview

This project follows a **feature-first, layered architecture** optimized for scalability and maintainability.

---

## 🧱 Layers

### 1. App Shell
- Bootstrapping
- Global routing
- Layout

### 2. Core Layer
Global infrastructure:
- HTTP interceptors
- Authentication
- Config
- Layout components

### 3. Shared Layer
Reusable UI:
- Buttons, tables, inputs
- Pipes, directives

### 4. Feature Layer
Business domains:
- Customers
- Orders
- Dashboard

Each feature contains:
- Pages
- Components
- State
- Data access
- Models

---

## 🔄 Data Flow
