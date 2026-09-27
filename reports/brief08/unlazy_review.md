# unlazy_review.md — every unlazy copy, version, drift, hook, tests (brief 08 job 5)

All commands run 2026-09-27 (raw output below, untrimmed except where a
command's own output is quoted in full).

## The copies

1. `/home/muads/unlazy` — a git clone of github.com/Leonxlnx/unlazy, HEAD
   `473d4b8` ("fix: flush lint reports before exit"), working tree clean
   (`git status --porcelain` → empty), package.json version **2.1.0**.
2. `/home/muads/.agents/skills/unlazy` — the deployed skill copy, NOT a git
   repo, package.json version **2.1.0**. `diff -rq` against
   `/home/muads/unlazy` (excluding .git) → **no differences** (identical).
3. `/home/muads/.claude/skills/unlazy` — a SYMLINK to
   `../../.agents/skills/unlazy` (created 2026-08-29). Not a third copy.

## Relationship to upstream

Upstream cloned to /tmp/unlazy_upstream: HEAD `1667149` ("fix: bind gate
evidence and harden Windows identity") — exactly the commit the brief named.
`git merge-base 473d4b8 1667149` → `473d4b8`, i.e. the deployed copy is an
ANCESTOR of upstream HEAD: upstream is **exactly 1 commit ahead**
(`473d4b8..1667149` = 1 commit). The deployed copy is not locally modified —
it is simply one upstream commit behind. `diff -rq` of the deployed copy vs
upstream shows 13 files differing (SKILL.md, gate-check.mjs, stop-hook.mjs,
lib/gates.mjs, install-hooks.mjs, 3 tests, 2 references/templates, CHANGELOG,
README, SECURITY, .github/workflows/test.yml) — all of that difference is
the one upstream commit, none of it is local editing.

## Hook

The unlazy Stop hook IS installed, in the GLOBAL settings file
(~/.claude/settings.json), and it runs the REPO copy, not the deployed copy:

    Stop -> '/home/muads/.nvm/versions/node/v24.18.0/bin/node'
            '/home/muads/unlazy/scripts/stop-hook.mjs' --unlazy-hook-v2

No unlazy entry exists in ~/yt-digest/.claude/settings.json,
soccer-channel/.claude/settings.json, or
soccer-channel/.claude/settings.local.json (grep → nothing). Fragility to
watch: the hook executes /home/muads/unlazy/scripts/stop-hook.mjs while the
skill runs from ~/.agents/skills/unlazy — two copies of the same code that
are identical today but can drift independently.

## Approvals and ignore status

    ls ~/.unlazy/approved | wc -l  →  135
    git check-ignore -v .unlazy .claude/settings.local.json
      /home/muads/.config/git/ignore:2:.unlazy/                  .unlazy
      /home/muads/.config/git/ignore:1:**/.claude/settings.local.json
                                                          .claude/settings.local.json

Both are ignored by the GLOBAL git ignore (~/.config/git/ignore), so the
approval cache and local settings stay untracked in every repo.

## Tests

`npm test` in /home/muads/.agents/skills/unlazy → exit 0:
contract tests 8/8 passed, self-check 15/15 ok (full tail below).

## Summary table

| Path | Commit / version | Modified? | Hook? | Tests pass? |
|---|---|---|---|---|
| /home/muads/unlazy | 473d4b8, v2.1.0 (clean clone) | No (working tree clean); 1 upstream commit behind (1667149) | YES — the global Stop hook runs THIS copy's scripts/stop-hook.mjs | yes (run here, identical code) |
| /home/muads/.agents/skills/unlazy | v2.1.0, not a git repo; identical to /home/muads/unlazy | No (diff -rq vs the clone: empty) | no (the skill itself; symlinked from ~/.claude/skills/unlazy) | yes (npm test exit 0: 8/8 + 15/15) |
| ~/.claude/skills/unlazy | symlink → ../../.agents/skills/unlazy | n/a (same files) | n/a | n/a |

Recommendation (not done in this brief — no job asks for it): update both
copies to upstream 1667149 in one step (clone is 1 commit behind, deployed
copy mirrors it), keeping hook + skill on the same commit.

## Raw output

### SKILL.md files under */skills/unlazy*

    $ find /home/muads -path "*/skills/unlazy/SKILL.md" | grep -v node_modules
    /home/muads/.agents/skills/unlazy/SKILL.md
    (only one real file; ~/.claude/skills/unlazy is a symlink to this path;
    /home/muads/unlazy/SKILL.md is the same file in the clone)

### Versions

    /home/muads/unlazy: package.json version 2.1.0, git HEAD 473d4b8
      ("fix: flush lint reports before exit")
    /home/muads/.agents/skills/unlazy: package.json version 2.1.0, no .git

### diff -rq: /home/muads/unlazy vs /tmp/unlazy_upstream (upstream 1667149)

Only .git internals differ (FETCH_HEAD, config, index, logs, loose objects
vs the upstream pack) — the working trees are identical.

### diff -rq: /home/muads/.agents/skills/unlazy vs /tmp/unlazy_upstream

    Files .github/workflows/test.yml differ
    Files CHANGELOG.md differ
    Files README.md differ
    Files SECURITY.md differ
    Files SKILL.md differ
    Files references/gates.md differ
    Files scripts/gate-check.mjs differ
    Files scripts/install-hooks.mjs differ
    Files scripts/lib/gates.mjs differ
    Files scripts/stop-hook.mjs differ
    Files templates/gates-node.md differ
    Files tests/hardening-tests.mjs differ
    Files tests/run-tests.mjs differ
    Files tests/stress-tests.mjs differ
    (all 13 = the single upstream commit 1667149; see
     `git -C /home/muads/unlazy log --oneline 473d4b8..1667149` → exactly 1)

### unlazy entries in settings files

    ~/.claude/settings.json:16:
        "command": "'/home/muads/.nvm/versions/node/v24.18.0/bin/node'
                    '/home/muads/unlazy/scripts/stop-hook.mjs' --unlazy-hook-v2"
    (Stop hook; the only unlazy entry in any settings file)

### Approvals

    $ ls ~/.unlazy/approved | wc -l
    135

### check-ignore

    .unlazy/                  ← ~/.config/git/ignore:2
    .claude/settings.local.json ← ~/.config/git/ignore:1

### npm test (tail 30)

    ok   contract: an unowned B is reported
    ok   contract: missing, stale, or wrong observations are reported
    ok   contract: explicit removal is distinct from abandonment or deferment
    ok   contract: amendment C invalidates the prior revision until reconciled
    ok   contract: acceptance constraints count while optional ideas do not
    ok   contract: a table cannot pass without the current request denominator

    8/8 passed
    ok   zero non-stdlib imports
    ok   one shared gate parser
    ok   no index-arithmetic argument filtering
    ok   gate files are written atomically
    ok   checks wait for close and cap output
    ok   approval identity binds execution semantics
    ok   the hook resolves a scope rather than globbing the tree
    ok   every local resource the skill names exists
    ok   all executable sources retain the Node 16 floor
    ok   the PLAN template carries a revisioned contract denominator
    ok   the PLAN dispatch table is the single operational authority
    ok   the PLAN structural check rejects contradictory representations
    ok   leaf release precedes dependent promotion everywhere
    ok   request reconciliation keeps the focused solo cheap path
    ok   abandonment is terminal handoff rather than ALL MET

    self-check ok (15/15)
    (exit 0)