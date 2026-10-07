This file defines global rules for agents. It is not a description of the application's runtime architecture and not an instruction to perform any particular task.

## 1. Understand the existing project

Before making changes, inspect the relevant existing architecture, conventions, implementation, tests, documentation, and current working-tree state.

Agents should consult project documentation files, such as README.md, CONTRIBUTING.md, etc., when they are relevant to the task.

Treat the actual implementation and tests as authoritative about what currently exists. The project documentation, on the other hand, is authoritative about intended design.

If the implementation, tests, and project documentation disagree, do not silently resolve the discrepancy. Determine whether the difference is intentional; otherwise flag and report it.

Do not import a generic application architecture if the repo already has one.

When inspecting large data files or exports, do not dump the entire file into context. Use targeted inspection tools such as `head`, `jq`, or small extracted fixture snippets to understand the data schema.

## 2. Preserve the working project

GitHub is the canonical source for the project's committed code and history.

The current working tree may contain legitimate uncommitted changes. Do not discard, reset, overwrite, or otherwise destroy existing work merely to make the workspace match GitHub.

When beginning work, inspect the actual repository and working-tree state before deciding what needs to change.

Preserve existing behavior unless the requested change intentionally changes it.

Do not use destructive Git operations unless the user explicitly requests them.

## 3. Build the requested behavior

When implementing requested behavior:

1. Understand the intended outcome.
2. Inspect the relevant code, tests, and project decisions.
3. Translate the request into concrete, observable behavior.
4. Implement it using the existing architecture where practical.
5. Add or update tests that verify the behavior.
6. Run the relevant checks.
7. If something fails, investigate and iterate rather than stopping at the first failure.
8. Verify that the finished implementation actually satisfies the request.

If the intended behavior is clear, proceed without waiting for approval.

Ordinary implementation choices may be made without asking the user, including choices about files, functions, components, abstractions, and test structure.

Do not invent product requirements merely because they seem conventional or technically convenient.

If a genuinely unresolved choice would establish product behavior, domain semantics, or architecture, ask the user rather than silently deciding for her.

If it is only an implementation detail, choose a sensible solution and proceed.

Use project-local tooling. Run commands using the project's established local environment, such as `uv run pytest`, `.venv/bin/pytest`, or `npm test`. Do not install packages globally or modify system-level Python.

## 4. Respect established architecture and domain meaning

Prefer extending the repo's existing architecture over replacing it with a framework, pattern, service, abstraction, or dependency that is merely familiar or convenient.

Do not introduce infrastructure, state-management systems, persistence mechanisms, or architectural layers without a concrete need.

Do not refactor unrelated code while implementing a feature unless the refactoring is necessary for correctness or maintainability of the requested change.

Do not import assumptions from generic CRUD applications or other software domains.

Preserve the distinctions and semantics established by existing code, tests, and project decisions.

When those sources leave a domain question unresolved, do not invent semantics merely to simplify implementation.

## 5. Tests are part of the specification

Tests should describe and protect observable behavior.

When implementing new behavior, create or update tests for the important acceptance criteria and failure cases.

Do not weaken, delete, bypass, or rewrite a test merely because the implementation cannot satisfy it.

If a test conflicts with clearly established intended behavior, investigate the discrepancy and correct the test only when the intended behavior is actually known.

A passing test suite is necessary but not sufficient. Use tests, type checking, builds, and other appropriate verification available in the project, and inspect the resulting behavior rather than assuming that passing checks means the feature is correct.

Do not run blocking or persistent processes such as `uvicorn`, `fastapi run`, `npm run dev`, or interactive shells as verification steps. Use automated test suites, build commands such as `npm run build`, or one-off verification scripts that terminate cleanly.

## 6. Keep scope under control

Build what is requested and what is necessary to support it.

Do not turn individual features into opportunities to build generalized infrastructure or speculative functionality.

Future possibilities are not requirements.

In particular, do not introduce unrelated:

* workflow engines
* complete metadata-management systems
* publication or migration systems
* authentication or user management
* preservation infrastructure
* AI features
* bulk-processing infrastructure
* analytics
* elaborate design systems
* abstractions for hypothetical future requirements

## 8. Communicate meaningful results

Be direct about meaningful decisions, assumptions, failures, unresolved questions, and verification.

Do not narrate every implementation step.

When reporting completed work, briefly state:

* what was changed
* what was verified
* any tests or checks that could not be run or did not pass
* any consequential issue Liz needs to decide

Do not claim that something works merely because the code was written or a build succeeded.

The goal is not merely to produce code that compiles. The goal is to make the code do what the user actually needs it to do while preserving the project she is building.
