# Brief 09 — overlay catalog (job 5)

15 distinct overlay styles across the three benchmark videos. Every entry
lists example frames (copied to `examples/`, 640px, unstamped originals from
the frame pool), what it shows tactically, how it animates (evidence:
consecutive frames_<L>.csv rows and the pod scene-cut list), its colors, and
whether it is drawn on footage or on a diagram.

Classification policy used throughout (applied consistently in
frames_<A|B|C>.csv): a diagram's BASE furniture (the pitch, the player dots,
the permanent name labels under dots, dashed alignment lines) is not counted
as overlay elements; ADDED marks (arrows, rings, zones, number tags, chips,
connection lines, text callouts) are. Example: B's plain white-card board at
t=120 has elements=[] but the same board with yellow lane lines at t=180 has
`line between players`.

---

## 1. Yellow ground ellipse (player highlight)
- Examples: `examples/yellow-ellipse-highlight_A_t30.jpg`, `..._A_t94.jpg`, `..._A_t120.jpg`
- Shows tactically: which player the narration is talking about right now
  (receiver of the next pass, the winger whose heatmap is discussed).
- Animation: the ellipse sits under one player and jumps to the next player
  at the relevant moment (A t=120 single ellipse -> t=122 ellipse + pass line
  -> t=124 two ellipses joined by a line); it does not smooth-track, it cuts
  with the shot. CSV rows A t=120/122/124 show the progression.
- Colors: yellow (saturated, semi-opaque disc).
- Drawn on: footage.

## 2. Position label chip + pointer
- Examples: `examples/position-label-chip_A_t80.jpg`, `..._A_t88.jpg`, `..._A_t174.jpg`
- Shows tactically: the role of the player in the mechanism (CB, LW, FB) at
  the moment the narration names the role.
- Animation: chip + small pointer marker appear next to the player's head,
  hold for the length of the point, vanish at the cut (CSV A t=80/82 hold,
  gone by t=84's board). White chip text, white or teal pointer.
- Colors: white text, teal or white pointer.
- Drawn on: footage (also over 3D renders, A t=6).

## 3. Glowing connection network on footage
- Examples: `examples/glowing-web-on-footage_A_t36.jpg`, `..._A_t32.jpg`, `..._A_t320.jpg`
- Shows tactically: the passing/pressing web the narration describes — which
  players are connected, which zone is occupied (orange polygon web at t=36,
  green zone + red dashed line at t=32, yellow triangle link at t=320).
- Animation: the web fades in over roughly a second while the clip plays
  (t=282 shows a single white pass streak mid-draw), then dies with the shot.
- Colors: orange, green, yellow, red, white (glow/blur strokes).
- Drawn on: footage.

## 4. Pressing unit overlay (red + yellow)
- Examples: `examples/pressing-unit-on-footage_A_t444.jpg`, `..._A_t448.jpg`, `..._B_t292.jpg`
- Shows tactically: the out-of-possession block — who presses, the trap
  shape, the passing lane being cut. A draws red glow ellipses under the
  pressing group plus a yellow polygon connecting them; B draws red rings on
  pressed players, shaded red triangles between them, red run arrows and name
  tags, all at once (B t=292 is the fullest example: arrow + ring + line +
  label + zone).
- Animation: markers appear player by player across 2s steps (A t=444 -> 446
  -> 448 add the polygon, then yellow beams at t=450).
- Colors: red, yellow (A); red, white, black, gray (B).
- Drawn on: footage.

## 5. Structure card over footage (B's rounded-card style)
- Examples: `examples/structure-card-on-footage_B_t100.jpg`, `..._B_t108.jpg`, `..._B_t288.jpg`
- Shows tactically: the in-possession or pressing shape ON live footage:
  shaded triangles between a player group, position tags (LB, LCB, CM, CF...),
  red rings around the referenced players. The footage sits inside a rounded
  rectangle card on B's dark blue grid background.
- Animation: tags and rings hold while the footage plays behind; triangles
  re-form between different player groups as the point advances (CSV B
  t=98-110, all `same overlay animating over footage`).
- Colors: white, red, gray, yellow.
- Drawn on: footage (presented inside a card frame).

## 6. Quote / title text over footage
- Examples: `examples/quote-title-text-on-footage_B_t0.jpg`, `..._B_t4.jpg`, `..._B_t16.jpg`
- Shows tactically: the hook (a critic's quote about tiki-taka) and the
  philosophy name (POSITIONAL PLAY) — the thesis setup.
- Animation: the quote BUILDS word by word across 2s steps (B t=0 `"I loathe
  all` -> t=4 longer -> t=12 complete with closing quote mark, CSV notes the
  build), then the big title fades in over a held still.
- Colors: yellow text.
- Drawn on: footage (quote), held still photo (title).

## 7. Animated replay board (B's white card)
- Examples: `examples/animated-replay-board_B_t336.jpg`, `..._B_t344.jpg`, `..._B_t364.jpg`
- Shows tactically: a full match sequence replayed on a clean board — the
  ball dot travels possession by possession, dashed pass options appear, red
  dashed boxes mark midfield matchups (t=344), the caption "VS RAYO
  VALLECANO" / "VS ATH BILBAO" anchors the match.
- Animation: continuous — CSV B t=336-380 shows the ball dot moving step by
  step while labels hold; marks (arrows, boxes, dashed lines) appear and
  clear per phase.
- Colors: red/blue striped dots vs blue dots, red/blue marks, black text on
  white card, dark blue grid background.
- Drawn on: diagram.

## 8. Lanes + numbered count board
- Examples: `examples/lanes-and-count-board_B_t168.jpg`, `..._B_t184.jpg`, `..._B_t164.jpg`
- Shows tactically: the build-up shape's passing lanes (yellow zigzag lines)
  and HOW MANY players are involved in the structure — black number tags 1-6
  appear on players one by one while the narration counts them.
- Animation: the count is the animation: t=180 no numbers -> t=184 numbers
  1-6 placed (CSV shows line+label appearing together).
- Colors: yellow lines, black number chips, white card.
- Drawn on: diagram.

## 9. Concept chips on the board
- Examples: `examples/diagram-concept-chips_B_t122.jpg`, `..._B_t260.jpg`
- Shows tactically: a named moment or concept attached to the diagram — the
  black "Open Touch" chip pinned to Pedri (t=122), gray space zones plus
  arrows showing where space opens (t=260).
- Animation: chip appears for one beat, clears by the next (t=124 plain
  board); zones persist while the explanation runs.
- Colors: black chip/white text; gray zones; red/blue arrows.
- Drawn on: diagram.

## 10. Stats card (torn-paper and crest styles)
- Examples: `examples/stats-card_B_t42.jpg`, `..._B_t408.jpg`, `..._B_t482.jpg`
- Shows tactically: the evidence numbers — goals conceded list with opponent
  crests building line by line (t=40-48), "DUELS WON %" list on a torn-paper
  card (t=408), the league table with P/Pts/GD (t=482), an xG difference
  chart (t=308).
- Animation: values/rows appear one by one (t=42 two rows -> t=48 four rows
  + footnote; the paper card is empty at t=404, populated by t=406).
- Colors: white text, orange values, dark textured or paper background.
- Drawn on: diagram (graphic). NOTE: these numbers carry no source line —
  the creator's claims, not checked facts.

## 11. Name tag + triangle pointer (C's signature)
- Examples: `examples/name-tag-triangle_C_t0.jpg`, `..._C_t20.jpg`, `..._C_t212.jpg`
- Shows tactically: where Messi is in a chaotic clip — white MESSI text chip
  with a red triangle pointing down at him.
- Animation: tag + triangle track the player across frames while he moves
  (CSV C t=0 -> t=2 "label tracks Messi during live play"); reappears for
  each new clip.
- Colors: white text, red triangle.
- Drawn on: footage.

## 12. Big white tracking arrow
- Examples: `examples/tracking-arrow_C_t98.jpg`, `..._C_t120.jpg`, `..._C_t90.jpg`
- Shows tactically: who to watch and where the danger goes — a thick white
  arrow pointing at (and following) Messi, or fanning out to show his options
  (C t=248's four-arrow fan is the maximal version).
- Animation: arrow appears, follows the player for a few seconds, fades out
  (C t=90-92 arrow present -> t=94 "faint translucent arrow" -> gone by t=96).
- Colors: white (semi-opaque).
- Drawn on: footage.

## 13. Spotlight cone
- Examples: `examples/spotlight-cone_C_t104.jpg`, `..._C_t134.jpg`, `..._C_t124.jpg`
- Shows tactically: isolates one player (or his sightline) with a translucent
  white beam from above.
- Animation: beam fades in on the player, holds 2-4s, fades (paired with the
  red "!" marker at t=134-136).
- Colors: white translucent.
- Drawn on: footage.

## 14. Locator ring and glow
- Examples: `examples/locator-ring-glow_C_t126.jpg`, `..._C_t82.jpg`, `..._C_t48.jpg`
- Shows tactically: a lighter-weight player highlight than A's disc — a
  spinning white ground ring that follows Messi (t=126-132), an orange-red
  glow ring on the ball carrier (t=82-84), a yellow glow in dimmed clips
  (t=48, t=236).
- Animation: the ring rotates (the white ellipse visibly spins across t=126
  -> 128 -> 130) and tracks the player.
- Colors: white, orange-red, yellow.
- Drawn on: footage.

## 15. Callout markers and face chips
- Examples: `examples/callout-markers_C_t80.jpg`, `..._C_t110.jpg`, `..._C_t40.jpg`
- Shows tactically: editorial punctuation drawn on the clip — "Z z z" over
  defenders who don't press (t=80), a trio of white "!" marks over players
  who lose their runner (t=110), the OLD MESSI / NEW MESSI face-chip captions
  that brand the comparison (t=40-46), green battery icon over a walking
  Messi (t=90-92), red "!" above a marker (t=134-138, t=236, t=252).
- Animation: markers pop in for one beat; face chips persist through a clip.
- Colors: white, red, green, yellow.
- Drawn on: footage.

---

## Count summary (from frames_<A|B|C>.csv, the same numbers as measurements.md)

- Overlay-bearing frames (non-empty elements): A 133/253, B 99/250, C 55/142.
- Style families by video: A leans on diagrams with badge tokens plus yellow
  discs/labels on footage; B alternates white-card boards with footage
  inside the card frame; C almost never leaves footage — its overlays are
  light marks (tags, arrows, cones, rings) on raw clips.