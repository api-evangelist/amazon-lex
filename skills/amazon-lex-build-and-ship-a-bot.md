---
name: Build and ship an Amazon Lex V2 bot
description: Create a bot, add a locale, intents and slots, build the locale, snapshot it as an immutable version, and point an alias at it — the full build-time path from nothing to callable, with the state machine an agent has to respect.
api: openapi/amazon-lex-models-v2-openapi.yml
operations: [CreateBot, DescribeBot, CreateBotLocale, DescribeBotLocale, CreateIntent, CreateSlotType, CreateSlot, BuildBotLocale, CreateBotVersion, DescribeBotVersion, CreateBotAlias, UpdateBotAlias, ListBots]
generated: '2026-09-17'
method: generated
source: openapi/amazon-lex-models-v2-openapi.yml + conventions/amazon-lex-conventions.yml + errors/amazon-lex-problem-types.yml + rate-limits/amazon-lex-rate-limits.yml
---

# Build and ship an Amazon Lex V2 bot

## The shape of the thing

Every build-time resource below the bot is addressed by a four-part key: `botId`,
`botVersion`, `localeId`, then its own id. While you are developing, `botVersion`
is the literal string `DRAFT`. There is no separate "development" account — you
are editing a real resource in a real AWS account the whole time.

Building is **asynchronous and stateful**. `BuildBotLocale` returns immediately;
the locale then moves through statuses and is not usable until it settles. Most
`ConflictException` (409) responses an agent sees on this path are not a bug —
they are "the resource is still building, come back".

## Steps

1. **Create the bot.** `CreateBot` (`PUT /bots`). Requires `botName`, `roleArn`
   (an IAM role Lex assumes), `dataPrivacy` and `idleSessionTTLInSeconds`. The
   response carries the generated `botId`.
2. **Wait for it.** `DescribeBot` (`GET /bots/{botId}`) until `botStatus` is
   `Available`.
3. **Add a locale.** `CreateBotLocale`
   (`PUT /bots/{botId}/botversions/{botVersion}/botlocales`) with
   `botVersion: DRAFT` and a `localeId` such as `en_US`, plus an
   `nluIntentConfidenceThreshold`.
4. **Define value domains first.** `CreateSlotType`
   (`PUT .../botlocales/{localeId}/slottypes`) for any custom value list. Skip
   this where a built-in type will do — list them with `ListBuiltInSlotTypes`
   (`POST /builtins/locales/{localeId}/slottypes`).
5. **Create the intent.** `CreateIntent`
   (`PUT .../botlocales/{localeId}/intents`) with `intentName` and
   `sampleUtterances`. The response carries `intentId`.
6. **Create its slots.** `CreateSlot`
   (`PUT .../intents/{intentId}/slots`) with `slotTypeId` and a
   `valueElicitationSetting`. Order matters: a slot cannot reference a slot type
   that does not exist yet.
7. **Build the locale.** `BuildBotLocale`
   (`POST .../botlocales/{localeId}`). Then poll `DescribeBotLocale` until
   `botLocaleStatus` is `Built`. `Failed` carries `failureReasons`.
8. **Snapshot it.** `CreateBotVersion` (`PUT /bots/{botId}/botversions`) with a
   `botVersionLocaleSpecification` naming the locales to freeze. This is the only
   rollback material Lex gives you — see
   `conventions/amazon-lex-conventions.yml`.
9. **Point an alias at the version.** `CreateBotAlias`
   (`PUT /bots/{botId}/botaliases`) with `botVersion`. Runtime traffic addresses
   the alias, never the version directly. To ship a new version later, call
   `UpdateBotAlias` and repoint it — that is the whole deployment story.

## Rules an agent must follow here

- **There is no idempotency key.** Not one operation in the service model takes
  one. A retried `CreateBot` or `CreateIntent` creates a second resource unless
  the name collides. Treat `ConflictException` (409) as "already exists or still
  building": call the matching `Describe*` and decide, never blind-retry the
  `Create*`.
- **Respect the build-time quotas.** 100 bots per account, 1,000 intents per
  en-US locale (250 elsewhere), 100 slots per intent, 5 parallel locale builds
  per account. Exceeding one returns `ServiceQuotaExceededException` (402), not
  a 429. Full table in `rate-limits/amazon-lex-rate-limits.yml`.
- **Back off on 429.** `ThrottlingException` carries no `Retry-After` and Lex
  publishes no rate-limit headers at all — exponential backoff with jitter is
  the only available strategy.
- **Do not test against `TSTALIASID`.** The built-in test alias is hard-capped at
  2 concurrent conversations on every dimension and is not adjustable.
