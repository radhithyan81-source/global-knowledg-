# Personal AI Bot Agent — Phase 1 (Frontend & Firebase Hybrid Workspace)

This revised plan implements **Phase 1: Frontend Only** for the Personal AI Bot Agent using React, React Router (`BrowserRouter`), Tailwind CSS, `localStorage`, and Firebase Authentication & Firestore. It strictly excludes backend AI, voice synthesis, memory engines, and third-party messaging apps for Phase 1 while delivering a complete, resilient, offline-capable frontend with reusable UI components, global loading spinners, and `try-catch` error recovery.

### User Review & Critical Decisions

> [!IMPORTANT]
> The plan has been updated to match your exact Phase 1 specifications, routing structure, validation messages, and confirmation workflows. Click **Proceed** to build and run the application.

- **Phase 1 Scope (Frontend Only)**: No backend LLM calls, voice synthesis engines, or external messaging apps yet. Chat uses an instant frontend responder with typing indicator and clears from screen upon logout.
- **Exact Routing & Module Architecture**: Uses `react-router-dom` (`BrowserRouter`, `Routes`, `Route`) mapping `/` (`Login`), `/dashboard` (`Dashboard`), `/chat` (`ChatPage` + `Chat`), `/tasks` (`Tasks` + `TaskList`), `/calendar` (`CalendarPage` + `CalendarView`), and `/settings` (`Settings`), supported by the Firebase service module and safe error handler utility (`showSafeError`).
- **Hybrid Local + Cloud Persistence**: Tasks (`localStorage` key `"tasks_list"` + Firestore) and Calendar events (`localStorage` + Firestore) work both online and offline. When offline, a persistent `"You are offline"` banner appears while all features continue working locally.
- **Mandatory Validation & Confirmation Gates**:
  - **Login**: Google Sign-In + Email/Password via Firebase Auth. Displays `"Wrong email or password. Please try again."` on failure. Never stores passwords in code.
  - **Chat**: Blocks empty submissions and displays `"Please type a message"`. Clears chat state on logout.
  - **Dashboard**: Displays Today's tasks, Today's events, and Daily briefing; displays `"No tasks for today"` when empty.
  - **Tasks**: Blocks duplicate task titles with `"Task already exists"` and prompts `"Are you sure? Yes / No"` before deleting any task.
  - **Calendar**: Monthly calendar grid that displays date and time and asks `"Is this correct? Yes / No"` before saving any event.

---

### 1. Overview & Core Concept

- **What It Does**: Provides a fast, resilient personal workspace where users authenticate via Google or Email/Password, view their daily briefing on the Dashboard, interact with the Chat interface, manage tasks with duplicate prevention and `localStorage` (`"tasks_list"`) + Firebase sync, schedule events on a monthly Calendar with pre-save verification, and configure Profile, Language, Voice ON/OFF, Memory ON/OFF, Backup, and Delete Data in Settings.
- **Target Audience / Persona**: Users who need a dependable, mobile-responsive personal assistant interface that never crashes to a white screen and continues functioning even when internet connectivity drops.
- **Key Value**: Combines strict defensive error handling (`try-catch` across all operations + `showSafeError()`), explicit user confirmations before destructive or scheduling actions, and seamless Light/Dark mode switching.

---

### 2. User Experience & Visual Design

- **Key User Flows**:
  1. **Login Page (`/`)**: Supports both Email/Password sign-in/sign-up and Google Sign-In via Firebase Auth. Any authentication error is caught in a `try-catch` block and renders `"Wrong email or password. Please try again."` Logging out clears all active chat messages from the screen.
  2. **Dashboard Page (`/dashboard`)**: Presents three structured sections: **Today's Tasks** (filtered from `"tasks_list"` / Firestore, showing `"No tasks for today"` when empty), **Today's Events**, and a **Daily Briefing** summary card with quick navigation links.
  3. **Chat Page (`/chat`)**: Renders the `Chat` component featuring user and bot message bubbles, an animated typing indicator, text input box, and send button. Submitting an empty string is blocked and shows `"Please type a message"`.
  4. **Tasks Page (`/tasks`)**: Renders the `TaskList` component to add, edit, complete, and delete tasks. Checks case-insensitively whether a task title already exists; if so, displays `"Task already exists"` and prevents creation. Clicking Delete opens a confirmation modal asking `"Are you sure? Yes / No"`. Synchronizes automatically with `localStorage` (`"tasks_list"`) and Firebase.
  5. **Calendar Page (`/calendar`)**: Renders the `CalendarView` monthly grid showing scheduled events for each day. Creating a new event opens a verification prompt displaying the chosen date and time and asking `"Is this correct? Yes / No"` before committing to storage.
  6. **Settings Page (`/settings`)**: Displays controls for Profile details, Language selection, Voice ON/OFF toggle, Memory ON/OFF toggle, Backup (one-click data export/sync), and Delete Data (with confirmation), auto-saving preferences immediately.

- **Visual Identity & Theme**:
  - *Aesthetic Direction*: Clean, modern SaaS workspace with responsive navigation, Light/Dark mode toggle button in the top bar, and zero visual clutter.
  - *Color Palette & Mood*:
    - **Light Mode**: Crisp slate background (`#F8FAFC`), white cards (`#FFFFFF`), subtle borders (`#E2E8F0`), slate text (`#0F172A`), and indigo primary actions (`#4F46E5`).
    - **Dark Mode**: Deep slate canvas (`#090D16`), elevated surface (`#111827`), muted borders (`#1E293B`), crisp light text (`#F8FAFC`), and vibrant indigo accents (`#6366F1`).
    - **Status Colors**: Amber warning banner for `"You are offline"`, Crimson alerts for validation/error messages (`"Something went wrong. Please try again."`), and Emerald indicators for completed tasks and saved states.
  - *Typography & Hierarchy*: `Plus Jakarta Sans` for headings and navigation, `DM Sans` for body copy and chat bubbles, and `JetBrains Mono` (`tabular-nums`) for calendar dates and task counts.
  - *Reusable UI Component System*: Consistent, responsive implementations of Buttons, Inputs, Cards, Modals (used for the `"Are you sure? Yes / No"` and `"Is this correct? Yes / No"` confirmations), and Navigation Bars.

- **Interactive Feedback & Motion**:
  - **Global Loading Spinner**: Shown during route transitions, authentication checks, and async operations so the user never sees a blank screen.
  - **Offline Banner**: Listens to browser `online` and `offline` events; immediately displays `"You are offline"` at the top of the viewport when disconnected while keeping all CRUD operations working via `localStorage`.
  - **Safe Error Recovery**: Every handler is wrapped in `try-catch`, surfacing `"Something went wrong. Please try again."` (or `showSafeError()`) without interrupting other components.

---

### 3. Key Product Decisions & Trade-Offs

- **Decision 1: Dual `localStorage` (`"tasks_list"`) + Firebase Persistence**
  - *Chosen Approach*: Read and write immediately to `localStorage` (using exact key `"tasks_list"` for tasks, plus local keys for events and settings) while asynchronously syncing with Firestore when authenticated and online.
  - *Why*: Satisfies the offline-first requirement (`"You are offline"` banner with full app functionality) and ensures instant UI responsiveness even if network or Firebase operations fail.

- **Decision 2: Defensive `try-catch` & Root Error Boundary**
  - *Chosen Approach*: Wrap every event handler, storage helper, and Firebase call in `try-catch` blocks that route errors through `showSafeError()`, paired with a React Error Boundary.
  - *Why*: Guarantees the app never renders a blank white screen under any failure condition.

---

### 4. Technical Architecture & Data Strategy *(Technical Reference)*

- **Architecture & Component Diagram**:

```
┌────────────────────────────────────────────────────────────────────────────┐
│            ErrorBoundary + Theme + Global Loading & Offline State          │
│     • Offline Banner ("You are offline")                                   │
│     • Safe Error Banner ("Something went wrong. Please try again.")        │
│     • Loading Spinner Overlay                                              │
├────────────────────────────────────────────────────────────────────────────┤
│                     React Router (BrowserRouter / Routes)                  │
└────┬─────────────┬──────────────┬──────────────┬───────────────┬───────────┘
     │             │              │              │               │
     ▼             ▼              ▼              ▼               ▼
┌─────────┐  ┌───────────┐  ┌──────────┐  ┌────────────┐  ┌──────────────┐
│  Login  │  │ Dashboard │  │ ChatPage │  │   Tasks    │  │ CalendarPage │
│   (/)   │  │(/dashboard│  │ (/chat)  │  │  (/tasks)  │  │ (/calendar)  │
└────┬────┘  └─────┬─────┘  └────┬─────┘  └─────┬──────┘  └──────┬───────┘
     │             │             │              │                │
     │             │             ▼              ▼                ▼
     │             │        ┌──────────┐  ┌────────────┐  ┌──────────────┐
     │             │        │   Chat   │  │  TaskList  │  │ CalendarView │
     │             │        └────┬─────┘  └─────┬──────┘  └──────┬───────┘
     │             │             │              │                │
     └─────────────┴─────────────┴──────────────┼────────────────┘
                                                │
                                                ▼
                                     ┌────────────────────┐
                                     │   Settings Page    │
                                     │    (/settings)     │
                                     └──────────┬─────────┘
                                                │
                  ┌─────────────────────────────┴───────────────────────────┐
                  ▼                                                         ▼
   ┌─────────────────────────────────┐                       ┌─────────────────────────────┐
   │  localStorage (Offline-First)   │                       │   Firebase Auth & Firestore │
   │  • "tasks_list"                 │                       │   • Google + Email/Password │
   │  • "calendar_events"            │                       │   • Cloud Sync & Backup     │
   │  • "app_settings" / "chat_msgs" │                       │   • errorHandler.js         │
   └─────────────────────────────────┘                       └─────────────────────────────┘
```

- **Interactive Component & State Mapping**:
  - **Authentication (`Login`)**: Calls Firebase `signInWithPopup` (Google) and `signInWithEmailAndPassword` / `createUserWithEmailAndPassword` inside `try-catch`. Catches failures to display `"Wrong email or password. Please try again."` Sign-out clears chat history from state/storage.
  - **Task Management (`Tasks` & `TaskList`)**: Validates trimmed task title against existing tasks; if found, shows `"Task already exists"`. Otherwise saves to `localStorage.setItem("tasks_list", ...)` and Firestore. Delete button triggers a `"Are you sure? Yes / No"` modal before removal.
  - **Calendar Scheduling (`CalendarPage` & `CalendarView`)**: Renders monthly days grid. Submitting the event form opens a confirmation modal displaying the formatted date and time with `"Is this correct? Yes / No"` buttons before saving.
  - **Settings Management (`Settings`)**: Auto-saves Profile, Language, Voice ON/OFF, and Memory ON/OFF toggles, provides JSON Backup download, and confirms before running Delete Data.
