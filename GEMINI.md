# Antigravity Guidelines & Rules

## iOS PWA Web Push Notifications & Cache Management Rules

Whenever implementing Web Push notifications or PWA updates for iOS / iPhone (iOS 16.4+):

1. **Direct User Gesture for Permission:**
   - On iOS standalone PWA, `Notification.requestPermission()` MUST be called directly inside an `onClick` / `onTouch` user gesture.
   - Never call desktop modal prompt scripts (like `webpushr('prompt')`) on mobile iOS. They throw exceptions and cause false "blocked" errors.

2. **Disable & Hide Desktop Widgets:**
   - In Webpushr or third-party web push services, permanently disable and hide the "Subscription Bell" / floating bell widget via CSS:
     ```css
     #webpushr-bell, .webpushr-bell, div[id*="webpushr-bell"], iframe[id*="webpushr"] { display: none !important; }
     ```
   - Embed the native button anchor: `<div id="webpushr-subscription-button"></div>`.

3. **Subdirectory Scope for GitHub Pages:**
   - When hosted on GitHub Pages subdirectories (e.g., `https://user.github.io/Repo/`), always initialize Webpushr with the full Service Worker URL and scope:
     ```javascript
     webpushr('init', 'VAPID_KEY', 'https://user.github.io/Repo/webpushr-sw.js', '/Repo/');
     ```
   - Always ensure `webpushr-sw.js` exists in the repository root with `importScripts('https://cdn.webpushr.com/sw-server.min.js');`.

4. **Web Push REST API CamelCase Headers:**
   - When sending push notifications via REST API (from serverless functions, Google Apps Script, or client):
     - Endpoint: `https://api.webpushr.com/v1/notification/send/attribute`
     - Headers must use camelCase: `webpushrKey` and `webpushrAuthToken`.
     - Target specific users using: `{ attribute: { workerId: targetAdmin } }`.

5. **Automatic iOS App Badge Cleanup:**
   - Always clear the app icon badge (`navigator.clearAppBadge()`) when:
     - The app is opened / becomes visible (`visibilitychange`, `focus`).
     - The user views the calendar or opens any day modal / feed comments.
   - Never leave unread badge counts stuck on the home screen icon.

6. **Anti-Freeze Cache Reloading on iOS PWA:**
   - iOS PWA caches Service Worker assets aggressively.
   - Always increment `CACHE_NAME` in `sw.js` on every release.
   - Always provide a dedicated hard-reload / cache-clearing function in settings:
     ```javascript
     async function forceAppUpdate() {
       if ('caches' in window) {
         const keys = await caches.keys();
         for (const k of keys) await caches.delete(k);
       }
       if ('serviceWorker' in navigator) {
         const regs = await navigator.serviceWorker.getRegistrations();
         for (const r of regs) await r.unregister();
       }
       window.location.href = window.location.origin + window.location.pathname + '?v=' + Date.now();
     }
     ```
