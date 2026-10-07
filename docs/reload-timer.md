# Reload Timer Bookmarklet

Asks how many seconds to wait, then shows an on-page countdown in the upper-right corner and reloads the page every time it reaches zero, forever. A configurable, visible cousin of [Reload300](reload300.md).

## Features

- Prompts for the reload interval in seconds (defaults to `60`; decimals like `2.5` work)
- Shows a countdown badge (`Reload in 1:05`, `Reload in 42s`) fixed to the upper-right corner, above everything else on the page
- Keeps running after every reload — no need to click the bookmarklet again
- Click the countdown badge, or click the bookmarklet a second time, to stop it
- Scrolls to the bottom of the page after each reload — handy for log files and other pages that grow at the bottom
- Pages served as `text/plain` are wrapped in a `<pre>` after reload so line breaks and spacing are preserved
- Falls back to a normal one-time `location.reload()` if something goes wrong

## How It Survives the Reload

Like Reload300, it doesn't use `location.reload()` for the repeating cycle. A real navigation destroys the page's whole JavaScript environment, including the bookmarklet's timer. Instead it re-fetches the current URL (bypassing the cache) and swaps the fresh content into the existing document with `document.open()` / `document.write()` / `document.close()`. The `window` object, and `window._reloadTimer` on it, survive, so the next countdown starts right away.

See [Why It Doesn't Use `location.reload()`](reload300.md#why-it-doesnt-use-locationreload) in the Reload300 docs for the full explanation and trade-offs.

## How It Works

The bookmarklet creates a singleton object (`window._reloadTimer`) that manages the countdown and reload cycle. When started, it:

1. Prompts for the interval and validates that it's a positive number
2. Records the target reload time (`Date.now() + PERIOD_MS`)
3. Starts a `setInterval` every 250ms that updates the countdown badge
4. When the countdown reaches zero, `fetch()`es the current URL fresh and swaps it into the document
5. Scrolls to the bottom, then (if still active) starts the next cycle with a fresh badge
6. If the fetch fails, falls back to a real `location.reload()`

## Annotated Source Code

```javascript
(function() {
    // If already running, a second click stops it
    if (window._reloadTimer) {
        window._reloadTimer.stop();
        return;
    }

    // Ask for the interval; Cancel aborts quietly
    var input = prompt('Reload this page every how many seconds?', '60');
    if (input === null) {
        return;
    }
    var secs = parseFloat(input);
    // !(secs > 0) also rejects NaN from non-numeric input
    if (!(secs > 0)) {
        alert('Please enter a positive number of seconds.');
        return;
    }
    var PERIOD_MS = secs * 1000;

    window._reloadTimer = {
        endTime: 0,  // timestamp of the next scheduled reload
        tick: null,  // setInterval handle for the countdown
        badge: null, // the countdown element in the corner

        // Start one countdown cycle
        start: function() {
            var self = this;
            this.endTime = Date.now() + PERIOD_MS;
            // 250ms keeps the display accurate without drifting a whole second
            this.tick = setInterval(function() {
                self.update();
            }, 250);
            this.update();
        },

        // Create the badge if it doesn't exist or was wiped out by the
        // page swapping in new content
        ensureBadge: function() {
            if (this.badge && document.documentElement.contains(this.badge)) {
                return;
            }
            var self = this;
            var el = document.createElement('div');
            el.title = 'Click to cancel auto-reload';
            Object.assign(el.style, {
                position: 'fixed',
                top: '10px',
                right: '10px',
                zIndex: '2147483647',  // max z-index: stay on top of everything
                padding: '6px 10px',
                backgroundColor: 'rgba(0,0,0,0.75)',
                color: 'white',
                fontFamily: 'monospace',
                fontSize: '14px',
                fontWeight: 'bold',
                borderRadius: '4px',
                boxShadow: '0 2px 6px rgba(0,0,0,0.4)',
                cursor: 'pointer',
                userSelect: 'none'
            });
            // Clicking the badge cancels the auto-reload
            el.addEventListener('click', function() {
                self.stop();
            });
            (document.body || document.documentElement).appendChild(el);
            this.badge = el;
        },

        // Refresh the countdown text and reload when it hits zero
        update: function() {
            var left = Math.max(0, Math.ceil((this.endTime - Date.now()) / 1000));
            var m = Math.floor(left / 60);
            var s = left - m * 60;
            this.ensureBadge();
            // "Reload in 1:05" when a minute or more is left, else "Reload in 42s"
            this.badge.textContent = 'Reload in ' + (m ? m + ':' + (s < 10 ? '0' : '') + s : s + 's');
            if (left <= 0) {
                clearInterval(this.tick);
                this.badge.textContent = 'Reloading...';
                this.doReload();
            }
        },

        // Re-fetch the page fresh and swap it into the current document,
        // instead of a real navigation, so this object survives
        doReload: function() {
            var self = this;
            fetch(location.href, { cache: 'reload', credentials: 'same-origin' })
                .then(function(res) {
                    if (!res.ok) {
                        throw new Error('HTTP ' + res.status);
                    }
                    // Remember whether this is a plain-text page (e.g. a log file)
                    var plain = /^text\/plain/i.test(res.headers.get('content-type') || '');
                    return res.text().then(function(body) {
                        return { body: body, plain: plain };
                    });
                })
                .then(function(r) {
                    document.open();
                    if (r.plain) {
                        // Writing raw text would be parsed as HTML and lose its
                        // line breaks, so put it in a <pre> via textContent
                        document.write('<!DOCTYPE html><html><head></head><body><pre></pre></body></html>');
                        document.close();
                        var pre = document.querySelector('pre');
                        pre.style.whiteSpace = 'pre-wrap';
                        pre.style.wordWrap = 'break-word';
                        pre.textContent = r.body;
                    } else {
                        document.write(r.body);
                        document.close();
                    }
                    self.scrollToBottom();
                    // Only restart if we weren't stopped while fetching
                    if (window._reloadTimer === self) {
                        self.badge = null;  // old badge was wiped with the old document
                        self.start();
                    }
                })
                .catch(function(err) {
                    // Soft-reload failed; fall back to a normal one-time reload
                    console.log('[reloadTimer] Fetch failed, falling back to a full navigation reload (timer will not survive):', err);
                    location.reload();
                });
        },

        // Jump to the bottom, retrying as late content/images change the height
        scrollToBottom: function() {
            var go = function() {
                var d = document.documentElement, b = document.body;
                window.scrollTo(0, Math.max(d ? d.scrollHeight : 0, b ? b.scrollHeight : 0));
            };
            go();
            setTimeout(go, 300);
            setTimeout(go, 1500);
        },

        // Cancel the countdown, remove the badge, and allow a fresh start
        stop: function() {
            clearInterval(this.tick);
            if (this.badge) {
                this.badge.remove();
            }
            window._reloadTimer = null;
        }
    };

    window._reloadTimer.start();
})();
```

## Minified Version

```javascript
javascript:(function(){if(window._reloadTimer){window._reloadTimer.stop();return;}var input=prompt('Reload this page every how many seconds?','60');if(input===null){return;}var secs=parseFloat(input);if(!(secs>0)){alert('Please enter a positive number of seconds.');return;}var PERIOD_MS=secs*1000;window._reloadTimer={endTime:0,tick:null,badge:null,start:function(){var self=this;this.endTime=Date.now()+PERIOD_MS;this.tick=setInterval(function(){self.update();},250);this.update();},ensureBadge:function(){if(this.badge&&document.documentElement.contains(this.badge)){return;}var self=this;var el=document.createElement('div');el.title='Click to cancel auto-reload';Object.assign(el.style,{position:'fixed',top:'10px',right:'10px',zIndex:'2147483647',padding:'6px 10px',backgroundColor:'rgba(0,0,0,0.75)',color:'white',fontFamily:'monospace',fontSize:'14px',fontWeight:'bold',borderRadius:'4px',boxShadow:'0 2px 6px rgba(0,0,0,0.4)',cursor:'pointer',userSelect:'none'});el.addEventListener('click',function(){self.stop();});(document.body||document.documentElement).appendChild(el);this.badge=el;},update:function(){var left=Math.max(0,Math.ceil((this.endTime-Date.now())/1000));var m=Math.floor(left/60);var s=left-m*60;this.ensureBadge();this.badge.textContent='Reload in '+(m?m+':'+(s<10?'0':'')+s:s+'s');if(left<=0){clearInterval(this.tick);this.badge.textContent='Reloading...';this.doReload();}},doReload:function(){var self=this;fetch(location.href,{cache:'reload',credentials:'same-origin'}).then(function(res){if(!res.ok){throw new Error('HTTP '+res.status);}var plain=/^text\/plain/i.test(res.headers.get('content-type')||'');return res.text().then(function(body){return{body:body,plain:plain};});}).then(function(r){document.open();if(r.plain){document.write('<!DOCTYPE html><html><head></head><body><pre></pre></body></html>');document.close();var pre=document.querySelector('pre');pre.style.whiteSpace='pre-wrap';pre.style.wordWrap='break-word';pre.textContent=r.body;}else{document.write(r.body);document.close();}self.scrollToBottom();if(window._reloadTimer===self){self.badge=null;self.start();}}).catch(function(err){console.log('[reloadTimer] Fetch failed, falling back to a full navigation reload (timer will not survive):',err);location.reload();});},scrollToBottom:function(){var go=function(){var d=document.documentElement,b=document.body;window.scrollTo(0,Math.max(d?d.scrollHeight:0,b?b.scrollHeight:0));};go();setTimeout(go,300);setTimeout(go,1500);},stop:function(){clearInterval(this.tick);if(this.badge){this.badge.remove();}window._reloadTimer=null;}};window._reloadTimer.start();})();
```

## Technical Notes

- Unlike Reload300, a second click **stops** the timer and clears `window._reloadTimer`, so the next click prompts for a new interval rather than resuming the old one
- The countdown is driven by `endTime` rather than counting ticks, so it stays accurate even if the browser throttles timers in a background tab
- `ensureBadge()` re-creates the badge whenever it's missing from the DOM, since replacing the document (or the page's own scripts) can remove it
- Stopping during a fetch is safe: the `window._reloadTimer === self` check prevents a stopped timer from restarting after the fresh content arrives
- Scrolling to the bottom is retried at 0ms, 300ms, and 1500ms because images and late-loading content can keep growing the page after it's written
- Plain-text detection uses the response's `Content-Type` header, so `.log` / `.txt` files served as `text/plain` keep their formatting instead of collapsing into one line of HTML
- If `fetch` throws or the response isn't `ok` (e.g. blocked by CSP, network failure, non-2xx status), it falls back to a genuine `location.reload()` — the page reloads once, but the bookmarklet must be clicked again to resume
