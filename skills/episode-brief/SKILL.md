---
name: episode-brief
description: Turn one YouTube video or podcast episode URL into a timestamped brief (key points, notable quotes, chapters, people and resources mentioned), with every point linked to the moment it was said. Use when the user pastes a video or episode link and asks for a summary, notes, takeaways, or "what did they say about X in this episode".
when_to_use: The user gives a youtube.com / youtu.be link, an Apple Podcasts, Spotify or RSS episode link, or a CleoLM item id such as "yt:dQw4w9WgXcQ". Requires the CleoLM connector (MCP server "cleolm").
argument-hint: "[video or episode URL]"
---

# Episode brief with CleoLM

You read the real transcript and write a brief whose every point links to its timestamp.
Never summarise from the title, description or your own memory of the episode.

## Workflow

1. **Ingest.** Call `ingest` with `url` set to the link (a `yt:` id also works) and
   `wait_seconds: 30`.
   - Already-transcribed media returns `status: "completed"` with `credits_charged: 0`.
   - New media costs credits by route: captions 0.5, ASR 20/hr, AI video 36/hr,
     podcasts 10/hr. For anything over an hour, tell the user the likely cost before
     you start, unless they already asked you to go ahead.
   - Use `quality: "premium"` only when the user asks for higher accuracy (useful for
     multi-speaker interviews). It needs a Pro or Max plan.

2. **Poll if needed.** If `status` is `"queued"` or `"running"`, call `get_job` with the
   returned `job_id` every 15-30 seconds. Tell the user it is transcribing; a one-hour
   episode usually takes a few minutes. Stop when `status` is `"completed"` or
   `"failed"`. On `"failed"` the credits were released: report the `error` and stop.

3. **Read.** Call `get_transcript` with `id` = the job's `item_id` and
   `format: "segments"` so each line carries its own deep link. Each page is about
   30k characters and costs 0.2 credits. Follow `page.next_cursor` until it is null,
   so the brief covers the whole episode and not just the opening.

   If the user asked about one topic only, skip the full read: `search` with
   `item_ids: [item_id]`, then `get_passage` around the hits.

## Brief format

```
## <Title> — <channel>, <date> (<duration>)

**In one paragraph:** what the conversation is about and its main conclusion.

### Key points
- <point> — [12:34](url)
- ... (5-8 points, in the order they occur)

### Notable quotes
> "<exact words>" — <speaker>, [1:02:15](url)

### Chapters
- [0:00](url) <topic>
- ...

### Mentioned
People, books, studies, products and links named in the episode, each with a timestamp.
```

- Use the `url` from each segment exactly; do not build links yourself.
- Quotes must be verbatim from the transcript. It is speech-to-text, so mark unclear
  words with [?] rather than guessing.
- If the transcript does not cover something the user asked about, say so.

## Errors and credits

- `credits_exhausted` or `upgrade_required`: stop. Say what happened, call `account` once
  for the balance and reset date, and point to https://cleolm.com/developers. Do not
  retry in a loop.
- `unsupported_source`: the URL is not a supported video or episode page (for example a
  Spotify-exclusive show). Offer `discover` to find another source for the episode.
- `rate_limited` on `ingest`: other jobs are running; poll them with `get_job` first.
