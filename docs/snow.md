# Snow Bookmarklet

Adds a peaceful snowfall effect to any web page. White snowflakes gently drift down from the top of the screen, creating a winter atmosphere.

## Features

- Press `S` to toggle snow on/off
- Continuous snowfall while active
- Varying snowflake sizes and opacity
- Subtle horizontal drift for realism
- Instant cleanup when toggled off
- Won't interfere with typing

## How It Works

The bookmarklet creates a singleton object (`window._snow`) that manages the snowfall. When active, it:

1. Uses `setInterval` to create new snowflakes every 50ms
2. Each snowflake is a small white circular div with a glow effect
3. CSS transitions animate the fall with linear timing
4. Snowflakes are tracked in an array for cleanup when toggled off

## Annotated Source Code

```javascript
(function() {
    // Check if snow system already exists
    // If so, just toggle it
    if (window._snow) {
        window._snow.toggle();
        return;
    }

    // Create the snow singleton object
    window._snow = {
        active: false,       // Current state (on/off)
        interval: null,      // Reference to the interval timer
        flakes: [],          // Array to track all snowflake elements

        // Toggle snowfall on/off
        toggle: function() {
            this.active = !this.active;
            if (this.active) {
                this.start();
            } else {
                this.stop();
            }
        },

        // Start the snowfall
        start: function() {
            var self = this;
            // Create a new snowflake every 50ms
            this.interval = setInterval(function() {
                self.createFlake();
            }, 50);
        },

        // Stop the snowfall and clean up
        stop: function() {
            // Stop creating new flakes
            clearInterval(this.interval);

            // Remove all existing flakes from the DOM
            this.flakes.forEach(function(f) {
                f.remove();
            });

            // Clear the tracking array
            this.flakes = [];
        },

        // Create a single snowflake
        createFlake: function() {
            var el = document.createElement('div');

            // Random size between 2-6 pixels
            var size = Math.random() * 4 + 2;

            // Random starting X position across the screen
            var startX = Math.random() * window.innerWidth;

            // Fall duration: 2000-5000ms (slower = more peaceful)
            var duration = Math.random() * 3000 + 2000;

            // Horizontal drift: -50 to +50 pixels
            var drift = Math.random() * 100 - 50;

            // Apply styles
            Object.assign(el.style, {
                position: 'fixed',
                width: size + 'px',
                height: size + 'px',
                backgroundColor: 'white',
                borderRadius: '50%',         // Circular snowflake
                left: startX + 'px',
                top: '-10px',                // Start above viewport
                pointerEvents: 'none',       // Don't block clicks
                zIndex: '2147483647',        // Stay on top

                // Random opacity for depth effect (0.5-1.0)
                opacity: Math.random() * 0.5 + 0.5,

                // Subtle glow effect
                boxShadow: '0 0 3px rgba(255,255,255,0.8)',

                // Linear timing for steady fall
                transition: 'all ' + duration + 'ms linear'
            });

            // Add to DOM and track in array
            document.body.appendChild(el);
            this.flakes.push(el);

            // Trigger the falling animation on next frame
            requestAnimationFrame(function() {
                // Fall to below the viewport
                el.style.top = window.innerHeight + 10 + 'px';

                // Apply horizontal drift
                el.style.left = (startX + drift) + 'px';
            });

            // Clean up after animation completes
            var self = this;
            setTimeout(function() {
                el.remove();

                // Remove from tracking array
                var idx = self.flakes.indexOf(el);
                if (idx > -1) {
                    self.flakes.splice(idx, 1);
                }
            }, duration + 100);
        }
    };

    // Set up keyboard listener for 'S' key
    document.addEventListener('keydown', function(e) {
        // Check for 'S' key without modifiers
        // Ignore if user is typing in an input field
        if (
            e.key.toLowerCase() === 's' &&
            !e.ctrlKey &&
            !e.metaKey &&
            !e.altKey &&
            e.target.tagName !== 'INPUT' &&
            e.target.tagName !== 'TEXTAREA' &&
            !e.target.isContentEditable
        ) {
            window._snow.toggle();
        }
    });

    // Start snowfall immediately when bookmarklet is clicked
    window._snow.toggle();
})();
```

## Minified Version

```javascript
javascript:(function(){if(window._snow){window._snow.toggle();return;}window._snow={active:false,interval:null,flakes:[],toggle:function(){this.active=!this.active;if(this.active){this.start();}else{this.stop();}},start:function(){var self=this;this.interval=setInterval(function(){self.createFlake();},50);},stop:function(){clearInterval(this.interval);this.flakes.forEach(function(f){f.remove();});this.flakes=[];},createFlake:function(){var el=document.createElement('div');var size=Math.random()*4+2;var startX=Math.random()*window.innerWidth;var duration=Math.random()*3000+2000;var drift=Math.random()*100-50;Object.assign(el.style,{position:'fixed',width:size+'px',height:size+'px',backgroundColor:'white',borderRadius:'50%25',left:startX+'px',top:'-10px',pointerEvents:'none',zIndex:'2147483647',opacity:Math.random()*0.5+0.5,boxShadow:'0 0 3px rgba(255,255,255,0.8)',transition:'all '+duration+'ms linear'});document.body.appendChild(el);this.flakes.push(el);requestAnimationFrame(function(){el.style.top=window.innerHeight+10+'px';el.style.left=(startX+drift)+'px';});var self=this;setTimeout(function(){el.remove();var idx=self.flakes.indexOf(el);if(idx>-1)self.flakes.splice(idx,1);},duration+100);}};document.addEventListener('keydown',function(e){if(e.key.toLowerCase()==='s'&&!e.ctrlKey&&!e.metaKey&&!e.altKey&&e.target.tagName!=='INPUT'&&e.target.tagName!=='TEXTAREA'&&!e.target.isContentEditable){window._snow.toggle();}});window._snow.toggle();})();
```

## Technical Notes

- Creates ~20 snowflakes per second (1000ms / 50ms interval)
- Uses linear easing for steady, natural-looking fall
- Tracks all flakes in an array to enable instant cleanup
- Varying opacity (0.5-1.0) creates depth perception
- Box shadow adds a soft glow effect to each flake
- Memory-efficient: flakes are removed from both DOM and tracking array after animation
