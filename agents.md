# opendish - Project Strategy & AI Agent Instructions (agents.md)

This document outlines the architecture, implementation guidelines, and specific instructions for building **opendish**, an open-source food image beautification tool using a zero-cost hobby stack.

## 1. Project Vision & Core Features
- **Objective**: Build an open-source alternative to Beauplat (https://www.beauplat.com/) for food image enhancement.
- **Monorepo**: Workspace-based architecture (e.g., `apps/web`, `packages/config`).
- **Image Processing**:
    - Model: **Nano Banana 2** (`gemini-3.1-flash-image-preview`).
    - Format: Strictly **JPEG** for both upload and download.
    - Style Selection: Users choose from predefined styles (mapped to static prompts):
        - **Gourmet**: Professional studio photography, soft lighting, fine dining vibe.
        - **Vibrant**: High-contrast, bright natural sunlight, energetic look.
        - **Rustic**: Warm tones, wooden backgrounds, cozy home-cooked feel.
        - **Minimalist**: Clean white background, top-down view, focus on texture.
        - **Neon Night**: Cyberpunk aesthetic, dramatic blue/pink lighting, urban vibe.
- **Privacy & Storage**:
    - All images are strictly **private** to the user account.
    - Enforced via Supabase RLS and private buckets.
- **Global Rate Limit**: 25 images per day total across **all** users (tracked in Supabase). No refunds on failure.
- **Authentication**: Email/Password with 15-day JWT expiry. *Reasoning: High convenience for a creative tool where 15 days is an acceptable risk-to-friction trade-off.*

## 2. Core Architecture (The Zero-Cost Stack)

| Component | Provider | Selection Logic |
| :--- | :--- | :--- |
| **Frontend/Hosting** | Vercel | Free hobby tier, managed SSL, fast deployment, Next.js App Router. |
| **Backend (API)** | Vercel Functions | Serverless functions in `/api` directory. Zero cost when not running. |
| **Database** | Supabase | Free Postgres. Stores users, tasks, and global daily usage counter. |
| **Storage** | Supabase Storage | Persistent storage for original and beautified images. |
| **Async Worker** | Upstash QStash | Orchestrates long-running Gemini calls to prevent serverless timeouts. |
| **AI Engine** | Gemini API | Using **Nano Banana 2** (`gemini-3.1-flash-image-preview`) for image editing. |

## 2. The Async Implementation Pattern (Webhook Trigger)

To handle Gemini API latency without hitting Vercel's free-tier timeout limits, use the following pattern:

1.  **Request/Start**: Frontend sends data to `/api/trigger` (or `/api/start`).
2.  **Queue**: `/api/trigger` sends a message to **Upstash QStash** with a destination URL of `/api/worker` (the webhook).
3.  **Process**: QStash invokes the `/api/worker` endpoint. This function performs the long-running call to the **Gemini API**.
4.  **Storage**: The worker saves the result directly into the **Supabase** database.
5.  **UI Update**: The frontend listens to the Supabase table for changes (Realtime) or polls for the result.

## 4. Developer Instructions for AI (Cline / Gemini)

When generating code or managing the project, strictly adhere to these rules:

-   **Environment Variables**: Access `GEMINI_API_KEY`, `SUPABASE_SERVICE_ROLE`, `QSTASH_TOKEN`, `JWT_SECRET` via `process.env`. Never expose these to the client.
-   **Security**: 
    -   Implement Upstash signature verification in `/api/worker`.
    -   Store hashed/salted passwords (bcrypt) in Supabase.
    -   Use 15-day JWTs for API authentication.
-   **Database Schema**: 
    -   Use the `supabase-js` client. 
    -   `tasks` table: `id`, `user_id`, `status`, `original_path`, `result_path`, `style`, `prompt`, and timestamps.
    -   `daily_usage` table: `id`, `count`, `date` (to track the global 25-image limit).
-   **Storage Strategy**: Upload images directly from the **client** to Supabase Storage (using RLS) to avoid Vercel serverless function payload/timeout limits.
-   **Realtime**: Use Supabase Realtime to push task status updates to the frontend.
-   **Connectivity**: Rely on Vercel's managed certificates for secure communication.

## 4. Maintenance & Optimization (Pro-Tips)

-   **Supabase Persistence**: Free databases pause after 7 days of inactivity. 
    -   *Action*: Set up a simple cron job, GitHub Action, or manual ping (e.g., a simple `SELECT` query) once a week to keep the instance active.
-   **Cold Starts**: Expect a 2-3 second delay on initial serverless requests. Design UI with appropriate loading states (spinners/skeletons).
-   **Rate Limiting**: Implement basic IP-based rate limiting in serverless functions to protect Gemini credits from bots.
-   **AI Context**: Always provide this `agents.md` file to the AI agent to ensure it follows the Upstash + Vercel pattern instead of trying to build a traditional long-running Express server.
