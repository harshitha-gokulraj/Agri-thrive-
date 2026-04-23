# AgriThrive

AgriThrive is a web platform for precision agriculture decision support, focused on Indian farming contexts.
It combines a public product website, a multi-widget analytics dashboard, multilingual UX, and optional OTP-based authentication through Supabase.

This repository contains the full frontend application built with React, TypeScript, Vite, Tailwind CSS, and shadcn/ui components.

## Table of Contents

1. [Overview](#overview)
2. [Core Capabilities](#core-capabilities)
3. [How the System Works](#how-the-system-works)
4. [Architecture](#architecture)
5. [Technology Stack](#technology-stack)
6. [Project Structure](#project-structure)
7. [Getting Started](#getting-started)
8. [Environment Variables](#environment-variables)
9. [Authentication and Profile Flow](#authentication-and-profile-flow)
10. [Data and Analytics Model](#data-and-analytics-model)
11. [Available Scripts](#available-scripts)
12. [Testing](#testing)
13. [Build and Deployment](#build-and-deployment)
14. [Known Behavior and Limitations](#known-behavior-and-limitations)
15. [Future Enhancements](#future-enhancements)

## Overview

AgriThrive provides two primary experiences:

- Public landing page with feature and architecture sections.
- Interactive dashboard for plot-level agronomic monitoring and decision support.

The dashboard aggregates soil, weather, NDVI, crop, and economic indicators into a unified workflow for recommendation, monitoring, and explainable decision logs.

## Core Capabilities

- Multi-plot farm management with add, select, delete, and refresh actions.
- Crop recommendation engine using soil, climate, season, and region scoring.
- Soil analytics with nutrient status, pH interpretation, and radar visualization.
- Live sensor simulation with rolling readings and trend indicators.
- Satellite-style NDVI tracking and crop health signaling.
- Yield prediction views with historical and projected patterns.
- Alerting and notification logic for heat, moisture, disease risk, and market context.
- Explainability through decision audit trail entries tied to input signals.
- Dashboard widget customization with local persistence.
- OTP authentication and profile management via Supabase.
- Multilingual interface (English, Hindi, Tamil, Telugu, Marathi, Punjabi, Bengali, Rajasthani).
- Light and dark theme support.

## How the System Works

### 1) Application bootstrap

- The app mounts from `src/main.tsx`.
- `src/App.tsx` composes the provider hierarchy:
	- `ThemeProvider`
	- `LanguageProvider`
	- `AuthProvider`
	- `QueryClientProvider`
	- `TooltipProvider`
- Routes are registered for:
	- `/` (landing page)
	- `/auth` (OTP authentication)
	- `/dashboard` (analytics workspace)
	- `*` (404 page)

### 2) State and context layers

- `AuthContext` manages session, user, profile fetch/update, OTP sign-in, OTP verification, and sign-out.
- `LanguageContext` provides translation lookup and stores selected language in localStorage.
- `FarmContext` holds plot data, selected plot, simulated refresh cycles, and localStorage persistence.

### 3) Dashboard data lifecycle

- Initial plot dataset is loaded from hardcoded defaults in `FarmContext`.
- User plot changes are persisted under `agrithrive_plots` in localStorage.
- A timed refresh cycle (every 30 seconds) perturbs weather, moisture, and NDVI values to emulate streaming updates.
- Widget visibility preferences are saved under `agrithrive_widgets`.

### 4) Recommendation and scoring flow

- Crop recommendation uses rules from `src/data/indianAgriculture.ts`.
- For each candidate crop, a normalized score is computed from:
	- Soil type match
	- pH range fit
	- Temperature fit
	- Nitrogen fit
	- Season match
	- State match
- Top-ranked crops are displayed with reasons, MSP references, and yield benchmarks.

### 5) Alerting and explainability flow

- Alert module generates contextual alerts from selected plot conditions:
	- High temperature
	- Low soil moisture
	- High humidity disease risk
	- NDVI stress or strong growth signals
	- Market reference updates
- Decision audit trail summarizes model outputs and reasoning text with traceable input signals.

### 6) Authentication and profile flow

- OTP is sent using Supabase Auth (`signInWithOtp`).
- OTP is verified using Supabase (`verifyOtp` with sms type).
- Profile data is stored in `public.profiles` and is user-scoped by RLS policies.
- Profile editing is available from the user dialog after login.

## Architecture

AgriThrive follows a layered architecture:

- Client Interface Layer
	- React components, route views, responsive UI, accessibility-friendly controls.
- Analytics Processing Layer
	- Rule-based scoring, threshold logic, simulated telemetry updates, explainability rendering.
- Backend Services Layer
	- Supabase authentication, profile persistence, RLS-protected records.
- External Integration Layer
	- Designed for weather, satellite, soil, and market APIs (UI supports this model; many values are currently simulated in-app).

## Technology Stack

- Runtime: React 18, TypeScript, Vite 5
- Styling: Tailwind CSS, shadcn/ui, Radix UI primitives
- State and data: React Context, React Query
- Charts and visualization: Recharts
- Motion and transitions: Framer Motion
- Authentication and backend: Supabase
- Testing: Vitest, Testing Library, Playwright config
- Linting: ESLint with TypeScript ESLint and React Hooks plugin

## Project Structure

```text
Agri-thrive/
	src/
		components/
			dashboard/                 # Dashboard widgets and tools
			ui/                        # Reusable UI primitives
		contexts/                    # Auth, language, farm contexts
		data/                        # Indian agriculture rules and reference datasets
		integrations/supabase/       # Supabase client and generated DB types
		pages/                       # Route-level pages (Index, Auth, Dashboard, NotFound)
		test/                        # Unit test setup and examples
	supabase/
		migrations/                  # SQL migrations for profiles and policies
```

## Getting Started

### Prerequisites

- Node.js 18+
- npm 9+

### Installation

Run commands from the project directory that contains `package.json`:

```bash
cd "/Users/gayathridevi/github repos/agri thrive/Agri-thrive"
npm install
```

### Start Development Server

```bash
npm run dev
```

The Vite dev server runs on port `8080` by default.

## Environment Variables

Create a `.env` file in the project root and set:

```env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_PUBLISHABLE_KEY=your_supabase_anon_or_publishable_key
```

Without these variables, OTP auth and profile operations will not function.

## Authentication and Profile Flow

- Authentication is phone OTP based.
- New authenticated users trigger automatic profile creation in Supabase.
- Profile fields include:
	- full_name
	- phone
	- state
	- district
	- preferred_language
	- farm_size_acres
- Profile reads and updates are restricted via row-level security.

## Data and Analytics Model

### Domain dataset

`src/data/indianAgriculture.ts` provides:

- Supported states, seasons, crops, and soil categories.
- MSP reference values and average yield references.
- Crop recommendation rules by soil type, pH, temperature, rainfall, season, state, and nitrogen range.
- Government scheme metadata.
- Pest and advisory datasets.

### Recommendation scoring

Scoring uses weighted matching and normalization to a 0-100 percentage.

### Fusion notation in UI

The dashboard uses a conceptual fused feature vector:

`z_{p,t} = phi(x_st, x_w, x_s, x_h, x_m)`

where the terms represent satellite, weather, soil, historical, and market features.

## Available Scripts

- `npm run dev` - Start local development server.
- `npm run build` - Build production bundle.
- `npm run build:dev` - Build in development mode.
- `npm run preview` - Preview built app.
- `npm run lint` - Run ESLint.
- `npm run test` - Run Vitest once.
- `npm run test:watch` - Run Vitest in watch mode.

## Testing

### Unit tests (Vitest)

```bash
npm run test
```

### Watch mode

```bash
npm run test:watch
```

Playwright configuration exists and can be extended for end-to-end tests.

## Build and Deployment

Build for production:

```bash
npm run build
```

Preview production build:

```bash
npm run preview
```

Deploy the generated `dist/` output to any static hosting provider.

## Known Behavior and Limitations

- Several dashboard streams are simulated (sensor and periodic refresh behavior).
- Many analytical outputs are deterministic or pseudo-randomized for demonstration.
- Route-level access guard for `/dashboard` is not currently enforced.
- Login button is intentionally removed from navbar UI, but `/auth` route remains available.

## Future Enhancements

- Integrate live external APIs for weather, satellite, and mandi price feeds.
- Add role-based access control and route guards.
- Persist plot and widget data to backend storage for multi-device continuity.
- Expand test coverage across business logic and UI flows.
- Add CI pipelines for lint, test, and build verification.
