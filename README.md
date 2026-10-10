# embedded-cstyle-team

面向固定硬件产品的嵌入式 C 编码 skill，用于 MCU 裸机和 RTOS 固件的编写、修改与评审。当前版本：**V1.0**。

## 1. 编码目标

- 高内聚、低耦合：数据与处理归属清楚，主流程直接，按职责组织函数和模块。
- 面向已知硬件与明确调用契约，不默认构造通用框架、ops 表或层层包装。
- 标量优先按值传递，缓冲区与较大对象按需要使用指针；保留真正必要的缓存、上下文与同步。
- 外部输入、通信失败和实际运行故障按边界处理，内部固定调用链避免重复判空、检查和断言。
- 中文注释说明接口契约、重要状态和关键处理原因；局部宏内容与简短行后注释适度对齐。

项目现有约定优先。没有约定的新 C 代码默认 4 空格缩进、80 字符行宽、PascalCase 类型、小写下划线函数与变量。此 skill 不用于 C++，不替代算法设计、目标构建或硬件验证。

## 2. 仓库结构

```text
README.md                         使用与维护说明
CHANGELOG.md                      版本更新记录，不随安装加载
skills/embedded-cstyle-team/
    SKILL.md                      当前有效规则
    references/open-source-style.md  按需读取的开源风格基准
```

安装时只复制 `skills/embedded-cstyle-team/`。说明和历史位于包外，入口不要求读取更新记录。

## 3. 安装

先克隆本仓库并切换到固定发布版本：

```powershell
git clone https://github.com/tlcrgbe/embedded-cstyle-team.git
Set-Location embedded-cstyle-team
git switch --detach v1.0.0
```

然后在 PowerShell 执行：

```powershell
$skillHome = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $HOME '.codex\skills'
}
$targetSkill = Join-Path $skillHome 'embedded-cstyle-team'
if (Test-Path -LiteralPath $targetSkill) {
    throw '已安装，请先备份旧版本，再按更新说明处理。'
}
New-Item -ItemType Directory -Path $skillHome -Force | Out-Null
Copy-Item -LiteralPath '.\skills\embedded-cstyle-team' `
    -Destination $targetSkill -Recurse
```

安装后在后续对话中调用；若技能列表未刷新，重新打开 Codex。已有同名安装时不叠加第二份，先备份原目录再完整替换，避免残留旧参考文件。

也可以直接对 Codex 说：

```text
使用 skill-installer，从 tlcrgbe/embedded-cstyle-team 仓库安装
skills/embedded-cstyle-team，固定版本 v1.0.0。
```

## 4. 使用

明确调用示例：

```text
使用 $embedded-cstyle-team 编写固定串口协议的接收解析代码。
从缓冲区读取，检查帧头、长度和 CRC；命令语义及业务动作分清职责。
沿用现有工程接口，不改变协议和任务调度行为。
```

```text
使用 $embedded-cstyle-team 评审当前设备采集模块。
重点检查职责、状态归属、故障处理和注释；先给问题和最小修改建议。
```

技能允许匹配任务时自动选择；需要确保应用时建议明确写出名称。提供当前代码、协议、工具链和调用契约，避免让模型猜测工况。

减少防御性代码并不表示取消长度、CRC、容量、运行状态或硬件超时检查。选择依据是当前真实边界与故障，不是机械统计指针、断言或函数数量。

## 5. 版本与维护

`main` 保存当前维护版本，正式版本使用 Git 标签，例如 `v1.0.0`。入口中的 `V1.0` 对应标签 `v1.0.0`。安装目录是部署副本，规则的维护以本仓库为准。

更新流程：在仓库中修改有效规则，针对真实任务核对输出，将变更和验证边界记入 [CHANGELOG.md](CHANGELOG.md)，再提交并发布新标签。按需采用有、无 skill 的同任务比较；比较前固定功能契约与风格评价问题，避免为了某个输出临时改标准。

V0.1 至 V0.9 是建立 V1.0 前的本地迭代记录，本仓库从 V1.0 开始保存完整 Git 版本；不将早期记录伪造为历史提交。

V1.0 的规则经过格式校验。开发阶段两份设备采集示例的调试与发布主机构建各通过 12 个场景；最终对齐调整核对了代码 token 不变和行宽。上述证据不代表所有使用场景都已验证，也不代表板上验收。

## 6. 参考与上下文

参考固定版本的 Grbl 和 Betaflight 局部源码取舍，详见 [开源风格基准](skills/embedded-cstyle-team/references/open-source-style.md)。本仓库未复制第三方项目源码。

选中技能后会读取完整 `SKILL.md`；参考文件只在相关问题需要时读取。因此入口只保留当前有效规则，更新记录和使用说明不放入安装目录。
