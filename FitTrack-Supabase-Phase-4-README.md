# FitTrack — Supabase Phase 4 Handoff README

## Purpose

FitTrack is a lightweight personal fitness tracker focused on:
- Weight tracking
- Intermittent fasting

Current app:
- Single `index.html`
- GitHub Pages hosted at `https://iamprashanthks.github.io/fittrack/`
- Browser `localStorage` persistence
- JSON export/import backup
- Existing UI is approved and MUST NOT be redesigned, rewritten, or migrated to another framework.

Phase 4 objective:
> Create the secure Supabase PostgreSQL schema required for authenticated FitTrack users. Do not implement frontend authentication/sync in this phase.

## Non-negotiable rules

- Preserve the existing UI exactly.
- Do not split or rewrite the app unless technically unavoidable.
- Do not add unrelated product features.
- Do not disable RLS.
- Do not expose a Supabase service-role key in browser code.
- Keep localStorage and JSON backup intact.
- Use `auth.users.id` as the user identity/ownership anchor.

## Current data model

### Profile/settings
The UI contains:
- Name
- Age
- Height
- Current weight
- Target weight
- BMI
- Body fat
- Custom fasting goal

Store:
- name
- age
- height_cm
- target_weight_kg
- body_fat_percent
- fasting_goal_hours

Do NOT store BMI permanently; calculate it from latest weight and height.

Do NOT store current weight as a second source of truth; latest `weight_entries` record is current weight.

### Weight tracking
Users can record weight multiple times per day. Therefore there must be no uniqueness rule based on calendar date.

### Fasting
A session has start time, optional end time, and target duration. `ended_at IS NULL` means the session is active.

## Required schema

Run the following SQL in Supabase SQL Editor.

```sql
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  name text,
  age integer,
  height_cm numeric(5,2),
  target_weight_kg numeric(5,2),
  body_fat_percent numeric(5,2),
  fasting_goal_hours numeric(5,2),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),

  constraint profiles_age_check
    check (age is null or (age >= 1 and age <= 120)),
  constraint profiles_height_check
    check (height_cm is null or (height_cm > 0 and height_cm <= 300)),
  constraint profiles_target_weight_check
    check (target_weight_kg is null or (target_weight_kg > 0 and target_weight_kg <= 500)),
  constraint profiles_body_fat_check
    check (body_fat_percent is null or (body_fat_percent >= 0 and body_fat_percent <= 100)),
  constraint profiles_fasting_goal_check
    check (fasting_goal_hours is null or (fasting_goal_hours > 0 and fasting_goal_hours <= 168))
);

create table public.weight_entries (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  weight_kg numeric(5,2) not null,
  recorded_at timestamptz not null default now(),
  created_at timestamptz not null default now(),

  constraint weight_entries_weight_check
    check (weight_kg > 0 and weight_kg <= 500)
);

create table public.fasting_sessions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  started_at timestamptz not null,
  ended_at timestamptz,
  target_hours numeric(5,2),
  created_at timestamptz not null default now(),

  constraint fasting_target_hours_check
    check (target_hours is null or (target_hours > 0 and target_hours <= 168)),
  constraint fasting_end_after_start_check
    check (ended_at is null or ended_at >= started_at)
);
```

## Required indexes

```sql
create index weight_entries_user_recorded_at_idx
on public.weight_entries (user_id, recorded_at desc);

create index fasting_sessions_user_started_at_idx
on public.fasting_sessions (user_id, started_at desc);

create index fasting_sessions_active_idx
on public.fasting_sessions (user_id, started_at desc)
where ended_at is null;
```

## Required RLS

```sql
alter table public.profiles enable row level security;
alter table public.weight_entries enable row level security;
alter table public.fasting_sessions enable row level security;
```

### Profiles policies

```sql
create policy "Users can view own profile"
on public.profiles for select to authenticated
using (id = auth.uid());

create policy "Users can create own profile"
on public.profiles for insert to authenticated
with check (id = auth.uid());

create policy "Users can update own profile"
on public.profiles for update to authenticated
using (id = auth.uid())
with check (id = auth.uid());

create policy "Users can delete own profile"
on public.profiles for delete to authenticated
using (id = auth.uid());
```

### Weight policies

```sql
create policy "Users can view own weight entries"
on public.weight_entries for select to authenticated
using (user_id = auth.uid());

create policy "Users can create own weight entries"
on public.weight_entries for insert to authenticated
with check (user_id = auth.uid());

create policy "Users can update own weight entries"
on public.weight_entries for update to authenticated
using (user_id = auth.uid())
with check (user_id = auth.uid());

create policy "Users can delete own weight entries"
on public.weight_entries for delete to authenticated
using (user_id = auth.uid());
```

### Fasting policies

```sql
create policy "Users can view own fasting sessions"
on public.fasting_sessions for select to authenticated
using (user_id = auth.uid());

create policy "Users can create own fasting sessions"
on public.fasting_sessions for insert to authenticated
with check (user_id = auth.uid());

create policy "Users can update own fasting sessions"
on public.fasting_sessions for update to authenticated
using (user_id = auth.uid())
with check (user_id = auth.uid());

create policy "Users can delete own fasting sessions"
on public.fasting_sessions for delete to authenticated
using (user_id = auth.uid());
```

## Updated-at trigger

```sql
create or replace function public.set_updated_at()
returns trigger
language plpgsql
as $$
begin
  new.updated_at = now();
  return new;
end;
$$;

create trigger profiles_set_updated_at
before update on public.profiles
for each row
execute function public.set_updated_at();
```

## Data ownership

Every application record must resolve to the authenticated user:

- `profiles.id = auth.users.id`
- `weight_entries.user_id = auth.users.id`
- `fasting_sessions.user_id = auth.users.id`

The client must never be trusted to access another user's data. RLS is the security boundary.

## Time handling

Use `timestamptz` for:
- recorded_at
- started_at
- ended_at
- created_at
- updated_at

Store timestamps consistently; display them in the user's local timezone.

## BMI calculation

BMI is derived:

`BMI = weight_kg / (height_m × height_m)`

Use the latest weight entry and profile height.

## Current weight

Latest record is the source of truth:

```sql
select weight_kg, recorded_at
from public.weight_entries
where user_id = auth.uid()
order by recorded_at desc
limit 1;
```

## Active fasting session

```sql
select *
from public.fasting_sessions
where user_id = auth.uid()
  and ended_at is null
order by started_at desc
limit 1;
```

## Future migration compatibility

Do NOT delete localStorage data in Phase 4.

Future flow:

`localStorage → Google login → compare cloud data → migrate/sync → cloud becomes primary`

JSON export/import must remain available as a manual backup.

## Phase 4 scope

### Create
- `profiles`
- `weight_entries`
- `fasting_sessions`
- Foreign keys
- Constraints
- Indexes
- RLS
- CRUD policies
- updated_at trigger

### Do NOT create
- Payments/subscriptions
- Admin tables
- Social features
- Friends/followers
- Leaderboards
- Achievements
- Nutrition tables
- Workout tables
- AI coaching tables
- Analytics tables
- Notification systems

## Acceptance tests

1. One profile per authenticated user.
2. User A cannot read User B's profile.
3. User A cannot read User B's weight entries.
4. Multiple weight entries on the same day are allowed.
5. User A cannot read User B's fasting sessions.
6. Active fasting works with `ended_at IS NULL`.
7. Invalid weight values are rejected.
8. Invalid profile values are rejected.
9. Event timestamps use `timestamptz`.
10. RLS remains enabled on all three tables.

## AntiGravity execution instruction

Read this document as the authoritative Phase 4 database specification.

Before changing anything:
1. Inspect the existing FitTrack project.
2. Do not redesign the UI.
3. Do not rewrite the application.
4. Do not migrate frameworks.
5. Create/verify only the Supabase schema described here.
6. Verify RLS and every policy.
7. Verify constraints, foreign keys, and indexes.
8. Report exactly what was created.
9. Stop after Phase 4.

Do NOT implement Google OAuth, frontend Supabase client code, local/cloud synchronization, or UI changes until Phase 5 is explicitly requested.

**Phase 4 success condition:** the Supabase database is secure and ready to accept authenticated FitTrack profile, weight-history, and fasting-history records.
