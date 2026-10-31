# WLOC Virtual Location · Web Console (worker)

**English** | [简体中文](./README.md)

This directory contains the WLOC project's web console and map-link parsing API: pick a target location on the map, write it to your proxy app in one tap, and change your iOS location — no jailbreak required. For the module itself, subscription URLs, and the full tutorial, see the [README](../README.md) in the repository root.

- 🌐 Live demo: https://wloc-pages.pages.dev/
- 📦 Repository: https://github.com/Armorys/wloc-ios-virtual-location (this `worker/` directory)

## Features

- 🗺️ **Map picker**: a Leaflet-based map interface — tap the map to set your target location
- 🎯 **Jitter radius**: set an offset radius (in meters); every location fix is randomly offset around the target point, `0` = off
- ⭐ **Saved locations**: bookmark frequent places and reuse them anytime
- 🔗 **Map link parsing**: paste links from Apple Maps, Google Maps, Amap (Gaode), or Baidu Maps, or plain coordinate text; short links are expanded automatically
- 🔍 **Place search**: search for a place by name and jump straight to it
- 💾 **Persistent write**: after picking a location, tap "Save to device" to write the target coordinates as `wloc_settings` into your proxy app, persisted on the device

## How It Works

The console page does not change your location by itself. It writes your chosen target coordinates as `wloc_settings` data, which the **WLOC location module** inside iOS proxy apps (Surge, Quantumult X, Shadowrocket, etc.) reads; the module intercepts Apple location service requests and returns the virtual coordinates.

For the module to take effect, all of the following must be in place (the console page also shows these hints at the top):

1. The WLOC location module is installed and enabled
2. MITM is enabled and the certificate is trusted
3. The MITM hostname list includes `gs-loc.apple.com`
4. Your current network traffic goes through the proxy

## Deployment

### Cloudflare Pages (recommended)

1. Connect the Armorys/wloc-ios-virtual-location repository in Cloudflare Pages, with the project root directory set to `worker`
2. Set the build output directory to `dist` (or just use the included `wrangler.pages.jsonc` in this directory)
3. After deployment, `functions/[[route]].js` automatically serves as the Pages Functions entry point; both the console page and the `/api/parse` endpoint are available

The live instance is hosted on Cloudflare Pages: https://wloc-pages.pages.dev/

### Local development

```bash
npm install
npm run pages:dev   # local preview at http://localhost:8787
```

### Standalone Worker version

The `wloc-worker.js` file in this directory is a standalone Worker entry point that can be deployed to Cloudflare Workers on its own (see `wrangler.jsonc` in this directory for its deploy config):

```bash
npm run deploy
```

## API

### GET /api/parse

Parses a map link or coordinate text into latitude/longitude. Mainly intended for scenarios such as iOS Shortcuts.

**Query parameters**

| Parameter | Required | Description |
| --- | --- | --- |
| `u` | Yes | A map link (Apple Maps / Google Maps / Amap / Baidu, including short links) or plain `latitude,longitude` text |
| `format` | No | `json` returns JSON; when omitted, returns a plain-text fragment |
| `cs` | No | `gcj` = force GCJ-02 conversion; `none` = force no conversion; omitted = auto-detect from the link source |

**Coordinate system rules**

- Coordinates from Amap and Apple Maps (mainland China) are GCJ-02 and are converted to WGS84 by default before being returned (iterative inverse calculation, sub-meter accuracy)
- Coordinates outside China are returned unchanged
- Returned latitude/longitude keep 6 decimal places

**Response example (JSON)**

```text
GET /api/parse?u=31.230416,121.473701&format=json
```

```json
{ "lat": 31.230416, "lon": 121.473701, "name": "" }
```

`name` is the place name carried in the link (empty for plain coordinate input).

**Response example (plain text, without `format=json`)**

```text
GET /api/parse?u=31.230416,121.473701
```

```text
lat=31.230416&lon=121.473701
```

On parse failure, the endpoint returns HTTP 422 with `{ "error": "..." }`.

## Project Structure

```text
worker/
├── src/
│   ├── index.js          # Hono app entry: GET / console page, GET /api/parse endpoint
│   ├── page.js           # Console page (Leaflet map, picking, bookmarks, search, write settings)
│   └── parse.js          # Map link parsing and GCJ-02 ⇄ WGS84 coordinate conversion
├── functions/
│   └── [[route]].js      # Cloudflare Pages Functions entry point
├── dist/
│   └── _routes.json      # Pages routing configuration
├── wloc-worker.js        # Standalone Worker entry point (single file)
├── wrangler.jsonc        # Worker deployment config
├── wrangler.pages.jsonc  # Pages deployment config (output dir dist)
└── package.json
```

## Getting the WLOC Location Module

This repository is only the web console + link parsing API. For the WLOC location module itself, join the Telegram group **Chinese Search Master** ([@Sousuozh_cn](https://t.me/Sousuozh_cn)) and **send any three messages first, then send 「看简介」**, to get it via the reply.


## Related Projects

- WLOC main project (module subscriptions, tutorial): https://github.com/Armorys/wloc-ios-virtual-location
- Upstream repository of the WLOC location module: https://github.com/ios151/wloc-ios

## Disclaimer

This project is for learning, development testing, and personal privacy protection only. Please comply with the laws and regulations of your jurisdiction and the terms of service of relevant platforms. You are solely responsible for any consequences arising from the use of this project.
