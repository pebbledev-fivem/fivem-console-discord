# fivem-console-discord

Forwards your FiveM server console (and join/leave/chat/resource events) to a Discord channel via webhook.

## Setup

1. **Create a Discord webhook**
   In Discord: *Server Settings → Integrations → Webhooks → New Webhook*. Pick the channel, click **Copy Webhook URL**.

2. **Drop the resource into your server**
   Copy the `fivem-console-discord` folder into your `resources/` directory.

3. **Configure**
   Open `server/config.lua` and paste your webhook URL into `Config.WebhookURL`.

4. **Start it**
   Add to your `server.cfg`:
   ```
   ensure fivem-console-discord
   ```

## What it captures

- All `print()` and `Citizen.Trace` output from any resource
- Player joins / leaves (with drop reason)
- In-game chat messages
- Resource start / stop events

## Notes & gotchas

- **Rate limits.** Discord webhooks allow ~5 requests per 2 seconds per webhook. The script batches lines and flushes every `Config.FlushInterval` ms (default 2000) to stay safe. Don't lower it below ~1000.
- **Secrets.** `Config.Blacklist` filters out lines containing `license:`, `steam:`, tokens, passwords, etc., so identifiers and credentials don't leak into Discord. Add more terms as needed for your server.
- **2000-char limit.** Discord caps messages at 2000 chars. The script splits long batches across multiple messages.
- **Mentions are suppressed.** A player typing `@everyone` in chat won't ping your Discord — `allowed_mentions` is set to an empty parse list.
- **Channel privacy.** Treat the target channel as sensitive. Console output can include error stack traces, IPs, and resource internals. Use a private staff/admin channel.
- **Performance.** Wrapping `print` adds a tiny overhead to every print call. On a busy server with very chatty resources you may want to set `Config.Capture.prints = false` and rely only on event hooks.

## Customising

- Want errors only? Filter inside `enqueue()` — e.g. only push lines matching `error`, `SCRIPT ERROR`, or `Citizen.Trace` outputs that start with `^1`.
- Want a different format? Replace the `"```\n" .. chunk .. "\n```"` wrapper in `sendToDiscord` with a Discord embed payload (`embeds = { { description = ..., color = 0xff0000 } }`).
- Want multiple channels (e.g. chat → #chat-relay, errors → #errors)? Duplicate `Config.WebhookURL` into multiple keys and route in `enqueue()` based on the line's prefix.
