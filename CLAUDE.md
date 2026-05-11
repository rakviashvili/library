# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the website for **წიგნიერება** (Tsigniereba), a Georgian project for developing Georgian scientific language through translating well-known books in social sciences and humanities. The project is a fully static single-page application with no build process.

## Running the Project

Open `index.html` directly in a browser — there is no build step, server, or dependencies to install.

The `generate_data.py` script reads `database.xlsx` and regenerates `data.js`, but `data.js` is **not referenced by `index.html`** — all data is hardcoded inside `index.html` directly. Do not add `<script src="data.js">` to the HTML; it broke the site before (CORS on `file://` protocol).

## Architecture

All application logic lives in a single file: `index.html`. It contains embedded CSS (in `<style>`) and embedded JavaScript (in `<script>`), with no external JS or CSS files.

**Data model — all hardcoded in `<script>`:**
- `const books = { ... }` — keyed by slug. Fields: `title`, `author`, `annotation`, `status`, `team`, and optionally `image` (image key for completed books). Active books: `leeson`, `oppenheimer`, `white`, `nozick`. Completed books: `block`, `rothbard-anatomy`.
- `const bookImages = { b1: "Images/b1.jpg", b2: "Images/b2.jpeg" }` — local cover images for completed books.
- `const activeBookIds = ['leeson', 'oppenheimer', 'white', 'nozick']` — ordered list of active translations.
- `const plannedBooks = [ ... ]` — 45 entries, each `{ title, author }`.
- `const mainPage = { name, heroText, heroSubtitle, mission }` — hero section text.
- `const events = [ ... ]` — event objects with `num`, `name`, `date`, `place`, `annotation`.
- `const eventImages = { 1: "Images/e1.jpg", ... }` — optional per-event images.
- `const partners = [ ... ]` — partner objects with `num`, `name`, `site`.
- `const partnerLogos = { 1: "Logos/PLogo _ 1.jpg", ... }` — local partner logos.

**Key JS functions:**
- `renderActiveBooks()` — renders active translation rows with Open Library cover images.
- `renderDoneBooks()` — renders completed books as compact rows with local cover images.
- `renderPlannedBooks()` — renders planned books list with Open Library cover images; first 4 visible, rest have `.todo-hidden`.
- `showAllPlanned(btn)` — toggles visibility of all planned books.
- `renderEvents()` — renders events, skipping entries with empty name/date/place.
- `renderPartners()` — renders partner logos in the footer.
- `openBook(id)` / `closeBookModal()` / `closeModal(e)` — book detail modal.
- Scroll listener on `window` — updates active nav link based on scroll position.

**Styling:** CSS custom properties (`--bg`, `--text`, `--muted`, `--border`, `--border-strong`, `--green`, `--blue`, `--gray`, `--gold`) define the color scheme. Layout uses Flexbox. Mobile breakpoint is `600px`. Section labels (`.section-label`) use Cormorant Garamond, uppercase, bold (`font-weight: 700`).

**Cover images:** Active and planned books load covers from Open Library API:
`https://covers.openlibrary.org/b/title/${encodeURIComponent(title)}-S.jpg`
Images that 404 are hidden via `onerror="this.style.display='none'"`.

**Sections (in DOM order):** Header/Nav → Hero → მიმდინარე თარგმანები (`#projects`) → გადათარგმნილი წიგნები (`#translated`) → დაგეგმილი წიგნები (`#planned`) → ღონისძიებები და კონფერენციები (`#events`) → მხარდაჭერა (`#donate`) → Footer (partners + copyright).

## Current UI Details

- **Header nav** is empty (no links currently).
- **Hero** shows title (`h1`), subtitle (`.hero-subtitle`), mission text (`.mission`), and a stats row with 4 stats (active, completed, planned, partners). All text is set from `mainPage` in JS.
- **Active Translations** (`#projects`): compact list rendered by `renderActiveBooks()`. Each row: 28×42px Open Library cover, green dot, title, author, optional progress bar.
- **Completed Books** (`#translated`): compact list rendered by `renderDoneBooks()`. Each row: 36×54px local cover image, title, author, "✓ დასრულებული" badge. Click opens modal with full annotation/team.
- **Planned Books** (`#planned`): compact list rendered by `renderPlannedBooks()`. Each row: 32×48px Open Library cover, title, author. Shows 4 by default; "see more" button toggles the rest.
- **Events** (`#events`): rendered by `renderEvents()`. Empty events are filtered. Each row shows a date column, event title, and place.
- **Donate** (`#donate`): static bank account info. Three rows: TBC (blue badge SVG), BOG (red badge SVG), Bitcoin (orange circle SVG), each followed by the account number in monospace.
- **Footer**: partner logo links rendered by `renderPartners()`. Email link in footer-bottom.

## Source Data

`database.xlsx` is the source of truth for content. Sheets and their use:
- `main page` — site name, hero text, mission (col A/B/C, row 2)
- `Hero saction` — hero subtitle
- `ჩვენს შესახებ` — about text (row 3, col 1) — section removed from page
- `done` — completed books (cols: num, author, title, translator, editor, image key, status, annotation, publisher)
- `books in translation` — active translation books
- `Plan` — planned books list
- `events` — event entries (cols: num, name, date, place, annotation)
- `Partners` — partner name and site (placeholder name: "ჯერ არავინ")
- `Donations` — intro text, TBC/BOG/Bitcoin account numbers

To regenerate `data.js` from the database run `python generate_data.py` (requires `pip install openpyxl`). This file is currently unused by the site.

## Non-Code Assets

- `database.xlsx` — master content database
- `Images/b1.jpg`, `Images/b2.jpeg` — completed book covers
- `Images/e1.jpg`, `Images/e2.jpeg` — event images (optional, keyed by event num)
- `Logos/PLogo _ 1.jpg`, `Logos/PLogo _ 2.jpg`, `Logos/PLogo _ 3.jpg` — partner logos
- `logo.svg` — standalone 512×512 site logo (uses `#1a1a18` hardcoded, not `currentColor`)
- `generate_data.py` — Python script to generate `data.js` from `database.xlsx`
- `data.js` — generated data file (exists on disk, not used by the site)
