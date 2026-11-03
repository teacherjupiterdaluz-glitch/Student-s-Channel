# Collab LMS — Student's Channel

An educational management platform designed for adult English learners in **Product Design, UX/UI, Software Engineering, and Tech Leadership**.

Built on Google Apps Script (V8 runtime), Google Sheets (database engine), Google Drive, Google Calendar, and the Google Gemini API.

---

## 🚀 The Five Educational Pillars

1. **Live Classes:** Live Google Meet sessions synchronized with Google Calendar, featuring automated attendance tracking and availability-based rescheduling.
2. **Class History & Lesson Recaps:** Transparent lesson logs paired with teacher-approved pedagogical summaries generated from class transcripts.
3. **Student Evolution:** Longitudinal competency assessments across Grammar, Vocabulary, Fluency/Interaction, and Professional Communication powered by Gemini 2.5 Flash.
4. **PDP Tracking:** Professional Development Plan deliverable tracker monitoring in-class speaking tasks and writing assignments.
5. **Practice (Writing Studio & Workbooks):** Native Google Docs essay drafting workspace with autosave and interactive Ebook unit exercises.

---

## 🏗️ Architecture & Technology Stack

- **Backend:** Google Apps Script (JavaScript V8 Engine).
- **Database Layer:** Google Sheets (Repository pattern with in-memory request-level memoization).
- **Frontend:** Single-Page Application (`src/Index.html`) using DaisyUI 5, custom design tokens (Lavender Analytics & Peach Workspace themes), Cally Calendar, and Quill 2.0.3.
- **AI Integration:** Google AI Studio / Gemini 2.5 Flash API for transcript parsing and educational synthesis.

[Client SPA: Index.html]
│ (google.script.run RPC)
▼
[04_BootstrapService.gs] ──► Single Check Roll Aggregator
│
├──► 03_AuthService.gs ──────► Token Verification & Role Security
├──► 02_DbRepository.gs ─────► In-Memory Memoized Sheet Access (0ms warm)
│ └──► Google Sheets: Students, Tasks, Reviews, Sessions
└──► Google Workspace APIs ──► CalendarApp, DriveApp, DocumentApp

---

## 📦 Project Structure

```text
├── .github/
│   └── copilot-instructions.md     <-- Copilot architecture instructions
├── docs/
│   ├── ARCHITECTURE.md             <-- Detailed system and data flow designs
│   ├── DATA_DICTIONARY.md          <-- Sheet tables, column schemas and types
│   └── RUNBOOK.md                  <-- Administrative workflows and troubleshooting
├── src/
│   ├── 01_Config.gs                <-- Constants, properties & schemas
│   ├── 02_DbRepository.gs          <-- In-memory cache & Sheets ORM
│   ├── 03_AuthService.gs           <-- Authentication & role guards
│   ├── 04_BootstrapService.gs      <-- Single check roll aggregator
│   ├── 05_PdpTaskService.gs        <-- Tasks & Speaking notes
│   ├── 06_WritingStudioService.gs  <-- Google Docs writing engine
│   ├── 07_ClassLifecycleService.gs <-- Calendar, classes & attendance
│   ├── 08_TeacherReviewService.gs  <-- Reviews & feedback queues
│   ├── 09_EvolutionService.gs      <-- Gemini AI pedagogical pipeline
│   ├── 10_LearningHubService.gs    <-- Community & Ebook repository
│   ├── appsscript.json             <-- Manifest & OAuth scopes
│   └── Index.html                  <-- Frontend application
└── README.md

