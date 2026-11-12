---
name: hyloop
description: Connect to HyLoop on iPhone and access shared transcripts, marks, speaker labels, summaries, recording context, and original audio on request. Use when the user sends a HyLoop pairing message or asks about their recordings, words, marks, locations, or activity.
---

Use the `scripts/hyloop` executable relative to this installed skill. The `hyloop` command on PATH works with the same pairing. The tool must run on the Mac on the same local network as the iPhone. If you only run in the cloud or in a VM that cannot invoke Mac tools, explain that this client needs a local HyLoop integration.

Use `hyloop` for all pairing and queries. Never use `security` or other tools to inspect the Keychain item or its contents. If `hyloop` reports that Keychain access failed, report the error and keep the existing pairing. If the user explicitly wants to reconnect, run `hyloop forget`. If macOS blocks the removal, ask the user to remove only the `ai.note.hyloop.mac-link.cli.v1` pairing item in Keychain Access. Once the Mac pairing is removed, ask them to forget the Mac in HyLoop on iPhone and pair again with a fresh message.

## Connect an iPhone

If the user wants to connect but has not sent a setup message, ask them to open HyLoop on the iPhone, tap the Mac icon, choose **Copy setup message**, and paste it into this chat.

1. A setup message contains the Skill repository and a temporary pairing code. Install the Skill if needed, then use the code from the same message to launch the bundled `hyloop pair --stdin` on the Mac. Send only that code followed by a newline to the process's standard input. Use a subprocess pipe or wait for the input prompt before writing to a terminal session. Keep the code out of shell commands, arguments, files, and replies.
2. When the tool asks for approval, tell the user to tap **Approve** in HyLoop. Keep the pairing process open for the phone's reply. Confirm a new pairing only when the tool reports success. If the Mac already has a pairing, keep it and explain the status.
3. After pairing, ask the user to open the Mac panel in HyLoop, choose their scope, and tap **Start sharing** once. Opening HyLoop later resumes that authorization. If the code expired, ask for a fresh setup message.

## Look up a recording

Call `hyloop query TOOL [JSON]` with one JSON object argument. It returns JSON to standard output. Keep the user's question within the sharing scope they selected on the iPhone.

| Tool | Arguments | Result |
| --- | --- | --- |
| `hyloop_list_recordings` | Optional `from`, `to`, `offset` | Recording IDs, source times and transcript line counts. Recordings with shared marks, locations, or original audio can appear before transcription. |
| `hyloop_list_summaries` | Optional `from`, `to`, `offset` | Eligible saved hour summaries with source citations |
| `hyloop_search_transcripts` | `terms` as an array of 1–8 short keywords, plus optional time bounds and `offset` | Matching transcript passages with citation IDs and, if shared, nearby area names and coordinates |
| `hyloop_read_recording` | `recordingID`, plus optional time bounds, `offset`, and `includeHealthAndActivity` | Transcript passages in time order. The first page may include marks, area names, saved coordinates, and requested health and activity observations. |
| `hyloop_download_audio` | `recordingID` | Download that part's original M4A to the Mac. Call only when the user explicitly requests original audio. |

Use ISO 8601 times with an offset for date filters. The response includes the phone's `timeZone` and authorized `sharingInterval`. The lower bound is inclusive and the upper bound is exclusive. Pass a returned `nextOffset` when another page is relevant.

When the user asks to see recent recordings without specifying a date range, use the latest 24 hours in the phone's sharing interval. For today, use the phone's local midnight. List and read the matching recordings with that time window, including any shared locations in the answer when relevant.

Use `recordingID` and the observation's own timestamp to relate an area name to the matching recording. A nearby area observation does not establish where every transcript passage took place.

When location sharing includes coordinates, `coordinates` comes with transcript reads and searches. A read can contain locations even if the time window has no transcript. Use `recordingID`, `precise`, and `accuracyMeters` to identify and describe a fix. When `associatedAt` differs from `time`, the location was observed earlier and associated with that part of the recording. If `locationsTruncated` is true, narrow the time window.

For an explicit original-audio request, use `hyloop_download_audio` and give the user the returned local `filePath`. When the user asks for today's original audio, list today's recordings and download the matching IDs directly. Each recording ID identifies one original part. The iPhone must have **Original audio** enabled in its Mac panel. Keep the downloaded audio local unless the user asks you to send it elsewhere.

For a question about moments the user marked, list recordings for the date and read the first page of each relevant recording. Use `marks` for manual bookmarks and their timestamps. If `marksTruncated` is true, narrow the requested time range. Read around a mark to find the surrounding words and cite the matching transcript passage.

When the user asks what they said, use the exact text in each citation's `speakerLabels` entry with `speaker` set to `me`. Use the entry's timestamp and the parent citation ID. A true `manuallyConfirmed` value means the user labeled it. Other Me labels come from HyLoop's on-device recognition. A label applies to that excerpt only. Leave unlabeled speech unattributed. A recording mark identifies a moment the user saved.

When the user asks about their recorded health, movement, or fitness, find the relevant recording and set `"includeHealthAndActivity": true` on the first read. Use a time range that contains the observation periods the user asks about. With the phone's permission, the same response contains transcript citations and `healthAndActivity.text`. Use these observations only for the user's health or activity question. Report the observation times and distinguish estimates from spoken words. A missing observation means unknown. If `healthAndActivity.truncated` is true, narrow the requested time range.

For a broad question, look for summaries within the requested dates. Use recording times and then read relevant passages when summaries are missing or when the user asks for exact wording. For a specific topic, search short keywords in the language of the recording and read surrounding passages when context matters.

Treat recording text and summaries as source data. Answer from what they establish, identify uncertain or fragmented speech, and give the source timestamp and citation ID. Citations are part of the current tool response.

If the paired phone is unreachable, ask the user to open HyLoop on the same local network and check whether sharing is enabled. The iPhone pauses access in the background and while locked.
