# opendish

**opendish** is an open-source food image beautification tool built with a zero-cost hobby stack. It serves as an open-source alternative to tools like Beauplat, focused on high-quality food image enhancement using AI.

## 🌟 Core Features

-   **AI-Powered Enhancement**: Uses **Nano Banana 2** (`gemini-3.1-flash-image-preview`) for professional-grade image editing.
-   **Style Selection**: Choose from predefined styles:
    -   **Gourmet**: Professional studio photography, soft lighting.
    -   **Vibrant**: High-contrast, bright natural sunlight.
    -   **Rustic**: Warm tones, wooden backgrounds.
    -   **Minimalist**: Clean white background, top-down view.
    -   **Neon Night**: Cyberpunk aesthetic, dramatic lighting.
-   **Privacy First**: All images are strictly private to the user account, enforced via Supabase RLS.
-   **Global Rate Limiting**: Managed usage (25 images/day total) to maintain zero-cost operations.

## 🛠️ The Zero-Cost Stack

| Component | Provider | Selection Logic |
| :--- | :--- | :--- |
| **Frontend/Hosting** | Vercel | Free hobby tier, Next.js App Router. |
| **Backend (API)** | Vercel Functions | Serverless functions in `/api` directory. |
| **Database** | Supabase | Free Postgres with RLS. |
| **Storage** | Supabase Storage | Persistent storage for images. |
| **Async Worker** | Upstash QStash | Orchestrates Gemini calls to prevent serverless timeouts. |
| **AI Engine** | Gemini API | Nano Banana 2 for image editing. |

## 🚀 Getting Started

This project uses a monorepo architecture. Detailed implementation guidelines and strategy can be found in [agents.md](./agents.md).
