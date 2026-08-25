# Contributing to the PSkill Whitepaper

Thank you for helping improve the PSkill specification.

## Proposing a change

1. Open a pull request that explains the problem, the proposed normative change, and any compatibility impact.
2. Keep requirements precise. Use the RFC 2119-style terms defined by the whitepaper (`MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY`) where normative meaning is intended.
3. Include examples when they clarify a new execution, validation, dependency, container, or failure rule.

## Language synchronization

The English edition, `docs/pskill-whitepaper.en.md`, is the authoritative edition. `docs/pskill-whitepaper.zh-CN.md` is an official Simplified Chinese translation. Every specification change must update both documents in the same pull request. Their section structure, version, status, and normative meaning must remain aligned.

## Versioning

The whitepaper follows Semantic Versioning:

- `PATCH`: clarifications and corrections that do not change the specification contract.
- `MINOR`: backward-compatible additions.
- `MAJOR`: incompatible changes to the specification.

Update every visible version reference when changing the version. Draft status remains `Draft / Request for Comment` unless the maintainers explicitly change it.

## Review expectations

PSkill is an executable specification. Reviewers should check that a change has clear inputs, decision branches, success and failure semantics, safety boundaries, and no unstated recovery behavior.
