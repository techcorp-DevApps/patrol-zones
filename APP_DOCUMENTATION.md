# Patrol Zones — Melbourne CBD

## Application Documentation

**Version**: 1.0.0
**Platform**: iOS, Android, Web
**Purpose**: A mobile-first application for Melbourne City Council patrol officers providing real-time GPS location tracking, zone assignment, and patrol area navigation within the Melbourne CBD.

---

## Table of Contents

1. [Technology Stack](#technology-stack)
2. [Application Architecture](#application-architecture)
3. [Features](#features)
4. [Components](#components)
5. [Constants & Data Modules](#constants--data-modules)
6. [Patrol Zones](#patrol-zones)
7. [Backend Server](#backend-server)
8. [Database](#database)
9. [Data Generation Pipeline](#data-generation-pipeline)
10. [Routing](#routing)
11. [State Management](#state-management)
12. [Permissions](#permissions)
13. [Configuration Files](#configuration-files)
14. [NPM Scripts](#npm-scripts)
15. [Running the Application](#running-the-application)

---

## Technology Stack

### Frontend

| Technology | Version | Purpose |
|---|---|---|
| Expo SDK | 54 | React Native framework and build tooling |
| React | 19.1.0 | UI library |
| React Native | 0.81.5 | Cross-platform mobile framework |
| Expo Router | 6.0.17 | File-based routing |
| react-native-maps | 1.18.0 (pinned) | Native map rendering (Google Maps) |
| expo-location | 19.0.8 | GPS location and heading tracking |
| expo-sensors | 15.0.8 | Device sensor access (compass/magnetometer) |
| react-native-reanimated | 4.1.1 | High-performance animations |
| @tanstack/react-query | 5.83.0 | Server state management |
| @react-native-async-storage/async-storage | 2.2.0 | Persistent local storage |
| @expo/vector-icons | 15.0.3 | Icon library (Ionicons, MaterialCommunityIcons) |
| @expo-google-fonts/roboto-mono | 0.4.1 | Monospace typography |
| react-native-gesture-handler | 2.28.0 | Touch gesture recognition |
| react-native-svg | 15.12.1 | SVG rendering |
| expo-haptics | 15.0.8 | Haptic feedback |
| expo-blur | 15.0.8 | Blur effects |
| expo-glass-effect | 0.1.4 | Glass/frosted UI effects |
| expo-linear-gradient | 15.0.8 | Gradient backgrounds |
| Zod | 3.24.2 | Schema validation |
| TypeScript | 5.9.2 | Type-safe JavaScript |

### Backend

| Technology | Version | Purpose |
|---|---|---|
| Express.js | 5.0.1 | HTTP server framework |
| tsx | 4.20.6 | TypeScript execution for Node.js |
| Drizzle ORM | 0.39.3 | Database query builder and ORM |
| PostgreSQL (pg) | 8.16.3 | Database driver |
| esbuild | — | Production server bundling |

### Data Processing (Python)

| Technology | Purpose |
|---|---|
| requests | HTTP client for Melbourne Open Data APIs |
| shapely | Geometric operations and spatial analysis |
| rtree | Spatial indexing for efficient lookups |
| pyproj | Coordinate system projection |
| orjson | Fast JSON serialization |
| tqdm | Progress bar for data processing |

---

## Application Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Expo App (Frontend)                │
│                                                      │
│  ┌──────────┐  ┌────────────┐  ┌──────────────────┐ │
│  │  Expo     │  │  React     │  │  react-native-   │ │
│  │  Router   │  │  Native    │  │  maps            │ │
│  └──────────┘  └────────────┘  └──────────────────┘ │
│                                                      │
│  ┌──────────┐  ┌────────────┐  ┌──────────────────┐ │
│  │  expo-    │  │  expo-     │  │  AsyncStorage    │ │
│  │  location │  │  sensors   │  │                  │ │
│  └──────────┘  └────────────┘  └──────────────────┘ │
│                                                      │
│  ┌──────────────────────────────────────────────────┐│
│  │              Constants / Data Layer               ││
│  │  zones.ts │ streets.ts │ streetNumbers.ts        ││
│  │  parkingZones.ts │ colors.ts                     ││
│  └──────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────┐
│              Express Backend (Port 5000)             │
│         Landing Page + Expo Manifest Serving          │
└──────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────┐
│                PostgreSQL (Drizzle ORM)               │
└──────────────────────────────────────────────────────┘
```

---

## Features

### 1. Live GPS Position Tracking

- Continuous GPS updates every **2 seconds** or **3 metres** of movement
- Displays current coordinates with accuracy indicator
- Animated user position marker on the map
- "Locate Me" button to re-centre the map on the user's position

### 2. Real-Time Zone Detection

- Automatic detection of which patrol zone the officer is currently in
- Uses **point-in-polygon ray casting** algorithm against zone polygon boundaries
- Instant zone identification as the officer moves between zones
- **Out-of-zone warnings** when the officer leaves all defined patrol areas

### 3. Animated Magnetic Compass

- Real-time compass widget using the device magnetometer
- Displays **4 cardinal directions** mapped to Melbourne CBD streets:
  - **North** = La Trobe Street
  - **East** = Spring Street
  - **South** = Flinders Street
  - **West** = Spencer Street
- Grid-relative direction labels (e.g., "Toward La Trobe", "Toward Spring")
- Aligned to Melbourne's ~30-degree rotated Hoddle Grid

### 4. Interactive Map Display

- Full native map rendering via Google Maps (react-native-maps)
- **Map type toggle** with platform-specific options:
  - **iOS**: Dark, Light, Satellite, Hybrid
  - **Android**: Standard, Satellite, Hybrid
- Dark/light style switching adapts UI elements to current map type
- Zone polygon overlays with colour-coded boundaries
- **Two-tier zoom system**:
  - Zone polygons visible at `latitudeDelta < 0.012`
  - Street number markers visible at `latitudeDelta < 0.004`
- `tracksViewChanges={false}` on all markers for performance (static bitmaps)

### 5. Zone Assignment

- Officers can select and assign themselves to a specific patrol zone
- Assignment persisted locally via AsyncStorage
- Persists across app restarts

### 6. Zone Information Modal

- Overlay modal displaying detailed information about each zone:
  - Zone boundary streets
  - Patrol streets within the zone
  - Laneways and places
  - Boundary descriptions
- Accessible via a dedicated button on the map

### 7. Current Street Position Display

- Shows the nearest street name to the officer's position
- Displays the **block** the officer is on (between which cross-streets)
- Indicates which **side of the street** (East/West for N-S streets, North/South for E-W streets)
- Uses perpendicular projection onto 17 Melbourne CBD street centrelines
- Maximum detection radius of **35 metres**

### 8. Parking Zone Display

- Shows parking zone numbers for the current street segment
- **711 parking zones** across **491 street segments** (Batches 1-7 merged)
- Displayed with amber-coloured "P" prefix
- Automatically updates based on the officer's current position

### 9. Granular Street Number Markers

- **886 representative street number markers** generated from 7,000+ City of Melbourne addresses
- Approximately 3-4 markers per block per side, spaced ~30 numbers apart
- Selection algorithm picks first, last, and evenly-spaced intermediate numbers (max 5 per side)
- All markers pre-computed at module level (no viewport-dependent rendering) to prevent iOS crashes
- High-contrast styling:
  - **Dark maps**: White (#FFFFFF) bold text with dark shadow
  - **Light maps**: Dark (#1A1A2E) bold text with light shadow
  - Font size: 13px bold

### 10. Collapsible Bottom Panel

- Information panel at the bottom of the screen showing zone and location details
- **Minimize button** (chevron-down icon) hides the panel entirely to maximize the map view
- **Pull-tab with swipe-up gesture** or tap to restore the panel
- Overlay buttons (map type, zone info, locate) animate position smoothly with panel state transitions

### 11. Address Markers

- Styled as Google Maps-style grey text (no badge)
- **22-metre offset** from street centre with **8-metre perpendicular nudge** to separate N-S and E-W markers at intersections
- Dark/light variants adapt to the current map type

---

## Components

### `app/index.tsx` — Main Map Screen

The primary application screen combining all features into a unified interface:

- GPS location initialization and continuous tracking
- Map rendering with zone overlays
- Compass widget display
- Zone detection logic
- Bottom information panel
- Zone info modal trigger
- Map type toggle controls
- Coordinate and accuracy display
- Street position and parking zone readout

### `app/_layout.tsx` — Root Layout

Application entry point and provider wrapper:

- Font loading (Roboto Mono)
- Theme providers
- Navigation structure setup via Expo Router

### `components/PatrolMap.tsx` — Native Map Component

The core map rendering component:

- Renders the Google Maps native view
- Draws zone polygon overlays with colour coding
- Displays street number markers at appropriate zoom levels
- Handles map type switching (dark/light/satellite/hybrid)
- User location marker rendering
- Two-tier zoom-dependent marker visibility

### `components/PatrolMap.web.tsx` — Web Map Stub

Placeholder component for web platform:

- Displays a message indicating that native maps are not supported on web
- Ensures the app does not crash when running in a web browser

### `components/Compass.tsx` — Magnetic Compass Widget

Animated compass displaying device heading:

- Reads magnetometer data via expo-sensors
- Renders rotating compass graphic using react-native-reanimated
- Displays 4 cardinal Melbourne CBD street labels
- Grid-relative heading direction labels

### `components/ZoneInfoModal.tsx` — Zone Information Modal

Full-screen modal overlay for zone details:

- Lists all patrol streets within the selected zone
- Shows laneways, places, and roads
- Displays boundary street descriptions
- Street type classification (Council Major, Council Minor, Private)

---

## Constants & Data Modules

### `constants/zones.ts` — Patrol Zone Definitions

Defines all **15 patrol zones** as polygon coordinate arrays:

- 4-point polygon coordinates derived from OpenStreetMap intersection data
- Aligned to Melbourne's ~30-degree rotated Hoddle Grid
- Each zone includes:
  - Polygon boundary coordinates (latitude/longitude pairs)
  - Zone name and display colour
  - Patrol streets list with "from" and "to" cross-streets
  - Boundary descriptions
  - Heading labels for compass reference

### `constants/streets.ts` — Street Geometry

Defines **18 Melbourne CBD streets** as polylines:

- Each street represented as a series of real OSM intersection coordinates
- `getCurrentStreetPosition()` function: finds the closest street segment to a given GPS point across all streets
- Used for current street position display and side-of-street detection

### `constants/streetNumbers.ts` — Street Number Blocks

Contains **886 representative street number markers** from City of Melbourne data:

- `getStreetNumberMarkers()` function: generates positioned map markers for rendering
- `selectRepresentativeNumbers()` function: picks optimal markers per block side (first, last, evenly-spaced intermediate; max 5 per side)
- Data sourced from `scripts/out/melbourne_cbd_street_blocks.json`

### `constants/parkingZones.ts` — Parking Zone Lookup

Defines **711 parking zones** across **491 street segments**:

- Merged from Batches 1-7
- `getParkingZones(street, from, to)` function: returns matching zone numbers for a given street segment
- Zone numbers displayed with amber "P" prefix

### `constants/colors.ts` — Theme Colours

Dark navy theme colour palette:

- Primary background: `#0A1628`
- Consistent colour tokens used across all components
- Dark-mode-first design for outdoor visibility

---

## Patrol Zones

The Melbourne CBD is divided into **15 patrol zones**, each defined as a quadrilateral polygon matching the Hoddle Grid:

| # | Zone Name | Boundary (E-W Streets) | Boundary (N-S Streets) |
|---|---|---|---|
| 1 | FLAGSTAFF | Lonsdale — La Trobe | Spencer — William |
| 2 | SPENCER | Little Bourke — Collins | Spencer — William |
| 3 | RIALTO | Collins — Flinders | Spencer — William |
| 4 | SUPREME | La Trobe — Lonsdale | William — Queen |
| 5 | TAVISTOCK | Lonsdale — Collins | William — Queen |
| 6 | TITLES | La Trobe — Lonsdale | Queen — Elizabeth |
| 7 | HARDWARE | Lonsdale — Collins | Queen — Elizabeth |
| 8 | BANKS | Little Collins — Flinders | Queen — Elizabeth |
| 9 | THE MAC | Franklin — La Trobe | Elizabeth — Swanston |
| 10 | LIBRARY | La Trobe — Little Bourke | Elizabeth — Russell |
| 11 | CHINA TOWN | Little Bourke — Collins | Elizabeth — Russell |
| 12 | CITY SQUARE | Collins — Flinders | Elizabeth — Russell |
| 13 | PRINCESS | Victoria — Bourke | Russell — Spring |
| 14 | HYATT | Bourke — Flinders | Russell — Exhibition |
| 15 | TWIN TOWERS | Bourke — Flinders | Exhibition — Spring |

Each zone contains a curated list of patrol streets, laneways, and places sourced from the `com_assets_by_area.json` dataset (City of Melbourne).

---

## Backend Server

### `server/index.ts` — Express Server

- **Port**: 5000
- **Responsibilities**:
  - Serves a landing page for browser-based access
  - Serves the Expo manifest for device connections
  - HTTP proxy middleware for development
- **Build**: Bundled with esbuild for production (`server_dist/index.js`)

---

## Database

### Configuration

- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Schema**: Defined in `shared/schema.ts`
- **Migrations**: Output to `./migrations` directory
- **Validation**: Drizzle-Zod integration for schema-to-validation bridging
- **Connection**: Via `DATABASE_URL` environment variable

### Schema (`shared/schema.ts`)

The Drizzle ORM schema defines the application's data model with Zod integration for runtime validation.

---

## Data Generation Pipeline

### `scripts/generate_melbourne_cbd_street_blocks.py`

A Python script that fetches authoritative street data from City of Melbourne Open Data:

**Data Sources**:
- **Road Segments** (`road-segment` dataset): Polygon corridors with `streetname`, `streetfrom`, `streetto` — converted to centrelines via polygon edge midpoint extraction for side-of-street inference
- **Street Addresses** (`street-addresses` dataset): Streamed via `/exports/jsonl` endpoint with `in_bbox()` spatial filter to avoid the 10,000 offset cap

**Output**: `scripts/out/melbourne_cbd_street_blocks.json`
- 198 streets
- ~14,000 street numbers
- Organised by block (between-streets) and side (north/south or east/west)

**Usage**:
```bash
python scripts/generate_melbourne_cbd_street_blocks.py --outdir scripts/out --single
```

**System Dependencies**: `libspatialindex` (for rtree spatial indexing)

### `com_assets_by_area.json`

Pre-generated dataset of Melbourne CBD assets organised by patrol zone:

- **Jurisdiction**: City of Melbourne (CBD)
- Contains lanes, places, streets, and roads per zone
- Each asset includes:
  - `name`: Street/lane/place name
  - `off` (optional): Parent street the asset is off of
  - `between` (optional): Cross-street boundaries (`from` and `to`)
  - `type`: Asset classification (`lane`, `place`, `street`, `road`)
  - `str_type`: Council classification (`Council Major`, `Council Minor`, `Private`)

---

## Routing

The application uses **Expo Router** with file-based routing:

| Route | File | Description |
|---|---|---|
| `/` | `app/index.tsx` | Main map screen with all patrol features |
| Layout | `app/_layout.tsx` | Root layout with font loading and providers |

**Configuration**:
- Typed routes enabled via `experiments.typedRoutes: true`
- Deep linking scheme: `patrolzones://`
- Web origin: configured for Replit deployment

---

## State Management

The application uses a layered state management approach:

| Layer | Technology | Usage |
|---|---|---|
| Local Component State | React `useState` / `useRef` | UI state, map region, panel visibility |
| Persistent Storage | AsyncStorage | Zone assignments, user preferences |
| Server State | React Query (@tanstack/react-query) | Cached data fetching and synchronisation |
| Sensor State | expo-location, expo-sensors | GPS coordinates, compass heading |

---

## Permissions

### iOS

| Permission | Description |
|---|---|
| `NSLocationWhenInUseUsageDescription` | Location access while the app is in use — for map position and zone detection |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Background location access — for continuous zone tracking |

### Android

| Permission | Description |
|---|---|
| `ACCESS_FINE_LOCATION` | Precise GPS location |
| `ACCESS_COARSE_LOCATION` | Approximate location |

---

## Configuration Files

| File | Purpose |
|---|---|
| `package.json` | NPM dependencies, scripts, and project metadata |
| `tsconfig.json` | TypeScript compiler options with path aliases (`@/*`, `@shared/*`) |
| `app.json` | Expo app manifest — name, icons, splash screen, permissions, plugins |
| `babel.config.js` | Babel preset for Expo with `unstable_transformImportMeta` |
| `metro.config.js` | Metro bundler configuration (default Expo config) |
| `eslint.config.js` | ESLint configuration (Expo preset) |
| `drizzle.config.ts` | Drizzle ORM migration configuration |
| `pyproject.toml` | Python dependencies for data generation scripts |

---

## NPM Scripts

| Script | Command | Description |
|---|---|---|
| `start` | `npx expo start` | Start Expo development server |
| `expo:dev` | `npx expo start --localhost` | Start Expo with Replit proxy configuration |
| `server:dev` | `tsx server/index.ts` | Start Express backend in development mode |
| `server:build` | `esbuild server/index.ts ...` | Bundle Express server for production |
| `server:prod` | `node server_dist/index.js` | Run production Express server |
| `db:push` | `drizzle-kit push` | Push Drizzle ORM schema migrations |
| `lint` | `npx expo lint` | Run ESLint |
| `lint:fix` | `npx expo lint --fix` | Run ESLint with auto-fix |
| `postinstall` | `patch-package` | Apply package patches after install |

---

## Running the Application

### Development

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Start the Express backend** (port 5000):
   ```bash
   npm run server:dev
   ```

3. **Start the Expo frontend** (port 8081):
   ```bash
   npm run expo:dev
   ```

4. **Test on device**: Scan the QR code from the terminal or Replit URL bar using Expo Go.

### Production

1. **Build the server**:
   ```bash
   npm run server:build
   ```

2. **Run production server**:
   ```bash
   npm run server:prod
   ```

### Data Generation

To regenerate street block data from City of Melbourne Open Data:

```bash
python scripts/generate_melbourne_cbd_street_blocks.py --outdir scripts/out --single
```

Requires Python with dependencies from `pyproject.toml` and `libspatialindex` system library.

---

## App Configuration

| Setting | Value |
|---|---|
| App Name | Patrol Zones |
| Bundle ID (iOS) | com.patrolzones |
| Package (Android) | com.patrolzones |
| Orientation | Portrait only |
| UI Style | Dark |
| New Architecture | Enabled |
| React Compiler | Enabled (experimental) |
| Typed Routes | Enabled (experimental) |
| Splash Background | #0A1628 (Dark Navy) |
