

## Code Review Workflow

For non-trivial changes (> ~100 LOC, new modules, critical paths like auth / persistence / streaming / billing, or cross-repo contract changes), use the PR + `/ultrareview` workflow:

```
1.  git checkout -b feat/<topic>          # (or fix/, chore/, refactor/)
2.  Implement + tests + lint/format
3.  Commit + push -u origin feat/<topic>
4.  gh pr create --base main              # body: motivation + changes + test plan
5.  Wait for CI green
6.  User runs /ultrareview <PR#> in Claude Code   # billed to user's Anthropic credits; user-only
7.  Read findings via task-notification           # not via PR comment in current setup
8.  Apply fixes on the SAME branch                # additive commits, no amend
9.  Verify CI re-passes green
10. gh pr merge --squash <PR#>                    # (or --merge per repo convention)
11. Bump submodule pointer in deposium_fullstack parent in a SEPARATE commit
```

**Direct-push to `main` acceptable ONLY for:** one-line CI/docs fixes, hotfixes where speed beats review (signal in the commit message).

**Pitfalls:**

- `/ultrareview` is a **Claude Code slash command**, not a shell command. Only the user can launch it from an interactive Claude Code prompt — the agent cannot trigger it.
- Findings arrive via task-notification in the Claude Code session, not as a PR comment in the current setup.
- Do NOT bundle pre-existing lint/clippy warnings into a feature PR — open a dedicated `chore(lint):` PR instead so the diff stays reviewable.
- Submodule bump in `deposium_fullstack` parent is **always a separate commit**, never bundled with the feature PR (the feature lives in its own repo, the bump lives in the parent).

**Rationale:** Direct-push to main on `deposium_rust_worker` for Phase 0 + Phase 1B async-offline (2026-05-24) shipped 6 real bugs into local production state — including a ~96s worker outage on a persist misconfig and a context-drop regression that silently dropped LLM memory after persist abandons. The retro-PRs `/ultrareview` ran post-merge caught them all, but at the cost of a follow-up patch commit. Pre-merge review catches them cheaper.
