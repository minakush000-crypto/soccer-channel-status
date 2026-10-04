# Brief 15 job 0: marketing kit removal record (2026-09-29)

## Provenance
- Kit identity: 60 skills whose SKILL.md carries the kit's documented
  product-marketing dependency marker (49 machine-grepped) plus 11 same-family
  marketing skills read individually (content-matrix, analytics-dashboard,
  post-writer, post-scorer, from a LinkedIn sub-pack; banner-design, brandkit,
  pinned-comment, newsletter-voice, reels-scripting, quote-post; launch).
- Remote source: text match on
  coreyhaines31/marketingskills (github) skills/ads/SKILL.md: same
  trigger/description verbatim; ours v2.2.0, theirs v2.3.2 (ours is an older
  snapshot of the same family). Installed as plain dirs under
  ~/.claude/skills at 2026-08-26 00:07 CDT; NO lock entry (skills-lock.json
  lists only unlazy), NOT an ECC component, NOT a plugin install
  (installed_plugins.json has only clauderem). NO local source clone
  (home-wide grep for the skill's distinctive body line found no second copy).
  Not in the two installed marketplaces' trees.

  Correction while writing: installed_plugins.json has only claude-mem
  (typo above corrected here).
- Use check: grep of both repos (py/sh/js/mjs/yaml), hooks and rules: every
  hit is a plain-word prose collision, not a reference to the skill ("too
  many ads" inside an LLM prompt; "attribution"/"paywalled" in a comment;
  "video" in a topics yaml). No pipeline invokes any kit skill. Zero code
  callers; no MCP tools.

## Backup-first removal (brief-14 method)
- Upload: b2:mend60929_marketing_kit_removed.tar.gz in the backups/ prefix —
  full object name: backups/2026-09-29_marketing_kit_removed.tar.gz
  (tar of the 60 dirs, streamed with rclone rcat).
  md5 upload-stream: 933da22419f7bb78d5118ed8a4f6a693
  md5 B2 download:  933da22419f7bb78d5118ed8a4f6a693 (match)
  tar listing: 413 entries.
- Removal: dirs deleted; skills 253 -> 193 (counted with ls | wc -l).
- ecc.js doctor after removal: checked=1, ok=1, warnings=0, errors=0.
  (ECC never managed the kit: doctor is unaffected; run to prove it.)

## Session-start tokens (brief-07 method, identical command both sides)
  before: first_turn_input=51175, num_turns=1, subtype=success
  after:  first_turn_input=51143, num_turns=1, subtype=success, init tools=41
  measured delta: -32 tokens (-0.06%).
  Honest caveat: the headless first turn evidently does not carry most
  skill descriptions; the 10.4k skills figure brief 12 measured came from
  /context inside a real interactive session. The brief-07 method cannot
  see the kit's share. Recorded as measured, no estimate substituted.

## Kept deliberately (owner classes inside the former "kit" of 127)
- obra/superpowers-family engineering skills (systematic-debugging,
  test-driven-development, verification-before-completion, brainstorming,
  writing-plans, executing-plans, using-git worktrees, dispatching/
  requesting/receiving-code-review, subagent-driven-development...)
- Anthropic official document/creative skills (docx pdf pptx xlsx,
  canvas-design, algorithmic-art, brand-guidelines, frontend-design,
  web-artifacts-builder, imagegen-*, slack-gif-creator, youtube-thumbnail)
- This repo's own skills (market-research, strategy-extraction,
  platform-review, knowledge-synthesis: yt-digest CLAUDE.md section 4)
- design/UI skills (ui-ux-pro-max, taste-skill-v1, minimalist-skill,
  brutalist-skill, stitch-skill, gpt-tasteskill, ui-styling)
- product-lens, growth-log: product/learning skills, not marketing

Both repos' DECISIONS.md record the removal (see commits below).