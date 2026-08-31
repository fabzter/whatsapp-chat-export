---
name: whatsapp-chat-export
description: "Use when exporting WhatsApp chats to HTML. Bubbles, media, reactions, here.now."
version: 1.1.0
---

# WhatsApp Chat Export

Export any WhatsApp chat (group or direct) to a single, self-contained HTML file styled like WhatsApp Web dark mode, with all media embedded as base64 and reactions displayed as emoji pills.

## Prerequisites

- `whatsapp-mcp-local-bridge` skill loaded (contains contact resolution workflow)
- `here.now` skill loaded if publishing

## Reuse the proven look (do this FIRST when the user references past exports)

The user remembers past exports being "really excellent" — the fastest path to reproduce one is NOT digging through session history (that mostly surfaces skill-creation noise). **The published artifact IS the technique.** Steps:

```bash
# 1. List all sites on the account.
#    Auth: export HERENOW_API_KEY first (key storage is handled by the here.now
#    skill — publish.sh reads ~/.herenow/credentials itself; never cat it here).
curl -sS "https://here.now/api/v1/publishes" \
  -H "Authorization: Bearer $HERENOW_API_KEY" | jq -r '.publishes[] | [.slug, .siteUrl, .updatedAt] | @tsv'

# 2. Identify WhatsApp exports: fetch each candidate, grep for the title pattern
curl -sS "https://<slug>.here.now/" -o /tmp/site_<slug>.html
grep -oE '<title>[^<]*</title>' /tmp/site_<slug>.html   # look for '· WhatsApp Export'

# 3. Extract the technique from the best artifact:
#    - <style> block  -> the CSS
#    - sample bubbles (text / img-msg / audio-msg / reactions) -> the HTML structure
#    - the header stats line ("N participants · N messages · N 🎤 · N 🖼️ · N ❤️")
```

The canonical good-looking template (Welp-style: dark WhatsApp Web theme, emoji avatar header, colored sender names, base64 media, reaction pills) is preserved in `references/export-technique.md` — copy its CSS and structure verbatim, then adapt.

## Full pipeline

### 1. Resolve the chat JID

- **Groups:** Direct name search works. `SELECT jid, name FROM messages.chats WHERE name LIKE '%GroupName%'`
- **Direct (1:1) chats:** REQUIRES the two-step resolution documented in `whatsapp-mcp-local-bridge`: find contact in `whatsapp.db` → resolve LID via `whatsmeow_lid_map` → `chat_jid = '<LID>@lid'`

### 2. Pull messages

```bash
sqlite3 -json ~/src/whatsapp-mcp/whatsapp-bridge/store/messages.db \
  "SELECT id, timestamp, sender, content, is_from_me, media_type
   FROM messages WHERE chat_jid = '<JID>' ORDER BY timestamp ASC LIMIT 500;" \
  > /tmp/chat_messages.json
```

**Time filtering pitfall:** the `timestamp` column is TEXT (ISO 8601 with `-06:00` offset), NOT epoch ms. Comparing it against an epoch integer (`timestamp >= 1788156000000`) returns ALL rows — SQLite type ordering puts every INTEGER below every TEXT, so the filter silently passes. Always filter with an ISO string: `WHERE timestamp >= '2026-08-31 00:00:00-06:00'`. Verify the first/last row of the result actually fall inside your window before building anything.

### 3. Pull reactions

```bash
sqlite3 -json ~/src/whatsapp-mcp/whatsapp-bridge/store/messages.db \
  "SELECT target_msg_id, sender, emoji
   FROM reactions WHERE chat_jid = '<JID>';" \
  > /tmp/chat_reactions.json
```

Reaction rows carry `sender` as `<lid>@lid` (with suffix) while message `sender` is the bare LID — strip `@lid` when joining. The reactions table has no timestamp column: filter by `target_msg_id IN (today's message ids)` in Python, not SQL.

### 4. Download media

Query media messages, then use MCP `download_media` for each.

**Media will fail for old messages** — CDN tokens expire. Newly received media downloads fine. HTML uses `[media_type — unavailable]` placeholders for failures.

**Filename collision pitfall:** the bridge saves downloads as `<media_type>_<YYYYMMDD>_<HHMMSS>.<ext>`. Two media messages within the same second (common for rapid photo sends) map to the SAME filename and the second download silently overwrites the first. After EACH download, immediately rename the file to include the message ID (e.g. `img_<MSG_ID>.jpg`). Then `md5` the files: identical hashes mean the same media was sent twice — embed once and reuse the base64 for both bubbles (no data lost).

### 5. Build HTML

Use the Welp-style template from `references/export-technique.md` (full CSS + structure). Key adaptations:

- **Sender names:** resolve group LIDs to real names via `whatsmeow_lid_map` → phone → `whatsmeow_contacts.full_name`/`push_name`. Real names beat the old masked-phone fallback when resolvable. One palette color per sender, consistent across the page.
- **Mentions:** content contains raw `@<LID>` tokens — replace with `@Name` before escaping HTML.
- **Message IDs:** every `.msg-row` gets `id="msg-<UUID5>"` (uuid5 of `wa-msg:<message_id>` — stable across rebuilds) plus `data-wa-id="<raw message id>"`. These are the annotation targets (comments/tooltips/stickies) and survive re-exports. See `references/export-technique.md`.
- **Images:** add `max-width:100%; border-radius:4px` to `.img-msg img` — the stock template can overflow the bubble with tall/wide phone photos.
- Reactions as emoji pills, date separators between days, 800px max-width column, footer `Updated <date> · Reactions live`.

### 6. Publish to here.now

```bash
mkdir -p /tmp/chat_site && cp /tmp/chat_output.html /tmp/chat_site/index.html
PUBLISH=~/.hermes/skills/productivity/here-now/scripts/publish.sh
bash "$PUBLISH" /tmp/chat_site --client hermes
```

To update: `bash "$PUBLISH" /tmp/chat_site --slug <slug> --client hermes`

Verify the live page before telling the user: curl the siteUrl and confirm HTTP 200, the `<title>`, and the stats line.
