# Galaxy ad controls: com.scanner.ones

Release 1.2.1 (13). On November 8, 2026 the owner confirmed this exact update is live and explicitly authorized real ads. Revision 2 enabled App Open, Banner, Interstitial, Native and Rewarded. Revision 3 adds the supplied Rewarded Interstitial unit `ca-app-pub-8235247040566090/6644732065` and enables that format too. All six formats are enabled with app-specific production units. IAP is unchanged and compiled off. Actual consent eligibility and ad delivery are separate from publishing this configuration.

## Configuration

- Repository: https://github.com/metosapps/scanner-ones-galaxy-config
- Endpoint: https://raw.githubusercontent.com/metosapps/scanner-ones-galaxy-config/main/config.json
- Edit `config.json` on GitHub (owner access required), increment `config_revision`, and commit.
- Only this package and Galaxy flavor are accepted. Invalid JSON/schema/package/store leaves the previous valid cache unchanged.
- HTTPS only, 10-second connect/read timeouts, 64 KiB maximum. Startup, resume and foreground polling fetch at most once per minute. GitHub CDN propagation can take additional minutes; changes are not instantaneous. Offline devices keep the last valid config; a new install without config stays OFF.
- No credentials or scan data are sent in config requests. The repository contains public settings only.

## Controls

| Key | Prepared value | Bounds / meaning |
| --- | --- | --- |
| `ad_mode` | `production` | `off`, `test`, `production` |
| `ads_enabled` | `true` | Global kill switch |
| `production_ads_approved` | `true` | Required in addition to production mode |
| `interstitial_every_n_scans` | `2` | 2-50 successful single scans |
| `interstitial_every_n_clicks` | `4` | 4-100 navigation tab changes, not arbitrary taps/ad clicks |
| `interstitial_max_per_session` | `20` | 1-50 actual interstitial displays per process lifetime |
| `fullscreen_cooldown_seconds` | `120` | 60-1800 seconds between automatic fullscreen ads |
| `open_ad_cooldown_seconds` | `300` | 120-3600 seconds between app-open ads |
| `open_ad_min_background_seconds` | `30` | 30-600 seconds away before app-open eligibility |
| `reward_hours` | `24` | 1-168 hours of ad-free access after optional rewarded completion |

Each format also has its own switch: `open_ad_enabled`, `banner_enabled`, `interstitial_enabled`, `native_enabled`, `rewarded_enabled`, `rewarded_interstitial_enabled`. All six are true in revision 3. Rewarded Interstitial remains an explicit user-requested access offer, used when ordinary Rewarded is not ready; it is not an automatic scan/navigation placement. `admob_app_id` is documentation only: changing the manifest App ID requires a new signed APK.

## Active configuration behavior

Either the scan threshold OR the navigation threshold makes an interstitial eligible. A scan ad is attempted when leaving the result, never over the camera or while reading a result. Opening old History results does not count as a scan. Batch mode does not interrupt a batch or count batch entries toward this counter. Navigation counts only a change to a different main tab. A displayed interstitial resets both counters; a skipped/unavailable ad does not. Both counters are cleared when ad eligibility is lost. The session cap resets on process restart.

All automatic fullscreen ads respect the shared cooldown, foreground/resumed state, and non-overlap gate. An ad not ready at a transition is skipped, not shown late. App-open is for eligible returns, not first cold launch. Native is in History; banner is below Generate content; rewarded is optional from Settings when available. No ad clicks are generated automatically.

The existing three-day trial and any rewarded/premium ad-free period suppress all requests. Consent remains required even for test ads. The UMP form and AdMob account eligibility need separate verification before activation. Release can be remotely put in test mode for authorized QA; debug can only use Google demo units. Neither build requests ads when `ads_enabled=false`/`ad_mode=off`.

## Rollback

Set `ads_enabled=false`, `ad_mode=off`, `production_ads_approved=false` and all format switches false; increment revision and commit. New requests and cached placements stop when the device receives the change. This does not instantly reach offline devices or forcibly dismiss a currently showing SDK ad.

## Scope and release checks

This release adds ad flags, pacing and links to the existing typed config manager. Maintenance/version gates and remotely delivered content are not implemented here; any future gate must remain package/store-bound and link to Galaxy Store, never self-install downloaded code.

Unit tests cover pacing, safety bounds, disabled defaults, publisher isolation and production approval. Local APK verification covers identity, signing and native alignment. Real ad serving, consent forms and store publication are separate checks, not implied by these tests. Update store privacy/data-safety disclosures for the SDK before submission; existing 1.2.0 listing translations are not a substitute for the updated disclosure.

References: https://support.google.com/admob/answer/6201362?hl=en and https://developers.google.com/admob/android/privacy/play-data-disclosure
