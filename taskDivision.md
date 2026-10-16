# Guardian: Task Division

**Project spec:** [Guardian.md](Guardian.md). Section numbers below (e.g. §9) refer to it.
**Deadline:** 20 Oct 2026, 23:59 UTC+8 · **Internal submission target:** 20 Oct, 18:00 UTC+8
**Feature freeze:** 19 Oct, 20:00 UTC+8 (after this, only bug fixes, docs, deck and video)

All times below are Beijing time (UTC+8).

---

## 1. Team and ownership

| Member | Role | Owns (folders / deliverables) |
|---|---|---|
| **Shawki** | Foundation & Setup Lead | Repo, monorepo scaffold, CI, interface specs (`spec/`), mock API, shared TypeScript library (`packages/shared`), Telegram alert bot (`bots/telegram/`), hosting |
| **Rafsan** | Blockchain & Red Team Engineer | `contracts/`, `deployments/`, `redteam/`: vault contract, tests, deployment, on-chain evidence, attack suite and results |
| **Mumti** | Team Leader · AI Agent, Docs & Submission Lead | Team registration, testnet wallets, `backend/`, `docs/` (except deck and video): agent backend, technical docs, security self-assessment, README, final submission |
| **Ashfaq** | Frontend, Deck & Video Lead | `frontend/`, `docs/deck.pdf`, `docs/demo.mp4`: all screens, presentation deck, demo video |

**Working rules**
- Each person works only inside their own folders. `spec/` is frozen after Shawki publishes it; changes need a one-line note in the group chat.
- Handovers happen through **files committed to the repo** (listed in Section 3), not meetings.
- When a handover file is pushed, post one line in the group chat: `HANDOVER H<n> ready: <file path>`.
- CI must stay green on `main`.

---

## 2. Responsibilities by member

### Shawki: Foundation & Setup Lead (all done by 18 Oct, 14:00)

| # | Task | Deliverable | Due |
|---|---|---|---|
| S1 | Create the GitHub repo and pnpm monorepo: `contracts/`, `backend/`, `frontend/`, `packages/shared/`, `bots/telegram/`, `redteam/`, `spec/`, `docs/`, `deployments/`; MIT license; `.gitignore`; `.env.example` | Repo link | 17 Oct, 11:00 |
| S2 | CI (GitHub Actions): `forge test`, backend typecheck and tests, frontend build | Green CI badge | 17 Oct, 14:00 |
| S3 | Write the interface specs from Guardian.md: `spec/policy.schema.json` (§8.2), `spec/IGuardianVault.sol` (interface, events, custom errors, §9), `spec/openapi.yaml` (all endpoints and SSE event shapes, §10.4, **including the extended-feature fields from Guardian.md §6: scam-radar flags, look-alike warning, rule-change diff, receipts, schedules**) | **H1** | 17 Oct, 13:00 |
| S4 | Mock API server generated from `openapi.yaml` (e.g. Prism) with realistic example responses for all six storyboard scenes; one command: `pnpm mock` | **H1** | 17 Oct, 15:00 |
| S5 | Shared TypeScript library `packages/shared`, with unit tests, used by both backend and frontend: zod types generated from `spec/policy.schema.json`; canonical JSON + keccak256 `constitutionHash` function; decoder that turns every vault custom error (§9.4) into a plain-English sentence (e.g. `ReserveLocked` → "Tuition is locked until Jan 15"); SSE event types; Basescan link helpers | **H1b** | 17 Oct, 18:00 |
| S6 | Create hosting projects (Vercel for `frontend/`, Railway or Render for `backend/`) with auto-deploy from `main`; wire the environment variable names from `.env.example` | **H1b** | 17 Oct, 18:00 |
| S7 | Root `README.md` skeleton: project summary, folder map, how to run each part locally | README | 17 Oct, 18:00 |
| S8 | Telegram alert bot for "Mom" (TypeScript, `bots/telegram/`): subscribes to the backend's `/api/events` stream and sends real phone messages for new-recipient requests, rule-change proposals and blocked payments ("Mei is trying to unlock her tuition. Call her."), with inline **Approve / Deny / Freeze** buttons that call the guardian endpoints. Built and tested against the mock API; reads the live backend URL from `deployments/backend.json` at startup, so it switches over with no extra work. Deployed on the backend host. | Working bot + `bots/telegram/README.md` | 18 Oct, 14:00 |
| S9 | Look-alike address checker (E3) in `packages/shared`: `checkLookalike(address, knownRecipients)` flags "address poisoning" (a new address whose first and last characters match a known recipient's) and returns the plain-English warning "This address looks like Mom's, but it isn't." Includes unit tests with real-world-style poisoned addresses. | Function + tests | 18 Oct, 12:00 |

**Definition of done:** a fresh clone runs `pnpm i && pnpm mock && pnpm --filter frontend dev` and shows the mock-driven app; `packages/shared` tests pass in CI; both hosting projects deploy from `main`; the bot sends a test alert to a phone from the mock event stream.

---

### Rafsan: Blockchain & Red Team Engineer

Works from `spec/IGuardianVault.sol`, Guardian.md §9 and §13. Outputs live in `contracts/`, `deployments/` and `redteam/`.

| # | Task | Deliverable | Due |
|---|---|---|---|
| R1 | `tUSD` test token (ERC20, 6 decimals, deployer mint) | `contracts/src/TestUSD.sol` | 17 Oct |
| R2 | `GuardianVault.sol` exactly per §9: roles, `pay`, recipients with cooling-off, reserve, caps, tighten / propose / approve / cancel / execute, freeze / unfreeze, `setAgent`, `canPay`, custom errors, events. NatSpec comments on every function. | Contract | 17 Oct, 23:00 |
| R3 | Foundry tests: unit tests for every function and error; fuzz tests on `pay`; **invariant tests 1-7** (§9.5); contract-level attacks (§13.2 category F) | `contracts/test/` | 18 Oct, 12:00 |
| R4 | `contracts/test-vectors.json`: 30+ `{state, call, expected result or error}` cases exported from the tests, for the backend's policy mirror | **H3** | 18 Oct, 12:00 |
| R5 | Deploy script: deploy tUSD and the vault to Base Sepolia (demo delays of 120s), mint 5,000 tUSD to the vault, seed recipients (Phone company, Mom, University as beneficiary), verify on Basescan, auto-write `deployments/base-sepolia.json` (addresses, ABI path, deploy block) | **H3** | 18 Oct, 12:00 |
| R6 | `contracts/script/Evidence.s.sol` (or `scripts/evidence.ts`): produces real hashes for a successful payment, a reverted scam payment (explicit gas), a policy proposal, a guardian cancel, and a freeze; writes `deployments/evidence.json` | **H5** | 18 Oct, 18:00 |
| R7 | `contracts/README.md`: how to build, test, deploy; contract addresses; design notes | README | 18 Oct, 18:00 |
| R8 | Attack catalogue: ~40 YAML cases across categories A-F (§13.2), each with payload, channel, goal and success condition | `redteam/attacks/*.yaml` | 18 Oct, 20:00 |
| R9 | Red-team runner: calls the backend API (URL from `deployments/backend.json`), runs every case in **Defences on** and **Jailbreak Mode**, reads vault balances and tx receipts before and after, and writes results | `redteam/run.ts`, one command: `pnpm redteam` | 19 Oct, 14:00 |
| R11 | Symbolic proofs with Halmos for invariants 2, 4, 5 and 7 (E8), running in CI | `contracts/test/halmos/` | 18 Oct, 22:00 |
| R12 | Adaptive attacker (E7): an LLM rewrites attacks that failed against the AI layer, up to 3 rounds; results reported separately | `redteam/adaptive.ts` | 19 Oct, 19:00 |
| R10 | Final red-team run (fixed suite + adaptive round) after the feature freeze; commit `redteam/results.json` + `redteam/results.md` (with explorer links) | **H5** | 19 Oct, 22:00 |

**Definition of done:** CI green; contracts verified on Basescan; `evidence.json` hashes visible on Basescan (success and "Fail"); `results.json` committed and reproducible with `pnpm redteam`.

---

### Mumti: Team Leader · AI Agent, Docs & Submission Lead

Works from `spec/openapi.yaml`, `spec/policy.schema.json`, Guardian.md §8, §10 and §12. Builds against the spec interface until **H3** lands, then switches to the real deployment.

**Part 0: Team setup (first thing on 17 Oct)**

| # | Task | Deliverable | Due |
|---|---|---|---|
| M0a | Send the team registration email to fintechathon@ican-x.com (subject: "International Track Registration - Country - Team Name - Team Leader's Full Name") with every member's details (name, nationality, ID document, university, proof of enrolment, major, year, contact); check the ≥60% non-Chinese rule; ask the organizers to confirm the submission channel and the required file list | Sent email + reply saved in `docs/admin/` | 17 Oct, 12:00 |
| M0b | Create and fund testnet wallets (deployer, agent, owner "Mei", guardian "Mom", attacker) with Base Sepolia ETH from two faucets; store the keys in the team's private secret store (never in git) and share access with all members | **H2** | 17 Oct, 15:00 |

**Part A: Agent backend**

| # | Task | Deliverable | Due |
|---|---|---|---|
| M1 | Backend skeleton (TypeScript, Hono/Express, viem, zod, Anthropic SDK, model `claude-opus-5-5`; see §10.1 for API rules: `auto` tool choice, `strict` tools, structured outputs, refusal handling, server-side fallbacks) | Running server | 17 Oct |
| M2 | `policyMirror`: TypeScript version of `canPay` and the tighten checks; passes 100% of `contracts/test-vectors.json` | Tests green | 18 Oct, 16:00 |
| M3 | Constitution compiler: English → policy JSON, clarifying questions, rule cards generated from JSON, deterministic examples via the mirror, canonical hash (§8.3) | `/api/constitution/*` | 18 Oct, 12:00 |
| M4 | Quarantine reader: no-tools model, fixed schema with red flags (§10.2) | Module + tests | 18 Oct, 14:00 |
| M5 | Planner: tools and behaviors from §10.3 (colloquial input, clarification, correction, fallback explanation, confirm threshold, never executing instructions found in untrusted text) | `/api/chat` | 18 Oct, 20:00 |
| M6 | Signer (agent key only; explicit gas in Jailbreak Mode; plain-English errors via `packages/shared`), indexer + SSE events, demo endpoints (inject scam, Jailbreak toggle, reset, owner and guardian actions) | All endpoints in `openapi.yaml` | 18 Oct, 22:00 |
| M7 | Deploy the backend; write `deployments/backend.json` with the live URL | **H4** | 18 Oct, 22:00 |
| M14 | Extended backend features: Scam Radar data (E1), English/Chinese input and replies with a Mandarin scam message in the demo inbox (E2), look-alike warning via `checkLookalike` from `packages/shared` (E3), rule-change diff text in policy events (E4), decision receipts + `/api/receipts/:id` (E5) | Endpoints per spec | 19 Oct, 14:00 |
| M15 | Recurring payments (E9), switched off if not stable by the freeze | `/api/schedules` | 19 Oct, 19:00 |
| M8 | Run all six storyboard scenes (§5) end to end against the live backend; fix issues until each produces real tx hashes | Checklist in `backend/README.md` | 19 Oct, 18:00 |

**Part B: Documentation and submission**

| # | Task | Deliverable | Due |
|---|---|---|---|
| M9 | Technical documentation: system architecture (diagrams from §7), core algorithms (constitution compiler, policy mirror, vault checks), security design. Use Claude Code to read `contracts/` and `backend/` directly. | `docs/technical.md` + PDF | 20 Oct, 12:00 |
| M10 | Security self-assessment: permission tiers (§12.1), threat mapping (§12.2), known risks (§12.3), red-team results from `redteam/results.md` | `docs/security-self-assessment.md` + PDF | 20 Oct, 12:00 |
| M11 | Execution evidence: tx hash table from `deployments/evidence.json` + red-team logs, each with a Basescan link | `docs/execution-evidence.md` + PDF | 20 Oct, 12:00 |
| M12 | Final root README: setup, deployment instructions, test commands, contract addresses, demo walkthrough | `README.md` | 20 Oct, 14:00 |
| M13 | Submission: collect all six items (Guardian.md §15), check English-only, file sizes and links (make the repo public), submit through the confirmed channel, save the confirmation | Confirmation in `docs/admin/` | **20 Oct, 18:00** |

**Definition of done:** all six storyboard scenes work against the live backend; all submission items submitted with confirmation saved.

---

### Ashfaq: Frontend, Deck & Video Lead

Works from `spec/openapi.yaml` (against the mock API from **H1** on day 1), Guardian.md §5 and §11.

| # | Task | Deliverable | Due |
|---|---|---|---|
| A1 | Next.js + Tailwind + Framer Motion app wired to the mock API | Running app | 17 Oct |
| A2 | `/`: Mei's phone chat, Vault panel (balance, locked tuition countdown, rule cards, recipients on hold with countdowns), live chain feed with Basescan links, labeled Jailbreak Mode switch | Screen | 18 Oct, 14:00 |
| A3 | Full-width red **BLOCKED** animation with the plain-English reason (from `packages/shared`) and a link to the failed transaction | Component | 18 Oct, 14:00 |
| A4 | `/setup`: constitution onboarding (text → rule cards → examples → Confirm → on-chain link) | Screen | 18 Oct, 18:00 |
| A5 | `/mom`: phone-frame view with alerts and Approve / Deny / Freeze buttons | Screen | 18 Oct, 18:00 |
| A6 | `/scoreboard`: big numbers and per-category table from `/api/redteam/results` | Screen | 19 Oct, 12:00 |
| A11 | Extended UI: Scam Radar card next to BLOCKED (E1, E3), receipt drawer from feed items (E5), rule-change diff on `/mom` (E4), Chinese text rendering and subtitles in chat (E2) | Components | 19 Oct, 14:00 |
| A7 | Switch from the mock API to the live backend (`deployments/backend.json`, **H4**); polish for 1080p recording | Live app | 19 Oct, 16:00 |
| A8 | Video script from the storyboard (§5), including on-screen honesty labels (demo delay, Jailbreak Mode, test token) | `docs/video-script.md` | 19 Oct, 12:00 |
| A9 | Presentation deck (≤10 min, English): problem, live-demo stills, how it works (one slide), novelty, security tiers, red-team numbers, prior art, team | `docs/deck.pdf` | 20 Oct, 12:00 |
| A10 | Record the screens after **H5** (final numbers), voiceover, edit; ≤5 minutes; export 1080p MP4 and upload an unlisted YouTube copy | **H6** | 20 Oct, 14:00 |

**Definition of done:** the app runs on the live backend; the video and deck are delivered in `docs/` with numbers matching `redteam/results.json`.

---

## 3. Handover map

There are only seven handovers, each a committed file (or, for H2, shared secret access):

| ID | From → To | File(s) | When |
|---|---|---|---|
| **H1** | Shawki → everyone | `spec/policy.schema.json`, `spec/IGuardianVault.sol`, `spec/openapi.yaml`, `pnpm mock` | 17 Oct, 15:00 |
| **H1b** | Shawki → Mumti, Ashfaq | `packages/shared`, hosting projects, `.env.example` | 17 Oct, 18:00 |
| **H2** | Mumti → everyone | Funded testnet wallets in the secret store | 17 Oct, 15:00 |
| **H3** | Rafsan → Mumti | `deployments/base-sepolia.json`, contract ABI, `contracts/test-vectors.json` | 18 Oct, 12:00 |
| **H4** | Mumti → Ashfaq, Rafsan | `deployments/backend.json` (live backend URL) | 18 Oct, 22:00 |
| **H5** | Rafsan → Mumti, Ashfaq | `deployments/evidence.json`, `redteam/results.json`, `redteam/results.md` | 19 Oct, 22:00 |
| **H6** | Ashfaq → Mumti | `docs/deck.pdf`, `docs/demo.mp4`, YouTube link | 20 Oct, 14:00 |

Until a handover arrives, each person keeps working against the spec: Ashfaq on the mock API, Mumti on the spec interface, Rafsan on a local backend run or the mock API.

---

## 4. Day-by-day schedule

| Day | Shawki | Rafsan | Mumti | Ashfaq |
|---|---|---|---|---|
| **Fri 17 Oct** | S1-S7 (all), **H1**, **H1b** | R1, R2 | M0a, M0b, **H2**, M1, start M3 | A1, start A2 |
| **Sat 18 Oct** | S9 (by 12:00), S8 (by 14:00) | R3-R7, **H3**, R8, R11 | M2-M7, **H4** | A2-A5 |
| **Sun 19 Oct** | - | R9, R12, R10, **H5** (after the 20:00 freeze) | M14, M15, M8, start M9-M11 | A6, A11, A7, A8; feature freeze at 20:00 |
| **Mon 20 Oct** | - | On call for questions about contracts and attacks | M9-M13; **submit by 18:00** | A9, A10, **H6** by 14:00 |

---

## 5. Submission checklist

| # | Item | Owner | Source |
|---|---|---|---|
| 1 | Technical documentation (architecture, core algorithms, security design) | Mumti | `docs/technical.pdf` |
| 2 | Presentation deck (≤10 min) | Ashfaq | `docs/deck.pdf` |
| 3 | Demo video (≤5 min) | Ashfaq | `docs/demo.mp4` + YouTube link |
| 4 | Source code (GitHub link, README, deployment instructions, test cases) | Mumti (README) · repo by Shawki | GitHub |
| 5 | Security self-assessment (permission tiers, known risks) | Mumti | `docs/security-self-assessment.pdf` |
| 6 | Execution evidence (real on-chain transaction hashes) | Mumti, from Rafsan's files | `docs/execution-evidence.pdf` |
| - | Registration confirmed and eligibility checked | Mumti | `docs/admin/` |
| - | Final submission sent, confirmation saved | Mumti | `docs/admin/` |
