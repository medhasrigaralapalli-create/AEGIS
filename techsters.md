# AEGIS — Autonomous Efficiency under Guarded Intervention & Supervision

## 1. Team Details

**Team Name / ID:** Techsters

**Team Lead:** Pratithi Rani Chawla

**Team Members:**
- Pratithi Rani Chawla | Team Lead
- G. Medha Sri
- Shalini Boddana

**Repo Link (Optional):** https://github.com/medhasrigaralapalli-create/AEGIS
**Demo Link (Optional):** N/A

---

## 2. Problem Statement

It is Monday morning; your team receives an alert: "Cloud spending is 37% higher than expected." Nobody knows why. Your company operates several backend services. Each service exposes information such as:

* CPU utilization
* memory utilization
* request count
* response latency
* number of running instances
* current hourly cost
* recent traffic
* availability

The Challenge
Build an autonomous cloud cost-optimization agent that monitors a simulated cloud environment, investigates unexpected spending, chooses safe actions, executes them through APIs, and verifies whether the action actually improved the situation. The agent receives natural-language requests and structured cloud state. AI must play a meaningful role in deciding what to investigate and what action to take; deterministic backend code must enforce safety constraints.

Core Requirements

* Accept a natural-language request together with the supplied service/environment data.
* Inspect service metrics, traffic, health, instance counts, pricing, constraints, and recent events through APIs/tools.
* Choose among available actions such as scale up, scale down, resize, stop an idle service, delay a batch workload, or take no action.
* Respect minimum/maximum capacity, latency, availability, and health constraints.
* Handle stale observations, changing conditions, failed actions, and post-action verification.
* Return a final response explaining the observed problem, action taken or not taken, and verification result.

Test Input A — Cost optimization request
User prompt:
"Review the current services and reduce unnecessary cost without breaking the latency or availability requirements."

services.json:
[
  {
    "service_id": "orders-api",
    "cpu_percent": 22,
    "memory_percent": 41,
    "requests_per_minute": 1200,
    "latency_ms": 180,
    "instances": 6,
    "cost_per_hour": 18.50,
    "min_instances": 2,
    "max_instances": 8,
    "max_latency_ms": 300,
    "healthy": true,
    "timestamp": "2026-09-17T10:30:00Z"
  },
  {
    "service_id": "reports-worker",
    "cpu_percent": 9,
    "memory_percent": 15,
    "requests_per_minute": 0,
    "latency_ms": 0,
    "instances": 4,
    "cost_per_hour": 11.00,
    "min_instances": 1,
    "max_instances": 6,
    "max_latency_ms": 900,
    "healthy": true,
    "timestamp": "2026-09-17T10:30:00Z"
  }
]

Test Input B — Rising traffic
User prompt:
"Orders traffic is increasing. Keep the service within its latency target."

service.json:
{
  "service_id": "orders-api",
  "cpu_percent": 28,
  "memory_percent": 48,
  "requests_per_minute": 4200,
  "previous_requests_per_minute": 2100,
  "latency_ms": 260,
  "instances": 4,
  "cost_per_hour": 18.50,
  "min_instances": 2,
  "max_instances": 8,
  "max_latency_ms": 300,
  "healthy": true,
  "timestamp": "2026-09-17T10:30:00Z"
}

Test Input C — Stale observation
User prompt:
"Reduce cost if it is safe."

metric.json:
{
  "service_id": "checkout-api",
  "cpu_percent": 24,
  "memory_percent": 39,
  "requests_per_minute": 900,
  "latency_ms": 170,
  "instances": 5,
  "cost_per_hour": 20.00,
  "min_instances": 2,
  "max_instances": 8,
  "max_latency_ms": 250,
  "healthy": true,
  "timestamp": "2026-09-17T08:00:00Z"
}

latest_traffic.json:
{
  "service_id": "checkout-api",
  "requests_per_minute": 5200,
  "timestamp": "2026-09-17T10:30:00Z"
}

Test Input D — Failed action
User prompt:
"Scale the payment service only if the current state requires it."

service.json:
{
  "service_id": "payment-api",
  "cpu_percent": 91,
  "memory_percent": 82,
  "requests_per_minute": 6400,
  "latency_ms": 410,
  "instances": 3,
  "cost_per_hour": 22.00,
  "min_instances": 2,
  "max_instances": 8,
  "max_latency_ms": 300,
  "healthy": true,
  "timestamp": "2026-09-17T10:30:00Z"
}

action_result.json:
{
  "action_id": "act-784",
  "action": "scale_up",
  "requested_instances": 5,
  "status": "failed",
  "error": "capacity_unavailable"
}

---

## 3. TL;DR

**Problem:** Cloud bills quietly balloon from idle or over-provisioned services, and nobody has time to audit them.

**Solution:** An agent that finds wasted spend, predicts the impact of fixing it, and acts in small, reversible, verified steps.

**Who benefits:** Engineering and finance teams get lower bills without the risk of an AI-caused outage.

---

## 4. Scope of the Project

**What are you building?**
An autonomous agent that continuously reviews service metrics, identifies wasteful spend (idle instances, over-provisioning, duplicate capacity), proposes a scaling action, checks it against safety rules and a latency/cost prediction model, executes it if safe, then watches the outcome and automatically reverts if it made things worse.

**How does it solve the problem statement?**
It closes the full loop the statement asks for: investigate → decide → act via API → verify. It treats stale data, constraint limits, and failed actions as first-class cases instead of edge cases.

**Key features you're building for this hackathon:**
- Metric-based waste detection across multiple services (idle, over-provisioned, duplicate load)
- Observation freshness check that blocks decisions made on stale data
- Pre-action impact prediction (estimated latency/cost after the change) before any action fires
- Deterministic safety envelope (min/max instances, blast-radius limit, protected services) that gates every AI proposal, with automatic self-correction to a smaller safe step when a proposal is rejected
- Full action range beyond scale up/down — resize (vertical, for memory/CPU-bound services), stop (for idle, explicitly stoppable workloads), and delay a batch workload (peak-pricing jobs) — each with a post-action watch window that rolls back if the outcome doesn't hold

**What are you deliberately NOT doing? (Optional)**
Not integrating a real cloud provider (AWS/GCP/Azure) — using a simulated service/environment API as the problem statement allows. Not building multi-cloud, multi-region, or multi-service concurrent blast-radius coordination.

---

## 5. Why an Agentic Approach?

**What does your agent decide or do on its own?**
It decides which services to investigate first, which of five actions fits the situation (scale up/down, resize, stop, delay a batch job, or no-op), whether its own data is trustworthy enough to act on, and whether to roll back an action it already took. If the safety gate rejects a proposal for being too large, it reasons about why and retries with the largest safe step on its own — a self-correction loop, not a retry with the same input.

**Why wouldn't a fixed script, if-else rules, or a simple chatbot be enough?**
A fixed script can't reason about competing signals (low CPU but rising traffic, stale vs fresh data, one failed action needing a different fallback) or explain its reasoning in plain language. The decision space is conditional and combinatorial — rules would multiply endlessly and still miss cases like partial data or unexpected constraint conflicts.

---

## 6. Who It's For & What Changes

**Who or what is this for?**
Engineering teams and the finance/FinOps function at any company running backend services in the cloud.

**The world today, without your solution:**
Engineers discover cost spikes reactively, after finance flags a bill. Investigating which service is responsible and whether it's safe to change takes hours of manual dashboard digging, and changes are made cautiously, if at all, because nobody wants to be the one who caused an outage.

**The world with your solution, fully built and scaled to production:**
The agent runs continuously in the background, catching waste the same day it appears, acting on low-risk cases automatically, and surfacing only the genuinely ambiguous decisions to a human — with every action logged, predicted, and verified.

**What your hackathon build actually delivers today:**
A working pipeline on simulated service data that detects waste, predicts the effect of a proposed action, checks it against hard safety limits, executes it against a mock API, and reverts automatically if the post-action watch window shows things got worse.

**Before vs. After**

| What Changes | Today | With Our Current Build | At Production Scale |
|---|---|---|---|
| Time to detect idle/wasteful service | Days, found manually | Under a minute, automatic scan | Continuous, real-time |
| Confidence to act without a human | Low — fear of outages | Actions gated by prediction + rollback | Autonomous for low-risk, escalated for high-risk |
| Recovery from a bad scaling decision | Manual, minutes to hours | Automatic revert within the watch window | Automatic, sub-minute |
| Traceability of "why was this changed" | Rare, undocumented | Full log: hypothesis, prediction, outcome | Full audit trail for compliance |

---

## 7. Architecture & Agents

**How is your system put together?**
A user (or scheduler) sends a natural-language request plus service state. An Investigator Agent reads metrics and flags candidate services. A Decision Agent proposes an action with a predicted outcome. A deterministic Safety Gate checks the proposal against hard rules before anything is executed. An Executor calls the (mocked) cloud API, and a Verifier Agent watches results and triggers rollback if needed.

### 7.1 Agents

- **Investigator Agent:** Reads service metrics (CPU, memory, traffic, latency, cost, recent deployment events) and checks how fresh each observation is; flags services worth acting on and holds off right after a recent deployment. Talks to the Service Metrics API and the Decision Agent.
- **Decision Agent:** Given a flagged service, proposes one of five actions (scale up, scale down, resize, stop, delay a batch workload, or no-op) with a predicted latency/cost outcome and a rollback condition. Runs on transparent rule-based heuristics by default for reproducible demos; switches to a real Claude Sonnet 4.6 call automatically when an API key is set, with no other code changes needed. Talks to the Safety Gate.
- **Verifier Agent:** After execution, re-reads metrics during the watch window and decides whether the outcome matches the prediction or needs a rollback. Talks to the Executor and the Regret Ledger.

### 7.2 Services, APIs, Databases & Memory

- **Service Metrics API (mocked):** Serves CPU, memory, traffic, latency, instance count, and cost per service, with timestamps. Used by the Investigator Agent.
- **Safety Gate (deterministic Python module):** Enforces min/max instances, blast-radius limit (one tier-1 service per window, max % capacity change), and protected-service rules. Used by the Decision Agent's output before execution.
- **Executor (mocked scale/resize/stop API):** Applies the approved action and returns success or a failure reason (e.g. capacity_unavailable). Used by the Safety Gate once an action passes.
- **Regret Ledger (SQLite):** Stores every hypothesis, prediction, actual outcome, and whether a rollback occurred. Used by the Verifier Agent and shown in the demo UI.
- **Web Dashboard (Streamlit or simple React page):** Shows current service state, proposed actions, and the action log. Used by the user/judges.

**How does your system remember things (memory & state)?**
Every action (proposal, prediction, execution result, verification outcome) is written to the Regret Ledger (SQLite), so the agent can reference past decisions on the same service and judges can review the full history during the demo.

**Diagram Link (Optional):** N/A

### 7.3 Example Walkthrough

**Example input:** "Reduce cost if it is safe" for checkout-api, where the last known metrics are 2.5 hours old and a fresher traffic feed shows a 5.7x spike.

1. [Web Dashboard] Sends the request and both data snapshots to the Investigator Agent.
2. [Investigator Agent] Checks observation freshness (uses: Service Metrics API), finds the primary snapshot is stale, and flags it as untrustworthy.
3. [Investigator Agent] Passes the fresher traffic feed to the Decision Agent instead.
4. [Decision Agent] Proposes "scale up," not down, predicting latency would breach the SLO on the stale reading's assumption otherwise.
5. [Safety Gate] Checks the proposal against min/max instance limits and blast-radius rules; approves it.
6. [Executor] Calls the mocked scale API and returns success.
7. [Verifier Agent] Watches metrics for the next interval, confirms latency stayed within target, and logs the outcome (uses: Regret Ledger).

**Final output:** A plain-language explanation that stale data was rejected, the service was scaled up instead of down, and the change was verified safe — with the full reasoning trail logged.

**A second walkthrough — self-correction and the `stop` action:** Input: an idle background worker (`email-digest-worker`, 0 requests/min, 3% CPU, marked as a stoppable batch workload) plus a separate cost-cleanup request.

1. [Investigator Agent] Confirms the data is fresh; sees 0 traffic and near-zero CPU.
2. [Decision Agent] Because this service is a stoppable batch workload (not a realtime API), it proposes `stop` (0 instances) rather than just scaling to a floor, since nothing needs it responding live.
3. [Safety Gate] Approves it — `stop` is only permitted when the service is explicitly marked stoppable; a realtime service would be blocked here and forced through the normal scale-down path instead.
4. [Executor] Sets the service to 0 instances.
5. [Ledger] Logs the hypothesis, the action, and the rollback condition ("restart if traffic returns or a scheduled run is due").

**Final output:** "email-digest-worker is a stoppable batch workload showing 0 req/min — stopped entirely rather than scaled to a floor, since nothing depends on it responding live."

**A third example — the self-correction retry (visible in Test Input A):** The agent's first idea for `reports-worker` is to scale straight to the minimum floor. The Safety Gate rejects that as too large a single change (blast-radius limit). Instead of stopping there, the agent computes the *largest step it's still allowed to take*, retries with that smaller number, gets approved, executes, and verifies — all in one pass, with the rejection and retry both shown in the output rather than hidden.

**Anything special about how your workflow runs? (Optional)**
Every proposed action must pass a deterministic Safety Gate before execution — the LLM never calls the cloud API directly. If a proposal is rejected for exceeding the blast-radius limit, the pipeline automatically retries once with the largest safe step in the same direction before giving up — a ReAct-style propose → reject → reason → retry loop, not a single-shot decision. If a post-action watch window shows the outcome missed its prediction (e.g. latency rose instead of held), the Verifier Agent triggers an automatic rollback without asking the LLM again.

---

## 8. Tech Stack

| Layer | Technology |
|---|---|
| Frontend / Interface | Streamlit — fastest way to show live metrics, proposals, and the ledger without custom frontend work |
| Backend | Python (FastAPI) — async, typed, easy to wire to both the LLM calls and the mocked service API |
| Agent Framework | LangGraph — gives explicit state-machine control between Investigator → Decision → Safety Gate → Verifier, which matters here since the Safety Gate must sit as a hard, non-LLM node in the graph, not a soft prompt instruction |
| Reasoning Model (Investigator + Decision Agents) | Claude Sonnet 4.6 — needed for the actual hard reasoning: weighing conflicting signals (low CPU but rising traffic, stale vs fresh data), predicting latency impact, and writing a defensible hypothesis. This is the step accuracy matters most on, so use the strongest model here |
| Verification Model (Verifier Agent) | Claude Haiku 4.5 — the verification task is narrow (compare predicted vs actual outcome against a threshold), so a faster, cheaper model is accurate enough and keeps the watch-window loop cheap to run repeatedly |
| Database / Storage | SQLite — simple, zero-setup, enough for the Regret Ledger and action history in a hackathon build |
| Hosting | Local machine for development; Render or Railway for a live demo link if needed |
| Other | Mocked cloud/service API (JSON-driven), matching the problem statement's simulated environment |

---

## 9. What to Expect From Our Current Build

**Working:**
- Waste detection across multiple mocked services, including full action range (scale up/down, resize, stop, delay batch workload)
- Freshness check that blocks stale-data decisions and switches to a fresher supplied observation when given one
- Safety Gate enforcing min/max instances, blast-radius limit, SLO breach prevention, and protected-service rules
- Self-correction: an oversized proposal rejected by the Safety Gate is automatically retried at the largest safe step, with the retry visible in the output, not hidden
- Failed-action handling: a forced `capacity_unavailable` error triggers an automatic fallback to a smaller achievable step
- Post-action watch window that compares predicted vs. observed latency and rolls back on an SLO breach
- Recent-deployment-event awareness: the agent holds off on scaling a service that was just deployed
- Full input-to-output dashboard (Streamlit) showing raw service metrics alongside every reasoning step

**Partly working, mocked, or hard-coded:**
- Decision Agent runs on rule-based heuristics by default (fully reproducible for judges); switching to a real Claude Sonnet call is a one-line environment variable change, already wired in
- Cloud API calls are mocked against sample JSON, not a real provider
- Latency prediction uses a simplified queueing formula, not a trained model
- The `stop` / `resize` / `delay_batch_workload` actions are demonstrated on three additional synthetic services we built, since the official four test inputs don't naturally call for those specific actions

**Not working or not built yet:**
- Multi-service concurrent action coordination / shared blast-radius budget across services
- Real-time streaming metrics (current build uses periodic polling)
- Savings-attribution against a no-intervention baseline (see Future Scope)

**What we'd most like to be judged on:**
The self-correction loop — a proposal gets rejected by the deterministic Safety Gate, the agent reasons about *why*, and retries with a smaller safe step instead of giving up — visible directly in the dashboard output, alongside the full five-action range (not just scale up/down).

---

## 10. Future Scope

**Idea 1**
Name: Real cloud provider integration
What it is: Connect the Executor to a real AWS/GCP account via their SDKs instead of a mocked API.
Why it matters: Proves the safety model holds under real infrastructure constraints and pricing.
How we'd build it: Swap the mocked Executor module for boto3/GCP SDK calls behind the same Safety Gate interface.
Done when: The agent scales a real test instance up/down and the action shows up in the provider's console.

**Idea 2**
Name: Learned safety margins
What it is: Use the Regret Ledger history to automatically tighten or loosen per-service safety thresholds over time.
Why it matters: The agent gets measurably more accurate and cautious specifically where it has been wrong before.
How we'd build it: Periodic job that aggregates ledger outcomes per service and adjusts Safety Gate parameters.
Done when: A service with past rollback events shows a stricter blast-radius limit than one with none.

**Idea 3 (Optional)**
Name: Savings attribution
What it is: Compare observed cost after an action to a forecast of what cost would have been without intervention.
Why it matters: Prevents the agent from claiming credit for savings caused by external traffic drops.
How we'd build it: Simple baseline forecast model (e.g. moving average) run alongside actual post-action cost.
Done when: The dashboard shows "attributed savings" separately from raw cost delta, with a confidence label.

---

## 11. Additional Notes (Optional)

N/A
