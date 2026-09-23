---
name: Snapshot an Amazon Lex bot before you destroy anything
description: Amazon Lex has no undo. Take an immutable version or a full export archive BEFORE any Delete* or destructive Update*, and know how to restore from it if the change goes wrong.
api: openapi/amazon-lex-models-v2-openapi.yml
operations: [CreateBotVersion, DescribeBotVersion, CreateExport, DescribeExport, CreateUploadUrl, StartImport, DescribeImport, DeleteBot, DeleteBotLocale, DeleteIntent, DeleteSlot, DeleteSlotType]
generated: '2026-09-17'
method: generated
source: openapi/amazon-lex-models-v2-openapi.yml + conventions/amazon-lex-conventions.yml (reversibility block) + smithy/amazon-lex-models-v2-2020-08-07.json
---

# Snapshot an Amazon Lex bot before you destroy anything

## The fact this skill exists for

Amazon Lex V2 publishes **no undo, restore, rollback or reverse operation, and no
recovery window**. Checked across all 107 Model Building V2 operations in the
AWS-published service model: the only `Stop*` operations cancel long-running
analysis jobs, and `DeleteBotReplica` merely removes a replica. `DeleteBot`'s own
documentation is unambiguous — it "deletes all versions of a bot, including the
Draft version".

So the reversal has to be arranged **in advance**. An agent that deletes a bot
with no prior version and no export cannot get it back.

## Steps — before the destructive call

1. **Cheapest snapshot: an immutable version.** `CreateBotVersion`
   (`PUT /bots/{botId}/botversions`) with a `botVersionLocaleSpecification`
   naming every locale to freeze. Poll `DescribeBotVersion`
   (`GET /bots/{botId}/botversions/{botVersion}`) until `botStatus` is
   `Available`. The version is immutable and cannot itself be edited away — only
   deleted deliberately.
2. **Portable snapshot: a full export.** `CreateExport` (`PUT /exports`) with a
   `resourceSpecification` naming the bot or bot locale and
   `fileFormat: LexJson`. Poll `DescribeExport` (`GET /exports/{exportId}`) until
   `exportStatus` is `Completed`, then fetch `downloadUrl`. This archive leaves
   AWS; the version does not.
3. **Only then destroy.** `DeleteBot`, `DeleteBotLocale`, `DeleteIntent`,
   `DeleteSlot`, `DeleteSlotType` — all permanent.

## Steps — restoring

1. **Get an upload URL.** `CreateUploadUrl` (`POST /createuploadurl`) returns a
   pre-signed S3 `uploadUrl` and an `importId`.
2. **PUT the archive to that URL.** Plain S3 upload, outside the Lex API.
3. **Start the import.** `StartImport` (`PUT /imports`) with the `importId`, the
   `resourceSpecification` for the target, and a `mergeStrategy`
   (`Overwrite` or `FailOnConflict`).
4. **Poll.** `DescribeImport` (`GET /imports/{importId}`) until `importStatus`
   is `Completed`; `Failed` carries `failureReasons`.

## Rules an agent must follow here

- **Do not treat a numbered version as a backup of a deleted bot.** Versions live
  *inside* the bot. `DeleteBot` takes them with it. Only the export archive
  survives the bot.
- **No idempotency key on any of these.** A retried `CreateExport` starts a
  second export job and bills a second time. Poll the first `exportId` instead.
- **`ConflictException` (409) on a delete usually means "still building".** Poll
  `DescribeBotLocale` until the status settles, then delete.
- **Deleting is not throttled the way you might hope.** There is no confirmation
  step, no soft-delete state and no grace period in the API. The check has to be
  yours.
