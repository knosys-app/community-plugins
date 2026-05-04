# Knosys Community Plugins

The plugin registry for [Knosys](https://github.com/knosys-app/mono). The
desktop app reads `index.json` from this repo to populate the **Browse**
tab in `Settings → Plugins`.

> **As of Knosys 2.0**, plugins must declare `knosysApi: 2` in their
> manifest. v1 plugins are surfaced as "needs update" in the desktop
> Settings panel and never executed. See
> [`docs/redesign/05-plugin-api-v2.md`](https://github.com/knosys-app/mono/blob/main/docs/redesign/05-plugin-api-v2.md)
> in the main repo for the full API spec.

---

## Repository layout

```
community-plugins/
├── index.json   ← registry index (consumed by the desktop installer)
├── README.md    ← this file
└── LICENSE
```

The desktop fetches:

```
https://raw.githubusercontent.com/knosys-app/community-plugins/main/index.json
```

Per-plugin metadata lives inline in `index.json` — there is no separate
`plugins/<id>.json` file at this time.

---

## `index.json` schema (v1)

```json
{
  "schemaVersion": 1,
  "fetchedAt": "2026-05-03T00:00:00Z",
  "plugins": [
    {
      "id": "knosys-weather",
      "name": "Weather",
      "description": "Weather forecasts for your location and saved locations.",
      "author": "Knosys",
      "version": "2.0.0",
      "knosysApi": 2,
      "categories": ["weather", "data"],
      "homepage": "https://github.com/knosys-app/atlas-weather",
      "repository": "https://github.com/knosys-app/atlas-weather",
      "downloadUrl": "https://github.com/knosys-app/atlas-weather/releases/download/v2.0.0/knosys-weather-2.0.0.tgz",
      "downloadSha256": "abc123...",
      "permissions": ["network", "storage", "core:locations"],
      "requiredApiKeys": [
        {
          "id": "openweather",
          "name": "OpenWeather API Key",
          "description": "Used to fetch current weather and forecasts.",
          "signupUrl": "https://openweathermap.org/api",
          "required": true
        }
      ]
    }
  ]
}
```

### Required fields

| Field | Type | Notes |
|---|---|---|
| `id` | string | Must match the plugin's manifest `id`. Kebab-case. Conventionally `knosys-*`. |
| `name` | string | Human display name. |
| `description` | string | One-sentence summary shown on the Browse card. |
| `author` | string | Author display name. |
| `version` | string | Semver. Must match the plugin's manifest `version`. |
| `knosysApi` | `2` | Discriminator. Older API levels are not supported by Knosys 2.x. |
| `downloadUrl` | string | Direct URL to a `.tgz` tarball (typically a GitHub Release asset). |
| `downloadSha256` | string | Hex sha256 of the tarball bytes. The desktop refuses installs that fail this check. |

### Optional fields

| Field | Notes |
|---|---|
| `categories` | Array of category slugs (`weather`, `maps`, `health`, `aviation`, `media`, `data`, `tracking`, `planning`, `ingest`). Drives the Browse-tab category pills. |
| `homepage` | Public-facing project page. Often the GitHub repo. |
| `repository` | Source repo URL. |
| `icon` | Path within the tarball, e.g. `assets/icon.png`. |
| `iconUrl` | Absolute URL to a hosted icon (used by the Browse list before install). |
| `tagline` | Short marketing line shown on the hero card. |
| `screenshotUrls` | Array of absolute URLs to screenshots. |
| `featured` | Boolean — surfaces the plugin in the "Featured" rail. |
| `verified` | Boolean — currently for first-party plugins; signing replaces this in v3 (see plan §14). |
| `permissions` | Array — preview the permissions the user will be granting. Mirrors the plugin's manifest. |
| `requiredApiKeys` | Mirrors the plugin's manifest. |
| `downloads` | Cumulative install count, when known. |
| `updatedAt` / `createdAt` | ISO timestamps. |
| `readmeMarkdown` | Inline README content shown on the plugin profile. |

---

## How to submit a plugin

1. **Build a v2 manifest.** Your `manifest.json` must declare:
   ```json
   {
     "knosysApi": 2,
     "id": "knosys-my-plugin",
     "name": "My Plugin",
     "version": "1.0.0",
     "description": "...",
     "author": { "name": "Your Name", "url": "https://github.com/you" },
     "compat": { "knosys": ">=2.0.0 <3.0.0" },
     "permissions": ["storage"],
     "main": "main.js"
   }
   ```
   `compat.kdl` is required only if your plugin imports anything from
   `Shared.kdl.*`.

2. **Publish a tarball.** Tag a release in your plugin repo and attach
   a `.tgz` containing `manifest.json` + `main.js` + optional
   `styles.css` + any assets. The expected layout inside the tarball is
   either flat (files at root) or wrapped in `package/` (npm pack
   convention) — the installer handles both.

3. **Compute the sha256.** Run `shasum -a 256 your-plugin-1.0.0.tgz` and
   copy the hex digest.

4. **Open a PR to this repo.** Add an entry to `index.json` with the
   fields above. Bump `fetchedAt` to the current ISO timestamp.

5. **Reviewer responsibilities.** PRs are reviewed for:
   - Manifest validity (schema fits, semver ranges parse).
   - That `downloadUrl` actually exists and serves a tarball.
   - That `downloadSha256` matches the served tarball.
   - The plugin's actual code is sane (no obvious abuse, no unrequested
     permissions in practice).

---

## Updating an existing plugin

Bump `version` (and the matching `version` in your plugin manifest), cut
a new release with the new tarball, update `downloadUrl` +
`downloadSha256`, bump `fetchedAt`. The desktop's update checker
periodically diffs installed versions against the registry and surfaces
"Update available" in `Settings → Plugins`.

---

## Removing a plugin

Delete the entry from `index.json`. Already-installed copies on user
desktops keep working (registry only drives Browse + Updates). To force
existing users to uninstall, ship a new version that throws or
otherwise refuses to activate — the registry can't push uninstalls.

---

## What's coming in v3

The current schema verifies *integrity* (the file you got matches what
the registry described) but not *authenticity* (the file the registry
described was built by the plugin author you trust). v3 adds Sigstore
signatures, GitHub OIDC publisher identity binding, and a `revoked`
field for compromised versions. Full plan in
[`docs/redesign/05-plugin-api-v2.md`](https://github.com/knosys-app/mono/blob/main/docs/redesign/05-plugin-api-v2.md)
§14.

Until v3, hand-review every PR.

---

## License

The contents of this repo are MIT-licensed. Each listed plugin carries
its own license — check the linked `repository` for details.
