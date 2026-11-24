---
name: financial-planning-app
description: Build and customize a personal finance and salary-cycle budget tracking web application (FinPlan) with 4-page navigation, Commitment/Wants/Savings allocation ratios, rolled-up subcategories, and Bring-Your-Own-Supabase (BYOS) cloud sync. Use when the user asks to create, update, or extend a personal finance tracker, salary-cycle budget planner, or financial planning PWA.
---

# Financial Planning App (FinPlan)

Build and maintain a mobile-first, single-file personal finance and salary-cycle budget planner (`index.html`) designed around real payday cycles rather than rigid calendar months, complete with optional Bring-Your-Own-Supabase (BYOS) cloud sync.

## When to Use

- The user asks to build, update, or customize a personal finance app, salary-cycle budget tracker, or expense planner.
- The user wants a 4-page budgeting structure (**Home**, **Income Plan**, **Budget**, **Analysis**) with **Commitment**, **Wants**, and **Savings** allocation ratios.
- The user wants Bring-Your-Own-Supabase (BYOS) sync so each user can connect their own private Supabase project without hardcoding keys in HTML.

## Core Architecture & Design Standards

### 1. Visual, Typography & Theme Standards
- **Automatic System Theme**: Sync with device light/dark mode (`prefers-color-scheme: dark`) using **Matte White** (`#F5F5F7` canvas, `#FFFFFF` cards) and **Matte Black** (`#09090B` canvas, `#111114` cards).
- **Typography**: `Inter` for UI/headings and `JetBrains Mono` for currency amounts (`RM`), percentages, dates, and metrics.
- **Navigation**: Floating Apple Liquid Glass bottom pill navigation bar (`Home`, `Income Plan`, `Budget`, `Analysis`) lifted safely above mobile gesture bars (`env(safe-area-inset-bottom)`), paired with a contextual floating `+` Quick-Add button on **Income Plan** and **Budget**.
- **Mobile Ergonomics**: Enforce `font-size: 16px` on form inputs under `768px` to prevent iOS Safari auto-zoom, and ensure crisp `11px–12px` secondary labels with high readability.

### 2. Persistent Header & Cycle Management
- **Header Controls**: Display the app logo/title with a live Supabase connection indicator dot, active page subtitle, a cycle switcher pill displaying the active cycle date range (e.g., `1 Nov – 30 Nov 2026`), a **Clone** button (copies ratios, income sources, and budget items from the previous cycle), and a **Data** backup/sync button.

### 3. Four-Page Application Structure
1. **Page 1 — Home (`Monthly Summary`)**:
   - Displays **Total Income**, **Total Budget Allocation** (with percentage and progress bar), and **Salary Balance** (remaining or over-budget status), plus a **Category Plan Snapshot** comparing actual spending vs. target ratios across `Commitment`, `Wants`, and `Savings`.
2. **Page 2 — Income Plan**:
   - **Monthly Income**: Record salary/inflow entries with **Income Source**, **Date Received**, **Amount (RM)**, sorting options, and an inline calculator inside the Add/Edit modal.
   - **Financial Plan Ratios**: Interactive percentage split for **Commitment**, **Wants**, and **Savings** (default `50% / 30% / 20%`) with segmented visual bar, live Target vs. Actual amounts, **Auto Balance 100%**, and **Set Default**.
3. **Page 3 — Budget**:
   - **Financial Cycle Card**: Displays the active cycle date range alongside an inline **Pick Cycle Calendar** button opening a range-selection calendar modal.
   - **Salary Balance Card**: Clean live remaining salary balance bar (title, balance value, and progress bar only).
   - **Monthly Budget**:
     - Clean category tabs (`Commitment`, `Wants`, `Savings`) without cluttered item counts on tab headers.
     - Active category summary showing **Subtotal**, **Allocation**, and **Balance**.
     - **Subcategories**: `Self`, `Bills`, `Parents`, `Loans`, `BNPL`, `Subscription`.
     - **Rolled-Up Subcategory Cards**: On the Budget page, group items by subcategory showing only the subcategory name and combined total amount, expandable on tap (`<details>`) to edit or delete individual items (useful when customizing cloned cycles).
4. **Page 4 — Analysis**:
   - **Income Budget Comparison** bars, **Category Budget Breakdown** Doughnut chart, **Subcategory Breakdown** (filterable by category, showing percentage bars and full itemized entries), **Cycle Performance History** Bar chart + clickable history table, and searchable **Past Income Ledger**.

## Bring-Your-Own-Supabase (BYOS) Cloud Sync

Never hardcode personal Supabase credentials in `index.html`. Inside the **Data & Cloud Sync** modal (`#data-modal`), provide:
- Inputs for **Project URL**, **Anon / Publishable Key**, and **Sync Profile ID** (stored in `localStorage`).
- **Connect & Sync**, **Pull from Cloud**, **Disconnect**, JSON/CSV export/import, and **Copy Table SQL** with full grants and RLS policy:

```sql
grant usage on schema public to anon, authenticated;

create table if not exists public.finplan_workspaces (
  workspace_id text primary key,
  app_data jsonb not null default '{}'::jsonb,
  default_ratios jsonb not null default '{"Commitment": 50, "Wants": 30, "Savings": 20}'::jsonb,
  default_cycle_start_day integer not null default 1,
  updated_at timestamptz not null default now()
);

grant all on table public.finplan_workspaces to anon, authenticated;
alter table public.finplan_workspaces enable row level security;

drop policy if exists "Allow public workspace read/write" on public.finplan_workspaces;

create policy "Allow public workspace read/write"
  on public.finplan_workspaces
  for all
  to anon, authenticated
  using (true)
  with check (true);

notify pgrst, 'reload schema';
```

## Gotchas & Guardrails
- Keep everything self-contained in a single `.html` file with zero external build tools required so it deploys cleanly on Cloudflare Pages, Netlify, Vercel, or GitHub Pages.
- Ensure all `<script>` tags and CDN imports (`Tailwind`, `Chart.js`, `Supabase JS`) stay strictly inside `<head>` or the bottom script block—never leave stray JS snippets in `<head>`.
- Always include `to anon, authenticated` and explicit `GRANT` statements in the Supabase SQL setup to avoid `[42501]` RLS permission errors.