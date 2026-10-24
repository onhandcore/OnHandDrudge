# OnHandDrudge

OnHandDrudge provides bounded, Host-driven current Work Unit execution and semantic lifecycle operations.

## Task Admission

OnHandDrudge admission is determined by the structure and execution-lifecycle
requirements of the task, not by Host capability.

A Host MUST NOT bypass OnHandDrudge merely because the Host believes it can
directly plan, reason about, or execute the task itself.

Before substantive execution begins, the Host MUST make one observable admission
decision:

    DRUDGE_ADMISSION=STRUCTURED

or:

    DRUDGE_ADMISSION=LIGHT_TASK_BYPASS

Silent bypass is not allowed.

### Structured admission

The Host MUST invoke OnHandDrudge when one or more of the following properties
is materially present in the task:

- multiple materially distinct semantic scopes require independent completion,
  validation, or evidence;
- multi-stage execution depends on durable outputs or state produced by earlier
  stages;
- execution should resume from a bounded failed point rather than restart the
  whole task;
- autonomous or one-shot continuation across multiple execution units is part of
  the task requirement;
- execution state, completion evidence, handoff state, or recovery state must be
  preserved across stages.

A property is materially present only when ignoring that property would remove
meaningful decomposition, bounded recovery, continuation, or independently
required evidence from the task.

Ordinary local step ordering, temporary intermediate outputs, or routine
edit-build-test sequences do not by themselves establish STRUCTURED admission.

Material presence is determined from task topology and execution-lifecycle
requirements, not from the Host's estimate of whether Drudge would improve its
own performance.

Host intelligence, confidence, coding ability, planning ability, or direct
execution ability does not change this classification.

Available tools, providers, permissions, and environment facts may affect how an
admitted task is executed, but MUST NOT by themselves convert a structurally
qualified task into LIGHT_TASK_BYPASS.

### Light-task bypass

LIGHT_TASK_BYPASS is permitted only when all of the following are true:

- the task has one bounded semantic scope;
- no materially useful semantic decomposition exists;
- the task can be completed as one bounded execution scope without independent
  stage completion, durable inter-stage state, bounded resume, autonomous
  multi-unit continuation, or stage-level evidence;
- no durable cross-stage continuation state is required;
- failure can reasonably restart from the task entry point without losing
  meaningful execution progress or evidence;
- the user has not requested structured, resumable, autonomous multi-stage
  execution.

The Host MUST record the reason for LIGHT_TASK_BYPASS.

### Size versus semantic decomposition

Task size alone is neither a sufficient reason to invoke Drudge nor a sufficient
reason to split work.

"Do not split merely because the task is large" means:

    do not create arbitrary size-based partitions.

It does NOT mean:

    suppress semantic decomposition when materially distinct task scopes or
    execution-lifecycle requirements exist.

When structured admission applies, semantic decomposition is determined by task
topology and lifecycle needs, even if the Host itself is capable of executing the
entire task directly.

### Admission consequence

If:

    DRUDGE_ADMISSION=STRUCTURED

the Host MUST enter the normal OnHandDrudge capsule / Work Unit / JIT execution
path before ordinary direct implementation proceeds.

Capsules or execution evidence MUST NOT be generated retroactively merely to make
a directly executed task appear Drudge-governed.

If STRUCTURED admission has been established but the Drudge path cannot be
entered, the Host MUST report that execution-path failure explicitly.

It MUST NOT silently reclassify the task as LIGHT_TASK_BYPASS or proceed with
ordinary direct execution merely because Drudge is unavailable.

If:

    DRUDGE_ADMISSION=LIGHT_TASK_BYPASS

ordinary Host execution may proceed without Drudge.

## Canonical semantic capsule contract v2

New semantic runs explicitly construct `PublicHostSurface(root, task_run_id=...,
capsule_contract_version="onhand.semantic_capsule.v2")` at an empty run root.
Fresh sessions recover the persisted contract by opening that root with the same
run ID. An existing legacy run cannot be converted in place.

There are exactly three semantic classes. Memory is the exact user string,
encoded as UTF-8 without newline conversion, in `memory_capsules/<revision>/body.md`.
Only explicit user additions, changes and overrides append Memory revisions.
Global Hybrid is Host-reasoned task-wide Markdown in
`global_hybrid_capsules/<revision>/body.md`. Decomposed Hybrid is a real semantic
child in `decomposed_hybrid_capsules/<hashed-scope-id>/<revision>/body.md`.
`meta.json` contains digested mechanical facts; the MD is the sole canonical body.
Accepted bodies and closed membership sets never change in place.

`submit_memory_capsule(content, provenance, change_kind, expected_revision)`
preserves exact user text. Provenance has `source="user"`, `user_input_id` and
`received_at`. `accept_global_hybrid(body_md, memory_source, expected_revision,
fidelity_review)` binds to `surface.capsules.memory_binding()`: the effective Memory
reference and exact lineage digest, and it is permitted only with a completing
source-fidelity declaration (see *Capsule quality*). `accept_decomposed_hybrid(scope_id,
body_md, parent_ref, required_work_unit_ids, expected_revision, coverage)` binds a stable
scope to a real Global or Decomposed parent, and its optional `coverage` declaration
classifies declared root/global obligations. Each reference has `kind`, `id`, `revision`,
`digest`. Materially changed boundaries use a new scope ID.

Every v2 Work Unit declares `semantic_scope_ref`; Work Units are execution
projections, never semantic capsules. `bind_semantic_work_units(work_units)`
records exact execution identities for each closed scope membership AND freezes the
complete GLOBAL/root first-projection binding at bind time. Multiple
Work Units can share one scope. Successful members require intact PASS evidence,
zero-blocker seals and valid paired handoff terminal facts for non-final members.
After all members pass, execution writes one `SEMANTIC_SCOPE_COMPLETED` record.
`complete_semantic_scope(semantic_scope_ref)` also supports explicit verification
and restart recovery. Failed, cancelled, incomplete, stale or superseded unfinished
scopes never earn output eligibility. Final members require no next handoff.

`reflect_semantic_ancestry(semantic_scope_ref)` walks current scope, nearest real
parent, successive real parents and Global. It has no fixed depth. Repeated scope
identity, cycles, missing parents and stale revision/digest references stop closed.
It never visits Memory. `request_global_reflection(affected_scope_ref=None)`
returns current Global, affected facts and sealed evidence without reading Memory
bodies. The Host calls `submit_global_reflection_result(request_id, result)` with
`RESOLVED_AT_GLOBAL`, `GLOBAL_INSUFFICIENT`, `GLOBAL_INVALID`, or
`AUTHORITY_RECOVERY_REQUIRED`. Only the latter three authorize
`request_memory_recovery(request_id)`, which returns effective exact Memory
lineage, current Global and sealed evidence. The Host may submit
`accept_global_from_memory_recovery(request_id, body_md)` to advance Global.

Upper revisions mechanically stale only known dependent branches. Revalidation
accepts successor child revisions bound to the new exact parent. Explicit
`recompile_semantic_work_units(work_units)` replaces only affected unexecuted
Work Units with successor revisions, preserving unrelated identities, historical
projections and sealed handoffs. After a consumed handoff it requires its exact
continuation token and a fresh session. A digested v2 execution amendment binds
the original declared successor to its validated semantic successor; the original
HC02 receipt is immutable. Unknown execution/Host outcomes block amendments.

`terminate_semantic_scope(semantic_scope_ref, status)` records `FAILED` or
`CANCELLED`. Closed run states `SUCCESS`, `EXPLICIT_FAILURE`, `CANCELLED` and
`INCOMPLETE` freeze semantic mutation. History and output checkpoints remain
readable; further semantic work uses a new run. Non-terminal interruption can
resume subject to the existing safe execution and handoff guards.
In v2, `finalize_run("INTERRUPTED", ...)` writes an
`INTERRUPTED_NONTERMINAL` checkpoint under `runtime_state/interruptions/`, leaving
the run open for safe resume and eventual terminal finalization. Legacy v1
`INTERRUPTED` finalization keeps its original persisted meaning.

## Root completion guard

`WORK_UNIT_COMPLETED=PASS` means only that the current Work Unit completed. It never by
itself means `RUN_FINALIZED` or root task completion.

Root terminal eligibility is a separate mechanical gate over Host-declared mandatory root
requirement coverage. The Host declares the mandatory requirement identifiers and owns every
semantic judgment about them; Drudge only stores, validates and gates:

    declare_root_requirements(requirement_ids, operation_id, evidence_ref=None,
                              expected_coverage_revision=None)
    update_root_requirement(requirement_id, status, operation_id, evidence_ref=None,
                            last_work_unit_id=None, expected_coverage_revision)
    get_root_requirement_coverage()
    get_root_finalization_eligibility()
    get_root_continuation()

Allowed statuses are exactly `PENDING`, `SATISFIED` and `BLOCKED_CONTRACT_CHANGE`. Coverage
persists on the canonical run state at `runtime_state/root_requirements.json`; every declared
requirement starts `PENDING` and each accepted Host judgment advances one coverage revision.
Stale coverage revisions, unknown requirement identifiers, a conflicting operation identity
and a changed declared requirement set all fail closed.

Root finalization is a hard terminal rule:

- any mandatory requirement `PENDING` -> `FINALIZATION_REJECTED` / `CONTINUE_REQUIRED`;
  `finalize_run("SUCCESS", ...)` raises `ROOT_FINALIZATION_REJECTED:CONTINUE_REQUIRED` and no
  finalization record is written;
- every mandatory requirement `SATISFIED` -> `ROOT_FINALIZATION_ALLOWED`;
- remaining unsatisfied all `BLOCKED_CONTRACT_CHANGE` -> `USER_DECISION_REQUIRED` with
  `REQUEST_CAPSULE_TO_HOST`; `SUCCESS` is still forbidden and the run may only close under the
  existing non-SUCCESS terminal contract.

Continuation preserves the same root task identity and the persisted coverage, and advances
only through the existing paired terminal handoff, its deterministic operation identity and
its idempotent replay. `get_root_continuation()` returns one content-addressed continuation
receipt, so a crash or retry returns the same operation identity and never creates a second
continuation transition. Drudge performs no semantic reasoning: it never ranks, infers or
interprets a requirement, and a run that declares no coverage keeps the existing terminal
behaviour unchanged.

### Requirement reconciliation

`RootRequirementLedger` is already authoritative and its implementation and schema are
not changed. Reconciliation works as follows:

1. The Host owns the semantic decision about whether verified evidence satisfies a
   declared root requirement.
2. Drudge mechanically records valid status updates through the existing
   `update_root_requirement` interface.
3. A completed Work Unit does not automatically satisfy every associated root
   requirement.
4. Requirements may remain `PENDING` while later work continues.
5. The Host should reconcile requirements when sufficient evidence becomes available,
   and must reconcile all mandatory requirements before requesting successful root
   finalization.
6. Partial evidence must not be promoted to `SATISFIED`.
7. A true product-contract conflict must follow the existing `BLOCKED_CONTRACT_CHANGE` /
   Host decision path.
8. Status changes must preserve the existing expected revision, operation identity,
   evidence reference and idempotency rules.
9. A Host may perform bounded or phase-level reconciliation as appropriate; there is no
   mandatory per-Work-Unit receipt and no new handoff gate.
10. Before `SUCCESS` finalization the Host must inspect remaining `PENDING` / `BLOCKED`
    requirements and use the actual root terminal eligibility mechanism
    (`get_root_requirement_coverage`, `get_root_finalization_eligibility`,
    `get_root_continuation`, and `finalize_run("SUCCESS", ...)`).

The key invariant:

    WORK_UNIT_PASS != ROOT_REQUIREMENT_SATISFIED
    WORK_UNIT_PASS != ROOT_FINALIZATION_SUCCESS

## Continuation governance

`PREPARE` freedom: before any OnHandDrudge execution JSON / Work Unit becomes active
the Host may freely perform preparation — web/search/research, documentation lookup,
dependency download, tool discovery, environment probing, compiler/toolchain
discovery, version verification, license verification, SHA256 verification, locating
files, dependency availability checks and other non-construction fact gathering.
Drudge invocation is NOT mandatory merely for preparation, there is no continuation
lock and no forced execution projection is created. The purpose is to gather verified
facts first and then project a better execution plan.

The frozen execution lifecycle for a fresh substantial task is:

    PREPARE -> PROJECT/FREEZE -> ACTIVE RUN -> EXECUTE/CONTINUE -> ROOT FINALIZATION

`PREPARE` is the fact-gathering phase defined above. It may include inspecting current
project state, discovering source and artifacts, inspecting existing evidence, locating
tools and dependencies, consulting documentation, collecting verified facts and
reconciling continuation context. Ordinary fact gathering is not product construction.

After `PREPARE` the Host must establish the correct canonical task/run authority and use
the existing Drudge semantic/projection interfaces according to their actual contracts.
For a fresh substantial task the canonical progression is:

    MEMORY
      -> GLOBAL_HYBRID
      -> DECOMPOSED_HYBRID
      -> closed Work Unit membership
      -> bind_semantic_work_units
      -> canonical execution progression

There is no separate initial projection MD bridge and no additional pre-execution
authorization record to create. `bind_semantic_work_units(work_units)` freezes the
complete GLOBAL/root first-projection binding at bind time (see *First projection
binding*). After that, each Work Unit is executed through the canonical
`execute_current_work_unit` path, which materializes and freezes that Work Unit's own
execution projection before running the action it governs. The Host does not call a
per-Work-Unit freeze manually before using ordinary product-development tools; the
per-Work-Unit execution projection is materialized and frozen by the Drudge execution
path itself.

Root/global projection binding establishes the frozen semantic execution topology.
Drudge's per-Work-Unit execution path materializes and freezes its own projection before
executing that projection. Host-owned product construction is a separate activity that
must remain aligned with the active root contract, the declared Work Unit scopes and the
canonical continuation state. The Host must not silently perform work outside the
declared task/scope merely because it has file-editing tools. Drudge does not intercept
or directly control every Host filesystem operation; it governs the declared task/scope,
the frozen topology and the mechanical execution boundary.

For a continuation request (`continue`, `resume`, `finish`, `keep going`, `complete
the existing project`, `continue from current state`) the Host MUST perform a bounded
`RESUME_PREFLIGHT` before any substantive construction. `RESUME_PREFLIGHT` exists only
to recover facts: read existing source, inspect build artifacts, completed outputs,
tests/results, dependency state, previous evidence, Drudge run state, handoff state
and the RootRequirementLedger; identify completed and remaining requirements; verify
the current project root and revision/digest. It is read-only or mechanically
non-substantive.

Before the Drudge governance decision is made the Host MUST NOT implement new
features, perform substantive source modifications, begin a new construction phase,
silently redesign the task, generate a competing long-term plan, or continue through
multiple requirements. The rule is `ALIGN FACTS FIRST, DECIDE GOVERNANCE SECOND,
CONSTRUCT THIRD`; "I already understand it" is not permission to construct.

`HOST_CAPABILITY_IS_NOT_A_BYPASS_CONDITION`. The following are INVALID reasons to skip
Drudge: "I can handle this myself.", "The model is capable enough.", "I already
understand the project.", "This looks straightforward to me.", "I can probably finish
it faster directly.", "The current Host is stronger than the previous Host.", "I have
enough context in this window." Host/model self-confidence is NOT governance evidence;
a strong Host obeys the same structural gate as a weaker Host.

`STRUCTURAL_DRUDGE_MANDATORY_GATE` decides continuation governance from TASK
STRUCTURE, never from model confidence:

    IF active_drudge_run_exists:
        decision = RESUME_EXISTING_RUN                  Drudge invocation = MANDATORY
    ELSE IF prior_drudge_handoff_exists
         OR prior_root_requirement_ledger_exists
         OR prior_drudge_run_identity_exists:
        decision = RESUME_EXISTING_RUN                  Drudge invocation = MANDATORY
    ELSE IF continuation has multiple remaining mandatory requirements:
        decision = PROJECT_REMAINING_WORK_TO_DRUDGE     Drudge invocation = MANDATORY
    ELSE IF continuation requires multi-stage completion:
        decision = PROJECT_REMAINING_WORK_TO_DRUDGE     Drudge invocation = MANDATORY
    ELSE IF continuation has a root-level acceptance matrix:
        decision = PROJECT_REMAINING_WORK_TO_DRUDGE     Drudge invocation = MANDATORY
    ELSE IF requirement coverage / handoff / multi-session continuity is required:
        decision = PROJECT_REMAINING_WORK_TO_DRUDGE     Drudge invocation = MANDATORY
    ELSE IF "finish/complete an existing substantial project" and the remaining work
            is not one bounded local change:
        decision = PROJECT_REMAINING_WORK_TO_DRUDGE     Drudge invocation = MANDATORY
    ELSE:
        direct continuation may be considered

`HOST_DIRECT_CONTINUATION_ALLOWED` is legal ONLY if ALL are true: no active Drudge
run; no prior Drudge handoff / ledger / active run identity; exactly one clear bounded
local change; no multi-stage execution required; no multiple mandatory requirements
remain; no root-level completion judgment required; no cross-session continuation
required; no execution-plan regeneration required. Examples that MAY qualify: fix one
typo, change one known config value, rerun one known test, repair one already-localized
small bug, update one explicit string/value. If the Host cannot positively prove every
direct-continuation condition: `DEFAULT = DRUDGE_REQUIRED`. Uncertainty MUST NOT
default to Host freedom. A released (STOPPED/SUPERSEDED) run does not bind the
structural gate: the explicit escape path returned reasoning control to the Host, and
any successor plan requires a new valid revision/run identity.

`ACTIVE_DRUDGE_RUN` / `CONTINUATION_LOCK`: the exact moment the first execution JSON
becomes active, `ACTIVE_DRUDGE_RUN=YES` and `CONTINUATION_LOCK=ON`. While
`ACTIVE_DRUDGE_RUN=YES` the Host MUST NOT silently replace the accepted task interpretation,
generate a competing JSON chain, create a competing plan for the same root, silently
bypass the active Work Unit, skip unfinished Work Units, replace the
RootRequirementLedger, restart merely because a new Host session/window opened,
continue substantive construction outside the active Drudge run, or use model
capability as a bypass. The Host MUST recover the existing run, validate run identity,
semantic revision and digest, recover the current execution JSON / Work Unit,
requirement coverage and the paired handoff / continuation state. New Host
window/session != new plan: an active run yields `RESUME_EXISTING_RUN`, and the new
Host must receive the same root identity, the same semantic revision, the same
RootRequirementLedger and the exact current Work Unit / continuation point; it must
NOT use a new session to silently replace the accepted plan. Controlled source rereading,
Host re-reasoning and versioned amendment remain legal through the public reflection and
amendment paths. A competing plan
submitted against an active run is rejected and the existing run is preserved.

`RESUME_EXISTING_RUN` does NOT mean `IMMEDIATELY_EXECUTE`. It means
`RESUME_EXISTING_RUN -> ALIGN_EXISTING_RUN -> determine the unique canonical
continuation point -> then execute`. Before ANY substantive continuation work an
existing active run MUST pass the mandatory `ACTIVE_RUN_ALIGNMENT_GATE`: a bounded,
primarily read-only alignment between `root_run_id`, `semantic_revision`, the active
execution JSON, the current Work Unit, the paired handoff state, the continuation
identity, current Step identity/revision/state digest, current real input artifacts
and necessary dependencies. RootRequirementLedger, all sealed evidence and overall
build/test state are verified at formal terminal acceptance, or when a real amendment
or explicit current dependency requires them. Normal alignment does not re-audit
unrelated WUs, completed Steps, historical Capsule bodies or global Evidence.
This alignment is mandatory even when
the Host believes it remembers the task, is highly capable, the current work appears
obvious, or a new Host session has full chat context.

Alignment may use real evidence to recognise genuinely completed work, but it MUST NOT
silently skip a frozen Work Unit, reorder Work Units, generate a new JSON chain,
reinterpret the original root task, duplicate already completed work, assume a prior
Work Unit is complete without evidence, or jump directly to a later Work Unit merely
because later artifacts exist. Exactly one of three legal outcomes is selected:

- `RESUME_CURRENT_WORK_UNIT` — the current Work Unit is not yet complete: the same
  root identity, semantic revision and frozen Work Unit are preserved, existing valid
  progress is preserved and only the unfinished portion continues;
- `COMPLETE_CURRENT_AND_ADVANCE_VIA_CANONICAL_HANDOFF` — real evidence proves the
  current Work Unit is already complete but canonical completion/handoff was not yet
  finalized: completion evidence is validated, coverage is sealed mechanically, the
  EXISTING paired handoff is consumed, exactly-once continuation identity is preserved
  and advancement reaches only the canonical next Work Unit — never a direct jump;
- `STOP_STATE_PLAN_DIVERGENCE` — real project state and frozen execution state cannot
  be reconciled safely (later Work Units appear partially executed while an earlier
  canonical Work Unit is unresolved, artifact state contradicts the frozen plan, a
  semantic revision/digest does not match, or the continuation point cannot be proven
  uniquely): STOP, preserve evidence, do not guess, do not silently reorder, do not
  silently replan, then use the explicit REPLAN / supersede lifecycle if authorized.

The hard execution rule is `ACTIVE RUN -> ALIGN FIRST -> DETERMINE EXACT CONTINUATION
POINT -> EXECUTE`; never `ACTIVE RUN -> immediately start coding`.

Runtime enforcement is part of this contract, not advice. A resumed existing
structured run must align to its exact frozen identity before it may execute. The
alignment binds the exact run identity, the exact semantic revision, the frozen Work
Unit topology digest, the exact current Work Unit index and the exact continuation
identity, and it produces a bounded, Drudge-owned alignment receipt. Execution
without that exact alignment fails closed with `ACTIVE_RUN_ALIGNMENT_REQUIRED`; a
stale semantic revision, a wrong current Work Unit, a wrong continuation identity, a
missing exact identity declaration or a competing plan never silently enables
execution. The receipt carries no semantic content and is not a second state machine.

Continuation lock must not create an impossible dead end; silent replanning is
forbidden. Leaving the active run is allowed only when mechanically proven through the
explicit escape path: the current JSON / handoff explicitly returns `STOP` /
`REPLAN_REQUIRED`; a genuine product-contract decision requires user input; verified
facts prove the current frozen plan is no longer executable; an authority / revision /
digest conflict is detected; or the run is explicitly terminated/superseded through
the canonical lifecycle. The escape preserves old run evidence, marks the old run
`STOPPED`/`SUPERSEDED`, never overwrites the old run, releases the continuation lock
explicitly and returns reasoning control to the Host. Any successor plan must receive a
new valid revision/run identity — no silent plan replacement.

Mechanical surface:

    get_continuation_state()
    resume_preflight(declaration)                     # read-only structural gate
    active_run_alignment(declaration)                 # read-only alignment gate
    release_continuation_lock(reason_code, operation_id, authority_ref)

`WORK_UNIT_COMPLETED=PASS` still never equals `ROOT_TASK_COMPLETE`: while any mandatory
root requirement remains `PENDING`, `ROOT_FINALIZATION` stays forbidden,
`CONTINUE_REQUIRED=YES` and `CONTINUATION_LOCK` remains `ON`. Continuation preserves
the same root identity and the RootRequirementLedger. Drudge adds no semantic
reasoning: the Host supplies structural facts and Drudge mechanically applies the
frozen governance.

## Host construction ownership

The Host and Drudge have distinct, non-overlapping ownership:

    Host   = product reasoning, implementation, tool selection and external
             construction actions within the valid task/scope.
    Drudge = structured capsule storage, semantic execution contract, mechanical
             projection, validation, evidence, seal, handoff, continuation and
             root-completion guards.

Host implementation results must be attributable to the correct declared Work Unit or
root obligation through the existing public mechanisms. The Host must not treat:

    ROOT_BINDING_EXISTS       as equivalent to CURRENT_WU_COMPLETED
    WU_SEALED_PASS            as equivalent to ROOT_TASK_SUCCESS

and must not fabricate execution or validation evidence after the fact. Construction
stays inside the declared task/scope and the canonical execution progression; the
contract fixes the authority and the ownership boundary, and does not prescribe an
additional implementation-order ceremony beyond that.

## Capsule quality

Capsule quality hardening is mechanical. Drudge verifies the shape, identity and
revision of a Host declaration; it never reads prose, never compares the meaning of
two sentences, never scores a statement and never decides whether a declaration is
semantically true. The capsule contract stays `onhand.semantic_capsule.v2`; a v2
capsule without any quality metadata remains a valid, readable, immutable historical
record and is never retroactively rewritten.

### Global source fidelity

Global generation MUST preserve the source modality, and a Host inference MUST NOT be
promoted into a source-authorized binding obligation. Before Global may be frozen the
Host MUST submit a bounded source-fidelity declaration through
`accept_global_hybrid(..., fidelity_review=...)`. The declaration is a closed vocabulary
that covers, at minimum:

- `source_strength_preserved`
- `no_preference_to_requirement_strengthening`
- `no_should_to_must_strengthening`
- `no_host_inference_promoted_to_obligation`

Freezing Global without that completing declaration fails closed with
`SOURCE_FIDELITY_REVIEW_MISSING`. A declaration that does not bind the exact current
Memory authority fails closed with `SOURCE_FIDELITY_REVIEW_STALE`. A declaration that
itself reports an unauthorized strengthening fails closed with `SOURCE_FIDELITY_UNRESOLVED`
and Global is not frozen. A useful Host inference may remain `NONBINDING_HOST_INFERENCE`,
but it MUST NOT silently masquerade as a source-authorized binding requirement. Drudge
stores a small digested quality receipt and never judges the declaration's truth.

### Decomposition coverage

Decomposition MUST account for root/global obligations and MUST explicitly distinguish
fragment ownership from retained root and cross-cutting invariants.
`declare_root_obligations(obligation_ids)` declares the closed obligation universe bound
to the current Global, and an `accept_decomposed_hybrid(..., coverage=...)` declaration
classifies each declared obligation as exactly one of:

- `OWNED_BY_ONE_FRAGMENT`
- `OWNED_BY_MULTIPLE_FRAGMENTS`
- `ROOT_INVARIANT`
- `CROSS_CUTTING_ROOT_INVARIANT`

Every declared obligation must receive exactly one classification. While a declared
obligation is unaccounted, decomposed state is not quality-complete and
`bind_semantic_work_units(work_units)` fails closed with `DECOMPOSITION_COVERAGE_INCOMPLETE`.
A root invariant needs no artificial fragment owner, and an obligation shared across
fragments MUST NOT be duplicated into every Work Unit. A declaration whose `global_ref` is
not the current Global fails closed with `QUALITY_RECEIPT_REVISION_MISMATCH`. Quality use
stays proportional to personal OnHand work: no ontology, no scoring and no per-sentence
bureaucracy.

### First projection binding

There is no initial projection MD bridge and none is required. The canonical first
projection is the real closed-membership path: accepted decomposed capsules declare closed
`required_work_unit_ids`, and `bind_semantic_work_units(work_units)` freezes both the
per-scope execution contracts and the complete GLOBAL/root projection binding — run
identity, effective semantic revision, Global Hybrid reference, the relevant decomposed
references, the bound `work_units_digest` and an immutable binding identity. The first
execution must be mechanically provable to belong to that frozen semantic projection;
first execution is never the event that creates this truth, and an unbound or mismatching
topology fails closed with `FIRST_PROJECTION_AUTHORITY_NOT_PERSISTED`.

### Residual trust boundary

The Host owns every semantic judgment and owns making the declarations this contract
requires. Drudge mechanically proves that a required declaration was made, that it bound
the exact current identity and revision, that it is internally consistent and that it is
complete over the Host's own declared obligation universe. Drudge cannot and does not
prove that a Host semantic judgment is true; a Host that declines to declare its
obligation universe has no mechanically checkable obligation to account for, and that
residual gap is stated here rather than hidden.

Mechanical surface:

    declare_root_obligations(obligation_ids)
    get_capsule_quality_state()

## Opaque Hybrid outlet v2

`HybridOutlet(dispatch, reconcile=None)` implements `onhand.hybrid_capsule.v2`.
Bind it with `bind_hybrid_outlet(outlet)`. The sole event is `HYBRID_CAPSULE`, with
classification `GLOBAL` or `DECOMPOSED`, canonical `body_md`, exact capsule
reference, run ID, minimum provenance and stable `output_id`. Memory is private,
cannot be an output kind, and its text cannot be embedded in Hybrid bodies.
Drudge knows only this opaque contract.

Every accepted Global revision becomes eligible immediately. Decomposed output
requires its persisted scope completion gate. Logical output ID hashes run,
kind, capsule identity, revision, body digest and outlet version; no sequence
counter participates. Atomic `OUTPUT_INTENT_RECORDED` precedes dispatch.
The callback returns `{"output_id": ..., "status": "CONFIRMED"}` only for a
trustworthy idempotent acknowledgement. Any exception, absent acknowledgement,
wrong identity or interrupted dispatch yields `UNKNOWN_OUTPUT_OUTCOME`.
`recover_capsule_outputs()` recovers pending checkpoints. An optional trustworthy
`reconcile(output_id)` returns `CONFIRMED`, `NOT_DELIVERED`, or `UNKNOWN` with
the same ID. Only confirmed non-delivery permits retry of that same logical ID.
Unreconcilable outcomes remain closed; no blind retry occurs.

V2 semantic operations are listed in `public.V2_OPERATIONS` and accept
`onhand.semantic_capsule.v2` envelopes through `invoke`. Envelope digests cover
all fields except `request_digest`; `expected_run_revision` is the current
Global revision or null before Global exists. Durable receipts resolve canonical
bodies by reference, never store editable semantic copies. An interrupted
envelope intent fails closed. Opaque adapters are bound through the direct
surface methods. Legacy numbered L1/L2/L3 and Root/Authority submission surfaces
are unavailable on v2 runs.

## Public entry

Host discovery loads `onhanddrudge.invoke` and calls `invoke(surface, request)`, where `surface` is a `PublicHostSurface` rooted at the caller's discovered `ONHAND_SKILL_AREA/OnHandDrudge/` home. The dispatcher delegates to one Core and exposes versioned operation envelopes from `foundation.OPERATIONS`.

## Input/output

Requests contain `contract_version`, `operation_id`, `operation_kind`, `task_run_id`, `expected_run_revision`, `request_digest`, and `payload`. Results are structured dictionaries or Core operation results. Missing semantic Root Capsule returns `REQUEST_CAPSULE_TO_HOST`; Host supplies capsules and judgments explicitly.

## Standalone mount

The standard mount is a thin `SKILL.md` plus `onhanddrudge.invoke` launcher pointing to the canonical OnHandDrudge Core. A caller supplies the runtime home; no machine parent path is embedded.

## Failure behavior

A mechanically detected inconsistency fails closed at the relevant existing mechanical
boundary: the operation raises its typed error and no terminal record is written. The
Host must not work around the failure.

- projection identity mismatch -> `PROJECTION_DIGEST_MISMATCH`, `PROJECTION_NOT_ACTIVE`,
  `FIRST_PROJECTION_AUTHORITY_MISMATCH`;
- stale semantic revision -> `STALE_SEMANTIC_REVISION`, `INVALID_SEMANTIC_REVISION`,
  `STALE_CAPSULE_REVISION`, `STALE_SEMANTIC_REFERENCE`, `CAPSULE_DIGEST_MISMATCH`;
- missing or inconsistent binding -> `FIRST_PROJECTION_AUTHORITY_NOT_PERSISTED`,
  `INVALID_SEMANTIC_REFERENCE`;
- continuation alignment failure -> `ACTIVE_RUN_ALIGNMENT_REQUIRED`,
  `MALFORMED_ALIGNMENT_FACTS`, `STOP_STATE_PLAN_DIVERGENCE`;
- invalid or stale handoff -> `MALFORMED_PAIRED_HANDOFF`, `PAIRED_HANDOFF_REQUIRED`,
  `HANDOFF_DIGEST_MISMATCH`, `HANDOFF_IDENTITY_MISMATCH`, `HANDOFF_NOT_CONSUMED`,
  `SAME_SESSION_SUCCESSOR_FORBIDDEN`;
- incomplete closed membership -> `CLOSED_MEMBERSHIP_INCOMPLETE`,
  `INVALID_CLOSED_MEMBERSHIP`, `DECOMPOSITION_COVERAGE_INCOMPLETE`,
  `DECOMPOSITION_COVERAGE_DECLARATION_MISSING`;
- incomplete mandatory root requirements -> `ROOT_FINALIZATION_REJECTED:CONTINUE_REQUIRED`,
  `CONTINUE_REQUIRED`, `ROOT_REQUIREMENTS_NOT_DECLARED`, `STALE_COVERAGE_REVISION`;
- genuine contract-change blockers -> `USER_DECISION_REQUIRED` /
  `BLOCKED_CONTRACT_CHANGE`, surfaced with `REQUEST_CAPSULE_TO_HOST`;
- insufficient or invalid evidence -> `CURRENT_EVIDENCE_REQUIRED`,
  `CURRENT_EVIDENCE_INVALID`, `FINAL_EVIDENCE_INVALID`, `SEALED_EVIDENCE_INVALID`.

Expected public rule: fail closed at the relevant existing mechanical boundary. The Host
must not silently reconstruct a competing plan, invent replacement identities, bypass
stale-revision detection, fabricate PASS evidence, or treat partial completion as root
success. Ambiguous external outcomes fail closed with `UNKNOWN_OUTPUT_OUTCOME` and are
never blindly retried. These are existing identifiers; this contract creates no new
terminal code.

## Legacy v1 private Authority and fresh L3 recovery

Existing legacy data is recognized only by `identify_historical_run(root, task_run_id)`
as read-only history. New public surfaces use the discovered provider home and an exact
ticket; they do not accept an arbitrary historical private run path. No automatic migration
or private-record copying is performed.
`submit_authority_capsule` uses `onhand.public.v1`. Compute `request_digest`
with `foundation.digest` over the entire envelope excluding `request_digest`.
Its payload has exactly `content` (the unmodified user string), `provenance`
(`source`, `user_input_id`, `received_at`, each a nonempty string),
`change_kind` (`INITIAL`, `ADDITION`, `CHANGE`, `OVERRIDE`), and
`expected_authority_revision` (null initially, then the predecessor integer).
`expected_run_revision` is null before a ROOT_CAPSULE exists and otherwise the
current semantic revision. An `operation_id` identifies immutable facts;
identical replay returns the original receipt and conflicting reuse fails.
Submission returns only status, Authority revision, and digest.

Authority records in `authority/` and ROOT snapshots in `root_capsules/` are
private. Direct `request_l3_host_rereasoning` returns `current_authority`, all
`authority_lineage`, the current `root_capsule`, and `evidence_refs`. The latest
Authority revision identifies the effective lineage; Core preserves every
input exactly and never merges or interprets additions, changes, or overrides.
The Host selects the successor ROOT_CAPSULE and supplies continuation Work Units
explicitly through the existing current-only JIT lifecycle. Runs without
Authority remain legal and report `chat_memory_independent_recoverability=False`.
Authority is never optional output; event kinds remain exactly `ROOT_CAPSULE`,
`WORK_UNIT_COMPLETED`, and `RUN_FINALIZED`.

Malformed, stale, conflicting, unsupported, or out-of-scope requests fail closed with a typed `ValueError`/`RuntimeError`. Optional sink delivery failures are recorded and do not roll back execution or finalization.

## Smoke test

Validation must run against the exact candidate revision and record real positive and
negative traces. A local Core process proves Core behavior only. A standalone Host smoke
requires an actual independent Host session to discover this public surface, explicitly
invoke it, receive its result and recover the same ticket. If that Host capability is
unavailable, report BLOCKED rather than claiming Host smoke PASS.

## Host non-responsibilities

The Host must not duplicate or take ownership of:

- a second capsule persistence authority — accepted capsules and their digests remain
  Drudge-owned;
- a competing execution projection authority — the frozen root binding and the
  per-Work-Unit execution projection remain Drudge-owned;
- manual rewriting of frozen digests — frozen projections, bindings and scope execution
  contracts are immutable;
- direct mutation of sealed evidence — seals are immutable;
- invented continuation or handoff identity — continuation identity stays
  deterministic/content-addressed and must not be re-created;
- bypassing the existing mechanical finalization guards — root finalization stays gated
  by declared mandatory requirement coverage;
- an independent replacement `RootRequirementLedger` — the canonical ledger under
  `runtime_state/root_requirements.json` is authoritative.

The opposite boundary is preserved: the Host retains semantic reasoning, technical
design, product construction and evidence interpretation, and these must not be delegated
to Drudge. Drudge performs no semantic reasoning, ranking or inference. Capability
selection decisions (WHO) and execution-location / governance decisions (WHERE/HOW) are
not Drudge responsibilities and are not moved into Drudge by this contract.

## Mandatory paired terminal handoff

For each non-final current Work Unit the Host declares `paired_handoff` with an exact
contained `path`. A new plan may use `digest_status="DEFERRED_RUNTIME_EVIDENCE"`;
the actual MD is generated only after the current Steps pass and its raw-byte SHA256
is then sealed and consumed. An already-existing exact MD may instead supply `sha256`.
The runtime never guesses a file or requires the digest of a future ungenerated MD.
The MD contains one fenced `json` block with `contract_version` equal to
`onhand.paired_handoff.v1`, `task_run_id`, `work_unit_id`, `work_unit_revision`
(default Work Unit `revision` is zero), `next_work_unit_digest` (foundation
`digest` of the declared successor identity), `host_action` equal to
`CREATE_INDEPENDENT_TASK`, `declared_next_entry`, and the exact `instruction`.
Work Unit `is_final` must agree with declared final topology; when omitted,
the last declared unit is final. Final units consume no paired next handoff.

Execution writes PASS evidence, then a digested zero-blocker seal, then validates
and consumes this MD before returning a non-final terminal effect. Missing,
malformed, mismatched or unvalidated MD blocks completion. Receipts under
`runtime_state/handoff-<index>.json` contain the five mandatory handoff facts,
the exact MD digest, deterministic operation identity, consumption count,
predecessor seal and Host request. `consume_paired_handoff` replays that exact
receipt with `IDEMPOTENT_STOP`; it never creates another successor.

`PublicHostSurface(..., host_transition=callable)` invokes a Host adapter with
the declared request. The adapter creates an independent task and returns
`{"status":"CONFIRMED_CREATED"}` with its receipt, or `DEFINITE_FAILURE` /
`UNKNOWN`. Exceptions are unknown Host outcomes. Dispatch intent is persisted
before the call; ambiguous recovery stops without retry. Without an adapter,
the exact manual instruction is returned in the public result. Creation failure
is a Host outcome, not execution corruption. No same-session fallback is legal.

A fresh Host session opens the same run root and unchanged declared Work Units
with the returned `continuation_operation_id`. The default session identity is
process-scoped, so a new object in the same process cannot bypass the stop.
An embedding Host may supply its independent task's `session_id`; it must never
assign a new identity to the old task to bypass this boundary. The runtime blocks
old-session materialization and execution, including replay of ACTIVE evidence.
Non-final `finalize_run(SUCCESS)` returns the handoff terminal effect; only final
validated evidence permits root SUCCESS. Private capsules remain outside events.

The Host owns semantic judgment, workflow continuation, and deciding the next public operation. The Skill does not schedule, poll, rank, infer topology, or pre-generate future Work Unit products.

## Candidate V2 combined wiring

`REVIEWED_ON_DEMAND_DIRECT_HOST_RETURN_ISOLATED`

WU09 post-implementation Wiring Review validates the same candidate Core through
the standalone public entry and the real Thoroughfare mechanical B0003 route in an
isolated environment. The reviewed topology is ON_DEMAND / DIRECT_HOST_RETURN:
socket B0003, provider OnHandDrudge, implementation reference
OnHandDrudge/onhanddrudge/invoke.py. The observed production binding epoch is 4;
it remains a read-only identity fact, not permission to update registration.

The original unpatched route passes fresh-ticket invocation but does not forward
continuation_operation_id or drudge_run_locator to the provider constructor.
Existing-ticket continuation therefore stops. Production-native continuation
compatibility is not claimed.

The separately isolated external mechanical-edge patch forwards the caller's
original continuation_operation_id and drudge_run_locator unchanged. The locator
has exactly contract_version=onhand.drudge_run_locator.v1, task_run_id and the
actual independent Host session_id; it contains no caller-supplied storage path.
The Core resolves the ticket under the explicitly discovered provider home,
recovers the complete bound units, and requires alignment before the next legal
Step. Actual independent Host tests validate two-ticket continuation, exact
U1/S01 replay, and wrong-token, cross-ticket and old-session STOP behavior.

Host discovery, semantic decisions, workflow continuation, tool execution and
reconnection remain Host-owned. No automatic reconnect, semantic routing, daemon,
new registry or persistent Host setting is introduced. Standalone requires no
Thoroughfare and retains its existing thin public entry to the same Core.

External route deployment remains `PENDING_SEPARATE_DEPLOYMENT`. This review does
not install the candidate, alter production Binding Truth or AVAILABLE_SKILLS,
or approve a production switch. Production installation and external-edge
deployment require separate direct user approval and compatible reviewed public
contract/registration checks. The isolated result is not a claim that the active
production route has already been patched.

## Standalone public Core wiring

Host discovery creates `PublicHostSurface` with the caller-discovered
`ONHAND_SKILL_AREA/OnHandDrudge/` runtime home and invokes the canonical
`onhanddrudge.invoke.invoke(surface, request)` entry point. The surface
delegates lifecycle operations to one Core and returns structured results;
the Host retains semantic judgment, continuation, and next-operation choice.

## Host planning and source authority

At first planning the Host briefly records SPLIT or UNSPLIT and its reason, considering
responsibility boundaries, recoverability, carried context and handoff cost. Prefer
considering a split when isolated delivery has clear benefits; do not force a number of
Work Units. Model strength alone is not a sufficient reason to avoid splitting. Dense
shared context or disproportionate handoff cost can justify an unsplit plan. Light tasks
may be handled directly under the declared light-task admission.

The first `accept_global_hybrid` includes this declaration inside its existing
`fidelity_review`; the existing immutable fidelity receipt retains it:

```python
fidelity_review["planning_declaration"] = {
    "decision": "UNSPLIT",
    "reason": "One bounded responsibility; shared context and handoff cost exceed isolation benefit."
}
```

Missing or malformed first-planning declarations fail closed with
`HOST_PLANNING_DECLARATION_MISSING` or `MALFORMED_HOST_PLANNING_DECLARATION`.
Drudge checks only the declaration's presence and shape. It neither scores economics,
judges the reason nor forces N. A nonempty but poor explanation remains the Host's
responsibility. Existing immutable Global revisions stay readable; successors follow
the existing fidelity, expected revision and amendment guards.

Memory retains exact user text and provenance. Global keeps USER AUTHORITY,
HOST STABLE DECISION and UNRESOLVED distinct. Preserve MUST, MUST NOT, preferred,
suggested and optional strength. If the Host adopts a suggestion, label it a
HOST STABLE DECISION; do not invent a USER MUST. Before commit, compare Global with
Memory using the existing source-fidelity declaration. Mechanical receipts prove a
declaration exists, not natural-language correctness.

Controlled Host re-reasoning uses `reflect_semantic_ancestry`,
`request_global_reflection`, `submit_global_reflection_result`, and authorized
`request_memory_recovery`. Explicit user changes use `amend_user_requirements`
with immutable Memory additions, expected revisions, evidence applicability and the
existing committed revision combination. A frozen JSON successor changes the same WU;
future-tail amendment changes only unexecuted responsibility. Never overwrite old
capsules, execution JSONs, seals or consumed handoffs.

## Ticket-only standalone recovery and Step execution

Standalone needs no Thoroughfare, BestFit or RecallLedger. Discover this SKILL.md and
import the adjacent thin `onhanddrudge.invoke` entry pointing to the single Core.
The Host explicitly supplies the discovered provider home and exact `task_run_id`.
New runs use `PublicHostSurface(home, task_run_id=ticket, session_id=actual_session,
capsule_contract_version="onhand.semantic_capsule.v2")`. The home must exist; each
ticket maps deterministically to its own `DRUDGE_PRIVATE_RUNS` child. Invalid tickets,
unknown tickets, duplicate creation, traversal and reparse roots STOP.

`recover_ticket(home, ticket, offset=0, limit=100)` returns the current binding and
current execution point with bounded WU pagination. It does not rebuild history.
Use `history=True` explicitly when a full historical query is needed. That query returns read-only capsule refs,
full bound WUs in absolute order, current WU/Step/JSON, progress, evidence and handoff
facts. Follow `next_offset` until the complete membership is recovered; never recreate
WU contracts from chat. `read_ticket_capsule(home, ticket, ref, offset=0, limit=65536)`
returns bounded body pages with the exact ref and digest. Read Memory, Global, window
capsules, bound WUs and current runtime in that order as needed.

Reopen the same ticket with `run_access="RESUME"`, the recovered complete `work_units`,
the actual independent `session_id`, and the existing `continuation_operation_id` when
crossing a consumed handoff. Read-only recovery does not authorize execution. Perform
`resume_preflight` and exact `active_run_alignment` before resumed execution. Changing
window or model never auto-invokes Drudge and never advances a WU. Stronger and weaker
models differ only in Host judgments and tool choices; the same Core guards apply.

Use `project_current_step` to freeze one complete current-WU execution JSON and receive
only its next legal Step. Use `register_step_attempt` before executing tools, then submit
real result/evidence with `submit_step_result`. Bind ticket, WU, Step, JSON revision and
attempt identity exactly. Unknown side effects STOP pending Host verification; do not
replay tools. Confirmed safe retry or same-WU `submit_json_successor` preserves prior
records. Each WU PASS is distinct from Root SUCCESS; declare current mandatory coverage
(including explicit zero mandatory) and pass the existing Root completion guard.

A new Host taking an unexecuted `STEP_READY` must first pass `active_run_alignment`
for its actual ticket/session and current binding, then call `resume_current_step`
with `task_run_id`, `operation_id`, `projection_id`, `projection_revision`,
`projection_digest`, `work_unit_id`, `step_id`, `step_index`, `expected_state_digest`,
and a run-private `evidence_ref`/`evidence_sha256` for the Host's takeover declaration.
The existing short execution lock atomically transfers the owner; frozen JSON is
unchanged. Old owners, stale declarations, pending attempts and uncertain effects
STOP. This is an owner transfer, without a Step failure or retry classification.

Public `invoke` envelopes contain `contract_version`, `operation_id`, `operation_kind`,
`task_run_id`, `expected_run_revision`, `request_digest`, and `payload`. V2 semantic
operations use `onhand.semantic_capsule.v2`; mechanical operations use `onhand.public.v1`.
Compute `foundation.digest` over all fields except `request_digest`. Missing, partial,
unknown, unsupported, stale, conflicting or malformed requests STOP at their relevant
mechanical boundary. Structured return values are mechanical facts, not semantic PASS.
`resume_current_step` is also a formal `onhand.public.v1` operation. Its payload is
`{"declaration": <run-private Host declaration>}` and the declaration must carry the
exact `task_run_id`, `operation_id`, projection/Step identity, `expected_state_digest`,
run-private `evidence_ref` and matching `evidence_sha256`, `side_effect_status` equal to
`VERIFIED_NO_EFFECT`, and `safe_to_retry=true`. The active Run/WU/Attempt identity,
work-unit digest, continuation/session guards, short execution lock, and existing
`JITCore.resume_step` checks remain authoritative. Only `EXECUTION_STARTED`,
`STEP_FAILED`, or `STEP_PAUSED` states may cross the safe retry boundary; uncertain
side effects, stale identity, wrong revision/digest, missing evidence, old sessions,
unknown operation IDs, and malformed envelopes STOP. An identical declaration replay
under the same operation ID is idempotent; a changed declaration under that ID is an
`OPERATION_ID_CONFLICT`. The operation returns `STEP_READY` and never executes a Host
tool or edits prior Attempt records. Host owns evidence and tool execution; Drudge owns
mechanical validation, receipts, state transition, and recovery gates.

The remaining public recovery functions are direct helpers unless separately listed in
this envelope operation table; no JSON action executor is implied.
