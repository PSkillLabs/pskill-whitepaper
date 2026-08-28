# PSkill 文档白皮书

**又称：可执行 Skill 文档规范（Executable Skill Document Specification）**

**白皮书版本：0.3.0-draft**

---

## 1. 概述

PSkill（Portable / Procedural Skill，暂定名称）是一种**面向 AI 执行环境的增强型可执行数据包规范**。

PSkill 与传统 Skill 文档的核心区别在于：

> **传统 Skill 主要用于向 AI 提供知识、指导和建议；PSkill 则用于向 AI 定义一个必须严格执行的、具有明确输入、输出、步骤、工具和失败条件的过程。**

因此，PSkill 不应仅被理解为“提示词集合”“知识库”或“说明文档”。

一个 PSkill 应被视作：

> **由自然语言描述程序逻辑，并携带其运行所需资源和工具的可执行软件包。**

PSkill 的主要逻辑通常存放于：

```text
main.pskill.html
```

该文件虽然使用 HTML 作为文档容器，但其本质并不是网页，而是一个**由 AI 解释执行的自然语言程序**。

PSkill 可以调用随包附带的：

* 二进制可执行程序；
* Python 脚本；
* Shell / PowerShell 脚本；
* Java / .NET 等运行时程序；
* WASM 模块；
* ONNX / GGUF / Safetensors 等模型；
* 字典、规则库、模板；
* 图片；
* 配置文件；
* 数据文件；
* 其他离线工具或资源。

因此，一个完整 PSkill 可以理解为：

```text
自然语言程序
+
确定版本的工具
+
确定版本的数据
+
确定版本的子 PSkill
+
确定的输入输出契约
+
确定的失败语义
```

---

# 2. 设计目标

PSkill 的设计目标包括以下几个方面。

## 2.1 可执行性

PSkill 必须明确告诉 AI：

* 接受什么输入；
* 拒绝什么输入；
* 如何验证输入；
* 按什么顺序执行；
* 调用什么工具；
* 每一步如何判断成功；
* 什么情况下失败；
* 分支流程
* 对预见异常的处理分支；
* 人工参与要求（如适用）；
* 最终输出什么。

一个合格的 PSkill 不应依赖 AI 自己重新设计业务流程。

---

## 2.2 可复现性

在相同：

* PSkill 版本；
* 输入；
* 执行环境；
* 工具版本；
* 模型资源；
* 配置；

条件下，PSkill 应尽可能产生一致或语义等价的结果。

PSkill 应尽量减少：

* 网络服务变化；
* 第三方 API 变化；
* 软件自动升级；
* 模型版本变化；
* 操作系统环境变化；

对执行结果造成的影响。

---

## 2.3 完整性

PSkill 的作者应尽可能将完成任务所需的全部资源包含在 PSkill 数据包内。

推荐遵循：

> **完全本地原则（Local-First Principle）**

以及：

> **完全版本原则（Fully Versioned Principle）**

即：

**一个 PSkill 包应尽可能能够在没有互联网的环境中完整执行，并且所有参与执行的工具和关键资源都具有明确版本。**

---

## 2.4 Fail-Closed 原则

PSkill 默认采用：

> **失败关闭（Fail Closed）**

而不是：

> **尽力完成（Best Effort）**

执行策略。

如果执行过程中出现无法按照 PSkill 规定验证的异常，则整个 PSkill 执行应当失败。

AI 不应自行修改业务逻辑以获得一个“看起来合理”的结果。

---

# 3. 规范术语

本白皮书使用以下术语表达规范等级。

### MUST / 必须

实现必须遵守。

违反该要求的 PSkill 应被认为不符合本规范。

### MUST NOT / 禁止

实现不得执行该行为。

### SHOULD / 应当

强烈推荐遵守。

仅在存在充分理由时允许偏离。

### SHOULD NOT / 不应

原则上不建议执行。

### MAY / 可以

属于可选行为。

---

### IF / 如果

用于声明一个可验证条件及其对应的唯一分支。条件必须可判断，且应同时定义条件成立和不成立时的去向；未覆盖的状态必须终止为失败。

### TRY / 尝试

用于声明可能失败的受控操作。`TRY` 必须指定成功判定、失败判定，以及成功与失败各自进入的后续步骤；不得把“尝试”理解为允许 AI 自行寻找替代方法。

### IF SUCCESS / 如果成功

仅当 `TRY` 或步骤满足其事先声明的成功条件时进入该分支。

### IF FAILURE / 如果失败

仅当 `TRY` 或步骤满足其事先声明的失败条件时进入该分支。若未规定可恢复路径，必须以明确错误码终止 PSkill。

### HUMAN STEP / 人工参与步骤

指必须由已声明的人类完成、且 AI 不得替代或伪造的步骤，例如视觉连续性判断、机器人实机验证或真人身份认证。

---

# 4. PSkill 数据包

## 4.1 文件格式

正式发布的 PSkill 数据包应使用：

```text
7z
```

作为标准压缩格式。

例如：

```text
ImageOCR.pskill.7z
```

PSkill 允许使用 7z 标准分卷压缩。

例如：

```text
ImageOCR.pskill.7z.001
ImageOCR.pskill.7z.002
ImageOCR.pskill.7z.003
```

AI 在执行 PSkill 前，必须确认所有必要分卷均已存在。

缺少任何必要分卷，应直接判定：

```text
PSkill Load Failed
```

不得尝试部分执行。

对于处于高速开发阶段的工程，PSkill 数据包允许以未压缩的根目录形式存在、分发和执行。该目录必须保留 PSkill 数据包的目录结构，并在根目录包含 `main.pskill.html`。此形式用于让 PSkill 的各个文件能与工程一同进行版本管理；Runtime 必须将其作为同一数据包内容进行验证，不得要求先生成 7z 压缩包或执行解压步骤。正式发布时，仍可由该目录构建为 7z 数据包。

---

# 5. 推荐目录结构

一个典型 PSkill 包可以采用：

```text
MyPSkill/
│
├── ReadMe.md
├── main.pskill.html
│
├── tools/
│   ├── ocr/
│   │   ├── windows-x64/
│   │   ├── linux-x64/
│   │   └── version.txt
│   │
│   └── image-converter/
│
├── models/
│   └── model-v1.2.0.onnx
│
├── scripts/
│   ├── preprocess.py
│   └── postprocess.py
│
├── resources/
│   ├── dictionary.json
│   └── template.png
│
├── pskills/
│   └── ImageOCR-2.1.4.pskill.7z
│
├── examples/
│
└── LICENSE
```

除：

```text
ReadMe.md
main.pskill.html
```

外，其余目录均属于推荐结构，而非强制结构。

---

# 6. `ReadMe.md`

每个正式发布的 PSkill 数据包应包含：

```text
ReadMe.md
```

ReadMe 面向：

* 人类开发者；
* PSkill 分发平台；

ReadMe 应简要说明：

* PSkill 名称；
* PSkill 用途；
* 当前版本；
* 支持的平台；
* 输入类型；
* 输出类型；
* 主要依赖；
* 文件大小；
* 许可协议；
* 作者；
* 项目地址；
* 已知限制。

ReadMe 不承担执行逻辑。

正式执行逻辑必须以：

```text
main.pskill.html
```

为准。

当：

```text
ReadMe.md
```

与：

```text
main.pskill.html
```

发生冲突时，以：

```text
main.pskill.html
```

为执行依据。

---

# 7. `main.pskill.html`

## 7.1 文件性质

`main.pskill.html` 是 PSkill 的**主程序入口**。

虽然该文件使用 HTML 格式保存，但：

> **HTML 仅作为结构化自然语言程序的载体。**

它不应被设计成传统网页。

不建议：

* JavaScript UI；
* 动态网页框架；
* CSS 动画；
* Web Component；
* SPA；
* 复杂响应式布局；
* 为视觉美观设计的大量 DOM；
* 交互式按钮；
* 表单。

推荐主要使用：

```html
<h1>
<h2>
<h3>
<p>
<ul>
<ol>
<li>
<table>
<pre>
<code>
<img>
<a>
```

等简单结构。

PSkill 的 HTML 结构应优先考虑：

> **机器理解性**

而不是：

> **人类视觉美观性。**

---

# 8. PSkill 文件头

`main.pskill.html` 顶部必须首先出现 PSkill 名称。

例如：

```html
<h1>Image Text Extractor</h1>
```

随后应提供徽章信息。

例如：

```text
PSkill
Version 1.4.2
Spec 0.3
MIT
Offline
Windows x64 / Linux x64
```

徽章可以使用：

* 简单文本；
* SVG；
* 本地图片。

---

# 9. 元信息

文件头之后必须声明至少以下信息：

## PSkill 名称

例如：

```text
Image Text Extractor
```

---

## PSkill 版本

必须使用 Semantic Versioning。

例如：

```text
1.3.2
```

推荐遵守：

```text
MAJOR.MINOR.PATCH
```

---

## 白皮书版本

必须声明该 PSkill 设计时遵循的 PSkill 白皮书版本。

例如：

```text
PSkill Specification: 0.3.0
```

或者提供对应规范链接。

---

## 授权协议

应声明 PSkill 的许可协议

例如：

```text
MIT License
Apache-2.0
GPL-3.0
BSD-3-Clause
```

如果使用非标准授权协议，则应：

* 在数据包内提供完整协议；
* 或提供明确协议链接。

---

# 10. 输入契约

PSkill 正文的第一个主要部分必须是：

> **Input / 输入定义**

PSkill 必须清晰定义它能够处理什么。

输入定义必须具有足够的信息，使 AI 可以在执行主体逻辑之前完成：

> **Input Validation**

即输入验证。

---

# 11. 输入是一种广义概念

PSkill 中的“输入”不仅指用户提供的文件。

以下内容均属于输入：

* 用户文件；
* 文本；
* 图片；
* 视频；
* 音频；
* 文件夹；
* URL；
* 参数；
* 配置；
* 操作系统；
* CPU 架构；
* GPU；
* 显存；
* 可用内存；
* Python Runtime；
* .NET Runtime；
* 文件系统权限；
* 环境变量；
* 已安装软件；
* 网络状态；
* 当前工作目录；
* AI 是否具有某项工具调用能力。

因此：

> **执行环境本身属于 PSkill 输入的一部分。**

---

# 12. 输入验证器

每个 PSkill 必须定义逻辑上的：

> **Input Validator**

输入验证器至少应检查：

1. 输入资源数量；
2. 输入资源类型；
3. 输入格式；
4. 文件扩展名；
5. 文件内容；
6. 文件大小；
7. 数据完整性；
8. 必需执行环境；
9. 必需工具；
10. 必需运行时；
11. 必需硬件条件。

必要时还应验证：

* 图片宽高；
* 图片色彩空间；
* 视频编码；
* 音频采样率；
* JSON Schema；
* 文档页数；
* 模型格式；
* 文件 Magic Number；
* 文件是否损坏。

---

# 13. 输入验证失败

当输入不满足 PSkill 输入契约时：

> **PSkill 必须立即失败。**

例如 PSkill 只支持：

```text
PNG / JPEG
```

用户却提供：

```text
PSD
```

即使 AI 本身知道如何读取 PSD，也不得自行：

```text
PSD → PNG
```

然后继续执行。

除非 PSkill 明确规定：

```text
PSD 输入允许通过工具 X 转换为 PNG。
```

否则必须返回：

```text
Unsupported Input
```

---

# 14. 输出契约

输入定义之后，PSkill 必须定义：

> **Output / 输出定义**

输出定义用于告诉 AI：

* PSkill 最终会生成什么；
* 输出是什么格式；
* 输出用于什么；
* 输出是否可以直接交付用户；
* 输出是否供后续 PSkill 使用。

例如：

```text
Output Type:
UTF-8 JSON

Purpose:
Machine-readable OCR result.

Files:
result.json
preview.png
```

---

# 15. 输出必须可验证

PSkill 应尽可能定义输出验证方法。

例如：

```text
result.json MUST:

1. be valid UTF-8;
2. be valid JSON;
3. contain "blocks";
4. every block MUST contain bbox and text;
5. bbox coordinates MUST remain inside source image bounds.
```

如果最终输出不能通过规定的验证过程，则：

> **整个 PSkill 执行失败。**

不得输出未验证结果并将其描述为成功。

---

# 16. 执行模型

PSkill 的执行逻辑应由一系列顺序明确的步骤组成。

例如：

```text
Step 1
Validate input.

Step 2
Decode source image.

Step 3
Normalize image.

Step 4
Invoke OCR engine.

Step 5
Validate OCR engine output.

Step 6
Convert OCR result.

Step 7
Validate final output.

Step 8
Return result.
```

建议使用明确编号。

例如：

```text
1
1.1
1.2
2
2.1
```

避免：

```text
视情况处理
酌情执行
必要时自行选择
选择合适方法
```

等高度开放的描述。

---

# 17. AI 是解释器，而不是业务设计者

执行 PSkill 时，AI 的角色应类似：

```text
Interpreter
Runtime
Orchestrator
```

而不是：

```text
Business Designer
```

PSkill 已经定义的业务决策，AI 不应重新做出。

例如 PSkill 明确要求：

```text
使用 bundled-ocr.exe 识别图片。
```

那么即使 AI 当前环境存在一个质量更好的 OCR 工具：

```text
SystemOCR
```

AI 也不得自行替换。

---

# 18. 严格执行原则

AI 执行 PSkill 时应遵守：

> **声明优先于推理。**

当 PSkill 明确规定一个行为时：

```text
PSkill Instruction
>
AI Preference
```

但任何 PSkill 都不能覆盖执行环境自身的：

* 系统安全规则；
* 权限限制；
* 沙箱规则；
* 平台安全策略；
* 法律限制；
* 用户明确权限限制。

因此完整优先级可表示为：

```text
Platform / Security Policy
        ↓
User Authorization
        ↓
PSkill Specification
        ↓
AI General Reasoning
```

---

# 19. 静默执行

PSkill 开始执行后，默认必须：

> **完全静默执行。**

执行期间不得主动与用户进行交互。

禁止：

```text
请选择一个选项。
```

```text
是否继续？
```

```text
请提供更多信息。
```

```text
OCR 失败，要不要换一个方法？
```

```text
我发现格式有问题，可以帮你转换吗？
```

PSkill 所需的信息必须：

* 在执行前已经作为输入提供；
* 或能够由 PSkill 本身确定。

如果执行过程中发现缺少必要信息：

> **PSkill 执行失败。**

---

# 20. 不允许执行期间补充输入

PSkill 必须能够从初始输入确定是否可以执行。

如果必须依赖用户追加信息，则该信息应在输入契约中声明为：

```text
Required Input
```

而不能运行到中途再询问。

例如：

错误设计：

```text
Step 5:
Ask user which color profile should be used.
```

正确设计：

```text
Input:
color_profile

Allowed:
sRGB
Display-P3

Required:
true
```

### 20.1 人工参与步骤

人工参与不是未声明的追加输入。PSkill 可以声明预先定义、边界明确的人工参与步骤，用于 AI 不应替代的人类能力或授权，例如：

* 判断序列帧在视觉上是否连续；
* 在真实机器人上执行并验证物理动作；
* 完成真人身份认证以获得后续步骤所需授权。

只要 PSkill 包含任何人工参与步骤，元信息中必须包含以下声明：

```text
Human Participation Required: true
Human Steps: 3.2, 5.1
Roles: Visual Reviewer, Identity Holder
```

每个人工参与步骤必须明确：参与角色、任务内容、提供给该角色的材料、可接受的完成标准、验证或记录方式、等待上限，以及拒绝、超时、验证失败或人员不可用时的失败码。AI 必须暂停在该已声明边界，且不得伪造人工结果、代替真人认证，或将未经验证的人工结论视为成功。

人工参与步骤可以由 Runtime 以预先声明的方式通知并收集结果；这属于流程定义的一部分，而非向用户临时索取业务决策。若该步骤不能按声明完成，PSkill 必须明确失败。

---

# 21. Fail-Closed 执行模型

这是 PSkill 的核心原则之一。

假设 PSkill 定义：

```text
调用 OCR 工具获取图像文字。
```

实际执行：

```text
OCR tool exited with code 1.
```

AI 不得：

* 自己看图片猜文字；
* 换一个 OCR 工具；
* 使用在线 OCR；
* 调用其他视觉模型；
* 忽略失败；
* 输出部分结果；
* 修改原始图片再试；

除非 PSkill **明确规定了对应备用流程**。

默认行为必须是：

```text
PSkill Failed
```

---

# 22. 禁止隐式补偿

PSkill 不允许 AI 进行未声明的：

> **Implicit Compensation**

包括但不限于：

* 自动修复；
* 自动格式转换；
* 自动重新编码；
* 自动安装依赖；
* 自动更新软件；
* 自动下载模型；
* 自动调用互联网服务；
* 自动更换工具；
* 自动降低质量要求；
* 自动猜测缺失参数；
* 自动修改输入。

除非对应行为已写入 PSkill。

---

# 23. 显式备用路径

PSkill 可以定义备用流程。

例如：

```text
Step 4:

Invoke OCR Engine A.

If and only if:

ExitCode == 12

then invoke OCR Engine B.

For any other error:

terminate PSkill.
```

这是合法行为。

因为备用逻辑已经由 PSkill 作者提前定义。

因此：

> PSkill 并不禁止容错。

PSkill 禁止的是：

> **由执行 AI 临时发明容错逻辑。**

PSkill 设计者应在编写执行逻辑时预见合理可预见的问题，包括工具错误码、超时、资源不足、依赖不可用、网络中断、数据不完整和人工步骤未完成，并为每一种可恢复情形定义明确分支。一个完整分支应表达为：

```text
TRY: invoke OCR Engine A.

IF SUCCESS: validate and continue to Step 5.

IF FAILURE AND ExitCode == 12: invoke OCR Engine B.

IF FAILURE for any other reason: terminate with TOOL_EXECUTION_ERROR.
```

对于无法预见或未定义的状态，PSkill 不得停留在不确定状态，也不得隐式继续；必须以适当的明确错误码进入 `FAILED`。设计者可以定义恢复分支，但恢复分支本身同样必须有成功、失败和终止语义。

---

# 24. 错误必须具有明确边界

建议 PSkill 将错误划分为：

```text
LOAD_ERROR
INPUT_ERROR
ENVIRONMENT_ERROR
DEPENDENCY_ERROR
EXECUTION_ERROR
VALIDATION_ERROR
OUTPUT_ERROR
SECURITY_ERROR
```

---

# 25. PSkill 执行状态

推荐定义以下标准状态：

```text
READY
RUNNING
SUCCEEDED
FAILED
```

其中：

### READY

输入验证完成，可以执行。

### RUNNING

正在执行。

### SUCCEEDED

所有必要步骤和输出验证均成功。

### FAILED

任意不可恢复步骤失败。

---

# 26. 原子成功原则

除非 PSkill 明确支持 Partial Result，否则：

> **PSkill 应采用原子成功语义。**

即：

```text
成功
```

或者：

```text
失败
```

不存在：

```text
大概成功
基本成功
部分成功但可以用
应该没问题
```

等非确定性状态。

---

# 27. 失败报告

虽然执行过程必须保持静默，但执行失败后允许向调用方返回失败报告。

建议至少包括：

```text
PSkill
PSkillVersion
Status
FailedStep
ErrorCode
Reason
```

例如：

```text
PSkill: ImageTextExtractor
Version: 1.4.2
Status: FAILED
FailedStep: 4.2
ErrorCode: TOOL_EXECUTION_ERROR
Reason: OCR process returned exit code 12.
```

不得用未验证的推测代替真实错误原因。

---

# 28. 本地优先原则

PSkill 强烈建议遵循：

> **Local-First**

完成核心业务所需要的资源，应尽可能包含在 PSkill 包中。

例如：

不推荐：

```text
Step 3:
Download latest OCR model from example.com.
```

推荐：

```text
models/
└── ocr-model-2.4.1.onnx
```

---

# 29. 容器化优先原则

PSkill 设计者应优先使用 Docker 或兼容 OCI 标准的容器化技术封装执行环境，而不是依赖用户机器上预先安装的运行时、库、命令行工具或系统配置。

容器化的目标是将以下内容与用户本地环境隔离并固定：

* 操作系统基础镜像；
* 运行时及其版本；
* 系统库与应用依赖；
* 关键工具和模型；
* 默认工作目录与启动命令；
* 网络、文件系统与设备访问边界。

正式 PSkill 若使用容器，必须在数据包内提供 Dockerfile、镜像归档或可验证的镜像引用，并声明：

```text
Container Runtime: Docker Engine 27.0.0
Image: registry.example.com/pskill/image-ocr@sha256:...
Entrypoint: /app/run-ocr
Network: disabled
Mounted Inputs: read-only
Mounted Outputs: write-only
```

镜像必须以不可变摘要（digest）或等价的内容哈希锁定版本；不得使用 `latest`、浮动标签或会自动更新的基础镜像。若容器运行时、镜像、挂载、设备权限或网络策略不满足声明，PSkill 必须以 `ENVIRONMENT_ERROR` 或 `DEPENDENCY_ERROR` 失败。

只有在容器化会不合理地妨碍任务、运行环境不支持容器，或 PSkill 已明确声明不可容器化的必要原因时，才可以依赖本地环境；此时必须完整列出、锁定并验证所有本地依赖。

---

# 30. 网络依赖

PSkill 并非绝对禁止网络访问。

但是任何网络依赖都必须明确声明。

例如：

```text
Network Requirement:

Required: true

Endpoint:
https://api.example.com/v1

Purpose:
Obtain current exchange rate.

Failure policy:
If endpoint is unavailable, terminate PSkill.
```

不得存在：

> **未声明网络访问。**

---

# 31. 禁止依赖“最新版”

正式 PSkill 不应使用：

```text
latest
current
newest
stable
auto-update
```

等不能唯一确定版本的依赖表达方式。

例如：

不推荐：

```text
Python >= 3
```

推荐：

```text
Python 3.12.x
```

更严格的 PSkill 可以指定：

```text
Python 3.12.8
```

---

# 32. 工具版本

PSkill 使用的每一个关键工具都应具有明确版本。

例如：

```text
Tesseract OCR 5.5.0
FFmpeg 8.0
Python 3.12.8
ONNX Runtime 1.21.0
```

如果工具由 PSkill 自身维护，应同样提供版本。

例如：

```text
image-normalizer.exe
Version: 1.2.7
```

---

# 33. 工具完整性

推荐为工具提供：

```text
SHA-256
```

例如：

```text
ffmpeg.exe

Version:
8.0

SHA256:
C11E...
```

AI 执行 PSkill 前可以据此验证工具是否被修改。

---

# 34. PSkill 包自身完整性

PSkill 发布者可以为整个包提供：

```text
SHA-256
```

或者签名文件。

例如：

```text
MySkill.pskill.7z
MySkill.pskill.7z.sha256
```

未来版本的 PSkill 规范可以进一步定义：

```text
PSkill Signature
```

标准。

---

# 35. 平台兼容性

PSkill 必须声明支持的平台。

例如：

```text
Windows 11 x64
Ubuntu 24.04 x64
macOS 15 arm64
```

或者：

```text
Any environment with Python 3.12.8
```

如果当前环境不满足要求：

```text
ENVIRONMENT_ERROR
```

不得自行尝试构建兼容层，除非 PSkill 有明确规定。

---

# 36. 多平台工具

推荐将平台工具进行明确隔离。

例如：

```text
tools/
├── windows-x64/
│   └── worker.exe
├── linux-x64/
│   └── worker
└── macos-arm64/
    └── worker
```

PSkill 应明确规定平台选择逻辑。

---

# 37. 路径规范

PSkill 内部资源推荐使用：

> **相对于 PSkill 根目录的相对路径。**

例如：

```text
./tools/ocr/ocr.exe
./models/model.onnx
```

不推荐：

```text
C:\Users\Developer\Desktop\Skill\ocr.exe
```

等机器相关绝对路径。

---

# 38. 工作目录

PSkill 应区分：

```text
Package Directory
Input Directory
Working Directory
Output Directory
Temporary Directory
```

避免工具在未知目录产生副作用。

---

# 39. 修改输入

默认情况下：

> **PSkill 不应修改原始输入资源。**

推荐：

```text
Input → Working Copy → Processing → Output
```

如果 PSkill 会直接修改输入文件，则必须在输入定义中显式声明。

---

# 40. 临时文件

PSkill 应规定临时文件的：

* 保存位置；
* 生命周期；
* 清理方式。

成功结束后，应删除不需要的临时资源。

失败时是否保留调试数据，应由 PSkill 明确声明。

---

# 41. 外部副作用

任何涉及：

* 删除文件；
* 修改文件；
* 修改数据库；
* 发送邮件；
* Git Commit；
* Git Push；
* 部署；
* 上传；
* 发布；
* 发送消息；
* 修改系统配置；

等行为，都属于：

> **Side Effect**

必须在 PSkill 的输入/能力声明区域提前说明。

---

# 42. 权限最小化

PSkill 应遵守：

> **最小权限原则。**

例如，一个只进行图片 OCR 的 PSkill 不应要求：

```text
Administrator
```

权限。

一个只读取文件的 PSkill 不应要求整个文件系统写权限。

---

# 43. AI 能力不能被视作隐式依赖

PSkill 不应假定 AI 一定具有某项外部能力。

例如：

```text
Use Photoshop.
Use Blender.
Execute Python.
Access Internet.
Use GPU.
```

均应当作为环境要求进行声明。

---

# 44. 内置 AI 推理

PSkill 可以使用 AI 自身的自然语言、视觉或推理能力。

例如：

```text
Read the generated OCR text.

Classify each block into:
Title
Body
Caption
Other
```

但是应尽可能明确：

* 输入是什么；
* 分类标准是什么；
* 允许哪些输出；
* 如何验证输出。

---

# 45. 非确定性步骤

如果某一步本质上具有非确定性，例如：

* 文本生成；
* 图像理解；
* 分类；
* 摘要；
* LLM 推理；

则 PSkill 应尽量通过：

* Schema；
* 枚举；
* 示例；
* 判断标准；
* 验证规则；

缩小输出空间。

---

# 46. 自然语言程序设计原则

`.pskill.html` 虽然使用自然语言，但应具有传统程序设计中的结构。

推荐具有：

```text
Inputs
Preconditions
Environment
Outputs

Variables

Step 1
Step 2
Step 3

Conditions

Validation

Failure Conditions
```

### 46.1 AI 辅助编写原则

PSkill 设计者应优先使用 AI 辅助编写、审阅和维护 PSkill，而不是完全依赖手工编写。AI 辅助编写可以用于：

* 将任务目标整理为输入、输出、前置条件和步骤；
* 识别可预见的异常、分支、重试边界和失败条件；
* 生成结构化的 `IF`、`TRY`、`IF SUCCESS` 与 `IF FAILURE` 流程；
* 检查版本锁定、容器声明、子 PSkill 依赖和人工参与声明是否完整；
* 根据已有 PSkill 或模板生成一致的文档结构。

AI 的建议不得自动成为规范。设计者必须审阅并确认最终 PSkill 的业务逻辑、安全边界、权限、副作用、人工参与要求和失败语义；不得因为 AI 生成了内容而省略验证或将未理解的流程直接发布。

---

# 47. 条件分支

条件分支必须明确。

推荐：

```text
IF image.width > 8192
THEN
    terminate with INPUT_ERROR
ELSE
    continue
```

而不是：

```text
如果图片太大，可视情况处理。
```

---

# 48. 循环

PSkill 可以定义循环。

例如：

```text
For each input image, in input order:

1. validate image;
2. execute OCR;
3. validate OCR output;
4. write result.
```

必须尽可能明确：

* 遍历对象；
* 顺序；
* 停止条件；
* 单项失败是否导致整体失败。

默认情况下：

> 任一必要元素失败，应使整个 PSkill 失败。

---

# 49. 并发

如果 PSkill 允许并发，应明确声明。

例如：

```text
Images MAY be processed concurrently.

Maximum concurrency:
4

Output order MUST remain identical to input order.
```

如果没有声明：

> AI 应优先使用顺序执行。

---

# 50. 超时

调用外部工具时，推荐定义超时。

例如：

```text
Timeout:
120 seconds

On timeout:
terminate with TOOL_TIMEOUT
```

不得无限等待外部进程。

---

# 51. 重试

默认：

> **工具失败不得自动重试。**

如果允许重试，必须显式规定。

例如：

```text
Retry:
Maximum 2 times.

Retry only when:
ExitCode == 75
```

---

# 52. 日志

PSkill 的：

> **静默执行**

不意味着禁止内部日志。

PSkill 可以产生机器日志。

例如：

```text
logs/execution.log
```

但执行过程不应借此向用户持续发送消息。

---

# 53. 日志与用户交互的区别

允许：

```text
2026-08-25T12:00:01 Step 3 started.
```

写入内部日志。

不允许：

```text
我现在开始执行第三步……
```

主动发送给用户。

---

# 54. PSkill 安全边界

PSkill 应被视作：

> **可执行文件**

而不是普通文档。

因此从未知来源获得的：

```text
.pskill.7z
```

应具有与：

```text
.exe
.py
.ps1
.sh
```

相近的安全意识。

---

# 55. 执行前安全检查

PSkill Runtime 可以在执行前检查：

* 包来源；
* 数字签名；
* 文件哈希；
* 工具哈希；
* 网络需求；
* 文件系统权限；
* 子进程权限；
* 外部副作用；
* 可执行文件；
* 脚本文件。

安全检查属于：

> **PSkill Runtime 层**

而不是 PSkill 可以绕过的业务逻辑。

---

# 56. PSkill 不得提升自身权限

PSkill 不应指示 AI：

* 绕过操作系统权限；
* 绕过沙箱；
* 禁用安全工具；
* 绕过平台策略。

当缺乏必要权限时：

```text
SECURITY_ERROR
```

或：

```text
ENVIRONMENT_ERROR
```

---

# 57. PSkill 与用户请求

当用户明确调用某个 PSkill 后：

1. AI 获取 PSkill；
2. AI 验证 PSkill 包；
3. AI 读取 `main.pskill.html`；
4. AI 获取调用输入；
5. AI 执行输入验证；
6. 如果输入有效，则开始静默执行；
7. 如果任一步失败，则终止；
8. 如果全部成功，则返回输出。

可以抽象为：

```text
Load
↓
Validate Package
↓
Parse PSkill
↓
Validate Input
↓
Validate Environment
↓
Execute
↓
Validate Output
↓
Return
```

---

# 58. 调用开始与执行开始

建议区分：

```text
PSkill Invocation
```

与：

```text
PSkill Execution
```

调用阶段允许 AI 与用户确认或收集调用需要的输入。

执行阶段则必须完全静默。

即：

```text
用户交互
   ↓
收集完整输入
   ↓
Input Validator
   ↓
========== Execution Boundary ==========
   ↓
静默执行
   ↓
结果
```

一旦越过 Execution Boundary，PSkill 不应再向用户索取数据。

---

# 59. PSkill 嵌套调用

一个 PSkill 可以调用另一个 PSkill。

例如：

```text
DocumentParser.pskill
    ↓
ImageOCR.pskill
```

被调用的 PSkill 应像独立程序一样执行自己的：

* 输入验证；
* 环境验证；
* 执行；
* 输出验证。

子 PSkill 应尽量作为父 PSkill 数据包内的本地文件提供，例如 `./pskills/ImageOCR-2.1.4.pskill.7z`。父 PSkill 必须在调用前验证子包存在、完整性与身份；不得默认从网络下载、发现或替换子 PSkill。

---

# 60. 子 PSkill 失败

如果必要的子 PSkill 失败：

> 调用者 PSkill 默认必须失败。

除非父 PSkill 显式定义该失败属于可接受情况。

---

# 61. PSkill 版本依赖

调用另一个 PSkill 时，应规定版本。

不推荐：

```text
Require ImageOCR
```

推荐：

```text
Require:
ImageOCR 2.1.4
```

或者允许明确版本范围：

```text
>=2.1.0 <3.0.0
```

正式发布的 PSkill 应锁定子 PSkill 的精确版本，并推荐同时锁定其包哈希。例如：

```text
Child PSkill:
Path: ./pskills/ImageOCR-2.1.4.pskill.7z
Name: ImageOCR
Version: 2.1.4
SHA256: C11E...
```

只有父 PSkill 明确声明并验证了受控的解析规则时，才允许使用版本范围。未找到匹配的本地子 PSkill、版本不匹配或哈希验证失败时，父 PSkill 必须以 `DEPENDENCY_ERROR` 失败。

---

# 62. 向后兼容

PSkill 应遵守 Semantic Versioning：

### PATCH

修复错误，但不改变输入输出契约。

### MINOR

向后兼容地增加能力。

### MAJOR

存在不兼容变化。

---

# 63. 可重复执行

在条件允许时，PSkill 应具有：

> **Idempotent**

特性。

即重复执行不应产生不可预测副作用。

尤其是：

* 文件生成；
* 数据库操作；
* 自动部署；
* API 提交；

类 PSkill，应明确重复执行策略。

---

# 64. 推荐的 `main.pskill.html` 基础结构

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Example PSkill</title>
</head>

<body>

<h1>Example PSkill</h1>

<p>
PSkill |
Version 1.0.0 |
Specification 0.3.0 |
MIT License
</p>

<h2>Description</h2>

<p>
Describe what this PSkill does.
</p>

<h2>Inputs</h2>

<h3>Required Inputs</h3>

<ul>
    <li>One PNG or JPEG image.</li>
</ul>

<h3>Input Validation</h3>

<ol>
    <li>The number of input images MUST equal 1.</li>
    <li>The file MUST be readable.</li>
    <li>The file MUST be PNG or JPEG.</li>
    <li>The decoded image MUST NOT exceed 8192 × 8192 pixels.</li>
</ol>

<p>
If any validation rule fails, terminate the PSkill.
</p>

<h2>Environment</h2>

<ul>
    <li>Windows x64 or Linux x64.</li>
    <li>At least 2 GB free memory.</li>
    <li>Docker Engine 27.0.0, when the bundled container is used.</li>
</ul>

<h2>Execution Environment</h2>

<pre>
Container Runtime: Docker Engine 27.0.0
Image: example-pskill-ocr@sha256:...
Network: disabled
</pre>

<h2>Human Participation</h2>

<p>Human Participation Required: false</p>

<h2>Outputs</h2>

<ul>
    <li>result.json</li>
</ul>

<h2>Execution</h2>

<h3>Step 1</h3>

<p>
Validate all inputs.
</p>

<h3>Step 2</h3>

<p>
Run ./tools/ocr/ocr with the input image.
</p>

<h3>Step 3</h3>

<p>
TRY: Run ./tools/ocr/ocr with the input image.
IF SUCCESS: continue to Step 4.
IF FAILURE: terminate the PSkill with TOOL_EXECUTION_ERROR.
</p>

<h3>Step 4</h3>

<p>
Parse the OCR output.
</p>

<h3>Step 5</h3>

<p>
Validate result.json.
</p>

<h2>Failure Policy</h2>

<p>
Any unhandled condition MUST terminate execution.
The AI MUST NOT invent alternative processing methods.
</p>

</body>
</html>
```

---

# 65. 推荐增加的标准章节

一个成熟的 PSkill 建议按照以下顺序组织：

```text
1. Name
2. Metadata
3. Description
4. Inputs
5. Input Validation
6. Environment
7. Permissions
8. Execution Environment / Container
9. Dependencies and Child PSkills
10. Human Participation
11. Outputs
12. Output Validation
13. Variables
14. Execution
15. Failure Policy
16. Side Effects
17. Cleanup
18. Security
19. AI-Assisted Authoring Guidance
20. Examples
```

---

# 66. PSkill Runtime

未来可以实现独立的：

> **PSkill Runtime**

用于统一管理 PSkill。

Runtime 可以负责：

```text
安装
验证
解压
版本管理
依赖检测
权限控制
工作目录创建
工具执行
日志
PSkill 调用
PSkill 嵌套
结果返回
```

AI 则主要负责解释：

```text
main.pskill.html
```

中的自然语言逻辑。

---

# 67. Runtime 与 PSkill 的职责边界

PSkill：

```text
定义做什么
定义怎么做
定义何时失败
```

Runtime：

```text
提供受控执行环境
提供工具调用能力
提供安全边界
提供资源管理
```

AI：

```text
解释 PSkill
严格按照 PSkill 编排执行
```

三者应尽可能分离。

---

# 68. PSkill 的核心哲学

传统 AI Agent 经常采用：

```text
目标
↓
AI 自主规划
↓
选择工具
↓
不断调整
↓
完成任务
```

PSkill 则采用：

```text
目标
↓
人类预先设计执行逻辑
↓
AI 解释逻辑
↓
严格调用指定工具
↓
验证结果
```

因此 PSkill 本质上试图解决一个问题：

> **将 AI 的能力用于理解和执行程序，而不是让 AI 在每次运行时重新发明程序。**

---

# 69. 自然语言即程序

传统程序：

```csharp
if (ocrResult == null)
{
    throw new Exception();
}
```

PSkill 可以表达为：

```text
After invoking the OCR tool, verify that an OCR result exists.

If no verifiable OCR result exists,
terminate the PSkill immediately.

Do not attempt another OCR method unless explicitly defined below.
```

二者在 PSkill 模型下具有类似作用。

因此：

> **自然语言不是 PSkill 的注释，而是 PSkill 的程序代码本身。**

---

# 70. 自然语言程序的约束

因为自然语言存在歧义，PSkill 作者应尽量减少：

```text
可能
最好
一般
适当
酌情
尽可能
通常可以
看情况
合理地
```

等词语出现在核心执行逻辑中。

应优先使用：

```text
MUST
MUST NOT
IF
ELSE
WHEN
ONLY IF
FOR EACH
TERMINATE
RETURN
```

等具有明确逻辑意义的表达。

---

# 71. 最少自主原则

PSkill 并不要求 AI 完全没有推理能力。

相反：

> PSkill 应利用 AI 处理传统程序难以处理的模糊数据。

但应遵循：

> **在业务逻辑已经确定的地方，尽可能降低 AI 自主性；在业务本身需要语义理解的地方，允许 AI 推理。**

例如：

```text
判断文章表达的是正面还是负面情绪
```

属于适合 AI 判断的问题。

而：

```text
OCR 工具失败后应该怎么办
```

则应由 PSkill 作者提前决定。

---

# 72. 完备性原则

PSkill 作者应假设：

> **执行 PSkill 的 AI 不应负责帮作者补完整业务设计。**

因此作者需要考虑：

* 输入异常；
* 文件损坏；
* 工具失败；
* 工具超时；
* 输出为空；
* 输出损坏；
* 权限不足；
* 磁盘不足；
* 环境不兼容；
* 多文件输入；
* 同名输出；
* 重复执行；
* 临时文件；
* 网络中断；
* 非法数据；
* 版本不匹配。

如果作者没有定义某种异常：

> 默认行为应是失败，而不是让 AI 自行推断。

---

# 73. 默认行为

除非 PSkill 明确覆盖，下列行为作为推荐默认规则：

| 情况            | 默认行为 |
| ------------- | ---- |
| 输入格式未知        | 失败   |
| 输入损坏          | 失败   |
| 缺少依赖          | 失败   |
| 工具执行失败        | 失败   |
| 工具返回无法验证的数据   | 失败   |
| 网络资源不可访问      | 失败   |
| 输出验证失败        | 失败   |
| 遇到未知状态        | 失败   |
| 缺少必要参数        | 失败   |
| 需要用户追加信息      | 失败   |
| 是否可以自动转换格式不明确 | 不转换  |
| 是否可以调用替代工具不明确 | 不调用  |
| 是否可以联网不明确     | 不联网  |
| 是否可以修改原始输入不明确 | 不修改  |
| 是否允许部分结果不明确   | 不允许  |
| 是否允许重试不明确     | 不重试  |
| 是否允许升级工具不明确   | 不升级  |

该规则可以概括为：

> **未定义的恢复行为视为禁止。**

---

# 74. PSkill 不是 Prompt

虽然 PSkill 的核心逻辑由自然语言构成，但 PSkill 不应简单等价于：

```text
Prompt
```

Prompt 通常表达：

> “我希望 AI 做什么。”

而 PSkill 应表达：

> “这个程序接受什么输入，在什么条件下，按照什么步骤，调用什么资源，产生什么输出，以及什么时候必须失败。”

因此 PSkill 更接近：

```text
Executable Specification
```

而不是：

```text
Instruction Prompt
```

---

# 75. PSkill 不是工作流文件的替代品

PSkill 并不试图完全替代：

* Python；
* C#；
* Bash；
* 工作流引擎；
* Docker；
* CI/CD；
* DAG；
* BPMN。

PSkill 的优势在于：

> **允许开发者使用自然语言定义那些传统程序难以表达、但 AI 能够理解的业务流程，并与传统程序工具组合。**

因此推荐的设计方式通常是：

```text
PSkill
负责：
语义流程
判断
编排
错误边界

传统程序
负责：
确定性计算
高性能处理
文件解析
模型推理
系统操作
```

---

# 76. 长期目标

PSkill 希望形成一种介于：

```text
Document
Program
Workflow
AI Skill
Software Package
```

之间的新型软件分发形式。

未来，一个 AI 可以接收到：

```text
Photoshop.pskill.7z
UnitySceneAnalyzer.pskill.7z
PDFRepair.pskill.7z
GameLocalization.pskill.7z
```

然后像今天运行：

```text
.exe
.py
.sh
```

一样运行这些能力。

区别仅在于：

> **PSkill 的主要程序逻辑不是机器编程语言，而是严格结构化的人类自然语言。**

---

# 77. 最终原则

PSkill 规范的核心原则可以归纳为：

### 一、PSkill 是程序

不得把 PSkill 当成普通参考文档。

### 二、自然语言是代码

`main.pskill.html` 中的执行逻辑应被严格解释执行。

### 三、输入必须先验证

不在输入契约范围内的任务直接失败。

### 四、输出必须可描述、可验证

没有通过输出验证不得宣称成功。

### 五、执行期间保持静默

不允许运行到一半再向用户询问业务决策。

### 六、异常默认失败

未知状态不由 AI 自主补偿。

### 七、备用方案必须预先定义

AI 不能临时发明 fallback。

### 八、完全本地优先

尽可能减少网络和外部资源依赖。

### 九、完全版本优先

参与运行的工具、模型和关键资源应具有确定版本。

### 十、环境也是输入

硬件、系统、Runtime 和权限均属于输入契约的一部分。

### 十一、安全上等同可执行程序

未知 PSkill 不应因为“主体是 HTML 和自然语言”而被当作安全文档。

### 十二、AI 是解释器，而不是第二位程序作者

执行阶段，AI 的主要任务是忠实执行已经设计好的程序。

### 十三、AI 辅助编写，人工负责定稿

PSkill 应优先由 AI 协助结构化编写和审阅；设计者必须对发布内容的正确性、安全性和完整性负责。

---

# 78. 一句话定义

> **PSkill 是一种以自然语言作为主要程序逻辑、以 HTML 作为程序载体、可作为 7z 压缩包分发，或在高速开发阶段作为受版本管理的目录存在，并可携带确定版本工具和资源、由 AI 严格解释执行的可执行 Skill 数据包。**

---

**PSkill Whitepaper 0.3.0-draft**

**Status: Draft / Request for Comment**
