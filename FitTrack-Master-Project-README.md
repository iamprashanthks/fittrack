# FitTrack — Product, Scope & Implementation Master README

> **FitTrack** is a lightweight personal fitness tracker for weight progress and intermittent fasting.
>
> This document is the **single project handoff specification** for planning, implementation, AI builders, future development, and product decisions.

---

## 1. Product Definition

### Product name

**FitTrack**

### Product type

Lightweight personal fitness / health tracking web app.

### Core promise

> **Track your weight, monitor your progress, and manage intermittent fasting in one simple place.**

### Core problem

People who are trying to lose or manage weight often need to track two recurring behaviors:

1. Their weight over time.
2. Their fasting sessions.

Many fitness products add unnecessary complexity around workouts, nutrition, social features, subscriptions, and coaching.

FitTrack deliberately starts narrower.

### Core product philosophy

**Utility first.**

FitTrack should make the user's daily tracking:

- fast
- simple
- understandable
- private
- persistent
- mobile-friendly

The product should not become a generic health platform unless future evidence justifies expansion.

---

# 2. Product Strategy

The project follows this sequence:

```text
Discovery
    ↓
Evaluation
    ↓
Action
    ↓
Success
    ↓
Retention
```

### Discovery

User discovers FitTrack through:

- search
- recommendations
- social posts
- developer/project communities
- direct sharing

### Evaluation

User immediately understands:

- what FitTrack does
- who it is for
- whether it is free
- how data is handled
- why it is simpler than larger fitness apps

### Action

User:

1. opens FitTrack
2. creates/signs into an account
3. enters profile information
4. records weight
5. starts a fasting session

### Success

The user can quickly answer:

- What is my current weight?
- What is my target?
- Am I progressing?
- How long have I been fasting?
- What does my recent trend look like?

### Retention

The user returns because recording a measurement or fasting session takes very little effort and their historical progress remains available.

---

# 3. Target User

## Primary user

A person who wants a simple tool for:

- weight-loss tracking
- weight-management tracking
- intermittent fasting
- viewing progress over time

## Secondary users

Potential future audiences:

- people starting a weight-loss journey
- intermittent fasting users
- people who dislike complicated fitness apps
- users who want a lightweight alternative to large fitness platforms

Do not broaden the product based on assumptions. Validate new audiences before building dedicated features.

---

# 4. MVP Definition

## The strict MVP

FitTrack's core MVP consists of:

### Weight

- Add weight measurements.
- Allow multiple measurements per day.
- Store measurement timestamp.
- Display weight history.
- Display a progress chart.
- Filter chart/history:
  - Day
  - Week
  - Month
  - 3 Months
  - 6 Months
  - 1 Year
- Set target weight.

### Fasting

- Start a fasting session.
- Stop/complete a fasting session.
- Configure fasting duration/goal.
- Display active fasting timer.
- Store fasting history.

### Profile

- Name
- Age
- Height
- Target weight
- Body fat
- Fasting goal

### Calculations

- Current weight = latest weight entry.
- BMI = derived from latest weight + height.
- BMI category can be displayed.
- Body fat is user-provided if available.

### Persistence

V1:

- Browser localStorage.
- JSON export.
- JSON import.

V2:

- Supabase Auth.
- Google authentication.
- Supabase PostgreSQL.
- Cloud synchronization.

### Deployment

- GitHub repository.
- GitHub Pages.
- Static frontend.

---

# 5. Explicitly OUT OF MVP

Do not add these unless future validation supports them:

- calorie tracking
- meal tracking
- food database
- workout tracking
- exercise library
- step tracking integrations
- Apple Health integration
- Google Fit integration
- wearable integrations
- social feed
- friends/followers
- leaderboards
- achievements
- challenges
- AI coach
- personalized medical advice
- subscriptions
- payments
- advertising
- marketplace
- community
- admin dashboard
- complex analytics
- unnecessary notifications

These are potential future ideas, not MVP requirements.

---

# 6. Current Product State

## V1 — Completed foundation

Current application:

```text
FitTrack
├── Weight tracking
├── Weight chart/history
├── Target weight
├── Fasting timer
├── Fasting history
├── Profile/settings
├── BMI calculation
├── Body-fat display/calculation support
├── localStorage persistence
├── JSON export
├── JSON import
├── Clear/reset data
└── Responsive UI
```

The current interface is based on an approved Meta AI prototype.

### UI rule

**The existing UI is frozen.**

Do not redesign it during backend integration.

Do not:

- change the visual style
- replace the layout
- change navigation
- change colors
- change typography
- introduce a new dashboard
- migrate to another framework simply to add Supabase
- split the single HTML file unless there is a demonstrated technical reason

Backend work must happen behind the approved UI.

---

# 7. Important Current Input Fixes

The following frontend behavior has already been corrected:

### Age

The input can be cleared while editing.

It must not force:

```text
empty → 0
```

and then produce:

```text
035
```

### Height

Same behavior.

### Target weight

Same behavior.

The target weight field must allow:

```text
55
```

rather than:

```text
055
```

and clearing the field must not crash the application.

### General rule

Numeric inputs must support normal editing.

Do not force a numeric default into the field merely because the temporary input value is empty.

---

# 8. Current V1 Architecture

Current architecture is intentionally simple:

```text
Static HTML
    ↓
Bundled React application
    ↓
Browser
    ├── localStorage
    └── JSON export/import
```

The current app is intentionally deployed as a static website.

No backend is required for V1.

---

# 9. V2 Architecture

The planned cloud architecture is:

```text
                    Google
                      │
                      ▼
              Supabase Auth
                      │
                      ▼
                authenticated
                    user
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      profiles   weight_entries   fasting_sessions
          │           │           │
          └───────────┼───────────┘
                      ▼
               FitTrack frontend
                      │
              local cache/fallback
                      │
                  localStorage
```

### Source of truth

After successful migration:

**Supabase = primary cloud source of truth**

**localStorage = local cache/fallback**

**JSON = portable manual backup**

---

# 10. Supabase Authentication

## Authentication method

Primary authentication:

**Google OAuth**

Supabase Auth handles:

- authentication
- sessions
- identity
- OAuth provider integration

FitTrack should not build its own password authentication system for the MVP.

## Login flow

```text
User opens FitTrack
       ↓
Not authenticated
       ↓
Google Sign In
       ↓
Google OAuth
       ↓
Supabase Auth
       ↓
Authenticated session
       ↓
Load profile/data
       ↓
FitTrack dashboard
```

## Logout

```text
User clicks logout
       ↓
Supabase session ends
       ↓
Clear authenticated cloud state
       ↓
Return to unauthenticated state
```

---

# 11. Supabase Database

The MVP cloud database contains three application tables.

```text
auth.users
    │
    ├── profiles
    │
    ├── weight_entries
    │
    └── fasting_sessions
```

---

# 12. `profiles` Table

## Purpose

Stores user profile/settings that are not already managed by Supabase Auth.

Recommended fields:

```text
id
name
age
height_cm
target_weight_kg
body_fat_percent
fasting_goal_hours
created_at
updated_at
```

### Ownership

```text
profiles.id = auth.users.id
```

One profile per authenticated user.

### Validation

Recommended constraints:

- age: 1–120
- height: >0 and <=300 cm
- target weight: >0 and <=500 kg
- body fat: 0–100%
- fasting goal: >0 and <=168 hours

---

# 13. `weight_entries` Table

## Purpose

Stores every individual weight measurement.

Fields:

```text
id
user_id
weight_kg
recorded_at
created_at
```

### Important

Multiple records on the same day are valid.

Example:

```text
68.4 kg — morning
68.1 kg — afternoon
67.9 kg — evening
```

Therefore there must NOT be a unique constraint such as:

```text
user_id + date
```

### Ownership

```text
weight_entries.user_id = auth.users.id
```

---

# 14. `fasting_sessions` Table

Fields:

```text
id
user_id
started_at
ended_at
target_hours
created_at
```

### Active session

```text
ended_at = NULL
```

means the session is still active.

### Ownership

```text
fasting_sessions.user_id = auth.users.id
```

---

# 15. RLS Security

Row Level Security is mandatory.

Every application table must have RLS enabled.

The core rule is:

```text
User can access ONLY their own data.
```

### Profiles

```text
id = auth.uid()
```

### Weight

```text
user_id = auth.uid()
```

### Fasting

```text
user_id = auth.uid()
```

Never solve an RLS error by disabling RLS.

Never expose a service-role key in frontend code.

---

# 16. Data Model Principles

## Current weight

Do not maintain an independent current-weight database field unless a future requirement proves it necessary.

Current weight should be derived from:

```text
latest weight_entries record
```

## BMI

Do not permanently store BMI.

Calculate:

```text
BMI = weight_kg / (height_m × height_m)
```

using:

- latest weight
- profile height

This prevents stale calculations.

## Timestamps

Use:

```text
timestamptz
```

for all event timestamps.

Store timestamps consistently and display them using the user's local timezone.

---

# 17. Local Data Migration

Existing V1 users may already have localStorage data.

Migration must be safe.

Desired flow:

```text
Existing localStorage
        ↓
User signs in with Google
        ↓
Load cloud data
        ↓
Detect whether cloud data exists
        ↓
Compare local + cloud data
        ↓
Migrate/merge safely
        ↓
Cloud becomes primary
        ↓
Keep local cache
```

Do not silently delete local data.

Do not overwrite existing cloud records blindly.

---

# 18. JSON Backup

JSON export/import remains a permanent safety mechanism.

Export should contain enough information to restore:

```text
profile
weights
fasting sessions
relevant settings
export timestamp
```

JSON backup is:

```text
Portable backup
```

It is NOT the primary synchronization mechanism.

---

# 19. Sync Strategy

The first cloud-sync implementation should be conservative.

### Login

```text
Authenticate
    ↓
Fetch profile
    ↓
Fetch weight entries
    ↓
Fetch fasting sessions
    ↓
Update local cache
```

### New weight

```text
User logs weight
      ↓
Update UI immediately
      ↓
Save locally
      ↓
Save to Supabase
```

### New fasting session

```text
Start
  ↓
Update local active session
  ↓
Persist cloud session
```

### Complete fasting session

```text
Stop
  ↓
Update session
  ↓
Set ended_at
  ↓
Update Supabase
  ↓
Update local cache
```

The UI should not feel slow because of network requests.

---

# 20. Offline / Network Failure Behavior

FitTrack should fail gracefully.

If cloud storage is temporarily unavailable:

```text
User action
   ↓
Save locally
   ↓
Show calm status
   ↓
Retry cloud synchronization later
```

Do not discard a user's measurement simply because the network failed.

Future sync can reconcile unsynced local records.

---

# 21. Frontend Integration Rules

When connecting Supabase to the existing app:

### Allowed

- Add Supabase client
- Add authentication logic
- Add session handling
- Add database CRUD
- Add sync logic
- Add minimal login/logout UI required by the existing design
- Add status/error messaging where necessary

### Not allowed

- Redesign dashboard
- Replace visual language
- Introduce a new navigation system
- Rewrite the application for Next.js
- Replace the existing chart unnecessarily
- Remove JSON backup
- Remove localStorage without migration
- Add unrelated features

---

# 22. Supabase Client Security

Frontend code may use the Supabase publishable/anon key intended for browser applications.

Never expose:

```text
service_role key
```

in:

- HTML
- JavaScript
- GitHub
- browser storage
- public repositories

Use RLS as the security boundary.

---

# 23. Technical Stack

## Current

```text
Frontend:
Single HTML file / bundled React application

Storage:
Browser localStorage

Backup:
JSON export/import

Hosting:
GitHub Pages

Source control:
Git + GitHub
```

## V2

```text
Frontend:
Existing FitTrack frontend

Authentication:
Supabase Auth

OAuth:
Google

Database:
Supabase PostgreSQL

Security:
Supabase RLS

Cloud:
Supabase

Hosting:
GitHub Pages
```

### Architecture principle

Do not introduce a heavier framework unless the current architecture becomes a proven limitation.

---

# 24. Repository

GitHub repository:

```text
https://github.com/iamprashanthks/fittrack
```

Main branch:

```text
main
```

Production URL:

```text
https://iamprashanthks.github.io/fittrack/
```

---

# 25. Git Workflow

For normal frontend changes:

```bash
git status
git add index.html
git commit -m "Describe the change"
git push
```

Do not commit personal JSON backups.

Recommended future `.gitignore`:

```text
*.json
```

or a more specific rule if JSON files are needed for development.

Do not blindly use:

```bash
git add .
```

if private data exists in the project directory.

---

# 26. Development Workflow

Every feature must follow:

```text
1. Define problem
2. Confirm MVP necessity
3. Define user flow
4. Define data requirement
5. Implement
6. Test locally
7. Test mobile
8. Test production
9. Commit
10. Deploy
```

Do not jump directly into coding.

---

# 27. UX Principles

The uploaded UX research is treated as a universal product framework.

FitTrack should prioritize:

- clarity
- low cognitive load
- short paths to value
- progressive disclosure
- accessible forms
- clear validation
- useful empty states
- graceful errors
- responsive layouts
- touch-friendly controls
- readable charts
- visible focus states
- keyboard usability
- reduced-motion support where motion exists

The UX research emphasizes structured onboarding, accessible controls, error handling, responsive breakpoints, touch target sizing, readable typography, and accessible data visualization. Apply these principles without changing the approved visual identity.

---

# 28. Form UX Rules

Numeric fields must behave like normal form controls.

For example:

```text
User focuses field
       ↓
Existing value visible
       ↓
User presses Backspace
       ↓
Field may temporarily become empty
       ↓
User types new value
       ↓
Value updates normally
```

Do not force:

```text
empty → 0
```

while the user is editing.

Validate on:

- meaningful input
- blur
- submit

depending on the field.

---

# 29. Error Handling

Errors should be:

- calm
- precise
- actionable
- non-destructive

Bad:

```text
ERROR!!!
```

Better:

```text
Unable to sync your data. Your latest entry is saved locally and will be retried.
```

The application should avoid blank screens and uncaught runtime failures.

If a cloud request fails, preserve the user's local action whenever possible.

---

# 30. Accessibility

Target:

**WCAG 2.1 AA principles**

Minimum expectations:

- sufficient contrast
- keyboard navigation
- visible focus
- usable form labels
- accessible chart information
- appropriate touch targets
- semantic controls
- no critical information conveyed only by color
- reduced-motion support where relevant
- usable at increased zoom

Accessibility must not be treated as a post-launch cleanup task.

---

# 31. Responsive Design

The existing UI is mobile-first.

Test at minimum:

```text
320px
375px
390px
430px
768px
1024px+
```

The application should work on:

- mobile
- tablet
- desktop

Do not introduce responsive redesigns during backend integration.

---

# 32. Performance

FitTrack is intentionally lightweight.

Priorities:

1. Fast initial load.
2. Minimal JavaScript overhead.
3. No unnecessary dependencies.
4. Efficient database queries.
5. Indexed user-owned records.
6. No repeated full-database downloads.
7. Avoid unnecessary re-renders.
8. Keep chart data scoped to the selected time period where practical.

Target good Core Web Vitals and fast mobile loading.

Do not add performance tooling merely for the sake of tooling; measure first, then optimize actual bottlenecks.

---

# 33. Database Performance

Required indexes:

```text
weight_entries(user_id, recorded_at DESC)

fasting_sessions(user_id, started_at DESC)

fasting_sessions(user_id, started_at DESC)
WHERE ended_at IS NULL
```

Queries should always be scoped to the authenticated user.

Avoid loading unlimited historical records if the UI only needs a defined date range.

---

# 34. Testing Strategy

## Manual testing

Before every release, test:

### Authentication

- Google sign in
- successful redirect
- session persistence
- logout
- returning user

### Profile

- name
- age
- height
- target weight
- body fat
- fasting goal

### Weight

- add weight
- multiple weights in one day
- edit if supported
- delete if supported
- chart filters
- current weight calculation
- target comparison

### Fasting

- start
- active timer
- stop
- history
- active session persistence
- page refresh during active fast

### Data

- localStorage persistence
- JSON export
- JSON import
- cloud save
- cloud reload
- logout/login
- cross-device data retrieval

### Failure cases

- network unavailable
- invalid form value
- empty form
- duplicate-looking weight records
- interrupted OAuth
- expired session
- failed cloud request

---

# 35. Security Testing

Verify:

```text
User A → only User A data
User B → only User B data
```

Test RLS directly.

Attempt unauthorized access to:

- profiles
- weight_entries
- fasting_sessions

Expected result:

```text
No cross-user access
```

Never rely solely on frontend filtering for security.

---

# 36. Analytics — Later

Analytics should not be introduced before the core product works reliably.

If analytics are later added, define a small event taxonomy such as:

```text
app_opened
google_login
profile_completed
weight_logged
fast_started
fast_completed
json_exported
json_imported
```

Collect only what is necessary.

Do not collect sensitive health data unnecessarily.

---

# 37. Privacy

FitTrack deals with personal health-related tracking data.

The product should be transparent about:

- what data is stored
- where it is stored
- authentication provider
- cloud synchronization
- JSON backup
- deletion behavior

Do not make exaggerated privacy claims.

Future production launch should include appropriate:

- Privacy Policy
- Terms
- data deletion guidance
- support/contact mechanism

Legal requirements should be validated for the intended launch jurisdictions before commercial operation.

---

# 38. SEO Strategy

SEO should support discovery without damaging product clarity.

The uploaded SEO research is treated as a universal framework.

Core principles:

- search intent first
- entity building
- topical relevance
- natural semantic language
- no keyword stuffing
- useful content
- clear H1/H2/H3 hierarchy
- strong internal linking when multiple pages exist
- trustworthy claims
- transparent limitations
- answer-first content
- AI-citable content structure

The research specifically emphasizes matching search intent, natural keyword usage, clear hierarchy, internal linking, authoritative references, transparency, and AEO-friendly answer structures.

---

# 39. Landing Page Strategy

When a public marketing landing page is built, the structure should follow:

```text
Hero
  ↓
What FitTrack does
  ↓
Core benefits
  ↓
Product screenshots/demo
  ↓
Weight tracking
  ↓
Fasting tracking
  ↓
Privacy/data model
  ↓
How it works
  ↓
FAQ
  ↓
CTA
```

The first part of the page must immediately answer:

- What is FitTrack?
- Who is it for?
- What can it track?
- Is it free?
- Where is data stored?

Do not fill the page with generic fitness content.

---

# 40. SEO Entity Strategy

Maintain consistent information for:

```text
FitTrack
```

Across:

- website
- GitHub
- social profiles
- launch posts
- documentation
- directory listings

The name, description, product purpose, and links should remain consistent.

Recognition comes before reputation, and reputation comes before revenue.

---

# 41. Content Strategy

Potential content themes:

### Weight tracking

- how to track weight consistently
- weight trend vs single measurement
- how frequently to weigh yourself
- understanding weight fluctuations

### Intermittent fasting

- what a fasting timer does
- how to track fasting sessions
- common fasting schedules
- how to interpret fasting duration

### Product education

- how FitTrack stores data
- localStorage vs cloud sync
- how Google login works
- how to export FitTrack data
- how to migrate local data to cloud

Content must be useful first.

Do not publish hundreds of thin keyword pages.

---

# 42. AEO / AI Search Strategy

Public content should be structured so an answer engine can understand it easily.

Use:

```text
Question
Short direct answer
Supporting explanation
Evidence/reference where appropriate
```

Example:

```text
What is FitTrack?

FitTrack is a lightweight fitness tracker for recording weight progress and intermittent fasting sessions.
```

Do not manipulate AI systems with artificial repetition.

---

# 43. Trust Strategy

Trust signals should include:

- clear product purpose
- transparent data model
- clear privacy explanation
- GitHub source visibility
- accurate feature descriptions
- honest limitations
- reliable error handling
- clear support/contact route
- no exaggerated health claims

Do not position FitTrack as a medical device unless that claim is actually justified and legally validated.

---

# 44. Monetization Strategy

## Current position

**No premature monetization.**

First establish:

```text
Utility
   ↓
Recognition
   ↓
Reputation
   ↓
Revenue
```

The initial goal is product validation and usage.

Potential future models can be evaluated only after actual usage data exists.

Possible future options:

- free core
- optional premium cloud features
- premium analytics
- family/multi-user plan
- paid advanced features

Do not build billing infrastructure now.

---

# 45. Product Metrics

The most useful early metrics are behavioral.

### Activation

Percentage of new users who:

```text
Sign in
   ↓
Complete profile
   ↓
Log first weight
```

### Engagement

- weight logs per active user
- fasting sessions per active user
- returning users
- weekly active users

### Retention

- 7-day return
- 30-day return

### Reliability

- failed syncs
- authentication failures
- data migration failures
- JavaScript errors

Do not optimize vanity metrics before the product is useful.

---

# 46. Roadmap

## Phase 0 — Product definition

Status: **Complete**

- product concept
- target user
- strict MVP
- UI direction
- local-first architecture

---

## Phase 1 — Frontend MVP

Status: **Complete**

- weight tracking
- fasting timer
- profile
- BMI
- target weight
- charts
- localStorage
- JSON backup
- responsive UI

---

## Phase 2 — Frontend bug fixing

Status: **Complete**

- Age input fix
- Height input fix
- Target KG input fix
- Clear-data behavior
- empty first-user state

---

## Phase 3 — Deployment

Status: **Complete**

```text
GitHub
  ↓
GitHub Pages
  ↓
https://iamprashanthks.github.io/fittrack/
```

---

## Phase 4 — Supabase database

Status: **Current**

Implement:

- Supabase project
- profiles table
- weight_entries table
- fasting_sessions table
- foreign keys
- constraints
- indexes
- RLS
- CRUD policies
- updated_at trigger

Stop after schema/security verification.

---

## Phase 5 — Google Authentication

Next.

Implement:

- Supabase client
- Google OAuth
- login
- logout
- session restoration
- auth state handling
- authenticated UI state

Do not redesign the application.

---

## Phase 6 — Cloud Sync

After authentication.

Implement:

- profile sync
- weight sync
- fasting sync
- active fasting synchronization
- local cache
- retry behavior
- migration from existing localStorage

---

## Phase 7 — Migration & Reliability

Implement:

- first-login migration
- conflict handling
- sync status
- error recovery
- data integrity tests
- cross-device testing

---

## Phase 8 — Public product layer

Only after core product stability:

- landing page
- privacy policy
- terms
- help/documentation
- SEO foundation
- Search Console
- analytics if justified
- public launch

---

## Phase 9 — Validation

Measure actual usage.

Do not expand the product based on imagination.

Use:

```text
User behavior
   ↓
Feedback
   ↓
Pain point
   ↓
Validated feature
   ↓
Small implementation
   ↓
Measure
```

---

# 47. Future Feature Gate

Every proposed feature must pass these questions:

### 1. What user problem does it solve?

If no clear problem:

**Reject.**

### 2. Is the problem frequent enough?

If rarely used:

**Defer.**

### 3. Does it strengthen FitTrack's core purpose?

If unrelated:

**Reject.**

### 4. Can the MVP work without it?

If yes:

**Defer unless evidence says otherwise.**

### 5. Does it increase complexity significantly?

If yes:

**Require stronger evidence.**

---

# 48. AI Builder Rules

Any AI coding agent working on FitTrack must follow:

## Before coding

1. Read this README.
2. Inspect the existing code.
3. Identify current behavior.
4. Identify what phase is active.
5. Do not implement later-phase features prematurely.

## During coding

- Preserve UI.
- Preserve existing working functionality.
- Make the smallest safe change.
- Do not refactor unrelated code.
- Do not add dependencies without a reason.
- Do not replace working architecture without evidence.
- Do not invent requirements.

## After coding

1. Test locally.
2. Check console errors.
3. Test affected functionality.
4. Test existing critical functionality.
5. Report files changed.
6. Report behavior changed.
7. Report known limitations.
8. Stop at the requested phase.

---

# 49. Anti-Overengineering Rules

FitTrack is intentionally small.

Do NOT introduce:

- microservices
- unnecessary APIs
- complex state management
- unnecessary backend servers
- custom authentication
- event-driven architecture
- queues
- Kubernetes
- complex CI/CD
- multiple databases
- unnecessary abstraction layers

unless actual scale or requirements make them necessary.

A simple architecture that works is preferable to an impressive architecture that adds maintenance.

---

# 50. Documentation Requirements

Maintain documentation for:

```text
README.md
├── product purpose
├── setup
├── local development
├── environment configuration
├── Supabase configuration
├── Google OAuth configuration
├── database schema
├── RLS model
├── data migration
├── deployment
└── troubleshooting
```

Future technical documentation should explain not only **what** exists, but **why**.

---

# 51. Environment / Secrets

Never commit secrets.

Local development may use environment variables when the architecture supports them.

Production frontend may safely contain the intended public Supabase project URL and publishable client key.

Never expose:

```text
SUPABASE_SERVICE_ROLE_KEY
```

or equivalent privileged credentials.

---

# 52. Backup & Recovery

Current:

```text
JSON export
```

Future:

```text
Supabase cloud data
+
JSON export
```

Users should be able to export their data even after cloud integration.

Future recovery process:

```text
Account
  ↓
Cloud data
  ↓
Export JSON
  ↓
Local backup
```

---

# 53. Data Deletion

Future production implementation must define:

### Delete account

Expected:

```text
Delete authenticated user
       ↓
Delete owned profile
       ↓
Delete owned weight entries
       ↓
Delete owned fasting sessions
```

Foreign keys with:

```text
ON DELETE CASCADE
```

should support this relationship safely.

Do not implement account deletion partially.

---

# 54. Current User Data Flow

```text
                ┌───────────────────┐
                │     FitTrack      │
                └─────────┬─────────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       localStorage                 Supabase
       local cache                 cloud data
             │                         │
             └────────────┬────────────┘
                          │
                       User ID
                          │
                     auth.users
```

---

# 55. Core User Journey

## First visit

```text
Open FitTrack
    ↓
Understand product
    ↓
Sign in with Google
    ↓
Profile setup
    ↓
Set target weight
    ↓
Log first weight
    ↓
Start fasting when appropriate
```

## Returning user

```text
Open FitTrack
    ↓
Session restored
    ↓
Cloud/local data loaded
    ↓
Current progress displayed
    ↓
Log weight or start/continue fast
```

---

# 56. Success Definition

FitTrack is successful when a user can complete this loop with minimal friction:

```text
Open
 ↓
See current status
 ↓
Log weight
 ↓
See trend
 ↓
Start fasting
 ↓
Return later
 ↓
See history
```

The product does not need dozens of features to succeed.

---

# 57. Product Positioning

FitTrack should be positioned as:

> **A simple weight and intermittent fasting tracker.**

Not:

- a complete medical platform
- an AI doctor
- a replacement for healthcare
- a complete nutrition system
- a social fitness network

Clarity is a product advantage.

---

# 58. Final Architecture

The intended stable architecture is:

```text
                    ┌─────────────────────┐
                    │      Google         │
                    │       OAuth         │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Supabase Auth     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   FitTrack Web App  │
                    │   Existing UI       │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
       ┌─────────────────┐         ┌──────────────────┐
       │   localStorage  │         │ Supabase Postgres│
       │  local cache    │         │                  │
       └─────────────────┘         │ profiles         │
                                   │ weight_entries   │
                                   │ fasting_sessions │
                                   └──────────────────┘
```

---

# 59. Current Project Status Checklist

## Product

- [x] Product concept defined
- [x] Target user defined
- [x] MVP defined
- [x] Feature boundaries defined
- [x] User journey defined

## Frontend

- [x] Weight tracking
- [x] Target weight
- [x] Weight chart
- [x] Chart filters
- [x] Fasting timer
- [x] Fasting history
- [x] Profile
- [x] BMI
- [x] Body fat
- [x] localStorage
- [x] JSON export
- [x] JSON import
- [x] Responsive UI
- [x] Age input fix
- [x] Height input fix
- [x] Target KG input fix

## Deployment

- [x] Git repository
- [x] `main` branch
- [x] GitHub Pages
- [x] Production URL

## Cloud

- [ ] Supabase project configuration
- [ ] Database schema
- [ ] RLS verification
- [ ] Google OAuth
- [ ] Supabase client
- [ ] Auth session handling
- [ ] Cloud sync
- [ ] Local data migration
- [ ] Cross-device testing

## Public product

- [ ] Landing page
- [ ] Privacy policy
- [ ] Terms
- [ ] Documentation
- [ ] SEO foundation
- [ ] Search Console
- [ ] Analytics decision
- [ ] Public launch

---

# 60. Immediate Execution Order

The project must proceed in this exact order:

```text
CURRENT
   │
   ▼
Phase 4
Supabase schema + RLS
   │
   ▼
Phase 5
Google authentication
   │
   ▼
Phase 6
Cloud synchronization
   │
   ▼
Phase 7
Migration + reliability testing
   │
   ▼
Phase 8
Public product/SEO layer
   │
   ▼
Phase 9
Real-user validation
   │
   ▼
Only then:
Validated feature expansion
```

---

# 61. Final Product Rule

When in doubt, choose:

> **The simplest implementation that reliably solves the user's actual problem without changing the approved experience.**

FitTrack should remain:

**Simple → Fast → Private → Useful → Reliable**

before becoming:

**Large → Complex → Feature-heavy.**

---

## Reference Frameworks

This project specification adapts the uploaded project-agnostic SaaS, UX, SEO, and development research frameworks to FitTrack.

The framework principles include evidence-first decisions, user empathy, transparency, progressive disclosure, accessibility, performance, security, clear error handling, documentation, and avoiding premature complexity.

The FitTrack implementation intentionally does **not** inherit irrelevant AstroAnimate-specific architecture such as Astro monorepos, animation packages, or component-library release systems.

---

# END OF FITTRACK MASTER README
