<p align="center">
  <br />
  <br />
  <strong>HELM</strong>
  <br />
  A personal operations system for freelancers, built on Claude Code.
  <br />
  <br />
  <a href="#architecture">Architecture</a> &nbsp;&middot;&nbsp; <a href="#the-standup">The Standup</a> &nbsp;&middot;&nbsp; <a href="#skills">Skills</a> &nbsp;&middot;&nbsp; <a href="#getting-started">Get Started</a>
  <br />
  <br />
</p>

---

Tracking should be a byproduct of working, not a separate activity. HELM makes that real. When you close a Claude Code session, hooks update your project status automatically. A morning standup conversation plans your day from actual data. A wrap at end of day captures what happened. Over weeks, the system learns how you work and adjusts.

No boards. No cards. No app to update. Markdown files, git history, and a conversation.

---

## Architecture

Six repos. One parent folder. Everything connected through shared state.

```
helm/
  business/     Daily command center. Standup, wrap, proposals, contracts.
  ops/          Shared state. Project files, daily plans, config, templates.
  personal/     Life operations. Goals, values, health tracking, personal tasks.
  financial/    Business finances. Evolving checklists, receipts, invoices.
  hub/          Background automation. Cron jobs, context builders.
  projects/     Client repos. One per engagement.
```

Each repo has its own `CLAUDE.md` scoped to its purpose. The business workspace loads planning context. Project repos load technical context. They share state through `ops/` without polluting each other's context windows.

### Why markdown?

Claude Code reads and writes it natively. Git gives version history. Files are human-readable in any editor. There's nothing to install, configure, or migrate. Your entire operations history is in `git log`.

### Information flow

```
                    ┌─────────────┐
                    │  Overnight  │
                    │  Cron Jobs  │
                    │  (hub/)     │
                    └──────┬──────┘
                           │ writes context
                           v
┌──────────┐    ┌─────────────────────┐    ┌──────────────┐
│ Personal │───>│    ops/ (shared)    │<───│  Financial   │
│ plan.md  │    │                     │    │  calendar    │
└──────────┘    │  projects/*.md      │    └──────────────┘
                │  daily/*.md         │
                │  config/            │
                │  templates/         │
                └──────────┬──────────┘
                           │ reads state
                           v
                ┌─────────────────────┐
                │   business/         │
                │   /standup          │──── writes daily plan
                │   /wrap             │──── writes end-of-day
                │   /retrospective    │──── writes patterns
                └─────────────────────┘
                           │ generates instructions
                           v
                ┌─────────────────────┐
                │  projects/*         │
                │  SessionEnd hooks   │──── auto-update ops/
                └─────────────────────┘
```

---

## The Standup

The standup is the heartbeat of the system. It's a morning conversation, not a form. It asks how you're doing, reads everything in your system, and helps you decide what matters today.

### What it draws from

The standup loads: overnight git diffs across all project repos, yesterday's daily file, this week's plan, your personal goals and values, active tasks across business and personal, financial deadlines, weather forecast, behavioral patterns learned over time, and anything you captured from your phone overnight.

### What it produces

A daily plan with 2-4 items scaled to your energy. Per-project instruction files that your project Claude picks up when you start working. A hit list synced to Notion. A cleared inbox.

### How it thinks

The conversational framework draws on practical ideas without naming them:

Avoidance is information, not a character flaw. The tasks you resist most often matter most. When the system notices you've been dodging something for three weeks, it names that directly: *"What's the actual blocker? Is it that you don't know where to start, or is it that you'd rather do literally anything else?"* Both are valid answers. Either one leads to a next step.

When motivation is low, make the task tiny. "Open the app and categorize 10 transactions" is a task. "Do the bookkeeping" is not. The standup breaks everything into the smallest action that moves it forward.

Focus on what you control. If a client hasn't responded, the standup doesn't plan work that depends on them. It redirects to what you can move today.

The list will never be empty. The standup limits your day to what's realistic, not what's possible. Permission to leave things for tomorrow is built in, not an afterthought.

### Default behavior

The standup is neurodivergent-friendly by default. This isn't a mode you toggle. It's how the system works:

- **3-5 items max** per day, scaled to stated energy
- **Micro-step breakdowns** (3-10 minutes per step)
- **Buffered time estimates** (1.5x your first guess)
- **Permission language** ("you don't have to finish this")
- **Revenue-weighted priorities** (what gets you paid comes first)
- **Condition-aware pairing** (outdoor tasks on nice days, desk work on high pollen days)

---

## Skills

### Daily Operations

| Skill | What it does |
|:------|:-------------|
| `/standup` | Morning planning conversation. Reads everything, plans your day, generates project instructions. |
| `/wrap` | End-of-day review. Scans what changed, captures what happened, stages tomorrow. |
| `/weekly-plan` | Monday strategic planning. Sets 3-5 weekly objectives that standups reference. |
| `/retrospective` | Friday analysis through four lenses: revenue, energy, avoidance, growth. Updates behavioral patterns. |

### Client Work

| Skill | What it does |
|:------|:-------------|
| `/new-proposal` | Guided questions, past pricing reference, voice-matched document generation. |
| `/new-contract` | Assembles contracts from reusable blocks. Playbook verification via reviewer agent. |
| `/follow-up` | Detects overdue client communication. Drafts messages in your voice. |
| `/new-project` | Full scaffolding: ops file, repo, hooks, GitHub, repo-map entry. |

### Status Updates

| Skill | What it does |
|:------|:-------------|
| `/done` | Manual project update. Fallback for when hooks miss context. |
| `/update` | Quick one-liner status append. No conversation. |

---

## Automation

### Hooks

**SessionEnd** hooks fire when you close a Claude Code session on any project repo. They scan git commits, update the project ops file, check milestone completion, and flag when something is ready for invoicing. You don't run anything. It just happens.

A **verification gate** checks that the ops file was actually modified and the YAML still parses. If the hook fails, the next morning's standup tells you.

### Overnight Jobs

A set of cron scripts run on the always-on machine:

| Job | Purpose |
|:----|:--------|
| Context builder | Git diffs across all repos since last standup |
| Weather fetch | Tomorrow's forecast + pollen via Open-Meteo (free, no API key) |
| Inbox capture | Pulls phone notes (Apple Notes via AppleScript) into inbox.md |
| Uncommitted check | Flags repos with uncommitted changes |
| Fallback wrap | 11:45pm safety net if you forgot to wrap |
| NAS backup | rsync critical directories to network storage |

---

## Data Model

### Project files

YAML frontmatter tracks what matters for a freelancer: contract value, milestone-based payment, deliverable status, and the gap between "done" and "paid."

```yaml
contract:
  milestones:
    - name: Phase 1 Build
      value: 5000
      status: in-progress
      deliverables:
        - Feature A: complete
        - Feature B: in-progress
      closing:
        review_complete: false
        client_approved: false
        invoice_sent: false
        payment_received: false

waiting_on:
  - what: Design assets
    who: client
    since: 2026-03-10
    follow_up_interval_days: 7
```

The standup reads this and knows the full picture: what's close to done, what's blocked on someone else, when to prompt a follow-up, and when a milestone is ready to invoice.

### The closing pipeline

The gap between "deliverables complete" and "money in the bank" is where freelancers lose revenue. HELM tracks it explicitly: client notification, demo scheduling, approval, invoice, payment. The standup walks you through whichever step is next.

### Personal tasks

Personal items carry more metadata than typical task lists:

```yaml
- task: Plan cycling routes for spring
  added: 2026-03-05
  goal: Have routes mapped before outdoor season
  window: Before April 15
  effort: medium
  pairs_with: nice weather day, good energy
```

The standup matches personal tasks to conditions. Nice day with manageable allergies? Suggest the outdoor task. Low energy? Suggest the desk task that doesn't require much thinking.

---

## Self-Improving System

The weekly `/retrospective` analyzes your daily files through four lenses:

1. **Revenue** - What moved toward payment? What stalled?
2. **Energy** - Which days were productive? What correlated?
3. **Avoidance** - What kept appearing in carried-forward?
4. **Growth** - What did you learn? What got easier?

Findings write to `memory/patterns.md`. The standup reads patterns every morning. Over months, the system learns: your best days of the week, which conditions produce focus, what you chronically avoid, how long things actually take versus your estimates.

The system doesn't just track. It gets smarter.

---

## Project Lifecycle

```
Lead ──> /new-proposal ──> /new-contract ──> /new-project
                                                  │
          ┌───────────────────────────────────────┘
          v
     Daily work (hooks auto-track)
          │
          v
     Milestone complete (system detects)
          │
          v
     Closing pipeline (review, approval, invoice, payment)
          │
          v
     Project complete ──> archive
```

The proposal and contract skills reference your past pricing, use reusable contract blocks (payment terms, IP assignment, termination, warranty, liability), and generate documents in your writing voice extracted from past work.

---

## Getting Started

```bash
git clone https://github.com/guntherb7/helm.git
```

1. Copy the structure to your own private repos (one per directory)
2. Fill in your project files with real client data
3. Write your `personal/plan.md` (goals, values, what matters)
4. Fill in `business/memory/clients.md` with client contacts
5. Open `business/` in VS Code with Claude Code
6. Type `/standup`

The system learns from use. Start with the standup and wrap. Add automation after a week. Let checklists and patterns build naturally from real work.

See **[SETUP.md](SETUP.md)** for the full walkthrough.

---

## Influenced By

The design philosophy draws from these works:

[Acceptance and Commitment Therapy](https://en.wikipedia.org/wiki/Acceptance_and_commitment_therapy) (Steven Hayes)
[Tiny Habits](https://tinyhabits.com/) (BJ Fogg) &middot; Stoic practical philosophy
[The War of Art](https://stevenpressfield.com/books/the-war-of-art/) (Steven Pressfield)
[Four Thousand Weeks](https://www.oliverburkeman.com/books) (Oliver Burkeman)
[PARA Method](https://fortelabs.com/blog/para/) (Tiago Forte)

Technical patterns adapted from [Anthropic Knowledge Work Plugins](https://github.com/anthropics/knowledge-work-plugins), [Longshore Plugins + Skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills), [COG Second Brain](https://github.com/huytieu/COG-second-brain), [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices), and [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md).

---

## License

MIT. Use it, modify it, build on it. See [LICENSE](LICENSE).

Built by [Gunther Beam](https://github.com/guntherb7).
