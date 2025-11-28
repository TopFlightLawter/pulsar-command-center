# Pulsar Command Center

A real-time WebSocket control interface for the Pulsar Rebirth v5.0 alarm clock system.

## Features

- **Real-time motor status** - See M1/M2 activity and intensity levels live
- **Alarm configuration** - Set alarm windows, intervals, and snooze duration
- **PWM tuning** - Adjust warning duration and pause timing
- **Pattern control** - Configure progressive intensity parameters
- **Persistent settings** - Save configuration to ESP32 flash memory
- **Mobile-optimized** - Designed for iPhone/Android with PWA support

## Usage

1. **Connect your Pulsar device** to WiFi
2. **Open the app** on your phone or computer
3. **Enter the IP address** or use `pulsar.local` (mDNS)
4. **Control your alarm** in real-time!

## Settings

Tap the ⚙ gear icon to configure the ESP32 connection:
- Use `pulsar.local` if mDNS is working
- Or enter the ESP32's IP address directly (shown in Serial Monitor)

## Add to iPhone Home Screen

1. Open in Safari
2. Tap the Share button
3. Select "Add to Home Screen"
4. Enjoy app-like experience!

## Technical Details

- Built with React 18 (CDN-loaded)
- WebSocket connection on port 81
- Compatible with Pulsar Rebirth v5.0 firmware
- No build process required - single HTML file

## License

MIT - Use freely for personal projects.

---

Built for the Pulsar Rebirth project 🌀
