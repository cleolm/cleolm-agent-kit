---
name: channel-monitor
description: Report what is new from specific YouTube channels or podcasts (latest episodes, and what they said about a topic) over a recent window such as this week or since a date, with timestamped links. Use for "what's new from Lex Fridman this week", "did any of my podcasts cover the Fed decision", or a weekly digest across several shows.
when_to_use: The user names one or more channels or shows and a time window, or asks for a recurring digest or catch-up. Requires the CleoLM connector (MCP server "cleolm").
argument-hint: "[channels or shows] [time window or topic]"
---

# Channel monitor with CleoLM

You tell the user what is new from the sources they follow, and what was said, with
links to the exact moments.

## Workflow

1. **Resolve the window.** Turn "this week", "since Monday" or "last 30 days" into a
   `YYYY-MM-DD` start date. Use today's date; state the window you used.

2. **Classify each source (free).** Call `list_channels` with `query` set to the name.
   - **Indexed channel:** search covers it continuously, so new uploads are already
     searchable.
   - **Not indexed:** you can still list its uploads; their transcripts need `ingest`.

3. **List new episodes (free).** Call `list_episodes` with `source` set to the channel
   @handle, channel URL, RSS feed, or Apple Podcasts / Spotify show URL, and `limit` 10-20.
   Keep episodes whose `published_at` falls inside the window. If you do not know the
   handle or feed, call `discover` first (`type: "podcast"` returns `feed_url`).

4. **What did they say?**
   - With a topic, for indexed channels: one `search` (1 credit) with `query`,
     `channels` (all of them in one call, up to 20), and `published_after` set to the
     window start. Then `get_passage` (free) around the best hits.
   - Without a topic: list the episodes with titles and dates. Offer an
     `episode-brief` for the ones the user picks instead of transcribing everything.
   - Non-indexed sources: `ingest` only the episodes the user cares about, and confirm
     first if that is more than about two hours of audio (ASR is 20 credits/hr,
     podcasts 10/hr). Poll `get_job` until `completed`, then `search` with
     `item_ids`.

## Digest format

```
## New from <sources>, <start date> – <today>

### <Channel>
- **<Episode title>** (<date>, <duration>) — <one line on what it covers>
  - On <topic>: "<short quote>" [41:10](url)
```

- Newest first within each source. Say explicitly when a source had nothing new.
- Use the `url` values from the tool results exactly.

## Errors and credits

- `credits_exhausted` or `upgrade_required`: stop calling paid tools, report it once,
  call `account` for the reset date, and link https://cleolm.com/developers. Never retry
  in a loop. `list_channels`, `list_episodes`, `get_passage`, `get_job` and `account`
  are free and still work, so you can still list what is new.
- `not_found` from `list_episodes`: try the channel URL or the show's RSS feed, or find
  it with `discover`.
