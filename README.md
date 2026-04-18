# prayertime
its a website you can see with it islamic prayer times
diff --git a/README.md b/README.md
index 34e064d393942036e69ef95726ea6fca05536146..5411362530d8c06f20a1001e63dcd7543e380320 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,29 @@
-# umop
-about a,imals
+# Prayer Time Finder
+
+A modern, simple website to check Islamic prayer times for any city/country combination on a selected date.
+
+## Features
+
+- Beautiful, responsive interface.
+- Search by **country**, **city**, and **date**.
+- Prayer times update when the selected date changes.
+- Shows: Fajr, Sunrise, Dhuhr, Asr, Maghrib, and Isha.
+- Uses the AlAdhan API for prayer-time data.
+
+## Run locally
+
+Because this app uses browser `fetch`, run it from a local server:
+
+```bash
+python3 -m http.server 8000
+```
+
+Then open:
+
+- <http://localhost:8000>
+
+## Files
+
+- `index.html` — page structure.
+- `styles.css` — UI styling.
+- `app.js` — logic for calling prayer-time API and rendering results.
