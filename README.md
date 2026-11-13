# Uber Ride Booking App

> A cross-platform ride-booking app built with Expo, React Native, and Expo Router.

This project demonstrates a complete ride-booking flow: users can create an account, choose a destination, view nearby drivers, select a ride, pay securely with Stripe, and review their recent rides.

## Features

- Onboarding, sign-up, sign-in, and Google OAuth with Clerk
- Current-location detection and reverse geocoding
- Google Places destination search and autocomplete
- Interactive map with nearby driver markers and route directions
- Driver selection with fare, pickup time, rating, and seat information
- Stripe payment flow for booking a ride
- Recent rides and ride history
- Profile and chat screens
- Expo Router file-based navigation
- NativeWind utility classes and custom Jakarta fonts
- iOS, Android, and web targets through Expo

## Tech stack

| Area | Technology |
| --- | --- |
| App | React Native 0.74, Expo SDK 51 |
| Navigation | Expo Router |
| Language | TypeScript |
| Styling | NativeWind and Tailwind CSS |
| Authentication | Clerk Expo |
| Maps and location | `react-native-maps`, Expo Location, Google Places |
| Payments | Stripe React Native and Stripe server SDK |
| Database | Neon PostgreSQL |
| State | Zustand |
| Testing | Jest and `jest-expo` |

## Requirements

Install the following before starting:

- Node.js 18 or newer
- npm
- An Expo-compatible device or emulator
- A Clerk application
- A Neon PostgreSQL database
- Google Maps Platform APIs
- A Geoapify API key
- A Stripe account

For the best native experience, use a physical device or an Android/iOS emulator. Some native modules, including maps and Stripe, may not behave fully in a browser.

## Getting started

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create a `.env` file in the project root:

```env
# Public values exposed to the Expo client
EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
EXPO_PUBLIC_PLACES_API_KEY=...
EXPO_PUBLIC_DIRECTIONS_API_KEY=...
EXPO_PUBLIC_GEOAPIFY_API_KEY=...
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

# Server-only values
DATABASE_URL=postgresql://...
STRIPE_SECRET_KEY=sk_test_...
```

Never commit `.env` or expose `STRIPE_SECRET_KEY` in client-side code. Restart Expo after changing environment variables so Metro reloads the configuration.

### 3. Prepare the database

Run the following SQL in your Neon SQL editor:

```sql
CREATE TABLE drivers (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    profile_image_url TEXT,
    car_image_url TEXT,
    car_seats INTEGER NOT NULL CHECK (car_seats > 0),
    rating DECIMAL(3, 2) CHECK (rating >= 0 AND rating <= 5)
);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    clerk_id VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE rides (
    ride_id SERIAL PRIMARY KEY,
    origin_address VARCHAR(255) NOT NULL,
    destination_address VARCHAR(255) NOT NULL,
    origin_latitude DECIMAL(9, 6) NOT NULL,
    origin_longitude DECIMAL(9, 6) NOT NULL,
    destination_latitude DECIMAL(9, 6) NOT NULL,
    destination_longitude DECIMAL(9, 6) NOT NULL,
    ride_time INTEGER NOT NULL,
    fare_price DECIMAL(10, 2) NOT NULL CHECK (fare_price >= 0),
    payment_status VARCHAR(20) NOT NULL,
    driver_id INTEGER REFERENCES drivers(id),
    user_id VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Add driver records to the `drivers` table before testing the driver selection flow.

### 4. Start the app

```bash
npm start
```

Then use the Expo CLI shortcuts:

```bash
# Android
npm run android

# iOS
npm run ios

# Web
npm run web
```

Scan the QR code with Expo Go, or run the project on an emulator from the Expo developer menu.

## Available scripts

| Command | Description |
| --- | --- |
| `npm start` | Start the Expo development server |
| `npm run android` | Start Expo and open Android |
| `npm run ios` | Start Expo and open iOS |
| `npm run web` | Start the web target |
| `npm run lint` | Run Expo ESLint checks |
| `npm test` | Run Jest in watch mode |
| `npm run reset-project` | Run the bundled project reset script |

## Project structure

```text
app/
├── (auth)/                 # Welcome, sign-in, and sign-up screens
├── (root)/                 # Main ride-booking experience
│   ├── (tabs)/             # Home, rides, chat, and profile tabs
│   ├── book-ride.tsx       # Payment and booking confirmation
│   ├── confirm-ride.tsx    # Driver selection
│   └── find-ride.tsx       # Ride search and route selection
├── (api)/                  # Expo Router API handlers
│   ├── driver+api.ts       # Driver list
│   ├── user+api.ts         # User creation
│   ├── ride/               # Ride creation and history
│   └── (stripe)/           # Payment intent and payment confirmation
└── _layout.tsx             # Clerk provider and root navigation
components/                 # Reusable UI, map, input, and payment components
constants/                  # Images, icons, and onboarding content
lib/                        # API, auth, map, and utility helpers
store/                      # Zustand location and driver state
types/                      # Shared TypeScript types
assets/                     # Fonts, icons, and images
```

## API routes

The API handlers live under `app/(api)` and are called through Expo Router:

| Route | Method | Purpose |
| --- | --- | --- |
| `/(api)/user` | `POST` | Create a user linked to Clerk |
| `/(api)/driver` | `GET` | Load available drivers |
| `/(api)/ride/create` | `POST` | Save a booked ride |
| `/(api)/ride/[id]` | `GET` | Load a user's recent rides |
| `/(api)/(stripe)/create` | `POST` | Create a Stripe customer and payment intent |
| `/(api)/(stripe)/pay` | `POST` | Attach a payment method and confirm payment |

## Configuration notes

- The Clerk publishable key is validated when the root layout loads.
- Location access is requested on the home screen. Denying permission prevents the app from reading the device's current location.
- Google Places and Directions APIs must be enabled for the configured Google Cloud project.
- Stripe test keys should be used during development. The server secret belongs only in the environment used by the API handlers.
- The Expo URL scheme is `myapp`, and Stripe uses it for return/deep-link handling.

## Troubleshooting

### Environment variables are undefined

Confirm the names match the `.env` example exactly, then stop and restart the Expo server. Variables used by the client must use the `EXPO_PUBLIC_` prefix.

### The map or search does not load

Check that the relevant Google APIs are enabled, the API key has the correct restrictions, and location permissions have been granted on the device.

### Payments fail

Verify that the publishable and secret Stripe keys belong to the same mode (both test or both live), and confirm the server can reach Stripe and Neon.

### Database requests fail

Check `DATABASE_URL`, confirm the three tables exist, and ensure the `drivers` table contains at least one record.

## Security

- Keep `.env` out of version control.
- Do not put database credentials or `STRIPE_SECRET_KEY` in `EXPO_PUBLIC_` variables.
- Use test credentials and Stripe test cards during development.
- Apply production authentication, authorization, rate limiting, and server-side validation before deploying publicly.

## License

This project is for learning and demonstration purposes. Add a project-specific license before distributing it.
