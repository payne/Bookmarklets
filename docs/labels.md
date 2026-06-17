# Labels Bookmarklet

Create draggable, editable sticky-note style labels on any web page. Perfect for annotating screenshots, marking areas of interest, or leaving visual notes on pages.

## Features

- Press `L` to create a new label at the center of the screen
- Drag labels anywhere on the page
- Double-click a label to edit its text
- Labels are numbered sequentially (Label 1, Label 2, etc.)
- Yellow sticky-note appearance
- Won't interfere with typing in input fields

## How It Works

The bookmarklet creates a singleton object (`window._labels`) that manages the labeling system. When a label is created, it:

1. Creates a styled div element positioned at the screen center
2. Attaches drag-and-drop functionality via mouse events
3. Enables text editing via double-click and prompt dialog
4. Maintains a counter for sequential numbering

## Annotated Source Code

```javascript
(function() {
    // Check if labels system already exists
    // If so, do nothing (labels persist until page refresh)
    if (window._labels) {
        return;
    }

    // Create the labels singleton object
    window._labels = {
        count: 0,  // Counter for label numbering

        // Create a new label at specified position
        create: function(x, y) {
            var el = document.createElement('div');

            // Increment counter and set initial text
            this.count++;
            el.className = '_label_item';
            el.textContent = 'Label ' + this.count;

            // Apply sticky-note styling
            Object.assign(el.style, {
                position: 'fixed',
                left: x + 'px',
                top: y + 'px',
                padding: '8px 12px',
                backgroundColor: '#ffeb3b',      // Yellow sticky note color
                color: '#333',                   // Dark text for readability
                fontFamily: 'sans-serif',
                fontSize: '14px',
                fontWeight: 'bold',
                borderRadius: '4px',

                // Drop shadow for depth
                boxShadow: '0 2px 8px rgba(0,0,0,0.3)',

                cursor: 'move',                  // Indicate draggability
                zIndex: '2147483647',            // Stay on top
                userSelect: 'none',              // Prevent text selection while dragging
                minWidth: '60px',
                textAlign: 'center'
            });

            // Add to DOM
            document.body.appendChild(el);

            // Enable drag functionality
            this.makeDraggable(el);

            // Enable double-click to edit
            el.addEventListener('dblclick', function() {
                var txt = prompt('Edit label:', el.textContent);
                if (txt !== null) {
                    el.textContent = txt;
                }
            });
        },

        // Add drag functionality to an element
        makeDraggable: function(el) {
            var offsetX, offsetY;
            var dragging = false;

            // Start dragging on mousedown
            el.addEventListener('mousedown', function(e) {
                // Ignore if double-click (for edit functionality)
                if (e.detail > 1) return;

                dragging = true;

                // Calculate offset from element's top-left corner
                offsetX = e.clientX - el.offsetLeft;
                offsetY = e.clientY - el.offsetTop;

                // Prevent text selection
                e.preventDefault();
            });

            // Update position while dragging
            document.addEventListener('mousemove', function(e) {
                if (dragging) {
                    el.style.left = (e.clientX - offsetX) + 'px';
                    el.style.top = (e.clientY - offsetY) + 'px';
                }
            });

            // Stop dragging on mouseup
            document.addEventListener('mouseup', function() {
                dragging = false;
            });
        }
    };

    // Set up keyboard listener for 'L' key
    document.addEventListener('keydown', function(e) {
        // Check for 'L' key without modifiers
        // Ignore if user is typing in an input field
        if (
            e.key.toLowerCase() === 'l' &&
            !e.ctrlKey &&
            !e.metaKey &&
            !e.altKey &&
            e.target.tagName !== 'INPUT' &&
            e.target.tagName !== 'TEXTAREA' &&
            !e.target.isContentEditable
        ) {
            // Create label at center of viewport
            window._labels.create(
                window.innerWidth / 2 - 50,   // Center horizontally (offset by ~half width)
                window.innerHeight / 2 - 20   // Center vertically (offset by ~half height)
            );
        }
    });
})();
```

## Minified Version

```javascript
javascript:(function(){if(window._labels){return;}window._labels={count:0,create:function(x,y){var el=document.createElement('div');this.count++;el.className='_label_item';el.textContent='Label '+this.count;Object.assign(el.style,{position:'fixed',left:x+'px',top:y+'px',padding:'8px 12px',backgroundColor:'%23ffeb3b',color:'%23333',fontFamily:'sans-serif',fontSize:'14px',fontWeight:'bold',borderRadius:'4px',boxShadow:'0 2px 8px rgba(0,0,0,0.3)',cursor:'move',zIndex:'2147483647',userSelect:'none',minWidth:'60px',textAlign:'center'});document.body.appendChild(el);this.makeDraggable(el);el.addEventListener('dblclick',function(){var txt=prompt('Edit label:',el.textContent);if(txt!==null)el.textContent=txt;});},makeDraggable:function(el){var offsetX,offsetY,dragging=false;el.addEventListener('mousedown',function(e){if(e.detail>1)return;dragging=true;offsetX=e.clientX-el.offsetLeft;offsetY=e.clientY-el.offsetTop;e.preventDefault();});document.addEventListener('mousemove',function(e){if(dragging){el.style.left=(e.clientX-offsetX)+'px';el.style.top=(e.clientY-offsetY)+'px';}});document.addEventListener('mouseup',function(){dragging=false;});}};document.addEventListener('keydown',function(e){if(e.key.toLowerCase()==='l'&&!e.ctrlKey&&!e.metaKey&&!e.altKey&&e.target.tagName!=='INPUT'&&e.target.tagName!=='TEXTAREA'&&!e.target.isContentEditable){window._labels.create(window.innerWidth/2-50,window.innerHeight/2-20);}});})();
```

## Technical Notes

- Uses `e.detail > 1` to differentiate double-clicks from single clicks
- `userSelect: 'none'` prevents accidental text selection while dragging
- Event listeners are attached to `document` for reliable drag tracking
- Labels persist until page refresh (no delete functionality)
- The `_label_item` class could be used for custom CSS styling

## Usage Tips

- **Screenshot annotation**: Create labels, position them, then take a screenshot
- **Multiple labels**: Press `L` multiple times to create several labels
- **Custom text**: Double-click any label to change its text
- **Drag anywhere**: Labels can be positioned anywhere on the visible page

## Potential Enhancements

This bookmarklet could be extended with:
- Right-click to delete labels
- Color customization
- Resize handles
- Arrow/connector lines between labels
- Export labels as overlay image
