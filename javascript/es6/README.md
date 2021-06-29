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

AMU HTML5/Javascript games are loaded into webpages within iframes. The website handles initializing the game with whatever data is necessary to run, such as the daily level data or game config data.

![AMU Game Architecture](diagrams/AMU-Game-Architecture.png)

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

The page communicates with the game via a Javascript Object that gives a command or commands and includes necessary data to fulfill that command.

| Property                            | Type                                | Description                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `initGame`                          | boolean                             | Sends instruction to begin game initialization. If no `initGame` data is sent and the game is not in an iframe, the game is expected to initialize automatically.                                                                                                                                                                                                                                                               |
| `loadLevel`                         | object                              | Sends data about the selected game level. If `loadLevel` is sent during an active game, the game should reset and load the new data. If no `loadLevel` data is sent, the game is expected to load static test or sample data.                                                                                                                                                                                                   |
| `loadLevel.issueDate`               | string (required)                   | The date associated with a given level. Expected format: ISO 8601 Calendar Date, `"YYYY-MM-DD"`                                                                                                                                                                                                                                                                                                                                 |
| `loadLevel.files`                   | array containing objects (required) | The available data files for a given level.                                                                                                                                                                                                                                                                                                                                                                                     |
| `loadLevel.files.url`               | string (required)                   | The URL to the file.                                                                                                                                                                                                                                                                                                                                                                                                            |
| `loadLevel.files.mimeType`          | string (required)                   | The mime type of the file.                                                                                                                                                                                                                                                                                                                                                                                                      |
| `loadLevel.files.originalFileName`  | string (required)                   | The filename with extension. This does not need to be appended to the URL and can be used to identify the file.                                                                                                                                                                                                                                                                                                                 |
| `loadConfig`                        | object                              | Sends config data for customizing game look and feel. This varies per game and may include a `config.json` containing game settings, and/or image files that are used to override default images (see `loadConfig.files.originalFileName` below for more information). If available, `loadConfig` data is sent to the game with `loadLevel`. If no `loadConfig` data is sent, the game is expected to load a default config.    |
| `loadConfig.name`                   | string                              | The internal product identifier for this set of config files.                                                                                                                                                                                                                                                                                                                                                                   |
| `loadConfig.files`                  | array containing objects (required) | The available data files for a given config.                                                                                                                                                                                                                                                                                                                                                                                    |
| `loadConfig.files.url`              | string (required)                   | The URL to the file.                                                                                                                                                                                                                                                                                                                                                                                                            |
| `loadConfig.files.mimeType`         | string (required)                   | The mime type of the file.                                                                                                                                                                                                                                                                                                                                                                                                      |
| `loadConfig.files.originalFileName` | string (required)                   | The filename with extension. This does not need to be appended to the URL and can be used to identify the file. If the file is `config.json`, it will contain settings to override default game properties, usually colors, sizes, and text strings, as defined by the game. If the file is an image, the name will match an existing image within the game, indicating that image file should be used instead of the original. |
| `loadState`                         | object                              | Sends state data that represents a replicable moment during a user's game session. This varies per game and will be a copy of what is received in `amuGame.state`. If available, `loadState` data is sent with `loadLevel`. If no `loadState` data is sent with `loadLevel`, the game is expected to start new session.                                                                                                         |
| `pauseGame`                         | boolean                             | If `true`, sends instruction to pause the game, the same as if triggered by a user.                                                                                                                                                                                                                                                                                                                                             |
| `getState`                          | boolean                             | Requests current `amuGame.state` data from the game. See `amuGame.state` below for more information.                                                                                                                                                                                                                                                                                                                            |
| `onEvent`                           | array containing strings            | Subscribes to an event or events emitted from the game per `amuGame.event.type` options below. If the page requests `"all"`, the game should emit data for all events, otherwise it should only emit data that matches one of the listed event `types`, such as `"change mode"` or `"print"`. When emitting events, the game should also include `amuGame.state`.                                                               |

An example complete message Object looks like this:

```javascript
let message = {
  initGame: true,
  loadLevel: {
    issueDate: "2021-01-31",
    files: [
      {
        url: "https://fileserver/123456",
        mimeType: "text/xml",
        originalFileName: "myfile.xml",
      },
    ],
  },
  loadConfig: {
    name: "productname",
    files: [
      {
        url: "https://fileserver/config.json",
        mimeType: "application/json",
        originalFileName: "config.json",
      },
      {
        url: "https://fileserver/ImageName.png",
        mimeType: "image/png",
        originalFileName: "ImageName.png",
      },
    ],
  },
  loadState: {
    // example game specific save data
    snapshot: {
      cells["A", "B", "C"],
    },
    isCompleted: false,
    madeMistakes: false,
    resetLevel: false,
    totalPlayTime: 1234456789,
    totalScore: 47,
    completionMode: "casual",
    differencesSpotted: 0,
    difficultyRating: 5,
    earnedPerfectScore: false,
    enabledDarkMode: false,
    modeChanged: false,
    orderSolved: [1, 2, 3, 5, 4],
    scoreBonuses: {
      bonus1x: 0,
      bonus2x: 1,
      bonus3x: 2,
      bonus4x: 3,
      bonus5x: 4,
    },
    usedHints: false,
    wordsSolved: 3,
  },
  pauseGame: false,
  getState: true,
  onEvent: ["end", "change mode"],
};
```

Usage examples:

```javascript
// Sending the game init and loadLevel commands to the game:
frame.contentWindow.postMessage(
  {
    initGame: true,
    loadLevel: {
      issueDate: "2021-01-31",
      files: [
        {
          url: "https://fileserver/123456",
          mimeType: "text/xml",
          originalFileName: "myfile.xml",
        },
      ],
    },
  },
  targetOrigin
);

// Sending the game getState command to the game:
frame.contentWindow.postMessage({ getState: true }, targetOrigin);

// Sending the entire message to the game:
frame.contentWindow.postMessage(message, targetOrigin);
```

#### Messages from the Game

The game communicates with the page via the `amuGame` Javascript Object and contains data needed by the page. It is important to include the `amuGame` key as part of the response, which the page uses to identify responses from game. For each data point, note the **strict type/s** and whether or not it is **required**. If a data point is not applicable to every game, or if it refers to an event that hasn't occurred yet (such as completing the game), it will not be required and should be omitted.

| Property                           | Type                        | Description                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `amuGame.windowLoaded`             | boolean                     | Sends confirmation to the page that the game's `window.onload` is completed, indicating the game is ready to recieve and act on messages from the page. The page will only send messages to the game after it receives `amuGame.windowLoaded: true`.                                                                                                 |
| `amuGame.event`                    | object                      | Emits requested event data to the page per `amuGame.event.type` options below. If the page requests `"all"`, the game should emit data for all available events, otherwise it should emit only the requested event data that matches an available event `type`, such as `"start"`. When emitting an event, also include the current `amuGame.state`. |
| `amuGame.event.type`               | string (required)           | The event name, available options are: `"start"`, `"pause"`, `"resume"`, `"end"`, `"print"`, `"help"`, `"change mode"`, `"change settings"`, `"spot difference"`, `"solve word"`                                                                                                                                                                     |
| `amuGame.event.timeStamp`          | string (required)           | Time the event occurred, use `new Date().toISOString()` to determine the current date and time. Expected format: ISO 8601 with zero UTC offset, `"YYYY-MM-DDTHH:mm:ss.sssZ"`                                                                                                                                                                         |
| `amuGame.event.details`            | string (required)           | If applicable, additional details about the event.                                                                                                                                                                                                                                                                                                   |
| `amuGame.state`                    | object                      | Sends all state data to the page after receiving the `getState` command. State should also be sent with any event, see `amuGame.event` below.                                                                                                                                                                                                        |
| `amuGame.state.snapshot`           | any (required)              | Used to recreate current game progress for the current level. The shape and contents of this data point is flexible and can be whatever is needed by the game, including (but not limited to) nested data, objects, and arrays. Level data is stored and sent to the game separately, and should not be needed to save the user's progress.          |
| `amuGame.state.isCompleted`        | boolean (required)          | `true` if the user completed this level and the game is final.                                                                                                                                                                                                                                                                                       |
| `amuGame.state.madeMistakes`       | boolean (required)          | `true` if the user entered an incorrect value during the game.                                                                                                                                                                                                                                                                                       |
| `amuGame.state.resetLevel`         | boolean (required)          | `true` if the user reset the game state. This should persist in the game state once set to `true`, even if other values are reset.                                                                                                                                                                                                                   |
| `amuGame.state.totalPlayTime`      | number (required)           | Milliseconds since game start, excluding paused time.                                                                                                                                                                                                                                                                                                |
| `amuGame.state.totalScore`         | number or string (required) | The total points earned when the game is completed. If the game doesn't use numerical points, it should return the score as a string. For instance, if scores are time-based, it should be formatted like `"HH:mm:ss.sss"`.                                                                                                                          |
| `amuGame.state.completionMode`     | string                      | The difficulty mode (`"expert"` or `"casual"`) when the user completed the game.                                                                                                                                                                                                                                                                     |
| `amuGame.state.differencesSpotted` | number                      | The number of differences the user identified.                                                                                                                                                                                                                                                                                                       |
| `amuGame.state.difficultyRating`   | number                      | The difficulty number for this game level.                                                                                                                                                                                                                                                                                                           |
| `amuGame.state.earnedPerfectScore` | boolean                     | `true` if the user earned the highest possible score once the game is completed.                                                                                                                                                                                                                                                                     |
| `amuGame.state.enabledDarkMode`    | boolean                     | `true` if the user enabled dark mode during the game.                                                                                                                                                                                                                                                                                                |
| `amuGame.state.modeChanged`        | boolean                     | `true` if the user changed difficulty modes during the game.                                                                                                                                                                                                                                                                                         |
| `amuGame.state.orderSolved`        | array containing numbers    | Track the order the in which the user solved the game's puzzles.                                                                                                                                                                                                                                                                                     |
| `amuGame.state.scoreBonuses`       | object                      | The types and number of bonus modifiers the user earned.                                                                                                                                                                                                                                                                                             |
| `amuGame.state.usedHints`          | boolean                     | `true` if the user used hint functionality.                                                                                                                                                                                                                                                                                                          |
| `amuGame.state.wordsSolved`        | number                      | The number of words the user successfully solved.                                                                                                                                                                                                                                                                                                    |

An example complete message Object looks like this:

```javascript
let message = {
  amuGame: {
    windowLoaded: true,
    event: {
      type: "mode change",
      timeStamp: "2021-01-31T12:34:56.789Z",
      details: "expert",
    },
    state: {
      // example game specific save data
      snapshot: {
        cells["A", "B", "C"],
      },
      isCompleted: false,
      madeMistakes: false,
      resetLevel: false,
      totalPlayTime: 1234456789,
      totalScore: 47,
      completionMode: "casual",
      differencesSpotted: 0,
      difficultyRating: 5,
      earnedPerfectScore: false,
      enabledDarkMode: false,
      modeChanged: false,
      orderSolved: [1, 2, 3, 5, 4],
      scoreBonuses: {
        bonus1x: 0,
        bonus2x: 1,
        bonus3x: 2,
        bonus4x: 3,
        bonus5x: 4,
      },
      usedHints: false,
      wordsSolved: 3,
    },
  },
};
```

Usage examples:

```javascript
// Sending amuGame window loaded confirmation to the page:
frame.contentWindow.postMessage({ amuGame.windowLoaded }, targetOrigin);

// Sending amuGame event and state data to the page:
frame.contentWindow.postMessage({ amuGame.event, amuGame.state }, targetOrigin);

// Sending the entire amuGame object:
frame.contentWindow.postMessage(message, targetOrigin);
```
