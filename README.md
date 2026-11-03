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
