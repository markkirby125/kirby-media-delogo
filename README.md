# Media Delogo

Removes the visible Gemini or Flow watermark (the sparkle or corner mark) from AI-generated images and video with ffmpeg and ImageMagick. Keeps the original, writes a `-clean` copy beside it, and checks the result before delivery.

It removes the visible mark only. Invisible provenance marks such as SynthID are untouched, and re-encoding drops EXIF and C2PA metadata.

### 🪄 The Magic Prompt
Copy and paste this into any AI coding app (Cursor, Windsurf, Claude Code, Grok, Kimi, Reasonix):

```markdown
@agent Install the Media Delogo skill from: https://github.com/markkirby125/kirby-media-delogo
1. Read `SKILL.md` and `references/` from the repository.
2. Place into your environment's skills/rules directory preserving the dispatcher structure.
3. Confirm when installation is complete.
```
