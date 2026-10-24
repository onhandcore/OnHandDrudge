# OnHandDrudge 1.0.0 — First Official Bilingual Release (OnHand Core)

OnHandDrudge 首次官方双语发行（OnHand Core 新官方发行体系）。
This is the first official bilingual release of the current rewritten OnHandDrudge under the new OnHand Core official distribution system, and the third published Skill in that system.

The old onhandcore-source account previously published OnHandDrudge v0.1.0. This is not the first public release in the product's entire history. That deprecated repository and its history remain unchanged.
旧帐号 onhandcore-source 曾公开发行旧版 v0.1.0；本次不代表产品历史上第一次公开发行。旧废弃仓库及其历史保持不变。

## Responsibilities / 职责

The Host understands requirements, plans, selects tools, and performs actual tool execution. Drudge provides structured execution governance, frozen Work Units and execution identities, Step gates, Evidence, recovery, and handoff. Drudge is not an independent Agent that understands, plans, and executes all work itself.
Host 负责理解需求、规划、工具选择与实际工具执行；Drudge 负责结构化执行治理、工作单元冻结、Step 门禁、Evidence、恢复及交棒，不是自行理解、规划并执行全部工作的独立 Agent。

## Official asset / 正式附件

- Product version / 正式产品版本: **1.0.0**
- Asset: `OnHandDrudge-1.0.0-BILINGUAL-RC-20261024-R01-Windows-CPython312.zip`
- SHA256: `f057a5f33b1722ff8900bb80e652013a358ff013c807f36891f74c8764b88eb5`
- Initial validated scope / 首发验证范围: **Windows x64 / CPython 3.12.14**, cpython-312, magic `cb0d0d0a`, optimize=0.

RC-R01 / BILINGUAL_RC_20261024_R01 identifies the independently reviewed sealed build provenance. This GitHub Release is an official, non-draft, non-prerelease release. The original ZIP is unchanged. Candidate/not-public/no-publication-authorization text inside the sealed ZIP describes its preparation stage and is superseded for publication status by this authorized official Release; it does not describe the current GitHub Release status.
RC-R01 是经过独立审查的封存构件来源标识，不代表本次 GitHub Release 是预发布版。ZIP 原字节不变；包内候选、未公开或尚无发布授权等文字属于封存准备阶段，其公开状态以本次已获授权的正式 Release 为准。

The preserved SKILL.md paragraphs mentioning epoch 4 / PENDING_SEPARATE_DEPLOYMENT describe historical wiring-review stages. Later existing deployment evidence covered epoch 7 / binding 7. This release does not install the new ZIP into production, modify B0003, or transfer any production binding/session state. A new Combined installation requires its own compatible transport and formal registration.
SKILL.md 中 epoch 4 / PENDING_SEPARATE_DEPLOYMENT 是历史接线审查阶段；后续既有部署证据覆盖 7/7。本次公开发行不代表新 ZIP 已完成生产切换，不修改 B0003，也不转移生产绑定或会话状态。新的 Combined 安装仍须独立完成兼容接入和正式注册。

## Validation and limitations / 验证与限制

Independent review verified ZIP integrity, license, bilingual README consistency, actual Standalone execution/recovery, 49 targeted checks, and disclosure boundaries. Unaffected historical acceptance evidence is reused; the full V2 RC suite was not rerun. The additional Combined probe used the unchanged deployed loader with isolated candidate files and explicit fixture Host/fabric ports: adapter compatibility only, not new production-Host E2E.
独立审查覆盖 ZIP 完整性、闭源许可、中英文一致性、Standalone 实际执行与恢复、49 项定向检查及泄露边界。复用未受影响的历史验收，未重跑整套 V2 RC；补充 Combined 验证使用隔离候选及显式测试 Host/fabric，仅证明适配兼容，不冒充新的生产 Host E2E。

Other Python versions (including other 3.12 patch versions), Linux/macOS, other Hosts, WorkBuddy Drudge E2E, clean machines and cross-device use remain unverified. Python and other Skills are not bundled. Bytecode is not a guarantee against decompilation. Host models and tools may have their own costs.
其他 Python 版本（含其他 3.12 补丁版）、Linux/macOS、其他 Host、WorkBuddy Drudge E2E、干净机器及跨设备使用尚未验证。不捆绑 Python 或其他 Skill；字节码不保证不可逆向，Host 模型与工具可能有各自费用。

The OnHand Family Closed-Source Distribution License applies; Chinese governs and English is a convenience translation. See LICENSE and THIRD_PARTY_NOTICES.md. First-party Core source is not published. Existing lawful grants and independent third-party licenses remain unaffected.
适用 OnHand 家族闭源发行许可，中文为准、英文为便利翻译，详见 LICENSE 与 THIRD_PARTY_NOTICES.md。不公开第一方 Core 源码；既有合法授权及独立第三方许可不受影响。
