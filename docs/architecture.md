# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision:
- Logic tier decision:
- Data tier decision:
- Activity tracking approach:
- Team management approach:
- Leaderboard approach:
- Workout suggestions approach:

## Current status summary

### Tier boundaries

- Presentation tier: React 19, Vite, and the shared API client in `octofit-tracker/frontend/src`; this layer should render activity data, team data, leaderboard rankings, and workout suggestions while showing loading, empty, and failure states.
- Logic tier: Express + TypeScript routes and services in `octofit-tracker/backend/src`; this layer owns validation, 4xx error handling, data shaping, leaderboard calculations, and workout recommendation logic.
- Data tier: Mongoose models and seed logic in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts`; this layer owns schema contracts, seeded fixtures, and MongoDB persistence rules.

### Current failing tests

The current implementation is intentionally incomplete, and the failing tests cluster around the contract gaps rather than UI polish:

- Data contract tests are failing because the models are not enforced correctly and the seed process is not idempotent enough for the expected data fixtures.
- API contract tests are failing because `createActivity`, `createTeam`, `getLeaderboard`, and `suggestWorkouts` in `octofit-tracker/backend/src/services/octofitService.ts` return stubs or 501 responses instead of real validation and business logic.
- Frontend contract tests are failing because the React app still renders placeholder text and never fetches or renders the real API contract data in `octofit-tracker/frontend/src/App.tsx`.

### Smallest safe implementation order

1. Data tier first: complete the schema contracts and seed behavior so MongoDB-backed fixtures match the expected User, Team, Activity, and Workout models.
2. Logic tier second: implement validation and business logic in the backend service layer and routes so the API returns correct 2xx/4xx responses, computed activity points, ordered leaderboard totals, and workout suggestions by user pattern.
3. Presentation tier last: implement the UI fetch and render flow to surface the API data and the required loading, empty, and accessible error states once the backend contract is stable.

This ordering is the lowest-risk path because each tier depends on the one below it: the frontend is only reliable once the data and API contracts are correct.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
- API command: `npm run test:api --prefix octofit-tracker/backend`
- Frontend command: `npm test --prefix octofit-tracker/frontend`
- Backend build command: `npm run build --prefix octofit-tracker/backend`
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
