# KhetMitra standalone API

Deploy this folder as a separate Node.js 20+ web service (Render, Railway, Fly.io, VPS, etc.). It is independent of the TanStack frontend.

## Run locally

```bash
cp .env.example .env
# Fill in SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY and optional data.gov.in key
npm start
```

The service listens on `PORT` (default 8080). `GET /health` is public. All `/api/*` routes require `Authorization: Bearer <Supabase access token>` and use the farmer's JWT against Supabase REST, so existing Row Level Security remains in force. No service-role key is required or exposed.

## Routes
- `POST /api/crop-advice` — saved farm profile + season-based crop ranking.
- `POST /api/crop-validation` — validates `{ "cropId": "tomato" }` against soil, season and rotation.
- `POST /api/crop-disease` — analyzes `{ "images": ["data:image/jpeg;base64,..."], "symptoms": [] }` with 1–3 photos and returns a cautious diagnosis and fertilizer guidance.
- `POST /api/mandi-prices` — `{ "district": "...", "state": "...", "crops": ["Tomato"] }`.
- `POST /api/fertilizer-plan` — crop-specific day-by-day schedule from the saved active crop and soil profile.

Set `ALLOWED_ORIGIN` to the exact deployed frontend origin(s), comma-separated, with no trailing slashes (for example, the production Vercel origin and a preview origin). Configure the data.gov.in key to enable live market quotes; without it the API explicitly returns unavailable prices rather than fabricated values.

## Crop disease analysis
Set `OPENAI_API_KEY` on the backend to enable `POST /api/crop-disease`. The authenticated route accepts `{ "images": ["data:image/jpeg;base64,..."], "symptoms": ["Yellowing"] }` with 1–3 photos (each up to 4 MB). It analyzes the photos together, checks that each shows a leaf, and returns a cautious classification, treatment and fertilizer guidance. If any photo is not clearly a leaf, the set is rejected with HTTP 422. `CROP_DISEASE_MODEL` optionally selects the vision model (default `gpt-4o-mini`). The model key stays on the backend and is never sent to the frontend. Fertilizer recommendations support crop nutrition; they are not presented as a cure for infectious disease. Multiple views can help, but image-based diagnosis cannot guarantee a correct result.

## Supabase schema
Uses existing `profiles` fields only: `id`, `farm_profile`, `onboarded`, `active_crop_id`, and `tasks`. No migration is included or required. Ensure the existing `profiles` RLS policy allows a signed-in user to select only their own row.
