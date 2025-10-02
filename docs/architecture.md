# ScripturePath Application Architecture

## Overview
ScripturePath is a production-ready Bible study platform that leverages AI services to generate personalized study content, facilitate collaborative study sessions, host quizzes, and support sermon preparation. The frontend is built with **React**, **TypeScript**, **Tailwind CSS**, and **Radix UI** components, while **Supabase** powers authentication, database storage, edge functions, and realtime collaboration.

## Core Application Purpose
- Deliver AI-generated Bible study content tailored to individual needs.
- Facilitate group study sessions with realtime collaboration and shared notes.
- Provide sermon preparation tools for ministry leaders.
- Offer AI-generated quizzes for personal learning or group events.
- Support monetization via subscriptions, lifetime purchases, and physical product sales.

## Technology Stack
- **Frontend**: React 18, TypeScript, Vite, Tailwind CSS, Radix UI, React Router DOM, React Query, React Hook Form, Zod, Sonner, Embla Carousel, Recharts, Next Themes.
- **Backend / BaaS**: Supabase (PostgreSQL, Auth, Edge Functions, Storage, Realtime).
- **AI Providers**: Lovable AI gateway (Google Gemini 2.5 family, OpenAI GPT-5 family).
- **Payments**: Stripe (subscriptions, one-time lifetime purchases).
- **Hosting**: Lovable Cloud staging for frontend, Supabase managed infrastructure for backend services.

## Application Routes
- `/` — Landing page with mode selection cards.
- `/auth` — Authentication (sign in/sign up) with Supabase Auth email/password.
- `/account` — Profile overview, subscription management, usage tracking.
- `/shop` — Subscription plans (Free, Plus, Lifetime) and physical product listings.
- `/alone` — Personal study generation workflow.
- `/preach` — Sermon outline builder.
- `/group` — Group study session generator.
- `/session/:id` — Realtime group study room with collaborative tools.
- `/quiz` — Quiz dashboard with featured quizzes.
- `/host-quiz` — Host live multiplayer quiz sessions.
- `/join-quiz/:code` — Join an active quiz session via invite code.
- `/study-review/:id` — Review AI-generated study before launching.
- `/study/:id` — Saved study viewer with export options.
- `/session-library` — Browse previously hosted sessions.
- `/bible` — Scripture reader with bookmarks and highlights.
- `/history` — Personal study history and statistics.
- `/templates` — Library of pre-made study templates.
- `/reset-password` — Password reset flow.

## Authentication & User Profile
- Supabase Auth handles email/password login with auto-confirmed emails and localStorage session persistence.
- Protected routes guard core functionality for authenticated users.
- `profiles` table stores subscription tier, usage counters, streaks, milestones, storage usage, and personalization preferences (translation, tone).

### Profile Schema
```ts
{
  id: string;
  email: string;
  name: string;
  subscription_tier: 'free' | 'plus' | 'lifetime';
  subscription_status: string;
  study_generations_this_month: number;
  study_generations_reset_date: string;
  bonus_credits_this_month: number;
  referral_code: string;
  referral_count: number;
  current_streak: number;
  longest_streak: number;
  total_studies_completed: number;
  current_milestone: string;
  milestone_progress: number;
  storage_bytes_used: number;
  preferred_translation: string;
  preferred_tone: 'expository' | 'narrative' | 'topical';
}
```

## Subscription Tiers & Monetization
- **Free**: 4 AI studies/month, bonus credits via referrals, markdown export, last 5 studies saved, 200MB storage, text-only group hosting.
- **Plus ($9.99/mo with 14-day trial)**: Unlimited studies, PDF/PPT exports, unlimited library, audio/video group sessions, 10GB storage, priority support.
- **Lifetime ($399 one-time)**: All Plus features without renewal, mission support messaging.

### Stripe Edge Functions
- `create-checkout` — Subscription checkout session creation.
- `create-payment` — Lifetime plan payment.
- `check-subscription` — Verify current subscription status.
- `customer-portal` — Access Stripe billing portal.
- `manage-subscription` — Cancel or modify subscriptions.

### Referral Program
- Unique referral codes per user stored in `profiles` and tracked in `referrals` table.
- Bonus credits awarded when referrals subscribe; 10 referrals grant a free month and 20% shop discount.

## AI Edge Functions (Supabase Deno Functions)
1. `generate-study`: Generates study material for alone, preach, group, or quiz contexts. Enforces usage limits (30-day rolling reset) and handles 402/429 errors gracefully.
2. `generate-quiz`: Produces AI quizzes from topics, passages, or saved studies.
3. `refine-study`: Applies requested refinements to existing studies.
4. `generate-daily-quizzes`: Scheduled function that creates featured quizzes (no JWT verification).
5. `fetch-bible-passage`: Public function fetching scripture text from external APIs.

## Database Schema Highlights
- `studies`: Stores AI-generated study content and metadata.
- `group_sessions`: Tracks group session state, agenda, notes, roles, and whether saved to library.
- `session_participants`: Participant roster with presence data.
- `featured_quizzes`: Daily featured quizzes.
- `quiz_sessions`: Live quiz hosting metadata.
- `user_quiz_responses`: Stored quiz attempts and scores.
- `session_library`: Archived session details.
- `bookmarks` & `highlights`: Scripture reader personalization.
- `user_usage_logs`: Usage tracking for rate limiting and analytics.

## Realtime Collaboration
- Supabase realtime channels power group sessions via `useRealtimeSession.tsx`, enabling participant presence, note synchronization, and section navigation.
- Quiz sessions use `useRealtimeQuizSession.tsx` for live participant management, answer submissions, and leaderboards.

## Export System
- `ExportDialog.tsx` manages export options.
- Markdown export available to all users.
- Plus tier unlocks HTML (via `generate-pdf` edge function) and PowerPoint (via `generate-powerpoint` edge function) exports.
- Frontend converts returned slide JSON into downloadable PPTX files.

## Gamification & Progress Tracking
- Milestones (new_believer → theologian → expert) tracked via `MilestoneCard` component and `useMilestoneCelebration` hook.
- Quiz streaks maintained through `update_user_quiz_streak()` database function triggered after quiz completion.
- Additional stats include total studies, accuracy, difficulty distribution, and session duration.

## Shop & Physical Products
- Offers Cross Necklace ($24.99), Cross Bracelet ($19.99), Study Bible ($39.99).
- Referral discounts apply automatically.
- Implements demo cart without full e-commerce fulfillment.
- Product assets stored under `src/assets/`.

## Security & RLS
- Row-level security ensures users can only access their own studies, sessions, and profile data unless resources are public or they participate in the session.
- `profiles` table restricted to owner access.
- `studies` table allows public visibility for shared content while preserving ownership for modifications.

## Usage Tracking & Limits
- `user_usage_logs` records resource interactions.
- Monthly study counters reset via `reset_monthly_study_counter()` (called by `generate-study`).
- Free tier limited to 4 studies/month; Plus tier unlimited.

## UI/UX Enhancements
- Theme support via `next-themes` with custom gradients (`bg-gradient-contemplative`, `bg-gradient-divine`, etc.).
- `BottomNav` component provides mobile-first navigation.
- `StudyCardSkeleton` delivers shimmer loading states.
- `ErrorBoundary` handles runtime errors with fallback UI.
- `useOfflineDetection` disables saving while offline and alerts users.

## Environment Variables
- `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_PROJECT_ID` for frontend Supabase access.
- Supabase secrets include Lovable AI key, Stripe secret key, Supabase service role key, and anon key for edge functions.

## Deployment & Infrastructure
- Frontend built with Vite and deployed to Lovable Cloud staging subdomain.
- Supabase hosts PostgreSQL, Auth, Functions, Realtime, and Storage.
- Configuration files: `vite.config.ts`, `tailwind.config.ts`, `tsconfig.json`, `supabase/config.toml`.

## Development Utilities
- React Query for data fetching and caching.
- React Hook Form + Zod for forms.
- Sonner for toast notifications.
- date-fns for formatting.

## User Journey Summary
1. New users arrive at landing page and are prompted to authenticate.
2. After signup, access to study modes unlocks; first study generation decrements free credits.
3. Users configure AI prompts by selecting modes, topics, passages, and preferences.
4. Generated studies can be reviewed, saved, exported, and used to launch group sessions.
5. Group sessions support realtime collaboration and can be archived.
6. Quizzes can be taken individually, generated by AI, or hosted live with leaderboards.
7. Gamification elements track milestones, streaks, and usage history.

## Future Enhancements (Ideas)
- Mobile-native apps using the same Supabase backend.
- Enhanced analytics dashboards for pastors and group leaders.
- Community marketplace for sharing public study templates.
- Offline-capable PWA for scripture reading and study review.

