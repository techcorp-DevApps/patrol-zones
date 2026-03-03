# Patrol Zones — Melbourne CBD

## Project Overview
A mobile-first React Native (Expo) app for Melbourne City Council patrol officers. Provides real-time GPS location on a Melbourne CBD patrol zone map, a live magnetic compass, and daily zone assignment management.

## Architecture

### Frontend (Expo + React Native)
- **Framework**: Expo SDK 54, Expo Router (file-based routing)
- **Maps**: react-native-maps@1.18.0 (pinned — required)
- **Location**: expo-location (GPS + heading watchdog)
- **Fonts**: @expo-google-fonts/roboto-mono
- **Animation**: react-native-reanimated
- **Storage**: @react-native-async-storage/async-storage
- **Icons**: @expo/vector-icons (Ionicons, MaterialCommunityIcons)

### Backend (Express + TypeScript)
- Serves landing page and Expo manifest
- Port 5000

## Key Files

| File | Purpose |
|------|---------|
| `app/index.tsx` | Main map screen — GPS, zones, compass, panel, zone info modal |
| `app/_layout.tsx` | Root layout with fonts, providers |
| `constants/zones.ts` | All 15 patrol zone polygon definitions + patrol streets + boundaries + heading labels |
| `constants/streets.ts` | Melbourne CBD street geometry — 18 streets defined as polylines through real OSM intersection coordinates; `getCurrentStreetPosition()` finds closest segment across all streets |
| `constants/colors.ts` | Dark navy theme colors |
| `components/Compass.tsx` | Animated magnetic compass widget |
| `components/PatrolMap.tsx` | Native map with zone polygons (react-native-maps) |
| `components/PatrolMap.web.tsx` | Web stub (native maps not supported) |
| `components/ZoneInfoModal.tsx` | Modal overlay showing zone boundaries + patrol streets |

## Patrol Zones (15 total)

All zones use proper 4-point polygon coordinates derived from OpenStreetMap intersection data, matching Melbourne's ~30° rotated Hoddle Grid. Each zone is a quadrilateral with corners at actual street intersection coordinates (not axis-aligned rectangles):

| Zone | Boundaries |
|------|-----------|
| FLAGSTAFF | Spencer–William, Lonsdale–La Trobe |
| SPENCER | Spencer–William, Little Bourke–Collins |
| RIALTO | Spencer–William, Collins–Flinders |
| SUPREME | William–Queen, La Trobe–Lonsdale |
| TAVISTOCK | William–Queen, Lonsdale–Collins |
| TITLES | Queen–Elizabeth, La Trobe–Lonsdale |
| HARDWARE | Queen–Elizabeth, Lonsdale–Collins |
| BANKS | Queen–Elizabeth, Little Collins–Flinders |
| THE MAC | Elizabeth–Swanston, Franklin–La Trobe |
| LIBRARY | Elizabeth–Russell, La Trobe–Little Bourke |
| CHINA TOWN | Elizabeth–Russell, Little Bourke–Collins |
| CITY SQUARE | Elizabeth–Russell, Collins–Flinders |
| PRINCESS | Russell–Spring, Victoria–Bourke |
| HYATT | Russell–Exhibition, Bourke–Flinders |
| TWIN TOWERS | Exhibition–Spring, Bourke–Flinders |

## Features
- Live GPS position (updates every 2s / 3m distance)
- Real-time zone detection (point-in-polygon ray casting)
- Magnetic compass with Melbourne CBD street labels (La Trobe=N, Spring=E, Flinders=S, Spencer=W) — 4 cardinal directions only, no sub-directional points
- Map type toggle (iOS: Dark/Light/Satellite/Hybrid; Android: Standard/Satellite/Hybrid) with dark/light style switching
- Zone assignment (persisted via AsyncStorage)
- Zone info modal showing patrol streets, laneways, and boundary details
- Out-of-zone warnings
- Coordinate display with accuracy indicator
- Current street position display showing nearest street name, which block (between cross-streets), and which side (East/West for N-S streets, North/South for E-W streets) — uses perpendicular projection onto 17 Melbourne CBD street centerlines with 35m max detection radius
- Compass heading with Melbourne CBD grid-relative direction labels (Toward La Trobe/Spring/Flinders/Spencer)

## Running the App
- Start Backend: `npm run server:dev` (port 5000)
- Start Frontend: `npm run expo:dev` (port 8081)
- Scan QR from Replit's URL bar to test on device via Expo Go
