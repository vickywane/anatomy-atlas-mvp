# Fieldnotes Anatomy Atlas

Fieldnotes is a responsive, guest-first anatomy learning prototype. It starts with a photo of an organism, identifies visible context, creates a 3D surface preview, and lets learners explore the generated model and its parts.

This is an early MVP. Generated geometry and AI labels are exploratory aids, not verified anatomy or medical advice. The built-in fly is an interface demo with illustrative, hand-built shapes and placeholder learning content.

## What works today

- **No account needed:** the current flow is available to guests. Accounts and saved study history are not implemented.
- **Photo identification and context:** an OpenAI Decisions request estimates the organism category and visible context, such as orientation, life stage, visibility, and obstructions. These are estimates from the submitted image.
- **Automatic 3D flow:** a usable photo proceeds to Tripo image-to-3D generation, then segmentation, then display of the whole model and its mesh parts.
- **Part exploration:** learners can rotate and zoom the model, select a mesh, show or hide parts, and view an explanation. The app renders the selected mesh by itself and highlighted in the whole model, then sends those views with the source photo to OpenAI for a tentative label and explanation.
- **Request feedback:** the interface shows detection, generation, segmentation, image preparation, and part-labeling progress. Part explanations are requested one at a time and cached in the browser session.
- **3D caching:** the whole model and segmented model are stored together in a local disk cache. The cache key includes the normalized image, detection context, Tripo model, and generation settings. Repeating the same inputs can reuse the result.
- **Quick demo:** a built-in fly exploration demonstrates the interface without requiring API keys. Its anatomy is illustrative and should not be used as reference material.

## Important limitations

- Tripo divides generated geometry into separate meshes; those meshes are not guaranteed to match real anatomical structures. OpenAI labels are visual suggestions and may be uncertain or wrong.
- A single photo cannot establish hidden anatomy, exact age, or reliable internal structure. The app should keep uncertainty visible and avoid presenting generated output as fact.
- The demo’s anatomy notes, practice content, and explanations are placeholders. They need to be replaced or reviewed by qualified educators.
- The 3D cache is local to one app filesystem. It may be lost on deployments with ephemeral storage and is not shared across multiple app instances.
- Automatic labeling sends one image-analysis request per segmented mesh. This is sequential to limit overlapping requests, but large models can take time and incur additional API usage.

## Run locally

Requirements: Node.js 20 or newer and pnpm.

1. Install dependencies with `pnpm install`.
2. Copy `.env.example` to `.env.local` and add the API keys you want to use.
3. Start the development server with `pnpm dev`.
4. Open `http://localhost:3000`.

The quick demo works without API keys. Photo identification and AI explanations use OpenAI; Tripo generation and segmentation require a Tripo API key.

Keep API keys server-side. Do not rename them with a `NEXT_PUBLIC_` prefix or commit `.env.local`.

### Environment variables

| Variable | Purpose |
| --- | --- |
| `OPENAI_API_KEY` | Enables photo identification, organism questions, and part explanations. |
| `OPENAI_DECISIONS_MODEL` | Optional model override for image identification and context. |
| `OPENAI_EXPLANATION_MODEL` | Optional model override for part labels and explanations. |
| `OPENAI_CHAT_MODEL` | Optional model override for organism questions. |
| `TRIPO_API_KEY` | Enables whole-model generation and segmentation. |
| `TRIPO_MODEL` | Optional Tripo generation model override. |
| `TRIPO_CACHE_DIR` | Optional persistent directory for cached GLB files and job records. Defaults to `.data/tripo-cache`. |

## Cache management

Clear all cached whole and segmented 3D models with:

```sh
pnpm clear:3d-cache
```

The command uses `TRIPO_CACHE_DIR` from the shell, `.env.local`, or `.env`, then falls back to `.data/tripo-cache`. Clearing the cache means a later upload may need to generate the model and segments again.

## How the request flow fits together

1. The browser sends the photo to `/api/identify` for organism and visible-context estimates.
2. If the photo is usable, `/api/generate-3d` checks the local cache and starts or reuses Tripo image-to-3D generation.
3. The job route polls Tripo, downloads and caches the whole GLB, then starts or resumes mesh segmentation.
4. Once the segmented GLB is ready and loaded, the browser reads mesh names and captures two views for each mesh: an isolated view and a whole-model view with the selected mesh highlighted.
5. The browser sends those renders and a resized source photo to `/api/explain`. Requests run serially, and returned explanations are cached for the current browser session.

## Good next improvements

- Replace placeholder demo content with reviewed, licensed, segmented models and cited learning material.
- Evaluate generated labels with biology and medical learners; show clear fallbacks when a mesh cannot be identified.
- Add durable shared object storage and background job handling before deploying to multiple or ephemeral servers.
- Add upload and API rate limits, abuse protection, and a clear image-retention policy before public launch.
- Improve accessibility and usability on touch screens, keyboards, and smaller displays.
- Add optional accounts, saved sessions, and progress tracking after the guest learning flow is validated.
- Explore a curated comparison mode for two organisms after reviewed model assets and defensible structure mappings are available.

## Tech stack

- Next.js App Router and React
- Three.js, React Three Fiber, and Drei for interactive 3D
- OpenAI Responses and Decisions APIs
- Tripo image-to-3D and mesh segmentation APIs
- Local filesystem cache for generated GLB models and job metadata
