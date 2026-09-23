# Project Wall: everything built so far, and research on each

Snapshot taken 23 Sep 2026 from every repo on the `mailadisin-hub` account (plus `Programmer1-Future/trackeralvl`). I read each repo's code, commit history and handoff notes. The findings below come from that reading. The market notes are general context, not fresh competitor pricing.

---

## The wall at a glance

| # | Project | What it is | Stack | Status | Commits |
|---|---|---|---|---|---|
| 1 | **SCG Tuitions App** | Parent/teacher/admin portal for a Swindon tuition centre | HTML/JS + Firebase, GitHub Actions deploy | Live at scg-tuitions.web.app, v1.3.0-beta | 27 |
| 2 | **SCG Masterminds** | Freemium learning quizzes for Reception–Year 3 | Single HTML file + plain Node server, Firebase Auth, Apps Script | Live at scgmasterminds.co.uk | 65 |
| 3 | **Westcote Place Finance Portal** (SCG People Ltd) | Service-charge invoicing portal for a 16-unit building | Next.js 14, Prisma, Neon Postgres, NextAuth, react-pdf, Resend | Deployed on Hostinger; handoff written | 24 |
| 4 | **BATEMAN / Trendzation** | Web-design agency site plus private CRM with an AI chat running on a home PC | Express, EJS, SQLite, Ollama/Qwen3 over Cloudflare Tunnel | SEO added; Hostinger deploy folder | 6 |
| 5 | **Adi.Study** | Personal A-level revision dashboard (Maths, Further Maths, Physics, ESAT/TMUA) | Vanilla JS PWA + Firestore | Most feature-rich solo project | 22 |
| 6 | **trackeralvl** (AQA checklist) | AS/A-level spec checklist synced to Supabase | React 18 + Vite + Supabase | Content corrected to AQA 7356 | 8 |
| 7 | **11+ Vocab Weekly** | Emails 10 vocab words to 11+ students every Monday | Flask, APScheduler, Gmail SMTP | Built in one commit; not deployed | 2 |
| 8 | **Insta Reels Blocker** | Android app that backs out of Instagram Reels | Kotlin AccessibilityService | Single commit, prototype | 1 |
| 9 | **VORTEX agency site** | One-page creative-agency template | Single HTML file | Done (template) | 1 |
| 10 | **Physics-A-level** | (private) | — | **Empty repo** | 0 |
| 11 | **Furthermath-A-level** | (private) | — | **Empty repo** | 0 |

**Themes across the wall:**
- **Education (6 of 9 real projects):** tuition portal, kids' quizzes, 11+ vocab, A-level tracking, revision dashboard. This is the core area.
- **Built for family and local clients:** SCG Tuitions, SCG People and Westcote Place are all real businesses with real users. That raises the bar for security and data protection.
- **Choosing simpler stacks after hosting trouble:** Masterminds dropped Next.js for a 50-line Node server after 503 errors on Hostinger. Across projects there is a clear preference for no build step.

---

## Security review

A separate security review of the live projects was done and has been shared privately with the owner, not published here.

---

## Project-by-project research

### 1. SCG Tuitions App: parent/teacher portal
**What it does:** WhatsApp-style chat between parents and teachers, plus attendance, homework, books given, payments with invoices, assessment reports and re-registration. It has three roles (parent, teacher, admin) and push notifications through Firebase Messaging. It's installable as a PWA.

**Strengths**
- Broad feature set for a no-framework app. Each feature has a matching teacher page and parent page.
- Well documented: an Obsidian `brain-dump/` with version history, learnings and caching notes.
- Deploys automatically through GitHub Actions to Firebase Hosting.

**Gaps and risks**
- Firestore security rules need a review (see the private security notes).
- Pending from its own notes: the admin role hasn't been set yet, file uploads need the Blaze plan, and there's no offline fallback page.
- It's deployed from a `claude/...` branch rather than `main`, so it's easy to deploy the wrong thing.

**Market context:** UK tutoring centres usually use tools like Teachworks, TutorBird or MyTutor-style marketplaces, or just WhatsApp and spreadsheets. A single-centre custom portal wins on fit and cost. The part most worth productising is the parent-facing layer (chat, attendance, invoices) for other small tuition centres.

**Next moves:** review the security rules and add rules unit tests with the Firebase emulator. Move deploys to `main`. Enable Blaze with a budget alert. Then do the App Store / Play Store prep via a PWA wrapper (for example Capacitor).

---

### 2. SCG Masterminds: kids' quiz platform
**What it does:** Audio-first quizzes for ages 4–8. Reception has 5 phonics phases with gated unlocking. Years 1–3 have English comprehensions and procedurally generated maths, and there's a 66-question 11+ bank. The freemium model is £5/mo for English or £8/mo for English + Maths. Results go to a Google Sheet via Apps Script and are emailed to parents.

**Strengths**
- Strong UX choice: text-to-speech on everything, so pre-readers can use it on their own.
- Very cheap to run: one HTML file, a Node server with no dependencies, and Sheets as the database.
- Most actively developed project (65 commits), with a clear handoff document.

**Gaps and risks**
- Stripe isn't wired up yet, so there's no revenue path so far. Subscription status should be checked on the server.
- The whole app is one ~2,500-line file, which is fragile to edit (the handoff already needs a custom syntax-check one-liner).
- Year 4–6 content is missing, and those are the 11+ prep years, which is where parents spend the most.
- Children's data (names, parent emails) goes to a Google Sheet. That needs a UK GDPR / Age Appropriate Design Code review.

**Market context:** Competitors in this space include Doodle Maths/English, IXL, Atom Learning (11+), Reading Eggs and Teach Your Monster to Read (phonics). The £5–8/mo price sits below most of them. The real differentiator is being "built by tutors" and linked to the physical tuition centre.

**Next moves:** set up Stripe Payment Links, a webhook, and a Firestore `tier` field. Prioritise Year 4–5 11+ content over Year 6. Split the HTML into modules (even plain `<script src>` files, no build step needed).

---

### 3. Westcote Place Finance Portal (SCG People Ltd)
**What it does:** A service-charge portal for a building with 15 flats and 1 commercial unit. It has two cost schedules (A: whole building, B: flats only), annual budgets, a management fee percentage, quarterly invoice generation with PDFs, payment reconciliation and document sharing with per-unit visibility. Leaseholders log in to see their own invoices.

**Strengths**
- The most professionally engineered project: typed stack, 14-model Prisma schema including an `AuditLog`, Zod validation, loading skeletons and query deduplication.
- It solves a real, specific problem that general accounting tools handle badly (apportioning schedule A and B costs by share percentage).

**Gaps and risks**
- Setup routes and credential handling need tidying (see the private security notes).
- Uploads are stored on Hostinger's local disk. A redeploy or migration can lose them, and there's no backup. Consider S3/R2 or Neon plus a storage bucket.
- It depends on NextAuth v5 **beta**.
- UK service-charge law (LTA 1985 s.21B) requires the "summary of rights and obligations" to go with demands. Check that invoice PDFs include it, or invoices may not be legally payable.

**Market context:** Block-management software (for example Qube, PropertyFile, Fixflo for repairs) is aimed at managing agents with hundreds of units and priced to match. For a single freeholder, a custom portal is justified. With some multi-building work it could also be sold to other small freeholders.

**Next moves:** act on the private security notes, move uploads to object storage, add the s.21B notice to the PDFs, and schedule a nightly database export.

---

### 4. BATEMAN / Trendzation Command Centre
**What it does:** A public marketing site for Trendzation (UK web design, targeting Swindon SEO) with lead capture, plus a private dashboard with a pipeline, clients, analytics, settings and an AI chat. The chat goes through a Cloudflare Tunnel to Ollama/Qwen3 running on a home Alienware PC.

**Strengths**
- Security basics are done properly: bcrypt, CSRF middleware, Helmet, rate limiting and SQLite-backed sessions.
- The self-hosted LLM costs nothing per token and keeps client data on your own hardware.
- Local SEO is in place (JSON-LD, sitemap, robots).

**Gaps and risks**
- The AI chat only works while the home PC is on and the tunnel is up. A quick `trycloudflare.com` URL changes on every restart, so use a named tunnel.
- The testimonials section was removed (it was placeholders). The site needs real case studies, and the SCG sites above could be those.
- The README tells people to clone `YOUR_USERNAME/trendzation`, which is out of date.

**Market context:** Swindon has plenty of freelance web designers. The strongest pitch here is a portfolio of real local-business portals (tuition centre, property management), not a generic brochure site.

**Next moves:** add the SCG projects as case studies, set up a named Cloudflare tunnel with a health check, and show an "AI offline" state in the dashboard.

---

### 5. Adi.Study: A-level revision dashboard
**What it does:** A personal PWA that tracks revision across Maths, Further Maths, Physics and ESAT/TMUA. It has subtopic-level tracking, XP, streaks with freezes, achievements, confidence-based review, a past-paper tracker with grade boundaries, per-question weak-area tagging, weekly goals (planned vs actual), a mock-exam timer, an ESAT drill, a formula sheet, a Cmd-K palette, and tutor/student roles linked by invite codes.

**Strengths**
- The most thought-through product design on the wall: spaced-repetition-style review and weak areas surfaced from real paper marks.
- Good commit hygiene. XP-farming exploits and a signup race condition were found and fixed.

**Gaps and risks**
- `app.js` is ~2,900 lines in one file.
- The tutor role reads student data, so it needs Firestore rules that restrict each tutor to their own students.
- It overlaps with trackeralvl (project 6), and there are two empty repos for Physics and Further Maths. The same idea now exists in three places.

**Market context:** Save My Exams, Physics & Maths Tutor, Seneca and Anki cover parts of this. None of them combine spec checklists, past-paper mark tracking and spaced review in one place, which is where this project is strongest. Oxbridge/ESAT/TMUA applicants are a small but motivated niche.

**Next moves:** merge trackeralvl's accurate AQA 7356 spec lists into Adi.Study and archive the duplicates (or use the empty Physics / Further Maths repos for content). Audit the tutor-access rules.

---

### 6. trackeralvl: AQA spec checklist
**What it does:** A React checklist of AS/A-level Maths topics, tied exactly to AQA 7356, with progress stored in Supabase.

**Strengths:** carefully checked against the spec (several commits fixing content). It's a clean single component.

**Gaps:** `node_modules` and `dist/` are committed. The Supabase anon key is in source, which is fine only if row-level security is on; check that it is. It also duplicates Adi.Study.

**Next moves:** fold it into Adi.Study, or keep it as a small public "AQA checklist" tool that brings people to Adi.Study.

---

### 7. 11+ Vocab Weekly
**What it does:** Subscribers sign up and get a 10-word pack (definition, example, synonyms, antonyms) every Monday. The admin uploads a CSV, previews packs and triggers sends. It handles unsubscribe links.

**Gaps and risks:** it was never deployed. APScheduler runs inside the web process, so free hosts that sleep (Render free tier) will miss Mondays. Gmail SMTP limits (~500/day) and deliverability will cap growth. Admin login should use a proper session.

**Market context:** this is a natural lead magnet for SCG Masterminds / SCG Tuitions. Free weekly 11+ vocab emails are a common funnel for 11+ tutors.

**Next moves:** fold it into the Masterminds Apps Script / Google Sheet setup (a time-driven Apps Script trigger plus MailApp needs no server at all), and link sign-ups to the Masterminds paywall.

---

### 8. Insta Reels Blocker (Android)
**What it does:** An AccessibilityService that watches Instagram's view tree. When the Reels tab is selected, it presses Back and shows a "Reels blocked 🚫" toast.

**Gaps and risks**
- Matching on a text or content-description label of "reel" is fragile. It breaks with UI changes and non-English locales, and it misses Reels opened from the feed, DMs or Explore.
- Google Play restricts AccessibilityService use to apps whose core purpose is accessibility or that have a clear declared use. It needs a prominent disclosure and may still be rejected, so sideloading or F-Droid is more realistic.

**Market context:** existing apps include one sec, Opal, ScreenZen, "Reels blocker" style apps and DF Tube (for YouTube). Most detect by view ID and also block the full-screen Reels viewer.

**Next moves:** also detect the Reels viewer by resource ID (`clips_viewer`-style IDs), add a daily allowance instead of a hard block, and add a disclosure screen.

---

### 9. VORTEX agency website
**What it does:** A polished one-page agency template (services, work, process, testimonials, pricing, CTA) in a single 1,074-line HTML file.

**Next moves:** reuse it as the Trendzation landing page or as a sellable template (Gumroad or ThemeForest-style). Replace the placeholder testimonials first.

---

### 10–11. Physics-A-level and Furthermath-A-level
Both private repos are **completely empty** (no commits). Either delete them, or use them as the content repos (spec lists, formula sheets, past-paper indexes) that Adi.Study loads.

---

## Suggested priority order

1. 🔴 Work through the private security notes (live sites first).
2. 🟠 Masterminds: Stripe plus a server-side subscription tier (turns it into revenue).
3. 🟠 trackeralvl: stop committing `node_modules`.
4. 🟠 Westcote: move uploads off local disk, add the s.21B notice.
5. 🟡 Consolidate Adi.Study, trackeralvl and the two empty repos.
6. 🟡 Turn Vocab Weekly into the Masterminds lead magnet.
7. 🟢 Trendzation: SCG case studies, named tunnel.
8. 🟢 Reels Blocker: resource-ID detection, disclosure screen.
