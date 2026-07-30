
# 🍅 Tomato

A Zomato/Swiggy-style food delivery platform built with a **microservices architecture**. Customers browse restaurants and order food, restaurants manage menus and orders, riders deliver orders in real time, and admins verify and moderate the platform — each backed by its own independent Node.js/Express service.

## Overview

Tomato is split into six independent backend services and one React frontend, communicating over REST, RabbitMQ (async messaging), and Socket.IO (real-time events). This design mirrors how production food-delivery platforms isolate concerns like auth, ordering, payments, and live tracking so each piece can scale and fail independently.

## Architecture

```
                         ┌─────────────┐
                         │   Frontend  │
                         │ React + Vite│
                         └──────┬──────┘
                                │ REST / Socket.IO
        ┌──────────┬───────────┼───────────┬──────────┬──────────┐
        │           │           │           │          │          │
   ┌────▼───┐  ┌────▼─────┐ ┌───▼────┐ ┌────▼────┐ ┌───▼───┐ ┌────▼────┐
   │  Auth  │  │Restaurant│ │  Rider │ │Realtime │ │ Admin │ │  Utils  │
   │ :5000  │  │  :5001   │ │ :5005  │ │ :5004   │ │ :5006 │ │  :5002  │
   └────────┘  └────┬─────┘ └───┬────┘ └─────────┘ └───────┘ └────┬────┘
                     │           │                                 │
                     └─────RabbitMQ (order/payment/rider queues)───┘
```

| Service | Port | Responsibility |
|---|---|---|
| **auth** | 5000 | User authentication (Google OAuth), JWT issuance, role management |
| **restaurant** | 5001 | Restaurants, menu items, cart, addresses, and order lifecycle |
| **utils** | 5002 | Shared utilities — image uploads (Cloudinary), payments (Stripe & Razorpay) |
| **realtime** | 5004 | Socket.IO gateway for pushing live order/rider status updates to clients |
| **rider** | 5005 | Rider profiles, availability, order acceptance, delivery status updates |
| **admin** | 5006 | Verifying/approving pending restaurants and riders |
| **frontend** | — | React + TypeScript SPA (customer, restaurant, rider, and admin views) |

Services communicate synchronously via internal REST calls (secured with an `INTERNAL_SERVICE_KEY`) and asynchronously via **RabbitMQ** queues for order, payment, and rider-assignment events.

## Tech Stack

**Backend**
- Node.js + Express 5 + TypeScript
- MongoDB with Mongoose (auth, restaurant, rider) and native driver (admin)
- RabbitMQ (`amqplib`) for inter-service messaging
- Socket.IO for real-time communication
- JWT for authentication, Google OAuth (`googleapis`) for login
- Stripe & Razorpay for payments, Cloudinary for image storage, Multer for uploads

**Frontend**
- React 19 + TypeScript + Vite
- Tailwind CSS 4
- React Router 7
- Socket.IO client
- Leaflet / react-leaflet + Leaflet Routing Machine (live delivery maps)
- Stripe.js, Google OAuth (`@react-oauth/google`)
- React Hot Toast for notifications

## Project Structure

```
tomato-code/
├── frontend/                 # React + Vite SPA
│   └── src/
│       ├── components/       # Reusable UI (order cards, maps, restaurant cards, etc.)
│       ├── pages/             # Route-level pages (Home, Cart, Checkout, RiderDashboard, Admin, etc.)
│       ├── context/           # App & Socket React contexts
│       └── utils/              # Order flow helpers
├── services/
│   ├── auth/                 # Login, JWT, role assignment
│   ├── restaurant/           # Restaurants, menu, cart, addresses, orders
│   ├── rider/                 # Rider profiles & delivery flow
│   ├── realtime/             # Socket.IO event gateway
│   ├── admin/                # Restaurant/rider verification
│   └── utils/                  # Uploads & payments
└── Project_Setup_Guide.pdf   # Original setup walkthrough
```

Each service follows the same internal layout: `src/controllers`, `src/routes`, `src/middlewares`, `src/config`, and (where applicable) `src/model(s)`.

## Prerequisites

- **Node.js** (v18+ recommended) and npm
- **MongoDB** instance (local or Atlas) — all services except `realtime` and `utils` use it
- **RabbitMQ** — run locally via Docker, e.g.:
  ```bash
  docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
  ```
- **Cloudinary** account (image uploads)
- **Stripe** and **Razorpay** accounts (payments)
- **Google Cloud OAuth** credentials (Google sign-in)

## Getting Started

Each service is independent and must be installed and run separately. Open a new terminal per service.

### 1. Frontend
```bash
cd frontend
npm install
```
- Open `src/main.tsx` and replace the Google Client ID with your own.
- Configure `frontend/.env` (see [Environment Variables](#environment-variables)).

### 2. Auth Service
```bash
cd services/auth
npm install
```
Configure `services/auth/.env`.

### 3. Restaurant Service
```bash
cd services/restaurant
npm install
```
Make sure RabbitMQ is running, then configure `services/restaurant/.env`.

### 4. Utils Service
```bash
cd services/utils
npm install
```
Add your payment gateway keys and Cloudinary credentials to `services/utils/.env`.

### 5. Realtime Service
```bash
cd services/realtime
npm install
```
Configure `services/realtime/.env`.

### 6. Rider Service
```bash
cd services/rider
npm install
```
Configure `services/rider/.env`.

### 7. Admin Service
```bash
cd services/admin
npm install
```
Use the **same database name** as the other services (`DB_NAME` in `.env`).

### Running everything

In each service directory:
```bash
npm run dev
```
This runs `tsc --watch` alongside `node --watch dist/index.js` for live-reloading TypeScript.

For the frontend:
```bash
cd frontend
npm run dev
```

> A full walkthrough (including a RabbitMQ Docker setup video reference) is included in `Project_Setup_Guide.pdf`.

## Environment Variables

Each service reads its own `.env` file. Fill these in with your own credentials — **never commit real secrets**.

**services/auth/.env**
```
PORT=5000
MONGO_URI=
JWT_SEC=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
```

**services/restaurant/.env**
```
PORT=5001
MONGO_URI=
JWT_SEC=
UTILS_SERVICE=
REALTIME_SERVICE=
INTERNAL_SERVICE_KEY=
RABBITMQ_URL=
PAYMENT_QUEUE=
RIDER_QUEUE=
ORDER_READY_QUEUE=
```

**services/utils/.env**
```
PORT=5002
CLOUD_NAME=
CLOUD_API_KEY=
CLOUD_SECRET_KEY=
STRIPE_SECRET_KEY=
FRONTEND_URL=
RESTAURANT_SERVICE=
INTERNAL_SERVICE_KEY=
RABBITMQ_URL=
PAYMENT_QUEUE=
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
```

**services/realtime/.env**
```
PORT=5004
JWT_SEC=
INTERNAL_SERVICE_KEY=
```

**services/rider/.env**
```
PORT=5005
MONGO_URI=
JWT_SEC=
UTILS_SERVICE=
REALTIME_SERVICE=
RESTAURANT_SERVICE=
INTERNAL_SERVICE_KEY=
RABBITMQ_URL=
RIDER_QUEUE=
ORDER_READY_QUEUE=
```

**services/admin/.env**
```
PORT=5006
MONGO_URI=
JWT_SEC=
DB_NAME=
```

**frontend/.env**
```
VITE_STRIPE_PUBLISHABLE_KEY=
VITE_INTERNAL_SERVICE_KEY=
```

## API Overview

All routes are JWT-authenticated (`isAuth`) unless noted, with additional role guards (`isSeller`, `isAdmin`) where relevant.

**Auth Service** — `/api/auth`
| Method | Route | Description |
|---|---|---|
| POST | `/login` | Log in via Google OAuth |
| PUT | `/add/role` | Assign a role to the current user |
| GET | `/me` | Get current user profile |

**Restaurant Service**
| Base | Routes |
|---|---|
| `/api/restaurant` | `POST /new`, `GET /my`, `PUT /status`, `PUT /edit`, `GET /all` (nearby), `GET /:id` |
| `/api/item` | `POST /new`, `GET /all/:id`, `DELETE /:itemId`, `PUT /status/:itemId` |
| `/api/cart` | `POST /add`, `GET /all`, `PUT /inc`, `PUT /dec`, `DELETE /clear` |
| `/api/address` | `POST /new`, `DELETE /:id`, `GET /all` |
| `/api/order` | `GET /myorder`, `GET /:id`, `POST /new`, `GET /payment/:id`, `PUT /:orderId`, `PUT /assign/rider`, `GET /current/rider`, `PUT /update/status/rider` |

**Rider Service** — `/api/rider`
| Method | Route | Description |
|---|---|---|
| POST | `/new` | Create rider profile (with document upload) |
| GET | `/myprofile` | Get rider's own profile |
| PATCH | `/toggle` | Toggle availability |
| POST | `/accept/:orderId` | Accept a delivery order |
| GET | `/order/current` | Get current assigned order |
| PUT | `/order/update/:orderId` | Update delivery status |

**Admin Service** — `/api/v1`
| Method | Route | Description |
|---|---|---|
| GET | `/admin/restaurant/pending` | List restaurants awaiting approval |
| GET | `/admin/rider/pending` | List riders awaiting approval |
| PATCH | `/verify/rider/:id` | Approve/verify a rider |
| PATCH | `/verify/restaurant/:id` | Approve/verify a restaurant |

**Utils Service**
| Base | Routes |
|---|---|
| `/api` | `POST /upload` (Cloudinary image upload) |
| `/api/payment` | `POST /create`, `POST /verify` (Razorpay); `POST /stripe/create`, `POST /stripe/verify` (Stripe) |

**Realtime Service** — `/api/v1/internal`
| Method | Route | Description |
|---|---|---|
| POST | `/emit` | Internal endpoint used by other services to push Socket.IO events to clients |

## Key Features

- **Role-based access** — customer, restaurant owner, rider, and admin roles with dedicated dashboards
- **Live order tracking** — Socket.IO + Leaflet map showing rider location and delivery route in real time
- **Async order pipeline** — RabbitMQ queues decouple order placement, payment confirmation, and rider assignment
- **Dual payment gateways** — Stripe and Razorpay support
- **Restaurant & rider verification** — admin approval workflow before restaurants/riders go live
- **Image uploads** — Cloudinary-backed uploads for restaurant, menu, and rider documents

## Notes

- Repo is currently a personal project without a dedicated license file — add one if you plan to publish or share it publicly.
- The `INTERNAL_SERVICE_KEY` is used to authenticate service-to-service calls (e.g., restaurant → realtime, restaurant → utils) — keep it consistent across services that talk to each other.
- All services expect the **same MongoDB database** so collections stay linked across services (see the Admin service setup note).
