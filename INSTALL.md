# Install, connect, run / 安装、连接与运行

[English overview](README.md) · [中文介绍](README.zh-CN.md) · [Contract / 合同](SKILL.md) · [License / 许可](LICENSE)

## Runtime and extraction / 环境与解压

Use Windows x64 and CPython 3.12.14. The bytecode cache tag is cpython-312 and magic is cb0d0d0a; other versions/implementations are not validated. Python is not bundled. No pip installation or third-party runtime package is required by this ZIP.

使用 Windows x64 / CPython 3.12.14。本包不含解释器，不需要 pip 安装本包或另装第三方运行库；其他版本/实现未验证。

Verify the ZIP using `Get-FileHash -Algorithm SHA256 <ZIP>` and the external SHA256SUMS.txt, then extract the entire archive. Its layout is `chosen-directory/OnHandDrudge/{SKILL.md,onhanddrudge/,...}`. Add the extracted **OnHandDrudge root** to the importing process's sys.path; do not use the inner package directory. FILES.sha256 covers every payload file except itself; the external ZIP digest covers it too.

使用外部 SHA256SUMS.txt 核对 ZIP。完整解压后，将产品根目录加入调用进程的 sys.path，不要使用内部 onhanddrudge 目录。FILES.sha256 不自引用，ZIP 外摘要覆盖整个 ZIP。

The Host needs read access to the product and permission to create/read/write/flush/fsync/replace/delete its run-private files, locks, and temporary files under `OnHandDrudge/DRUDGE_PRIVATE_RUNS/`. Do not place the product under a read-only location. No background service or automatic Host configuration is installed.

Host 须能读产品，并在产品 home 下的 DRUDGE_PRIVATE_RUNS 内完成私有状态、锁及临时文件操作；不要安装在实际只读目录。本包不安装后台服务或自动修改 Host 配置。

## Standalone example / 独立运行示例

Let your Host discover the product's SKILL.md through that Host's supported discovery mechanism. The Python API is `onhanddrudge.invoke`; running invoke.py as an arbitrary-JSON CLI is not supported. Save the following as **host_example.py outside the product directory**, then run it with the validated interpreter and extracted home as its sole argument. It creates one fresh synthetic ticket, registers an Attempt before an actual marker-file action, submits evidence, declares the marker requirement satisfied, and finalizes the test Run. Do not use it to resume a business ticket.

按 Host 正式发现机制暴露 SKILL.md。将下面代码保存为产品目录外的 host_example.py，以受验证解释器执行，并将解压出的产品 home 作为唯一参数。示例新建合成票据，动作前登记 Attempt，实际写入标记、提交证据并核销标记要求，最后完成测试 Run；不能用于恢复业务票据。

```python
import hashlib, json, os, sys, uuid
from pathlib import Path

home = Path(sys.argv[1]).resolve()
sys.path.insert(0, str(home))
from onhanddrudge.invoke import PublicHostSurface, invoke, recover_ticket
from onhanddrudge.foundation import digest
from onhanddrudge.capsules import QUALITY_CONTRACT, SOURCE_FIDELITY_BOOLS

ticket = "example-" + uuid.uuid4().hex
session = os.environ.get("CODEX_SESSION_ID") or os.environ.get("CODEX_THREAD_ID")
kwargs = {"session_id": session} if session else {}  # otherwise Core uses process identity
s = PublicHostSurface(home, task_run_id=ticket,
    capsule_contract_version="onhand.semantic_capsule.v2", **kwargs)
request = dict(contract_version="onhand.semantic_capsule.v2", operation_id="memory",
    operation_kind="submit_memory_capsule", task_run_id=ticket,
    expected_run_revision=None, payload=dict(content="Verify a local marker.",
    provenance=dict(source="user", user_input_id="example", received_at="example")))
request["request_digest"] = digest(request)
invoke(s, request)
binding = s.capsules.memory_binding()
review = dict(contract_version=QUALITY_CONTRACT, memory_source=binding,
    planning_declaration=dict(decision="UNSPLIT", reason="One local marker check"),
    **{key: True for key in SOURCE_FIDELITY_BOOLS})
g = s.accept_global_hybrid(body_md="# Local verification\nCreate and read back a marker.",
    memory_source=binding, fidelity_review=review)
ref = s.capsules.ref(g)
s.bind_semantic_work_units([dict(id="verify", is_final=True, semantic_scope_ref=ref,
    steps=[dict(step_id="marker", instruction="Write and verify a local marker")])])
view = s.project_current_step()
attempt = s.register_step_attempt(view)
out = s.root / "host_evidence"
out.mkdir(exist_ok=True)
marker = out / "marker.txt"
with marker.open("xb") as f:
    f.write(b"marker verified\n"); f.flush(); os.fsync(f.fileno())
hash_file = lambda p: hashlib.sha256(p.read_bytes()).hexdigest()
evidence = out / "marker-result.json"
evidence.write_text(json.dumps(dict(marker_sha256=hash_file(marker),
    attempt_id=attempt["identity"]["attempt_id"])), encoding="utf-8")
s.submit_step_result(view, attempt, dict(ok=True, blocker_count=0,
    evidence_ref=str(evidence), evidence_sha256=hash_file(evidence)))
applicability = dict(semantic_binding=ref, artifact_version="example-1",
    artifact_ref=str(marker), artifact_sha256=hash_file(marker),
    evidence_ref=str(evidence), evidence_sha256=hash_file(evidence))
s.submit_root_coverage(dict(operation_id="coverage", semantic_binding=ref,
    expected_coverage_revision=None, mandatory_requirement_ids=["marker"],
    zero_mandatory=False, requirements={"marker":dict(requirement_id="marker",
    status="SATISFIED", evidence_ref=str(evidence), last_work_unit_id="verify",
    applicability=applicability)}))
assert s.get_root_finalization_eligibility()["status"] == "ROOT_FINALIZATION_ALLOWED"
result = s.finalize_run("SUCCESS", run_mode="SEMANTIC", semantic_revision=0,
    final_evidence_refs=[str(evidence)], finalization_id="example-final")
print(json.dumps(dict(ticket=ticket, run_home=str(s.root), result=result,
    recovery=recover_ticket(home, ticket, history=True)), default=str))
```

Example command / 命令示例：

```powershell
& 'C:\Path\To\Python312\python.exe' -I -B .\host_example.py 'C:\Skills\OnHandDrudge'
```

Host session identity is supplied from a real session when available; the default process identity is not an independent successor-task adapter. The example is synchronous local API use by a Host, not a background Agent or automatic planner.

有真实 Host session 时提供该身份；默认进程身份不代表独立后继任务适配器。示例是 Host 同步调用本地 API，不是自动规划器或后台 Agent。

## Recovery / 恢复

Use `recover_ticket(home, exact_ticket)` for current state, or `history=True` with next_offset pagination for history. Reopen with `run_access="RESUME"` and the complete recovered work_units. Perform resume_preflight and active_run_alignment before continuation. A consumed handoff requires its exact continuation_operation_id and a real independent Host session. Read-only recovery does not authorize tools. Never infer WUs from chat, invent a new identity, or retry an unknown side effect.

用准确 ticket 读取当前状态；历史查询显式 history=True 并完成分页。以 RESUME 和完整 work_units 重开，先 preflight/Alignment 再继续；已消费交棒需原 continuation 身份和真实独立会话。只读恢复不授权执行工具。不可从聊天重建身份或盲目重试未知副作用。字段及 typed STOP 以 SKILL.md 为准。

## Combined / 经中央大道连接

Install compatible Thoroughfare separately and read its actual public contract. The Host must complete formal wiring review and registration with the new package's actual SKILL.md digest and `OnHandDrudge/onhanddrudge/invoke.py` entry. Use ON_DEMAND / DIRECT_HOST_RETURN and current valid session attachment. Do not import production Binding files, assign an assumed B0003 epoch, or copy a carrier. Combined surface_construction is a sibling transport field with capsule_contract_version/run_access/work_units; session comes from verified Host governance. It is not added to the Drudge operation payload.

另装兼容中央大道，按其实际合同完成接线审查与注册，使用新包合同实际摘要和上述入口。不得复制生产 Binding、假定 epoch 或复用 carrier；surface_construction 与操作 envelope 分离，session 来自受验证接线。本包没有安装/更新正式注册的脚本，不自动改 B0003 或 Host 配置。

## Removal / 卸载

Stop active calls and confirm quiescence. Remove the Standalone discovery link through the Host's supported mechanism; for Combined, perform Thoroughfare's formal removal lifecycle first. Decide separately whether to archive or delete private Runs. Delete product files only after those decisions. No uninstall script is provided, and no user state is included in the distribution.

先停止调用并确认静止；Standalone 用 Host 正式机制移除发现入口，Combined 先合法解除注册/连接。私有 Run 的保留或删除单独决定，再处理产品文件。本包不含一键卸载脚本，也不含用户状态。
