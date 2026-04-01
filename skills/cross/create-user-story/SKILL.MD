---
name: create-user-story
description: "Create a well-structured Jira user story ticket following Product Team best practices. USE FOR: writing new user stories; drafting Jira tickets from a feature idea or business requirement; turning a vague request into a structured story with value hypothesis and acceptance criteria; reviewing or improving an existing user story. DO NOT USE FOR: technical task breakdowns (sub-tasks); bug reports; sprint planning or estimation; writing technical design documents."
---

# Create Jira User Story — Best Practices

You are an expert product analyst. When asked to create a user story, follow these rules strictly and output a complete Jira-ready ticket using the template below.

---

## Core Principles

A good user story must:
- Be written from the **customer/user perspective** — never from an internal or technical perspective
- Include a **measurable value hypothesis** so the story can be prioritised
- Leave room for the tech team to suggest solutions — do NOT describe detailed technical designs
- Use **plain, simple language** — no jargon or TLAs (Three-Letter-Acronyms)

---

## Step 1 — Gather Required Information

Before writing, confirm you have answers to all of these. If any are missing, ask the user:

| # | Question | Why It Matters |
|---|----------|----------------|
| 1 | **Who is the user?** | Role + market/channel/situation. Be specific. |
| 2 | **What do they want to do?** | The Job-to-be-Done (JTBD) — action, not solution. |
| 3 | **Why do they want it?** | The goal or benefit. This is the most critical part. |
| 4 | **What is the current state?** | How do users solve this today? Pain points? |
| 5 | **What is the measurable impact?** | 1 primary metric: $, %, conversion, NPS, time saved… |
| 6 | **Which OKR / strategic objective?** | Maps the story to business goals. |
| 7 | **Solution ideas** | At least 2 high-level options (MVP vs full feature). |
| 8 | **Acceptance Criteria** *(optional)* | When is the story "done"? |

---

## Step 2 — Choose the Right User Type

Use specific, real user types. Never write "As a product owner…" for customer-facing stories.

| Segment | Example User Types |
|---------|--------------------|
| **Customer** (works at a CPG business) | Trade Manager, Marketing Manager, Commercial Manager |
| **Staff Member** (works in Retail / Wholesale / Supply Chain) | Cashier, Store Manager, Wholesaler |
| **Consumer** (buys CPG items in a retail store, bar, café, online…) | Supermarket Shopper, Café Visitor |

---

## Step 3 — Write the Ticket

Use this exact template. Do not skip any section. Add `*(Optional)*` only for Supporting Materials and Acceptance Criteria if not provided.

```markdown
## User Story

As a [specific type of user] [in context/situation],
I want to [perform some action],
so that [I can achieve a specific goal or benefit].

---

## Context / Problem

> Describe the current state. How does the user solve this problem today?
> What pain points, friction, or gaps exist?

[Describe current state here]

---

## Value Hypothesis

> Choose 1 primary impact measure. Estimate a measurable value with rationale.
> Include Reach (how many users affected?) and Impact (what changes for them?).

- **Primary Metric:** [e.g., conversion rate / revenue / time saved / NPS]
- **Estimated Value:** [e.g., +5% conversion → ~$50K ARR based on X active customers]
- **Reach:** [e.g., affects ~200 Trade Managers across MX and BR markets]
- **Strategic Mapping:** [e.g., OKR: Grow Active Users Q2 | North Star: Monthly Active Campaigns]

---

## Possible Solutions

> At least 2 high-level options. Do NOT include detailed technical designs.

1. **Option A (MVP / Quick Win):** [Short description]
2. **Option B (Full Feature):** [Short description]
3. *(Optional)* **Option C (Experimental):** [Short description]

---

## Supporting Materials *(Optional)*

- [ ] Customer flow / user journey diagram
- [ ] UI/UX designs, wireframes, or rough sketches
- [ ] Screenshots or real examples from other products
- [ ] Client examples most affected by this issue
- [ ] Customer verbatims or stories of when things went wrong

---

## Acceptance Criteria *(Optional)*

- [ ] Given [context], when [action], then [expected result]
- [ ] ...
```

---

## Quality Checklist

Before outputting the ticket, verify every item:

- [ ] Written from the **user's perspective**, not the team's
- [ ] User is **specific** — includes role + market/channel/context
- [ ] The **"so that"** clause is meaningful and non-vague
- [ ] **One** measurable metric chosen for the value hypothesis
- [ ] **At least 2** solution options provided — no detailed tech designs
- [ ] No jargon, acronyms, or internal terminology used

---

## Common Mistakes to Avoid

| ❌ Mistake | ✅ Correct Approach |
|-----------|-------------------|
| "As a product owner..." for customer-facing stories | Use the real end user (e.g., "Trade Manager in MX") |
| "Implement an API endpoint that returns..." | Describe the user action, not the technical solution |
| Multiple KPIs in the value hypothesis | Pick exactly one primary metric |
| Vague "so that" (e.g., "so that it's better") | Describe a concrete, observable benefit |
| Jargon like "leverage synergies to drive KPIs" | Use plain language a customer would understand |

---

## Example Output

```markdown
## User Story

As a Trade Manager at a CPG company in Mexico,
I want to see a real-time summary of my active campaign performance on a single dashboard,
so that I can quickly decide which campaigns to boost or pause without switching between multiple reports.

---

## Context / Problem

Currently, Trade Managers must export data from 3 separate screens and manually compile
an Excel report each week. This takes ~3 hours per manager per week and often results in
delayed decisions. Campaigns sometimes run past their optimal window due to lack of visibility.

---

## Value Hypothesis

- **Primary Metric:** Campaign decision turnaround time
- **Estimated Value:** Reduce weekly reporting effort by ~3 hrs/manager → ~$15K/year saved
  across 20 active Trade Managers in MX
- **Reach:** ~20 Trade Managers in MX market (expandable to BR: ~35 users)
- **Strategic Mapping:** OKR: Improve Customer Retention Q2 | North Star: Active Monthly Campaigns

---

## Possible Solutions

1. **Option A (MVP):** Add a summary widget to the existing Campaign List page showing
   top 5 KPIs (impressions, clicks, redemptions, spend, ROI) with a 24h refresh.
2. **Option B (Full Feature):** Build a dedicated Campaign Performance Dashboard with
   real-time data, filters by date/region/product, and export capability.
3. **Option C (Experimental):** Send an automated daily WhatsApp/email digest summarising
   the top 3 campaigns by performance for each Trade Manager.

---

## Acceptance Criteria

- [ ] Given I am on the dashboard, when I open it, then I see performance data refreshed within the last 24 hours
- [ ] Given I have multiple active campaigns, when I view the summary, then I can see at least: impressions, clicks, redemptions, spend
- [ ] Given I am a Trade Manager in MX, when I log in, then I only see campaigns assigned to my account
```
