# OnHandDrudge

[English](README.md) · 简体中文

**OnHandDrudge 1.0.0 首次官方双语发行（OnHand Core 新官方发行体系）。** [正式下载与发行说明](https://github.com/onhandcore/OnHandDrudge/releases/tag/first-public-bilingual-20261024)。这是当前重写版本首次在新官方体系正式双语公开发行；旧帐号 onhandcore-source 曾公开发行 v0.1.0，本次不代表产品历史上第一次公开发行。RC-R01 为经过独立审查的封存构件来源标识，不代表 GitHub 预发布版。原 ZIP 内候选文字属于封存准备阶段，详见[正式发行说明](RELEASE_NOTES.md)。

OnHandDrudge 是面向 Agent 复杂任务的**结构化执行治理 Skill**。Host 负责思考、规划和工具执行；Drudge 负责机械治理、执行冻结、Evidence、恢复及交棒。

## 为什么使用 Drudge

复杂任务可能在长对话、失败重试或换窗口后失去准确执行点。Drudge 将 Host 声明的任务范围、工作单元、当前 Step、Attempt 与完成证据绑定到可持久读取的身份和摘要。它检查当前可以执行什么、证据是否与冻结身份一致，以及局部完成是否足以进入根任务成功。

Drudge 不替代 Agent 的自主规划，也不判断自然语言语义是否正确。Host 仍需正确声明要求、解释证据并决定下一步；机械回执不能证明这些判断正确，未声明的遗漏不会自动被发现。

## 核心能力与职责

- **语义胶囊**：Memory 保留用户原文及来源；Global Hybrid 保存 Host 整理的全局语义；Decomposed Hybrid 表达真实子范围和父子关系。Work Unit 是执行投影，不是第四类胶囊。
- **JIT 与冻结**：先绑定声明的工作拓扑，再按当前 Work Unit 生成并冻结执行 JSON，只提供下一合法 Step；不预生成未来工作单元的执行产物。
- **Step 与 Evidence**：动作前登记 Attempt，工具由 Host 实际执行，动作后提交真实结果及证据。修订、摘要和封存关联执行身份；不能覆盖旧记录补造成功。
- **恢复与交棒**：用准确 ticket 恢复绑定和执行点；继续前须完成 preflight、Alignment 及副作用核验。非最终工作单元使用成对交棒和 continuation 身份，由真实独立 Host 任务承接。
- **根完成门禁**：WU PASS 不等于 Run SUCCESS。Host 需声明并核销实际必要要求，Drudge 检查相应机械资格。

Host 保留语义推理、范围拆分、工具选择、授权、执行位置、证据解释及后续任务创建。Drudge 不自带 Skill 筛选、安装、长期知识库、并行代理调度或后台守护服务。

## 安装与真实调用方式

本候选是单 ZIP，解压后只有一个 `OnHandDrudge/` 产品根目录，内含三个薄 Python 入口、九个字节码实现成员、正式合同及双语文档；不捆绑解释器或其他 Skill。首验环境为 **Windows x64 / CPython 3.12.14**，字节码 magic 为 `cb0d0d0a`，优化级别 0。

1. 核对 ZIP 外的 `SHA256SUMS.txt`，完整解压到用户选择的目录。产品 home 是解压出的 `OnHandDrudge/`，不能指向 ZIP、包内 `onhanddrudge/` 或某个旧 Run。
2. 让 Host 使用该 home 的 [SKILL.md](SKILL.md) 发现产品；Python 导入路径需包含该 home。Host 需能读取运行成员，并获准在 home 下的 `DRUDGE_PRIVATE_RUNS/` 创建、替换和读取私有状态；同目录锁/临时文件也须可用。不会自动修改 Host 配置。
3. **Standalone**：从 `onhanddrudge.invoke` 导入 `PublicHostSurface` 与 `invoke`，显式提供 home、准确 ticket 和适用的真实 Host 会话身份。它不依赖中央大道、BestFit 或 RecallLedger。`invoke.py` 是导入入口，不是任意 JSON 命令行执行器。
4. **Combined**：另装兼容 OnHandThoroughfare，按其正式接线审查及生命周期注册 Drudge，再通过其公开 transport 调用。所需入口仍为 `OnHandDrudge/onhanddrudge/invoke.py`，profile 为 ON_DEMAND / DIRECT_HOST_RETURN。不得复制生产 Binding 或其他会话 attachment 作为安装配置；发现成功不等于 Host 必然主动调用。

完整步骤、解释器检查及可运行的 Standalone 示例见 [INSTALL.md](INSTALL.md)。接口构造概要：

```python
from onhanddrudge.invoke import PublicHostSurface, invoke

surface = PublicHostSurface(
    discovered_product_home,
    task_run_id=new_exact_ticket,
    session_id=actual_host_session,
    capsule_contract_version="onhand.semantic_capsule.v2",
)
# 按 SKILL.md 提交胶囊、绑定工作单元，再投影、登记 Attempt、执行工具及提交结果。
```

操作 envelope 包含合同、operation/ticket 身份、预期修订、payload 和 `foundation.digest` 计算的 request_digest。Combined 的 surface_construction 属于 transport 独立字段，不混入操作 envelope。恢复使用 RESUME 和读取到的完整 work_units；跨已消费交棒还必须携带其 continuation 身份，不能重新 CREATE 或伪造新 session。

## 验证范围与限制

已有 Windows/Codex 实际独立 Host 的 Standalone、Combined、G2 跨会话和 ExecutionRecovery 验收；后续 P1/P2 维护有定向回归与实际生产 smoke。本候选复用未受影响的业务证据，仅补实际 ZIP 解压安装、字节码加载、最小 Host 动作/恢复，以及 P1/P2、Alignment 和 ExecutionRecovery 包装影响检查；没有重跑整套 V2 RC。

本轮 Combined 补充范围是未修改的现役 loader 对隔离候选的适配测试，Host/fabric 使用显式测试端口；不冒充新的生产会话 E2E，不修改正式 B0003。详见 [RELEASE_NOTES.md](RELEASE_NOTES.md)。

WorkBuddy 的 Drudge E2E、Linux/macOS、其他 Host、干净机器、其他 Python 版本和跨设备移动尚未验证。字节码不是 Python >=3.10 通用包；其他 3.12 补丁版也没有本候选实测。配套 Host 模型/工具可能有各自费用。

未知副作用、未知外部输出或未知任务创建结果必须停止核实，不能盲目重试。没有自动语义修复、自动跨 Host 接续或自动重连保证。旧 v1 Run 不原位转换为 v2。字节码不保证不可反编译。

卸载前停止活动调用并确认静止；Standalone 按 Host 自身方式解除薄入口，Combined 按中央大道正式移除生命周期解除连接。私有 Run 和 Evidence 的保留或删除由用户决定，不能当发行文件传播。没有一键卸载脚本。

## 许可与获取

[LICENSE](LICENSE) 为 OnHand 家族闭源发行许可：允许安装、运行和自由转发官方完整、未经修改的成品；不授予修改、改编或重新包装的权利，也不授予分发修改版、改编版或重新包装版的权利。第一方 Core 源码不因成品发行而公开。既有合法授权、第三方许可及依法不能排除的权利不受影响；中文正文适用，英文仅为便利翻译。[第三方说明](THIRD_PARTY_NOTICES.md)对应本包实际边界。