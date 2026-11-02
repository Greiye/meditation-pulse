# Pulse Counter

A meditation timer that vibrates your phone once per second, so you can keep track of time with your eyes closed.

- Tap the circle to start, tap again to stop.
- **Open** counts up; **1 / 5 / 10 / 20 min** or a custom value counts down and ends with three long buzzes and a chime.
- Optional longer buzz at each minute and a soft tick sound.
- Keeps the screen awake while running (pulses stop if the screen locks).

Live: https://greiye.github.io/meditation-pulse/ (Add to Home screen to install it like an app.)

## Platform notes

- **Android / Chrome:** full support through the Vibration API.
- **iPhone / Safari:** Safari has no Vibration API. The page tries the iOS 18+ switch-toggle haptic as a fallback, which is unreliable; use the tick sound instead.

Plain HTML, CSS and JS with no build step. GitHub Pages serves it from the `main` branch root.
