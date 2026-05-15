# WorkOS — Role-Based Project Management SaaS

WorkOS is a full-stack MERN application designed to demonstrate production-grade architecture, authentication, authorization, and frontend–backend integration.

This project was built as a **single evolving codebase** to reflect real-world engineering workflows rather than isolated tutorial exercises.

---

<!-- ## 🚀 Live Demo- Frontend: https://your-frontend-url.com -->

<!-- - Backend API: https://your-backend-url.com -->

---

## 🧠 Problem Statement

Teams need a simple but structured way to manage projects and tasks while enforcing access control.

WorkOS solves this by allowing:

- Users to belong to organizations
- Organizations to manage projects
- Projects to contain tasks
- Access to be controlled via role-based permissions

---

## 🏗️ Architecture Overview

### Backend

- **Node.js + Express**
- Layered architecture:

- Routes (HTTP wiring)
- Controllers (request/response handling)
- Services (business logic)
- Models (data structure & validation)
- JWT authentication (access + refresh tokens)
- Centralized error handling

### Frontend

- **React 18 + Vite**
- Feature-based folder structure

- Server state managed with TanStack Query
- Form validation using Zod
- Protected routes based on authentication state

---

## 🗂️ WorkOS Folder Structure

```bash
workos/
├── client/                         # React frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── App.jsx
│   │   │   └── router.jsx
│   │   │
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── auth.api.js
│   │   │   │   ├── auth.hooks.js
│   │   │   │   ├── Login.jsx
│   │   │   │   └── Register.jsx
│   │   │   │
│   │   │   ├── projects/
│   │   │   │   ├── projects.api.js
│   │   │   │   ├── projects.hooks.js
│   │   │   │   ├── ProjectList.jsx
│   │   │   │   └── ProjectDetails.jsx
│   │   │   │
│   │   │   └── tasks/
│   │   │       ├── tasks.api.js
│   │   │       ├── tasks.hooks.js
│   │   │       └── TaskList.jsx
│   │   │
│   │   ├── components/
│   │   │   ├── ProtectedRoute.jsx
│   │   │   └── Navbar.jsx
│   │   │
│   │   ├── lib/
│   │   │   ├── apiClient.js
│   │   │   └── queryClient.js
│   │   │
│   │   ├── hooks/
│   │   │   └── useAuth.js
│   │   │
│   │   ├── styles/
│   │   │   └── index.css
│   │   │
│   │   └── main.jsx
│   │
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
│
├── server/                         # Node / Express backend
│   ├── src/
│   │   ├── app.js                  # Express app configuration
│   │   ├── server.js               # Entry point (listen)
│   │   │
│   │   ├── config/
│   │   │   ├── env.js              # Environment variable validation
│   │   │   └── db.js               # MongoDB connection
│   │   │
│   │   ├── routes/
│   │   │   ├── auth.routes.js
│   │   │   ├── project.routes.js
│   │   │   └── task.routes.js
│   │   │
│   │   ├── controllers/
│   │   │   ├── auth.controller.js
│   │   │   ├── project.controller.js
│   │   │   └── task.controller.js
│   │   │
│   │   ├── services/
│   │   │   ├── auth.service.js
│   │   │   ├── project.service.js
│   │   │   └── task.service.js
│   │   │
│   │   ├── models/
│   │   │   ├── user.model.js
│   │   │   ├── organization.model.js
│   │   │   ├── membership.model.js
│   │   │   ├── project.model.js
│   │   │   └── task.model.js
│   │   │
│   │   ├── middlewares/
│   │   │   ├── auth.middleware.js
│   │   │   ├── role.middleware.js
│   │   │   └── error.middleware.js
│   │   │
│   │   ├── utils/
│   │   │   ├── ApiError.js
│   │   │   └── asyncHandler.js
│   │   │
│   │   └── tests/
│   │       ├── auth.test.js
│   │       ├── project.test.js
│   │       └── task.test.js
│   │
│   ├── .env.example
│   ├── package.json
│   └── README.md                   # Optional backend-only notes
│
├── .gitignore
├── README.md
└── docker-compose.yml              # Optional (introduced later)
```

```
Each layer has a single responsibility. Business logic never lives in routes or components.
```

---

## 🔐 Authentication & Authorization

- Users authenticate using JWT access tokens
- Refresh tokens are used to maintain sessions securely
- Role-based access control:
  - **Admin**: Can create and manage projects
  - **Member**: Can view and interact with assigned projects
- Authorization is enforced at the API level, not the UI.

---

## 🗄️ Data Model

Core entities:

- User
- Organization
- Membership (User↔Organization+Role)
- Project
- Task Schemas enforce validation and indexes to prevent invalid or inefficient data access.

---

## 🧪 Testing Strategy

- Integration tests for:
- Authentication flows
- Protected routes
- Project creation and access control
- Tests fail when business logic breaks (no false positives)

Tools used:

- Vitest
- Supertest

---

## ⚙️ Environment Variables

Example backend environment variables:

PORT=5000
MONGO_URI=your_mongodb_connection
JWT_ACCESS_SECRET=your_secret
JWT_REFRESH_SECRET=your_secret

_Secrets are never committed to the repository._

---

## 📦 Deployment

- Backend deployed on: Render/Railway/Fly.io
- Frontend deployed on: Netlify/Vercel
- CI checks run via GitHub Actions Separate environments are used for development and production.

---

## 🧠 Key Engineering Decisions

- Chose layered backend architecture to enforce separation of concerns
- Used TanStack Query to manage server state predictably
- Centralized API client to avoid duplicated fetch logic
- Enforced schema validation at the database layer Trade offs and limitations are documented intentionally.

---

## ⚠️ Known Limitations

- No real-time updates (WebSockets not implemented)
- Limited role granularity
- No audit logs (out of scope)

_These were excluded to maintain focus on core architecture._

---

## 🛠️ Local Development

### Backend

```sh
cd server
npm install
npm run dev
```

<!-- Copy code -->

### Frontend

```Sh
cd client
npm install
npm run dev
```

<!-- Copy code -->

---

## 👤 Author (Built by): **Akinwumi ONI**

_This project reflects understanding of full-stack application architecture and engineering best practices._
