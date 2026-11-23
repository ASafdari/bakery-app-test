# Status

<!-- Injected into every session (and after compaction) by the SessionStart hook.
     Keep it under ~40 lines. Claude updates it when phase, decisions or next step
     change. -->

- **Phase:** spec
- **State:** in progress
- **Feature:** Bakery pre-orders · spec `docs/spec.md` · design `docs/design.md` (not written)
- **Approvals:** Spec approved 2026-11-23 · UI pending · Design pending · Test report pending
- **Slice PRs:** none
- **Next step:** `/ui-design` for the customer order page, counter view and admin pages.

## Open decisions
- None. All spec questions resolved 2026-11-23.

## Notes that must survive
- Repo config is bootstrapped without commands. Re-run `/repo-setup` once the first
  slice builds (adds allow rules, Stop hook, CI, branch protection).
- Spec work is on branch `docs/preorders-spec` (off `main` at 854186a), uncommitted.
