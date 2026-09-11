# AI Home Organizer — Development Guide

## 1. Project Goal

This project is a personal AI-powered home organization and cleaning system.

The system allows the user to:

1. Organize the home by Floor → Room → Zone.
2. Create recurring cleaning and organization tasks.
3. Mark tasks as completed.
4. Automatically calculate the next due date based on recurrence.
5. Optionally upload photos of a zone.
6. Use AI vision to identify visible organization or cleaning issues.
7. Match AI findings against existing tasks to avoid duplicates.
8. Ask the user to confirm new AI findings before creating tasks.

The system is designed for personal use first.

Do not build unnecessary multi-user, billing, or enterprise functionality during the MVP phase.

---

## 2. Core Concept

The fundamental hierarchy is:

House
→ Floor
→ Room
→ Zone
→ Photo
→ AI Analysis
→ Task

Example:

House
└── 1F
└── Living Room
├── Sofa Area
├── TV Area
└── Storage Cabinet

Tasks belong to a specific Zone.

Photos also belong to a specific Zone.

AI analysis operates on photos and produces findings related to that Zone.

---

## 3. Important Product Principles

### 3.1 Photos are optional

The user does NOT need to take photos every day.

Recurring tasks must continue to work normally without photos.

Photos are only used when the user chooses to upload one.

AI analysis runs when:

* a new photo is uploaded
* the user explicitly requests re-analysis

Do not design the system around daily photo-taking.

---

### 3.2 Existing recurring tasks are persistent

Completing a recurring task does NOT delete the task.

Instead:

1. Create a completion history record.
2. Update `last_completed_at`.
3. Calculate the next `next_due_at`.

Example:

Weekly task:

* Completed: September 11
* Next due: September 18

The task remains active.

---

### 3.3 AI should not create duplicate tasks

When AI analyzes a photo:

1. Identify visible actionable issues.
2. Search existing active tasks in the same Zone.
3. Determine whether the finding corresponds to an existing task.
4. If matched, show the existing task.
5. If it is genuinely new, propose a new task.
6. New AI-generated tasks require user confirmation.

Do not automatically create new tasks without confirmation.

Semantic similarity should be preferred over exact string matching.

---

### 3.4 AI should only identify visible and actionable issues

AI should avoid:

* guessing hidden problems
* inventing objects that cannot be seen
* creating duplicate findings
* making subjective judgments unrelated to cleaning or organization
* identifying dangerous or irrelevant conditions unless explicitly requested

Example:

Visible clothes piled on floor:

> Organize clothes on floor

Good.

Invisible assumption:

> Closet probably needs cleaning

Bad.

---

## 4. Recommended Technology Stack

Use:

* Next.js
* TypeScript
* React
* Supabase PostgreSQL
* Supabase Storage
* OpenAI Vision API or equivalent vision-capable AI model
* GitHub
* Vercel

Use environment variables for all secrets.

Never commit API keys or service-role keys to GitHub.

---

## 5. Suggested Project Structure

```text
family/
├── CLAUDE.md
├── README.md
├── docs/
│   ├── PRODUCT.md
│   ├── DATABASE.md
│   ├── AI_ANALYSIS.md
│   ├── TASK_SCHEDULER.md
│   ├── ARCHITECTURE.md
│   └── ROADMAP.md
│
├── app/
│   ├── page.tsx
│   ├── home/
│   ├── rooms/
│   ├── zones/
│   ├── tasks/
│   ├── photos/
│   └── api/
│
├── components/
│   ├── TaskCard.tsx
│   ├── TodayList.tsx
│   ├── PhotoUpload.tsx
│   ├── AIProposalList.tsx
│   └── HomeTree.tsx
│
├── lib/
│   ├── supabase/
│   ├── services/
│   │   ├── task-service.ts
│   │   ├── scheduler-service.ts
│   │   ├── photo-service.ts
│   │   ├── ai-analysis-service.ts
│   │   └── task-matching-service.ts
│   ├── validation/
│   └── types/
│
└── supabase/
    └── migrations/
```

The exact structure may be adjusted if the existing codebase provides a better implementation approach.

---

## 6. Existing Repository

The repository may contain legacy HTML files from an earlier project.

Do NOT immediately delete old files.

First:

1. Inspect the existing repository.
2. Understand what the existing files do.
3. Determine whether anything can be reused.
4. Determine what should eventually be replaced.
5. Propose a migration strategy.

Do not perform a destructive rewrite without confirmation.

---

## 7. MVP Features

Phase 1 should focus on:

### Home Structure

* House
* Floors
* Rooms
* Zones

### Task Management

* Create task
* Edit task
* Delete/archive task
* Set priority
* Set estimated duration
* Set recurrence
* Mark complete
* View completion history

### Recurrence

Support at minimum:

* No repeat
* Daily
* Every N days
* Weekly
* Every N weeks
* Monthly
* Every N months
* Yearly

Monthly recurrence must use calendar-month arithmetic rather than simply adding 30 days.

### Today

Show tasks that are:

* overdue
* due today
* optionally upcoming

Ranking should consider:

* priority
* overdue status
* how late the task is
* estimated duration

### Photo Analysis

User can:

1. Select a Zone.
2. Upload a photo.
3. Ask AI to analyze it.
4. Review AI findings.
5. Match findings to existing tasks.
6. Confirm new tasks.
7. Optionally edit proposed recurrence.

---

## 8. Task Data Principles

Each task should have a stable ID.

A recurring task should contain information such as:

* name
* description
* zone
* source
* task type
* priority
* estimated minutes
* recurrence unit
* recurrence interval
* last completed time
* next due time
* active status
* created time
* updated time

Completion history must be stored separately.

Do not overwrite or destroy historical completion records.

---

## 9. AI Data Principles

Store:

* photo record
* AI analysis record
* model used
* prompt version
* raw structured result
* individual findings
* matching result
* accepted/rejected status

AI analysis should be historical.

Do not overwrite previous analysis records when a photo is analyzed again.

---

## 10. Validation

AI output must be validated before writing to the database.

Prefer a typed schema validation system such as Zod.

Never trust raw AI output.

Validate:

* finding name
* description
* priority
* estimated minutes
* recurrence unit
* recurrence interval
* matching task ID

Invalid AI output must not create database records.

---

## 11. Scheduling Rules

The scheduler must be deterministic.

For recurring tasks:

```text
completion
    ↓
record completion history
    ↓
update last_completed_at
    ↓
calculate next_due_at
```

For example:

```text
Task: Clean bathroom
Frequency: Every 7 days
Completed: 2026-09-11
Next due: 2026-09-18
```

Displaying a task must never change its schedule.

Only completing the task changes its schedule.

---

## 12. Time Budget

A future or MVP-compatible Today feature may support a time budget.

Example:

```text
I have 20 minutes.
```

The system can recommend tasks whose estimated total duration fits within approximately 20 minutes.

Prioritize:

1. overdue high-priority tasks
2. due-today high-priority tasks
3. overdue medium-priority tasks
4. other useful tasks

Do not automatically split a task unless task splitting is explicitly implemented.

---

## 13. Security

Never commit:

```text
.env
.env.local
API keys
Supabase service-role keys
private credentials
```

Use environment variables.

If authentication is introduced later, implement appropriate Supabase Row Level Security.

---

## 14. Development Rules

Before making major changes:

1. Read this file.
2. Read all files in `docs/`.
3. Inspect the existing repository.
4. Understand existing functionality.
5. Avoid unnecessary rewrites.
6. Keep changes modular.
7. Prefer small, testable commits.
8. Do not remove legacy functionality without understanding its purpose.
9. Do not introduce unnecessary dependencies.
10. Do not implement features that are outside the current roadmap without confirmation.

---

## 15. Testing Expectations

Important logic should have tests, especially:

* recurrence calculation
* monthly recurrence
* overdue calculation
* Today ranking
* task completion
* AI result validation
* AI finding matching
* duplicate prevention

The scheduler should be testable independently from the UI.

---

## 16. Development Workflow

For each major feature:

```text
Requirement
    ↓
Design
    ↓
Database / API
    ↓
Service logic
    ↓
UI
    ↓
Validation
    ↓
Tests
```

Avoid putting business logic directly into React components.

Business logic should live in reusable services or domain functions.

---

## 17. First Development Step

Before changing the existing repository, Claude should:

1. Read `CLAUDE.md`.
2. Read every file in `docs/`.
3. Inspect the existing source code.
4. Explain the existing architecture.
5. Identify reusable code.
6. Identify obsolete code.
7. Propose the migration plan.
8. Propose the Phase 1 implementation order.

Do not make large modifications until the user confirms the plan.

---

## 18. Product Priority

When there is a conflict between features, prioritize:

1. Correct task scheduling
2. Reliable task completion history
3. Clear home/zone structure
4. Accurate AI findings
5. Duplicate prevention
6. Simple user experience
7. Visual polish
8. Future advanced features

Correctness is more important than adding more features.

---

## 19. Out of Scope for Initial MVP

Do not prioritize:

* multi-user accounts
* subscriptions
* billing
* social features
* public sharing
* complex permissions
* advanced analytics
* IoT integrations
* smart-home automation
* elaborate gamification

These can be considered later.

---

## 20. Definition of Done for MVP

The MVP is considered functional when the user can:

1. Create a house.
2. Create floors.
3. Create rooms.
4. Create zones.
5. Create recurring cleaning/organization tasks.
6. See tasks due today.
7. Complete a task.
8. Automatically receive the correct next due date.
9. View completion history.
10. Upload a photo to a zone.
11. Run AI analysis.
12. See AI findings.
13. Match findings against existing tasks.
14. Confirm new tasks.
15. Avoid duplicate task creation.

The system should remain useful even if the user never uploads a photo.
