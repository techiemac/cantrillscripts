# cantrillscripts

A set of scripts to augment [Adrian Cantrill's](cantrill.io) excellent lessons. This was fueled by a caffine
induced coding spree so consider this code the ravings of a madman. PRs are always accepted, especially if 
they somehow involve trash pandas.

## Overview and Getting Started

This script uses [tampermonkey](https://www.tampermonkey.net/) to modify the lesson video player on cantrill.io. 
The script will remove the gradient overlay from the player and make the controls more minimal, allowing you
to bask in the glorious content. Additionally, this enables keyboard shortcuts so you don't need to click on the
video window first to enable the shortcuts. Additional shortcuts have been added, because why not.

To start, install tampermonkey and then add cantrillvid.js manually.

In the future this may be installable from [greasyfork](https://greasyfork.org/). Currently greasyfork does not
recognize `video.min.js` as an approved external script, likely because it's hoested by `zencdn.net`. I am going 
to either look for a CDN currently hosting this in the approved list or see if I can pester the fine folks over
at greasyfork to approve this. `cdn.js` does currently host video.js and is on the approved list.

## Usage

There are additional keyboard shortcuts. They are:

| Key | Description                                                |
|-----|------------------------------------------------------------|
| p   | Play/Pause (because the spacebar would cause scrolling)    |
| 1   | Playback at 0.5 speed                                      |
| 2   | Playback at 0.75 speed                                     |
| 3   | Playback at 1.0 speed                                      |
| 4   | Playback at 1.25 speed                                     |
| 5   | Playback at 1.5 speed                                      |
| 6   | Playback at 2.0 speed (AAAAALVIN!)                         |
| ←   | Back 10 seconds                                            |
| f   | Fullscreen (if supported, the iframe needs to be in focus) |

To modify the behavior, say if you like the gradient overlay. Click on tampermonkey and then click on Create New Script.
Then click on Installed Userscripts and click on Cantrill Video Control. There are 3 JavaScript const's at the top.

```javascript
// Disable gradient overlay
const disableGradientOverlay = true;
// Uses minimal video controls
const useMinimalControls = true;
// Enable additional keyboard controls
const useAdditionalKeyboardControls = true;
```

These should (hopefully) be self explanatory. Flip any to false to disable that functionality.

## Dependencies

This relies on the following dependencies:
* [jQuery](https://jquery.com/) - Manipulation of the DOM because I'm lazy :)
* [video.js](https://videojs.com/) - What is used by Cantrill's courses under the hood.

## Internals

The video player on Cantrill's lessons uses [video.js](https://docs.videojs.com/) under the covers. This is complicated
by the fact it's in an iframe so good old [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) 
applies. To get around this, the tampermonkey script actually runs twice(!). Once in the main window and then in the iframe
video player window.

When the script loads, not all objects are populated in the DOM. The script then uses a utility function called `waitForElement` which
waits for up to 1 second for the object to populate in the DOM. Note to future self, you are probably going to tweak this down the road.

In the iframe, the gradient is removed and the bottom bar is given a more minimal overlay. The padding, width, and height of the buttons
is then tweaked which makes the lower bar much more compact. This is potentially going to cause issues with tablet/phone users which is
why it can be disabled by setting `useMinimalControls` to false.

The iframe then sets up a [key event listener](https://developer.mozilla.org/en-US/docs/Web/API/Element/keyup_event) and a 
[postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) listener. This sets the state of the video.js object.

The main window sets up a key event listener that it forwards using postMessage to the iframe.