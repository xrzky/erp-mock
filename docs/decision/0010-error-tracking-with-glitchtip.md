# 0010: Browser error tracking with GlitchTip (Sentry SDK)

- Status: Accepted
- Date: 2026-11-22

## Context

Errors in the browser (render crashes, failing loaders, click handlers, unhandled rejections) were only visible in the user's console. GlitchTip accepts events from the Sentry SDK, so we use it as the error tracker. It is open source and can be hosted (app.glitchtip.com) or self-hosted.

## Decision

- Use `@sentry/react`, initialised in `src/lib/sentry.ts` and called from `getRouter()` in `src/router.tsx` (TanStack Start has no client entry file we own).
- The DSN comes from `VITE_SENTRY_DSN` at build time. With no DSN, Sentry stays off, so e2e and any run without it send nothing. `VITE_SENTRY_ENV` names the environment; it falls back to the Vite mode.
- The router's `defaultOnCatch` reports errors that its error screen catches (render and loader errors). Sentry's global handlers cover the rest.
- `tracesSampleRate` is 0.01 and the session integration is removed, because GlitchTip does not support sessions.

## Consequences

- For local development put the DSN in `.env.local` (git-ignored). Deploy scripts must export `VITE_SENTRY_DSN` before `vite build`, because Vite bakes it into the browser bundle.
- The DSN is a public client key, but keep it out of the repo anyway so each developer reports to their own project.
- The hosted free plan allows 1,000 events a month. Do not loop error buttons.
- Server-side errors (server functions) are not reported yet. Add `@sentry/node` later if we want them.
- Source maps are not uploaded, so production stack traces point at minified code.
