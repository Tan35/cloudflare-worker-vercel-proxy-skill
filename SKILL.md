---
name: cloudflare-worker-vercel-proxy
description: Create an isolated Vercel external-rewrite proxy in front of a Cloudflare Workers app, including a custom-domain-ready setup, NDJSON/SSE streaming validation, Git identity checks, and a clean rollback. Use when a user asks to serve a Cloudflare Worker or workers.dev URL through Vercel or a Vercel custom domain without modifying the original Worker.
---

# Cloudflare Worker → Vercel Proxy

Use this Skill to put a **separate Vercel project** in front of a Cloudflare Worker while keeping the Worker, its source repository, and existing Vercel projects untouched.

## Required inputs

Obtain the Worker origin URL, for example:

```text
https://worker-name.account-subdomain.workers.dev
```

Confirm whether the app has streaming endpoints such as NDJSON, SSE, model output, uploads, or auth cookies. If the user references an existing proxy project, inspect its deployed `vercel.json` or Vercel Source view before inferring a new configuration.

## Isolation rules

1. Create a **new private GitHub repository** and a **new Vercel project**. Do not add Vercel configuration to the Worker’s source repository unless the user explicitly requests that coupling.
2. Do not alter the Worker deployment, Worker secrets, Cloudflare DNS, or an existing proxy project during proxy setup.
3. Put no API key or Worker secret in the proxy repository. A transparent rewrite needs none.
4. Before deploying, ensure the repository’s Git author email is linked to the user’s GitHub account. Check with:

```bash
git config user.email
git log -1 --format='%an <%ae>'
```

If Vercel blocks a deployment because the commit email is unmatched, set the local identity and rewrite only the new proxy repository’s history before force-pushing:

```bash
git config user.name "<GitHub name>"
git config user.email "<GitHub-linked email>"
GIT_SEQUENCE_EDITOR=: git rebase --root --exec 'git commit --amend --no-edit --reset-author'
git push --force-with-lease origin main
```

## Minimal proxy configuration

Create `package.json` only as a minimal deployment manifest and create `vercel.json` with the following shape. Use the **regular-expression source** `/(.*)` and capture replacement `$1`; it covers the root path and nested paths consistently.

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "https://<worker-origin>/$1"
    }
  ],
  "headers": [
    {
      "source": "/api/:path*",
      "headers": [
        {
          "key": "x-vercel-enable-rewrite-caching",
          "value": "0"
        }
      ]
    }
  ]
}
```

Replace `<worker-origin>` with the hostname only, without a trailing slash. For example, the destination should be:

```text
https://pi-web-agent-workers.1486973169tan.workers.dev/$1
```

> Do **not** add `public/index.html` or another static homepage merely to make Vercel deploy. A static root file can win routing precedence and hide the Worker homepage. Do **not** replace the rewrite with a Vercel Function that reads and rebuilds the upstream response body; that can buffer or break streaming.

The API cache opt-out is required for dynamic endpoints. Do not add CDN cache headers to `/api/*`; preserve the origin’s headers, especially `Cache-Control: no-cache, no-transform` on streaming endpoints.

## Deployment workflow

1. Commit and push the configuration to the isolated private repository.
2. Import that repository as a new Vercel project. Select the generic/Other preset; no build command, environment variable, or function is required for a pure rewrite project.
3. Wait for the production deployment to be Ready. Treat any build, author-email, or root-path 404 as a blocker; do not proceed to a custom domain yet.
4. If the user wants a custom domain, add it only after the default `.vercel.app` domain passes the complete validation suite below.

## Validation suite

Run no-cost checks against both the Vercel proxy and the Workers.dev origin. Require equal status codes and equal bodies for:

| Check | Expected result |
|---|---|
| `GET /` | Worker homepage renders through Vercel; browser title and core controls load. SSR HTML may differ byte-for-byte because the request host differs, so use visual and functional comparison instead of a strict homepage hash. |
| `GET /favicon.svg` | `200` and matching asset bytes. |
| `GET /api/models` | Same `200` and matching JSON. |
| Invalid `POST /api/chat` | Same validation `400` response and JSON body. |
| Empty prompt `POST /api/chat` | Same validation `400` response and JSON body. |

For an application with streaming model output, ask the user for confirmation before a real model invocation. Then use the proxy URL to send a minimal prompt and verify all of the following:

1. The first plan, text, or tool event appears before the response completes.
2. The Network response has the expected streaming content type, such as `application/x-ndjson` or `text/event-stream`.
3. Incremental content continues to render and a terminal event such as `done` arrives.
4. Cancelling a test run stops new visible output promptly.

Do not claim that “everything works” until this real streaming check is complete. A user-performed streaming test that they explicitly report as successful counts as acceptance evidence; record it without exposing any prompts, secrets, or model output.

## Custom domain and rollback

Attach the user’s custom domain to the **new Vercel project only** after passing validation. Verify both the default Vercel hostname and custom hostname before changing links or retiring an old entry point.

Rollback is isolated: remove the custom domain from the new project or delete that Vercel project. The Cloudflare Worker and its Workers.dev URL must continue working unchanged.
