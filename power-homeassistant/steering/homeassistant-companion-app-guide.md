# Home Assistant Companion App Guide

This guide covers the Home Assistant Companion App features including core functionality, location tracking, sensors, and notifications for iOS, macOS, and Android platforms.

## Overview

The Home Assistant Companion App extends your Home Assistant instance by:
- Providing numerous sensors (battery, network status, etc.)
- Creating a `device_tracker` entity for location updates
- Offering action shortcuts to trigger scripts or automations
- Sending rich notifications with attachments, actions, and commands

## Platform Availability

### Android Flavors
- **Full**: Available on Play Store, requires Google Play Services, includes all features
- **Minimal**: Available on GitHub/F-Droid, no Google Play Services required, no location tracking, limited sensors

## Core Features

### Actions (iOS/macOS Only)

Actions allow integration with Home Assistant automations from iOS, Apple Watch, and CarPlay.

#### Creating Actions

**In the App:**
- Name: Returned in the `ios.action_fired` event
- Server: Target Home Assistant server
- Text: Descriptive text shown on phone/watch
- Icon: Display icon from mdi set
- Colors: Background, text, and icon colors (requires `use_custom_colors`)

**In configuration.yaml:**
```yaml
ios:
  actions:
    - name: Fred
      background_color: "#000000"
      label:
        text: "Hello, World"
        color: "#ff0000"
      icon:
        icon: earth
        color: "#ffffff"
      show_in_carplay: false
      show_in_watch: true
      use_custom_colors: true
```

#### Using Actions

When pressed, fires `ios.action_fired` event with:
- `actionName`: Name of the action
- `sourceDeviceID`: Device ID
- `triggerSource`: Origin (widget, appShortcut, watch, carPlay)

**Automation Example:**
```yaml
automation:
  - alias: "Action Turn Lights Off"
    trigger:
      - platform: event
        event_type: ios.action_fired
        event_data:
          actionName: "Bed Time"
    action:
      - action: light.turn_off
        entity_id: group.all_lights
```

### Location Tracking

Location updates are sent in various situations:
- App opened/refreshed
- Background updates
- Zone enter/exit
- Significant location change
- Via notification command
- iBeacon detection (iOS)

Check `sensor.last_update_trigger` to see the cause of the most recent update.

#### Location Privacy
- Data sent directly to your Home Assistant instance
- Can use Home Assistant Cloud Service depending on URL configuration
- No third-party servers involved

### Sensors

The companion app provides numerous sensors depending on platform:

#### Common Sensors (All Platforms)
- Battery Level/State
- Connection Type (WiFi/Cellular)
- BSSID/SSID
- Storage
- Last Update Trigger

#### iOS/macOS Specific
- Activity sensors (walking, running, etc.)
- Pedometer (steps, distance, floors)
- Geocoded Location

#### Android Specific
- Audio sensors
- Bluetooth sensors
- Do Not Disturb
- Light sensor
- Next Alarm
- Phone state
- Proximity sensor

#### Sensor Updates
- iOS: Updates on location change, foreground activity, pull-to-refresh, background rate determined by iOS
- Android: Configurable update intervals per sensor

#### Multi-Server Support
Configure which sensors are sent to each connected server in Settings > Companion App.

## Notifications

This section covers all notification features including basic notifications, actionable notifications, attachments, and platform-specific notification commands.

### Basic Notification
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "Notification text"
  title: "Optional Title"
```

### Sending to Multiple Devices
```yaml
notify:
  - name: ALL_DEVICES
    platform: group
    services:
      - action: mobile_app_iphone_one
      - action: mobile_app_pixel_4
```

### Opening a URL
```yaml
data:
  message: "Motion Detected"
  data:
    url: "/lovelace/cameras"           # iOS
    clickAction: "/lovelace/cameras"   # Android
```

### Grouping Notifications
```yaml
data:
  message: "Something happened!"
  data:
    group: "example-notification-group"
```

### Replacing Notifications (Tag)
```yaml
data:
  message: "Updated message"
  data:
    tag: "my-notification-tag"
```

### Clearing Notifications
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "clear_notification"
  data:
    tag: "my-notification-tag"
```

### Actionable Notifications

Add buttons to notifications that fire events when tapped.

```yaml
action: notify.mobile_app_<device_id>
data:
  message: "Motion detected!"
  data:
    actions:
      - action: "ALARM"
        title: "Sound Alarm"
      - action: "URI"
        title: "Open Camera"
        uri: "/lovelace/cameras"
```

#### Action Parameters
- `action`: Required identifier (use "REPLY" for text input)
- `title`: Required button text
- `uri`: Optional URL to open (Android requires action: "URI")
- `behavior`: Set to "textInput" for text response

#### Handling Action Events
```yaml
automation:
  - alias: "Handle Alarm Action"
    trigger:
      - platform: event
        event_type: mobile_app_notification_action
        event_data:
          action: "ALARM"
    action:
      - action: alarm_control_panel.alarm_trigger
        entity_id: alarm_control_panel.home
```

### Notification Attachments

Attach images, videos, or audio to notifications.

#### Supported Media
- Image: JPEG, GIF, PNG (max 10MB)
- Video: MPEG, MPEG2, MPEG4, AVI (max 50MB)
- Audio: AIFF, WAV, MP3, MPEG4 Audio (max 5MB, iOS only)

#### Storage Locations
- `media_source` (recommended): `/media/local/file.jpg` - requires authentication
- `www` folder: `/local/file.jpg` - publicly accessible

#### Example
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "Motion detected"
  data:
    image: "/media/local/snapshot.jpg"
    # Or for camera proxy (Android)
    image: "/api/camera_proxy/camera.front_door"
```

### Dynamic Content (iOS)

#### Map Display
```yaml
data:
  message: "Something happened at home!"
  data:
    action_data:
      latitude: "40.785091"
      longitude: "-73.968285"
```

#### Camera Stream
Use with actionable notifications to show live camera feeds.

### Critical Notifications

High-priority notifications that bypass Do Not Disturb.

#### iOS Critical Alert
```yaml
action: notify.mobile_app_<device_id>
data:
  title: "Wake up!"
  message: "Emergency alert!"
  data:
    push:
      sound:
        name: "default"
        critical: 1
        volume: 1.0
```

Or using interruption-level:
```yaml
data:
  push:
    interruption-level: critical
```

#### Android Critical
```yaml
data:
  message: "Emergency!"
  data:
    ttl: 0
    priority: high
    channel: alarm_stream  # Plays even on silent
```

#### Android TTS Alarm
```yaml
data:
  message: TTS
  data:
    ttl: 0
    priority: high
    media_stream: alarm_stream_max
    tts_text: "Emergency alert message!"
```

### Notification Sounds

#### iOS Custom Sounds
- Format: 32bit float 48000Hz wav files
- Import via iTunes File Sharing or Cloud Storage
- System sounds can be imported from app settings

```yaml
data:
  message: "Roommate arrived"
  data:
    push:
      sound: "US-EN-Morgan-Freeman-Roommate-Is-Arriving.wav"
```

#### Android
Configure sounds in Settings > Companion App > Notification Channels.

### Notification Commands

Send commands instead of displaying notifications.

#### iOS Commands
- `request_location_update`: Request location update
- `clear_badge`: Remove app badge
- `clear_notification`: Remove specific notification
- `update_complications`: Update Apple Watch complications
- `update_widgets`: Update iOS widgets

#### Android Commands
- `command_activity`: Launch an activity
- `command_bluetooth`: Toggle Bluetooth
- `command_dnd`: Control Do Not Disturb
- `command_flashlight`: Toggle flashlight
- `command_high_accuracy_mode`: Control GPS accuracy
- `command_launch_app`: Launch an application
- `command_media`: Control media playback
- `command_ringer_mode`: Change ringer mode
- `command_screen_on`: Turn on screen
- `command_volume_level`: Control volume
- `command_webview`: Open app to specific view
- `request_location_update`: Request location update

#### Request Location Update
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "request_location_update"
```

#### Open WebView (Android)
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "command_webview"
  data:
    command: "/lovelace/cameras"
```

### Local Push Notifications

Delivers notifications via WebSocket instead of cloud services.

#### Requirements
- Home Assistant Core 2021.6+
- iOS 2021.7+ / macOS 2021.7+ / Android 2022.2+

#### iOS Limitations
- Only works on Internal URL
- Requires configuring SSIDs as internal
- Slightly increased battery usage

#### Benefits
- Works without internet
- Faster delivery
- No rate limits
- More reliable on local network

### Notification Events

#### Notification Received Event
Set `confirmation: true` to receive `mobile_app_notification_received` event:
```yaml
data:
  message: "Test"
  data:
    confirmation: true
```

#### Notification Cleared Event (Android)
Fires `mobile_app_notification_cleared` when notification is swiped away or cleared.

Event data includes all original notification data plus `device_id`.

## Platform-Specific Features

This section covers features that are specific to Android or iOS platforms.

### Android Features

#### Device Controls
2. **Group related notifications** to reduce clutter
3. **Use critical notifications sparingly** - only for true emergencies
4. **Test on all target platforms** - features vary between iOS and Android
5. **Use Local Push** when possible for reliability
6. **Store attachments in media_source** for security
7. **Handle action events** to complete the notification workflow



Android 11+ devices can integrate with the smart home device controls feature, accessible from quick settings, notification drawer, or power menu.

##### Supported Domains
- `automation`, `button`, `camera` (Android 12+), `climate`, `cover`, `fan`, `humidifier`
- `input_boolean`, `input_button`, `input_number`, `light`, `lock`, `media_player`
- `number`, `remote`, `scene`, `script`, `siren`, `switch`, `vacuum`

##### Use When Locked
- Android 11: Controls work when locked
- Android 12: Controls don't work when locked
- Android 13+: Configurable in Settings > Display > Lock screen

##### Dashboard Mode (Android 14+)
Show a Home Assistant dashboard instead of built-in controls. Configure in Settings > Companion App > Manage device controls.

#### Quick Settings Tiles

Add tiles to the notification pull-down menu for quick actions. Available on Android 7.0+.

##### Setup
1. Navigate to Settings > Companion App > Manage Tiles
2. Set label and optionally sublabel (Android 10+)
3. Select entity
4. Edit quick settings panel and drag the tile to active section

##### Supported Domains
- `automation`, `button`, `cover`, `fan`, `humidifier`, `input_boolean`
- `input_button`, `light`, `lock`, `media_player`, `remote`, `siren`
- `scene`, `script`, `switch`

##### Options
- Custom icon (tap icon to change)
- Vibrate when clicked
- Require unlocked device

#### Shortcuts

Navigate to specific dashboard pages or entities directly from home screen.

##### Shortcut Types
- **Dashboard**: Enter path like `/lovelace/default_view` or `/lovelace-dashboardname/viewname`
- **Entity**: Select from list of entities

##### Dynamic Shortcuts (Android 7.1+)
- Add from Settings > Companion App > Manage Shortcuts
- Access via long press on app icon
- Maximum 5 shortcuts (most launchers show 4)

##### Pinned Shortcuts (Android 8.0+)
- Created from Settings > Companion App
- Appear directly on home screen
- No limit on number of shortcuts

#### WebView Settings

##### Autoplay Video
Enable in Settings > Companion App to autoplay videos in more info panel. May increase data usage.

##### Always Show First View
Open the first view of default dashboard when app starts.

##### Keep Screen On
Prevent screen from turning off while using the app.

##### Pinch to Zoom
Enable zoom gestures in the webview.

#### Widgets

Add widgets to home screen for various actions. Widgets update every 30 minutes by default.

##### Widget Types

**Action Button**: Perform an action when tapped. Shows green check on success, red on failure.

**Entity State**: Display entity state and attributes with customizable text size.

**Media Player**: Control media players with play/pause, seek, skip, and volume buttons.

**Picture**: Show camera snapshots, updates hourly or on tap.

**Template**: Display custom text using Home Assistant templating. Supports HTML formatting.

**To-do List**: Display items from a to-do list entity.

##### Real-time Updates
Grant notification access in Android Settings > Notifications > Notification read, reply & control to enable real-time widget updates.

### iOS Features

#### Widgets

##### Available Widgets
- **Assist In-App**: Open Assist with preferred pipeline
- **Scripts**: Execute scripts with optional completion notification
- **Sensors**: Display sensor values (~15 min update interval)
- **Open Page**: Open any Home Assistant sidebar page
- **Gauge** (Advanced): Create gauge using templates
- **Details** (Advanced): Display up to 3 lines using templates
- **Custom Widgets** (BETA): Fully customizable tile-card-like widgets

##### Widget Sizes
- System: Small, Medium, Large, Extra Large (macOS)
- Accessory (Lock screen): Circular, Inline, Rectangular

##### Custom Widget Actions
- Default (toggle/refresh)
- Navigate to path
- Run script
- Assist
- Nothing (refresh only)

##### Updating Widgets
Send silent push notification with `update_widgets` command to refresh more frequently.

#### Siri Shortcuts

Create shortcuts to perform Home Assistant tasks via Siri voice commands.

##### Available Actions
- **Call Service**: Call any Home Assistant action
- **Fire Event**: Fire event on event bus
- **Get Camera Image**: Get still frame from camera
- **Perform Action**: Execute an action
- **Render Template**: Render template for use in subsequent actions
- **Send Location**: Send location to Home Assistant
- **Update Sensors**: Update all sensors

##### Launching Shortcuts
- Siri voice command
- Widget on Today View
- Shortcuts app
- Apple Watch (watchOS 7+)
- Spotlight Search
- Add to Home Screen
- Push notification
- Back Tap (iOS 14+)

##### Triggering via Notification
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "Trigger a Shortcut!"
  data:
    shortcut:
      name: "Shortcut Name"
      key_for_shortcut: "value"
```

##### Shortcut Result Event
After completion, fires `ios.shortcut_run` event with:
- `status`: success, failure, or cancelled
- `result`: Shortcut result
- `error`: Error details if failed

## Integrations

This section covers features that work across platforms for integrating with other apps and services.

### App Events

The companion apps fire events on the Home Assistant event bus during certain situations.

#### iOS Events
- `ios.finished_launching`: App opens when not running in background
- `ios.became_active`: App becomes active
- `ios.entered_background`: App closed but running in background
- `ios.will_resign_active`: App about to become inactive

#### Android Events
- `android.intent_received`: Intent received by the app

### Gestures

Both platforms support gestures for navigation and interaction.

#### iOS Gestures
- Swipe from left edge: Go back
- Pull down: Refresh

#### Android Gestures (BETA)
- Configurable swipe gestures for navigation
- Enable in Settings > Companion App

### Haptics

Physical feedback when interacting with UI elements.

#### Supported Interactions
- Toggles (lights, switches, input_booleans)
- Input selects
- Invalid action errors

#### Supported Devices
- iPhone 7 and later
- Android devices with haptic feedback support

### Sharing

Share content from any app to Home Assistant, firing a `mobile_app.share` event.

#### Event Data
- `url`: Shared URL
- `text`: Shared text
- `subject`: Webpage title (Android)
- `caller`: Source app package (Android)
- `entered`: Text typed with share (iOS)
- `device_id`: Device identifier

#### Example Event
```json
{
  "event_type": "mobile_app.share",
  "data": {
    "url": "https://example.com",
    "text": "Shared content",
    "device_id": "DEVICE_ID"
  }
}
```

### Theming

Customize app appearance using Home Assistant themes.

#### Android
- Colors must be specified in hex format (e.g., `#0099ff`)
- Variable names not supported

#### iOS
- Supports theme colors from Home Assistant
- Follows system dark/light mode

### Universal Links (NFC/QR Tags)

Scan NFC tags or QR codes to trigger automations.

#### Tag Format
URL: `https://www.home-assistant.io/tag/<tag_id>`

#### Platform Support
- iOS: 2020.5+
- Android: 2.2.0+

#### Tag Management
Manage tags in Settings > Tags panel. Assign friendly names and view scan history.

### URL Handler

Open Home Assistant from other apps via URL schemes.

#### Navigate
```
homeassistant://navigate/dashboard-mobile/my-subview
```

With server selection:
```
homeassistant://navigate/webcams?server=My%20home
```

#### Call Service (iOS)
```
homeassistant://call_service/light.turn_on?entity_id=light.porch
```

#### Fire Event (iOS)
```
homeassistant://fire_event/custom_event?entity_id=MY_CUSTOM_EVENT
```

#### Send Location (iOS)
```
homeassistant://send_location/
```

### X-Callback-URL (iOS)

Full support for X-Callback-URL standard for inter-app communication. All URL handler functions are supported via X-Callback-URL format.

## Best Practices

1. **Use tags** for notifications that should be updated/replaced
2. **Group related notifications** to reduce clutter
3. **Use critical notifications sparingly** - only for true emergencies
4. **Test on all target platforms** - features vary between iOS and Android
5. **Use Local Push** when possible for reliability
6. **Store attachments in media_source** for security
7. **Handle action events** to complete the notification workflow

## Troubleshooting

### FAQs

#### App Crashes on Setup

If the app crashes after clicking "continue" during setup (Home Assistant 0.110+), add values for `internal_url` and `external_url`:

**Via UI:** Settings > General Settings (enable "Advanced Mode" in profile if not visible)

**Via configuration.yaml:**
```yaml
homeassistant:
  external_url: https://your-ha-instance.duckdns.org
  internal_url: http://192.168.1.x:8123
```

#### notify.mobile_app Action Not Appearing

1. Restart Home Assistant after app setup
2. Force quit the app (iOS) or force stop (Android)
3. Relaunch the app
4. Restart Home Assistant again
5. Check Developer Tools > Actions for the action

**iOS:** Check Settings > Companion App > Notifications. If "Push ID" is empty, tap Reset.

**Android:** Follow the [Starting Fresh](#starting-fresh-with-the-android-app) steps if still not working.

#### Notifications Not Received

1. Check message limits (500/day per device, resets at midnight UTC)
   - iOS: Settings > Companion App > Notifications (scroll to bottom)
2. Reset push ID token (iOS: Settings > Companion App > Notifications > Reset)
3. Check system notification settings
4. Android: Try [Starting Fresh](#starting-fresh-with-the-android-app)

**Test with simple payload:**
```yaml
action: notify.mobile_app_<device_id>
data:
  message: "Hello World"
```

#### SSL Error / Cannot Connect Away from Home

**With Home Assistant Cloud:** Enable Remote UI or disable "Connect Via Cloud" in app settings and enter External URL manually.

**Without Cloud:** Remote connection requires encryption. Set up SSL using:
- [Home Assistant Remote Access Guide](https://www.home-assistant.io/docs/configuration/remote/)
- [Let's Encrypt with DuckDNS](https://www.home-assistant.io/docs/ecosystem/certificates/lets_encrypt/)

#### Something Works Differently Than Desktop

1. Pull down to refresh (iOS) or force stop and relaunch (Android)
2. Test in Safari/Chrome browser on mobile
3. If issue exists in browser: Report to [Home Assistant Frontend GitHub](https://github.com/home-assistant/frontend/issues)
4. If only in app: Report to [iOS](https://github.com/home-assistant/iOS/issues) or [Android](https://github.com/home-assistant/android/issues) GitHub

#### Sensor Names Too Similar Across Devices

Rename sensors via Integrations Dashboard:
1. Settings > Integrations
2. Find "Mobile App: *Device Name*"
3. Click sensor > cog icon
4. Change Entity ID (keep `sensor.` or `device_tracker.` prefix)

#### kCLError on Location Refresh (iOS)

Change location permission to "Always" in iOS Settings > Privacy > Location Services > Home Assistant.

#### Person Entity Not Updating

1. Settings > People
2. Select the person
3. Remove unused/stationary devices from tracker list
4. Keep only devices that travel with you
5. Save changes

#### Starting Fresh with the Android App

1. Update Home Assistant Core, Android app, and Android System WebView
2. Clear Storage/App data (don't uninstall - avoids auto-backup)
3. Remove all Mobile App entries in Settings > Integrations
4. Restart Home Assistant
5. Log back in with credentials (not Trusted Networks)

#### Device Tracker Not Updating (Android)

**Prerequisites:**
- Enable remote access for your server
- Enable Location sensors: Background location, Location zone, Single accurate Location
- Use `full` flavor if not from Play Store

**Settings to check:**
- Location permissions: "Allow all the time" + Precise location (Android 12+)
- GPS enabled on device
- Background access enabled (check in Companion App settings)
- Battery optimization disabled for the app
- Data Saver disabled / Unrestricted data enabled

**Debug with location history:**
Settings > Companion App > Troubleshooting > Location tracking > Enable location history

#### Self-Signed Certificate Shows Blank Page (Android)

Import the certificate into Android's Trusted Certificates:
1. Download certificate to device
2. Settings > Security > Encryption & credentials > Install a certificate
3. Select the certificate file

#### Widget Not Working (Android)

1. Disable Data Saver
2. Enable background data for Home Assistant
3. Remove and recreate the widget

#### Notify Action Similar or Missing (Android)

1. Settings > Companion App
2. Change Device Name under Device Registration
3. Restart Home Assistant

#### Sensors Missing or Not Updating

**iOS:** Sensor updates tied to location updates when app not in foreground.
- Set location to "Always Allow"
- Use Shortcuts app automation for reliable updates (e.g., when charging starts)

**Android:** Sensors appear when they have an update. Wait for state change or grant required permissions.

#### Text to Speech Not Working (Android)

1. Update [Speech Recognition & Synthesis](https://play.google.com/store/apps/details?id=com.google.android.tts)
2. Set as default Text to Speech engine

#### Android App Battery Drain

Check and disable these features one by one:
1. High accuracy mode (full version) - don't leave enabled all the time
2. Single Accurate Location sensor - disable "Include in sensor updates"
3. Persistent Connection - set to "Never"
4. Bluetooth Transmitter - only enable when needed
5. Sensor Update Frequency - set to "Normal"
6. Notification Sensors - always define an allow list

#### Android Crash Logs

Access via Settings > Companion App > Troubleshooting > Show and Share Logs.

Remove sensitive information (like HA URL) before sharing logs in GitHub issues.

### Error Codes

#### "Invalid Client ID or Redirect URI" + "OS Error while looking up redirect_uri"

**Cause:** Broken IPv6 implementation

**Verify:** Run `curl -v6 https://home-assistant.io/iOS/beta-auth` on HA machine

**Fix:** Disable IPv6 if connection fails

#### "Invalid Client ID or Redirect URI" + "Timeout while looking up redirect_uri"

**Cause:** DNS problem

**Verify:** Run `dig home-assistant.io` and `nslookup home-assistant.io`

**Fix:** Try Google DNS servers:
```bash
# For Home Assistant OS
ha dns options --servers dns://8.8.8.8 --servers dns://1.1.1.1
```

#### "SSL error while looking up redirect_uri"

**Cause:** Home Assistant can't negotiate encrypted connection to home-assistant.io

**Common on:** macOS installations where certificate installer was skipped

**Fix:** Run `/Applications/Python 3.x/Install Certificates` command

#### "Setup failed for dependencies: zeroconf"

**Causes:**
- Two Home Assistant instances with identical names → Rename one
- Missing `default_config:` in configuration.yaml → Add it back

#### "Response status code was unacceptable: 400"

**Causes:**
- Home Assistant version older than 0.104.0 → Update
- Unexpected characters/emoji in device name → Simplify device name

#### "URLSessionTask failed with error"

**Causes:**
- Local network access denied → Enable in iOS Settings > Home Assistant > Local Network
- Incorrect external URL (e.g., wrong port when using port forwarding)
- Logged into Home Assistant Cloud without subscription while using reverse proxy → Log out of Cloud

### Networking

#### How the App Connects to Home Assistant

The app needs to know your HA address:
- Local: `192.168.1.x:8123` or `http://homeassistant.local:8123`
- Remote: Requires either Home Assistant Cloud or port forwarding with SSL

#### Home Assistant Cloud (Easiest)

[Subscribe to Home Assistant Cloud](https://www.nabucasa.com/) for encrypted remote access without opening your network.

#### Manual Remote Access Setup

1. **Dynamic DNS:** Use services like [DuckDNS](https://github.com/home-assistant/addons/blob/master/duckdns/README.md) for a stable domain name
2. **Port Forwarding:** Forward TCP 8123 on router to HA's internal IP
3. **SSL Certificate:** Use DuckDNS add-on for free Let's Encrypt certificate

#### Hairpin NAT

Test if your router supports it: Access `http://my-home.duckdns.org:8123` from inside your network.

If it doesn't work, you need Split Brain DNS.

#### Split Brain DNS

Required when hairpin NAT doesn't work. Returns different IPs for the same domain based on network location.

**Setup with AdGuard Home add-on:**
1. Install AdGuard Home add-on
2. Set router DHCP to use HA's IP as DNS server
3. In AdGuard: Settings > Filters > DNS rewrites
4. Add rewrite: `my-home.duckdns.org` → `192.168.1.x` (HA's internal IP)

#### CG-NAT / DS-Lite Issues

If your ISP uses Carrier-grade NAT:
- Ask ISP for a real public IP address
- Use Home Assistant Cloud

#### IPv6 Setup

For future-proofing with both IPv4 and IPv6:
- Keep HA on port 8123 (don't change ports)
- Create both A-record (IPv4) and AAAA-record (IPv6) for your domain

#### Reverse Proxy with NGINX

Use when devices require unencrypted HTTP (e.g., ESP-based IoT):

```yaml
# configuration.yaml
homeassistant:
  external_url: https://my-home.duckdns.org  # No port

http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.32.0/23  # Docker subnet
    - 127.0.0.1
    - ::1
  # Remove SSL certificate lines - NGINX handles SSL
```

Port forwarding: `TCP 443` → `192.168.1.x:443` (not 8123)

#### App Connection Settings

**With proper SSL and Hairpin NAT:** Use same URL everywhere (e.g., `https://my-home.duckdns.org:8123`)

**With Home Assistant Cloud or different URLs:**
1. Set External URL in app connection settings
2. Configure Internal URL for home network
3. Add home WiFi SSID to use internal URL when connected

**Multiple Access Points:** Use BSSID format: `BSSID:1A:2B:3C:4D:5E:6F`

## Resources

- [Companion App Documentation](https://companion.home-assistant.io/)
- [Notification Details](https://companion.home-assistant.io/docs/notifications/notifications-basic)
- [Location Tracking](https://companion.home-assistant.io/docs/core/location)
- [Sensors Reference](https://companion.home-assistant.io/docs/core/sensors)
- [Android Device Controls](https://companion.home-assistant.io/docs/integrations/android-device-controls)
- [Android Quick Settings](https://companion.home-assistant.io/docs/integrations/android-quick-settings)
- [Android Shortcuts](https://companion.home-assistant.io/docs/integrations/android-shortcuts)
- [Android WebView](https://companion.home-assistant.io/docs/integrations/android-webview)
- [Android Widgets](https://companion.home-assistant.io/docs/integrations/android-widgets)
- [iOS Widgets](https://companion.home-assistant.io/docs/integrations/ios-widgets)
- [App Events](https://companion.home-assistant.io/docs/integrations/app-events)
- [Gestures](https://companion.home-assistant.io/docs/integrations/gestures)
- [Haptics](https://companion.home-assistant.io/docs/integrations/haptics)
- [Sharing](https://companion.home-assistant.io/docs/integrations/sharing)
- [Siri Shortcuts](https://companion.home-assistant.io/docs/integrations/siri-shortcuts)
- [Theming](https://companion.home-assistant.io/docs/integrations/theming)
- [Universal Links](https://companion.home-assistant.io/docs/integrations/universal-links)
- [URL Handler](https://companion.home-assistant.io/docs/integrations/url-handler)
- [X-Callback-URL](https://companion.home-assistant.io/docs/integrations/x-callback-url)
- [Companion App FAQs](https://companion.home-assistant.io/docs/troubleshooting/faqs)
- [Error Codes Reference](https://companion.home-assistant.io/docs/troubleshooting/errors)
- [Networking Guide](https://companion.home-assistant.io/docs/troubleshooting/networking)
- [More Help](https://companion.home-assistant.io/docs/troubleshooting/more-help)
- [iOS GitHub Issues](https://github.com/home-assistant/iOS/issues)
- [Android GitHub Issues](https://github.com/home-assistant/android/issues)
