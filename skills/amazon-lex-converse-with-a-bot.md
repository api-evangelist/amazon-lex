---
name: Converse with an Amazon Lex V2 bot
description: Drive a live conversation against a built bot alias — send a turn, read dialog state and slots, inspect or set session state, and know which of the three runtime entry points to use.
api: openapi/amazon-lex-runtime-v2-openapi.yml
operations: [RecognizeText, RecognizeUtterance, StartConversation, GetSession, PutSession, DeleteSession]
generated: '2026-09-17'
method: generated
source: openapi/amazon-lex-runtime-v2-openapi.yml + conventions/amazon-lex-conventions.yml + errors/amazon-lex-problem-types.yml + rate-limits/amazon-lex-rate-limits.yml
---

# Converse with an Amazon Lex V2 bot

## Pick the right entry point

Amazon Lex Runtime V2 has six operations and only three of them start a turn.
The choice is not a style preference — it changes the price and the transport.

| Operation | Input | Transport | en-US price/request |
|---|---|---|---|
| `RecognizeText` | text | plain request/response | $0.00075 |
| `RecognizeUtterance` | audio or text | request/response, streamed body | $0.004 speech / $0.00075 text |
| `StartConversation` | audio, text or DTMF | bidirectional HTTP/2 stream | $0.0065 speech / $0.002 text |

`StartConversation` is the only one that supports barge-in and continuous
multi-turn streaming — and the only one with **no AWS CLI command**. It is
SDK-only. Prices are from `plans/amazon-lex-plans-pricing.yml` (read from the
AWS Price List API).

## Steps

1. **Address the alias, not the version.**
   `POST /bots/{botId}/botAliases/{botAliasId}/botLocales/{localeId}/sessions/{sessionId}/text`
   — `RecognizeText`. You invent the `sessionId`; Lex uses it to thread the
   conversation.
2. **Send the turn.** `RecognizeText` with `text` and optional
   `sessionState` / `requestAttributes`.
3. **Read what came back.** The response carries `messages[]` (each with a
   `contentType` of `PlainText`, `SSML`, `CustomPayload` or `ImageResponseCard`),
   `sessionState` (including `dialogAction`, `intent` and `slots`), and
   `interpretations[]` ranked by NLU confidence.
4. **Decide from `dialogAction.type`.** `ElicitSlot` means Lex wants another
   value from the user; `ElicitIntent` means it did not understand;
   `Close` means the intent is fulfilled or failed; `Delegate` means your Lambda
   hook decides.
5. **Inspect or steer state out of band.** `GetSession`
   (`GET .../sessions/{sessionId}`) reads the live state; `PutSession`
   (`POST .../sessions/{sessionId}`) writes it — use it to pre-fill slots or jump
   the dialog to a known intent.
6. **End the session.** `DeleteSession` (`DELETE .../sessions/{sessionId}`).
   Sessions also expire on the bot's `idleSessionTTLInSeconds` and are hard-capped
   at 15 minutes total duration.

## Rules an agent must follow here

- **Retries replay the user.** There is no idempotency key. A retried
  `RecognizeText` sends the same turn into the live session a second time and
  advances dialog state. Deduplicate on your side, keyed on `sessionId` + turn.
- **Concurrency is capped per alias.** 50 concurrent text-mode conversations on a
  normal alias, 125 voice via `RecognizeUtterance`, 400 voice via
  `StartConversation` — and **2 of everything on `TSTALIASID`**. Exceeding a cap
  returns 429 `ThrottlingException` with no `Retry-After`.
- **Input caps are hard.** 1,024 characters of text; 55 seconds of speech; 15
  minutes per conversation.
- **`DependencyFailedException` (424) means your Lambda.** The fulfillment or
  dialog code hook failed, timed out at 30 seconds, or exceeded the 50 KB output
  cap. Read the function's CloudWatch logs, not Lex's.
- **`AccessDeniedException` (403) is IAM, not a bad key.** There are no API keys.
  Check the calling principal against the `lex:*` actions published at
  `https://servicereference.us-east-1.amazonaws.com/v1/lex/lex.json`.
