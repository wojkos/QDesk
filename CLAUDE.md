# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

QDesk is a single-file, no-build web app for running a live Q&A queue at events: add or import questions, generate TTS audio for each via ElevenLabs, and play them back in order during the event ("Ask" marks a question as read). Users can export their question list as JSON to reuse at future events.

There is no build step, no package.json, no bundler, and no test runner. All logic lives in `index.html` (inline `<style>` + inline `<script>`, vanilla JS, no framework).

## Running locally

```
python -m http.server 8000
```
Then open `http://localhost:8000`.

There's no lint/test/build command — validate changes by loading the page in a browser and exercising the flow manually (add/generate/play a question).

## Architecture

Everything is in `index.html`, structured as one IIFE:

- **State**: a single `state` object (questions, voices, API key, editing/playing ids) persisted to `localStorage` under the `qdesk_*` keys. Audio blobs themselves go in IndexedDB (`qdesk` DB, `audio` store), keyed by question `id`, since they're too large for localStorage.
- **Initial questions**: startup loads 2 sample questions for the user to edit, delete, or build upon. Persisted questions restore from localStorage on refresh.
- **Render loop**: one `render()` function does a full innerHTML re-render of `#questions-list` from `state.questions`. There's no diffing/vdom — mutate `state`, call `saveQuestions()`, call `render()`. All row interactions (up/down/bump/delete/generate/ask/restore) are handled via a single delegated click listener on `#questions-list` keyed off `data-action`/`data-id` attributes.
- **ElevenLabs integration**: the API key is stored in localStorage and sent directly from the browser to `api.elevenlabs.io` (`/v1/voices` to list voices, `/v1/text-to-speech/{voiceId}` to synthesize with `eleven_multilingual_v2`) — there is no backend/proxy. "Generate All Missing" runs sequentially, not in parallel, to avoid hitting ElevenLabs rate limits before a live event (see the `ponytail:` comment in `index.html`).
- **Export/Import**: "Export JSON" downloads the current question list (text/voiceId/header). Users can then upload previously exported JSON via "Import JSON" to reload questions at a later event.

## Brand/styling

`context/brand.md` documents the DXC 2025 brand palette/typography this UI's CSS variables (`--bg`, `--amber`, `--red`, etc. in `index.html`) are derived from. Consult it before changing colors or fonts.

## Sensitive files

`.env` and `context/` are gitignored — don't unignore or commit them.
