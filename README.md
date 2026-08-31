# whatsapp-chat-export

Hermes skill: export a WhatsApp chat (group or direct) into a single, self-contained HTML file styled like WhatsApp Web dark mode — base64-embedded media, reaction pills, date separators — ready to publish to [here.now](https://here.now).

## Layout

```
skills/whatsapp-chat-export/
├── SKILL.md                       # pipeline: JID resolution → SQL pulls → media download → HTML build → publish
└── references/export-technique.md # canonical Welp-style template: full CSS + bubble/reaction/media structure
```

## Install

Copy `skills/whatsapp-chat-export/` into your agent's skills directory:

```bash
cp -r skills/whatsapp-chat-export ~/.hermes/skills/social-media/
```

Requires the companion skills on the host:
- `whatsapp-mcp-local-bridge` (WhatsApp MCP bridge + contact/LID resolution)
- `here.now` (publishing)
