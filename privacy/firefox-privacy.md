# Firefox Privacy

## Preferences

### General

* Check `Enable Container tabs`.
* Uncheck `Automatically update search engines`.

#### Home

* Homepage and New Windows: `Custom URLs` with `https://start.duckduckgo.com/`.

### Search

* Default search engine: `DuckDuckGo`.
* Uncheck `Provide search suggestions`.
* Uncheck `Show search suggestions in address bar results`.
* Uncheck `Show search suggestions ahead of browsing history in address bar results`.
* Remove all search engines except `DuckDuckGo`.

### Privacy & Security

* Uncheck everything in `Forms & Passwords`.
* Set `Use custom history setings`.
* Uncheck `Remember browsing and download history`.
* Uncheck `Remember search and form history`.

#### Cookies and Site Data

* Keep cookies: `Until Firefox closes`.
* Never accept third party cookies.

#### Tracking Protection

* Disable it completely. Extensions should do this job, enabling it will add info to your browser fingerprint.

#### Permissions

* Check `Block new requests asking to access your location` in Location.
* Check `Block new requests asking to access your camera` in Camera.
* Check `Block new requests asking to access your microphone` in Microphone.
* Check `Block pop-up windows`, `Warn you when websites try to install add-ons` and `Prevent accessibility services from accessing your browser`.

#### Firefox Data Collection and Use

* Uncheck everything.

#### Certificates

* Check `Ask you everytime`.
* Check `Query OCSP responder servers to confirm the current validity of certificates`.

## Addons

* [Bloody Vikings!](https://addons.mozilla.org/en-US/firefox/addon/bloody-vikings/)
* [CanvasBlocker](https://addons.mozilla.org/en-US/firefox/addon/canvasblocker/)
* [Cookie AutoDelete](https://addons.mozilla.org/en-US/firefox/addon/cookie-autodelete/)
* [Decentraleyes](https://addons.mozilla.org/en-US/firefox/addon/decentraleyes/)
* [Google Container](https://addons.mozilla.org/en-US/firefox/addon/canvasblocker/)
* [Google Redirects Fixer & Tracking Remover](https://addons.mozilla.org/en-US/firefox/addon/google-no-tracking-url/)
* [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)
* [uBlock Origin](https://addons.mozilla.org/en-GB/firefox/addon/ublock-origin/)
* [uMatrix](https://addons.mozilla.org/en-GB/firefox/addon/umatrix/)
* [User-Agent Switcher](https://addons.mozilla.org/en-US/firefox/addon/uaswitcher/)
* [Window Resizer](https://addons.mozilla.org/en-US/firefox/addon/window-resizer-webextension/)

## about:config

* Disable the pocket extension:
    * extensions.pocket.enabled = false
    * services.sync.prefs.sync.extensions.pocket.enabled = true

* Disable pocket in new tabs:
    * browser.newtabpage.activity-stream.section.highlights.includePocket = false
    * services.sync.prefs.sync.browser.newtabpage.activity-stream.section.highlights.includePocket = true

* Isolate all browser identifier sources (e.g. cookies) to the first party domain:
    * extensions.pocket.oAuthConsumerKey = ''
    * extensions.pocket.api = ''
    * extensions.webextensions.themes.icons.buttons = ''
    * extensions.pocket.site = ''
    * browser.newtabpage.activity-stream.feeds.section.topstories.options = ''
    * privacy.firstparty.isolate = true
    * services.sync.prefs.sync.extensions.pocket.oAuthConsumerKey = true
    * services.sync.prefs.sync.extensions.pocket.api = true
    * services.sync.prefs.sync.extensions.webextensions.themes.icons.buttons = true
    * services.sync.prefs.sync.extensions.pocket.site = true
    * services.sync.prefs.sync.browser.newtabpage.activity-stream.feeds.section.topstories.options = true
    * services.sync.prefs.sync.privacy.firstparty.isolate = true

* Make Firefox more resistant to browser fingerprinting:
    * privacy.resistFingerprinting = true
    * services.sync.prefs.sync.privacy.resistFingerprinting = true

* Disables offline cache:
    * browser.cache.offline.enable = false
    * services.sync.prefs.sync.browser.cache.offline.enable = true

* Disable web sites tracking clicks:
    * browser.send\_pings = false
    * browser.send\_pings.max\_per\_link = '0'
    * services.sync.prefs.sync.browser.send\_pings = true
    * services.sync.prefs.sync.browser.send\_pings.max\_per\_link = true

* Disable preloading of autocomplete URLs:
    * browser.urlbar.speculativeConnect.enabled = false
    * services.sync.prefs.sync.browser.urlbar.speculativeConnect.enabled = true

* Disable websites getting notifications when you copy, paste, or cut:
    * dom.event.clipboardevents.enabled = false
    * services.sync.prefs.sync.dom.event.clipboardevents.enabled = true

* Disable geolocation:
    * geo.enabled = false
    * services.sync.prefs.sync.geo.enabled = true

* Disable playback of DRM-controlled HTML5 content:
    * media.eme.enabled = false
    * services.sync.prefs.sync.media.eme.enabled = true

* Disables the Widevine Content Decryption Module provided by Google Inc.:
    * media.gmp-widevinecdm.enabled = false
    * services.sync.prefs.sync.media.gmp-widevinecdm.enabled = true

* Disable microphone and camera status tracking:
    * media.navigator.enabled = false
    * services.sync.prefs.sync.media.navigator.enabled = true

* Only accept from the originating site (block third-party cookies):
    * network.cookie.cookieBehavior = 1
    * services.sync.prefs.sync.network.cookie.cookieBehavior = true

* Accept for current session only:
    * network.cookie.lifetimePolicy = 2
    * services.sync.prefs.sync.network.cookie.lifetimePolicy = true

* Send only the scheme, host, and port in the Referer header:
    * network.http.referer.trimmingPolicy = 2
    * services.sync.prefs.sync.network.http.referer.trimmingPolicy = true

* Send Referer only when the full hostnames match:
    * network.http.referer.XOriginPolicy = 2
    * services.sync.prefs.sync.network.http.referer.XOriginPolicy = true

* Only send scheme, host, and port in Referer:
    * network.http.referer.XOriginTrimmingPolicy = 2
    * services.sync.prefs.sync.network.http.referer.XOriginTrimmingPolicy = true

* Disable WebGL:
    * webgl.disabled = true
    * services.sync.prefs.sync.webgl.disabled = true

* Never store extra session data:
    * browser.sessionstore.privacy\_level = 2
    * services.sync.prefs.sync.browser.sessionstore.privacy\_level = true

* Enable rendering IDNs as their Punycode equivalent:
    * network.IDN\_show\_punycode = true
    * services.sync.prefs.sync.network.IDN\_show\_punycode = true

* Limit the amount of identifiable information sent when requesting the Mozilla harmful extension blocklist:
    * extensions.blocklist.url = https://blocklists.settings.services.mozilla.com/v1/blocklist/3/%APP_ID%/%APP_VERSION%/%PRODUCT%/%BUILD_ID%/%BUILD_TARGET%/%LOCALE%/%CHANNEL%/%OS_VERSION%/%DISTRIBUTION%/%DISTRIBUTION_VERSION%/%PING_COUNT%/%TOTAL_PING_COUNT%/%DAYS_SINCE_LAST_PING%/
    * media.peerconnection.enabled = false
    * services.sync.prefs.sync.extensions.blocklist.url = true
    * services.sync.prefs.sync.media.peerconnection.enabled = true

* Don't allow websites to prevent use of right-click:
    * dom.event.contextmenu.enabled = false
    * services.sync.prefs.sync.dom.event.contextmenu.enabled = true

* Don't allow websites to prevent copy and paste:
    * dom.event.clipboardevents.enabled = false
    * services.sync.prefs.sync.dom.event.clipboardevents.enabled = true

* Disable site reading installed plugins:
    * plugins.enumerable\_names = ''
    * services.sync.prefs.sync.plugins.enumerable\_names = true

* Disables geolocation and firefox logging geolocation requests:
    * geo.wifi.uri = ''
    * browser.search.geoip.url = ''
    * services.sync.prefs.sync.geo.wifi.uri = true
    * services.sync.prefs.sync.browser.search.geoip.url = true

* Disable fonts fingerprinting
    * browser.display.use\_document\_fonts = 0
    * services.sync.prefs.sync.browser.display.use\_document\_fonts = true

## Additional Info

* Do not track and tracking protection are obselete and just add to your fingerprint.
* [Disabling Google safebrowsing won't do much as it happens locally and Google never gets your URL.](https://feeding.cloud.geek.nz/posts/how-safe-browsing-works-in-firefox/)
* [Disabling the referrer header is useless and just adds to your fingerprint.](https://www.privacy-handbuch.de/handbuch_21p.htm)
* For each new option, add corresponding `services.sync.prefs.sync.<new option>` entry, to sync it with firefox sync across your browsers.
