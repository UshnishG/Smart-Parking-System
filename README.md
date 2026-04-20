# Smart Parking System

Case-study-ready documentation for a full-stack parking management prototype combining C++, Python, MongoDB, and a web UI.

---

## 1) Project Summary

The Smart Parking System is designed to reduce parking search time and improve parking-lot utilization through:

- real-time slot visibility
- digital slot booking and vehicle check-in/check-out
- QR-based parking session identity
- reservation lifecycle handling
- admin analytics dashboard
- basic user authentication and history tracking

The repository contains **two implementations**:

1. **Modern web system** (`web/`)  
   Production-style architecture with backend APIs, realtime updates, and browser UI.
2. **Legacy/academic C++ console system** (`cpp/`)  
   Menu-driven simulation with OOP concepts and CSV persistence.

---

## 2) Problem Statement (Case Study Context)

Urban drivers spend significant time finding parking, creating:

- traffic congestion
- fuel waste and carbon emissions
- poor driver experience
- inefficient parking space utilization

This project models a smart parking platform that addresses these issues through realtime occupancy data, automated fee logic, reservations, and digital workflows.

---

## 3) Solution Overview

### Core idea

Use a hybrid stack where performance-oriented pricing/QR/time utilities are written in C++, exposed to Python via a shared library, and delivered through REST + WebSocket APIs to a browser interface.

### High-level architecture

- **Frontend (HTML/CSS/JS)**: booking, dashboard, history, login/register, payment simulation
- **Backend (Flask + Socket.IO + ASGI wrapper)**: APIs, auth, business logic orchestration
- **C++ shared library (`libparking.so`)**: fee calculation, QR generation/validation, time math
- **MongoDB**: slots, vehicles, reservations, users, sensors, payments, login attempts

---

## 4) Key Features

### Parking operations

- list all slots and available slots
- register parked vehicles
- vehicle status lookup (live fee + duration)
- vehicle exit with final fee computation

### Reservations

- create reservation by slot type / VIP preference
- reservation cancellation
- reservation expiry checks and notifications

### Pricing and QR

- C++-powered fee calculation
- cost estimation endpoint
- QR code generation and validation endpoints

### Authentication and user data

- register/login/logout/refresh profile endpoints
- bcrypt password hashing
- JWT access and refresh tokens in HTTP-only cookies
- brute-force login attempt limiting
- user-specific reservation and vehicle history

### Admin and realtime analytics

- occupancy, revenue, VIP, reservation, sensor, and climate metrics
- occupancy by zone
- recent activity feed
- Socket.IO push updates for slots and stats

---

## 5) Repository Structure

```text
Smart-Parking-System/
├── cpp/                      # Legacy C++ console implementation
│   ├── main.cpp
│   ├── classes.h
│   ├── classes.cpp
│   └── data.csv
├── documentation/            # Existing project PDFs
│   ├── ProjectReport.pdf
│   └── UMLdiagram.pdf
└── web/
    ├── backend/
    │   ├── server.py         # Main backend (Flask + Socket.IO + ASGI)
    │   ├── parking_system.cpp# C++ utility library source
    │   ├── wsgi.py
    │   └── requirement.txt
    └── frontend/
        ├── public/           # Main served web pages and JS
        ├── src/              # React/shadcn assets (not primary runtime path)
        └── package.json
```

---

## 6) Technology Stack

- **Languages**: C++, Python, JavaScript, HTML, CSS
- **Backend**: Flask, python-socketio, a2wsgi, Uvicorn
- **Database**: MongoDB (PyMongo)
- **Security/Auth**: bcrypt, JWT, cookie-based sessions
- **Frontend**: static pages in `web/frontend/public` (plus React toolchain files)
- **Build dependency**: `g++` for `libparking.so`

---

## 7) Data Model (Collections)

Main MongoDB collections used by `server.py`:

- `parking_slots` (slot_id, type, status, floor, zone, coordinates)
- `vehicles` (vehicle_id, vehicle_number, slot_id, entry/exit times, fee, status)
- `reservations` (reservation_id, vehicle_number, slot, duration, expiry, status)
- `users` (user_id, email, password_hash, role)
- `payment_transactions` (transaction_id, amount, provider, status)
- `sensor_data` (sensor_id, zone, battery, status)
- `login_attempts` (brute-force tracking)

---

## 8) API Surface (for Case Study)

Major endpoint groups:

- **Auth**: `/api/auth/register`, `/login`, `/logout`, `/me`, `/refresh`
- **Slots**: `/api/slots/all`, `/available`, `/<slot_id>`
- **Vehicles**: `/api/vehicles/register`, `/<vehicle_number>`, `/<vehicle_number>/exit`, `/history`
- **Fees**: `/api/fee/calculate`, `/api/fee/estimate`
- **Reservations**: `/api/reservations/create`, `/<id>`, `/<id>/cancel`, `/history`, `/check-expiring`
- **QR**: `/api/qr/generate`, `/api/qr/validate`
- **Admin**: `/api/admin/statistics`, `/occupancy-by-zone`, `/recent-activity`, `/sensors`
- **Payment simulation**: `/api/payment/create-checkout`, `/api/payment/confirm`

Realtime channel:

- Socket.IO path: `/api/socket.io`
- Events: `slots_update`, `stats_update`, `reservation_notification`

---

## 9) Setup and Run

### Prerequisites

- Python 3.11+
- MongoDB
- g++ compiler

### Backend setup

```bash
cd web/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirement.txt
g++ -shared -fPIC parking_system.cpp -o libparking.so
```

Create `web/backend/.env`:

```env
MONGO_URL="mongodb://localhost:27017"
DB_NAME="test_database"
CORS_ORIGINS="*"
JWT_SECRET="CHANGE_ME_INSECURE_PLACEHOLDER"
ADMIN_EMAIL="admin@smartpark.com"
ADMIN_PASSWORD="admin123"
```

Security note: generate `JWT_SECRET` using a cryptographically random value (for example, `openssl rand -hex 32`) and never commit real secrets to version control.

Start server:

```bash
uvicorn server:app --host 0.0.0.0 --port 8001
```

Open:

- `http://localhost:8001/index.html`

Default seeded admin (if DB empty):

- email: `admin@smartpark.com`
- password: `admin123`

---

## 10) Business Flow (Case Study Narrative)

1. System seeds slots/sensors/admin user.
2. User views available slots and estimated fee.
3. User parks vehicle (slot becomes occupied, QR generated).
4. Dashboard updates in realtime via Socket.IO.
5. On exit, duration and fee are computed via C++ utility library.
6. Payment transaction is recorded (simulated confirmation flow).
7. Reservation and vehicle history become available to authenticated users.

---

## 11) Engineering Highlights

- hybrid Python+C++ integration via `ctypes`
- realtime push model with polling fallback
- cookie-based JWT auth with refresh token support
- seed-based demo readiness for local evaluation
- clear modular split between API layer and computational utility layer

---

## 12) Known Gaps / Improvement Opportunities

- no comprehensive automated test suite in repository
- payment is simulation-oriented (not full gateway completion lifecycle)
- cookie security flags are local-dev friendly (`secure=False`), but production deployments must set `secure=True` to prevent cookies from traveling over unencrypted HTTP
- reservation/slot assignment logic can be expanded for advanced optimization
- deployment, observability, and role-based access can be hardened further

---

## 13) Existing Documentation Assets

For case-study references, also review:

- `documentation/ProjectReport.pdf`
- `documentation/UMLdiagram.pdf`

---

## 14) Quick Case Study Outline (How to Use This README)

You can directly convert this README into a case study with sections:

1. problem context
2. system architecture
3. implementation details
4. key workflows
5. outcomes/metrics and climate rationale
6. limitations and future roadmap
