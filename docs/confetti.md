# Confetti Bookmarklet

Triggers a colorful confetti explosion on any web page. Perfect for celebrations, achievements, or just adding some fun to your browsing experience.

## Features

- Press `C` to trigger a confetti burst
- 80 colorful pieces in rainbow colors
- Pieces fall with random rotation and drift
- Automatic cleanup after animation completes
- Won't interfere with typing in input fields

## How It Works

The bookmarklet creates a singleton object (`window._confetti`) that manages the confetti system. When triggered, it:

1. Creates 80 individual confetti pieces as div elements
2. Each piece gets random color, size, position, and rotation
3. CSS transitions animate the pieces falling with physics-like curves
4. Pieces automatically remove themselves after the animation

## Annotated Source Code

```javascript
(function() {
    // Initialize confetti system if it doesn't exist
    if (!window._confetti) {
        window._confetti = {
            // Rainbow color palette (URL-encoded hex colors)
            colors: [
                '#ff0000',  // Red
                '#ff7700',  // Orange
                '#ffdd00',  // Yellow
                '#00ff00',  // Green
                '#0099ff',  // Blue
                '#6600ff',  // Purple
                '#ff00ff'   // Magenta
            ],

            // Fire a burst of confetti
            fire: function() {
                var count = 80;  // Number of confetti pieces
                for (var i = 0; i < count; i++) {
                    this.createPiece();
                }
            },

            // Create a single confetti piece
            createPiece: function() {
                var el = document.createElement('div');

                // Random color from palette
                var color = this.colors[Math.floor(Math.random() * this.colors.length)];

                // Random size between 4-12 pixels
                var size = Math.random() * 8 + 4;

                // Start position: random X across screen, above viewport
                var startX = Math.random() * window.innerWidth;
                var startY = -20;

                // End position: drifts left/right up to 200px, falls below viewport
                var endX = startX + (Math.random() - 0.5) * 400;
                var endY = window.innerHeight + 20;

                // Random initial rotation
                var rotation = Math.random() * 360;

                // Animation duration: 800-1300ms
                var duration = Math.random() * 500 + 800;

                // Apply styles to the confetti piece
                Object.assign(el.style, {
                    position: 'fixed',
                    width: size + 'px',

                    // Height varies to create rectangular pieces too
                    height: size * (Math.random() * 0.5 + 0.5) + 'px',

                    backgroundColor: color,
                    left: startX + 'px',
                    top: startY + 'px',
                    pointerEvents: 'none',        // Don't block clicks
                    zIndex: '2147483647',         // Stay on top

                    // Randomly circular or square
                    borderRadius: Math.random() > 0.5 ? '50%' : '0',

                    transform: 'rotate(' + rotation + 'deg)',

                    // Easing curve for natural falling motion
                    transition: 'all ' + duration + 'ms cubic-bezier(0.25, 0.46, 0.45, 0.94)',

                    opacity: '1'
                });

                // Add to page
                document.body.appendChild(el);

                // Trigger animation on next frame
                requestAnimationFrame(function() {
                    el.style.left = endX + 'px';
                    el.style.top = endY + 'px';

                    // Add extra rotation during fall
                    el.style.transform = 'rotate(' + (rotation + Math.random() * 360) + 'deg)';

                    // Fade out near the end
                    el.style.opacity = '0';
                });

                // Clean up after animation completes
                setTimeout(function() {
                    el.remove();
                }, duration + 100);
            }
        };

        // Set up keyboard listener for 'C' key
        document.addEventListener('keydown', function(e) {
            // Check for 'C' key without modifiers
            // Ignore if user is typing in an input field
            if (
                e.key.toLowerCase() === 'c' &&
                !e.ctrlKey &&
                !e.metaKey &&
                !e.altKey &&
                e.target.tagName !== 'INPUT' &&
                e.target.tagName !== 'TEXTAREA' &&
                !e.target.isContentEditable
            ) {
                window._confetti.fire();
            }
        });
    }

    // Fire confetti immediately when bookmarklet is clicked
    window._confetti.fire();
})();
```

## Minified Version

```javascript
javascript:(function(){if(!window._confetti){window._confetti={colors:['%23ff0000','%23ff7700','%23ffdd00','%2300ff00','%230099ff','%236600ff','%23ff00ff'],fire:function(){var count=80;for(var i=0;i<count;i++){this.createPiece();}},createPiece:function(){var el=document.createElement('div');var color=this.colors[Math.floor(Math.random()*this.colors.length)];var size=Math.random()*8+4;var startX=Math.random()*window.innerWidth;var startY=-20;var endX=startX+(Math.random()-0.5)*400;var endY=window.innerHeight+20;var rotation=Math.random()*360;var duration=Math.random()*500+800;Object.assign(el.style,{position:'fixed',width:size+'px',height:size*(Math.random()*0.5+0.5)+'px',backgroundColor:color,left:startX+'px',top:startY+'px',pointerEvents:'none',zIndex:'2147483647',borderRadius:Math.random()>0.5?'50%25':'0',transform:'rotate('+rotation+'deg)',transition:'all '+duration+'ms cubic-bezier(0.25, 0.46, 0.45, 0.94)',opacity:'1'});document.body.appendChild(el);requestAnimationFrame(function(){el.style.left=endX+'px';el.style.top=endY+'px';el.style.transform='rotate('+(rotation+Math.random()*360)+'deg)';el.style.opacity='0';});setTimeout(function(){el.remove();},duration+100);}};document.addEventListener('keydown',function(e){if(e.key.toLowerCase()==='c'&&!e.ctrlKey&&!e.metaKey&&!e.altKey&&e.target.tagName!=='INPUT'&&e.target.tagName!=='TEXTAREA'&&!e.target.isContentEditable){window._confetti.fire();}});}window._confetti.fire();})();
```

## Technical Notes

- Uses `cubic-bezier(0.25, 0.46, 0.45, 0.94)` for natural falling motion
- `requestAnimationFrame` ensures smooth animation start
- Colors are URL-encoded (`%23` for `#`) in the minified version
- Pieces are automatically garbage collected after removal
