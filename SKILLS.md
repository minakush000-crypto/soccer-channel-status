# SKILLS.md — what installed skills are, where they live, why most never fire

> **Purpose:** one-time analysis of installed skills and why most never fire (2026-09-05). A completed-stage working artifact.
> **Reader:** historical reference; not read by any pipeline tool, not mirrored.
> **Last verified against code:** 2026-09-09.

Built 2026-09-05. Evidence is pasted from commands run today.

## Where skills live

```
$ ls -d ~/.claude/skills/*/ | wc -l
127
$ ls -d ../.claude/skills/*/ | wc -l      # = ~/yt-digest/.claude/skills/
125
$ ls -d .claude/skills/*/ | wc -l         # project-level
0
$ ls -la .claude/
no .claude in project
```

Two real skill trees, no project-level skills:
- `~/.claude/skills/` — global, 127 directories (user-installed).
- `~/yt-digest/.claude/skills/` — parent repo, 125 directories.
- `/home/muads/yt-digest/soccer-channel/.claude/skills/` — does not exist.

The global and parent trees are near-duplicates (same names, same
frontmatter, checked on `video`, `voice-builder`, `unlazy`). `unlazy` exists
only in global, not parent. The two `soccer-channel` SKILL.md files are
slightly different sizes (global 40-dir, parent 28925-byte SKILL.md) but both
say DEPRECATED.

## What they are

Most are marketing/content skills (ad-creative, ads, analytics, brand,
copywriting, cold-email, cro, seo-audit, social, reels-scriptging, etc.).
A few are engineering-adjacent: `unlazy` (completion gates), `claude-api`
(reference), `video` (AI video generation), `voice-builder` (writing voice),
`taste-skill`, `frontend-design`, `dataviz`, `artifact-design`,
`dispatching-parallel-agents`, `executing-plans`, `writing-plans`,
`systematic-debugging`, `verification-before-completion`,
`finishing-a-development-branch`, `using-git-worktrees`,
`subagent-driven-development`, `test-driven-development`.

Each skill is a directory with a `SKILL.md` whose YAML frontmatter has a
`name` and a `description`. Example (`~/.claude/skills/video/SKILL.md`):

```
---
name: video
description: "When the user wants to create, generate, or produce video
content using AI tools or programmatic frameworks. Also use when the user
mentions 'video production,' 'AI video,' 'Remotion,' ..."
---
```

## What triggers a skill

Two mechanisms, and this is the actual mechanism, not a guess:

1. **The model invokes the Skill tool.** The harness loads every SKILL.md it
   finds at startup and puts a one-line description for each into a
   system-reminder listing (visible in this session's system prompt under
   "The following skills are available"). The model reads that listing and
   decides whether one matches the task. The `Skill` tool description says:
   "When the task at hand is one a listed skill covers, call this tool first."
   So firing is **model judgment**, not a rule engine. The description's
   trigger phrases are the only signal the model has.

2. **The user types `/<skill-name>`.** The `Skill` tool description says:
   "Users may also ask for one by name (`/<name>`...); that's a request to
   invoke it." This is the only deterministic trigger.

The CLAUDE.md "autonomous skill activation" instruction ("at the start of
every task, silently check if any installed skill matches") is an instruction
TO THE MODEL, not a harness feature. It tells the model to do step 1
proactively. If the model doesn't think a skill matches, it doesn't fire. No
code forces it.

## Are any relevant to this project

Relevant or partially relevant:
- `unlazy` — completion gates. Already wired into the global CLAUDE.md and
  used on multi-step tasks here. Relevant.
- `soccer-channel` — DEPRECATED, and actively misleading. Its frontmatter
  says "Do NOT use this skill for new work" and points at `~/soccer-pipeline/`
  which is retired (DECISIONS.md). It would fire on "make a soccer video" but
  it would send work to the wrong project. **Do not invoke it.**
- `video` — AI video generation (Remotion, HeyGen, Sora, etc.). Not relevant:
  this project uses ffmpeg + YOLOv8, not generative video.
- `claude-api` — only relevant if calling the Anthropic API. Not used here.
- `dataviz` — for charts/dashboards. Not relevant to tactical boards.

The marketing/content majority are not relevant to a video pipeline codebase.

## Why an installed skill would not fire

The actual reasons, in order of likelihood for this project:

1. **No description match.** The skill's `description` trigger phrases don't
   overlap the task wording. 100+ marketing skills have no trigger phrase a
   code-editing task will hit. This is why most never fire — they're for
   marketing work, not engineering.

2. **The model judges it irrelevant.** Even when a description could match
   ("video production"), the model reads the actual task (edit
   `tactical_overlay.py`) and skips the skill because the skill is about
   generative AI video tools, not ffmpeg code. Firing is model judgment, so a
   reasonable non-match stays a non-match.

3. **The skill is DEPRECATED in its own frontmatter.** `soccer-channel` says
   "Do NOT use this skill for new work." The model honors that and won't
   invoke it.

4. **No project-level skills.** `.claude/skills/` does not exist in the
   project (ls confirmed). So nothing project-specific can fire; only global
   and parent skills can, and none of those describe this project's actual
   pipeline.

5. **Wrong cwd for path-scoped skills.** The `Skill` tool notes
   directory-scoped skills are listed with a path prefix and "most specific
   wins." With no project-level skills, only global/parent skills compete,
   and none are scoped to this directory.

## Concrete consequence for this project

The one skill that SHOULD govern this project (`soccer-channel`) is
DEPRECATED and points at a retired folder. So today, no installed skill
correctly encodes this project's pipeline. The CLAUDE.md files (global,
parent, project) and CONTEXT.md are doing the job skills would do. If we
want a skill to fire on "make a soccer video," we would need to either
un-deprecate and rewrite the soccer-channel SKILL.md to point here, or create
a project-level `.claude/skills/soccer-channel/SKILL.md` that overrides the
global one. Neither is done today.