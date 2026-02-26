# Terra / Sentinel Project Handoff Document

This is a comprehensive reference document for any future developer or AI agent picking up this project. It details the system architecture, how the frontend, backend, and database interact, the current progress, and the roadmap for planned features.

## 🏢 Platform Overview: SENTINEL

**Purpose:** A Global Damage Intelligence Platform.
**Core Value Proposition:** Providing structured, AI-driven disaster severity analytics, active threat metrics, and structural damage distributions by combining live localized feeds with rapid satellite imagery analysis.

## 🛠️ Tech Stack & Architecture

- **Frontend:** React 19 + Vite + Tailwind CSS. 
    - **Aesthetic:** High-fidelity "Cinematic" design system with a True Black (`#05050A`) background and bold Electric Cyan (`#00E5FF`) accents. It uses a "Hub & Spoke" navigation model. 
    - **State Management:** Zustand.
- **Backend:** FastAPI (Python), utilizing Asyncpg for database pooling and Uvicorn for asynchronous execution.
    - **Data Flow Model:** A **Hybrid API Strategy**:
        - **WebSockets** for Live Feeds (Events and Alerts). Eliminates database polling overhead.
        - **REST endpoints** for Heavy Data (Satellite Passes, Analysis Stats, GeoJSON Cartography).
- **Databases:** A **Dual-Database Setup**.
    - **Neon PostgreSQL:** The primary application warehouse (Events, Satellite Metadata, Alerts, Structural Damage Analytics).
    - **Supabase:** Exclusively handles Authentication (Civilian vs Organization) and Storage Buckets (Images, Geotiffs).

## 🔄 How the Components Work Together

### 1. The Global Data Intake Pipeline (Backend -> DB -> WebSocket)
1. **Background Polling:** The backend runs asynchronous polling jobs (`modules/event_monitor/service.py`) tracking live global datasets (e.g., GDACS).
2. **Database Ingestion:** When a new disaster (Flood, Earthquake, Wildfire) is detected, it is upserted into the Neon PostgreSQL `events` table.
3. **Live Broadcasting:** Instantly upon DB insertion, the new event is broadcasted over the global WebSocket (`/api/ws`).
4. **Frontend Consumption:** The frontend React application connects to this WebSocket on load via the Zustand store. It listens for `events_update` and populates the `EventSidebar` live without requesting standard HTTP refreshes.

### 2. The Granular Analytics Pipeline (Frontend SDK -> REST -> Backend HTTP -> DB)
1. **User Interaction:** When a user clicks a specific disaster in the Live Monitor sidebar, the React app stores the event ID in Zustand.
2. **On-Demand Fetching:** Zustand immediately triggers an async REST GET request fetching detailed `/api/intelligence/{analysis_id}` and `/api/satellite/passes/{event_id}`.
3. **Data Return & Rendering:** The backend pulls the heavy statistics and GeoJSON polygons from the Neon DB and returns it via HTTP payload. 
4. **Charting & Maps:** The frontend pipes the GeoJSON vector coordinates into the Leaflet Map layer and feeds structured severity metrics into the Recharts components (SeverityChart, RecoveryChart).

## 📊 Current Progress (As of Initial Handoff)

We have successfully built a robust, skeleton structure and fully integrated the Live WebSocket connections from the database through to the UI.

**Completed Milestones:**
- [x] **Initial Database Seeding:** The `.env` configurations for Neon (Data) and Supabase (Auth) are established. The base `001_schema.sql` migration table layout was created.
- [x] **Cinematic UI Redesign:** Replaced the initial dense monolithic dashboard with a clean "Vapor Clinic" Hub & Spoke navigation structure. Built highly interactive `.btn-solid` UI paradigms. Features are neatly segregated into `/monitor`, `/intelligence`, and `/satellite`.
- [x] **Live Event Sourcing (GDACS):** Extracted real-time geographic data feeds into the Neon database and fixed core coordinate parsing errors.
- [x] **WebSocket API Layer Integration:** Ripped out hardcoded mock data in the React frontend. Re-wired `DamageMap`, `SeverityChart`, `InfraRiskPanel`, and `EventSidebar` components to map directly to real `analysisData` streams via REST and WebSockets in the Zustand store.

## 🚀 Planned Features & Roadmap (Where to pick up from here)

The Next Agent/Developer should focus on the remaining functional modules:

### 1. Satellite Imagery Pipeline (`modules/satellite_pipeline`)
- **Goal:** Connect to the actual Sentinel Hub API using the provided `.env` keys.
- **Task:** When a new event happens, coordinate bounding boxes must be generated, and requests sent to Sentinel Hub to fetch before/after satellite rasters. These need to be stored in Supabase Storage and fed to the `BeforeAfterSlider` UI component.

### 2. AI Damage Intelligence (`modules/damage_intelligence`)
- **Goal:** Use AI multimodal models (Gemini / Groq) to analyze the fetched satellite data.
- **Task:** Hook the image URLs into an AI vision endpoint. Ask the AI to identify damaged structures and return GeoJSON polygons and JSON damage distributions. Store those JSON reports in Neon so the frontend charts can display real impact assessments.

### 3. Authentication & Field Reporting (`modules/ground_truth`)
- **Goal:** Deploy the standard Supabase Auth SDK.
- **Task:** Let users sign in. Implement the `Assess My Area` React module to allow civilians on the ground to upload photos natively. 
- **Task:** Feed the civilian photos back into an AI validation loop to categorize on-the-ground damage severity.

### 4. Advanced Alerts System (`modules/alerts_engine`)
- **Goal:** Watch all datasets and push automated hazard updates.
- **Task:** Complete the python watcher cron jobs that check if a current event is breaching specified user thresholds (e.g. population exposed > 1M) and push active warnings to the global WebSocket for the frontend `AlertPanel`.

### 5. Public URL Reports (`modules/public_report`)
- **Goal:** Allow emergency response teams to share disaster briefs rapidly.
- **Task:** Ensure the `/report/{slug}` React route generates a clean printable PDF layout querying the backend by a unique hash ID.

---
**Terminal Run Commands:**
- Backend: `cd backend && venv\Scripts\activate && uvicorn main:app --reload`
- Frontend: `cd frontend && npm run dev`
