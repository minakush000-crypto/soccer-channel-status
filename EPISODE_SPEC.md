# EPISODE_SPEC.md — measurable definition of a finished episode

> **Purpose:** defines a finished soccer tactical-analysis episode in measurable
> terms. Every number traces to a Stage 9A measurement of a real reference
> video (raw data + commands in LANE_PLAN.md §Stage 9). No number that cannot
> be traced to a reference. Production is frozen against this spec until
> approved.
> **Derived:** 2026-09-12, from 6 reference videos (5 tactical/explainer + 1 highlights).
> **Last verified against code:** 2026-09-13 (§9 source-footage provenance added
> and verified this pass — gemini_judge.py exists, the 12B/Sofascore findings are
> real; §1-8, 9B.x trace to LANE_PLAN.md §Stage 9A reference measurements, not
> re-derived this pass).

The project goal (CONTEXT.md) is the Coaches' Voice style: dark pitch, orange
accents, data-driven graphics, functional arrows, deliberate pace. The spec
therefore targets the **deliberate** reference style (Football Meta, Football
Made Simple) for rhythm, and the **graphics-dominant** tactical references
(Football Meta, Football Made Simple, DK FALCON) for content mix. The full
reference range is stated beside each MUST so the choice is auditable.

## Reference videos measured (Stage 9A)

| ID | Channel | Type | Runtime (s) | Shots | Mean shot (s) | Longest (s) | WPM | Graphics % | Footage % | Talking-head % |
|---|---|---|---|---|---|---|---|---|---|---|
| kRLtilxlEj4 | Football Meta | tactical | 698 | 43 | 16.2 | 70.8 | 195 | 72 | 6 | 17 |
| URlf-04YYLk | Football Made Simple | tactical | 530 | 46 | 11.5 | 69.3 | 181 | 78 | 11 | 0 |
| 98BkEsAUr9k | DK FALCON | tactical | 507 | 133 | 3.8 | 20.2 | 193 | 89 | 0 | 6 |
| -MLGcROAr8c | Mega Football | tactical | 741 | 301 | 2.5 | 12.7 | 179 | 6 | 33 | 22 |
| f1xtEHOjrRA | Ball Explained | explainer | 810 | 211 | 3.8 | 14.1 | 166 | 22 | 0 | 0 |
| iaLdVWUrq5Q | AlfonsoR10 | highlights | 906 | 197 | 4.6 | 19.1 | 155 | 0 | 100 | 0 |

Trace: LANE_PLAN.md §Stage 9A. Runtimes/WPM from `yt-dlp --print` + srt
word-count; shots from `scenedetect --downscale 4 detect-content -t 27`;
content ratios from 18 vision-classified frames per video (gemma4:cloud).

## 1. Runtime  (trace: 9A.1)

- **MUST:** 8 to 14 minutes (480 to 840 s).
- Target: 9 to 12 minutes.
- Reference range: tactical/explainer 507 to 810 s (8.5 to 13.5 min); highlights 906 s (15.1 min). The 8 to 14 min MUST covers every tactical reference.

## 2. Shot rhythm  (trace: 9A.1)

- **MUST:** 40 to 80 shots across the runtime.
- **MUST:** mean shot length 8 to 16 s; longest held shot 30 to 70 s.
- Reference basis (deliberate style): Football Meta 43 shots / mean 16.2 s / longest 70.8 s; Football Made Simple 46 shots / mean 11.5 s / longest 69.3 s. These two hold on tactical diagrams, which is the Coaches' Voice pace.
- The four other references cut fast (133 to 301 shots, mean 2.5 to 4.6 s). That is a different sub-format; this spec does **not** target it. A single static board held for the whole runtime (1 shot) is a FAIL.

## 3. Content mix  (trace: 9A.2)

- **MUST:** tactical graphics >= 60 % of runtime.
- **SHOULD:** live footage 5 to 20 %.
- **MUST:** voiceover throughout. Talking-head is deliberately absent — this
  channel is voiceover-only by format decision (10A.1), following Football
  Made Simple at 0 % talking-head. Three references use talking-head (Football
  Meta 17 %, Mega Football 22 %, DK FALCON 6 %); this channel does not
  replicate that segment type.
- Reference basis: Football Meta 72 % graphics / 6 % footage / 17 % talking-head; Football Made Simple 78 % / 11 % / 0 %; DK FALCON 89 % / 0 % / 6 %. 100 % static boards with 0 % footage (lane B as built) is outside every tactical reference except DK FALCON, whose "graphics" are animated 3D pitch renders cut 133 times, not static PNGs.

## 4. Graphic types required  (trace: 9A.3)

- **MUST include:** pitch diagram, formation board, stat card.
- **SHOULD include:** arrows drawn on footage, lower third.
- Graphics appear both on their own screen (dark pitch) and overlaid on moving footage (Football Made Simple frames a highlight circle over a player on live footage at t=3 s).
- Reference basis: pitch_diagram + formation_board + stat_card appear in Football Meta, Football Made Simple, DK FALCON; arrows_on_footage in Football Meta; lower_third in Football Made Simple + AlfonsoR10.

## 5. Typography and colour  (trace: 9A.4)

- **MUST:** bold weight, condensed athletic feel.
- **MUST:** dark background (references use #0d to #1c dark greens/greys and #000000).
- **MUST:** white primary text, 1 to 2 bright accent colours (references: red #c41230, green #4CAF50, yellow #c7c74c).
- **MUST:** large text for titles/numbers, medium for stat labels.
- Reference basis: bold-dominant in 5 of 6 (Football Made Simple 5:1, DK FALCON 4:2, Mega Football 5:0, Ball Explained 8:2, AlfonsoR10 6:2). Football Meta read regular-heavy (8:4) because its frames are dense stat tables; the channel still uses bold headers. Palettes sampled across 10+ frames per video.

## 6. Opening, first 15 seconds  (trace: 9A.5)

- **MUST:** a hook in the first 5 s — live footage of the subject, a striking data visual, or a title over footage.
- **MUST NOT:** open with a static formation board held silently.
- **MUST:** establish the match/topic within 15 s.
- Reference openings (frame by frame, t = 0, 3, 6, 9, 12, 15 s):
  - Football Meta: dark transition -> stat tables (xG league data) — data hook.
  - Football Made Simple: live footage -> highlight circle overlaid on player -> match-result title card -> live footage (Arteta) -> pitch diagram + formation board — footage+graphic mix.
  - DK FALCON: 3D pitch diagram -> passing-lane graphics -> floating tactical boards — animated tactical hook.
  - Mega Football: black -> jersey footage -> text over blurred tunnel -> crowd -> goal celebration — footage hook.
  - Ball Explained: vintage team photo -> archival photo -> map -> title cards — documentary stills hook.
  - AlfonsoR10: tunnel footage -> Ronaldo -> locker room — behind-the-scenes footage hook.
- None opens with a static board. Every opening moves or shows footage.

## 7. Audio  (trace: 9A.6)

- **MUST:** voiceover at 155 to 195 WPM.
- **SHOULD:** low music bed under the voice.
- Highlights lane only: **MUST** retain original crowd audio.
- Reference basis: WPM 155 (AlfonsoR10) to 195 (Football Meta), median 181. All streams AAC stereo ~128 kbps / 44.1 kHz. Music presence was not measurable with available tools (no audio-listening path) and is marked unverified, but the tactical-explainer convention is a subtle bed under voice.

## 8. Framing  (trace: 9A.7)

- **MUST:** 16:9, 1920x1080, full-frame.
- **MUST NOT:** black pillarbox or letterbox bars inside the container.
- Reference basis: all six are 16:9 (1.78 DAR); opening-frame edge-pixel checks are 0 % black on five of six (full-frame). DK FALCON showed black at one opening frame (a dark graphic frame, not a persistent bar). No reference is persistently letterboxed.

## 9. Source-footage provenance and branding  (trace: Stage 14, 2026-09-13)

- **MUST:** before any clip enters the pipeline, extract three frames (early,
  middle, late — e.g. 5 s, 50 % of duration, duration minus 5 s) and judge each
  through Gemini (`tools/gemini_judge.py`, the authoritative judge) for burned-in
  third-party branding: channel logos, reuploader watermarks, betting/sponsor
  overlays, or overlaid text that is not official broadcast graphics.
- **MUST:** reject the clip if third-party branding appears in more than an
  occasional frame. "Occasional" = at most one of the three sampled frames, and
  only incidental (e.g. a passing advertising hoarding caught mid-frame), not a
  persistent burned-in logo or watermark.
- **MUST NOT:** accept a clip whose middle or late frame carries a persistent
  third-party logo or watermark (e.g. a casino sponsor burned into a corner, a
  reuploader's channel bug). These break the channel's own look.
- **SHOULD:** prefer official league/club sources (clean broadcast graphics, no
  third-party watermarks) over reuploads. Official broadcast score bugs and
  league logos are NOT third-party branding for this rule.
- Reference basis: the 12B episode carried an uploader's burned-in "ARSENAL" +
  emoji watermark in the footage (a reupload signal). The Bournemouth-Brentford
  Sofascore-sourced test clip (2026-09-13) carried a MrQ casino watermark
  (top-left) + AFCBTV logo (top-right), Gemini branding-intrusion 5/10 — a
  reject under this rule. The check is Gemini-judged because the main model is
  text-only and cannot read frames directly; every visual claim must paste
  verbatim `gemini_judge.py` output.

---

## 9B.2 — Score the rejected lane B episode against this spec

Episode: `renders/2026-09-12_preview-manc-derby/final_video.mp4`, post-9C-fix
state (pillarbox and fabrication defects already fixed this stage). Measured:
`ffprobe` 1920x1080 / 41.02 s; `scenedetect` **1 shot**, longest 41.0 s; 3
static boards (formation, stat_card, possession), no footage, voice only.

Biggest gap first: **runtime is 41 s, which is 8.5 % of the 480 s MUST
minimum.** The episode is more than 10x too short. That single gap is
disqualifying on its own.

| Spec item | Target | Lane B measured | Score | Verdict |
|---|---|---|---|---|
| 1. Runtime | 480 to 840 s | 41 s | 0/2 | FAIL (8.5 % of min) |
| 2. Shot rhythm | 40 to 80 shots, mean 8 to 16 s | 1 shot, mean 41 s | 0/2 | FAIL (one static hold) |
| 3. Content mix | graphics >= 60 %, footage 5 to 20 % | 100 % static boards, 0 % footage | 1/2 | graphics OK, footage FAIL |
| 4. Graphic types | pitch + formation + stat (MUST) | has formation + stat + possession; no arrows_on_footage, no lower_third | 1/2 | partial |
| 5. Typography | bold, dark, white + accents | matplotlib defaults; Opus rated formation 3/10, possession 4/10, stat 5/10 | 1/2 | below quality |
| 6. Opening | hook in 5 s, no static board | opens on static formation board | 0/2 | FAIL (exactly the forbidden open) |
| 7. Audio | voice 155 to 195 WPM, music bed | voice ~200 WPM, no music | 1/2 | voice present, no bed, WPM high |
| 8. Framing | 1920x1080 full-frame | 1920x1080, 0 % black edges (post-fix) | 2/2 | PASS (after 9C.2) |
| **Total** | | | **6/16 (3.75/10)** | |

Before the 9C fixes this was 2/10 (pillarbox + fabricated zeros). The fixes
removed two real defects but did not move the format: a 41 s boards-only video
is not a tactical-analysis episode by any reference. See 9D.1.

## 9B.3 — Score the two previously uploaded private videos

Episode A: `renders/2026-09-06_arsenal-chelsea/final_video.mp4` (words-match
build, uploaded as WFi2LBwXINU). Measured: `ffprobe` 1280x720 / 45.20 s;
`scenedetect` 14 shots, mean 3.23 s, longest 8.04 s. Has footage + boards +
voice (scorer captions on goals).

| Spec item | Episode A measured | Score |
|---|---|---|
| 1. Runtime | 45 s | 0/2 FAIL (9 % of min) |
| 2. Shot rhythm | 14 shots, mean 3.23 s (fast-style range) | 1/2 (matches fast references but too short to judge) |
| 3. Content mix | footage + boards + captions | 1/2 (has footage; mix unquantified) |
| 4. Graphic types | boards + on-screen scorer captions | 1/2 |
| 5. Typography | white text on footage | 1/2 |
| 6. Opening | footage-led | 1/2 |
| 7. Audio | voice, ~? WPM, no measured music | 1/2 |
| 8. Framing | 1280x720, not 1920x1080 | 0/2 FAIL (wrong resolution) |
| **Total** | | **6/16 (3.75/10)** |

Episode B (shorts crop of the same): `.../shorts/final_video_shorts.mp4`,
720x1280 / 65.2 s. This is a vertical Shorts reframe, outside the 16:9 spec
entirely; it is a different product and is not scored against this 16:9 spec.

Both private uploads share the same disqualifying gap as lane B: **runtime
(~45 s) is under 10 % of the MUST minimum.** Episode A additionally fails the
1920x1080 framing MUST. The shot rhythm of Episode A (14 cuts in 45 s) is the
one dimension that lands inside a reference range (the fast-style references).

## 9B.4 — What the pipeline can hit, must build, and cannot do

**Already hittable with current tools:**
- Framing 1920x1080 full-frame, no pillarbox — fixed this stage (9C.2); boards now normalize to 1920x1080.
- Boards that refuse to fabricate — fixed this stage (9C.1); unavailable stats are omitted, previews hide the scoreline.
- Dark pitch + bold white text + accent colours — `tactical_boards.py` palette.
- Voiceover at target WPM — ElevenLabs; WPM is a script-length knob.

**Needs building:**
- A script long enough for 8 to 14 min (current template yields ~150 words / ~41 s; need ~1300 to 1900 words). This is the single biggest build item and it is a text task, not a tooling task.
- A cut list that produces 40 to 80 cuts with content-matched footage windows (the existing `cut_list_gen.py` is not folded into a boards+footage assembler; lane B has no wired assembler at all — 12 manual steps).
- Tactical graphics overlaid on footage (arrows on footage, highlight circles) — `tactical_render.py` draws on a top-down pitch, not on broadcast footage; overlay-on-footage does not exist.
- Lower thirds, animated transitions, a music bed under voice, and an opening-hook structure (first 5 s footage/data, not a static board).
- Footage acquisition at scale (see 9D.2).

**Format decisions (10A.1 — deliberate, not gaps):**
- Voiceover-only, no talking-head. Following Football Made Simple (0 % talking-head). Three references use talking-head (Football Meta 17 %, Mega Football 22 %, DK FALCON 6 %); this channel does not replicate that segment type. There is no presenter, camera, or studio, and none is planned.

**Promoted from impossible to build targets (10A.2, 10A.3):**
- 3D tactical renders (DK FALCON's floating 3D pitch boards) — now a build target, scoped in 10C / LANE_PLAN.md §Stage 10. The 2D `tactical_render.py` stays until a 3D replacement is proven at/above spec (10C.3 first-version rule; 10C.5 second-version rule holds deletion while no replacement can run).
- Broadcast-footage acquisition — the obstacle was rule 1's old wording, not YouTube. Residential download works (proven Stage 9A on 6 references). Rule 1 amended (10A.3): residential → /mnt/f → pod → delete locally. Remaining constraint: /mnt/f must be mounted (USB present).

**Impossible with the tools available:**
- None after the 10A amendments. The three items previously listed (talking-head, 3D renders, footage) are respectively a format choice, a build target, and a solved acquisition path.