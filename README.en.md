<p align="center">
  <img src="icon.svg" width="160" />
</p>

# WLOC Virtual Location: Change iOS Network Location Without Jailbreak

**English** | [简体中文](./README.md)

![iOS](https://img.shields.io/badge/iOS-Virtual%20Location-blue)
![No Jailbreak](https://img.shields.io/badge/No-Jailbreak-green)
![Surge](https://img.shields.io/badge/Surge-Module-orange)
![Quantumult X](https://img.shields.io/badge/Quantumult%20X-Module-orange)
![Loon](https://img.shields.io/badge/Loon-Plugin-orange)
![Stash](https://img.shields.io/badge/Stash-Override-orange)
![Shadowrocket](https://img.shields.io/badge/Shadowrocket-Module-orange)

WLOC virtual location rewrites the coordinates returned by Apple's network location service (WLOC — Wi-Fi / cell-tower positioning) to change your iOS location **without a jailbreak** and without typing in latitude/longitude by hand: tap a point on the online map picker, or share a location from your maps app, and the location switches. This repository provides ready-made modules for Surge, Quantumult X, Loon, Stash, and Shadowrocket, the two MITM scripts behind them, plus the companion online location picker and map-link parsing service.

## Contents

- [Quick Start](#quick-start)
- [Get the Module & Community](#get-the-module--community)
- [Subscription URLs](#subscription-urls)
- [Shortcuts (Recommended)](#shortcuts-recommended)
- [Online Location Picker](#online-location-picker)
- [Map Link Parsing & Privacy](#map-link-parsing--privacy)
- [Self-Hosting the Picker](#self-hosting-the-picker)
- [How It Works](#how-it-works)
- [Parameters](#parameters)
- [iOS 26 / 27: Location Cache & Reboot](#ios-26--27-location-cache--reboot)
- [Saved Locations](#saved-locations)
- [Restoring Your Real Location](#restoring-your-real-location)
- [Notes](#notes)
- [FAQ](#faq)
- [Credits](#credits)
- [Disclaimer](#disclaimer)

## Quick Start

Three paths — pick any one:

| Path | Best for | Entry point |
| --- | --- | --- |
| ① Subscribe to a module | Everyone (required first step) | [Subscription URLs](#subscription-urls) |
| ② Shortcuts | One-tap switching / restoring | [Shortcuts (Recommended)](#shortcuts-recommended) |
| ③ Online picker | Carefully choosing a spot on a map | https://wloc-pages.pages.dev/ |

Whichever path you take, the same prerequisites apply: **proxy connected, module enabled, MITM on with the certificate trusted, and `gs-loc.apple.com` included in the MITM hostname list**.

## Get the Module & Community

> [!IMPORTANT]
> **The only way to get the module and subscription addresses:**
>
> 1. Join the Telegram group **Chinese Search Master** ([@Sousuozh_cn](https://t.me/Sousuozh_cn))
> 2. Send **any three messages** in the group (any content, three in total)
> 3. Read the **group description** to get the module and subscription addresses
>
> This repository does not provide module downloads or subscription links directly. If you don't join the group or don't send the three messages, you can't get them — please don't ask in Issues.

Usage questions are also welcome in the group.

## Subscription URLs

Subscription addresses are not published here. To get them, join the Telegram group **Chinese Search Master** ([@Sousuozh_cn](https://t.me/Sousuozh_cn)) — send any three messages first, then read the group description — see [Get the Module & Community](#get-the-module--community).

Two notes:

- **Stash** users: please use the `.stoverride` version; no Script Hub conversion is needed
- **Egern** can use the Surge version of the module directly

## Shortcuts (Recommended)

Once the module is installed, iOS Shortcuts is the most convenient way to use it: no need to open the picker page — just share a location from your maps app to switch, and one tap to clear.

- **wloc 设置地理位置 (Set location)**: https://www.icloud.com/shortcuts/a82717d8fdad4e6280866fcf911173f7
- **wloc 清理恢复位置 (Clear & restore location)**: https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f

**Setting a location**

1. Pick a location in your maps app (long-press the map to drop a pin)
2. Tap "Share" → choose "wloc 设置地理位置"
   - Apple Maps: pick a spot → Share → "wloc 设置地理位置"
   - Amap (Gaode): pick a spot → Share → **More** → "wloc 设置地理位置"

**Restoring your location**: run "wloc 清理恢复位置" to return to your real location.

The shortcuts currently support Apple Maps and Amap, including short links (redirects are followed automatically and GCJ-02 coordinates are converted to WGS84 — see [Map Link Parsing & Privacy](#map-link-parsing--privacy)). A fuller illustrated walkthrough is in the [Shortcuts guide](docs/shortcut-guide.md). The picker page (Worker / Pages) remains available as well — see the next section.

## Online Location Picker

Public picker page: https://wloc-pages.pages.dev/ (add it to your Home Screen and use it like an app)

Steps:

1. Subscribe to a module above and enable MITM
2. Open the online picker page
3. Pick a location on the map / search for a place name / paste a map link (Apple Maps, Google Maps, Amap, and Baidu links, or plain coordinate text)
4. Tap "储存到设备" (Save to device)
5. The new coordinates take effect the next time Apple location services are triggered

Picked locations can be saved as favorites for quick switching between several places — see [Saved Locations](#saved-locations).

## Map Link Parsing & Privacy

When a shortcut processes a map link, it first sends the link to the parsing endpoint `wloc-spoofer.wloc.workers.dev/api/parse`, where a worker handles it uniformly. The two kinds of links are tricky in different ways:

- **Amap (Gaode)**: shares come out as short links; the real coordinates are hidden in the `Location` header of a 302 redirect, and they are GCJ-02 offset coordinates. Shortcuts can neither read redirect headers nor easily do coordinate conversion, so the worker does it: follow the redirect → extract the coordinates → convert GCJ-02 to WGS84 → return latitude/longitude.
- **Apple Maps**: the link carries `coordinate=latitude,longitude` directly, but coordinates in mainland China are likewise GCJ-02 offset, so they are converted before being returned, just like Amap; coordinates outside China skip conversion automatically (the `out_of_china` check) and are returned unchanged.

Routing everything through one endpoint also conveniently handles short-link expansion, text-wrapped links, and place-name decoding.

**Privacy**: `/api/parse` is a pure relay parser — receive a link → follow the redirect → parse the coordinates → return JSON. Nothing is written to storage, nothing is logged, nothing is cached; data is discarded as soon as it is processed.

The parsing logic and routing are open source in this repository ([`worker/src/parse.js`](worker/src/parse.js), [`worker/src/index.js`](worker/src/index.js)). If you prefer not to use the public endpoint, self-host your own (next section) and replace `wloc-spoofer.wloc.workers.dev` in the shortcuts with your own worker domain.

## Self-Hosting the Picker

The public picker has request limits, so for long-term use, deploy your own instance. Two ready-made entry points with identical functionality:

- **Workers**: https://wloc-spoofer.wloc.workers.dev/
- **Pages**: https://wloc-pages.pages.dev/

**One-click deploy (Workers mode only)**

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/Armorys/wloc-ios-virtual-location/tree/main/worker)

Click the button and follow the authorization prompts to finish deployment.

**Manual deploy (Workers)**

```bash
# 1. Clone the repository
git clone https://github.com/Armorys/wloc-ios-virtual-location.git
cd wloc/worker

# 2. Install dependencies
npm install

# 3. Log in to Cloudflare (first time only)
npx wrangler login

# 4. Deploy
npm run deploy
```

After deployment you get your own Worker URL (e.g. `https://wloc-spoofer.<your-subdomain>.workers.dev`); open the picker at that address. Cloudflare's free tier includes 100,000 requests per day, which is plenty for personal use.

<details>
<summary>Advanced: Pages deployment</summary>

Pages deployment has no one-click button and must be run manually:

```bash
git clone https://github.com/Armorys/wloc-ios-virtual-location.git
cd wloc/worker
npm install
npx wrangler pages deploy dist --project-name <your-project-name>
```

When prompted for the production branch, enter `main`. After deployment you get a `https://<project-name>.pages.dev` address. Pages and Workers are functionally identical — choose whichever you prefer.

</details>

## How It Works

```text
The picker page sends the target coordinates to gs-loc.apple.com/wloc-settings/save
  → the proxy module intercepts the request; wloc-settings.js writes the coordinates into $persistentStore
  → on the next WLOC location request, wloc.js reads the coordinates back
  → it parses the /clls/wloc protobuf response, replaces the coordinates, and returns it to the system
```

The module consists of two script rules, both in the `dist/` directory:

- **`wloc.js`**: intercepts `/clls/wloc` responses, parses the protobuf payload, and replaces the coordinates
- **`wloc-settings.js`**: intercepts `/wloc-settings/save` requests and writes the coordinates submitted by the picker page into persistent storage

In other words, this approach only rewrites network location (Wi-Fi / cell-tower) results; it does not touch GPS hardware positioning.

## Parameters

| Parameter | Description | Default |
| --- | --- | --- |
| longitude | Target longitude (online picker takes priority) | null (passthrough) |
| latitude | Target latitude (online picker takes priority) | null (passthrough) |
| accuracy | Accuracy (meters) | 25 |
| logLevel | Log level | info |

Priority: **coordinates saved from the online picker > module parameters > defaults**.

## iOS 26 / 27: Location Cache & Reboot

Starting with iOS 26, Apple greatly strengthened the `locationd` location cache: the system keeps previously obtained real location results in memory and reuses them for a long time. As a result, even when the script has successfully rewritten the WLOC response (the log shows "modified"), the system may keep using the cached old coordinates, making it look as if the location never changed.

**The fix: reboot the device.** A reboot clears the `locationd` in-memory cache, so the next WLOC request picks up the modified coordinates. On iOS 26+, toggling airplane mode or location services **cannot** clear this cache — a reboot is required. iOS 15–18 usually takes effect without a reboot.

**Recommended flows for newer iOS versions (highest success rate, choose one)**

Method 1:

1. First pick your target location in the picker page and save it to the device
2. Turn on airplane mode → turn off location services → reboot the device
3. Turn off airplane mode (keep Wi-Fi off too) → connect the proxy app (confirm the VPN icon appears) → turn on location services
4. Open a maps app to verify

Method 2:

1. Turn off location services
2. Pick a location in the picker page and save it to the device
3. Turn on location services → when the "Allow location access" prompt appears, choose **"Ask Next Time Or When I Share"**
4. Open a maps app to verify

## Saved Locations

The online picker can save multiple favorite locations, making it easy to hop between a few regular spots:

- **Add a favorite**: pick a location, tap "收藏位置" (Save location), enter a note name (Chinese / English / digits, up to 30 characters), and save
- **Quick switch**: tap a location in the favorites list and the map jumps to it; tap "储存到设备" (Save to device) to switch
- **Currently active marker**: a favorite matching the coordinates saved on the device shows "✓ 当前生效" (currently active)
- **Manage**: delete individually (× button) or clear all
- **Current active coordinates**: the page displays the device-side persistent data (`wloc_settings`), with refresh and clear actions

**Don't confuse the two kinds of storage**:

| Data | Stored in | Notes |
| --- | --- | --- |
| Favorites list | Browser `localStorage` | Only UI convenience data for the picker page; clearing the browser cache or switching browsers means re-saving favorites |
| Active coordinates | The proxy app's `$persistentStore` | The data the scripts actually read at runtime; unaffected by browser cache |

## Restoring Your Real Location

**Method 1: Disable or delete the module (recommended)**

Once the module is off, the scripts no longer intercept WLOC requests and the system automatically returns to your real location. On iOS 26+, also reboot the device to clear the location cache.

**Method 2: Clear persistent data to enter passthrough mode**

After the saved coordinates are cleared, the script enters **passthrough mode**: it stops modifying WLOC responses and passes the original data through, so the system returns to real GPS positioning.

Passthrough triggers only when both conditions hold: **persistent data is empty (null), and the module parameters are at their default values (113.94114, 22.544577)**. The script then decides the user has not customized coordinates and skips modification automatically. Leave the module parameters alone — just clear the persistent data.

Delete the persistent data named `wloc_settings` in your proxy app:

- **Surge** — run in the script editor: `$persistentStore.write(null, "wloc_settings")`
- **Quantumult X** — run: `$prefs.removeValueForKey("wloc_settings")`
- **Loon** — run: `$persistentStore.write(null, "wloc_settings")`

Reboot the device after clearing to restore your real location; there is no need to disable the module.

> Note: if you manually changed the latitude/longitude in the module parameters (anything other than the defaults 113.94114, 22.544577), the script will keep modifying with the parameter coordinates even after persistent data is cleared. Passthrough only engages while the parameters stay at their defaults.

## Notes

- The MITM certificate must trust both `gs-loc.apple.com` and `gs-loc-cn.apple.com`
- This approach only modifies network location (Wi-Fi / cell-tower), not GPS hardware positioning
- When the GPS signal is strong, iOS may ignore network location results
- Indoor scenarios dominated by Wi-Fi positioning work best
- The picker page must be used with the proxy active (Safari traffic must go through the proxy so the save request can be intercepted)

## FAQ

**Q: Is a jailbreak required?**
A: No. The whole setup builds on the MITM and scripting capabilities of proxy apps and works on a stock iOS device.

**Q: Which proxy apps are supported?**
A: Ready-made modules are provided for Surge, Quantumult X, Loon, Stash, and Shadowrocket (see [Subscription URLs](#subscription-urls)); Egern can use the Surge module directly.

**Q: I set everything up, but my location didn't change. What now?**
A: First confirm the proxy is connected, the module is enabled, the certificate is trusted, and `gs-loc.apple.com` is in the MITM hostname list. On iOS 26 or later, it is most likely the `locationd` cache — follow the reboot flows in [iOS 26 / 27: Location Cache & Reboot](#ios-26--27-location-cache--reboot).

**Q: How do I get my real location back?**
A: The quickest way is the "wloc 清理恢复位置" shortcut; you can also disable / delete the module, or clear the `wloc_settings` persistent data to enter passthrough mode — see [Restoring Your Real Location](#restoring-your-real-location).

**Q: Is my location data uploaded anywhere?**
A: No. The map-link parsing endpoint `/api/parse` is a pure relay parser — no storage writes, no logging, no caching — and picked coordinates live only in your own proxy app's persistent storage on your device. If you don't trust the public endpoint, self-host the worker.

## Credits

This project builds on ideas and foundations from the following open-source work:

- [proxypin-wloc-spoofer](https://github.com/FFF686868/proxypin-wloc-spoofer) — the original WLOC location-modification idea, by FFF686868
- [NSNanoCat/Util](https://github.com/NSNanoCat/util) — cross-platform scripting utility framework

For the module or usage discussion, you can also join the Telegram group 中文搜索大师 (Chinese Search Master, [@Sousuozh_cn](https://t.me/Sousuozh_cn)) — see [Get the Module & Community](#get-the-module--community).

## Disclaimer

This project is for learning, development testing, and personal privacy protection only. Please comply with the laws and regulations of your jurisdiction and the terms of service of relevant platforms. You are solely responsible for any consequences arising from the use of this project.
