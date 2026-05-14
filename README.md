# HireFlow — Frontend

Vue 3 single-page application for the HireFlow recruitment platform. Supports two user roles — **Employer** and **Candidate** — each with a dedicated dashboard and feature set.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Project Structure](#project-structure)
- [Routes & Pages](#routes--pages)
- [State Management](#state-management)
- [API Integration](#api-integration)
- [Authentication Flow](#authentication-flow)
- [Component Library](#component-library)
- [Design System](#design-system)
- [Testing](#testing)
- [Docker](#docker)

---

## Tech Stack

| Category | Library / Tool | Version |
|---|---|---|
| Framework | Vue 3 (Composition API) | 3.x |
| Build Tool | Vite | 7.3.1 |
| Language | TypeScript | ~5.9.3 |
| Routing | Vue Router | 5.0.3 |
| State Management | Pinia | 3.0.4 |
| Styling | Tailwind CSS | 4.2.1 |
| UI Components | Reka UI (headless) | 2.9.2 |
| Icons | Lucide Vue Next | 0.577.0 |
| HTTP Client | Axios | 1.13.6 |
| Form Validation | Vee-Validate + Zod | 4.15.1 / 3.25.76 |
| Tables | TanStack Vue Table | 8.21.3 |
| Toast Notifications | Vue Sonner | 2.0.9 |
| Date Picker | Vue DatePicker | 12.1.0 |
| Composition Utilities | VueUse | 14.2.1 |
| Testing | Vitest + Vue Test Utils | 4.1.0 / 2.4.6 |
| Formatting | Prettier | 3.8.1 |

---

## Prerequisites

- Node.js 18+
- npm 9+
- All backend microservices running (see [Backend README](../Backend/README.md))

---

## Getting Started

```bash
# Navigate to the frontend directory
cd frotend

# Install dependencies
npm install

# Start the development server (http://localhost:5173)
npm run dev
```

### All Available Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start development server with hot reload |
| `npm run build` | Type-check then bundle for production (`dist/`) |
| `npm run preview` | Serve the production build locally |
| `npm run type-check` | Run `vue-tsc` type validation |
| `npm run test:unit` | Run unit tests with Vitest |
| `npm run test:cov` | Run tests with v8 coverage report |
| `npm run format` | Format `src/` with Prettier |

---

## Environment Variables

Create a `.env` file in the `frotend/` directory. An `.env.example` file is provided as a reference.

```env
VITE_AUTH_API_URL=http://localhost:8081
VITE_JOB_API_URL=http://localhost:3090
VITE_INTERVIEW_API_URL=http://localhost:3000
VITE_COMPANY_API_URL=http://localhost:8080/api
VITE_CV_API_URL=http://localhost:8085
```

All variables are prefixed with `VITE_` and accessed in code via `import.meta.env.VITE_*`.

For production, duplicate to `.env.production` and replace localhost URLs with your deployed service URLs.

---

## Project Structure

```
frotend/
├── public/                         # Static assets served as-is
├── src/
│   ├── main.ts                     # App entry point — mounts Vue, registers Pinia & Router
│   ├── App.vue                     # Root component — hosts <RouterView> and <Toaster>
│   ├── constants.ts                # Shared enums (UserRole, HiringStage, etc.) and route names
│   │
│   ├── router/
│   │   └── index.ts                # Route definitions and navigation guards
│   │
│   ├── stores/                     # Pinia stores (one per domain)
│   │   ├── auth.ts                 # Session, tokens, user identity
│   │   ├── jobs.ts                 # Job listings (employer & public)
│   │   ├── interview.ts            # Interview scheduling
│   │   ├── pipeline.ts             # Hiring pipeline progression
│   │   ├── company.ts              # Company profiles & analytics
│   │   └── cv.ts                   # Candidate CV & applications
│   │
│   ├── services/                   # Axios API clients (one per microservice)
│   │   ├── api.ts                  # Auth service client + shared request interceptor
│   │   ├── jobApi.ts               # Job Listing Service client
│   │   ├── interviewApi.ts         # Interview Service client
│   │   ├── pipelineApi.ts          # Hiring Pipeline endpoints (Interview Service)
│   │   ├── companyApi.ts           # Profile Service client
│   │   └── cvApi.ts                # CV Service client
│   │
│   ├── views/                      # Page-level components (matched to routes)
│   │   ├── HomeView.vue
│   │   ├── auth/
│   │   │   ├── LoginView.vue
│   │   │   └── RegisterView.vue
│   │   ├── employer/
│   │   │   ├── EmployerDashboardView.vue
│   │   │   ├── EmployerJobsView.vue
│   │   │   ├── EmployerJobApplicationsView.vue
│   │   │   ├── PipelinesView.vue
│   │   │   ├── PipelineDetailView.vue
│   │   │   ├── InterviewsView.vue
│   │   │   └── CompanyProfileView.vue
│   │   └── candidate/
│   │       ├── CandidateDashboardView.vue
│   │       ├── CandidateJobBoardView.vue
│   │       ├── CandidateInterviewsView.vue
│   │       ├── CandidatePipelineView.vue
│   │       ├── CandidateCvProfileView.vue
│   │       ├── ExploreCompaniesView.vue
│   │       └── CompanyDetailsView.vue
│   │
│   ├── components/
│   │   ├── ui/                     # Headless, reusable design-system components
│   │   ├── layout/
│   │   │   ├── AppShell.vue        # Authenticated layout (sidebar + header + slot)
│   │   │   └── AppSidebar.vue      # Role-aware sidebar navigation
│   │   ├── jobs/
│   │   │   └── JobForm.vue         # Create / edit job form
│   │   ├── interview/
│   │   │   └── InterviewTable.vue  # Interview list table
│   │   ├── pipeline/
│   │   │   ├── HiringStageProgress.vue  # Visual stage progress bar
│   │   │   └── PipelineRow.vue          # Single pipeline row
│   │   ├── icons/                  # Custom SVG icon components
│   │   └── ModeToggle.vue          # Dark / light theme toggle
│   │
│   ├── types/                      # TypeScript interfaces
│   │   ├── interview.ts            # Interview, HiringPipeline, HiringStage types
│   │   └── cv.ts                   # CandidateProfile, WorkExperience, Education types
│   │
│   ├── lib/
│   │   └── utils.ts                # cn() — Tailwind class merging (clsx + tailwind-merge)
│   │
│   └── assets/
│       ├── main.css                # Global CSS — design tokens, dark mode, typography
│       └── base.css                # CSS reset and base styles
│
├── .env                            # Development environment variables
├── .env.example                    # Reference template
├── .env.production                 # Production environment variables
├── vite.config.ts                  # Vite configuration
├── vitest.config.ts                # Vitest test configuration
├── tsconfig.json                   # TypeScript configuration
├── package.json
├── Dockerfile                      # Container build
├── nginx.conf                      # Nginx config for serving SPA in production
└── index.html                      # HTML entry point
```

---

## Routes & Pages

Route guards are applied globally. Routes with `meta.requiresAuth` redirect unauthenticated users to `/login`. Routes with `meta.guest` redirect authenticated users to their role dashboard. Role-specific routes check `meta.role` and redirect on mismatch.

### Guest Routes

| Path | Component | Description |
|---|---|---|
| `/login` | `LoginView` | Email + password login |
| `/register` | `RegisterView` | Account creation with role selection (Employer / Candidate) |

### Employer Routes — requires `role: EMPLOYER`

| Path | Component | Description |
|---|---|---|
| `/employer/dashboard` | `EmployerDashboardView` | Stats overview, quick actions, recent activity |
| `/employer/jobs` | `EmployerJobsView` | Create, edit, and close job postings |
| `/employer/jobs/:jobId/applications` | `EmployerJobApplicationsView` | Review applications for a specific job |
| `/employer/pipelines` | `PipelinesView` | All active hiring pipelines |
| `/employer/pipelines/:id` | `PipelineDetailView` | Manage a single pipeline, advance candidates through stages |
| `/employer/interviews` | `InterviewsView` | Schedule and manage interviews |
| `/employer/company` | `CompanyProfileView` | Company info, logo upload, analytics |

### Candidate Routes — requires `role: CANDIDATE`

| Path | Component | Description |
|---|---|---|
| `/candidate/dashboard` | `CandidateDashboardView` | Application status summary, upcoming interviews |
| `/candidate/jobs` | `CandidateJobBoardView` | Browse and search open jobs, submit applications |
| `/candidate/interviews` | `CandidateInterviewsView` | View scheduled interviews, accept or decline |
| `/candidate/pipeline` | `CandidatePipelineView` | Track application progress across all applied jobs |
| `/candidate/cv-profile` | `CandidateCvProfileView` | Manage CV: bio, skills, work history, education, resume upload |
| `/candidate/companies` | `ExploreCompaniesView` | Browse and follow companies |
| `/candidate/companies/:id` | `CompanyDetailsView` | Company detail page with open positions |

---

## State Management

Pinia stores follow the Composition API `defineStore()` pattern. All stores are in `src/stores/`.

### `useAuthStore` — `stores/auth.ts`

Manages the active session. State is persisted to `localStorage`.

| State | Type | Description |
|---|---|---|
| `token` | `string \| null` | JWT access token |
| `refreshToken` | `string \| null` | JWT refresh token |
| `userId` | `string \| null` | Authenticated user ID |
| `name` | `string \| null` | User's display name |
| `role` | `UserRole \| null` | `EMPLOYER` or `CANDIDATE` |

| Getter | Description |
|---|---|
| `isAuthenticated` | `true` if token is present |
| `isEmployer` | `true` if role is `EMPLOYER` |
| `isCandidate` | `true` if role is `CANDIDATE` |

| Action | Description |
|---|---|
| `login(email, password)` | Authenticates and populates store |
| `register(payload)` | Registers and auto-logs-in |
| `logout()` | Calls logout API, clears all state |

### `useJobStore` — `stores/jobs.ts`

| State | Description |
|---|---|
| `jobs` | All open jobs (public browse) |
| `myJobs` | Jobs posted by the current employer |
| `currentJob` | Single job detail |
| `loading` / `error` | Request status |

Actions: `fetchMyJobs`, `createJob`, `updateJob`, `closeJob`, `fetchOpenJobs`, `searchJobs`, `fetchJobById`

### `useInterviewStore` — `stores/interview.ts`

Actions: `fetchForEmployer`, `fetchForCandidate`, `schedule`, `update`, `cancel`, `accept`, `decline`

### `usePipelineStore` — `stores/pipeline.ts`

Actions: `fetchForEmployer`, `fetchForCandidate`, `advanceStage`

### `useCompanyStore` — `stores/company.ts`

Actions: `fetchAllCompanies`, `fetchCompanyById`, `search`, `fetchAnalytics`, `follow`, `unfollow`, `fetchByEmployee`, `createCompany`, `updateCompany`, `deleteCompany`

### `useCvStore` — `stores/cv.ts`

Actions: `fetchMyProfile`, `upsertProfile`, `uploadResume`

Includes profile normalization to ensure consistent data shape before rendering.

---

## API Integration

Each backend microservice has a dedicated Axios client in `src/services/`. Every instance attaches the JWT token automatically via a request interceptor.

```ts
// Applied to all Axios instances
config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`
```

| File | Env Var | Target Service |
|---|---|---|
| `api.ts` | `VITE_AUTH_API_URL` | Auth Service `:8081` |
| `jobApi.ts` | `VITE_JOB_API_URL` | Job Listing Service `:3090` |
| `interviewApi.ts` | `VITE_INTERVIEW_API_URL` | Interview Service `:3000` |
| `pipelineApi.ts` | `VITE_INTERVIEW_API_URL` | Interview Service `:3000` (pipelines) |
| `companyApi.ts` | `VITE_COMPANY_API_URL` | Profile Service `:8080/api` |
| `cvApi.ts` | `VITE_CV_API_URL` | CV Service `:8085` |

File uploads (resume PDFs, company logos) use `FormData` with `Content-Type: multipart/form-data`.

---

## Authentication Flow

```
Register / Login
    │
    ▼
POST /api/auth/register  or  /api/auth/login
    │
    ▼
Response: { accessToken, refreshToken, userId, name, role }
    │
    ▼
Store in localStorage + populate useAuthStore
    │
    ▼
Router redirects to role-specific dashboard
(/employer/dashboard  or  /candidate/dashboard)
    │
    ▼
All subsequent API requests auto-attach Bearer token (interceptor)
    │
    ▼
Logout → POST /api/auth/logout → clear localStorage → redirect to /login
```

**Route guard logic (`router/index.ts`):**

1. `requiresAuth` — redirect unauthenticated users to `/login`
2. `guest` — redirect authenticated users away from login/register to their dashboard
3. `meta.role` — redirect users who don't match the required role to their own dashboard

---

## Component Library

Reusable UI components live in `src/components/ui/` and are built on [Reka UI](https://reka-ui.com/) (headless, accessible primitives) styled with Tailwind CSS.

| Component | Description |
|---|---|
| `Button` | Variants: default, outline, ghost, destructive; sizes: sm, md, lg |
| `Input` | Styled text input |
| `Label` | Form field label |
| `Badge` | Status pills (colors map to job / application / pipeline status) |
| `Card` | Container with `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`, `CardFooter` |
| `Dialog` | Modal dialog with overlay |
| `AlertDialog` | Confirmation dialog for destructive actions |
| `DropdownMenu` | Contextual action menus |
| `Select` | Accessible select input |
| `Form` | Vee-Validate integrated form wrapper |
| `Progress` | Progress bar (used in hiring stage visualization) |
| `Avatar` | User/company avatar with fallback initials |
| `Separator` | Horizontal/vertical divider |
| `Sheet` | Side drawer panel |
| `Skeleton` | Loading placeholder |
| `Sonner` (Toaster) | Toast notifications — `toast.success()` / `toast.error()` |
| `Sidebar` family | `Sidebar`, `SidebarContent`, `SidebarMenu`, `SidebarMenuItem`, etc. |
| `Table` family | Data table primitives |

**Class merging utility:**

```ts
import { cn } from '@/lib/utils'

// Merges Tailwind classes and resolves conflicts
cn('px-4 py-2', isActive && 'bg-blue-500', className)
```

---

## Design System

Defined via CSS custom properties in `src/assets/main.css`.

### Color Tokens

| Token | Usage |
|---|---|
| `--color-primary` | Deep Professional Blue — primary actions, CTAs |
| `--color-primary-container` | Light primary background for cards/containers |
| `--color-tertiary` | Emerald — success states, positive indicators |
| `--color-surface` | Page background |
| `--color-surface-container` | Card/panel background |
| `--color-on-surface` | Primary text |
| `--color-on-surface-variant` | Secondary/muted text |
| `--color-destructive` | Error and destructive action red |

Dark mode is applied via the `.dark` class on `<html>`, toggled by `ModeToggle.vue`.

### Typography

| Font | Role | Tailwind Class |
|---|---|---|
| Manrope | Headings, display text | `font-headline` |
| Inter | Body text, UI labels | `font-body` |

Both fonts are loaded from Google Fonts in `index.html`.

### Utility Classes

| Class | Effect |
|---|---|
| `.editorial-shadow` | Subtle blue-tinted box shadow |
| `.glass-surface` | Glassmorphism background blur |
| `.gradient-cta` | Primary → primary-container gradient |

---

## Testing

**Framework:** Vitest 4.1.0 with JSDOM environment
**Component testing:** Vue Test Utils 2.4.6
**Coverage:** v8

```bash
# Run all unit tests
npm run test:unit

# Generate coverage report
npm run test:cov
```

Test files follow the pattern `src/**/__tests__/**/*.spec.ts`.

Configuration is in `vitest.config.ts`, which extends `vite.config.ts` so all path aliases and plugins apply in tests as well.

---

## Docker

A `Dockerfile` is included for containerized deployment. The production build is served by Nginx using `nginx.conf`, which handles SPA routing (all unmatched paths fall back to `index.html`).

```bash
# Build the image
docker build -t hireflow-frontend .

# Run the container
docker run -p 80:80 hireflow-frontend
```

The frontend container is included in the root `docker-compose.yml` alongside the backend services.
