# Git Workflow

MOO uses GitFlow and small Conventional Commits.

## Branches

- `main`: stable and releasable history.
- `develop`: integration branch for completed work.
- `feature/<short-name>`: new behavior or documentation work.
- `fix/<short-name>`: non-release corrective work.
- `release/<version>`: release stabilization.
- `hotfix/<short-name>`: urgent fixes based on `main`.

Create branches for one focused outcome. Keep them short-lived and merge them into the appropriate GitFlow target after review and verification.

## Commits

Each commit should represent one coherent, reviewable step. Prefer several understandable commits over a large mechanical dump, but do not split changes so finely that intermediate commits are misleading or broken.

Use short Conventional Commit subjects:

```text
<type>(optional-scope): <imperative summary>
```

Common types:

- `feat`: user-visible capability.
- `fix`: defect correction.
- `docs`: documentation only.
- `test`: tests only.
- `refactor`: behavior-preserving code change.
- `chore`: tooling, dependencies, or repository maintenance.
- `ci`: continuous-integration changes.
- `build`: build-system or packaging changes.

Examples:

```text
docs: define mvp requirements
chore: initialize typescript tooling
feat(discovery): list stdio tools
test(catalog): cover name collisions
fix(invocation): reject invalid arguments
```

Subjects should be lowercase, imperative, concise, and have no trailing period. Use the body for reasoning, trade-offs, compatibility notes, or issue references.

## Agent behavior

- Create local commits at coherent milestones during authorized implementation work.
- Before committing, review the staged change and run relevant checks.
- Use the exact short Conventional Commit message that describes the change.
- Do not mix unrelated cleanup into feature commits.
- Never push, publish, add a remote, or open a pull request.
- Do not rewrite history, squash, rebase, merge, or delete branches without explicit authorization.
- Preserve changes that predate the current task.

All repository history remains local unless the user explicitly changes this policy in the future.
