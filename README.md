# kami-eizo-timeline

Portable `.cljc` EDL/timeline data model — the video-domain analog of what
Premiere Pro / DaVinci Resolve / Avid Media Composer store internally as an
Edit Decision List. Part of the `kami-eizo` (映像, video) family defined by
ADR-2607121400.

- Frame-accurate integer timecode over an explicit rational timebase
  (`kami.eizo.timeline.timecode`) — no float seconds, no native Clojure
  ratios (ClojureScript has no ratio type). Includes correct SMPTE
  drop-frame timecode math for 29.97/59.94fps.
- Track / clip / transition / effect-stack / marker / timeline data model
  (`kami.eizo.timeline`) with a referential-integrity validator: unbridged
  clip overlaps, transitions longer than an adjacent clip, out-of-bounds
  markers, and drop-frame flags on ineligible frame rates are all rejected.

## v0 scope — what this is NOT

- No interchange-format export (CMX3600 / OpenTimelineIO / FCPXML) yet.
- No render execution — this is pure data + validation. `kami-eizo-*`
  render/codec execution lives in `utsushi`; `douga` builds ffmpeg
  command-vectors and is expected to be rewired onto this IR in a later
  wave.
- Effect and plugin *implementations* are out of scope — a clip/track just
  carries an ordered list of opaque effect-instance references with
  enable/bypass state.

## Usage

```clojure
(require '[kami.eizo.timeline :as tl]
         '[kami.eizo.timeline.timecode :as tc])

(def c1 (tl/clip {:id :v1 :source-id :src-a :source-in 0 :source-out 120 :timeline-start 0}))
(def c2 (tl/clip {:id :v2 :source-id :src-b :source-in 0 :source-out 90 :timeline-start 105}))
(def tr (tl/transition {:id :t1 :type :dissolve :from-clip :v1 :to-clip :v2 :duration 15}))
(def track (tl/track {:id :video-1 :type :video :clips [c1 c2] :transitions [tr]}))
(def timeline (tl/timeline {:timebase tc/ntsc-2997-df :tracks [track]}))

(tl/validate-timeline timeline)
;; => {:valid? true :errors []}

(tc/format-timecode 1800 tc/ntsc-2997-df)
;; => "00:01:00;02"   (drop-frame: labels :00/:01 skipped at the minute-1 boundary)
```

## Test

```bash
clojure -M:test
clojure -M:lint
```

## License

Apache-2.0
