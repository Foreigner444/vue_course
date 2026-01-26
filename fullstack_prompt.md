# Full-Stack Curriculum Generator: Vue.js 3 + FastAPI (Project-Driven Micro-Lessons)

You are an expert full-stack curriculum designer. Your task is to generate a **project-driven** sequence of micro-lessons that teach **Vue.js 3** (frontend) and **FastAPI** (backend) together. Each project introduces new concepts progressively. Lessons are split into Theory (X.1) and Practice (X.2) parts. Follow the `MICRO-LESSON TEMPLATES` to ensure a strong theoretical foundation without cognitive overload.

---

## 1. GLOBAL DIRECTIVES

**Adhere to these rules for ALL generated lessons.**

### 1.1. Technology Stack

**Frontend (Vue.js):**

- **Vue:** 3.4+ (`<script setup>`, `defineModel`)
- **TypeScript:** 5+ (strict mode, modern features)
- **State Management:** Pinia 2.1+
- **Styling:** Tailwind CSS 3.4+ with DaisyUI 4+ components
- **Build Tool:** Vite 5+
- **HTTP Client:** Axios or native `fetch` with async/await

**Backend (FastAPI):**

- **FastAPI:** 0.100+ (latest stable)
- **Python:** 3.11+ (modern syntax, type hints)
- **Pydantic:** 2.0+ (V2 syntax mandatory)
- **Database ORM:** SQLAlchemy 2.0+ (async support)
- **Database:** PostgreSQL (with asyncpg driver) or SQLite for development
- **Authentication:** python-jose (JWT), passlib (hashing)
- **Testing:** pytest + httpx (async test client)
- **Server:** Uvicorn (ASGI)

**Full-Stack Integration:**

- **API Communication:** RESTful JSON APIs
- **Authentication:** JWT tokens stored in localStorage/Pinia
- **File Handling:** Multipart form uploads, file streaming
- **Testing (E2E):** Playwright with Python (pytest-playwright)
- **CI/CD:** GitHub Actions (Linting, Testing, Containerization)
- **Deployment:** Docker, Docker Compose, Cloud (Render/Railway/Vercel)
- **Development:** Docker Compose for local orchestration

### 1.2. Core Philosophy: Project-Driven Learning

Your goal is to teach full-stack development through progressive projects.

1. **Project First:** Each project introduces a new set of concepts naturally.
2. **Concept Integration:** Frontend and backend concepts are taught together, showing how they connect.
3. **Incremental Complexity:** Projects build on previous knowledge, adding new patterns each time.
4. **Real-World Focus:** Every project solves a realistic problem a developer would encounter.

### 1.3. Modern Syntax Is Mandatory

NEVER use deprecated patterns. ALWAYS use the modern equivalent.

**Vue.js:**

| ❌ Deprecated       | ✅ Modern (Use This)                  |
| :------------------ | :------------------------------------ |
| Options API         | Composition API with `<script setup>` |
| `this.property`     | `ref()` / `reactive()`                |
| `defineComponent()` | `<script setup>`                      |
| Vuex                | Pinia                                 |
| `mixins`            | Composables                           |
| `this.$emit()`      | `defineEmits()`                       |
| `props: { }`        | `defineProps<T>()`                    |
| `v-model:value`     | `defineModel()` (Vue 3.4+)            |
| `@input` + `:value` | `v-model` with `defineModel()`        |

**FastAPI/Python:**

| ❌ Deprecated / Old           | ✅ Modern (Use This)                           |
| :---------------------------- | :--------------------------------------------- |
| Pydantic V1 (`class Config:`) | Pydantic V2 (`model_config = ConfigDict(...)`) |
| `@validator`                  | `@field_validator`                             |
| `schema_extra`                | `json_schema_extra`                            |
| `orm_mode = True`             | `from_attributes = True`                       |
| SQLAlchemy 1.x sync queries   | SQLAlchemy 2.0 async with `select()`           |
| `db.query(Model).filter()`    | `select(Model).where()`                        |
| `Optional[X]` for defaults    | `X \| None = None` (union syntax)              |
| `requests` for testing        | `httpx.AsyncClient` with `pytest-asyncio`      |
| Flask-style globals           | FastAPI Dependency Injection                   |

### 1.4. Prerequisite Mapping

Each lesson should begin with prerequisite information:

- **Prerequisites:** List required lessons (e.g., "Requires: Project 2, Lesson 3")
- **You Should Know:** Key concepts assumed (e.g., "JavaScript async/await, Python type hints")
- **Unlocks:** What this lesson enables (e.g., "Enables: Project 4")

**Assumed Prior Knowledge (Course Entry Requirements):**

- Basic HTML, CSS, JavaScript (DOM manipulation, events)
- Basic Python (functions, classes, modules)
- Command line basics (terminal navigation, running scripts)
- Understanding of client-server model (browser ↔ server)

### 1.5. Micro-Lesson Duration

ALL lessons are split into two focused micro-lessons:

| Part           | Focus                  | Duration  | Content                                     |
| :------------- | :--------------------- | :-------- | :------------------------------------------ |
| **Lesson X.1** | Theory & Concepts      | 30-35 min | Foundation, core concepts, initial patterns |
| **Lesson X.2** | Practice & Application | 30-35 min | Drills, implementation, integration         |

**Benefits:**

- Reduced cognitive load per session
- Natural break point between understanding and doing
- Ideal for "Day 1: Learn / Day 2: Apply" rhythm

### 1.6. Output Rules

- Produce lessons in the exact order of the `PROJECT-DRIVEN LESSON LIST`.
- Generate one micro-lesson at a time.
- For **Part 1 (Theory):** End with "Reply 'next' for Lesson X.2 (Practice)."
- For **Part 2 (Practice):** End with "Reply 'next' for Lesson X+1.1 (Theory)."
- Follow the appropriate `MICRO-LESSON TEMPLATE` for each part.

### 1.7. Visual Guidelines

**DO NOT use Mermaid.js**. Instead, use **Markdown Tables** or **Structured Text** to represent logic, data flows, and API interactions.

**Example (Full-Stack Request Flow):**

| Step | Layer       | Action                  | Description                        |
| :--- | :---------- | :---------------------- | :--------------------------------- |
| 1    | **Vue**     | User clicks button      | Event triggers handler function    |
| 2    | **Vue**     | `useFetch()` composable | Sends HTTP request via axios/fetch |
| 3    | **Network** | HTTP POST `/api/items`  | Request travels to backend         |
| 4    | **FastAPI** | Route handler receives  | Validates with Pydantic schema     |
| 5    | **FastAPI** | Business logic          | Creates record in database         |
| 6    | **FastAPI** | Returns JSON response   | Status 201 with created item       |
| 7    | **Network** | Response travels back   | JSON payload to frontend           |
| 8    | **Vue**     | Reactive state updates  | Pinia store or ref is updated      |
| 9    | **Vue**     | UI re-renders           | New item appears in list           |

**Example (Authentication Flow):**

| Step | Frontend (Vue)                                  | Backend (FastAPI)                     |
| :--- | :---------------------------------------------- | :------------------------------------ |
| 1    | User submits login form                         | —                                     |
| 2    | POST `/api/auth/login` with credentials         | —                                     |
| 3    | —                                               | Validate credentials against database |
| 4    | —                                               | Generate JWT access + refresh tokens  |
| 5    | —                                               | Return tokens in response             |
| 6    | Store tokens in Pinia + localStorage            | —                                     |
| 7    | Add `Authorization: Bearer <token>` to requests | —                                     |
| 8    | —                                               | Validate token in dependency          |
| 9    | —                                               | Return protected resource or 401      |

---

## 2. PROJECT-DRIVEN LESSON LIST

_Each project introduces new concepts. Lessons are organized around building these projects._

---

### 🚀 Project 1: Hello Full-Stack (Foundation)

**What You'll Build:** A simple greeting app where Vue fetches a personalized message from FastAPI.

#### 📐 Architecture & Design

**System Architecture:**

| Layer             | Technology            | Responsibility                   |
| :---------------- | :-------------------- | :------------------------------- |
| **Presentation**  | Vue 3 + Vite          | User interface, display greeting |
| **API Gateway**   | FastAPI               | RESTful endpoint, JSON responses |
| **Configuration** | Environment Variables | API URLs, settings               |

**Component Design:**

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Vue 3)                        │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   App.vue       │───▶│  GreetingCard   │                │
│  │   (Entry)       │    │  (Component)    │                │
│  └─────────────────┘    └────────┬────────┘                │
│                                  │ fetch()                  │
└──────────────────────────────────┼──────────────────────────┘
                                   │ HTTP GET /api/greet?name=
┌──────────────────────────────────┼──────────────────────────┐
│                     BACKEND (FastAPI)                       │
│                         ┌────────▼────────┐                │
│                         │  /api/greet     │                │
│                         │  (Endpoint)     │                │
│                         └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

**API Design:**

| Method | Endpoint      | Request      | Response                      |
| :----- | :------------ | :----------- | :---------------------------- |
| GET    | `/api/greet`  | `?name=John` | `{"message": "Hello, John!"}` |
| GET    | `/api/health` | —            | `{"status": "ok"}`            |

#### 📋 Requirements

**Epic 1: Project Setup & Configuration**

| ID     | User Story                                                                       | Features                                          | Acceptance Criteria                           |
| :----- | :------------------------------------------------------------------------------- | :------------------------------------------------ | :-------------------------------------------- |
| US-1.1 | As a developer, I want to set up a Vue project so I can build the frontend       | Vite initialization, folder structure, dev server | `npm run dev` starts server on port 5173      |
| US-1.2 | As a developer, I want to set up a FastAPI project so I can build the backend    | Project structure, main.py, uvicorn               | `uvicorn main:app` starts server on port 8000 |
| US-1.3 | As a developer, I want environment variables configured so I can manage settings | `.env` files, Pydantic Settings                   | API URL configurable via environment          |

**Epic 2: API Communication**

| ID     | User Story                                                          | Features                         | Acceptance Criteria                        |
| :----- | :------------------------------------------------------------------ | :------------------------------- | :----------------------------------------- |
| US-2.1 | As a user, I want to see a greeting message when I visit the app    | Greeting endpoint, JSON response | API returns `{"message": "Hello, World!"}` |
| US-2.2 | As a user, I want to enter my name and see a personalized greeting  | Query parameter handling         | `?name=Alice` returns "Hello, Alice!"      |
| US-2.3 | As a developer, I want CORS configured so frontend can call backend | CORS middleware                  | No CORS errors in browser console          |

**Epic 3: E2E Testing**

| ID     | User Story                                       | Features               | Acceptance Criteria                       |
| :----- | :----------------------------------------------- | :--------------------- | :---------------------------------------- |
| US-3.1 | As a developer, I want to test the greeting flow | Playwright test script | Test verifies greeting text matches input |

**Epic 4: Containerization**

| ID     | User Story                                        | Features           | Acceptance Criteria                         |
| :----- | :------------------------------------------------ | :----------------- | :------------------------------------------ |
| US-4.1 | As a developer, I want to containerize my app     | Dockerfiles        | Images build successfully for both services |
| US-4.2 | As a developer, I want to run with Docker Compose | docker-compose.yml | `docker-compose up` starts whole stack      |

**Feature Checklist:**

- [ ] F1.1: Vite project with TypeScript template
- [ ] F1.2: FastAPI project with `main.py`
- [ ] F1.3: GET `/api/greet` endpoint
- [ ] F1.4: Query parameter `name` support
- [ ] F1.5: CORS middleware configuration
- [ ] F1.6: Vue component fetches and displays greeting
- [ ] F1.7: Environment variables for API URL
- [ ] F1.8: Health check endpoint
- [ ] F1.9: Playwright test for greeting message
- [ ] F1.10: Dockerfile for Frontend & Backend
- [ ] F1.11: Docker Compose configuration

#### 📚 New Concepts Introduced

| Lesson | Topic                       | Frontend (Vue)           | Backend (FastAPI)            | Testing (Playwright/Python)              | Deployment (Docker/CI-CD)         |
| :----: | :-------------------------- | :----------------------- | :--------------------------- | :--------------------------------------- | :-------------------------------- |
|   1    | Project Structure & Setup   | Vite project creation    | FastAPI project structure    | —                                        | —                                 |
|   2    | Your First API Call         | `fetch()` in `onMounted` | JSON response                | —                                        | —                                 |
|   3    | CORS & Development Workflow | —                        | CORSMiddleware configuration | —                                        | —                                 |
|   4    | Environment Variables       | `.env` files             | Pydantic Settings            | —                                        | —                                 |
|   5    | E2E Testing Foundation      | —                        | —                            | `pytest-playwright` setup, `page.goto()` | —                                 |
|   6    | Containerization Basics     | Dockerizing Vue (Nginx)  | Dockerizing FastAPI          | —                                        | Dockerfile, Docker Compose basics |

#### ✅ Project Deliverable

- Vue app displays "Hello, [Name]!" fetched from FastAPI
- Name is passed as a query parameter
- Playwright test validates the full-stack flow
- Project runs inside Docker containers via Docker Compose
- Both frontend and backend run locally with proper CORS

---

### 📝 Project 2: Task Manager (CRUD Fundamentals)

**What You'll Build:** A task list with create, read, update, and delete operations.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology       | Responsibility                    |
| :--------------- | :--------------- | :-------------------------------- |
| **Presentation** | Vue 3 Components | Task list UI, forms, interactions |
| **State**        | Pinia Store      | Local task state management       |
| **API**          | FastAPI          | CRUD endpoints, validation        |
| **Data**         | In-Memory List   | Temporary storage (no DB yet)     |

**Component Design:**

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Vue 3)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ TaskForm    │  │ TaskList    │  │ TaskItem            │ │
│  │ (Add/Edit)  │  │ (Container) │──▶│ (Single Task)       │ │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────┘ │
│         │                │                                  │
│         └────────────────┼──────────────────────────────────│
│                          │ Pinia Store                      │
│                    ┌─────▼─────┐                            │
│                    │ taskStore │                            │
│                    └─────┬─────┘                            │
└──────────────────────────┼──────────────────────────────────┘
                           │ HTTP (CRUD)
┌──────────────────────────┼──────────────────────────────────┐
│                     BACKEND (FastAPI)                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ GET /   │  │ POST /  │  │ PUT /:id│  │DELETE/:id│        │
│  │ tasks   │  │ tasks   │  │         │  │         │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘
```

**Data Model:**

| Field        | Type       | Description                        |
| :----------- | :--------- | :--------------------------------- |
| `id`         | `int`      | Auto-generated unique ID           |
| `title`      | `str`      | Task title (required)              |
| `completed`  | `bool`     | Completion status (default: false) |
| `created_at` | `datetime` | Timestamp of creation              |

**API Design:**

| Method | Endpoint          | Request Body           | Response | Status  |
| :----- | :---------------- | :--------------------- | :------- | :------ |
| GET    | `/api/tasks`      | —                      | `Task[]` | 200     |
| GET    | `/api/tasks/{id}` | —                      | `Task`   | 200/404 |
| POST   | `/api/tasks`      | `{title: str}`         | `Task`   | 201     |
| PUT    | `/api/tasks/{id}` | `{title?, completed?}` | `Task`   | 200/404 |
| DELETE | `/api/tasks/{id}` | —                      | —        | 204/404 |

#### 📋 Requirements

**Epic 1: Task Display**

| ID     | User Story                                               | Features                       | Acceptance Criteria                     |
| :----- | :------------------------------------------------------- | :----------------------------- | :-------------------------------------- |
| US-1.1 | As a user, I want to see all my tasks in a list          | Task list component, API fetch | All tasks render with titles visible    |
| US-1.2 | As a user, I want to see empty state when no tasks exist | Empty state message            | "No tasks yet" shows when list is empty |
| US-1.3 | As a user, I want to see task completion status          | Checkbox or visual indicator   | Completed tasks visually distinct       |

**Epic 2: Task Creation**

| ID     | User Story                                   | Features                   | Acceptance Criteria                      |
| :----- | :------------------------------------------- | :------------------------- | :--------------------------------------- |
| US-2.1 | As a user, I want to add a new task          | Input field, submit button | New task appears in list after creation  |
| US-2.2 | As a user, I want validation on task title   | Required field, min length | Cannot submit empty or too-short title   |
| US-2.3 | As a user, I want input cleared after adding | Form reset                 | Input field empties after successful add |

**Epic 3: Task Updates**

| ID     | User Story                                           | Features               | Acceptance Criteria               |
| :----- | :--------------------------------------------------- | :--------------------- | :-------------------------------- |
| US-3.1 | As a user, I want to mark a task complete/incomplete | Toggle button/checkbox | Status updates immediately in UI  |
| US-3.2 | As a user, I want to edit a task title               | Edit mode, save button | Title updates after save          |
| US-3.3 | As a user, I want to cancel editing                  | Cancel button, ESC key | Original title restored on cancel |

**Epic 4: Task Deletion**

| ID     | User Story                                   | Features                  | Acceptance Criteria          |
| :----- | :------------------------------------------- | :------------------------ | :--------------------------- |
| US-4.1 | As a user, I want to delete a task           | Delete button             | Task removed from list       |
| US-4.2 | As a user, I want confirmation before delete | Confirm dialog (optional) | Prevents accidental deletion |

**Epic 5: E2E Testing**

| ID     | User Story                                | Features               | Acceptance Criteria                  |
| :----- | :---------------------------------------- | :--------------------- | :----------------------------------- |
| US-5.1 | As a developer, I want to test CRUD flows | Playwright test script | Test verifies create, update, delete |

**Epic 6: Continuous Integration**

| ID     | User Story                                             | Features                 | Acceptance Criteria                    |
| :----- | :----------------------------------------------------- | :----------------------- | :------------------------------------- |
| US-6.1 | As a developer, I want automated linting on every push | GitHub Actions (Linters) | Workflow fails if linting issues exist |
| US-6.2 | As a developer, I want automated tests on every push   | GitHub Actions (Pytest)  | Workflow fails if tests do not pass    |

**Feature Checklist:**

- [ ] F2.1: Task Pydantic schema with validation
- [ ] F2.2: In-memory task storage with auto-increment ID
- [ ] F2.3: GET `/api/tasks` returns all tasks
- [ ] F2.4: GET `/api/tasks/{id}` returns single task or 404
- [ ] F2.5: POST `/api/tasks` creates and returns new task
- [ ] F2.6: PUT `/api/tasks/{id}` updates task
- [ ] F2.7: DELETE `/api/tasks/{id}` removes task
- [ ] F2.8: Vue TaskList component with `v-for`
- [ ] F2.9: Vue TaskForm component with `v-model`
- [ ] F2.10: Pinia store for task state
- [ ] F2.11: Optimistic UI updates
- [ ] F2.12: Playwright tests for full CRUD flow
- [ ] F2.13: GitHub Actions workflow for YAML lint/test

#### 📚 New Concepts Introduced

| Lesson | Topic                       | Frontend (Vue)                     | Backend (FastAPI)         | Testing (Playwright/Python)               | Deployment (Docker/CI-CD)         |
| :----: | :-------------------------- | :--------------------------------- | :------------------------ | :---------------------------------------- | :-------------------------------- |
|   7    | Reactive State with `ref()` | Declaring reactive variables       | —                         | —                                         | —                                 |
|   8    | Path & Query Parameters     | Fetching single items by ID        | Path params, Query params | —                                         | —                                 |
|   9    | Request Bodies & Pydantic   | Sending JSON POST requests         | Pydantic models           | —                                         | —                                 |
|   10   | List Rendering with `v-for` | Rendering arrays, `:key` attribute | Returning lists           | —                                         | —                                 |
|   11   | Event Handling & Forms      | `@click`, `v-model`                | POST/PUT endpoints        | —                                         | —                                 |
|   12   | Completing CRUD             | DELETE requests                    | DELETE endpoint           | —                                         | —                                 |
|   13   | Testing CRUD Operations     | —                                  | —                         | `page.fill()`, `page.click()`, assertions | —                                 |
|   14   | GitHub Actions CI           | —                                  | —                         | —                                         | YAML syntax, lint/test automation |

#### ✅ Project Deliverable

- Vue app with task input field and list display
- Tasks persist via API (in-memory list for now)
- Full CRUD: Add, view, edit, delete tasks
- Playwright tests for all CRUD operations
- GitHub Actions CI pipeline running tests and linting

---

### 💾 Project 3: Notes App with Database (Persistence)

**What You'll Build:** A note-taking app with SQLite/PostgreSQL backend storage.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology        | Responsibility                       |
| :--------------- | :---------------- | :----------------------------------- |
| **Presentation** | Vue 3 Components  | Note list, editor, loading states    |
| **State**        | Pinia Store       | Notes cache, loading/error flags     |
| **API**          | FastAPI Routers   | CRUD endpoints with async operations |
| **ORM**          | SQLAlchemy 2.0    | Database models, async queries       |
| **Database**     | SQLite/PostgreSQL | Persistent data storage              |

**Component Design:**

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Vue 3)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ NoteEditor  │  │ NoteList    │  │ LoadingSpinner      │ │
│  │             │  │             │  │ ErrorMessage        │ │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────┘ │
│         │                │                                  │
│    ┌────▼────────────────▼────┐                            │
│    │     useFetch Composable   │                            │
│    └────────────┬──────────────┘                            │
└─────────────────┼───────────────────────────────────────────┘
                  │ HTTP (async)
┌─────────────────┼───────────────────────────────────────────┐
│                 │    BACKEND (FastAPI)                      │
│           ┌─────▼─────┐                                     │
│           │  Routers  │                                     │
│           └─────┬─────┘                                     │
│           ┌─────▼─────┐                                     │
│           │  Schemas  │ (Pydantic)                          │
│           └─────┬─────┘                                     │
│           ┌─────▼─────┐                                     │
│           │  Models   │ (SQLAlchemy)                        │
│           └─────┬─────┘                                     │
└─────────────────┼───────────────────────────────────────────┘
                  │
            ┌─────▼─────┐
            │  Database │
            │ (SQLite)  │
            └───────────┘
```

**Data Model:**

| Field        | Type          | Constraints                 |
| :----------- | :------------ | :-------------------------- |
| `id`         | `Integer`     | Primary Key, Auto-increment |
| `title`      | `String(200)` | Not Null                    |
| `content`    | `Text`        | Nullable                    |
| `created_at` | `DateTime`    | Default: now()              |
| `updated_at` | `DateTime`    | On update: now()            |

**API Design:**

| Method | Endpoint          | Description     | Response       |
| :----- | :---------------- | :-------------- | :------------- |
| GET    | `/api/notes`      | List all notes  | `Note[]` + 200 |
| GET    | `/api/notes/{id}` | Get single note | `Note` / 404   |
| POST   | `/api/notes`      | Create note     | `Note` + 201   |
| PATCH  | `/api/notes/{id}` | Update note     | `Note` + 200   |
| DELETE | `/api/notes/{id}` | Delete note     | 204            |

#### 📋 Requirements

**Epic 1: Database Integration**

| ID     | User Story                                                        | Features                    | Acceptance Criteria            |
| :----- | :---------------------------------------------------------------- | :-------------------------- | :----------------------------- |
| US-1.1 | As a developer, I want SQLAlchemy configured for async operations | Async engine, session maker | Database connection works      |
| US-1.2 | As a developer, I want database models defined                    | SQLAlchemy models           | Models create tables correctly |
| US-1.3 | As a developer, I want migrations for schema changes              | Alembic setup               | Can run `alembic upgrade head` |

**Epic 2: Persistent CRUD**

| ID     | User Story                                    | Features        | Acceptance Criteria           |
| :----- | :-------------------------------------------- | :-------------- | :---------------------------- |
| US-2.1 | As a user, I want my notes saved permanently  | Database INSERT | Notes survive server restart  |
| US-2.2 | As a user, I want to see all my saved notes   | Database SELECT | All notes load on page visit  |
| US-2.3 | As a user, I want to update my notes          | Database UPDATE | Changes persist after refresh |
| US-2.4 | As a user, I want to delete notes permanently | Database DELETE | Deleted notes don't reappear  |

**Epic 3: User Experience**

| ID     | User Story                                  | Features                    | Acceptance Criteria            |
| :----- | :------------------------------------------ | :-------------------------- | :----------------------------- |
| US-3.1 | As a user, I want to see loading indicators | Loading state, spinner      | Spinner shows during API calls |
| US-3.2 | As a user, I want clear error messages      | Error state, error display  | Friendly message on failure    |
| US-3.3 | As a user, I want smooth transitions        | Skeleton loaders (optional) | No content flashing            |

**Epic 4: E2E Testing**

| ID     | User Story                                      | Features               | Acceptance Criteria                |
| :----- | :---------------------------------------------- | :--------------------- | :--------------------------------- |
| US-4.1 | As a developer, I want to test data persistence | Playwright test script | Verifies data survives page reload |

**Epic 5: Staging Deployment**

| ID     | User Story                                              | Features                  | Acceptance Criteria                         |
| :----- | :------------------------------------------------------ | :------------------------ | :------------------------------------------ |
| US-5.1 | As a developer, I want my app deployed to a staging URL | Render/Railway Setup      | Prototype accessible via public link        |
| US-5.2 | As a developer, I want a managed database               | Cloud Database (Postgres) | App connects to managed DB instead of local |

**Feature Checklist:**

- [ ] F3.1: SQLAlchemy 2.0 async engine setup
- [ ] F3.2: Note model with all fields
- [ ] F3.3: Alembic migrations configured
- [ ] F3.4: Async CRUD operations with `select()`
- [ ] F3.5: Database session dependency
- [ ] F3.6: `useFetch` composable with loading/error
- [ ] F3.7: Loading spinner component
- [ ] F3.8: Error message display
- [ ] F3.9: Timestamps auto-update
- [ ] F3.10: Playwright tests for persistent notes
- [ ] F3.11: Render/Railway blueprint configuration

#### 📚 New Concepts Introduced

| Lesson | Topic                            | Frontend (Vue)            | Backend (FastAPI)            | Testing (Playwright/Python)                   | Deployment (Docker/CI-CD)                     |
| :----: | :------------------------------- | :------------------------ | :--------------------------- | :-------------------------------------------- | :-------------------------------------------- |
|   15   | Database Setup                   | —                         | SQLAlchemy 2.0 setup         | —                                             | —                                             |
|   16   | Models & Migrations              | —                         | SQLAlchemy models, Alembic   | —                                             | —                                             |
|   17   | Database CRUD Operations         | —                         | Create, Read with `select()` | —                                             | —                                             |
|   18   | Update & Delete with Database    | —                         | Update/Delete queries        | —                                             | —                                             |
|   19   | Loading & Error States           | `isLoading`, `error` refs | —                            | —                                             | —                                             |
|   20   | Building a `useFetch` Composable | Reusable fetching logic   | —                            | —                                             | —                                             |
|   21   | Testing Persistence              | —                         | —                            | Testing data lifecycle, DB state verification | —                                             |
|   22   | Cloud Staging Deployment         | —                         | Database as a Service        | —                                             | Cloud deployment (Render), Managed PostgreSQL |

#### ✅ Project Deliverable

- Notes stored in a real database
- Vue app shows loading spinners and error messages
- Reusable `useFetch` composable for API calls
- Playwright tests verifying end-to-end persistence
- Application deployed and running on a staging cloud platform

---

### 🏷️ Project 4: Categorized Notes (Relationships & Filtering)

**What You'll Build:** Extend the notes app with categories, tags, and search functionality.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology        | Responsibility                       |
| :--------------- | :---------------- | :----------------------------------- |
| **Presentation** | Vue 3 Components  | Filter UI, search bar, category tabs |
| **State**        | Pinia + Computed  | Filtered/searched results            |
| **API**          | FastAPI           | Query-based filtering, search        |
| **ORM**          | SQLAlchemy        | Relationships, LIKE queries          |
| **Database**     | SQLite/PostgreSQL | Notes, Categories tables             |

**Data Model (ERD):**

```
┌─────────────────┐       ┌─────────────────┐
│    Category     │       │      Note       │
├─────────────────┤       ├─────────────────┤
│ id         (PK) │───┐   │ id         (PK) │
│ name            │   │   │ title           │
│ color           │   │   │ content         │
│ created_at      │   └──▶│ category_id (FK)│
└─────────────────┘       │ created_at      │
                          │ updated_at      │
                          └─────────────────┘
```

**API Design:**

| Method | Endpoint          | Query Params                    | Description        |
| :----- | :---------------- | :------------------------------ | :----------------- |
| GET    | `/api/notes`      | `?category_id=1&search=keyword` | Filtered notes     |
| GET    | `/api/categories` | —                               | All categories     |
| POST   | `/api/categories` | —                               | Create category    |
| GET    | `/api/notes/{id}` | —                               | Note with category |

#### 📋 Requirements

**Epic 1: Categories**

| ID     | User Story                                            | Features                       | Acceptance Criteria                 |
| :----- | :---------------------------------------------------- | :----------------------------- | :---------------------------------- |
| US-1.1 | As a user, I want to create categories                | Category CRUD                  | Can create "Work", "Personal", etc. |
| US-1.2 | As a user, I want to assign notes to categories       | Category dropdown in note form | Note saves with category_id         |
| US-1.3 | As a user, I want to see category name with each note | Nested data response           | JSON includes category object       |

**Epic 2: Filtering**

| ID     | User Story                                    | Features               | Acceptance Criteria       |
| :----- | :-------------------------------------------- | :--------------------- | :------------------------ |
| US-2.1 | As a user, I want to filter notes by category | Category tabs/dropdown | Only matching notes shown |
| US-2.2 | As a user, I want to clear filters            | "All" option           | Shows all notes again     |
| US-2.3 | As a user, I want URL to reflect filters      | Query params sync      | Shareable filtered URLs   |

**Epic 3: Search**

| ID     | User Story                                 | Features         | Acceptance Criteria              |
| :----- | :----------------------------------------- | :--------------- | :------------------------------- |
| US-3.1 | As a user, I want to search notes by title | Search input     | Matches partial titles           |
| US-3.2 | As a user, I want to search note content   | Full-text search | Finds text in content            |
| US-3.3 | As a user, I want instant search results   | Debounced input  | Results update as I type (300ms) |

**Epic 4: E2E Testing**

| ID     | User Story                                          | Features               | Acceptance Criteria                             |
| :----- | :-------------------------------------------------- | :--------------------- | :---------------------------------------------- |
| US-4.1 | As a developer, I want to test filtering and search | Playwright test script | Verifies results change based on filters/search |

**Epic 5: Continuous Deployment**

| ID     | User Story                                              | Features               | Acceptance Criteria                                |
| :----- | :------------------------------------------------------ | :--------------------- | :------------------------------------------------- |
| US-5.1 | As a developer, I want my app to redeploy on every push | GitHub Actions (CD)    | Successful merge to main triggers production build |
| US-5.2 | As a developer, I want to use production Docker images  | Docker Registry (GHCR) | Images are built and stored in cloud registry      |

**Feature Checklist:**

- [ ] F4.1: Category model with color field
- [ ] F4.2: One-to-many relationship (Category→Notes)
- [ ] F4.3: Nested Pydantic response schemas
- [ ] F4.4: Filter query parameter handling
- [ ] F4.5: ILIKE/LIKE search queries
- [ ] F4.6: Vue `computed()` for client-side filtering
- [ ] F4.7: Debounced search input
- [ ] F4.8: Category filter tabs/buttons
- [ ] F4.9: Playwright tests for search and filtering
- [ ] F4.10: GitHub Actions CD workflow

#### 📚 New Concepts Introduced

| Lesson | Topic                             | Frontend (Vue)                  | Backend (FastAPI)                  | Testing (Playwright/Python)                       | Deployment (Docker/CI-CD)                        |
| :----: | :-------------------------------- | :------------------------------ | :--------------------------------- | :------------------------------------------------ | :----------------------------------------------- |
|   23   | One-to-Many Relationships         | —                               | Foreign keys, relationship loading | —                                                 | —                                                |
|   24   | Computed Properties for Filtering | `computed()` for filtered lists | Query parameters                   | —                                                 | —                                                |
|   25   | Search Implementation             | Debounced search input          | LIKE queries                       | —                                                 | —                                                |
|   26   | Nested Pydantic Models            | Displaying nested data          | Nested response schemas            | —                                                 | —                                                |
|   27   | Testing Search & Filtering        | —                               | —                                  | `page.get_by_placeholder()`, filtering assertions | —                                                |
|   28   | Continuous Deployment (CD)        | —                               | —                                  | —                                                 | GitHub Actions CD, Docker Build & Push, Webhooks |

#### ✅ Project Deliverable

- Notes have categories (one-to-many)
- Filter notes by category
- Search notes by title/content
- Proper nested JSON responses
- Playwright tests for search and category filtering
- Automated CD pipeline that builds Docker images and deploys to cloud

---

### 🔐 Project 5: User Authentication System

**What You'll Build:** Complete user registration, login, and protected routes.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology            | Responsibility                      |
| :--------------- | :-------------------- | :---------------------------------- |
| **Presentation** | Vue 3 Components      | Login/Register forms, auth state UI |
| **State**        | Pinia Auth Store      | Tokens, user data, isAuthenticated  |
| **Routing**      | Vue Router Guards     | Route protection, redirects         |
| **API**          | FastAPI + OAuth2      | Auth endpoints, token generation    |
| **Security**     | Passlib + python-jose | Password hashing, JWT tokens        |
| **Database**     | SQLAlchemy            | User model storage                  |

**Authentication Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                     REGISTRATION FLOW                       │
│  User ──▶ Register Form ──▶ POST /auth/register ──▶ Hash   │
│           Password ──▶ Save User ──▶ Return Success        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      LOGIN FLOW                             │
│  User ──▶ Login Form ──▶ POST /auth/token ──▶ Verify      │
│           Password ──▶ Generate JWT ──▶ Return Tokens      │
│                                   │                         │
│  Store in Pinia ◀─────────────────┘                        │
│  Store in localStorage                                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   PROTECTED REQUEST FLOW                    │
│  Vue ──▶ Add Authorization Header ──▶ FastAPI ──▶ Decode  │
│          JWT ──▶ Get Current User ──▶ Return Protected Data │
└─────────────────────────────────────────────────────────────┘
```

**Data Model:**

| Field             | Type          | Constraints             |
| :---------------- | :------------ | :---------------------- |
| `id`              | `Integer`     | Primary Key             |
| `email`           | `String(255)` | Unique, Not Null, Index |
| `hashed_password` | `String(255)` | Not Null                |
| `is_active`       | `Boolean`     | Default: True           |
| `created_at`      | `DateTime`    | Default: now()          |

**API Design:**

| Method | Endpoint             | Request Body                | Response                        | Auth |
| :----- | :------------------- | :-------------------------- | :------------------------------ | :--- |
| POST   | `/api/auth/register` | `{email, password}`         | `User` + 201                    | No   |
| POST   | `/api/auth/token`    | `OAuth2PasswordRequestForm` | `{access_token, refresh_token}` | No   |
| POST   | `/api/auth/refresh`  | `{refresh_token}`           | `{access_token}`                | No   |
| GET    | `/api/auth/me`       | —                           | `User`                          | Yes  |
| POST   | `/api/auth/logout`   | —                           | 204                             | Yes  |

**Token Structure:**

| Field  | Description           |
| :----- | :-------------------- |
| `sub`  | User ID (subject)     |
| `exp`  | Expiration timestamp  |
| `type` | "access" or "refresh" |

#### 📋 Requirements

**Epic 1: User Registration**

| ID     | User Story                                          | Features                | Acceptance Criteria             |
| :----- | :-------------------------------------------------- | :---------------------- | :------------------------------ |
| US-1.1 | As a visitor, I want to create an account           | Registration form       | Account created with valid data |
| US-1.2 | As a visitor, I want email validation               | Email format check      | Invalid emails rejected         |
| US-1.3 | As a visitor, I want password requirements shown    | Password strength rules | Min 8 chars, complexity hints   |
| US-1.4 | As a visitor, I cannot register with existing email | Unique constraint       | Error message for duplicates    |

**Epic 2: User Login**

| ID     | User Story                                      | Features                | Acceptance Criteria              |
| :----- | :---------------------------------------------- | :---------------------- | :------------------------------- |
| US-2.1 | As a user, I want to log in with email/password | Login form, OAuth2 flow | Successful login returns tokens  |
| US-2.2 | As a user, I want to see login errors           | Error handling          | Wrong credentials show message   |
| US-2.3 | As a user, I want to stay logged in             | Token storage           | Refresh on page keeps session    |
| US-2.4 | As a user, I want to log out                    | Logout endpoint         | Tokens cleared, redirect to home |

**Epic 3: Protected Routes**

| ID     | User Story                                       | Features        | Acceptance Criteria            |
| :----- | :----------------------------------------------- | :-------------- | :----------------------------- |
| US-3.1 | As a user, I want to access my dashboard         | Protected route | Only accessible when logged in |
| US-3.2 | As a visitor, I am redirected to login           | Router guards   | Protected routes redirect      |
| US-3.3 | As a user, I want my identity sent with requests | Auth header     | Bearer token in API calls      |

**Epic 6: Security & Secrets**

| ID     | User Story                                           | Features               | Acceptance Criteria                          |
| :----- | :--------------------------------------------------- | :--------------------- | :------------------------------------------- |
| US-6.1 | As a developer, I want to store JWT secrets securely | GitHub Secrets         | Secrets not committed to git, loaded via ENV |
| US-6.2 | As a developer, I want to inject production secrets  | CI/CD Secret Injection | App runs with production keys in cloud       |

**Epic 5: E2E Testing**

| ID     | User Story                                          | Features               | Acceptance Criteria                            |
| :----- | :-------------------------------------------------- | :--------------------- | :--------------------------------------------- |
| US-5.1 | As a developer, I want to test authentication flows | Playwright test script | Verifies register, login, and protected access |

**Feature Checklist:**

- [ ] F5.1: User SQLAlchemy model
- [ ] F5.2: Password hashing with Passlib bcrypt
- [ ] F5.3: Registration endpoint with email uniqueness
- [ ] F5.4: OAuth2PasswordBearer token endpoint
- [ ] F5.5: JWT access token creation (python-jose)
- [ ] F5.6: JWT refresh token creation
- [ ] F5.7: `get_current_user` dependency
- [ ] F5.8: Pinia auth store with `isAuthenticated` getter
- [ ] F5.9: Login/Register Vue components
- [ ] F5.10: Vue Router `beforeEach` guard
- [ ] F5.11: Axios interceptor for auth headers
- [ ] F5.12: Token refresh logic
- [ ] F5.13: Logout with state clear
- [ ] F5.14: Playwright tests for Auth (Login/Logout/Register)
- [ ] F5.15: Production secrets configuration in Cloud UI

#### 📚 New Concepts Introduced

| Lesson | Topic                  | Frontend (Vue)    | Backend (FastAPI)    | Testing (Playwright/Python)             | Deployment (Docker/CI-CD)    |
| :----: | :--------------------- | :---------------- | :------------------- | :-------------------------------------- | :--------------------------- |
|   25   | Password Hashing       | —                 | Passlib, bcrypt      | —                                       | —                            |
|   26   | User Registration      | Registration form | User creation        | —                                       | —                            |
|   27   | JWT Token Basics       | Storage options   | python-jose          | —                                       | —                            |
|   28   | Login Flow             | Token storage     | OAuth2PasswordBearer | —                                       | —                            |
|   29   | Auth State with Pinia  | Auth store        | —                    | —                                       | —                            |
|   30   | Protected API Routes   | Auth headers      | Dependency Injection | —                                       | —                            |
|   31   | Vue Router Guards      | `beforeEach`      | —                    | —                                       | —                            |
|   32   | Token Refresh & Logout | Refresh flow      | Token invalidation   | —                                       | —                            |
|   33   | Testing Authentication | —                 | —                    | Testing login flow, redirects, and 401s | —                            |
|   34   | Secrets Management     | —                 | —                    | —                                       | GitHub Secrets, ENV security |

#### ✅ Project Deliverable

- User registration with email/password
- Login returns JWT tokens
- Protected routes require authentication
- Vue Router guards redirect unauthenticated users
- Logout clears tokens and redirects
- Playwright tests covering registration and login

---

### 📰 Project 6: Personal Blog Platform

**What You'll Build:** A full-featured blog with posts, authors, and public/private content.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology         | Responsibility                   |
| :--------------- | :----------------- | :------------------------------- |
| **Presentation** | Vue 3 + Vue Router | Multi-page blog UI, post pages   |
| **State**        | Pinia              | Posts cache, pagination state    |
| **API**          | FastAPI            | Blog CRUD, pagination, ownership |
| **Database**     | SQLAlchemy         | Posts, Users with relationships  |

**Component Design:**

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Vue 3)                        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────────────┐   │
│  │  HomePage   │  │ PostDetail  │  │  Dashboard        │   │
│  │  (Public)   │  │  (Public)   │  │  (Protected)      │   │
│  └──────┬──────┘  └──────┬──────┘  └────────┬──────────┘   │
│         │                │                   │              │
│         └────────────────┼───────────────────┘              │
│                          │                                  │
│                    ┌─────▼─────┐                            │
│                    │ Vue Router│                            │
│                    │  Guards   │                            │
│                    └───────────┘                            │
└─────────────────────────────────────────────────────────────┘
```

**Data Model (ERD):**

```
┌─────────────────┐       ┌─────────────────┐
│      User       │       │      Post       │
├─────────────────┤       ├─────────────────┤
│ id         (PK) │───┐   │ id         (PK) │
│ email           │   │   │ title           │
│ name            │   │   │ content (Text)  │
│ avatar_url      │   └──▶│ author_id  (FK) │
└─────────────────┘       │ published       │
                          │ created_at      │
                          │ updated_at      │
                          └─────────────────┘
```

**API Design:**

| Method | Endpoint          | Query Params       | Auth        | Description      |
| :----- | :---------------- | :----------------- | :---------- | :--------------- |
| GET    | `/api/posts`      | `?page=1&limit=10` | No          | Public post list |
| GET    | `/api/posts/{id}` | —                  | No          | Single post      |
| POST   | `/api/posts`      | —                  | Yes         | Create post      |
| PATCH  | `/api/posts/{id}` | —                  | Yes (Owner) | Update post      |
| DELETE | `/api/posts/{id}` | —                  | Yes (Owner) | Delete post      |
| GET    | `/api/me/posts`   | —                  | Yes         | User's own posts |

#### 📋 Requirements

**Epic 1: Multi-Page Routing**

| ID     | User Story                                    | Features                   | Acceptance Criteria              |
| :----- | :-------------------------------------------- | :------------------------- | :------------------------------- |
| US-1.1 | As a visitor, I want to browse the homepage   | Public route               | Posts list visible without login |
| US-1.2 | As a visitor, I want to read individual posts | Dynamic route `/posts/:id` | Post detail page loads correctly |
| US-1.3 | As a user, I want to access my dashboard      | Protected route            | Requires authentication          |

**Epic 2: Blog Content Management**

| ID     | User Story                                | Features                       | Acceptance Criteria        |
| :----- | :---------------------------------------- | :----------------------------- | :------------------------- |
| US-2.1 | As an author, I want to create blog posts | Post editor, create endpoint   | New post appears in list   |
| US-2.2 | As an author, I want to edit my posts     | Edit form, PATCH endpoint      | Only owner can edit        |
| US-2.3 | As an author, I want to delete my posts   | Delete button, DELETE endpoint | Post removed from database |
| US-2.4 | As a visitor, I cannot edit others' posts | Ownership validation           | 403 for non-owners         |

**Epic 5: Automated Migrations**

| ID     | User Story                                            | Features             | Acceptance Criteria                                 |
| :----- | :---------------------------------------------------- | :------------------- | :-------------------------------------------------- |
| US-5.1 | As a developer, I want my DB schema updated on deploy | CI/CD Alembic runner | Deployment runs `alembic upgrade head` before start |
| US-5.2 | As a developer, I want to check for migration drift   | CI Check (Alembic)   | Workflow fails if local schema ≠ model definitions  |

**Epic 4: E2E Testing**

| ID     | User Story                                   | Features               | Acceptance Criteria                          |
| :----- | :------------------------------------------- | :--------------------- | :------------------------------------------- |
| US-4.1 | As a developer, I want to test the blog flow | Playwright test script | Verifies post creation and public visibility |

**Feature Checklist:**

- [ ] F6.1: Post SQLAlchemy model with author relationship
- [ ] F6.2: Vue Router with public and protected routes
- [ ] F6.3: Dynamic route `/posts/:id` with `useRoute()`
- [ ] F6.4: Owner-only edit/delete validation
- [ ] F6.5: Markdown rendering for post content
- [ ] F6.6: Pagination with LIMIT/OFFSET
- [ ] F6.7: Total count in API response
- [ ] F6.8: Page navigation component
- [ ] F6.9: Optional auth dependency for public routes
- [ ] F6.10: Playwright tests for Blog CRUD and Pagination
- [ ] F6.11: Production deployment script with migrations

#### 📚 New Concepts Introduced

| Lesson | Topic                     | Frontend (Vue)          | Backend (FastAPI)     | Testing (Playwright/Python)                      | Deployment (Docker/CI-CD)             |
| :----: | :------------------------ | :---------------------- | :-------------------- | :----------------------------------------------- | :------------------------------------ |
|   34   | Multi-Page Routing        | Vue Router setup        | —                     | —                                                | —                                     |
|   35   | Dynamic Routes            | `useRoute()` params     | —                     | —                                                | —                                     |
|   36   | Post CRUD with Ownership  | Author-only controls    | Owner validation      | —                                                | —                                     |
|   37   | Rich Text Content         | Markdown rendering      | Text storage          | —                                                | —                                     |
|   38   | Public vs. Private Routes | Mixed auth requirements | Optional dependencies | —                                                | —                                     |
|   39   | Pagination                | Page controls           | LIMIT/OFFSET          | —                                                | —                                     |
|   40   | Testing Complex Routing   | —                       | —                     | Testing navigation, dynamic URLs, and pagination | —                                     |
|   41   | Automated Migrations (CI) | —                       | —                     | —                                                | Alembic in CI/CD, migration rollbacks |

#### ✅ Project Deliverable

- Blog with list and detail views
- Only authors can edit their posts
- Public homepage, protected dashboard
- Paginated post listing
- Playwright tests for blog navigation and content management

---

### 📸 Project 7: Image Gallery with Uploads

**What You'll Build:** A gallery where users upload, view, and manage images.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology                     | Responsibility                            |
| :--------------- | :----------------------------- | :---------------------------------------- |
| **Presentation** | Vue 3 + Tailwind               | Gallery grid, upload modal, progress bars |
| **API**          | FastAPI                        | Multipart form parsing, validation        |
| **Storage**      | Local Filesystem / StaticFiles | Physical storage and public URL serving   |
| **Database**     | SQLAlchemy                     | Image metadata (filename, owner, size)    |

**Upload & Serving Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                     UPLOAD FLOW                             │
│  User ──▶ File Input ──▶ FormData ──▶ POST /api/upload ───┐│
│           (Client Validation)                             ││
│                                                           ││
│  ┌────────────────────────────────────────────────────────┘│
│  ▼                                                          │
│  FastAPI ──▶ Validate Type/Size ──▶ Save to Disk ──▶ Save DB│
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     SERVING FLOW                            │
│  Browser ──▶ GET /static/image.jpg ──▶ FastAPI StaticFiles │
│  (UI)        (Public URL)              (Path Mapping)       │
└─────────────────────────────────────────────────────────────┘
```

**Data Model:**

| Field           | Type      | Description                 |
| :-------------- | :-------- | :-------------------------- |
| `id`            | `Integer` | PK                          |
| `filename`      | `String`  | Unique name on disk         |
| `original_name` | `String`  | Original user filename      |
| `file_path`     | `String`  | Relative path for serving   |
| `mime_type`     | `String`  | image/jpeg, image/png, etc. |
| `size`          | `Integer` | File size in bytes          |
| `owner_id`      | `Integer` | FK to User                  |

**API Design:**

| Method | Endpoint             | Content-Type          | Response    | Auth        |
| :----- | :------------------- | :-------------------- | :---------- | :---------- |
| POST   | `/api/images/upload` | `multipart/form-data` | `Image`     | Yes         |
| GET    | `/api/images`        | —                     | `Image[]`   | No/Yes      |
| DELETE | `/api/images/{id}`   | —                     | 204         | Yes (Owner) |
| GET    | `/api/static/{path}` | —                     | File Stream | No          |

#### 📋 Requirements

**Epic 1: File Upload**

| ID     | User Story                                        | Features           | Acceptance Criteria              |
| :----- | :------------------------------------------------ | :----------------- | :------------------------------- |
| US-1.1 | As a user, I want to select images from my device | File input, DnD    | Only images visible in picker    |
| US-1.2 | As a user, I want to see upload progress          | Progress bar       | Updates in real-time %           |
| US-1.3 | As a user, I want to upload multiple files        | Batch upload       | All files processed sequentially |
| US-1.4 | As a dev, I want to validate file size/type       | Server-side checks | Rejects non-images or >5MB       |

**Epic 2: Gallery Display**

| ID     | User Story                                            | Features        | Acceptance Criteria       |
| :----- | :---------------------------------------------------- | :-------------- | :------------------------ |
| US-2.1 | As a user, I want to see a grid of my images          | Responsive grid | Images render in cards    |
| US-2.2 | As a user, I want to click an image to view full size | Lightbox/Modal  | Modal opens with high-res |
| US-2.3 | As a user, I want to see upload timestamps            | Date formatting | "Uploaded 2 mins ago"     |

**Epic 4: E2E Testing**

| ID     | User Story                                   | Features               | Acceptance Criteria                                   |
| :----- | :------------------------------------------- | :--------------------- | :---------------------------------------------------- |
| US-4.1 | As a developer, I want to test image uploads | Playwright test script | Verifies file selection and successful upload display |

**Epic 5: Asset Optimization & CDN**

| ID     | User Story                                       | Features                  | Acceptance Criteria                           |
| :----- | :----------------------------------------------- | :------------------------ | :-------------------------------------------- |
| US-5.1 | As a developer, I want my images to load quickly | Nginx Caching / CDN Setup | Assets are cached by edge servers             |
| US-5.2 | As a developer, I want to optimize image sizes   | Image compression (Opt)   | Uploaded images are resized/optimized for web |

**Feature Checklist:**

- [ ] F7.1: Image SQLAlchemy model
- [ ] F7.2: FastAPI `UploadFile` endpoint
- [ ] F7.3: Unique filename generation (UUIDs)
- [ ] F7.4: Static files mounting in FastAPI
- [ ] F7.5: Vue file upload component
- [ ] F7.6: `FormData` API integration
- [ ] F7.7: Axios upload progress tracking
- [ ] F7.8: Server-side MIME type validation
- [ ] F7.9: Image grid with DaisyUI cards
- [ ] F7.10: Playwright tests for Image Upload and Gallery
- [ ] F7.11: Nginx configuration for static assets

#### 📚 New Concepts Introduced

| Lesson | Topic                 | Frontend (Vue)      | Backend (FastAPI)    | Testing (Playwright/Python)           | Deployment (Docker/CI-CD) |
| :----: | :-------------------- | :------------------ | :------------------- | :------------------------------------ | :------------------------ |
|   41   | File Input Handling   | `input type="file"` | —                    | —                                     | —                         |
|   42   | Multipart Form Upload | `FormData`, axios   | `UploadFile`         | —                                     | —                         |
|   43   | File Storage          | —                   | Disk storage         | —                                     | —                         |
|   44   | Serving Static Files  | Displaying images   | `StaticFiles`        | —                                     | —                         |
|   45   | Upload Progress       | Progress bars       | —                    | —                                     | —                         |
|   46   | File Validation       | Client-side checks  | Server-side security | —                                     | —                         |
|   47   | Testing File Uploads  | —                   | —                    | `set_input_files()`, media assertions | —                         |
|   48   | Asset Optimization    | —                   | —                    | —                                     | Nginx Caching, CDN basics |

#### ✅ Project Deliverable

- Upload images via drag-and-drop or file picker
- Images stored on server, served statically
- Gallery grid view of all uploads
- Upload progress indicator
- File type and size validation
- Playwright tests for image upload and display gallery

---

### 🛒 Project 8: E-Commerce Product Catalog

**What You'll Build:** A product catalog with categories, images, and advanced filtering.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology      | Responsibility                           |
| :--------------- | :-------------- | :--------------------------------------- |
| **Presentation** | Vue 3 + DaisyUI | Product grid, multi-faceted filtering UI |
| **API**          | FastAPI         | Complex filtering logic, nested schemas  |
| **ORM**          | SQLAlchemy 2.0  | Join queries, association tables (M2M)   |
| **Database**     | PostgreSQL      | Relational data, indexed search fields   |

**Relational Design (ERD):**

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│    Category     │       │       Product        │       │       Tag       │
├─────────────────┤       ├──────────────────────┤       ├─────────────────┤
│ id         (PK) │───┐   │ id              (PK) │   ┌──▶│ id         (PK) │
│ name            │   │   │ name                 │   │   │ name            │
│ slug            │   └──▶│ category_id     (FK) │   │   └─────────────────┘
└─────────────────┘       │ base_price           │   │
                          │ stock_count          │   │   ┌─────────────────┐
                          ├──────────────────────┤   │   │   ProductTag    │
                          │   (Many-to-Many)     │───┘   ├─────────────────┤
                          │       tags           │       │ product_id (FK) │
                          └──────────────────────┘       │ tag_id     (FK) │
                                                         └─────────────────┘
```

**API Design:**

| Method | Endpoint            | Query Params                                          | Description          |
| :----- | :------------------ | :---------------------------------------------------- | :------------------- |
| GET    | `/api/product`      | `?cat=slug&tags=1,2&min_p=0&max_p=100&sort=price_asc` | Filtered list        |
| GET    | `/api/product/{id}` | —                                                     | Detail with variants |
| GET    | `/api/categories`   | —                                                     | Sidebar data         |
| GET    | `/api/tags`         | —                                                     | Filter options       |

#### 📋 Requirements

**Epic 1: Product Browsing**

| ID     | User Story                                       | Features          | Acceptance Criteria              |
| :----- | :----------------------------------------------- | :---------------- | :------------------------------- |
| US-1.1 | As a user, I want to see products in a grid      | Card grid layout  | Responsive 1/2/4 columns         |
| US-1.2 | As a user, I want to see price and stock clearly | Pricing display   | Shows "Out of Stock" labels      |
| US-1.3 | As a user, I want to view product details        | Detail page/modal | Shows description and all images |

**Epic 2: Advanced Filtering**

| ID     | User Story                                   | Features                | Acceptance Criteria            |
| :----- | :------------------------------------------- | :---------------------- | :----------------------------- |
| US-2.1 | As a user, I want to filter by price range   | Dual-range slider       | Only shows products in range   |
| US-2.2 | As a user, I want to filter by multiple tags | Multi-select checkboxes | Implements "IN" logic for tags |
| US-2.3 | As a user, I want to sort by price (hi/lo)   | Sort dropdown           | List re-orders instantly       |
| US-2.4 | As a user, I want to filter by category      | Sidebar navigation      | Deep linking via URL slug      |

**Epic 4: E2E Testing**

| ID     | User Story                                                | Features               | Acceptance Criteria                                     |
| :----- | :-------------------------------------------------------- | :--------------------- | :------------------------------------------------------ |
| US-4.1 | As a developer, I want to test advanced catalog filtering | Playwright test script | Verifies facets (price, tags) correctly filter products |

**Epic 5: Multi-Container CI/CD**

| ID     | User Story                                          | Features                  | Acceptance Criteria                             |
| :----- | :-------------------------------------------------- | :------------------------ | :---------------------------------------------- |
| US-5.1 | As a developer, I want to test with a real DB in CI | GitHub Actions + Postgres | CI runs tests against a real Postgres container |
| US-5.2 | As a developer, I want to build multi-arch images   | Buildx / QEMU             | Docker images work on ARM and x86               |

**Feature Checklist:**

- [ ] F8.1: Product, Category, and Tag SQLAlchemy models
- [ ] F8.2: Many-to-Many association table for Tags
- [ ] F8.3: Advanced query builder in FastAPI (Dynamic WHERE)
- [ ] F8.4: Nested Pydantic schemas for Product + Tags + Category
- [ ] F8.5: Vue Filter sidebar component
- [ ] F8.6: Range slider for price filtering
- [ ] F8.7: URL Query parameter sync for filters
- [ ] F8.8: "Reset Filters" functionality
- [ ] F8.9: Sorting implementation (SQL `order_by`)
- [ ] F8.10: Playwright tests for faceted search and filtering
- [ ] F8.11: GitHub Actions with Services (Postgres)

#### 📚 New Concepts Introduced

| Lesson | Topic                      | Frontend (Vue)       | Backend (FastAPI)      | Testing (Playwright/Python)                       | Deployment (Docker/CI-CD)            |
| :----: | :------------------------- | :------------------- | :--------------------- | :------------------------------------------------ | :----------------------------------- |
|   49   | Complex Data Schemas       | Product variants     | Pydantic validators    | —                                                 | —                                    |
|   50   | Many-to-Many Relationships | Tagging products     | Association tables     | —                                                 | —                                    |
|   51   | Advanced Filtering UI      | Multi-select filters | Complex query building | —                                                 | —                                    |
|   52   | Sorting & Ordering         | URL sync             | SQL `order_by`         | —                                                 | —                                    |
|   53   | Product Image Uploads      | Multiple images      | Multi-file handling    | —                                                 | —                                    |
|   54   | Responsive Product Grid    | Grid layouts         | —                      | —                                                 | —                                    |
|   55   | Faceted Search Testing     | —                    | —                      | Testing multi-filter state, sliders, and URL sync | —                                    |
|   56   | Multi-Container CI         | —                    | —                      | —                                                 | GitHub Actions Services (PostgreSQL) |

#### ✅ Project Deliverable

- Product catalog with images, categories, tags
- Filter by category, price range, tags
- Sort by price, date, name
- Responsive card grid layout
- Multi-image upload per product
- Playwright tests for advanced filtering and faceted search

---

### 🛍️ Project 9: Shopping Cart & Checkout

**What You'll Build:** Cart functionality, order placement, and order history.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology    | Responsibility                     |
| :--------------- | :------------ | :--------------------------------- |
| **Presentation** | Vue 3 + Pinia | Cart UI, drawer, checkout form     |
| **Persistence**  | LocalStorage  | Saving cart items across refreshes |
| **API**          | FastAPI       | Order creation, stock validation   |
| **Database**     | SQLAlchemy    | Orders, OrderItems tables          |

**Cart State Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Vue/Pinia)                    │
│  ┌─────────────┐        ┌─────────────┐       ┌────────────┐│
│  │ ProductCard │──Add──▶│  CartStore  │──Sync─▶│ Local      ││
│  │             │        │   (Pinia)   │       │ Storage    ││
│  └─────────────┘        └──────┬──────┘       └────────────┘│
│                                │                            │
│                                │ POST /api/orders           │
└────────────────────────────────┼────────────────────────────┘
                                 │
┌────────────────────────────────┼────────────────────────────┐
│                     BACKEND (FastAPI)                       │
│  ┌─────────────┐        ┌──────▼──────┐       ┌────────────┐│
│  │ Stock Check │◀─Validate─│ Order Logic │──Save─▶│ Database   ││
│  └─────────────┘        └─────────────┘       └────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Data Model:**

| Table         | Fields                                             | Description            |
| :------------ | :------------------------------------------------- | :--------------------- |
| **Order**     | `id, user_id, status, total, created_at`           | Header record          |
| **OrderItem** | `id, order_id, product_id, quantity, price_at_buy` | Line items (immutable) |

**API Design:**

| Method | Endpoint           | Request Body             | Response           | Auth        |
| :----- | :----------------- | :----------------------- | :----------------- | :---------- |
| POST   | `/api/orders`      | `{items: [{p_id, qty}]}` | `Order`            | Yes         |
| GET    | `/api/orders`      | —                        | `Order[]`          | Yes         |
| GET    | `/api/orders/{id}` | —                        | `Order` + `Item[]` | Yes (Owner) |

#### 📋 Requirements

**Epic 1: Shopping Cart**

| ID     | User Story                                  | Features                 | Acceptance Criteria             |
| :----- | :------------------------------------------ | :----------------------- | :------------------------------ |
| US-1.1 | As a user, I want to add items to my cart   | Add button, store action | Item count updates in header    |
| US-1.2 | As a user, I want to change item quantities | Qty selector in cart     | Subtotals update instantly      |
| US-1.3 | As a user, I want my cart saved             | LocalStorage sync        | Items remain after page refresh |
| US-1.4 | As a user, I want to remove items           | Remove button            | Cart total re-calculates        |

**Epic 2: Checkout Process**

| ID     | User Story                                         | Features            | Acceptance Criteria               |
| :----- | :------------------------------------------------- | :------------------ | :-------------------------------- |
| US-2.1 | As a user, I want to review my total before buying | Checkout summary    | Shows tax, shipping, total        |
| US-2.2 | As a user, I want to place an order                | Order submission    | Cart clears after successful POST |
| US-2.3 | As a user, I want stock validation                 | Backend check       | Error if qty > stock_available    |
| US-2.4 | As a user, I want to see my order confirmation     | Redirect to Success | Shows order ID and summary        |

**Epic 4: E2E Testing**

| ID     | User Story                                       | Features               | Acceptance Criteria                                   |
| :----- | :----------------------------------------------- | :--------------------- | :---------------------------------------------------- |
| US-4.1 | As a developer, I want to test the checkout flow | Playwright test script | Verifies cart-to-order transition and stock reduction |

**Epic 5: Health & Uptime Monitoring**

| ID     | User Story                                       | Features                   | Acceptance Criteria                             |
| :----- | :----------------------------------------------- | :------------------------- | :---------------------------------------------- |
| US-5.1 | As a developer, I want to monitor app health     | /api/health with DB check  | Returns 200 only if DB and App are responsive   |
| US-5.2 | As a developer, I want to be alerted on downtime | Uptime Robot / BetterStack | External service pings health check every 1 min |

**Feature Checklist:**

- [ ] F9.1: Pinia `cartStore` with state/getters/actions
- [ ] F9.2: LocalStorage persistence for Pinia state
- [ ] F9.3: Order and OrderItem SQLAlchemy models
- [ ] F9.4: Atomic transaction for order creation
- [ ] F9.5: Stock decrement logic in backend
- [ ] F9.6: Cart drawer component (DaisyUI)
- [ ] F9.7: Checkout form with summary
- [ ] F9.8: "My Orders" view with status badges
- [ ] F9.9: Clear cart after order placement
- [ ] F9.10: Playwright tests for Cart and Checkout Flow
- [ ] F9.11: Advanced /api/health with SQLAlchemy connection pool check

#### 📚 New Concepts Introduced

| Lesson | Topic                   | Frontend (Vue)     | Backend (FastAPI)     | Testing (Playwright/Python)                       | Deployment (Docker/CI-CD)          |
| :----: | :---------------------- | :----------------- | :-------------------- | :------------------------------------------------ | :--------------------------------- |
|   57   | Cart State Management   | Pinia cart actions | —                     | —                                                 | —                                  |
|   58   | Cart Persistence        | localStorage sync  | —                     | —                                                 | —                                  |
|   59   | Order Schema & Creation | Order submission   | Atomic transactions   | —                                                 | —                                  |
|   60   | Order History           | User past orders   | User-specific queries | —                                                 | —                                  |
|   61   | Cart-to-Order Flow      | Success redirects  | Stock validation      | —                                                 | —                                  |
|   62   | Order Status Updates    | Real-time UI feel  | Enum status           | —                                                 | —                                  |
|   63   | Testing E-Commerce Flow | —                  | —                     | Testing multi-step checkout and stock persistence | —                                  |
|   64   | Application Monitoring  | —                  | —                     | —                                                 | Health Checks, Uptime Alerts, Logs |

#### ✅ Project Deliverable

- Add/remove products from cart
- Cart persists across sessions
- Checkout creates an order
- View order history with status
- Order detail page
- Playwright tests for the full checkout and cart lifecycle

---

### 👥 Project 10: User Profiles & Social Features

**What You'll Build:** User profiles, avatars, and social interactions (likes/favorites).

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology          | Responsibility                               |
| :--------------- | :------------------ | :------------------------------------------- |
| **Presentation** | Vue 3 + Tailwind    | Profile pages, follow buttons, activity feed |
| **API**          | FastAPI             | Profile updates, social graph management     |
| **Logic**        | Aggregation Queries | Calculating follower counts, feeds           |
| **Database**     | SQLAlchemy          | Users, Likes, Follows (Self-Referential M2M) |

**Social Graph Design:**

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│      User       │       │    Follows (M2M)     │       │      Post       │
├─────────────────┤       ├──────────────────────┤       ├─────────────────┤
│ id         (PK) │◀──┐   │ follower_id     (FK) │       │ id         (PK) │
│ display_name    │   └───│ following_id    (FK) │       │ title           │
│ bio             │       └──────────────────────┘       └────────┬────────┘
│ avatar_url      │                                               │
└────────┬────────┘       ┌──────────────────────┐                │
         │                │     Likes (M2M)      │                │
         └───────────────▶│ user_id         (FK) │◀───────────────┘
                          │ post_id         (FK) │
                          └──────────────────────┘
```

**API Design:**

| Method | Endpoint                | Request Body          | Description         | Auth |
| :----- | :---------------------- | :-------------------- | :------------------ | :--- |
| PATCH  | `/api/user/profile`     | `{bio, name, avatar}` | Update profile      | Yes  |
| GET    | `/api/user/{id}`        | —                     | View public profile | No   |
| POST   | `/api/posts/{id}/like`  | —                     | Toggle like         | Yes  |
| POST   | `/api/user/{id}/follow` | —                     | Follow user         | Yes  |
| GET    | `/api/feed`             | `?limit=20`           | Personalized feed   | Yes  |

#### 📋 Requirements

**Epic 1: Profile Management**

| ID     | User Story                                 | Features              | Acceptance Criteria                  |
| :----- | :----------------------------------------- | :-------------------- | :----------------------------------- |
| US-1.1 | As a user, I want to edit my bio and name  | Profile update form   | Profile reflects changes             |
| US-1.2 | As a user, I want to upload an avatar      | Avatar upload modal   | Image replaces default placeholder   |
| US-1.3 | As a user, I want to view others' profiles | Profile view (Public) | Shows their bio/posts/follower count |

**Epic 2: Social Interactions**

| ID     | User Story                                            | Features                | Acceptance Criteria                 |
| :----- | :---------------------------------------------------- | :---------------------- | :---------------------------------- |
| US-2.1 | As a user, I want to "Like" posts                     | Like button, M2M record | Button color toggles, count updates |
| US-2.2 | As a user, I want to see how many likes a post has    | Like count              | Accurate count displayed            |
| US-2.3 | As a user, I want to follow other users               | Follow button           | Following relationship established  |
| US-2.4 | As a user, I want to see my follower/following counts | Stats display           | Counts updated on profile           |

**Epic 3: Activity Feed**

| ID     | User Story                                  | Features           | Acceptance Criteria             |
| :----- | :------------------------------------------ | :----------------- | :------------------------------ |
| US-3.1 | As a user, I want a personalized feed       | Aggregated feed    | Shows posts from users I follow |
| US-3.2 | As a user, I want to see my recent activity | Recent events list | Shows list of my likes/follows  |

**Epic 4: E2E Testing**

| ID     | User Story                                         | Features               | Acceptance Criteria                                   |
| :----- | :------------------------------------------------- | :--------------------- | :---------------------------------------------------- |
| US-4.1 | As a developer, I want to test social interactions | Playwright test script | Verifies likes and follows correctly update UI and DB |

**Epic 5: Performance & Error Tracking**

| ID     | User Story                                         | Features                 | Acceptance Criteria                                 |
| :----- | :------------------------------------------------- | :----------------------- | :-------------------------------------------------- |
| US-5.1 | As a developer, I want to track client-side errors | Sentry SDK Integration   | Exceptions in Vue components are reported to Sentry |
| US-5.2 | As a developer, I want to monitor API performance  | Sentry Performance / APM | Slow endpoints are flagged with traces              |

**Feature Checklist:**

- [ ] F10.1: Profile update endpoint with partial validation
- [ ] F10.2: Self-referential Many-to-Many for Follows
- [ ] F10.3: Many-to-Many for Post Likes
- [ ] F10.4: Aggregation queries for counts (followers/likes)
- [ ] F10.5: Vue Profile page with dynamic layouts
- [ ] F10.6: Toggle Like button component
- [ ] F10.7: Follow/Unfollow UI logic
- [ ] F10.8: Activity feed generator (FastAPI logic)
- [ ] F10.9: Skeleton loaders for feed loading
- [ ] F10.10: Playwright tests for social graph interactions
- [ ] F10.11: Sentry integration for Frontend/Backend

#### 📚 New Concepts Introduced

| Lesson | Topic                          | Frontend (Vue)         | Backend (FastAPI)             | Testing (Playwright/Python)                   | Deployment (Docker/CI-CD)         |
| :----: | :----------------------------- | :--------------------- | :---------------------------- | :-------------------------------------------- | :-------------------------------- |
|   65   | User Profile Management        | Profile edit form      | Profile update endpoint       | —                                             | —                                 |
|   66   | Avatar Uploads                 | Image cropping/preview | Profile image storage         | —                                             | —                                 |
|   67   | Favorites/Likes System         | Like toggle UI         | Many-to-many user↔item        | —                                             | —                                 |
|   68   | Following/Followers (Optional) | User relationships     | Self-referential many-to-many | —                                             | —                                 |
|   69   | Activity Feed                  | Activity feed display  | Aggregated queries            | —                                             | —                                 |
|   70   | Testing Social Features        | —                      | —                             | Testing M2M state and real-time count updates | —                                 |
|   71   | Error & Perf Monitoring        | Sentry Vue SDK         | Sentry FastAPI Middleware     | —                                             | Error Tracking, APM, Sentry setup |

#### ✅ Project Deliverable

- Edit user profile with avatar
- Like/favorite products or posts
- View list of favorites
- Basic activity feed
- Playwright tests for profile editing and social interactions

---

### 🔧 Project 11: Admin Dashboard

**What You'll Build:** An admin panel with role-based access control.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology            | Responsibility                              |
| :--------------- | :-------------------- | :------------------------------------------ |
| **Presentation** | Vue 3 + DaisyUI Stats | Analytics dashboards, management tables     |
| **RBAC Logic**   | FastAPI Dependencies  | Restricting endpoints by role (Admin/Staff) |
| **Analytics**    | SQL Aggregations      | COUNT, SUM, AVG across entities             |
| **Audit**        | Audit Trails          | Tracking who changed what content           |

**RBAC Pattern:**

```
┌─────────────────────────────────────────────────────────────┐
│                     ACCESS CONTROL FLOW                     │
│  User ──▶ Request /api/admin/* ──▶ FastAPI Dependency      │
│                                     (Requires Role: Admin)   │
│                                     │                        │
│  ┌──────────────────────────────────┤                        │
│  ▼ No                               ▼ Yes                    │
│  Return 403 Forbidden               Check Scope ──▶ Proceed  │
└─────────────────────────────────────────────────────────────┘
```

**Data Model (Extended User):**

| Field            | Type                       | Description           |
| :--------------- | :------------------------- | :-------------------- |
| `role`           | `Enum(User, Staff, Admin)` | User permission level |
| `last_login`     | `DateTime`                 | Tracking engagement   |
| `account_status` | `Enum(Active, Suspended)`  | Account moderation    |

**API Design:**

| Method | Endpoint                 | Description           | Role Required |
| :----- | :----------------------- | :-------------------- | :------------ |
| GET    | `/api/admin/stats`       | Site summary metrics  | Staff         |
| GET    | `/api/admin/users`       | All users (paginated) | Admin         |
| PATCH  | `/api/admin/users/{id}`  | Change roles/status   | Admin         |
| GET    | `/api/admin/audit-logs`  | System activity       | Admin         |
| DELETE | `/api/admin/content/{i}` | Moderate content      | Staff         |

#### 📋 Requirements

**Epic 1: RBAC Foundations**

| ID     | User Story                                   | Features        | Acceptance Criteria        |
| :----- | :------------------------------------------- | :-------------- | :------------------------- |
| US-1.1 | As a developer, I want roles defined         | Role Enum       | Roles: Admin, Staff, User  |
| US-1.2 | As an admin, I want to restrict access       | Role dependency | 403 for unauthorized users |
| US-1.3 | As a user, I want the UI to hide admin links | Conditional nav | Admin menu only for Admins |

**Epic 2: Analytics Dashboard**

| ID     | User Story                                    | Features           | Acceptance Criteria          |
| :----- | :-------------------------------------------- | :----------------- | :--------------------------- |
| US-2.1 | As a staff member, I want to see usage stats  | Stats endpoint     | Count of users, posts, sales |
| US-2.2 | As a staff member, I want to see daily trends | Aggregated queries | Data for simple line charts  |
| US-2.3 | As a staff member, I want to see active users | Active count       | Users logged in last 24h     |

**Epic 4: E2E Testing**

| ID     | User Story                                    | Features               | Acceptance Criteria                                |
| :----- | :-------------------------------------------- | :--------------------- | :------------------------------------------------- |
| US-4.1 | As a developer, I want to test admin security | Playwright test script | Verifies other roles cannot access admin dashboard |

**Epic 5: Audit Logging & Production Security**

| ID     | User Story                                               | Features                 | Acceptance Criteria                               |
| :----- | :------------------------------------------------------- | :----------------------- | :------------------------------------------------ |
| US-5.1 | As a developer, I want to track admin actions            | Audit Log Database       | All admin PATCH/DELETE actions are logged         |
| US-5.2 | As a developer, I want to restrict admin to specific IPs | IP Whitelisting (Theory) | Production Nginx restricts admin dashboard access |

**Feature Checklist:**

- [ ] F11.1: RBAC Dependency Factory in FastAPI
- [ ] F11.2: Stats service with optimized SQL aggregations
- [ ] F11.3: User management table with filters
- [ ] F11.4: Bulk action API (e.g., delete selected)
- [ ] F11.5: Vue Admin layout with Sidebar
- [ ] F11.6: Dashboard cards with DaisyUI `stats`
- [ ] F11.7: Simple charts using Chart.js or similar
- [ ] F11.8: Audit trail records on data changes
- [ ] F11.9: "Impersonate User" feature (Optional/Advanced)
- [ ] F11.10: Playwright tests for RBAC and Admin Security
- [ ] F11.11: Production audit logging middleware

#### 📚 New Concepts Introduced

| Lesson | Topic                     | Frontend (Vue)           | Backend (FastAPI)         | Testing (Playwright/Python)                    | Deployment (Docker/CI-CD)            |
| :----: | :------------------------ | :----------------------- | :------------------------ | :--------------------------------------------- | :----------------------------------- |
|   72   | Role-Based Access Control | Role-based UI visibility | Roles enum                | —                                              | —                                    |
|   73   | Admin Route Protection    | Admin-only routes        | Role dependencies         | —                                              | —                                    |
|   74   | Dashboard Analytics       | Stats cards              | Pagination & Aggregation  | —                                              | —                                    |
|   75   | User Management           | Admin user list          | User moderation endpoints | —                                              | —                                    |
|   76   | Content Moderation        | Moderation workflows     | Audit fields              | —                                              | —                                    |
|   77   | Testing RBAC & Security   | —                        | —                         | Testing unauthorized access and role-switching | —                                    |
|   78   | Audit Logging (Prod)      | —                        | Audit Trail Middleware    | —                                              | Production Security, IP Whitelisting |

#### ✅ Project Deliverable

- Admin and user roles
- Admin-only dashboard section
- View site statistics
- Manage users (view, edit roles, deactivate)
- Moderate content

---

### 📦 Project 12: File Manager & Downloads (Capstone)

**What You'll Build:** A file management system with folders, uploads, downloads, and sharing.

#### 📐 Architecture & Design

**System Architecture:**

| Layer            | Technology           | Responsibility                               |
| :--------------- | :------------------- | :------------------------------------------- |
| **Presentation** | Vue 3 + DnD API      | Folder tree, drag-and-drop, breadcrumbs      |
| **API**          | FastAPI              | Hierarchical navigation, stream downloads    |
| **Storage**      | Local Filesystem     | Nested directory matching DB structure       |
| **Security**     | Signed URLs / Tokens | Temporary access for file sharing            |
| **Database**     | SQLAlchemy           | Hierarchical model (Recursive relationships) |

**Hierarchical Logic:**

```
┌─────────────────────────────────────────────────────────────┐
│                    HIERARCHY MODEL (DB)                     │
│                                                             │
│  ┌────────────┐       ┌────────────┐       ┌────────────┐   │
│  │  Folder A  │◀─────▶│  Folder B  │◀─────▶│   File C   │   │
│  │ (Parent=N) │  p=A  │ (Parent=A) │  p=B  │ (Parent=B) │   │
│  └────────────┘       └────────────┘       └────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Secure Download Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                   SECURE DOWNLOAD FLOW                      │
│  User ──▶ Request Download ──▶ FastAPI ──▶ Verify JWT/Auth │
│                                             │               │
│  ┌──────────────────────────────────────────┘               │
│  ▼                                                          │
│  FastAPI ──▶ Open File Stream ──▶ StreamingResponse ──▶ Vue │
│              (Chunked transfer)                             │
└─────────────────────────────────────────────────────────────┘
```

**Data Model:**

| Field       | Type                 | Description                      |
| :---------- | :------------------- | :------------------------------- |
| `id`        | `UUID`               | Unique identifier (used in URLs) |
| `name`      | `String`             | Display name                     |
| `parent_id` | `UUID` (Nullable)    | Self-link for hierarchy          |
| `is_folder` | `Boolean`            | Type differentiator              |
| `file_size` | `Integer` (Nullable) | Size in bytes                    |

**API Design:**

| Method | Endpoint                | Request             | Description               |
| :----- | :---------------------- | :------------------ | :------------------------ |
| GET    | `/api/files`            | `?parent_id=uuid`   | List folder contents      |
| POST   | `/api/folders`          | `{name, parent_id}` | Create new directory      |
| POST   | `/api/files/upload`     | `multipart`         | Upload to specific folder |
| GET    | `/api/files/{id}/dl`    | —                   | Authenticated download    |
| POST   | `/api/files/{id}/share` | `{expires_days}`    | Generate share link       |

#### 📋 Requirements

**Epic 1: Hierarchical Storage**

| ID     | User Story                                 | Features         | Acceptance Criteria                 |
| :----- | :----------------------------------------- | :--------------- | :---------------------------------- |
| US-1.1 | As a user, I want to create folders        | Folder creation  | Folders appear in current directory |
| US-1.2 | As a user, I want to navigate into folders | Directory change | Content updates to sub-items        |
| US-1.3 | As a user, I want breadcrumb navigation    | Breadcrumbs UI   | Can click any parent to go back     |
| US-1.4 | As a user, I want to move files/folders    | Drag and Drop    | `parent_id` updates on drop         |

**Epic 2: Advanced File Operations**

| ID     | User Story                                | Features            | Acceptance Criteria                        |
| :----- | :---------------------------------------- | :------------------ | :----------------------------------------- |
| US-2.1 | As a user, I want to download large files | Streaming downloads | Downloads start instantly, no memory spike |
| US-2.2 | As a user, I want to upload batch files   | Queue management    | Multiple files upload concurrently         |
| US-2.3 | As a user, I want to see file previews    | Preview modal       | PDFs and Images viewable in-app            |

**Epic 4: E2E Testing (Capstone)**

| ID     | User Story                                                        | Features               | Acceptance Criteria                                                      |
| :----- | :---------------------------------------------------------------- | :--------------------- | :----------------------------------------------------------------------- |
| US-4.1 | As a developer, I want to test the full file management lifecycle | Playwright test script | Verifies create folder, upload file, move to folder, share, and download |

**Epic 5: Scalability & Zero-Downtime Deployment**

| ID     | User Story                                        | Features               | Acceptance Criteria                              |
| :----- | :------------------------------------------------ | :--------------------- | :----------------------------------------------- |
| US-5.1 | As a developer, I want to handle high traffic     | Load Balancing (Nginx) | Traffic is distributed across multiple instances |
| US-5.2 | As a developer, I want no downtime during updates | Blue-Green / Rolling   | App remains accessible during deployment         |

**Feature Checklist:**

- [ ] F12.1: Recursive SQLAlchemy model for folders/files
- [ ] F12.2: FastAPI `StreamingResponse` for secure downloads
- [ ] F12.3: Breadcrumb path generator (recursive CTE or logic)
- [ ] F12.4: UUID-based routing for security (no ID guessing)
- [ ] F12.5: Vue Tree view component for navigation
- [ ] F12.6: Drag-and-Drop integration (HTML5 or library)
- [ ] F12.7: Batch upload manager with progress per file
- [ ] F12.8: Temporary Signed URL generator for sharing
- [ ] F12.9: File type icons based on extension
- [ ] F12.10: Playwright tests for Capstone file management
- [ ] F12.11: Docker Compose with Horizontal Scaling

#### 📚 New Concepts Introduced

| Lesson | Topic                 | Frontend (Vue)   | Backend (FastAPI)   | Testing (Playwright/Python) | Deployment (Docker/CI-CD)                |
| :----: | :-------------------- | :--------------- | :------------------ | :-------------------------- | :--------------------------------------- |
|   79   | Folder Structure      | Tree navigation  | Hierarchical models | —                           | —                                        |
|   80   | Multi-File Uploads    | Queue management | Batch handling      | —                           | —                                        |
|   81   | File Downloads        | Download logic   | `StreamingResponse` | —                           | —                                        |
|   82   | Download with Auth    | Secure links     | Protected streaming | —                           | —                                        |
|   83   | Drag-and-Drop UI      | DnD API          | —                   | —                           | —                                        |
|   84   | Sharing & Permissions | Share links      | Tokenized access    | —                           | —                                        |
|   85   | Capstone E2E Testing  | —                | —                   | Testing complex hierarchies | —                                        |
|   86   | Zero-Downtime Deploys | —                | —                   | —                           | Load Balancing, Rolling Updates, Scaling |

#### ✅ Project Deliverable

- Create/navigate folder hierarchy
- Upload multiple files with progress
- Download files securely
- Generate shareable links
- Move files between folders
- Playwright tests for full file management lifecycle
- Horizontal scaling configuration with Nginx load balancer

---

## 📊 Curriculum Summary

| Metric                            |       Count        |
| :-------------------------------- | :----------------: |
| **Total Projects**                |       **12**       |
| **Total Lessons**                 |       **86**       |
| **Micro-Lessons** (×2 per lesson) |      **172**       |
| **Estimated Duration**            | **~100-110 hours** |

**Skill Progression:**

| Phase                    | Projects                       | Focus Areas                            |
| :----------------------- | :----------------------------- | :------------------------------------- |
| **Foundation** (1-2)     | Hello Full-Stack, Task Manager | Setup, API calls, CRUD basics          |
| **Persistence** (3-4)    | Notes App, Categorized Notes   | Database, relationships, filtering     |
| **Authentication** (5-6) | Auth System, Blog              | JWT, protected routes, ownership       |
| **File Handling** (7-8)  | Image Gallery, E-Commerce      | Uploads, static files, complex schemas |
| **Advanced** (9-12)      | Cart, Profiles, Admin, Files   | State management, RBAC, full features  |

---

## 3. MICRO-LESSON TEMPLATE: PART 1 — THEORY & CONCEPTS

_(Est. Time: 30-35 minutes)_

---

### Lesson [X.1]: [TOPIC] — Theory & Concepts

---

#### 1. Lesson Metadata

| Field               | Value                                 |
| :------------------ | :------------------------------------ |
| **Lesson Number**   | X.1 of 68 (Part 1 of 2)               |
| **Project**         | Project Y: [Project Name]             |
| **Topic**           | [English] / [Russian]                 |
| **Prerequisites**   | Lessons X, Y (or "None" for Lesson 1) |
| **You Should Know** | [Key JS/Python concepts assumed]      |
| **Unlocks**         | Lesson X.2, then Lessons Y, Z         |
| **Duration**        | 30-35 minutes                         |

**Learning Objectives — Part 1 (Theory):**

1. **Remember:** [Define/list key concepts]
2. **Understand:** [Explain how the feature works conceptually]

#### 2. Prerequisite Refresher

_Briefly explain 1-2 specific JavaScript or Python concepts essential for this lesson._

**JavaScript Example:**

- **Concept Name:** Brief explanation.
  ```javascript
  // Short, clear code example
  const example = async () => { ... };
  ```

**Python Example:**

- **Concept Name:** Brief explanation.
  ```python
  # Short, clear code example
  async def example(): ...
  ```

#### 3. Real-World Scenario & Context

- **Scenario (1 paragraph):** Describe where this pattern is used in real applications.
- **Full-Stack Context (1-2 sentences):** How do frontend and backend work together here?

#### 4. Core Concepts Explained (Deep Dive)

**Frontend Perspective (Vue):**

- What is Vue doing? How does it handle this pattern?
- _Visual Instruction:_ Include a **Markdown Table** for the flow.

**Backend Perspective (FastAPI):**

- What is FastAPI doing? How does it process the request?
- _Visual Instruction:_ Include a **Markdown Table** for the flow.

**Integration Point:**

- How do they connect? What data format is exchanged?
- _Visual Instruction:_ Request/Response table showing the handshake.

**Mental Model:** Provide a clear analogy or metaphor.

**When to Use / When NOT to Use:**

| Scenario        | Recommendation          |
| :-------------- | :---------------------- |
| [Use case]      | ✅ Use this pattern     |
| [Anti-use case] | ❌ Consider alternative |

#### 5. New Terminology

_A list defining 3-4 key terms introduced in this lesson._

| Term       | Definition |
| :--------- | :--------- |
| **Term 1** | Definition |
| **Term 2** | Definition |
| **Term 3** | Definition |

#### 6. Algorithmic Thinking (Planning the Solution)

_Explain the thinking process for implementing this feature:_

| Step | Frontend (Vue)          | Backend (FastAPI)        |
| :--- | :---------------------- | :----------------------- |
| 1    | What state do we need?  | What endpoint structure? |
| 2    | What actions/handlers?  | What validation/logic?   |
| 3    | What template elements? | What response format?    |

#### 7. Initial Pattern Introduction

_Show the basic syntax pattern for both frontend and backend._

**Frontend (Vue):**

| What You Want | Code Pattern |
| :------------ | :----------- |

```vue
<script setup lang="ts">
// Example code with comments
</script>

<template>
  <!-- Template usage -->
</template>
```

**Backend (FastAPI):**

| What You Want | Code Pattern |
| :------------ | :----------- |

```python
# Example code with comments explaining the pattern
from fastapi import FastAPI
```

#### 8. Comprehension Check

_3 quick questions to verify understanding before moving to practice._

1. [Conceptual question about Vue]
2. [Conceptual question about FastAPI]
3. [Integration question about how they connect]

---

## 4. MICRO-LESSON TEMPLATE: PART 2 — PRACTICE & APPLICATION

_(Est. Time: 30-35 minutes)_

---

### Lesson [X.2]: [TOPIC] — Practice & Application

---

#### 1. Lesson Metadata

| Field              | Value                     |
| :----------------- | :------------------------ |
| **Lesson Number**  | X.2 of 68 (Part 2 of 2)   |
| **Project**        | Project Y: [Project Name] |
| **Topic**          | [English] / [Russian]     |
| **Continues From** | Lesson X.1                |
| **Duration**       | 30-35 minutes             |

**Learning Objectives — Part 2 (Practice):**

1. **Apply:** [Create/build the feature using the pattern]
2. **Analyze:** [Compare/contrast with alternatives or debug issues]

#### 2. Quick Recap

_2-3 bullet points summarizing key concepts from X.1._

- Key concept 1
- Key concept 2
- How frontend connects to backend

#### 3. The Variable Frame & Complexity Scale

**Basic Form (Minimal):**

_Frontend:_

```vue
<script setup lang="ts">
// Simplest usage
</script>
```

_Backend:_

```python
# Simplest usage
```

**With Options (Standard):**

_Frontend:_

```vue
<script setup lang="ts">
// Adding configurations
</script>
```

_Backend:_

```python
# Adding optional parameters
```

**Advanced Form (Production-Ready):**

_Frontend:_

```vue
<script setup lang="ts">
// Complex usage with error handling, loading states
</script>
```

_Backend:_

```python
# Complex usage with validation, transactions
```

#### 4. Micro-Implementation Drills

_4 progressive coding drills (2 frontend, 2 backend):_

**Drill 1 (Frontend - Basic):** [Simplest Vue usage]

```vue
<!-- Starter code with TODO -->
```

**Drill 2 (Backend - Basic):** [Simplest FastAPI usage]

```python
# Starter code with TODO
```

**Drill 3 (Integration):** [Connect frontend to backend]

```vue
<!-- Vue code with TODO for API call -->
```

```python
# FastAPI endpoint for the above
```

**Drill 4 (Edge Case):** [Handle a tricky scenario]

```vue
<!-- Handle errors, loading, edge cases -->
```

#### 5. Common Pitfalls & Anti-Patterns

_3 common mistakes for this topic._

| Layer           | ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
| :-------------- | :---------------- | :------------------ | :------------- |
| **Vue**         | [Mistake]         | [Fix]               | [Consequence]  |
| **FastAPI**     | [Mistake]         | [Fix]               | [Consequence]  |
| **Integration** | [Mistake]         | [Fix]               | [Consequence]  |

#### 6. Mini-Project Implementation

_Complete, runnable code implementing part of the current project._

**File Structure:**

```
project/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   └── [Component].vue
│   │   ├── composables/
│   │   │   └── use[Feature].ts
│   │   ├── stores/
│   │   │   └── [store].ts
│   │   ├── App.vue
│   │   └── main.ts
│   └── package.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── models.py
│   │   ├── schemas.py
│   │   └── routers/
│   │       └── [resource].py
│   └── requirements.txt
└── docker-compose.yml (optional)
```

**Backend - main.py:**

```python
# Complete implementation with detailed comments
```

**Frontend - Component.vue:**

```vue
<script setup lang="ts">
// Complete implementation with detailed comments
</script>

<template>
  <!-- Template with DaisyUI/Tailwind styling -->
</template>
```

**How to Test:**

_Backend:_

```bash
# Start the server
uvicorn app.main:app --reload

# Test with curl
curl -X GET http://localhost:8000/api/endpoint
```

_Frontend:_

```bash
# Start development server
npm run dev

# Open http://localhost:5173
```

#### 7. Implementation Exercises (Progressive)

**Level 1 (Recall):** Fill-in-the-blank

_Frontend:_

```vue
<!-- Code with blanks: _____ -->
```

_Backend:_

```python
# Code with blanks: _____
```

**Level 2 (Apply):** 2 independent coding tasks

- Exercise 2.1: [Frontend task]
- Exercise 2.2: [Backend task]

**Level 3 (Debug):** Find and fix the bugs

_Frontend bug:_

```vue
<!-- Broken code -->
```

_Backend bug:_

```python
# Broken code
```

**Self-Check:** Reference implementations with explanatory comments.

---

## 5. PROJECT CHECKPOINT TEMPLATE

_After completing all lessons for a project, synthesize with a checkpoint._

### Project Checkpoint: [PROJECT NAME]

- **Project Number:** [X of 12]
- **Lessons Covered:** [X through Y]
- **What You Built:** 1-2 sentence description.
- **Patterns Mastered:** List 6-10 patterns from this project.

**Project File Structure:**

```
[Complete structure for the project]
```

**API Specification:**

| Method | Endpoint        | Description     | Auth Required |
| :----- | :-------------- | :-------------- | :-----------: |
| GET    | `/api/resource` | List resources  |      No       |
| POST   | `/api/resource` | Create resource |      Yes      |
| ...    | ...             | ...             |      ...      |

**Frontend Routes:**

| Path         | Component | Description    | Protected |
| :----------- | :-------- | :------------- | :-------: |
| `/`          | HomePage  | Landing page   |    No     |
| `/dashboard` | Dashboard | User dashboard |    Yes    |
| ...          | ...       | ...            |    ...    |

**Self-Assessment Checklist:**

- [ ] All API endpoints return correct status codes
- [ ] Pydantic validation works properly
- [ ] Vue components use `<script setup>` with TypeScript
- [ ] Error and loading states are handled
- [ ] Protected routes work correctly
- [ ] Code follows the patterns from the lessons

**Stretch Goals:**

- Optional advanced feature 1
- Optional advanced feature 2

---

## 6. TESTING CONVENTIONS

_Apply these conventions in all code examples._

### 6.1. Backend Testing (pytest + httpx)

```python
# tests/test_[module].py
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app

@pytest.fixture
async def client():
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as ac:
        yield ac

@pytest.mark.asyncio
async def test_endpoint(client: AsyncClient):
    response = await client.get("/api/endpoint")
    assert response.status_code == 200
```

### 6.2. Frontend Testing (Vitest + Vue Test Utils)

```typescript
// src/components/__tests__/ComponentName.spec.ts
import { describe, it, expect, vi } from "vitest";
import { mount } from "@vue/test-utils";
import ComponentName from "../ComponentName.vue";

describe("ComponentName", () => {
  it("renders properly", () => {
    const wrapper = mount(ComponentName, {
      props: {
        /* props */
      },
    });
    expect(wrapper.text()).toContain("Expected text");
  });

  it("calls API on mount", async () => {
    const fetchSpy = vi.spyOn(global, "fetch");
    mount(ComponentName);
    expect(fetchSpy).toHaveBeenCalled();
  });
});
```

### 6.3. E2E Testing (Playwright + Python)

```python
# tests/e2e/test_app_flow.py
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="session")
def base_url():
    # Use environment variable or default to local dev server
    return "http://localhost:5173"

def test_user_can_navigate_dashboard(page: Page, base_url: str):
    page.goto(base_url)

    # Assert branding is visible
    expect(page.locator("nav")).to_contain_text("FullStack App")

    # Navigate to login
    page.click("text=Login")
    expect(page).to_have_url(f"{base_url}/login")

    # Fill form
    page.fill('input[type="email"]', "test@example.com")
    page.fill('input[type="password"]', "securepassword")
    page.click('button:has-text("Sign In")')

    # Assert successful login
    expect(page).to_have_url(f"{base_url}/dashboard")
    expect(page.locator(".alert")).to_contain_text("Welcome back")
```

### 6.4. Running Tests

**Backend (Unit/Integration):**

```bash
pytest -v                        # Run all tests
pytest -v -k "test_name"         # Run specific test
pytest --cov=app --cov-report=html  # With coverage
```

**Frontend (Component):**

```bash
npm run test              # Run all unit/component tests
npm run test -- --watch   # Watch mode
```

**E2E (Playwright):**

```bash
# Set up browser binaries (first time only)
playwright install

# Run E2E tests
pytest tests/e2e -v

# Run with visible browser
pytest tests/e2e -v --headed

# Specify base URL
BASE_URL=http://localhost:5173 pytest tests/e2e -v
```

---

## 7. DEVELOPMENT ENVIRONMENT SETUP

_Include this setup guide at the start of the curriculum._

### 7.1. Backend Setup

```bash
# Create project directory
mkdir fullstack-project && cd fullstack-project
mkdir backend && cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install fastapi uvicorn[standard] pydantic[email] sqlalchemy[asyncio] asyncpg python-jose[cryptography] passlib[bcrypt] python-multipart python-dotenv pytest-playwright

# Create requirements.txt
pip freeze > requirements.txt
```

### 7.2. Frontend Setup

```bash
# From project root
cd ..
npm create vite@latest frontend -- --template vue-ts
cd frontend

# Install dependencies
npm install
npm install axios pinia vue-router
npm install -D tailwindcss postcss autoprefixer daisyui @types/node

# Initialize Tailwind
npx tailwindcss init -p
```

### 7.3. Docker Compose (Optional)

```yaml
# docker-compose.yml
version: "3.8"
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/app
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    depends_on:
      - backend

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=app
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```
