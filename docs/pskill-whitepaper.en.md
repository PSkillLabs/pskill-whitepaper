# PSkill Whitepaper

**Also known as: Executable Skill Document Specification**  
**Whitepaper version: 0.4.0-draft**

---

## 1. Overview

PSkill (Portable / Procedural Skill, provisional name) is an enhanced executable package specification for AI execution environments. Unlike a conventional Skill document, it defines a process with explicit inputs, outputs, steps, tools, branches, validation, and failure conditions that an AI runtime must execute.

A PSkill is neither a prompt collection nor a knowledge base. It is a natural-language program and the resources required to run it. Its main logic normally lives in `main.pskill.html`; HTML is a structured document container, not a web application.

A complete PSkill consists of a natural-language program, versioned tools and data, versioned child PSkills, deterministic input/output contracts, and deterministic failure semantics. It may carry executables, scripts, runtimes, WASM modules, models, templates, configuration, and offline resources.

# 2. Design goals

## 2.1 Executability

PSkill MUST specify accepted and rejected inputs, validation, execution order, tools, success tests, failure cases and branches, human participation where applicable, and final outputs. A conforming PSkill must not require the AI to redesign the business process.

## 2.2 Reproducibility

Given the same PSkill version, input, execution environment, tools, model assets, and configuration, a PSkill SHOULD produce the same or semantically equivalent result. It SHOULD minimize the influence of changing services, APIs, software, models, and operating-system state.

## 2.3 Completeness

Authors SHOULD package everything required to perform the task. The Local-First and Fully-Versioned principles mean a package should run offline where possible and every critical tool and resource must have an explicit version.

## 2.4 Fail-Closed

PSkill uses fail-closed behavior, not best effort. Any condition that cannot be validated according to the specification MUST fail the PSkill. The AI MUST NOT alter business logic merely to obtain a plausible-looking result.

# 3. Normative terminology

**MUST / required** means a conforming implementation is required to comply. **MUST NOT / prohibited** means the behavior is forbidden. **SHOULD / recommended** means deviation needs a sound reason. **SHOULD NOT / discouraged** means deviation is normally inappropriate. **MAY / optional** means an optional behavior.

**IF** declares a verifiable condition and a unique branch. It SHOULD state both true and false destinations; an uncovered state MUST fail. **TRY** declares a controlled operation that may fail, together with success and failure tests and destinations. It never authorizes the AI to invent an alternative.

**IF SUCCESS** is entered only when a step satisfies its declared success criterion. **IF FAILURE** is entered only when a declared failure criterion is met; absent a declared recovery path, it MUST terminate with an explicit error code.

**HUMAN STEP** is a declared action that a human must perform and the AI must neither impersonate nor fabricate, such as judging continuity of image frames, validating a physical robot, or completing real-person identity verification.

# 4. PSkill package

The standard released distribution format is `7z`, for example `ImageOCR.pskill.7z`. Split 7z archives are allowed, but all required volumes must be present before execution. A missing volume MUST result in `PSkill Load Failed`; partial execution is prohibited.

For a PSkill under active, high-velocity development, the package MAY instead be distributed and executed as its unpacked root directory. The directory MUST preserve the package layout and contain `main.pskill.html` at its root. This form exists so that individual PSkill files can be version-controlled alongside the engineering project; the runtime MUST apply the same layout, entry-point, dependency, resource, and integrity validation that it applies to the extracted contents of a 7z package, and MUST NOT require an intermediate archive or unpacking step. A released package MAY still be assembled from that directory into a 7z archive.

# 5. Recommended package layout

```text
MyPSkill/
├── ReadMe.md
├── main.pskill.html
├── tools/
├── models/
├── scripts/
├── resources/
├── pskills/
│   └── ImageOCR-2.1.4.pskill.7z
├── examples/
└── LICENSE
```

Only `ReadMe.md` and `main.pskill.html` are core files; other directories are conventions.

# 6. `ReadMe.md`

Every released package SHOULD include `ReadMe.md` for human developers and distribution platforms. It should state name, purpose, version, platforms, input/output types, dependencies, size, license, author, project URL, and known limits. It does not define executable logic: `main.pskill.html` prevails on conflict.

# 7. `main.pskill.html`

`main.pskill.html` is the main program entry point. Use simple semantic HTML such as headings, paragraphs, lists, tables, preformatted blocks, code, images, and links. Do not design it as a traditional interactive site: JavaScript UI, SPA frameworks, forms, animated or presentation-heavy DOM, and web components are discouraged. Machine comprehension has priority over visual design.

# 8. PSkill header

The document MUST begin with the PSkill name, followed by metadata badges or equivalent text. Typical data includes `PSkill`, a package version, `Spec 0.4`, license, offline status, and supported platforms. Badges may be text, SVG, or local images.

# 9. Metadata

Metadata MUST declare the PSkill name, a Semantic Versioning package version (`MAJOR.MINOR.PATCH`), and the PSkill specification version (`PSkill Specification: 0.4.0`). It SHOULD declare a license. Non-standard licenses must be included in the package or linked unambiguously.

# 10. Input contract

The first major body section MUST be **Input**. It MUST enable validation before execution starts: accepted inputs, rejected inputs, constraints, and transformation rules that are expressly permitted.

# 11. Inputs are broad

Inputs include user files and parameters, but also the operating system, CPU/GPU, memory, runtimes, permissions, environment variables, installed software, network state, working directory, and available AI tool capabilities. The execution environment is itself an input.

# 12. Input validator

Every PSkill MUST logically define an Input Validator. It checks quantity, type, format, extension, content, size, integrity, required environment, tools, runtimes, and hardware. Where relevant it also checks image dimensions/color space, media codecs or sample rate, JSON Schema, page count, model format, magic numbers, and corruption.

# 13. Invalid input

If an input violates the contract, the PSkill MUST immediately fail. For example, a PNG/JPEG-only PSkill MUST return `Unsupported Input` for PSD; it may convert PSD only when an explicit conversion through a named tool is declared.

# 14. Output contract

After inputs, the PSkill MUST define outputs: what is generated, format, purpose, whether it is ready for delivery, and whether a later PSkill consumes it. Specify filenames and types, such as UTF-8 JSON, `result.json`, and `preview.png`.

# 15. Verifiable output

PSkill SHOULD state how outputs are validated. A JSON result, for example, can require UTF-8, valid JSON, required fields, and coordinates inside image bounds. If final output cannot pass its declared validation, the whole PSkill MUST fail and MUST NOT describe unvalidated output as success.

# 16. Execution model

Execution is an explicitly ordered series of numbered steps, preferably with nested numbering. Avoid open-ended phrases such as “handle as appropriate” or “choose a suitable method.”

# 17. AI is an interpreter, not a business designer

The AI acts as an interpreter, runtime, and orchestrator. It must follow declared business decisions. If a PSkill specifies a bundled OCR executable, the AI MUST NOT replace it with a better tool that happens to exist locally.

# 18. Strict execution

Declared instruction overrides AI preference. Nothing in a PSkill overrides platform/security policy, user authorization, sandbox restrictions, legal limits, or explicit user permission limits. The priority order is Platform/Security Policy, User Authorization, PSkill Specification, then AI general reasoning.

# 19. Silent execution

After execution starts, a PSkill MUST be silent to the user. It must not request choices, confirmation, more data, or a different method. Required information must be supplied before execution or determinable by the PSkill; otherwise it fails.

# 20. No mid-execution input

All required inputs must be declared and collected before the Execution Boundary. A color profile, for example, is an explicit required parameter with allowed values, not a question at Step 5.

## 20.1 Human participation

Human participation is not undeclared extra input. A PSkill may include predeclared human steps for tasks AI should not replace: visual frame-continuity review, physical robot validation, or real-person identity verification.

Any PSkill with a human step MUST declare `Human Participation Required: true`, step identifiers, and roles. Each human step MUST state role, task, materials, acceptance criteria, verification record, wait limit, and error code for refusal, timeout, failed verification, or unavailability. The runtime may notify and collect the declared result; this is a defined workflow boundary, not an ad-hoc request for a business decision. If the step cannot complete as specified, the PSkill MUST fail.

# 21. Fail-closed execution

If an OCR process exits with code 1, the AI must not read the image itself, switch tools, use online OCR, invoke another model, ignore the error, return partial results, or modify the image, unless an explicit fallback permits it. The default is `PSkill Failed`.

# 22. No implicit compensation

Undeclared automatic repair, format conversion, re-encoding, installation, updating, model download, network access, tool replacement, quality reduction, parameter guessing, or input modification is prohibited.

# 23. Explicit fallback paths

A declared fallback is permitted, for example: invoke Engine A; only for exit code 12 invoke Engine B; for every other error terminate. Authors SHOULD anticipate foreseeable errors such as tool exit codes, timeouts, resource exhaustion, missing dependencies, network interruption, incomplete data, and human-step failure.

```text
TRY: invoke OCR Engine A.
IF SUCCESS: validate and continue to Step 5.
IF FAILURE AND ExitCode == 12: invoke OCR Engine B.
IF FAILURE otherwise: terminate with TOOL_EXECUTION_ERROR.
```

Every recovery path needs its own success, failure, and termination semantics. An unforeseen or undefined state MUST enter `FAILED` with an appropriate explicit error code; it may never continue implicitly.

# 24. Error boundaries

Recommended categories are `LOAD_ERROR`, `INPUT_ERROR`, `ENVIRONMENT_ERROR`, `DEPENDENCY_ERROR`, `EXECUTION_ERROR`, `VALIDATION_ERROR`, `OUTPUT_ERROR`, and `SECURITY_ERROR`.

# 25. Execution state

Recommended states are `READY` (input validation complete), `RUNNING`, `SUCCEEDED` (all required steps and output validation succeeded), and `FAILED` (a non-recoverable step failed).

# 26. Atomic success

Unless Partial Result is explicitly supported, success is atomic: success or failure, never “mostly successful” or “probably usable.”

# 27. Failure report

Silent execution does not prohibit a final failure report. It SHOULD include PSkill name, package version, status, failed step, error code, and factual reason. Do not substitute speculation for the actual cause.

# 28. Local-first

Resources needed for core work SHOULD be packaged locally. A bundled, versioned OCR model is preferred over downloading the latest model during execution.

# 29. Container-first

Authors SHOULD prefer Docker or an OCI-compatible container to relying on a user’s locally installed runtimes, libraries, CLI tools, or system configuration. A container fixes the base image, runtimes, libraries, critical tools/models, work directory, entry point, and access boundaries.

Containerized PSkills MUST provide a Dockerfile, image archive, or verifiable image reference and declare runtime version, immutable image digest, entry point, network policy, input mounts, and output mounts. Do not use `latest`, floating tags, or auto-updating base images. Missing runtime/image/mount/device/network conditions MUST fail as `ENVIRONMENT_ERROR` or `DEPENDENCY_ERROR`.

Local dependencies are permitted only where containerization is unreasonably obstructive, unsupported, or explicitly unsuitable; the PSkill must explain why and fully list, lock, and validate local dependencies.

# 30. Network dependencies

Network access is allowed only when explicitly declared with required/optional status, endpoint, purpose, and failure policy. Undeclared network access is forbidden; an unavailable required endpoint must terminate the PSkill.

# 31. No “latest” dependencies

Released PSkills SHOULD NOT use `latest`, `current`, `newest`, `stable`, or `auto-update`. Use exact or otherwise uniquely determinable versions, such as Python 3.12.8 rather than Python >= 3.

# 32. Tool versions

Every critical tool MUST have a declared version, including package-maintained utilities. Typical declarations include Tesseract, FFmpeg, Python, ONNX Runtime, and package executables.

# 33. Tool integrity

Tools SHOULD include SHA-256 values so a runtime can verify they were not modified.

# 34. Package integrity

Publishers MAY provide a package SHA-256 or signature file. Future PSkill versions may define a standard signature format.

# 35. Platform compatibility

PSkill MUST declare supported operating systems and architectures, or an exact portable environment requirement. A nonconforming environment is `ENVIRONMENT_ERROR`; do not build an undeclared compatibility layer.

# 36. Multi-platform tools

Platform-specific tools SHOULD be isolated by platform directory. The PSkill must state platform-selection logic.

# 37. Paths

Use paths relative to the PSkill root, not developer-machine absolute paths.

# 38. Working directories

Distinguish Package Directory, Input Directory, Working Directory, Output Directory, and Temporary Directory to avoid unintended side effects.

# 39. Input modification

PSkill SHOULD NOT change original input resources. Prefer `Input → Working Copy → Processing → Output`. Direct modification must be explicitly declared in the input definition.

# 40. Temporary files

Specify temporary-file location, lifetime, and cleanup. Whether failures preserve debugging data must also be declared.

# 41. External side effects

Deletion or modification of files, database changes, email/message sending, Git commits/pushes, deployment, upload, publication, or system configuration changes are side effects. They must be disclosed in the input/capability declaration before execution.

# 42. Least privilege

PSkill SHOULD request only the permissions necessary. Read-only OCR does not justify administrator access or global filesystem write access.

# 43. AI capabilities are not implicit dependencies

Capabilities such as Photoshop, Blender, Python execution, internet access, GPU access, and external tools must be declared as environment requirements; do not assume them.

# 44. Built-in AI reasoning

PSkill may use AI language, vision, or reasoning for semantic work, but should tightly define inputs, criteria, allowed outputs, and validation.

# 45. Non-deterministic steps

For inherently non-deterministic operations—generation, visual understanding, classification, summaries, or LLM reasoning—use schema, enumerations, examples, rules, and validators to reduce output freedom.

# 46. Natural-language program design

A `.pskill.html` SHOULD include Inputs, Preconditions, Environment, Outputs, Variables, numbered Steps, Conditions, Validation, and Failure Conditions.

## 46.1 AI-assisted authoring

PSkill authors SHOULD use AI to author, review, and maintain PSkills rather than relying only on manual drafting. AI may structure goals into contracts and steps, identify foreseeable branches and failures, produce `IF`/`TRY` flow, check version/container/child-skill/human-step declarations, and create consistent templates.

AI suggestions are not automatically normative. Authors MUST review and approve final logic, safety boundaries, permissions, side effects, human-participation needs, and failure semantics. Never publish an unreviewed or misunderstood AI-generated flow.

# 47. Conditional branches

Conditions must be explicit and testable: `IF image.width > 8192 THEN terminate with INPUT_ERROR ELSE continue`, not “handle large images as appropriate.”

# 48. Loops

Loops must state the iterated objects, order, stop condition, and whether an item failure fails the whole PSkill. By default, failure of any required item fails the PSkill.

# 49. Concurrency

Concurrency must be declared, including maximum concurrency and output ordering. Without such a declaration, the AI should execute sequentially.

# 50. Timeouts

External calls SHOULD have a timeout and a timeout error, and MUST NOT wait indefinitely.

# 51. Retries

Automatic retries are forbidden by default. If permitted, state maximum count and the precise retryable condition.

# 52. Logs

Silent execution permits internal machine logs such as `logs/execution.log`; it does not permit ongoing user-facing progress messages.

# 53. Logs versus user interaction

Writing `Step 3 started` to an internal log is allowed. Proactively telling a user “I am starting Step 3” is not.

# 54. Security boundary

Treat a PSkill package as executable content, not an ordinary document. Untrusted `.pskill.7z` packages deserve the same caution as executables and scripts.

# 55. Pre-execution security checks

The runtime may check origin, signatures, hashes, network requirements, filesystem/subprocess permissions, side effects, executables, and scripts. These are runtime security boundaries that business logic cannot bypass.

# 56. No privilege escalation

PSkill must not instruct the AI to bypass OS permissions, sandboxes, security tools, or platform policy. Missing permission is `SECURITY_ERROR` or `ENVIRONMENT_ERROR`.

# 57. Invocation and execution

On invocation, the AI obtains and validates the package, parses `main.pskill.html`, collects inputs, validates input and environment, then crosses the Execution Boundary for silent execution, output validation, and return. Any failed step terminates.

# 58. Invocation versus execution start

User interaction is permitted before the Execution Boundary to collect complete declared inputs. It is not permitted after that boundary except for a declared human step.

# 59. Nested PSkill invocation

One PSkill may invoke another. The child runs its own input/environment validation, execution, and output validation as an independent program. Child PSkills SHOULD be supplied locally inside the parent package; the parent must verify presence, identity, and integrity before invocation. It must not download, discover, or replace a child by default.

# 60. Child PSkill failure

If a required child PSkill fails, its parent MUST fail unless the parent explicitly declares that failure acceptable.

# 61. Child PSkill versions

Calls must declare child versions. Released PSkills SHOULD lock exact version and package hash, for example a local `./pskills/ImageOCR-2.1.4.pskill.7z`, name, version `2.1.4`, and SHA-256. Version ranges are allowed only with explicit, verified resolution rules. Missing local child, mismatch, or hash failure is `DEPENDENCY_ERROR`.

# 62. Backward compatibility

Package versions follow Semantic Versioning: PATCH fixes defects without changing contracts; MINOR adds backward-compatible capability; MAJOR contains incompatible change.

# 63. Repeatability

Where possible, PSkills SHOULD be idempotent. File generation, database work, deployment, and API submission must state repeat-execution behavior.

# 64. Recommended base structure

```html
<!DOCTYPE html><html><head><meta charset="utf-8"><title>Example PSkill</title></head>
<body><h1>Example PSkill</h1>
<p>PSkill | Version 1.0.0 | Specification 0.4.0 | MIT License</p>
<h2>Inputs</h2><h2>Environment</h2><h2>Outputs</h2><h2>Execution</h2>
<p>TRY: invoke the declared tool. IF SUCCESS: continue. IF FAILURE: terminate.</p>
<h2>Failure Policy</h2><p>Any unhandled condition terminates execution.</p>
</body></html>
```

The complete template should also declare input validation, output validation, container environment where used, and `Human Participation Required: false` or its required human-step details.

# 65. Recommended section order

1. Name; 2. Metadata; 3. Description; 4. Inputs; 5. Input Validation; 6. Environment; 7. Permissions; 8. Execution Environment / Container; 9. Dependencies and Child PSkills; 10. Human Participation; 11. Outputs; 12. Output Validation; 13. Variables; 14. Execution; 15. Failure Policy; 16. Side Effects; 17. Cleanup; 18. Security; 19. AI-Assisted Authoring Guidance; 20. Examples.

# 66. PSkill Runtime

A future runtime can manage installation, validation, unpacking, version/dependency detection, permissions, directories, tool calls, logging, nesting, and result return. AI primarily interprets the declared `main.pskill.html` logic.

# 67. Responsibility boundary

PSkill defines what to do, how to do it, and when to fail. The runtime supplies the controlled environment, tool access, security boundary, and resource management. AI interprets and strictly orchestrates the PSkill.

# 68. Core philosophy

Conventional agents autonomously plan, select tools, and adapt at runtime. PSkill has humans predesign the process, AI interpret it, specified tools execute it, and validators check results. It applies AI to understanding and running the program rather than reinventing it each run.

# 69. Natural language is program code

Natural-language statements such as “verify an OCR result exists; if not, terminate immediately; do not try another method unless defined below” have the role of program code, not comments.

# 70. Natural-language constraints

Avoid ambiguous words such as possible, best, normally, appropriate, discretionary, or reasonable in core execution logic. Prefer `MUST`, `MUST NOT`, `IF`, `ELSE`, `WHEN`, `ONLY IF`, `FOR EACH`, `TERMINATE`, and `RETURN`.

# 71. Minimum autonomy

PSkill does not eliminate AI reasoning. It allows reasoning for genuine semantic tasks, but minimizes autonomy where business logic is predetermined. The PSkill author—not the executing AI—decides what happens after an OCR tool fails.

# 72. Completeness

Authors must assume the executing AI will not finish missing business design. Consider malformed/damaged files, tool failure and timeout, empty/corrupt output, permission and disk limits, environment mismatch, multi-file input, output collisions, repeat execution, temporary files, network interruption, illegal data, and version mismatch. An undefined exception defaults to failure.

# 73. Default behavior

Unknown/damaged input, missing dependencies, failed tools, unverifiable data, inaccessible network resources, failed output validation, unknown state, missing parameter, user information requested after start, and unapproved partial results all fail. Unclear permission to convert, substitute a tool, use network, modify inputs, retry, or upgrade means do not perform that behavior. Undefined recovery is prohibited.

# 74. PSkill is not a prompt

A prompt says what the AI should do. A PSkill says what the program accepts, conditions and steps, resources, outputs, and mandatory failure conditions. It is an executable specification.

# 75. PSkill does not replace workflow files

PSkill does not replace Python, C#, Bash, workflow engines, Docker, CI/CD, DAGs, or BPMN. It combines natural-language semantic process and judgment with traditional programs for deterministic computation, performance, parsing, model inference, and system operations.

# 76. Long-term goal

PSkill aims to become a distribution form between document, program, workflow, AI Skill, and software package: packages such as `Photoshop.pskill.7z` or `PDFRepair.pskill.7z` could be run by an AI as executably as `.exe`, `.py`, or `.sh`, with structured natural language as the primary logic.

# 77. Final principles

1. PSkill is a program. 2. Natural language is code. 3. Validate inputs first. 4. Describe and validate outputs. 5. Remain silent during execution. 6. Unknown exceptions fail. 7. Define fallbacks in advance. 8. Prefer fully local resources. 9. Prefer fully versioned resources. 10. Environment is input. 11. Treat packages as executable content for security. 12. AI interprets rather than becomes a second program author at execution time. 13. Use AI-assisted authoring, while the human designer remains responsible for correctness, security, and completeness.

# 78. One-sentence definition

> **PSkill is an executable Skill package whose main program logic is structured natural language carried in HTML, distributed as a 7z archive or, during active development, as a version-controlled directory, able to carry versioned tools and resources, and strictly interpreted by AI.**

**PSkill Whitepaper 0.4.0-draft**  
**Status: Draft / Request for Comment**
