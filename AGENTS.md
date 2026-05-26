# Coding Agent Guidelines

Reflex: Python web **framework** compiling to React. Monorepo using uv workspace — main package in `reflex/`, sub-packages in `packages/`, docs site in `docs/`.

## Workflow

1. **Plan first.** Ensure the task is well-defined before writing code. If unclear, work with the user to flesh out details. No sloppy/spaghetti code — every feature/fix must be clearly understood first.
2. **Bugfixes:** write a regression test that fails before writing the fix.
3. **After implementation:** act as an adversarial reviewer. Scrutinize the diff against all rules in this file. Call out numbered issues, then wait for the user to request followup changes.

## Commands

Use `uv` for everything — never bare `python` or `python3`.

```
uv sync                                                          # install deps
uv run pytest tests/units --cov --no-cov-on-fail --cov-report=   # unit tests (>=72% coverage)
uv run pytest tests/integration                                  # integration tests (slow)
uv run ruff check .                                              # lint
uv run ruff format .                                             # format
uv run pyright reflex tests                                      # type check
uv run python scripts/make_pyi.py                                # regenerate .pyi stubs
uv run pre-commit run --all-files                                # all pre-commit hooks
```

## Layout

```
reflex/                 # main framework package (app, state, compiler, components, utils, istate)
packages/               # workspace sub-packages (reflex-base, reflex-components-*, reflex-docgen, reflex-components-internal)
tests/units/            # unit tests, mirrors source tree
tests/integration/      # Selenium integration tests (run in dev+prod modes)
  tests_playwright/     # Playwright integration tests (preferred for new tests)
tests/benchmarks/       # performance benchmarks
docs/                   # documentation site (separate workspace member)
```

## Code style

- Concise, robust code. Reflex is a framework used in many ways — handle edge cases without unnecessary complexity.
- Performance matters. Avoid suboptimal patterns (e.g. iterating a dict to find a value by identity). Suggest restructuring data/APIs if an operation can't be done efficiently.
- Don't add expensive workarounds (e.g. `isinstance` checks) to paper over type-level problems — fix the root cause instead.
- Don't repeat validation or be over-defensive; trust data that was already validated upstream.
- Think in CPU cycles: avoid unnecessary data copies, redundant allocations, and gratuitous indirection.
- Extract duplicated code into parameterized helpers.
- No block comments (`# --- Section ---`, `# ============`). Plain inline comments only.
- Be cautious creating new public APIs — they must be documented and supported long-term.
- Google-style docstrings on all functions: one-line summary, optional detail sentence(s), then Args/Returns (or Yields)/Raises.
- Prefer imports at the top of the module in isort order. Only use inline imports when necessary to avoid circular dependencies.

## Testing

- Write comprehensive tests for new/changed features; extend existing test files where possible.
- Test functions at module level, not wrapped in classes.
- **Unit tests:** `tests/units/`, run with `uv run pytest tests/units`.
  - unit tests should primarily cover a single module, and should be named accordingly, including subdirectories (e.g. `tests/units/istate/test_manager.py` for `reflex/istate/manager.py`). For subpackages, also include the corresponding path below `src/` (e.g. `tests/units/reflex_base/event/test_context.py` for `packages/reflex-base/src/reflex_base/event/context.py`).
- **Integration tests:** prefer Playwright (`tests/integration/tests_playwright/`). Integration tests are slow — extend existing test apps rather than creating new ones for trivial functionality. Multiple test cases sharing one app is fine.

### Integration test patterns

Apps as factory functions, run via `AppHarness`:

```python
def SomeApp():
    import reflex as rx

    class State(rx.State):
        value: str = ""

    def index():
        return rx.box(rx.text(State.value))

    app = rx.App()
    app.add_page(index)


@pytest.fixture(scope="module")
def some_app(tmp_path_factory) -> Generator[AppHarness, None, None]:
    with AppHarness.create(
        root=tmp_path_factory.mktemp("some_app"), app_source=SomeApp
    ) as harness:
        yield harness
```

Playwright tests use the `page` fixture and navigate to `harness.frontend_url`. Utilities in `tests/integration/utils.py` (polling, event ordering, storage).

## .pyi stubs

When adding/modifying components: `uv run python scripts/make_pyi.py`. Commit `pyi_hashes.json` (not `.pyi` files). If the diff removes many modules, run `uv sync`, delete `.pyi_generator_last_run`, and regenerate.

## Breaking changes and deprecation

Reflex has downstream users — don't break them. Provide a fallback path during deprecation.

**Runtime warning** via `console.deprecate()`:
```python
from reflex_base.utils import console

console.deprecate(
    feature_name="OldFeature",
    reason="Use NewFeature instead.",
    deprecation_version="<next dot version of latest git tag>",
    removal_version="1.0",
)
```
Set `deprecation_version` to the next dot version of the latest tag (`git fetch --tags` if needed, e.g. tag `v0.7.3` -> `"0.7.4"`). Set `removal_version` to next major unless directed otherwise.

**Type-level deprecation** for deprecated methods/overloads using `typing_extensions.deprecated`, always inside a `TYPE_CHECKING` guard to avoid double warnings:
```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from typing_extensions import deprecated

    @deprecated("Use new_method() instead")
    def old_method(self) -> str: ...
```

## Checklist

Before submitting:
1. Tests pass with adequate coverage
2. `uv run ruff check .` and `uv run ruff format .` clean
3. `uv run pyright reflex tests` passes
4. `pyi_hashes.json` updated if components changed
5. Documentation updated if user-facing behavior changed
6. Deprecation warnings added if breaking changes introduced


<!-- steward:start -->
## Steward MCP — Agent Workflow

Steward is the fact provider. You are the reasoning engine.
`steward-arch.toml` = engineer-confirmed architectural truth (layers, rules, overrides — written by MCP tools only).
`steward.toml` = user-editable project config (project name, root, policy).
`steward_out/claude-suggestions.json` = your pending reasoning.

### Hard Rules

1. **Session start** — call `session_start` before ANY other action. No exceptions. If `[layers]` is empty, call `start_bootstrap` instead of `session_start`. The individual tools (`get_architecture_map`, `get_health_pulse`, `start_refresh`) are still available for mid-session use.

2. **Three-tier context protocol** — NEVER use Read/Bash/grep to understand what code does. Query in tier order:
   - **Tier 0 — Orientation:** `get_project_atlas` — folder-level map + layer counts. ~1KB. Use when starting in an unfamiliar area or after a major refactor. Not in a normal edit flow.
   - **Tier 0.5 — Module Zoom:** `zoom_to_module(module)` — all symbols in a module with risk scores, purity, entropy, and violation counts. No source peek. Use when you know which module a task lives in but not which specific symbol. Bridges atlas orientation (Tier 0) → symbol inspection (Tier 2) without the broad fuzzy match of Tier 1. Skip for single-file tasks where you already know the symbol.
   - **Tier 1 — Scoping:** `get_context_bundle(task="...")` — all matched files + signatures + logic profiles + violations in one call. Use at the start of any multi-file task. Cap `max_files=5` for single-feature work. Skip for single-function edits — go directly to Tier 2.
   - **Tier 2 — Operation:** `inspect_neighborhood(symbol)` — signature, wiring, patterns, raises, depth_risk, 10-line source peek. Call for every function being modified, not just the entry point.

   - **Tier 3 — Direct Inspection (bug-hunt only):** Read is permitted on a specific file **only when all three gates are met**: (1) the task is explicitly a correctness bug audit ("find all bugs", "what's broken in this file"), not architecture/planning/editing; (2) Tier 1 or Tier 2 has already been called and returned the file as relevant; (3) `inspect_neighborhood` was called on the key symbols and the 10-line peek was insufficient to confirm or rule out the suspected bug. Read only the files Tier 1/2 identified — do not expand scope. After reading, if a bug is found, write it as a suggestion via `write_claude_suggestions` before fixing anything.

   **Never skip tiers upward** — Tier 1 → Tier 2, never Tier 1 → raw Read. Tier 3 is a last resort for bug-hunt tasks only, not a shortcut for any other purpose.
   **For performance/concurrency/memory questions:** call `get_logic_profile` on the primary file AND all its direct callees. A `global_state` or `concurrent_futures_pool` in a callee is invisible if you only profile the primary file.

   **`steward_out/` files are never valid Read targets.** `dense-map.json`, `call-graph.json`, and all other generated artifacts are the backing store for MCP tools — reading them directly bypasses the tier system and wastes tokens parsing raw JSON. If the engineer asks about a `steward_out/` file that doesn't exist, do not fall back to reading an equivalent file. Instead redirect: "That file isn't in steward_out/ — I'll use `zoom_to_module` / `inspect_neighborhood` / `check_impact` to get the same data." If a subagent was spawned to extract data from a file and the file doesn't exist, do not re-spawn with a different approach — call the equivalent MCP tool directly.

3. **The Sandwich Rule** — call `inspect_neighborhood(symbol)` before editing any named function. Do not edit based on signatures alone. A signature without callers, depth_risk, and patterns is half the picture.

4. **Impact gate** — before changing any symbol where `depth_risk=HIGH` or `impact_radius=High`, call `check_impact(changes=[symbol])` first. If the blast radius is unexpected, surface it to the engineer before proceeding.

5. **Modifying files** — ALWAYS call `pre_edit_bundle(filepath, planned_imports=[...])` before editing any file with planned imports. For skeleton-only checks use `get_interface_skeleton`; for single imports use `validate_import`. If any import check returns `warn` or `violation`, state it explicitly — never silently ignore it. If `validate_import` returns `confidence_notes`, report it as advisory.

6. **Stale index** — if any tool returns `{"status": "error", "hint": "...rebuild..."}`, stop. Tell the engineer: "Steward index is stale — run: steward rebuild." Do NOT read files as a fallback. Do NOT try locate_symbol or other tools as a workaround — stop completely.

7. **No bootstrap** — if any tool returns `{"status": "error", "hint": "...bootstrap..."}`, stop. Tell the engineer: "Layers not defined — run bootstrap first." Do NOT attempt symbol lookups or file reads.

8. **start_refresh is not steward rebuild** — `start_refresh` re-indexes the codebase (phases 1–7) but does NOT update Phase 5 test coverage — it runs in analysis-only mode using cached AST where available. `steward rebuild` is the only operation that refreshes coverage data. If you need fresh coverage before editing a high-risk function, ask the engineer to run `steward rebuild` from the CLI. NEVER call `start_refresh` when `[layers]` is empty — call `start_bootstrap` instead. If any tool returns a stale-index error, tell the engineer to run `steward rebuild` — do not attempt a rebuild via MCP.

9. **Session end** — before closing: run tests for any modified code, then call `preview_doc_drift`. If drift found, call `sync_documentation` and show the patch — never auto-apply. Then call `record_session_end`.

10. **Bad write recovery** — if a write_layer_assignments / write_flow_rules / apply_suggestion produces wrong results, tell the engineer to run `steward undo` to restore the previous steward-arch.toml.

11. **Never edit Steward's files directly** — `steward_out/` files are generated artifacts overwritten on every rebuild; editing them creates false state. `steward-arch.toml` must only be written through MCP tools (`write_layer_assignments`, `write_flow_rules`, `write_alignment`, `apply_suggestion`). `steward.toml` is user-editable but only for `[project]`, `[docs]`, `[dynamic_calls]`, `[suppressed_pairs]`. Never use Edit/Write on `steward_out/` files.

12. **simulate_apply_suggestions** — always call this between `get_pending_suggestions` and the first `apply_suggestion`. If `health_delta < -20` OR `conflicts_detected` is non-empty, do NOT apply blindly — surface the issue to the engineer first.

13. **Challenge Mode priority order** — always address Challenge Mode observations in `priority_score` order: start with `rank=1` (highest) and work down. If `priority_label="HIGH"` is present, do not defer it. If `intent_conflict=true` in the impact block, explicitly surface the contradiction to the engineer before deciding keep/refactor/split/merge.

14. **Session end snapshot** — before closing every session, after `preview_doc_drift`: call `record_session_end(challenge_mode_run=true/false)`. Pass `challenge_mode_run=true` if `challenge_architecture` was called this session. If `record_session_end` returns `drift_alert` with severity "mandatory", note it to the engineer: "Drift alert recorded — Challenge Mode recommended next session." Do NOT skip this step.

15. **Summary display** — when a Steward tool returns `_summary`, display that line to the engineer first as plain text. Show the full JSON body only if `action_required` is true or the engineer asks for detail.

16. **Daemon staleness fallback** — `session_start` and `get_health_pulse` include `daemon_state`. Act on it only when `[daemon] enabled = true` in steward.toml (check `daemon_state.status != "disabled"`).
    - If `session_start` returns `status == "bootstrapping"`: tell the engineer "Steward daemon is running the initial project rebuild — wait ~30s and I'll call session_start again." Wait, then retry. Do NOT call start_bootstrap yourself.
    - If `daemon_state.stale_warning` is non-null AND `daemon_state.status="running"`: the daemon is alive but behind — call `start_refresh` immediately without stopping work. Report: "Steward daemon warning: [stale_warning] — refreshing index." If `start_refresh` also returns a stale-index error: tell engineer "Both daemon and index are stale — run: steward rebuild."
    - If `daemon_state.stale_warning` is non-null AND `daemon_state.status != "running"`: the daemon itself has a problem — report the warning to the engineer and tell them to run `steward daemon status` in the terminal. Do NOT call `start_refresh` as a workaround. Do NOT attempt to restart the daemon via MCP.
    - If `daemon_state.status == "disabled"`: ignore daemon_state entirely — engineer is using on-demand mode.

---

### The Edit Loop (every function-level change)

```
1. inspect_neighborhood(symbol)       → understand what you're changing: signature, callers, depth_risk, patterns
2. check_impact(changes=[symbol])     → ONLY if depth_risk=HIGH or impact_radius=High (Hard Rule 4)
3. pre_edit_bundle(filepath, [...])   → import boundary check — fires via hook, call explicitly for multi-import edits
4. Edit the file
5. auto_apply_safe                    → fires automatically via PostToolUse hook; call on-demand if hook didn't fire
```

`validate_import` returns `ok` with "same layer — intra-layer call": no boundary crossed, no action needed, do not flag.

**Before editing any function where `entropy_score > 0.6`**: state the score to the engineer and suggest writing a test first. Also ask whether the function should be split — high entropy often signals a function doing too much, and splitting before editing is cheaper than untangling after. This is not a hard block — the engineer decides. But never silently proceed on a high-entropy edit.

---

### Common Scenario Quick Reference

| Scenario | Tool sequence | Key logic |
|----------|--------------|-----------|
| **Simple bug fix** | `session_start` → `get_developer_scan(file)` → `inspect_neighborhood(symbol)` → `pre_edit_bundle` → edit | `get_developer_scan` first — confirms whether the bug matches a known pattern (e.g. concurrent mutation) before drilling into implementation. Only go to Tier 3 if the 10-line peek is insufficient after `inspect_neighborhood`. |
| **Major refactor planning** | `get_planning_context` → `validate_feature_plan` → `get_functional_clusters` → `simulate_refactoring_curvature(node, layer)` | Use clusters to find natural subsystem boundaries. Use `simulate_refactoring_curvature` to check whether proposed moves increase architectural heat before committing to the plan. Geometry tools only when `[geometry] enabled = true`. |
| **New import into Core** | `inspect_neighborhood(symbol)` → `pre_edit_bundle(file, planned_imports=[...])` | If `pre_edit_bundle` returns `warn` or `violation` on the import, stop. Offer A/B/C triage (`steward_explain`) before touching the file. Never silently add a boundary-crossing import. |

---

### Session Start (every session)

```
1. session_start
   → runs get_architecture_map + get_health_pulse + start_refresh in one call
   → if [layers] empty → call start_bootstrap instead (skip session_start)
   → if action_required=false → report delta.summary to engineer, proceed
   → if action_required=true  → triage the triage block before engineer's request:
       - triage.new_violations:   decide A/B/C for each
       - triage.new_unformalized: decide ok/warn/violation
       - triage.drift_alert:
           severity="warning"   → mention: "Architecture showing drift signs."
           severity="recommend" → state before new work: "Drift alert: [recommendation]. Recommend Challenge Mode."
           severity="mandatory" → blocker: run challenge_architecture before any other work
       - triage.intent_conflicts: surface active intent conflicts before new work

   If session_start returns a stale-index error AND [layers] is empty:
     → call start_bootstrap immediately — stale state is expected on a fresh project.

   If index may be stale (code changed outside this session):
     session_start(rebuild=true) → full steward rebuild first, then checks
```

The three individual tools remain available for mid-session use — do not call them at startup.

---

### Bootstrap (`[layers]` is empty)

Run autonomously — do not pause for input between steps 1–9:

```
1. start_bootstrap                → file list with imports + heuristic suggestions
2. classify every file            → is_zero_in=true → Entry; topology_confidence="high" → use topology;
                                    medium → naming/imports tiebreaker; None → steward_suggestion;
                                    conflict present → flag + prefer topology; 4–6 layer names; skip __init__.py and tests
3. write_layer_assignments({…})   → writes [layers], reruns phases 1-3, returns rule gaps
4. decide ok/warn/violation       → unformalized=code already does it; never_seen=violation
5. write_flow_rules({…})          → writes [flow_rules], full rebuild, returns violations
6. validate_bootstrap             → high-confidence anomalies: revise + repeat; low: note and continue
   6b. Check hierarchy_suggestion in result:
       - hierarchy_mode_recommended: ask engineer if they want hierarchy mode
         If yes: call write_flow_rules(hierarchy={"Entry": 0, "Core": 1, ...})
         If no: continue with explicit rules
       - hierarchy_mode_active: log it, hierarchy already set
7. triage violations              → A=fix code  B=intentional override  C=change the rule
8. write_claude_suggestions([…])  → record all decisions
9. get_planning_context           → writes steward_out/planning-context.md (architectural DNA)
10. offer to apply: "Bootstrap complete. Health: [score]. I've written [N] suggestions. Apply now?"
    If yes: get_pending_suggestions → simulate_apply_suggestions → apply_suggestion(index=0) until list is empty
    If no:  "Run: steward suggestions / steward apply --all when ready."
    Then:   "Architectural DNA → steward_out/planning-context.md (paste into Gemini for external review)"
```

---

### Refresh (after code changes)

```
1. start_refresh                  → delta by default; full=true for complete scan
2. all deltas=0 → "stable, no new findings"
3. triage new findings            → write_claude_suggestions([…new only…])
4. if tool calls > 10 this session → run get_session_pulse; if drift_risk="high" or session_health < 60:
   surface to engineer before continuing
5. offer to apply: "[N] suggestions written. Apply now?"
   If yes: get_pending_suggestions → simulate_apply_suggestions → apply_suggestion(index=0) until list is empty
   If no:  "Run: steward suggestions / steward apply --all when ready."
```

---

### Challenge Mode (every ~10 refreshes)

```
1. challenge_architecture         → structural observations (not violations)
   → observations with intent_conflict=true are higher priority than structural-only observations
   → state explicitly: "This observation conflicts with stated architectural intent: [rule]"
   → if intent_conflict=true, default recommendation is 'fix code' not 'keep'
   → if active_intent shows stale_warning, note it alongside observations
2. observations are ranked by priority_score (rank=1 = highest).
   For each observation in rank order:
   a. state priority_label + key metrics: violation_density_pct, cognitive_load_score,
      call_volume_at_risk, projected_resolution_pct
   b. if intent_conflict=true: surface it to the engineer before deciding
   c. decide: keep / refactor / split / merge
3. write_claude_suggestions with type='architecture_challenge'
4. get_planning_context           → updates steward_out/planning-context.md with post-challenge state
5. offer to apply if actionable suggestions exist:
   "[N] suggestions written (ranked by priority).
    I recommend applying rank [1] first — it accounts for X% of open violations.
    Want me to apply now?"
   If yes: get_pending_suggestions → simulate_apply_suggestions → apply_suggestion(index=0) until list is empty
```

---

### Feature Plan

```
0. validate_feature_plan(candidate_files=[...], description="...")
   → multi-file features only — skip for single-file edits
   → if predicted_violations: resolve before planning
   → if missing_abstractions: add Protocol/ABC to target layers first
   → if open_violations_touched: triage those violations before proceeding

1. get_planning_context
   → generates steward_out/planning-context.md (layer map, alignment status, open violations)

2. reason: which layers are touched? open violations blocking? flow rules respected?

3. plan: files to create/modify (with layer assignments); validate each import; note new boundaries

4. set_intent_context(feature="...", constraints=[...], architectural_intent=[...])
   → multi-session features only; use actual layer names for conflict detection to work

Tell engineer: "Architectural DNA updated → steward_out/planning-context.md"
For Gemini brief: run steward plan "<feature>" → steward_out/architect-brief.md
```

---

### Applying Suggestions Inline

After any `write_claude_suggestions` call, **do not just tell the engineer to run CLI commands**. Offer to apply via MCP:

```
"I've written [N] suggestions. Want me to apply them now?"

If yes:
  1. get_pending_suggestions          → show the full list
  2. simulate_apply_suggestions([])   → preview impact
     - health_delta < -20 → flag before proceeding
     - conflicts_detected non-empty → resolve before applying
     - new_violations_created → triage after apply
  3. apply_suggestion(index=0)        → repeat until list is empty (index always 0 — list shifts)

If no: "Run: steward suggestions / steward apply --all when ready."
```

`flow_rule`, `layer_assignment`, and resolved Decision B suggestions are auto-applied by the PostToolUse hook after each file edit — check `session_state.auto_applied`. Decision A (code fix) and Decision C (rule change) are never auto-applied.

---

### Suggestion Format

```json
{
  "type": "violation_triage | flow_rule | layer_assignment | doc_suggestion | smell | test_gap",
  "severity": "critical | high | medium | low",
  "reasoning": "one sentence",
  "decision": "A | B | C  (violation_triage only)",
  "finding": { "from_file": "src/...", "to_file": "src/...", "pair": "Layer -> Layer",
               "suggested_value": "warn", "file": "src/...", "suggested_layer": "Core" },
  "toml_patch": "TOML fragment if decision=B",
  "code_note":  "code instruction if decision=A"
}
```

Severity: `critical`=Core violation/circular · `high`=Interface violation/>10-callsite unformalized/Core test gap · `medium`=Entry violation/low-confidence layer · `low`=doc drift/smell

**Decision B (`toml_patch`) requirement:** When writing a Decision B suggestion, generate `toml_patch` by looking up the `violation_pair` in the architecture map and matching the syntax of the existing `[intentional_overrides]` block in steward-arch.toml. A patch with wrong key format will be rejected by `apply_suggestion`. Required fields: `justification`, `approved_by`, `date`.

---

### Tool Reference

| Tool | When |
|------|------|
| `session_start` | **Session start — required** |
| `get_architecture_map` | Mid-session — layer map when needed |
| `get_health_pulse` | Mid-session — health detail when needed |
| `start_refresh` | Mid-session — re-index after code changes |
| **`get_project_atlas`** | **Tier 0** — folder-level orientation, ~1KB; use when starting in unfamiliar area |
| **`get_context_bundle`** | **Tier 1** — all matched files + logic profiles + violations in one call; start of every multi-file task |
| **`inspect_neighborhood`** | **Tier 2** — signature, wiring, depth_risk, source peek; call before every function edit (Hard Rule 3) |
| **`check_impact`** | **Impact gate** — fan_in + depth_risk + caller files; call before any HIGH-impact change (Hard Rule 4) |
| **`trace_call_path`** | BFS from a function to a target layer — understand request paths and cross-layer flows |
| `pre_edit_bundle` | **Before modifying any file with imports** — skeleton + validate_import + pre_edit_check in one call |
| `auto_apply_safe` | On-demand auto-apply of safe suggestions; fires via PostToolUse hook automatically |
| `get_interface_skeleton` | Skeleton-only check |
| `validate_import` | Single-import check |
| `preview_doc_drift` | **Session end — required** |
| `record_session_end` | **Session end — required** |
| `start_bootstrap` | `[layers]` is empty |
| `write_layer_assignments` | After bootstrap classification |
| `write_flow_rules` | After rule decisions; also for hierarchy mode |
| `validate_bootstrap` | After write_flow_rules |
| `write_claude_suggestions` | After bootstrap or refresh triage |
| `get_pending_suggestions` | Before writing new suggestions; before applying |
| `simulate_apply_suggestions` | Between `get_pending_suggestions` and `apply_suggestion` |
| `apply_suggestion` | Apply suggestions inline |
| `get_decision_history` | Before deciding a violation — check past decisions |
| `challenge_architecture` | Every ~10 refreshes |
| `get_context_summary` | Full codebase overview when bundle is too broad |
| `get_layer_files` | List files in a layer |
| `get_file_profile` | Understand one file — Tier 1 fallback |
| `get_logic_profile` | Behavioral patterns per function — for perf/concurrency: call on primary AND all callees |
| `locate_symbol` | Find a function or class (never grep) |
| `steward_explain` | Understand a violation with A/B/C context |
| `validate_feature_plan` | Before planning any multi-file feature |
| `pre_edit_check` | Boundary check (also fires via PreToolUse hook) |
| `get_session_pulse` | After >10 tool calls — live health score and drift risk |
| `get_planning_context` | Before planning a multi-file feature |
| `set_intent_context` | Start of a multi-session feature |
| `get_active_intent` | Check current feature intent mid-session |
| `get_generated_rules` | Inspect computed rule set from [layer_hierarchy] |
| `sync_documentation` | After drift found |
| `report_symbol_miss` | locate_symbol gave wrong result |
| `zoom_to_module` | **Tier 0.5** — all symbols in one module with risk/entropy/violations; bridges atlas → inspect |
| `write_alignment` | Write alignment score after `steward align` confirms natural path |
| `suggest_layers` | Heuristic layer suggestions — call when `start_bootstrap` hints are unclear |
| `suggest_rules` | Analyze call graph for unformalized cross-layer patterns |
| `get_test_readiness` | Pre-modification coverage/risk check — call before editing untested high-risk functions |
| `get_developer_scan` | File-wide bug triage — call at start of bug-hunt mode, before Tier 3 reads |
| `get_pattern_match` | Architectural pattern scores for a file or symbol |
| `generate_reference_doc` | Generate machine-readable reference doc (API skeleton, layer map) |

**Geometry tools** — only call when `[geometry] enabled = true` in steward.toml:

| Tool | When |
|------|------|
| `get_geometric_summary` | Session start (geometry on) — compact overview: clusters · simplices · sep · isolated |
| `get_functional_clusters` | Before a cross-cluster refactor — shows coupling patterns and cluster membership |
| `zoom_to_cluster` | Deep-dive into one cluster's nodes, coupling strength, and boundary members |
| `get_orchestration_patterns` | Before cross-layer feature design — detects hub/pipeline/fan-out patterns |
| `simulate_refactoring_curvature` | Before a large refactor — preview curvature impact of planned moves |
| `find_similar_by_distance` | Hyperbolic nearest-neighbor search — find functions analogous to a known symbol |
| `detect_architectural_drift` | After a large batch of edits — nodes drifted from their layer centroid |

> **Compact mode:** `get_architecture_map`, `get_health_pulse`, `start_refresh`, `get_file_profile`, and `get_layer_files` all accept `format="compact"`. Use compact for routine checks; use full when you need complete data.

---

### Hierarchy Mode (optional)

Only use when the project has a genuine level ordering (Entry → Core → Infrastructure). Do NOT use for hexagonal, event-driven, or mesh architectures.

Trigger: engineer says "use hierarchy mode" OR `validate_bootstrap` returns `hierarchy_suggestion.suggestion = "hierarchy_mode_recommended"`.

```
1. Decide levels: 0=Entry/CLI/API  1=Core/Domain  2=Internal/Shared  3=Storage/DB/External

2. write_flow_rules(
     hierarchy={"Entry": 0, "Core": 1, "Internal": 2, "Storage": 3},
     hierarchy_config={"mode": "strict", "same_level": "warn"}
   )
   → writes [layer_hierarchy] + [hierarchy_config], runs full rebuild, returns generated_rules_preview

3. get_generated_rules             → review computed rules; identify any needing explicit overrides

4. If overrides needed:
   write_flow_rules(rules={"Core -> Storage": "violation"})
   → merges with hierarchy without replacing it

5. Verify: steward status --show-generated-rules
```

When in hierarchy mode: `validate_import` uses the merged rule set (hierarchy + explicit). `steward undo` restores the full previous steward-arch.toml including hierarchy sections.

---

### Geometry Mode (when `[geometry] enabled = true`)

Geometry phases (A–E) enrich the dense map with clustering, simplicial topology, curvature, hyperbolic positioning, and derived signals. Geometry is **disabled by default** — only call geometry tools when `[geometry] enabled = true` in steward.toml.

**Session start addition (geometry on):**

```
After session_start, if [geometry] enabled = true:
1. get_geometric_summary    → compact ~200-token overview of all geometry phases
   - isolated_nodes > 0      → note: "X nodes are architecturally isolated"
   - separation_ratio < 1.5  → note: "Layer separation is weak — geometry review recommended"
   - top coupling_strength   → flag any node with coupling_strength > 20
```

**Geometry as a context filter:** When `get_geometric_summary` returns `isolated_nodes > 0`, include those files explicitly in the next `get_context_bundle` call via the `files` parameter — even if token-matching from the task description would not have surfaced them. Isolated nodes are architecturally invisible to text-matching but are the most likely sources of undetected violations.

**When to call each geometry tool:**

- `get_geometric_summary` — session start when geometry is on; compact health check for all geometry phases
- `get_functional_clusters` — before planning a refactor that touches multiple tightly coupled files; shows which files form natural clusters and what pattern binds them
- `zoom_to_cluster(cluster_id)` — after `get_functional_clusters` identifies a suspect cluster; deep-dives into nodes, coupling strength, and boundary members
- `get_orchestration_patterns` — before designing a new feature that crosses layer boundaries; detects hub-and-spoke, pipeline, or fan-out patterns in the call graph
- `simulate_refactoring_curvature` — before a major refactor; shows which moves reduce or increase architectural curvature
- `find_similar_by_distance` — when looking for functions analogous to a known symbol (hyperbolic nearest-neighbor lookup)
- `detect_architectural_drift` — after a large batch of edits; surfaces nodes that have drifted away from their layer's geometric centroid

**Interpreting `get_geometric_summary` output:**

```
geometry: N clusters · M simplices · sep=X.XX · Y ctx-sensitive · Z isolated
```

- `clusters` — number of functional groups (Phase A)
- `simplices` — higher-order relationship count (Phase B)
- `sep` — layer separation ratio from hyperbolic embedding (Phase D); healthy ≥ 2.0
- `ctx-sensitive` — nodes in high-curvature zones (Phase C); >5% of nodes warrants review
- `isolated` — nodes with `architectural_isolation=true` (Phase E); any non-zero is worth noting
<!-- steward:end -->

