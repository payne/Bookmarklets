# Reload300 Session Notes

**Date:** September 15, 2026

## Request

> Create a new bookmarklet named reload300 that reloads the page every 300 seconds. It should console.log when reloading and console.log every 30 seconds how much time is left until a reload. After the reload it should remain in effect.

## The Core Design Problem

The last requirement — "after the reload it should remain in effect" — runs into a real browser constraint: a genuine `location.reload()` (or any real navigation) destroys the entire JavaScript execution context. Every variable, timer, and object the bookmarklet created is wiped out. There is no code left running afterward to check any stored flag and re-arm itself. This is a browser architecture/security boundary, not a gap that can be coded around.

Two ways to reconcile that with the request were considered:

1. **Soft reload (chosen):** Instead of a real navigation, `fetch()` the current URL fresh (bypassing cache) and swap the content into the existing document via `document.open()` / `document.write()` / `document.close()`. This replaces the page's HTML the same way a reload would, but doesn't destroy `window` — so the bookmarklet's own timers survive and can immediately re-arm the next cycle. Trade-offs: no new browser history/reload entry, and pages with heavy `<script type="module">` use, hydration, or strict CSP may not re-initialize exactly like a true navigation would.
2. **True reload + sessionStorage resume:** Use a real `location.reload()` and stash countdown state in `sessionStorage` so *if* the script ran again it would resume correctly — but nothing can make it run again automatically after a true reload, so in practice this would require manually re-clicking the bookmarklet every 300 seconds.

Asked the user which to use via `AskUserQuestion`; they chose **option 1 (soft reload)**, since it's the only one that actually satisfies "remains in effect" without manual intervention.

## What Was Built

### `window._reload300` bookmarklet

A singleton object (consistent with the repo's existing pattern used by `_laserPointer`, `_confetti`, `_snow`, `_labels`) that:

- Starts a `setInterval` every 30s that logs seconds remaining until the next reload
- Starts a `setTimeout` for 300s that triggers the reload
- On reload: logs `[reload300] Reloading now...`, then `fetch(location.href, {cache:'reload'})`s a fresh copy of the page and swaps it in via `document.open/write/close`
- On success, immediately re-arms the next 300s cycle (this is what makes it loop forever)
- On fetch failure, falls back to a real one-time `location.reload()`, with a console warning that the timer won't survive that fallback
- Clicking the bookmarklet again calls `toggle()`, stopping both timers (`clearInterval`/`clearTimeout`)

### Annotated source

```javascript
(function() {
    // If already running, treat a second click as a stop/start toggle
    if (window._reload300) {
        window._reload300.toggle();
        return;
    }

    var RELOAD_MS = 300000; // 5 minutes
    var LOG_MS = 30000;     // 30 seconds

    window._reload300 = {
        active: false,
        endTime: 0,        // timestamp of the next scheduled reload
        reloadTimer: null, // setTimeout handle for the reload
        logTimer: null,    // setInterval handle for the countdown log

        // Start/stop the whole cycle
        toggle: function() {
            this.active = !this.active;
            if (this.active) {
                this.start();
            } else {
                this.stop();
            }
        },

        start: function() {
            console.log('[reload300] Started - reloading every ' + (RELOAD_MS / 1000) + 's.');
            this.scheduleCycle();
        },

        // Arms the countdown log + the reload timer for one 300s cycle
        scheduleCycle: function() {
            var self = this;
            this.endTime = Date.now() + RELOAD_MS;

            // Log time remaining every 30 seconds
            this.logTimer = setInterval(function() {
                var remaining = Math.max(0, Math.round((self.endTime - Date.now()) / 1000));
                console.log('[reload300] ' + remaining + 's until reload.');
            }, LOG_MS);

            // Fire the reload itself after 300 seconds
            this.reloadTimer = setTimeout(function() {
                self.doReload();
            }, RELOAD_MS);
        },

        // Re-fetch the page fresh and swap it into the current document,
        // instead of a real navigation, so our timers survive
        doReload: function() {
            var self = this;
            console.log('[reload300] Reloading now...');
            clearInterval(this.logTimer);

            fetch(location.href, { cache: 'reload', credentials: 'same-origin' })
                .then(function(res) {
                    if (!res.ok) {
                        throw new Error('HTTP ' + res.status);
                    }
                    return res.text();
                })
                .then(function(html) {
                    // Wipes and replaces the document's content, but NOT the
                    // window object our timers and this object live on
                    document.open();
                    document.write(html);
                    document.close();

                    console.log('[reload300] Reload complete, still active.');

                    // Immediately arm the next cycle so it repeats forever
                    if (self.active) {
                        self.scheduleCycle();
                    }
                })
                .catch(function(err) {
                    // Something prevented the soft-reload (e.g. the fetch was
                    // blocked); fall back to a normal one-time reload
                    console.log('[reload300] Fetch failed, falling back to a full navigation reload (timer will not survive):', err);
                    location.reload();
                });
        },

        stop: function() {
            clearInterval(this.logTimer);
            clearTimeout(this.reloadTimer);
            console.log('[reload300] Stopped.');
        }
    };

    window._reload300.toggle();
})();
```

### Minified version (as embedded in `index.html`)

```javascript
javascript:(function(){if(window._reload300){window._reload300.toggle();return;}var RELOAD_MS=300000;var LOG_MS=30000;window._reload300={active:false,endTime:0,reloadTimer:null,logTimer:null,toggle:function(){this.active=!this.active;if(this.active){this.start();}else{this.stop();}},start:function(){console.log('[reload300] Started - reloading every '+(RELOAD_MS/1000)+'s.');this.scheduleCycle();},scheduleCycle:function(){var self=this;this.endTime=Date.now()+RELOAD_MS;this.logTimer=setInterval(function(){var remaining=Math.max(0,Math.round((self.endTime-Date.now())/1000));console.log('[reload300] '+remaining+'s until reload.');},LOG_MS);this.reloadTimer=setTimeout(function(){self.doReload();},RELOAD_MS);},doReload:function(){var self=this;console.log('[reload300] Reloading now...');clearInterval(this.logTimer);fetch(location.href,{cache:'reload',credentials:'same-origin'}).then(function(res){if(!res.ok){throw new Error('HTTP '+res.status);}return res.text();}).then(function(html){document.open();document.write(html);document.close();console.log('[reload300] Reload complete, still active.');if(self.active){self.scheduleCycle();}}).catch(function(err){console.log('[reload300] Fetch failed, falling back to a full navigation reload (timer will not survive):',err);location.reload();});},stop:function(){clearInterval(this.logTimer);clearTimeout(this.reloadTimer);console.log('[reload300] Stopped.');}};window._reload300.toggle();})();
```

## Files Changed

| File | Change |
|------|--------|
| `index.html` | Added the Reload300 bookmarklet as a 6th `<li>` entry, matching the existing collapsible `<details>`/`<summary>` list format |
| `docs/reload300.md` | New doc file: features, the "why not `location.reload()`" explanation, how-it-works, annotated source, minified version, technical notes — follows the same structure as `docs/snow.md`, `docs/laser-pointer.md`, etc. |
| `INTERACTIONS.md` | Appended "Interaction 7: Reload300 Bookmarklet" entry, following the log's existing per-addition convention |

## Verification Performed

- `node --check` on both the full and minified versions of the script — syntax valid
- Checked the minified code for characters requiring URL-encoding inside the `href="..."` attribute (`#`, `%`, `"`) — none present, consistent with how the other non-legacy bookmarklets in this repo (laser pointer, confetti, snow, labels) are embedded unencoded
- Extracted the bookmarklet code back out of `index.html` via `grep`/`sed` and confirmed via `diff` that it exactly matches the tested minified source, and re-ran `node --check` on the extracted copy
- Checked `index.html` tag balance after the edit: 6 `<details>`/`</details>` pairs (one per bookmarklet) and balanced `<li>`/`</li>` counts

No browser-based UI testing was performed in this session — the checks above cover syntax and structural correctness, not runtime behavior in an actual browser.
