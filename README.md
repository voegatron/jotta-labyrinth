# Find de Buddy! – Jotta Labyrinth

A browser-based maze game for kids. Help Jotta find his buddy Yumi through procedurally generated mazes.

## Features

- 5 levels with increasing maze size (7x7 up to 15x15)
- Keyboard arrows, on-screen buttons, and touch swipe support
- Step counter and level timer
- Star rating per level based on efficiency
- Personal best score saved in browser (localStorage)
- Global top 3 leaderboard per level via Firebase Realtime Database

## Project structure

```
jotta-labyrinth/
├── index.html       # Game + all JavaScript
├── style.css        # Styles
├── static/
│   ├── player.png   # Player character image
│   └── exit.png     # Exit/buddy image
└── README.md
```

## Local development

Open `index.html` directly in a browser (no server needed).

Make sure the image path at the top of the `<script>` in `index.html` is set to:

```js
const IMAGE_PATH = './static/'; // local: './static/'  |  server: '/static/'
```

## Deployment (FTP)

1. Change `IMAGE_PATH` to `'/static/'` before uploading
2. Upload all files maintaining the same folder structure:
   - `index.html`
   - `style.css`
   - `static/player.png`
   - `static/exit.png`

## Firebase Leaderboard

The global leaderboard uses [Firebase Realtime Database](https://firebase.google.com/products/realtime-database) (free Spark plan).

### Setup (already done)

- Project: `jotta-labyrinth` on Firebase
- Database URL: `https://jotta-labyrinth-default-rtdb.europe-west1.firebasedatabase.app`
- Data is stored under `/scores/level_0`, `/scores/level_1`, etc.
- Scores are fetched and submitted via the Firebase REST API (no SDK needed)

### Database rules

In the Firebase Console under **Build → Realtime Database → Rules**, the rules should be:

```json
{
  "rules": {
    "scores": {
      ".read": true,
      "level_$lvl": {
        "$entry": {
          ".write": "!data.exists()",
          ".validate": "newData.hasChildren(['name', 'steps', 'time', 'ts'])
            && newData.child('name').isString()
            && newData.child('name').val().length >= 1
            && newData.child('name').val().length <= 12
            && newData.child('steps').isNumber()
            && newData.child('steps').val() > 0
            && newData.child('time').isNumber()
            && newData.child('time').val() >= 0"
        }
      }
    }
  }
}
```

These rules:
- Allow anyone to **read** scores
- Allow only **new entries** — existing scores cannot be overwritten or deleted
- **Validate** that submitted data has the correct structure and types

### How scores are stored

Each score entry has:

```json
{
  "name": "Urs",
  "steps": 14,
  "time": 37,
  "ts": 1712345678000
}
```

- `steps` — number of moves taken
- `time` — seconds elapsed
- `ts` — Unix timestamp (milliseconds) of when the score was submitted

### Player name

On first visit the player is asked for their name. It is saved in `localStorage` under the key `jotta_name` and reused on subsequent visits.

To reset the name (e.g. to switch player), run in the browser console:
```js
localStorage.removeItem('jotta_name')
```
Then reload the page.
