# Codex Advisor

A reusable consultation workflow for local Codex coding sessions. The primary agent
handles implementation and routine work; an independent **GPT-6 Astra** advisor at
**high reasoning** helps with consequential decisions, difficult debugging,
competing approaches, conflicting evidence, and substantial completion reviews.

The workflow consists of a custom agent and an instruction-only skill. It needs no
new dependencies, API service, or MCP server, and does not change the primary model
or the defaults for other subagents.

## Files

The `codex/` directory contains the files to copy into your Codex installation.

```text
codex/
├── agents/
│   └── advisor.toml
└── skills/
    └── consult-advisor/
        ├── SKILL.md
        └── agents/
            └── openai.yaml
```

`advisor.toml` pins the model and reasoning level, defines the consultant role,
defaults to a read-only sandbox with approvals disabled, and disables further
delegation. `SKILL.md` defines when to consult, how to hand off context, how to
reconcile advice, and how to adapt to the available delegation tool. `openai.yaml`
provides display metadata and permits implicit skill selection.

This tree mirrors the personal installation under `~/.codex`. Current documentation
uses `.agents/skills` for repository skill discovery; see the repository-scoped
installation below rather than assuming the distribution tree alone activates a
skill in every client.

## Personal installation

From the root of this repository:

```bash
advisor_install_root="${CODEX_HOME:-$HOME/.codex}"
mkdir -p "$advisor_install_root/agents" "$advisor_install_root/skills"
cp -i codex/agents/advisor.toml "$advisor_install_root/agents/"
cp -Ri codex/skills/consult-advisor "$advisor_install_root/skills/"
```

Review any overwrite prompt when updating an existing installation. This copies
only the advisor files and leaves `config.toml` alone. The `~/.codex/skills` personal
location was verified with Codex CLI 0.159.2; current documentation also lists
`~/.agents/skills` as a user skill location.

Start a fresh local session and check `/skills` for `consult-advisor`. Skill changes
can be detected automatically; restart the client if discovery does not update.
No daemon restart was needed for the validated fresh CLI sessions. A custom
CODEX_HOME may require the corresponding client-specific discovery check.

## Use

Explicit requests include:

```text
Consult the advisor about these competing implementation approaches.
Use $consult-advisor to assess the recurring failure before changing strategy.
Use $consult-advisor to review the completed work against the user decisions.
```

The primary can also select the skill implicitly when its description applies.
The skill explicitly requests delegation, so it supplies the delegation instruction
required by current local Codex. Selection is discretionary; routine edits, factual
lookups, and individual tool calls do not require consultation.

Before consulting, the primary checks active session metadata. It skips delegation
when the primary is already `gpt-6-astra` at `high`, `xhigh`, `max`, or `ultra`, even
on explicit skill invocation. If active settings cannot be determined, it states
that limitation. If the exact advisor model, effort, or delegation mechanism is
unavailable, it reports this and continues the primary work without substituting
another model.

## Activate it for a project

With the personal installation available, add this to the project's root
`AGENTS.md`:

```markdown
- Use the `consult-advisor` skill when its scope applies, and delegate a bounded
  consultation as instructed by that skill. Keep routine work in the primary
  agent and follow the skill's Astra/high skip rule.
```

This is an ordinary instruction, not a special activation command. It requests
selective consultation while leaving the skill in charge of the handoff and skip
rule.

For a self-contained project installation, copy the agent to
`<project>/.codex/agents/advisor.toml` and the entire skill directory to
`<project>/.agents/skills/consult-advisor/`, then use the same root `AGENTS.md`
instruction. Project skill and agent discovery should be verified in a fresh session.
Avoid installing duplicate skill copies in both user and project locations unless
you intend to manage both; clients may list both instead of merging them.

A `.codex/agents/advisor.toml` file defines an available role; its presence alone
does not require the primary to consult it. `.codex/config.toml` can enable agents
with `[agents] enabled = true` if they have been disabled, but this is already the
documented default and does not itself define consultation triggers. Put the
activation instruction in the root `AGENTS.md`, not an assumed auto-loaded
`.codex/AGENTS.md`.

## Handoff and reconciliation

The primary announces the consultation and supplies the objective, latest user
decisions, constraints, relevant evidence, alternatives, uncertainties, and a
precise question. The brief preserves contrary evidence and separates settled
decisions from unanswered suggestions. The advisor may inspect relevant files
when the brief is insufficient.

The advisor returns concise guidance with reasoning, assumptions, material risks,
missing evidence, and file or source references where useful. Advice is a
recommendation, never approval to override user decisions, scope, or dependency
restrictions. The primary reconciles it with the evidence and may ask one focused
follow-up if a material conflict remains.

## Client and permission limits

- **Native custom-agent selector:** use the role named `advisor` in a fresh context.
  Its TOML provides model, effort, and permission defaults. Parent live permission
  overrides can replace the sandbox defaults; inspect effective child permissions
  when available.
- **Model-override collaboration tool without a custom-agent selector:** explicitly
  select `gpt-6-astra` and `high`, include the consultant instructions, and use
  `fork_turns="none"` or a supported positive integer string. Full-history forks
  cannot be combined with model overrides in the validated tool. This adapter
  does not apply the TOML and inherits parent permissions. It reads the personal
  agent file when accessible; a project-only installation does not supply that
  personal path, so the skill instead includes its own consultant restrictions.
- **Instruction restrictions:** no implementation, file edits, mutating commands or
  tests, commits, pushes, external messages, external service changes, permission
  escalation, or further delegation. A shell read-only sandbox does not itself
  prevent connector writes; those restrictions also require instructions.
- **Desktop:** current OpenAI documentation supports standalone skills and local
  custom agents. Desktop discovery, model selection, and invocation were not tested
  because no desktop client was available on the validation host.
- **Hosted ChatGPT:** local files are not discovered. The shared skill must be made
  available through a supported UI or plugin mechanism, and exact model and effort
  selection require corresponding delegation controls. Hosted support is untested.

This is a Codex subagent consultation with a selected brief and optional evidence
reads. It does not reproduce Claude's server-side, tool-free advisor receiving the
entire transcript.

## Validation

The personal workflow was validated on 2026-09-30 with Codex CLI 0.159.2 and a local
IDE session using synthetic, non-mutating exercises:

| Check | Result |
| --- | --- |
| Agent TOML and skill validation | Passed parsing and the bundled skill validator. |
| Fresh CLI skill discovery | `consult-advisor` appeared in the available skill catalog. |
| Explicit consultation | Native `advisor` child metadata confirmed `gpt-6-astra/high`, read-only permissions, and approvals `never`. |
| Evidence and returned guidance | The advisor read a retry-policy fixture, considered contrary evidence, and preserved every settled constraint. |
| Implicit consultation | A substantial completion-review request naming neither advisor nor skill selected the skill and spawned the native advisor. |
| Redundant escalation | An Astra/high primary verified active metadata and spawned no advisor. |
| IDE model-override adapter | Confirmed Astra/high and useful referenced guidance; permissions remained inherited workspace-write. |

No material conflict needed a follow-up. Unavailable-model handling, deliberate
permission-denial behavior, and attempted enforcement of the native delegation ban
were not separately exercised. No test advisor modified project files or services.
These are prior installation results, not a guarantee for every client or account.

To validate the skill after changing these files, use the `quick_validate.py` script
provided by the installed `skill-creator` skill. Parse the agent with Python's
`tomllib` or the client's configuration loader. After installation, check discovery
and run a small synthetic consultation; verify child model and effort from runtime
metadata rather than the advisor's self-report.

## References

- [OpenAI subagents and custom-agent configuration](https://learn.chatgpt.com/docs/agent-configuration/subagents?surface=app)
- [OpenAI skill discovery and invocation](https://learn.chatgpt.com/docs/build-skills)
- [OpenAI configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Claude Code advisor workflow](https://code.claude.com/docs/en/advisor)
- [Claude server-side advisor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool)

## License

Distributed under the CC0 1.0 Universal Public Domain Dedication. See LICENSE for details.
