# Tilt Racer

A small top-down Canvas racer controlled by phone motion readings sent through nikki Streams. The laptop runs the game; the phone acts as a steering wheel. The page uses the existing nikki Streams SDK and the project WebSocket address, prefilled as `10.89.84.190:3000`.

## Phone sensor setup

In the Streams screen shown for this demo, switch on **Orientation**, **Gyroscope**, and **Game Rotation Vector**, then start streaming. Orientation alone drives steering. Game Rotation Vector and Gyroscope remain visible in diagnostics until their axis mapping is verified. Ambient Light and the other sensors shown can stay off.

## Run

1. Connect the phone and laptop to the same Wi-Fi or phone hotspot.
2. In this folder, run `py -m http.server 8000` (or `python -m http.server 8000`).
3. Open `http://localhost:8000` on the laptop.
4. Confirm the Streams address is the same local address used by the phone session (`10.89.84.190:3000` by default) and enter the Streams session ID.
5. Choose **Connect phone**, start streaming in the phone app, then choose **Start racing**.
6. Hold the phone in your steering position before connecting; the first sensor reading auto-centers the wheel. Tap **Center phone / calibrate** any time to reset neutral. Smaller tilts now steer the car, and **Tilt sensitivity** adjusts the response. Arrows or A/D work as a fallback.

A hosted HTTPS page needs a secure `wss://` endpoint. The SDK bundled here only accepts local WebSocket addresses, so use the same LAN/hotspot connection for this demo. The session ID and address are saved in this browser's local storage.

## How the sensor mapping works

`game.js` receives parsed WebSocket frames through `streams-sdk.js`. The `readMotion()` parser searches nested sensor payloads. For an Orientation vector in Android order (azimuth, pitch, roll), it uses the third value (roll); named `roll`/`gamma` fields are accepted. An unlabelled x/y/z object is not guessed as an axis. Orientation is the sole steering source. Game Rotation Vector and Gyroscope values are shown for diagnosis only: a captured frame showed the quaternion-derived angle differing from Orientation roll, and gyro is angular velocity rather than an absolute angle. The first valid sensor reading automatically sets the current angle as neutral, preventing the car from pulling as the race starts. The calibration button can reset neutral at any time. A small dead zone filters hand tremor, then the maximum-tilt slider maps modest tilt into progressive left/right steering. A 35 ms time-based filter smooths sensor noise without adding a long reversal delay; input older than 500 ms eases back to neutral.

Open **Sensor message details** to inspect the actual frames from your app. Sensor field names and vector conventions can vary. For axis verification, record the debug block at calibrated neutral, slight left, and slight right; the current static frame is not enough to confirm the physical axis direction. If your app sends a different layout, adjust `readMotion()` in `game.js`; the sensor panel shows the exact data received.

The animation loop draws the road and traffic on Canvas, moves the player car from the steering value, and checks for collisions. Passing traffic adds distance. Everything runs in the laptop browser; it receives sensor numbers and does not record raw motion data.

## Files

- `index.html`: game canvas and connection controls
- `styles.css`: responsive layout
- `game.js`: WebSocket connection, sensor parsing, calibration, and game loop
- `streams-sdk.js`: nikki Streams browser SDK
- `POC.md`: proof-of-concept flow and implementation notes
