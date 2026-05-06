1) Requirements overview
1. Platform: Desktop-first browser app (landscape), single-page workspace, no mobile layout obligations, minimum viewport 1280x720, optimized at 1920x1080.
2. Primary navigation: Top menu fixed at y=0 with actions Upload, Analyze, Merge, Settings; each action keyboard-addressable and available via command palette.
3. Workspace panes: Canvas/Preview, Source Code View, Browser Context View, Prompt/Agent Console visible simultaneously with draggable splitters and persistent layout profiles.
4. Visual frame rules: Every major panel has `border: 2px solid #FFFFFF`, `padding: 10px`, visible panel titles, and resize handles sized minimum 20x20 px with deadzone hit target minimum 28x28 px.
5. Base typography and colors: Default theme `background: #0047B3` (blue), `color: #FFFFFF` (white), font size user-selectable 9-12 px, no unstyled white blank regions.
6. Alternate theme mode: Muted dark palette preset (e.g., `#111827`, `#1F2937`, `#374151`) with same white text and border rules; live theme switching updates all panes immediately.
7. Vibe command scope: Natural-language input can create/edit components, sections, states, routes, APIs, styles, animation timelines, charts, media placeholders, and integration configs.
8. Traceable editing: Every accepted agent/code action generates patch diff, affected file list, before/after snippet, authoring agent id, and timestamp.
9. Multi-agent core: Per-agent config includes role, character, backend provider, model id, runtime prompt, collaboration policy, execution constraints, and tool permissions.
10. Backend providers: OpenRouter required with per-agent API key; Ollama/OpenLLM optional local providers; provider adapters must expose normalized `generate()`, `stream()`, and `embed()` interfaces.
11. Agent controls: Pause/resume per run, delay range (min/max ms), forced minimum iterations N, hard maximum iterations M, timeout budget, and cancellation token.
12. Parallel/layered execution: Run N agents concurrently, then choose mode `compare`, `merge`, or `fastest`; sequential chaining supported by stage graph (`scaffold->implement->test->polish`).
13. Prompt/context assignment: Prompts can be bound by project stage, skill module, agent, or task template; runtime edits apply next step without page reload.
14. Context augmentation inputs: Manual notes, pasted errors, avoid-list reminders, file references, snippets, diffs, uploaded txt/md/pdf assets.
15. Document ingestion: txt/md parsed directly, pdf parsed via extractor pipeline (OCR fallback), chunked into embeddings, indexed with metadata (`source`, `page`, `section`, `hash`).
16. Catalogue pool: Named document collections with polling interval, trigger rules, selector query, and freshness policy; agents pull permitted docs per run policy.
17. Logging: Store all run events and API payload metadata with key redaction, token counts, latency, cost estimates, and model/provider identifiers.
18. Security: API keys encrypted at rest, masked in UI/logs, reveal-once workflow, RBAC for secrets, and strict separation between user-supplied context and agent-inferred context.
19. Replayability: Run replay endpoint re-executes workflow using frozen config/context snapshot and reports deterministic or drifted outcomes.
20. CLI integration: Optional `vibecoding` CLI supports `run`, `plan`, `diff`, `apply`, `replay`; stdin piping for prompt/context and stdout for machine-readable JSON or unified diff.

2) Component architecture
1. Frontend shell: React/TypeScript app with panel manager, command bus, Monaco-based code editor, preview iframe, run console, and document manager.
2. State layer: Client store (Zustand/Redux) for UI state, WebSocket stream for run events, optimistic patch preview, and undo/redo stack.
3. Preview runtime: Sandboxed iframe/container serving built artifact, DOM snapshot API, element locator map, and screenshot hooks.
4. Orchestrator API: Node/Go service coordinating agents, scheduling steps, enforcing min/max iterations, managing parallel strategies, and applying merge policy.
5. Provider adapters: `OpenRouterAdapter`, `OllamaAdapter`, `OpenLLMAdapter` implementing unified interface for completion/stream/tool-call behavior.
6. Prompt engine: Template resolver merges base system prompt + role prompt + stage prompt + dynamic context + safety policies.
7. Diff engine: AST-aware and text diff services, conflict detector, patch applicator, and rollback manager.
8. Storage services: Project/file store, run store, log store, secret vault, vector index for catalogue pool, and blob storage for uploads.
9. Ingestion workers: Async jobs for pdf/txt/md parse, chunking, embedding, indexing, and checksum dedupe.
10. Observability: Structured logs, metrics (latency/token/cost/failures), tracing ids per run and per agent step.

3) Data model
1. Project: `id`, `name`, `description`, `createdAt`, `updatedAt`, `layoutPreset`, `themePreset`, `activeBranch`, `settings`.
2. File: `id`, `projectId`, `path`, `language`, `content`, `hash`, `version`, `lastEditedBy`, `updatedAt`.
3. Agent: `id`, `projectId`, `name`, `character`, `role`, `provider`, `model`, `apiKeyRef`, `systemPrompt`, `taskPrompt`, `collabPolicy`, `toolPolicy`, `enabled`.
4. AgentLink: `id`, `projectId`, `fromAgentId`, `toAgentId`, `shareLevel` (none/summary/full), `direction`.
5. Run: `id`, `projectId`, `trigger` (ui/cli/api), `mode` (compare/merge/fastest), `status`, `startedAt`, `endedAt`, `configSnapshotId`, `contextSnapshotId`.
6. RunStep: `id`, `runId`, `agentId`, `iteration`, `inputRef`, `outputRef`, `latencyMs`, `tokenIn`, `tokenOut`, `cost`, `status`.
7. Patch: `id`, `runId`, `fileId`, `diff`, `mergeState`, `appliedBy`, `appliedAt`, `rollbackRef`.
8. LogEvent: `id`, `runId`, `stepId`, `type`, `payloadMeta`, `redactionMap`, `timestamp`.
9. ContextAsset: `id`, `projectId`, `type` (txt/md/pdf/snippet/error/diff), `source`, `contentRef`, `checksum`, `ingestStatus`.
10. CataloguePool: `id`, `projectId`, `name`, `selectionRule`, `pollIntervalSec`, `triggerRule`, `maxDocsPerRun`, `updatedAt`.
11. ReplayRecord: `id`, `runId`, `snapshotRefs`, `replayRunId`, `result` (match/drift), `driftReportRef`.

4) Key user flows
1. Create project: User clicks New Project -> chooses template -> workspace opens with 4-pane default layout -> system creates project record + starter files + first snapshot.
2. Vibe-edit component: User selects element in preview -> writes command in Prompt Console -> orchestrator resolves context (DOM + files + notes) -> agent proposes patch -> user previews diff -> apply/rollback.
3. Parallel compare: User selects 3 agents and mode Compare -> orchestrator runs agents concurrently -> scoring rubric evaluates maintainability/perf/accessibility/risk -> side-by-side diffs displayed -> user picks winner or merge.
4. Apply merged patch: User selects Merge mode policy (`majority`, `best-score`, `lead-agent`) -> merge engine resolves conflicts -> generated commit candidate shown -> apply to working tree.
5. Ingest pdf: User uploads pdf -> ingestion worker extracts text/OCR -> chunks and indexes -> asset appears in catalogue -> agent run references selected sections by citation metadata.
6. Replay run: User opens run history -> clicks Replay -> system clones snapshots + config -> executes same graph -> displays output diff against original with drift reasons.

5) Acceptance criteria
1. Top menu always visible and includes Upload/Analyze/Merge/Settings with click + keyboard activation.
2. All workspace panels render with 2 px white border and 10 px internal padding.
3. Resize handles pass hit-area test (>=28x28 px deadzone) and support mouse drag for width/height.
4. Font size selector enforces 9-12 px range and applies globally in under 100 ms after change.
5. Natural-language command modifying UI updates live preview and creates persisted file diff entry.
6. Per-agent config allows distinct provider/model/key/prompt and can be edited while app is running.
7. Execution engine enforces min iteration N before completion unless cancelled and never exceeds max iteration M.
8. Compare mode returns at least two candidate outputs plus rubric scores and rationale.
9. Merge mode produces deterministic patch output for same candidate set + policy.
10. Logs mask API keys in UI and stored payloads; plaintext key never appears after save action.
11. Run history supports filtering by project, agent, status, date, and text query.
12. Replay reproduces same input snapshot and marks run as `match` or `drift` with report.
13. txt/md/pdf uploads become queryable context assets with status indicators.
14. CLI `vibecoding run` accepts stdin prompt and returns JSON including run id and diff summary.

6) Risk list and mitigations
1. Prompt injection from uploaded docs; mitigate with content trust labels, sandboxed retrieval, policy guardrail prompt, and high-risk action confirmation.
2. Secret leakage in logs/output; mitigate with deterministic redaction middleware, secret scanners, and denylist filters before persistence.
3. Runaway loops/cost spikes; mitigate with max iterations/time budget/token budget and circuit breaker alerts.
4. Non-deterministic merges causing regressions; mitigate with merge policy tests, semantic conflict detection, and mandatory preview before apply.
5. Malicious code generation affecting preview host; mitigate with iframe sandbox, CSP, blocked network by default, and explicit allowlist toggles.
6. Privacy issues in catalogue pool sharing; mitigate with per-pool ACL, project scoping, encrypted storage, and audit trail for document access.
7. Provider outages/rate limits; mitigate with retry/backoff, fallback model routing, and graceful partial-run completion states.
8. Large pdf ingestion failures; mitigate with async chunk pipeline, OCR fallback, resumable jobs, and per-stage error reporting.
9. UI overload for power features; mitigate with presets, command palette, progressive disclosure, and customizable panel layouts.
