# LGU Travel Order Dashboard

A web-based dashboard and automation system that generates, tracks, and manages official Travel Orders for a local government unit — built to replace a manual, paper-heavy process with a single Google Sheets–connected interface.

Built as an intern project for the **Human Resource Management Office (HRMO), Municipal Government, Ilocos Sur.**

## Preview

<p align="center">
  <img src="./Dashboard%20Screenshot.png" width="800" alt="Dashboard Preview">
</p>

---

## Overview

Employees submit a Google Form with their travel details (name, department, purpose, destination, dates, etc.). This system takes over from there:

- Automatically generates a formatted Travel Order document from the employee's personal template
- Assigns and tracks official Travel Order numbers (TO#) in sequence
- Surfaces everything in a live dashboard — no more digging through the spreadsheet
- Lets HRMO staff review, mark as done, correct, cancel, or reissue Travel Orders without touching Drive or the sheet directly
- Keeps an audit trail of every action taken

It's built entirely on **Google Apps Script** and the **Google Workspace** stack (Sheets, Docs, Drive, Forms), with a custom HTML/CSS/JS frontend served through `doGet()`.

---

## Why this exists

The HRMO previously processed Travel Orders manually — each request meant locating the employee's folder, copying a template, filling in the details by hand, and manually tracking the next TO# in a logbook. Errors (skipped numbers, name typos, misfiled documents) were common and hard to catch.

This project automates that entire pipeline while keeping a human in the loop for review, corrections, and sign-off — the system drafts and organizes, HRMO staff verify and finalize.

---

## Features

### Automated document generation
- Triggered on every new form submission (`onFormSubmit`)
- Matches the submitted name to the employee's Drive folder using fuzzy, order-independent name matching (handles `Last, First`, `First Last`, middle names, and formatting inconsistencies)
- Copies the employee's personal template and fills in trip details, dates, purpose, destination, and computed fields (remarks, charges) based on department and expense type
- Assigns TO numbers automatically, computed by chronological sequence rather than row order, so the numbering stays consistent even when rows are edited or reprocessed

### Live dashboard
- Real-time stats (total, processed, done, cancelled) with animated counters
- Searchable, filterable table of all Travel Orders with status pills (New, Processed, Done, Error, Empty, Cancelled, Ignored, In Progress)
- Full-detail modal per Travel Order, with inline editing that regenerates the document automatically
- Document preview, print, and "visit folder" shortcuts without leaving the dashboard

### Error handling & recovery
- Flags rows with incorrect name formatting or missing employee folders as **Error**, with a guided "1ClickFix" flow
- Detects and repairs rows with partially or fully missing Status/TO#/Comments cells
- TO# sequence checker to catch duplicate or out-of-order numbers
- Recovery logic that can reconstruct a row's correct status by reading the actual filename/content of its document in Drive — useful if the sheet gets manually cleared or corrupted

### Cancellation & reissue workflow
- Cancel a Travel Order outright (with a required reason), or
- "Update TO Details" — cancels the old TO# and reissues a corrected one, either reusing the same number or assigning the next available one depending on what's already been filed
- Full history view showing what changed between the old and new version
- "Report Late Travel" — lets a travel order filed out of chronological order be logged against the correct date without breaking the TO# sequence, with its own audit trail

### Employee management
- Add, rename, move, deactivate, or delete employee folders directly from the dashboard
- Auto-creates the Drive folder structure and copies the correct signatory template
- Bulk view across all departments with active/inactive filtering

### Notifications & activity log
- In-app bell/alert system for new submissions and unresolved errors, with browser tab flashing and optional sound when the tab is backgrounded
- Full activity log (monthly sheets) recording every action: creation, edits, cancellations, reissues, deletions, and fixes
- Automatic yearly backups of every finalized row to a separate backup spreadsheet

---

## Tech stack

| Layer | Technology |
|---|---|
| Backend logic | Google Apps Script (JavaScript, V8 runtime) |
| Data store | Google Sheets |
| Document generation | Google Docs API (via Apps Script `DocumentApp`) |
| File storage | Google Drive (per-employee folders + templates) |
| Frontend | Vanilla HTML / CSS / JavaScript, served via `HtmlService` |
| Intake | Google Forms |

No external frameworks, no build step — everything runs inside the Apps Script container and is deployed as a web app.

---

## How it works (high level)

1. **Form submission** → `onFormSubmit()` fires, locks the sheet, sorts and locates the new row.
2. **Name resolution** → the system tries multiple parsing strategies (comma-separated, space-separated, reordered) to match the submitted name against the correct employee folder in Drive.
3. **TO# assignment** → `getCorrectToNumber()` recalculates the correct sequential number based on submission timestamp order, not sheet row position.
4. **Document creation** → the employee's personal template is copied, the header and body tables are filled in, and the file is renamed with the TO# for traceability.
5. **Sheet update** → Status, TO# (as a hyperlink to the folder), and a rich-text comment (linking to the file) are written back, with conditional formatting for quick visual scanning.
6. **Dashboard sync** → `getDashboardData()` reads the sheet fresh on every dashboard load/poll and reflects it in the UI, including derived states like "In Progress" (partially completed sign-off checklist) or "Empty" (missing cells).
7. **HRMO review** → staff use the dashboard to mark orders as Done (with a 5-point sign-off checklist), fix errors, cancel/reissue, or manage employees — every action is logged.

---

## Repository contents

- `Dashboard.html` — the full frontend: markup, styles, and client-side logic for the dashboard, all modals, search, employee management, and notifications.
- `Code.gs` — the Apps Script backend: form trigger, document generation, TO# sequencing, error recovery, cancellation/reissue, employee folder management, activity logging, and all `google.script.run` endpoints the frontend calls.

---

## Notes on portability

This project is wired directly to a specific Google Sheet, Drive folder structure, and set of department/signatory data for the Municipality of Santa's HRMO, so it isn't a drop-in template — the spreadsheet IDs, folder IDs, and department mappings (`SPREADSHEET_ID`, `DEPARTMENTS_FOLDER_ID`, `ACTIVITY_LOG_SPREADSHEET_ID`, `MAIN_TEMPLATE_FOLDER_ID`, `DEPARTMENT_CODES`, etc.) are hardcoded at the top of `Code.gs` and would need to be replaced with your own for a different deployment.

That said, the underlying patterns — form-triggered document generation, sequential numbering that survives out-of-order edits, name-matching against a folder tree, and a dashboard layered on top of a spreadsheet via Apps Script's `HtmlService` — are reusable for similar internal tools.

---

## Author

Built by **Rio**, HRMO Intern — Municipal Government of Santa, Ilocos Sur, Philippines.
