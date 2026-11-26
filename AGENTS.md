# AGENTS.md

Guidance for coding agents working in this repository. This file is the
rules-and-pointers index. Keep it under about 300 lines: add one linked line per
new rule and put the long form in a linked doc. `CLAUDE.md` imports this file;
do not duplicate rules there.

## Working agreements (must follow)

1. **Use PMT for durable work.** Mera PMT is the canonical durable plan system.
   Use the `pmt-century-invoice` connector (`pmt_century_invoice` in the
   repo `.mcp.json`; token from `PMT_MCP_TOKEN_CENTURY_INVOICE`) and call
   `list_projects` first in each PMT task; continue only when it resolves to
   this project, and never write to another project's connector. Search before creating
   plans, follow the human-governed lifecycle (New → Thinking → WIP → Done),
   claim only approved WIP work, and record decisions and evidence there.
   Status-change requests never approve themselves. If PMT is unavailable, use
   `docs/plans/` as an offline fallback marked `Mera sync pending`.
2. **Align behavior before code.** For a non-trivial behavior, schema,
   authentication, deployment, or product-UI change, state `Now / We Want /
   Gap` and the main trade-off in 2–3 sentences, then wait for sign-off. Do not
   ask again once the owner has made the direction explicit. An approved
   prototype or reference is an acceptance contract, not inspiration.
3. **Investigate bugs before patching.** Trace the failing workflow and report
   `Wrong value / First source / Boundary that failed / Why it was allowed /
   Minimal fix boundary`. Product invariants beat accidental current behavior.
   A request to diagnose does not authorize a fix.
4. **Test proportionately before commit or push.** Run `git diff --check` and
   the repository's static checks and focused tests for every touched boundary
   (see [Verification matrix](#verification-matrix)). Never push a locally red
   result, and never call a local-green result CI-green. After a push, verify the
   exact SHA through the required CI checks before handing back.
5. **Keep handwritten files small.** Aim for at most about **500 lines** per
   new handwritten file, and split one before it passes about **600 lines**.
   Generated files, lockfiles, migrations and vendored assets are exempt.
6. **Verify dependencies from the registry.** Check the registry (`npm view`,
   PyPI, etc.) and official docs before changing a manifest or lockfile. Never
   choose a version from memory. Update lockfiles with the repository's package
   manager.
7. **Keep contracts and docs current.** API contracts, runtime configuration,
   persistence semantics and deployment changes update their source-of-truth
   docs in the same change. Plans are working context, not proof of runtime
   behavior.
8. **Resolve the environment before operations.** Ask `local, UAT, or
   production?` before running deployments, seeds, restores, release builds, or
   anything that can affect external state. Deploy only on the owner's explicit
   instruction.
9. **Ground recommendations and answer directly.** Answer in the first line.
   Name the established pattern, standard or comparable product behind a
   recommendation, give your verdict and why it fits, and mark preferences as
   opinions. Do not stop at "it depends".
10. **Preserve worktree and remote safety.** Keep commits scoped, stage explicit
    paths, preserve unrelated changes, and use isolated worktrees for concurrent
    work. When the owner asks to push without naming a remote, push the approved
    branch to both `origin` (GitHub) and `gitea` after local gates, and report
    each result.
11. **Clean up after tools and finished work.** Stop dev servers, browser
    sessions and background commands you started. Delete tool output nobody
    needs (`.playwright-mcp/`, ad-hoc screenshots, scratch files). When a task is
    done, remove your worktrees, merged branches and temporary test data. If you
    are unsure whether something is yours or still needed, ask first.
12. **Give the whole picture before asking for a decision.** Describe the flow
    as the user sees it (what they do, what the screen shows, what is saved),
    then say where the open point sits, and only then ask. Avoid table, column
    and function names unless asked. When asked for the logic, give rules, not
    examples.
13. **Protect client and financial data.** Invoices, customer records, bank
    details and tax identifiers are sensitive. Never commit real client data,
    credentials or exported invoices; use synthetic fixtures. Never print
    secrets, cookies or session files.

## Fixes must fit the full system

Before choosing a fix, assess the intended user outcome, the end-to-end
workflow, the authoritative state, and the relevant constraints. Trace the root
cause across boundaries and check related callers for the same failure. Fix at
the layer that owns the broken invariant. A narrow patch is enough only when the
evidence shows the defect is local. If immediate containment is needed, label it
temporary and state the proper fix that remains. Validate the original failure
and the adjacent paths, and state any remaining gaps before declaring the fix
complete.

## Problem-solving principles

- **No blame-shifting.** Do not blame a model, library or API without evidence;
  find out what actually changed.
- **Generic solutions only.** Do not hardcode specific examples or values into
  shared code.
- **Step up a layer after repeated failure.** After two or three failed
  attempts at the same layer, stop and propose that the layer may be wrong.
- **Treat pushback as a steer.** When the owner challenges a premise, verify
  it before defending the answer.

## Project at a glance

Century Invoice is an invoicing application for AMEO. The stack and layout are
not chosen yet. When the first scaffold lands, replace this section with the
stack, the directory map, and the product invariants, for example:

```text
apps/web/    (frontend)
apps/api/    (backend / API)
db/          (migrations and seed data)
tests/       (unit, integration, e2e)
deploy/      (deployment bundle)
docs/        (contracts, architecture, runbooks)
```

### Invoicing invariants (draft, confirm with the owner)

- The database is the source of truth for customers, invoices, line items,
  payments and audit events. The browser never reports a browser-only mutation
  as success.
- Money is stored as integer minor units (or a fixed-precision decimal) with an
  explicit currency, never as a binary float. Rounding and tax rules live in
  one shared module.
- Issued invoices are immutable. Corrections create credit notes or successor
  versions with linked audit evidence. Invoice numbers are sequential and
  never reused.
- UI visibility is not authorization. The API enforces every protected read and
  write.

## Source-of-truth documents

- Product boundary and commands: [README.md](README.md) (to be written)
- Detailed working agreements: `docs/working-agreements.md` (to be written)

When documents conflict, explicit current owner direction and verified product
invariants come first, then code, data and tests, then contracts and runbooks,
then PMT working context.

## Verification matrix

Minimum baseline until the stack defines its own:

```sh
git diff --check
```

Add the stack's check, test, build and e2e commands here when they exist.

## Change and push discipline

- Inspect `git status` before editing, staging, committing, rebasing or
  switching branches.
- Re-run the affected verification after the final edit, not only before it.
- Commit messages describe the product outcome, not the plan number.
- Before pushing, confirm the branch, the upstream, the intended remote(s) and
  the local verification result.
- A push is not a deploy. Report the commit, the remote branch, the CI state if
  observed, and any uncommitted files left in the worktree.

## Remotes

```text
origin  https://github.com/AMEO-USA/century-invoice.git
gitea   https://gitea.lab.pltc.me/paulleungtc/century-invoice.git
```

Inspect Gitea CI with `tea actions runs list|view|logs` and pass
`--repo paulleungtc/century-invoice --login gitea.lab.pltc.me`.
