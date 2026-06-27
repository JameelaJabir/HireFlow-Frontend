# HireFlow — Frontend

> Vue 3 SPA for the HireFlow job recruitment platform | SE4010 Cloud Computing Assignment | SLIIT 2026

A single-page application serving two user roles — **Employer** and **Candidate** — each with a dedicated dashboard, connected to five backend microservices via REST. Deployed on Google Cloud Run using Docker + Nginx.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Cloud Deployment](#cloud-deployment)
- [CI/CD Pipeline](#cicd-pipeline)
- [DevSecOps — SonarCloud](#devsecops--sonarcloud)
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

## Architecture

The frontend communicates directly with each backend microservice via dedicated Axios clients. There is no API gateway — each service is called by its own environment variable URL.

```
HireFlow Frontend (Vue 3 SPA)
│
├── api.ts          ──►  Auth Service        :8081
├── jobApi.ts       ──►  Job Listing Service :3002
├── cvApi.ts        ──►  CV Service          :8085
├── interviewApi.ts ──►  Interview Service   :3090
├── pipelineApi.ts  ──►  Interview Service   :3090  (pipelines)
└── companyApi.ts   ──►  Profile Service     :8080
```

JWT tokens issued by the Auth Service are attached to every outbound request via an Axios request interceptor.

---

## Cloud Deployment

The frontend is containerized and deployed to **Google Cloud Run** as a static site served by Nginx.

- **Container**: Node 22 (build stage) + Nginx Alpine (serve stage)
- **Port**: 8080 (Nginx listens on 8080; Cloud Run forwards traffic here)
- **Container Registry**: GCP Artifact Registry
- **SPA Routing**: Nginx `try_files` falls back to `index.html` for all routes (Vue Router history mode)
- **Caching**: Static assets cached 1 year with `immutable`; `index.html` is never cached

---

## CI/CD Pipeline

### Deploy Workflow (`.github/workflows/deploy.yml`)

Triggers on every push to `main`:

```
Push to main
    │
    ├── Checkout code
    ├── Authenticate to GCP (Service Account key)
    ├── Configure Docker → Artifact Registry
    ├── Build Docker image (tagged :sha + :latest)
    ├── Push image to Artifact Registry
    └── Deploy to Cloud Run (--allow-unauthenticated --port=8080)
```

### SonarCloud Workflow (`.github/workflows/sonar.yml`)

Triggers on push to `main`/`develop` and on pull requests:

```
Push / PR
    │
    ├── Checkout (full history)
    ├── Set up Node.js 22
    ├── npm ci
    ├── npm run test:cov  (Vitest with v8 coverage)
    └── SonarCloud scan (SonarSource/sonarqube-scan-action)
```

### Required GitHub Secrets

| Secret | Description |
|---|---|
| `GCP_PROJECT_ID` | GCP project ID |
| `GCP_SA_KEY` | GCP Service Account JSON key |
| `GCP_REGION` | Cloud Run region (e.g. `asia-east1`) |
| `GCP_ARTIFACT_REPO` | Artifact Registry repo name (e.g. `hireflow`) |
| `SONAR_TOKEN` | SonarCloud token |

---

## DevSecOps — SonarCloud

SonarCloud is integrated via GitHub Actions. On every push and pull request:

1. Vitest runs with v8 coverage
2. Results are submitted to SonarCloud for static analysis
3. Reports: bugs, vulnerabilities, code smells, coverage %, security hotspots

Configuration is in `sonar-project.properties` at the repo root.

---

## Prerequisites

- Node.js 18+
- npm 9+
- All backend microservices running (see [Backend README](../HireFlow-Backend/README.md))

---

## Getting Started

```bash
# Install dependencies
npm install

# Start the development server (http://localhost:5173)
npm run dev
```

### Available Scripts

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

Create a `.env` file in the project root. Reference `.env.example` for the template.

```env
VITE_AUTH_API_URL=http://localhost:8081
VITE_JOB_API_URL=http://localhost:3002
VITE_INTERVIEW_API_URL=http://localhost:3090
VITE_COMPANY_API_URL=http://localhost:8080/api
VITE_CV_API_URL=http://localhost:8085
```

All variables are prefixed with `VITE_` and accessed via `import.meta.env.VITE_*`.

For production, create `.env.production` and replace localhost URLs with your deployed Cloud Run service URLs.

---

## Project Structure

```
HireFlow-Frontend/
├── .github/
│   └── workflows/
│       ├── deploy.yml           # GCP Cloud Run deployment
│       └── sonar.yml            # SonarCloud SAST
├── public/                      # Static assets served as-is
├── src/
│   ├── main.ts                  # App entry — mounts Vue, Pinia, Router
│   ├── App.vue                  # Root component — <RouterView> + <Toaster>
│   ├── constants.ts             # Shared enums (UserRole, HiringStage, etc.)
│   │
│   ├── router/
│   │   └── index.ts             # Route definitions and navigation guards
│   │
│   ├── stores/                  # Pinia stores (one per domain)
│   │   ├── auth.ts              # Session, tokens, user identity
│   │   ├── jobs.ts              # Job listings (employer & public)
│   │   ├── interview.ts         # Interview scheduling
│   │   ├── pipeline.ts          # Hiring pipeline progression
│   │   ├── company.ts           # Company profiles & analytics
│   │   └── cv.ts                # Candidate CV & applications
│   │
│   ├── services/                # Axios API clients (one per microservice)
│   │   ├── api.ts               # Auth service client + shared JWT interceptor
│   │   ├── jobApi.ts            # Job Listing Service client
│   │   ├── interviewApi.ts      # Interview Service client
│   │   ├── pipelineApi.ts       # Hiring Pipeline endpoints
│   │   ├── companyApi.ts        # Profile Service client
│   │   └── cvApi.ts             # CV Service client
│   │
│   ├── views/                   # Page-level components (matched to routes)
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
│   │   ├── ui/                  # Headless design-system components (Reka UI)
│   │   ├── layout/
│   │   │   ├── AppShell.vue     # Authenticated layout (sidebar + header)
│   │   │   └── AppSidebar.vue   # Role-aware sidebar navigation
│   │   ├── jobs/
│   │   │   └── JobForm.vue      # Create/edit job form
│   │   ├── interview/
│   │   │   └── InterviewTable.vue
│   │   ├── pipeline/
│   │   │   ├── HiringStageProgress.vue
│   │   │   └── PipelineRow.vue
│   │   ├── icons/               # Custom SVG icon components
│   │   └── ModeToggle.vue       # Dark/light theme toggle
│   │
│   ├── types/                   # TypeScript interfaces
│   │   ├── interview.ts
│   │   └── cv.ts
│   │
│   ├── lib/
│   │   └── utils.ts             # cn() — Tailwind class merging utility
│   │
│   └── assets/
│       ├── main.css             # Design tokens, dark mode, typography
│       └── base.css             # CSS reset and base styles
│
├── .env                         # Development environment variables
├── .env.example                 # Reference template
├── .env.production              # Production environment variables
├── sonar-project.properties     # SonarCloud configuration
├── vite.config.ts               # Vite configuration
├── vitest.config.ts             # Vitest test configuration
├── tsconfig.json                # TypeScript configuration
├── nginx.conf                   # Nginx SPA config for production container
├── Dockerfile                   # Multi-stage build (Node → Nginx)
├── package.json
└── index.html                   # HTML entry point
```

---

## Routes & Pages

Route guards are applied globally via `router/index.ts`.

| Guard | Behaviour |
|---|---|
| `meta.requiresAuth` | Redirects unauthenticated users to `/login` |
| `meta.guest` | Redirects authenticated users to their role dashboard |
| `meta.role` | Redirects users to their own dashboard if role doesn't match |

### Guest Routes

| Path | Component | Description |
|---|---|---|
| `/login` | `LoginView` | Email + password login |
| `/register` | `RegisterView` | Account creation with role selection |

### Employer Routes — requires `role: EMPLOYER`

| Path | Component | Description |
|---|---|---|
| `/employer/dashboard` | `EmployerDashboardView` | Stats overview and recent activity |
| `/employer/jobs` | `EmployerJobsView` | Create, edit, and close job postings |
| `/employer/jobs/:jobId/applications` | `EmployerJobApplicationsView` | Review applications for a job |
| `/employer/pipelines` | `PipelinesView` | All active hiring pipelines |
| `/employer/pipelines/:id` | `PipelineDetailView` | Manage a pipeline, advance stages |
| `/employer/interviews` | `InterviewsView` | Schedule and manage interviews |
| `/employer/company` | `CompanyProfileView` | Company info, logo upload, analytics |

### Candidate Routes — requires `role: CANDIDATE`

| Path | Component | Description |
|---|---|---|
| `/candidate/dashboard` | `CandidateDashboardView` | Application summary and upcoming interviews |
| `/candidate/jobs` | `CandidateJobBoardView` | Browse and apply to open jobs |
| `/candidate/interviews` | `CandidateInterviewsView` | View and respond to interview invitations |
| `/candidate/pipeline` | `CandidatePipelineView` | Track application progress |
| `/candidate/cv-profile` | `CandidateCvProfileView` | Manage CV: skills, experience, resume upload |
| `/candidate/companies` | `ExploreCompaniesView` | Browse and follow companies |
| `/candidate/companies/:id` | `CompanyDetailsView` | Company detail with open positions |

---

## State Management

Pinia stores use the Composition API `defineStore()` pattern. All stores are in `src/stores/`. Auth state is persisted to `localStorage`.

### `useAuthStore`

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

| Action | Description |
|---|---|
| `fetchOpenJobs()` | Load all open jobs for candidate job board |
| `searchJobs(query)` | Filter jobs by title, location, skills |
| `fetchJobById(id)` | Load a single job's details |
| `fetchMyJobs()` | Load employer's own job postings |
| `createJob(dto)` | Post a new job |
| `updateJob(id, dto)` | Edit an existing job |
| `closeJob(id)` | Close a job posting |

### `useInterviewStore` — `stores/interview.ts`

| Action | Description |
|---|---|
| `fetchForEmployer()` | Load all interviews scheduled by the employer |
| `fetchForCandidate()` | Load all interviews for the candidate |
| `schedule(dto)` | Schedule a new interview |
| `update(id, dto)` | Update interview details |
| `cancel(id)` | Cancel an interview |
| `accept(id)` | Accept an interview invitation |
| `decline(id)` | Decline an interview invitation |

### `usePipelineStore` — `stores/pipeline.ts`

| Action | Description |
|---|---|
| `fetchForEmployer()` | Load all hiring pipelines for the employer |
| `fetchForCandidate()` | Load own pipeline status across all applications |
| `advanceStage(id, stage)` | Move a candidate to the next hiring stage |

### `useCompanyStore` — `stores/company.ts`

| Action | Description |
|---|---|
| `fetchAllCompanies()` | Load all companies for candidate explore view |
| `fetchCompanyById(id)` | Load a single company's profile |
| `search(query)` | Search companies by name, industry, or location |
| `fetchByEmployee(employeeId)` | Load company by the employer's user ID |
| `fetchAnalytics(id)` | Load company analytics (views, followers, applications) |
| `createCompany(formData)` | Create a new company profile with logo |
| `updateCompany(id, formData)` | Update company profile |
| `deleteCompany(id)` | Delete a company profile |
| `follow(id)` | Follow a company |
| `unfollow(id)` | Unfollow a company |

### `useCvStore` — `stores/cv.ts`

| Action | Description |
|---|---|
| `fetchMyProfile()` | Load own CV profile (bio, skills, experience, education) |
| `upsertProfile(dto)` | Create or update CV profile |
| `uploadResume(file)` | Upload a resume PDF to Azure Blob Storage |

Profile data is normalised before being stored to ensure a consistent shape regardless of which fields the server returns.

---

## API Integration

Each microservice has a dedicated Axios client in `src/services/`. A shared request interceptor attaches the JWT from `localStorage` to every request.

| File | Env Var | Target Service |
|---|---|---|
| `api.ts` | `VITE_AUTH_API_URL` | Auth Service `:8081` |
| `jobApi.ts` | `VITE_JOB_API_URL` | Job Listing Service `:3002` |
| `interviewApi.ts` | `VITE_INTERVIEW_API_URL` | Interview Service `:3090` |
| `pipelineApi.ts` | `VITE_INTERVIEW_API_URL` | Interview Service `:3090` (pipelines) |
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
Stored in localStorage + useAuthStore populated
    │
    ▼
Router redirects to role dashboard
(/employer/dashboard  or  /candidate/dashboard)
    │
    ▼
All API requests auto-attach Bearer token via Axios interceptor
    │
    ▼
Logout → POST /api/auth/logout → clear localStorage → redirect /login
```

---

## Component Library

Reusable UI components live in `src/components/ui/` — built on [Reka UI](https://reka-ui.com/) (headless, accessible primitives) styled with Tailwind CSS.

| Component | Description |
|---|---|
| `Button` | Variants: default, outline, ghost, destructive; sizes: sm, md, lg |
| `Input` | Styled text input |
| `Badge` | Status pills (colors map to job/application/pipeline status) |
| `Card` | Container with Header, Title, Description, Content, Footer slots |
| `Dialog` | Modal dialog with overlay |
| `AlertDialog` | Confirmation dialog for destructive actions |
| `DropdownMenu` | Contextual action menus |
| `Select` | Accessible select input |
| `Form` | Vee-Validate integrated form wrapper |
| `Progress` | Progress bar (used in hiring stage visualization) |
| `Avatar` | User/company avatar with fallback initials |
| `Sheet` | Side drawer panel |
| `Skeleton` | Loading placeholder |
| `Sonner` | Toast notifications — `toast.success()` / `toast.error()` |
| `Sidebar` | Role-aware app navigation |
| `Table` | Data table primitives |

**Class merging utility:**

```ts
import { cn } from '@/lib/utils'
cn('px-4 py-2', isActive && 'bg-blue-500', className)
```

---

## Design System

Defined via CSS custom properties in `src/assets/main.css`.

### Color Tokens

| Token | Usage |
|---|---|
| `--color-primary` | Deep Professional Blue — primary actions, CTAs |
| `--color-primary-container` | Light primary background for cards |
| `--color-tertiary` | Emerald — success states, positive indicators |
| `--color-surface` | Page background |
| `--color-surface-container` | Card/panel background |
| `--color-on-surface` | Primary text |
| `--color-on-surface-variant` | Secondary/muted text |
| `--color-destructive` | Error and destructive action red |

Dark mode via `.dark` class on `<html>`, toggled by `ModeToggle.vue`.

### Typography

| Font | Role |
|---|---|
| Manrope | Headings, display text (`font-headline`) |
| Inter | Body text, UI labels (`font-body`) |

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
`vitest.config.ts` extends `vite.config.ts` so all path aliases apply in tests.

---

## Docker

Multi-stage `Dockerfile` — Node 22 builds the app, Nginx Alpine serves it.

```bash
# Build the image
docker build -t hireflow-frontend .

# Run locally
docker run -p 8080:8080 hireflow-frontend
```

Nginx config (`nginx.conf`) handles:
- SPA routing — all unmatched paths fall back to `index.html`
- Static asset caching — 1 year, immutable
- `index.html` — never cached (`no-cache, no-store`)
