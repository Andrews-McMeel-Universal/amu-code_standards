# ES6 Coding Standards

We follow the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript), which is sensible and widely adopted, and includes ready-made configs for our linter of choice, [ESLint](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/javascript/es6/linters).

## Javascript Game Development

### Game Development Toolkit Recommendations

There are numerous game development engines for creating HTML5/JS games. A popular engine we recommend considering is [Phaser](https://phaser.io/).

Phaser supports:

- [ES6](https://phaser.io/news/2020/04/modern-javascript-phaser-3-tutorial-part-1), which can be set up with your package manager of choice. We recommend using Parcel, as shown in the linked tutorial.
  - There is also this video tutorial about [using ES6 in Phaser](https://phaser.io/news/2020/03/how-to-use-github-and-es6-tutorial).
- [SVG images](https://phaser.io/examples/v3/search?search=svg), which is our preferred image format due to it's small file size and scalability (can be rendered at any size).
- [Touch inputs](https://phaser.io/examples/v3/search?search=touch), which is necessary for mobile web games.

### amuGame and Handling Page/Game Communication

AMU HTML5/Javascript games are loaded into webpages within iframes. The website handles initializing the game with whatever data is necessary to run, such as the daily level data.

![AMU Game Architecture](diagrams/AMU-Game-Architecture.png)

#### amuGame

Game data unique to AMU will be stored in a Javascript Object for consistent access and organization.

```javascript
amuGame = {
  gameName: String,
  debug: Boolean,
  // etc...
};
```

> All dates and times should be compatible with [UTC format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/UTC). We recommend using [`Date.now()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now) to retrieve the current time.

#### Communication between the Page and the Game

All interactions between the page and game need to be done through [`window.postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) in order to support cross origin requests. These messages are used to exchange data and trigger events, both in the game and on the page.

To ensure the game begins when both the page and game are ready, the initial messages are structured like a handshake. Here is a walkthrough of how to get both the client page and game communicating. First set an event listener and callback to respond to events:

```javascript
// 1. Page

window.addEventListener("message", function (event) {
  // Callback to respond to the message
  handleGameMessage(event);
});
```

Once the client page is ready to listen to events from the game, load the game into a new iframe.

The game should also set a listener as soon as possible, so that it doesn't miss messages from the client page. This illustrates an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE), which will run when it is defined, rather than waiting on a page load event:

```javascript
// 2. Game

(function () {
  window.addEventListener("message", (event) => {
    // Always check the origin of the data before responding to messages
    var parentOrigin =
      window.location != window.parent.location
        ? document.referrer
        : document.location.href;

    // Do not proceed if the message came from an unexpected source
    if (!document.referrer.startsWith(parentOrigin)) {
      return;
    }

    // Callback to respond to the message
    handlePageMessage(event);
  });
})();

// An example of handlePageMessage, which needs to handle any message from the page
// See Messages to the Game below for more information
function handlePageMessage(event) {
  if (event.data.initGame === true) {
    // Dispatch game start event/s
  } else if (event.data.pauseGame == true) {
    // Dispatch game pause event/s
  }
  // Handle other messages as needed
  // ...
}
```

Now that both the client page and the game are capable of listening to one another, the game should dispatch a message to confirm to the page it is ready to receive additional messages:

```javascript
// 3. Game

window.onload = function () {
  // Set a target window for the message, which will be the client page
  const parentOrigin =
    window.location != window.parent.location
      ? document.referrer
      : document.location.href;

  if (document.referrer.startsWith(parentOrigin)) {
    // The game is in an iframe, so send a confirmation message to the client window
    const message = {
      amuGame: {
        windowLoaded: true,
      },
    };
    window.parent.postMessage(message, parentOrigin);
  } else {
    // The game isn't in an iframe, so it's ok to start the game automatically
    // Dispatch game start event/s
    // ...
  }

  // Other game initialization can happen here, like mobile detection and resize listeners
  // ...
};
```

The handshake is now complete, and the client page and game can now confidently exchange data. The first command sent to the game will request it to start loading and send whatever data is needed by the game to load the correct level and save state (if any).

```javascript
// 4. Page

function handleGameMessage(event) {
  if (event.data === amuGame.windowLoaded) {
    const message = { initGame: true };
    const frame = document.getElementById(/* game iframe ID */);
    const targetOrigin = /* game source URL */;

    // Send the initialize event
    frame.contentWindow.postMessage(message, targetOrigin);
  }

  // Handle other events as needed
  // ...
}
```

#### Messages to the Game

An Object that specifies which command or commands and includes necessary data to fulfill that command. A complete message Object looks like this:

```javascript
let message = {
  // Sends instruction to begin game initialization
  initGame: Boolean,

  // Sends instruction to pause the game, same as if the player triggered it
  pauseGame: Boolean,

  // Sends config data for customizing game look and feel
  // This varies per game and is sent as an Object from a JSON response
  loadConfig: Object,

  // Sends level data for each day
  // This varies per game, and is usually in XML or JSON format
  // If loadLevel is sent during an active game, the game should reset and load the new data
  loadLevel: {
    // URL to or Blob of data
    puzzleData: String or Blob,

    // Format: ISO Date (YYYY-MM-DD)
	  puzzleDate: String,
  },

  // Requests save data that represents a replicable play state
  // This varies per game and is sent as an Object from a JSON response
  loadSaveState: {
    // The difficulty mode ("expert" or "casual") when the user finished the game
    // Required
    completionMode: String,

    // True if the user changed difficulty modes during the game
    // Required
    modeChanged: Boolean,

    // Time since game start, excluding paused time
    // Required
    // Format: UTC
    totalPlayTime: Number,

    // Additional data as needed, which varies per game
    // ...
  },

  // Requests data points from the game, see amuGame.data below
  // Options: ["all", "currentScore", "totalPlayTime", "saveState"]
  getData: Array,

  // Subscribes to an event or events emitted from the game, see amuGame.event below
  // Options: ["all", "start", "pause", "resume", "end", "modeChange"]
  onEvent: Array,
};

// Sending a message to the game
frame.contentWindow.postMessage(message, targetOrigin);
```

#### Messages from the Game

The `amuGame` Object that contains data needed by the page. It is important to include the `amuGame` key as part of the response, which the page uses to identify responses from game. A complete message Object looks like this:

```javascript
let message = {
  amuGame: {
    // The name of the game
    gameName: String,

    // Confirmation that the game's window.onload is completed
    // This is used to to verify the game is ready to receive and act on messages from the page
    // Default: false
    windowLoaded: Boolean,

    // If the page requests "all", the game should send all data points
    // Otherwise it should send only the requested data
    data: {
      // Used to recreate current game progress for the current level
      // Level data is stored separately, and is not expected as part of this Object
      saveState: {
        // The difficulty mode ("expert" or "casual") when the user finished the game
        // Required
        completionMode: String,

        // True if the user changed difficulty modes during the game
        // Required
        modeChanged: Boolean,

        // Time since game start, excluding paused time
        // Required
        // Format: UTC
        totalPlayTime: Number,

        // Additional data as needed, which varies per game
        // ...
      },
    },

    // If the page requests "all", the game should emit all events
    // Otherwise it should emit only the requested events
    event: {
      // Time the game was started
      // Format: UTC
      start: Number,

      // Time the game was paused
      // Format: UTC
      pause: Number,

      // Time the game was restarted
      // Format: UTC
      resume: Number,

      // Time the game was ended
      // Format: UTC
      end: Number,

      // The difficulty mode ("expert" or "casual") change history
      modeChange: {
        previousMode: String,
        currentMode: String,
      },
    },
  },
};

// Sending amuGame to the page:
frame.contentWindow.postMessage({ amuGame.event.start }, targetOrigin);
```
