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
- `const books = { ... }` — keyed by slug. Fields: `title`, `author`, `annotation`, `status`, `team`, and optionally `image` (path to local cover image). Active books: `leeson`, `oppenheimer`, `white`, `nozick`, `arendt`. Completed books: `block`, `rothbard-anatomy`, `hayek`.
- `const activeBookIds = ['leeson', 'oppenheimer', 'white', 'nozick', 'arendt']` — ordered list of active translations.
- `const plannedBooks = [ ... ]` — 44 entries, each `{ title, author }` with optional `image` for books that have a local cover.
- `const mainPage = { name, heroText, heroSubtitle, mission }` — hero section text.
- `const events = [ ... ]` — event objects with `num`, `name`, `date`, `time`, `place`, `annotation`.
- `const eventImages = { 1: "Images/The Road to Serfdom.webp", ... }` — optional per-event images.
- `const partners = [ ... ]` — partner objects with `num`, `name`, `site`.
- `const partnerLogos = { 1: "Logos/საქართველოს უნივერსიტეტი Logo.png" }` — local partner logos.

**Key JS functions:**
- `renderActiveBooks()` — renders active translation rows with Open Library cover images.
- `renderDoneBooks()` — renders completed books as compact rows with local cover images (uses `b.image` field on the book object directly).
- `renderPlannedBooks()` — renders planned books list with local cover images where available, Open Library fallback; first 4 visible, rest have `.todo-hidden`.
- `showAllPlanned(btn)` — toggles visibility of all planned books.
- `renderEvents()` — renders events, skipping entries with empty name/date/place. Each event has a Google Calendar button.
- `renderPartners()` — renders partner logos in the footer with name below logo.
- `openBook(id)` / `closeBookModal()` / `closeModal(e)` — book detail modal.
- Scroll listener on `window` — updates active nav link based on scroll position.

**Styling:** CSS custom properties (`--bg`, `--text`, `--muted`, `--border`, `--border-strong`, `--green`, `--blue`, `--gray`, `--gold`) define the color scheme. Layout uses Flexbox. Mobile breakpoint is `600px`. Section labels (`.section-label`) use Cormorant Garamond, uppercase, bold (`font-weight: 700`).

**Cover images:** Active and planned books load covers from local `image` field where set; Open Library API is used as fallback:
`https://covers.openlibrary.org/b/title/${encodeURIComponent(title)}-S.jpg`
Images that 404 are hidden via `onerror="this.style.display='none'"`.

**Sections (in DOM order):** Header/Nav → Hero → მიმდინარე თარგმანები (`#projects`) → გადათარგმნილი წიგნები (`#translated`) → დაგეგმილი წიგნები (`#planned`) → ღონისძიებები და კონფერენციები (`#events`) → Footer (partners + copyright).

## Current UI Details

- **Header nav**: links to `#projects` (თარგმანები), `#events` (ღონისძიებები), and modal openers for პარტნიორები and კონტაქტი.
- **Hero** shows title (`h1`), subtitle (`.hero-subtitle`), mission text (`.mission`), and a stats row: **5 active, 3 done, 44 planned, 1 partner**. All text is set from `mainPage` in JS.
- **Active Translations** (`#projects`): compact list rendered by `renderActiveBooks()`. Each row: 28×42px cover, green dot, title, author, optional progress bar.
- **Completed Books** (`#translated`): compact list rendered by `renderDoneBooks()`. Each row: 36×54px local cover image, title, author, "✓ დასრულებული" badge. Click opens modal with full annotation/team.
- **Planned Books** (`#planned`): compact list rendered by `renderPlannedBooks()`. Each row: 32×48px cover, title, author. Shows 4 by default; "see more" button toggles the rest.
- **Events** (`#events`): rendered by `renderEvents()`. Empty events are filtered. Each row shows a date column, optional thumbnail, event title, place, and a Google Calendar button.
- **Footer**: partner logo+name links rendered by `renderPartners()` (`.partner-item` / `.partner-name` classes). Email link in footer-bottom. No donate section.

## Source Data

`database.xlsx` is the source of truth for content. Sheets and their use:
- `Hero section` — site name, hero text, mission (col A/B/C, row 2)
- `ჩვენს შესახებ` — about text (row 3, col 1) — section removed from page
- `done` — completed books (cols: num, author, title, translator, editor, image key, status, annotation, publisher)
- `books in translation` — active translation books (cols: author, title, translator, editor, progress, annotation)
- `Plan` — planned books list
- `events` — event entries (cols: num, name, date, place, annotation)
- `Partners` — partner name and site
- `Donations` — bank account numbers (donate section currently removed from site)

To regenerate `data.js` from the database run `python generate_data.py` (requires `pip install openpyxl`). This file is currently unused by the site (the script does not yet read books data).

## Non-Code Assets

- `Images/` — book cover images. Files present: `The Invisible Hook.jpeg`, `The State.jpeg`, `Better Money.jpg`, `Anarchy, State, and Utopia.jpg`, `The Road to Serfdom.webp`, `On Violence.jpg`, `დაცვა ვერდასაცავის.jpg`, `სახელმწიფოს ანატომია.jpeg`, plus covers for several planned books (Bureaucracy, Freedom and the Law, On Power…, On Revolution, The Noblest Triumph, The Rise and Decline of the State, Theory and History, Antitrust and Monopoly, Economics in One Lesson, The Morality of Law). Also `favicon.png`.
- `Logos/საქართველოს უნივერსიტეტი Logo.png` — partner logo.
- `generate_data.py` — Python script to generate `data.js` from `database.xlsx`
- `data.js` — generated data file (exists on disk, not used by the site)
