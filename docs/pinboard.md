# Pinboard.in Bookmarklet

A utility bookmarklet for saving web pages to [Pinboard.in](https://pinboard.in), a paid bookmarking service. This is Pinboard's official bookmarklet for quickly saving links.

## Features

- Saves the current page URL to Pinboard
- Pre-fills the title with the page title
- Uses any selected text as the description
- Opens Pinboard's "add bookmark" dialog in a popup window
- Shows tag suggestions for easier organization

## How It Works

The bookmarklet:

1. Captures the current page URL
2. Gets any text the user has selected on the page
3. Gets the page title
4. Opens a Pinboard popup window with all fields pre-filled

## Annotated Source Code

```javascript
// Capture the current page URL
q = location.href;

// Get any text the user has selected
// Uses document.getSelection() if available, otherwise empty string
if (document.getSelection) {
    d = document.getSelection();
} else {
    d = '';
}

// Get the page title
p = document.title;

// Open Pinboard's add bookmark dialog in a popup window
// void() prevents the page from navigating away
void(
    open(
        // Pinboard URL with query parameters
        'https://pinboard.in/add' +
        '?showtags=yes' +                              // Show tag suggestions
        '&url=' + encodeURIComponent(q) +              // Page URL
        '&description=' + encodeURIComponent(d) +      // Selected text
        '&title=' + encodeURIComponent(p),             // Page title

        'Pinboard',                                     // Window name

        // Popup window options
        'toolbar=no,scrollbars=yes,width=750,height=700'
    )
);
```

## Minified Version

```javascript
javascript:q=location.href;if(document.getSelection){d=document.getSelection();}else{d='';};p=document.title;void(open('https://pinboard.in/add?showtags=yes&url='+encodeURIComponent(q)+'&description='+encodeURIComponent(d)+'&title='+encodeURIComponent(p),'Pinboard','toolbar=no,scrollbars=yes,width=750,height=700'));
```

## Usage Tips

- **Select text before clicking**: Any text you select on the page will be used as the bookmark description
- **Login first**: Make sure you're logged into Pinboard.in for the bookmarklet to work
- **Popup blockers**: You may need to allow popups from the site you're bookmarking

## About Pinboard.in

Pinboard is a fast, no-nonsense bookmarking site for people who value speed and privacy. Unlike free bookmarking services, Pinboard:

- Has no ads
- Doesn't track you
- Offers full-text search of bookmarked pages
- Provides reliable, long-term archiving
- Has a clean, minimal interface

The service requires a one-time payment (approximately $22) to use.

## Alternative: Browser Extension

Pinboard also offers browser extensions that provide additional features like:
- Keyboard shortcuts
- Quick tagging
- Private bookmark toggle
- Read later functionality

Visit [pinboard.in/tour](https://pinboard.in/tour/) for more information.
