# Radio.co API Documentation for Elevate Radio

## Overview

Radio.co provides a comprehensive API for integrating internet radio functionality into web and mobile applications. This document outlines all available features and how to implement them in Elevate Radio.

---

## Table of Contents

1. [Player Setup](#player-setup)
2. [Player Options](#player-options)
3. [Player Methods](#player-methods)
4. [Player Events](#player-events)
5. [Station Info API](#station-info-api)
6. [Implementation Examples](#implementation-examples)

---

## Player Setup

### Required Scripts

Add to your HTML `<head>`:

```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://public.radio.co/playerapi/jquery.radiocoplayer.min.js"></script>
```

Add before closing `</body>`:

```javascript
<script>$('.radioplayer').radiocoPlayer();</script>
```

### Player Element

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
  data-showartwork="false">
</div>
```

---

## Player Options

| Option | Type | Description |
|--------|------|-------------|
| `data-src` | String | Your Radio.co stream URL (required) |
| `data-autoplay` | Boolean | Auto-play on page load |
| `data-playbutton` | Boolean | Show play/pause button |
| `data-volumeslider` | Boolean | Show volume slider |
| `data-nowplaying` | Boolean | Show current track info |
| `data-elapsedtime` | Boolean | Show streaming time |
| `data-showplayer` | Boolean | Show/hide entire player |
| `data-volume` | Number (0-100) | Initial volume level |
| `data-showartwork` | Boolean | Show current song artwork |
| `data-image` | String | Custom player image |
| `data-bg` | String | Custom background image |

---

## Player Methods

### Basic Controls

```javascript
var player = $('.radioplayer').radiocoplayer();

// Load stream
player.load(callback);

// Play stream
player.play(callback);

// Toggle play/pause
player.playToggle(playCallback, pauseCallback);

// Set/get volume (0-100)
player.volume(50);        // Set to 50%
var vol = player.volume(); // Get current

// Mute
player.mute();

// Check if playing
var isPlaying = player.isPlaying();

// Get elapsed time
var time = player.getTime(true); // formatted
var rawTime = player.getTime(false); // raw

// Get stream state (0-4)
var state = player.getStreamState();
// 0 = No info, 1 = Metadata ready, 2 = Loading
// 3 = Next packet ready, 4 = Ready to play

// Get artwork
player.getArtwork(300, 300, function(url) {
  console.log(url);
}, 'high');

// Check if previously loaded
var loaded = player.hasLoaded();
```

---

## Player Events

```javascript
var player = $('.radioplayer').radiocoplayer();

// Audio loaded successfully
player.event('audioLoaded', function(event) {
  console.log('Stream loaded');
});

// Time update (every second)
player.event('timeChanged', function(event) {
  console.log('Time:', player.getTime(true));
});

// User plays stream
player.event('audioPlay', function(event) {
  console.log('Playing');
});

// User pauses stream
player.event('audioPause', function(event) {
  console.log('Paused');
});

// Stream error
player.event('invalidStream', function(event) {
  console.log('Invalid stream');
});

// Song changed
player.event('songChanged', function(event) {
  console.log('New song playing');
});

// Stream info error
player.event('streamInfoError', function(event) {
  console.log('Could not get stream info');
});
```

---

## Station Info API

### Endpoints

Replace `YOUR_STATION_ID` with your Radio.co station ID (found in dashboard URL).

#### Full Station Status
```
GET https://public.radio.co/stations/YOUR_STATION_ID/status
```

Returns: Track history, bitrate, listeners, current track, etc.

#### Station Info (Name, Logo, Stream URL)
```
GET https://public.radio.co/api/v2/YOUR_STATION_ID
```

#### Current Track
```
GET https://public.radio.co/api/v2/YOUR_STATION_ID/track/current
```

Returns: Current track name, start time, artwork URL

### Example Response (Status)

```json
{
  "status": "online",
  "source": {
    "type": "autodj",
    "collaborator": null
  },
  "collaborators": [],
  "relays": [],
  "current_track": {
    "title": "Artist - Song Title",
    "start_time": "2026-01-23T19:00:00Z",
    "artwork_url": "https://..."
  },
  "history": [
    {
      "title": "Previous Song",
      "start_time": "..."
    }
  ],
  "logo_url": "https://...",
  "streaming_hostname": "streaming.radio.co",
  "outputs": [
    {
      "name": "Default",
      "format": "aac",
      "bitrate": 128
    }
  ]
}
```

---

## Implementation Examples

### Custom Now Playing Widget

```javascript
function updateNowPlaying() {
  fetch('https://public.radio.co/api/v2/YOUR_STATION_ID/track/current')
    .then(res => res.json())
    .then(data => {
      document.getElementById('track-title').textContent = data.title;
      document.getElementById('track-artwork').src = data.artwork_url;
    });
}

// Update every 30 seconds
setInterval(updateNowPlaying, 30000);
updateNowPlaying();
```

### Listener Count

```javascript
function getListenerCount() {
  fetch('https://public.radio.co/stations/YOUR_STATION_ID/status')
    .then(res => res.json())
    .then(data => {
      document.getElementById('listeners').textContent = data.listeners || 0;
    });
}
```

### Track History

```javascript
function getTrackHistory() {
  fetch('https://public.radio.co/stations/YOUR_STATION_ID/status')
    .then(res => res.json())
    .then(data => {
      const history = data.history.slice(0, 10); // Last 10 tracks
      // Render history...
    });
}
```

---

## Elevate Radio Station Info

- **Stream URL**: `https://s2.radio.co/s10a9e6f11/listen`
- **Station ID**: `s10a9e6f11`
- **Status API**: `https://public.radio.co/stations/s10a9e6f11/status`
- **Current Track API**: `https://public.radio.co/api/v2/s10a9e6f11/track/current`

---

## Features We Can Build

### From Radio.co API

1. **Real-time Now Playing** - Show current track with artwork
2. **Track History** - Display recently played songs
3. **Listener Count** - Show live listener numbers
4. **Stream Status** - Online/offline indicator
5. **Live DJ Detection** - Detect when a live DJ is broadcasting vs AutoDJ
6. **Bitrate/Quality Info** - Show stream quality
7. **Volume Control** - Adjustable volume with memory
8. **Play Time Tracking** - Show how long user has been listening

### Custom Backend Features (Need to Build)

1. **Song Requests** - Submit and queue requests
2. **Show Schedules** - Display upcoming live shows
3. **Podcast Archive** - Store and playback recorded shows
4. **User Accounts** - Favorites, history, preferences
5. **Push Notifications** - Notify when shows go live
6. **Analytics** - Track listening habits, popular times
7. **Chat/Comments** - Live interaction during shows
