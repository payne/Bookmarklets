# Laser Pointer Bookmarklet

Adds a Google Slides-style laser pointer to any web page. A glowing red dot follows your cursor, making it easy to highlight content during presentations or screen sharing.

## Features

- Press `L` to toggle the laser pointer on/off
- Red glowing dot follows your cursor
- Normal cursor is hidden while laser is active
- Won't interfere with typing in input fields or textareas
- Works on any webpage

## How It Works

The bookmarklet creates a singleton object (`window._laserPointer`) that manages the laser pointer state. When activated, it:

1. Creates a fixed-position div styled as a glowing red dot
2. Listens for mouse movements to update the dot's position
3. Listens for the `L` key to toggle visibility
4. Hides the normal cursor when active

## Annotated Source Code

```javascript
(function() {
    // Check if laser pointer already exists on this page
    // If so, just toggle it instead of reinitializing
    if (window._laserPointer) {
        window._laserPointer.toggle();
        return;
    }

    // Create the laser pointer singleton object
    window._laserPointer = {
        active: false,  // Current state (on/off)
        dot: null,      // Reference to the DOM element

        // Initialize the laser pointer
        init: function() {
            // Create the dot element
            this.dot = document.createElement('div');
            this.dot.id = 'laser-pointer-dot';

            // Style the dot to look like a laser pointer
            Object.assign(this.dot.style, {
                position: 'fixed',           // Stay in viewport
                width: '12px',
                height: '12px',
                borderRadius: '50%',         // Make it circular
                backgroundColor: '#ff0000',  // Red color

                // Create the glowing effect with multiple box shadows
                boxShadow: '0 0 10px 3px rgba(255,0,0,0.7), 0 0 20px 6px rgba(255,0,0,0.4)',

                pointerEvents: 'none',       // Don't block clicks
                zIndex: '2147483647',        // Maximum z-index to stay on top
                display: 'none',             // Hidden by default

                // Center the dot on the cursor position
                transform: 'translate(-50%, -50%)',

                // Smooth opacity transitions
                transition: 'opacity 0.1s'
            });

            // Add the dot to the page
            document.body.appendChild(this.dot);

            // Set up event listeners
            document.addEventListener('mousemove', this.onMove.bind(this));
            document.addEventListener('keydown', this.onKey.bind(this));
        },

        // Toggle the laser pointer on/off
        toggle: function() {
            this.active = !this.active;

            // Show/hide the dot
            this.dot.style.display = this.active ? 'block' : 'none';

            // Hide/show the normal cursor
            document.body.style.cursor = this.active ? 'none' : '';
        },

        // Handle mouse movement - update dot position
        onMove: function(e) {
            if (this.active) {
                // Position the dot at the cursor location
                this.dot.style.left = e.clientX + 'px';
                this.dot.style.top = e.clientY + 'px';
            }
        },

        // Handle keyboard input
        onKey: function(e) {
            // Check for 'L' key without modifiers
            // Also ignore if user is typing in an input field
            if (
                e.key.toLowerCase() === 'l' &&
                !e.ctrlKey &&
                !e.metaKey &&
                !e.altKey &&
                e.target.tagName !== 'INPUT' &&
                e.target.tagName !== 'TEXTAREA' &&
                !e.target.isContentEditable
            ) {
                this.toggle();
            }
        }
    };

    // Initialize and activate the laser pointer
    window._laserPointer.init();
    window._laserPointer.toggle();
})();
```

## Minified Version

```javascript
javascript:(function(){if(window._laserPointer){window._laserPointer.toggle();return;}window._laserPointer={active:false,dot:null,init:function(){this.dot=document.createElement('div');this.dot.id='laser-pointer-dot';Object.assign(this.dot.style,{position:'fixed',width:'12px',height:'12px',borderRadius:'50%',backgroundColor:'#ff0000',boxShadow:'0 0 10px 3px rgba(255,0,0,0.7), 0 0 20px 6px rgba(255,0,0,0.4)',pointerEvents:'none',zIndex:'2147483647',display:'none',transform:'translate(-50%, -50%)',transition:'opacity 0.1s'});document.body.appendChild(this.dot);document.addEventListener('mousemove',this.onMove.bind(this));document.addEventListener('keydown',this.onKey.bind(this));},toggle:function(){this.active=!this.active;this.dot.style.display=this.active?'block':'none';document.body.style.cursor=this.active?'none':'';},onMove:function(e){if(this.active){this.dot.style.left=e.clientX+'px';this.dot.style.top=e.clientY+'px';}},onKey:function(e){if(e.key.toLowerCase()==='l'&&!e.ctrlKey&&!e.metaKey&&!e.altKey&&e.target.tagName!=='INPUT'&&e.target.tagName!=='TEXTAREA'&&!e.target.isContentEditable){this.toggle();}}};window._laserPointer.init();window._laserPointer.toggle();})();
```

## Usage Tips

- Great for presentations, screen sharing, or video tutorials
- The laser stays visible over all page content due to maximum z-index
- Toggle off when you need to type to avoid conflicts with the `L` key
