# Sendcove

The `sendcove` MCP server manages the user's Sendcove social media posts, media library, analytics and inbox (X, Instagram, TikTok, Threads, Bluesky, YouTube, Facebook Pages, LinkedIn). On first use, Gemini CLI opens a browser to sign in to Sendcove; if the tools say they need authentication, ask the user to run `/mcp auth sendcove`.

For any Sendcove task, activate the `sendcove` skill for the full guidance. The essentials:

- Call `list_accounts` first and pass its ids as `accountIds`.
- `create_post` makes a draft by default. Only schedule, publish or reply when the user asked for it.
- Use `platformContent` for per-platform text and keep each platform within its character limit; take `platformSettings` fields from the tool schema.
- Pass an `idempotencyKey` to `create_post` and retry with the same key after a timeout.
- Upload local files with `create_media_upload_url` and the curl command it returns.
- A 403 "can only create drafts" means the Basic plan or a drafts-only connection: keep the draft and tell the user publishing needs Pro or Max and reconnecting with "Allow publishing".
