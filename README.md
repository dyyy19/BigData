# NTU Archive

A contribution-gated platform for archiving and accessing past exams, homework, and project files. Built as the final project for Big Data Systems, Spring 2026, National Taiwan University.
For now only covers the area of National Taiwan University.

**Live demo:** https://ntuarchive.vercel.app

## What this is

NTU students currently get past exams through personal networks such as assigned mentors (直屬學長姐), classmates, or scattered forum posts. Students without those connections (freshmen, exchange students, anyone without the right social ties) are left out. NTU Archive replaces that informal, gatekept system with a shared, searchable archive open to any student with an `@ntu.edu.tw` email.

Access is structured as a contribution-gated freemium model: uploading a resource earns merit points, and previewing another student's upload costs merit points (paid to the uploader). This converts the existing "social cost" of asking around for materials into a transparent exchange.

Full project report ('B13902092') covers the target customer analysis, demand evidence, and system design rationale.

## How it works

- **Sign up / sign in** — restricted to `@ntu.edu.tw` email addresses, verified via Supabase Auth
- **Browse** — search and filter resources by department, type (exam/homework/project), or course code. Preview PDFs inline without downloading
- **Upload** — contribute a file with course metadata --> earns merit points
- **Merit system** — new accounts start with 10 merits, uploading earns +10, previewing another student's resource costs 10 merits from the viewer and credits 5 to the uploader; previewing your own uploads is always free
- **Leaderboard** — ranks users by merit balance
- **One-time nickname** — set once at signup or via account settings

## Tech stack

- **Frontend:** Plain HTML, CSS, and JavaScript — no framework or build step
- **Backend:** [Supabase](https://supabase.com) (PostgreSQL database, authentication, file storage)
- **Deployment:** [Vercel](https://vercel.com) (static hosting)

This stack was chosen specifically to avoid writing or maintaining any server code. All backend logic runs through Supabase's JavaScript SDK directly from the frontend, with security enforced at the database level via Row Level Security policies and PostgreSQL functions, not in client-side code.

## Project structure

```
.
├── code
    ├── index.html              # Sign up / sign in page
    ├── browse.html              # Browse, search, filter, preview, leaderboard
    ├── upload.html               # Upload form
├── attachments
    ├── Data_GoogleForm.xlsx
    ├── Data_Internet.xlsx
├── b13902092.pdf              # Full project report
└── README.md
```

## Running locally

No build step or local server framework is required.

1. Clone this repository
2. Open `index.html` with a local server (opening the file directly via `file://` may cause issues with browser security policies for some Supabase calls)
3. The Supabase project URL and publishable (anon) API key are already embedded in each HTML file's `<script>` block — no additional environment configuration is needed to run the demo against the live backend

## Database schema

Three core tables in Supabase PostgreSQL:

- **`resources`** — catalogue of uploaded files (title, course code, department, semester, type, file URL, uploader ID)
- **`contributions`** — logs each upload event per user, powering the contribution-gate logic
- **`profiles`** — per-user nickname, merit balance, premium status, and one-time nickname-change flag

All tables enforce Row Level Security. Merit balance changes are never written directly from the client, they run through two `SECURITY DEFINER` PostgreSQL functions (`award_upload_merit`, `spend_preview_merit`) so a user cannot manipulate their own merit balance by bypassing the frontend.

## Known limitations

- Merit-gating on previews is enforced at the application layer (via the `spend_preview_merit` function), not at the storage layer. A user could in principle bypass the merit cost by directly accessing a file's public storage URL rather than using the Preview button. A production version would issue short-lived signed URLs per preview request to close this gap.
- Department categorization is a fixed dropdown list rather than a normalized lookup table — a reasonable simplification at this scale, but would need to be replaced by an official NTU department reference table for full coverage.
- No automated cleanup of unconfirmed sign-up accounts; Supabase creates a user record on signup before email confirmation, which is a known spam vector at production scale.

See the full report for a complete discussion of these and other go-to-market considerations.

## Reproducing the demand evidence

The demand evidence cited in the report was gathered from:
- Manual search of Dcard, PTT, and Threads for posts related to past-exam access (考古題) within NTU-specific boards/hashtags
- A short Google Forms survey of 22 NTU students across multiple departments

Raw data is summarized in the report (Component 2). Source posts are linked directly in the report's citations.
