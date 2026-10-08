---
name: consult-advisor
description: Consult GPT-6 Astra when the user explicitly authorizes advisor use in a prompt, including "consult-advisor is authorized if useful". Within that scope, decide whether consultation is useful.
---

# Consult the advisor

Activate only when the user explicitly authorizes advisor use in a prompt. This
includes a direct request such as "consult the advisor" and discretionary permission
such as "the consult-advisor skill is authorized if useful" or "use $consult-advisor
when helpful". Permission may be given in the initial prompt and remains available
throughout that task unless the user narrows or revokes it.

With discretionary permission, the primary decides whether and when consultation
would help with consequential or ambiguous decisions, competing approaches, recurring
failures, strategy changes, contradictory evidence, or substantial completion reviews.
No additional user judgment or confirmation is needed for a consultation within the
authorized scope. Keep routine edits and factual lookups in the primary agent.
These situations do not authorize consultation on their own. A request to edit or
discuss this skill is not authorization to consult.

Within the authorized scope, this skill authorizes delegation to one independent
advisor. The primary owns implementation and reconciliation.

## Check before delegating

- Confirm the user explicitly authorized advisor use and keep it within that scope.
  If authorization is discretionary, consult only when it would materially help.
  The primary's model and reasoning level do not affect activation; do not inspect
  them or report missing primary-model metadata as part of this workflow.
- Confirm the available delegation tool can select the exact model and effort. If
  Astra, high reasoning, or delegation is unavailable, report that clearly and continue
  the primary work; never silently substitute another model or build an API/MCP service.
- Briefly announce the consultation and its precise purpose.

## Handoff and invocation

Give the advisor a self-contained brief containing:

1. Objective and the consequential decision or review boundary.
2. Latest user decisions, scope, constraints, and approval boundaries; label settled
   decisions separately from unanswered recommendations.
3. Relevant observations, failures, test results, and file/source references, including
   evidence against the preferred approach.
4. Plausible alternatives, uncertainties, and the precise question to answer.

Request concise guidance with reasoning, assumptions, material risks, missing
evidence, references where relevant, and a practical next step. Permit targeted
read-only evidence gathering when the brief is insufficient. Prohibit file edits,
implementation, mutating commands/tests, commits, pushes, external communication,
external service mutations, permission escalation, and all further delegation.

Use the client adapter matching the actual tool schema:

- **Native custom-agent selection:** spawn the personal agent named `advisor`
  (typically `agent_type = "advisor"`). Its user-level TOML pins `gpt-6-astra` and
  `high`, defaults to `read-only` with no approvals, and disables delegation. Prefer
  a fresh context (`fork_turns = "none"`, or `fork_context = false`, according to
  the exposed schema) and send the complete brief.
- **Collaboration tool exposing model overrides but no custom-agent selector:** when
  local files are accessible, read the `developer_instructions` from
  `$CODEX_HOME/agents/advisor.toml`, or `~/.codex/agents/advisor.toml` when CODEX_HOME
  is unset, and include them with the brief. If the file is inaccessible (for example,
  a hosted chat), disclose that and include this skill's consultant restrictions,
  handoff, and response requirements instead. Use `model = "gpt-6-astra"`,
  `reasoning_effort = "high"`, and
  `fork_turns = "none"` (or a supported positive integer string when recent context
  helps). Never combine model overrides with omitted/`"all"` full-history forks.
  The TOML configuration is NOT applied through this adapter: permissions and the
  ban on further delegation are instruction-only unless the host independently
  enforces them. Disclose that difference; do not claim a read-only sandbox.

Inspect effective child permissions if metadata is available. Native agent sandbox
defaults can be overridden by parent live permission settings; when this occurs,
the non-mutation restrictions remain instructions rather than an enforced read-only
boundary. Never weaken the parent sandbox to obtain evidence. Read-only shell
permissions do not themselves prohibit connector writes or external messages.
This is a Codex subagent consultation with a selected brief and optional reads, not
Claude's server-side, tool-free advisor receiving the entire transcript.

## Reconcile and stop

Treat the answer as an independent recommendation, not authorization. Reconcile it
with user decisions and observed evidence; state any material disagreement and why
you accept or reject the recommendation. Use at most one focused follow-up on the
same advisor for a material conflict or missing evidence. Reconsult only when new
evidence or a changed decision justifies it; avoid repetitive reviews. Resume primary
work and report meaningful effects on the plan or conclusion. Report model errors
or rerouting as an unavailable exact advisor, never as a successful Astra consultation.
