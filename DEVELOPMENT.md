# Development — proxy-only Chrome reader 1.2

This plain Chrome Manifest V3 extension retains its existing player, paragraph extraction, pronunciation helpers, MP3 export, native side panel and floating controls. It no longer accepts or connects with end-user Gemini keys. The owner deploys `proxy/` to Cloudflare Workers Free, then runs `scripts/configure-service.mjs` with the public origin before distribution.

## Client

`extension/service_config.js` contains a public proxy origin only. `manifest.json` host and CSP connect permissions are generated for that origin alone. The unconfigured source has no network host permissions and connect-src none. `manifest.key` is a public identity key, not a provider credential or authentication secret; the unpacked ID is stable. No private key is bundled.

`worker.js` owns local settings, explicit versioned Cloudflare→Google consent, local attempt cap, anonymous session creation and trusted sender checks. Legacy `apiKey` values are removed from local and session storage when upgraded. Old provider consent does not transfer to proxy consent. Storage access levels remain TRUSTED_CONTEXTS. Session bearer is visible only in trusted extension contexts; content scripts cannot fetch it.

`proxy_client.js` validates the packaged origin and maps small sanitized server errors. `offscreen.js` requests a permit from the worker, sends the prepared text/style/requestId to /v1/tts with a short-lived session bearer, parses successful provider-format audio and plays WAV. No direct Google URL or Google key is present in the client's generation path. On 401 it invalidates the session and stops; a user's manual resume obtains a fresh session. There is no automatic generation retry.

`common.js` prepares spoken text without rewriting page words. Its model/voice allowlists contain only gemini-3.8-flash-tts and Charon. `mp3-export.js` uses a separately bundled unmodified lamejs script. Exports only use generated WAV already in the current task cache; full export rejects incomplete queues instead of inventing a full-length file.

## Public backend contract

- GET /health: fixed model/voice readiness, never keys or credential metadata.
- POST /v1/session: installation UUID. Returns {ok, token, expiresAt} in milliseconds. The extension obtains this automatically.
- POST /v1/tts: Bearer session and {text, style, requestId}. Returns successful Gemini Interactions audio JSON as a stream. The client controls neither upstream URL nor model/voice nor provider key.
- Failures: small sanitized {ok:false,error:{code,message}} with optional Retry-After.

Signed anonymous sessions are NOT proof of a real user and do not hide a bearer from an extension user. They protect token integrity/expiry; quotas bound to network IP and service/project budgets are the important abuse controls. Origin allowlisting is only an additional browser restriction. For a large public service add stronger user authorization, without embedding shared secret credentials in the extension.

## Backend and rotation

See proxy/README_AR.md and the backend source. Secrets contain Gemini keys tagged by real project, and SESSION_SECRET signs anonymous sessions. Durable state accounts for project-wide quotas and round-robin key selection. Sibling keys do not evade a project's 429. Attempts and fallback are bounded. No provider errors or sensitive data are returned or logged. Audio conversion stays client-side to avoid large JSON/audio transformations under the Worker Free CPU limit.

The owner's Google projects must be Free Tier without billing. Code cannot verify their billing state. Free-plan quotas of Google, Workers and Durable Objects are separate ceilings. This is not an unlimited or forever-free promise.

## Tests and delivery

Root npm ci && npm test covers the configured extension in Chromium using a local proxy fixture and generated test-tone WAV. No real Google synthesis, provider key, or charge is involved. The test copy permits localhost to simulate activeTab toolbar authorization; production permissions do not expand to arbitrary web pages.

Backend tests run from proxy/ with npm ci && npm test. They cover token validation, bounded rotation, quota grouping, durable counter behavior, fail-closed configuration, input validation and stream handling using test credentials only. A Worker dry-run is not a deployment; a health call is not proof of actual synthesis.

`node scripts/package-extension.mjs` refuses an unconfigured non-HTTPS service, packages extension and license notices only, and checks for accidental Google key patterns. Source archive delivery excludes .env, .dev.vars, secrets, node_modules and .wrangler. Actual deployment requires owner's authorized Cloudflare account and keys entered securely as secrets.
