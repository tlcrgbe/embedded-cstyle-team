<div align="center">

# embedded-cstyle-team

### 让 AI 写出的 C 代码，更贴近嵌入式工程现场

固定硬件 · 直接实现 · 职责清晰 · 适度防御

[![版本](https://img.shields.io/badge/版本-V1.1-2563eb?style=flat-square)](https://github.com/tlcrgbe/embedded-cstyle-team/tree/v1.1.0)
[![语言](https://img.shields.io/badge/语言-嵌入式_C-0f766e?style=flat-square)](skills/embedded-cstyle-team/SKILL.md)
[![注释](https://img.shields.io/badge/注释-简体中文-ea580c?style=flat-square)](#4-你会看到怎样的代码)

[快速安装](#2-一分钟开始使用) · [使用示例](#3-把任务说清楚效果才稳定) · [完整规则](skills/embedded-cstyle-team/SKILL.md) · [迭代路线](ROADMAP.md) · [更新记录](CHANGELOG.md)

</div>

---

你可能见过这样的 AI 代码：一个固定设备也要上下文和函数指针，内部每层都在判空，几行业务被拆成一串包装函数，真正的处理顺序反而难以看清。

这个 skill 来自实际编码比较与持续修订，希望让 AI 先理解硬件、工况和调用关系，再写出便于团队维护的嵌入式 C。**读得懂主流程，找得到状态归属，改需求时能定位到负责它的代码。**

> 当前正式版本是 **V1.1**。方向已经明确，真实工程适用性仍需持续验证。后续按案例证据改规则，见 [迭代计划](ROADMAP.md)。

## 1. 它能帮你做什么

| 你的任务 | skill 关注的取舍 |
| --- | --- |
| 写串口协议接收与命令处理 | 组帧、完整性校验、命令语义和业务动作各有归属 |
| 写设备采集或周期任务 | 初始化、采样、重试、缓存和结果发布顺序清楚 |
| 修改已有模块 | 延续项目接口和风格，围绕本次需求做局部调整 |
| 评审 AI 生成的代码 | 检查抽象是否有用、内部检查是否重复、注释是否解释关键约定 |

适合 **固定硬件产品的 MCU 裸机与 RTOS 固件**。遇到实际多实例、共享数据或移植需求，仍按需要使用上下文、同步和适配接口。项目已有规范优先，C++ 编码需要另选适用规范。

## 2. 一分钟开始使用

### 推荐：直接让 Codex 安装

将下面这段话发给 Codex：

```text
使用 skill-installer，从 tlcrgbe/embedded-cstyle-team 仓库安装
skills/embedded-cstyle-team，固定版本 v1.1.0，安装到全局 skills。
已有同名版本时先检查并备份，再更新。
```

安装后，在后续对话中明确调用：

```text
使用 $embedded-cstyle-team，按当前工程约定完成这次嵌入式 C 修改。
```

若技能列表尚未刷新，重新打开 Codex。固定到发布标签可以避免 `main` 的后续修订改变当前规则。

<details>
<summary><b>手动安装：Windows / PowerShell</b></summary>

先下载固定版本：

```powershell
git clone https://github.com/tlcrgbe/embedded-cstyle-team.git
Set-Location embedded-cstyle-team
git switch --detach v1.1.0
```

只安装 skill 子目录：

```powershell
$skillHome = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $HOME '.codex\skills'
}
$targetSkill = Join-Path $skillHome 'embedded-cstyle-team'
if (Test-Path -LiteralPath $targetSkill) {
    throw '已安装，请先备份旧版本，再完整替换。'
}
New-Item -ItemType Directory -Path $skillHome -Force | Out-Null
Copy-Item -LiteralPath '.\skills\embedded-cstyle-team' `
    -Destination $targetSkill -Recurse
```

默认目标为用户目录下的 `.codex/skills/embedded-cstyle-team`；设置了 `CODEX_HOME` 时使用该目录。更新时先备份旧目录，再完整替换，避免残留旧规则。

</details>

## 3. 把任务说清楚，效果才稳定

skill 提供编码取舍，实际的协议、时序和硬件约定仍来自你的工程。开始前给出以下信息通常就够了：

| 信息 | 例子 |
| --- | --- |
| 改哪些代码 | 当前模块、调用方、可修改文件和已有接口 |
| 在什么环境运行 | MCU、编译器、C 标准、裸机或 RTOS |
| 谁在什么时候调用 | 单任务周期调用、ISR 收数据、DMA 写缓冲区 |
| 哪些行为必须保持 | 帧格式、超时、重试次数、失败后保留的结果 |

### 场景 A：新写串口协议接收

```text
使用 $embedded-cstyle-team 编写串口协议接收解析模块。

环境：C99，固定单串口，任务周期调用，接收缓冲区由现有接口提供。
协议：帧头、长度、cmd、data、CRC 的定义见附带协议说明。
要求：支持分批接收和连续多帧；校验完整帧后再处理 cmd。
职责：组帧与校验归接收模块，参数语义与业务动作有明确归属。
沿用现有缓冲区和设备接口，主要接口和关键状态用中文注释。
先核对协议缺失项，再实现；不要自行猜测 CRC 参数。
```

### 场景 B：修改已有设备任务

```text
使用 $embedded-cstyle-team 修改当前设备采集任务。

只调整通信失败后的重试和日志节流，保留现有采样周期与接口。
失败时继续保留最近一次完整结果，成功后恢复正常状态。
按现有任务与驱动分工处理，不新增通用调度框架。
说明修改位置、关键行为，以及实际完成的验证。
```

### 场景 C：评审一份 AI 代码

```text
使用 $embedded-cstyle-team 评审以下模块，先给建议，不改代码。

检查主流程、职责、状态归属、必要边界和注释。
指出哪些抽象或检查缺少实际用途，并给出具体源码依据。
对已有工程接口保持克制，给出能保持行为的最小修改方案。
```

## 4. 你会看到怎样的代码

| 关注点 | 希望形成的习惯 |
| --- | --- |
| 接口 | 标量按值传递；缓冲区与较大对象按实际需要用指针 |
| 组织 | 具名直接调用；按职责拆函数，独立维护或复用时再拆模块 |
| 状态 | 所有者和更新时机明确；单实例可用文件内 `static` |
| 故障 | 外部输入在入口校验；超时、通信失败和恢复由适当层处理 |
| 注释 | 说明用途、单位、时序与原因；局部行后注释适度对齐 |

没有既有项目约定时，新 C 代码默认使用 4 空格、120 字符行宽、PascalCase 类型、小写下划线函数与变量。例如：

```c
#define DEVICE_PAGE_COUNT         3U
#define DEVICE_PAGE_BYTES         4U
#define DEVICE_RETRY_INTERVAL_MS  10U

uint32_t sample_ms;   // 本次有效采样的时间戳，单位 ms
uint8_t retry_count;  // 当前事务的连续失败次数，成功后清零
```

**少防御，依靠的是明确契约与正确设计。** 长度、CRC、容量、硬件超时和实际动作条件仍要处理；内部同一条件无需每层重复检查。是否使用指针、回调或锁，取决于真实数据与调用关系。

## 5. 从 V1.0 往哪里走

| 阶段 | 重点 | 希望回答的问题 |
| --- | --- | --- |
| V1.1 · 固定评价依据 | 真实参考片段、独立盲评、未见过的验收任务 | 风格进步能否被作者之外的人确认？ |
| V1.2 · 验证工程边界 | 单任务、ISR/DMA、多实例、数值与时间约束 | 简洁实现是否仍能保留必需的正确性处理？ |
| V1.3 · 收敛与稳定 | 合并规则、按需参考、旧任务回归 | 能否减少指令负担，同时保持输出质量？ |

以上为 V1.0 时制定的验证计划，尚未完成。2026-10-15 发布的 V1.1 是依据实际开发反馈做的局部规则修订，不表示同名计划阶段已完成；详见 [ROADMAP.md](ROADMAP.md)。

<details>
<summary><b>当前验证做到哪一步？</b></summary>

V1.1 的 skill 格式校验已通过，尚未进行新的独立行为验证。以下为 V1.0 的历史验证：开发阶段，两份设备采集示例的调试与发布主机构建各通过 12 个场景；最后的对齐调整核对了代码 token 不变和行宽。

这些是有限示例的证据，还不足以证明所有任务都有效。最终排版由根代理整理，不是新一轮独立双 agent 验证。目标构建与板上验证需要按具体工程另行完成。

</details>

## 6. 规则、说明与历史各放哪里

```text
embedded-cstyle-team/
├── README.md                           从这里开始
├── ROADMAP.md                          迭代方向与验收方式
├── CHANGELOG.md                        已完成的版本记录
└── skills/embedded-cstyle-team/         实际安装的目录
    ├── SKILL.md                        当前有效规则
    └── references/
        └── open-source-style.md        按需读取的风格参考
```

完整规则在 [SKILL.md](skills/embedded-cstyle-team/SKILL.md)。参考固定版本的 Grbl 与 Betaflight 局部实现，取其职责与实现取舍，详见 [开源风格基准](skills/embedded-cstyle-team/references/open-source-style.md)。本仓库没有复制第三方项目源码。

### 常见问题

**会强制把所有函数改成没有指针吗？**

不会。传递缓冲区、处理较大对象和多实例上下文时，指针有实际用途。

**会为了“职责清晰”拆出很多文件吗？**

不会。文件内私有函数也能表达边界，是否拆模块由维护与复用需求决定。

**历史记录会进入每次编码的上下文吗？**

安装目录只包含有效规则和按需参考；入口不要求读取 README、路线图和更新记录。选中 skill 时读取的是当前 `SKILL.md`。

**如何反馈一个写法问题？**

在 [Issues](https://github.com/tlcrgbe/embedded-cstyle-team/issues) 提供最小可公开案例：任务约束、输出片段、希望的取舍和验证依据。避免只描述“AI 味重”，尽量指出重复检查、状态混杂、无用途包装或难懂注释所在的位置。

---

正式发布见 [Releases](https://github.com/tlcrgbe/embedded-cstyle-team/releases)，维护中的文档见 `main`。V0.1～V0.9 的本地迭代摘要保留在 [CHANGELOG.md](CHANGELOG.md)，完整 Git 版本管理从 V1.0 开始。
