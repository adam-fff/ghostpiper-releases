# GhostPiper Release Mirror

This repository exists for **one reason**: to expose a small, public version-manifest JSON file (`latest.json`) so that deployed [GhostPiper](https://github.com/adam-fff/GhostPiper) instances can poll for new releases without needing GitHub authentication.

The source project [`adam-fff/GhostPiper`](https://github.com/adam-fff/GhostPiper) is private. Its `/releases/latest` GitHub API endpoint therefore 404s for any unauthenticated caller, including the in-process HTTP client inside GhostPiper itself. This mirror is the workaround.

## How it works

A GitHub Action in the private GhostPiper repo (`.github/workflows/publish-version.yml`) triggers on `release: published` and force-pushes a fresh `latest.json` into this repo, authenticated via an SSH deploy key.

```
[private]                                [public]
GhostPiper ─── release published ───→ ghostpiper-releases
                                          │
[any client]                              │
ghostpiper web  ←── HTTP GET ─────────── latest.json
```

Clients hit `https://raw.githubusercontent.com/adam-fff/ghostpiper-releases/master/latest.json` directly — no GitHub auth, no per-IP API rate limit, fast CDN-cached.

## Manifest shape

```json
{
  "tag_name": "v0.6.1",
  "name": "v0.6.1 — Animated splash, filter-cycle headers, compliance-refs fix",
  "html_url": "https://github.com/adam-fff/GhostPiper/releases/tag/v0.6.1",
  "published_at": "2026-05-03T13:41:28Z",
  "body": "Release notes markdown..."
}
```

The `html_url` points at the **private** GhostPiper repo's release page. Anyone authenticated and authorised on that repo can read it; everyone else will hit GitHub's login wall. That's expected — the only people who run GhostPiper are people who have access.

## Manual edits

**Don't.** The workflow force-pushes on every release. Any manual edit you make here will be overwritten on the next release publication. If you need to fix something, change the workflow in the source repo and re-publish.
