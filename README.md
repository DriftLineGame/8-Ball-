# 🎱 Rack Tracker

The 8-ball league between Linden and Malachi. One HTML file, hosted on GitHub
Pages, with Firebase Firestore keeping every phone in sync.

## Files

```
index.html   the whole app
README.md    this file
```

## Firestore security rules

Paste these into **Firestore Database → Rules** and press **Publish**. They
only accept real games (a winner of `linden` or `malachi`, 0–7 balls left, a
short note), so nothing else can be written to your database.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /leagues/{league}/games/{gameId} {
      allow read, delete: if true;
      allow create, update: if
        request.resource.data.keys().hasOnly(['winner','ballsLeft','playedAt','note','created'])
        && request.resource.data.winner in ['linden', 'malachi']
        && request.resource.data.ballsLeft is int
        && request.resource.data.ballsLeft >= 0
        && request.resource.data.ballsLeft <= 7
        && request.resource.data.playedAt is int
        && request.resource.data.note is string
        && request.resource.data.note.size() <= 60;
    }

    match /leagues/{league}/seasons/{seasonId} {
      allow read: if true;
      allow create: if request.resource.data.games is list
        && request.resource.data.games.size() <= 5000;
    }
  }
}
```

## Connecting Firebase

Near the top of `index.html`:

```js
window.RACK_FIREBASE_CONFIG = null;
```

Replace `null` with the `firebaseConfig` object from your Firebase project.
While it's `null`, games save on that phone only.

The Firebase web config isn't a password — Google designs it to sit in public
web pages. The rules above are what protect the data.

## Sync status

The pill in the top-right corner shows:

- **Live** — connected; new games appear on every phone within a second
- **Saving…** — a change is on its way to the league
- **Offline, will sync** — no signal; games are kept and sent when you reconnect
- **Blocked by rules** — the Firestore rules above weren't published
- **This phone only** — no Firebase config yet

## League rules

- **Balls left** = how many balls the *loser* still had (0–7). Tap the 8-ball
  when they were down to the 8.
- **Ranking** = win rate, highest first.
- **Tiebreak** = if win rates match, the lower average balls left on wins ranks higher.

## Data layout

```
leagues/main/games/{id}    winner, ballsLeft, playedAt, note, created
leagues/main/seasons/{id}  endedAt, startedAt, champion, games[]
```

Games logged on a phone before sync was set up are offered for upload once,
the first time that phone connects to an empty league.

## Changelog

### 3.1.0
- Official standings table: position, games played, W, L, win percentage (.636), average balls left and last-five form
- Seasons are numbered (header, reports, past seasons, recap)
- History entries are numbered as games
- Emojis replaced with line icons and a brass crown mark
- Formal wording: season records, largest margin, closest finish, season report
- Recap finale uses light rays instead of confetti

### 3.0.1
- Connected to the league's Firebase project (`rack-tracker-fa690`)

### 3.0.0
- Firebase Firestore sync: games, edits and deletes show up live on every phone,
  with offline saving and a status pill
- New standings screen built as a pool table, with a ball tray showing the
  last 10 games
- New log sheet: tap the balls to set how many were left, pick the date the
  game was played, quick-note chips, and a live summary before saving
- Edit or delete any game from History, with Undo on every change
- Past seasons, synced across phones, each with a PDF
- End the season now refuses to archive unless the PDF was created
- League intro rebuilt: 15 seconds, opening with a rack-break animation
- Redesigned PDF with standings table, highlights and a full game log
- Dark mode, reduced-motion support, keyboard focus styles

### 2.x
- Trends graph, streaks, highlights, notes, League Intro, PDF export,
  End League, local storage only
