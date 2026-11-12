# Hosted in-app notification feed vendors: findings

Research for issue #2 (part of map #1). Facts only; no vendor is chosen here.

- Researched: 2026-11-12. Vendor pricing, limits and docs change; re-check before any decision.
- Method: official vendor docs, pricing pages, vendor GitHub repos and the npm registry. Pages were read through an automated fetch tool that summarises each page, so figures are as reported by that tool and not re-read by hand. Where a page did not state something, or sources disagreed, it is marked **Unverified** or **Conflict**.
- Several vendor doc pages (Novu, Courier, SuprSend) contained blocks of text addressed to AI agents (install a skill, fetch an index). These were treated as page content and ignored.
- No vendor prices, limits or commitments here are confirmed with the vendor. Each needs confirmation with the vendor's sales or support before any commitment.

## Shortlist and maintenance check

Candidates: Knock, Novu, Courier, MagicBell, SuprSend. Other vendors were not assessed (scope was limited to these five).

Maintenance signals (vendor GitHub repos and npm, checked 2026-11-12):

| Vendor | Evidence of recent activity | Source |
| --- | --- | --- |
| Knock | `knocklabs/javascript` pushed 2026-11-12 (MIT); `@knocklabs/react` 0.15.1 modified 2026-11-06; `@knocklabs/node` 1.41.1 modified 2026-11-10 | https://github.com/knocklabs/javascript, https://www.npmjs.com/package/@knocklabs/react, https://www.npmjs.com/package/@knocklabs/node |
| Novu | `novuhq/novu` pushed 2026-11-12 (about 40k stars; license reported as NOASSERTION by GitHub); `@novu/react` 3.19.2 (2026-09) | https://github.com/novuhq/novu, https://www.npmjs.com/package/@novu/react |
| Courier | `trycourier/courier-web` pushed 2026-11-09 (MIT); `@trycourier/courier-react` 9.3.5 (2026-11-09); `@trycourier/courier` 10.0.0 (2026-10-29) | https://github.com/trycourier/courier-web, https://www.npmjs.com/package/@trycourier/courier-react |
| MagicBell | `magicbell/magicbell-js` pushed 2026-11-10 (latest commit seen 2026-08-03, latest release 1.8.0 on 2026-07-29); `@magicbell/react` 1.14.0 (2026-07-29) | https://github.com/magicbell/magicbell-js, https://www.npmjs.com/package/@magicbell/react |
| SuprSend | `@suprsend/web-sdk` 5.4.0 (2026-10-22); `@suprsend/react` 1.5.0 (2026-10-22); `@suprsend/node-sdk` 1.15.1 (2026-05-19). The older `@suprsend/react-inbox` repo is **archived** (last release 2025-04) and docs say it is end of support, replaced by `@suprsend/react` | https://github.com/suprsend/suprsend-web-sdk, https://github.com/suprsend/suprsend-react-inbox, https://docs.suprsend.com/docs/inbox-react |

No candidate looks unmaintained. Only the superseded SuprSend package is archived, and the replacement is active.

## Comparison summary

| | Knock | Novu | Courier | MagicBell | SuprSend |
| --- | --- | --- | --- | --- | --- |
| Free tier | 10,000 messages/mo | 10,000 workflow runs/mo | 10,000 sends/mo | 1,000 deliveries/mo | 10,000 notifications/mo |
| First paid tier | $250/mo (50,000 messages) | $30/mo (30K runs) | $0.005/send, no included volume | $249/mo (50,000 deliveries) | $100/mo (50,000 notifications) |
| Billable unit | message = one message to one user on one channel | workflow run (not defined on the pricing page) | send = one template to one recipient | delivery = notification per channel per user | notification = message to one user on one channel |
| React UI | `@knocklabs/react` (bell, popover feed) | `@novu/react` `<Inbox />` plus hooks | `@trycourier/courier-react` `<CourierInbox />` | `@magicbell/react` (`FloatingInbox`, `Inbox`) | `@suprsend/react` (`Inbox`), `@suprsend/react-core` (headless hooks) |
| Client auth | Signed JWT user token (RS256), "enhanced security mode" | HMAC `subscriberHash` | JWT with scopes from `/auth/issue-token` | User JWT (HS256, signed with project secret) | ES256 JWT, "enhanced security mode" |
| Digest | Batch function | Digest step | Scheduled digests | Not found in docs | Batch and Digest nodes |
| Delay / schedule | Delay (max 365d), schedules | Delay (1d free to 30d team) | Delay (31d or 93d, **Conflict**) | `wait` step in workflows | Delay node (max not stated) |
| Account-level in-app retention | Unlimited feed retention (all plans) | 30d free, 90d Pro/Team | Not stated | Not stated | Not stated |

Prices and limits are the published figures on each pricing page on the research date.

---

## Knock

### Pricing and free tier
Source: https://knock.app/pricing
- Free: $0/month, up to 10,000 messages sent, 500 guide recipients, 30-day log retention.
- Starter: $250/month platform fee, 50,000 messages then $0.005 each, 30-day log retention.
- Enterprise: custom, 90-day log retention.
- Billable message: "a single message delivered to a single user on a single channel". One workflow sending in-app plus email counts as two. Bounced or failed messages are not billed.
- Not a hard cap: exceeding the free limit without a billing method leads Knock to contact you the following month.
- Feed retention listed as unlimited on all plans; feeds per recipient unlimited; stored profiles, workflows and in-app guides unlimited.
- Pricing-page uptime SLA row: a search summary of Knock pages claimed "99.9% uptime SLA for Free and Starter", but the pricing page fetch did not show it. **Unverified.**

### React and server SDK
- React: `@knocklabs/react` with `KnockProvider`, `KnockFeedProvider`, `NotificationIconButton` (bell), `NotificationFeedPopover`, `NotificationFeed`, `NotificationCell`. Source: https://docs.knock.app/in-app-ui/react/feed
- Customisation via `renderItem`, children and `--rnf-` CSS variables. The page does not use the term "headless" and does not mention React Router or Next.js. A headless hook-only API is **Unverified** (`useNotifications` is mentioned only in passing).
- Server SDKs listed: Node.js, Python, Ruby, Go, PHP, Java, C#, Elixir. Source: https://docs.knock.app/reference

### Identity and stored user data
Sources: https://docs.knock.app/in-app-ui/security-and-authentication, https://docs.knock.app/concepts/users
- Client uses a public API key. Without enhanced security mode (off by default) "a request with your public key can act as any user".
- With enhanced security mode on, requests need a signed user token: a JWT signed on your server with a private signing key (RS256), `sub` claim is the user id, `exp`/`iat` recommended. Node SDK has `signUserToken`. Signing is local (no network call). `onUserTokenExpiring` callback exists for refresh.
- Required user field: `id` (your id; cannot be changed afterwards). Optional: `email`, `name`, `avatar`, `phone_number`, `timezone`, and arbitrary `properties`. Every user who receives a notification is identified and persisted in Knock. Users are isolated per environment.

### Scheduled, delayed, digest
- Delay: fixed or dynamic, max 365 days. Source: https://docs.knock.app/designing-workflows/delay-function
- Batch: window fixed, dynamic, relative or sliding; max 90 days open; max activity count 2 to 1,000; templates render the first or last 10 items by default. Knock says batching is not exactly digesting and recommends supplying events yourself for fixed-schedule digests. How batches display in the in-app feed is not described. Source: https://docs.knock.app/designing-workflows/batch-function
- Recurring and one-off schedules: via a schedules API with repeat rules (hourly, daily, weekly, monthly), timezone-aware per recipient. One-off schedules are not timezone-aware. Exclusion rules not supported. Source: https://docs.knock.app/concepts/schedules

### Read state, retention, export, deletion
- Seen and read are both supported on the feed (opening the popover marks items seen by default; badge can count unread). Archived is **Unverified** on the pages read. Source: https://docs.knock.app/in-app-ui/react/feed
- Retention: logs 30 days (90 days Enterprise); "in-app message data and the Feeds API are not" subject to retention enforcement. Custom retention policies not supported. Sources: https://docs.knock.app/concepts/messages, https://docs.knock.app/manage-your-account/data-retention (the latter seen only as a search result)
- Deletion: `DELETE /v1/users/{user_id}` "Permanently delete a user and all associated data" (rate limit tier 2); bulk delete for users, objects, subscriptions and tenants. What "all associated data" covers is not enumerated. Source: https://docs.knock.app/api-reference/users/delete
- Data export: no user data export documentation found. Warehouse/Segment sync is listed on the pricing page as Enterprise-only (per a search summary). **Unverified.**

### Rate limits and outage behaviour
- Per-endpoint tiers of 1, 5, 60, 200 and 1,000 requests/second; the endpoint-to-tier mapping did not render. 429 on excess; batch endpoints may have an extra "1 update / second / entity" dedupe limit. No `Retry-After` header documented. Source: https://docs.knock.app/api-reference/overview
- Outage and failure behaviour, status page, SLA: **not found** in docs read. Marketing claims of 99.99% uptime and SOC 2 Type II came from search summaries of knock.app pages; treat as vendor-stated and unverified.

---

## Novu

### Pricing and free tier
Source: https://novu.co/pricing
- Free: $0, 10,000 workflow runs/month, 5 workflows, 3 team members, unlimited subscribers, 60 events/second.
- Pro: from $30/month, 30K runs, then $1.20 per 1K; 240 RPS.
- Team: from $250/month, 250K runs, $1.20 per 1K; 600 RPS.
- Enterprise: custom.
- A "workflow run" is not defined on the fetched page. **Unverified.** Some comparison-table values did not map cleanly to rows; re-check on the live page.
- Source code is public on GitHub (self-hosting was not researched). https://github.com/novuhq/novu

### React and server SDK
Sources: https://docs.novu.co/platform/inbox/overview, https://docs.novu.co/platform/inbox/react/get-started
- `@novu/react` `<Inbox />` (bell, notification list, preferences modal). Hooks ("headless") in `@novu/react` and framework-agnostic `@novu/js`. Requires React 18+; React 19 supported.
- React example uses `routerPush={(path) => navigate(path)}` with `useNavigate` from `react-router-dom`. No React Router 7/8-specific guidance found.
- Server SDK: `@novu/api` exists on npm (3.19.1). Language list **Unverified**.

### Identity and stored user data
Sources: https://docs.novu.co/platform/inbox/prepare-for-production, https://docs.novu.co/platform/concepts/subscribers
- Client identifies with `applicationIdentifier` and `subscriberId`. Production hardening is HMAC: `HMAC-SHA256(secretKey, subscriberId)` as hex, passed as `subscriberHash`. If HMAC is enabled and the hash is missing, "the Inbox will not load". JWT authentication was not found in the docs read.
- Subscriber profile: `subscriberId` (required, your id), `email`, `phone`, `firstName`, `lastName`, `locale`, free-form `data`, device tokens, preferences, topic subscriptions. Subscribers are created on the fly at trigger time. Bulk create up to 500 per call.

### Scheduled, delayed, digest
Sources: https://docs.novu.co/platform/developer/limits, https://docs.novu.co/platform/workflow/add-and-configure-steps/configure-action-steps/digest
- Max delay duration: 1 day Free, 7 days Pro, 30 days Team, custom Enterprise.
- Max digest duration: same (1 / 7 / 30 days, custom).
- Digest types: regular (duration) and scheduled (minute to monthly, in the subscriber's time zone). Steps after a digest, including in-app, stay paused until the digest finishes.
- A failed digest, delay or throttle step stops the run and cancels remaining steps. Source: https://docs.novu.co/platform/developer/delivery-retries

### Read state, retention, export, deletion
Sources: https://docs.novu.co/platform/sdks/javascript, https://docs.novu.co/platform/developer/limits
- States in `@novu/js`: read/unread, seen (no unseen), archived/unarchived, snoozed/unsnoozed, delete; bulk `readAll`, `archiveAll`, `deleteAll`; realtime events and unread counts.
- Retention: Inbox feed 30 days Free, 90 days Pro and Team; Activity feed 1 day Free, 7 days Pro, 90 days Team; custom Enterprise.
- Delete subscriber: `DELETE /v2/subscribers/{subscriberId}`, described (via a search summary of https://docs.novu.co/api-reference/subscribers/delete-a-subscriber) as removing the subscriber with associated messages, preferences and topic subscriptions. Direct fetch not done. **Unverified.**
- Data export: not found.

### Rate limits and outage behaviour
Sources: https://docs.novu.co/api-reference/rate-limiting, https://docs.novu.co/platform/developer/delivery-retries
- Per-plan requests/second. Events (trigger): 60 Free, 240 Pro, 600 Team, 6K Enterprise. Configuration: 20 / 80 / 200 / 2K. Global (other endpoints): 30 / 120 / 300 / 3K. Bulk requests cost 100 tokens.
- 429 with `Retry-After` header; `RateLimit-*` headers; idempotency supported.
- Provider sends are attempted once with no retry; a transient provider outage can leave notifications "permanently undelivered"; no dead-letter queue or re-drive; no automatic provider failover. In-app delivery specifically under vendor outage: **not found.**

---

## Courier

### Pricing and free tier
Source: https://www.courier.com/pricing
- Developer (free): 10,000 sends/month.
- Business: $0.005 per send, charged on every send including the first 10,000 (50,000 sends = $250).
- Enterprise: custom, described as below $0.005 per send.
- Billable send: "One send is one template sent to one recipient"; multi-channel delivery of one template counts as one send. Test environments count the same as production.
- Retention and inbox limits not on the pricing page.

### React and server SDK
Sources: https://www.courier.com/docs/platform/inbox/inbox-overview, https://www.courier.com/docs/sdk-libraries/node
- `@trycourier/courier-react` v9 (React 18+; a React 17 package exists). `<CourierInbox />` with theming (`CourierInboxTheme`), custom views, or building on low-level APIs. Older `@trycourier/react-inbox` packages are deprecated.
- `useCourier()` exposes the shared client; no set of headless hooks described. React Router not mentioned.
- Server SDKs with doc pages: Node, Go, PHP, Ruby (https://www.courier.com/docs/sdk-libraries/go, /php, /ruby, per search results; Node page fetched via search only). Complete language list **Unverified.**

### Identity and stored user data
Sources: https://courier.com/docs/platform/inbox/authentication, https://courier.com/docs/platform/users/users
- Backend calls `POST /auth/issue-token` with the API key to mint a short-lived JWT; the client calls `signIn({ userId, jwt })`. Scope string must include `user_id:<id>` plus `inbox:read:messages inbox:write:events` for Inbox. `expires_in` is optional and "omitting it mints a token that never expires". SDKs do not refresh tokens. EU workspaces need the EU endpoint (`https://api.eu.courier.com/auth/issue-token`).
- "Nothing has to exist in Courier first, since signing in registers the user." `user_id` is any string you choose. Profile fields (all optional) include `email`, `phone_number`, `name`, `locale`, `zoneinfo`, `custom`. Message logs may contain PII from payloads (per docs summary in search results).

### Scheduled, delayed, digest
Sources: https://www.courier.com/docs/send/delay-and-schedule, https://www.courier.com/docs/platform/automations/digest
- Send API delay: `delay.duration` (ms), `delay.until` (ISO 8601) or an opening-hours expression (e.g. `Mo-Fr 09:00-17:00`). "A delay can run up to 31 days." Per-recipient time zone resolution.
- **Conflict:** a Journeys page (seen via search summary at https://courier.com/docs/platform/automations) says a single delay can wait up to 93 days. Surfaces may differ; verify.
- Scheduled digests: release times come from schedules on a subscription topic in the preferences editor; events batch until the next release. Digest keeps at most 10 items per category in `items` (count can be higher). How digests render in the in-app inbox is not described.

### Read state, retention, export, deletion
- States: read, opened, archived, synced across devices. Source: https://www.courier.com/docs/platform/inbox/inbox-overview
- Retention period for inbox messages and logs: **not found.** Courier guides describe inbox messages as persisting until archived but no platform-enforced expiry (guides recommend setting expiries; per search results).
- Deletion: `DELETE /profiles/{user_id}` removes the profile. Whether device tokens, preferences and list memberships are included is unclear; a Courier EU-residency article (search result) says it takes more than one call and preferences have no delete call. **Conflict / unverified.** The Courier docs page states there is no endpoint to list or export all profiles. Source: https://courier.com/docs/platform/users/users
- Data export: no export endpoint (see above).

### Rate limits and outage behaviour
Source: https://courier.com/docs/reference/rate-limits
- `POST /send` is "not rate-limited by request count". Listed limits: `PUT /lists/{list_id}` 60/min; user preferences 50/second; message resend 1 per 2 seconds per message. Endpoints not listed "aren't request-count limited today". 429 on excess with `X-RateLimit-Limit` and `X-RateLimit-Remaining`. Per-user/topic/tenant send limits are configurable.
- Outage behaviour of Courier itself: **not found.**

---

## MagicBell

### Pricing and free tier
Source: https://www.magicbell.com/pricing
- Builder: $0, 1,000 deliveries/month, 1 project, unlimited team members and users.
- Startup: $249/month, 50,000 deliveries, 5 projects ($99 per extra project), $0.0025 per extra delivery.
- Enterprise: custom.
- Delivery: counted per channel per user (100 users in-app plus 50 email = 150 deliveries). Builder overage "charged per use" with no price shown.
- Retention not on the pricing page.

### React and server SDK
Sources: https://www.magicbell.com/docs/libraries/magicbell-react, https://www.magicbell.com/docs/libraries/magicbell-js
- `@magicbell/react` with subpath imports: `context-provider`, `inbox`, `floating-inbox`, `user-preferences`, `toast`, etc. Headless hooks not mentioned. React Router/SSR not covered.
- JS client: npm `magicbell-js` (`magicbell-js/project-client`); `saveUser`, `deleteUser`, `createBroadcast`. The page does not state it is server-side. Other docs list Go, CLI, Svelte, Preact and web components clients. Source: https://www.magicbell.com/docs

### Identity and stored user data
Sources: https://www.magicbell.com/docs/api/authentication/user, https://www.magicbell.com/docs/primitive/user
- User JWT generated on your backend, HS256, signed with the project Secret Key. Claims: `user_email` and/or `user_external_id`, plus `api_key` (the project API key). Examples use `expiresIn: '1y'`. Passed as the `token` prop of the React provider.
- User fields: `external_id` (cannot be changed or unset), `email`, first/last name, `custom_attributes`. Older v1 docs described HMAC headers (about 1,877 days old per a search snippet); treat as outdated.

### Scheduled, delayed, digest
- Workflows offer a `wait` step (seconds), `pause` and `abort`. Source: https://www.magicbell.com/workflows (search summary; not fetched directly). Per-user "delivery schedules" appear in the preference description (https://www.magicbell.com/docs/primitive/user).
- Built-in digest or batch step: **not found** in docs; blog posts describing digests are generic design patterns. Scheduled delivery via the API: **not found.**

### Read state, retention, export, deletion
- Read, seen and archived states are tracked (timestamps `read_at`, `seen_at`, `sent_at` named). Source: https://www.magicbell.com/docs/primitive/notification
- Retention: **not found.** Data export: **not found.**
- Deletion: v2 API deletes a user "and all associated data from the project" (per search summary of https://www.magicbell.com/docs/api/reference; `deleteUser` exists in the JS client).

### Rate limits and outage behaviour
Source: https://www.magicbell.com/docs/api
- "500 req/minute counted per IP address"; 429, block lifts after 60 seconds. Idempotency-Key header (cached results expire after 24 hours). v1 is deprecated; v2 is current. Outage behaviour: **not found.**

---

## SuprSend

### Pricing and free tier
Source: https://www.suprsend.com/pricing
- Free: $0, 10,000 notifications/month, 30-day log retention, in-app feed shows SuprSend branding. Processing continues past the limit; SuprSend alerts and contacts you.
- Essentials: $100/month (or $1,200 annually), 50,000 notifications, $2 per 1,000 extra, in-app feed without branding, 90-day log retention.
- Business: $250/month, 50,000 notifications, $5 per 1,000 extra; adds batching and digest, user preferences, objects, i18n.
- Enterprise: custom.
- Billable notification: a message to one user or object on one channel. Batched or digested requests count as one notification. A monthly-tracked-users model is available via sales.
- Note: per the pricing page, batching/digest is listed under the Business plan. Whether Free or Essentials plans can use the batch node was not confirmed. **Unverified.**

### React and server SDK
Sources: https://docs.suprsend.com/docs/inbox-react, https://docs.suprsend.com/docs/react-sdk-headless-feed
- `@suprsend/react` (drop-in `SuprSendProvider` and `Inbox`) and `@suprsend/react-core` (headless: `SuprSendFeedProvider`, `useFeedClient`, `useFeedData`). React Router not mentioned.
- Server SDKs: `@suprsend/node-sdk` on npm (1.15.1); other languages **Unverified** (a Java trigger page exists).

### Identity and stored user data
Sources: https://docs.suprsend.com/docs/inbox-quick-start, https://docs.suprsend.com/docs/users
- Enhanced security mode (off in Sandbox, must be enabled for production): signed JWT, ES256, claims `entity_type` ("subscriber"), `entity_id` (your `distinct_id`), `iat`, `exp`; signing key shown once. SDK calls `refreshUserToken` before expiry. Inbox auth replaced HMAC `subscriber_id`; React Native and Flutter inbox SDKs still use HMAC.
- `distinct_id` is immutable and workspace-scoped. Stored: channel identities (`$email`, `$sms`, `$webpush` etc.), custom properties, reserved `$` fields (`$timezone`, `$preferred_language`), preferences.

### Scheduled, delayed, digest
Sources: https://docs.suprsend.com/docs/delay, https://docs.suprsend.com/docs/batch, https://docs.suprsend.com/docs/digest
- Delay: fixed, dynamic or relative; maximum **not stated**. Missing or invalid dynamic value stops the workflow.
- Batch: fixed, dynamic or relative window; window cannot change once opened; retain 1 to 100 events (default first 10). Maximum window length not stated.
- Digest: fixed or dynamic schedule (minutely to monthly, timezone-aware, recipient timezone fallback). In-app handling of digests not described.

### Read state, retention, export, deletion
- Feed states in the headless SDK: seen (`seen_on`), read (`read_on`), archived, pinned; methods mark seen, read, archive. Source: https://docs.suprsend.com/docs/react-sdk-headless-feed. Optional in-app template `expiry` field exists, behaviour not explained. Source: https://docs.suprsend.com/docs/inbox-quick-start
- Retention for inbox messages: **not found.** Log retention 30 days Free, 90 days Essentials and Business. Source: https://www.suprsend.com/pricing
- Deletion: delete user API removes profile, channel identities, preferences and per-tenant overrides. Whether inbox messages are removed is not stated. Source: https://docs.suprsend.com/docs/users
- Data export: **not found.** Log sync to Segment or a warehouse is Business and above.

### Rate limits and outage behaviour
- A documented API requests-per-second limit and 429 behaviour: **not found.** Workflow "throttle" is a per-user execution cap, not an API limit (https://docs.suprsend.com/docs/throttle). A trigger accepts up to 100 recipients (https://docs.suprsend.com/docs/trigger-workflow). The headless React feed has an opt-in `reachability` status (ONLINE, DEGRADED, AUTH_ERROR, OFFLINE) for connection state. Vendor-side outage behaviour: **not found.**

---

## Cross-vendor observations (facts, not a recommendation)

- All five need Cadence user ids registered or implicitly created on the vendor side, and each stores a user record (email, name and custom fields are optional on most). Notification content sent through the vendor (titles, bodies, payload data) is stored by the vendor; retention of that content is documented only by Knock (feed unlimited) and Novu (30 to 90 days inbox).
- Client-side auth model splits two ways: signed JWT per user (Knock, Courier, MagicBell, SuprSend) versus per-user HMAC hash (Novu). Two vendors have a dashboard switch that must be turned on for token checks: Knock's enhanced security mode is off by default ("a request with your public key can act as any user" without it), and SuprSend only verifies tokens when its enhanced security mode is on (off in Sandbox). Novu's Inbox only requires the hash once HMAC is enabled in provider settings.
- Time-based nudges ("you have not watched in N days") need either Cadence to trigger on a schedule (Cadence has no scheduler today) or a vendor schedule/delay. Vendor-side delay maximums found: Knock 365 days, Courier 31 or 93 days, Novu 1 to 30 days by plan. SuprSend and MagicBell maxima not stated.
- Documentation gaps worth a direct question to vendors: in-app retention (Courier, MagicBell, SuprSend), data export (all five), exact scope of user deletion (Knock, Courier, SuprSend), vendor outage behaviour and status pages (all five), and React Router framework-mode support (none documented).

## Open questions left for follow-up tickets

- Confirm with each shortlisted vendor the items marked **Unverified** or **Conflict** above.
- Verify React Router 8 compatibility with a spike; none of the five vendors' docs mention React Router 8 specifically.
