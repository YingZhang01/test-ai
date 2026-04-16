---
name: Volleyball Hub Tech Plan
overview: Plan a collegiate club volleyball tournament management platform as a responsive web app using a fully free tech stack (Next.js, Supabase, Tailwind, shadcn/ui), targeting team captains and players as primary users.
todos:
  - id: scaffold
    content: Scaffold Next.js 15 project with TypeScript, Tailwind CSS, shadcn/ui, Drizzle ORM, and Supabase client
    status: completed
  - id: db-schema
    content: Define Drizzle database schema (users, teams, tournaments, divisions, registrations, pools, brackets, matches, sets, courts)
    status: completed
  - id: auth
    content: Set up Supabase Auth with email/password, middleware for protected routes, and role-based access
    status: completed
  - id: layout
    content: Build responsive app layout with navigation, sidebar, and mobile-friendly design
    status: completed
  - id: teams
    content: Implement team creation, roster management, and invite flow
    status: completed
  - id: tournaments
    content: Build tournament CRUD, division configuration, and status lifecycle
    status: completed
  - id: registration
    content: Implement team registration flow with organizer approval
    status: completed
  - id: pools-brackets
    content: Build pool play generation, bracket generation (single/double elim), and interactive display components
    status: completed
  - id: scheduling
    content: Implement game scheduling with court assignments and time slots
    status: completed
  - id: live-scoring
    content: Build real-time scoring interface with Supabase Realtime and auto-updating standings
    status: completed
isProject: false
---

# Collegiate Club Volleyball Hub -- Project Plan

## Tech Stack (All Free Tier)


| Layer | Choice | Why |
| ----- | ------ | --- |


- **Framework**: Next.js 15 (App Router) -- full-stack React framework, SSR/SSG, API routes built in
- **Language**: TypeScript -- type safety across the entire stack
- **Styling**: Tailwind CSS + shadcn/ui -- rapid, beautiful, accessible UI with no cost
- **Database**: Supabase (PostgreSQL) -- free tier gives 500MB DB, 1GB storage, 50K MAU, built-in auth, and realtime subscriptions (useful for live scoring)
- **ORM**: Drizzle ORM -- lightweight, type-safe, SQL-like syntax, pairs well with Supabase Postgres
- **Auth**: Supabase Auth -- free, supports email/password + OAuth (Google), row-level security
- **Realtime**: Supabase Realtime -- free WebSocket-based subscriptions for live score updates
- **Hosting**: Vercel (free hobby tier) -- zero-config Next.js deployment, preview deploys on PRs
- **Validation**: Zod -- runtime schema validation shared between client and server

## Architecture Overview

```mermaid
graph TD
    subgraph client [Client - Browser]
        NextPages[Next.js Pages]
        ReactComponents[React Components]
        SupabaseClient[Supabase JS Client]
    end

    subgraph server [Server - Vercel]
        AppRouter[Next.js App Router]
        ServerActions[Server Actions]
        DrizzleORM[Drizzle ORM]
    end

    subgraph supabase [Supabase - Free Tier]
        PostgresDB[PostgreSQL Database]
        SupaAuth[Auth Service]
        Realtime[Realtime Subscriptions]
    end

    NextPages --> ReactComponents
    ReactComponents --> SupabaseClient
    SupabaseClient -->|"Auth, Realtime"| SupaAuth
    SupabaseClient -->|"Live scores"| Realtime
    ReactComponents --> ServerActions
    ServerActions --> DrizzleORM
    DrizzleORM --> PostgresDB
    Realtime --> PostgresDB
```



## Data Model

```mermaid
erDiagram
    users ||--o{ team_members : "belongs to"
    users ||--o{ tournaments : "organizes"
    teams ||--o{ team_members : "has"
    teams ||--o{ registrations : "registers for"
    tournaments ||--o{ registrations : "receives"
    tournaments ||--o{ divisions : "has"
    divisions ||--o{ pools : "contains"
    divisions ||--o{ brackets : "contains"
    pools ||--o{ pool_teams : "has"
    pools ||--o{ matches : "schedules"
    brackets ||--o{ matches : "schedules"
    matches ||--o{ sets : "has"
    tournaments ||--o{ courts : "uses"

    users {
        uuid id PK
        string email
        string full_name
        string university
        enum role
    }
    teams {
        uuid id PK
        string name
        string university
        string season
    }
    team_members {
        uuid id PK
        uuid team_id FK
        uuid user_id FK
        enum role
        int jersey_number
    }
    tournaments {
        uuid id PK
        uuid organizer_id FK
        string name
        date start_date
        date end_date
        string location
        string address
        enum status
    }
    divisions {
        uuid id PK
        uuid tournament_id FK
        string name
        enum format
        int team_cap
    }
    registrations {
        uuid id PK
        uuid team_id FK
        uuid tournament_id FK
        uuid division_id FK
        enum status
        timestamp registered_at
    }
    pools {
        uuid id PK
        uuid division_id FK
        string name
    }
    pool_teams {
        uuid id PK
        uuid pool_id FK
        uuid team_id FK
    }
    brackets {
        uuid id PK
        uuid division_id FK
        enum bracket_type
        int seed_count
    }
    matches {
        uuid id PK
        uuid pool_id FK
        uuid bracket_id FK
        uuid court_id FK
        uuid team_a_id FK
        uuid team_b_id FK
        int bracket_round
        int bracket_position
        timestamp scheduled_time
        enum status
        uuid winner_id FK
    }
    sets {
        uuid id PK
        uuid match_id FK
        int set_number
        int team_a_score
        int team_b_score
    }
    courts {
        uuid id PK
        uuid tournament_id FK
        string name
    }
```



## Project Structure

```
collegiate-club-volleyball-hub/
  src/
    app/                          # Next.js App Router
      (auth)/                     # Auth route group (login, signup)
      (dashboard)/                # Authenticated layout
        dashboard/                # Home dashboard
        tournaments/              # Tournament CRUD, detail, bracket views
        teams/                    # Team management, roster
        schedule/                 # Game schedule views
      api/                        # API routes (if needed beyond server actions)
    components/
      ui/                         # shadcn/ui primitives
      tournament/                 # Tournament-specific components
      bracket/                    # Bracket/pool display components
      scoring/                    # Live scoring components
      layout/                     # Nav, sidebar, footer
    lib/
      db/
        schema.ts                 # Drizzle schema definitions
        index.ts                  # DB client
        migrations/               # Drizzle migrations
      supabase/
        client.ts                 # Browser Supabase client
        server.ts                 # Server Supabase client
        middleware.ts             # Auth middleware
      validators/                 # Zod schemas
      utils/
        bracket.ts                # Bracket generation logic
        pool.ts                   # Pool play generation + standings calc
        scheduling.ts             # Court/time assignment algorithm
    hooks/                        # Custom React hooks
    types/                        # Shared TypeScript types
  public/                         # Static assets
  drizzle.config.ts               # Drizzle config
  tailwind.config.ts
  next.config.ts
```

## Key Feature Breakdown

### 1. Auth and User Management

- Supabase Auth with email/password signup
- Role system: `player`, `captain`, `organizer`
- Captains can manage their team roster; organizers can create tournaments
- Middleware protects authenticated routes

### 2. Team Management

- Create a team tied to a university
- Invite players by email or shareable link
- Roster management (add/remove members, set jersey numbers)
- Captain role for administrative actions

### 3. Tournament Creation

- Organizers create tournaments with: name, dates, location, description
- Add divisions (e.g., Men's A, Women's B) with format (pool play -> bracket, straight bracket)
- Set team caps per division, registration deadlines
- Tournament status lifecycle: `draft` -> `registration_open` -> `registration_closed` -> `in_progress` -> `completed`

### 4. Team Registration

- Captains browse open tournaments and register their team for a division
- Registration status: `pending` -> `confirmed` -> `checked_in`
- Organizers can approve/reject registrations

### 5. Pool Play and Bracket Generation

- **Pool play**: Algorithm distributes registered teams evenly across pools, avoiding same-university matchups where possible
- **Bracket generation**: Single-elimination or double-elimination brackets seeded from pool standings
- Pool standings calculated by: wins, then point differential, then head-to-head

### 6. Game Scheduling and Court Assignment

- Organizers assign courts to the tournament
- Auto-schedule pool play matches across courts and time slots
- Manual override for adjustments
- Schedule view filterable by team, court, or time

### 7. Live Scoring

- Organizers or designated scorers enter scores set-by-set
- Supabase Realtime pushes score updates to all viewers instantly
- Match status: `upcoming` -> `in_progress` -> `completed`
- Standings auto-update as matches complete

## Implementation Phases

### Phase 1: Foundation (Week 1-2)

- Project scaffolding (Next.js, Tailwind, shadcn/ui, Drizzle, Supabase)
- Database schema and migrations
- Auth flow (signup, login, logout)
- Basic responsive layout (nav, sidebar, mobile hamburger menu)

### Phase 2: Teams and Tournaments (Week 3-4)

- Team CRUD and roster management
- Tournament creation and listing
- Division configuration
- Registration flow

### Phase 3: Brackets and Scheduling (Week 5-6)

- Pool play generation algorithm
- Bracket generation (single/double elimination)
- Schedule builder with court assignments
- Interactive bracket/pool display components

### Phase 4: Live Scoring and Polish (Week 7-8)

- Real-time scoring interface
- Auto-updating standings
- Realtime bracket progression
- Mobile UX polish, loading states, error handling

## Free Tier Limits to Watch

- **Supabase free**: 500MB DB, 2GB bandwidth/month, 50K MAU, 500K edge function invocations -- sufficient for early usage
- **Vercel free**: 100GB bandwidth/month, 6000 build minutes/month, serverless function limits -- fine for a hobby/club project
- No credit card required for either service to start

