# Radio.co API Documentation for Elevate Radio

## Overview

Radio.co provides a comprehensive API for building custom radio players and integrating station data into websites and applications. This documentation covers how we can leverage these features for Elevate Radio.

---

## Station Information Endpoints

### 1. Station Status (Full)
```
GET https://public.radio.co/stations/{station_id}/status
```

Returns complete station information including:
- Current track info
- Track history
- Bitrate
- Stream status
- Listener count

**Example Response:**
```json
{
  "status": "online",
  "source": {
    "type": "automated",
    "collaborator": null
  },
  "collaborators": [],
  "relays": [],
  "current_track": {
    "title": "Artist - Song Name",
    "start_time": "2026-01-22T10:30:00Z",
    "artwork_url": "https://...",
    "artwork_url_large": "https://..."
  },
  "history": [...],
  "logo_url": "https://...",
  "streaming_hostname": "s2.radio.co",
  "outputs": [...]
}
```

### 2. Station Basic Info
```
GET https://public.radio.co/api/v2/{station_id}
```

Returns:
- Station name
- Logo URL
- Stream URL

### 3. Current Track
```
GET https://public.radio.co/api/v2/{station_id}/track/current
```

Returns:
- Current track title
- Start time
- Artwork URL

---

## Player API

### Setup Requirements

Include these scripts in your HTML:
```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://public.radio.co/playerapi/jquery.radiocoplayer.min.js"></script>
```

Initialize after body:
```javascript
<script>$('.radioplayer').radiocoPlayer();</script>
```

### Player Element Configuration

```html
<div class="radioplayer" 
     data-src="https://s2.radio.co/s10a9e6f11/listen"
     data-autoplay="false"
     data-playbutton="true"
     data-volumeslider="true"
     data-elapsedtime="true"
     data-nowplaying="true"
     data-showplayer="true"
     data-volume="50"
     data-showartwork="true">
</div>
```

### Player Options

| Option | Type | Description |
|--------|------|-------------|
| `data-src` | string | Stream URL (required) |
| `data-autoplay` | boolean | Auto-play on page load |
| `data-playbutton` | boolean | Show play/pause button |
| `data-volumeslider` | boolean | Show volume control |
| `data-nowplaying` | boolean | Show current track info |
| `data-elapsedtime` | boolean | Show stream duration |
| `data-showplayer` | boolean | Show/hide entire player |
| `data-volume` | number | Initial volume (0-100) |
| `data-showartwork` | boolean | Show track artwork |
| `data-image` | string | Custom player image |
| `data-bg` | string | Custom background image |

---

## Player Methods (JavaScript API)

### Playback Control

```javascript
var player = $('.radioplayer').radiocoplayer();

// Load stream
player.load(callback);

// Play stream
player.play(callback);

// Toggle play/pause
player.playToggle(playCallback, pauseCallback);

// Check if playing
player.isPlaying(); // returns boolean

// Check if loaded
player.hasLoaded(); // returns boolean
```

### Volume Control

```javascript
// Get current volume
var currentVolume = player.volume();

// Set volume (0-100)
player.volume(75);

// Mute stream
player.mute();
```

### Track Information

```javascript
// Get elapsed time (formatted)
player.getTime(true); // "05:32"

// Get elapsed time (raw)
player.getTime(false); // 332

// Get track artwork
player.getArtwork(300, 300, function(artwork) {
  console.log(artwork);
}, 'high');
```

### Stream State

```javascript
player.getStreamState();
// Returns:
// 0 → No information
// 1 → Metadata ready
// 2 → Some data available
// 3 → Next packet available
// 4 → Ready to play
```

---

## Player Events

Listen to events for reactive UI updates:

```javascript
var player = $('.radioplayer').radiocoplayer();

// Audio loaded successfully
player.event('audioLoaded', function(event) {
  console.log('Stream loaded');
});

// Time update (fires every second)
player.event('timeChanged', function(event) {
  updateUI();
});

// User plays stream
player.event('audioPlay', function(event) {
  showPlayingState();
});

// User pauses stream
player.event('audioPause', function(event) {
  showPausedState();
});

// Song changed
player.event('songChanged', function(event) {
  fetchNewTrackInfo();
});

// Invalid stream
player.event('invalidStream', function(event) {
  showError('Stream unavailable');
});

// Stream info error
player.event('streamInfoError', function(event) {
  console.error('Could not fetch stream info');
});
```

---

## Features We Can Build

### 1. Real-time Now Playing
- Display current track with artwork
- Auto-update when song changes
- Track history display

### 2. Live Listener Count
- Show real-time listener statistics
- Historical analytics

### 3. Song Request System
- Queue management via backend
- User request submissions
- Request moderation

### 4. Schedule Display
- Show upcoming shows
- Live show indicators
- Countdown to next show

### 5. Podcast Integration
- Archive recorded shows
- YouTube integration for video podcasts
- Episode management

### 6. Live Broadcasting
- Detect when DJ goes live
- Switch between automated/live modes
- Collaborator information

---

## Our Station Details

**Station ID:** `s10a9e6f11`
**Stream URL:** `https://s2.radio.co/s10a9e6f11/listen`

### API Endpoints for Elevate Radio

```
Status: https://public.radio.co/stations/s10a9e6f11/status
Info: https://public.radio.co/api/v2/s10a9e6f11
Current Track: https://public.radio.co/api/v2/s10a9e6f11/track/current
```

---

## Rate Limits & Best Practices

1. **Polling Frequency:** Don't poll more than once every 10-15 seconds
2. **Caching:** Cache station info, refresh on song change events
3. **Error Handling:** Always handle `invalidStream` and `streamInfoError`
4. **Mobile:** Use lower bitrate streams for mobile data savings

---

## Next Steps

See `app-architecture.md` for the Next.js implementation plan and mobile conversion strategy.
