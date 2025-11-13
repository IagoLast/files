# Agent Workflow Guidelines

## Core Principles

- **KISS**: Keep it simple
- **YAGNI**: You aren't gonna need it - only implement what's explicitly required
- **TDD**: Tests are formal task definitions

## Workflow

- Define expected behavior using tests.
- Write the minimal code to make test pass
- Run tests, build, lint after each step
- Refactor
- Handle errors and boundaries
- Small iterations until done
- After each code change, always run checks (tests, lint build)
