# Daily Dashboard

A beautiful, responsive web dashboard that displays the current time, weather conditions, and inspirational quotes with dynamic backgrounds that change based on time of day and weather.

## Features

- **City Search**: Search for any city worldwide using the OpenWeatherMap Geocoding API
- **Multiple City Support**: When searching for common city names (like "Springfield" or "Portland"), a dropdown shows all matching cities to choose from
- **Current Time**: Displays the local time for the selected city's timezone
- **Weather Conditions**: Shows current temperature and conditions with weather icons
- **Inspirational Quotes**: Random motivational quotes from various authors
- **Dynamic Backgrounds**: Background changes based on time of day:
  - Morning (5am-8am): Sunrise gradient with sun
  - Day (8am-5pm): Blue sky gradient with sun
  - Evening (5pm-8pm): Sunset gradient with warm colors
  - Night (8pm-5am): Dark gradient with moon and twinkling stars
- **Weather Effects**: Animated weather overlays including:
  - Floating clouds for cloudy conditions
  - Rain drops for rainy weather
  - Snow particles for snowy conditions
- **Location History**: View and quickly switch between previously searched cities
- **Cloud Storage**: Location history stored in Supabase (persists across sessions)
- **Responsive Design**: Works on desktop and mobile devices

## Tech Stack

- **HTML5** - Single-page application structure
- **Tailwind CSS** - Utility-first CSS framework (loaded via CDN)
- **Vanilla JavaScript** - No framework dependencies
- **OpenWeatherMap API** - Weather data and geocoding
- **Supabase** - Cloud database for location history
- **Vercel** - Hosting and deployment

## APIs & Services

### OpenWeatherMap Geocoding API
Used to search for cities by name and get coordinates.
```
https://api.openweathermap.org/geo/1.0/direct?q={city}&limit=5&appid={API_KEY}
```

### OpenWeatherMap Weather API
Used to fetch current weather conditions using latitude/longitude.
```
https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API_KEY}&units=imperial
```

### DummyJSON Quotes API
Used to fetch random inspirational quotes.
```
https://dummyjson.com/quotes/random
```

### Supabase
Cloud PostgreSQL database for storing location history.
- **Table**: `location_history`
- **Authentication**: Anonymous (device ID based)
- **Security**: Row Level Security (RLS) enabled

## Database Schema

The `location_history` table in Supabase:

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

ALTER TABLE location_history ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow anonymous access" ON location_history
  FOR ALL USING (true) WITH CHECK (true);
```

## Project Structure

```
daily-dashboard/
├── index.html      # Main application (HTML, CSS, and JavaScript)
└── README.md       # This documentation file
```

## Hosting Setup

### GitHub Repository
- **Repository**: https://github.com/todddata/daily-dashboard
- **Branch**: `main`
- Changes pushed to `main` automatically trigger deployment

### Vercel Deployment
- **Live URL**: https://daily-dashboard-rho.vercel.app
- **Deployment**: Automatic on push to GitHub
- **Framework**: None (static HTML)
- **Build Settings**: No build step required

### Supabase Project
- **Project**: WeatherTime
- **Region**: US East
- **Database**: PostgreSQL with Row Level Security

### Deployment Workflow
1. Make changes to `index.html` locally
2. Commit changes: `git add . && git commit -m "Description of changes"`
3. Push to GitHub: `git push`
4. Vercel automatically detects the push and deploys (typically within 30 seconds)

## Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/todddata/daily-dashboard.git
   cd daily-dashboard
   ```

2. Open `index.html` in a web browser, or use a local server:
   ```bash
   # Using Python
   python -m http.server 8000

   # Using Node.js (npx)
   npx serve
   ```

3. Visit `http://localhost:8000` in your browser

## Configuration

### API Keys

The app uses the following API keys (configured in `index.html`):

| Service | Key Type | Notes |
|---------|----------|-------|
| OpenWeatherMap | API Key | Free tier, rate-limited |
| Supabase | Anon/Public Key | Designed to be public, RLS protects data |

**Note**: These keys are intentionally included in the client-side code:
- OpenWeatherMap free tier keys are rate-limited
- Supabase anon keys are meant to be public (Row Level Security controls access)

### Setting Up Your Own Instance

1. **OpenWeatherMap**:
   - Sign up at [openweathermap.org](https://openweathermap.org/api)
   - Generate a free API key
   - Replace `OPENWEATHER_API_KEY` in `index.html`

2. **Supabase**:
   - Create a project at [supabase.com](https://supabase.com)
   - Run the SQL schema (see Database Schema section)
   - Replace `SUPABASE_URL` and `SUPABASE_ANON_KEY` in `index.html`

## Browser Support

- Chrome (recommended)
- Firefox
- Safari
- Edge

## License

This project is open source and available for personal use.

---

Built with Claude Code
