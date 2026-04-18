index 0000000000000000000000000000000000000000..a929b2edb8e8c613d0756152b5517454175e5b83
--- /dev/null
+++ b/app.js
@@ -0,0 +1,111 @@
+const form = document.getElementById('prayer-form');
+const countryInput = document.getElementById('country');
+const cityInput = document.getElementById('city');
+const dateInput = document.getElementById('date');
+const statusEl = document.getElementById('status');
+const locationTitle = document.getElementById('location-title');
+const dateTitle = document.getElementById('date-title');
+const timezoneBadge = document.getElementById('timezone-badge');
+const prayerGrid = document.getElementById('prayer-grid');
+
+const prayerIds = {
+  Fajr: document.getElementById('fajr'),
+  Sunrise: document.getElementById('sunrise'),
+  Dhuhr: document.getElementById('dhuhr'),
+  Asr: document.getElementById('asr'),
+  Maghrib: document.getElementById('maghrib'),
+  Isha: document.getElementById('isha')
+};
+
+function setStatus(message, type = '') {
+  statusEl.textContent = message;
+  statusEl.className = `status ${type}`.trim();
+}
+
+function toDisplayDate(rawDate) {
+  const parsed = new Date(`${rawDate}T00:00:00`);
+  return parsed.toLocaleDateString(undefined, {
+    weekday: 'long',
+    year: 'numeric',
+    month: 'long',
+    day: 'numeric'
+  });
+}
+
+function stripTimezone(timeValue) {
+  return timeValue.split(' ')[0];
+}
+
+async function fetchPrayerTimes({ country, city, date }) {
+  const targetDate = new Date(`${date}T00:00:00`);
+  const day = targetDate.getDate();
+  const month = targetDate.getMonth() + 1;
+  const year = targetDate.getFullYear();
+
+  const params = new URLSearchParams({
+    city,
+    country,
+    method: '2'
+  });
+
+  const url = `https://api.aladhan.com/v1/timingsByCity/${day}-${month}-${year}?${params.toString()}`;
+  const response = await fetch(url);
+
+  if (!response.ok) {
+    throw new Error('Unable to reach prayer time service.');
+  }
+
+  const payload = await response.json();
+
+  if (payload.code !== 200 || !payload.data?.timings) {
+    throw new Error(payload?.data || 'No data found for this location.');
+  }
+
+  return payload.data;
+}
+
+async function handleSubmit(event) {
+  event.preventDefault();
+
+  const country = countryInput.value.trim();
+  const city = cityInput.value.trim();
+  const date = dateInput.value;
+
+  if (!country || !city || !date) {
+    setStatus('Please fill in country, city, and date.', 'error');
+    return;
+  }
+
+  setStatus('Loading prayer times...');
+
+  try {
+    const result = await fetchPrayerTimes({ country, city, date });
+    const timings = result.timings;
+
+    Object.keys(prayerIds).forEach((key) => {
+      prayerIds[key].textContent = stripTimezone(timings[key]);
+    });
+
+    locationTitle.textContent = `${city}, ${country}`;
+    dateTitle.textContent = toDisplayDate(date);
+    timezoneBadge.textContent = `Timezone: ${result.meta.timezone}`;
+    prayerGrid.hidden = false;
+    setStatus('Prayer times updated successfully.', 'success');
+  } catch (error) {
+    prayerGrid.hidden = true;
+    setStatus(`Could not load prayer times. ${error.message}`, 'error');
+  }
+}
+
+function setupDefaults() {
+  const today = new Date();
+  const todayIso = today.toISOString().split('T')[0];
+
+  countryInput.value = 'United States';
+  cityInput.value = 'New York';
+  dateInput.value = todayIso;
+}
+
+setupDefaults();
+form.addEventListener('submit', handleSubmit);
+form.requestSubmit();
diff --git a/index.html b/index.html
new file mode 100644
index 0000000000000000000000000000000000000000..3b9fb35fb036424a3f9639f0276d08215bfacbf0
--- /dev/null
+++ b/index.html
@@ -0,0 +1,93 @@
+<!doctype html>
+<html lang="en">
+  <head>
+    <meta charset="UTF-8" />
+    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
+    <title>Prayer Time Finder</title>
+    <link rel="preconnect" href="https://fonts.googleapis.com" />
+    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
+    <link
+      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
+      rel="stylesheet"
+    />
+    <link rel="stylesheet" href="styles.css" />
+  </head>
+  <body>
+    <main class="app-shell">
+      <section class="hero-card">
+        <div>
+          <p class="eyebrow">Islamic Prayer Times</p>
+          <h1>Find daily Salah timings for any country</h1>
+          <p class="subtitle">
+            Select a country, city, and date to view accurate prayer times that update
+            automatically for each day.
+          </p>
+        </div>
+
+        <form id="prayer-form" class="controls" autocomplete="off">
+          <label>
+            Country
+            <input id="country" name="country" type="text" placeholder="e.g. United States" required />
+          </label>
+
+          <label>
+            City
+            <input id="city" name="city" type="text" placeholder="e.g. New York" required />
+          </label>
+
+          <label>
+            Date
+            <input id="date" name="date" type="date" required />
+          </label>
+
+          <button type="submit">Get prayer times</button>
+        </form>
+      </section>
+
+      <section class="result-card" aria-live="polite">
+        <div class="result-header">
+          <div>
+            <h2 id="location-title">No location selected</h2>
+            <p id="date-title">Choose a location to see today's timings.</p>
+          </div>
+          <span id="timezone-badge" class="badge">Timezone: —</span>
+        </div>
+
+        <p id="status" class="status">Enter location details and click “Get prayer times”.</p>
+
+        <div id="prayer-grid" class="prayer-grid" hidden>
+          <article class="prayer-item">
+            <h3>Fajr</h3>
+            <p id="fajr">—</p>
+          </article>
+          <article class="prayer-item">
+            <h3>Sunrise</h3>
+            <p id="sunrise">—</p>
+          </article>
+          <article class="prayer-item">
+            <h3>Dhuhr</h3>
+            <p id="dhuhr">—</p>
+          </article>
+          <article class="prayer-item">
+            <h3>Asr</h3>
+            <p id="asr">—</p>
+          </article>
+          <article class="prayer-item">
+            <h3>Maghrib</h3>
+            <p id="maghrib">—</p>
+          </article>
+          <article class="prayer-item">
+            <h3>Isha</h3>
+            <p id="isha">—</p>
+          </article>
+        </div>
+      </section>
+    </main>
+
+    <footer class="footer-note">
+      Data powered by AlAdhan API.
+    </footer>
+
+    <script src="app.js" type="module"></script>
+  </body>
+</html>
diff --git a/styles.css b/styles.css
new file mode 100644
index 0000000000000000000000000000000000000000..eb878d991efae2182101fb22938b0e3ac5007717
--- /dev/null
+++ b/styles.css
@@ -0,0 +1,180 @@
+:root {
+  --bg: #07142b;
+  --card: rgba(255, 255, 255, 0.1);
+  --card-border: rgba(255, 255, 255, 0.2);
+  --text-main: #f8fafc;
+  --text-soft: #cbd5e1;
+  --accent: #22d3ee;
+  --accent-strong: #0ea5e9;
+  --ok: #22c55e;
+  --error: #fb7185;
+}
+
+* {
+  box-sizing: border-box;
+}
+
+body {
+  margin: 0;
+  min-height: 100vh;
+  font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
+  background: radial-gradient(circle at top, #0f3b74 0%, #07142b 55%, #020617 100%);
+  color: var(--text-main);
+  display: flex;
+  flex-direction: column;
+  padding: 1.25rem;
+}
+
+.app-shell {
+  max-width: 960px;
+  width: 100%;
+  margin: 0 auto;
+  display: grid;
+  gap: 1rem;
+}
+
+.hero-card,
+.result-card {
+  background: var(--card);
+  border: 1px solid var(--card-border);
+  border-radius: 20px;
+  backdrop-filter: blur(10px);
+  padding: 1.5rem;
+  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
+}
+
+.eyebrow {
+  margin: 0 0 0.4rem;
+  color: var(--accent);
+  letter-spacing: 0.04em;
+  text-transform: uppercase;
+  font-size: 0.78rem;
+  font-weight: 600;
+}
+
+h1 {
+  margin: 0;
+  line-height: 1.2;
+  font-size: clamp(1.45rem, 2.8vw, 2.1rem);
+}
+
+.subtitle {
+  color: var(--text-soft);
+  line-height: 1.5;
+  margin: 0.7rem 0 1.35rem;
+}
+
+.controls {
+  display: grid;
+  gap: 0.9rem;
+  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
+}
+
+label {
+  display: grid;
+  gap: 0.35rem;
+  font-size: 0.92rem;
+  color: var(--text-soft);
+}
+
+input,
+button {
+  width: 100%;
+  border-radius: 12px;
+  border: 1px solid transparent;
+  padding: 0.75rem 0.9rem;
+  font: inherit;
+}
+
+input {
+  background: rgba(15, 23, 42, 0.85);
+  color: var(--text-main);
+  border-color: rgba(148, 163, 184, 0.3);
+}
+
+input:focus,
+button:focus {
+  outline: none;
+  border-color: var(--accent);
+  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.2);
+}
+
+button {
+  cursor: pointer;
+  border: none;
+  background: linear-gradient(135deg, var(--accent) 0%, var(--accent-strong) 100%);
+  color: #021225;
+  font-weight: 700;
+  margin-top: auto;
+}
+
+.result-header {
+  display: flex;
+  justify-content: space-between;
+  gap: 1rem;
+  align-items: flex-start;
+  flex-wrap: wrap;
+}
+
+h2 {
+  margin: 0;
+}
+
+#date-title {
+  margin: 0.3rem 0 0;
+  color: var(--text-soft);
+}
+
+.badge {
+  font-size: 0.82rem;
+  padding: 0.4rem 0.7rem;
+  border-radius: 999px;
+  background: rgba(148, 163, 184, 0.2);
+  border: 1px solid rgba(148, 163, 184, 0.4);
+}
+
+.status {
+  margin: 1rem 0 0;
+  color: var(--text-soft);
+}
+
+.status.success {
+  color: var(--ok);
+}
+
+.status.error {
+  color: var(--error);
+}
+
+.prayer-grid {
+  margin-top: 1rem;
+  display: grid;
+  gap: 0.9rem;
+  grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
+}
+
+.prayer-item {
+  background: rgba(15, 23, 42, 0.8);
+  border: 1px solid rgba(148, 163, 184, 0.25);
+  border-radius: 14px;
+  padding: 0.8rem;
+}
+
+.prayer-item h3 {
+  margin: 0;
+  color: var(--text-soft);
+  font-size: 0.9rem;
+}
+
+.prayer-item p {
+  margin: 0.35rem 0 0;
+  font-size: 1.15rem;
+  font-weight: 700;
+}
+
+.footer-note {
+  text-align: center;
+  color: var(--text-soft);
+  font-size: 0.85rem;
+  margin-top: 1rem;
+}

