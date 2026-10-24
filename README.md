# OnHandDrudge

English · [简体中文](README.zh-CN.md)

**OnHandDrudge 1.0.0 — First Official Bilingual Release under OnHand Core.** [Official download and release notes](https://github.com/onhandcore/OnHandDrudge/releases/tag/first-public-bilingual-20261024). This is the current rewritten version’s first official bilingual release under the new OnHand Core distribution system. The old onhandcore-source account previously published v0.1.0; this is not the first public release in the product’s entire history. RC-R01 identifies the independently reviewed sealed build, not a GitHub prerelease. Candidate wording in the unchanged ZIP describes its preparation stage. See [official release notes](RELEASE_NOTES.md).

OnHandDrudge is a **structured execution governance Skill for complex Agent tasks**. The Host handles reasoning, planning, and tool execution. Drudge handles mechanical checks, frozen execution identities, Evidence, recovery, and handoff.

## Why use Drudge?

Long conversations, failed attempts, and window changes can leave an Agent uncertain about its exact execution point. Drudge binds Host-declared scopes, Work Units, the current Step, Attempts, and completion evidence to persistent identities and digests. It checks what may execute next, whether evidence matches the frozen identity, and whether local completion qualifies the root task for success.

Drudge does not replace Agent planning or judge the meaning of natural language. The Host must declare requirements correctly, interpret evidence, and choose the next action. A mechanical receipt cannot prove those judgments correct or discover every undeclared omission.

## Capabilities and responsibilities

- **Semantic capsules:** Memory preserves exact user text and provenance. Global Hybrid records the Host's task-wide interpretation. Decomposed Hybrid represents real subscopes and ancestry. A Work Unit is an execution projection, not a fourth capsule class.
- **JIT and freezing:** the declared topology is bound first. The current Work Unit's execution JSON is then materialized and frozen, exposing only the next legal Step. Future Work Unit execution products are not generated ahead of time.
- **Steps and Evidence:** an Attempt is registered before action. The Host executes the tool and submits its actual result and evidence. Revisions, digests, and seals bind execution identities; historical records cannot be overwritten to manufacture success.
- **Recovery and handoff:** an exact ticket recovers the binding and execution point. Preflight, Alignment, and side-effect verification are required before continuation. Non-final Work Units use paired handoff and continuation identities, with a real independent Host task taking over.
- **Root completion:** WU PASS is not Run SUCCESS. The Host declares and reconciles the actual mandatory requirements; Drudge checks the corresponding mechanical eligibility.

The Host retains semantics, decomposition, tool choice, authorization, execution-location decisions, evidence interpretation, and successor-task creation. Drudge does not bundle Skill selection, installation, a long-term knowledge base, parallel-agent scheduling, or a background service.

## Installation and actual entry points

This is one ZIP with a single `OnHandDrudge/` product root containing three thin Python entry surfaces, nine bytecode implementation members, the formal contract, and bilingual documentation. Python and other Skills are not bundled. Initial validation targets **Windows x64 / CPython 3.12.14**, bytecode magic `cb0d0d0a`, optimization level 0.

1. Verify the ZIP against the external `SHA256SUMS.txt`, then extract it completely to a directory you choose. The product home is the extracted `OnHandDrudge/`, not the ZIP, its `onhanddrudge/` package, or an old Run.
2. Let the Host discover [SKILL.md](SKILL.md) at that home. Python's import path must include the home. The Host needs read access to runtime members and permission to create, replace, and read private state under `DRUDGE_PRIVATE_RUNS/`, including associated lock and temporary files. Host configuration is not changed automatically.
3. **Standalone:** import `PublicHostSurface` and `invoke` from `onhanddrudge.invoke`, supplying the home, exact ticket, and appropriate real Host session identity. Thoroughfare, BestFit, and RecallLedger are not required. `invoke.py` is an import entry, not an arbitrary-JSON command-line executor.
4. **Combined:** install a compatible OnHandThoroughfare separately, complete its formal wiring review and registration lifecycle, and invoke Drudge through its public transport. The entry remains `OnHandDrudge/onhanddrudge/invoke.py`, using ON_DEMAND / DIRECT_HOST_RETURN. Do not copy production Bindings or another session's attachment into installation configuration. Discovery does not guarantee proactive Host invocation.

See [INSTALL.md](INSTALL.md) for detailed steps, interpreter checks, and a runnable Standalone example. The constructor shape is:

```python
from onhanddrudge.invoke import PublicHostSurface, invoke

surface = PublicHostSurface(
    discovered_product_home,
    task_run_id=new_exact_ticket,
    session_id=actual_host_session,
    capsule_contract_version="onhand.semantic_capsule.v2",
)
# Follow SKILL.md: submit capsules, bind Work Units, project, register an Attempt,
# execute the tool, and submit the actual result.
```

Operation envelopes carry the contract, operation/ticket identities, expected revision, payload, and request_digest computed with `foundation.digest`. Combined surface_construction is a separate transport field, not part of the operation envelope. Recovery uses RESUME and the complete recovered work_units; crossing a consumed handoff also requires its continuation identity. Do not CREATE again or invent a new session.

## Validation and limitations

Existing Windows/Codex acceptance covers real independent-Host Standalone, Combined, cross-session G2, and ExecutionRecovery. Later P1/P2 maintenance has targeted regressions and actual production smoke evidence. This candidate reuses unaffected business evidence and adds actual ZIP extraction, bytecode loading, a minimal Host action/recovery flow, and packaging checks for P1/P2, Alignment, and ExecutionRecovery. The full V2 RC suite has not been rerun.

The additional Combined check runs the unchanged deployed loader against the isolated candidate with explicit test Host/fabric ports. It is not a new production-session E2E and does not modify formal B0003. See [RELEASE_NOTES.md](RELEASE_NOTES.md).

Drudge E2E on WorkBuddy, Linux/macOS, other Hosts, clean machines, other Python versions, and cross-device portability remain unverified. This bytecode is not a general Python >=3.10 distribution; other 3.12 patch versions have not been tested with this candidate. Host models and tools may have their own costs.

Unknown side effects, external outputs, or task-creation outcomes require verification before proceeding, never blind retry. Automatic semantic repair, automatic cross-Host continuation, and automatic reconnection are not promised. Legacy v1 Runs are not converted in place to v2. Bytecode is not a guarantee against decompilation.

Before uninstalling, stop active calls and confirm quiescence. Remove the thin discovery entry through the Host's own mechanism for Standalone; use Thoroughfare's formal removal lifecycle for Combined. The user decides whether to retain or delete private Runs and Evidence; they must not be circulated as distribution files. No one-click uninstaller is provided.

## License and availability

The [LICENSE](LICENSE) is the OnHand Family Closed-Source Distribution License. It permits installation, operation, and free redistribution of complete, unmodified official distributions. It grants no rights to modify, adapt, or repackage the materials, or to distribute modified, adapted, or repackaged versions. Distribution does not disclose first-party Core source code. Existing lawful grants, independent third-party licenses, and rights that cannot legally be excluded remain unaffected. The Chinese text governs; English is a convenience translation. [Third-party notices](THIRD_PARTY_NOTICES.md) reflect this package's actual boundary.