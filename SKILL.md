---
name: twitter
description: Read tweets, threads, or user timelines using bird CLI. Triggers: "read tweet", "get tweet", "fetch tweet", "read thread", "user's tweets", "tweets from @handle", "save tweets".
---

# Twitter skill

Read tweets from X/Twitter using the bird CLI.

## Prerequisites

- bird CLI installed globally (`npm install -g @steipete/bird`)
- Credentials stored in `~/.claude/skills/twitter/.env`

## Authentication

**Always source the .env file before running bird commands:**

```bash
source ~/.claude/skills/twitter/.env && bird <command>
```

The .env file contains AUTH_TOKEN and CT0 cookies. It is gitignored.

## Commands

### Read single tweet

```bash
source ~/.claude/skills/twitter/.env && bird read <url-or-id> --json
```

### Read thread (OP only)

```bash
source ~/.claude/skills/twitter/.env && bird thread <url-or-id> --json
```

**Important:** `bird thread` returns ALL tweets in the conversation including replies from others. Filter the JSON output to only tweets where `author.username` matches the root tweet's author.

### Recent tweets from user

```bash
source ~/.claude/skills/twitter/.env && bird search "from:<handle>" -n <count> --json
```

Default count is 10. Can request more with `-n 20`, etc.

## Output handling

### Single thread: Display in chat

Format tweets as clean text with images:
```
**1/N:** Tweet text here...

![](https://pbs.twimg.com/media/xxx.jpg)

**2/N:** Next tweet...
```

Include any image URLs using markdown image syntax.

### Bulk operations (multiple threads)

**Before fetching, ask the user:**
- One combined file for all threads, or one file per thread?

Then save both formats:
1. **Markdown** (`.md`) - Human-readable with embedded images
2. **JSON** (`.json`) - Full API response data

### Save location

Default: `~/.claude/skills/twitter/saved-tweets/YYYY-MM-DD/`

Files are saved in a date subfolder (today's date). Only use a different location if the user explicitly requests it.

### Filename format

`YYYY-MM-DD-<username>-<brief-description>.{md,json}`

Examples:
- `2025-01-07-tobyordoxford-evaluating-infinite.md`
- `2025-01-07-multiple-threads.md` (for combined file)

### Markdown template

```markdown
# Thread from @username

**Date:** YYYY-MM-DD
**URL:** https://x.com/username/status/...

---

**1/N:** Tweet text here...

![](https://pbs.twimg.com/media/xxx.jpg)

**2/N:** Next tweet in thread...

**3/N:** Third tweet...

![](https://pbs.twimg.com/media/yyy.png)
```

For combined files with multiple threads, use `---` separators between threads.

## JSON schema reference

Tweet objects from `--json` include:

| Field | Description |
|-------|-------------|
| `id` | Tweet ID |
| `text` | Full tweet text |
| `author.username` | Author's handle |
| `author.name` | Display name |
| `createdAt` | Timestamp |
| `conversationId` | Thread conversation ID |
| `inReplyToStatusId` | Parent tweet ID (if reply) |
| `media` | Array of media objects (optional) |
| `media[].type` | `photo`, `video`, `animated_gif` |
| `media[].url` | Direct URL to media file |
| `replyCount` | Number of replies |
| `retweetCount` | Number of retweets |
| `likeCount` | Number of likes |

## Updating credentials

When cookies expire, get fresh ones manually:

1. Open Chrome → x.com (logged in)
2. Open DevTools (F12 or ⌘⌥I)
3. Application tab → Cookies → https://x.com
4. Copy values for `auth_token` and `ct0`
5. Update `~/.claude/skills/twitter/.env`:
   ```
   export AUTH_TOKEN=<auth_token value>
   export CT0=<ct0 value>
   ```

## Troubleshooting

**Missing credentials:** Ensure `~/.claude/skills/twitter/.env` exists with AUTH_TOKEN and CT0 values.

**Query ID errors:** Run `source ~/.claude/skills/twitter/.env && bird query-ids --fresh` to refresh GraphQL query IDs.

**Rate limiting:** X may rate limit requests. Wait and retry.

## Update check

This is a shared skill. Before executing, check `~/.claude/skills/.update-config.json`.
If `auto_check_enabled` is true and `last_checked_timestamp` is older than `check_frequency_days`,
mention: "It's been a while since skill updates were checked. Run `/update-skills` to see available updates."
Do NOT perform network operations - just check the local timestamp.
