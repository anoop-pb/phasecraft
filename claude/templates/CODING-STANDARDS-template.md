# Coding standards

Baseline for all code and tests in this repo. The user edits or replaces this file at onboarding. These rules override the developer's defaults for style; they never override the architecture files.

## General

- Production quality, not prototype quality.
- The simplest code that meets the spec. No abstraction, layer, or configuration point the specs do not need yet.
- Every error is handled or passed on deliberately, never dropped. Handle it consistently at the layer the architecture assigns, with errors from user input and external dependencies handled explicitly.
- Avoid shared mutable state. Where it is needed, one owner guards it.
- Comments for complex logic only; code should be self-documenting.
- Match existing patterns before writing new ones: naming, error handling, file placement, library choices.
- Before writing a new feature, find a similar existing one and follow its pattern.
- Dependencies point one way between layers, as `ARCHITECTURE.md` defines them. Lower layers never import higher ones, such as UI or API.
- No unrequested features.

## Commits

- Conventional Commits. Override here if the repo uses another convention.

## Stack-specific

{Empty until filled. The user may fill it at onboarding; otherwise the developer appends a section for the chosen stack at the first build — formatting, linting, naming, test patterns, and the commands that enforce them, including lint rules for function length, complexity, and duplication where the linter supports them — and reports it.}

## Verification commands

The operator runs these from the code root after every build and fix-build. A non-zero exit from any of them fails the build. Filled with the stack section.

- Full test suite: {command}
- Lint: {command}
- Format check: {command}
- Type check: {command, or — where the stack has none}
- Dependency audit: {command, or — where the stack has none}
