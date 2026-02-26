1) Product goal: Build a desktop-first browser app/website builder where users modify UI, logic, styles, animations, media, integrations, and automation through semantic natural-language commands, with fully traceable code generation and multi-agent orchestration.

2) UI contract (strict visual rules): App shell uses blue background (#0B3D91 default), white text (#FFFFFF), global font size control constrained to 9–12px (default 10px), all major panes/components use 2px solid white borders with 10px internal padding, top menu contains Upload | Analyze | Merge | Settings, no unstyled white-empty surfaces are allowed, all panel separators remain visible at all zoom levels.

3) Workspace layout (desktop landscape): Single screen with resizable panes for Canvas/Preview, Source Code, Browser Context, Prompt/Agent Console; panes support drag-reorder and snap layouts (2-column, 3-column, quad), every pane includes oversized resize hit areas (minimum 16px deadzone beyond visible handle on each draggable edge/corner).

4) Drag/drop interaction: Feature blocks (Component, Section, Logic, Style, Animation, Media, Integration, Agent Task) are draggable from a left library into canvas or pipeline lanes, insertion indicators must show exact drop index, keyboard alternative supports Move Up/Down + Place shortcuts.

5) Natural-language edit engine behaviors: Command parser outputs structured intents `{target, operation, constraints, style, dataSources}`, each run must generate (a) preview update, (b) patch/diff against source files, (c) run log entry linked to prompt and context snapshot.

6) Multi-agent orchestration requirements: Agent entity fields = `id,name,character,role,backend,model,apiKeyRef,systemPrompt,taskPrompt,sharingPolicy,stepMin,stepMax,delayMinMs,delayMaxMs,status`; supported backends = OpenRouter (required), Ollama/OpenLLM (optional); runtime supports pause/resume/cancel, bounded iterations (must execute at least N, never exceed M), and deterministic scheduling option.

7) Agent collaboration graph: Directed permission graph defines who can read whose outputs/context; collaboration modes = `isolated`, `shared-pool`, `pair-review`, `pipeline`; per-task execution modes = `compare` (parallel + rubric scoring), `merge` (policy merge), `fastest` (first acceptable output).

8) Prompt assignment and dynamic context: Prompt panel supports stage presets (`scaffold|implement|test|polish`), skill modules (`security-review|a11y-audit|perf-opt|refactor`), ad hoc attachments (error text, avoid-list, code snippets, file paths, diffs), and per-agent overrides editable during active run with versioned prompt revisions.

9) File/document ingestion: Accept pasted/uploaded `.txt/.md/.pdf`; ingestion pipeline = parse -> chunk -> embed/index -> citation map; each chunk stores origin metadata `{file,page,offset,hash}`; agent context assembly must include selected chunk IDs and token budget report.

10) Catalogue pool feature: Project-level document repository with fetch policies `{manual,interval,trigger}`; interval polling configurable in minutes; trigger rules include on-run-start/on-error/on-stage-change; selection rules include tag match, recency, relevance score threshold, and max-doc cap per run.

11) Logging and traceability: Persist full run graph (agent steps, tool calls, prompts, context IDs, model responses, patch artifacts), API payload metadata logged with redaction of secrets, searchable history by project/file/agent/tag/date, per-file timeline with reversible diffs, and one-click run replay using frozen config + context snapshot.

12) Security model: API keys stored encrypted at rest via server-side KMS envelope keys, never shown plaintext after save, UI only shows masked form (`sk-****`), logs redact credentials and PII patterns, strict boundary views: `user-supplied`, `agent-inferred`, `external-tool-fetched` data are separately labeled and queryable.

13) Source control + patch application: Generated edits become patch sets with file-level confidence score, conflict detector blocks unsafe auto-apply, user can apply per-hunk/per-file/all; every apply creates checkpoint commit metadata in project history and rollback point.

14) CLI integration (optional but first-class): `vibecoding` CLI commands include `run`, `plan`, `diff`, `apply`, `replay`; stdin piping allowed for prompt/context (`cat err.log | vibecoding run --agent debugger`), stdout emits machine-readable JSON or unified diff; browser app remains fully operational without CLI installed.

15) Accessibility/productivity requirements: Full keyboard navigation across panes, visible focus ring (2px white + 1px blue offset), command palette, configurable keymap, UI density slider, text scale constrained to 9–12px with live preview, reduced-motion toggle for animations.

16) Frontend component architecture: Modules = `ShellLayout`, `TopMenu`, `PaneManager`, `CanvasRenderer`, `CodeEditor`, `DOMContextInspector`, `AgentConsole`, `PromptComposer`, `RunTimeline`, `DiffViewer`, `CatalogueManager`, `Settings`; state managed via event bus + normalized store with optimistic updates and rollback on failed apply.

17) Backend architecture: Services = `ProjectService`, `FileService`, `AgentOrchestrator`, `PromptContextService`, `IngestionService`, `RunLogger`, `DiffPatchService`, `ReplayService`, `SecretsService`; queue-based execution with idempotent job IDs and per-agent concurrency limits.

18) Data model (minimum entities): `Project`, `ProjectFile`, `LayoutPreset`, `AgentProfile`, `Run`, `RunStep`, `PromptRevision`, `ContextAttachment`, `CatalogueDocument`, `CatalogueChunk`, `PatchSet`, `PatchHunk`, `ReplaySnapshot`, `AuditLog`, `SecretRef`; all entities require `id,createdAt,updatedAt,createdBy` and soft-delete flag where applicable.

19) Key user flow A (create project): User creates project -> selects template/blank -> shell initializes 4-pane layout -> default agents loaded -> initial scaffold prompt executed -> preview and source appear -> first checkpoint recorded.

20) Key user flow B (vibe-edit component): User selects DOM node or file symbol -> enters NL command -> parser creates intent -> orchestrator assigns agent(s) -> proposed patch shown -> user previews + approves -> patch applied -> run + diff + checkpoint stored.

21) Key user flow C (parallel compare): User selects multiple agents + `compare` mode -> agents run in parallel with shared rubric -> system displays side-by-side diffs + scores + cost/tokens -> user picks winner or merged result -> apply flow continues.

22) Key user flow D (ingest pdf + use context): User uploads PDF -> ingestion indexes chunks -> user tags doc into catalogue -> prompt references tag -> context assembler injects relevant chunks with citations -> agent output links back to chunk origins.

23) Key user flow E (replay run): User opens run history -> selects run -> system restores exact config/prompts/context pointers/model settings -> executes replay in sandbox mode -> compares new output vs original for drift.

24) Acceptance criteria (testable): AC1 top menu has exactly Upload/Analyze/Merge/Settings; AC2 all primary panes render 2px white border + 10px padding; AC3 global font size selector clamps to 9–12px; AC4 resize handles draggable with >=16px hit area; AC5 every NL edit yields preview update + source diff + run log; AC6 per-agent step bounds enforce N<=steps<=M; AC7 secret values never appear unmasked in UI/log exports; AC8 compare mode returns rubric scores and diff artifacts for each candidate; AC9 replay reproduces same input envelope and records drift metrics; AC10 CLI run works with stdin piping and JSON output mode.

25) Risk list + mitigations: R1 prompt injection via ingested docs -> mitigation: trust labels, context firewall, allowlist tools; R2 runaway agent loops/cost spikes -> mitigation: hard step/token/cost caps + watchdog cancels; R3 secret leakage -> mitigation: redaction middleware + vault-backed references only; R4 unsafe code patches -> mitigation: policy checks + test gates + partial apply; R5 model nondeterminism -> mitigation: replay snapshots + temperature controls; R6 privacy exposure in logs -> mitigation: field-level retention policy + scrub jobs + scoped access control.

26) Non-functional targets: Initial load <2.5s on desktop broadband, patch preview generation p95 <4s for small edits, run log indexing near-real-time (<2s), pane drag/resize at 60fps target on modern desktop GPU.

27) Implementation interfaces (example contracts): `POST /runs {projectId,mode,agents[],promptRevisionId,contextIds[]}`, `GET /runs/:id/diff`, `POST /patches/:id/apply`, `POST /catalogue/ingest`, `POST /replay/:runId`; websocket channel `run.{id}` streams step events `{agentId,step,status,tokens,cost,artifactRefs}`.
