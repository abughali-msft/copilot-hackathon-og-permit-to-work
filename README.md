# Digital permit-to-work

**GitHub Copilot Hackathon: Build Smart, Spend Smart** · Oil & Gas breakout · Challenge B: Workflow app

Request, approve and close work permits with the right person at every step, and a full history for every permit.

> [!IMPORTANT]
> Sample data only. Never bring real personnel data or credentials into this repo. This is a prototype, not a certified safety system, so never use it to authorize real work.

## Start here

1. Select **Use this template** > **Create a new repository** to create your team's copy. Make it public, or give the judges access.
2. Clone it and open it in your IDE with GitHub Copilot Chat.
3. Read this brief and the [data dictionary](data/README.md).
4. **Before you build**, plan with Copilot Plan mode and estimate your AI Credits per phase in [docs/ai-credit-budget.md](docs/ai-credit-budget.md).
5. Build, test, review and document with Copilot. Commit often: your commit history is evidence.
6. Before your demo, complete [Your solution](#your-solution) at the end of this README.

## The day at a glance

The day has two goals:

1. Use GitHub Copilot beyond autocomplete: plan, agent, instructions, prompt files and review.
2. Build the habit of planning AI Credit consumption before you start prompting.

| | |
|---|---|
| **Team** | 3–5 developers |
| **Build time** | 4 hours |
| **Usage checkpoints** | Your facilitator records each seat's AI Credit usage before the build starts, at the mid-point check (2:00) and at the end. |
| **Demo** | 5 minutes, then Q&A |
| **Scoring** | 100 points. See the [scoring rubric](#scoring-rubric). |

## The challenge

**Who it's for:** area authorities, work-party supervisors and HSE officers at an oil and gas site.

**The problem:** paper permits get lost, steps get skipped and nobody can prove who approved what. On a hazardous site, the wrong person approving hot work can be fatal.

**What to build:** a workflow app to request, approve and close work permits. Every step checks the user's role and site, and every permit keeps a full, append-only history.

### Workflow

| From | Action | To | Who can do it |
|---|---|---|---|
| (new) | Request | `REQUESTED` | A `REQUESTER` assigned to the site |
| `REQUESTED` | Approve | `APPROVED`, once every required approval is in | The site's `AREA_AUTHORITY`, plus an `HSE_OFFICER` for types that need HSE approval |
| `REQUESTED` | Reject, with a reason | `REJECTED` | Anyone who could approve it |
| `REQUESTED` | Cancel | `CANCELLED` | The requester |
| `APPROVED` | Close, once the work is done and the site is safe | `CLOSED` | The site's `AREA_AUTHORITY` |

### Checks

| Check ID | Rule |
|---|---|
| `ROLE_REQUIRED` | Each action needs the role shown in the workflow table. |
| `SITE_ASSIGNMENT` | Users act only at sites they're assigned to (`ALL` covers every site). |
| `SEPARATION_OF_DUTIES` | Nobody approves or closes a permit they requested. |
| `ACTIVE_USER` | Inactive users can't do anything. |
| `HSE_APPROVAL` | Types with `requires_hse_approval` = `Y` stay `REQUESTED` until both the area authority and an HSE officer approve. |
| `GAS_TEST` | For types with `requires_gas_test` = `Y`, the HSE officer records a gas test that passes: oxygen **19.5% to 23.5%** inclusive and flammable gas **below 10% LEL** (illustrative suggested default). |
| `VALIDITY_LIMIT` | `valid_to` is after `valid_from`, and the permit lasts no longer than its type's `max_validity_hours`. |
| `VALID_TRANSITION` | Only the transitions in the workflow table are allowed. |

Every refused action returns a clear reason that names the check, for example `SEPARATION_OF_DUTIES: U-007 requested this permit, so U-007 can't approve it`.

Every action appends an entry to the permit's history: the permit, the action, the from and to state, the user, the role used, the time and a comment. Entries can't be edited or deleted.

The defaults are a starting point. You can change them if you record the change in your plan and test it. Keep limits in configuration, not in code.

Decide these in your plan and cover them with tests:

- **Refused attempts:** whether they're logged in the history. We suggest yes: a refused approval is evidence too.
- **Expired permits:** what happens to a permit that passes its `valid_to` before it's closed.
- **More than one role:** which role a user acts in, and whether one person can give both approvals on a permit that needs HSE approval.
- **No area authority:** what happens at a site with no active area authority.
- **Boundaries:** test values exactly at the limits, such as a permit of exactly `max_validity_hours`, oxygen of exactly 19.5% and flammable gas of exactly 10% LEL.
- **Messy data:** the data has deliberate quality issues. See [Things to know](data/README.md#things-to-know).

### What to build

**Must have**

- [ ] Load the CSV files in [`data/`](data/): users, roles, sites and permit types.
- [ ] Request, approve, reject, cancel and close permits.
- [ ] All the checks, each with a clear refusal reason.
- [ ] A permit list that filters by site, status and type, and a page or endpoint that shows a permit's full history.
- [ ] Automated tests for every check and every transition.

**Stretch goals (optional)**

- `ACTIVE` and `SUSPENDED` states, with a shift handover.
- Automatic expiry when a permit passes its `valid_to`.
- A conflict check, so two hot-work permits at the same site can't overlap (simultaneous operations).
- A printable permit with a QR code.
- An audit view for `VIEWER` users.

Use any language, framework or database, as long as it runs locally from your README. No machine learning is needed: plain business rules only.

## Starter data

| File | Rows | Contents |
|---|---:|---|
| [`sites.csv`](data/sites.csv) | 6 | Sites, with their area and hazardous-area zone |
| [`permit_types.csv`](data/permit_types.csv) | 6 | Permit types, with the approvals, gas test and maximum validity each one needs |
| [`roles.csv`](data/roles.csv) | 4 | The four roles in the workflow |
| [`users.csv`](data/users.csv) | 24 | Users, with their roles, their sites and whether they're active |
| [`sample_requests.csv`](data/sample_requests.csv) | 20 | Permit requests for one work week (15 to 19 November 2026) to try in your app |

All data is synthetic. The sites, people and work are fictional, and the gas-test limits are illustrative, not regulatory. The [data dictionary](data/README.md) explains every column and the data-quality issues.

## Copilot focus for this challenge

Every team should use the practices the rubric scores: Plan mode, agent mode, a `.github/copilot-instructions.md` file with your team's conventions, reusable prompt files and Copilot Chat review. This challenge puts extra weight on three of them:

| Practice | How to apply it here |
|---|---|
| **Instructions file for your stack** | Describe your stack, project structure and conventions in `.github/copilot-instructions.md` before you scaffold, so the code agent mode writes fits together. |
| **Agent scaffolding** | Use agent mode to scaffold the data model, the state machine, the checks and the UI or API, seeded from `data/`. |
| **Copilot Chat review** | Ask Copilot Chat to review every action for missing role, site or state checks. Fix the findings and note what it caught. |

<details>
<summary>Example prompts</summary>

**Instructions file** (for example, a first draft of `.github/copilot-instructions.md`)

```text
Read README.md and data/README.md. Draft .github/copilot-instructions.md for our stack:
<language, framework, database and test runner>. Describe the project structure, naming
and test conventions, and say that every action runs all the checks and appends to the history.
```

**Agent scaffolding**

```text
Using our instructions file, scaffold the permit-to-work app. Load the CSV files in data/,
model permits and their append-only history, build the state machine from the workflow
table, and add one function per check that returns the check ID and a clear reason.
```

**Review**

```text
Review every action in the permit workflow for missing role, site, active-user or state
checks, and for any way to approve or close a permit you requested. List the findings by
severity. Don't change any code yet.
```

</details>

## AI Credit budget

Estimate AI Credits per phase before you build, then track what you actually use in [docs/ai-credit-budget.md](docs/ai-credit-budget.md). The example below is from the kickoff and assumes a team of 3–5 (950 credits is about USD 9.50). Make your own estimate: the point is the habit of estimating, not hitting an exact number.

| Phase | Copilot feature | Model tier | Budget (AI Credits) |
|---|---|---|---:|
| Plan | Plan mode / Ask | Frontier | 150 |
| Scaffold | Agent mode + instructions file | Low-cost | 100 |
| Build features | Agent / Edit mode | Mixed | 500 |
| Tests + review | Prompt file for tests, chat review | Low-cost | 150 |
| Docs + demo | Ask, README generation | Low-cost | 50 |
| **Total** | | | **950** |

Tips to spend smart:

- Match the model to the task. Save frontier models for planning and tricky logic, and use low-cost models for scaffolding, tests and docs.
- Keep context scoped. Attach only the files Copilot needs.
- Reuse prompt files instead of retyping long prompts.

## Scoring rubric

The rubric awards 100 points across the lifecycle. It's the same in all three sectors, so winners are comparable.

| Stage | Points |
|---|---:|
| Plan | 15 |
| Build & quality with Copilot | 40 |
| Token efficiency | 25 |
| Impact & demo | 20 |
| **Total** | **100** |

### How scoring works

- Judges pick one level per stage: **Solid**, **Strong** or **Outstanding**. Each level is worth the points shown.
- Your score is the sum of the four stage scores. A stage with no evidence scores 0.
- Build & quality plus Token efficiency carry 65 points: they're the two goals of the day.
- Judges score from evidence, not claims: your repo, budget sheet, usage export and demo.
- Tie-break: the team with the higher Token efficiency score wins.
- Solid is the expected baseline. Most teams should reach Strong in four hours. Outstanding should stand out.

### Levels

| Stage | Solid | Strong | Outstanding |
|---|---|---|---|
| **Plan**<br>15 points | **5 pts**<br>Plan written with Copilot Plan mode; credit budget estimated per phase. | **10 pts**<br>Plan has acceptance criteria and a model choice per phase, with the reasoning explained. | **15 pts**<br>Plan drove the build, and the team re-planned at the mid-point check using actual usage. |
| **Build & quality with Copilot**<br>40 points | **16 pts**<br>Agent mode built the main features; `copilot-instructions.md` sets team conventions; runs end to end with tests on the core logic. | **28 pts**<br>Adds reusable prompt files for repeat tasks (tests, docs); edge cases tested; Copilot asked in chat to review the code and findings fixed. | **40 pts**<br>Copilot used across the whole loop (plan, build, test, review, docs); team shows where it saved the most time and where they had to correct it; another team could run it from the README. |
| **Token efficiency**<br>25 points | **10 pts**<br>Usage tracked; actual within ±30% of budget. | **18 pts**<br>Within ±15% of budget; models matched to tasks; context kept scoped. | **25 pts**<br>A concrete before/after where a prompt, context or model change cut credits without hurting the output. |
| **Impact & demo**<br>20 points | **8 pts**<br>Clear user and problem; working demo in 5 minutes. | **14 pts**<br>Rules or workflow are realistic for the sector; Q&A handled well. | **20 pts**<br>Credible path to a real pilot; judges can picture it being used. |

### What judges look at

| Stage | Evidence |
|---|---|
| Plan | `docs/plan.md` from Plan mode, and the budget and mid-point re-plan in `docs/ai-credit-budget.md` |
| Build & quality | `.github/copilot-instructions.md`, `.github/prompts/*.prompt.md`, tests, commit history and this README |
| Token efficiency | Budget versus actual usage in `docs/ai-credit-budget.md`, the usage export, and your before/after example |
| Impact & demo | Your 5-minute demo and Q&A |

### Clarifications

- **Token efficiency isn't about using the fewest credits.** It's judged against your own budget plus proof of optimization, so a team that barely used Copilot can't win it by spending little. Judges reward value delivered per credit. The ±30% and ±15% bands apply to overspending and underspending.
- **IDE features are enough.** Every level can be reached with Copilot Chat features in the IDE: Plan mode, agent mode, instructions, prompt files and chat review. If your organization also has Copilot code review on pull requests, MCP or the Copilot coding agent, they count as extra evidence for Outstanding, but they're never required.
- **Sector judgement counts.** Sector judges apply domain judgement in Impact & demo. For this challenge, treat the app as safety-critical: test every check and transition, and refuse unsafe actions with a clear reason.

## Your solution

> [!NOTE]
> Complete this section before your demo. Judges, and other teams, will run your app from it.

- **Team name:**
- **Team members:**
- **Stack:**

### Run it

```bash
# Replace with the exact steps to install and run your app.
```

### Test it

```bash
# Replace with the exact command that runs your tests.
```

### Check-to-test traceability

| Check | Implemented in | Tested in |
|---|---|---|
| `ROLE_REQUIRED` | | |
| `SITE_ASSIGNMENT` | | |
| `SEPARATION_OF_DUTIES` | | |
| `ACTIVE_USER` | | |
| `HSE_APPROVAL` | | |
| `GAS_TEST` | | |
| `VALIDITY_LIMIT` | | |
| `VALID_TRANSITION` | | |

### Copilot retrospective

- **Where Copilot saved us the most time:**
- **Where we had to correct it:**

### Evidence checklist

- [ ] Plan: `docs/plan.md`
- [ ] Budget, actual usage, mid-point re-plan and before/after example: [`docs/ai-credit-budget.md`](docs/ai-credit-budget.md)
- [ ] Usage export: `docs/usage/`
- [ ] Team conventions: `.github/copilot-instructions.md`
- [ ] Prompt files: `.github/prompts/`
