This document establishes a comprehensive long-term development context for the **No Fixed Address (NFA)** project. It is based on a deep analysis of the provided Repomix-packed codebase.

---

# 1. PROJECT KNOWLEDGE BASE

## 1. PROJECT IDENTITY
*   **App Name:** No Fixed Address (NFA).
*   **Nature:** A boutique travel agency platform combining a high-aesthetic public storefront with a robust administrative back-office.
*   **Purpose:** To curate and sell unconventional travel packages. It moves away from "corporate" travel vibes toward a "Nomadic Manifesto" or "Cyber-Brutalist" experience.
*   **Target Users:**
    *   **Travelers:** Seekers of unique, curated experiences who appreciate modern/edgy design.
    *   **Admins:** Agency operators managing inventory, tracking revenue, and processing bookings.
*   **Main Business Workflow:**
    1.  Admin creates a "Package" (trip) with itinerary, images, and payment rules.
    2.  User explores trips (optionally via the AI "Oracle").
    3.  User selects dates and travelers, then pays (Full or Advance) via Razorpay.
    4.  Admin monitors the booking and updates status (Pending -> Paid).

## 2. STACK & INFRASTRUCTURE
*   **Frontend:** React 19 (latest stable).
*   **Language:** TypeScript (Strict typing for Firestore entities).
*   **Build Tool:** Vite 6.
*   **Routing:** React Router DOM v7 (standard and protected routes).
*   **Styling:** **Tailwind CSS 4.0**. (Note: Uses the new CSS-first configuration in `src/index.css` rather than a legacy `tailwind.config.js`).
*   **Firebase Integration:**
    *   **Auth:** Email/Password for Admin access.
    *   **Firestore:** Real-time NoSQL database for packages, bookings, and settings.
    *   **Storage:** Multi-image hosting for travel destination photos.
    *   **Hosting:** Configured for Firebase Hosting (via `firebase.json`).
*   **AI Integration:** Google Gemini API (`@google/genai`) used for the "Travel Oracle" matching engine.
*   **Payments:** Razorpay (Client-side integration via CDN script in `index.html`).
*   **Important Dependencies:** `react-hook-form` (forms), `lucide-react` (icons), `motion/react` (animations), `react-toastify` (notifications).

## 3. FOLDER & MODULE MAP
*   **`src/admin`**: The "Command Center." Contains the layout and pages for managing the business (Dashboard, Package CRUD, Booking logs, Settings).
*   **`src/components`**:
    *   `SharedBrutal.tsx`: Design tokens, Logos, and the "Brutal" Navbar/Footer.
    *   `PackageCard.tsx` / `SkeletonCard.tsx`: Reusable UI for the travel grid.
    *   `ProtectedRoute.tsx`: Route guard for the admin area.
*   **`src/contexts`**: `AuthContext.tsx` manages the Firebase `onAuthStateChanged` listener.
*   **`src/firebase`**: Low-level service layer. Separates Firebase config from business logic (`firestoreService.ts`, `storageService.ts`).
*   **`src/pages`**: Public-facing views (Home, Listing, Details, Booking process).
*   **`src/services`**: `geminiService.ts` handles the AI recommendation logic.
*   **Root Files**:
    *   `firestore.rules` & `storage.rules`: Security definitions.
    *   `package.json`: Dependency manifest (Includes some unused items like `better-sqlite3`—likely for local testing or future expansion).

## 4. ARCHITECTURE OVERVIEW
*   **Organization:** The app is a Single Page Application (SPA) divided into two distinct logical zones: **Public** (lightweight, marketing-heavy) and **Admin** (data-dense, CRUD-heavy).
*   **Service-UI Connection:** Components do not call Firebase directly. They use abstractions in `src/firebase/firestoreService.ts`. This allows for easier testing and potential migration.
*   **Auth Flow:**
    *   State is initialized in `AuthContext.tsx`.
    *   Admin logins at `/admin/login` using `authService.ts`.
    *   Protected routes redirect to login if no session is found.
*   **Reusable Patterns:**
    *   **Brutalism:** Consistent use of `brutal-border` (2px solid) and `brutal-shadow` (colored offsets) defined in `index.css`.
    *   **Data Labels:** `DataLabel` component used for monospaced, uppercase "System" metadata.

## 5. ROUTE MAP
| Path | Component | Purpose |
| :--- | :--- | :--- |
| `/` | `HomePage` | Hero, AI Oracle, Featured Packages. |
| `/packages` | `PackagesPage` | Full searchable inventory. |
| `/packages/:id` | `PackageDetailPage` | Full itinerary, pricing, and gallery. |
| `/book/:id` | `BookingPage` | Payment collection via Razorpay. |
| `/booking-success`| `BookingSuccessPage`| Confirmation after payment/request. |
| `/admin/login` | `AdminLogin` | Entry point for system operators. |
| `/admin/dashboard`| `Dashboard` | Stats overview (Revenue, Bookings). |
| `/admin/packages` | `PackagesManager` | List/Delete existing trips. |
| `/admin/packages/add`| `PackageForm` | Authoring tool for new trips. |
| `/admin/bookings` | `BookingsManager` | Operational log for managing sales. |
| `/admin/settings` | `AdminSettings` | Global site metadata (Phone, Email, Social). |

## 6. FEATURE MAP
*   **Travel Oracle (AI):** Located in `HomePage.tsx`. Uses `geminiService.ts` to take a text "vibe" and match it to a `packageId`.
*   **Package Authoring (CRUD):** `PackageForm.tsx` is the most complex feature. It handles:
    *   Multi-image upload with progress bars.
    *   Dynamic arrays for Itinerary days, Inclusions, and Highlights.
    *   Flexible payment rules (Advance vs Full).
*   **Booking Engine:** `BookingPage.tsx` integrates the frontend with Razorpay. It supports "Request Only" (free/manual) and "Instant Pay."
*   **Settings Sync:** Global variables (like the contact phone number) are stored in a specific Firestore document (`settings/main`) and synced across the footer and booking pages.

## 7. IMPORTANT FILES TO KNOW
| File Path | Responsibility | Why/When to Edit |
| :--- | :--- | :--- |
| `src/firebase/firestoreService.ts` | **Data Access Layer** | Change the schema or add new DB queries. |
| `src/admin/PackageForm.tsx` | **Inventory Logic** | Modify how trips are created or add new trip attributes. |
| `src/pages/BookingPage.tsx` | **Payment Logic** | Update Razorpay integration or booking validation. |
| `src/index.css` | **Design System** | Update the Tailwind 4 theme (colors, shadows, borders). |
| `src/firebase/config.ts` | **Firebase Init** | Change Firebase project credentials or app environment. |
| `firestore.rules` | **Security** | Restrict or grant permissions to DB collections. |

## 8. DATA MODEL / ENTITY MAP
### **Package (Collection: `packages`)**
*   `title`, `description`, `category`: Text.
*   `price`, `advanceAmount`: Numbers.
*   `images`: Array of Storage URLs.
*   `itinerary`: Array of strings (Day 1, Day 2...).
*   `paymentConfig`: Booleans (`allowFullPayment`, `allowAdvancePayment`, `allowRequestBooking`).
*   `availableDates`: Array of `{startDate, endDate}` objects.

### **Booking (Collection: `bookings`)**
*   `packageId`, `packageTitle`: References to the trip.
*   `status`: `'Pending' | 'Paid' | 'Cancelled'`.
*   `paymentMode`: `'full' | 'advance' | 'request'`.
*   `customerInfo`: Name, Email, Phone, Travelers count.
*   `razorpayPaymentId`: Linked transaction ID.

### **Settings (Collection: `settings`, Doc: `main`)**
*   Global contact info and the list of `categories` used in dropdowns.

## 9. FIREBASE UNDERSTANDING
*   **Firestore:** Uses standard collections. The `settings` collection uses a fixed ID (`main`) to act as a global config object.
*   **Storage:** Images are organized in folders by package ID: `packages/{id}/{filename}`.
*   **Security Concern:** `firestore.rules` allows `create` on the `bookings` collection for unauthenticated users. This is necessary for customers to book but makes the collection vulnerable to spam/bots without a server-side check (like App Check or Cloud Functions).

## 10. AUTH / ACCESS CONTROL
*   **Strength:** Standard Firebase Auth. Protected routes use a `<ProtectedRoute />` component which wraps the `<Outlet />` in `App.tsx`.
*   **Weakness:** The `AdminLogin.tsx` file contains a **"CREATE DEMO ADMIN"** button that calls `signUp`. This is a significant risk if left in a production deployment.
*   **Authorization:** The app currently treats any authenticated user as an admin. There is no "customer" role yet.

## 11. UI / DESIGN SYSTEM UNDERSTANDING
*   **System:** "Neo-Brutalism."
*   **Colors:** `brand-yellow` (#F2B233), `brand-red` (#C23A2B), `void` (#0A0A0A), `paper` (#F5F5F5).
*   **Components:** Components like `btn-brutal` and `kinetic-card` (hover effects) are defined via Tailwind classes and standard CSS in `index.css`.
*   **Responsive Logic:** Uses standard Tailwind `md:` and `lg:` prefixes. The sidebar in `AdminLayout.tsx` uses an overlay/drawer pattern for mobile.

## 12. DEVELOPMENT GUIDE
*   **New Page:** Add to `src/pages`, then register route in `src/App.tsx`.
*   **New Admin Tool:** Add to `src/admin`, register in `App.tsx` (inside the protected group), and add a link to the `NAV_ITEMS` array in `AdminLayout.tsx`.
*   **Firebase Change:** Edit `firestoreService.ts` first, then update the interfaces in the components.
*   **Styles:** Tailwind 4 configuration is in `src/index.css`. Use the `@theme` block.

## 13. RISK / TECH DEBT REGISTER
| Issue | Severity | Affected Files | Recommendation |
| :--- | :--- | :--- | :--- |
| **Demo Admin Button** | **CRITICAL** | `AdminLogin.tsx` | Remove the "Create Demo Admin" button immediately before production deploy. |
| **Client-side Price Trust** | **HIGH** | `BookingPage.tsx` | The final amount is calculated on the client. A user could manipulate the payload to pay 1 INR. Move price verification to a Cloud Function. |
| **No Payment Webhooks** | **MEDIUM** | `firestoreService.ts` | Status is updated on the client after the Razorpay popup. If the user closes the tab mid-redirect, the DB remains "Pending." Implement Razorpay Webhooks. |
| **Large Package Fetches** | **LOW** | `PackagesPage.tsx` | Currently fetches all packages. Will cause lag if inventory reaches 100+. Add Firestore pagination. |

## 14. HANDOFF SUMMARY
*   **Project Type:** A specialized Travel E-commerce CMS.
*   **Core Structure:** React SPA + Firebase BaaS.
*   **Mental Model:** Public site reads `packages` $\rightarrow$ User creates a `booking` $\rightarrow$ Admin manages `bookings` and `packages`.
*   **Files to learn first:** `firestoreService.ts` (data), `PackageForm.tsx` (admin logic), `BookingPage.tsx` (payment flow).
*   **Top Risk:** The pricing logic is currently client-side. Validate all payments against the admin-set package price before fulfilling.

---

# DEVELOPER CONTEXT SNAPSHOT

*   **Type:** Travel CMS & Booking SPA.
*   **Stack:** React 19, TS, Vite, Tailwind 4.0, Firebase (Auth/Store/Storage).
*   **Major Modules:** `src/admin` (Dashboard), `src/pages` (Booking/Details), `src/firebase` (Service Layer).
*   **Key Routes:** `/admin/packages/add` (Authoring), `/book/:id` (Checkout), `/packages` (Discovery).
*   **Core Entities:** `Package` (Trip details), `Booking` (Customer transaction), `Settings` (Global config).
*   **Critical Services:** `firestoreService.ts` (CRUD), `geminiService.ts` (AI Matching), `authService.ts` (Session).
*   **Auth Model:** Firebase Email/Password + `ProtectedRoute` wrapper.
*   **Major Risks:** Client-side pricing calculation; Demo-account creation button in UI; No payment webhooks.
*   **Best Entry Point:** Start at `App.tsx` for routing and `firestoreService.ts` for data structure understanding.
