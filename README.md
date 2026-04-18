

A modern, simple website to check Islamic prayer times for any city/country combination on a selected date.

## Features

- Beautiful, responsive interface.
- Search by **country**, **city**, and **date**.
- Prayer times update when the selected date changes.
- Shows: Fajr, Sunrise, Dhuhr, Asr, Maghrib, and Isha.
- Uses the AlAdhan API for prayer-time data.

## Run locally

Because this app uses browser `fetch`, run it from a local server:

```bash
python3 -m http.server 8000
```

Then open:

- <http://localhost:8000>

## Files

- `index.html` — page structure.
- `styles.css` — UI styling.
- `app.js` — logic for calling prayer-time API and rendering results
