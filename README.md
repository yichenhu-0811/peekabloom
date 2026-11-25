# Peekabloom — *a little peek, a little bloom*

A pair of connected ambient clocks. Each clock face is a small island seen from above. The clock tells real local time. If you stand in front of it for about three seconds, the other person's character appears on the island, and a bud starts growing on *their* clock.

The whole prototype is one file, `index.html`, with no build step and no dependencies. It works offline.

## Run it

```bash
cd peekabloom
python3 -m http.server 8000      # any static server works
```

| URL | What it is |
|---|---|
| `http://localhost:8000/#demo` | Developer view: both clocks side by side, with every control |
| `http://localhost:8000/parents/` or `/#parents` | The parents' clock (China, sage). It shows **Yichen**, the character *Pip* |
| `http://localhost:8000/me/` or `/#me` | Yichen's clock (Philadelphia, peach). It shows **Mom & Dad**, the characters *the Tangyuans* |

The clock pages hide all UI. To open the developer panel, press **d**, add `?dev=1`, or tap the top-left corner of the screen three times. Pressing **p** turns the simulated "someone is standing here" signal on and off.

## Try the interactions (in `#demo`)

1. **Presence.** Press **Approach & stay** (or turn on the camera) and wait three seconds. Nothing happens on a quick glance. While you wait, the flowers sway faster and a thin ring traces the rim. Then the hands wiggle and turn toward where the other person is: **their house** when they're home, **the pond and woods** when they're out. There are no icons. When they're home, the windows glow and the chimney smokes; when they're out, the house is dark and quiet. Then the other person's character appears and does their current activity. The sky changes to *their* time of day. If they are resting at home while it is night where they are, the character is asleep.
   In Ambient Mode the flowers grow across the **whole island**. Their spots are chosen so that a few flowers start far apart, and new ones fill the gaps as the garden grows.
2. **Reciprocal flower.** The same visit grows one bud on the *other* clock: it plants a new bud, or moves the oldest bud one stage closer to blooming. One sustained visit counts once, however long it lasts. Flowers never die. The garden shows no counts, streaks or notifications. A recently grown bud just glows softly for a few hours.
3. **Activity.** Change Home/Out and Rest / Walk / Active / Casual. Each combination has its own scene:

   | | Rest | Walk | Active | Casual |
   |---|---|---|---|---|
   | **Home** | reading by the house (asleep at night) | pottering around | hopping / morning exercise | watering the garden plots |
   | **Out** | Pip fishing at the pond; the parents with tea on the bench | walking around the pond | cycling around the pond (a tandem for the parents) | strolling with a shopping bag |

4. **Togetherness.** Press **Both approach & stay**. The two clocks start 1.4 seconds apart on purpose, to show that both people only need to be there at the same time. Then the rim glows, the visiting character steps off the little dock, the two greet each other with a hop, and a luminous **moonbloom** grows between them. Afterwards they sit together, a butterfly circles them, and every few seconds they do a little dance. Each simultaneous session adds exactly one moonbloom to each garden. When one person leaves, the visitor waves and walks off.
5. **Time Machine.** While a clock is in Presence Mode, tap the small ↺ under the digital time. The hands spin backwards, a 24-hour ring appears (their midnight at the top, noon at the bottom, colored by home and out), and a golden hand plays back the other person's last day while the character acts out each part. You can drag the ring to scrub. The labels follow the clock's language (Chinese on the parents' clock).
6. **Sound.** Turn on **Sound** to hear an original music-box motif in Togetherness Mode and soft chimes when a bud grows. On iPad, audio starts after the first touch.

## Simulated vs. live

| Mode | How | What's real |
|---|---|---|
| **Local simulation** (default) | Nothing to set up | State is kept in `localStorage` and shared between tabs of **one browser** (BroadcastChannel). Open `#parents` and `#me` in two tabs to see them talk. This is **not** cross-device. |
| **Live (Firebase)** | Add `?db=https://<project>-default-rtdb.firebaseio.com` (and optionally `&auth=<token>`) | Presence, sessions, status, history and both gardens sync through the Firebase Realtime Database REST and streaming API, with no SDK. **I wrote this against the documented API but haven't tested it against a real Firebase project.** |

Status (home/out, activity) comes **only** from the developer controls for now. The demo "yesterday" timeline is generated and marked `demo: true`. The local mode adds it automatically; the Firebase mode never does.

Logic that matters for sync:
- Presence carries a random `session` id and a heartbeat every 5 seconds. A clock whose heartbeat is older than 25 seconds counts as gone (for example, a sleeping iPad or a closed tab).
- Togetherness key = the two session ids, sorted. The moonbloom id comes from that key, so the special flower is created once per simultaneous session, even if both clocks see the overlap.
- Flowers are written as individual nodes (`gardens/<clock>/<flowerId>`), so the two clocks never overwrite each other's gardens.

## Deploying on two iPads

1. **Host it.** Put the folder on any static HTTPS host (Netlify, Vercel, GitHub Pages, Cloudflare Pages). `/parents/` and `/me/` are copies of `index.html`, so those paths work without server rewrites.
2. **Firebase.** Create a Realtime Database. For the exhibit, the simplest rules allow read and write on `/peekabloom` only. Before leaving it running in someone's home, add auth (an anonymous auth token or a database secret passed as `&auth=`).
3. **Each iPad.** Open `https://<host>/parents/?camera=1&db=…` (or `/me/?camera=1&db=…`) in Safari and choose **Share → Add to Home Screen**, so it launches full screen and black around the circle. Then:
   - Turn on **Settings → Accessibility → Guided Access** to lock the iPad to the page.
   - Set **Display → Auto-Lock → Never** and keep the iPad on power. The page also requests a Screen Wake Lock (Safari 16.4 and later).
   - Use `?scale=0.92&ox=0&oy=-12` to fit and center the circle in your 3D-printed opening. `scale` is a fraction of the shorter screen side, and `ox`/`oy` are pixel offsets.
4. **Time zone.** Each clock shows its owner's time zone (Asia/Shanghai, America/New_York), so it's right even when you test both on one laptop. Use `?tz=device` to follow the iPad's own clock instead.

## Camera presence on the iPad (built)

The page uses the iPad's **front camera** with MediaPipe face detection. It runs in the browser, on the device. Frames are never uploaded, recorded or stored. The detector outputs one value: *is a face close to the clock and roughly facing it?* That value goes into the `PresenceDetector`, which applies the **3-second dwell** (so a glance doesn't count) and a 2.5-second grace period (so turning your head doesn't end the moment).

- Turn it on with `?camera=1` (for example `/parents/?camera=1&db=…`). In the developer panel, use **Use camera**.
- "Close" means the face is at least 14% of the camera frame's width, which is roughly within 1–1.5 m. Tune it with `?minface=0.10` (farther) or `0.2` (closer). Add `?camdebug=1` (or press **Preview**) to see a small preview with the face box while you calibrate. Remove it for the exhibit.
- "Facing" means the nose sits roughly between the eyes, so someone walking past in profile doesn't count. This is **not** eye-tracking: it detects a person standing in front of the clock and facing it.
- A face has to be seen in 3 of the last 4 checks (about 6 checks per second), which filters out flicker.
- Files: the MediaPipe library and WASM are included in `vendor/mediapipe/`. The face model (`blaze_face_short_range.tflite`, about 230 KB) is downloaded from Google the first time it runs. To run fully offline, download it from `https://storage.googleapis.com/mediapipe-models/face_detector/blaze_face_short_range/float16/1/blaze_face_short_range.tflite` and put it in `vendor/mediapipe/`. The page looks there first.
- Requirements: **HTTPS** (Safari only allows the camera on secure pages) and camera permission (Settings → Safari → Camera → Allow, or accept the prompt once). In the 3D-printed frame, leave a small opening in front of the iPad's front camera. In landscape on recent iPads it sits on the long edge, so place the circle so the camera isn't covered.
- Not tested here: I checked that the camera stream starts and that the library loads from `vendor/`, but this environment can't download the face model or use a real camera. The first real test needs to be on the iPad.

Alternatives if the camera is a problem (privacy, lighting, frame design): an mmWave presence sensor + ESP32 that sends `near`/`far` over `?sensor=ws://<ip>:81`. That adapter is also in the code (also untested with hardware).

## Future phone integration (not built)

The data model is ready: write to `status/<person>` and `history/<person>/<ts>`.
- **Home / Out:** an iOS Shortcuts personal automation, "When I arrive / leave Home", runs **Get Contents of URL** with a `PUT` to `…/peekabloom/status/me.json`, using a body like `{"place":"home","activity":"resting","ts":…}`.
- **Activity:** iOS doesn't expose live CoreMotion to Shortcuts. Options are a tiny companion app, or rough proxies (Focus modes, CarPlay/Bluetooth connection, Health workouts starting).

## Files

- `index.html`: everything (themes, characters, garden, presence logic, Togetherness, Time Machine, hubs, developer panel)
- `parents/index.html`, `me/index.html`: copies of `index.html` so the path routes work on any static host
