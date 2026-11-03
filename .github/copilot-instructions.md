# Collab LMS — Project Overview & GitHub Copilot Agent Instructions

## 1. Project Overview & Product Vision

### 1.1 Purpose & Target Audience
Collab LMS is a specialized Learning Management System tailored for adult English learners working in **Product Design, UX/UI, Software Engineering, and Tech Leadership**.
- **Pedagogical Persona:** Professional, technically precise, respectful, and outcome-oriented.
- **Strict Tone Constraints:** Avoid childish gamification, patronizing encouragement, arbitrary progress percentages, and detached academic assessment language. Address students directly as "you/your".
- **Primary Delivery Channels:** Desktop and mobile web application hosted on Google Apps Script Web App infrastructure.

### 1.2 The Five Core Pillars
1. **Live Classes & Schedule:** Google Calendar integration, Google Meet deep-links, and availability-governed self-service rescheduling.
2. **Class History & Pedagogical Recaps:** Recorded class attendance logs paired with teacher-reviewed, transcript-grounded pedagogical summaries (strictly avoiding personal/administrative teacher notes).
3. **Student Evolution:** Longitudinal competency tracking across Grammar, Vocabulary, Fluency/Interaction, and Professional Communication powered by Google Gemini API analysis of verified student work.
4. **Tracking (PDP):** Professional Development Plan tracker monitoring Speaking and Writing deliverables.
5. **Practice (Writing Studio & Workbooks):** Native Google Docs essay drafting with live autosave alongside interactive Ebook unit exercises.
- **Contextual Anchor ("Never Be Lost"):** The LMS tracks the student's current Ebook, Unit, and physical Page number across *Home*, *My Classes*, and *Resources*, ensuring students resume directly from where their last class ended.
- **Community & Resources:** A shared feed for design teardowns, technical links, and discussions (separated from formal grading) combined with an Ebook PDF library.

---

## 2. Technical Stack & Runtime Environment

- **Server Runtime:** Google Apps Script (V8 Engine).
- **Backend File Extensions:** Must strictly be `.gs` (or `.js` via Clasp). Never use `.java`.
- **Database Layer:** Google Sheets accessed via `SpreadsheetApp`.
- **File & Document Storage:** Google Drive (`DriveApp`) and Google Docs (`DocumentApp`).
- **Calendar & Video:** Google Calendar (`CalendarApp`) and Google Meet.
- **External AI Integration:** Google AI Studio / Gemini API (`UrlFetchApp` targeting `gemini-2.5-flash`).
- **Frontend Layer:** Single-Page Application (`Index.html`) served via `HtmlService.createHtmlOutputFromFile('Index')`.
- **Client-Server Bridge:** Asynchronous RPC via `google.script.run` with explicit success and failure handlers.
- **Styling & UI:** DaisyUI 5, Tailwind CSS tokens, bespoke theme variables (Lavender Analytics and Peach Workspace modes, Light/Dark modes), Cally Web Component Calendar, and Quill.js 2.0.3 editor.

---

## 3. Strict Google Apps Script Architecture Rules

When generating or editing server-side code, GitHub Copilot MUST respect these non-negotiable constraints:

1. **Flat Global Namespace:** All `.gs` files run in a single shared global scope.
   - Never declare duplicate function names across different files.
   - Private helpers must end with a trailing underscore (e.g., `calculateHash_()`).
   - Public RPC endpoints callable from the client via `google.script.run` must NOT have a trailing underscore.
2. **No Synchronous Waterfall Requests:**
   - Client-side startup must use the unified **"Single Check Roll"** endpoint: `getStudentBootstrapContext(token)`. Never allow the browser to make multiple sequential roundtrips on page load.
3. **In-Memory Request Memoization:**
   - Spreadsheet operations (`SpreadsheetApp.openById`) incur a 600ms–1500ms network penalty.
   - All spreadsheet reads must use `02_DbRepository.gs` (`getTableRecords_`), which memoizes data in `_tableRecordsCache` during execution.
4. **Lock Contention Management:**
   - `LockService.getScriptLock()` is a global mutex across all concurrent users.
   - Never acquire a lock during read operations or external API calls (e.g., `UrlFetchApp` or `MailApp`).
   - Compute data, validate schemas, and prepare payloads *before* acquiring locks.
   - Wrap all writes in `try ... finally { lock.releaseLock(); }` and immediately flush via `SpreadsheetApp.flush()`.
5. **Fail-Closed Security Principle:**
   - Always validate bearer tokens using `assertStudentSession_(token)` or `assertTeacherSession_(token)` before reading or writing data.
   - If an account is inactive, missing, or has a mismatched credential hash, abort execution immediately with an explicit error.
   - Never log raw passwords, plain bearer tokens, or cryptographic peppers.

---

## 4. Database Schema & Data Dictionary

### 4.1 Script Properties Registry (`CONFIG.PROPERTIES`)
- `STUDENT_LOGIN_REGISTRY_ID`: ID of the master Student Registry spreadsheet.
- `STUDENT_LOGIN_AUTH_PEPPER`: Secret cryptographic pepper for PBKDF2/HMAC-SHA256 password hashing.
- `STUDENT_CLASS_REPORT_SPREADSHEET_ID`: ID of the Class Reports & Attendance spreadsheet.
- `STUDENT_PDP_TASK_REGISTRY_ID`: ID of the consolidated Task Registry spreadsheet.
- `STUDENT_EVOLUTION_REGISTRY_ID`: ID of the Student Evolution reports and goals spreadsheet.
- `GEMINI_API_KEY`: Google AI Studio API key.
- `GEMINI_MODEL`: Gemini model identifier (Default: `gemini-2.5-flash`).
- `TEACHER_ADMIN_EMAIL`: Verified teacher administrator email address.
- `TEACHER_ADMIN_PASSWORD_SALT`: Salt for teacher authentication.
- `TEACHER_ADMIN_PASSWORD_HASH`: Salted PBKDF2 hash for teacher authentication.

### 4.2 Key Spreadsheet Tabs & Schemas (`CONFIG.SHEETS` & `CONFIG.SCHEMAS`)
- **`Students`**: `['Student ID', 'Display Name', 'Email', 'Active', 'Temporary Password', 'Password Salt', 'Password Hash', 'Must Change Password', 'Created At', 'Updated At', 'Last Login', 'Drive Folder ID', 'PDP Spreadsheet ID', 'Task Descriptions File ID', 'Technical Feedbacks Folder ID', 'Approved Transcripts Folder ID', 'Class Transcripts Folder ID']`
- **`Tasks`**: `['Task ID', 'Student ID', 'Student Name', 'Class/Unit', 'Task Type', 'Activity', 'Status', 'Source PDP Spreadsheet ID', 'Source Tab', 'Source Row', 'Last Synced At', 'Submission Status', 'Submission File ID', 'Submission URL', 'Submitted At']`
- **`Portal Sessions`**: `['Token SHA-256', 'Session JSON', 'Expires At']`
- **`Student Profiles`**: `['Student ID', 'Level', 'Professional Track', 'Updated At', 'Version']`
- **`Teacher Feedback Reviews`**: `['Review ID', 'Review Key', 'Student ID', 'Type', 'Source ID', 'Task ID', 'Status', 'Revision', 'Source Fingerprint', 'Source Title', 'Feedback JSON', 'Created At', 'Updated At', 'Published At', 'Read At']`
- **`_Collab Class Cycle`**: `['Cycle Key', 'Event ID', 'Student ID', 'Student Name', 'Original Class Date', 'Original Start ISO', 'Report Row', 'Status', 'Updated At', 'Requested New Start ISO', 'Reschedule Reason', 'Revision']`
- **`Reschedules`**: `['Date Submitted', 'Student', 'Event ID', 'Reason', 'Requested Time', 'Completed in Google', 'Email Sent']`
- **`Speaking Notes`**: `['Student ID', 'Task ID', 'Last Updated', 'Notes Content']`

### 4.3 Obsolete / Discarded Artifacts
- **`Private Beta Learning.xlsx`**: Deprecated test artifact. Do not create dependencies or endpoints for it.

---

## 5. Current System State & Refactoring Status

### Completed Operations (Phase 1 & Phase 2 Foundation):
- [x] Converted all server files from `.java` to `.gs`.
- [x] Removed obsolete, broken files (`StudentEvolution.gs`).
- [x] Cleaned boundary offset errors in `Students` (removed empty Row 2).
- [x] Standardized `appsscript.json` OAuth scopes and upgraded Calendar scope to full access.
- [x] Implemented `01_Config.gs`: Frozen schema constants, properties dictionary, and limits.
- [x] Implemented `02_DbRepository.gs`: In-memory memoized spreadsheet repository (`_tableRecordsCache`), verified with sub-5ms warm read benchmarks.
- [x] Implemented `03_AuthService.gs`: Two-tier session validator (CacheService + Portal Sessions sheet) and fail-closed role guards (`assertStudentSession_`, `assertTeacherSession_`).
- [x] Implemented `04_BootstrapService.gs`: Unified `getStudentBootstrapContext(token)` endpoint aggregating identity, contextual Ebook anchor, calendar, PDP tasks, and review badges into one roundtrip.

### Active Work-in-Progress (Phase 3 & 4):
1. **Frontend Authentication Isolation (`Index.html`):**
   - Decouple the login form submission from the monolithic script tag using an isolated `handleLoginClick_()` function.
   - Enforce `onsubmit="return false;"` and `type="button"` on forms to prevent cross-origin `javascript:void(0)` iframe navigation warnings.
2. **Deterministic Client Lifecycle:**
   - Gate initial workspace hydration behind `customElements.whenDefined('calendar-date')` to prevent un-upgraded Web Component crashes.
3. **Client-Side Bootstrap Wiring:**
   - Finalize `loadStudentHomeSummary` in `Index.html` to consume `getStudentBootstrapContext`.
4. **Modular Service Extraction:**
   - Move remaining features out of the monolithic `Code.gs` into:
     - `05_PdpTaskService.gs` (PDP tasks and speaking preparation notes).
     - `06_WritingStudioService.gs` (Google Docs writing drafts and submissions).
     - `07_ClassLifecycleService.gs` (Calendar sync, availability slots, attendance reports).
     - `08_TeacherReviewService.gs` (Feedback authoring and review queues).
     - `09_EvolutionService.gs` (Gemini transcript analysis and synthesis).
     - `10_LearningHubService.gs` (Community feed and Ebook PDF repository).

---

## 6. Guidelines for GitHub Copilot Code Generation

When acting on this codebase, Copilot must follow these rules:

1. **Check for Existing Functions:** Before defining any backend function, check `01_Config.gs` through `10_LearningHubService.gs` to ensure it is not already defined elsewhere.
2. **Use the Database Repository:** Never call `SpreadsheetApp.openById()` directly inside business logic. Always use `openSpreadsheet_()` or `getTableRecords_()` from `02_DbRepository.gs`.
3. **Strict Parameter Validation:** Validate and sanitize every parameter received from `google.script.run`. Treat client input as untrusted.
4. **Form Elements in HTML:** Never use `action="javascript:..."` or standard `GET` form submission in `Index.html`. Use `type="button"` and `onsubmit="return false;"`.
5. **No Broken String Concatenations:** Ensure multiline string literals use backticks (`` `...` ``) or clean concatenation without unescaped newlines.
6. **No Phantom Endpoints:** If a UI button calls `runServer({ name: 'myFunction', ... })`, verify that `myFunction` exists as an exposed server function without a trailing underscore.
