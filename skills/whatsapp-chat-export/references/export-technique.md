# Export Technique — canonical Welp-style template

Extracted from the published "Welp · WhatsApp Export" artifact (the export the user calls "really excellent"). Copy verbatim and adapt; do not redesign.

## CSS (drop in as-is)

```css
  * { margin:0; padding:0; box-sizing:border-box; }
  body { 
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    background: #0B141A; color: #E9EDEF;
  }
  .chat-bg { 
    background: #0B141A;
    background-image: url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' fill-rule='evenodd'%3E%3Cg fill='%23182129' fill-opacity='0.6'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E");
    min-height: 100vh; padding: 20px; max-width: 800px; margin: 0 auto;
  }
  .header { 
    background: #202C33; padding: 12px 20px; border-radius: 8px 8px 0 0; 
    display: flex; align-items: center; gap: 12px; position: sticky; top: 0; z-index: 10;
  }
  .header h2 { font-size: 18px; font-weight: 600; }
  .header .sub { font-size: 12px; color: #8696A0; }
  .date-separator { text-align: center; margin: 12px 0; }
  .date-separator span { 
    background: #182229; color: #8696A0; font-size: 12px; 
    padding: 5px 12px; border-radius: 6px; 
  }
  .msg-row { display: flex; margin: 2px 40px; }
  .msg-row-in { justify-content: flex-start; }
  .msg-row-out { justify-content: flex-end; }
  .bubble-in { 
    background: #202C33; max-width: 75%; padding: 6px 10px 3px; border-radius: 8px; 
    border-top-left-radius: 0; position: relative;
  }
  .bubble-out { 
    background: #005C4B; max-width: 75%; padding: 6px 10px 3px; border-radius: 8px; 
    border-top-right-radius: 0; position: relative;
  }
  .sender-name { font-size: 13px; font-weight: 600; margin-bottom: 2px; }
  .msg-content { font-size: 14.2px; line-height: 1.5; word-wrap: break-word; }
  .msg-time { font-size: 11px; color: #8696A0; text-align: right; margin-top: 2px; }
  .audio-msg { margin: 4px 0; }
  .audio-msg audio { width: 280px; height: 40px; border-radius: 6px; accent-color: #00A884; }
  .img-msg { margin: 4px 0; }
  .img-msg img { display: block; max-width: 100%; border-radius: 4px; }
  .audio-meta { font-size: 10px; color: #8696A0; margin-top: 2px; }
  .media-placeholder { font-size: 13px; color: #8696A0; padding: 4px 0; }
  .reactions { margin-top: 4px; display: flex; gap: 3px; flex-wrap: wrap; }
  .rx-pill { 
    background: #0B141A; border-radius: 10px; padding: 1px 6px; 
    font-size: 13px; display: inline-flex; align-items: center; gap: 2px;
  }
  .rx-count { font-size: 10px; color: #8696A0; }
  .footer { text-align: center; color: #8696A0; font-size: 12px; padding: 20px; }
```

NOTE: `max-width:100%; border-radius:4px` on `.img-msg img` is a deliberate improvement over the original artifact — the stock version lets tall phone photos (738x1600) overflow the bubble.

## Structure

### Header

```html
<div class="header">
  <div style="width:42px;height:42px;border-radius:50%;background:#3B4A54;display:flex;align-items:center;justify-content:center;font-size:20px;">🐾</div>
  <div><h2>Caro Fabs Pp</h2><div class="sub">3 participants · 54 messages · 4 🖼️ · 1 ❤️</div></div>
</div>
```

Stats line: `N participants · N messages` + optional ` · N 🎤` (audio) ` · N 🖼️` (images) ` · N ❤️` (reactions) — only include nonzero counts.

### Date separator

```html
<div class="date-separator"><span>Monday, August 31, 2026</span></div>
```

`%A, %B %d, %Y` — one per new day.

### Text bubble (outgoing / right / green)

```html
<div class="msg-row msg-row-out" id="msg-<UUID5>" data-wa-id="AC697359">
    <div class="bubble-out">
        <div class="sender-name" style="color:#E06469">Fabz</div>
        <div class="msg-content">Ok</div>
        <div class="msg-time">09:03 AM</div>
    </div>
</div>
```

### Text bubble (incoming / left / gray) with reaction

```html
<div class="msg-row msg-row-in" id="msg-<UUID5>" data-wa-id="AC5557A9">
    <div class="bubble-in">
        <div class="sender-name" style="color:#7D8CC4">Caro</div>
        <div class="msg-content">Sipi, y wi mal no rcierdo dow vabezqles</div>
        <div class="reactions"><span class="rx-pill">😢<span class="rx-count">1</span></span></div>
        <div class="msg-time">09:02 AM</div>
    </div>
</div>
```

### Image bubble

```html
<div class="img-msg">
    <img src="data:image/jpeg;base64,<BASE64>">
</div>
```

### Audio bubble

```html
<div class="audio-msg">
    <audio controls preload="metadata" src="data:audio/ogg;base64,<BASE64>"></audio>
</div>
```

### Footer

```html
<div class="footer">Updated August 31, 2026 at 01:45 PM · Reactions live</div>
```

## Message element IDs (for annotations)

Every message row carries a stable, unique GUID so annotations (comments, tooltips, labels, stickies) can target specific bubbles and survive re-exports:

```html
<div class="msg-row msg-row-in" id="msg-<UUID5>" data-wa-id="<whatsapp_message_id>">
```

- `id` = UUID5 GUID: `uuid.uuid5(uuid.NAMESPACE_URL, "wa-msg:" + message_id)` — same message → same GUID on every rebuild, so annotation anchors stay valid.
- `data-wa-id` = raw WhatsApp message id, for source correlation and re-annotation tooling.
- Python: `from uuid import uuid5, NAMESPACE_URL; elid = str(uuid5(NAMESPACE_URL, f"wa-msg:{m['id']}"))`

## Conventions

- **Page title:** `<Chat Name> · WhatsApp Export`
- **Sender colors:** palette `['#E06469', '#7D8CC4', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7', '#DDA0DD', '#98D8C8']`, assigned by first appearance order, one per sender, used consistently.
- **Sender names:** real contact names via `whatsmeow_lid_map` → `whatsmeow_contacts`. Fall back to masked phone `+552****6690` (first 4 + last 4 digits) only when unresolvable.
- **Time format:** 12-hour `%I:%M %p`, no leading zero.
- **is_from_me=1** → `msg-row-out` / `bubble-out` (green, right). Everything else → `msg-row-in` / `bubble-in`.
- **Mentions:** raw `@<LID>` tokens in content → `@<Name>` before HTML-escaping.
- **Reacted messages:** pills go between `.msg-content` and `.msg-time`.
- **Failed media:** `<div class="media-placeholder">📎 <em>[image — unavailable]</em></div>`.
- **Viewport:** add `<meta name="viewport" content="width=device-width, initial-scale=1">` for mobile.
