---
name: shotcat
description: Make App Store preview videos, App Store / Google Play screenshots and vertical social videos (Reels, TikTok, Shorts) of the app in this repo with the Shotcat CLI, with 3D devices, captions, taps, hook clips and voiceovers. Publish them to App Store Connect, YouTube, TikTok and Instagram, or set up Catpilot to post on a schedule. Use when the user asks for app previews, store screenshots, a launch video, marketing or social videos of their app, or to post them.
---

# Shotcat: store assets and social videos of this app

Shotcat turns screen recordings and screenshots of an app into store previews, screenshot sets
and social videos with 3D devices. You drive it with the `shotcat` CLI; a project is one
`shotcat.json` next to the media. Everything renders locally in headless Chrome.

Check the CLI first: `shotcat --version`. If it's missing, ask the user to install it themselves from
https://shotcat.app/cli (the installer is a script they run in their terminal; don't run it for
them). `shotcat doctor` checks ffmpeg, Chrome, GPU and sign-in.

## Workflow

1. **Get footage of the app.** Prefer real recordings of the app running over mockups.
   - Easiest, once a project exists (step 2): `shotcat capture --seconds 8` records the booted iOS
     Simulator or Android emulator/phone and adds the file as the next screen;
     `shotcat capture --screenshot` takes a screenshot. `--ios` / `--android` and
     `--device <udid|serial>` pick one when several run; `--out flow.mp4` names the file
     (default `captures/` next to `shotcat.json`), `--no-add` only saves it, `--hook` adds a
     hook clip. Without `--seconds` a person at a terminal stops it with Enter; you (no terminal)
     get 10 s, so always pass `--seconds`, and start driving the app right after you run it.
     Android records at most 180 s. `--json` gives the path and the new screen id.
   - By hand, iOS simulator: boot the app, then `xcrun simctl io booted recordVideo --codec=h264 --force flow.mp4`
     (Ctrl-C stops it). Screenshot: `xcrun simctl io booted screenshot home.png`.
     Set a clean status bar first: `xcrun simctl status_bar booted override --time 9:41 --batteryState charged --batteryLevel 100 --cellularBars 4 --wifiBars 3`.
   - Android emulator/device: `adb shell screenrecord /sdcard/flow.mp4` then `adb pull /sdcard/flow.mp4`;
     `adb exec-out screencap -p > home.png`.
   - Drive the app with UI tests or `simctl`/`adb` input while recording. Keep each recording to
     one flow, 4–12 s, no loading spinners or debug overlays, realistic demo data (no lorem ipsum,
     no personal data).
   - Note the moments you tap (seconds from the start, position as fractions of the screen):
     they become visible taps with `shotcat taps`.
2. **Start a project:** `shotcat init --template hero-spin --name "<App>" --description "<what the app does, for whom>"`
3. **Add screens in order:** `shotcat add onboarding.mp4 plan.mp4 stats.png` → `shotcat screens`.
   Trim dead time: `shotcat trim plan 0.4-6.8`. Taps: `shotcat taps plan '[{"t":1.2,"x":0.5,"y":0.82}]'`
   (`hold` seconds for long presses, `"to":{"x":…,"y":…}` for swipes). A tap's `t` is in seconds
   of the **whole, untrimmed** recording, even after `trim`; taps in cut parts drop out.
4. **Pick a template:** `shotcat templates` lists them. Good defaults: `hero-spin` (one device,
   headline), `walkthrough` (a caption per screen), `feature`, `orbit`, `story`, `dive`;
   `duo` / `cascade` for several devices; `screenshot-set` / `panorama` for store screenshots.
   `shotcat use <id>` switches and keeps the look.
5. **Write the captions.** `shotcat schema` prints the template's props as JSON Schema; set them
   with `shotcat set title "…" subtitle "…"` and `shotcat set nextCaptions.0.title "…"` (one per
   later screen; `walkthrough` has `steps.0.title` per screen instead). An unknown path is an
   error that lists the template's props. Look: `shotcat set scene aurora sceneColor "#336cfa" captionStyle words`.
   Write like an App Store page: a benefit, not a feature list; 2–6 words a headline; no
   "best"/"#1" claims, no prices, no mentions of other platforms.
6. **Look before rendering.** `shotcat preview --format social-vertical` writes a few JPEG
   stills along the timeline and prints their paths. **Open and look at them** (they're images):
   check text isn't cut off or covering the device, the right screen is showing, nothing looks
   broken. Fix and preview again. Previews need no sign-in and are watermarked.
7. **Render:** `shotcat render --pack app-store` (App Store previews and screenshots),
   `--pack google-play`, `--pack social`, or `--formats social-vertical,appstore-preview-iphone`.
   `shotcat formats` lists all. Files land in `shotcat-out/`. Rendering needs `shotcat login`
   (a device code the user approves in the browser: tell them the URL and code it prints).
   Free accounts get the shotcat.app tag on the device; a Launch Pack or Lifetime removes it.
   Check videos with `ffprobe` if in doubt.

## Social videos: hook, voiceover, music, end card

Only the `social-*` and `web-*` formats get these; store previews must show the app alone
(Apple rejects anything else), so the same project renders both.

- **Hook clip** (the first 1–3 s that stop the scroll): a short real clip, e.g. someone reacting,
  the problem in real life. `shotcat add --hook reaction.mp4`, then
  `shotcat hook reaction --text "I kept forgetting my runs" --layout full --seconds 2`
  (`split`: clip on top, text below on the scene). `shotcat hook off` removes it.
- **Voiceover:** `shotcat say "Okay so. I kept skipping runs. [sighs] Then I tried this."`
  records one and shows its words as subtitles. `shotcat voices` lists the voices for
  `--voice <id>` (pick one that fits the video's tone); `--locale de` for other languages. Audio
  tags like `[laughs]`, `[sighs]`, `[whispers]`, `[short pause]` are acted, never shown (two at
  most). Or bring a file: `shotcat voiceover vo.m4a [--words words.json]`. ~2.7 words a second.
- **Write it like a creator, not an ad.** The opening line is about the viewer's life and lands in
  the first 1.5 s ("You're storing tomatoes wrong", "POV: it's 9pm and…"), never the app's name.
  Short spoken sentences, contractions. Avoid ad-speak: Introducing, Meet, Say goodbye to,
  game-changer, seamless, effortless, unlock, elevate, "Download … now". The ending answers what
  the opening asked. Formats that work: POV, "stop doing this", myth vs fact, "things I wish I
  knew", before/after, unpopular opinion, "not gatekeeping this", a tiny challenge.
- **End card:** `shotcat outro "Stride" --cta "Link in bio"` ends the video on the app's name, a
  call to action and a screenshot (`--image <screen id>`, the first still by default), instead of
  stopping on the last screen. Put the voice's last line ("It's called Stride.") over it: the
  card shows for `--seconds` (2.5) after the template, or from `--start <s>`.
- **Music:** `shotcat music track.mp3 --volume 0.15` (only music the user has the rights to).
  Catpilot brings its own (made for it, no copyright claims).

## Publishing

- YouTube / TikTok / Instagram, with the accounts the user connected in the studio
  (studio.shotcat.app → Export → Publish): `shotcat publish youtube shotcat-out/hero-spin-social-vertical.mp4 --caption "Title line\nDescription #hashtags" --privacy public`.
  TikTok `--privacy draft` sends it to the TikTok inbox. Ask before posting publicly.
- App Store Connect and Google Play uploads happen in the studio's Publish step.
- **Catpilot** (subscription): the server plans, renders and posts a new video on a schedule.
  `shotcat catpilot create --to youtube,tiktok,instagram --days mon,wed,fri --times 17:00 --locales en --voice --app-name "Stride"`
  pushes this project to the cloud and starts the campaign; `shotcat catpilot status`, `runs`,
  `pause <id>`, `run <id>` (post one now). Add hook clips to the project before pushing: the
  planner picks among them. `--tips` opens each video with a tip or fact about the app's topic
  over stock footage, then the app. `--app-name` is the name videos and posts use (default: the
  project's name). Each video gets a new format, a fitting voice and background music, and ends
  on an end card; `--voice-id <id>` keeps one voice, `--no-music` leaves music out (e.g. to add a
  trending TikTok sound by hand).
- One app with several projects (other features, or screens in other languages: set each one's
  language with `shotcat describe "…" --locale ru`): `shotcat push` each, then
  `shotcat catpilot create --project-ids <id1>,<id2> --locales en,ru …`. Each video takes one,
  in the slot's language when one is; the ids are in `shotcat catpilot status --json` (projects).

## CI

- Sign-in without a browser: the user runs `shotcat login --print-token` once (a separate
  session; it prints the token and saves nothing) and stores it as the CI secret
  `SHOTCAT_TOKEN`. When set, it wins over any saved sign-in. Never print, log or commit it; only
  run `--print-token` when the user asks for a CI token.
- GitHub Actions: the workflow installs the CLI with the installer from https://shotcat.app/cli,
  then runs `shotcat render`; the Shotcat GitHub Action does the same and uploads the files as an
  artifact. fastlane: the
  `fastlane-plugin-shotcat` gem's `shotcat` action renders and copies screenshots into
  `fastlane/screenshots/<locale>/` (deliver) and `fastlane/metadata/android/<locale>/images/`
  (supply).
- Without a GPU (most CI runners) each frame takes seconds: render the still formats
  (`appstore-screenshot-*`, `play-*`) there, videos on a Mac or a GPU machine.

## Tips

- `--json` on any command prints machine-readable output.
- `shotcat update --check` says whether a newer CLI is out; `shotcat update` installs it and
  refreshes this skill (ask the user first, it changes their install).
- `shotcat show` prints the project; `shotcat show title` one prop.
- Up to 10 screens play; videos set the length (images hold ~3.5 s). App Store previews are
  15–30 s: shorter ones hold the last frame, longer ones are cut.
- Renders use the GPU through Chrome; on a Mac that's fast. On Linux without a GPU each frame
  takes seconds: use `--frames 0-59` for a quick check, or preview stills.
- If something fails, `shotcat doctor` and rerun with the error in mind; don't edit
  `shotcat.json` by hand unless a command can't do it.
