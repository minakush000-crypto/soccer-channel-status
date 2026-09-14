# Football Visualisation Design Findings

> **Extracted:** 2026-09-13 from 10 design-focused YouTube tutorials (McKay Johns, Friends of Tracking, John Burn-Murdoch/FT), 5 coding-only tutorials, and 5 similar sources reported for breadth.
> **Method:** Text-only main model. All visual judgment is pasted Gemini output (gemini-3.1-pro-preview, AUTHORITATIVE per project judge rule). No frames reproduced, no layouts copied, no charts replicated. Design principles only.
> **Age of everything:** all extractions are from 2026-09-13. The source videos predate that; their content is the analysts' published work as of that date.
> **Public mirror notice:** this document is mirrored publicly via the Stop hook. No secrets, no API keys, no copyrighted reproductions. Every Gemini quote is a judgment of a frame, not a reproduction of the frame itself.

---

## 1. Visualisation types observed across all sources

Every distinct viz type seen in the 10 design-focused videos, the 5 coding-only videos, and the brief's data-scope (average-positions, momentum are Sofascope data we have but no tutorial built). "Pizza chart" is included because the brief names it; it did not appear in any extracted video but is a known variant of the radar with two semicircular halves (attacking + defensive metrics).

| Visualisation type | What question it answers | What data it needs | Do we have that data (Sofascore) | Readable at 1080p on a phone |
|---|---|---|---|---|
| Shot map (scatter on pitch, dot=shot, size=xG, color=goal) | Where did a team shoot from, how dangerous, and which were goals? | Per-shot: x/y coords, xG, outcome (goal/miss), body part, minute. Team filter. | YES (shotmap endpoint: x/y/xg/outcome/body-part) | Yes (Gemini: pitch and data points large enough; smallest labels may need zoom) |
| Pass map (comet lines on pitch, filtered to a zone or type) | Where did a player deliver passes from / to, and which succeeded? | Per-pass: start x/y, end x/y, outcome, player ID, minute. Filter to a subset (final third, key passes, etc.). | PARTIAL (Sofascore has pass stats per player but not per-pass start/end coordinates. Would need StatsBomb or a tracking provider for true comet lines.) | Yes (Gemini: pitch and lines visible; code text above would be small, but the viz portion is readable) |
| Pass network (node-link on pitch, thresholded by pass count) | What was the team's passing structure, who was the hub, who was isolated? | Pairwise pass counts between players (A passed to B N times), average positions per player, minimum threshold. | PARTIAL (average-positions gives node locations, but pairwise pass counts are NOT in Sofascore. Need StatsBomb/Edge-style event data for edge weights.) | Yes for structure (Gemini: nodes and arrows large enough), but without labels the network is "uninformative regardless of screen size" |
| xG flow chart (cumulative step chart over 90 min, per team) | When did each team create chances, and who dominated xG over the match arc? | Per-shot: minute, xG, team. Cumulative sum per team. Half-time split. | YES (shotmap has minute + xg + team; match_data has half boundaries) | Yes (Gemini: lines and title large enough; axis numbers slightly small but functional) |
| Radar chart (percentile, single or two-player overlay) | How does a player compare to peers across multiple metric categories? | Per-player season stats (goals, assists, xG, xA, progressive carries/passes/receptions, etc.), per-90 normalised, percentile vs peer group (forwards >400 min). | PARTIAL (Sofascore has player ratings and match-level stats, but season-aggregate per-player percentiles need a league-wide stats endpoint. May be available via standings/player-stats API.) | No (Gemini: "text is small, rotated, and cluttered with decimals"; desktop screencast, not designed for phone) |
| Pizza chart (two semicircular percentile halves, attack + defense) | Same as radar but splits attacking vs defensive metrics into two halves of a circle | Same as radar, plus defensive metrics (tackles, interceptions, blocks, aerials won) | PARTIAL (same data gap as radar) | Unknown (not tested in any extracted video; structurally similar readability to radar) |
| Heatmap (KDE or 2D histogram on pitch) | Where did a player/team concentrate activity (passes, touches, shots)? | Per-event x/y coordinates (pass starts, shot locations, defensive actions). Single team filter. | PARTIAL (shotmap gives x/y for shots; average-positions gives one point per player. True heatmaps need per-event coordinates from tracking or event data.) | Yes for the heat field (Gemini: "pitch and heatmap are large enough"), but "the lack of labels means the meaning wouldn't be readable" |
| Small multiples heatmap (grid of mini-pitches, normalized) | How do multiple teams/players compare on a spatial metric across a season? | Per-event x/y for each entity, normalized by a denominator (e.g., opposition passes for defensive actions). | NO (needs season-level event data for multiple teams; Sofascore match-level data is one match at a time) | No (Gemini: "team names and legend text would be illegible" at phone scale; desktop/reference graphic) |
| Pass-sequence diagram (tactical buildup arrows on pitch) | What passing path led to a chance, in what order? | Per-pass: start/end x/y, sequence order, player IDs. A specific buildup sequence. | NO (needs per-pass event data with sequence reconstruction; Sofascore does not provide this) | No (Gemini: "small A/B/C/D labels would be unreadable"; diagram shapes visible but narrative labels too small) |
| xG probability rings (concentric contour rings on pitch, labeled with %) | What is the probability of scoring from each zone on the pitch? | xG model output mapped to pitch zones (precomputed: 30% / 15% / 7% / 1% rings). Static reference graphic, not match-specific. | NO (this is a precomputed pedagogical graphic, not match data. Could be built once as a reusable explainer board.) | Yes (Gemini: "text and lines are large enough to be legible at 1080p on a phone") |
| Season-over-season shot map comparison (two pitches side-by-side) | How did a player's shooting behavior change between periods? | Two shot maps for the same player across different time periods. | PARTIAL (Sofascore shotmap is per-match; aggregating across a season needs multiple match calls or a season endpoint) | Partially (Gemini: "period titles and overall pattern visible, but individual dots, faint pitch lines, player name, and legend would be unreadable") |
| Final-third entry map with xG zones (For vs Against, side-by-side) | Where does a team enter the final third, and where do opponents enter against them? | Per-pass: entry point x/y, xG contribution per zone. For and against. | PARTIAL (Sofascore has pass stats but not per-pass entry coordinates; xG zones could be derived from shotmap) | Partial (Gemini: "wing-cluster patterns and the For/Against split read; the xG numeric values inside zones do not") |
| Average-positions scatter (dots on pitch, one per player) | Where did each player actually spend time on average, and does the data-shape match the nominal formation? | Per-player average x/y coordinates for one team. | YES (Sofascope average-positions endpoint) | Yes (projecting from similar pitch-scatter verdicts: dots on pitch are large enough; labels would need to be sized for phone) |
| Momentum chart (per-minute timeline, -100 to +100) | When did the match turn, and which team was on top at each minute? | Per-minute momentum value (-100 to +100), 90+ minutes. | YES (Sofascope momentum endpoint) | Yes (a simple two-color timeline is inherently phone-readable; no extracted video tested this but the format is a bar/area chart, which Gemini scored 8-9/10 for readability in the Burn-Murdoch line-chart verdicts) |
| Dual-panel line chart (same data, two scales) | Does the data tell a different story under a different scale/perspective? | Two time series of the same metric, plotted on linear + log (or raw + rolling average). | N/A for football (COVID data viz; the principle transfers but the chart type is not a standard football board) | Yes (Gemini: "high contrast, clean layout, and direct labelling make the main narrative clear") |
| Small multiples with area shading (grid of line charts, filled areas) | How does a metric compare across many entities at a glance? | Time series for many entities, area-filled. | N/A (not a standard football board; principle transfers) | No (Gemini: "text and individual chart details too small for phone without zooming") |
| Annotated scatter plot (quadrant labels, target zone) | Which entities meet a target threshold across two dimensions? | Two metrics per entity, quadrant boundaries, annotation labels. | N/A (not football-specific; could be repurposed for xG vs shots, etc.) | Yes (Gemini: "text and data points large enough to be legible") |
| Side-by-side line chart with direct labels | How do two groups diverge over time? | Two+ time series, directly labeled (no legend). | N/A (design principle transfers to any timeline comparison) | Yes (Gemini: "text and lines large and clear enough for phone") |
| Small multiples with shaded difference areas | What is the gap between two groups, and when does it appear? | Two time series per panel, area between them shaded and labeled. | N/A (design principle transfers) | Borderline/No (Gemini: "main title and gap shapes visible, but axis numbers, line labels, and annotations too small") |
| Bounding-box / tracking overlay (on-frame, not a board) | Where are players and the ball in a given frame? | Per-frame detections (bbox, team, ID). | NO (this is CV output, not a Sofascore board; produced by runpod_fulltrack.py, not a designed graphic) | N/A (on-frame overlay, not a standalone board) |
| Ball acquisition % text overlay (on-frame) | Which team controlled the ball in a window? | Ball possession % per team. | YES (possession is in match_data) but this is an on-frame overlay, not a board |

---

## 2. Which visualisations appear that we cannot currently produce?

Our pipeline produces three board types: **formation**, **possession**, **stat_card**. Every other viz type in the table above is one we cannot currently produce as a designed board. The full list of missing boards:

1. **Shot map** (scatter on pitch, xG-sized dots, goal-colored)
2. **Pass map** (comet lines, filtered to a zone or type)
3. **Pass network** (node-link, thresholded by pass count)
4. **xG flow chart** (cumulative step chart over 90 min)
5. **Radar chart** (percentile, single or two-player)
6. **Pizza chart** (two-halves percentile)
7. **Heatmap** (KDE or 2D histogram on pitch)
8. **Small multiples heatmap** (defensive pressing, normalized)
9. **Pass-sequence diagram** (tactical buildup arrows)
10. **xG probability rings** (contour map, static explainer)
11. **Season-over-season shot map comparison**
12. **Final-third entry map with xG zones** (For vs Against)
13. **Average-positions scatter** (data-derived formation shape)
14. **Momentum chart** (per-minute timeline)
15. **Annotated scatter plot** (quadrant, repurposable for football metrics)
16. **Side-by-side comparison layout** (applicable to any pair of boards)

The coding-only videos produced on-frame overlays (bounding boxes, team-color boxes, ball-acquisition % text, perspective-transformed top-down, speed/distance text). These are not designed boards and are not production-quality graphics; they are OpenCV `cv2.putText`/`rectangle` outputs on broadcast frames. The only board-like output from those videos is the perspective-transformed 2D top-down rectangle, which our runpod_fulltrack.py already produces a version of.

---

## 3. Which of those does Sofascore data already support?

Cross-referencing the missing boards against the project's Sofascore data-scope (match_data, average-positions, momentum, shotmap, player ratings, injuries, standings/form/H2H, referee/venue):

| Missing board | Sofascore data supports it? | What we have | What we lack |
|---|---|---|---|
| Shot map | **YES, fully** | shotmap: per-shot x/y/xg/outcome/body-part + team | Nothing missing for a basic shot map |
| xG flow chart | **YES, fully** | shotmap: per-shot minute + xg + team; match_data for half boundaries | Nothing missing |
| Momentum chart | **YES, fully** | momentum: per-minute -100..+100 | Nothing missing |
| Average-positions scatter | **YES, fully** | average-positions: x/y per player | Nothing missing (could overlay on formation board) |
| Pass network | **PARTIAL** | average-positions (node locations) | Pairwise pass counts between players (edge weights). Need event-level data (StatsBomb/Edge) or a Sofascore endpoint not yet tested. |
| Pass map (comet lines) | **PARTIAL** | pass stats per player (totals) | Per-pass start/end coordinates. Need event-level data. |
| Heatmap | **PARTIAL** | shotmap x/y (for shot heatmaps only) | Per-event coordinates for passes, touches, defensive actions. A shot-only heatmap is possible. |
| Radar / pizza chart | **PARTIAL** | player ratings, match-level stats per player | Season-aggregate per-player stats + peer-group percentiles. May be reachable via a Sofascore season-stats endpoint but not confirmed. |
| Final-third entry map | **PARTIAL** | shotmap (xG zones derivable) | Per-pass entry coordinates. Could approximate zones from shotmap + average-positions. |
| Season-over-season comparison | **PARTIAL** | shotmap per match (could aggregate) | Needs multiple match calls per player per season; feasible but expensive |
| xG probability rings | **NO (but buildable as static)** | This is a precomputed explainer, not match data. Could be rendered once and reused across episodes. | No match data needed; it is a reference graphic. |
| Small multiples heatmap | **NO** | One match at a time from Sofascore | Season-level event data for multiple teams |
| Pass-sequence diagram | **NO** | No per-pass event data | Needs event-level sequence data (StatsBomb) |
| Annotated scatter plot | **YES (derivable)** | match_data stats (shots, xG, possession, etc.) | Could plot any two match stats as a scatter with quadrant labels; data is there |
| Side-by-side comparison layout | **YES (design pattern)** | Any two boards we already build | This is a layout decision, not a data question |

**Summary:** 4 boards are fully supported by Sofascore data today (shot map, xG flow, momentum, average-positions). 5 are partially supported (pass network, pass map, heatmap, radar/pizza, final-third entry). 3 need event-level data we do not have (small multiples, pass-sequence, season comparison). 1 is a static graphic (xG rings). 2 are design patterns applicable to existing data (annotated scatter, side-by-side layout).

---

## 4. What makes a good one readable?

The Gemini judgments (gemini-3.1-pro-preview, AUTHORITATIVE per project judge rule) surfaced four recurring design properties. Every verdict below is pasted from the extraction reports; no frame is reproduced.

### 4a. Labelling

Labelling is the single most consistent differentiator between a viz that communicates and one that is "uninformative regardless of screen size" (Gemini, on the unlabeled pass network).

**The worst:** McKay Johns' pass network (GnAqgv-Heb0) scored labelling 1/10. Gemini: "No labels for players, positions, or pass volumes; completely lacks context." His raw shot map (04zmAyYEgs4) scored 2/10: "No title, legend, or axis labels; relies entirely on code context." His raw xG flow chart (WrhfIYxeiwE) scored 2/10: "Missing x-axis label, y-axis label, title, and legend."

**The best:** McKay Johns' Haaland shot map (v3uI44ZA_WU) scored 8/10: "Clear title, subtitle, legend, and data labels; some text is slightly small." Burn-Murdoch's dual-panel line chart (uoFN3nxeMco) scored 9/10: "Direct line labels eliminate legend-hunting; titles provide clear narrative context." Burn-Murdoch's annotated scatter scored 8/10: "Clear axes, title, and legend; country labels are mostly readable."

**The pattern:** labelling jumps from 2-4/10 to 7-9/10 when the analyst adds (a) a title that states the argument, (b) direct labels on data marks instead of a separate legend, and (c) a subtitle or caption that identifies the player/match/metric. The single biggest labelling failure across all 10 videos is the y-axis on the xG flow chart, which is NEVER labeled (Gemini: 2-6/10 across all states). The viewer must infer "cumulative xG" from context.

### 4b. Colour discipline

**The worst:** McKay Johns' single-player radar (PHTLEsEUrxQ) scored 1/10. Gemini: "Fatal flaw: The data polygon fill colour is identical to the outer background ring colour. The actual data shape is invisible." His raw xG flow chart scored 4/10: "Uses four distinct colours for the lines, but without a legend, their meaning is unclear."

**The best:** McKay Johns' Haaland shot map scored 9/10. Gemini: "Excellent use of red for goals and grey for misses against a dark background." Burn-Murdoch's dual-panel chart scored 9/10: "Bold red perfectly highlights the primary subject against muted secondary lines." Burn-Murdoch's small multiples with area shading scored 9/10: "Excellent use of a single highlight colour (red/pink) to draw attention to the key metric."

**The pattern:** the best colour scores (8-9/10) all follow the same rule: a dark or neutral background, ONE accent color for the focal data, and muted/grey for everything else. McKay Johns states this explicitly as "concept 2" in the Haaland shot map video: exactly three colours (background + red + white). When analysts use 4+ default matplotlib colours without a legend, colour discipline drops to 4/10. When the accent colour matches the background (the radar flaw), it drops to 1/10.

### 4c. Use of space

**The worst:** McKay Johns' radar template (PHTLEsEUrxQ) scored 3/10: "Chart is scaled too large for the viewable area, resulting in cropped titles and labels." His raw shot map in notebook context scored 5/10: "Large empty margins and notebook UI leave the pitch visual smaller than ideal."

**The best:** Burn-Murdoch's small multiples with area shading scored 9/10: "Very efficient packing of many charts into a single view." McKay Johns' titled shot map scored 7-8/10: "The pitch fills the available space well, making the data points clear."

**The pattern:** space scores highest when the viz fills the frame (7-9/10) and lowest when notebook chrome, webcam insets, or excessive margins steal real estate (3-6/10). For our pipeline, this means boards should be rendered as full-frame 1920x1080 images, not embedded in a notebook or surrounded by UI.

### 4d. Where the eye goes first

**The worst:** McKay Johns' radar template scored 4/10: "Eye drawn to dense, messy cluster of numbers in center rings, rather than the metrics." The camouflaged single-player radar drew the eye to "the large solid green block at the top, a camouflage effect creating meaningless visual mass."

**The best:** McKay Johns' Haaland shot map scored 9/10: "The cluster of red dots (goals) in the penalty area immediately draws attention." Burn-Murdoch's small multiples with shaded difference areas scored 10/10: "The bold, declarative title and the prominent green wedges immediately and successfully communicate the core message." The Friends of Tracking passing network (fCjGS7If_E0) scored 9/10: "The eye is immediately drawn to the dense, thick-lined network on the left (Italy)."

**The pattern:** the eye goes to (1) the brightest/most-saturated color block, (2) the largest text, (3) the densest cluster of marks. When the title is the largest text, it wins (Burn-Murdoch). When a red cluster of goals is the brightest block, it wins (McKay Johns). When a camouflage effect creates a meaningless green block, the eye is misled (the radar failure). The design judgment is: make the focal data the brightest and most clustered thing on the canvas, and make the title the largest text.

### 4e. The title-as-message principle (Burn-Murdoch)

Burn-Murdoch's thesis, supported by Borkin et al. eye-tracking research: people read Z-shape (title first, then axes/text, then plot). The title is not a label; it is the message. Gemini confirmed this empirically: Burn-Murdoch's charts scored 9-10/10 on eye-attraction because the bold declarative title communicates the core message before the viewer even reads the plot. McKay Johns' untitled charts scored 7-8/10 because the eye went to the data marks, not to a message.

Gemini on Burn-Murdoch's vaccine-effect chart (10/10 eye-attraction): "The bold, declarative title and the prominent green wedges immediately and successfully communicate the core message." On his dual-panel chart (9/10): "The sharp upward trajectory of the bright red line immediately commands attention."

The transferable rule: a title that says "Cumulative xG" is a waste of the most-read pixels. A title that says "Liverpool dominated chances but Real Madrid won" is a message the viewer carries away.

---

## 5. What do these analysts show that we do not, and why does it matter to a viewer?

### 5a. Selective visibility (show only what matters)

Every experienced analyst in the sample filters aggressively before plotting. McKay Johns filters penalties out of shot maps ("unless we're doing separate penalty analysis, it doesn't really help us in our actual open play shot analysis"). He filters pass maps to final-third entries only ("plotting every pass a player made is not effective"). He thresholds pass networks to pairs with 4+ passes ("we only want the connections where the pass count was greater than four"). David Sumpter (Soccermatics) thresholds passing networks to >=13 completed passes and says the threshold IS the design: below it, the network is clutter and the shape is lost. He treats decluttering as a narrative act, not a cosmetic one.

Our pipeline does not filter. Our formation board shows every player at their nominal position. Our possession board shows a single percentage. Our stat_card shows every stat Sofascore returns. We have no threshold, no filter, no editorial choice about what to leave out.

**Why it matters to a viewer:** an unfiltered board is a data dump. A viewer's eye has nowhere to go because everything is equally present. A filtered board has a focal point because the analyst chose what to remove. This is the difference between a board that reads in 2 seconds and one that reads never.

### 5b. Encoding economy (one mark, one meaning)

David Sumpter's shot-map grammar: dot = shot, ring around dot = goal. No legend needed in principle. McKay Johns' shot map: dot size = xG value, dot color = goal vs miss. Two variables in one mark. His xG flow chart: step line (not smooth), team color in line and title. The title IS the legend.

Our boards use no encoding beyond text and bar length. We do not encode a second variable in mark size, mark shape, or mark color. A viewer of our stat_card reads numbers; a viewer of a shot map reads a spatial argument.

**Why it matters to a viewer:** encoding economy lets a viewer perceive two things (volume + outcome, location + danger) in the time it takes to perceive one. A board that encodes one variable per mark is a table. A board that encodes two variables per mark is an argument.

### 5c. Honesty over beauty (reject dishonest heatmaps)

David Sumpter explicitly denounces smoothed KDE heatmaps: the Gaussian smudge "isn't necessarily the case that he made passes from these places" because it invents data between actual pass locations. He prefers raw 2D histograms. He also reframes heatmaps by choosing the denominator: "touches within 15 seconds of a shot" instead of "all touches" because wing touches are irrelevant to a striker's danger. The denominator IS the design judgment.

Our pipeline has no heatmap. If we build one, the temptation will be to use seaborn's KDE with fill=True because it looks scientific. The experienced analyst says: that is dishonest. Use a raw histogram, or anchor the heatmap to a question (touches within N seconds of a shot, passes that entered the final third).

**Why it matters to a viewer:** a smoothed heatmap looks authoritative but lies. A raw histogram looks blocky but tells the truth. A viewer who internalises a dishonest heatmap makes wrong conclusions. A viewer who internalises an honest one makes right ones.

### 5d. Normalization as honesty (defensive metrics)

Sumpter's defensive-heatmap rule: divide defensive actions by opposition passes, otherwise the worst teams (who defend all game) look "most intense." The normalization is invisible in the final graphic but it is the whole point. Without it, the viz is actively misleading.

Our stat_card shows raw stats. We do not normalize. A team with 60% possession will have fewer tackles per minute of defensive play than a team with 40% possession. Our stat_card makes the 40%-possession team look "more defensive" without context.

**Why it matters to a viewer:** unnormalized defensive stats reward bad teams. A viewer who sees "Team A made 30 tackles, Team B made 15" thinks Team A defended better. But if Team A had the ball 30% of the game and Team B had it 70%, Team A was just defending longer. Normalization reveals who defended efficiently, not who defended a lot.

### 5e. Comparative layout as the story

Nearly every viz in the sample is a comparison: three strikers side-by-side shot maps (Sumpter), two teams side-by-side passing networks (Italy wheel vs England line), two players side-by-side heatmaps (Pirlo vs Schweinsteiger), For-vs-Against side-by-side final-third maps, before/after season comparison (Xc6IG9-Dt18), linear-vs-log dual panel (Burn-Murdoch). The contrast IS the narrative. A single isolated pitch is weaker; the analysts default to pairs or triples.

Our boards are all singular. One formation, one possession bar, one stat card. No comparison. No before/after, no for/against, no player-vs-player.

**Why it matters to a viewer:** a single board presents a fact. A comparison presents an argument. "Haaland scored 36 goals" is a fact. "Haaland's goals cluster inside the box while Ronaldo's sprawl outside it" is an argument. Arguments are what make a tactical-analysis video worth watching.

### 5f. Convention discipline (attack left-to-right, always)

Sumpter states the rule: always plot attacking left-to-right unless matching video direction. McKay Johns mirrors one team's shots to the left half so both teams do not overlap. These are reading-convention choices so the audience learns one direction.

Our formation board does not have a consistent attacking direction. Our boards do not mirror. A viewer cannot build a spatial intuition because the orientation changes.

**Why it matters to a viewer:** convention consistency lets a viewer build a mental model. After 3 boards, they know "left = attacking" and read instantly. Without convention, every board is a fresh puzzle.

### 5g. Title as message, not label

Burn-Murdoch: the title is the most-read pixel on the chart. A title that says "Most Western countries are on the same trajectory" is a statement. A title that says "Cumulative COVID cases" is a waste. Gemini confirmed: charts with declarative titles scored 9-10/10 on eye-attraction; charts with no title or a generic title scored 7-8/10.

Our boards have no titles beyond the board type ("Formation", "Possession", "Stats"). These are labels, not messages.

**Why it matters to a viewer:** the title is what a viewer reads first and remembers last. A label is forgotten. A message is carried away.

### 5h. The KPI = single number + spatial viz

Sumpter's Hammarby example: one number (10% ball recovery in a zone) shown on a pitch with zones. "It doesn't tell you the solution but it tells you part of the problem." The design judgment is boiling a season down to one number the whole club agrees on.

Our stat_card shows many numbers. None of them are boiled down to one. None are placed in a spatial context.

**Why it matters to a viewer:** one number in context is memorable. Many numbers in a table are forgettable. A viewer who remembers "10% in the left wing" has a takeaway. A viewer who sees 15 stats has nothing.

---

## 6. Which visualisations carry a story on their own?

Ranked from highest narrative carriage (tells the full story without voiceover) to lowest (requires voiceover to be meaningful). Justification for each is drawn from the extraction reports' `narrative_carriage` field and the Gemini verdicts.

### Rank 1: xG flow chart (cumulative, per team, titled) — HIGH

**Justification:** McKay Johns' completed xG flow chart (WrhfIYxeiwE, final state) carries the full match narrative alone: title identifies the match, team-colored step lines show cumulative xG over two halves, the gap between xG totals and the actual result tells the "Liverpool should have scored more" story without a single word of commentary. The analyst calls it "a very powerful use of our graphing" and "one of the most powerful uses of our graphing." Gemini narrative_carriage: HIGH. The title IS the legend (team names colored to match the lines), so no separate legend is needed. The step shape encodes discrete chance events; the slope encodes dominance periods.

**Why it ranks above shot map:** the xG flow chart has a time dimension the shot map lacks. It tells you not just WHERE chances came from but WHEN, which is the match arc. A viewer reads the whole game state trajectory in one image.

### Rank 2: Shot map (scatter on pitch, titled, with legend) — HIGH

**Justification:** McKay Johns' Haaland shot map (v3uI44ZA_WU) carries the whole story alone: WHO (Haaland, titled), WHAT (every non-penalty shot, 22/23 PL), OUTCOME (red = goal, grey = miss), and summary stats (total shots, goals, xG, xG/shot, avg distance) sit around the pitch as furniture. Gemini narrative_carriage: HIGH. "A viewer gets the argument, Haaland's goals cluster central-left inside the box, without any voiceover. It is a single-metric, single-argument graphic by design." Gemini eye-attraction 9/10: the red goal cluster is the undisputed focal pull.

**Why it ranks here:** the shot map tells a spatial argument (where danger comes from) but not a temporal one (when). It is the strongest single-frame narrative but lacks the match-arc dimension the xG flow chart has.

### Rank 3: Pass network (node-link, thresholded, with labels) — HIGH

**Justification:** David Sumpter's passing networks (fCjGS7If_E0) carry the argument through SHAPE alone. Italy's dense wheel-around-Pirlo vs England's sparse two-node Hart-Carroll line tells the centralized-vs-decentralized thesis in one glance. Gemini narrative_carriage: HIGH. "The SHAPE of the network IS the argument." Gemini eye-attraction 9/10: "The eye is immediately drawn to the dense, thick-lined network on the left (Italy)."

**Why it ranks here:** the network tells a structural argument (how the team plays) but requires labels to identify WHO. Without labels (as in McKay Johns' version, labelling 1/10), it drops to MEDIUM because the viewer cannot identify the hub player. With labels, it is HIGH. Our Sofascore data has average-positions (node locations) but not pairwise pass counts (edge weights), so a true pass network is only partially buildable today.

### Rank 4: Momentum chart (per-minute timeline) — MEDIUM-HIGH

**Justification:** not directly tested in any extracted video (no tutorial built one), but the data is a per-minute signed value (-100 to +100). A two-color timeline (red for home dominance, blue for away) is inherently readable: a viewer sees WHEN the match turned and which team was on top at each minute. It carries the temporal dimension of the match arc, similar to the xG flow chart but without the xG quantification. Gemini's verdicts on Burn-Murdoch's timeline charts (8-9/10 eye-attraction, 9/10 labelling with direct labels) support the readability of simple timelines.

**Why it ranks here:** it carries WHEN but not WHERE or WHAT. A viewer sees "the match turned at 60 minutes" but not "because of a substitution" or "in the left wing." It is a companion to the xG flow chart, not a replacement.

### Rank 5: Final-third entry map (For vs Against, side-by-side) — HIGH (with zones)

**Justification:** Sumpter's For-vs-Against final-third map (fCjGS7If_E0) makes the asymmetry the story: Liverpool dangerous from both wings, opposition slightly more dangerous down Liverpool's right (Alexander-Arnold's side). Gemini narrative_carriage: HIGH. "The For-vs-Against split on two pitches makes the asymmetry the story." The xG values quantify it but the spatial pattern carries the argument first.

**Why it ranks here (below the top 3):** it needs xG zone labels to be fully persuasive, and those were the least-readable element (Gemini: "the small xG numbers inside the zones are very hard to read"). Without the numbers, it is a directional argument; with them, it is a quantified one. Our data partially supports it (shotmap xG zones derivable, but per-pass entry coordinates are missing).

### Rank 6: xG probability rings (static explainer) — HIGH (as a reusable explainer)

**Justification:** the xG rings (Xc6IG9-Dt18) are the core explanatory device of the entire video. Gemini narrative_carriage: HIGH. "Every subsequent viz and the goal-clip analysis refers back to these rings. Alone, it communicates the central thesis: scoring probability is a function of position, not finishing skill." The rings are phone-readable (Gemini: "text and lines are large enough").

**Why it ranks here:** it is a static reference graphic, not a match-specific board. It carries a conceptual story (what xG means) but not a match story (what happened in this game). It would be built once and reused across episodes as an explainer, not regenerated per match.

### Rank 7: Annotated scatter plot (quadrant, target zone) — HIGH (design pattern)

**Justification:** Burn-Murdoch's annotated scatter (uoFN3nxeMco) scored 8/10 labelling, 9/10 colour, 8/10 space. Gemini narrative_carriage: HIGH. "The annotations transform an abstract scatter plot into a story. Without the green target zone label and the quadrant annotations, this is dots and numbers. With them, it is a quest." The annotations ARE the narrative.

**Why it ranks here:** this is a design pattern, not a football-specific board. But it transfers directly: any two match stats (xG vs shots, possession vs passes completed, etc.) can be plotted as an annotated scatter with quadrant labels ("dominant", "efficient", "wasteful", "passive"). It would be a comparison board across teams or matches.

### Rank 8: Season-over-season shot map comparison — MEDIUM-HIGH

**Justification:** the before/after comparison (Xc6IG9-Dt18) shows behavioral change: 2018 last 12 games (scattered long shots) vs 2019 first 12 games (concentrated closer to goal). Gemini narrative_carriage: MED-HIGH. "The side-by-side layout makes the improvement immediately visible through dot density and location shift." It demonstrates that xG is prescriptive, not just descriptive.

**Why it ranks here:** it needs two time periods of data (season aggregation), which is expensive with per-match Sofascore calls. The narrative is strong but the data cost is high.

### Rank 9: Pass map (comet lines, filtered, with title + legend) — MEDIUM

**Justification:** McKay Johns' final pass map (l53Ur6itLh0, with title + legend) carries: who (Enzo Fernandez), what (final-third entries), where (pass start/end locations), outcome (green=complete, red=incomplete). Gemini narrative_carriage: MEDIUM. "The title and legend together give a viewer enough to understand the viz standalone. But there is no annotation of key passes, no directional arrows, no stat callouts. The narrative is 'here are the passes' not 'here is why they matter.'"

**Why it ranks here:** the pass map shows a pattern but not an argument. A viewer sees where passes went but not which ones were important. It needs annotation to become HIGH, and our Sofascore data lacks per-pass coordinates to build it at all.

### Rank 10: Average-positions scatter — MEDIUM

**Justification:** not directly tested in any video, but the data is one dot per player at their average x/y. It shows the data-derived formation shape, which is a different question from the nominal formation (which our formation board already answers). The narrative is "this is where the team actually played, not where the formation card says they played." The contrast between nominal and actual positions is the argument.

**Why it ranks here:** it shows a shape but not a story. A viewer sees dots on a pitch and must infer whether the shape is good or bad. It becomes HIGH when overlaid on the nominal formation (comparison) or when annotated with player names and a message ("Liverpool's fullbacks pushed higher than their nominal position").

### Rank 11: Radar / pizza chart (percentile) — MEDIUM

**Justification:** McKay Johns' two-player comparison radar (PHTLEsEUrxQ) shows where one player dominates the other across 8 attacking metrics. Gemini narrative_carriage: MEDIUM. "The narrative potential is there (two elite forwards compared) but the design fails to deliver it." The single-player radar carries even less: "this player is good everywhere" but no framing of why it matters.

**Why it ranks here:** the radar shows a profile but not a story. A viewer sees a shape and must know what a "good" shape looks like to interpret it. Without a comparison peer, it is just a filled polygon. With one, it is an argument, but only if labeled. The design execution in every extracted example was poor (1-4/10 labelling, 1-6/10 colour).

### Rank 12: Heatmap (KDE / 2D histogram on pitch) — MEDIUM-LOW

**Justification:** McKay Johns' heatmap (CwVKlCMPFLc) shows WHERE passes concentrated but without title, legend, or annotations it cannot tell WHO, WHAT, or WHY. Gemini narrative_carriage: LOW (without title) to MEDIUM-LOW (with title). "The viz shows the pattern; the narrator must explain the meaning." Sumpter adds: a single heatmap is "dishonest and meaningless without context." It becomes MEDIUM-HIGH when paired side-by-side with another player's heatmap (comparison).

**Why it ranks here:** the heatmap is the most narratively dependent on voiceover. It shows density but not direction, not outcome, not importance. It is a background layer, not a foreground argument.

### Rank 13: Possession bar (we already have) — LOW

**Justification:** a single percentage (54.6% vs 45.4%) tells the viewer one fact with no spatial, temporal, or qualitative dimension. It is the lowest-narrative board we produce. It becomes MEDIUM when paired with a pass-network or average-positions overlay (showing HOW possession translated to spatial dominance), but alone it is a number.

### Rank 14: Formation board (we already have) — LOW-MEDIUM

**Justification:** the nominal formation (4-3-3, 4-2-3-1, etc.) tells the viewer the team's intended shape. It is a reference, not an argument. It becomes MEDIUM when overlaid with average positions (showing actual vs intended), but alone it is a labeled diagram.

### Rank 15: Stat card (we already have) — LOW

**Justification:** a table of stats (shots, xG, passes, tackles) is a data dump. No spatial context, no temporal context, no comparative layout, no message. A viewer reads numbers and forgets them. It is the board most in need of redesign: the same data, plotted as an annotated scatter or a KPI-with-spatial-context, would carry far more narrative.

---

## 7. Recommendation: which missing boards to build FIRST

Ranked by (narrative carriage x data-we-have x readability), with tool guidance (2D top-down vs 3D).

### Build #1: xG flow chart

**Narrative carriage:** HIGH (rank 1). The match arc in one image. Title + team-colored step lines = full story without voiceover.

**Data we have:** YES, fully. Sofascore shotmap gives per-shot minute + xG + team. Cumulative sum per team, split at 45 min, is a 20-line pandas operation.

**Readability:** HIGH. Gemini scored the titled version 8/10 labelling, 8/10 colour, 7-8/10 space, 7-8/10 eye-attraction. Phone-readable: "Yes, lines and title large enough; axis numbers slightly small but functional." The only fix needed: label the y-axis (every extracted example forgot to).

**Tool:** 2D. This is a timeline chart, not a pitch graphic. A 3D perspective would distort the time axis. Use matplotlib with drawstyle='steps-post', team colors in lines and title (via highlight_text or manual), 0/45/90 x-ticks only, remove top/right spines. Landscape 16:9.

**Build effort:** LOW. ~50 lines of plotting code on top of existing shotmap data fetching. No new data source needed.

**Why first:** it fills the most watchable seconds per build-hour. A 30-60s board that tells the whole match arc, readable on a phone, from data we already have, in 50 lines of code. It is also the board three judges would most want to see because it answers the question every tactical video must answer: "who deserved to win?"

### Build #2: Shot map

**Narrative carriage:** HIGH (rank 2). Where goals come from, which shots were dangerous, which were goals. The spatial argument.

**Data we have:** YES, fully. Sofascore shotmap gives per-shot x/y/xg/outcome/body-part + team.

**Readability:** HIGH. Gemini scored the titled version 8/10 labelling, 9/10 colour, 7-8/10 space, 9/10 eye-attraction. Phone-readable: "Yes, mostly, though the smallest text may require zooming." The design template: dark background (#0C0D0E), one accent color for goals (red), grey for misses, white for text/lines. Exactly three colors. Dot size = xG. Title states the argument ("Haaland's goals cluster inside the box").

**Tool:** 2D top-down pitch. Part 1 of this research found that 3D perspective distorts 2D positional data, so a shot map must be 2D top-down. Use mplsoccer's VerticalPitch (vertical orientation matches how a fan reads a half from behind the goal). Mirror one team's shots to the left half so both teams do not overlap.

**Build effort:** MEDIUM. ~100-150 lines of plotting code. Need to integrate mplsoccer (or build a pitch-drawing utility) into the board-generation step. The design layer (dark pitch, 3-color rule, xG-sized dots, title-as-message) is the hard part, not the data.

**Why second:** it is the strongest single-frame spatial narrative. It answers "where did the danger come from?" which is the second question every tactical video must answer (after "who deserved to win?"). It is also the most-featured board across the entire sample (4 of 10 videos built one), proving experienced analysts reach for it first.

### Build #3: Momentum chart

**Narrative carriage:** MEDIUM-HIGH (rank 4). When the match turned. Complementary to the xG flow chart (temporal, but without xG quantification).

**Data we have:** YES, fully. Sofascore momentum endpoint gives per-minute -100..+100.

**Readability:** HIGH (projecting from Burn-Murdoch's timeline verdicts: 8-9/10 eye-attraction, 9/10 labelling with direct labels). A two-color area chart (red for home, blue for away, centered at zero) is inherently phone-readable because it is one shape with one color split.

**Tool:** 2D. Timeline chart, same as xG flow. No 3D benefit.

**Build effort:** LOW. ~30-40 lines of plotting code. The data is a 90-element array. Plot as a filled area chart with a zero line, team colors, minute ticks at 0/45/90, and a declarative title ("The match turned at 65 minutes").

**Why third:** it is the cheapest board to build (data is a single array, plot is a filled area) and it fills a temporal gap the xG flow chart does not cover (xG flow shows chance quality over time; momentum shows territorial/pressure dominance over time). Together, xG flow + momentum + shot map give a viewer the full match story: when (momentum), what quality (xG flow), and where (shot map).

### Build #4: Average-positions scatter (data-derived formation)

**Narrative carriage:** MEDIUM (rank 10). Shows where the team actually played, not where the formation card says they played. The contrast with the nominal formation is the argument.

**Data we have:** YES, fully. Sofascore average-positions endpoint gives x/y per player.

**Readability:** MEDIUM-HIGH (projecting from pitch-scatter verdicts: dots on pitch are large enough; labels need to be sized for phone). The design layer: team-colored dots at each player's average position, player names or jersey numbers as labels, pitch markings as context. Optionally overlay on the nominal formation to show the gap between intended and actual.

**Tool:** 2D top-down pitch. Same constraint as shot map: 3D distorts positional data.

**Build effort:** LOW-MEDIUM. ~60-80 lines. The data is one row per player with x/y. The design challenge is making labels readable at phone scale (Gemini's recurring labelling demerit) and deciding whether to show one team or both (comparison).

**Why fourth:** it is the board that most directly improves what we already have. Our formation board shows a nominal shape; the average-positions scatter shows the actual shape. Overlaying the two (or showing them side-by-side, per the comparison-layout principle) turns a reference diagram into an argument: "Liverpool's fullbacks played 15m higher than their nominal position." It is also a stepping stone to a pass network (same node locations, just add edges when we get pairwise pass data).

### What NOT to build first (and why)

- **Pass network:** HIGH narrative but only PARTIAL data (no pairwise pass counts from Sofascore). Build it when we integrate StatsBomb or a pass-event source. Average-positions is the foundation.
- **Radar / pizza chart:** MEDIUM narrative, PARTIAL data (need season-aggregate percentiles), and the worst readability scores in the entire sample (1-4/10 labelling, 1/10 colour on the single-player version). The design execution is hard and the payoff is lower than spatial boards.
- **Heatmap:** MEDIUM-LOW narrative alone, PARTIAL data (need per-event coordinates for non-shot heatmaps), and the most voiceover-dependent board in the sample. Build it after the top 4, and only as a shot-location heatmap (data we have) or a comparison pair (Sumpter's side-by-side principle).
- **Small multiples:** NO data (need season-level event data for multiple teams). Not feasible with Sofascore per-match scope.
- **xG probability rings:** HIGH narrative as a reusable explainer but not match-specific. Build it once as a static asset, not as a per-episode board. Low priority because it does not fill 583s of per-episode board time.
- **3D boards for any of the above:** Part 1 found 3D perspective distorts 2D positional data (Stage 12: "DESIGN not DIMENSION; a well-designed 2D board would beat this 3D frame"). All four recommended boards are 2D. Non-positional charts (xG flow, momentum) are inherently 2D timelines. Positional charts (shot map, average-positions) are 2D top-down. 3D is not the bottleneck; design is.

### Summary: the first 4 boards, in build order

| Order | Board | Narrative | Data | Readability | Tool | Effort |
|---|---|---|---|---|---|---|
| 1 | xG flow chart | HIGH (match arc) | YES (shotmap) | HIGH (Gemini 8/10) | 2D timeline | LOW (~50 lines) |
| 2 | Shot map | HIGH (spatial argument) | YES (shotmap) | HIGH (Gemini 9/10 eye) | 2D top-down pitch | MEDIUM (~150 lines) |
| 3 | Momentum chart | MED-HIGH (when it turned) | YES (momentum) | HIGH (simple timeline) | 2D timeline | LOW (~30 lines) |
| 4 | Average-positions | MEDIUM (actual vs nominal) | YES (avg-positions) | MED-HIGH (dots on pitch) | 2D top-down pitch | LOW-MED (~70 lines) |

Combined, these four boards add approximately 120-240 seconds of watchable, narratively-loaded graphics per episode (30-60s each), from data we already have, at a total build cost of ~300 lines of plotting code plus the shared design layer (dark pitch, 3-color rule, Bebas Neue / Barlow Condensed typography, title-as-message). The design layer is the hard part and it is shared across all four.

The design layer to adopt (from the extraction, not copied from any single source):
- **Background:** dark (#0C0D0E or similar), not white. Every high-scoring viz in the sample used a dark or neutral background.
- **Colour rule:** exactly 3 colors per board (background + one accent for focal data + white for text/lines). McKay Johns states this as "concept 2"; Gemini confirmed it with 9/10 colour scores.
- **Typography:** a condensed athletic face (Bebas Neue, Barlow Condensed). No tutorial in the sample used one; all scored lower on labelling than they could have. This is our differentiation opportunity.
- **Title:** a declarative message, not a label. Burn-Murdoch's principle, confirmed by Gemini 9-10/10 eye-attraction scores.
- **Comparison:** default to side-by-side when possible (two teams, two halves, for vs against). Every experienced analyst in the sample did this.
- **Convention:** attack left-to-right, always, unless matching video direction. Sumpter's rule.

These six design rules are not copied from any single tutorial. They are the intersection of what every high-scoring viz in the sample did and what every low-scoring viz did not do. They are design principles, not designs.