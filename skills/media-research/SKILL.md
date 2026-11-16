---
name: media-research
description: Research what experts, founders, scientists or commentators have said about a topic in YouTube videos and podcasts, and answer with quotes cited to timestamped deep links. Use for questions like "what has Huberman said about creatine", "how do VCs on podcasts talk about AI agents", or "find interviews where X discusses Y".
when_to_use: The user wants evidence from spoken media (interviews, podcasts, talks, lectures), wants to compare what different people said, or asks for sources with timestamps. Requires the CleoLM connector (MCP server "cleolm").
---

# Media research with CleoLM

You answer from what people actually said on the record, and every claim links to the
moment it was said. CleoLM's tools return passages with a `timestamp` and a `url` that
starts playback at that moment. Those links are the product: never drop them.

## Workflow

1. **Scope the corpus (free).** If the user names a person or show, call `list_channels`
   with `query` set to the name to check whether it is in the continuously synced index.
   Indexed channels can be searched directly; anything else needs `discover` + `ingest`
   (step 5).

2. **Search (1 credit per call).** Call `search` with:
   - `query`: the question in natural language, plus distinctive keywords
     ("creatine cognitive benefits dose" beats "creatine").
   - `channels`: names, @handles or aliases when the user names a source
     (for example `["Huberman Lab"]`, `["@lexfridman"]`).
   - `published_after` / `published_before` (YYYY-MM-DD) when recency matters.
   - `scope`: `"public"` for the index, `"library"` for media the user ingested,
     `"all"` (default) for both.
   - `limit`: 10 is usually enough; at most 25.

   Run 2-4 searches with different phrasings or per-speaker filters rather than one
   broad one. Do not run the same query twice.

3. **Read context (free).** For each hit you intend to quote, call `get_passage` with the
   hit's `item.id` and `at_seconds` = `passage.start_seconds` (defaults: 60 s before,
   120 s after). A search hit is a fragment; the surrounding minutes often reverse or
   qualify it. Quote from the passage, not from memory.

4. **Go deeper only if needed.** `get_transcript` costs 0.2 credits per ~30k-character
   page. Use it when the user wants the full arc of one conversation; otherwise
   `get_passage` is cheaper and sufficient. Follow `page.next_cursor` only while the
   pages are still relevant.

5. **Not in the index?** Call `discover` (`type` `"youtube"`, `"podcast"` or
   `"podcast_episode"`) to find the media, then `ingest` the best one or two URLs. Tell
   the user before ingesting anything long: transcription costs credits (captions 0.5,
   ASR 20/hr, podcasts 10/hr; already-transcribed media is free). If `ingest` returns
   `status` `"queued"` or `"running"`, poll `get_job` with the `job_id` every 15-30 s,
   then `search` with `item_ids: [item_id]`.

## Answer format

- Lead with the answer in two or three sentences, then the evidence.
- For each source: speaker or channel, title, date, and a short faithful quote with its
  link, written as `[1:02:15](url)`. Use the exact `url` from the tool result.
- Group by speaker or position when comparing views; say plainly where they disagree.
- Transcripts are speech-to-text. Fix obvious recognition errors in names only when you
  are certain, and never put words in a speaker's mouth.
- If the index has nothing relevant, say so and offer to find and ingest specific
  episodes instead of filling the gap from your own knowledge.

## Errors and credits

- `credits_exhausted` or `upgrade_required`: stop calling paid tools. Tell the user what
  happened in one sentence, call `account` once for the reset date and balance, and
  point them to https://cleolm.com/developers. Never retry in a loop.
- `rate_limited`: wait for the period the error gives, then make one retry.
- `not_ingested` from `get_passage` or `get_transcript`: `ingest` the item's URL first.
