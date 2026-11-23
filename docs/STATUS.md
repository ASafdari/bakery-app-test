# Status

<!-- Injected into every session (and after compaction) by the SessionStart hook.
     Keep it under ~40 lines. Claude updates it when phase, decisions or next step
     change. -->

- **Phase:** idea
- **State:** in progress
- **Feature:** TBD · spec `docs/spec.md` (not written) · design `docs/design.md` (not written)
- **Approvals:** Spec pending · UI pending · Design pending · Test report pending
- **Slice PRs:** none
- **Next step:** Draft `docs/spec.md` for the first feature.

## Open decisions
- What the bakery app does and who uses it (owner: repo owner, blocks: spec)
- Stack: which of the usual stacks this repo uses (owner: repo owner, blocks: design)

## Notes that must survive
- Repo config is bootstrapped without commands. Re-run `/repo-setup` once the first
  slice builds (adds allow rules, Stop hook, CI, branch protection).
