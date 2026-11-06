# Shotcat skill for Claude Code

Teach Claude Code (and Codex) to make **App Store preview videos, App Store and Google Play
screenshots and vertical social videos** (Reels, TikTok, Shorts) of the app in your repo, with
3D devices, captions, taps, hook clips and voiceovers, and to publish them to App Store Connect,
YouTube, TikTok and Instagram. It drives the [Shotcat CLI](https://shotcat.app/cli); everything
renders on your machine.

> "Record the onboarding in the Simulator and make an App Store preview and a TikTok out of it."

## Install

**Claude Code (plugin):**

```
/plugin marketplace add maratmuslimov/shotcat-skill
/plugin install shotcat@shotcat
```

Claude then uses the skill on its own when you ask for previews, screenshots or social videos,
or you can call it with `/shotcat:shotcat`.

**With the Shotcat CLI** (Claude Code and Codex, keeps the skill updated with `shotcat update`).
Install the CLI as described on [shotcat.app/cli](https://shotcat.app/cli), then:

```bash
shotcat skill install            # ~/.claude/skills and ~/.codex/skills
shotcat skill install --project  # only this repo: .claude/skills and .agents/skills
```

**By hand:** copy `skills/shotcat/SKILL.md` to `~/.claude/skills/shotcat/SKILL.md`.

## What you need

- The Shotcat CLI (`shotcat doctor` checks ffmpeg, Chrome and the GPU).
- A free Shotcat account for renders (`shotcat login`, a device code you approve in the browser).
  Free renders carry a small shotcat.app tag on the device; a Launch Pack or Lifetime removes it.
- For recordings: Xcode's iOS Simulator or an Android emulator/phone with `adb`.

## What it runs and where data goes

This plugin is one skill: instructions, no code, hooks or MCP servers. Following it, Claude runs
commands in your terminal, with your usual permission prompts:

- **The Shotcat CLI** (`shotcat …`). The skill doesn't install it: if it's missing, Claude asks you
  to install it yourself from [shotcat.app/cli](https://shotcat.app/cli). `shotcat update` reinstalls it; the skill asks you first.
- **Recording tools** you already have: `xcrun simctl` (iOS Simulator), `adb` (Android), `ffprobe`.

Data: renders happen on your machine. The CLI talks to Shotcat's servers (studio.shotcat.app) to
sign in (email and a device code), to check your plan, and, only when you ask, to upload a project
for Catpilot or to publish a video to YouTube, TikTok or Instagram through the accounts you
connected. The skill tells Claude to ask before posting anything publicly and never to print or
commit CI tokens. See the [privacy policy](https://shotcat.app/privacy).

## Links

[shotcat.app](https://shotcat.app) · [CLI](https://shotcat.app/cli) · [Studio](https://studio.shotcat.app) ·
[Privacy](https://shotcat.app/privacy) · support@shotcat.app

## License

MIT
