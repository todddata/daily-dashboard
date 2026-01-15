# Daily Dashboard - Technical Architecture

> Complete technical documentation for Claude and developers

## Architecture Overview

This is a **serverless single-page application** that combines:
- **Frontend**: Static HTML/CSS/JS hosted on Vercel
- **Backend**: Vercel Serverless Functions (Node.js)
- **Database**: Supabase (PostgreSQL)
- **APIs**: OpenWeatherMap (weather data)

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │
       ├─────────────────► index.html (Static)
       │                   └─ Tailwind CSS (CDN)
       │                   └─ Supabase Client (CDN)
       │
       ├─────────────────► /api/geocode (Serverless)
       │                   └─ OpenWeatherMap Geocoding API
       │
       ├─────────────────► /api/weather (Serverless)
       │                   └─ OpenWeatherMap Weather API
       │
       └─────────────────► Supabase Database
                           └─ location_history table
```

## Tech Stack

### Frontend
- **HTML5**: Single-page application structure
- **Tailwind CSS v3**: Utility-first CSS (CDN)
- **Vanilla JavaScript**: No framework dependencies
- **Supabase JS Client v2**: Database client (CDN)

### Backend (Serverless Functions)
- **Vercel Serverless Functions**: Node.js runtime
- **API Routes**: `/api/geocode.js`, `/api/weather.js`
- **Environment Variables**: Secure key storage

### Database
- **Supabase**: Managed PostgreSQL
- **Row Level Security**: Enabled for public access control
- **Anonymous Authentication**: Device ID-based tracking

### External APIs
- **OpenWeatherMap API**: Weather and geocoding data
- **DummyJSON API**: Random inspirational quotes

### Hosting & Deployment
- **Vercel**: Edge hosting with automatic deployments
- **GitHub**: Source control with automatic CI/CD

## Project Structure

```
daily-dashboard/
├── index.html              # Main SPA (HTML + CSS + JS)
├── api/
│   ├── geocode.js         # Serverless function for city search
│   └── weather.js         # Serverless function for weather data
├── README.md              # User-facing documentation
└── CLAUDE.md              # This file - technical architecture
```

## API Routes (Serverless Functions)

### GET /api/geocode
**Purpose**: Search for cities by name (proxies OpenWeatherMap Geocoding API)

**Parameters**:
- `q` (required): City name to search (e.g., "Denver")

**Response**: Array of city objects
```json
[
  {
    "name": "Denver",
    "state": "Colorado",
    "country": "US",
    "lat": 39.7392358,
    "lon": -104.990251
  }
]
```

**Implementation**: `api/geocode.js`
- Validates query parameter
- Reads `OPENWEATHER_API_KEY` from environment
- Proxies request to OpenWeatherMap
- Returns JSON response

### GET /api/weather
**Purpose**: Get current weather for coordinates (proxies OpenWeatherMap Weather API)

**Parameters**:
- `lat` (required): Latitude
- `lon` (required): Longitude

**Response**: Weather object
```json
{
  "main": {
    "temp": 53.5
  },
  "weather": [
    {
      "description": "overcast clouds"
    }
  ],
  "name": "Denver",
  "sys": {
    "country": "US"
  },
  "timezone": -25200
}
```

**Implementation**: `api/weather.js`
- Validates lat/lon parameters
- Reads `OPENWEATHER_API_KEY` from environment
- Requests data in imperial units (Fahrenheit)
- Returns JSON response

## Database Schema (Supabase)

### Table: `location_history`

Stores all cities searched by users, keyed by device ID.

```sql
CREATE TABLE location_history (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  device_id TEXT NOT NULL,
  city_name TEXT NOT NULL,
  state TEXT,
  country TEXT NOT NULL,
  lat DECIMAL NOT NULL,
  lon DECIMAL NOT NULL,
  display_name TEXT NOT NULL,
  first_used TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(device_id, lat, lon)
);
```

**Indexes**:
- Primary key on `id`
- Unique constraint on `(device_id, lat, lon)` - prevents duplicate cities per device

**Row Level Security**:
```sql
ALTER TABLE location_history ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow anonymous access" ON location_history
  FOR ALL USING (true) WITH CHECK (true);
```

**Data Model**:
- `device_id`: Generated client-side, stored in localStorage
- `city_name`, `state`, `country`: From OpenWeatherMap geocoding
- `lat`, `lon`: Coordinates for weather lookup
- `display_name`: Formatted string (e.g., "Denver, CO, US")
- `first_used`: Timestamp of first search

## Environment Variables (Vercel)

| Variable | Type | Purpose | Location |
|----------|------|---------|----------|
| `OPENWEATHER_API_KEY` | Secret | OpenWeatherMap API access | Vercel Settings |
| `SUPABASE_URL` | Public | Supabase project URL | Client code |
| `SUPABASE_ANON_KEY` | Public* | Supabase anon key | Client code |

*SUPABASE_ANON_KEY is designed to be public - Row Level Security protects the data.

### Setting Environment Variables in Vercel

1. Go to project **Settings** → **Environment Variables**
2. Add `OPENWEATHER_API_KEY` with your API key
3. Select all environments (Production, Preview, Development)
4. Redeploy to apply changes

## Security Model

### API Key Protection
- **OpenWeatherMap Key**: Server-side only (environment variable)
- Never exposed in client code
- Serverless functions proxy all API requests
- GitHub secret scanning won't detect it

### Supabase Security
- **Anon Key**: Public by design (included in client code)
- **Row Level Security**: Controls data access at database level
- **Anonymous Authentication**: Device ID-based (no user accounts)

### Device Tracking
```javascript
// Generated on first visit
const DEVICE_ID = 'device_' + random() + '_' + timestamp;
// Stored in localStorage
// Used for all Supabase queries
```

## Deployment Pipeline

### Automatic Deployment (via GitHub)
```
Local Changes → Git Commit → Git Push → GitHub → Vercel Deploy
```

**Trigger**: Any push to `main` branch

**Build Process**:
1. Vercel detects new commit
2. Builds serverless functions (`api/*.js`)
3. Deploys static files (`index.html`)
4. Deploys to edge network
5. Updates live site (30-60 seconds)

**No build step required** for static HTML, but Vercel processes:
- Serverless function bundling
- CDN distribution
- SSL certificate

### Manual Deployment
Via Vercel dashboard:
1. **Deployments** → Latest deployment
2. Click **...** → **Redeploy**

## Client-Side Architecture

### Key Components

**1. City Search Flow**
```
User types city → /api/geocode → Display results → User selects
→ Save to Supabase → /api/weather → Update UI
```

**2. Weather Display**
```
City coordinates → /api/weather → Parse response
→ Update temp, conditions, icon → Calculate time of day
→ Update background theme → Store timezone offset
```

**3. History Management**
```
Click "Previous Locations" → Query Supabase by device_id
→ Render sorted list (most recent first) → Click city
→ Load weather → Update UI
```

**4. Dynamic Backgrounds**
- Time-based: Morning (5-8am), Day (8am-5pm), Evening (5-8pm), Night (8pm-5am)
- Weather effects: Clouds, rain, snow, stars, sun/moon
- Smooth CSS transitions (1s duration)

### State Management
- **Current City**: localStorage (`selectedCity`)
- **Device ID**: localStorage (`deviceId`)
- **Location History**: Supabase (`location_history` table)
- **Timezone Offset**: Memory variable (from weather API)

### API Client Pattern

All external API calls follow this pattern:
```javascript
async function apiCall(endpoint, params) {
  try {
    const response = await fetch(endpoint + queryString(params));
    if (!response.ok) throw new Error('API Error');
    return await response.json();
  } catch (error) {
    // Handle error, show user message
  }
}
```

No external HTTP libraries - using native `fetch()`.

## Development Workflow

### Local Development
```bash
cd /Users/toddmcguire/daily-dashboard
python -m http.server 8000
# Visit http://localhost:8000
```

**Note**: Serverless functions won't work locally (they need Vercel runtime). Test against the live Vercel deployment or use `vercel dev`.

### Making Changes
1. Edit `index.html` for frontend changes
2. Edit `api/*.js` for backend changes
3. Commit: `git add . && git commit -m "description"`
4. Push: `git push`
5. Wait for Vercel deploy (~30s)
6. Test on live site

### Testing
- **No automated tests** currently
- Manual testing via browser
- Check browser console for errors
- Verify Supabase table for data writes

## API Rate Limits

### OpenWeatherMap (Free Tier)
- **60 calls/minute**
- **1,000,000 calls/month**
- Current usage: ~2-3 calls per user session

### Supabase (Free Tier)
- **500 MB database**
- **2 GB bandwidth/month**
- **Unlimited API requests**

### DummyJSON
- **No documented limits**
- Public free API

## Monitoring & Debugging

### Vercel Function Logs
- **Dashboard** → **Deployments** → Click deployment → **Functions**
- Shows request/response for each serverless function call
- Console logs from `console.log()` appear here

### Supabase Table Editor
- **Dashboard** → **Table Editor** → `location_history`
- View all stored cities and device IDs
- Manually query/edit data

### Browser DevTools
- **Console**: Check for JavaScript errors
- **Network**: Inspect API calls (`/api/geocode`, `/api/weather`)
- **Application** → **Local Storage**: View `deviceId`, `selectedCity`

## Common Issues & Solutions

### "API key not configured"
- Environment variable not set in Vercel
- Solution: Add `OPENWEATHER_API_KEY` in Vercel settings, redeploy

### "Failed to fetch weather data"
- Invalid coordinates or network error
- Check Vercel function logs for details

### Location history not appearing
- Supabase connection issue or RLS policy problem
- Check browser console for errors
- Verify RLS policy exists in Supabase

### Background not updating
- JavaScript error preventing execution
- Check browser console
- Verify time-of-day calculation logic

## Future Enhancement Ideas

- [ ] User authentication (replace device ID with actual accounts)
- [ ] Forecast data (next 5 days)
- [ ] Weather alerts
- [ ] Multiple saved locations
- [ ] Celsius/Fahrenheit toggle
- [ ] Dark mode toggle (manual override)
- [ ] PWA support (offline mode)
- [ ] Analytics (PostHog, Plausible)

## Links

- **Live Site**: https://daily-dashboard-rho.vercel.app
- **GitHub Repo**: https://github.com/todddata/daily-dashboard
- **Supabase Dashboard**: https://supabase.com/dashboard/project/lqbsykztkcjtwlyjqzaq
- **Vercel Dashboard**: https://vercel.com/dashboard
- **OpenWeatherMap API Docs**: https://openweathermap.org/api

---

**Last Updated**: January 14, 2026
**Built with**: Claude Code (Opus 4.5)
