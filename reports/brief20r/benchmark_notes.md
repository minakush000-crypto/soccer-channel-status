# Brief 20r pre-work — one full benchmark episode watched (not imagined)

Source: Football Oculus, "Why Barcelona is the Most Dangerous Team in
Europe | Analysis of Hansi Flick's Barca" — https://www.youtube.com/watch?v=q_bkQib7Sk8
Downloaded 2026-10-10 into /mnt/d/scratch/soccer-staging/benchmark/ as
football_oculus_barca.mp4 (1280x720 av1 60fps + opus, merged mp4).
Measured with ffprobe: duration 500.841 s (8:21).

## Measured

- 62 stamped frames sampled every 8 s (committed under bench_frames/,
  12 of them the representative set read this session).
- Scene cuts (ffmpeg scene > 0.30): 7. Extended scan (scene > 0.14):
  15 transitions; segment lengths 11.7, 11.9, 15.6, 10.7, 80.3, 68.3
  (split by 2 sub-cuts), 43.2, 56.4, 71.2, 7.0, 0.75, 12.2, 77.6 s.
  Chapters run 60-90 s with internal motion (draw-on lines, moving dots,
  card pops) instead of constant hard cuts.
- Visual grammar (read from the frames):
  - light paper pitch panel inset on a dark navy grid page; the panel is
    near-square and centered; big bright titles live in the page band
    above it (~90 px at 1080p equivalent);
  - dots large (~28-34 px), name labels ~24-30 px, dashed-yellow links,
    red tactical shapes;
  - footage is support: an inset rounded panel (f_104s), yellow quote
    subtitles burned on clips (f_8s), inset stat cards over footage
    (f_24s);
  - still photos (club crest + coach) as breathing beats (f_40s, f_72s,
    f_488s); the episode closes on a full-bleed still;
  - diagrams evolve INSIDE a chapter rather than by cuts.
- Pacing: a tactical point runs 20-45 s; the visual holds while the voice
  finishes the point; goal clips are 10-20 s beats.

## Applied to ep02 v2

- 12 voice-led rows (measured VO 180.5 s + pads = 192.68 s = 3:12.7).
- Light style block: page #101A2E, panel #F2F2EC, title 40 pt (~56 px) in
  the page band, labels 18 pt (~25 px), dot radius 1.7 units, away dots
  #1C4A8A at full opacity.
- Goal strikes frame-mapped on the verified fan VOD (DKRQ8SH6cZU):
  gordon in_start 5.5 (dribble t7, net t9-10), jesus in_start 16.0 (net
  at 20), kounde in_start 31.5 (net at 35); each 6 s of source -> 12 s
  slowed output; voice rides over the slowed bed at volume 1.5 vs 0.30.
- Cards: xg_comparison on the gordon clip, big_chances and rodri_rating
  pop on diagram rows (the f_24s treatment), finalboard closes
  (the f_488s treatment).

## Not transferred (honest list)

- inset footage panels (our fan clips go full-frame);
- burned quote subtitles (no transcript pipeline for this match);
- rights-cleared still photos (none available locally for Barca);
- the 8:21 length (brief 20r caps the episode at 3-4 min).