---
name: financial-planning-app
description: Build and customize a personal finance, salary-cycle budget, and loan tracking web application (FinPlan) with 5-page navigation, centered Quick-Add pill bar, Commitment/Wants/Savings allocation ratios, rolled-up subcategories, and Bring-Your-Own-Supabase (BYOS) cloud sync. Use when the user asks to create, update, or extend a personal finance tracker, salary-cycle budget planner, loan tracker, or financial planning PWA.
---

# Financial Planning App (FinPlan)

Build and maintain a mobile-first, single-file personal finance, salary-cycle budget, and loan tracker (`index.html`) designed around real payday cycles rather than rigid calendar months, complete with optional Bring-Your-Own-Supabase (BYOS) cloud sync.

## When to Use

* The user asks to build, update, or customize a personal finance app, salary-cycle budget tracker, loan repayment tracker, or expense planner.
* The user wants a 5-page budgeting structure (**Home**, **Income**, **Budget**, **Loans**, **Analysis**) with **Commitment**, **Wants**, and **Savings** allocation ratios.
* The user wants Bring-Your-Own-Supabase (BYOS) sync so each user can connect their own private Supabase project without hardcoding keys in HTML.

## Core Architecture & Design Standards

### 1. Visual, Typography & Theme Standards

* **Automatic System Theme**: Sync with device light/dark mode (`prefers-color-scheme: dark`) using **Matte White** (`#F5F5F7` canvas, `#FFFFFF` cards) and **Matte Black** (`#09090B` canvas, `#111114` cards).
* **Typography**: `Inter` for UI/headings and `JetBrains Mono` for currency amounts (`RM`), percentages, dates, and metrics.
* **Navigation**: Floating Apple Liquid Glass bottom pill navigation bar (`Home`, `Income`, centered `+` Quick-Add button, `Budget`, `Loans`, `Analysis`) lifted safely above mobile gesture bars (`env(safe-area-inset-bottom)`).
* **Mobile Ergonomics**: Enforce `font-size: 16px` on form inputs under `768px` to prevent iOS Safari auto-zoom, and use compact `44px × 44px` icon-only `+` / calendar action buttons so titles and cycle dates have maximum width on mobile screens.

### 2. Persistent Header & Cycle Management

* **Header Controls**:
  * App logo/title (`FinPlan`) with no extra status dot next to the title, plus active page subtitle.
  * Cycle switcher pill displaying the active cycle date range (e.g., `1 Nov – 30 Nov 2026`).
  * Borderless icon-only **Clone** button (copies ratios, income sources, and budget items from the previous cycle) and borderless icon-only **Connect** link icon (colored **emerald green** when connected to Supabase, **rose red** when disconnected).

### 3. Five-Page Application Structure & Centered Quick-Add

* **Centered Pill Quick-Add (`+`)**: Opens a unified modal with an **Entry Type** dropdown (`Budget` or `Income`) and an inline calculator so users can record either entry type from any page.
1. **Page 1 — Home (`Monthly Summary`)**:
   * Displays **Total Income**, **Total Budget Allocation** (with percentage and progress bar), and **Salary Balance**, plus a **Category Plan Snapshot** comparing actual spending vs. target ratios across `Commitment`, `Wants`, and `Savings`.
2. **Page 2 — Income Plan**:
   * **Monthly Income**: Icon-only `+` button to record inflows with **Income Source**, **Date Received**, **Amount (RM)**, sorting options, and an inline calculator.
   * **Financial Plan Ratios**: Interactive percentage split for **Commitment**, **Wants**, and **Savings** (default `50% / 30% / 20%`) with segmented visual bar, live Target vs. Actual amounts, **Auto Balance 100%**, and **Set Default**.
3. **Page 3 — Budget**:
   * **Financial Cycle Card**: Displays the active cycle date range (`11px–14px` monospace) alongside an icon-only calendar button opening a range-selection calendar modal.
   * **Salary Balance Card**: Clean title, live remaining salary balance, and progress bar only (no subtext).
   * **Monthly Budget**: Icon-only `+` button, clean category tabs (`Commitment`, `Wants`, `Savings`), active category summary labeled **Subtotal**, **Allocation**, and **Balance**, and rolled-up subcategory cards (`Self`, `Bills`, `Parents`, `Loans`, `BNPL`, `Subscription`) expandable on tap (`<details>`) to edit or delete individual items.
4. **Page 4 — Loans (`Loan Tracker`)**:
   * **Overview Card**: Subtitle `Accumulated Loans`, icon-only `+` button, and KPI cards for **Total Remaining**, **Monthly Installment**, and **Total Paid Off** with progress bar.
   * **Loan Entry Cards**: Compact cards displaying **Loan Title**, **Remaining / Total Loan Amount** (numbers only), icon-only `+` Payment Entry button (pre-filled with monthly installment), icon-only **Edit** button (with **Delete Loan** inside the Edit modal), and a full-width **Payment History** button at the bottom of the card.
5. **Page 5 — Analysis**:
   * **Income Budget Comparison** bars, **Category Budget Breakdown** Doughnut chart, filterable **Subcategory Breakdown**, **Cycle Performance History** Bar chart + clickable history table (labeled by cycle month span such as `Nov-Dec`, filterable by **Year** dropdown, with a delete cycle button per row), and searchable **Past Income Ledger**.

## Bring-Your-Own-Supabase (BYOS) Cloud Sync

Never hardcode personal Supabase credentials in `index.html`. Inside the **Data & Cloud Sync** modal (`#data-modal`), provide inputs for **Project URL**, **Anon / Publishable Key**, and **Sync Profile ID** (stored in `localStorage`), **Connect & Sync**, **Pull from Cloud**, **Disconnect**, JSON/CSV export/import (storing `loansData` inside `app_data.__loans__` so no extra SQL columns are needed), and **Copy Table SQL**:

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

* Keep everything self-contained in a single `.html` file (`index.html`) with zero external build tools required so it deploys cleanly on Cloudflare Pages, Netlify, Vercel, or GitHub Pages.
* Store `loansData` inside `app_data.__loans__` when syncing to Supabase so the existing `finplan_workspaces` table works seamlessly without requiring schema migrations.
* Always include `to anon, authenticated` and explicit `GRANT` statements in the Supabase SQL setup to avoid `[42501]` RLS permission errors.