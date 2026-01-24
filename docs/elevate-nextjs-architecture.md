# Elevate Radio - Next.js Architecture & Mobile Conversion

## Overview

This document outlines the architecture for converting Elevate Radio from a static HTML site to a full-stack Next.js application with iOS and Android mobile app support.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Project Structure](#project-structure)
3. [Core Features](#core-features)
4. [Backend Architecture](#backend-architecture)
5. [Mobile App Strategy](#mobile-app-strategy)
6. [Database Schema](#database-schema)
7. [API Routes](#api-routes)
8. [Implementation Roadmap](#implementation-roadmap)

---

## Tech Stack

### Frontend (Web)
| Technology | Purpose |
|------------|---------|
| **Next.js 14+** | React framework with App Router |
| **TypeScript** | Type safety |
| **Tailwind CSS** | Styling (matching current design) |
| **Zustand** | State management |
| **React Query** | Data fetching & caching |
| **Framer Motion** | Animations |

### Backend
| Technology | Purpose |
|------------|---------|
| **Next.js API Routes** | Backend API |
| **Prisma** | Database ORM |
| **PostgreSQL** | Primary database (Supabase/Neon) |
| **NextAuth.js** | Authentication |
| **Upstash Redis** | Caching & rate limiting |
| **Vercel** | Hosting & Edge Functions |

### Mobile (iOS & Android)
| Technology | Purpose |
|------------|---------|
| **React Native** | Cross-platform mobile |
| **Expo** | Development & deployment |
| **Expo Router** | Navigation (matches Next.js patterns) |

### Why This Stack?

1. **Code Sharing**: React components work in both Next.js and React Native
2. **Expo Router**: Similar file-based routing to Next.js App Router
3. **TypeScript**: Shared types across web and mobile
4. **Tailwind + NativeWind**: Same styling system for web and mobile
5. **Vercel + Expo**: Seamless deployment pipeline

---

## Project Structure

```
elevate/
├── apps/
│   ├── web/                    # Next.js web app
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   │   ├── login/
│   │   │   │   └── register/
│   │   │   ├── (main)/
│   │   │   │   ├── page.tsx           # Home
│   │   │   │   ├── shows/
│   │   │   │   │   ├── page.tsx       # All shows
│   │   │   │   │   └── [slug]/
│   │   │   │   │       └── page.tsx   # Show detail
│   │   │   │   ├── schedule/
│   │   │   │   ├── podcasts/
│   │   │   │   └── request/
│   │   │   ├── api/
│   │   │   │   ├── radio/
│   │   │   │   │   ├── status/
│   │   │   │   │   ├── now-playing/
│   │   │   │   │   └── history/
│   │   │   │   ├── shows/
│   │   │   │   ├── podcasts/
│   │   │   │   ├── requests/
│   │   │   │   ├── schedule/
│   │   │   │   └── auth/
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── ui/                    # Shared UI components
│   │   │   ├── player/
│   │   │   │   ├── BottomPlayer.tsx
│   │   │   │   ├── NowPlaying.tsx
│   │   │   │   └── VolumeControl.tsx
│   │   │   ├── shows/
│   │   │   └── layout/
│   │   └── lib/
│   │       ├── radio-co.ts            # Radio.co API client
│   │       ├── db.ts                  # Prisma client
│   │       └── auth.ts                # Auth config
│   │
│   └── mobile/                 # React Native/Expo app
│       ├── app/
│       │   ├── (tabs)/
│       │   │   ├── index.tsx          # Home
│       │   │   ├── shows.tsx
│       │   │   ├── schedule.tsx
│       │   │   └── profile.tsx
│       │   ├── show/[id].tsx
│       │   └── _layout.tsx
│       ├── components/
│       └── hooks/
│
├── packages/
│   ├── shared/                 # Shared code
│   │   ├── types/              # TypeScript types
│   │   ├── constants/          # Shared constants
│   │   ├── utils/              # Utility functions
│   │   └── hooks/              # Shared React hooks
│   │
│   └── ui/                     # Shared UI (optional)
│       └── components/
│
├── prisma/
│   └── schema.prisma
│
├── package.json                # Monorepo config (Turborepo)
└── turbo.json
```

---

## Core Features

### Phase 1: Foundation
- [ ] Next.js app setup with current design
- [ ] Radio.co API integration
- [ ] Real-time now playing
- [ ] Track history display
- [ ] Listener count
- [ ] Responsive player (matches current sticky player)

### Phase 2: Shows & Podcasts
- [ ] Show listings (Apollo Pulse, etc.)
- [ ] Show detail pages with episodes
- [ ] YouTube video integration
- [ ] Podcast RSS feed parsing
- [ ] Episode playback (audio + video)

### Phase 3: Live Features
- [ ] Show schedule (weekly calendar)
- [ ] Live show indicator
- [ ] "Going live" notifications
- [ ] Live chat during shows
- [ ] DJ/Host profiles

### Phase 4: User Features
- [ ] User authentication (social login)
- [ ] Song request system
- [ ] Favorite shows/episodes
- [ ] Listening history
- [ ] User preferences (notifications, theme)

### Phase 5: Mobile App
- [ ] React Native/Expo setup
- [ ] Background audio playback
- [ ] Push notifications
- [ ] Offline mode (downloaded episodes)
- [ ] CarPlay/Android Auto support

### Phase 6: Analytics & Admin
- [ ] Admin dashboard
- [ ] Listener analytics
- [ ] Show performance metrics
- [ ] Content management system

---

## Backend Architecture

### Radio.co Integration Service

```typescript
// lib/radio-co.ts

const STATION_ID = 's10a9e6f11';
const BASE_URL = 'https://public.radio.co';

export const radioCo = {
  // Get full station status
  async getStatus() {
    const res = await fetch(`${BASE_URL}/stations/${STATION_ID}/status`);
    return res.json();
  },

  // Get current track
  async getCurrentTrack() {
    const res = await fetch(`${BASE_URL}/api/v2/${STATION_ID}/track/current`);
    return res.json();
  },

  // Get station info
  async getStationInfo() {
    const res = await fetch(`${BASE_URL}/api/v2/${STATION_ID}`);
    return res.json();
  },

  // Stream URL
  streamUrl: 'https://s2.radio.co/s10a9e6f11/listen'
};
```

### Real-time Updates (WebSocket/SSE)

```typescript
// app/api/radio/stream/route.ts

export async function GET() {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    async start(controller) {
      const sendUpdate = async () => {
        const status = await radioCo.getStatus();
        controller.enqueue(
          encoder.encode(`data: ${JSON.stringify(status)}\n\n`)
        );
      };

      // Send initial data
      await sendUpdate();
      
      // Poll every 10 seconds
      const interval = setInterval(sendUpdate, 10000);
      
      return () => clearInterval(interval);
    }
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

---

## Mobile App Strategy

### React Native with Expo

**Why Expo?**
- Simplified development (no native code required initially)
- Over-the-air updates
- Easy App Store/Play Store deployment
- Built-in audio/video support
- Push notifications
- Background audio

### Key Mobile Features

```typescript
// Mobile audio player setup (expo-av)

import { Audio } from 'expo-av';

export const setupAudioPlayer = async () => {
  await Audio.setAudioModeAsync({
    allowsRecordingIOS: false,
    staysActiveInBackground: true,  // Background playback
    playsInSilentModeIOS: true,
    shouldDuckAndroid: true,
    playThroughEarpieceAndroid: false,
  });
};

export const createPlayer = async (streamUrl: string) => {
  const { sound } = await Audio.Sound.createAsync(
    { uri: streamUrl },
    { shouldPlay: false }
  );
  return sound;
};
```

### Push Notifications

```typescript
// notifications.ts

import * as Notifications from 'expo-notifications';

export const registerForPushNotifications = async () => {
  const { status } = await Notifications.requestPermissionsAsync();
  if (status !== 'granted') return;

  const token = await Notifications.getExpoPushTokenAsync();
  // Save token to backend for show notifications
  return token;
};

// Schedule show reminder
export const scheduleShowReminder = async (show: Show) => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: `${show.name} is going live!`,
      body: 'Tap to tune in now',
      data: { showId: show.id },
    },
    trigger: {
      date: new Date(show.startTime),
    },
  });
};
```

---

## Database Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Users
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  image         String?
  createdAt     DateTime  @default(now())
  
  requests      SongRequest[]
  favorites     Favorite[]
  listeningHistory ListeningHistory[]
  pushTokens    PushToken[]
}

// Shows (Apollo Pulse, Elevated Show, etc.)
model Show {
  id          String    @id @default(cuid())
  name        String
  slug        String    @unique
  description String?
  image       String?
  isLive      Boolean   @default(false)
  isActive    Boolean   @default(true)
  youtubeChannel String?
  createdAt   DateTime  @default(now())
  
  episodes    Episode[]
  schedules   ShowSchedule[]
  favorites   Favorite[]
}

// Episodes (YouTube videos, podcasts)
model Episode {
  id          String    @id @default(cuid())
  showId      String
  show        Show      @relation(fields: [showId], references: [id])
  title       String
  description String?
  thumbnail   String?
  duration    Int?      // seconds
  type        EpisodeType
  sourceUrl   String    // YouTube URL, audio URL, etc.
  sourceId    String?   // YouTube video ID
  publishedAt DateTime
  createdAt   DateTime  @default(now())
  
  @@index([showId])
}

enum EpisodeType {
  YOUTUBE
  AUDIO
  LIVE
}

// Show Schedule
model ShowSchedule {
  id        String    @id @default(cuid())
  showId    String
  show      Show      @relation(fields: [showId], references: [id])
  dayOfWeek Int       // 0-6 (Sunday-Saturday)
  startTime String    // "19:00"
  endTime   String    // "21:00"
  timezone  String    @default("Africa/Johannesburg")
  isRecurring Boolean @default(true)
  
  @@index([showId])
}

// Song Requests
model SongRequest {
  id        String    @id @default(cuid())
  userId    String?
  user      User?     @relation(fields: [userId], references: [id])
  songName  String
  artist    String?
  message   String?
  status    RequestStatus @default(PENDING)
  createdAt DateTime  @default(now())
  
  @@index([status])
}

enum RequestStatus {
  PENDING
  APPROVED
  PLAYED
  REJECTED
}

// User Favorites
model Favorite {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  showId    String
  show      Show      @relation(fields: [showId], references: [id])
  createdAt DateTime  @default(now())
  
  @@unique([userId, showId])
}

// Listening History
model ListeningHistory {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  trackName String
  artist    String?
  duration  Int       // seconds listened
  timestamp DateTime  @default(now())
  
  @@index([userId, timestamp])
}

// Push Notification Tokens
model PushToken {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  token     String    @unique
  platform  String    // ios, android, web
  createdAt DateTime  @default(now())
  
  @@index([userId])
}

// Analytics Events
model AnalyticsEvent {
  id        String    @id @default(cuid())
  type      String    // play, pause, request, favorite, etc.
  userId    String?
  metadata  Json?
  timestamp DateTime  @default(now())
  
  @@index([type, timestamp])
}
```

---

## API Routes

### Radio

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/radio/status` | Get station status |
| GET | `/api/radio/now-playing` | Current track |
| GET | `/api/radio/history` | Track history |
| GET | `/api/radio/stream` | SSE for real-time updates |

### Shows

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/shows` | List all shows |
| GET | `/api/shows/[slug]` | Get show details |
| GET | `/api/shows/[slug]/episodes` | Get show episodes |
| POST | `/api/shows/[slug]/favorite` | Favorite a show |

### Schedule

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/schedule` | Get weekly schedule |
| GET | `/api/schedule/today` | Today's shows |
| GET | `/api/schedule/upcoming` | Next live show |

### Requests

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/requests` | List requests (admin) |
| POST | `/api/requests` | Submit song request |
| PATCH | `/api/requests/[id]` | Update request status |

### User

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/user/profile` | Get user profile |
| PATCH | `/api/user/profile` | Update profile |
| GET | `/api/user/favorites` | Get favorites |
| GET | `/api/user/history` | Listening history |
| POST | `/api/user/push-token` | Register push token |

---

## Implementation Roadmap

### Stage 1: Project Setup (Foundation)
```
1. Initialize Next.js 14 with TypeScript
2. Set up Tailwind CSS with current design tokens
3. Configure Prisma with PostgreSQL
4. Set up Turborepo for monorepo
5. Migrate current HTML/CSS to React components
6. Implement Radio.co API integration
```

### Stage 2: Core Features
```
1. Build sticky bottom player component
2. Implement now playing with real-time updates
3. Create shows listing and detail pages
4. Add YouTube video integration
5. Build schedule calendar
```

### Stage 3: User System
```
1. Set up NextAuth.js (Google, Apple, Email)
2. Implement song request system
3. Add favorites functionality
4. Build user preferences
```

### Stage 4: Mobile App
```
1. Initialize Expo app
2. Share types and utilities from packages/shared
3. Implement audio player with background support
4. Set up push notifications
5. Build offline mode for podcasts
6. Submit to App Store / Play Store
```

### Stage 5: Admin & Analytics
```
1. Build admin dashboard
2. Implement content management
3. Add analytics tracking
4. Set up monitoring and alerts
```

---

## Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL database
- Radio.co account with station ID

### Installation

```bash
# Clone and install
git clone https://github.com/0x3Matt/elevate.git
cd elevate

# Install dependencies
npm install

# Set up environment
cp .env.example .env.local

# Run database migrations
npx prisma migrate dev

# Start development
npm run dev
```

### Environment Variables

```env
# Database
DATABASE_URL="postgresql://..."

# Radio.co
RADIO_CO_STATION_ID="s10a9e6f11"
RADIO_CO_STREAM_URL="https://s2.radio.co/s10a9e6f11/listen"

# Auth
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000"
GOOGLE_CLIENT_ID="..."
GOOGLE_CLIENT_SECRET="..."

# Push Notifications (Expo)
EXPO_ACCESS_TOKEN="..."

# Analytics
ANALYTICS_ID="..."
```

---

## Resources

- [Radio.co API Documentation](./radio-co-api.md)
- [Next.js Documentation](https://nextjs.org/docs)
- [Expo Documentation](https://docs.expo.dev)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
