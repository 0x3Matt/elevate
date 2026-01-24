# Elevate Radio - Features Roadmap

## Current Features (Static HTML)

- [x] Hero section with rotating logo
- [x] Sticky bottom audio player
- [x] Apollo Pulse featured show with YouTube videos
- [x] Coming soon shows (Elevated Show, Head to Head EDM, etc.)
- [x] Responsive design (mobile-first)
- [x] PWA meta tags

---

## Feature Details

### 1. Song Request System

**User Flow:**
1. User clicks "Request Song" button
2. Modal opens with form (song name, artist, optional message)
3. User submits request
4. Request goes to queue for DJ approval
5. DJ can approve/reject from admin panel
6. Approved songs appear in "Upcoming Requests" display

**Backend:**
- `POST /api/requests` - Submit request
- `GET /api/requests` - Get queue (admin)
- `PATCH /api/requests/:id` - Update status
- Rate limiting: Max 3 requests per user per hour

**UI Components:**
- Request button on player
- Request modal/form
- Request queue display (optional)
- Admin queue management

---

### 2. Live Shows & Schedule

**How Live Shows Work:**

1. **Scheduled Shows**
   - Shows have recurring schedules (e.g., "Apollo Pulse - Tuesdays & Thursdays 7pm")
   - Calendar view shows weekly schedule
   - Users can set reminders

2. **Going Live**
   - When DJ starts broadcasting on Radio.co, status changes from "autodj" to "live"
   - App detects this via API polling
   - Push notification sent: "Apollo Pulse is LIVE!"
   - UI updates to show live indicator

3. **Recording**
   - Live shows can be recorded
   - Recording uploaded to YouTube
   - Episode added to show's podcast feed

**API Integration:**
```typescript
// Detect live status from Radio.co
const status = await radioCo.getStatus();
const isLive = status.source.type === 'live';
const currentDJ = status.source.collaborator;
```

**Database:**
- `ShowSchedule` - Recurring show times
- `Episode` - Recorded shows (YouTube links)

---

### 3. Podcast System

**Features:**
- Each show has its own podcast feed
- Episodes from YouTube automatically imported
- Audio-only episodes supported
- Download for offline listening (mobile)

**Episode Sources:**
1. **YouTube Videos** - Embedded player
2. **Audio Files** - Native audio player
3. **Live Recordings** - Archived from Radio.co

**YouTube Integration:**
```typescript
// Fetch videos from YouTube channel
const response = await fetch(
  `https://www.googleapis.com/youtube/v3/playlistItems?` +
  `playlistId=${PLAYLIST_ID}&part=snippet&maxResults=50&key=${API_KEY}`
);

// Transform to episodes
const episodes = data.items.map(item => ({
  title: item.snippet.title,
  thumbnail: item.snippet.thumbnails.high.url,
  sourceId: item.snippet.resourceId.videoId,
  publishedAt: item.snippet.publishedAt,
  type: 'YOUTUBE'
}));
```

---

### 4. User Accounts

**Authentication Options:**
- Google Sign-In
- Apple Sign-In (required for iOS)
- Email/Password
- Guest mode (limited features)

**User Features:**
- Favorite shows
- Listening history
- Request history
- Notification preferences
- Theme preference (dark/light)

**Profile Data:**
```typescript
interface UserProfile {
  id: string;
  name: string;
  email: string;
  image?: string;
  preferences: {
    notifications: {
      showGoingLive: boolean;
      newEpisodes: boolean;
      requestPlayed: boolean;
    };
    theme: 'dark' | 'light' | 'system';
  };
}
```

---

### 5. Push Notifications

**Notification Types:**

| Type | Trigger | Message |
|------|---------|---------|
| Show Live | DJ starts broadcasting | "Apollo Pulse is LIVE! Tune in now" |
| New Episode | Episode published | "New episode: [Title] available now" |
| Request Played | User's request played | "Your song request is playing!" |
| Reminder | 15 min before scheduled show | "Apollo Pulse starts in 15 minutes" |

**Implementation:**
- Web: Service Worker + Push API
- iOS/Android: Expo Push Notifications
- Backend: Store tokens, send via Expo Push Service

---

### 6. Analytics & Tracking

**What to Track:**

| Event | Data |
|-------|------|
| `play` | timestamp, duration, user_id |
| `pause` | timestamp, total_time_listened |
| `request_submit` | song, artist, user_id |
| `show_view` | show_id, user_id |
| `episode_play` | episode_id, user_id, watch_time |
| `favorite_add` | show_id, user_id |

**Dashboard Metrics:**
- Total listeners (live)
- Peak listening times
- Popular shows/episodes
- Request volume
- User retention
- Geographic distribution

---

### 7. Admin Dashboard

**Features:**

1. **Content Management**
   - Add/edit shows
   - Manage episodes
   - Update schedule

2. **Request Queue**
   - View pending requests
   - Approve/reject
   - Mark as played

3. **Analytics**
   - Listener graphs
   - Show performance
   - User engagement

4. **User Management**
   - View users
   - Manage roles
   - Handle reports

**Access Control:**
```typescript
enum UserRole {
  USER = 'user',
  DJ = 'dj',           // Can manage requests
  ADMIN = 'admin',     // Full access
}
```

---

### 8. Mobile-Specific Features

**iOS:**
- Background audio playback
- Lock screen controls
- CarPlay integration
- Siri shortcuts ("Hey Siri, play Elevate Radio")
- Widget for home screen

**Android:**
- Background audio playback
- Notification controls
- Android Auto integration
- Widget for home screen

**Both Platforms:**
- Offline downloads (podcasts)
- Picture-in-picture (video)
- AirPlay/Chromecast support
- Sleep timer

---

## Feature Priority Matrix

| Feature | Impact | Effort | Priority |
|---------|--------|--------|----------|
| Next.js Migration | High | High | 1 |
| Radio.co Integration | High | Low | 1 |
| User Authentication | High | Medium | 2 |
| Song Requests | High | Medium | 2 |
| Show Schedule | Medium | Low | 2 |
| Push Notifications | High | Medium | 3 |
| Mobile App | High | High | 3 |
| Podcast Downloads | Medium | Medium | 4 |
| Analytics Dashboard | Medium | High | 4 |
| Admin CMS | Medium | High | 5 |

---

## Technical Decisions

### Why Next.js?
1. **Server Components** - Better performance, SEO
2. **API Routes** - Backend in same codebase
3. **Vercel** - Easy deployment, edge functions
4. **React** - Component reuse with React Native

### Why React Native/Expo?
1. **Code Sharing** - 60-80% shared with web
2. **Native Feel** - Real native components
3. **Expo** - Simplified development, OTA updates
4. **Background Audio** - Critical for radio app

### Why PostgreSQL?
1. **Relational** - Shows, episodes, users relationships
2. **Scalable** - Handles growth
3. **Prisma** - Great TypeScript integration
4. **Hosting** - Supabase, Neon, Railway options

### Why Monorepo (Turborepo)?
1. **Shared Types** - One source of truth
2. **Shared Utils** - Reusable functions
3. **Consistent** - Same linting, formatting
4. **Efficient** - Caching, parallel builds

---

## Milestones

### MVP (4-6 weeks)
- [ ] Next.js app with current design
- [ ] Radio.co real-time integration
- [ ] Basic show pages
- [ ] Song request form

### V1.0 (8-10 weeks)
- [ ] User authentication
- [ ] Show schedules
- [ ] Request system with admin
- [ ] Push notifications (web)

### V2.0 - Mobile (12-16 weeks)
- [ ] React Native app
- [ ] Background playback
- [ ] Mobile push notifications
- [ ] App Store submission

### V3.0 - Growth (Ongoing)
- [ ] Analytics dashboard
- [ ] Advanced admin features
- [ ] Community features
- [ ] Monetization options
