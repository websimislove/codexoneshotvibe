Role: You are a senior product engineer + UX architect. Output an implementable specification for a desktop browser-based HTML app/website builder with multi-agent vibe coding.

Output format rules: keep lines dense, no fluff, no redundant headings, prefer one-line items, no interpretation beyond stated requirements, assume expert reader, produce concrete interfaces/behaviors.

1) Product goal
1.1 Build a web app + website builder where natural-language commands can create/modify any component/section/function/style/animation/media/integration.
1.2 Provide agentic multi-agent collaboration where agents can coordinate, share selected context, and operate on source + runtime browser state.
1.3 Every accepted change must update live preview + source files + traceable diff history.

2) UI requirements (must implement exactly)
2.1 Top menu actions: Upload, Analyze, Merge, Settings.
2.2 Main workspace panels: Canvas/Preview | Source Code View | Browser Context View | Prompt/Agent Console.
2.3 Drag-and-drop for selecting/reordering features, components, and agent tasks.
2.4 Visual style baseline: blue background, white font, font size 9–12 px, 2 px white borders, 10 px edge padding, no white empty areas.
2.5 Component separation: every panel/frame/card has visible 2 px white border + internal padding.
2.6 Resize/drag interaction: large deadzone handles on each panel edge/corner for width/height resizing; handles must be easy to grab with coarse mouse movement.
2.7 Desktop-first landscape layout, no mobile layout requirements.
2.8 Accessibility/productivity: keyboard shortcuts, visible focus states, adjustable UI density and text scale.

3) Natural-language capability scope
3.1 Generate/edit UI sections, components, routing, state logic, data fetching, event handlers.
3.2 Generate/edit styling: themes, tokens, spacing, layout systems, component variants.
3.3 Generate/edit animation and visualization code with parameterized controls.
3.4 Media workflows: image/audio/video placeholders + provider adapters; if provider unavailable, scaffold interface contracts and mock assets.
3.5 Integrations: REST/GraphQL/webhooks/embeds/library install hooks with explicit config schema.

4) Multi-agent orchestration
4.1 Agent config entity fields: id, name, character, role, enabled, backend, model, apiKeyRef, systemPrompt, taskPrompt, sharePolicy, toolsPolicy, delayMinMs, delayMaxMs, minStepsN, maxStepsM, pauseState.
4.2 Backends: OpenRouter required (per-agent API key + model), optional Ollama/OpenLLM adapters.
4.3 Collaboration graph: define which agents can read/write shared context channels and which are isolated.
4.4 Execution modes: parallel N agents; sequential pipelines; stacked/layered orchestration.
4.5 Output handling modes: compare (diff + rubric scoring), merge (policy rules), efficient (fastest acceptable output, no compare).
4.6 Controls: start, pause, resume, cancel, retry-from-step, enforce min/max iterations.

5) Prompt assignment + dynamic context
5.1 Prompt assignment matrix dimensions: agent × projectStage × skillModule.
5.2 Project stages examples: scaffold, implement, test, polish; stage is editable per run.
5.3 Skill modules examples: security review, accessibility audit, performance optimization, refactor.
5.4 Runtime context attachments: pasted errors, avoid-list reminders, file paths, snippets, diffs, uploaded files (.txt/.md/.pdf).
5.5 Ingestion pipeline: parse text/md directly, pdf via extractor to chunked text, store embeddings/keywords + raw source reference.
5.6 Catalogue pool: document repository with scheduled fetch triggers; define polling interval, document selection rules, TTL/cache policy, and per-agent subscription filters.

6) Logging, traceability, replay, safety
6.1 Log all runs: prompts, selected context IDs, tool calls, API request/response metadata, token/cost metrics, outputs, patches, approvals.
6.2 Secrets policy: never display API keys in plaintext; store encrypted at rest; show masked values in UI/logs.
6.3 Separation model: explicitly track what user provided vs what agent inferred/generated.
6.4 History views: searchable run history, per-file timeline, side-by-side diffs, rollback points.
6.5 Reproducibility: run replay must rehydrate same config + context snapshot + model settings and record deterministic/non-deterministic flags.

7) Terminal + piping interface
7.1 Provide optional VibeCoding CLI that can trigger runs, stream logs, generate/apply patches, and query run history.
7.2 CLI must support stdin context input and stdout machine-readable diff/result output.
7.3 Browser app remains fully functional without CLI enabled.

8) Component architecture (implementation-oriented)
8.1 Frontend modules: LayoutManager, PanelRegistry, DragDropEngine, ResizeHandleSystem, ThemeEngine, CommandComposer, DiffViewer, RunTimeline, AgentConfigEditor, CatalogueManager.
8.2 Backend services: ProjectService, FileService, OrchestratorService, PromptContextService, IngestionService, RunLogService, ReplayService, SecretVaultService, PolicyEngine.
8.3 Orchestrator internals: scheduler, dependency graph executor, shared memory bus, step guardrails, output evaluator, merge engine.
8.4 Storage layers: relational DB for metadata, object store for artifacts/uploads, vector index for catalogue/context retrieval, append-only event log for run replay.

9) Data model (minimum entities)
9.1 Project(id, name, settings, theme, createdAt, updatedAt).
9.2 File(id, projectId, path, language, contentRef, hash, version).
9.3 Agent(id, projectId, configFields...).
9.4 Run(id, projectId, mode, stage, startedAt, endedAt, status, replaySeed).
9.5 RunStep(id, runId, agentId, stepIndex, inputContextRefs, outputRef, tokenUsage, cost).
9.6 Patch(id, runId, fileId, diffRef, appliedBy, appliedAt, rollbackRef).
9.7 LogEvent(id, runId, type, payloadRef, redactionLevel, timestamp).
9.8 CatalogueDoc(id, projectId, sourceType, sourceRef, parsedRef, embeddingRef, tags, schedule, lastFetchedAt).
9.9 SecretRef(id, ownerType, ownerId, provider, maskedLabel, vaultKeyRef).

10) Key user flows (must specify behavior)
10.1 Create project: user selects template/settings -> workspace opens with default panels/theme -> initial file graph created.
10.2 Vibe-edit component: user enters NL command -> orchestrator proposes patch set -> preview updates in sandbox -> user applies/rejects per hunk.
10.3 Parallel compare: run N agents -> score outputs via rubric -> show ranked diffs -> user selects winner or merge policy.
10.4 Apply patch: validate build/lint/tests hooks -> commit to project history -> broadcast panel refresh.
10.5 Ingest pdf: upload -> extract/chunk/index -> attach doc to selected agents/stages -> cite chunk IDs in agent context.
10.6 Replay run: pick historical run -> lock snapshot -> re-execute with same config -> compare outputs and drift report.

11) Acceptance criteria (testable)
11.1 Given a NL command, system produces preview update + file diffs linked to run ID.
11.2 Panel borders render 2 px white and padding 10 px across workspace.
11.3 Resize handles are large enough to grab and resize both width/height on all panels.
11.4 Agent config supports per-agent backend/model/API key reference with masking.
11.5 Scheduler enforces minStepsN/maxStepsM and supports pause/resume.
11.6 Compare mode displays rubric score + diff for each agent output.
11.7 Merge mode applies policy-driven synthesis and emits provenance map.
11.8 Uploaded txt/md/pdf content becomes selectable context for runs.
11.9 Catalogue pool fetches by schedule and attaches matching docs by rules.
11.10 Logs exclude plaintext secrets and include request/response metadata.
11.11 Replay reproduces prior run inputs/config and reports output drift.
11.12 CLI accepts stdin context and emits stdout diff/results.

12) Risks + mitigations
12.1 Prompt injection via uploaded docs -> content trust labels, sandboxed tool permissions, policy filters, human approval for sensitive ops.
12.2 Secret leakage -> vault-backed storage, runtime redaction middleware, scoped tokens, rotation workflows.
12.3 Runaway loops/cost spikes -> hard iteration caps, token/cost budgets, kill switches, anomaly alerts.
12.4 Unsafe code patches -> pre-apply validation pipeline, static checks, test gates, rollback strategy.
12.5 Privacy/compliance issues -> data classification tags, retention controls, audit logs, tenant isolation.
12.6 Non-deterministic agent drift -> snapshotting + replay seeds + evaluator baselines.
