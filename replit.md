# Deal Stocks — Project Overview

## Architecture

**Monorepo** managed by pnpm workspaces. Frontend-only (no backend required in development).

### Artifacts
| Name | Kind | Dir | Purpose |
|------|------|-----|---------|
| Deal Stocks | web | `artifacts/deal-stocks` | Main frontend (React + Vite + Tailwind) |

---

## Frontend — `artifacts/deal-stocks`

### Stack
- React 18 + Vite + TypeScript
- Tailwind CSS v4 (dark mode: class strategy)
- Wouter (routing)
- Lucide React (icons)
- Radix UI, React Hook Form, TanStack Query, Recharts

### Auth
- **User auth**: localStorage key `ds_auth_user`. Any email/password creates a session.
- **Admin auth**: localStorage key `ds_admin_session`. Local fallback credentials:
  - `adtsofttech` / `Ranat.888` (superadmin)
  - `admin` / `admin123` (admin)
  - Also tries `/api/admin/login` first if a backend is running.

### Key Contexts
| Context | File | Purpose |
|---------|------|---------|
| `AuthContext` | `src/contexts/AuthContext.tsx` | localStorage-based auth with `isAuthenticated` flag |
| `AdminContext` | `src/contexts/AdminContext.tsx` | Admin auth with API + local fallback |
| `LangContext` | `src/contexts/LangContext.tsx` | 5-language i18n (en, ar, fr, fr-ca, fa) with RTL |
| `useTheme` | `src/components/ThemeToggle.tsx` | dark/light hook + toggle button component |

### Key Data Files
| File | Purpose |
|------|---------|
| `src/data/packages.json` | Package definitions with tasks (20 packages, 5 tiers) |
| `src/data/packagesData.ts` | Typed access to packages.json, helper functions |
| `src/data/platformsData.ts` | Amazon/Alibaba/AliExpress platform config |
| `src/data/adminData.ts` | Seed data + ExtraTask/OrderRecord types with CRUD helpers; localStorage-backed |
| `src/data/teamData.ts` | Team/referral seed data |
| `src/data/products.json` | Product listings for task UI |
| `src/data/marketplaces.json` | Marketplace metadata |

### Routes

#### Public
| Path | Component |
|------|-----------|
| `/` | `Home.tsx` — Landing page with header, hero, products |
| `/login` | `auth/Login.tsx` |
| `/register` | `auth/Register.tsx` |

#### Dashboard (Protected — requires user login)
| Path | Component |
|------|-----------|
| `/dashboard` | `DashboardHome.tsx` — brand strip + product rows |
| `/dashboard/wallet` | `Wallet.tsx` |
| `/dashboard/deposit` | `Deposit.tsx` — with request status history |
| `/dashboard/withdraw` | `Withdraw.tsx` — with request status history |
| `/dashboard/records` | `Records.tsx` |
| `/dashboard/notifications` | `Notifications.tsx` |
| `/dashboard/team` | `Team.tsx` — My Team (no referral language) |
| `/dashboard/mine` | `Mine.tsx` — user hub (Profile/Settings/Team shortcuts + Extra Task card) |
| `/dashboard/analytics` | `Analytics.tsx` — 5 stat cards |
| `/dashboard/packages/:code` | `PackageTask.tsx` (admin-assigned only, no list page) |
| `/dashboard/menu` | `Menu.tsx` — platform selector (loads from platformsData.ts) |
| `/dashboard/menu/:platformId` | `PlatformTask.tsx` — GRAB modal task flow |
| `/dashboard/support` | `Support.tsx` |
| `/dashboard/settings` | `Settings.tsx` |
| `/dashboard/profile` | `Profile.tsx` |
| `/dashboard/extra-task` | `ExtraTask.tsx` — admin-gated bonus task (approval via Support) |

#### Bottom Navigation (all dashboard pages)
Fixed bottom bar with 5 items: **Home** | **Menu** | **Wallet** | **Records** | **Mine**
- Provided by `MobileBottomNav.tsx` (standalone pages include it directly; DashboardLayout injects it automatically)
- Support removed from bottom nav (accessible via Mine → Settings or dedicated link)

#### Admin Panel (Protected — requires admin login)
Sidebar is a **flat nav** (7 items, no dropdowns). Each page may have in-page top tabs.

| Path | Component | In-page tabs |
|------|-----------|--------------|
| `/admin/login` | `AdminLogin.tsx` | — |
| `/admin` | `AdminDashboard.tsx` | Dashboard · Analytics |
| `/admin/users` | `AdminUsers.tsx` | Users & Team · **Audit Log** (AuditLogPanel) |
| `/admin/ads` | `AdminAds.tsx` | **Menu** (library/assign/bulk-assign/create) · **Extra Tasks** (ExtraTasksPanel) |
| `/admin/records` | `AdminRecords.tsx` | Transaction History · **Approval Transactions** (TransactionsApprovalPanel) · **Orders** (OrdersPanel) · Task History |
| `/admin/support` | `AdminSupport.tsx` | Support · **Notifications** (NotificationsPanel) |
| `/admin/landing-page` | `AdminLandingPage.tsx` | — |
| `/admin/settings` | `AdminSettings.tsx` | — |
| `/admin/audit-log` | `AdminAuditLog.tsx` | (also accessible as Users & Team → Audit Log tab) |
| `/admin/extra-tasks` | `AdminExtraTasks.tsx` | (also accessible as Menu → Extra Tasks tab) |
| `/admin/orders` | `AdminOrders.tsx` | (also accessible as Records → Orders tab) |
| `/admin/transactions` | `AdminTransactions.tsx` | (also accessible as Records → Approval Transactions tab) |
| `/admin/notifications` | `AdminNotifications.tsx` | (also accessible as Support → Notifications tab) |

Panel extraction pattern: child pages export both `export function FooPanel()` (content only) and `export default function AdminFoo()` (wraps panel in AdminLayout). Parent pages import the panel component for use as an in-page tab.

Sidebar badges: `/admin/records` shows `pendingOrders + pendingTransactions`; `/admin/support` shows `openTickets + pendingScheduled`.

### Package System
- **5 tiers**: Starter (300s), Silver (400s), Gold (500s), Diamond (600s), Elite (700s)
- **Kill mode**: Packages ending in `4` (304, 404, 504, 604, 704) loop indefinitely
- **Lock points**: Random task numbers trigger payment requirement interruptions
- Each package has 25 tasks with product images, commission amounts, etc.

### Platform System
- 3 platforms: Amazon (VIP1, $20-499 balance), Alibaba (VIP2, $500-899), AliExpress (VIP3, $900+)
- Users unlock platforms based on wallet balance
- Each platform has separate task flows

### Theme
- Dark/light mode via `useTheme()` hook from `ThemeToggle.tsx`
- Persisted to `localStorage` key `ds-theme`
- Default: follows system preference

### Language System
- 5 languages: English, Arabic (RTL), French, French-CA, Persian (RTL)
- Stored in `localStorage` key `ds_lang`
- Auto-detected from `navigator.language` on first visit
- RTL applied via `<div dir="rtl">` wrapper in `LangProvider`

### Assets
- Logo: `attached_assets/Deal_Stocks_Icon_1775288841076.png` (imported via `@assets/`)
- Logo alias configured in `vite.config.ts`: `@assets` → `attached_assets/`

### Maintenance Mode System
Maintenance mode is a global platform toggle stored in `ds_admin_settings` (localStorage key).

**Layers of enforcement (defense in depth):**
1. **Route gate** — `ProtectedRoute` in `App.tsx` reads `loadAdminSettings().maintenanceMode`; any `/dashboard/*` route shows `MaintenanceScreen` when ON. Admin routes (`/admin/*`) are never checked.
2. **In-page overlay** — `MaintenanceOverlay` component (`src/components/MaintenanceOverlay.tsx`) renders as a full-screen `fixed z-[70]` modal on every action page: Deposit, Withdraw, PlatformTask, PackageTask, AdCodeTask, ExtraTask. This is the defence-in-depth layer.
3. **Login page banner** — `Login.tsx` shows an amber notice when maintenance is ON (login is still permitted; users are informed).

**Hook:** `useMaintenanceMode()` (`src/hooks/useMaintenanceMode.ts`) — polls localStorage every 1.5s and subscribes to `ds:data-changed` + `storage` events. Updates reactively without page reload.

**Admin toggle:** Admin Settings → Platform Controls → Maintenance Mode switch. The toggle calls `patchAdminSettings({ maintenanceMode })` immediately (no Save button needed), dispatches `ds:data-changed`, and shows a "Saved" confirmation. Admin sidebar shows a pulsing amber "Maintenance ON" banner when active.

**Persistence:** Stored in `ds_admin_settings` via `saveAdminSettings()`. Survives page refresh, app restart, and code redeployment. Load logic uses `{ ...DEFAULT_SETTINGS, ...stored }` merge so new fields always default safely without overwriting existing values.

---

## Performance & Stability Improvements (Applied)

### Code Splitting (Lazy Loading)
All 40+ page-level route components in `App.tsx` are imported via `React.lazy()` + `Suspense`. Named exports use `.then(m => ({ default: m.Name }))`. Initial JS bundle only contains the shell, contexts, and shared utilities; each page chunk loads on first visit. Fallback: `PageFallback` (spinner on `bg-gray-950`).

### ProtectedRoute — No Tick Polling
`ProtectedRoute` previously ran `setInterval(bump, 1500)` which caused the **entire dashboard tree** to re-render every 1.5 s. This is removed. Suspension and maintenance are now monitored by `useSuspensionGuard()` and `useMaintenanceMode()` directly inside `ProtectedRoute`. Re-renders only happen when those states actually change.

### Error Boundary
`src/components/ErrorBoundary.tsx` — class-based React error boundary wrapping all routes in `AppRoutes`. Shows a user-friendly "Something went wrong" card with Retry + Refresh buttons. Logs stack traces to console.

### Body Scroll Lock on Modals
`src/hooks/useBodyScrollLock.ts` — sets `document.body.style.overflow = 'hidden'` while active (restores on cleanup). Applied to:
- `PlatformTask.tsx` — GrabModal (`showModal`)
- `PackageTask.tsx` — both modals (`showModal || showBalanceModal`)
- `AdCodeTask.tsx` — ad review modal (`showModal`)

### Null Safety
- `DashboardHome` BrandStrip: removed `!` non-null assertion on logo images → conditional render `{b.logo && <img ...>}`

---

## SEO & Hostinger Deployment Readiness (Applied)

### index.html — Full SEO Head
- Added `<meta name="description">`, `<meta name="theme-color" content="#8A4FFF">`
- Added Open Graph tags: `og:title`, `og:description`, `og:image` (→ `/opengraph.jpg`), `og:url`, `og:type`, `og:site_name`
- Added Twitter/X card tags: `summary_large_image`
- Added `<link rel="canonical">` and `<link rel="apple-touch-icon">`
- Removed `maximum-scale=1` from viewport (was blocking pinch-to-zoom on mobile)
- Added placeholder comment for Google Search Console verification meta tag
- All `YOUR_DOMAIN` placeholders in `index.html`, `robots.txt`, and `sitemap.xml` must be replaced before going live

### Public Hosting Files (all new in `public/`)
- `robots.txt` — allows all crawlers; disallows `/admin/` and `/dashboard/`; references sitemap
- `sitemap.xml` — includes `/`, `/login`, `/register` as public routes
- `.htaccess` — Apache SPA routing (rewrites non-file paths to `index.html`); browser caching headers; gzip compression; security headers (X-Content-Type-Options, X-Frame-Options, Referrer-Policy); HTTPS redirect (commented, enable after SSL is active on Hostinger)

### vite.config.ts — Build Optimisation
- `PORT` and `BASE_PATH` env vars are now **optional** — they default to `3000` and `/` respectively, so `vite build` works without any env vars set (required for Hostinger local/CI builds)
- `runtimeErrorOverlay()` plugin moved to dev-only (was included in every production build)
- `build.sourcemap: false` — removes source maps from production output
- `build.rollupOptions.output.manualChunks` — vendor chunk splitting: `vendor-react`, `vendor-router`, `vendor-icons`, `vendor-charts`

### Auth UX Improvements
- `Register.tsx` — real-time inline password feedback: colour-coded border + checkmark/X icon showing length validity and match status as the user types; `autocomplete="new-password"` on both password fields
- `Login.tsx` — when error contains "no account / not found", shows a direct inline "Create a free account →" link inside the error banner; added `autocomplete` attributes on both fields
