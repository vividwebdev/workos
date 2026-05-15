# WorkOS — Frontend (Client)

> React 18 + Vite single-page application for the WorkOS project management platform.
> Implements feature-based architecture, TanStack Query for server state, and Zod for validation.

---

## 📐 Architecture

The frontend follows a **feature-based folder structure** where each domain feature (auth, projects, tasks) is self-contained with its own API layer, hooks, and UI components.

```
User Interaction ──▶ Component ──▶ Hook ──▶ API Layer ──▶ Backend
                                     │
                               TanStack Query
                          (caching · refetching · mutations)
```

| Layer            | Responsibility                                            |
| ---------------- | --------------------------------------------------------- |
| **Components**   | UI rendering — pages and reusable elements                |
| **Hooks**        | Data-fetching orchestration via TanStack Query            |
| **API Layer**    | HTTP requests via centralized `apiClient`                 |
| **Lib**          | Shared infrastructure — API client config, query client   |
| **Shared Hooks** | Cross-feature logic — `useAuth` for auth state            |

> Business logic lives in hooks and API layers, never directly in components.

---

## 🗂️ Folder Structure

```bash
client/
├── src/
│   ├── app/
│   │   ├── App.jsx               # Root component — providers, layout wrapper
│   │   └── router.jsx            # Route definitions — public & protected routes
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── auth.api.js       # Login, register, refresh, logout API calls
│   │   │   ├── auth.hooks.js     # useLogin, useRegister, useLogout hooks
│   │   │   ├── Login.jsx         # Login page component
│   │   │   └── Register.jsx      # Registration page component
│   │   │
│   │   ├── projects/
│   │   │   ├── projects.api.js   # Project CRUD API calls
│   │   │   ├── projects.hooks.js # useProjects, useProject, useCreateProject hooks
│   │   │   ├── ProjectList.jsx   # Projects listing page
│   │   │   └── ProjectDetails.jsx# Single project view with tasks
│   │   │
│   │   └── tasks/
│   │       ├── tasks.api.js      # Task CRUD API calls
│   │       ├── tasks.hooks.js    # useTasks, useCreateTask, useUpdateTask hooks
│   │       └── TaskList.jsx      # Task listing component
│   │
│   ├── components/
│   │   ├── ProtectedRoute.jsx    # Route guard — redirects unauthenticated users
│   │   └── Navbar.jsx            # Global navigation bar
│   │
│   ├── lib/
│   │   ├── apiClient.js          # Axios/fetch wrapper — base URL, interceptors, token injection
│   │   └── queryClient.js        # TanStack Query client configuration
│   │
│   ├── hooks/
│   │   └── useAuth.js            # Global auth state hook — current user, token status
│   │
│   ├── styles/
│   │   └── index.css             # Global styles and design tokens
│   │
│   └── main.jsx                  # Entry point — renders App into DOM
│
├── index.html                    # HTML shell — Vite entry
├── vite.config.js                # Vite configuration — proxy, plugins
└── package.json                  # Dependencies and scripts
```

---

## 🧩 Feature Modules

Each feature module follows a consistent pattern:

```
feature/
├── feature.api.js       # Raw HTTP calls (axios/fetch)
├── feature.hooks.js     # TanStack Query hooks wrapping the API
├── FeaturePage.jsx      # Page-level component
└── FeatureComponent.jsx # (optional) Sub-components
```

### Auth (`features/auth/`)

| File             | Purpose                                                  |
| ---------------- | -------------------------------------------------------- |
| `auth.api.js`    | `registerUser()`, `loginUser()`, `refreshToken()`, `logoutUser()` |
| `auth.hooks.js`  | `useLogin()`, `useRegister()`, `useLogout()` — TanStack mutations |
| `Login.jsx`      | Login form with Zod validation                           |
| `Register.jsx`   | Registration form with Zod validation                    |

### Projects (`features/projects/`)

| File                | Purpose                                               |
| ------------------- | ----------------------------------------------------- |
| `projects.api.js`   | `getProjects()`, `getProject()`, `createProject()`, `updateProject()`, `deleteProject()` |
| `projects.hooks.js` | `useProjects()` query, `useCreateProject()` mutation   |
| `ProjectList.jsx`   | Lists all projects for the current organization        |
| `ProjectDetails.jsx`| Displays project details and its associated tasks      |

### Tasks (`features/tasks/`)

| File              | Purpose                                                 |
| ----------------- | ------------------------------------------------------- |
| `tasks.api.js`    | `getTasks()`, `createTask()`, `updateTask()`, `deleteTask()` |
| `tasks.hooks.js`  | `useTasks()` query, `useCreateTask()` mutation          |
| `TaskList.jsx`    | Renders task list within a project context              |

---

## 🔄 State Management Strategy

### Server State — TanStack Query

All data from the backend is treated as **server state** and managed via TanStack Query:

| Concept          | Implementation                                          |
| ---------------- | ------------------------------------------------------- |
| **Queries**      | `useQuery` for fetching lists and details               |
| **Mutations**    | `useMutation` for create, update, delete operations     |
| **Cache**        | Automatic caching with configurable stale time          |
| **Invalidation** | Mutations trigger query invalidation for fresh data     |
| **Loading/Error**| Built-in `isLoading`, `isError`, `error` states         |

### Client State

Minimal client state is managed via React hooks (`useState`, `useContext`) for:
- Authentication status
- UI-only state (modals, form inputs)

---

## ✅ Form Validation — Zod

All forms use **Zod schemas** for client-side validation:

```
User Input ──▶ Zod Schema ──▶ Validation Result ──▶ Submit / Show Errors
```

| Feature      | Validated Fields                              |
| ------------ | --------------------------------------------- |
| **Login**    | `email` (valid format), `password` (min length)|
| **Register** | `name`, `email`, `password`, `confirmPassword`|
| **Project**  | `title` (required), `description`             |
| **Task**     | `title` (required), `status` (enum)           |

---

## 🛡️ Route Protection

The `ProtectedRoute` component wraps authenticated-only pages:

```
router.jsx
├── /login          → Login.jsx          (public)
├── /register       → Register.jsx       (public)
│
├── 🔒 ProtectedRoute
│   ├── /projects       → ProjectList.jsx
│   ├── /projects/:id   → ProjectDetails.jsx
│   └── /tasks          → TaskList.jsx
```

- Unauthenticated users are redirected to `/login`
- Authentication state is checked via the `useAuth` hook
- Token presence and validity determines access

---

## 🌐 API Client

The centralized API client (`lib/apiClient.js`) handles:

| Feature               | Details                                         |
| --------------------- | ----------------------------------------------- |
| **Base URL**          | Configured via environment variable             |
| **Token Injection**   | Automatically attaches `Authorization: Bearer` header |
| **Interceptors**      | Handles 401 responses — triggers token refresh   |
| **Error Normalization** | Standardizes error format for consistent handling |

All feature API files import from this single client — no duplicated fetch logic.

---

## 🎨 Styling

| Aspect          | Approach                                          |
| --------------- | ------------------------------------------------- |
| **System**      | Custom CSS with design tokens                     |
| **Entry Point** | `styles/index.css` — global resets, variables     |
| **Philosophy**  | Utility styles defined centrally; components reference tokens |

---

## 🛠️ Local Development

### Prerequisites

- **Node.js** ≥ 18
- Backend server running (see `server/README.md`)

### Getting Started

```sh
# 1. Install dependencies
npm install

# 2. Start the development server
npm run dev
```

The client will start on `http://localhost:5173` (Vite default).

### Vite Proxy

The `vite.config.js` can be configured to proxy API requests to the backend:

```js
// vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:5000'
    }
  }
})
```

---

## 🧠 Key Design Decisions

| Decision                                | Rationale                                                |
| --------------------------------------- | -------------------------------------------------------- |
| Feature-based folder structure          | Co-locates related files; scales with features, not file types |
| TanStack Query for server state         | Eliminates manual loading/error state; built-in caching  |
| Zod for form validation                 | Type-safe schemas; reusable between client and server    |
| Centralized API client                  | Single source of truth for HTTP config; DRY principle    |
| `ProtectedRoute` component             | Declarative auth gating; clean router definition         |
| Hooks as data layer                     | Components stay presentational; data logic is reusable   |

---

## ⚠️ Known Limitations

- No real-time updates (WebSocket subscriptions not implemented)
- No offline support or optimistic updates
- Limited role-based UI gating (authorization is enforced server-side)

_These were excluded intentionally to maintain focus on core frontend architecture._

---

## 📦 Deployment

| Target        | Options                          |
| ------------- | -------------------------------- |
| **Platform**  | Netlify / Vercel                 |
| **Build**     | `npm run build` → `dist/`       |
| **CI**        | GitHub Actions                   |
| **Environments** | Separate dev and production configs |

---

## 👤 Author

**Akinwumi ONI**

_This frontend reflects understanding of modern React architecture, server-state management, and scalable frontend engineering practices._
