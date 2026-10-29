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

- Presentation tier decision: Keep route-specific rendering in React components and send all HTTP requests through the shared API client. Components own loading, empty, and accessible error states; validation and domain calculations stay on the backend.
- Logic tier decision: Implement the `/api/*` contracts in Express routes and services. Validate request shape and referenced records before writes, return deliberate 4xx responses for invalid input or missing users, and avoid exposing Mongoose cast errors as 500 responses.
- Data tier decision: Define required fields, references, and suitable defaults in the Mongoose models. Keep seed data representative of all four capabilities and make repeated seeding safe.
- Activity tracking approach: Validate `userId`, `teamId`, activity type, and minutes; persist references and submitted values with server-computed points and `occurredAt`.
- Team management approach: Require a name and mascot, default members to an empty array, and preserve name, mascot, and member references in create and list responses.
- Leaderboard approach: Sum activity points per team, sort totals highest first, and assign ranks starting at 1.
- Workout suggestions approach: Find the user by email, use their fitness goal and recent activities to select and explain relevant suggestions, and return a deliberate 4xx response when no user matches.

## Test plan

Record the commands you run and what each failure means.

- Data tier, first: `npm run test:data --prefix octofit-tracker/backend`. Failures indicate model validation, reference/default, or seed behavior does not meet the data contract.
- Logic tier, after data tests pass: `npm run test:api --prefix octofit-tracker/backend`. Failures indicate endpoint behavior, input/error handling, points, leaderboard aggregation, or workout selection does not meet the API contract.
- Presentation tier, after API behavior is in place: `npm test --prefix octofit-tracker/frontend`. Failures indicate API integration, rendered data, loading/empty/error states, or accessibility behavior is incomplete.
- Backend compile check: `npm run build --prefix octofit-tracker/backend`. Failures indicate TypeScript errors in the API or data tier.
- Frontend compile/bundle check: `npm run build --prefix octofit-tracker/frontend`. Failures indicate frontend TypeScript or Vite build errors.

Baseline before implementation: `npm test --prefix octofit-tracker/backend` reports 8 failures and 1 pass (the idempotent seed test); `npm test --prefix octofit-tracker/frontend` reports 15 failures. Use the focused commands above to track progress by tier, then run both build checks before considering the implementation complete.
