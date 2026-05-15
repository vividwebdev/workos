# WorkOS — Backend (API Server)

> Node.js + Express REST API powering the WorkOS project management platform.
> Implements layered architecture, JWT authentication, and role-based access control.

---

## 📐 Architecture

The backend follows a **strict layered architecture** where each layer has a single responsibility. Business logic never lives in routes or controllers.

```
Request ──▶ Route ──▶ Controller ──▶ Service ──▶ Model ──▶ MongoDB
                          │
                     Middleware
               (auth · role · error)
```

| Layer          | Responsibility                                      |
| -------------- | --------------------------------------------------- |
| **Routes**     | HTTP wiring — maps endpoints to controllers          |
| **Controllers**| Request/response handling — parsing, status codes    |
| **Services**   | Business logic — validation rules, data orchestration|
| **Models**     | Data structure, schema validation, indexes           |
| **Middlewares**| Cross-cutting concerns — auth, RBAC, error handling  |
| **Utils**      | Shared helpers — custom errors, async wrappers       |

---

## 🗂️ Folder Structure

```bash
server/
├── src/
│   ├── app.js                    # Express app configuration (middleware, routes, error handler)
│   ├── server.js                 # Entry point — connects to DB and starts listening
│   │
│   ├── config/
│   │   ├── env.js                # Environment variable validation & export
│   │   └── db.js                 # MongoDB connection via Mongoose
│   │
│   ├── routes/
│   │   ├── auth.routes.js        # POST /register, POST /login, POST /refresh, POST /logout
│   │   ├── project.route.js      # CRUD endpoints for projects
│   │   └── task.route.js         # CRUD endpoints for tasks
│   │
│   ├── controllers/
│   │   ├── auth.controller.js    # Handles auth request/response cycle
│   │   ├── project.controller.js # Handles project request/response cycle
│   │   └── task.controller.js    # Handles task request/response cycle
│   │
│   ├── services/
│   │   ├── auth.service.js       # Registration, login, token refresh logic
│   │   ├── project.service.js    # Project CRUD business logic
│   │   └── task.service.js       # Task CRUD business logic
│   │
│   ├── models/
│   │   ├── user.model.js         # User schema — email, hashed password, name
│   │   ├── organization.model.js # Organization schema — name, owner
│   │   ├── membership.model.js   # Membership schema — User ↔ Organization + Role
│   │   ├── project.model.js      # Project schema — title, description, org reference
│   │   └── task.model.js         # Task schema — title, status, assignee, project ref
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.js     # JWT verification — attaches user to req
│   │   ├── role.middleware.js     # RBAC enforcement — checks membership role
│   │   └── error.middleware.js   # Centralized error handler — formats ApiError responses
│   │
│   ├── utils/
│   │   ├── ApiError.js           # Custom error class with HTTP status codes
│   │   └── asyncHandler.js       # Wraps async route handlers to catch errors
│   │
│   └── tests/
│       ├── auth.test.js          # Auth flow integration tests
│       ├── project.test.js       # Project CRUD + access control tests
│       └── task.test.js          # Task CRUD + access control tests
│
├── .env.example                  # Template for required environment variables
├── package.json                  # Dependencies and scripts
└── README.md                     # ← You are here
```

---

## 🗄️ Data Model

Five core entities connected through references:

```
┌──────────┐       ┌────────────────┐       ┌──────────┐
│   User   │◄──────│   Membership   │──────►│   Org    │
│          │       │  (role: enum)  │       │          │
└──────────┘       └────────────────┘       └────┬─────┘
                                                 │
                                            ┌────▼─────┐
                                            │ Project  │
                                            └────┬─────┘
                                                 │
                                            ┌────▼─────┐
                                            │   Task   │
                                            └──────────┘
```

| Entity         | Key Fields                                             |
| -------------- | ------------------------------------------------------ |
| **User**       | `name`, `email`, `password` (hashed)                   |
| **Organization** | `name`, `owner` (ref → User)                         |
| **Membership** | `user` (ref), `organization` (ref), `role` (enum)      |
| **Project**    | `title`, `description`, `organization` (ref)           |
| **Task**       | `title`, `status`, `assignee` (ref → User), `project` (ref) |

Schemas enforce validation and define indexes to prevent invalid or inefficient data access.

---

## 🔐 Authentication & Authorization

### Authentication (JWT)

| Mechanism         | Details                                                |
| ----------------- | ------------------------------------------------------ |
| **Access Token**  | Short-lived JWT sent in `Authorization: Bearer <token>`|
| **Refresh Token** | Long-lived token used to obtain new access tokens      |
| **Password**      | Hashed with bcrypt before storage                      |

### Authorization (RBAC)

Authorization is enforced **at the API level**, not the UI.

| Role       | Permissions                                       |
| ---------- | ------------------------------------------------- |
| **Admin**  | Create, update, delete projects and tasks         |
| **Member** | View and interact with assigned projects/tasks    |

The `role.middleware.js` checks the user's membership role against the required role for each endpoint.

---

## 🛣️ API Endpoints

### Auth

| Method | Endpoint          | Description              | Auth Required |
| ------ | ----------------- | ------------------------ | ------------- |
| POST   | `/api/auth/register` | Register a new user    | No            |
| POST   | `/api/auth/login`    | Login, receive tokens  | No            |
| POST   | `/api/auth/refresh`  | Refresh access token   | Refresh Token |
| POST   | `/api/auth/logout`   | Invalidate session     | Yes           |

### Projects

| Method | Endpoint              | Description              | Auth | Role   |
| ------ | --------------------- | ------------------------ | ---- | ------ |
| GET    | `/api/projects`       | List org projects        | Yes  | Member |
| GET    | `/api/projects/:id`   | Get project details      | Yes  | Member |
| POST   | `/api/projects`       | Create a project         | Yes  | Admin  |
| PUT    | `/api/projects/:id`   | Update a project         | Yes  | Admin  |
| DELETE | `/api/projects/:id`   | Delete a project         | Yes  | Admin  |

### Tasks

| Method | Endpoint                        | Description          | Auth | Role   |
| ------ | ------------------------------- | -------------------- | ---- | ------ |
| GET    | `/api/projects/:projectId/tasks`| List project tasks   | Yes  | Member |
| POST   | `/api/projects/:projectId/tasks`| Create a task        | Yes  | Admin  |
| PUT    | `/api/tasks/:id`                | Update a task        | Yes  | Admin  |
| DELETE | `/api/tasks/:id`                | Delete a task        | Yes  | Admin  |

---

## ⚙️ Environment Variables

Copy `.env.example` to `.env` and populate all values:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_ACCESS_SECRET=your_access_token_secret
JWT_REFRESH_SECRET=your_refresh_token_secret
```

> ⚠️ **Secrets are never committed to the repository.** The `.env` file is git-ignored.

---

## 🛠️ Local Development

### Prerequisites

- **Node.js** ≥ 18
- **MongoDB** (local instance or Atlas connection string)

### Getting Started

```sh
# 1. Install dependencies
npm install

# 2. Create your environment file
cp .env.example .env
# Then edit .env with your values

# 3. Start the development server
npm run dev
```

The server will start on `http://localhost:5000` (or your configured `PORT`).

---

## 🧪 Testing

Integration tests cover:

- ✅ Authentication flows (register, login, refresh, logout)
- ✅ Protected route access (valid token, expired token, no token)
- ✅ Project CRUD with role-based access control
- ✅ Task CRUD with role-based access control

Tests are designed to **fail when business logic breaks** — no false positives.

### Running Tests

```sh
npm test
```

### Tools

| Tool         | Purpose                        |
| ------------ | ------------------------------ |
| **Vitest**   | Test runner and assertions     |
| **Supertest**| HTTP integration testing       |

---

## 🧩 Middleware Pipeline

Every request passes through these middleware layers in order:

```
Incoming Request
      │
      ▼
  CORS / Body Parser        ← app.js (Express built-ins)
      │
      ▼
  auth.middleware.js         ← Verifies JWT, attaches req.user
      │
      ▼
  role.middleware.js         ← Checks membership role against required role
      │
      ▼
  Controller → Service      ← Business logic execution
      │
      ▼
  error.middleware.js        ← Catches errors, returns structured JSON response
```

---

## 🧠 Key Design Decisions

| Decision                              | Rationale                                              |
| ------------------------------------- | ------------------------------------------------------ |
| Layered architecture                  | Enforces separation of concerns; each layer is testable in isolation |
| Business logic in services only       | Controllers stay thin; logic is reusable and mockable  |
| Schema validation at database layer   | Guarantees data integrity regardless of entry point    |
| Centralized error handling            | Consistent error responses; no scattered try/catch     |
| `asyncHandler` utility                | Eliminates repetitive try/catch in every controller    |
| Custom `ApiError` class               | Typed errors with HTTP status codes for clean handling |

---

## ⚠️ Known Limitations

- No real-time updates (WebSockets not implemented)
- Limited role granularity (Admin / Member only)
- No audit logging (out of scope for current iteration)

_These were excluded intentionally to maintain focus on core architecture._

---

## 📦 Deployment

| Target        | Options                          |
| ------------- | -------------------------------- |
| **Platform**  | Render / Railway / Fly.io        |
| **CI**        | GitHub Actions                   |
| **Environments** | Separate dev and production configs |

---

## 👤 Author

**Akinwumi ONI**

_This backend reflects understanding of production-grade Node.js API architecture and engineering best practices._
