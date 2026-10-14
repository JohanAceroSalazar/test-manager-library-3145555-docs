# test-manager-library-3145555-docs

> **Technical assessment · ADSO Cohort 3145555**
> Schedule: **Wednesday, October 14, 2026, from 6:50 a.m. to 11:00 a.m.** (Colombia time). Submission is due at **11:00 a.m.**

This repository contains the *Manager Library* **specification**. It is the only repository with content; the other five start empty.

## Instructions

Each learner must **fork** the project's six repositories and **complete the project using the
provided specification**.

1. Fork each repository in the table at the end to your GitHub account.
2. Read the specification in [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs).
   It is provided in two versions: `spec-python/` and `spec-.net/`.
3. Develop the solution in the forks.

## The challenge uses Laravel, React, PostgreSQL 16, and Docker Compose

The challenge's mandatory stack is:

| Layer | Challenge technology |
|---|---|
| Backend | PHP with Laravel |
| Frontend | React |
| Database | PostgreSQL 16 |
| Infrastructure | Docker Compose |

The `spec-python/` and `spec-.net/` directories are reference specifications; they do not require
implementing Python or .NET. The technology adaptation to the mandatory stack preserves the
business rules, data model, HTTP contract, and acceptance criteria.

## The assessment is not about writing code

Its main purpose is to assess **Spec-Driven Development (SDD)** capability: how a specification is
read, interpreted, and applied to a different stack. Code is the means, not the end.

## Submission

Only work **pushed to the forks before 11:00 a.m.** counts. Work submitted afterward will not be reviewed.
There are no extensions: push partial progress through small commits during the assessment.

## Repository contents

| Directory | Contents |
|---|---|
| [`spec-python/`](spec-python/) | Reference specification for Manager Library written for a Python backend |
| [`spec-.net/`](spec-.net/) | Reference specification for the same system written for a .NET backend |

Both references describe the **same product** (the same user stories, business rules, data model,
HTTP contract, acceptance criteria, and endpoints); only the technology decisions differ. Neither
is the challenge's mandatory stack.

## How to read the specification

Read either directory in this order:

1. `constitution.md` — non-negotiable principles.
2. `spec.md` — what the system must do: actors, user stories, acceptance criteria, and business rules.
3. `plan.md`, `architecture.md`, and `arquitectura-panoramica.md` — how it is built.
4. `data-model.md` and `api-contract.md` — the data model and API contract.
5. `tasks.md` — what must be done and the evidence required to consider it complete.
6. `adr/` — architecture decisions, their alternatives, and consequences.

## The six repositories

| Repository | Contents |
|---|---|
| [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs) | The specification: `spec-python/` and `spec-.net/` |
| [`test-manager-library-3145555-api`](https://github.com/code-sena/test-manager-library-3145555-api) | PHP backend (Laravel) |
| [`test-manager-library-3145555-app`](https://github.com/code-sena/test-manager-library-3145555-app) | React frontend |
| [`test-manager-library-3145555-page`](https://github.com/code-sena/test-manager-library-3145555-page) | Public static presentation site |
| [`test-manager-library-3145555-infra`](https://github.com/code-sena/test-manager-library-3145555-infra) | Containers, network, volumes, and an empty database engine |
| [`test-manager-library-3145555-tool`](https://github.com/code-sena/test-manager-library-3145555-tool) | Utilities: demonstration data seeder |
