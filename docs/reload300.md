# Reload300 Bookmarklet

Automatically reloads the current page every 300 seconds (5 minutes), forever. Logs to the console when it reloads and how much time is left every 30 seconds.

## Features

- Reloads the page every 300 seconds
- Logs `[reload300] Reloading now...` to the console at each reload
- Logs the remaining time (e.g. `[reload300] 270s until reload.`) every 30 seconds
- Keeps running after every reload — no need to click the bookmarklet again
- Click the bookmarklet a second time to stop it
- Falls back to a normal one-time `location.reload()` if something goes wrong

## Why It Doesn't Use `location.reload()`

A real browser navigation (`location.reload()`, typing a URL, clicking a link) tears down the entire JavaScript environment on that page — every variable, timer, and object the bookmarklet created is gone. There is no way for a plain bookmarklet to make code "survive" a real reload and start itself back up; nothing is left running to do so. That's a browser security/architecture boundary, not a gap in the code.

To actually satisfy "keeps reloading forever without re-clicking," this bookmarklet uses a different trick: instead of navigating, it re-fetches the current URL over the network (bypassing cache) and then replaces the document in place with `document.open()` / `document.write()` / `document.close()`. This gets you the same visible result as a reload — fresh HTML from the server — but it happens *inside* the same `window`, so the JS object and its timers are never destroyed and can immediately schedule the next cycle.

Trade-offs of this approach, compared to a true reload:

- It won't add a new entry to browser history or show the browser's reload spinner
- Pages that rely heavily on `<script type="module">`, complex build-time hydration, or strict CSP may not re-initialize exactly like a fresh navigation would
- If the fetch fails (network blip, CORS/CSP restriction on refetching the page, etc.) it falls back to a real `location.reload()` — which works fine as a one-off reload, but at that point the timer is gone and you'd need to click the bookmarklet again

## How It Works

The bookmarklet creates a singleton object (`window._reload300`) that manages the reload/countdown cycle. When active, it:

1. Records the target reload time (`Date.now() + 300000`)
2. Starts a `setInterval` every 30 seconds that logs the seconds remaining
3. Starts a `setTimeout` for 300 seconds that triggers the reload
4. On reload, `fetch()`s the current URL fresh, then swaps it into the document
5. If successful and still active, immediately schedules the next 300-second cycle
6. If the fetch fails, falls back to a real `location.reload()`

## Annotated Source Code

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

## Minified Version

```javascript
javascript:(function(){if(window._reload300){window._reload300.toggle();return;}var RELOAD_MS=300000;var LOG_MS=30000;window._reload300={active:false,endTime:0,reloadTimer:null,logTimer:null,toggle:function(){this.active=!this.active;if(this.active){this.start();}else{this.stop();}},start:function(){console.log('[reload300] Started - reloading every '+(RELOAD_MS/1000)+'s.');this.scheduleCycle();},scheduleCycle:function(){var self=this;this.endTime=Date.now()+RELOAD_MS;this.logTimer=setInterval(function(){var remaining=Math.max(0,Math.round((self.endTime-Date.now())/1000));console.log('[reload300] '+remaining+'s until reload.');},LOG_MS);this.reloadTimer=setTimeout(function(){self.doReload();},RELOAD_MS);},doReload:function(){var self=this;console.log('[reload300] Reloading now...');clearInterval(this.logTimer);fetch(location.href,{cache:'reload',credentials:'same-origin'}).then(function(res){if(!res.ok){throw new Error('HTTP '+res.status);}return res.text();}).then(function(html){document.open();document.write(html);document.close();console.log('[reload300] Reload complete, still active.');if(self.active){self.scheduleCycle();}}).catch(function(err){console.log('[reload300] Fetch failed, falling back to a full navigation reload (timer will not survive):',err);location.reload();});},stop:function(){clearInterval(this.logTimer);clearTimeout(this.reloadTimer);console.log('[reload300] Stopped.');}};window._reload300.toggle();})();
```

## Technical Notes

- `RELOAD_MS` (300000) and `LOG_MS` (30000) are the only two constants that control timing
- `fetch(location.href, { cache: 'reload' })` forces a real network round-trip instead of serving a cached copy, matching what a normal hard reload does
- `document.open()` / `document.write()` / `document.close()` after page load clears and re-parses the whole document (including re-running the page's own `<script>` tags), but does **not** reset the `window` global object — that's why `window._reload300` and its active timers keep running across the "reload"
- Clicking the bookmarklet a second time calls `toggle()`, which stops both timers via `clearInterval` / `clearTimeout`
- If `fetch` throws or the response isn't `ok` (e.g. blocked by CSP, network failure, non-2xx status), it falls back to a genuine `location.reload()` — a real reload will happen once, but since that destroys the JS context, the bookmarklet would need to be clicked again afterward to resume the cycle
