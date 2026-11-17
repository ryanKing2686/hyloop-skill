---
name: hyloop
description: Connect to HyLoop on iPhone and access shared transcripts, marks, speaker labels, summaries, recording context, and original audio on request. Use when the user sends a HyLoop pairing message or asks about their recordings, words, marks, locations, or activity.
---

Use the `scripts/hyloop` executable relative to this installed skill. The `hyloop` command on PATH works with the same pairing. The tool must run on the Mac on the same local network as the iPhone. If you only run in the cloud or in a VM that cannot invoke Mac tools, explain that this client needs a local HyLoop integration.

Use `hyloop` for all pairing and queries. Never use `security` or other tools to inspect the Keychain item or its contents. If `hyloop` reports that Keychain access failed, report the error and keep the existing pairing. If the user explicitly wants to reconnect, run `hyloop forget`. If macOS blocks the removal, ask the user to remove only the `ai.note.hyloop.mac-link.cli.v1` pairing item in Keychain Access. Once the Mac pairing is removed, ask them to forget the Mac in HyLoop on iPhone and run `hyloop connect` again.

## Install and connect an iPhone

For an installation request, use [references/installation.md](references/installation.md). Verify the installed executable with `scripts/hyloop doctor`: four-digit phone approval requires connector 0.2.2+, `stagedConnection: true`, `phoneApprovalPairing: true`, and `pairingProtocol: 3`, plus the matching updated iPhone app. Installation alone does not prove phone connectivity.

Guide the user one step at a time, in their language:

1. **Install and check.** Install the complete Skill in this host's supported location and run `doctor`. Explain any real OS permission prompt when it occurs. If the host cannot execute tools on the Mac, say so and guide the user to a local-capable assistant; do not claim installation or pairing succeeded.
2. **Open the phone.** Tell the user: “Open HyLoop on iPhone, tap Mac, and keep that page open. Connect the iPhone and Mac to the same Wi-Fi.” Run the installed `scripts/hyloop connect` (or `connect --device ID` after an explicit multi-phone choice). It returns JSON with `state: awaitingDisplay`, a four-character `code`, and a public `session` ID, then exits BEFORE phone approval. It takes no password or code argument. Do not chain this command with a wait command or start duplicate attempts. An `alreadyPaired`, `failed`, or `expired` result needs its reported action, not a code display.
3. **Compare four digits.** After the first command returns `awaitingDisplay`, immediately send a visible message containing the exact `code` digits in the chat as a Markdown level-three heading (`### ` followed by the actual digits). Keep the digits alone on that line: no label, punctuation, code block, extra spacing or other text. Preserve leading zeros. This gives a medium-size, easy-to-read number in clients that support Markdown headings. On the next paragraph, say in the user's language: “Check that your iPhone shows the same four digits, review the sharing scope, then tap **Allow connection and sharing**. If they differ, tap **Decline**. No typing needed.” Use only actual tool output; never invent sample digits. Do not open a browser or create a webpage, image or QR code for this step. These digits are public verification information, not a password. Never compare or approve on the user's behalf, ask them to send the digits/screenshot back, or treat a device name as verification.
4. **Then wait.** Only AFTER sending the visible code and guidance, call `scripts/hyloop connect --wait SESSION` with the returned session ID. This starts phone approval and waits up to 20 seconds per call. If its state is `waiting`, repeat **the same wait command** until a terminal result; do not run `connect` again. Never batch the first call and wait into one tool execution: buffered tool output would hide the digits until too late. Do not claim to be waiting for approval before you have shown the digits. If the host cannot show an intermediate message, send the code in a normal reply, then resume waiting on the next turn while it remains valid. **Confirm the result** from `connected`, `pairingSaved`, `alreadyPaired`, `failed`, or `expired`, not from the process exit code alone. Distinguish **pairing saved** from **sharing service answered**. If already paired, preserve it; ask the user to enable sharing on iPhone if stopped. Do not erase keys or reconnect merely because the phone is temporarily unreachable.

Handle failures with a specific next step:

| Result | Guidance |
| --- | --- |
| No phone found | Keep the Mac page open in HyLoop, check the same Wi-Fi and Local Network permission on both devices, and verify that the iPhone app supports this flow. Then retry. |
| Multiple phones | Run `scripts/hyloop discover`; have the user expand **Connection options** in the top Mac status card and identify the **This iPhone** label (`HyLoop · XXXX`), then use `connect --device ID`. This label identifies a discovery window; it is not the four-digit connection check. Never pick the first phone arbitrarily. |
| Digits differ / declined | Stop that attempt. Tell the user to decline on iPhone. Only when they want another attempt, have them tap **Allow new connection**, then run `connect` again and compare the new digits. Never retry automatically to obtain a match. |
| Expired / busy | Review or cancel the pending request on iPhone. Tap **Allow new connection** before retrying. A check lasts at most two minutes; the fixed installation message never expires and need not be copied again. |
| Keychain could not save | Follow the command's specific recovery instruction. Preserve existing pairings; do not inspect secrets or claim success. If iPhone approved but this Mac never saved, remove only that incomplete phone connection before a fresh attempt. |
| Old phone / incompatible protocol | Update both the connector and iPhone app for four-digit approval. Updating the Skill alone is insufficient; do not silently switch to the old invitation workflow. |

Legacy `pair --stdin` is retained only for users explicitly connecting an older phone with an existing temporary invitation. Keep a legacy code out of shell arguments, files and replies. Use a separate process stdin channel, or have the user run `scripts/hyloop pair` in their local Terminal and paste it into the hidden prompt. Do not ask permission to embed the secret in a command as a workaround. New setup messages contain no code.

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
